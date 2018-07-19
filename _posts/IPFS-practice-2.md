---
title: IPFS实践之功能封装

date: 2018-06-07 11:05:24

thumbnail: http://210.73.213.194:8080/ipfs/QmYt32dq2xp7TF8BbHs62bMi1d6ai38TEiXsF1WvCsayCE
categories:
- 技术

tags: 
- IPFS
- c-sharp
---
![](\res\ipfs-2\IMG_1817.JPG)
在[IPFS实践之初体验](\2018\06\06\IPFS-practice-1\index.html)中以命令行的方式演示了如何使用IPFS的基础命令。然而，如果基于IPFS做二次开发应用程序，就需要对这些功能进行封装。本文介绍两种调用IPFS功能的方式。

#### Cmd
以C#为例，采用启动cmd进程的方式调用命令行,命令行执行程序如下：
```csharp
public const int WAIT_FOR_EXIT = 2 * 60 * 000;
public const int CRASH_EXIT_CODE = 0xFFFF;
public static int RunExe(string exeFileName, string args, out string output)
{
    output = "";
    int iCode = CRASH_EXIT_CODE;
    if (exeFileName == null)
    {
        return CRASH_EXIT_CODE;
    }
    ProcessStartInfo procInfo = new ProcessStartInfo
    {
        FileName = exeFileName,
        Arguments = args,
        RedirectStandardOutput = true,
        CreateNoWindow = true,
        UseShellExecute = false,
    };
    Process exeProc = null;
    StreamReader reader = null;
    try
    {
        exeProc = Process.Start(procInfo);
        if (exeProc == null)
        {
            return CRASH_EXIT_CODE;
        }
        if (exeProc.StandardOutput != null && exeProc.StandardOutput.BaseStream != null)
        {
            output = exeProc.StandardOutput.ReadToEnd();
        }
        if (exeProc != null)
        {
            if (exeProc.WaitForExit(WAIT_FOR_EXIT))
            {
                iCode = exeProc.ExitCode;
            }
            int cnt = 0;
            while (!exeProc.HasExited && cnt++ < 15)
            {
                Thread.Sleep(5);
            }
            exeProc.Close();
        }
    }
    catch (Exception e)
    {
        iCode = CRASH_EXIT_CODE;
    }
    finally
    {
        DisposeResource(reader);
    }
    return iCode;
}    
```

有了以上的封装，那么执行ipfs add的调用代码可以是

```csharp
string args = string.Format("add {0}", filepath)
ScriptTool.RunExe("ipfs", args, out output);
```

这样的方式，有几种弊端：

- 由于每执行一条命令都需要启动一个进程，而这些进程是互斥的，在IPFS repo下，有一个repo_lock,导致同时命令多条执行的失败率很高。
- 由于命令行执行时，执行完成的输出都是字符串，处理起来比较麻烦。

#### Http Api
之前提过，官方提供的go-ipfs实现的demo中提供了http api的功能，也就是说所有的IPFS功能都可以通过http请求的方式来实现，http接口详见[http API docs](https://ipfs.io/docs/api/)
以http api的方式执行，可以完美解决cmd的问题：
- 它无需另起进程，可以使用多线程，同时执行多条指令
- api的返回是一个json string，方便解析我们所需要的结果参数。

还是以ipfs add为例，接口说明如下：
> /api/v0/add
Add a file or directory to ipfs.
_Arguments_
arg [file]: The path to a file to be added to ipfs. Required: yes.
recursive [bool]: Add directory paths recursively. Default: “false”. Required: no.
quiet [bool]: Write minimal output. Required: no.
quieter [bool]: Write only final hash. Required: no.
silent [bool]: Write no output. Required: no.
progress [bool]: Stream progress data. Required: no.
trickle [bool]: Use trickle-dag format for dag generation. Required: no.
only-hash [bool]: Only chunk and hash - do not write to disk. Required: no.
wrap-with-directory [bool]: Wrap files with a directory object. Required: no.
hidden [bool]: Include files that are hidden. Only takes effect on recursive add. Required: no.
chunker [string]: Chunking algorithm to use. Required: no.
pin [bool]: Pin this object when adding. Default: “true”. Required: no.
raw-leaves [bool]: Use raw blocks for leaf nodes. (experimental). Required: no.
nocopy [bool]: Add the file using filestore. (experimental). Required: no.
fscache [bool]: Check the filestore for pre-existing blocks. (experimental). Required: no.
cid-version [int]: Cid version. Non-zero value will change default of ‘raw-leaves’ to true. (experimental). Default: “0”. Required: no.
hash [string]: Hash function to use. Will set Cid version to 1 if used. (experimental). Default: “sha2-256”. Required: no.
_Request Body_
Argument “path” is of file type. This endpoint expects a file in the body of the request as ‘multipart/form-data’.
_Response_
On success, the call to this endpoint will return with 200 and the following body:
{
    "Name": "[string]"
    "Hash": "[string]"
    "Bytes": "[int64]"
    "Size": "[string]"
}
>

所以add一个文件,需要用post的方式，通过mutipart/form-data格式提交文件内容
这个请求的url：
<http://localhost:5001/api/v0/add?recursive=false>
其中上传文件夹的时候，recursive参数需要设置为true。

Mutipart上传代码如下
```csharp
public static string HttpMultiPartPost(string url, int timeOut, string path, bool isDir)
{
    string responseContent;
    var webRequest = (HttpWebRequest)WebRequest.Create(url);
    // 边界符  
    var boundary = "---------------" + DateTime.Now.Ticks.ToString("x");
    // 边界符  
    var beginBoundary = Encoding.ASCII.GetBytes("\r\n" + "--" + boundary + "\r\n");
    // 最后的结束符  
    var endBoundary = Encoding.ASCII.GetBytes("\r\n" + "--" + boundary + "--\r\n");
    // 设置属性  
    webRequest.Method = "POST";
    webRequest.Timeout = timeOut;
    webRequest.ContentType = "multipart/form-data; boundary=" + boundary;
    var requestStream = webRequest.GetRequestStream();
    if (isDir)
    {
        //发送分割符
        requestStream.Write(beginBoundary, 0, beginBoundary.Length);
        //发送文件夹头
        const string folderHeaderFormat =
        "Content-Disposition: file; filename=\"{0}\"\r\n" +
        "Content-Type: application/x-directory\r\n\r\n";
        var folderHeader = string.Format(folderHeaderFormat, GetDirectoryName(path));
        var headerbytes = Encoding.UTF8.GetBytes(folderHeader);
        requestStream.Write(headerbytes, 0, headerbytes.Length);
        DirectoryInfo directory = new DirectoryInfo(path);
        FileInfo[] fileInfo = directory.GetFiles();
        foreach (FileInfo item in fileInfo)
        {
            PostFileItem(requestStream, beginBoundary, item, GetDirectoryName(path));
        }
    }
    else
    {
        PostFileItem(requestStream, beginBoundary, new FileInfo(path), null);
    }
    // 写入最后的结束边界符  
    requestStream.Write(endBoundary, 0, endBoundary.Length);
    requestStream.Close();
    var httpWebResponse = (HttpWebResponse)webRequest.GetResponse();
    using (var httpStreamReader = new StreamReader(httpWebResponse.GetResponseStream(),Encoding.GetEncoding("utf-8")))
    {
        responseContent = httpStreamReader.ReadToEnd();
    }
    httpWebResponse.Close();
    webRequest.Abort();
    return responseContent;
}

private static void PostFileItem(Stream stream, byte[] boundary, FileInfo fileInfo, string dirName)
{
    string name = fileInfo.Name;
    if (!string.IsNullOrEmpty(dirName))
    {
        name = dirName + "/" + name;
    }
    //发送分割符
    stream.Write(boundary, 0, boundary.Length);
    //发送文件头
    const string headerFormat =
                    "Abspath: {0}\r\n" +
                    "Content-Disposition: file; filename=\"{1}\"\r\n" +
                    "Content-Type: application/octet-stream\r\n\r\n";
    var header = string.Format(headerFormat, fileInfo.FullName, name);
    var headerbytes = Encoding.UTF8.GetBytes(header);
    stream.Write(headerbytes, 0, headerbytes.Length);
    //发送文件流
    var fileStream = new FileStream(fileInfo.FullName, FileMode.Open, FileAccess.Read);
    var buffer = new byte[1024];
    int bytesRead; // =0  
    while ((bytesRead = fileStream.Read(buffer, 0, buffer.Length)) != 0)
    {
        stream.Write(buffer, 0, bytesRead);
    }
    fileStream.Close();
}
```

add Api调用代码如下：

```csharp
public static string Add(string path, bool isDir)
{
    if (string.IsNullOrEmpty(path))
    {
                return null;
    }
    string format = "";
    if (isDir)
    {
        format = "{0}?chunker=size-262144&recursive=true";
    }
    else
    {
        //如果需要保存名字，需要wrap一个dir
        //format = "{0}?chunker=size-262144&recursive=false&wrap-with-directory=true";
        format = "{0}?chunker=size-262144&recursive=false";
    }
    string url = string.Format(format, "http://localhost:5001/api/v0/add");
    string output = HttpMultiPartPost(url, 500000, path, isDir);
    return output;
}
```

大部分的api都是以get的方式请求，相比add简单许多，这里提供一个C#的httpget方法供参考.

```csharp
public static string HttpGet(string url, int timeOut)
{
    string responseContent;
    var webRequest = (HttpWebRequest)WebRequest.Create(url);
    webRequest.Method = "GET";
    webRequest.Timeout = timeOut;
    var httpWebResponse = (HttpWebResponse)webRequest.GetResponse();
    using (var httpStreamReader = new StreamReader(httpWebResponse.GetResponseStream(), Encoding.GetEncoding("utf-8")))
    {
        responseContent = httpStreamReader.ReadToEnd();
    }
    httpWebResponse.Close();
    webRequest.Abort();
    return responseContent;
}

```

如name publish这样的操作就可以这样调用

```csharp
public static string NamePublish(string hash, int hours)
{
    //lifetime失效期
    string format = "{0}?arg={1}&lifetime={2}h";
    string url = string.Format(format, @"http://localhost:5001/api/v0/name/publish", hash, hours);
    return HttpTool.HttpGet(url, 100000);
}
```
封装了这些操作，我们就可以将IPFS用起来，做一些我们所需要的application。
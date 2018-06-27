---
title: 简单合同管理程序开发记录
date: 2018-06-25
categories:
- 项目
thumbnail: http://210.73.213.194:8080/ipfs/QmcJzUnLgFGNKq3rpxpXREctpuNAhGeygqZNfHx8rzTzge
tags: 
- c-sharp
- wpf
---

最近一个朋友表示，他们公司的合同用Excel管理，多了之后变得很麻烦，让我帮他做个合同管理的小工具，利用最近的闲暇时间，用WPF写了一个小工具。

#### 需求
微信上简单的聊天，把他一堆不太具象的想法归纳起来，罗列需求，化成思维导图比较清晰了。

![](http://210.73.213.194:8080/ipfs/QmcJzUnLgFGNKq3rpxpXREctpuNAhGeygqZNfHx8rzTzge)

#### 开发环境
由于朋友的电脑都使用的windows，所以采用vs2017 + WPF的开发桌面应用是比较方便的选择。

#### 模块划分
- UI

    采用Blend 2017画一个简单的UI图，然后在稍修改xaml文件，UI便可成型

- 数据模型
  由于WPF采用MVVM的模式，数据模型只需实现INotifyPropertyChanged接口即可
```csharp
namespace SimpleCM.Data
{
    public class Contract: INotifyPropertyChanged
    {
        public long Id { get; set; }
        private string _contractNumber;

        public string ContractNumber
        {
            set
            {
                _contractNumber = value;
                OnPropertyChanged("ContractNumber");
            }

            get => _contractNumber;
        }
        private string _projectCategory;
        public string ProjectCategory
        {
            get => _projectCategory;
            set
            {
                _projectCategory = value;
                OnPropertyChanged("ProjectCategory");
            }
        }

        private string _contractName;
        public string ContractName
        {
            set
            {
                _contractName = value;
                OnPropertyChanged("ContractName");
            }
            get => _contractName;
        }

        private long _contractDate;
        public long ContractDate
        {
            set
            {
                _contractDate = value;
                OnPropertyChanged("ContractDate");
            }
            get => _contractDate;
        }

        private string _projectDescription;
        public string ProjectDescription
        {
            set
            {
                _projectDescription = value;
                OnPropertyChanged("ProjectDescription");
            }
            get => _projectDescription;
        }

        private string _contractCompany;
        public string ContractCompany
        {
            set
            {
                _contractCompany = value;
                OnPropertyChanged("ContractCompany");
            }
            get => _contractCompany;
        }

        private string _cooperatorCompany;
        public string CooperatorCompany
        {
            set
            {
                _cooperatorCompany = value;
                OnPropertyChanged("CooperatorCompany");
            }
            get => _cooperatorCompany;
        }

        private long _cost;
        public long Cost
        {
            set
            {
                _cost = value;
                OnPropertyChanged("Cost");
            }
            get => _cost;
        }

        private long _peceivables_1;
        public long Peceivables_1
        {
            get => _peceivables_1;
            set
            {
                _peceivables_1 = value;
                OnPropertyChanged("Peceivables_1");
            }
        }

        private long _peceivables_2;
        public long Peceivables_2
        {
            get => _peceivables_2;
            set
            {
                _peceivables_2 = value;
                OnPropertyChanged("Peceivables_2");
            }
        }

        private long _peceivables_3;
        public long Peceivables_3
        {
            get => _peceivables_3;
            set
            {
                _peceivables_3 = value;
                OnPropertyChanged("Peceivables_3");
            }
        }

        private string _baseInfo;
        readonly string format = "项目编号:{0}\n项目名称:{1}\n项目类别:{2}\n项目日期:{3}\n" +
            "签约单位:{4}\n合作单位:{5}\n项目描述:{6}\n总费用:{7}\n" +
            "应付款项一:{8}\n应付款项二:{9}\n应付款项三{10}"; 
        public string BaseInfo
        {
            get
            {
                DateTime datetime = Util.GetTimeFromMillis(ContractDate);
                string dateInfo = string.Format("{0}年{1}月{2}日", datetime.Year, datetime.Month, datetime.Day);
                return string.Format(format, ContractNumber, ContractName, ProjectCategory,
                    dateInfo, ContractCompany, CooperatorCompany, ProjectDescription, Cost, Peceivables_1,Peceivables_2, Peceivables_3);
            }
            set
            {
                _baseInfo = value;
                OnPropertyChanged("BaseInfo");
            }
        }

        private string _aditionals;
        public string Aditionals
        {
            get => _aditionals;
            set
            {
                _aditionals = value;
                OnPropertyChanged("Aditionals");
            }
        }

        public ObservableCollection<string> AddtionList
        {
            get
            {
                ObservableCollection<string> list = new ObservableCollection<string>();
                if (!string.IsNullOrEmpty(_aditionals))
                {
                    string[] infos = _aditionals.Split(' ');
                    if (infos != null && infos.Length > 0)
                    {
                        for (int i = 0; i < infos.Length; i++)
                        {
                            list.Add(infos[i]);
                        }
                    }
                }
                return list;
            }
        }


        public string BillNoteTostring()
        {
            //不能加@,否则原样输出\r\n,不换行
            string format = "公司名称:{0}\n纳税人识别号:{1}\n地址:{2}\n电话:{3}\n银行:{4}\n账号:{5}";
            return string.Format(format, BillNoteCompanyName, BillNoteTaxFileNum,
                BillNoteAddress, BillNotePhoneNumber, BillNoteBank, BillNoteBankAccount);
        }

        public void FromBillNoteString()
        {

            string[] noteItems = _billNoteInfo.Split('\n');
            if (noteItems != null && noteItems.Length == 6)
            {
                BillNoteCompanyName = noteItems[0].Substring(noteItems[0].IndexOf(':') + 1);
                BillNoteTaxFileNum = noteItems[1].Substring(noteItems[1].IndexOf(':') + 1);
                BillNoteAddress = noteItems[2].Substring(noteItems[2].IndexOf(':') + 1);
                BillNotePhoneNumber = noteItems[3].Substring(noteItems[3].IndexOf(':') + 1);
                BillNoteBank = noteItems[4].Substring(noteItems[4].IndexOf(':') + 1);
                BillNoteBankAccount = noteItems[5].Substring(noteItems[5].IndexOf(':') + 1);
            }
        }

        private string _billNoteInfo;
        public string BillNoteInfo
        {
            get => _billNoteInfo;
            set
            {
                _billNoteInfo = value;
                OnPropertyChanged("BillNoteInfo");
            }
        }

        private string _billNoteCompanyName;
        public string BillNoteCompanyName
        {
            get
            {
                return _billNoteCompanyName;
            }
            set
            {
                _billNoteCompanyName = value;
                OnPropertyChanged("BillNoteCompanyName");
            }
        }
        private string _billNoteTaxFileNum;
        public string BillNoteTaxFileNum
        {
            get
            {
                return _billNoteTaxFileNum;
            }
            set
            {
                _billNoteTaxFileNum = value;
                OnPropertyChanged("BillNoteTaxFileNum");
            }
        }

        private string _billNoteAddress;
        public string BillNoteAddress
        {
            get
            {
                return _billNoteAddress;
            }
            set
            {
                _billNoteAddress = value;
                OnPropertyChanged("BillNoteAddress");
            }
        }

        private string _billNotePhoneNumber;
        public string BillNotePhoneNumber
        {
            get
            {
                return _billNotePhoneNumber;
            }
            set
            {
                _billNotePhoneNumber = value;
                OnPropertyChanged("BillNotePhoneNumber");
            }
        }

        private string _billNoteBank;
        public string BillNoteBank
        {
            get
            {
                return _billNoteBank;
            }
            set
            {
                _billNoteBank = value;
                OnPropertyChanged("BillNoteBank");
            }
        }

        private string _billNoteBankAccount;
        public string BillNoteBankAccount
        {
            get
            {
                return _billNoteBankAccount;
            }
            set
            {
                _billNoteBankAccount = value;
                OnPropertyChanged("BillNoteBankAccount");
            }
        }



        public override string ToString() => ContractName;

        public event PropertyChangedEventHandler PropertyChanged;
        protected void OnPropertyChanged(string info)
        {
            var handler = PropertyChanged;
            handler?.Invoke(this, new PropertyChangedEventArgs(info));

        }
    }
}

```

- 持久化存储
采用sqlite3进行存储

- 数据绑定

```xml
<ListBox x:Name="contract_list_box" Grid.Row="1" HorizontalAlignment="Left" Width="250" Margin="20" IsSynchronizedWithCurrentItem="True"
                        ItemsSource="{Binding Source={x:Static util:Contracts.Instance}}">
```

```csharp
using System.Collections.ObjectModel;

namespace SimpleCM.Data
{
    class Contracts: ObservableCollection<Contract>
    {
        public static Contracts Instance { get; } = new Contracts();
        public void AddContact(Contract c)
        {
            Add(c);
            ContractDB.Instance.InsertItem(c);
        }
    }
}
```

#### 代码

[Github链接](https://github.com/vandep/simplecm)
---
title: 获取网络文件内容，http、https通用
date: 2020-01-04 18:32:55
tags: CSharp
---

## 准备

文件最开始引入`using System.Net`。

## 代码

```c#
try
{
    using (WebClient wc = new WebClient())
    {
        //下面这句是解析https的必备，http不用
        ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12 | SecurityProtocolType.Tls11 | SecurityProtocolType.Tls;
        //调整编码
        wc.Encoding = Encoding.UTF8;
        string txt = wc.DownloadString(url);
    }
}
catch
{
    MessageBox.Show("解析错误");
}

```


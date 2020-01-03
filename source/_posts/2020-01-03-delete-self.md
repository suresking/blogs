---
title: C#程序删除自己
date: 2020-01-03 18:48:40
tags: CSharp
---

代码如下：

```c#
ProcessStartInfo psi = new ProcessStartInfo("cmd.exe", "/C ping 1.1.1.1 -n 1 -w 1000 > Nul & Del " + Application.ExecutablePath);
psi.WindowStyle = ProcessWindowStyle.Hidden;
psi.CreateNoWindow = true;
Process.Start(psi);
Application.Exit();
System.Environment.Exit(0);
```


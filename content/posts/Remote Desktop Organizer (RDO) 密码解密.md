+++
title = "Remote Desktop Organizer (RDO) 密码解密"
author = "Hzllaga"
date = 2020-04-23
categories = [ "原创" ]
tags = ["笔记","逆向",".NET"]
+++
前段时间渗透过程发现目标某个服务器上用RDO连接了多个rdp，而这软件的配置文件是以二进制方式保存的，网上也找不到公开的解密方法，就开始逆向这软件。

软件是.NET写的，用ConfuserEx混淆代码了，先用网上的工具反混淆，没办法处理的很干净，至少变量名字都还原了，花了点时间看代码找到关键的解密方法：
{{< highlight Csharp >}}
        public static byte[] smethod_4(byte[] byte_0, byte[] byte_1, byte[] byte_2)
        {
            MemoryStream memoryStream = new MemoryStream();
            Rijndael rijndael = Rijndael.Create();
            rijndael.Key = byte_1;
            rijndael.IV = byte_2;
            CryptoStream cryptoStream = new CryptoStream(memoryStream, rijndael.CreateDecryptor(), CryptoStreamMode.Write);
            cryptoStream.Write(byte_0, 0, byte_0.Length);
            cryptoStream.Close();
            return memoryStream.ToArray();
        }

        public static string smethod_5(string string_0)
        {
            string strPassword = "swevA2t62We?5Cr+he4Tac?_E!redafa?re5+2huv*$rU9eS8Ub4?W!!R+s7uthU";
            byte[] array = new byte[string_0.Length / 2];
            for (int i = 0; i < string_0.Length / 2; i++)
            {
                int num = Convert.ToInt32(string_0.Substring(i * 2, 2), 16);
                array[i] = (byte)num;
            }
            PasswordDeriveBytes passwordDeriveBytes = new PasswordDeriveBytes(strPassword, new byte[]
            {
                73,
                118,
                97,
                110,
                32,
                77,
                101,
                100,
                118,
                101,
                100,
                101,
                118
            });
            byte[] bytes = smethod_4(array, passwordDeriveBytes.GetBytes(32), passwordDeriveBytes.GetBytes(16));
            return Encoding.Unicode.GetString(bytes);
        }
{{< /highlight >}}

由于时间有限，稍微根据目标情况写了个一键解密的工具，有需求的可以根据目标环境修改代码，或是直接把配置文件的密码调用上面的方法就可以解密了。

相关代码都在GitHub上面：[https://github.com/Hzllaga/RDODecrypt](https://github.com/Hzllaga/RDODecrypt)
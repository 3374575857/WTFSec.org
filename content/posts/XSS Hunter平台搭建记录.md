+++
title = "XSS Hunter平台搭建记录"
author = "Hzllaga"
date =  2020-02-25T17:21:52+08:00
description = ""
categories = ["原创"]
tags = ["XSS","笔记"]
+++

> XSS Hunter：[https://github.com/mandatoryprogrammer/xsshunter/](https://github.com/mandatoryprogrammer/xsshunter/)
>
>The XSS Hunter service - a portable version of XSSHunter.com<!--more-->

## 搭建需求：
+ 服务器最好是Ubuntu
+ 一个Mailgun账号，用来发送xss触发时的邮件，由于该平台并非免费，本文将会改为自建发信服务器
+ 一个域名，越短越好，避免有些地方因字符数量限制无法成功利用
+ 一个通配符SSL证书，可至SSL FOR FREE网站申请

首先设置域名的DNS记录：
```
    A 记录:
        主机名: @
        内容: 服务器IP
    CNAME 记录:
        主机名： *
        内容: 域名
```
![](https://cdn.wtfsec.org/img/20200225205745.png)

再来安装Nginx:
```
sudo apt-get install nginx
```
安装Postgresql：
```
sudo apt-get install postgresql postgresql-contrib
```
设定Postgresql：
```
sudo -i -u postgres  //切换为postgres用户
psql template1  //登录postgresql
CREATE USER xsshunter WITH PASSWORD 'PASSWORD'; //新建账号为xsshunter，密码为PASSWORD的用户
CREATE DATABASE xsshunter;  //新建名为xsshunter的数据库
\q  //退出postgresql
exit  //回到原本的用户
```
下载最新版XSSHunter：
```
git clone https://github.com/mandatoryprogrammer/xsshunter
```
## 配置自己的发信服务器
如果有钱的朋友可以直接买Mailgun服务并跳过这一段，先来看看程序是怎么发邮件的：
{{< highlight python>}}
def send_email( to, subject, body, attachment_file, body_type="html" ):
    if body_type == "html":
        body += "<br /><img src=\"https://api." + settings["domain"] + "/" + attachment_file.encode( "utf-8" ) + "\" />" # I'm so sorry.

    email_data = {
        "from": urllib.quote_plus( settings["email_from"] ),
        "to": urllib.quote_plus( to ),
        "subject": urllib.quote_plus( subject ),
        body_type: urllib.quote_plus( body ),
    }

    thread = unirest.post( "https://api.mailgun.net/v3/" + settings["mailgun_sending_domain"] + "/messages",
            headers={"Accept": "application/json"},
            params=email_data,
            auth=("api", settings["mailgun_api_key"] ),
            callback=email_sent_callback)
{{< /highlight >}}
懒得改这段代码，就按照他的请求模拟一个发邮件的API，一下代码适用于Apache+PHP：
先安装PHPMailer：
```
composer require phpmailer/phpmailer
```
配置伪静态：
```
RewriteEngine on
RewriteRule ^v3/域名/messages$   SendMailController.php?action=send
RewriteCond %{HTTP:Authorization} ^(.*)
RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
```
新建SendMailController.php：
{{< highlight php>}}
<?php

use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;
use PHPMailer\PHPMailer\Exception;

require 'vendor/autoload.php';
$mail = new PHPMailer(true);

if (isset($_SERVER['HTTP_AUTHORIZATION'])) {
    $authkey = "设置一个api密钥，一会平台根据这个密钥接入";
    $auth = base64_decode(str_replace("Basic ", "", $_SERVER['HTTP_AUTHORIZATION']));
    if ($auth == "api:" . $authkey) {
        @$action = $_GET['action'];
        if (isset($action)) { 
            if ($action == "send") { 
                @$from = urldecode($_POST['from']);
                @$to = urldecode($_POST['to']);
                @$subject = urldecode($_POST['subject']);
                @$html = urldecode($_POST['html']);
                @$text = urldecode($_POST['text']);
                if ($html != ""){
                    $mail->isHTML(true);
                    $mail->Body = $html;
                }else if ($text != ""){
                    $mail->isHTML(false);
                    $mail->Body = $text;
                }
                try {
                    $mail->isSMTP();
                    $mail->Host = 'smtp地址';
                    $mail->SMTPAuth = true;
                    $mail->Username = '账号';
                    $mail->Password = '密码';
                    $mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;
                    $mail->Port = 587;
                    $mail->setFrom('发信地址，也可以用$from', '昵称(选填)');
                    $mail->addAddress($to);
                    $mail->Subject = $subject;
                    $mail->send();
                    echo 'Mail sent to ' . $to . ' status: OK'; //这边会在xss平台打印出来
                } catch (Exception $e) {
                    echo "Message could not be sent. Mailer Error: {$mail->ErrorInfo}";
                }
            }
        }
    } else { 
        header("HTTP/1.1 403 Forbidden");
        exit;
    }
} else { 
    header("HTTP/1.1 401 Unauthorized");
    exit;
}

{{< /highlight >}}
配置完毕后可以测试一下，如果没问题就把程序的`api.mailgun.net`换成你的api域名。
![](https://cdn.wtfsec.org/img/20200225215804.png)

![](https://cdn.wtfsec.org/img/20200225215926.png)

接着搭建XSS平台，运行`generate_config.py`：
```
python generate_config.py
```
根据提示填好配置，密钥就填上面设置的，然后把生成的nginx配置文件移动到正确位置：
```
sudo mv default /etc/nginx/sites-enabled/default
```
接着申请SSL证书，地址：[https://www.sslforfree.com/](https://www.sslforfree.com/)
![](https://cdn.wtfsec.org/img/20200225212555.png)

然后把申请到的SSL证书移动到指定目录：
```
mkdir /etc/nginx/ssl
/etc/nginx/ssl/yourdomain.com.crt; #certificate.crt
/etc/nginx/ssl/yourdomain.com.key; #private.key
```
重启nginx让配置文件生效：
```
sudo service nginx restart
```
避免断开ssh连接就停止运行，使用`tmux`或`screen`来运行，以下使用`tmux`：
```
tmux
sudo apt-get install python-virtualenv python-dev libpq-dev libffi-dev
cd xsshunter/api/
virtualenv env
. env/bin/activate
pip install -r requirements.txt
python apiserver.py
```
运行apiserver后新建一个新的`tmux`会话，运行guiserver：
```
cd xsshunter/gui/
virtualenv env
. env/bin/activate
pip install -r requirements.txt
python guiserver.py
```
当XSS触发的时候，你会发现请求的状态码是500，那是因为他会保存网页图片，而程序默认是没有uploads这个文件夹的，新建一下就好了：
```
mkdir xsshunter/api/uploads
```
至此，自己的XSS Hunter平台已经搭建完毕，并且可以正常接收邮件。
![](https://cdn.wtfsec.org/img/20200225222330.png)

![](https://cdn.wtfsec.org/img/20200225220136.png)

## 下面是对他进行的一些修改：
+ 移除疑似后门的代码：

guiserver.py 21行：`<script src=//y.vg></script>`

apiserver.py 64行：`<script src=//y.vg></script>`
+ 添加验证码（以Google Recaptcha为例）

新增一段验证代码：
{{< highlight python>}}
    def validate_recaptcha( self, recaptcha_response, ip ):
        if recaptcha_response == "":
            self.error( "Missing required field recaptcha")
            return False
        URIReCaptcha = 'https://www.google.com/recaptcha/api/siteverify'
        recaptchaResponse = recaptcha_response
        private_recaptcha = '密钥'
        remote_ip = ip
        params = urllib.urlencode({
        'secret': private_recaptcha,
        'response': recaptchaResponse,
        'remote_ip': remote_ip,
        })
        data = urllib.urlopen(URIReCaptcha, params).read()
        result = json.loads(data)
        success = result.get('success', None)
        if success == False:
            self.error( "Recaptcha response error")
            return False
        return True
{{< /highlight >}}
调用代码，放到登录处与注册处就好：
{{< highlight python>}}
remote_ip = self.request.remote_ip
if not self.validate_recaptcha( user_data["recaptcha_response"], remote_ip ):
    return
{{< /highlight >}}
登录模板与注册模板文件新增：
{{< highlight html>}}
<div class="g-recaptcha" data-sitekey="密钥"></div><br />
<script type="text/javascript" src="https://www.google.com/recaptcha/api.js"></script>
{{< /highlight >}}
登录js文件新增：
{{< highlight javascript>}}
//发送请求处
"recaptcha_response": grecaptcha.getResponse(),
//登陆失败处
grecaptcha.reset();
{{< /highlight >}}
注册js文件新增：
{{< highlight javascript>}}
//发送请求处
USER.recaptcha_response = grecaptcha.getResponse();
//判断返回处
if( $( ".bad_signup_text_fields" ).text().indexOf( "Invalid CAPTCHA" ) > -1 || $( ".bad_signup_text_fields" ).text().indexOf( "Invite code not valid" ) > -1 ) {
    grecaptcha.reset();
}
{{< /highlight >}}
修改CSP，`guiserver.py` 22行：
{{< highlight python>}}
self.set_header("Content-Security-Policy", "default-src 'self' " + DOMAIN + " api." + DOMAIN + "; style-src 'self' fonts.googleapis.com; img-src 'self' api." + DOMAIN + "; font-src 'self' fonts.googleapis.com fonts.gstatic.com; script-src 'self' https://www.google.com/recaptcha/ https://www.gstatic.com/recaptcha/; frame-src 'self' https://www.google.com/recaptcha/")
{{< /highlight >}}
效果：
![](https://cdn.wtfsec.org/img/20200225215455.png)

所有修改过的代码可以到我的Github上看到，链接：[https://github.com/Hzllaga/xsshunter](https://github.com/Hzllaga/xsshunter)
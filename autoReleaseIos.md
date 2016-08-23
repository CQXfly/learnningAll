###learn autp package
* 学习自动打包 
* 打包iOS发不到**fir** 发邮件通知到对方

***

* 配置需要的路径 token
** project_path 需要打包的项目所在的路径
** app_path 打包后生成的。app文件所在的路径
** targetIPA_path 打包完成后的ipa文件所在路径
** fir_api_token 上传到fir的apptoken
** 以及配置你的邮箱信息
```
from_addr = "hanfengaotian@sina.com"
password = ""       
smtp_server = "smtp.sina.com"
to_addr = '871708991@qq.com,905799827@qq.com'
to_addr1 = '657483726.qq.com'
```
上代码 
使用的是python2.7版本进行开发脚本
```
import os
import sys
import time 
import hashlib
from email import encoders
from email.header import Header 
from email.mime.text import MIMEText
from email.utils import parseaddr,formataddr
import smtplib
```

```
project_path = "/Users/chongqingxu/muxing_user"

app_path = "/Users/chongqingxu/muxing_user/build/Release-iphoneos/jupiter.app"

build_path = "build"

targetIPA_path = "/Users/chongqingxu/Desktop"

fir_api_token = "45d5b9d27109a8c54f109ef145e0683c"

from_addr = "hanfengaotian@sina.com"
password = "cqx09143103"       
smtp_server = "smtp.sina.com"
to_addr = '871708991@qq.com,905799827@qq.com'
to_addr1 = '657483726.qq.com'
```

```
def clean_project_mkdir_build():
	os.system('cd %s;xcodebuild clean' % project_path)
	os.system('cd %s;mkdir build ' % project_path)

def build_project():
	print("building")
	os.system('cd %s;xocdebuild -list' % project_path)
	os.system ('cd %s;xcodebuild build' % project_path)
    
def build_ipa():
     	
	global ipa_filename
	ipa_filename = time.strftime('muxing_%Y-%m-%d_%H-%M-%S.ipa',time.localtime(time.time()))
	os.system ('xcrun -sdk iphoneos PackageApplication -v %s -o %s/%       s' % (app_path,targetIPA_path,ipa_filename))

def upload_fir():
    if os.path.exists("%s/%s" % (targetIPA_path,ipa_filename)):
        
        # 直接使用fir 有问题 这里使用了绝对地址 在终端通过 which fir 获得
     	ret = os.system("fir p '%s/%s' -T '%s'" % (targetIPA_path,ipa_filename,fir_api_token))
    else:
        print("没有找到ipa文件")

def _format_addr(s):
    name, addr = parseaddr(s)
    return formataddr((Header(name, 'utf-8').encode(), addr))

def send_mail():
    msg = MIMEText('xxx iOS测试项目已经打包完毕，请前往 http://fir.im/25fy 下载测试！', 'plain', 'utf-8')
    msg['From'] = _format_addr('自动打包系统 <%s>' % from_addr)
    msg['To'] = _format_addr('york测试人员 <%s>' % to_addr)
   
    msg['Subject'] = Header('木星 iOS客户端打包程序', 'utf-8').encode()
    server = smtplib.SMTP(smtp_server, 25)
    server.set_debuglevel(1)
    server.login(from_addr, password)
    server.sendmail(from_addr, [to_addr], msg.as_string())
    server.quit()   

def send_mail1():
    msg = MIMEText('xxx iOS测试项目已经打包完毕，请前往 http://fir.im/25fy 下载测试！', 'plain', 'utf-8')
    msg['From'] = _format_addr('自动打包系统 <%s>' % from_addr)
    msg['To'] = _format_addr('york测试人员 <%s>' % to_addr1)
   
    msg['Subject'] = Header('木星 iOS客户端打包程序', 'utf-8').encode()
    server = smtplib.SMTP(smtp_server, 25)
    server.set_debuglevel(1)
    server.login(from_addr, password)
    server.sendmail(from_addr, [to_addr], msg.as_string())
    server.quit()      

def main():
	clean_project_mkdir_build()
	build_project()
	build_ipa()
	upload_fir()
	send_mail()
	send_mail1()

main()
```

* 说明:打包主要用到了xcode打包命令 资料很多 
* fri-cli的工具需要集成 再fir官网是有说明 有一点安装xcode工具的时候可能会失败 多安装几次即可
* fri的命令行敲是可以的但是python必须要到他的文件夹中使用命令才可以 使用 which fir 即可


*** 
- [ ] 有时候打包的文件测试下载不成功 应该是证书配置的问题 
- [ ] 暂时还不会使用群发邮件.... python 遍历下循环发?
- [ ] 继续学习python

*** 

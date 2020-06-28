### 前言
打 XCTF 的时候场景不能创建，于是[@Xinian](https://community.ixinian.com/u/Xinian) 给我了个 [HackTheBox](https://www.hackthebox.eu/)  
![image.png](https://i.loli.net/2020/04/12/iQ1YFbsLGjpH4e7.png)
找了半天，在最下面找到了Join
![image.png](https://i.loli.net/2020/04/12/UKQbia5lgjp4hRs.png)
结果。。。。
![image.png](https://i.loli.net/2020/04/12/4rKB8HJMGyoqli5.png)
果然大佬玩的就是不一样 😅 
![image.png](https://i.loli.net/2020/04/12/x3zS5sguAdBZikh.png)

### 正文
F12，在Elements看到有个inviteapi.min.js
![image.png](https://i.loli.net/2020/04/12/6E5XDkmZ4ShnJqa.png)
在sources看了下，是个[eval](https://www.w3school.com.cn/js/jsref_eval.asp)，对后面字符串进行处理然后执行
![image.png](https://i.loli.net/2020/04/12/FymkrPoDzLOV2Te.png)
在字符串中找到了makeInviteCode这个函数名,在Console看了下，是用ajax发送post请求到/api/invite/how/to/generate
![image.png](https://i.loli.net/2020/04/12/P9TOcJAxSCR6rXV.png)
![image.png](https://i.loli.net/2020/04/12/9XfrkUNlwjYhAd8.png)
Evaluate in Console
![image.png](https://i.loli.net/2020/04/12/irauDtqxcG287FR.png)
我得到的是Base64编码的，但是@Xinian说他的是ROT13，于是我又试了一遍发现编码格式确实是随机的，但是解码的结果都是一样的
![image.png](https://i.loli.net/2020/04/12/OQHbuNkaVFxg4mr.png)
![image.png](https://i.loli.net/2020/04/12/jdVhQc1LE3Mnamb.png)

_In order to generate the invite code, make a POST request to /api/invite/generate_

按照提示，发送一个post请求到/api/invite/generate
![image.png](https://i.loli.net/2020/04/12/YOknV1WSBTGKzp2.png)
得到data，看到是以‘=’结尾的，多半是Base64，拿去解码，得到Invite Code
![image.png](https://i.loli.net/2020/04/12/eoBfwGSi8MgOK5N.png)

终于可以注册了。。。。
![image.png](https://i.loli.net/2020/04/12/Q5dAKbUBi1aNumG.png)

一个注册就花了一个小时，小白起步磕磕绊绊挺多啊。。。。。。。。。

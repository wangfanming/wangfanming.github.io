# Maven中的依赖传递

Maven

---

## 一、依赖范围scope
>Maven因为执行一系列编译、测试、和部署等操作，在不同的操作下使用的classpath不同，**依赖范围就是控制依赖与三种classpath(编译classpath、测试classpath、运行classpath)的关系**。

一共有5种，compile(编译)、test(测试)、runtime(运行时)、provided、system不指定，则范围默认为compile。

 - compile：编译依赖范围，在编译、测试、运行时都需要。    
 - test：测试依赖范围，测试时需要。编译和运行不需要。
 - runtime：运行时以来范围，测试和运行时需要，编译不需要。
 - provided：已提供依赖范围，编译和测试时需要。运行时不需要。
 - system：系统依赖范围。本地依赖，不在maven中央仓库。
 
--- 
## 二、依赖的传递
A->B(compile)   a依赖b      
    >b是A的编译依赖范围，在A编译、测试、运行时都依赖b

B->C(compile)   B依赖C      
    >c是B的编译依赖范围，在B编译、测试、运行时都依赖c

当在A中配置

    <dependency>  
            <groupId>com.B</groupId>  
            <artifactId>B</artifactId>  
            <version>1.0</version>  
    </dependency>

则会自动导入c包。关系传递如下表：


由上表不难看出，项目A具体会不会导入B依赖的包c，取决于第一和第二依赖，但是依赖的范围不会超过第一直接依赖，即具体会不会引入c包，要看第一直接依赖的依赖范围。

## 三、以来冲突的调节   

    A -> B -> C -> X(1.0)
    A -> D -> X(2.0)
由于只能引入一个版本的包，此时Maven按照最短路径选择导入X(2.0).

    A -> B -> X(1.0)
    A -> D -> X(2.0)
路径长度一致，则优先选择第一个，此时导入X(1.0).

## 四、排除依赖

    A -> B -> C(1.0)

此时在A项目中，不想使用C(1.0)，而使用C(2.0)

则需要使用exclusion排除B对C(1.0)的依赖。并在A中引入C(2.0).

 

    pom.xml中配置
    <!--排除B对C的依赖-->

    <dependency>  
            <groupId>B</groupId>  
            <artifactId>B</artifactId>  
            <version>0.1</version>  
            <exclusions>
                 <exclusion>
                    <groupId>C</groupId>  
                 <artifactId>C</artifactId><!--无需指定要排除项目的                                         版本号-->
                 </exclusion>
            </exclusions>
    </dependency> 

    <!---在A中引入C(2.0)-->
    <dependency>  
            <groupId>C</groupId>  
            <artifactId>C</artifactId>  
            <version>2.0</version>  
    </dependency> 


 

## 五、依赖关系的查看

cmd进入工程根目录，执行  mvn dependency:tree

会列出依赖关系树及各依赖关系

**mvn dependency:analyze    分析依赖关系**   

    


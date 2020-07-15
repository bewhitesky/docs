## maven导入本地jar包

* maven导入本地oracle jar包
```text
mvn install:install-file -DgroupId=com.Oracle -DartifactId=ojdbc14 -Dversion=11.2.0.1.0 -Dpackaging=jar -D file=D:\apache-maven-3.6.3\Repository\com\oracle\ojdbc14\11.2.0.1.0\ojdbc14.jar
```
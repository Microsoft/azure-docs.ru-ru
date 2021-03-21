---
title: Добавление Application Insights ВИРТУАЛЬНОЙ машины Java ARG-Azure Monitor для Java
description: Добавление аргумента ВИРТУАЛЬНОЙ машины Java, который включает Application Insights Azure Monitor для Java
ms.topic: conceptual
ms.date: 04/16/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: bd6ed2e5c848b39ad946209d5ea92fcf6ce3b438
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98625370"
---
# <a name="adding-the-jvm-arg---azure-monitor-application-insights-for-java"></a>Добавление Application Insights ВИРТУАЛЬНОЙ машины Java ARG-Azure Monitor для Java



## <a name="azure-environments"></a>Среды Azure

Настройка [служб приложений](../../app-service/configure-language-java.md#set-java-runtime-options).

## <a name="spring-boot"></a>Spring Boot

Добавьте аргумент ВИРТУАЛЬНОЙ машины Java в `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` место до `-jar` , например:

```
java -javaagent:path/to/applicationinsights-agent-3.0.2.jar -jar <myapp.jar>
```

## <a name="spring-boot-via-docker-entry-point"></a>Пружинная загрузка через точку входа DOCKER

Если используется форма *exec* , добавьте параметр `"-javaagent:path/to/applicationinsights-agent-3.0.2.jar"` в список параметров в месте перед `"-jar"` параметром, например:

```
ENTRYPOINT ["java", "-javaagent:path/to/applicationinsights-agent-3.0.2.jar", "-jar", "<myapp.jar>"]
```

Если вы используете форму *оболочки* , добавьте аргумент виртуальной машины Java в `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` место до, например `-jar` :

```
ENTRYPOINT java -javaagent:path/to/applicationinsights-agent-3.0.2.jar -jar <myapp.jar>
```

## <a name="tomcat-8-linux"></a>Tomcat 8 (Linux)

### <a name="tomcat-installed-via-apt-get-or-yum"></a>Tomcat, установленный с помощью `apt-get` или `yum`

Если вы установили Tomcat через `apt-get` или `yum` , то у вас должен быть файл `/etc/tomcat8/tomcat8.conf` .  Добавьте следующую строку в конец этого файла:

```
JAVA_OPTS="$JAVA_OPTS -javaagent:path/to/applicationinsights-agent-3.0.2.jar"
```

### <a name="tomcat-installed-via-download-and-unzip"></a>Tomcat, установленные с помощью скачивания и распаковки

Если вы установили Tomcat с помощью Download и unzip из [https://tomcat.apache.org](https://tomcat.apache.org) , то у вас должен быть файл `<tomcat>/bin/catalina.sh` .  Создайте новый файл в том же каталоге с именем и `<tomcat>/bin/setenv.sh` следующим содержимым:

```
CATALINA_OPTS="$CATALINA_OPTS -javaagent:path/to/applicationinsights-agent-3.0.2.jar"
```

Если файл `<tomcat>/bin/setenv.sh` уже существует, измените его и добавьте `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` в `CATALINA_OPTS` .


## <a name="tomcat-8-windows"></a>Tomcat 8 (Windows)

### <a name="running-tomcat-from-the-command-line"></a>Запуск Tomcat из командной строки

Найдите файл `<tomcat>/bin/catalina.bat`.  Создайте новый файл в том же каталоге с именем и `<tomcat>/bin/setenv.bat` следующим содержимым:

```
set CATALINA_OPTS=%CATALINA_OPTS% -javaagent:path/to/applicationinsights-agent-3.0.2.jar
```

Кавычки не являются обязательными, но если вы хотите включить их, следует учитывать следующее:

```
set "CATALINA_OPTS=%CATALINA_OPTS% -javaagent:path/to/applicationinsights-agent-3.0.2.jar"
```

Если файл `<tomcat>/bin/setenv.bat` уже существует, просто измените его и добавьте `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` в `CATALINA_OPTS` .

### <a name="running-tomcat-as-a-windows-service"></a>Запуск Tomcat как службы Windows

Найдите файл `<tomcat>/bin/tomcat8w.exe`.  Запустите этот исполняемый файл и добавьте `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` его на `Java Options` `Java` вкладку.


## <a name="jboss-eap-7"></a>JBoss EAP 7

### <a name="standalone-server"></a>Отдельный сервер

Добавьте `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` в существующую `JAVA_OPTS` переменную среды в файле `JBOSS_HOME/bin/standalone.conf` (Linux) или `JBOSS_HOME/bin/standalone.conf.bat` (Windows):

```java    ...
    JAVA_OPTS="<b>-javaagent:path/to/applicationinsights-agent-3.0.2.jar</b> -Xms1303m -Xmx1303m ..."
    ...
```

### <a name="domain-server"></a>Сервер домена

Добавьте `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` к существующему `jvm-options` в `JBOSS_HOME/domain/configuration/host.xml` :

```xml
...
<jvms>
    <jvm name="default">
        <heap size="64m" max-size="256m"/>
        <jvm-options>
            <option value="-server"/>
            <!--Add Java agent jar file here-->
            <option value="-javaagent:path/to/applicationinsights-agent-3.0.2.jar"/>
            <option value="-XX:MetaspaceSize=96m"/>
            <option value="-XX:MaxMetaspaceSize=256m"/>
        </jvm-options>
    </jvm>
</jvms>
...
```

Если на одном узле выполняется несколько управляемых серверов, необходимо добавить в `applicationinsights.agent.id` `system-properties` для каждого из них `server` :

```xml
...
<servers>
    <server name="server-one" group="main-server-group">
        <!--Edit system properties for server-one-->
        <system-properties> 
            <property name="applicationinsights.agent.id" value="..."/>
        </system-properties>
    </server>
    <server name="server-two" group="main-server-group">
        <socket-bindings port-offset="150"/>
        <!--Edit system properties for server-two-->
        <system-properties>
            <property name="applicationinsights.agent.id" value="..."/> 
        </system-properties>
    </server>
</servers>
...
```

Указанное `applicationinsights.agent.id` значение должно быть уникальным. Он используется для создания вложенного каталога в каталоге applicationinsights, так как каждому процессу ВИРТУАЛЬНОЙ машины Java требуется собственная локальная applicationinsights config и локальный файл журнала applicationinsights. Кроме того, при отправке отчетов центральному сборщику `applicationinsights.properties` файл совместно используется несколькими управляемыми серверами, поэтому `applicationinsights.agent.id` для переопределения `agent.id` параметра в этом общем файле необходимо указать указанный параметр. `applicationinsights.agent.rollup.id` можно также указать на сервере, если необходимо `system-properties` переопределить `agent.rollup.id` параметр для каждого управляемого сервера.


## <a name="jetty-9"></a>Jetty 9

Добавьте эти строки в `start.ini`

```
--exec
-javaagent:path/to/applicationinsights-agent-3.0.2.jar
```


## <a name="payara-5"></a>Пайара 5

Добавьте `-javaagent:path/to/applicationinsights-agent-3.0.2.jar` к существующему `jvm-options` в `glassfish/domains/domain1/config/domain.xml` :

```xml
...
<java-config ...>
    <!--Edit the JVM options here-->
    <jvm-options>
        -javaagent:path/to/applicationinsights-agent-3.0.2.jar>
    </jvm-options>
        ...
</java-config>
...
```

## <a name="websphere-8"></a>WebSphere 8

Откройте консоль управления, перейдите в раздел **серверы > WebSphere приложений серверы > серверы приложений**, выберите соответствующие серверы приложений и щелкните: 

```
Java and Process Management > Process definition >  Java Virtual Machine
```
В «Generic ВИРТУАЛЬНОЙ машины Java arguments» добавьте следующее:
```
-javaagent:path/to/applicationinsights-agent-3.0.2.jar
```
После этого сохраните и перезапустите сервер приложений.


## <a name="openliberty-18"></a>Опенлиберти 18

Создайте новый файл `jvm.options` в каталоге сервера (например `<openliberty>/usr/servers/defaultServer` ,) и добавьте следующую строку:
```
-javaagent:path/to/applicationinsights-agent-3.0.2.jar
```

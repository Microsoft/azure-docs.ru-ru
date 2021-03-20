---
title: Настройка приложений Java
description: Узнайте, как настроить приложения Java для запуска в службе приложений Azure. В этой статье показаны наиболее распространенные задачи настройки.
keywords: служба приложений Azure, веб-приложение, Windows, OSS, Java, Tomcat, JBoss
author: jasonfreeberg
ms.devlang: java
ms.topic: article
ms.date: 04/12/2019
ms.author: jafreebe
ms.reviewer: cephalin
ms.custom: seodec18, devx-track-java, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
ms.openlocfilehash: 0334e259f75440cae25f1e165c0621c85f7c7705
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97804009"
---
# <a name="configure-a-java-app-for-azure-app-service"></a>Настройка приложения Java для Службы приложений Azure

Служба приложений Azure позволяет разработчикам Java быстро создавать, развертывать и масштабировать веб-приложения Java SE, Tomcat и JBoss на полностью управляемой службе. Развертывайте приложения с помощью подключаемых модулей Maven, из командной строки или в таких редакторах, как IntelliJ, Eclipse или Visual Studio Code.

Это краткое описание содержит основные понятия и инструкции для разработчиков Java, использующих службу приложений. Если вы никогда не использовали службу приложений Azure, сначала ознакомьтесь с [кратким](quickstart-java.md) руководством по Java. Ответы на часто задаваемые вопросы об использовании службы приложений, не относящейся к разработке Java, задаются в [разделе вопросов и ответов службы приложений](faq-configuration-and-management.md).

## <a name="deploying-your-app"></a>Развертывание приложения

[Подключаемый модуль веб-приложения Azure для Maven](https://github.com/microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md) можно использовать для развертывания файлов. War или. jar. Развертывание с популярным IDE также поддерживается с [Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/) или [Azure Toolkit for Eclipse](/azure/developer/java/toolkit-for-eclipse).

В противном случае метод развертывания будет зависеть от типа вашего архива.

### <a name="java-se"></a>Java SE

Чтобы развернуть JAR-файлы в Java SE, используйте `/api/zipdeploy/` конечную точку сайта KUDU. Дополнительные сведения об этом API см. в [этой документации](./deploy-zip.md#rest).

### <a name="tomcat"></a>Tomcat

Чтобы развернуть файлы WAR в Tomcat, используйте конечную точку `/api/wardeploy/` для публикации файла архива. Дополнительные сведения об этом API см. в [этой документации](./deploy-zip.md#deploy-war-file).

::: zone pivot="platform-linux"

### <a name="jboss-eap"></a>JBoss EAP

Чтобы развернуть WAR-файлы в JBoss, используйте `/api/wardeploy/` конечную точку для публикации файла архива. Дополнительные сведения об этом API см. в [этой документации](./deploy-zip.md#deploy-war-file).

Для развертывания файлов. «.» [Используйте протокол FTP](deploy-ftp.md).

::: zone-end

Не развертывайте файлы WAR или JAR с помощью FTP. Средство FTP предназначено для передачи сценариев запуска, зависимостей или других файлов среды выполнения. Оно не является оптимальным решением для развертывания веб-приложений.

## <a name="logging-and-debugging-apps"></a>Ведение журнала и отладка приложений

Отчеты о производительности, визуализация трафика и проверка работоспособности доступны на портале Azure для каждого приложения. Дополнительные сведения см. в статье [Обзор диагностики Службы приложений Azure](overview-diagnostics.md).

### <a name="stream-diagnostic-logs"></a>Потоковая передача журналов диагностики

::: zone pivot="platform-windows"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end
::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

::: zone-end

Дополнительные сведения см. в разделе [Потоковая передача журналов в Cloud Shell](troubleshoot-diagnostic-logs.md#in-cloud-shell).

::: zone pivot="platform-linux"

### <a name="ssh-console-access"></a>Доступ к консоли SSH

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

### <a name="troubleshooting-tools"></a>Средства диагностики

Встроенные образы Java основаны на операционной системе [Alpine Linux](https://alpine-linux.readthedocs.io/en/latest/getting_started.html). Используйте диспетчер пакетов `apk`, чтобы установить любые средства или команды для устранения неполадок.

::: zone-end

### <a name="flight-recorder"></a>«Черный ящик»

Все среды выполнения Java в службе приложений с использованием Azul виртуальных машин Java поставляются с Zuluным средством записи черного ящика. Его можно использовать для записи событий ВИРТУАЛЬНОЙ машины Java, System и Application, а также для устранения неполадок в приложениях Java.

::: zone pivot="platform-windows"

#### <a name="timed-recording"></a>Запись за интервал времени

Для получения времени записи потребуется идентификатор процесса (ID) приложения Java. Чтобы найти PID, откройте браузер на сайте SCM веб-приложения по адресу https://<имя вашего сайта>. scm.azurewebsites.net/ProcessExplorer/. На этой странице отображаются запущенные процессы в веб-приложении. Найдите процесс с именем "Java" в таблице и скопируйте соответствующий PID (идентификатор процесса).

Затем откройте **консоль отладки** на верхней панели инструментов сайта SCM и выполните следующую команду. Замените `<pid>` идентификатором процесса, который вы скопировали ранее. Эта команда запускает на 30-секундную запись профилировщика приложения Java и создает файл с именем `timed_recording_example.jfr` в `D:\home` каталоге.

```
jcmd <pid> JFR.start name=TimedRecording settings=profile duration=30s filename="D:\home\timed_recording_example.JFR"
```

::: zone-end
::: zone pivot="platform-linux"

Подключитесь к службе приложений по протоколу SSH и выполните `jcmd` команду, чтобы просмотреть список всех запущенных процессов Java. Кроме jcmd, вы также должны увидеть, что приложение Java выполняется с идентификатором процесса (PID).

```shell
078990bbcd11:/home# jcmd
Picked up JAVA_TOOL_OPTIONS: -Djava.net.preferIPv4Stack=true
147 sun.tools.jcmd.JCmd
116 /home/site/wwwroot/app.jar
```

Выполните приведенную ниже команду, чтобы запустить 30-секундную запись виртуальной машины Java. Это приведет к профилированию виртуальной машины Java и созданию JFR-файла с именем *jfr_example.jfr* в домашнем каталоге. (Замените 116 идентификатором PID приложения Java.)

```shell
jcmd 116 JFR.start name=MyRecording settings=profile duration=30s filename="/home/jfr_example.jfr"
```

В течение 30-секундного интервала можно проверить запись, выполнив команду `jcmd 116 JFR.check`. Будут показаны все записи для данного процесса Java.

#### <a name="continuous-recording"></a>Непрерывная запись

Вы можете использовать "черный ящик" Zulu для непрерывного профилирования приложения Java с минимальным влиянием на производительность среды выполнения ([источник](https://assets.azul.com/files/Zulu-Mission-Control-data-sheet-31-Mar-19.pdf)). Для этого выполните следующую команду Azure CLI, чтобы создать параметр приложения JAVA_OPTS с требуемой конфигурацией. Содержимое параметра приложения JAVA_OPTS передается команде `java` при запуске приложения.

```azurecli
az webapp config appsettings set -g <your_resource_group> -n <your_app_name> --settings JAVA_OPTS=-XX:StartFlightRecording=disk=true,name=continuous_recording,dumponexit=true,maxsize=1024m,maxage=1d
```

После начала записи вы можете в любое время сохранить дамп текущих данных, используя команду `JFR.dump`.

```shell
jcmd <pid> JFR.dump name=continuous_recording filename="/home/recording1.jfr"
```

::: zone-end

#### <a name="analyze-jfr-files"></a>Анализ `.jfr` файлов

Используйте [FTPS](deploy-ftp.md), чтобы скачать JFR-файл на локальный компьютер. Чтобы проанализировать JFR-файл, скачайте и установите[Zulu Mission Control](https://www.azul.com/products/zulu-mission-control/). Инструкции по Zulu Mission Control см. в [документации Azul](https://docs.azul.com/zmc/) и [инструкциях по установке](/java/azure/jdk/java-jdk-flight-recorder-and-mission-control).

### <a name="app-logging"></a>Ведение журнала приложений

::: zone pivot="platform-windows"

Включите [ведение журнала приложений](troubleshoot-diagnostic-logs.md#enable-application-logging-windows) с помощью портала Azure или [Azure CLI](/cli/azure/webapp/log#az-webapp-log-config), чтобы настроить службу приложений для записи выходных данных стандартной консоли приложения и потоков ошибок стандартной консоли в локальную файловую систему или хранилище BLOB-объектов Azure. Запись журналов в локальную файловую систему экземпляра службы приложений отключается через 12 часов после настройки ведения журнала. Если необходимо более длительное хранение, настройте приложение для записи выходных данных в контейнер больших двоичных объектов. Журналы приложений Java и Tomcat можно найти в каталоге */home/LogFiles/Application/* .

::: zone-end
::: zone pivot="platform-linux"

Включите [ведение журнала приложений](troubleshoot-diagnostic-logs.md#enable-application-logging-linuxcontainer) с помощью портала Azure или [Azure CLI](/cli/azure/webapp/log#az-webapp-log-config), чтобы настроить службу приложений для записи выходных данных стандартной консоли приложения и потоков ошибок стандартной консоли в локальную файловую систему или хранилище BLOB-объектов Azure. Если необходимо более длительное хранение, настройте приложение для записи выходных данных в контейнер больших двоичных объектов. Журналы приложений Java и Tomcat можно найти в каталоге */home/LogFiles/Application/* .

Ведение журнала хранилища BLOB-объектов Azure для служб приложений на основе Linux можно настроить только с помощью [Azure Monitor (Предварительная версия)](./troubleshoot-diagnostic-logs.md#send-logs-to-azure-monitor-preview) 

::: zone-end

Если приложение использует [Logback](https://logback.qos.ch/) или [Log4j](https://logging.apache.org/log4j) для трассировки, то эти данные трассировки можно передать в Azure Application Insights для просмотра, выполнив инструкции по настройке платформы ведения журнала в разделе [Просмотр журналов трассировки Java в Application Insights](../azure-monitor/app/java-trace-logs.md).

## <a name="customization-and-tuning"></a>Настройка

Служба приложений Azure для Linux поддерживает настройку с помощью портала Azure и интерфейса командной строки. Ознакомьтесь со следующими статьями о настройке конкретных веб-приложений не на платформе Java:

- [Настройка параметров приложения](configure-common.md#configure-app-settings)
- [Настройка личного домена](app-service-web-tutorial-custom-domain.md)
- [Настройка привязки SSL](configure-ssl-bindings.md)
- [Добавление CDN](../cdn/cdn-add-to-web-app.md)
- [Настройка сайта Kudu](https://github.com/projectkudu/kudu/wiki/Configurable-settings#linux-on-app-service-settings)


### <a name="set-java-runtime-options"></a>Настройка параметров среды выполнения Java

Чтобы задать выделенную память или другие параметры среды выполнения ВИРТУАЛЬНОЙ машины Java, создайте [параметр приложения](configure-common.md#configure-app-settings) `JAVA_OPTS` с именем с параметрами. Служба приложений передает этот параметр в среду выполнения Java при запуске в качестве переменной среды.

На портале Azure в разделе **Параметры приложения** для веб-приложения создайте параметр приложения `JAVA_OPTS`, включающий в себя дополнительные параметры, такие как `-Xms512m -Xmx1204m`.

Чтобы настроить параметр приложения из подключаемого модуля Maven, добавьте теги "параметр/значение" в раздел подключаемого модуля Azure. В следующем примере задаются минимальный и максимальный размеры кучи для Java.

```xml
<appSettings>
    <property>
        <name>JAVA_OPTS</name>
        <value>-Xms512m -Xmx1204m</value>
    </property>
</appSettings>
```

Разработчики, запускающие отдельное приложение в одном слоте развертывания в плане службы приложений, могут использовать следующие параметры.

- Экземпляры B1 и S1: `-Xms1024m -Xmx1024m`
- Экземпляры B2 и S2: `-Xms3072m -Xmx3072m`
- Экземпляры B3 и S3: `-Xms6144m -Xmx6144m`

При настройке параметров кучи приложения просмотрите сведения о плане службы приложений и примите во внимание, что наличие нескольких приложений и одного слота развертывания требует оптимального выделения памяти.

### <a name="turn-on-web-sockets"></a>Включение веб-сокетов

Включите для приложения поддержку веб-сокетов на портале Azure в разделе **Параметры приложения**. Необходимо будет перезапустить приложение, чтобы этот параметр вступил в силу.

Включите поддержку веб-сокетов с помощью Azure CLI, выполнив следующую команду.

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --web-sockets-enabled true
```

Затем перезапустите приложение.

```azurecli-interactive
az webapp stop --name <app-name> --resource-group <resource-group-name>
az webapp start --name <app-name> --resource-group <resource-group-name>
```

### <a name="set-default-character-encoding"></a>Настройка кодировки по умолчанию

На портале Azure в разделе **Параметры приложения** для веб-приложения создайте параметр приложения `JAVA_OPTS` со значением `-Dfile.encoding=UTF-8`.

Можно также настроить параметр приложения с помощью подключаемого модуля Maven для службы приложений. Добавьте теги имени и значения параметра в конфигурацию подключаемого модуля.

```xml
<appSettings>
    <property>
        <name>JAVA_OPTS</name>
        <value>-Dfile.encoding=UTF-8</value>
    </property>
</appSettings>
```

### <a name="pre-compile-jsp-files"></a>Предварительная компиляция JSP-файлов

Чтобы повысить производительность приложений Tomcat, можно скомпилировать JSP-файлы перед развертыванием в службе приложений. Вы можете использовать [подключаемый модуль Maven](https://sling.apache.org/components/jspc-maven-plugin/plugin-info.html), предоставляемый Apache Sling, или использовать этот [файла сборки Ant](https://tomcat.apache.org/tomcat-9.0-doc/jasper-howto.html#Web_Application_Compilation).

## <a name="secure-applications"></a>Защита приложений

Приложения Java, работающие в службе приложений, имеют тот же набор [рекомендаций по обеспечению безопасности](../security/fundamentals/paas-applications-using-app-services.md) , что и другие приложения.

### <a name="authenticate-users-easy-auth"></a>Аутентификация пользователей (Easy Auth)

Настройте аутентификацию приложения на портале Azure с помощью параметра **Проверка подлинности и авторизация**. Вы можете включить аутентификацию с помощью Azure Active Directory или имен для входа в социальные сети, таких как Facebook, Google или GitHub. На портале Azure можно настроить только один поставщик аутентификации. Дополнительные сведения приведены в разделе [Настройка приложения службы приложений для использования входа с помощью Azure Active Directory](configure-authentication-provider-aad.md) и связанных статьях о других поставщиках удостоверений. Если необходимо включить несколько поставщиков входа, следуйте инструкциям в статье [Настройка проверки подлинности и авторизации в службе приложений Azure](app-service-authentication-how-to.md).

#### <a name="java-se"></a>Java SE

Разработчики для Spring Boot могут использовать [краткое руководство по использованию Spring Boot и Azure Active Directory](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-azure-active-directory), чтобы защитить приложения с помощью привычных заметок и интерфейсов API Spring Security. Увеличьте максимальный размер заголовка в файле *application.properties*. Мы рекомендуем использовать значение `16384`.

#### <a name="tomcat"></a>Tomcat

Приложение Tomcat может получить доступ к утверждениям пользователя непосредственно из сервлета, приведя объект Principal к объекту Map. Объект Map будет сопоставлять каждый тип утверждения с коллекцией утверждений для этого типа. В приведенном ниже коде `request` является экземпляром `HttpServletRequest`.

```java
Map<String, Collection<String>> map = (Map<String, Collection<String>>) request.getUserPrincipal();
```

Теперь можно проверить объект `Map` для любого конкретного утверждения. Например, следующий фрагмент кода выполняет перебор всех типов утверждений и выводит содержимое каждой коллекции.

```java
for (Object key : map.keySet()) {
        Object value = map.get(key);
        if (value != null && value instanceof Collection {
            Collection claims = (Collection) value;
            for (Object claim : claims) {
                System.out.println(claims);
            }
        }
    }
```

Чтобы произвести выход пользователей, используйте путь `/.auth/ext/logout`. Для выполнения других действий см. документацию по [аутентификации и авторизации в Службе приложений](./app-service-authentication-how-to.md). Существует также официальная документация по интерфейсу Tomcat [HttpServletRequest](https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletRequest.html) и его методам. В зависимости от конфигурации Службы приложений также происходит расконсервация следующих методов сервлета:

```java
public boolean isSecure()
public String getRemoteAddr()
public String getRemoteHost()
public String getScheme()
public int getServerPort()
```

Чтобы отключить эту функцию, создайте параметр приложения с именем `WEBSITE_AUTH_SKIP_PRINCIPAL` со значением `1`. Чтобы отключить все фильтры сервлета, добавленные службой приложений, создайте параметр с именем `WEBSITE_SKIP_FILTERS` со значением `1`.

### <a name="configure-tlsssl"></a>Настройка TLS/SSL

Следуйте инструкциям в разделе [Обеспечение безопасности настраиваемого DNS-имени с помощью привязки SSL в Службе приложений Azure](configure-ssl-bindings.md), чтобы передать существующий SSL-сертификат и привязать его к доменному имени приложения. По умолчанию приложение по-прежнему будет разрешать HTTP-подключения. Выполните соответствующие инструкции в этом руководстве, чтобы принудительно включить SSL и TLS.

### <a name="use-keyvault-references"></a>Использование возможностей Key Vault

[Azure Key Vault](../key-vault/general/overview.md) обеспечивает централизованное управление секретами с помощью политик доступа и журнала аудита. Вы можете хранить секреты (например, пароли или строки подключения) в Key Vault и обращаться к этим секретам в приложении с помощью переменных среды.

Прежде всего, следуйте инструкциям, чтобы [предоставить приложению доступ к Key Vault](app-service-key-vault-references.md#granting-your-app-access-to-key-vault) и [сделать ссылку Key Vault на секрет в параметре приложения](app-service-key-vault-references.md#reference-syntax). Чтобы проверить, что ссылка разрешается в секрет, можно выполнить печать переменной среды во время удаленного доступа к терминалу Службы приложений.

Чтобы внедрить эти секреты в файл конфигурации Spring или Tomcat, используйте синтаксис внедрения переменных среды (`${MY_ENV_VAR}`). Дополнительные сведения о файлах конфигурации Spring см. в этой документации по [внешним конфигурациям](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).

::: zone pivot="platform-linux"

### <a name="use-the-java-key-store"></a>Использование хранилища ключей Java

По умолчанию все общедоступные или частные сертификаты, [отправленные в Службу приложений Linux](configure-ssl-certificate.md), будут загружены в соответствующие хранилища ключей Java при запуске контейнера. После отправки сертификата необходимо перезапустить Службу приложений, чтобы она загрузилась в хранилище ключей Java. Общедоступные сертификаты загружаются в хранилище ключей в `$JAVA_HOME/jre/lib/security/cacerts`, а частные сертификаты хранятся в `$JAVA_HOME/lib/security/client.jks`.

Для шифрования подключения JDBC с сертификатами в хранилище ключей Java может потребоваться дополнительная настройка. Дополнительные сведения см. в документации по выбранному драйверу JDBC.

- [PostgreSQL](https://jdbc.postgresql.org/documentation/head/ssl-client.html)
- [SQL Server](/sql/connect/jdbc/connecting-with-ssl-encryption)
- [MySQL](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-using-ssl.html)
- [MongoDB](https://mongodb.github.io/mongo-java-driver/3.4/driver/tutorials/ssl/)
- [Cassandra](https://docs.datastax.com/en/developer/java-driver/4.3/)

#### <a name="initialize-the-java-key-store"></a>Инициализация хранилища ключей Java

Чтобы инициализировать объект `import java.security.KeyStore`, загрузите файл хранилища ключей с паролем. Пароль по умолчанию для обоих хранилищ ключей — "changeit".

```java
KeyStore keyStore = KeyStore.getInstance("jks");
keyStore.load(
    new FileInputStream(System.getenv("JAVA_HOME")+"/lib/security/cacets"),
    "changeit".toCharArray());

KeyStore keyStore = KeyStore.getInstance("pkcs12");
keyStore.load(
    new FileInputStream(System.getenv("JAVA_HOME")+"/lib/security/client.jks"),
    "changeit".toCharArray());
```

#### <a name="manually-load-the-key-store"></a>Загрузка хранилища ключей вручную

Сертификаты можно загрузить в хранилище ключей вручную. Создайте параметр приложения `SKIP_JAVA_KEYSTORE_LOAD` со значением `1`, чтобы отключить автоматическую загрузку сертификатов в хранилище ключей Службой приложений. Все общедоступные сертификаты, отправленные в Службу приложений через портал Azure, хранятся в `/var/ssl/certs/`. Закрытые сертификаты хранятся в `/var/ssl/private/`.

Вы можете взаимодействовать с инструментом для работы с ключами Java или отлаживать его, [открыв SSH-подключение](configure-linux-open-ssh-session.md) к Службе приложений и выполнив команду `keytool`. Список команд см. в документации по [инструменту для работы с ключами](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html). Дополнительные сведения об API хранилища ключей см. в [официальной документации](https://docs.oracle.com/javase/8/docs/api/java/security/KeyStore.html).

::: zone-end

## <a name="configure-apm-platforms"></a>Настройка платформ APM

В этом разделе показано, как подключить приложения Java, развернутые в Службе приложений Azure в Linux, к платформам мониторинга производительности (APM) приложений NewRelic и AppDynamics.

### <a name="configure-new-relic"></a>Настройка New Relic

::: zone pivot="platform-windows"

1. Создайте учетную запись NewRelic на сайте [NewRelic.com](https://newrelic.com/signup)
2. Скачайте агент Java с NewRelic. Его имя файла будет аналогично *newrelic-java-x.x.x.zip*.
3. Скопируйте ключ лицензии. Он понадобиться позже для настройки агента.
4. [Подключитесь к экземпляру службы приложений по протоколу SSH](configure-linux-open-ssh-session.md) и создайте каталог */home/site/wwwroot/apm*.
5. Отправьте распакованные файлы агента Java NewRelic в каталог */home/site/wwwroot/apm*. Файлы агента должны располагаться в */home/site/wwwroot/apm/newrelic*.
6. Измените файл YAML в каталоге */home/site/wwwroot/apm/newrelic/newrelic.yml* и замените значение лицензии в заполнителе собственным ключом лицензии.
7. На портале Azure перейдите к приложению в службе приложений и создайте параметр приложения.

    - Для приложений **Java SE** создайте переменную среды с именем `JAVA_OPTS` и значением `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar` .
    - Для **Tomcat** создайте переменную среды с именем `CATALINA_OPTS` и значением `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar` .

::: zone-end
::: zone pivot="platform-linux"

1. Создайте учетную запись NewRelic на сайте [NewRelic.com](https://newrelic.com/signup)
2. Скачайте агент Java с NewRelic. Его имя файла будет аналогично *newrelic-java-x.x.x.zip*.
3. Скопируйте ключ лицензии. Он понадобиться позже для настройки агента.
4. [Подключитесь к экземпляру службы приложений по протоколу SSH](configure-linux-open-ssh-session.md) и создайте каталог */home/site/wwwroot/apm*.
5. Отправьте распакованные файлы агента Java NewRelic в каталог */home/site/wwwroot/apm*. Файлы агента должны располагаться в */home/site/wwwroot/apm/newrelic*.
6. Измените файл YAML в каталоге */home/site/wwwroot/apm/newrelic/newrelic.yml* и замените значение лицензии в заполнителе собственным ключом лицензии.
7. На портале Azure перейдите к приложению в службе приложений и создайте параметр приложения.
   
    - Для приложений **Java SE** создайте переменную среды с именем `JAVA_OPTS` и значением `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar` .
    - Для **Tomcat** создайте переменную среды с именем `CATALINA_OPTS` и значением `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar` .

::: zone-end

>  Если у вас уже есть переменная среды `JAVA_OPTS` или `CATALINA_OPTS`, добавьте параметр `-javaagent:/...` в конец текущего значения.

### <a name="configure-appdynamics"></a>Настройка AppDynamics

::: zone pivot="platform-windows"

1. Создайте учетную запись AppDynamics на сайте [AppDynamics.com](https://www.appdynamics.com/community/register/)
2. Скачайте агент Java с веб-сайта AppDynamics. Имя файла будет аналогично *AppServerAgent-x.x.x.xxxxx.zip*.
3. С помощью [консоли KUDU](https://github.com/projectkudu/kudu/wiki/Kudu-console) создайте новый каталог */Хоме/Сите/ввврут/АПМ*.
4. Отправьте файлы агента Java в каталог */home/site/wwwroot/apm*. Файлы агента должны располагаться в */home/site/wwwroot/apm/appdynamics*.
5. На портале Azure перейдите к приложению в службе приложений и создайте параметр приложения.

   - Для приложений **Java SE** создайте переменную среды с именем `JAVA_OPTS` и значением, `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` где `<app-name>` — имя службы приложений.
   - Для приложений **Tomcat** создайте переменную среды с именем `CATALINA_OPTS` и значением, `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` где `<app-name>` — имя службы приложений.

::: zone-end
::: zone pivot="platform-linux"

1. Создайте учетную запись AppDynamics на сайте [AppDynamics.com](https://www.appdynamics.com/community/register/)
2. Скачайте агент Java с веб-сайта AppDynamics. Имя файла будет аналогично *AppServerAgent-x.x.x.xxxxx.zip*.
3. [Подключитесь к экземпляру службы приложений по протоколу SSH](configure-linux-open-ssh-session.md) и создайте каталог */home/site/wwwroot/apm*.
4. Отправьте файлы агента Java в каталог */home/site/wwwroot/apm*. Файлы агента должны располагаться в */home/site/wwwroot/apm/appdynamics*.
5. На портале Azure перейдите к приложению в службе приложений и создайте параметр приложения.

   - Для приложений **Java SE** создайте переменную среды с именем `JAVA_OPTS` и значением, `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` где `<app-name>` — имя службы приложений.
   - Для приложений **Tomcat** создайте переменную среды с именем `CATALINA_OPTS` и значением, `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>` где `<app-name>` — имя службы приложений.

::: zone-end

> [!NOTE]
>  Если у вас уже есть переменная среды `JAVA_OPTS` или `CATALINA_OPTS`, добавьте параметр `-javaagent:/...` в конец текущего значения.

## <a name="configure-data-sources"></a>Настройка источников данных

### <a name="java-se"></a>Java SE

Чтобы подключиться к источникам данных в приложениях Spring Boot, мы рекомендуем создавать строки подключения и внедрять их в файл *application.properties*.

1. В разделе "Конфигурация" на странице Службы приложений задайте имя строки, вставьте строку подключения JDBC в поле "Значение" и присвойте типу значение "Пользовательский". При необходимости можно задать эту строку подключения в качестве параметра слота.

    Эта строка подключения доступна для нашего приложения в виде переменной среды с именем `CUSTOMCONNSTR_<your-string-name>`. Например, строка подключения, созданная выше, будет называться `CUSTOMCONNSTR_exampledb`.

2. В файле *application.properties* нужно сослаться на эту строку подключения с именем переменной среды. В нашем примере мы будем использовать следующее.

    ```yml
    app.datasource.url=${CUSTOMCONNSTR_exampledb}
    ```

Дополнительные сведения по этой теме см. в документации по [Spring Boot при доступе к данным](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-data-access.html) и [внешних конфигурациях](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).

::: zone pivot="platform-windows"

### <a name="tomcat"></a>Tomcat

Эти инструкции применимы ко всем подключениям к базе данных. Необходимо будет заменить значения заполнителей на имя класса драйвера и JAR-файл выбранной базы данных. Ниже приведена таблица с именами классов и ссылками для скачивания драйверов для распространенных баз данных.

| База данных   | Имя класса драйвера                             | Драйвер JDBC                                                                      |
|------------|-----------------------------------------------|------------------------------------------------------------------------------------------|
| PostgreSQL | `org.postgresql.Driver`                        | [Загрузить](https://jdbc.postgresql.org/download.html)                                    |
| MySQL      | `com.mysql.jdbc.Driver`                        | [Скачать](https://dev.mysql.com/downloads/connector/j/) (выберите "Platform Independent" (Независимо от платформы)) |
| SQL Server | `com.microsoft.sqlserver.jdbc.SQLServerDriver` | [Загрузить](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017#download)                                                           |

Чтобы настроить Tomcat для использования Java Database Connectivity (JDBC) или Java Persistence API (JPA), сначала настройте переменную среды `CATALINA_OPTS`, считываемую Tomcat при запуске. Задайте эти значения с помощью параметра приложения в [подключаемом модуле Maven для службы приложений](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md):

```xml
<appSettings>
    <property>
        <name>CATALINA_OPTS</name>
        <value>"$CATALINA_OPTS -Ddbuser=${DBUSER} -Ddbpassword=${DBPASSWORD} -DconnURL=${CONNURL}"</value>
    </property>
</appSettings>
```

Переменные среды можно также задать на странице **Конфигурация** > **Параметры приложения** на портале Azure.

Затем определите, должен ли источник данных быть доступным для одного приложения или для всех приложений, работающих в сервлете Tomcat.

#### <a name="application-level-data-sources"></a>Источники данных уровня приложения

1. Создайте файл *context.xml* в каталоге проекта *META-INF/* . Создайте каталог *META-INF/* , если его не существует.

2. В *context.xml* добавьте элемент `Context`, чтобы связать источник данных с адресом JNDI. Замените заполнитель `driverClassName` именем класса драйвера из приведенной выше таблицы.

    ```xml
    <Context>
        <Resource
            name="jdbc/dbconnection"
            type="javax.sql.DataSource"
            url="${dbuser}"
            driverClassName="<insert your driver class name>"
            username="${dbpassword}"
            password="${connURL}"
        />
    </Context>
    ```

3. Обновите *web.xml* приложения для использования источника данных в этом приложении.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="finalize-configuration"></a>Завершение конфигурации

Наконец, мы поместим драйвер JAR в путь к классам Tomcat и перезапустите службу приложений. Убедитесь, что файлы драйвера JDBC доступны для загрузчика классов Tomcat, разместив их в каталоге */home/tomcat/lib*. (Если этот каталог отсутствует, создайте его.) Чтобы передать эти файлы в экземпляр службы приложений, выполните следующие действия.

1. В [Cloud Shell](https://shell.azure.com) установите расширение webapp:

    ```azurecli-interactive
    az extension add -–name webapp
    ```

2. Выполните следующую команду CLI, чтобы создать туннель SSH между локальной системой и службой приложений.

    ```azurecli-interactive
    az webapp remote-connection create --resource-group <resource-group-name> --name <app-name> --port <port-on-local-machine>
    ```

3. Подключитесь к локальному порту туннелирования с помощью клиента SFTP и передайте эти файлы в папку */home/tomcat/lib*.

Кроме того, драйвер JDBC можно отправить с помощью FTP-клиента. Чтобы получить учетные данные FTP, следуйте этим [инструкциям](deploy-configure-credentials.md).

---

::: zone-end
::: zone pivot="platform-linux"

### <a name="tomcat"></a>Tomcat

Эти инструкции применимы ко всем подключениям к базе данных. Необходимо будет заменить значения заполнителей на имя класса драйвера и JAR-файл выбранной базы данных. Ниже приведена таблица с именами классов и ссылками для скачивания драйверов для распространенных баз данных.

| База данных   | Имя класса драйвера                             | Драйвер JDBC                                                                      |
|------------|-----------------------------------------------|------------------------------------------------------------------------------------------|
| PostgreSQL | `org.postgresql.Driver`                        | [Загрузить](https://jdbc.postgresql.org/download.html)                                    |
| MySQL      | `com.mysql.jdbc.Driver`                        | [Скачать](https://dev.mysql.com/downloads/connector/j/) (выберите "Platform Independent" (Независимо от платформы)) |
| SQL Server | `com.microsoft.sqlserver.jdbc.SQLServerDriver` | [Загрузить](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017#download)                                                           |

Чтобы настроить Tomcat для использования Java Database Connectivity (JDBC) или Java Persistence API (JPA), сначала настройте переменную среды `CATALINA_OPTS`, считываемую Tomcat при запуске. Задайте эти значения с помощью параметра приложения в [подключаемом модуле Maven для службы приложений](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md):

```xml
<appSettings>
    <property>
        <name>CATALINA_OPTS</name>
        <value>"$CATALINA_OPTS -Ddbuser=${DBUSER} -Ddbpassword=${DBPASSWORD} -DconnURL=${CONNURL}"</value>
    </property>
</appSettings>
```

Переменные среды можно также задать на странице **Конфигурация** > **Параметры приложения** на портале Azure.

Затем определите, должен ли источник данных быть доступным для одного приложения или для всех приложений, работающих в сервлете Tomcat.

#### <a name="application-level-data-sources"></a>Источники данных уровня приложения

1. Создайте файл *context.xml* в каталоге проекта *META-INF/* . Создайте каталог *META-INF/* , если его не существует.

2. В *context.xml* добавьте элемент `Context`, чтобы связать источник данных с адресом JNDI. Замените заполнитель `driverClassName` именем класса драйвера из приведенной выше таблицы.

    ```xml
    <Context>
        <Resource
            name="jdbc/dbconnection"
            type="javax.sql.DataSource"
            url="${dbuser}"
            driverClassName="<insert your driver class name>"
            username="${dbpassword}"
            password="${connURL}"
        />
    </Context>
    ```

3. Обновите *web.xml* приложения для использования источника данных в этом приложении.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="shared-server-level-resources"></a>Общие ресурсы уровня сервера

Для добавления общего источника данных на уровне сервера потребуется изменить файл server.xml Tomcat. Сначала отправьте [сценарий запуска](faq-app-service-linux.md#built-in-images) и задайте путь к скрипту в разделе **Конфигурация** > **Команда запуска**. Сценарий запуска можно добавить с помощью [FTP](deploy-ftp.md).

Сценарий запуска [преобразует XSL](https://www.w3schools.com/xml/xsl_intro.asp) в файл server.xml и выведет полученный XML-файл в `/usr/local/tomcat/conf/server.xml`. Сценарий запуска должен установить libxslt через apk. XSL-файл и скрипт запуска можно отправить через FTP. Ниже приведен пример скрипта запуска.

```sh
# Install libxslt. Also copy the transform file to /home/tomcat/conf/
apk add --update libxslt

# Usage: xsltproc --output output.xml style.xsl input.xml
xsltproc --output /home/tomcat/conf/server.xml /home/tomcat/conf/transform.xsl /usr/local/tomcat/conf/server.xml
```

Пример XSL-файла приведен ниже. В примере XSL-файл добавляет новый узел соединителя в server.xml Tomcat.

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="xml" indent="yes"/>

  <xsl:template match="@* | node()" name="Copy">
    <xsl:copy>
      <xsl:apply-templates select="@* | node()"/>
    </xsl:copy>
  </xsl:template>

  <xsl:template match="@* | node()" mode="insertConnector">
    <xsl:call-template name="Copy" />
  </xsl:template>

  <xsl:template match="comment()[not(../Connector[@scheme = 'https']) and
                                 contains(., '&lt;Connector') and
                                 (contains(., 'scheme=&quot;https&quot;') or
                                  contains(., &quot;scheme='https'&quot;))]">
    <xsl:value-of select="." disable-output-escaping="yes" />
  </xsl:template>

  <xsl:template match="Service[not(Connector[@scheme = 'https'] or
                                   comment()[contains(., '&lt;Connector') and
                                             (contains(., 'scheme=&quot;https&quot;') or
                                              contains(., &quot;scheme='https'&quot;))]
                                  )]
                      ">
    <xsl:copy>
      <xsl:apply-templates select="@* | node()" mode="insertConnector" />
    </xsl:copy>
  </xsl:template>

  <!-- Add the new connector after the last existing Connnector if there is one -->
  <xsl:template match="Connector[last()]" mode="insertConnector">
    <xsl:call-template name="Copy" />

    <xsl:call-template name="AddConnector" />
  </xsl:template>

  <!-- ... or before the first Engine if there is no existing Connector -->
  <xsl:template match="Engine[1][not(preceding-sibling::Connector)]"
                mode="insertConnector">
    <xsl:call-template name="AddConnector" />

    <xsl:call-template name="Copy" />
  </xsl:template>

  <xsl:template name="AddConnector">
    <!-- Add new line -->
    <xsl:text>&#xa;</xsl:text>
    <!-- This is the new connector -->
    <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true" 
               maxThreads="150" scheme="https" secure="true" 
               keystroreFile="${{user.home}}/.keystore" keystorePass="changeit"
               clientAuth="false" sslProtocol="TLS" />
  </xsl:template>
  
</xsl:stylesheet>
```

#### <a name="finalize-configuration"></a>Завершение конфигурации

Наконец, разместите JAR-файлы драйверов в пути к классам Tomcat и перезапустите службу приложений.

1. Убедитесь, что файлы драйвера JDBC доступны для загрузчика классов Tomcat, разместив их в каталоге */home/tomcat/lib*. (Если этот каталог отсутствует, создайте его.) Чтобы передать эти файлы в экземпляр службы приложений, выполните следующие действия.

    1. В [Cloud Shell](https://shell.azure.com) установите расширение webapp:

      ```azurecli-interactive
      az extension add -–name webapp
      ```

    2. Выполните следующую команду CLI, чтобы создать туннель SSH между локальной системой и службой приложений.

      ```azurecli-interactive
      az webapp remote-connection create --resource-group <resource-group-name> --name <app-name> --port <port-on-local-machine>
      ```

    3. Подключитесь к локальному порту туннелирования с помощью клиента SFTP и передайте эти файлы в папку */home/tomcat/lib*.

    Кроме того, драйвер JDBC можно отправить с помощью FTP-клиента. Чтобы получить учетные данные FTP, следуйте этим [инструкциям](deploy-configure-credentials.md).

2. Если вы создали источник данных на уровне сервера, перезапустите приложение Linux службы приложений. Tomcat сбросит `CATALINA_BASE` до значения `/home/tomcat` и будет использовать обновленную конфигурацию.

### <a name="jboss-eap"></a>JBoss EAP

При [регистрации источника данных с помощью JBoss EAP](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html/configuration_guide/datasource_management)необходимо выполнить три основных шага: загрузить драйвер JDBC, добавить драйвер JDBC в качестве модуля и зарегистрировать модуль. Служба приложений — это служба размещения без отслеживания состояния, поэтому команды конфигурации для добавления и регистрации модуля источника данных должны быть внесены в скрипт и применены при запуске контейнера.

1. Получите драйвер JDBC для вашей базы данных. 
2. Создайте файл определения модуля XML для драйвера JDBC. Ниже приведен пример определения модуля для PostgreSQL.

    ```xml
    <?xml version="1.0" ?>
    <module xmlns="urn:jboss:module:1.1" name="org.postgres">
        <resources>
        <!-- ***** IMPORTANT : REPLACE THIS PLACEHOLDER *******-->
        <resource-root path="/home/site/deployments/tools/postgresql-42.2.12.jar" />
        </resources>
        <dependencies>
            <module name="javax.api"/>
            <module name="javax.transaction.api"/>
        </dependencies>
    </module>
    ```

1. Вставьте команды интерфейса командной строки JBoss в файл с именем `jboss-cli-commands.cli` . Команды JBoss должны добавить модуль и зарегистрировать его в качестве источника данных. В приведенном ниже примере показаны команды интерфейса командной строки JBoss для PostgreSQL.

    ```bash
    #!/usr/bin/env bash
    module add --name=org.postgres --resources=/home/site/deployments/tools/postgresql-42.2.12.jar --module-xml=/home/site/deployments/tools/postgres-module.xml

    /subsystem=datasources/jdbc-driver=postgres:add(driver-name="postgres",driver-module-name="org.postgres",driver-class-name=org.postgresql.Driver,driver-xa-datasource-class-name=org.postgresql.xa.PGXADataSource)

    data-source add --name=postgresDS --driver-name=postgres --jndi-name=java:jboss/datasources/postgresDS --connection-url=${POSTGRES_CONNECTION_URL,env.POSTGRES_CONNECTION_URL:jdbc:postgresql://db:5432/postgres} --user-name=${POSTGRES_SERVER_ADMIN_FULL_NAME,env.POSTGRES_SERVER_ADMIN_FULL_NAME:postgres} --password=${POSTGRES_SERVER_ADMIN_PASSWORD,env.POSTGRES_SERVER_ADMIN_PASSWORD:example} --use-ccm=true --max-pool-size=5 --blocking-timeout-wait-millis=5000 --enabled=true --driver-class=org.postgresql.Driver --exception-sorter-class-name=org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter --jta=true --use-java-context=true --valid-connection-checker-class-name=org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker
    ```

1. Создайте сценарий запуска, `startup_script.sh` который вызывает команды интерфейса командной строки JBoss. В приведенном ниже примере показано, как вызвать `jboss-cli-commands.cli` . Позже вы конфигре службу приложений для запуска этого скрипта при запуске контейнера. 

    ```bash
    $JBOSS_HOME/bin/jboss-cli.sh --connect --file=/home/site/deployments/tools/jboss-cli-commands.cli
    ```

1. Используя FTP-клиент по своему усмотрению, отправьте драйвер JDBC, `jboss-cli-commands.cli` , `startup_script.sh` и определение модуля в `/site/deployments/tools/` .
2. Настройте сайт для запуска `startup_script.sh` при запуске контейнера. На портале Azure перейдите к разделу **Конфигурация**  >  **Общие параметры**  >  **запуска**. Задайте для поля команда запуска значение `/home/site/deployments/tools/startup_script.sh` . **Сохраните** внесенные изменения.

Чтобы убедиться, что источник данных добавлен на сервер JBoss, SSH-подключение в webapp и запустите `$JBOSS_HOME/bin/jboss-cli.sh --connect` . После подключения к JBoss выполните команду, `/subsystem=datasources:read-resource` чтобы распечатать список источников данных.

::: zone-end

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

## <a name="choosing-a-java-runtime-version"></a>Выбор версии среды выполнения Java

Служба приложений позволяет пользователям выбрать основную версию ВИРТУАЛЬНОЙ машины Java, например Java 8 или Java 11, а также дополнительный номер версии, например 1.8.0 _232 или 11.0.5. Кроме того, можно выбрать автоматическое обновление дополнительной версии по мере появления новых дополнительных версий. В большинстве случаев для рабочих сайтов следует использовать закрепленные дополнительные версии ВИРТУАЛЬНОЙ машины Java. Это предотвратит уннантиЦипатедие простоев во время автоматического обновления дополнительной версии.

Если вы решили закрепить дополнительный номер версии, необходимо периодически обновлять дополнительный номер версии ВИРТУАЛЬНОЙ машины Java на сайте. Чтобы обеспечить выполнение приложения в более новой дополнительной версии, создайте промежуточный слот и увеличьте дополнительный номер версии на промежуточном сайте. После подтверждения того, что приложение работает правильно в новой дополнительной версии, можно поменять местами промежуточные и рабочие слоты.

## <a name="jboss-eap-hardware-options"></a>Параметры оборудования JBoss EAP

JBoss EAP доступен только в вариантах "Премиум" и "изолированное оборудование". Клиенты, которые создали сайт EAP JBoss на уровне Free, Shared, Basic или Standard во время общедоступной предварительной версии, должны масштабироваться до уровня Premium или изолированного аппаратного обеспечения во избежание непредвиденного поведения.

## <a name="java-runtime-statement-of-support"></a>Заявление о поддержке среды выполнения Java

### <a name="jdk-versions-and-maintenance"></a>Версии JDK и обслуживание

Azure поддерживает пакет Java Development Kit (JDK) [Zulu](https://www.azul.com/downloads/azure-only/zulu/), предоставляемый компанией [Azul Systems](https://www.azul.com/). Сборка OpenJDK типа Azul Zulu Enterprise — это бесплатный мультиплатформенный, готовый дистрибутив OpenJDK для Azure и Azure Stack, который поддерживается корпорацией Майкрософт и Azul Systems. Они содержат все компоненты для сборки и запуска приложений Java SE. Вы можете установить JDK со страницы [установки Java JDK](/azure/developer/java/fundamentals/java-jdk-long-term-support).

Обновления основной версии будут предоставляться с помощью новых параметров среды выполнения в службе приложений Azure. Пользователи, устанавливающие более новые версии Java посредством настройки развернутой службы приложений, несут ответственность за тестирование выбранных обновлений для основного номера версии и их соответствие своим потребностям.

Каждый квартал в поддерживаемые пакеты JDK автоматически вносятся исправления. Это происходит в январе, апреле, июле и октябре. Дополнительные сведения об Java в Azure см. в [этом документе поддержки](/azure/developer/java/fundamentals/java-jdk-long-term-support).

### <a name="security-updates"></a>Обновления для системы безопасности

Исправления для устранения серьезных уязвимостей в системе безопасности будут выпускаться по мере выпуска компанией Azul Systems. "Серьезными" считаются уязвимости с базовым индексом не меньше 9.0 в [NIST Common Vulnerability Scoring System версии 2](https://nvd.nist.gov/vuln-metrics/cvss).

Tomcat 8,0 достигла [окончания срока жизни (конца строки) с 30 сентября 2018 г](https://tomcat.apache.org/tomcat-80-eol.html). Хотя среда выполнения по-прежнему доступна в службе приложений Azure, Azure не будет применять обновления безопасности к Tomcat 8,0. По возможности перенесите приложения на Tomcat 8,5 или 9,0. В службе приложений Azure доступны как Tomcat 8,5, так и 9,0. Дополнительные сведения см. на [официальном веб-сайте Tomcat](https://tomcat.apache.org/whichversion.html) . 

### <a name="deprecation-and-retirement"></a>Нерекомендуемые версии и прекращение использования

Если планируется прекращение использования какой-либо поддерживаемой среды выполнения Java, то разработчики для Azure, использующие эту среду выполнения, получат соответствующее уведомление по крайней мере за шесть месяцев до прекращения использования.


### <a name="local-development"></a>Локальная разработка

Разработчики могут скачать выпуск Production Edition пакета Azul Zulu Enterprise JDK для локальной разработки с [сайта загрузки Azul](https://www.azul.com/downloads/azure-only/zulu/).

### <a name="development-support"></a>Поддержка разработки

Поддержка продуктов для [Zulu JDK, поддерживаемых Azure](https://www.azul.com/downloads/azure-only/zulu/) , доступна в корпорации Майкрософт при разработке для Azure или [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) с [полным планом поддержки Azure](https://azure.microsoft.com/support/plans/).

## <a name="next-steps"></a>Дальнейшие действия

Посетите центр [Azure для разработчиков Java](/java/azure/), чтобы найти краткие руководства Azure, руководства и справочную документацию по Java.

Ответы на общие вопросы об использовании службы приложений для Linux, не относящиеся к разработке для Java, можно найти в разделе [вопросов и ответов о службе приложений для Linux](faq-app-service-linux.md).

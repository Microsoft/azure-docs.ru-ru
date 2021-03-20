---
title: Создание контейнера для Apache Tomcat в Linux
description: Создание контейнера Linux для предоставления приложения, работающего на сервере Apache Tomcat в Service Fabric. Создание образа Docker с приложением и сервером Apache Tomcat, отправка этого образа в реестр контейнеров, сборка и развертывание приложения-контейнера Service Fabric.
ms.topic: conceptual
ms.date: 6/08/2018
ms.author: pepogors
ms.openlocfilehash: 3de97bc277195dff2daf5868c0eb9aec5d6e27c0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96534035"
---
# <a name="create-service-fabric-container-running-apache-tomcat-server-on-linux"></a>Создание контейнера Service Fabric в Linux с сервером Apache Tomcat
Apache Tomcat — это популярная реализация технологий Java Servlet и Java Server с открытым кодом. В этой статье показано, как создать контейнер с Apache Tomcat и простым веб-приложением, а затем развернуть этот контейнер в кластере Service Fabric под управлением Linux и подключить его к веб-приложению.  

Дополнительные сведения об Apache Tomcat см. на [главной странице Apache Tomcat](https://tomcat.apache.org/). 

## <a name="prerequisites"></a>Предварительные условия
* Компьютер для разработки, на котором установлено ПО, перечисленное ниже.
  * [Service Fabric пакет SDK и средства](service-fabric-get-started-linux.md).
  * [Предварительные выпуски](https://docs.docker.com/engine/installation/#prior-releases). 
  * [Интерфейс командной строки Service Fabric](service-fabric-cli.md)

* Реестр контейнеров в службе "Реестр контейнеров Azure". Вы можете создать реестр контейнеров в подписке Azure с помощью [портала Azure](../container-registry/container-registry-get-started-portal.md) или [Azure CLI](./service-fabric-tutorial-create-container-images.md#deploy-azure-container-registry). 

## <a name="build-a-tomcat-image-and-run-it-locally"></a>Создание образа Tomcat и его запуск в локальной среде
Выполните действия, описанные в этом разделе, чтобы создать образ Docker на основе образа Apache Tomcat, включить в него простое веб-приложение и запустить этот образ в контейнере в локальной системе. 
 
1. Клонируйте репозиторий примеров для [начала работы с Java в Service Fabric](https://github.com/Azure-Samples/service-fabric-java-getting-started) на компьютер разработки.

   ```bash
   git clone https://github.com/Azure-Samples/service-fabric-java-getting-started.git
   ```

1. На сервере Apache Tomcat перейдите в каталог с примером (*service-fabric-java-getting-started/container-apache-tomcat-web-server-sample*):

   ```bash
   cd service-fabric-java-getting-started/container-apache-tomcat-web-server-sample
   ```

1. Создайте файл Docker на основе официального [образа Tomcat](https://hub.docker.com/_/tomcat/), который размещен в центре Docker и в примере сервера Tomcat. В каталоге *service-fabric-java-getting-started/container-apache-tomcat-web-server-sample* создайте файл с именем *Dockerfile* (без расширения). Добавьте следующий код в *Dockerfile* и сохраните изменения:

   ```
   FROM library/tomcat

   EXPOSE 8080

   COPY ./ApacheTomcat /usr/local/tomcat
   ```

   Дополнительные сведения см. в [справочнике по Dockerfile](https://docs.docker.com/engine/reference/builder/).


4. Войдите в DOCKER и выполните `docker build` команду, чтобы создать образ, запускающий веб-приложение:

   ```bash
   docker login
   docker build . -t tomcattest
   ```

   Эта команда создает новый образ по инструкциям из Dockerfile и присваивает этому образу имя (тэг) `tomcattest` с помощью атрибута -t. Чтобы создать образ контейнера, необходимо скачать из центра Docker базовый образ. Именно к нему будут добавлено приложение. 

   После выполнения команды сборки выполните команду `docker images`, чтобы получить информацию о новом образе:

   ```bash
   $ docker images
    
   REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
   tomcattest                    latest              86838648aab6        2 minutes ago       194 MB
   ```

5. Убедитесь, что контейнерное приложение успешно запускается локально, прежде чем отправлять его в реестр контейнеров:
 
   ```bash
   docker run -itd --name tomcat-site -p 8080:8080 tomcattest.
   ```
   
   * Атрибут `--name` задает имя контейнера, которое вы можете использовать вместо идентификатора при обращении к этому контейнеру.
   * Атрибут `-p` задает сопоставление портов между контейнером и ОС узла. 

   > [!Note]
   > Порт, который вы открываете с помощью параметра `-p`, должен совпадать с тем портом, на котором приложение Tomcat ожидает передачи запросов. В этом примере в файле *ApacheTomcat/conf/server.xml* настроен соединитель, который ожидает передачи запросов HTTP через порт 8080. Этот порт сопоставлен с портом 8080 в узле. 

   Сведения о других параметрах можно найти в [документации по запуску Docker](https://docs.docker.com/engine/reference/commandline/run/).

1. Чтобы протестировать созданный контейнер, откройте браузер и введите один из указанных ниже URL-адресов. Вы увидите вариант приветственного экрана "Hello World" для каждого URL-адреса.

   - `http://localhost:8080/hello` 
   - `http://localhost:8080/hello/sayhello` 
   - `http://localhost:8080/hello/sayhi` 

   ![Hello world /sayhi](./media/service-fabric-get-started-tomcat/hello.png)

2. Остановите контейнер и удалите его с компьютера разработки:

   ```bash
   docker stop tomcat-site
   docker rm tomcat-site
   ```

## <a name="push-the-tomcat-image-to-your-container-registry"></a>Отправка образа Tomcat в реестр контейнеров
Теперь, убедившись, что образ Tomcat выполняется в контейнере на компьютере разработчика, отправьте его в репозиторий в реестре контейнеров, чтобы [снизить нагрузку](../container-registry/buffer-gate-public-content.md) на рабочие процессы разработки и развертывания образов. В этой статье для хранения образа используется реестр контейнеров Azure, но вы можете использовать любой другой реестр контейнеров, внеся минимальные изменения в эту процедуру. В этой статье предполагается, что реестр имеет краткое имя *myregistry* и полное имя myregistry.azurecr.io. Измените эти значения в соответствии с параметрами вашей системы. 

1. Выполните команду `docker login` , чтобы войти в реестр контейнеров с [учетными данными реестра](../container-registry/container-registry-authentication.md).

   Следующая команда передает идентификатор и пароль [субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory. Например, назначение субъекта-службы для реестра позволяет автоматизировать некоторые сценарии. Также можно выполнить вход с помощью имени пользователя и пароля реестра.

   ```bash
   docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
   ```

2. Следующая команда создает тег (псевдоним) образа с полным путем к вашему реестру. Чтобы избежать беспорядка в корне реестра, эта команда помещает образ в пространство имен `samples`.

   ```bash
   docker tag tomcattest myregistry.azurecr.io/samples/tomcattest
   ```

3. Отправьте образ в реестр контейнеров:

   ```bash
   docker push myregistry.azurecr.io/samples/tomcattest
   ```

## <a name="build-and-deploy-the-service-fabric-container-application"></a>Сборка и развертывание приложения контейнера Service Fabric
Теперь, когда вы отправили образ Tomcat в реестр контейнеров, можно собрать и развернуть приложение контейнера Service Fabric, которое извлекает из реестра образ Tomcat и запускает его в вашем кластере в качестве контейнерной службы. 

1. Создайте новый каталог за пределами локальной копии (то есть не в каталоге *service-fabric-java-getting-started* и не в его дочерних каталогах). Перейдите в этот каталог и создайте шаблон для приложения контейнера с помощью Yeoman: 

   ```bash
   yo azuresfcontainer 
   ```
   Введите следующие значения в ответ на соответствующие запросы:

   * Name your application (Имя приложения): ServiceFabricTomcat
   * Name of the application service (Имя службы приложения): TomcatService
   * Input the Image Name (Имя образа): укажите URL-адрес образа контейнера, размещенного в реестре контейнеров (например, myregistry.azurecr.io/samples/tomcattest).
   * Commands (Команды): оставьте это поле пустым. Так как в образе определена точка входа рабочей нагрузки, вам не нужно явно указывать входные команды, выполняемые внутри контейнера, которые обеспечат выполнение контейнера после запуска.
   * Number of instances of guest container application (Число экземпляров гостевого приложения контейнера): 1

   ![Генератор Service Fabric Yeoman для контейнеров](./media/service-fabric-get-started-tomcat/yo-generator.png)

10. В манифест службы (*ServiceFabricTomcat/ServiceFabricTomcat/TomcatServicePkg/ServiceManifest.xml*) добавьте приведенный ниже код XML в корне тега **ServiceManfest**, чтобы открыть порт, передачи запросов через который ожидает приложение. Тег **Endpoint** объявляет протокол и порт для конечной точки. В этой статье контейнерная служба ожидает передачи данных через порт 8080. 

   ```xml
   <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
       listen. Please note that if your service is partitioned, this port is shared with 
       replicas of different partitions that are placed in your code. -->
      <Endpoint Name="endpointTest" Port="8080" Protocol="tcp"/>
    </Endpoints>
   </Resources>
   ```

11. В манифесте приложения (*ServiceFabricTomcat/ServiceFabricTomcat/ApplicationManifest.xml*) добавьте приведенный ниже код XML в тег **ServiceManifestImport**. Замените **AccountName** и **Password** в теге **репоситорикредентиалс** именем реестра контейнеров и паролем, необходимым для входа в него.

   ```xml
   <Policies>
    <ContainerHostPolicies CodePackageRef="Code">
      <PortBinding ContainerPort="8080" EndpointRef="endpointTest"/>
      <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
    </ContainerHostPolicies>
   </Policies>
   ```

   Тег **ContainerHostPolicies** задает политики активации для узлов контейнеров.
    
   * Тег **PortBinding** настраивает политику сопоставления портов для контейнера. Атрибут **ContainerPort** имеет значение 8080, так как контейнер предоставляет для подключений порт 8080, как указано в Dockerfile. Атрибут **EndpointRef** имеет значение "endpointTest", то есть содержит имя конечной точки, настроенной на предыдущем шаге в манифесте службы. Таким образом, поступающие в порт 8080 запросы к службе передаются в порт 8080 в контейнере. 
   * Тег **RepositoryCredentials** задает учетные данные, которые контейнер использует для аутентификации в (закрытом) репозитории, из которого он извлекает образ. Эта политика не требуется, если образ извлекается из общедоступного репозитория.
    

12. Перейдите в папку *ServiceFabricTomcat* и подключитесь к кластеру Service Fabric. 

   * Подключитесь к удаленному кластеру Service Fabric, выполнив такую команду:

     ```bash
     sfctl cluster select --endpoint http://localhost:19080
     ```
    
   * Чтобы подключиться к защищенному кластеру Azure, проверьте наличие сертификата клиента в PEM-файле в каталоге *ServiceFabricTomcat*, а затем выполните следующую команду: 

     ```bash
     sfctl cluster select --endpoint https://PublicIPorFQDN:19080 -pem your-certificate.pem -no-verify
     ```
     В приведенной выше команде замените `your-certificate.pem` правильным именем файла сертификата клиента. В средах разработки и тестирования в качестве сертификата клиента обычно используют сертификат кластера. Если сертификат не является самозаверяющим, не указывайте параметр `-no-verify`. 
       
     Сертификаты кластера обычно сохраняются локально в PFX-файлах. Если у вас нет сертификата в PEM-формате, выполните следующую команду, чтобы создать PEM-файл из PFX-файла:

     ```bash
     openssl pkcs12 -in your-certificate.pfx -out your-certificate.pem -nodes -passin pass:your-pfx-password
     ```

     Если PFX-файл не защищен паролем, укажите `-passin pass:` в качестве последнего параметра.


13. Запустите включенный в шаблон скрипт установки, чтобы развернуть приложение в кластере. Этот скрипт позволяет скопировать пакет приложения в хранилище образов кластера, зарегистрировать тип приложения и создать экземпляр приложения.

     ```bash
     ./install.sh
     ```

   После выполнения скрипта установки откройте в окне браузера страницу Service Fabric Explorer:
    
   * В локальном кластере используйте значение `http://localhost:19080/Explorer` (замените *localhost* значением частного IP-адреса виртуальной машины, если вы используете Vagrant под управлением Mac OS X).
   * В защищенном кластере Azure используйте значение `https://PublicIPorFQDN:19080/Explorer`. 
    
   Разверните узел **Приложения**. Здесь вы увидите новую запись **ServiceFabricTomcatType** для типа приложения и еще одну для первого экземпляра этого типа. Запаситесь терпением: полное развертывание приложения может занять несколько минут.

   ![Service Fabric Explorer](./media/service-fabric-get-started-tomcat/service-fabric-explorer.png)


1. Чтобы открыть приложение на сервере Tomcat, откройте окно браузера и введите один из указанных ниже URL-адресов. Если вы развернули приложение в локальном кластере, используйте значение *localhost* для параметра *PublicIPorFQDN*. Вы увидите вариант приветственного экрана "Hello World" для каждого URL-адреса.

   * http://PublicIPorFQDN:8080/hello  
   * http://PublicIPorFQDN:8080/hello/sayhello
   * http://PublicIPorFQDN:8080/hello/sayhi

## <a name="clean-up"></a>Очистка
Чтобы удалить экземпляр приложения из кластера и отменить регистрацию типа приложения, используйте предоставленный в шаблоне скрипт удаления.

```bash
./uninstall.sh
```

После передачи образа в реестр контейнеров можно удалить локальный образ с компьютера для разработки:

```
docker rmi tomcattest
docker rmi myregistry.azurecr.io/samples/tomcattest
```

## <a name="next-steps"></a>Дальнейшие действия
* Краткую инструкцию по настройке дополнительных возможностей контейнера Linux вы найдете в статье [Создание первого контейнера-приложения Service Fabric в Linux](service-fabric-get-started-containers-linux.md).
* Подробные инструкции приведены в руководстве по [созданию образов контейнеров в кластере Service Fabric в Linux](service-fabric-tutorial-create-container-images.md).
* Дополнительные сведения о запуске [контейнеров в Service Fabric](service-fabric-containers-overview.md).



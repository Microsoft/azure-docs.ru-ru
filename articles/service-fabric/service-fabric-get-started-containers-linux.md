---
title: Создание приложения контейнера Azure Service Fabric в Linux
description: Создание первого приложения-контейнера Linux в Azure Service Fabric. Создание образа Docker с приложением, отправка образа в реестр контейнеров, сборка и развертывание приложения-контейнера Service Fabric.
ms.topic: conceptual
ms.date: 1/4/2019
ms.custom: devx-track-python
ms.openlocfilehash: 0481cc2d36f7882bbd8eea9b984c3dc388de5dee
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96534086"
---
# <a name="create-your-first-service-fabric-container-application-on-linux"></a>Создание первого контейнера-приложения Service Fabric в Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

Чтобы запустить существующее приложение в контейнере Linux кластера Service Fabric не требуется вносить изменения в приложение. В этой статье описано создание образа Docker, содержащего веб-приложение [Flask](http://flask.pocoo.org/) Python, и его развертывание в кластер Service Fabric. Кроме того, вы предоставите общий доступ к контейнерному приложению через [реестр контейнеров Azure](../container-registry/index.yml). Для работы с этой статьей необходимо знание основных понятий Docker. Чтобы ознакомиться с Docker, см. этот [обзор Docker](https://docs.docker.com/engine/understanding-docker/).

> [!NOTE]
> Эта статья касается среды разработки Linux.  Среда выполнения кластера Service Fabric и среда выполнения Docker должны работать под управлением одной операционной системы.  Контейнеры Linux нельзя запускать в кластере Windows.

## <a name="prerequisites"></a>Предварительные условия
* Компьютер для разработки, на котором установлено ПО, перечисленное ниже.
  * [Service Fabric пакет SDK и средства](service-fabric-get-started-linux.md).
  * [Предварительные выпуски](https://docs.docker.com/engine/installation/#prior-releases). 
  * [Интерфейс командной строки Service Fabric](service-fabric-cli.md)

* Кластер Linux с тремя или более узлами.

* Реестр контейнеров Azure. [Создайте реестр контейнеров](../container-registry/container-registry-get-started-portal.md) в своей подписке Azure. 

## <a name="define-the-docker-container"></a>Определение образа контейнера Docker
Создайте образ на основе [образа Python](https://hub.docker.com/_/python/) из репозитория Docker Hub. 

Определите образ Docker в файле Dockerfile. Файл Dockerfile содержит инструкции по настройке среды внутри контейнера, загрузке приложения, которое необходимо выполнить, и сопоставлению портов. Dockerfile — это входные данные для команды `docker build`, которая создает образ. 

Создайте пустой каталог и файл *Dockerfile* (без расширения файла). Добавьте следующий код в *Dockerfile* и сохраните изменения:

```
# Use an official Python runtime as a base image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Дополнительные сведения см. в [справочнике по Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="create-a-basic-web-application"></a>Создание базового веб-приложения
Создайте веб-приложение Flask, ожидающее передачи данных в порт 80, возвращающее строку Hello, World!. В том же каталоге создайте файл *requirements.txt*. Добавьте следующее и сохраните изменения:
```
Flask
```

Кроме того, создайте файл *app.py* и добавьте следующий фрагмент кода:

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def hello():

    return 'Hello World!'


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

## <a name="login-to-docker-and-build-the-image"></a>Войдите в DOCKER и создайте образ.

Далее мы создадим образ, который запускает веб-приложение. При получении общедоступных образов из DOCKER (например `python:2.7-slim` , в нашей Dockerfile) рекомендуется выполнять проверку подлинности в учетной записи центра DOCKER вместо анонимного запроса на включение внесенных изменений.

> [!NOTE]
> При частом анонимных запросах на включение внесенных изменений могут возникнуть ошибки DOCKER, аналогичные `ERROR: toomanyrequests: Too Many Requests.` или `You have reached your pull rate limit.` прошедшие проверку подлинности в концентраторе DOCKER, чтобы предотвратить эти ошибки. Дополнительные сведения см. [в статье Управление общедоступным содержимым с помощью реестра контейнеров Azure](../container-registry/buffer-gate-public-content.md) .

Откройте окно PowerShell и перейдите в каталог, содержащий Dockerfile. Затем выполните следующие команды:

```
docker login
docker build -t helloworldapp .
```

Эта команда создает образ согласно инструкциям в Dockerfile, и присваивает образу имя `helloworldapp` (-t — добавление тега). Чтобы создать образ контейнера, необходимо скачать из Docker Hub базовый образ, к которому добавляется приложение. 

После выполнения команды сборки выполните команду `docker images`, чтобы получить информацию о новом образе:

```bash
$ docker images
    
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              86838648aab6        2 minutes ago       194 MB
```

## <a name="run-the-application-locally"></a>Локальный запуск приложения
Убедитесь, что контейнерное приложение выполняется локально перед принудительной отправкой реестр контейнеров. 

Запустите приложение, сопоставив порт 4000 компьютера с предоставленным портом 80 контейнера:

```bash
docker run -d -p 4000:80 --name my-web-site helloworldapp
```

Параметр *name* присваивает имя запущенному контейнеру (вместо идентификатора контейнера).

Подключитесь к запущенному контейнеру. Откройте веб-браузер, указывающий на IP-адрес, возвращенный через порт 4000, например "http: \/ /ЛОКАЛХОСТ: 4000". Вы должны увидеть заголовок Hello World! в браузере.

![Пример "Здравствуй,][hello-world]

Чтобы остановить контейнер, выполните следующую команду:

```bash
docker stop my-web-site
```

Удаление контейнера с компьютера для разработки:

```bash
docker rm my-web-site
```

## <a name="push-the-image-to-the-container-registry"></a>Отправка образа в реестр контейнеров
Убедившись, что приложение запускается в Docker, отправьте образ в реестр в реестре контейнеров Azure.

Выполните команду `docker login` , чтобы войти в реестр контейнеров с [учетными данными реестра](../container-registry/container-registry-authentication.md).

Следующая команда передает идентификатор и пароль [субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory. Например, назначение субъекта-службы для реестра позволяет автоматизировать некоторые сценарии. Также можно выполнить вход с помощью имени пользователя и пароля реестра.

```bash
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

Следующая команда создает тег (псевдоним) образа с полным путем к вашему реестру. Чтобы избежать беспорядка в корне реестра, эта команда помещает образ в пространство имен `samples`.

```bash
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

Отправьте образ в реестр контейнеров:

```bash
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="package-the-docker-image-with-yeoman"></a>Упаковка образа Docker с помощью Yeoman
Пакет SDK Service Fabric для Linux включает в себя генератор [Yeoman](https://yeoman.io/), который упрощает создание приложения и добавление образа контейнера. Воспользуемся Yeoman, чтобы создать приложение с одним контейнером Docker — *SimpleContainerApp*.

Чтобы создать приложение-контейнер Service Fabric, откройте окно терминала и выполните `yo azuresfcontainer`. 

Присвойте имя приложению, например `mycontainer`, и службе приложения, например `myservice`.

В качестве имени образа предоставьте URL-адрес для образа контейнеров в реестре контейнеров (например, myregistry.azurecr.io/samples/helloworldapp). 

Так как в образе определена точка входа рабочей нагрузки, вам не нужно явно указывать входные команды, выполняемые внутри контейнера, которые обеспечат выполнение контейнера после запуска. 

Укажите число экземпляров — 1.

Укажите сопоставление портов в соответствующем формате. Для выполнения задач в этой статье необходимо указать ```80:4000``` для сопоставления портов. После этого все входящие запросы, поступающие в порт 4000 на хост-компьютере, будут перенаправляться в порт 80 в контейнере.

![Генератор Service Fabric Yeoman для контейнеров][sf-yeoman]

## <a name="configure-container-repository-authentication"></a>Настройка аутентификации в репозитории

Сведения о настройке различных типов проверки подлинности для загрузки образа контейнера см. в статье [Проверка подлинности репозитория контейнеров](configure-container-repository-credentials.md).

## <a name="configure-isolation-mode"></a>Настройка режима изоляции
В выпуске среды выполнения 6,3 изоляция виртуальных машин поддерживается для контейнеров Linux, тем самым обеспечивая поддержку двух режимов изоляции для контейнеров: Process и Hyper-V. В режиме изоляции Hyper-V ядра изолированы между каждым контейнером и узлом контейнера. Изоляция Hyper-V реализуется с помощью [очистки контейнеров](https://software.intel.com/en-us/articles/intel-clear-containers-2-using-clear-containers-with-docker). Режим изоляции кластеров Linux указывается в элементе `ServicePackageContainerPolicy` в файле манифеста приложения. Вы можете указать следующие режимы изоляции: `process`, `hyperv` и `default`. По умолчанию используется режим изоляции процессов. В указанном ниже фрагменте кода показано, как режим изоляции указывается в файле манифеста приложения.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyServicePkg" ServiceManifestVersion="1.0.0"/>
      <Policies>
        <ServicePackageContainerPolicy Hostname="votefront" Isolation="hyperv">
          <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
        </ServicePackageContainerPolicy>
    </Policies>
  </ServiceManifestImport>
```


## <a name="configure-resource-governance"></a>Настройка управления ресурсами
[Управление ресурсами](service-fabric-resource-governance.md) позволяет ограничить ресурсы, которые контейнер может использовать на узле. Элемент `ResourceGovernancePolicy`, определяемый в манифесте приложения, позволяет объявить ограничения для ресурсов, доступных из пакета кода службы. Ограничения можно задать для следующих ресурсов: Memory, MemorySwap, CpuShares (относительный вес ЦП), MemoryReservationInMB, BlkioWeight (относительный вес BlockIO). В этом примере пакет службы Guest1Pkg получает одно ядро на узлах кластера, где он размещается. Ограничения по памяти абсолютны, так что пакет кода получает только 1024 МБ (и имеет соответствующие мягкие гарантии резервирования). Пакеты кода (контейнеры или процессы) не могут выделить больше памяти, чем задано ограничением, а при попытке сделать это возникнет проблема нехватки памяти. Чтобы принудительное ограничение ресурсов работало, для всех пакетов кода в пакете службы должны быть указаны ограничения по памяти.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="MyServicePKg" ServiceManifestVersion="1.0.0" />
  <Policies>
    <ServicePackageResourceGovernancePolicy CpuCores="1"/>
    <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
  </Policies>
</ServiceManifestImport>
```




## <a name="configure-docker-healthcheck"></a>Настройка инструкции HEALTHCHECK в Docker 

Начиная с версии 6.1 Service Fabric автоматически интегрирует события [Docker HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck) в отчеты о работоспособности системы. Это означает, что если в вашем контейнере включена инструкция **HEALTHCHECK**, то Service Fabric будет отправлять отчет о работоспособности при каждом изменении состояния контейнера согласно сведениям Docker. В [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) будет отображаться состояние работоспособности **ОК**, если значение *health_status* равно *healthy*. Если же значение *health_status* равно *unhealthy*, отобразится **предупреждение**. 

Начиная с последнего обновления версии 6.4, вы можете указать, что оценки DOCKER HEALTHCHECK должны выводиться как ошибки. Если этот параметр включен, появится отчет о работоспособности **ОК** , если *health_status* *работоспособен* и при *неработоспособности* *health_status* появится **сообщение об ошибке** .

В файле Dockerfile, который используется при создании образа контейнера, должна содержаться инструкция **HEALTHCHECK**, указывающая на фактическую проверку для отслеживания работоспособности контейнера.

![На снимке экрана показаны сведения о развернутом пакете службы Нодесервицепаккаже.][1]

![HealthCheckUnhealthyApp][2]

![HealthCheckUnhealthyDsp][3]

Поведение инструкции **HEALTHCHECK** можно настроить для каждого контейнера, указав параметры **HealthConfig** в составе политик **ContainerHostPolicies** в манифесте приложения.

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="ContainerServicePkg" ServiceManifestVersion="2.0.0" />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <HealthConfig IncludeDockerHealthStatusInSystemHealthReport="true"
              RestartContainerOnUnhealthyDockerHealthStatus="false" 
              TreatContainerUnhealthyStatusAsError="false" />
      </ContainerHostPolicies>
    </Policies>
</ServiceManifestImport>
```
По умолчанию *инклудедоккерхеалсстатусинсистемхеалсрепорт* имеет значение **true**, *RestartContainerOnUnhealthyDockerHealthStatus* имеет значение **false**, а *треатконтаинерунхеалсистатусасеррор* — **значение false**. 

Если для *RestartContainerOnUnhealthyDockerHealthStatus* задано значение **true**, контейнер, для которого несколько раз отображалось неработоспособное состояние, перезапускается (возможно, на других узлах).

Если для *треатконтаинерунхеалсистатусасеррор* задано **значение true**, отчеты о работоспособности **ошибок** отобразятся, когда *health_status* контейнера *неработоспособен*.

Если вы хотите отключить интеграцию с **HEALTHCHECK** во всем кластере Service Fabric, задайте для [EnableDockerHealthCheckIntegration](service-fabric-cluster-fabric-settings.md) значение **false**.

## <a name="deploy-the-application"></a>Развертывание приложения
Созданное приложение можно развернуть в локальном кластере с помощью интерфейса командной строки Service Fabric.

Подключитесь к удаленному кластеру Service Fabric.

```bash
sfctl cluster select --endpoint http://localhost:19080
```

С помощью скрипта установки, который включен в шаблоны https://github.com/Azure-Samples/service-fabric-containers/, скопируйте пакет приложения в хранилище образов кластера, зарегистрируйте тип приложения и создайте экземпляр приложения.


```bash
./install.sh
```

Откройте браузер и перейдите в Service Fabric Explorer по адресу http: \/ /ЛОКАЛХОСТ: 19080/Explorer (замените localhost на частный IP-адрес виртуальной машины, если используется Vagrant на Mac OS X). Разверните узел приложения. Вы увидите одну запись для типа приложения и еще одну — для первого экземпляра этого типа.

Подключитесь к запущенному контейнеру. Откройте веб-браузер, указывающий на IP-адрес, возвращенный через порт 4000, например "http: \/ /ЛОКАЛХОСТ: 4000". Вы должны увидеть заголовок Hello World! в браузере.

![Пример "Здравствуй,][hello-world]


## <a name="clean-up"></a>Очистка
Для удаления экземпляра приложения из локального кластера разработки и отмены регистрации типа приложения используйте сценарий удаления, предоставленный в шаблоне.

```bash
./uninstall.sh
```

После передачи образа в реестр контейнеров можно удалить локальный образ с компьютера для разработки:

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Полные примеры манифестов службы и приложения Service Fabric
Ниже приведены полные коды манифестов службы и приложения, используемых в этой статье.

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="myservicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="myserviceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying containers 
      to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->
    
    <EnvironmentVariables>
      <!--
      <EnvironmentVariable Name="VariableName" Value="VariableValue"/>
      -->
    </EnvironmentVariables>
    
  </CodePackage>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="myServiceTypeEndpoint" UriScheme="http" Port="4000" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="mycontainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="myservicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
        <PortBinding ContainerPort="80" EndpointRef="myServiceTypeEndpoint"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="myservice">
      <!-- On a local development cluster, set InstanceCount to 1. On a multi-node production 
      cluster, set InstanceCount to -1 for the container service to run on every node in 
      the cluster.
      -->
      <StatelessService ServiceTypeName="myserviceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
## <a name="adding-more-services-to-an-existing-application"></a>Добавление дополнительных служб в существующее приложение

Чтобы добавить службу контейнеров в приложение, созданное с использованием yeoman, выполните следующие действия:

1. Перейдите в корневой каталог существующего приложения. Например, `cd ~/YeomanSamples/MyApplication`, если `MyApplication` является приложением, созданным с помощью Yeoman.
2. Выполнить `yo azuresfcontainer:AddService`

<a id="manually"></a>


## <a name="configure-time-interval-before-container-is-force-terminated"></a>Настройка интервала времени перед принудительным завершением контейнера

Вы можете настроить интервал времени для среды выполнения перед удалением контейнера после начала удаления службы (или перехода на другой узел). При настройке интервала времени в контейнер отправляется команда `docker stop <time in seconds>`.  Дополнительные сведения см. в статье [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) (Остановка docker). Интервал времени ожидания указывается в разделе `Hosting`. В следующем фрагменте манифеста кластера показано, как задать интервал ожидания:


```json
{
        "name": "Hosting",
        "parameters": [
          {
                "name": "ContainerDeactivationTimeout",
                "value" : "10"
          },
          ...
        ]
}
```

Интервал времени по умолчанию установлен на 10 секунд. Так как эта конфигурация является динамической, файл конфигурации обновляет только кластер для которого обновляется время ожидания. 

## <a name="configure-the-runtime-to-remove-unused-container-images"></a>Настройка среды выполнения для удаления неиспользуемых образов контейнера

Можно настроить кластер Service Fabric для удаления неиспользуемых образов контейнера из узла. Эта конфигурация позволяет месту на диске перезаписываться, если на узле находится слишком много образов контейнера. Чтобы включить эту функцию, обновите раздел `Hosting` в манифесте кластера, как показано в следующем фрагменте кода: 


```json
{
        "name": "Hosting",
        "parameters": [
          {
                "name": "PruneContainerImages",
                "value": "True"
          },
          {
                "name": "ContainerImagesToSkip",
                "value": "mcr.microsoft.com/windows/servercore|mcr.microsoft.com/windows/nanoserver|mcr.microsoft.com/dotnet/framework/aspnet|..."
          }
          ...
          }
        ]
} 
```

Образы, которые не должны удаляться, можно указать в параметре `ContainerImagesToSkip`. 

## <a name="configure-container-image-download-time"></a>Настройка времени загрузки образа контейнера

Среда выполнения Service Fabric выделяет 20 минут для скачивания и извлечения образов контейнеров. Этого достаточно для большинства образов контейнеров. В случае с большими образами или при медленном сетевом подключении, возможно, потребуется увеличить время ожидания перед прерыванием загрузки и извлечения образа. Это значение можно задать с помощью атрибута **ContainerImageDownloadTimeout** в разделе **Hosting** манифеста кластера, как показано в следующем фрагменте кода:

```json
{
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
}
```


## <a name="set-container-retention-policy"></a>Настройка политики хранения контейнера

Чтобы упростить процесс диагностики сбоев при запуске контейнеров, Service Fabric (версии 6.1 и выше) поддерживает хранение контейнеров, работа которых завершилась или при запуске которых произошел сбой. Эту политику можно настроить в файле **ApplicationManifest.xml**, как показано в следующем фрагменте кода:

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

В атрибуте **ContainersRetentionCount** указывается число контейнеров, которые следует сохранить после сбоя. Если указано отрицательное значение, будут сохраняться все контейнеры после сбоя. Если значение **ContainersRetentionCount** не указано, контейнеры не будут сохраняться. Атрибут **ContainersRetentionCount** также поддерживает параметры приложения, поэтому пользователи могут указывать разные значения для тестовых и рабочих кластеров. Чтобы предотвратить перемещение службы контейнеров на другие узлы, при использовании этих функций используйте ограничения на размещение, указав для службы контейнеров конкретный узел. Удаление всех контейнеров, сохраненных с помощью этой функции, выполняется вручную.

## <a name="start-the-docker-daemon-with-custom-arguments"></a>Запуск управляющей программы Docker с пользовательскими аргументами

Используя среду выполнения Service Fabric версии 6.2 и выше, можно запустить управляющую программу Docker с пользовательскими аргументами. Если пользовательские аргументы указаны, Service Fabric не передает никакие другие аргументы модулю Docker, кроме аргумента `--pidfile`. Таким образом, не нужно передавать `--pidfile` в качестве аргумента. Кроме того, аргумент должен поддерживать прослушивание модулем Docker стандартного конвейера имен Windows (или сокета домена Unix в Linux), обеспечивая обмен данными между Service Fabric и управляющей программой. Пользовательские аргументы определяются в манифесте кластера в блоке **Размещение** раздела **ContainerServiceArguments**, как показано в следующем фрагменте: 
 

```json
{ 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
} 

```

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о запуске [контейнеров в Service Fabric](service-fabric-containers-overview.md).
* Ознакомьтесь с руководством [Развертывание приложения-контейнера .NET в Azure Service Fabric](service-fabric-host-app-in-a-container.md).
* Дополнительные сведения о [жизненном цикле приложения](service-fabric-application-lifecycle.md) Service Fabric.
* Извлечение [примеров кода контейнера Service Fabric](https://github.com/Azure-Samples/service-fabric-containers) на GitHub.

[hello-world]: ./media/service-fabric-get-started-containers-linux/HelloWorld.png
[sf-yeoman]: ./media/service-fabric-get-started-containers-linux/YoSF.png

[1]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[2]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png

---
title: Создание приложения контейнера Service Fabric Azure
description: Создание первого приложения-контейнера Windows в Azure Service Fabric. Создайте образ DOCKER с помощью приложения Python, отправьте образ в реестр контейнеров, а затем выполните сборку и развертывание контейнера в Azure Service Fabric.
ms.topic: conceptual
ms.date: 01/25/2019
ms.custom: devx-track-python
ms.openlocfilehash: 197423670ffe05f15fdc5bfd351efdfba33b53cd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96533780"
---
# <a name="create-your-first-service-fabric-container-application-on-windows"></a>Создание первого контейнера-приложения Service Fabric в Windows

> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started-containers.md)
> * [Linux](service-fabric-get-started-containers-linux.md)

Чтобы запустить существующее приложение в контейнере Windows кластера Service Fabric, не требуется вносить изменения в приложение. В этой статье описывается создание образа DOCKER, содержащего веб-приложение Python [Flask](http://flask.pocoo.org/) , и его развертывание в кластере Azure Service Fabric. Кроме того, вы предоставите общий доступ к контейнерному приложению через [реестр контейнеров Azure](../container-registry/index.yml). Для работы с этой статьей необходимо знание основных понятий Docker. Чтобы ознакомиться с Docker, см. этот [обзор Docker](https://docs.docker.com/engine/understanding-docker/).

> [!NOTE]
> Эта статья касается среды разработки Windows.  Среда выполнения кластера Service Fabric и среда выполнения Docker должны работать под управлением одной операционной системы.  Контейнеры Windows нельзя запускать в кластере Linux.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

* Компьютер для разработки, на котором установлено ПО, перечисленное ниже.
  * Visual Studio 2015 или Visual Studio 2019.
  * [Service Fabric пакет SDK и средства](service-fabric-get-started.md).
  *  Docker для Windows. [Получите DOCKER CE для Windows (стабильный)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description). После установки и запуска Docker щелкните правой кнопкой мыши значок в области уведомлений и выберите **Switch to Windows containers** (Переключиться на контейнеры Windows). Это необходимо для запуска образов Docker на базе Windows.

* Кластер Windows с тремя или более узлами под управлением Windows Server с контейнерами. 

  В этой статье версия (сборка) Windows Server с контейнерами, запущенная на узлах кластера, должна соответствовать версии на компьютере разработки. Это связано с тем, что вы создаете образ Docker на компьютере разработки и между версиями ОС контейнера и ОС узла, где он развертывается, есть ограничения совместимости. Дополнительные сведения см. в разделе [Совместимость ОС контейнера Windows Server и ОС узла](#windows-server-container-os-and-host-os-compatibility). 
  
    Чтобы определить версию Windows Server с контейнерами, необходимыми для кластера, выполните `ver` команду из командной строки Windows на компьютере разработки. Перед [созданием кластера](service-fabric-cluster-creation-via-portal.md)ознакомьтесь с разрядом о [совместимости ОС контейнера Windows Server и операционной системы узла](#windows-server-container-os-and-host-os-compatibility) .

* Реестр контейнеров Azure. [Создайте реестр контейнеров](../container-registry/container-registry-get-started-portal.md) в своей подписке Azure.

> [!NOTE]
> Поддерживается развертывание контейнеров в кластере Service Fabric под управлением Windows 10.  Сведения о настройке Windows 10 для запуска контейнеров Windows см.в [этой статье](service-fabric-how-to-debug-windows-containers.md).

> [!NOTE]
> Service Fabric 6.2 и более поздних версий поддерживает развертывание контейнеров в кластерах под управлением Windows Server версии 1709.

## <a name="define-the-docker-container"></a>Определение образа контейнера Docker

Создайте образ на основе [образа Python](https://hub.docker.com/_/python/) из репозитория Docker Hub.

Определите образ Docker в файле Dockerfile. Файл Dockerfile содержит инструкции по настройке среды внутри контейнера, загрузке приложения, которое необходимо выполнить, и сопоставлению портов. Dockerfile — это входные данные для команды `docker build`, которая создает образ.

Создайте пустой каталог и файл *Dockerfile* (без расширения файла). Добавьте следующий код в *Dockerfile* и сохраните изменения:

```
# Use an official Python runtime as a base image
FROM python:2.7-windowsservercore

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

Дополнительные сведения см. в [справочнике по Dockerfile](https://docs.docker.com/engine/reference/builder/).

## <a name="create-a-basic-web-application"></a>Создание базового веб-приложения
Создайте веб-приложение Flask, ожидающее передачи данных в порт 80 и возвращающее `Hello World!`. В том же каталоге создайте файл *requirements.txt*. Добавьте следующее и сохраните изменения:
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

<a id="Build-Containers"></a>

## <a name="login-to-docker-and-build-the-image"></a>Войдите в DOCKER и создайте образ.

Далее мы создадим образ, который запускает веб-приложение. При получении общедоступных образов из DOCKER (например `python:2.7-windowsservercore` , в нашей Dockerfile) рекомендуется выполнять проверку подлинности в учетной записи центра DOCKER вместо анонимного запроса на включение внесенных изменений.

> [!NOTE]
> При частом анонимных запросах на включение внесенных изменений могут возникнуть ошибки DOCKER, аналогичные `ERROR: toomanyrequests: Too Many Requests.` или `You have reached your pull rate limit.` прошедшие проверку подлинности в концентраторе DOCKER, чтобы предотвратить эти ошибки. Дополнительные сведения см. [в статье Управление общедоступным содержимым с помощью реестра контейнеров Azure](../container-registry/buffer-gate-public-content.md) .

Откройте окно PowerShell и перейдите в каталог, содержащий Dockerfile. Затем выполните следующие команды:

```
docker login
docker build -t helloworldapp .
```

Эта команда создает образ согласно инструкциям в Dockerfile, и присваивает образу имя `helloworldapp` (-t — добавление тега). Чтобы создать образ контейнера, необходимо скачать из Docker Hub базовый образ, к которому добавляется приложение. 

После выполнения команды сборки выполните команду `docker images`, чтобы получить информацию о новом образе:

```
$ docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
helloworldapp                 latest              8ce25f5d6a79        2 minutes ago       10.4 GB
```

## <a name="run-the-application-locally"></a>Локальный запуск приложения
Проверьте работу образа локально, прежде чем отправлять его в реестр контейнеров. 

Запустите приложение:

```
docker run -d --name my-web-site helloworldapp
```

Параметр *name* присваивает имя запущенному контейнеру (вместо идентификатора контейнера).

После запуска контейнера найдите его IP-адрес, чтобы иметь возможность подключаться к запущенному контейнеру из браузера:
```
docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" my-web-site
```

Если эта команда не возвращает ничего, выполните следующую команду и проверьте элемент **нетворксеттингс** -> **Networks** для IP-адреса:
```
docker inspect my-web-site
```

Подключитесь к запущенному контейнеру. Откройте веб-браузер, указывающий на возвращенный IP-адрес, например "http: \/ /172.31.194.61". Вы должны увидеть заголовок Hello World! в браузере.

Чтобы остановить контейнер, выполните следующую команду:

```
docker stop my-web-site
```

Удаление контейнера с компьютера для разработки:

```
docker rm my-web-site
```

<a id="Push-Containers"></a>
## <a name="push-the-image-to-the-container-registry"></a>Отправка образа в реестр контейнеров

Убедившись, что контейнер запускается на компьютере разработки, отправьте образ в реестр в реестре контейнеров Azure.

Выполните команду ``docker login`` , чтобы войти в реестр контейнеров с [учетными данными реестра](../container-registry/container-registry-authentication.md).

Следующая команда передает идентификатор и пароль [субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md) Azure Active Directory. Например, назначение субъекта-службы для реестра позволяет автоматизировать некоторые сценарии. Также можно выполнить вход с помощью имени пользователя и пароля реестра.

```
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

Следующая команда создает тег (псевдоним) образа с полным путем к вашему реестру. Чтобы избежать беспорядка в корне реестра, эта команда помещает образ в пространство имен ```samples```.

```
docker tag helloworldapp myregistry.azurecr.io/samples/helloworldapp
```

Отправьте образ в реестр контейнеров:

```
docker push myregistry.azurecr.io/samples/helloworldapp
```

## <a name="create-the-containerized-service-in-visual-studio"></a>Создание контейнерной службы в Visual Studio
Пакет SDK и средства для Service Fabric предоставляют шаблон службы для создания контейнерного приложения.

1. Запустите среду Visual Studio. Выберите **File** > **New** > **Project** ( Файл > Создать > Проект).
2. Выберите **Приложение Service Fabric**, назовите его MyFirstContainer и нажмите кнопку **ОК**.
3. В списке **шаблонов служб** выберите значение **Container** (Контейнер).
4. В поле **Имя образа** введите myregistry.azurecr.io/samples/helloworldapp — это образ, который вы отправили в репозиторий контейнера.
5. Присвойте службе имя и нажмите кнопку **ОК**.

## <a name="configure-communication"></a>настройка обмена данными;
Контейнерной службе для обмена данными требуется конечная точка. В файле ServiceManifest.xml добавьте элемент `Endpoint` и укажите протокол, порт и тип. В этом примере используется фиксированный порт 8081. Если порт не указан, выбирается случайный порт из диапазона портов приложения. 

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
  </Endpoints>
</Resources>
```
> [!NOTE]
> Дополнительные конечные точки для службы могут быть добавлены путем объявления дополнительных элементов EndPoint с соответствующими значениями свойств. Для каждого отдельного порта может быть объявлено только одно значение протокола.

При определении конечной точки Service Fabric публикует ее в службе именования. Другие службы, работающие в кластере, могут разрешать этот контейнер. Кроме того, возможен обмен данными между контейнерами с помощью [обратного прокси-сервера](service-fabric-reverseproxy.md). Для этого нужно указать в переменных среды HTTP-порт прослушивания обратного прокси-сервера и имена служб, с которыми будет выполняться обмен данными.

Служба ожидает передачи данных через определенный порт (в этом примере — 8081). Если приложение развертывается в кластере в Azure, то приложение и кластер запускаются под контролем Azure Load Balancer. Чтобы входящий трафик поступал в службу, необходимо открыть порт приложения в Azure Load Balancer.  Этот порт можно открыть в Azure Load Balancer с помощью [скрипта PowerShell](./scripts/service-fabric-powershell-open-port-in-load-balancer.md) или на [портале Azure](https://portal.azure.com).

## <a name="configure-and-set-environment-variables"></a>Настройка и установка переменных среды
Переменные среды можно указать для каждого пакета кода в манифесте служб. Эта функция доступна для всех служб, вне зависимости от того, как они развернуты: в качестве контейнеров, процессов или гостевых исполняемых файлов. Вы можете переопределить значения переменных среды в манифесте приложения или указать их в качестве параметров приложения во время развертывания.

Следующий фрагмент XML-кода из манифеста службы демонстрирует, как определить переменные среды для пакета кода.
```xml
<CodePackage Name="Code" Version="1.0.0">
  ...
  <EnvironmentVariables>
    <EnvironmentVariable Name="HttpGatewayPort" Value=""/>    
  </EnvironmentVariables>
</CodePackage>
```

Эти переменные среды можно переопределить в манифесте приложения:

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
  <EnvironmentOverrides CodePackageRef="FrontendService.Code">
    <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
  </EnvironmentOverrides>
  ...
</ServiceManifestImport>
```

## <a name="configure-container-port-to-host-port-mapping-and-container-to-container-discovery"></a>Настройка сопоставления порта контейнера с портом узла и межконтейнерного обнаружения
Настройте порт узла, который будет использоваться для обмена данными с контейнером. Привязка порта сопоставляет порт, на котором служба ожидает передачи данных в контейнере, с портом на узле. Добавьте элемент `PortBinding` в элемент `ContainerHostPolicies` в файле ApplicationManifest.xml. В этой статье у `ContainerPort` будет значение 80 (контейнер предоставляет порт 80, как указано в Dockerfile), а у `EndpointRef` — Guest1TypeEndpoint (конечная точка, определенная ранее в манифесте служб). Входящие запросы к службе через порт 8081 сопоставляются с портом 80 в контейнере.

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```
> [!NOTE]
> Дополнительные PortBindings для службы могут быть добавлены путем объявления дополнительных элементов PortBinding с соответствующими значениями свойств.

## <a name="configure-container-repository-authentication"></a>Настройка аутентификации в репозитории

Сведения о настройке различных типов проверки подлинности для загрузки образа контейнера см. в статье [Проверка подлинности репозитория контейнеров](configure-container-repository-credentials.md).

## <a name="configure-isolation-mode"></a>Настройка режима изоляции
Windows поддерживает два режима изоляции для контейнеров: режим изоляции процессов и Hyper-V. В режиме изоляции процесса все контейнеры, запущенные на одном хост-компьютере, совместно используют ядро и узел. В режиме изоляции Hyper-V все контейнеры Hyper-V и узлы контейнера используют отдельные ядра. Режим изоляции указывается в элементе `ContainerHostPolicies` в файле манифеста приложения. Вы можете указать следующие режимы изоляции: `process`, `hyperv` и `default`. По умолчанию на узлах Windows Server используется режим изоляции процессов. На узлах Windows 10 поддерживается только режим изоляции Hyper-V, поэтому контейнер работает в этом режиме независимо от параметра режима изоляции. В указанном ниже фрагменте кода показано, как режим изоляции указывается в файле манифеста приложения.

```xml
<ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
```
   > [!NOTE]
   > Режим изоляции Hyper-v доступен для номеров SKU Azure Ev3 и Dv3 с поддержкой вложенной виртуализации. 
   >
   >

## <a name="configure-resource-governance"></a>Настройка управления ресурсами
[Управление ресурсами](service-fabric-resource-governance.md) позволяет ограничить ресурсы, которые контейнер может использовать на узле. Элемент `ResourceGovernancePolicy`, определяемый в манифесте приложения, позволяет объявить ограничения для ресурсов, доступных из пакета кода службы. Ограничения можно задать для следующих ресурсов: Memory, MemorySwap, CpuShares (относительный вес ЦП), MemoryReservationInMB, BlkioWeight (относительный вес BlockIO). В этом примере пакет службы Guest1Pkg получает одно ядро на узлах кластера, где он размещается. Ограничения по памяти абсолютны, так что пакет кода получает только 1024 МБ (и имеет соответствующие мягкие гарантии резервирования). Пакеты кода (контейнеры или процессы) не могут выделить больше памяти, чем задано ограничением, а при попытке сделать это возникнет проблема нехватки памяти. Чтобы принудительное ограничение ресурсов работало, для всех пакетов кода в пакете службы должны быть указаны ограничения по памяти.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
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

![На снимке экрана показаны сведения о развернутом пакете службы Нодесервицепаккаже.][3]

![HealthCheckUnhealthyApp][4]

![HealthCheckUnhealthyDsp][5]

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

## <a name="deploy-the-container-application"></a>Развертывание приложения-контейнера
Сохраните все изменения и скомпилируйте приложение. Чтобы опубликовать приложение, щелкните правой кнопкой мыши **MyFirstContainer** в обозревателе решений и выберите действие **Опубликовать**.

В поле **Конечная точка подключения** введите конечную точку управления для кластера. Например, `containercluster.westus2.cloudapp.azure.com:19000`. Конечную точку подключения к клиенту можно найти на вкладке "Обзор" кластера на [портале Azure](https://portal.azure.com).

Нажмите кнопку **Опубликовать**.

[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) — это веб-средство для проверки приложений и узлов в кластере Service Fabric и управления ими. Откройте браузер, перейдите по адресу `http://containercluster.westus2.cloudapp.azure.com:19080/Explorer/` и разверните приложение. Приложение развертывается, но находится в состоянии ошибки, пока на узлы кластера не будет загружен образ (что может занять некоторое время в зависимости от размера образа): ![Ошибка][1]

Приложение будет готово, когда оно перейдет в состояние ```Ready```: ![Готово][2]

Откройте веб-браузер и перейдите по адресу `http://containercluster.westus2.cloudapp.azure.com:8081`. Вы должны увидеть заголовок Hello World! в браузере.

## <a name="clean-up"></a>Очистка

Пока кластер работает, с вас будет взиматься плата. Рекомендуем [удалить кластер](./service-fabric-tutorial-delete-cluster.md).

После передачи образа в реестр контейнеров можно удалить локальный образ с компьютера для разработки:

```
docker rmi helloworldapp
docker rmi myregistry.azurecr.io/samples/helloworldapp
```

## <a name="windows-server-container-os-and-host-os-compatibility"></a>Совместимость ОС контейнера Windows Server и ОС узла

Контейнеры Windows Server совместимы не со всеми версиями ОС узла. Пример:
 
- Контейнеры Windows Server, созданные с использованием Windows Server 1709, не работают на узле с ОС Windows Server версии 2016. 
- Контейнеры Windows Server, созданные с помощью Windows Server 2016, работают в режиме изоляции Hyper-V только на узле под управлением Windows Server версии 1709. 
- Для контейнеров Windows Server, созданных с использованием Windows Server 2016, может потребоваться убедиться, что номера редакции версии ОС контейнера и узла ОС совпадают при работе в режиме изоляции процессов на узле под управлением Windows Server 2016.
 
Дополнительные сведения см. в разделе [Совместимость версий контейнеров Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

Учитывайте совместимость ОС узла и контейнера при создании и развертывании контейнеров в кластере Service Fabric. Пример:

- Развертывайте контейнеры с ОС, совместимой с ОС на узлах кластера.
- Убедитесь, что режим изоляции, указанный для приложения-контейнера, поддерживается для ОС контейнера на узле, где оно развертывается.
- Рассмотрите, как обновления операционной системы на узлах кластера или контейнерах могут повлиять на их совместимость. 

Ниже приведены рекомендации, которые помогут гарантировать правильное развертывание контейнеров в кластере Service Fabric.

- Добавляйте к образам Docker явные теги, содержащие версию ОС Windows Server, на основе которой создан контейнер. 
- Укажите [теги ОС](#specify-os-build-specific-container-images) в файле манифеста приложения, чтобы обеспечить совместимость приложения между различными версиями Windows Server и обновлениями.

> [!NOTE]
> В Service Fabric 6.2 и более поздних версий можно развернуть контейнеры, основанные на Windows Server 2016, локально на узле Windows 10. В Windows 10 контейнеры работают в режиме изоляции Hyper-V, независимо от режима изоляции, заданного в манифесте приложения. Дополнительные сведения см. в разделе о [настройке режима изоляции](#configure-isolation-mode).   
>
 
## <a name="specify-os-build-specific-container-images"></a>Указание образов контейнеров для конкретной сборки ОС 

Контейнеры Windows Server могут быть несовместимы в разных версиях ОС. Например, контейнеры Windows Server, созданные с помощью Windows Server 2016, не работают в Windows Server версии 1709 в режиме изоляции. Таким образом, если узлы кластера обновлены до последней версии, в службах контейнеров, созданных с использованием предыдущих версий операционной системы, может произойти сбой. Для решения этой проблемы в версии среды выполнения 6.1 и более поздних в Service Fabric поддерживается указание нескольких образов ОС для одного контейнера и добавление к ним тегов версии сборки ОС в манифесте приложения. Вы можете узнать версию сборки операционной системы, выполнив в командной строке Windows команду `winver`. Перед обновлением операционной системы на узлах обновите манифесты приложений и укажите переопределения образов для каждой версии ОС. В следующем фрагменте кода показано, как указать несколько образов контейнеров в манифесте приложения **ApplicationManifest.xml**:


```xml
      <ContainerHostPolicies> 
         <ImageOverrides> 
           <Image Name="myregistry.azurecr.io/samples/helloworldappDefault" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1701" Os="14393" /> 
               <Image Name="myregistry.azurecr.io/samples/helloworldapp1709" Os="16299" /> 
         </ImageOverrides> 
      </ContainerHostPolicies> 
```
Версия сборки для Windows Server 2016 — 14393, а для Windows Server версии 1709 — 16299. В манифесте службы по-прежнему указывается только один образ на службу контейнеров:

```xml
<ContainerHost>
    <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName> 
</ContainerHost>
```

   > [!NOTE]
   > Функции добавления тегов версии сборки ОС доступны только в Service Fabric для Windows.
   >

Если на виртуальной машине используется базовая ОС сборки 16299 (версия 1709), Service Fabric выбирает образ контейнера, который соответствует этой версии Windows Server. Если в манифесте приложения наряду с образами контейнеров с тегами также указывается образ контейнера без тегов, Service Fabric рассматривает образ без тегов как работающий в разных версиях. Явным образом отметьте тегами образы контейнеров, чтобы избежать проблем во время обновления.

Непомеченный образ контейнера будет работать как переопределение для указанного в ServiceManifest. Поэтому образ "myregistry.azurecr.io/samples/helloworldappDefault" переопределит ImageName "myregistry.azurecr.io/samples/helloworldapp" в ServiceManifest.

## <a name="complete-example-service-fabric-application-and-service-manifests"></a>Полные примеры манифестов службы и приложения Service Fabric
Ниже приведены полные коды манифестов службы и приложения, используемых в этой статье.

### <a name="servicemanifestxml"></a>ServiceManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Guest1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="Guest1Type" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
      <ContainerHost>
        <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
        <!-- Pass comma delimited commands to your container: dotnet, myproc.dll, 5" -->
        <!--Commands> dotnet, myproc.dll, 5 </Commands-->
        <Commands></Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container: -->    
    <EnvironmentVariables>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
    </EnvironmentVariables>

  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to
           listen. Please note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="Guest1TypeEndpoint" UriScheme="http" Port="8081" Protocol="http"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```
### <a name="applicationmanifestxml"></a>ApplicationManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="MyFirstContainerType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Guest1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Guest1Pkg" ServiceManifestVersion="1.0.0" />
    <EnvironmentOverrides CodePackageRef="FrontendService.Code">
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
    </EnvironmentOverrides>
    <ConfigOverrides />
    <Policies>
      <ContainerHostPolicies CodePackageRef="Code">
        <RepositoryCredentials AccountName="myregistry" Password="MIIB6QYJKoZIhvcNAQcDoIIB2jCCAdYCAQAxggFRMIIBTQIBADA1MCExHzAdBgNVBAMMFnJ5YW53aWRhdGFlbmNpcGhlcm1lbnQCEFfyjOX/17S6RIoSjA6UZ1QwDQYJKoZIhvcNAQEHMAAEg
gEAS7oqxvoz8i6+8zULhDzFpBpOTLU+c2mhBdqXpkLwVfcmWUNA82rEWG57Vl1jZXe7J9BkW9ly4xhU8BbARkZHLEuKqg0saTrTHsMBQ6KMQDotSdU8m8Y2BR5Y100wRjvVx3y5+iNYuy/JmM
gSrNyyMQ/45HfMuVb5B4rwnuP8PAkXNT9VLbPeqAfxsMkYg+vGCDEtd8m+bX/7Xgp/kfwxymOuUCrq/YmSwe9QTG3pBri7Hq1K3zEpX4FH/7W2Zb4o3fBAQ+FuxH4nFjFNoYG29inL0bKEcTX
yNZNKrvhdM3n1Uk/8W2Hr62FQ33HgeFR1yxQjLsUu800PrYcR5tLfyTB8BgkqhkiG9w0BBwEwHQYJYIZIAWUDBAEqBBBybgM5NUV8BeetUbMR8mJhgFBrVSUsnp9B8RyebmtgU36dZiSObDsI
NtTvlzhk11LIlae/5kjPv95r3lw6DHmV4kXLwiCNlcWPYIWBGIuspwyG+28EWSrHmN7Dt2WqEWqeNQ==" PasswordEncrypted="true"/>
        <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
      </ContainerHostPolicies>
      <ServicePackageResourceGovernancePolicy CpuCores="1"/>
      <ResourceGovernancePolicy CodePackageRef="Code" MemoryInMB="1024"  />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this
         application type is created. You can also create one or more instances of service type using the
         ServiceFabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Guest1">
      <StatelessService ServiceTypeName="Guest1Type" InstanceCount="[Guest1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="configure-time-interval-before-container-is-force-terminated"></a>Настройка интервала времени перед принудительным завершением контейнера

Вы можете настроить интервал времени для среды выполнения перед удалением контейнера после начала удаления службы (или перехода на другой узел). При настройке интервала времени в контейнер отправляется команда `docker stop <time in seconds>`.  Дополнительные сведения см. в статье [docker stop](https://docs.docker.com/engine/reference/commandline/stop/) (Остановка docker). Интервал времени ожидания указывается в разделе `Hosting`. Этот `Hosting` раздел можно добавить при создании кластера или позднее при обновлении конфигурации. В следующем фрагменте манифеста кластера показано, как задать интервал ожидания:

```json
"fabricSettings": [
    ...,
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
]
```
Интервал времени по умолчанию установлен на 10 секунд. Так как эта конфигурация является динамической, файл конфигурации обновляет только кластер для которого обновляется время ожидания. 


## <a name="configure-the-runtime-to-remove-unused-container-images"></a>Настройка среды выполнения для удаления неиспользуемых образов контейнера

Можно настроить кластер Service Fabric для удаления неиспользуемых образов контейнера из узла. Эта конфигурация позволяет месту на диске перезаписываться, если на узле находится слишком много образов контейнера. Чтобы включить эту функцию, обновите раздел [Hosting](service-fabric-cluster-fabric-settings.md#hosting) в манифесте кластера, как показано в следующем фрагменте кода: 


```json
"fabricSettings": [
    ...,
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
]
```

Образы, которые не должны удаляться, можно указать в параметре `ContainerImagesToSkip`.  


## <a name="configure-container-image-download-time"></a>Настройка времени загрузки образа контейнера

Среда выполнения Service Fabric выделяет 20 минут для скачивания и извлечения образов контейнеров. Этого достаточно для большинства образов контейнеров. В случае с большими образами или при медленном сетевом подключении, возможно, потребуется увеличить время ожидания перед прерыванием загрузки и извлечения образа. Это значение можно задать с помощью атрибута **ContainerImageDownloadTimeout** в разделе **Hosting** манифеста кластера, как показано в следующем фрагменте кода:

```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
              "name": "ContainerImageDownloadTimeout",
              "value": "1200"
          }
        ]
    }
]
```


## <a name="set-container-retention-policy"></a>Настройка политики хранения контейнера

Для упрощения диагностики сбоев при запуске контейнеров Service Fabric (версии 6.1 и более поздних версий) поддерживает хранение контейнеров, работа которых завершилась или при запуске которых произошел сбой. Эту политику можно настроить в файле **ApplicationManifest.xml**, как показано в следующем фрагменте кода:

```xml
 <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" ContainersRetentionCount="2"  RunInteractive="true"> 
```

В атрибуте **ContainersRetentionCount** указывается число контейнеров, которые следует сохранить после сбоя. Если указано отрицательное значение, будут сохраняться все контейнеры после сбоя. Если значение **ContainersRetentionCount** не указано, контейнеры не будут сохраняться. Атрибут **ContainersRetentionCount** также поддерживает параметры приложения, поэтому пользователи могут указывать разные значения для тестовых и рабочих кластеров. Чтобы предотвратить перемещение службы контейнеров на другие узлы, при использовании этих функций используйте ограничения на размещение, указав для службы контейнеров конкретный узел. Удаление всех контейнеров, сохраненных с помощью этой функции, выполняется вручную.

## <a name="start-the-docker-daemon-with-custom-arguments"></a>Запуск управляющей программы Docker с пользовательскими аргументами

Используя среду выполнения Service Fabric версии 6.2 и выше, можно запустить управляющую программу Docker с пользовательскими аргументами. Если пользовательские аргументы указаны, Service Fabric не передает никакие другие аргументы модулю Docker, кроме аргумента `--pidfile`. Таким образом, не нужно передавать `--pidfile` в качестве аргумента. Кроме того, аргумент должен поддерживать прослушивание модулем Docker стандартного конвейера имен Windows (или сокета домена Unix в Linux), обеспечивая обмен данными между Service Fabric и управляющей программой. Пользовательские аргументы передаются в манифесте кластера в блоке **Размещение** раздела **ContainerServiceArguments**, как показано в следующем фрагменте: 
 

```json
"fabricSettings": [
    ...,
    { 
        "name": "Hosting", 
        "parameters": [ 
          { 
            "name": "ContainerServiceArguments", 
            "value": "-H localhost:1234 -H unix:///var/run/docker.sock" 
          } 
        ] 
    } 
]
```

## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о запуске [контейнеров в Service Fabric](service-fabric-containers-overview.md).
* Ознакомьтесь с руководством [Развертывание приложения-контейнера .NET в Azure Service Fabric](service-fabric-host-app-in-a-container.md).
* Дополнительные сведения о [жизненном цикле приложения](service-fabric-application-lifecycle.md) Service Fabric.
* Извлечение [примеров кода контейнера Service Fabric](https://github.com/Azure-Samples/service-fabric-containers) на GitHub.

[1]: ./media/service-fabric-get-started-containers/MyFirstContainerError.png
[2]: ./media/service-fabric-get-started-containers/MyFirstContainerReady.png
[3]: ./media/service-fabric-get-started-containers/HealthCheckHealthy.png
[4]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_App.png
[5]: ./media/service-fabric-get-started-containers/HealthCheckUnhealthy_Dsp.png

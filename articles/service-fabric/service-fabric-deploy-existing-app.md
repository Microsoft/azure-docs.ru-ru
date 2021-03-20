---
title: Развертывание существующего исполняемого файла в Azure Service Fabric
description: Узнайте, как упаковать существующее приложение в качестве гостевого исполняемого файла для его развертывания в кластере Service Fabric.
ms.topic: conceptual
ms.date: 03/30/2020
ms.openlocfilehash: 72fde75e16341164106bb952d0bb66b83be744e1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86259257"
---
# <a name="package-and-deploy-an-existing-executable-to-service-fabric"></a>Упаковка и развертывание существующего исполняемого файла вручную

При упаковке существующего исполняемого файла в качестве [гостевого](service-fabric-guest-executables-introduction.md) вы можете использовать шаблон проекта Visual Studio или [создать пакет приложения вручную](#manually). При использовании Visual Studio шаблон проекта создает для вас структуру пакета приложения и файлы манифеста.

> [!TIP]
> Упаковать имеющийся исполняемый файл Windows в службу проще всего с помощью Visual Studio. В Linux можно использовать Yeoman.
>

## <a name="use-visual-studio-to-package-and-deploy-an-existing-executable"></a>Упаковка и развертывание существующего исполняемого приложения с использованием Visual Studio

Visual Studio предоставляет шаблон службы Service Fabric для развертывания гостевого исполняемого файла в кластере Service Fabric.

1. Выберите **файл**  >  **создать проект** и создайте приложение Service Fabric.
2. Выберите **гостевой исполняемый файл** в качестве шаблона службы.
3. Нажмите кнопку **Обзор** , чтобы выбрать папку с исполняемым файлом, и заполните остальные параметры, чтобы создать службу.
   * *Поведение пакета кода*. Здесь можно выбрать копирование всего содержимого папки в проект Visual Studio. Это удобно, если исполняемый файл не будет изменяться. Если исполняемый файл будет изменяться и требуется возможность выбора новых сборок в динамическом режиме, можно задать привязку к папке. При создании проекта приложения в Visual Studio можно использовать связанные папки. С их помощью можно ссылаться на исходное расположение в проекте, что позволяет обновлять гостевой исполняемый файл в источнике. Эти обновления при создании становятся частью пакета приложения.
   * *Program* обозначает исполняемый файл, который следует выполнить для запуска службы.
   * *Arguments* позволяет указать аргументы, которые нужно передать в исполняемый файл. Здесь можно указать список параметров с аргументами.
   * *WorkingFolder* позволяет определить рабочий каталог для процесса, который будет запущен. Можно задать три значения:
     * `CodeBase`. Рабочим каталогом будет каталог Code в пакете приложения (каталог `Code` в показанной выше структуре файлов).
     * `CodePackage`. Рабочим каталогом будет корневой каталог в пакете приложения (каталог `GuestService1Pkg` в показанной выше структуре файлов).
     * `Work`. Файлы помещаются в подкаталог, который называется рабочим.
4. Присвойте службе имя и нажмите кнопку **ОК**.
5. Если службе нужна конечная точка для обмена данными, можно добавить значения протокола, порта и типа в файл ServiceManifest.xml. Например, `<Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" UriScheme="http" PathSuffix="myapp/" Type="Input" />`.
6. Теперь можно выполнить упаковку и публикацию на локальном кластере, выполнив отладку решения в Visual Studio. Когда все будет готово, можно опубликовать приложение на удаленном кластере или вернуть решение в систему управления версиями.
7. Прочтите сведения о [проверке работающего приложения](#check-your-running-application), чтобы узнать, как просмотреть данные о работе гостевого исполняемого файла в Service Fabric Explorer.

Инструкции с примерами см. в статье о [создании первого гостевого исполняемого приложения с помощью Visual Studio](quickstart-guest-app.md).

### <a name="packaging-multiple-executables-with-visual-studio"></a>Упаковка нескольких исполняемых файлов с помощью Visual Studio

Visual Studio можно использовать для создания пакета приложения, содержащего несколько гостевых исполняемых файлов. После добавления первого гостевого исполняемого файла щелкните правой кнопкой мыши проект приложения и выберите **Добавить->новая служба Service Fabric** , чтобы добавить в решение второй гостевой исполняемый проект.

> [!NOTE]
> Если связать источник с проектом Visual Studio, при построении решения Visual Studio следует убедиться, что в пакете приложения отображаются последние изменения в источнике.

## <a name="use-yeoman-to-package-and-deploy-an-existing-executable-on-linux"></a>Упаковка и развертывание существующего исполняемого файла в Linux с помощью Yeoman

Процедура создания и развертывания гостевого исполняемого файла в Linux совпадает с процедурой развертывания приложения csharp или java.

1. В окне терминала введите `yo azuresfguest`.
2. Присвойте имя приложению.
3. Введите имя службы и укажите сведения, включая путь к исполняемому файлу и параметры, с которыми он должен вызываться.

Yeoman создает пакет приложения с соответствующими файлами приложения и манифеста, а также со сценариями установки и удаления.

### <a name="packaging-multiple-executables-using-yeoman-on-linux"></a>Упаковка нескольких исполняемых файлов с помощью Yeoman в Linux

Чтобы добавить службу в приложение, созданное с использованием `yo`, сделайте следующее:

1. Перейдите в корневой каталог существующего приложения.  Например, `cd ~/YeomanSamples/MyApplication`, если `MyApplication` является приложением, созданным с помощью Yeoman.
2. Выполните команду `yo azuresfguest:AddService`, чтобы предоставить необходимые сведения.

<a id="manually"></a>

## <a name="manually-package-and-deploy-an-existing-executable"></a>Упаковка и развертывание имеющегося исполняемого файла вручную

Процесс упаковки гостевого исполняемого файла вручную включает следующие этапы:

1. Создание структуры каталогов пакета.
2. Добавление файлов конфигурации и кода приложения.
3. Редактирование файла манифеста службы.
4. Редактирование файла манифеста приложения.

### <a name="create-the-package-directory-structure"></a>Создание структуры каталогов пакета

Вы можете начать с создания структуры каталогов, как описано в разделе [Упаковка приложения Service Fabric Azure](./service-fabric-package-apps.md).

### <a name="add-the-applications-code-and-configuration-files"></a>Добавление файлов конфигурации и кода приложения

Создав структуру каталогов, добавьте файлы кода и конфигурации приложения в каталоги Code и Config. При необходимости в этих каталогах можно создать дополнительные папки (подкаталоги).

Service Fabric создает расширенную копию содержимого корневого каталога приложения (выполняется операция `xcopy`), поэтому вам нужно создать только два каталога верхнего уровня: для кода и для параметров. (Вы можете выбрать любые имена. Дополнительные сведения см. в следующем разделе.)

> [!NOTE]
> Обязательно добавьте все файлы и зависимости, необходимые приложению. Service Fabric скопирует содержимое пакета приложения на все узлы кластера, где будут развертываться службы приложения. Пакет должен содержать весь код, необходимый для работы приложения. Не рассчитывайте на то, что зависимости уже установлены.
>
>

### <a name="edit-the-service-manifest-file"></a>Редактирование файла манифеста службы

Далее нужно добавить в манифест службы следующие сведения:

* Имя типа службы. Это идентификатор, по которому Service Fabric определяет службу.
* Команда для запуска приложения (ExeHost).
* Любой сценарий, который нужно выполнить для установки приложения (SetupEntrypoint).

Ниже приведен пример файла `ServiceManifest.xml` .

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Name="NodeApp" Version="1.0.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceTypes>
      <StatelessServiceType ServiceTypeName="NodeApp" UseImplicitHost="true"/>
   </ServiceTypes>
   <CodePackage Name="code" Version="1.0.0.0">
      <SetupEntryPoint>
         <ExeHost>
             <Program>scripts\launchConfig.cmd</Program>
         </ExeHost>
      </SetupEntryPoint>
      <EntryPoint>
         <ExeHost>
            <Program>node.exe</Program>
            <Arguments>bin/www</Arguments>
            <WorkingFolder>CodePackage</WorkingFolder>
         </ExeHost>
      </EntryPoint>
   </CodePackage>
   <Resources>
      <Endpoints>
         <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />
      </Endpoints>
   </Resources>
</ServiceManifest>
```

В следующих разделах мы рассмотрим те элементы файла, которые вам нужно изменить.

#### <a name="update-servicetypes"></a>Обновление ServiceTypes

```xml
<ServiceTypes>
  <StatelessServiceType ServiceTypeName="NodeApp" UseImplicitHost="true" />
</ServiceTypes>
```

* Выберите для `ServiceTypeName`любое имя. Значение используется в файле `ApplicationManifest.xml` для идентификации службы.
* Задайте имя `UseImplicitHost="true"`. Этот атрибут сообщает платформе Service Fabric, что служба использует автономное приложение, поэтому платформе нужно только запустить службу как процесс и следить за ее работоспособностью.

#### <a name="update-codepackage"></a>Обновление CodePackage
Элемент CodePackage указывает на расположение и версию кода службы.

```xml
<CodePackage Name="Code" Version="1.0.0.0">
```

Атрибут `Name` определяет имя каталога в пакете приложения, где хранится код службы. `CodePackage` также имеет атрибут `version`. Он определяет версию кода, а также может использоваться для обновления кода службы с помощью инфраструктуры Service Fabric для управления жизненным циклом приложения.

#### <a name="optional-update-setupentrypoint"></a>Обновление SetupEntrypoint (необязательно)

```xml
<SetupEntryPoint>
   <ExeHost>
       <Program>scripts\launchConfig.cmd</Program>
   </ExeHost>
</SetupEntryPoint>
```
Элемент SetupEntrypoint позволяет определить исполняемый или пакетный файл, который следует запустить перед выполнением кода службы. Это необязательный шаг, поэтому этот элемент можно не указывать, если инициализация не требуется. Файл, указанный в SetupEntryPoint, выполняется при каждом перезапуске службы.

Есть только один элемент SetupEntrypoint, следовательно, если для установки приложения требуется несколько скриптов, эти скрипты необходимо сгруппировать в один пакетный файл. SetupEntryPoint может выполнять файлы любого типа, включая исполняемые и пакетные, а также командлеты PowerShell. Подробнее см. в статье [Настройка SetupEntryPoint](service-fabric-application-runas-security.md).

В приведенном выше примере в разделе SetupEntrypoint запускается пакетный файл `LaunchConfig.cmd`, расположенный в подкаталоге `scripts` каталога Code (при условии, что в элементе WorkingFolder указано имя каталога CodeBase).

#### <a name="update-entrypoint"></a>Обновление EntryPoint

```xml
<EntryPoint>
  <ExeHost>
    <Program>node.exe</Program>
    <Arguments>bin/www</Arguments>
    <WorkingFolder>CodeBase</WorkingFolder>
  </ExeHost>
</EntryPoint>
```

Элемент `EntryPoint` в файле манифеста службы указывает, как нужно запускать службу.

В элементе `ExeHost` указывается исполняемый файл (вместе с аргументами), который будет использоваться для запуска службы. При необходимости можно добавить атрибут `IsExternalExecutable="true"` в `ExeHost`, чтобы указать, что программа является внешним исполняемым файлом вне пакета кода. Например, `<ExeHost IsExternalExecutable="true">`.

* `Program` — имя исполняемого файла, который будет запускать службу.
* `Arguments` — аргументы, которые нужно передать в исполняемый файл. Здесь можно указать список параметров с аргументами.
* `WorkingFolder` — рабочий каталог запускаемого процесса. Можно задать три значения:
  * `CodeBase`. Рабочим каталогом будет каталог Code в пакете приложения (каталог `Code` в показанной выше структуре файлов).
  * `CodePackage`. Рабочим каталогом будет корневой каталог в пакете приложения (каталог `GuestService1Pkg` в показанной выше структуре файлов).
    * `Work`. Файлы помещаются в подкаталог, который называется рабочим.

Элемент WorkingFolder позволяет задать правильный рабочий каталог, чтобы приложение или сценарии инициализации могли использовать относительные пути.

#### <a name="update-endpoints-and-register-with-naming-service-for-communication"></a>Обновление конечных точек и регистрация в службе именования для обмена данными

```xml
<Endpoints>
   <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000" Type="Input" />
</Endpoints>

```

В приведенном выше примере в элементе `Endpoint` указываются конечные точки, через которые приложение может ожидать передачи данных. В этом примере приложение Node.js прослушивает HTTP-порт 3000.

Кроме того, можно настроить так, чтобы служба Service Fabric опубликовала конечную точку в службе именования. Таким образом, другие службы смогут обнаружить адрес конечной точки для этой службы.
Опубликованный адрес конечной точки имеет вид `UriScheme://IPAddressOrFQDN:Port/PathSuffix`. `UriScheme` и `PathSuffix` являются необязательными атрибутами. `IPAddressOrFQDN` — IP-адрес или полное доменное имя узла, на котором размещен исполняемый файл. Значение вычисляется автоматически.

В примере ниже после развертывания службы в Service Fabric Explorer появится конечная точка в формате `http://10.1.4.92:3000/myapp/`, опубликованная для экземпляра службы. Если используется локальный компьютер, она будет в формате `http://localhost:3000/myapp/`.

```xml
<Endpoints>
   <Endpoint Name="NodeAppTypeEndpoint" Protocol="http" Port="3000"  UriScheme="http" PathSuffix="myapp/" Type="Input" />
</Endpoints>
```

Эти адреса можно использовать для обмена данными между службами через [обратный прокси-сервер](service-fabric-reverseproxy.md) .

### <a name="edit-the-application-manifest-file"></a>Редактирование файла манифеста приложения

Настроив файл `Servicemanifest.xml`, вам нужно внести в файл `ApplicationManifest.xml` изменения, которые позволят использовать правильные тип и имя службы.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="NodeAppType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <ServiceManifestImport>
      <ServiceManifestRef ServiceManifestName="NodeApp" ServiceManifestVersion="1.0.0.0" />
   </ServiceManifestImport>
</ApplicationManifest>
```

#### <a name="servicemanifestimport"></a>ServiceManifestImport

В элементе `ServiceManifestImport` можно указать одну или несколько служб, которые нужно добавить в приложение. Службы указываются в атрибуте `ServiceManifestName` в виде имени каталога, в котором хранится файл `ServiceManifest.xml`.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="NodeApp" ServiceManifestVersion="1.0.0.0" />
</ServiceManifestImport>
```

## <a name="set-up-logging"></a>Настройка ведения журнала

В журналах консоли можно отслеживать работоспособность гостевых исполняемых файлов. В частности, из этих журналов можно узнать, не возвращают ли сценарии приложения и конфигурации какие-либо ошибки.
Перенаправление консоли можно настроить в файле `ServiceManifest.xml` в элементе `ConsoleRedirection`.

> [!WARNING]
> Никогда не используйте политику перенаправления консоли для приложений в рабочей среде, так как она может повлиять на отработку отказа приложения. Используйте ее *только* для локальной разработки и отладки.  
>
>

```xml
<EntryPoint>
  <ExeHost>
    <Program>node.exe</Program>
    <Arguments>bin/www</Arguments>
    <WorkingFolder>CodeBase</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="5" FileMaxSizeInKb="2048"/>
  </ExeHost>
</EntryPoint>
```

`ConsoleRedirection` может использоваться для перенаправления выходных данных консоли (stdout и stderr) в рабочий каталог. Это позволяет проверить, не было ли ошибок во время установки или выполнения приложения в кластере Service Fabric.

`FileRetentionCount` определяет, сколько файлов будет сохраняться в рабочем каталоге. Например, значение 5 означает, что в рабочем каталоге будут храниться файлы журналов для пяти предыдущих запусков.

`FileMaxSizeInKb` определяет максимальный размер файла журнала.

Файлы журналов сохраняются в один из рабочих каталогов службы. Чтобы определить, где находятся файлы, откройте Service Fabric Explorer и посмотрите, на каком узле запущена служба и какой рабочий каталог используется. Эта процедура рассмотрена далее в этой статье.

## <a name="deployment"></a>Deployment (Развертывание)

Последним шагом является [развертывание приложения](service-fabric-deploy-remove-applications.md). В приведенном ниже скрипте PowerShell показано, как можно развернуть приложение в локальном кластере разработки и запустить новую службу Service Fabric.

```powershell

Connect-ServiceFabricCluster localhost:19000

Write-Host 'Copying application package...'
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath 'C:\Dev\MultipleApplications' -ImageStoreConnectionString 'file:C:\SfDevCluster\Data\ImageStoreShare' -ApplicationPackagePathInImageStore 'nodeapp'

Write-Host 'Registering application type...'
Register-ServiceFabricApplicationType -ApplicationPathInImageStore 'nodeapp'

New-ServiceFabricApplication -ApplicationName 'fabric:/nodeapp' -ApplicationTypeName 'NodeAppType' -ApplicationTypeVersion 1.0

New-ServiceFabricService -ApplicationName 'fabric:/nodeapp' -ServiceName 'fabric:/nodeapp/nodeappservice' -ServiceTypeName 'NodeApp' -Stateless -PartitionSchemeSingleton -InstanceCount 1

```

>[!TIP]
> [Сожмите пакет](service-fabric-package-apps.md#compress-a-package) перед копированием в хранилище образов, если пакет имеет большой размер или содержит много файлов. Дополнительные сведения см. [здесь](service-fabric-deploy-remove-applications.md#upload-the-application-package).
>

Службу Service Fabric можно развертывать в разных конфигурациях. Например, службу можно развернуть с одним или несколькими экземплярами. Также можно развернуть по одному экземпляру службы на каждом узле кластера Service Fabric.

Параметр `InstanceCount` командлета `New-ServiceFabricService` позволяет указать, сколько экземпляров службы следует запускать в кластере Service Fabric. Значение `InstanceCount` следует задавать в зависимости от типа развертываемого приложения. Ниже приводятся два самых распространенных сценария.

* `InstanceCount = "1"`. В этом случае в кластере развертывается только один экземпляр службы. На каком именно узле будет развернута служба, определяет планировщик Service Fabric.
* `InstanceCount ="-1"`. В этом случае на каждом узле кластера Service Fabric будет развернуто по одному экземпляру службы. В результате на каждом узле кластера будет по одному (и только одному) экземпляру службы.

Эта конфигурация подходит для клиентских приложений (например, конечной точки REST), ведь для работы с конечной точкой таким приложениям достаточно подключиться к любому узлу кластера. Эту конфигурацию можно использовать и в том случае, если все узлы в кластере Service Fabric подключены к подсистеме балансировки нагрузки. Тогда клиентский трафик будет распределяться между экземплярами службы, которые запущены на всех узлах кластера.

## <a name="check-your-running-application"></a>Проверка работающего приложения
В обозревателе Service Fabric определите узел, где запущена служба. В этом примере она выполняется на узле Node1.

![Узел, на котором запущена служба](./media/service-fabric-deploy-existing-app/nodeappinsfx.png)

Если вы перейдете к узлу, а затем к приложению, вы увидите основные сведения об узле, включая его расположение на диске.

![Расположение на диске](./media/service-fabric-deploy-existing-app/locationondisk2.png)

Если перейти в этот каталог в обозревателе сервера, вы увидите рабочий каталог и папку журналов службы, как показано на рисунке ниже.

![Расположение журнала](./media/service-fabric-deploy-existing-app/loglocation.png)

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали об основной процедуре упаковки гостевого исполняемого файла и его развертывания в Service Fabric. В приведенных ниже статьях описаны связанные сведения и задачи.

* [Пример для упаковки и развертывания гостевого исполняемого файла](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started), включая ссылку на предварительную версию средства упаковки
* [Пример двух гостевых исполняемых файлов (C# и Node.js), которые взаимодействуют через службу именования с помощью REST](https://github.com/Azure-Samples/service-fabric-containers)
* [Создание первого приложения Service Fabric в Visual Studio](service-fabric-tutorial-create-dotnet-app.md)

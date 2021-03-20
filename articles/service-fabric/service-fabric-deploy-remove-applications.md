---
title: Развертывание Service Fabric Azure с помощью PowerShell
description: Узнайте, как удалять и развертывать приложения в Service Fabric Azure и как выполнять эти действия в PowerShell.
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: d428a51c0bc224ca8706403ae176d46f1db82a32
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98785453"
---
# <a name="deploy-and-remove-applications-using-powershell"></a>Развертывание и удаление приложений с помощью PowerShell

> [!div class="op_single_selector"]
> * [Resource Manager](service-fabric-application-arm-resource.md)
> * [PowerShell](service-fabric-deploy-remove-applications.md)
> * [Интерфейс командной строки Service Fabric](service-fabric-application-lifecycle-sfctl.md)
> * [API FabricClient](service-fabric-deploy-remove-applications-fabricclient.md)

<br/>

После [создания пакета приложения][10] его можно развернуть в кластере Azure Service Fabric. Развертывание включает следующие три шага:

1. Отправка пакета приложения в хранилище образов.
2. Регистрация типа приложения с помощью относительного пути к хранилищу образов.
3. Создайте экземпляр приложения.

Если развернутое приложение больше не нужно, можно удалить экземпляр приложения и соответствующий тип приложения. Полное удаление приложения из кластера включает следующие действия:

1. Удаление запущенного экземпляра приложения.
2. Отмена регистрации типа приложения, если он больше не нужен.
3. Удаление пакета приложения из хранилища образов.

Если вы используете Visual Studio для развертывания и отладки приложений в локальном кластере разработки, все описанные выше действия выполняются автоматически с помощью сценария PowerShell.  Этот скрипт находится в папке *Scripts* проекта приложения. Эта статья содержит основные сведения о действиях, выполняемых этим сценарием. Они помогут вам выполнять те же операции вне Visual Studio. 

Также можно развернуть приложение с помощью внешней подготовки. Пакет приложения можно [упаковать`sfpkg` в файл ](service-fabric-package-apps.md#create-an-sfpkg) и передать во внешнее хранилище. В этом случае его не нужно отправлять в хранилище образов. Для развертывания сделайте следующее:

1. Отправьте файл `sfpkg` во внешнее хранилище. Внешним хранилищем может быть любое хранилище, предоставляющее конечную точку REST HTTP или HTTPS.
2. Зарегистрируйте тип приложения, указав внешний URI скачивания и тип приложения.
2. Создайте экземпляр приложения.

Для очистки удалите экземпляры приложения и отмените регистрацию типа приложения. Так как пакет не был скопирован в хранилище образов, очистка во временном расположении не требуется. Подготовка во внешнем хранилище доступна начиная с версии Service Fabric 6.1.

>[!NOTE]
> В Visual Studio внешняя подготовка сейчас не поддерживается.

 

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Перед выполнением команд PowerShell, описанных в этой статье, нужно подключиться к кластеру Service Fabric с помощью команды [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster). Чтобы подключиться к локальному кластеру разработки, выполните следующую команду:

```powershell
Connect-ServiceFabricCluster
```

Примеры подключения к удаленному кластеру или кластеру, защищенному с помощью Azure Active Directory, сертификатов X509 или Windows Active Directory, см. в статье [Безопасное подключение к кластеру](service-fabric-connect-to-secure-cluster.md).

## <a name="upload-the-application-package"></a>Загрузка пакета приложения

Отправка пакета приложения означает, что он помещается в расположение, доступное внутренним компонентам Service Fabric.
Если вы хотите проверить пакет приложения локально, используйте командлет [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage).

Команда [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) передает пакет приложения в хранилище образов кластера.

Предположим, вы собрали и упаковали в Visual Studio 2015 приложение с именем *MyApplication*. По умолчанию имя типа приложения отображается в файле ApplicationManifest.xml как MyApplicationType.  Пакет приложения, содержащий необходимый манифест приложения, манифесты служб, пакеты кода, конфигурации и данных, находится в папке *C:\Users \<username\> \Documents\Visual Studio 2015 \ Projects\MyApplication\MyApplication\pkg\Debug*. 

Следующая команда выводит содержимое пакета приложения:

```powershell
$path = 'C:\Users\<user\>\Documents\Visual Studio 2015\Projects\MyApplication\MyApplication\pkg\Debug'
tree /f $path
```

```Output
Folder PATH listing for volume OSDisk
Volume serial number is 0459-2393
C:\USERS\USER\DOCUMENTS\VISUAL STUDIO 2015\PROJECTS\MYAPPLICATION\MYAPPLICATION\PKG\DEBUG
│   ApplicationManifest.xml
│
└───Stateless1Pkg
    │   ServiceManifest.xml
    │
    ├───Code
    │       Microsoft.ServiceFabric.Data.dll
    │       Microsoft.ServiceFabric.Data.Interfaces.dll
    │       Microsoft.ServiceFabric.Internal.dll
    │       Microsoft.ServiceFabric.Internal.Strings.dll
    │       Microsoft.ServiceFabric.Services.dll
    │       ServiceFabricServiceModel.dll
    │       Stateless1.exe
    │       Stateless1.exe.config
    │       Stateless1.pdb
    │       System.Fabric.dll
    │       System.Fabric.Strings.dll
    │
    └───Config
            Settings.xml
```

Если пакет приложения большой или имеет большое количество файлов, его можно [сжать](service-fabric-package-apps.md#compress-a-package). Сжатие уменьшает размер и количество файлов.
Это приводит к более быстрой регистрации и отмене регистрации типа приложения. Время загрузки может увеличиться, особенно если требуется сжать пакет. 

Чтобы сжать пакет, используйте команду [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage). Сжатие можно выполнить отдельно от загрузки с помощью флага `SkipCopy` или во время операции загрузки. Применить сжатие к сжатому пакету невозможно.
Чтобы распаковать пакет, используйте команду [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) с параметром `UncompressPackage`.

Следующий командлет сжимает пакет, не копируя его в хранилище образов. Теперь пакет содержит ZIP-файлы для пакетов `Code` и `Config`. Приложение и манифесты служб не сжимаются, так как они используются для множества внутренних операций (совместный доступ к пакетам, извлечение имени типа или версии приложения для некоторых проверок и т. д.). Сжатие манифеста снизит эффективность таких операций.

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -CompressPackage -SkipCopy
tree /f $path
```

```Output
Folder PATH listing for volume OSDisk
Volume serial number is 0459-2393
C:\USERS\USER\DOCUMENTS\VISUAL STUDIO 2015\PROJECTS\MYAPPLICATION\MYAPPLICATION\PKG\DEBUG
|   ApplicationManifest.xml
|
└───Stateless1Pkg
       Code.zip
       Config.zip
       ServiceManifest.xml
```

Для больших пакетов приложений сжатие занимает некоторое время. Для получения наилучших результатов используйте быстрый SSD-диск. Время сжатия и размер сжатого пакета также зависят от содержимого пакета.
Ниже приведена статистика сжатия для некоторых пакетов, отражающая первоначальный и сжатый размер, а также время сжатия.

|Исходный размер (МБ)|Число файлов|Время сжатия|Размер сжатого пакета (МБ)|
|----------------:|---------:|---------------:|---------------------------:|
|100|100|00:00:03.3547592|60|
|512|100|00:00:16.3850303|307|
|1024|500|00:00:32.5907950|615|
|2048|1000|00:01:04.3775554|1231|
|5012|100|00:02:45.2951288|3074|

При необходимости после сжатия пакет можно загрузить в один или несколько кластеров Service Fabric. Механизм развертывания одинаков для сжатых и несжатых пакетов. Сжатые пакеты хранятся в неизменном виде в хранилище образов кластера. Пакеты распаковываются на узле перед запуском приложения.


Следующий пример передает пакет в папку MyApplicationV1 хранилища образов:

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -ApplicationPackagePathInImageStore MyApplicationV1 -TimeoutSec 1800
```

Если вы не укажете параметр *-ApplicationPackagePathInImageStore*, пакет приложения будет скопирован в папку Debug хранилища образов.

>[!NOTE]
>Командлет **Copy-ServiceFabricApplicationPackage** автоматически обнаружит строку подключения к соответствующему хранилищу образов, если сеанс PowerShell подключен к кластеру Service Fabric. Для версий Service Fabric старше 5.6 необходимо явно указать аргумент **-ImageStoreConnectionString**.
>
>```powershell
>PS C:\> Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -ApplicationPackagePathInImageStore MyApplicationV1 -ImageStoreConnectionString (Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)) -TimeoutSec 1800
>```
>
>Командлет **Get-ImageStoreConnectionStringFromClusterManifest** , являющийся частью модуля PowerShell пакета SDK для Service Fabric, позволяет получить строку подключения хранилища образов.  Чтобы импортировать модуль пакета SDK, выполните следующую команду.
>
>```powershell
>Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
>```
>
>В статье [Общие сведения о параметре ImageStoreConnectionString](service-fabric-image-store-connection-string.md) вы найдете дополнительные сведения о хранилище образов и строке подключения к этому хранилищу.
>
>
>

Время, необходимое для загрузки пакета, зависит от нескольких факторов. Некоторые из них: число файлов в пакете, размер пакета и размеры файлов. Скорость сетевого подключения между исходным компьютером и кластером Service Fabric также влияет на время загрузки. Время ожидания по умолчанию для [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) — 30 минут.
В зависимости от указанных факторов время ожидания может увеличиться. При сжатии пакета в вызове копирования необходимо также учитывать время сжатия.



## <a name="register-the-application-package"></a>Регистрация пакета приложения

При регистрации пакета приложения его тип и версия, объявленные в манифесте приложения, становятся доступными для использования. Система считывает содержимое пакета, переданного на предыдущем этапе, проверяет пакет, обрабатывает его содержимое и копирует обработанный пакет во внутреннее системное расположение.  

Выполните команду [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype), чтобы зарегистрировать тип приложения в кластере и сделать его доступным для развертывания:

### <a name="register-the-application-package-copied-to-image-store"></a>Регистрация пакета приложения, скопированного в хранилище образов

Если пакет ранее был скопирован в хранилище образов, операция регистрации указывает относительный путь к хранилищу образов.

```powershell
Register-ServiceFabricApplicationType -ApplicationPathInImageStore MyApplicationV1
```

```Output
Register application type succeeded
```

MyApplicationV1 — это папка в хранилище образов, в которой находится пакет приложения. Тип приложения с именем MyApplicationType и версией 1.0.0 (оба параметра находятся в манифесте приложения) зарегистрирован в кластере.

### <a name="register-the-application-package-copied-to-an-external-store"></a>Регистрация пакета приложения, скопированного во внешнее хранилище

Начиная с версии Service Fabric 6.1 при подготовке поддерживается скачивание пакета из внешнего хранилища. URI скачивания представляет путь к [ `sfpkg` пакету приложения](service-fabric-package-apps.md#create-an-sfpkg) , из которого можно скачать пакет приложения с помощью протоколов HTTP или HTTPS. Пакет необходимо заранее отправить в это внешнее расположение. Для скачивания файла с помощью Service Fabric необходимо, чтобы URI предоставлял доступ на чтение. Файл `sfpkg` должен иметь расширение SFPKG. Операция подготовки должна включать сведения о типе приложения, который указан в манифесте приложения.

```powershell
Register-ServiceFabricApplicationType -ApplicationPackageDownloadUri "https://sftestresources.blob.core.windows.net:443/sfpkgholder/MyAppPackage.sfpkg" -ApplicationTypeName MyApp -ApplicationTypeVersion V1 -Async
```

Команда [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype) возвращает данные только после того, как система успешно зарегистрирует пакет приложения. Продолжительность регистрации зависит от размера и содержимого пакета приложения. При необходимости можно увеличить время ожидания, указав параметр **-TimeoutSec** (по умолчанию время ожидания составляет 60 секунд).

Если размер пакета настолько большой, что приводит к превышению времени ожидания, используйте параметр **-Async**. Эта команда возвращается, когда кластер принимает команду регистрации. Операция регистрации продолжается при необходимости.
Чтобы вывести список версий типов приложения и их состояние регистрации, используйте команду [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype). Эта команда используется, чтобы определить, что регистрация выполнена.

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

## <a name="remove-an-application-package-from-the-image-store"></a>Удаление пакета приложения из хранилища образов

Если пакет был скопирован в хранилище образов, его следует удалить из временного расположения после успешной регистрации приложения. Удаление пакетов приложений из хранилища образов освобождает системные ресурсы. Если хранить неиспользуемые пакеты, они занимают место на диске и создают проблемы производительности приложения.

```powershell
Remove-ServiceFabricApplicationPackage -ApplicationPackagePathInImageStore MyApplicationV1
```

## <a name="create-the-application"></a>Создание приложения

Вы можете создать экземпляр приложения из любой версии типа приложения, которая успешно зарегистрирована с помощью командлета [New-ServiceFabricApplication](/powershell/module/servicefabric/new-servicefabricapplication) . Имя каждого приложения должно начинаться с схемы *"Fabric:"* и должно быть уникальным для каждого экземпляра приложения. Если в манифесте приложения для конкретного его типа были определены службы по умолчанию, то они также будут созданы.

```powershell
New-ServiceFabricApplication fabric:/MyApp MyApplicationType 1.0.0
```

```Output
ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
ApplicationParameters  : {}
```

Для каждой версии зарегистрированного типа приложения можно создать несколько экземпляров приложения. Каждый экземпляр выполняется изолированно, используя собственный рабочий каталог и процесс.

Чтобы увидеть, какие приложения и службы работают в кластере, выполните командлеты [Get-ServiceFabricApplication](/powershell/module/servicefabric/get-servicefabricapplication) и [Get-ServiceFabricService](/powershell/module/servicefabric/get-servicefabricservice):

```powershell
Get-ServiceFabricApplication  
```

```Output
ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Ok
ApplicationParameters  : {}
```

```powershell
Get-ServiceFabricApplication | Get-ServiceFabricService
```

```Output
ServiceName            : fabric:/MyApp/Stateless1
ServiceKind            : Stateless
ServiceTypeName        : Stateless1Type
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
ServiceStatus          : Active
HealthState            : Ok
```

## <a name="remove-an-application"></a>Удаление приложения

Экземпляр приложения, который больше не нужен, можно удалить без возможности восстановления с помощью командлета [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication). Командлет [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication) также удаляет все службы, которые относятся к этому приложению, и все их состояния. 

> [!WARNING]
> Эта операция необратима, и вы не сможете восстановить состояние приложения.

```powershell
Remove-ServiceFabricApplication fabric:/MyApp
```

```Output
Confirm
Continue with this operation?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
Remove application instance succeeded
```

```powershell
Get-ServiceFabricApplication
```

## <a name="unregister-an-application-type"></a>Отмена регистрации типа приложения

Если определенная версия типа приложения больше не требуется, отмените регистрацию типа приложения, используя командлет [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype). Отмена регистрации неиспользуемых типов приложений позволяет освободить пространство в хранилище, которое занимает хранилище образов, за счет удаления файлов типов приложений. При отмене регистрации типа приложения пакет приложения, скопированный во временное расположение хранилища образов, не удаляется, если использовалась команда копирования в хранилище образов. Регистрацию типа приложения можно отменить в том случае, если на его основе не были созданы экземпляры приложений и нет ссылающихся на него незавершенных обновлений приложений.

Чтобы просмотреть все типы приложений, зарегистрированных в кластере, запустите командлет [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype):

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

Запустите командлет [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype), чтобы отменить регистрацию приложения определенного типа:

```powershell
Unregister-ServiceFabricApplicationType MyApplicationType 1.0.0
```

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="copy-servicefabricapplicationpackage-asks-for-an-imagestoreconnectionstring"></a>Команда Copy-ServiceFabricApplicationPackage запрашивает строку ImageStoreConnectionString

В пакете разработки Service Fabric SDK уже предусмотрены все необходимые значения по умолчанию. Тем не менее, при необходимости значение ImageStoreConnectionString для всех команд должно совпадать со значением, используемым кластером Service Fabric. Значение ImageStoreConnectionString можно найти в манифесте кластера, используя команды [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest) и Get-ImageStoreConnectionStringFromClusterManifest.

```powershell
Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)
```

Командлет **Get-ImageStoreConnectionStringFromClusterManifest** , являющийся частью модуля PowerShell пакета SDK для Service Fabric, позволяет получить строку подключения хранилища образов.  Чтобы импортировать модуль пакета SDK, выполните следующую команду.

```powershell
Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
```

Ниже представлено значение ImageStoreConnectionString из манифеста кластера.

```xml
<ClusterManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Name="Server-Default-SingleNode" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

    [...]

    <Section Name="Management">
      <Parameter Name="ImageStoreConnectionString" Value="file:D:\ServiceFabric\Data\ImageStore" />
    </Section>

    [...]
```

В статье [Общие сведения о параметре ImageStoreConnectionString](service-fabric-image-store-connection-string.md) вы найдете дополнительные сведения о хранилище образов и строке подключения к этому хранилищу.

### <a name="deploy-large-application-package"></a>Развертывание пакета приложения большего размера

Проблема. Время ожидания выполнения команды [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) истекает для пакета большого приложения (несколько ГБ).
Попробуйте следующее:
- Задайте большее время ожидания для команды [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) с помощью параметра `TimeoutSec`. По умолчанию время ожидания составляет 30 минут.
- Проверьте сетевое подключение между исходным компьютером и кластером. Если подключение медленное, рассмотрите возможность использовать машину с лучшим сетевым соединением.
Возможно, клиентский компьютер находится не в одном регионе с кластером, тогда перейдите на компьютер, который находится с ним в одном регионе или в регионе поблизости.
- Проверьте, достигнуто ли внешнее регулирование. Например, если хранилище образов настроено для использования хранилища Аzure, загрузку можно регулировать.

Причина: Отправка пакета успешно завершена, но время ожидания для [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype) истекает. Повторите
- [Выполните сжатие пакета](service-fabric-package-apps.md#compress-a-package) перед копированием в хранилище образов.
Сжатие уменьшает размер и число файлов, что, в свою очередь, снижает объем трафика и работу, которую необходимо выполнить Service Fabric. Операция загрузки может выполняться медленнее (особенно при выполнении сжатия), но регистрация и отмена регистрации типа приложения выполняются быстрее.
- Задайте большее время ожидания для выполнения команды [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype) с помощью параметра `TimeoutSec`.
- Задайте параметр `Async` для команды [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype). Эта команда возвращается, когда кластер принимает команду, и подготовка продолжается асинхронно. По этой причине здесь нет необходимости указывать большее значение времени ожидания. Чтобы вывести список зарегистрированных версий типов приложения и их состояние регистрации, используйте команду [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype). Эта команда используется, чтобы определить, что регистрация выполнена.

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

### <a name="deploy-application-package-with-many-files"></a>Развертывание пакета приложения с несколькими файлами

Проблема. Время ожидания для выполнения команды [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype) истекает для пакета приложения с несколькими файлами (несколько тысяч).
Попробуйте следующее:
- [Выполните сжатие пакета](service-fabric-package-apps.md#compress-a-package) перед копированием в хранилище образов. Сжатие уменьшает количество файлов.
- Задайте большее время ожидания для выполнения команды [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype) с помощью параметра `TimeoutSec`.
- Задайте параметр `Async` для команды [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype). Эта команда возвращается, когда кластер принимает команду, и подготовка продолжается асинхронно.
По этой причине здесь нет необходимости указывать большее значение времени ожидания. Чтобы вывести список зарегистрированных версий типов приложения и их состояние регистрации, используйте команду [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype). Эта команда используется, чтобы определить, что регистрация выполнена.

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

## <a name="next-steps"></a>Дальнейшие действия

[Создание пакета приложения](service-fabric-package-apps.md)

[Обновление приложения Service Fabric](service-fabric-application-upgrade.md)

[Общие сведения о работоспособности Service Fabric](service-fabric-health-introduction.md)

[Диагностика и устранение неполадок службы Service Fabric](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Моделирование приложения в Service Fabric](service-fabric-application-model.md)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-package-apps.md
[11]: service-fabric-application-upgrade.md

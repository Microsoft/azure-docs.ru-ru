---
title: Упаковка приложения Service Fabric Azure
description: Узнайте, как упаковать приложение Service Fabric Azure и как подготовиться к развертыванию в кластере.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 168e6d6dc7ab5bfeccc4e1dabc7bd50efcbe8f34
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98789708"
---
# <a name="package-an-application"></a>Создание пакета приложения

В этой статье описывается, как упаковать приложение Service Fabric и подготовить его для развертывания.

## <a name="package-layout"></a>Макет пакета

Манифест приложения, один или несколько манифестов служб и другие необходимые файлы пакетов должны быть организованы в определенный макет для развертывания в кластере структуры службы. Примеры манифестов в этой статье потребовали бы организации в следующую структуру каталогов.

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

Папки должны иметь имена, соответствующие значениям атрибута **Name** каждого соответствующего элемента. Например, если манифест служб содержал два пакета кода с именами **MyCodeA** и **MyCodeB**, то две папки с такими же именами будут содержать необходимые двоичные файлы для каждого пакета кода.

## <a name="use-setupentrypoint"></a>Использование SetupEntryPoint

Типичные сценарии использования **SetupEntryPoint** относятся к ситуации, когда необходимо запустить исполняемый файл перед запуском службы или выполнить операцию с повышенными привилегиями. Пример:

* Настройка и инициализация переменных среды, необходимых исполняемому файлу службы. Это касается не только исполняемых файлов, написанных с использованием моделей программирования Service Fabric. Например, npm.exe нужны определенные переменные среды, настроенные для развертывания приложения node.js.
* Настройка контроля доступа посредством установки сертификатов безопасности.

Дополнительные сведения о настройке **SetupEntryPoint** см. в статье [Настройка политики для точки входа настройки службы](service-fabric-application-runas-security.md) .

<a id="Package-App"></a>

## <a name="configure"></a>Configure

### <a name="build-a-package-by-using-visual-studio"></a>Создание пакета с помощью Visual Studio

Если вы использовали Visual Studio для создания приложения, можно использовать команду *Package* , чтобы автоматически создать пакет, соответствующий описанному выше макету.

Чтобы создать пакет, щелкните правой кнопкой мыши проект приложения в *Обозреватель решений* и выберите команду **пакет** :

![Создание пакета приложения с помощью Visual Studio][vs-package-command]

После завершения создания пакета в окне **Вывод** отобразятся сведения о расположении пакета. Шаг создания пакета выполняется автоматически при развертывании или отладке вашего приложения в Visual Studio.

### <a name="build-a-package-by-command-line"></a>Создание пакета с помощью командной строки

Приложение можно также упаковать программным способом, используя `msbuild.exe`. Этот файл выполняется в Visual Studio, так что результат будет тем же.

```shell
D:\Temp> msbuild HelloWorld.sfproj /t:Package
```

## <a name="test-the-package"></a>Тестирование пакета

Структуру пакета можно проверить локально средствами PowerShell, используя команду [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage) .
Эта команда проверит манифест на наличие ошибок при анализе, а также все ссылки. Эта команда позволяет проверить только правильность структуры каталогов и файлов в пакете.
Она не проверяет содержимое пакетов кода или данных, а только наличие всех необходимых файлов.

```powershell
Test-ServiceFabricApplicationPackage .\MyApplicationType
```

```Output
False
Test-ServiceFabricApplicationPackage : The EntryPoint MySetup.bat is not found.
FileName: C:\Users\servicefabric\AppData\Local\Temp\TestApplicationPackage_7195781181\nrri205a.e2h\MyApplicationType\MyServiceManifest\ServiceManifest.xml
```

Эта ошибка показывает, что файл *MySetup.bat* , на который ссылается манифест служб **SetupEntryPoint** , отсутствует в пакете кода. После добавления нужного файла будет выполнена проверка приложения.

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

```powershell
Test-ServiceFabricApplicationPackage .\MyApplicationType
```

```Output
True
```

Если для приложения определены [параметры приложения](service-fabric-manage-multiple-environment-app-configuration.md), их можно передать в командлет [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage) для полной проверки.

Если вы уже знаете, в какой кластер будет развернуто приложение, мы советуем передать параметр `ImageStoreConnectionString`. В этом случае для пакета будет выполнена проверка на совместимость с предыдущими версиями приложения, уже запущенными в кластере. Например, такая проверка позволит обнаружить, если пакет имеет одинаковую версию, но разное содержимое с развернутым ранее пакетом.  

Как только приложение будет упаковано должным образом и пройдет проверку, попробуйте сжать пакет для ускорения операций развертывания.

## <a name="compress-a-package"></a>Сжатие пакета

Если пакет содержит большое число файлов или имеет большой размер, можно выполнить сжатие пакета для более быстрого развертывания. Сжатие уменьшает число файлов и размер пакета.
[Отправка пакета приложения](service-fabric-deploy-remove-applications.md#upload-the-application-package), который был сжат, может занять больше времени по сравнению с отправкой пакета без сжатия, особенно если сжатие выполняется в процессе копирования. При сжатии [регистрация](service-fabric-deploy-remove-applications.md#register-the-application-package) и [отмена регистрации типа приложения](service-fabric-deploy-remove-applications.md#unregister-an-application-type) работают быстрее.

Для сжатых и несжатых пакетов используется одинаковый механизм развертывания. Если пакет сжат, он хранится в хранилище образов кластера именно в таком виде, а перед запуском приложения распаковывается на целевом узле.
Сжатие заменяет допустимый пакет Service Fabric его сжатой версией. Для этого нужны права на запись в папку. Если выполнить сжатие для уже сжатого пакета, никаких изменений не происходит.

Чтобы сжать пакет, выполните команду Powershell [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) с параметром `CompressPackage`. Чтобы распаковать сжатый пакет, выполните эту же команду с параметром `UncompressPackage`.

Следующая команда сжимает пакет, не копируя его в хранилище образов. Сжатый пакет можно скопировать в один или несколько кластеров Service Fabric, выполняя по мере необходимости командлет [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage) без флага `SkipCopy`.
Теперь пакет содержит ZIP-файлы для пакетов `code`, `config` и `data`. Манифест приложения и манифесты служб не упаковываются в ZIP-файлы, так как они требуются для многих внутренних операций. Например, при совместном использовании пакетов, извлечении имени типа приложения и версии для определенных проверок всегда требуется доступ к манифестам. Сжатие манифеста снизит эффективность таких операций.

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -CompressPackage -SkipCopy
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
       ServiceManifest.xml
       MyCode.zip
       MyConfig.zip
       MyData.zip

```

Кроме того, сжатие и копирование пакета можно выполнить одним действием, используя командлет [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage).
Если пакет имеет большой размер, обеспечьте достаточное время ожидания для сжатия пакета и его отправки в кластер.

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -ApplicationPackagePathInImageStore MyApplicationType -ImageStoreConnectionString fabric:ImageStore -CompressPackage -TimeoutSec 5400
```

Для внутренней проверки Service Fabric вычисляет контрольные суммы для пакетов приложений. Если используется сжатие, контрольные суммы вычисляются для ZIP-версий каждого пакета. При создании нового ZIP-файла из одного и того же пакета приложения получаются разные контрольные суммы. Чтобы избежать ошибок проверки, используйте [подготовку diff](service-fabric-application-upgrade-advanced.md). В этом режиме не следует включать пакеты без изменений в новую версию. Вместо этого на них нужно непосредственно ссылаться в новом манифесте службы.

Если подготовку diff выполнить нельзя и необходимо включить пакеты, создайте новые версии для пакетов `code`, `config` и `data`, чтобы избежать несовпадения контрольной суммы. Создание новых версий для пакетов без изменений необходимо, когда используется сжатый пакет, независимо от того, выполнено ли сжатие в предыдущей версии.

Теперь пакет правильно упакован, проверен и сжат (если нужно), так что все готово для [развертывания](service-fabric-deploy-remove-applications.md) в один или несколько кластеров Service Fabric.

### <a name="compress-packages-when-deploying-using-visual-studio"></a>Сжатие пакетов при развертывании с помощью Visual Studio

Можно настроить Visual Studio для сжатия пакетов при развертывании, добавив элемент `CopyPackageParameters` в профиль публикации, а также задав для атрибута `CompressPackage` значение `true`.

``` xml
    <PublishProfile xmlns="http://schemas.microsoft.com/2015/05/fabrictools">
        <ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com" />
        <ApplicationParameterFile Path="..\ApplicationParameters\Cloud.xml" />
        <CopyPackageParameters CompressPackage="true"/>
    </PublishProfile>
```

## <a name="create-an-sfpkg"></a>Создание пакета sfpkg

Начиная с версии 6.1 Service Fabric позволяет выполнять подготовку из внешнего хранилища.
В этом режиме нет необходимости копировать пакет приложения в хранилище образов. Вместо этого можно создать `sfpkg` и отправить его во внешнее хранилище, а затем указать URI для Service Fabric во время подготовки. Один и тот же пакет можно подготовить в нескольких кластерах. Подготовка из внешнего хранилища экономит время, необходимое для копирования пакета в каждый кластер.

Файл `sfpkg` — это ZIP-архив, содержащий изначальный пакет приложения с расширением .sfpkg.
Внутри ZIP-архива пакет приложения может быть сжатым или несжатым. Сжатие пакета приложения внутри ZIP-архива выполняется на уровнях кода, конфигурации и пакетов данных, как [упоминалось ранее](service-fabric-package-apps.md#compress-a-package).

Чтобы создать `sfpkg`, начните с папки, содержащей исходный пакет приложения, будь он сжатым или нет. Затем с помощью любой программы упакуйте папку в ZIP-архив с расширением .sfpkg. Например, используйте метод [ZipFile.CreateFromDirectory](/dotnet/api/system.io.compression.zipfile.createfromdirectory#System_IO_Compression_ZipFile_CreateFromDirectory_System_String_System_String_System_IO_Compression_CompressionLevel_System_Boolean_).

```csharp
ZipFile.CreateFromDirectory(appPackageDirectoryPath, sfpkgFilePath);
```

`sfpkg` необходимо отправить во внешнее хранилище за пределами диапазона — за пределами Service Fabric. Внешним хранилищем может быть любое хранилище, предоставляющее конечную точку REST HTTP или HTTPS. Во время подготовки Service Fabric выполняет операцию GET для скачивания пакета приложения `sfpkg`. Поэтому хранилище должно разрешать доступ на чтение к пакету.

Чтобы подготовить пакет, используйте внешнюю подготовку, которой требуется URI скачивания и сведения о типе приложения.

>[!NOTE]
> Сейчас подготовка на основе относительного пути в хранилище образов не поддерживает файлы `sfpkg`. Таким образом, `sfpkg` не следует копировать в хранилище образов.

## <a name="next-steps"></a>Дальнейшие действия

[Развертывание и удаление приложений][10] описание использования PowerShell для управления экземплярами приложения

В статье [Управление параметрами приложения для нескольких сред][11] описано, как настроить параметры и переменные среды для различных экземпляров приложений.

В статье [Настройка политик безопасности для приложения][12] содержатся данные о работе служб в соответствии с политиками безопасности, ограничивающими доступ.

<!--Image references-->
[vs-package-command]: ./media/service-fabric-package-apps/vs-package-command.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-manage-multiple-environment-app-configuration.md
[12]: service-fabric-application-runas-security.md

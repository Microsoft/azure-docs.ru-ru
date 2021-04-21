---
title: Выполнение скрипта при запуске службы Azure Service Fabric
description: Сведения о настройке политики для точки входа настройки службы Service Fabric и выполнения скрипта во время запуска службы.
author: athinanthny
ms.topic: conceptual
ms.date: 03/21/2018
ms.author: atsenthi
ms.openlocfilehash: a25f16f08ab8ae9564363f179d19d4b30c5315fa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96012533"
---
# <a name="run-a-service-startup-script-as-a-local-user-or-system-account"></a>Выполнение скрипта при запуске службы от имени локального пользователя или системной учетной записи
Перед запуском исполняемого файла службы Service Fabric могут потребоваться некоторые действия по настройке,  например для установки переменных среды. Вы можете указать в манифесте службы скрипт, который будет выполняться перед запуском исполняемого файла службы. Настроив политику запуска от имени для точки входа установки службы, вы можете изменить учетную запись, от имени которой выполняется исполняемый файл установки.  Отдельная точка входа установки позволяет на короткое время запускать конфигурацию с высоким уровнем привилегий, чтобы исполняемый узел службы не работал с высоким уровнем привилегий в течение продолжительного периода.

Привилегированная точка входа настройки определяется в [манифесте службы](service-fabric-application-and-service-manifests.md) как **SetupEntryPoint** и выполняется до запуска других точек входа с теми же учетными данными, что и служба Service Fabric (обычно это учетная запись *NetworkService*). Исполняемый файл, указанный для точки входа **EntryPoint**, обычно представлен длительно выполняемым узлом службы. Исполняемый файл **EntryPoint** запускается после успешного выхода из исполняемого файла **SetupEntryPoint**. Созданный им процесс отслеживается и перезапускается (снова начиная с точки входа **SetupEntryPoint**), если произойдет непредвиденное завершение его работы или сбой. 

## <a name="configure-the-service-setup-entry-point"></a>Настройка точки входа установки службы
Ниже приведен простой манифест службы без отслеживания состояния, в котором задан сценарий установки *MySetup.bat* в службе **SetupEntryPoint**.  Параметр **Arguments** используется для передачи аргументов в скрипт при его выполнении.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="MyStatelessServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Description>An example service manifest.</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="MyStatelessServiceType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>MySetup.bat</Program>
        <Arguments>MyValue</Arguments>
        <WorkingFolder>Work</WorkingFolder>        
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>MyStatelessService.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```
## <a name="configure-the-policy-for-a-service-setup-entry-point"></a>Определение политики для точки входа настройки службы
По умолчанию исполняемый файл точки входа установки службы запускается с теми же учетными данными, что и Service Fabric (обычно это учетная запись *NetworkService*).  В манифесте приложения вы можете изменить разрешения безопасности для запуска скрипта от имени администратора или локальной системной учетной записи.

### <a name="configure-the-policy-by-using-a-local-system-account"></a>Настройка политики с локальной системной учетной записью
В следующем примере манифест приложения настраивает запуск от имени учетной записи администратора (SetupAdminUser) для точки входа установки службы.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="MyStatelessService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyStatelessServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="MyStatelessService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyStatelessServiceType" InstanceCount="[MyStatelessService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

Сначала создайте раздел **Principals** с именем пользователя, например SetupAdminUser. Учетная запись SetupAdminUser входит в системную группу "Администраторы".

Затем в разделе **ServiceManifestImport** настройте политику для применения этого субъекта к точке **SetupEntryPoint**. Эта политика сообщает Service Fabric, что файл **MySetup.bat** должен всегда выполняться с правами администратора (от имени учетной записи SetupAdminUser). Так как вы *не* применяете политику к главной точке входа, код из файла **MyServiceHost.exe** будет запущен от имени системной учетной записи **NetworkService**. Это учетная запись по умолчанию, с использованием которой запускаются все точки входа службы.

### <a name="configure-the-policy-by-using-local-system-accounts"></a>Настройка политики с использованием учетных записей локальной системы
Как правило, выполнять скрипты запуска лучше от имени локальной системной учетной записи, а не от имени администратора. Обычно невозможно выполнять политику запуска от имени с учетными записями из группы администраторов, так как на компьютерах по умолчанию включен контроль учетных записей (UAC) на уровне пользователей. В таких случаях мы рекомендуем запускать SetupEntryPoint от имени учетной записи LocalSystem, а не локального пользователя из группы "Администраторы". В следующем примере показана настройка SetupEntryPoint для запуска с учетной записью LocalSystem.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="MyStatelessService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyStatelessServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
         <RunAsPolicy CodePackageRef="Code" UserRef="SetupLocalSystem" EntryPointType="Setup" />
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="MyStatelessService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyStatelessServiceType" InstanceCount="[MyStatelessService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
      <Users>
         <User Name="SetupLocalSystem" AccountType="LocalSystem" />
      </Users>
   </Principals>
</ApplicationManifest>
```

> [!NOTE]
> Чтобы запустить службу или точку входа установки в качестве **корня** на кластерах Linux, можно указать для **AccountType** значение **LocalSystem**.

## <a name="run-a-script-from-the-setup-entry-point"></a>Запуск скрипта из точки входа установки
Теперь добавьте в проект скрипт запуска, который будет выполняться с правами администратора. 

В Visual Studio щелкните проект службы правой кнопкой мыши и добавьте новый файл *MySetup.bat*.

Затем убедитесь, что файл *MySetup.bat* включен в пакет службы. По умолчанию он не включен. Щелкните файл правой кнопкой мыши, чтобы открыть контекстное меню, и выберите **Свойства**. Убедитесь, что в диалоговом окне "Свойства" параметр **Копировать в выходной каталог** имеет значение **Копировать, если новее**. Экран должен выглядеть следующим образом.

![Свойство CopyToOutput в Visual Studio для пакетного файла SetupEntryPoint][image1]

Теперь измените файл *MySetup.bat* и добавьте следующие команды, которые устанавливают системную переменную среды и выводят текстовый файл:

```
REM Set a system environment variable. This requires administrator privilege
setx -m TestVariable %*
echo System TestVariable set to > out.txt
echo %TestVariable% >> out.txt

REM To delete this system variable us
REM REG delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v TestVariable /f
```

Затем скомпилируйте и разверните решение в кластере локальной разработки. После запуска службы (это будет видно в [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)) вы можете убедиться в успешном выполнении файла MySetup.bat двумя способами. Откройте командную строку PowerShell и введите следующую команду.

```
PS C:\ [Environment]::GetEnvironmentVariable("TestVariable","Machine")
MyValue
```

В [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) запишите имя узла, на котором развернута и запущена служба. Предположим, что это узел Node 2. Затем перейдите в рабочую папку экземпляра приложения и найдите файл out.txt, в котором хранится значение **TestVariable**. Например, если служба была развернута на узле Node 2, для приложения **MyApplicationType** используйте такой путь:

```
C:\SfDevCluster\Data\_App\Node.2\MyApplicationType_App\work\out.txt
```

## <a name="run-powershell-commands-from-a-setup-entry-point"></a>Запуск команд PowerShell из точки входа установки
Чтобы запустить PowerShell из точки входа **SetupEntryPoint**, запустите **PowerShell.exe** в пакетном файле, который указывает на файл PowerShell. Сначала добавьте файл PowerShell в проект службы, например с именем **MySetup.ps1**. Не забудьте указать свойство *Копировать, если новее* , чтобы включить этот файл в состав пакета службы. В следующем примере показан пакетный файл для запуска файла PowerShell с именем MySetup.ps1, который задает системную переменную среды с именем **TestVariable**.

Файл MySetup.bat для запуска файла PowerShell.

```
powershell.exe -ExecutionPolicy Bypass -Command ".\MySetup.ps1"
```

В файле PowerShell добавьте следующую команду, чтобы задать системную переменную среды.

```
[Environment]::SetEnvironmentVariable("TestVariable", "MyValue", "Machine")
[Environment]::GetEnvironmentVariable("TestVariable","Machine") > out.txt
```

> [!NOTE]
> По умолчанию пакетный файл ищет файлы в папке приложения с именем **work**. В этом примере нам нужно, чтобы пакетный файл MySetup.bat при запуске нашел файл MySetup.ps1 в той же папке, в которой расположен **пакет кода** приложения. Чтобы изменить эту папку, задайте рабочую папку:
> 
> 

```xml
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    </ExeHost>
</SetupEntryPoint>
```

## <a name="debug-a-startup-script-locally-using-console-redirection"></a>Локальная отладка скрипта запуска с помощью перенаправления консоли
Иногда для отладки нужно просмотреть выходные данные консоли, в которой выполняется скрипт. С помощью манифеста службы вы можете задать для точки входа установки политику перенаправления консоли, которая записывает выходные данные в файл. Выходной файл сохраняется в папку приложения с именем **log** на том узле кластера, где развертывается и выполняется приложение. 

> [!WARNING]
> Никогда не используйте политику перенаправления консоли для приложений в рабочей среде, так как она может повлиять на отработку отказа приложения. Используйте ее *только* для локальной разработки и отладки.  
> 
> 

В приведенном ниже примере манифеста службы показано, как установить для перенаправления консоли значение FileRetentionCount.

```xml
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="10"/>
    </ExeHost>
</SetupEntryPoint>
```

Если теперь изменить файл MySetup.ps1 для вывода данных командой **Echo**, эти данные будут записаны в выходной файл, который можно использовать для отладки.

```
Echo "Test console redirection which writes to the application log folder on the node that the application is deployed to"
```

> [!WARNING]
> Завершив отладку сценария, немедленно удалите эту политику перенаправления консоли.



<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>Дальнейшие действия
* Ознакомьтесь с информацией о [безопасности приложений и служб](service-fabric-application-and-service-security.md).
* [Сведения о модели приложения](service-fabric-application-model.md)
* [Указание ресурсов в манифесте службы](service-fabric-service-manifest-resources.md)
* [Развертывание приложения](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png

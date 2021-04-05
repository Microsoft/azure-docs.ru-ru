---
title: Добавление конечной точки HTTPS с помощью Kestrel
description: Из этого руководства вы узнаете, как добавить конечную точку HTTPS в интерфейсную веб-службу ASP.NET Core с использованием Kestrel и развернуть приложение в кластере.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: c675f8ece8369bcfc0055343221ac82aea59dec1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91326241"
---
# <a name="tutorial-add-an-https-endpoint-to-an-aspnet-core-web-api-front-end-service-using-kestrel"></a>Руководство по Добавление конечной точки HTTPS в интерфейсную службу веб-API ASP.NET Core с использованием Kestrel

Это руководство представляет собой первую часть цикла.  Вы узнаете, как включить HTTPS в службе ASP.NET Core, работающей в Service Fabric. Когда вы закончите, у вас будет приложение для голосования с внешним веб-интерфейсом ASP.NET Core с поддержкой HTTPS, прослушивающего порт 443. Если вы не хотите вручную создавать приложение для голосования в статье [Руководство по развертыванию приложения в кластере Service Fabric в Azure](service-fabric-tutorial-deploy-app-to-party-cluster.md), вы можете [скачать исходный код](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/).

В третьей части цикла вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Определение конечной точки HTTPS в службе.
> * Настройка Kestrel для использования HTTPS
> * Настройка TLS/SSL-сертификата на узлах удаленного кластера
> * Предоставление службе NETWORK SERVICE доступа к закрытому ключу сертификата
> * Открытие порта 443 в подсистеме балансировки нагрузки Azure
> * Развертывание приложения в удаленном кластере.

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Создание приложения .NET Service Fabric](service-fabric-tutorial-deploy-app-to-party-cluster.md).
> * [Развертывание приложения в удаленном кластере](service-fabric-tutorial-deploy-app-to-party-cluster.md).
> * Добавление конечной точки HTTPS к интерфейсной службе ASP.NET Core.
> * [Настройка CI/CD с помощью Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Настройка мониторинга и диагностики приложения](service-fabric-tutorial-monitoring-aspnet.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этим руководством выполните следующие действия:

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Установите Visual Studio 2019](https://www.visualstudio.com/) версии 16.5 (или выше), а также рабочие нагрузки **Разработка для Azure** и **ASP.NET и разработка веб-приложений**.
* [Установите пакет SDK для Service Fabric](service-fabric-get-started.md)

## <a name="obtain-a-certificate-or-create-a-self-signed-development-certificate"></a>Получение сертификата или создание самозаверяющего сертификат разработки

Для рабочих приложений в качестве сертификата нужно использовать сертификат [из центра сертификации (ЦС)](https://wikipedia.org/wiki/Certificate_authority). В целях тестирования и разработки вы можете создать и использовать самозаверяющий сертификат. SDK Service Fabric предоставляет скрипт *CertSetup.ps1*, который создает самозаверяющий сертификат и импортирует его в хранилище сертификатов `Cert:\LocalMachine\My`. Откройте командную строку с правами администратора и выполните следующую команду для создания сертификата с субъектом CN=mytestcert:

```powershell
PS C:\program files\microsoft sdks\service fabric\clustersetup\secure> .\CertSetup.ps1 -Install -CertSubjectName CN=mytestcert
```

Если у вас уже есть файл сертификата PFX, для импорта сертификата в хранилище сертификатов `Cert:\LocalMachine\My` выполните следующие действия:

```powershell

PS C:\mycertificates> Import-PfxCertificate -FilePath .\mysslcertificate.pfx -CertStoreLocation Cert:\LocalMachine\My -Password (ConvertTo-SecureString "!Passw0rd321" -AsPlainText -Force)


   PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\My

Thumbprint                                Subject
----------                                -------
3B138D84C077C292579BA35E4410634E164075CD  CN=zwin7fh14scd.westus.cloudapp.azure.com
```

## <a name="define-an-https-endpoint-in-the-service-manifest"></a>Определение конечной точки HTTPS в манифесте службы

Запустите Visual Studio с правами **администратора** и откройте решение для голосования. В обозревателе решений откройте файл *VotingWeb/PackageRoot/ServiceManifest.xml*. Манифест службы определяет конечные точки службы.  Найдите раздел **Endpoints** и измените имеющуюся конечную точку ServiceEndpoint.  Измените имя на EndpointHttps, установите для протокола значение *https*, тип *Input* и порт *443*.  Сохраните изменения.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="configure-kestrel-to-use-https"></a>Настройка Kestrel для использования HTTPS

В обозревателе решений откройте файл *VotingWeb/VotingWeb.cs*.  Настройте Kestrel для использования HTTPS и найдите сертификат в хранилище `Cert:\LocalMachine\My`. Добавьте операторы using:

```csharp
using System.Net;
using Microsoft.Extensions.Configuration;
using System.Security.Cryptography.X509Certificates;
```

Обновите `ServiceInstanceListener`, чтобы использовать новую конечную точку *EndpointHttps* и ожидать передачи данных на порт 443. При настройке веб-узла для использования сервера Kestrel, нужно настроить на сервере Kestrel ожидание передачи данных на IPv6-адреса для всех сетевых интерфейсов: `opt.Listen(IPAddress.IPv6Any, port, listenOptions => {...}`.

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(FindMatchingCertificateBySubject());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

                .ConfigureServices(
                    services => services
                        .AddSingleton<HttpClient>(new HttpClient())
                        .AddSingleton<FabricClient>(new FabricClient())
                        .AddSingleton<StatelessServiceContext>(serviceContext))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                .UseUrls(url)
                .Build();
        }))
```

Кроме того, добавьте следующий метод, чтобы Kestrel мог найти сертификат в хранилище `Cert:\LocalMachine\My` с использованием субъекта.  

Замените &lt;your_CN_value&gt; на mytestcert, если вы создали самозаверяющий сертификат с помощью предыдущей команды PowerShell или используете CN своего сертификата.
Имейте в виду, что при локальном развертывании на `localhost` предпочтительнее использовать CN = localhost, чтобы избежать исключений при проверке подлинности.

```csharp
private X509Certificate2 FindMatchingCertificateBySubject(string subjectCommonName)
{
    using (var store = new X509Store(StoreName.My, StoreLocation.LocalMachine))
    {
        store.Open(OpenFlags.OpenExistingOnly | OpenFlags.ReadOnly);
        var certCollection = store.Certificates;
        var matchingCerts = new X509Certificate2Collection();
    
    foreach (var enumeratedCert in certCollection)
    {
      if (StringComparer.OrdinalIgnoreCase.Equals(subjectCommonName, enumeratedCert.GetNameInfo(X509NameType.SimpleName, forIssuer: false))
        && DateTime.Now < enumeratedCert.NotAfter
        && DateTime.Now >= enumeratedCert.NotBefore)
        {
          matchingCerts.Add(enumeratedCert);
        }
    }

        if (matchingCerts.Count == 0)
    {
        throw new Exception($"Could not find a match for a certificate with subject 'CN={subjectCommonName}'.");
    }
        
        return matchingCerts[0];
    }
}


```

## <a name="grant-network-service-access-to-the-certificates-private-key"></a>Предоставление службе NETWORK SER VICE доступа к закрытому ключу сертификата

На предыдущем шаге вы импортировали сертификат в хранилище `Cert:\LocalMachine\My` на компьютере разработки.  Теперь нужно предоставить доступ к закрытому ключу сертификата учетной записи, из которой запускается служба (по умолчанию NETWORK SERVICE). Этот шаг можно выполнить вручную (используя средство certlm.msc), но лучше автоматически запустить скрипт PowerShell, [настроив скрипт запуска](service-fabric-run-script-at-service-startup.md) в **SetupEntryPoint** манифеста службы.

>[!NOTE]
> Service Fabric поддерживает объявление сертификатов конечной точки по отпечатку или общему имени субъекта. В этом случае среда выполнения настроит привязку и список управления доступом (ACL) закрытого ключа сертификата для удостоверения, от имени которого выполняется служба. Среда выполнения также будет отслеживать изменения и продления сертификата и будет вносить изменения в список ACL для соответствующего закрытого ключа.

### <a name="configure-the-service-setup-entry-point"></a>Настройка точки входа установки службы

В обозревателе решений откройте файл *VotingWeb/PackageRoot/ServiceManifest.xml*.  В разделе **CodePackage** добавьте узел **SetupEntryPoint**, а затем узел **ExeHost**.  В **ExeHost** установите для **Program** значение Setup.bat и для **WorkingFolder** — CodePackage.  При запуске службы VotingWeb скрипт Setup.bat выполняется в папке CodePackage до запуска файла VotingWeb.exe.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>Setup.bat</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </SetupEntryPoint>

    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

### <a name="add-the-batch-and-powershell-setup-scripts"></a>Добавление пакета и скриптов установки PowerShell

Чтобы запустить PowerShell из точки входа **SetupEntryPoint**, запустите PowerShell.exe в пакетном файле, который указывает на файл PowerShell. Сначала добавьте пакетный файл в проект службы.  В обозревателе решений щелкните правой кнопкой мыши **VotingWeb**, выберите **Добавить**->**Новый элемент** и добавьте новый файл с именем Setup.bat.  Измените файл *Setup.bat*, добавив следующую команду:

```cmd
powershell.exe -ExecutionPolicy Bypass -Command ".\SetCertAccess.ps1"
```

Измените свойства файла *Setup.bat*, чтобы установить для параметра **Копировать в выходной каталог** значение "Копировать, если новее".

![Настройка свойств файла][image1]

В обозревателе решений щелкните правой кнопкой мыши **VotingWeb** и щелкните **Добавить**->**Новый элемент** и добавьте новый файл с именем SetCertAccess.ps1.  Измените файл *SetCertAccess.ps1* и добавьте следующий скрипт:

```powershell
$subject="mytestcert"
$userGroup="NETWORK SERVICE"

Write-Host "Checking permissions to certificate $subject.." -ForegroundColor DarkCyan

$cert = (gci Cert:\LocalMachine\My\ | where { $_.Subject.Contains($subject) })[-1]

if ($cert -eq $null)
{
    $message="Certificate with subject:"+$subject+" does not exist at Cert:\LocalMachine\My\"
    Write-Host $message -ForegroundColor Red
    exit 1;
}elseif($cert.HasPrivateKey -eq $false){
    $message="Certificate with subject:"+$subject+" does not have a private key"
    Write-Host $message -ForegroundColor Red
    exit 1;
}else
{
    $keyName=$cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName

    $keyPath = "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\"
    $fullPath=$keyPath+$keyName
    $acl=(Get-Item $fullPath).GetAccessControl('Access')


    $hasPermissionsAlready = ($acl.Access | where {$_.IdentityReference.Value.Contains($userGroup.ToUpperInvariant()) -and $_.FileSystemRights -eq [System.Security.AccessControl.FileSystemRights]::FullControl}).Count -eq 1

    if ($hasPermissionsAlready){
        Write-Host "Account $userGroup already has permissions to certificate '$subject'." -ForegroundColor Green
        return $false;
    } else {
        Write-Host "Need add permissions to '$subject' certificate..." -ForegroundColor DarkYellow

        $permission=$userGroup,"Full","Allow"
        $accessRule=new-object System.Security.AccessControl.FileSystemAccessRule $permission
        $acl.AddAccessRule($accessRule)
        Set-Acl $fullPath $acl

        Write-Output "Permissions were added"

        return $true;
    }
}

```

Измените свойства файла *SetCertAccess.ps1*, чтобы установить для параметра **Копировать в выходной каталог** значение "Копировать более позднюю версию".

### <a name="run-the-setup-script-as-a-local-administrator"></a>Запуск скрипта установки с правами локального администратора

По умолчанию исполняемый файл точки входа установки службы запускается с теми же учетными данными, что и Service Fabric (обычно это учетная запись NetworkService). Для выполнения *SetCertAccess.ps1* требуются права администратора. В манифесте приложения вы можете изменить разрешения безопасности для выполнения скрипта запуска от имени локальной учетной записи администратора.

В обозревателе решений откройте *Voting/ApplicationPackageRoot/ApplicationManifest.xml*. Сначала создайте раздел **Участники** и добавьте нового пользователя, например SetupAdminUser. Добавьте учетную запись SetupAdminUser в системную группу "Администраторы".
Затем в разделе VotingWebPkg **ServiceManifestImport** настройте **RunAsPolicy**, чтобы применить субъект SetupAdminUser к точке входа установки. Эта политика сообщает Service Fabric, что файл MySetup.bat должен всегда выполняться с правами администратора (от имени учетной записи SetupAdminUser).

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
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

## <a name="run-the-application-locally"></a>Локальный запуск приложения

В обозревателе решений выберите приложение для **голосования** и установите для свойства **URL-адрес приложения** значение https:\//localhost:443.

Сохраните все файлы и нажмите клавишу F5 для запуска приложения локально.  После развертывания приложения веб-браузер откроется на странице https:\//localhost:443. Если вы используете самозаверяющий сертификат, вы увидите предупреждение о том, что ваш компьютер не доверяет безопасности этого веб-сайта.  Перейдите к веб-странице.

![Снимок экрана: приложение Service Fabric Voting Sample, выполняющееся в окне браузера с URL-адресом https://localhost/.][image2]

## <a name="install-certificate-on-cluster-nodes"></a>Установка сертификата на узлы кластера

Перед развертыванием приложения в Azure установите сертификат в хранилище всех удаленных узлов кластера `Cert:\LocalMachine\My`.  Службы можно переместить в различные узлы кластера.  Когда интерфейсная веб-служба запускается на узле кластера, скрипт запуска будет искать сертификат и настраивать разрешения доступа.

Экспортируйте его в формате PFX-файла. Откройте приложение certlm.msc и щелкните **Личные**>**Сертификаты**.  В файле *mytestcert* щелкните сертификат правой кнопкой мыши и выберите **Все задачи**>**Экспорт**.

![Экспорт сертификата][image4]

В окне мастера экспорта выберите **Да, экспортировать закрытый ключ** и выберите формат PFX.  Экспортируйте файл в *C:\Users\sfuser\votingappcert.pfx*.

Затем установите сертификат в удаленном кластере, используя [указанные скрипты PowerShell](./scripts/service-fabric-powershell-add-application-certificate.md).

> [!Warning]
> Самозаверяющего сертификата достаточно для разработки и тестирования приложений. Для рабочих приложений вместо самозаверяющего сертификата в качестве сертификата нужно использовать сертификат [из центра сертификации (ЦС)](https://wikipedia.org/wiki/Certificate_authority).

## <a name="open-port-443-in-the-azure-load-balancer-and-virtual-network"></a>Открытие порта 443 в подсистеме балансировки нагрузки Azure и виртуальной сети

Откройте порт 443 в подсистеме балансировки нагрузки, если это еще не сделано.

```powershell
$probename = "AppPortProbe6"
$rulename="AppPortLBRule6"
$RGname="voting_RG"
$port=443

# Get the load balancer resource
$resource = Get-AzResource | Where {$_.ResourceGroupName –eq $RGname -and $_.ResourceType -eq "Microsoft.Network/loadBalancers"}
$slb = Get-AzLoadBalancer -Name $resource.Name -ResourceGroupName $RGname

# Add a new probe configuration to the load balancer
$slb | Add-AzLoadBalancerProbeConfig -Name $probename -Protocol Tcp -Port $port -IntervalInSeconds 15 -ProbeCount 2

# Add rule configuration to the load balancer
$probe = Get-AzLoadBalancerProbeConfig -Name $probename -LoadBalancer $slb
$slb | Add-AzLoadBalancerRuleConfig -Name $rulename -BackendAddressPool $slb.BackendAddressPools[0] -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -Probe $probe -Protocol Tcp -FrontendPort $port -BackendPort $port

# Set the goal state for the load balancer
$slb | Set-AzLoadBalancer
```

Сделайте то же самое для связанной виртуальной сети.

```powershell
$rulename="allowAppPort$port"
$nsgname="voting-vnet-security"
$RGname="voting_RG"
$port=443

# Get the NSG resource
$nsg = Get-AzNetworkSecurityGroup -Name $nsgname -ResourceGroupName $RGname

# Add the inbound security rule.
$nsg | Add-AzNetworkSecurityRuleConfig -Name $rulename -Description "Allow app port" -Access Allow `
    -Protocol * -Direction Inbound -Priority 3891 -SourceAddressPrefix "*" -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange $port

# Update the NSG.
$nsg | Set-AzNetworkSecurityGroup
```

## <a name="deploy-the-application-to-azure"></a>Развертывание приложения в Azure

Сохраните все файлы, переключитесь с версии для отладки на версию выпуска и нажмите F6 для повторного создания.  Щелкните правой кнопкой мыши проект в **приложении для голосования** и выберите пункт **Опубликовать**. Выберите конечную точку подключения кластера, созданного в статье [Руководство по развертыванию приложения в кластере Service Fabric в Azure](service-fabric-tutorial-deploy-app-to-party-cluster.md) или выберите другой кластер.  Нажмите кнопку **Опубликовать**, чтобы опубликовать приложение на удаленном кластере.

При развертывании приложения откройте веб-браузер и перейдите по адресу `https://mycluster.region.cloudapp.azure.com:443` (обновите URL-адрес, используя конечную точку подключения кластера). Если вы используете самозаверяющий сертификат, вы увидите предупреждение о том, что ваш компьютер не доверяет безопасности этого веб-сайта.  Перейдите к веб-странице.

![Снимок экрана: приложение Service Fabric Voting Sample, выполняющееся в окне браузера с URL-адресом https://mycluster.region.cloudapp.azure.com:443.][image3]

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Определение конечной точки HTTPS в службе.
> * Настройка Kestrel для использования HTTPS
> * Настройка TLS/SSL-сертификата на узлах удаленного кластера
> * Предоставление службе NETWORK SERVICE доступа к закрытому ключу сертификата
> * Открытие порта 443 в подсистеме балансировки нагрузки Azure
> * Развертывание приложения в удаленном кластере.

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Настройка CI/CD с помощью Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)

[image1]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/SetupBatProperties.png
[image2]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppLocal.png
[image3]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppAzure.png
[image4]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/ExportCert.png

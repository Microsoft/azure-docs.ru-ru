---
title: Безопасное подключение к кластеру Azure Service Fabric
description: Сведения о способах проверки подлинности клиентского доступа к кластеру Service Fabric, а также об обеспечении безопасного обмена данными между клиентами и кластером.
ms.topic: conceptual
ms.date: 01/29/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 3a669f7400eb2daf2d71f6caed41560020699c58
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791127"
---
# <a name="connect-to-a-secure-cluster"></a>Безопасное подключение к кластеру

Когда клиент подключается к узлу кластера Service Fabric, он может пройти проверку подлинности и установить безопасную связь на основе сертификатов или с помощью Azure Active Directory (AAD). Такая проверка подлинности гарантирует, что только авторизованные пользователи могут получить доступ к кластеру и развернутым приложениям для выполнения задач управления.  Конфигурацию обеспечения безопасности на основе сертификатов или AAD необходимо предварительно включить в кластере при его создании.  Дополнительные сведения о сценариях обеспечения безопасности кластеров см. в разделе [Безопасность кластера](service-fabric-cluster-security.md). Если безопасность в кластере обеспечивается сертификатами, то на компьютере, который подключается к нему, необходимо [настроить сертификат клиента](service-fabric-connect-to-secure-cluster.md#connectsecureclustersetupclientcert). 

<a id="connectsecureclustercli"></a> 

## <a name="connect-to-a-secure-cluster-using-azure-service-fabric-cli-sfctl"></a>Подключение к защищенному кластеру с помощью интерфейса командной строки Azure Service Fabric CLI (sfctl)

К защищенному кластеру можно подключиться с помощью интерфейса командной строки Service Fabric CLI (sfctl) несколькими способами. При использовании сертификата клиента для проверки подлинности сведения о сертификате должны соответствовать сертификату, развернутому в узлах кластера. Если сертификат выдан центром сертификации (ЦС), необходимо дополнительно указать доверенные ЦС.

Можно подключиться к кластеру с помощью команды `sfctl cluster select`.

Сертификаты клиента можно указать как сертификат и пару ключей или как отдельный PFX-файл. Вам автоматически будет предложено ввести пароль для защищенных файлов формата PEM. Если вы получили сертификат клиента в файле формата PFX, сначала преобразуйте формат PFX в PEM, используя следующую команду. 

```shell
openssl pkcs12 -in your-cert-file.pfx -out your-cert-file.pem -nodes -passin pass:your-pfx-password
```

Если PFX-файл не защищен паролем, укажите -passin pass: в качестве последнего параметра.

Чтобы указать сертификат клиента как PEM-файл, укажите путь к файлу в аргументе `--pem`. Пример:

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem
```

Прежде чем выполнять любые команды, для защищенных паролем PEM-файлов необходимо будет ввести пароль.

Чтобы указать сертификат, пара ключей использует аргументы `--cert` и `--key`, чтобы указать пути для каждого соответствующего файла.

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --cert ./client.crt --key ./keyfile.key
```

Иногда сертификаты, используемые для защиты тестового кластера и кластера для разработки не проходят проверку сертификата. Чтобы не выполнять проверку сертификата, укажите параметр `--no-verify`. Пример:

> [!WARNING]
> При подключении к производственным кластерам Service Fabric не используйте параметр `no-verify`.

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --no-verify
```

Кроме того, вы можете указать пути к каталогам доверенных сертификатов ЦС или отдельных сертификатов. Чтобы указать эти пути, используйте аргумент `--ca`. Пример:

```shell
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem --ca ./trusted_ca
```

После подключения вы сможете взаимодействовать с кластером с помощью [других команд sfctl](service-fabric-cli.md).

<a id="connectsecurecluster"></a>

## <a name="connect-to-a-cluster-using-powershell"></a>Подключение к кластеру с помощью PowerShell
Перед выполнением операций в кластере с помощью PowerShell к этому кластеру сначала нужно подключиться. Это подключение используется для выполнения всех последующих команд в течение определенного сеанса PowerShell.

### <a name="connect-to-an-unsecure-cluster"></a>Подключение к незащищенному кластеру

Чтобы подключиться к незащищенному кластеру, добавьте в следующую команду **Connect-ServiceFabricCluster** адрес конечной точки кластера.

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 
```

### <a name="connect-to-a-secure-cluster-using-azure-active-directory"></a>Подключение к защищенному кластеру с помощью Azure Active Directory

Чтобы подключиться к защищенному кластеру, использующему Azure Active Directory для проверки подлинности административного доступа, предоставьте отпечаток сертификата кластера и используйте флаг *AzureActiveDirectory*.  

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
-ServerCertThumbprint <Server Certificate Thumbprint> `
-AzureActiveDirectory
```

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>Подключение к защищенному кластеру с использованием сертификата клиента
Чтобы подключиться к защищенному кластеру, использующему сертификаты клиента для проверки подлинности административного доступа, выполните следующую команду PowerShell. 

#### <a name="connect-using-certificate-common-name"></a>Подключение с помощью общего имени сертификата
Предоставьте общее имя сертификата кластера, а также общее имя сертификата клиента с разрешениями на управление кластером. Сведения о сертификате должны соответствовать сертификату на узлах кластера.

```powershell
Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCommonName <certificate common name>  `
    -FindType FindBySubjectName `
    -FindValue <certificate common name> `
    -StoreLocation CurrentUser `
    -StoreName My 
```
*ServerCommonName* — общее имя сертификата сервера, установленного на узлах кластера. *FindValue* — общее имя сертификата клиента администрирования. Когда все параметры указаны, команда выглядит так:
```powershell
$ClusterName= "sf-commonnametest-scus.southcentralus.cloudapp.azure.com:19000"
$certCN = "sfrpe2eetest.southcentralus.cloudapp.azure.com"

Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCommonName $certCN  `
    -FindType FindBySubjectName `
    -FindValue $certCN `
    -StoreLocation CurrentUser `
    -StoreName My 
```

#### <a name="connect-using-certificate-thumbprint"></a>Подключение с помощью отпечатка сертификата
Предоставьте отпечаток сертификата кластера, а также отпечаток сертификата клиента с разрешениями на управление кластером. Сведения о сертификате должны соответствовать сертификату на узлах кластера.

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `  
          -KeepAliveIntervalInSec 10 `  
          -X509Credential -ServerCertThumbprint <Certificate Thumbprint> `  
          -FindType FindByThumbprint -FindValue <Certificate Thumbprint> `  
          -StoreLocation CurrentUser -StoreName My
```

*ServerCertThumbprint* — отпечаток сертификата сервера, установленного на узлах кластера. *FindValue* — отпечаток сертификата клиента администрирования.  Когда все параметры указаны, команда выглядит так:

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint clustername.westus.cloudapp.azure.com:19000 `  
          -KeepAliveIntervalInSec 10 `  
          -X509Credential -ServerCertThumbprint A8136758F4AB8962AF2BF3F27921BE1DF67F4326 `  
          -FindType FindByThumbprint -FindValue 71DE04467C9ED0544D021098BCD44C71E183414E `  
          -StoreLocation CurrentUser -StoreName My 
```

### <a name="connect-to-a-secure-cluster-using-windows-active-directory"></a>Подключение к защищенному кластеру с помощью Windows Active Directory
При развертывании автономного кластера с помощью группы безопасности Active Directory подключитесь к кластеру, добавив параметр "WindowsCredential".

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <Cluster FQDN>:19000 `
          -WindowsCredential
```

<a id="connectsecureclusterfabricclient"></a>

## <a name="connect-to-a-cluster-using-the-fabricclient-apis"></a>Подключение к кластеру с помощью интерфейсов API FabricClient
В пакете SDK для Service Fabric предусмотрен класс [FabricClient](/dotnet/api/system.fabric.fabricclient) для управления кластером. Чтобы использовать интерфейсы API FabricClient, получите пакет NuGet Microsoft.ServiceFabric.

### <a name="connect-to-an-unsecure-cluster"></a>Подключение к незащищенному кластеру

Чтобы подключиться к удаленному незащищенному кластеру, создайте экземпляр FabricClient и укажите адрес кластера.

```csharp
FabricClient fabricClient = new FabricClient("clustername.westus.cloudapp.azure.com:19000");
```

В коде, выполняемом в кластере (например, в надежной службе), создайте класс FabricClient, *не* указывая адрес кластера. Класс FabricClient подключается к локальному шлюзу управления на узле, где в настоящее время выполняется код, без дополнительного перехода по сети.

```csharp
FabricClient fabricClient = new FabricClient();
```

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>Подключение к защищенному кластеру с использованием сертификата клиента

У узлов в кластере должны быть действительные сертификаты, общее имя или DNS-имя которых в сети SAN отображается в [свойстве RemoteCommonNames](/dotnet/api/system.fabric.x509credentials), заданном для [FabricClient](/dotnet/api/system.fabric.fabricclient). Это обеспечивает взаимную проверку подлинности между клиентом и узлами кластера.

```csharp
using System.Fabric;
using System.Security.Cryptography.X509Certificates;

string clientCertThumb = "71DE04467C9ED0544D021098BCD44C71E183414E";
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string CommonName = "www.clustername.westus.azure.com";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var xc = GetCredentials(clientCertThumb, serverCertThumb, CommonName);
var fc = new FabricClient(xc, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

static X509Credentials GetCredentials(string clientCertThumb, string serverCertThumb, string name)
{
    X509Credentials xc = new X509Credentials();
    xc.StoreLocation = StoreLocation.CurrentUser;
    xc.StoreName = "My";
    xc.FindType = X509FindType.FindByThumbprint;
    xc.FindValue = clientCertThumb;
    xc.RemoteCommonNames.Add(name);
    xc.RemoteCertThumbprints.Add(serverCertThumb);
    xc.ProtectionLevel = ProtectionLevel.EncryptAndSign;
    return xc;
}
```

### <a name="connect-to-a-secure-cluster-interactively-using-azure-active-directory"></a>Интерактивное подключение к защищенному кластеру с помощью Azure Active Directory

В следующем примере в качестве удостоверения клиента используется Azure Active Directory, а сертификат сервера — в качестве удостоверения сервера.

При подключении к кластеру автоматически появляется всплывающее окно интерактивного входа в систему.

```csharp
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);

var fc = new FabricClient(claimsCredentials, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}
```

### <a name="connect-to-a-secure-cluster-non-interactively-using-azure-active-directory"></a>Неинтерактивное подключение к защищенному кластеру с помощью Azure Active Directory

Приведенный ниже пример основан на пространстве имен Microsoft.IdentityModel.Clients.ActiveDirectory версии 2.19.208020213.

Дополнительные сведения о получении маркеров AAD см. в описании [Microsoft.IdentityModel.Clients.ActiveDirectory](/dotnet/api/microsoft.identitymodel.clients.activedirectory).

```csharp
string tenantId = "C15CFCEA-02C1-40DC-8466-FBD0EE0B05D2";
string clientApplicationId = "118473C2-7619-46E3-A8E4-6DA8D5F56E12";
string webApplicationId = "53E6948C-0897-4DA6-B26A-EE2A38A690B4";

string token = GetAccessToken(
    tenantId,
    webApplicationId,
    clientApplicationId,
    "urn:ietf:wg:oauth:2.0:oob");

string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);
claimsCredentials.LocalClaims = token;

var fc = new FabricClient(claimsCredentials, connection);

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

...

static string GetAccessToken(
    string tenantId,
    string resource,
    string clientId,
    string redirectUri)
{
    string authorityFormat = @"https://login.microsoftonline.com/{0}";
    string authority = string.Format(CultureInfo.InvariantCulture, authorityFormat, tenantId);
    var authContext = new AuthenticationContext(authority);

    var authResult = authContext.AcquireToken(
        resource,
        clientId,
        new UserCredential("TestAdmin@clustenametenant.onmicrosoft.com", "TestPassword"));
    return authResult.AccessToken;
}

```

### <a name="connect-to-a-secure-cluster-without-prior-metadata-knowledge-using-azure-active-directory"></a>Подключение к защищенному кластеру без предварительного получения метаданных с помощью Azure Active Directory

В следующем примере используется неинтерактивное получение маркера, но используемый подход пригоден и для реализации интерактивного получения маркера. Метаданные Azure Active Directory, необходимые для получения маркера, считываются из конфигурации кластера.

```csharp
string serverCertThumb = "A8136758F4AB8962AF2BF3F27921BE1DF67F4326";
string connection = "clustername.westus.cloudapp.azure.com:19000";

var claimsCredentials = new ClaimsCredentials();
claimsCredentials.ServerThumbprints.Add(serverCertThumb);

var fc = new FabricClient(claimsCredentials, connection);

fc.ClaimsRetrieval += (o, e) =>
{
    return GetAccessToken(e.AzureActiveDirectoryMetadata);
};

try
{
    var ret = fc.ClusterManager.GetClusterManifestAsync().Result;
    Console.WriteLine(ret.ToString());
}
catch (Exception e)
{
    Console.WriteLine("Connect failed: {0}", e.Message);
}

...

static string GetAccessToken(AzureActiveDirectoryMetadata aad)
{
    var authContext = new AuthenticationContext(aad.Authority);

    var authResult = authContext.AcquireToken(
        aad.ClusterApplication,
        aad.ClientApplication,
        new UserCredential("TestAdmin@clustenametenant.onmicrosoft.com", "TestPassword"));
    return authResult.AccessToken;
}

```

<a id="connectsecureclustersfx"></a>

## <a name="connect-to-a-secure-cluster-using-service-fabric-explorer"></a>Подключение к защищенному кластеру с помощью Service Fabric Explorer
Чтобы подключиться к [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) для заданного кластера, просто перейдите в браузере по адресу:

`http://<your-cluster-endpoint>:19080/Explorer`

Полный URL-адрес доступен также на панели основных компонентов кластера портала Azure.

Чтобы подключиться к безопасному кластеру в Windows или OS X с помощью браузера, можно импортировать сертификат клиента. Браузер запросит его, чтобы подключиться к кластеру.  Для компьютеров Linux необходимо импортировать сертификат с помощью расширенных параметров браузера (в каждом браузере используется свой механизм), а затем указать его расположение на диске. Дополнительные сведения см. в разделе [Настройка сертификата клиента на удаленном компьютере](#connectsecureclustersetupclientcert).

### <a name="connect-to-a-secure-cluster-using-azure-active-directory"></a>Подключение к защищенному кластеру с помощью Azure Active Directory

Чтобы подключиться к кластеру, защищенному с помощью AAD, перейдите в браузере по адресу:

`https://<your-cluster-endpoint>:19080/Explorer`

Вам будет автоматически предложено войти в систему с помощью AAD.

### <a name="connect-to-a-secure-cluster-using-a-client-certificate"></a>Подключение к защищенному кластеру с использованием сертификата клиента

Чтобы подключиться к кластеру, защищенному с использованием сертификатов, перейдите в браузере по адресу:

`https://<your-cluster-endpoint>:19080/Explorer`

После этого автоматически появится запрос на выбор сертификата клиента.

<a id="connectsecureclustersetupclientcert"></a>

## <a name="set-up-a-client-certificate-on-the-remote-computer"></a>Настройка сертификата клиента на удаленном компьютере

Для обеспечения безопасности кластера следует использовать по крайней мере два сертификата: один для кластера и сервера, а второй для клиентского доступа.  Рекомендуется также использовать дополнительные сертификаты и сертификаты клиентского доступа.  Чтобы защитить обмен данными между клиентом и узлом кластера с помощью сертификатов, сначала необходимо получить и установить сертификат клиента. Его можно установить в личное хранилище на локальном компьютере или в личное хранилище текущего пользователя.  Отпечаток в сертификате сервера также нужен, чтобы клиент мог аутентифицировать кластер.

* В Windows: дважды щелкните PFX-файл и следуйте инструкциям на экране для установки сертификата в личном хранилище, `Certificates - Current User\Personal\Certificates`. Кроме того, вы можете использовать команду PowerShell.

    ```powershell
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
            -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
            -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
    ```

    Если это самозаверяющий сертификат, то перед использованием для подключения к безопасному кластеру его необходимо импортировать в хранилище "Доверенные лица" на своем компьютере.

    ```powershell
    Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople `
    -FilePath C:\docDemo\certs\DocDemoClusterCert.pfx `
    -Password (ConvertTo-SecureString -String test -AsPlainText -Force)
    ```

* В Mac: дважды щелкните PFX-файл и следуйте инструкциям на экране, чтобы установить сертификат в цепочке ключей.

## <a name="next-steps"></a>Дальнейшие действия

* [Обновление кластера Service Fabric](service-fabric-cluster-upgrade.md)
* [Управление приложениями Service Fabric в Visual Studio](service-fabric-manage-application-in-visual-studio.md)
* [Общие сведения о модели работоспособности в Service Fabric](service-fabric-health-introduction.md)
* [Безопасность приложений и запуск от имени](service-fabric-application-runas-security.md)
* [Командная строка Azure Service Fabric](service-fabric-cli.md)

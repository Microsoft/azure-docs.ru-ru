---
title: Не удается получить доступ к файлам Data Lakeного хранилища в Azure HDInsight
description: Не удается получить доступ к файлам Data Lakeного хранилища в Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 08/13/2019
ms.openlocfilehash: f4c5a23b604334952730fcc4cf1fcb3fcbed6237
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944402"
---
# <a name="unable-to-access-data-lake-storage-files-in-azure-hdinsight"></a>Не удается получить доступ к файлам Data Lakeного хранилища в Azure HDInsight

В этой статье описываются действия по устранению неполадок и возможные способы решения проблем при взаимодействии с кластерами Azure HDInsight.

## <a name="issue-acl-verification-failed"></a>Ошибка: сбой проверки ACL

Появится сообщение об ошибке следующего вида:

```
LISTSTATUS failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.).
```

### <a name="cause"></a>Причина

Возможно, пользователь отменил разрешения субъекта-службы (Service Principal, SP) для файлов и папок.

### <a name="resolution"></a>Решение

1. Убедитесь, что у пакета обновления есть разрешения "x" для обхода по пути. Дополнительные сведения см. в разделе [Разрешения](https://hdinsight.github.io/ClusterCRUD/ADLS/adls-create-permission-setup.html). Пример `dfs` команды для проверки доступа к файлам и папкам в Data Lake учетной записи хранения:

    ```
    hdfs dfs -ls /<path to check access>
    ```

1. Настройте необходимые разрешения для доступа к пути на основе выполняемой операции чтения-записи. Дополнительные сведения о разрешениях, необходимых для различных операций файловой системы, см. здесь.

---

## <a name="issue-service-principal-certificate-expiry"></a>Ошибка: истекает срок действия сертификата субъекта-службы

Появится сообщение об ошибке следующего вида:

```
Token Refresh failed - Received invalid http response: 500
```

### <a name="cause"></a>Причина

Возможно, истек срок действия сертификата, предоставленного для доступа субъекта-службы.

1. SSH в головного узла. Проверьте доступ к учетной записи хранения с помощью следующей `dfs` команды:

    ```
    hdfs dfs -ls /
    ```

1. Убедитесь, что сообщение об ошибке похоже на следующие выходные данные:

    ```
    {"stderr": "-ls: Token Refresh failed - Received invalid http response: 500, text = Response{protocol=http/1.1, code=500, message=Internal Server Error, url=http://gw0-abccluster.24ajrd4341lebfgq5unsrzq0ue.fx.internal.cloudapp.net:909/api/oauthtoken}}...
    ```

1. Получите один из URL-адресов из `core-site.xml property`  -  `fs.azure.datalake.token.provider.service.urls` .

1. Выполните следующую команду, чтобы получить токен OAuth.

    ```
    curl gw0-abccluster.24ajrd4341lebfgq5unsrzq0ue.fx.internal.cloudapp.net:909/api/oauthtoken
    ```

1. Выходные данные для допустимого субъекта-службы должны быть примерно такими:

    ```
    {"AccessToken":"MIIGHQYJKoZIhvcNAQcDoIIGDjCCBgoCAQA…….","ExpiresOn":1500447750098}
    ```

1. Если срок действия сертификата субъекта-службы истек, выходные данные будут выглядеть примерно так:

    ```
    Exception in OAuthTokenController.GetOAuthToken: 'System.InvalidOperationException: Error while getting the OAuth token from AAD for AppPrincipalId 23abe517-2ffd-4124-aa2d-7c224672cae2, ResourceUri https://management.core.windows.net/, AADTenantId https://login.windows.net/80abc8bf-86f1-41af-91ab-2d7cd011db47, ClientCertificateThumbprint C49C25705D60569884EDC91986CEF8A01A495783 ---> Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException: AADSTS70002: Error validating credentials. AADSTS50012: Client assertion contains an invalid signature. **[Reason - The key used is expired.**, Thumbprint of key used by client: 'C49C25705D60569884EDC91986CEF8A01A495783', Found key 'Start=08/03/2016, End=08/03/2017, Thumbprint=C39C25705D60569884EDC91986CEF8A01A4956D1', Configured keys: [Key0:Start=08/03/2016, End=08/03/2017, Thumbprint=C39C25705D60569884EDC91986CEF8A01A4956D1;]]
    Trace ID: e4d34f1c-a584-47f5-884e-1235026d5000
    Correlation ID: a44d870e-6f23-405a-8b23-9b44aebfa4bb
    Timestamp: 2017-10-06 20:44:56Z ---> System.Net.WebException: The remote server returned an error: (401) Unauthorized.
    at System.Net.HttpWebRequest.GetResponse()
    at Microsoft.IdentityModel.Clients.ActiveDirectory.HttpWebRequestWrapper.<GetResponseSyncOrAsync>d__2.MoveNext()
    ```

1. Любые другие ошибки, связанные с Azure Active Directory, а также ошибки, связанные с сертификатом, можно узнать с помощью команды ping по URL-адресу шлюза, чтобы получить маркер OAuth.

1. Если при попытке получить доступ к ADLS из кластера HDI возникает следующая ошибка. Проверьте срок действия сертификата, выполнив описанные выше действия.

    ```
    Error: java.lang.IllegalArgumentException: Token Refresh failed - Received invalid http response: 500, text = Response{protocol=http/1.1, code=500, message=Internal Server Error, url=http://clustername.hmssomerandomstringc.cx.internal.cloudapp.net:909/api/oauthtoken}
    ```

### <a name="resolution"></a>Решение

Создайте новый сертификат или назначьте существующий сертификат, используя следующий сценарий PowerShell:

```powershell
$clusterName = 'CLUSTERNAME'
$resourceGroupName = 'RGNAME'
$subscriptionId = 'SUBSCRIPTIONID'
$appId = 'APPLICATIONID'
$generateSelfSignedCert = $false
$addNewCertKeyCredential = $true
$certFilePath = 'NEW_CERT_PFX_LOCAL_PATH'
$certPassword = Read-Host "Enter Certificate Password"

if($generateSelfSignedCert)
{
    Write-Host "Generating new SelfSigned certificate"
    
    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=hdinsightAdlsCert" -KeySpec KeyExchange
    $certBytes = $cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $certPassword);
    $certString = [System.Convert]::ToBase64String($certBytes)
}
else
{

    Write-Host "Reading the cert file from path $certFilePath"

    $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($certFilePath, $certPassword)
    $certString = [System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes($certFilePath))
}


Login-AzureRmAccount

if($addNewCertKeyCredential)
{
    Write-Host "Creating new KeyCredential for the app"

    $keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())
    
    New-AzureRmADAppCredential -ApplicationId $appId -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore

    Write-Host "Waiting for 30 seconds for the permissions to get propagated"

    Start-Sleep -s 30
}

Select-AzureRmSubscription -SubscriptionId $subscriptionId

Write-Host "Updating the certificate on HDInsight cluster."

Invoke-AzureRmResourceAction `
    -ResourceGroupName $resourceGroupName `
    -ResourceType 'Microsoft.HDInsight/clusters' `
    -ResourceName $clusterName `
    -ApiVersion '2015-03-01-preview' `
    -Action 'updateclusteridentitycertificate' `
    -Parameters @{ ApplicationId = $appId.ToString(); Certificate = $certString; CertificatePassword = $certPassword.ToString() } `
    -Force

```

Для назначения существующего сертификата создайте сертификат, допуская его PFX-файл и пароль. Свяжите сертификат с субъектом-службой, с которым был создан кластер, с помощью идентификатора приложения.

После замены параметров фактическими значениями выполните команду PowerShell.

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]
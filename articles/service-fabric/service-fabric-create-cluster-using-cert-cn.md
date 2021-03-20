---
title: Создание кластера с использованием общего имени сертификата
description: Подробные сведения о создании кластера Service Fabric с помощью общего имени сертификата на основе шаблона.
ms.topic: conceptual
ms.date: 09/06/2019
ms.openlocfilehash: c852b40d35f936753d3c16420159676da239b6c6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86246441"
---
# <a name="deploy-a-service-fabric-cluster-that-uses-certificate-common-name-instead-of-thumbprint"></a>Развертывание кластера Service Fabric, использующего вместо отпечатка общее имя сертификата
Два сертификата не могут иметь один и тот же отпечаток. Это затрудняет смену сертификатов кластера и управление им. Тем не менее несколько сертификатов могут иметь одно общее имя или тему.  Кластер, использующий общие имена сертификата, упрощает управление сертификатами. В этой статье описывается развертывание кластера Service Fabric для использования общего имени сертификата вместо отпечатка сертификата.
 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="get-a-certificate"></a>Получение сертификата
Сначала получите сертификат в [центре сертификации (ЦС)](https://wikipedia.org/wiki/Certificate_authority).  Обычное имя сертификата должно соответствовать вашему личному домену, который вы приобрели у регистратора доменов. Например, "azureservicefabricbestpractices.com"; пользователи, которые не являются сотрудниками корпорации Майкрософт, не могут предоставлять сертификаты для доменов MS. Поэтому вы не можете использовать DNS-имена вашего LB или диспетчер трафика в качестве общих имен для сертификата. Вам необходимо подготовить зону Azure DNS к работе, если ваш личный домен разрешен в Azure. Подробнее см. в статье [Tutorial: Host your domain in Azure DNS](../dns/dns-delegate-domain-azure-dns.md) (Руководство. Размещение домена в Azure DNS). Вы захотите объявить личный домен собственным в качестве "managementEndpoint" кластера, если хотите, чтобы портал отражал псевдоним личного домена для вашего кластера.

В целях тестирования можно получить сертификат, подписанный ЦС, из бесплатного или открытого центра сертификации.

> [!NOTE]
> Самозаверяющие сертификаты, включая созданные при развертывании кластера Service Fabric на портале Azure, не поддерживаются. 

## <a name="upload-the-certificate-to-a-key-vault"></a>Отправка сертификата в хранилище ключей
В Azure кластер Service Fabric развертывается в масштабируемом наборе виртуальных машин.  Отправьте сертификат в хранилище ключей.  При развертывании кластера сертификат устанавливается на масштабируемом наборе виртуальных машин, в котором запускается кластер.

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser -Force

$SubscriptionId  =  "<subscription ID>"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $SubscriptionId

$region = "southcentralus"
$KeyVaultResourceGroupName  = "mykeyvaultgroup"
$VaultName = "mykeyvault"
$certFilename = "C:\users\sfuser\myclustercert.pfx"
$certname = "myclustercert"
$Password  = "P@ssw0rd!123"

# Create new Resource Group 
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Create the new key vault
$newKeyVault = New-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName -Location $region -EnabledForDeployment 
$resourceId = $newKeyVault.ResourceId 

# Add the certificate to the key vault.
$PasswordSec = ConvertTo-SecureString -String $Password -AsPlainText -Force
$KVSecret = Import-AzKeyVaultCertificate -VaultName $vaultName -Name $certName  -FilePath $certFilename -Password $PasswordSec

$CertificateThumbprint = $KVSecret.Thumbprint
$CertificateURL = $KVSecret.SecretId
$SourceVault = $resourceId
$CommName    = $KVSecret.Certificate.SubjectName.Name

Write-Host "CertificateThumbprint    :"  $CertificateThumbprint
Write-Host "CertificateURL           :"  $CertificateURL
Write-Host "SourceVault              :"  $SourceVault
Write-Host "Common Name              :"  $CommName    
```

## <a name="download-and-update-a-sample-template"></a>Загрузка и обновление образца шаблона
В этой статье используется пример шаблона [защищенного кластера с 5 узлами](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure) и его параметры. Скачайте на компьютер файлы *azuredeploy.json* и *azuredeploy.parameters.json*.

### <a name="update-parameters-file"></a>Обновление файла параметров
Сначала откройте файл *azuredeploy.parameters.json* в текстовом редакторе и добавьте следующее значение параметра:
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
"certificateIssuerThumbprint": {
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
},
```

Затем задайте для параметров *certificateCommonName*, *sourceVaultValue* и *certificateUrlValue* значения, возвращенные предыдущим скриптом:
```json
"certificateCommonName": {
    "value": "myclustername.southcentralus.cloudapp.azure.com"
},
"certificateIssuerThumbprint": {
    "value": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
},
"sourceVaultValue": {
  "value": "/subscriptions/<subscription>/resourceGroups/testvaultgroup/providers/Microsoft.KeyVault/vaults/testvault"
},
"certificateUrlValue": {
  "value": "https://testvault.vault.azure.net:443/secrets/testcert/5c882b7192224447bbaecd5a46962655"
},
```

### <a name="update-the-template-file"></a>Обновление файла шаблона
Затем откройте файл *azuredeploy.json* в текстовом редакторе и создайте три обновления для поддержки общего имени сертификата.

1. В разделе **параметров** добавьте параметр *certificateCommonName*:
    ```json
    "certificateCommonName": {
      "type": "string",
      "metadata": {
        "description": "Certificate Commonname"
      }
    },
    "certificateIssuerThumbprint": {
      "type": "string",
      "metadata": {
        "description": "Certificate Authority Issuer Thumpbrint for Commonname cert"
      }
    },
    ```

    Кроме того, рассмотрите возможность удаления *certificateThumbprint*, так как он больше не понадобится.

2. Задайте значение "2018-02-01" для переменной *sfrpApiVersion*:
    ```json
    "sfrpApiVersion": "2018-02-01",
    ```

3. В ресурсе **Microsoft.Compute/virtualMachineScaleSets** обновите расширение виртуальной машины, чтобы использовать общее имя в параметрах сертификата вместо отпечатка.  В **virtualMachineProfile** -> **extensionProfile** -> **Extensions** -> **Свойства** -> **Параметры** -> **сертификат**, добавить 
    ```json
       "commonNames": [
        "[parameters('certificateCommonName')]"
       ],
    ```

    и удалите `"thumbprint": "[parameters('certificateThumbprint')]",`.

    ```json
    "virtualMachineProfile": {
      "extensionProfile": {
        "extensions": [
          {
            "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
            "properties": {
              "type": "ServiceFabricNode",
              "autoUpgradeMinorVersion": true,
              "protectedSettings": {
                "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
                "StorageAccountKey2": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key2]"
              },
              "publisher": "Microsoft.Azure.ServiceFabric",
              "settings": {
                "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
                "nodeTypeRef": "[variables('vmNodeType0Name')]",
                "dataPath": "D:\\SvcFab",
                "durabilityLevel": "Bronze",
                "enableParallelJobs": true,
                "nicPrefixOverride": "[variables('subnet0Prefix')]",
                "certificate": {
                  "commonNames": [
                     "[parameters('certificateCommonName')]"
                  ],
                  "x509StoreName": "[parameters('certificateStoreValue')]"
                }
              },
              "typeHandlerVersion": "1.0"
            }
          },
    ```

4. В ресурсе **Microsoft.ServiceFabric/clusters** обновите версию API до версии "2018-02-01".  Кроме того, добавьте параметр **certificateCommonNames**, свойство **commonNames** и удалите параметр **certificate** (со свойством отпечатка), как в следующем примере:
   ```json
   {
       "apiVersion": "2018-02-01",
       "type": "Microsoft.ServiceFabric/clusters",
       "name": "[parameters('clusterName')]",
       "location": "[parameters('clusterLocation')]",
       "dependsOn": [
       "[concat('Microsoft.Storage/storageAccounts/', variables('supportLogStorageAccountName'))]"
       ],
       "properties": {
       "addonFeatures": [
           "DnsService",
           "RepairManager"
       ],        
       "certificateCommonNames": {
           "commonNames": [
           {
               "certificateCommonName": "[parameters('certificateCommonName')]",
               "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprint')]"
           }
           ],
           "x509StoreName": "[parameters('certificateStoreValue')]"
       },
       ...
   ```
   > [!NOTE]
   > Поле "certificateIssuerThumbprint" разрешает указывать ожидающих издателей сертификатов с общим именем данного субъекта. В это поле необходимо вводить перечисление отпечатков SHA1, разделенное запятыми. Обратите внимание, что это расширение проверки сертификата. В случае, если издатель не задан или является пустым, сертификат будет принят для проверки подлинности, если его цепочка будет создана, и в конечном итоге окажется в корне, которому доверяет проверяющий серверный элемент управления. Если издатель указан, сертификат будет принят, если отпечаток его прямого издателя соответствует любому из значений, указанных в этом поле, независимо от того, является ли корень доверенным или нет. Обратите внимание, что PKI может использовать разные центры сертификации для того же субъекта, поэтому важно указать все ожидаемые отпечатки издателя для данного субъекта.
   >
   > Указание издателя является лучшей методикой. Если его пропустить, сертификаты, связанные с доверенным корнем продолжат работу, но это поведение имеет ограничения и может прекратиться в ближайшем будущем. Обратите внимание, что кластеры, развернутые в Azure и защищенные сертификатами X509, которые выданные частной PKI и объявленные субъектом, могут не пройти проверку службой Azure Service Fabric (для обмена данными между кластерами), если политика сертификата PKI не является видимой и доступной. 

## <a name="deploy-the-updated-template"></a>Развертывание обновленного шаблона
Повторно разверните обновленный шаблон после внесения изменений.

```powershell
# Variables.
$groupname = "testclustergroup"
$clusterloc="southcentralus"  
$id="<subscription ID"

# Sign in to your Azure account and select your subscription
Login-AzAccount -SubscriptionId $id 

# Create a new resource group and deploy the cluster.
New-AzResourceGroup -Name $groupname -Location $clusterloc

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateParameterFile "C:\temp\cluster\AzureDeploy.Parameters.json" -TemplateFile "C:\temp\cluster\AzureDeploy.json" -Verbose
```

## <a name="next-steps"></a>Дальнейшие действия
* Сведения о [безопасности кластера](service-fabric-cluster-security.md).
* Дополнительные сведения о [выделении сертификата кластера](service-fabric-cluster-rollover-cert-cn.md).
* [Обновление сертификатов кластера и управление ими](service-fabric-cluster-security-update-certs-azure.md)
* Упростите управление сертификатами с помощью [Перехода с отпечатка на общее имя сертификата для кластера](service-fabric-cluster-change-cert-thumbprint-to-cn.md)

[image1]: .\media\service-fabric-cluster-change-cert-thumbprint-to-cn\PortalViewTemplates.png

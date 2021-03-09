---
title: Расширение виртуальной машины Azure Key Vault для Windows
description: С помощью расширения виртуальной машины вы можете развернуть агент, выполняющий автоматическое обновление секретов Key Vault на виртуальных машинах.
services: virtual-machines-windows
author: msmbaldwin
tags: keyvault
ms.service: virtual-machines-windows
ms.subservice: extensions
ms.topic: article
ms.date: 12/02/2019
ms.author: mbaldwin
ms.openlocfilehash: 42212294580b85c2b06cd98603d0f7337cee92e3
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102485782"
---
# <a name="key-vault-virtual-machine-extension-for-windows"></a>Расширение виртуальной машины Key Vault для Windows

Расширение виртуальной машины Key Vault обеспечивает автоматическое обновление секретов, хранящихся в Azure Key Vault. В частности, расширение отслеживает список наблюдаемых сертификатов, хранящихся в хранилищах ключей, и при обнаружении изменения извлекает и устанавливает соответствующие сертификаты. В этом документе подробно описаны поддерживаемые платформы, конфигурации и параметры развертывания для расширения виртуальной машины Key Vault для Windows. 

### <a name="operating-system"></a>Операционная система

Расширение Key Vault VM поддерживает следующие версии Windows:

- Windows Server 2019
- Windows Server 2016
- Windows Server 2012

Расширение Key Vault VM также поддерживается на пользовательской локальной виртуальной машине, которая передается и преобразуется в специализированный образ для использования в Azure с помощью установки Windows Server 2019 Core.

### <a name="supported-certificate-content-types"></a>Поддерживаемые типы содержимого сертификатов

- PKCS #12
- PEM

## <a name="prerequisites"></a>Предварительные требования

  - Key Vault экземпляр с сертификатом. См. раздел [создание Key Vault](../../key-vault/general/quick-create-portal.md)
  - Виртуальная машина должна иметь назначенное [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md)
  - Политика доступа Key Vault должна быть установлена с секретами `get` и `list` разрешениями на управляемое удостоверение VM/VMSS для получения части сертификата, относящейся к секрету. См. статью [Проверка подлинности в Key Vault](../../key-vault/general/authentication.md) и [назначение политики доступа Key Vault](../../key-vault/general/assign-access-policy-cli.md).
  -  Масштабируемые наборы виртуальных машин должны иметь следующие параметры идентификации:

  ``` 
  "identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "[parameters('userAssignedIdentityResourceId')]": {}
    }
  }
  ```
  
  - Расширение AKV должно иметь этот параметр:

  ```
  "authenticationSettings": {
    "msiEndpoint": "[parameters('userAssignedIdentityEndpoint')]",
    "msiClientId": "[reference(parameters('userAssignedIdentityResourceId'), variables('msiApiVersion')).clientId]"
  }
  ```

## <a name="extension-schema"></a>Схема расширения

В следующем JSON-файле показана схема для расширения виртуальной машины Key Vault. Для расширения не требуются защищенные параметры. все его параметры считаются общедоступными сведениями. Для расширения требуется список отслеживаемых сертификатов, периодичность опроса и хранилище сертификатов назначения. В частности:  

```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KVVMExtensionForWindows",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForWindows",
      "typeHandlerVersion": "1.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g: "3600">,
          "certificateStoreName": <certificate store name, e.g.: "MY">,
          "linkOnRenewal": <Only Windows. This feature ensures s-channel binding when certificate renews, without necessitating a re-deployment.  e.g.: false>,
          "certificateStoreLocation": <certificate store location, currently it works locally only e.g.: "LocalMachine">,
          "requireInitialSync": <initial synchronization of certificates e..g: true>,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net/secrets/mycertificate"
        },
        "authenticationSettings": {
                "msiEndpoint":  <Optional MSI endpoint e.g.: "http://169.254.169.254/metadata/identity">,
                "msiClientId":  <Optional MSI identity e.g.: "c7373ae5-91c2-4165-8ab6-7381d6e75619">
        }
       }
      }
    }
```

> [!NOTE]
> Наблюдаемые URL-адреса сертификатов должны иметь формат `https://myVaultName.vault.azure.net/secrets/myCertName`.
> 
> Это связано с тем, что путь `/secrets` возвращает полный сертификат, включая закрытый ключ, а путь `/certificates` — нет. Дополнительные сведения о сертификатах можно найти в статье [Сертификаты Key Vault](../../key-vault/general/about-keys-secrets-certificates.md).

> [!IMPORTANT]
> Свойство "authenticationSettings" **требуется** только для виртуальных машин с **назначенными пользователем удостоверениями**.
> В нем указывается удостоверение, используемое для проверки подлинности в Key Vault.


### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных |
| ---- | ---- | ---- |
| версия_API | 07.01.2019 | Дата |
| publisher | Microsoft.Azure.KeyVault | строка |
| type | KeyVaultForWindows | строка |
| typeHandlerVersion | 1.0 | INT |
| pollingIntervalInS | 3600 | строка |
| certificateStoreName | MY | строка |
| linkOnRenewal | false | Логическое |
| certificateStoreLocation  | LocalMachine или CurrentUser (с учетом регистра) | строка |
| рекуиреинитиалсинк | Да | Логическое |
| observedCertificates  | ["https://myvault.vault.azure.net/secrets/mycertificate","https://myvault.vault.azure.net/secrets/mycertificate2"] | массив строк
| мсиендпоинт | http://169.254.169.254/metadata/identity | строка |
| msiClientId | c7373ae5-91c2-4165-8ab6-7381d6e75619 | строка |


## <a name="template-deployment"></a>Развертывание шаблона

Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Шаблоны идеально подходят для развертывания одной или нескольких виртуальных машин, требующих обновления сертификатов, выполняемого после развертывания. Расширение можно развертывать на отдельных виртуальных машинах или в масштабируемых наборах виртуальных машин. Для обоих типов шаблонов используются общие схема и конфигурация. 

Файл конфигурации JSON для расширения виртуальной машины должен быть вложен во фрагмент ресурса виртуальной машины в шаблоне, в частности в объект `"resources": []` для шаблона виртуальной машины или в объект `"virtualMachineProfile":"extensionProfile":{"extensions" :[]`, если используется масштабируемый набор виртуальных машин.

 > [!NOTE]
> Расширение виртуальной машины требует назначения управляемого удостоверения системы или пользователя для проверки подлинности в хранилище ключей.  См [. статью Проверка подлинности в Key Vault и назначение политики доступа Key Vault.](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
> 

```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KeyVaultForWindows",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForWindows",
      "typeHandlerVersion": "1.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g: "3600">,
          "certificateStoreName": <certificate store name, e.g.: "MY">,
          "certificateStoreLocation": <certificate store location, currently it works locally only e.g.: "LocalMachine">,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: ["https://myvault.vault.azure.net/secrets/mycertificate", "https://myvault.vault.azure.net/secrets/mycertificate2"]>
        }      
      }
      }
    }
```

### <a name="extension-dependency-ordering"></a>Упорядочение зависимостей расширений
Key Vaultное расширение виртуальной машины поддерживает упорядочение расширений, если оно настроено. По умолчанию расширение сообщает о том, что оно успешно запущено, как только начнется опрос. Однако можно настроить ожидание до успешной загрузки полного списка сертификатов, прежде чем сообщать об успешном запуске. Если другие расширения зависят от установки полного набора сертификатов перед запуском, то включение этого параметра позволит этому расширению объявить зависимость от расширения Key Vault. Это предотвратит запуск этих расширений до тех пор, пока не будут установлены все сертификаты, от которых они зависят. Расширение будет повторять начальную загрузку неограниченно долго и останется в `Transitioning` состоянии.

Чтобы включить эту функцию, установите следующие значения:
```
"secretsManagementSettings": {
    "requireInitialSync": true,
    ...
}
```

> [!Note] 
> Использование этой функции несовместимо с шаблоном ARM, который создает удостоверение, назначенное системой, и обновляет политику Key Vaultного доступа с этим удостоверением. Это приведет к взаимоблокировке, так как политика доступа к хранилищу невозможно обновить до тех пор, пока не будут запущены все расширения. Вместо этого следует использовать *одно назначенное пользователем удостоверение MSI* и предварительно список ACL для хранилищ с этим удостоверением перед развертыванием.

## <a name="azure-powershell-deployment"></a>Развертывание с помощью Azure PowerShell
> [!WARNING]
> Клиенты PowerShell часто добавляют в `\` `"` settings.js, что приведет к сбою akvvm_service с ошибкой: `[CertificateManagementConfiguration] Failed to parse the configuration settings with:not an object.`

Azure PowerShell можно использовать для развертывания расширения виртуальной машины Key Vault на имеющейся виртуальной машине или в масштабируемых наборах виртуальных машин. 

* Развертывание расширения на виртуальной машине:
    
    ```powershell
        # Build settings
        $settings = '{"secretsManagementSettings": 
        { "pollingIntervalInS": "' + <pollingInterval> + 
        '", "certificateStoreName": "' + <certStoreName> + 
        '", "certificateStoreLocation": "' + <certStoreLoc> + 
        '", "observedCertificates": ["' + <observedCert1> + '","' + <observedCert2> + '"] } }'
        $extName =  "KeyVaultForWindows"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForWindows"
       
    
        # Start the deployment
        Set-AzVmExtension -TypeHandlerVersion "1.0" -ResourceGroupName <ResourceGroupName> -Location <Location> -VMName <VMName> -Name $extName -Publisher $extPublisher -Type $extType -SettingString $settings
    
    ```

* Чтобы развернуть расширение в масштабируемом наборе виртуальных машин, сделайте следующее:

    ```powershell
    
        # Build settings
        $settings = '{"secretsManagementSettings": 
        { "pollingIntervalInS": "' + <pollingInterval> + 
        '", "certificateStoreName": "' + <certStoreName> + 
        '", "certificateStoreLocation": "' + <certStoreLoc> + 
        '", "observedCertificates": ["' + <observedCert1> + '","' + <observedCert2> + '"] } }'
        $extName = "KeyVaultForWindows"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForWindows"
        
        # Add Extension to VMSS
        $vmss = Get-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName>
        Add-AzVmssExtension -VirtualMachineScaleSet $vmss  -Name $extName -Publisher $extPublisher -Type $extType -TypeHandlerVersion "1.0" -Setting $settings

        # Start the deployment
        Update-AzVmss -ResourceGroupName <ResourceGroupName> -VMScaleSetName <VmssName> -VirtualMachineScaleSet $vmss 
    
    ```

## <a name="azure-cli-deployment"></a>Развертывание с помощью Azure CLI

Для развертывания расширения виртуальной машины Key Vault на имеющейся виртуальной машине или в масштабируемом наборе виртуальных машин можно использовать Azure CLI. 
 
* Развертывание расширения на виртуальной машине:
    
    ```azurecli
       # Start the deployment
         az vm extension set --name "KeyVaultForWindows" `
         --publisher Microsoft.Azure.KeyVault `
         --resource-group "<resourcegroup>" `
         --vm-name "<vmName>" `
         --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\" <observedCert1> \", \" <observedCert2> \"] }}'
    ```

* Чтобы развернуть расширение в масштабируемом наборе виртуальных машин, сделайте следующее:

   ```azurecli
        # Start the deployment
        az vmss extension set -name "KeyVaultForWindows" `
         --publisher Microsoft.Azure.KeyVault `
         -resource-group "<resourcegroup>" `
         --vmss-name "<vmName>" `
         --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\" <observedCert1> \", \" <observedCert2> \"] }}'
    ```

Ознакомьтесь со следующими ограничениями и требованиями.
- Ограничения, касающиеся решения Key Vault:
  - оно должно существовать во время развертывания; 
  - Политика доступа Key Vault должна быть установлена для удостоверения виртуальной машины или VMSS с помощью управляемого удостоверения. См. статью [Проверка подлинности в Key Vault](../../key-vault/general/authentication.md) и [назначение политики доступа Key Vault](../../key-vault/general/assign-access-policy-cli.md).

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

* Существует ли ограничение на количество Обсерведцертификатес, которые можно настроить?
  Нет, Key Vaultное расширение виртуальной машины не ограничивает число Обсерведцертификатес.

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертываний расширения можно получить на портале Azure, а также с помощью Azure PowerShell. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните следующую команду в Azure PowerShell.

**Azure PowerShell**
```powershell
Get-AzVMExtension -VMName <vmName> -ResourceGroupname <resource group name>
```

**Azure CLI**
```azurecli
 az vm get-instance-view --resource-group <resource group name> --name  <vmName> --query "instanceView.extensions"
```

#### <a name="logs-and-configuration"></a>Журналы и конфигурация

```
%windrive%\WindowsAzure\Logs\Plugins\Microsoft.Azure.KeyVault.KeyVaultForWindows\<version>\akvvm_service_<date>.log
```

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

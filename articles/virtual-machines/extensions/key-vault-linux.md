---
title: Расширение виртуальной машины Azure Key Vault для Linux
description: С помощью расширения виртуальной машины вы можете развернуть агент, выполняющий автоматическое обновление сертификатов Key Vault на виртуальных машинах.
services: virtual-machines
author: msmbaldwin
tags: keyvault
ms.service: virtual-machines
ms.subservice: extensions
ms.collection: linux
ms.topic: article
ms.date: 12/02/2019
ms.author: mbaldwin
ms.openlocfilehash: a674f4a2a31fd217307ff373cba2b883a4d129f8
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102557069"
---
# <a name="key-vault-virtual-machine-extension-for-linux"></a>Расширение виртуальной машины Key Vault для Linux

Расширение виртуальной машины Key Vault обеспечивает автоматическое обновление секретов, хранящихся в Azure Key Vault. В частности, расширение отслеживает список наблюдаемых сертификатов, хранящихся в хранилищах ключей.  При обнаружении изменений расширение получает и устанавливает соответствующие сертификаты. Публикация и поддержка расширения виртуальной машины Key Vault осуществляется корпорацией Майкрософт, теперь и для виртуальных машин Linux. В этом документе подробно описаны поддерживаемые платформы, конфигурации и параметры развертывания для расширения виртуальной машины Key Vault для Linux. 

### <a name="operating-system"></a>Операционная система

Расширение виртуальной машины Key Vault поддерживает следующие версии Linux:

- Ubuntu-1604;
- Ubuntu-1804;
- Debian-9;
- Suse-15. 

### <a name="supported-certificate-content-types"></a>Поддерживаемые типы содержимого сертификатов

- PKCS #12
- PEM

## <a name="prerequisities"></a>Предварительные требования
  - Key Vault экземпляр с сертификатом. См. раздел [создание Key Vault](../../key-vault/general/quick-create-portal.md)
  - Виртуальная машина или VMSS должна иметь назначенное [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) .
  - Политика доступа Key Vault должна быть установлена с секретами `get` и `list` разрешениями на управляемое удостоверение VM/VMSS для получения части сертификата, относящейся к секрету. См. статью [Проверка подлинности в Key Vault](../../key-vault/general/authentication.md) и [назначение политики доступа Key Vault](../../key-vault/general/assign-access-policy-cli.md).
  -  VMSS должен иметь следующий параметр Identity: ` 
  "identity": {
  "type": "UserAssigned",
  "userAssignedIdentities": {
  "[parameters('userAssignedIdentityResourceId')]": {}
  }
  }
  `
  
 - Расширение AKV должно иметь этот параметр: `
                 "authenticationSettings": {
                    "msiEndpoint": "[parameters('userAssignedIdentityEndpoint')]",
                    "msiClientId": "[reference(parameters('userAssignedIdentityResourceId'), variables('msiApiVersion')).clientId]"
                  }
   `

## <a name="extension-schema"></a>Схема расширения

В следующем JSON-файле показана схема для расширения виртуальной машины Key Vault. Расширению не требуются защищенные параметры, так как все его параметры считаются информационными и не влияют на безопасность. Для расширения необходим список отслеживаемых секретов, указание частоты опроса и конечное хранилище сертификатов. В частности:  
```json
    {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "KVVMExtensionForLinux",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "1.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
        "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <It is ignored on Linux>,
          "linkOnRenewal": <Not available on Linux e.g.: false>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "requireInitialSync": <initial synchronization of certificates e..g: true>,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: ["https://myvault.vault.azure.net/secrets/mycertificate", "https://myvault.vault.azure.net/secrets/mycertificate2"]>
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
| type | KeyVaultForLinux | строка |
| typeHandlerVersion | 1.0 | INT |
| pollingIntervalInS | 3600 | строка |
| certificateStoreName | Он игнорируется в Linux | строка |
| linkOnRenewal | false | Логическое |
| certificateStoreLocation  | /вар/либ/ваажент/микрософт.азуре.кэйваулт | строка |
| рекуиреинитиалсинк | Да | Логическое |
| observedCertificates  | ["https://myvault.vault.azure.net/secrets/mycertificate", "https://myvault.vault.azure.net/secrets/mycertificate2"] | массив строк
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
      "name": "KeyVaultForLinux",
      "apiVersion": "2019-07-01",
      "location": "<location>",
      "dependsOn": [
          "[concat('Microsoft.Compute/virtualMachines/', <vmName>)]"
      ],
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "typeHandlerVersion": "1.0",
      "autoUpgradeMinorVersion": true,
      "settings": {
          "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <ingnored on linux>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net/secrets/mycertificate"
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
> Метим Использование этой функции несовместимо с шаблоном ARM, который создает удостоверение, назначенное системой, и обновляет политику Key Vaultного доступа с этим удостоверением. Это приведет к взаимоблокировке, так как политика доступа к хранилищу невозможно обновить до тех пор, пока не будут запущены все расширения. Вместо этого следует использовать *одно назначенное пользователем удостоверение MSI* и предварительно список ACL для хранилищ с этим удостоверением перед развертыванием.

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
        $extName =  "KeyVaultForLinux"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForLinux"
       
    
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
        $extName = "KeyVaultForLinux"
        $extPublisher = "Microsoft.Azure.KeyVault"
        $extType = "KeyVaultForLinux"
        
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
         az vm extension set -n "KeyVaultForLinux" `
         --publisher Microsoft.Azure.KeyVault `
         -g "<resourcegroup>" `
         --vm-name "<vmName>" `
         --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\" <observedCert1> \", \" <observedCert2> \"] }}'
    ```

* Чтобы развернуть расширение в масштабируемом наборе виртуальных машин, сделайте следующее:

   ```azurecli
        # Start the deployment
        az vmss extension set -n "KeyVaultForLinux" `
        --publisher Microsoft.Azure.KeyVault `
        -g "<resourcegroup>" `
        --vmss-name "<vmssName>" `
        --settings '{\"secretsManagementSettings\": { \"pollingIntervalInS\": \"<pollingInterval>\", \"certificateStoreName\": \"<certStoreName>\", \"certificateStoreLocation\": \"<certStoreLoc>\", \"observedCertificates\": [\" <observedCert1> \", \" <observedCert2> \"] }}'
    ```
Ознакомьтесь со следующими ограничениями и требованиями.
- Ограничения, касающиеся решения Key Vault:
  - оно должно существовать во время развертывания; 
  - Политика доступа Key Vault должна быть установлена для удостоверения виртуальной машины или VMSS с помощью управляемого удостоверения. См. статью [Проверка подлинности в Key Vault](../../key-vault/general/authentication.md) и [назначение политики доступа Key Vault](../../key-vault/general/assign-access-policy-cli.md).

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
/var/log/waagent.log
/var/log/azure/Microsoft.Azure.KeyVault.KeyVaultForLinux/*
/var/lib/waagent/Microsoft.Azure.KeyVault.KeyVaultForLinux-<most recent version>/config/*
```
### <a name="using-symlink"></a>Использование символьную ссылку

Символьные ссылки или символических ссылок по сути являются расширенными сочетаниями клавиш. Чтобы избежать наблюдения за папкой и автоматического получения последнего сертификата, вы можете использовать эту символьную ссылку `([VaultName].[CertificateName])` для получения последней версии сертификата в Linux.

### <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

* Существует ли ограничение на количество Обсерведцертификатес, которые можно настроить?
  Нет, Key Vaultное расширение виртуальной машины не ограничивает число Обсерведцертификатес.

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

---
title: Необходимые условия для развертывания облачных служб Azure (Расширенная поддержка)
description: Необходимые условия для развертывания облачных служб Azure (Расширенная поддержка)
ms.topic: article
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: 79d6fecddf060909a74664ff29e08301f45d7042
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103472303"
---
# <a name="prerequisites-for-deploying-azure-cloud-services-extended-support"></a>Необходимые условия для развертывания облачных служб Azure (Расширенная поддержка)

> [!IMPORTANT]
> Облачные службы (расширенная поддержка) сейчас предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Чтобы убедиться в успешном развертывании облачных служб (Расширенная поддержка), выполните описанные ниже действия и заполните каждый элемент до попыток развертывания. 

## <a name="register-the-cloudservices-feature"></a>Регистрация функции Клаудсервицес
Зарегистрируйте функцию для своей подписки. Для завершения регистрации может потребоваться несколько минут. 

```powershell
Register-AzProviderFeature -FeatureName CloudServices -ProviderNamespace Microsoft.Compute
```

Проверьте состояние регистрации, выполнив следующие действия.  
```powershell
Get-AzProviderFeature 

#Sample output
FeatureName               ProviderName      RegistrationState
CloudServices           Microsoft.Compute    Registered
```

## <a name="required-service-configuration-cscfg-file-updates"></a>Обновления файлов требуемой конфигурации службы (. cscfg)

### <a name="1-virtual-network"></a>1) виртуальная сеть;
Все развертывания облачной службы (расширенная поддержка) обязательно размещаются в виртуальной сети. Виртуальную сеть можно создать с помощью шаблона [портал Azure](../virtual-network/quick-create-portal.md), [PowerShell](../virtual-network/quick-create-powershell.md), [Azure CLI](../virtual-network/quick-create-cli.md) или [ARM](../virtual-network/quick-create-template.md). На виртуальную сеть и подсети также необходимо ссылаться в конфигурации службы (cscfg) в разделе [NetworkConfiguration](schema-cscfg-networkconfiguration.md) . 

Для виртуальных сетей, принадлежащих к той же группе ресурсов, что и облачная служба, достаточно ссылаться только на имя виртуальной сети в файле конфигурации службы (cscfg). Если виртуальная сеть и облачная служба находятся в двух разных группах ресурсов, то полный идентификатор Azure Resource Manager виртуальной сети необходимо указать в файле конфигурации службы (cscfg).
 
#### <a name="virtual-network-located-in-same-resource-group"></a>Виртуальная сеть, расположенная в той же группе ресурсов
```xml
<VirtualNetworkSite name="<vnet-name>"/> 
  <AddressAssignments> 
    <InstanceAddress roleName="<role-name>"> 
     <Subnets> 
       <Subnet name="<subnet-name>"/> 
     </Subnets> 
    </InstanceAddress> 
```

#### <a name="virtual-network-located-in-different-resource-group"></a>Виртуальная сеть, расположенная в другой группе ресурсов
```xml
<VirtualNetworkSite name="/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.Network/virtualNetworks/<vnet-name>"/> 
   <AddressAssignments> 
     <InstanceAddress roleName="<role-name>"> 
       <Subnets> 
        <Subnet name="<subnet-name>"/> 
       </Subnets> 
     </InstanceAddress> 
```
### <a name="2-remove-the-old-plugins"></a>2) удалите старые подключаемые модули.

Удалите старые параметры удаленного рабочего стола из файла конфигурации службы (cscfg).  

```xml
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.Enabled" value="true" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountUsername" value="gachandw" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountEncryptedPassword" value="XXXX" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteAccess.AccountExpiration" value="2021-12-17T23:59:59.0000000+05:30" /> 
<Setting name="Microsoft.WindowsAzure.Plugins.RemoteForwarder.Enabled" value="true" /> 
```
Удалите старые параметры диагностики для каждой роли в файле конфигурации службы (CSCFG-файл).

```xml
<Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
```

## <a name="required-service-definition-file-csdef-updates"></a>Обновления требуемого файла определения службы (CSDEF)

### <a name="1-virtual-machine-sizes"></a>1) размеры виртуальных машин
Следующие размеры не рекомендуются в Azure Resource Manager. Однако если вы хотите продолжить их использование, обновите `vmsize` имя с помощью связанного Azure Resource Manager соглашения об именовании.  

| Имя предыдущего размера | Имя измененного размера | 
|---|---|
| Очень малый | Standard_A0 | 
| Small | Standard_A1 |
| Средний | Standard_A2 | 
| Большой | Standard_A3 | 
| Очень большой | Standard_A4 | 
| A5 | Standard_A5 | 
| A6 | Standard_A6 | 
| A7 | Standard_A7 |  
| A8 | Standard_A8 | 
| A9 | Standard_A9 |
| A10 | Standard_A10 | 
| A11 | Standard_A11 | 
| MSODSG5 | Standard_MSODSG5 | 

 Например, для `<WorkerRole name="WorkerRole1" vmsize="Medium"` это `<WorkerRole name="WorkerRole1" vmsize="Standard_A2"`.
 
> [!NOTE]
> Чтобы получить список доступных размеров, ознакомьтесь со [списком SKU ресурсов](/rest/api/compute/resourceskus/list) и примените следующие фильтры: <br>
`ResourceType = virtualMachines ` <br>
`VMDeploymentTypes = PaaS `


### <a name="2-remove-old-remote-desktop-plugins"></a>2) удалите старые подключаемые модули удаленных рабочих столов
Для развертываний, использующих старые подключаемые модули удаленного рабочего стола, необходимо, чтобы модули были удалены из файла определения службы (CSDEF) и всех связанных с ним сертификатов. 

```xml
<Imports> 
<Import moduleName="RemoteAccess" /> 
<Import moduleName="RemoteForwarder" /> 
</Imports> 
```
Для развертываний, использующих старые подключаемые модули диагностики, необходимо удалить параметры для каждой роли из файла определения службы (. CSDEF).

```xml
<Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
```

## <a name="key-vault-creation"></a>Создание Key Vault 

Хранилище ключей используется для хранения сертификатов, связанных с Облачными службами (расширенная поддержка). Добавьте сертификаты в Key Vault, а затем сослаться на отпечатки сертификата в файле конфигурации службы. Также для хранилища ключей необходимо включить соответствующие разрешения, позволяющие ресурсу Облачных служб (расширенная поддержка) получить из хранилища ключей сертификат, хранимый в виде секрета. Хранилище ключей можно создать в [портал Azure](../key-vault/general/quick-create-portal.md) или с помощью [PowerShell](../key-vault/general/quick-create-powershell.md). Хранилище ключей должно быть создано в том же регионе и подписке, что и облачная служба. Дополнительные сведения см. в статье [ Использование сертификатов с Облачными службами Azure (расширенная поддержка) ](certificates-and-key-vault.md).

## <a name="next-steps"></a>Следующие шаги 
- Ознакомьтесь с [предварительными требованиями для развертывания](deploy-prerequisite.md) облачных служб (Расширенная поддержка).
- Разверните Облачную службу (расширенная поддержка) с помощью [портала Azure](deploy-portal.md), [PowerShell](deploy-powershell.md), [шаблона](deploy-template.md) или [Visual Studio](deploy-visual-studio.md).
- Ознакомьтесь с [часто задаваемыми вопросами об Облачных службах (расширенная поддержка)](faq.md).
- Перейдите в [репозиторий примеров для Облачных служб (расширенная поддержка) ](https://github.com/Azure-Samples/cloud-services-extended-support)

---
title: Включаемый файл для PowerShell для Azure DNS
description: Включаемый файл для PowerShell для Azure DNS
services: dns
author: subsarma
ms.service: dns
ms.topic: include file for PowerShell for Azure DNS
ms.date: 03/21/2018
ms.author: subsarma
ms.custom: include file for PowerShell for Azure DNS
ms.openlocfilehash: 32c516ccee3a9f4f7604a3e330285703a776b47d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67133194"
---
## <a name="set-up-azure-powershell-for-azure-dns"></a>Настройка Azure PowerShell для Azure DNS

### <a name="before-you-begin"></a>Перед началом

[!INCLUDE [requires-azurerm](requires-azurerm.md)]

Перед началом настройки убедитесь, что у вас есть следующие компоненты.

* Подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/).
* Вам потребуется установить последнюю версию командлетов PowerShell Azure Resource Manager. Подробнее: [Установка и настройка Azure PowerShell](/powershell/azureps-cmdlets-docs).

Кроме того, для использования Частных зон (предварительная версия) необходимо убедиться, что у вас есть следующие модули и версии PowerShell. 
* AzureRM.Dns [версии 4.1.0 ](https://www.powershellgallery.com/packages/AzureRM.Dns/4.1.0) или более поздней
* AzureRM.Network [версии 5.4.0 ](https://www.powershellgallery.com/packages/AzureRM.Network/5.4.0) или более поздней

```powershell 
Find-Module -Name AzureRM.Dns 
``` 
 
```powershell 
Find-Module -Name AzureRM.Network 
``` 
 
Выходные данные описанных выше команд должны показать версию 4.1.0 или более позднюю для AzureRM.Dns и версию 5.4.0 или более позднюю для AzureRM.Network.  

Если в системе установлены более ранние версии, вы можно установить последнюю версию Azure PowerShell или скачать и установить описанные выше модули из коллекции PowerShell, перейдя по ссылке рядом с версией модуля. Затем их можно установить с помощью следующих команд. Оба модуля являются обязательными. Они характеризуются полной обратной совместимостью. 

```powershell
Install-Module -Name AzureRM.Dns -Force
```

```powershell
Install-Module -Name AzureRM.Network -Force
```

### <a name="sign-in-to-your-azure-account"></a>Вход в учетную запись Azure

Откройте консоль PowerShell и подключитесь к своей учетной записи. Дополнительные сведения см. [в разделе Вход с помощью AzureRM](/powershell/azure/azurerm/authenticate-azureps).

```powershell
Connect-AzureRmAccount
```

### <a name="select-the-subscription"></a>Выбор подписки
 
Просмотрите подписки учетной записи.

```powershell
Get-AzureRmSubscription
```

Выберите, какие подписки Azure будут использоваться.

```powershell
Select-AzureRmSubscription -SubscriptionName "your_subscription_name"
```

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Диспетчер ресурсов Azure требует, чтобы все группы ресурсов указывали расположение. Это расположение используется в качестве расположения по умолчанию для всех ресурсов данной группы. Но так как все ресурсы DNS глобальные, а не региональные, выбор расположения группы ресурсов не влияет на Azure DNS.

Если используется существующая группа ресурсов, можно пропустить этот шаг.

```powershell
New-AzureRmResourceGroup -Name MyAzureResourceGroup -location "West US"
```

### <a name="register-resource-provider"></a>Регистрация поставщика ресурсов

Служба Azure DNS управляется поставщиком ресурсов Microsoft.Network. Вашу подписку Azure необходимо зарегистрировать, чтобы использовать этот поставщик ресурсов, прежде чем работать с Azure DNS. Эта операция выполняется один раз для каждой подписки.

```powershell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

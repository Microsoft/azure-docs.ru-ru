---
ms.openlocfilehash: cc628b1f1fcae5e837f7f61db584c8747100f353
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105644728"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/dotnet/) бесплатно.
- Установка модуля [Azure Az PowerShell](https://docs.microsoft.com/powershell/azure/)

## <a name="create-azure-communication-resource"></a>Создание ресурса Служб коммуникации Azure

Чтобы создать ресурс Служб коммуникации Azure, [войдите в Azure CLI](/cli/azure/authenticate-azure-cli). Это можно сделать с помощью терминала, используя команду ```Connect-AzAccount``` и указав учетные данные. Выполните следующую команду для создания рабочей области.

```PowerShell
PS C:\> New-AzCommunicationService -ResourceGroupName ContosoResourceProvider1 -Name ContosoAcsResource1 -DataLocation UnitedStates -Location Global
```

Если вы хотите выбрать конкретную подписку, можно также указать флаг ```--subscription``` и указать идентификатор подписки.
```PowerShell
PS C:\> New-AzCommunicationService -ResourceGroupName ContosoResourceProvider1 -Name ContosoAcsResource1 -DataLocation UnitedStates -Location Global -SubscriptionId SubscriptionID
```

Теперь можно настроить ресурс Служб коммуникации с помощью следующих параметров:

* Группа ресурсов
* Имя ресурса Служб коммуникации
* Географический регион, с которым будет связан ресурс

На следующем шаге можно назначить ресурсу теги. Теги можно использовать для упорядочения ресурсов в Azure. Дополнительные сведения о тегах см. в [документации по маркировке ресурсов](../../../azure-resource-manager/management/tag-resources.md).

## <a name="manage-your-communication-services-resource"></a>Управление ресурсом Служб коммуникации

Чтобы добавить теги в ресурс Служб коммуникации, выполните следующие команды: Вы также можете ориентироваться на конкретную подписку.

```PowerShell
PS C:\> Update-AzCommunicationService -Name ContosoAcsResource1 -ResourceGroupName ContosoResourceProvider1 -Tag @{ExampleKey1="ExampleValue1"}

PS C:\> Update-AzCommunicationService -Name ContosoAcsResource1 -ResourceGroupName ContosoResourceProvider1 -Tag @{ExampleKey1="ExampleValue1"} -SubscriptionId SubscriptionID
```

Чтобы получить список всех ресурсов Служб коммуникации Azure в заданной подписке, используйте следующую команду:

```PowerShell
PS C:\> Get-AzCommunicationService -SubscriptionId SubscriptionID
```

Чтобы получить список всех сведений об определенном ресурсе, используйте следующую команду:

```PowerShell
PS C:\> Get-AzCommunicationService -Name ContosoAcsResource1 -ResourceGroupName ContosoResourceProvider1
```

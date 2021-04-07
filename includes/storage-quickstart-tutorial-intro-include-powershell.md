---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: 1f8f8d314a8bb37a08b3696f597b395a8a4beb8e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "67185511"
---
## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

Если вы не знаете, какое расположение нужно использовать, можно получить список доступных расположений. Получите список расположений, используя приведенный ниже пример кода, и найдите нужное расположение. В этом примере используется **eastus**. Сохраните расположение в переменной и используйте ее. Так вы сможете изменить расположение в одном месте.

```powershell
Get-AzLocation | select Location
$location = "eastus"
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими.

```powershell
$resourceGroup = "myResourceGroup"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="create-a-storage-account"></a>Создание учетной записи хранения

Создайте стандартную учетную запись хранения общего назначения с репликацией LRS с помощью командлета [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount). Затем получите контекст учетной записи хранения, определяющий необходимую учетную запись хранения. Действуя в учетной записи хранения, ссылайтесь на контекст, вместо того чтобы многократно передавать учетные данные. Используйте следующий пример, чтобы создать учетную запись хранения *mystorageaccount* с включенными по умолчанию локально избыточным хранилищем (LRS) и шифрованием больших двоичных объектов.

```powershell
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name "mystorageaccount" `
  -SkuName Standard_LRS `
  -Location $location `

$ctx = $storageAccount.Context
```

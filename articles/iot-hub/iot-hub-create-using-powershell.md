---
title: Создание Центра Интернета вещей Azure с помощью командлета PowerShell | Документация Майкрософт
description: Узнайте, как с помощью командлетов PowerShell создать группу ресурсов, а затем создать центр Интернета вещей в группе ресурсов. Также Узнайте, как удалить концентратор.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/29/2018
ms.author: robinsh
ms.openlocfilehash: da021e3ba0fd93a182ea76a1ba4b7042b325aacc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92142369"
---
# <a name="create-an-iot-hub-using-the-new-aziothub-cmdlet"></a>Создание центра Интернета вещей с помощью командлета New-AzIotHub

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

## <a name="introduction"></a>Введение

Для создания Центров Интернета вещей и управления ими можно использовать командлеты Azure PowerShell. В этом руководстве показано, как создать Центр Интернета вещей с помощью PowerShell.

Для работы с этим руководством вам потребуется подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="connect-to-your-azure-subscription"></a>Подключение к подписке Azure

Если вы используете Cloud Shell, вы уже вошли в свою подписку. Если вместо этого вы используете PowerShell в локальной среде, введите следующую команду, чтобы войти в подписку Azure.

```powershell
# Log into Azure account.
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Для развертывания Центра Интернета вещей необходима группа ресурсов. Вы можете выбрать существующую группу ресурсов или создать новую.

Чтобы создать группу ресурсов для центра Интернета вещей, используйте команду [New-азресаурцеграуп](/powershell/module/az.Resources/New-azResourceGroup) . В этом примере создается группа ресурсов с именем **MyIoTRG1**, размещенная в регионе **Восточная часть США**:

```azurepowershell-interactive
New-AzResourceGroup -Name MyIoTRG1 -Location "East US"
```

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

Чтобы создать центр Интернета вещей в группе ресурсов, созданной на предыдущем шаге, используйте команду [New-азиосуб](/powershell/module/az.IotHub/New-azIotHub) . В этом примере создается центр категории **S1** с именем **MyTestIoTHub**, размещенный в регионе **Восточная часть США**:

```azurepowershell-interactive
New-AzIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub `
    -SkuName S1 -Units 1 `
    -Location "East US"
```

Имя Центра Интернета вещей должно быть глобально уникальным.

[!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

Список всех центров Интернета вещей в подписке можно получить с помощью команды [Get-азиосуб](/powershell/module/az.IotHub/Get-azIotHub) :

```azurepowershell-interactive
Get-AzIotHub
```

Это пример стандартного Центра Интернета вещей S1, который вы создали на предыдущем шаге.

Вы можете удалить центр Интернета вещей с помощью команды [Remove-азиосуб](/powershell/module/az.iothub/remove-aziothub) :

```azurepowershell-interactive
Remove-AzIotHub `
    -ResourceGroupName MyIoTRG1 `
    -Name MyTestIoTHub
```

Кроме того, можно удалить группу ресурсов и все содержащиеся в ней ресурсы с помощью команды [Remove-азресаурцеграуп](/powershell/module/az.Resources/Remove-azResourceGroup) :

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyIoTRG1
```

## <a name="next-steps"></a>Дальнейшие действия

После развертывания Центра Интернета вещей с помощью командлета PowerShell вам могут понадобиться дополнительные сведения, с которыми можно ознакомиться в следующих статьях:

* [AzureRM.IotHub](/powershell/module/az.iothub/).

* [REST API поставщика ресурсов центра Интернета вещей](/rest/api/iothub/iothubresource).

Дополнительные сведения о разработке для Центра Интернета вещей см. в следующих статьях:

* [Пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md)

* [Пакеты SDK для Центра Интернета вещей Azure](iot-hub-devguide-sdks.md)

Для дальнейшего изучения возможностей Центра Интернета вещей см. следующие статьи:

* [Краткое руководство. Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md)
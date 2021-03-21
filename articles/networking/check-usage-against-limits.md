---
title: Проверка использования ресурсов Azure относительно установленных ограничений | Документация Майкрософт
description: Узнайте, как проверить текущие данные об использовании ресурсов Azure в соответствии с ограничениями подписки.
services: networking
documentationcenter: na
author: KumudD
ms.author: kumud
tags: azure-resource-manager
ms.service: azure
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/05/2018
ms.openlocfilehash: 31eeb31fb78a4e9552e64121e0e85b5fd8d9b773
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102210654"
---
# <a name="check-resource-usage-against-limits"></a>Проверка использования ресурсов в соответствии с ограничениями

В этой статье описывается, как определить количество каждого типа сетевых ресурсов, развернутых в подписке, и каковы [ограничения подписки](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fnetworking%2ftoc.json#networking-limits). Благодаря возможности просмотра потребления ресурсов в отношении ограничений можно отслеживать текущее использование и планировать будущее использование. Для этого можно использовать [портал Azure](#azure-portal), [PowerShell](#powershell) или [Azure CLI](#azure-cli).

## <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал](https://portal.azure.com)Azure.
2. В верхнем левом углу окна портала Azure выберите **Все службы**.
3. В поле **Фильтр** введите *Подписки*. Когда пункт **Подписки** появится в результатах поиска, выберите его.
4. Выберите имя подписки, сведения об использовании которой вы хотите просмотреть.
5. В разделе **Параметры** выберите **Usage + quota** (Использование + квота).
6. Выберите следующие параметры.
   - **Типы ресурсов**. Вы можете выбрать все типы ресурсов или определенные типы, которые вы хотите просмотреть.
   - **Поставщики**. Вы можете выбрать всех поставщиков ресурсов или выбрать **Вычисление**, **Сети** или **Хранение**.
   - **Расположения**. Можно выбрать все расположения Azure или конкретные.
   - Вы можете выбрать отображение всех ресурсов или только ресурсов по крайней мере с одним развернутым экземпляром.

     На следующем рисунке показаны все сетевые ресурсы по крайней мере с одним экземпляром, развернутым в восточной части США.

       ![Просмотр данных об использовании](./media/check-usage-against-limits/view-usage.png)

     Вы можете отсортировать столбцы, выбрав заголовок. Отображаются ограничения для вашей подписки. Если вам нужно увеличить ограничение по умолчанию, выберите **Запросить увеличение**, затем заполните и отправьте запрос в службу поддержки. Все ресурсы имеют максимальное ограничение, указанное в [разделе об ограничениях Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fnetworking%2ftoc.json#networking-limits). Если вы уже используете максимальный предел, ограничение не может быть увеличено.

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Вы можете выполнить приведенные ниже команды в [Azure Cloud Shell](https://shell.azure.com/powershell) или с помощью PowerShell на своем компьютере. Azure Cloud Shell — это бесплатная интерактивная оболочка. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. При запуске PowerShell с компьютера необходим модуль Azure PowerShell версии 1.0.0 или более поздней. Выполните `Get-Module -ListAvailable Az` на компьютере, чтобы получить сведения об установленной версии. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Login-AzAccount`, чтобы войти в Azure.

Просмотрите сведения об использовании с ограничениями с помощью [Get-азнетворкусаже](/powershell/module/az.network/get-aznetworkusage). В следующем примере показано использование ресурсов по крайней мере с одним экземпляром, развернутым в восточной части США:

```azurepowershell-interactive
Get-AzNetworkUsage `
  -Location eastus `
  | Where-Object {$_.CurrentValue -gt 0} `
  | Format-Table ResourceType, CurrentValue, Limit
```

Вы получите форматированные выходные данные, аналогичные приведенным ниже.

```output
ResourceType            CurrentValue Limit
------------            ------------ -----
Virtual Networks                   1    50
Network Security Groups            2   100
Public IP Addresses                1    60
Network Interfaces                 1 24000
Network Watchers                   1     1
```

## <a name="azure-cli"></a>Azure CLI

При использовании команд интерфейса командной строки Azure (CLI) для работы с этой статьей выполняйте их в [Azure Cloud Shell](https://shell.azure.com/bash) или в интерфейсе командной строки на своем компьютере. Для этой статьи требуется Azure CLI 2.0.32 или более поздней версии. Выполните командлет `az --version`, чтобы узнать установленную версию. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI](/cli/azure/install-azure-cli). Если Azure CLI запущена локально, необходимо также выполнить командлет `az login`, чтобы войти в Azure.

Просмотрите использование ваших ресурсов относительно установленных ограничений с помощью команды [az network list-usages](/cli/azure/network#az-network-list-usages). В следующем примере показано использование ресурсов в регионе "Восточная часть США":

```azurecli-interactive
az network list-usages \
  --location eastus \
  --out table
```

Вы получите форматированные выходные данные, аналогичные приведенным ниже.

```output
Name                    CurrentValue Limit
------------            ------------ -----
Virtual Networks                   1    50
Network Security Groups            2   100
Public IP Addresses                1    60
Network Interfaces                 1 24000
Network Watchers                   1     1
Load Balancers                     0   100
Application Gateways               0    50
```
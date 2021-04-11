---
title: Обновление службы в пуле узлов Виртуального рабочего стола Windows (классического) в Azure
description: Узнайте, как в службе "Виртуальный рабочий стол Windows (классический)" создать пул узлов проверки для мониторинга обновлений службы перед их развертыванием в рабочую среду.
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
manager: femila
ms.openlocfilehash: ad79ad31678f698c0f034b39bab1e3a19a48d3f2
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106444978"
---
# <a name="tutorial-create-a-host-pool-to-validate-service-updates-in-windows-virtual-desktop-classic"></a>Руководство по Создание пула узлов для проверки обновлений службы в Виртуальном рабочем столе Windows (классическом)

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows (классическому), который не поддерживает объекты Azure Resource Manager для Виртуального рабочего стола Windows. Сведения об обеспечении управления объектами Azure Resource Manager для Виртуального рабочего стола Windows см. в [этой статье](../create-validation-host-pool.md).

Пулы узлов — это коллекция, состоящая из одной или нескольких идентичных виртуальных машин в средах клиента Виртуального рабочего стола Windows. Мы рекомендуем создать пул узлов проверки, в котором сначала применяются обновления службы. Это позволяет отслеживать обновления службы до того, как служба применяет их к стандартной или не поддерживающей проверку среде. Без пула узлов проверки вы не сможете обнаружить изменения, которые могут привести к ошибкам и, следовательно, к простоям для пользователей в рабочей среде.

Чтобы обеспечить совместимость приложения с последними обновлениями, пул узлов проверки должен быть максимально похож на пулы узлов в неподдерживающей проверку среде. Пользователи должны подключаться к пулу узлов проверки так же часто, как и к стандартному пулу узлов. Если в пуле узлов есть автоматическое тестирование, необходимо включить его в пул узлов проверки.

Вы можете выполнять отладку ошибок в пуле узлов проверки с помощью [функции диагностики](diagnostics-role-service-2019.md) или сведений из [статей по устранению неполадок Виртуального рабочего стола Windows](troubleshoot-set-up-overview-2019.md).

>[!NOTE]
> Мы рекомендуем не менять расположение пула узлов проверки, чтобы протестировать все будущие обновления.

Перед началом работы при необходимости [скачайте и импортируйте модуль PowerShell для Виртуального рабочего стола Windows](/powershell/windows-virtual-desktop/overview/). После этого выполните следующий командлет, чтобы войти в учетную запись:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="create-your-host-pool"></a>Создание пула узлов

Пул узлов можно создать, следуя инструкциям в любой из этих статей:
- [Руководство. Создание пула узлов с помощью Azure Marketplace](create-host-pools-azure-marketplace-2019.md)
- [Create a host pool with an Azure Resource Manager template](create-host-pools-arm-template.md) (Создание пула узлов с помощью шаблона Azure Resource Manager)
- [Create a host pool with PowerShell](create-host-pools-powershell-2019.md) (Создание пула узлов с помощью PowerShell)

## <a name="define-your-host-pool-as-a-validation-host-pool"></a>Определение пула узлов в качестве пула узлов проверки

Выполните следующие командлеты PowerShell, чтобы определить новый пул узлов в качестве пула узлов проверки. Замените значения в кавычках значениями, относящимися к вашему сеансу:

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
Set-RdsHostPool -TenantName $myTenantName -Name "contosoHostPool" -ValidationEnv $true
```

Выполните следующий командлет PowerShell, чтобы убедиться, что свойство проверки установлено. Замените значения в кавычках значениями, относящимися к вашему сеансу.

```powershell
Get-RdsHostPool -TenantName $myTenantName -Name "contosoHostPool"
```

Результаты из командлета должны выглядеть подобным образом:

```
    TenantName          : contoso
    TenantGroupName     : Default Tenant Group
    HostPoolName        : contosoHostPool
    FriendlyName        :
    Description         :
    Persistent          : False
    CustomRdpProperty    : use multimon:i:0;
    MaxSessionLimit     : 10
    LoadBalancerType    : BreadthFirst
    ValidationEnv       : True
    Ring                :
```

## <a name="update-schedule"></a>Обновление расписания

Обновления службы выполняются ежемесячно. Если есть серьезные проблемы, критические обновления будут предоставляться чаще.

## <a name="next-steps"></a>Дальнейшие действия

Создав пул узлов проверки, вы можете узнать, как использовать решение "Работоспособность служб Azure" для мониторинга развертывания Виртуального рабочего стола Windows.

> [!div class="nextstepaction"]
> [Настройка оповещений служб](set-up-service-alerts-2019.md)

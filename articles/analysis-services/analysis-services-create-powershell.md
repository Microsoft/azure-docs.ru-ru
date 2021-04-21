---
title: Краткое руководство. Создание сервера Azure Analysis Services с помощью PowerShell | Документация Майкрософт
description: В этом кратком руководстве показано, как создать сервер Azure Analysis Services с помощью PowerShell.
author: minewiskan
ms.author: owend
ms.reviewer: minewiskan
ms.date: 08/31/2020
ms.topic: quickstart
ms.service: azure-analysis-services
ms.custom: devx-track-azurepowershell - references_regions - devx-track-azurepowershell - mode-api
ms.openlocfilehash: f1902a7192fc2153725e2b1d1e76f23cbd1426f3
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107769374"
---
# <a name="quickstart-create-a-server---powershell"></a>Краткое руководство по созданию сервера с помощью PowerShell

В этом кратком руководстве объясняется, как с помощью PowerShell и командной строки создать сервер Azure Analysis Services в подписке Azure.

## <a name="prerequisites"></a>предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Подписка Azure**: откройте ссылку на [бесплатную пробную версию Azure](https://azure.microsoft.com/offers/ms-azr-0044p/), чтобы создать учетную запись.
- **Azure Active Directory**: ваша подписка должна быть связана с клиентом Azure Active Directory, а учетная запись настроена в этом каталоге. Дополнительные сведения см. в руководстве по [аутентификации и настройке пользовательских разрешений](analysis-services-manage-users.md).
- **Azure PowerShell**. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Чтобы выполнить установку или обновление, см. статью [Установка и настройка Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="import-azanalysisservices-module"></a>Импорт модуля Az.AnalysisServices

Чтобы создать сервер в подписке, используйте модуль компонентов [Az.AnalysisServices](/powershell/module/az.analysisservices). Загрузите модуль Az.AnalysisServices в сеанс PowerShell.

```powershell
Import-Module Az.AnalysisServices
```

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью команды [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Выполните инструкции на экране.

```powershell
Connect-AzAccount
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[Группа ресурсов Azure](../azure-resource-manager/management/overview.md) — это логический контейнер, в котором ресурсы Azure развертываются и администрируются как группа. При создании сервера необходимо указать группу ресурсов в вашей подписке. Если у вас еще нет группы ресурсов, вы можете создать ее с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов с именем `myResourceGroup` и в регионе "Западная часть США".

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "WestUS"
```

## <a name="create-a-server"></a>Создание сервера

Создайте сервер с помощью команды [New-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver). В следующем примере создается сервер с именем MyServer в группе myResourceGroup, в регионе "Западная часть США", на уровне D1 и указывается philipc@adventureworks.com в качестве администратора сервера.

```powershell
New-AzAnalysisServicesServer -ResourceGroupName "myResourceGroup" -Name "myserver" -Location WestUS -Sku D1 -Administrator "philipc@adventure-works.com"
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить сервер из подписки, используйте команду [Remove-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver). Не удаляйте сервер, если вы планируете использовать другие шаблоны для быстрого начала работы и руководства в этой коллекции. В следующем примере показано, как удалить сервер, созданный на предыдущем шаге.


```powershell
Remove-AzAnalysisServicesServer -Name "myserver" -ResourceGroupName "myResourceGroup"
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать сервер в своей подписке Azure с помощью PowerShell. Теперь, когда у вас есть сервер, вы можете защитить его, настроив брандмауэр сервера (необязательно). Можно также добавить базовый образец модели данных на сервер непосредственно с портала. Наличие образца модели полезно для изучения настройки ролей модели базы данных и тестирования клиентских подключений. Для получения дополнительных сведений перейдите к руководству по добавлению образца модели.

> [!div class="nextstepaction"]
> [Краткое руководство. Настройка брандмауэра сервера с помощью портала](analysis-services-qs-firewall.md)      

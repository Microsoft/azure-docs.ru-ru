---
title: Как найти идентификатор клиента — Azure Active Directory
description: Инструкции по поиску и Azure Active Directory идентификатора клиента в существующую подписку Azure.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 10/30/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, devx-track-azurecli
ms.collection: M365-identity-device-management
ms.openlocfilehash: 910e68e5b8933875f252faf65fb58e514dee8254
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96326596"
---
# <a name="how-to-find-your-azure-active-directory-tenant-id"></a>Как найти идентификатор клиента Azure Active Directory

Подписки Azure имеют отношение доверия с Azure Active Directory (Azure AD). Azure AD является доверенным для проверки подлинности пользователей, служб и устройств для подписки. С каждой подпиской связан идентификатор клиента, и существует несколько способов найти идентификатор клиента для подписки.

## <a name="find-tenant-id-through-the-azure-portal"></a>Поиск идентификатора клиента с помощью портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).
 
1. Выберите **Azure Active Directory**.

1. Выберите **Свойства**.

1. Затем прокрутите вниз до поля " **идентификатор клиента** ". Ваш идентификатор клиента будет находиться в поле.

:::image type="content" source="media/active-directory-how-to-find-tenant/portal-tenant-id.png" alt-text="Azure Active Directory-Properties — идентификатор клиента — поле идентификатора клиента":::

## <a name="find-tenant-id-with-powershell"></a>Поиск идентификатора клиента с помощью PowerShell

Кроме того, клиент можно найти программно. Чтобы найти идентификатор клиента с Azure PowerShell, используйте командлет `Get-AzTenant` .

```azurepowershell-interactive
Connect-AzAccount
Get-AzTenant
```
   
Дополнительные сведения см. в этой Azure PowerShell ссылке на командлет [Get-азтенант](/powershell/module/az.accounts/get-aztenant).


## <a name="find-tenant-id-with-cli"></a>Поиск идентификатора клиента с помощью интерфейса командной строки
Если вы хотите использовать интерфейс командной строки для поиска идентификатора клиента, это можно сделать с помощью [Azure CLI](/cli/azure/install-azure-cli) или [Microsoft 365 CLI](https://pnp.github.io/cli-microsoft365/). 

Для Azure CLI используйте одну из команд **AZ login**, **AZ Account List** или **AZ Account List** , как показано в следующем примере. Обратите внимание на свойство **tenantId** для каждой из подписок в выходных данных каждой команды.

```azurecli-interactive
az login
az account list
az account tenant list
```

Дополнительные сведения см. в статьях о команде [AZ login](/cli/azure/reference-index#az_login) , справочнике по командам AZ [Account](/cli/azure/ext/account/account) или [AZ Account](/cli/azure/ext/account/account/tenant) , ссылка на команду.


Для Microsoft 365 CLI используйте **идентификатор клиента** командлета, как показано в следующем примере:
 
```cli
m365 tenant id get
```

Дополнительные сведения см. в справочнике по [идентификатору клиента Microsoft 365 Get](https://pnp.github.io/cli-microsoft365/cmd/tenant/id/id-get/) .


## <a name="next-steps"></a>Дальнейшие действия

- Сведения о создании нового клиента Azure AD см. [в разделе Краткое руководство. Создание нового клиента в Azure Active Directory](active-directory-access-create-new-tenant.md).

- Сведения о том, как связать или добавить подписку на клиент, см. в статье [связывание или Добавление подписки Azure в клиент Azure Active Directory](active-directory-how-subscriptions-associated-directory.md).

- Сведения о том, как найти идентификатор объекта, см. в разделе [Поиск идентификатора объекта пользователя](/partner-center/find-ids-and-domain-names#find-the-user-object-id).
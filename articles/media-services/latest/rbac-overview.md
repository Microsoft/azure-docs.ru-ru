---
title: Управление доступом на основе ролей для учетных записей служб мультимедиа
description: В этой статье рассматривается управление доступом на основе ролей Azure (Azure RBAC) для учетных записей служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 7c9edce71e0d0565de80153bfd5b3d5f1edb2e8f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955162"
---
# <a name="azure-role-based-access-control-azure-rbac-for-media-services-accounts"></a>Управление доступом на основе ролей Azure (Azure RBAC) для учетных записей служб мультимедиа

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Сейчас службы мультимедиа Azure не определяют никаких пользовательских ролей, относящихся к службе. Чтобы получить полный доступ к учетной записи служб мультимедиа, клиенты могут использовать встроенные роли **владельца** или **участника**. Основное различие между этими ролями состоит в том, что **владелец** может контролировать доступ к ресурсу, а **участник** — нет. Также можно использовать встроенную роль **читателя** , но пользователь или приложение будут иметь доступ только для чтения к API-интерфейсам служб мультимедиа. 

## <a name="design-principles"></a>Принципы проектирования

Один из ключевых принципов проектирования API версии 3 — сделать API более безопасным. API V3 не возвращают секреты или учетные данные при операциях **Get** или **List** . Ключи всегда являются NULL, пустыми или исключенными из ответа. Пользователю необходимо вызвать отдельный метод действия для получения секретов или учетных данных. Роль **читателя** не может вызывать такие операции, как Asset. Листконтаинерсас, Стреаминглокатор. Листконтенткэйс, КонтенткэйполиЦиес. жетполиципропертиесвиссекретс. Наличие отдельных действий позволяет при необходимости задать более детализированные разрешения безопасности RBAC Azure в настраиваемой роли.

Чтобы получить список поддерживаемых служб мультимедиа, выполните следующие действия.

```csharp
foreach (Microsoft.Azure.Management.Media.Models.Operation a in client.Operations.List())
{
    Console.WriteLine($"{a.Name} - {a.Display.Operation} - {a.Display.Description}");
}
```

В статье [встроенные определения ролей](../../role-based-access-control/built-in-roles.md) содержатся сведения о том, что предоставляет роль. 

Дополнительные сведения см. в этих статьях:

- [Роли классического администратора подписки, роли Azure и роли Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md)
- [Что такое управление доступом на основе ролей в Azure (RBAC)?](../../role-based-access-control/overview.md)
- [Добавление и удаление назначений ролей Azure с помощью REST API](../../role-based-access-control/role-assignments-rest.md)
- [Операции поставщика ресурсов служб мультимедиа](../../role-based-access-control/resource-provider-operations.md#microsoftmedia)

## <a name="next-steps"></a>Дальнейшие действия

- [Разработка с помощью API-интерфейсов служб мультимедиа v3](media-services-apis-overview.md)
- [Получение политики ключей содержимого с помощью служб мультимедиа .NET](get-content-key-policy-dotnet-howto.md)

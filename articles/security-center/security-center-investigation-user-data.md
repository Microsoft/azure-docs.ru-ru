---
title: Управление данными пользователя, находящимися в анализе центра безопасности Azure
description: " Сведения об управлении данными пользователя, находящимися в анализе центра безопасности Azure. "
services: operations-management-suite
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 411d7bae-c9d4-4e83-be63-9f2f2312b075
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/20/2018
ms.author: memildin
ms.openlocfilehash: 89d47f1fe941dad6a2450df86bfed2d25a1d8f7d
ms.sourcegitcommit: e559daa1f7115d703bfa1b87da1cf267bf6ae9e8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/17/2021
ms.locfileid: "100595663"
---
# <a name="manage-user-data-found-in-an-azure-security-center-investigation"></a>Управление данными пользователя, находящимися в анализе центра безопасности Azure
В этой статье содержатся сведения об управлении данными пользователя, находящимися в анализе центра безопасности Azure. Данные расследования хранятся в [журналах Azure Monitor](../azure-monitor/logs/log-query-overview.md) и доступны в центре безопасности. Управление данными пользователя включает в себя возможность удаления и экспорта данных.

[!INCLUDE [gdpr-intro-sentence.md](../../includes/gdpr-intro-sentence.md)]

## <a name="searching-for-and-identifying-personal-data"></a>Поиск и идентификация персональных данных
На портале Azure можно использовать функцию анализа центра безопасности, чтобы осуществлять поиск персональных данных. Функция анализа доступна в разделе **Оповещения системы безопасности**.

Все сущности, сведения о пользователе и данные отображаются на вкладке **Сущности**.

## <a name="securing-and-controlling-access-to-personal-information"></a>Защита и контроль доступа к персональным данным
Пользователь центра безопасности Azure, которому назначена роль читателя, владельца, участника или администратора учетной записи, может получить доступ к данным клиента в рамках этого центра.

Дополнительные сведения о ролях читателя, владельца и участника см. в статье [Встроенные роли управления доступом на основе ролей в Azure](../role-based-access-control/built-in-roles.md). Дополнительные сведения о роли администратора учетной записи см. в статье [Добавление или изменение администраторов подписки Azure](../cost-management-billing/manage/add-change-subscription-administrator.md).

## <a name="deleting-personal-data"></a>Удаление персональных данных
Пользователи центра безопасности, которым назначены роли читателя, владельца, участника или администратора учетной записи, могут удалять сведения об анализе.

Чтобы удалить анализ, можно отправить запрос `DELETE` в REST API Azure Resource Manager:

```HTTP
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/features/security/incidents/{incidentName}
```

Входные данные `incidentName` можно найти, получив список всех инцидентов с помощью запроса `GET`.

```HTTP
GET
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/features/security/incidents
```

## <a name="exporting-personal-data"></a>Экспорт персональных данных
Пользователи центра безопасности, которым назначены роли читателя, владельца, участника или администратора учетной записи, могут экспортировать сведения об анализе. Чтобы экспортировать сведения об анализе, перейдите на вкладку **Сущности**, где можно скопировать и вставить соответствующие сведения.

## <a name="next-steps"></a>Дальнейшие шаги
Дополнительные сведения об управлении данными пользователя в центре безопасности Azure см. в [этой статье](security-center-privacy.md).
Дополнительные сведения об удалении личных данных в журналах Azure Monitor см. в разделе [Экспорт и удаление закрытых данных](../azure-monitor/logs/personal-data-mgmt.md#how-to-export-and-delete-private-data).
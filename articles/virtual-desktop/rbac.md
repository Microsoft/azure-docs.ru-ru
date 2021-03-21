---
title: Встроенные роли виртуальный рабочий стол Windows — Azure
description: Обзор встроенных ролей для виртуальных рабочих столов Windows, доступных для Azure RBAC.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 1120db685c54ff062f03aca9002bf77af549bc26
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582125"
---
# <a name="built-in-roles-for-windows-virtual-desktop"></a>Встроенные роли для виртуальных рабочих столов Windows

Виртуальный рабочий стол Windows использует управление доступом на основе ролей Azure (RBAC) для назначения ролей пользователям и администраторам. Эти роли дают администраторам разрешение на выполнение определенных задач. Дополнительные сведения о встроенных ролях для Azure RBAC см. в статье [встроенные роли Azure](../role-based-access-control/built-in-roles.md).

Стандартные встроенные роли для Azure — владелец, участник и читатель. Однако виртуальные рабочие столы Windows имеют дополнительные роли, позволяющие разделять роли управления для пулов узлов, групп приложений и рабочих областей. Это разделение позволяет более детально управлять задачами администрирования. Имена этих ролей соответствуют стандартным ролям Azure и методологии с минимальными правами доступа.

Виртуальный рабочий стол Windows не имеет определенной роли владельца. Однако для объектов службы можно использовать роль стандартного владельца.

## <a name="desktop-virtualization-contributor"></a>Участник виртуализации настольных систем

Роль участника виртуализации настольных систем позволяет управлять всеми аспектами развертывания. Однако он не предоставляет доступ к ресурсам вычислений. Кроме того, вам потребуется роль администратора доступа пользователей для публикации групп приложений для пользователей или групп пользователей.


- Microsoft. Десктопвиртуализатион/\* 
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="desktop-virtualization-reader"></a>Средство чтения виртуализации рабочего стола

Роль читатель виртуализации рабочих столов позволяет просматривать все компоненты в развертывании, но не позволяет вносить изменения.

- Microsoft. Десктопвиртуализатион/ \* /Реад
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="host-pool-contributor"></a>Участник пула узлов

Роль участник пула узлов позволяет управлять всеми аспектами пулов узлов, включая доступ к ресурсам. Для создания виртуальных машин вам потребуются дополнительные роли участника, участники виртуальных машин. Для создания пула узлов с помощью портала вам потребуются роли участников Аппграуп и рабочей области, или вы можете использовать роль участника виртуализации рабочих столов.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/хостпулс/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="host-pool-reader"></a>Средство чтения пула узлов

Роль "читатель пула узлов" позволяет просматривать все объекты в пуле узлов, но не позволяет вносить изменения.

- Microsoft. Десктопвиртуализатион/хостпулс/ \* /Реад
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="application-group-contributor"></a>Участник группы приложений

Роль участника группы приложений позволяет управлять всеми аспектами групп приложений. Если вы хотите опубликовать группы приложений для пользователей или групп пользователей, вам потребуется роль администратора доступа пользователей.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/аппликатионграупс/\*
- Microsoft. Десктопвиртуализатион/хостпулс/Read
- Microsoft. Десктопвиртуализатион/хостпулс/сессионхостс/Read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="application-group-reader"></a>Читатель группы приложений

Роль Читатель группы приложений позволяет просматривать все объекты в группе приложений и не позволяет вносить изменения.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/аппликатионграупс/ \* /Реад
- Microsoft. Десктопвиртуализатион/аппликатионграупс/Read
- Microsoft. Десктопвиртуализатион/хостпулс/Read
- Microsoft. Десктопвиртуализатион/хостпулс/сессионхостс/Read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="workspace-contributor"></a>Участник рабочей области

Роль участника рабочей области позволяет управлять всеми аспектами рабочих областей. Чтобы получить сведения о приложениях, добавленных в группы приложений, необходимо также назначить роль читателя группы приложений.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/рабочие области/\*
- Microsoft. Десктопвиртуализатион/аппликатионграупс/Read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/\*
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="workspace-reader"></a>Читатель рабочей области

Роль читатель рабочей области позволяет просматривать все в рабочей области, но не позволяет вносить изменения.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/рабочие области/чтение
- Microsoft. Десктопвиртуализатион/аппликатионграупс/Read
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="user-session-operator"></a>Оператор сеанса пользователя

Роль оператора сеанса пользователя позволяет отсылать сообщения, отключать сеансы и использовать функцию "выход из системы" для подписывания сеансов из узла сеансов. Однако эта роль не позволяет выполнять управление узлами сеансов, например удаление узла сеансов, изменение режима стока и т. д. Эта роль может просматривать назначения, но не может изменять администраторов. Рекомендуется назначить эту роль конкретным пулам узлов. Если предоставить это разрешение на уровне группы ресурсов, администратор будет иметь разрешение на чтение всех пулов узлов в группе ресурсов.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/хостпулс/Read
- Microsoft. Десктопвиртуализатион/хостпулс/сессионхостс/Read
- Microsoft. Десктопвиртуализатион/хостпулс/сессионхостс/усерсессионс/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

## <a name="session-host-operator"></a>Оператор узла сеанса

Роль "оператор узла сеанса" позволяет просматривать и удалять узлы сеансов, а также режим очистки изменений. Они не могут добавлять узлы сеансов с помощью портал Azure, так как они не имеют разрешения на запись для объектов пула узлов. Если маркер регистрации является допустимым (создан и не просрочен), эту роль можно использовать для добавления узлов сеансов в пул узлов за пределами портал Azure, если администратор имеет разрешения на вычисление с помощью роли участника виртуальной машины.

В следующем списке перечислены разрешения, к которым имеет доступ эта роль:

- Microsoft. Десктопвиртуализатион/хостпулс/Read
- Microsoft. Десктопвиртуализатион/хостпулс/сессионхостс/\*
- Microsoft.Resources/subscriptions/resourceGroups/read
- Microsoft.Resources/deployments/read
- Microsoft.Authorization/\*/read
- Microsoft.Insights/alertRules/\*
- Microsoft.Support/\*

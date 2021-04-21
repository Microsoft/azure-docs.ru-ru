---
author: baanders
description: Включаемый файл для этапа задания прав доступа при настройке Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: a905bb3b4effb0381facfbfaa37c8ea412b81287
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "100551948"
---
Для реализации управления доступом на основе ролей (RBAC) в Azure Digital Twins используется платформа [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md). Это означает, что прежде чем пользователь сможет выполнять вызовы плоскости данных в ваш экземпляр Azure Digital Twins, ему нужно назначить роль с соответствующими правами доступа.

Для Azure Digital Twins такой ролью является _**Владелец данных Azure Digital Twins**_. Дополнительные сведения о назначении ролей и безопасности см. в статье [*Основные понятия: безопасность для решений Azure Digital Twins*](../articles/digital-twins/concepts-security.md).

В этом разделе показано, как создать процедуру назначения ролей для пользователей в экземпляре Azure Digital Twins, используя электронную почту пользователей в клиенте Azure AD в подписке Azure. В зависимости от вашей роли в организации вы можете задать такую привилегию для себя или настроить возможность ее реализации другим пользователем, который будет управлять экземпляром Azure Digital Twins.

### <a name="assign-the-role"></a>Назначение роли

Чтобы предоставить пользователю привилегии на управление экземпляром Azure Digital Twins, необходимо назначить им в экземпляре роль _**Владелец данных Azure Digital Twins**_.

> [!NOTE]
> Обратите внимание, что эта роль отличается от роли *Владельца* Azure AD, которая также может быть назначена в области экземпляра Azure Digital Twins. Это две отдельные роли с правами управления, и *Владелец* Azure AD не предоставляет доступ к функциям плоскости данных, предоставляемым *Владельцем данных Azure Digital Twins*.
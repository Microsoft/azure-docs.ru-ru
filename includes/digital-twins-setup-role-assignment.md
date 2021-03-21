---
author: baanders
description: включить файл для шага разрешений доступа в программе установки Azure Digital двойников
ms.service: digital-twins
ms.topic: include
ms.date: 7/17/2020
ms.author: baanders
ms.openlocfilehash: a905bb3b4effb0381facfbfaa37c8ea412b81287
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100551948"
---
Azure Digital двойников использует [Azure Active Directory (Azure AD)](../articles/active-directory/fundamentals/active-directory-whatis.md) для управления доступом на основе РОЛЕЙ (RBAC). Это означает, что прежде чем пользователь сможет выполнять вызовы плоскости данных в ваш экземпляр Azure Digital двойников, ему нужно назначить роль с соответствующими разрешениями.

Для Azure Digital двойников эта роль является _**владельцем Azure Digital двойников данных**_. Дополнительные сведения о ролях и безопасности см. в статье [*Security for Azure Digital двойников Solutions*](../articles/digital-twins/concepts-security.md).

В этом разделе показано, как создать назначение ролей для пользователя в своем экземпляре Azure Digital двойников, используя электронную почту этого пользователя в клиенте Azure AD в подписке Azure. В зависимости от вашей роли в организации вы можете настроить это разрешение самостоятельно или настроить его от имени другого пользователя, который будет управлять экземпляром Digital двойников Azure.

### <a name="assign-the-role"></a>Назначение роли

Чтобы предоставить пользователю разрешения на управление экземпляром Digital двойников Azure, необходимо назначить им роль _**владельца данных Digital двойников**_ в экземпляре.

> [!NOTE]
> Обратите внимание, что эта роль отличается от роли *владельца* Azure AD, которая также может быть назначена в области экземпляра Azure Digital двойников. Это две отдельные роли управления, и *владелец* Azure AD не предоставляет доступ к функциям плоскости данных, предоставленным *владельцем данных Azure Digital двойников*.
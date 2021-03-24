---
title: Что такое подготовка на основе управления персоналом с помощью Azure Active Directory? | Документы Майкрософт
description: В этой статье приведены общие сведения о подготовке на основе управления персоналом.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 10/30/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1ffe8c1397525348e472e965a407909fee36152d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96168454"
---
# <a name="what-is-hr-driven-provisioning"></a>Что такое подготовка на основе управления персоналом?

![Подготовка на основе управления персоналом](./media/what-is-hr-driven-provisioning/cloud2a.png)

Подготовка на основе управления персоналом — это процесс создания цифровых удостоверений на основе системы управления персоналом.  Системы управления персоналом становятся исходным источником полномочий для этих недавно созданных цифровых удостоверений и часто являются отправной точкой для многочисленных процессов подготовки.  Например, если новый сотрудник присоединяется к компании, в системе управления персоналом создается соответствующая запись.  Создание этой записи активирует подготовку учетной записи пользователя в Active Directory, а затем Azure AD Connect подготавливает эту учетную запись для Azure AD и т. д.

Подготовка кадров может выполняться как в локальной среде, так и в облаке.

## <a name="on-premises-based-hr-provisioning"></a>Подготовка кадров в локальной среде
Подготовка кадров в локальной среде осуществляется с помощью локальной системы управления персоналом и средств подготовки новых цифровых удостоверений.

Системы управления персоналом могут принимать форму пакетов, пакетов программного обеспечения и использовать серверы SQL, каталоги LDAP и т. д.

Сейчас решения для подготовки кадров в локальной среде Майкрософт используют Microsoft Identity Manager для активации подготовки при создании нового удостоверения в этих системах управления персоналом.

С помощью MIM вы сможете подготавливать пользователей из локальных систем управления персоналом в Active Directory или Azure AD.

Сведения о продукте Microsoft Identity Manager и поддерживаемых им системах см. в документации по [Microsoft Identity Manager](/microsoft-identity-manager/microsoft-identity-manager-2016).

[!INCLUDE [active-directory-hr-provisioning.md](../../../includes/active-directory-hr-provisioning.md)]



## <a name="next-steps"></a>Дальнейшие действия 
- [Что такое управление жизненным циклом удостоверений](what-is-identity-lifecycle-management.md)
- [Что собой представляет подготовка?](what-is-provisioning.md)
- [Что такое подготовка приложений?](what-is-app-provisioning.md)
- [Что такое подготовка между каталогами?](what-is-inter-directory-provisioning.md)
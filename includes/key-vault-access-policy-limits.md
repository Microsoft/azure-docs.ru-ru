---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 08/27/2020
ms.author: msmbaldwin
ms.openlocfilehash: 9e4d60e501ffc5b61c87a80b8dd13698cca28737
ms.sourcegitcommit: 2aa52d30e7b733616d6d92633436e499fbe8b069
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/06/2021
ms.locfileid: "97934550"
---
Хранилище ключей поддерживает до 1024 политик доступа, при этом каждая запись предоставляет отдельный набор разрешений для определенного субъекта безопасности. Из-за этого ограничения рекомендуется назначать политики доступа группам пользователей, где это возможно, а не отдельные пользователи. Использование групп значительно упрощает управление разрешениями для нескольких пользователей в Организации. Дополнительные сведения см. в статье [Управление доступом к приложениям и ресурсам с помощью групп Azure Active Directory](../articles/active-directory/fundamentals/active-directory-manage-groups.md) .

Дополнительные сведения об управлении доступом в Key Vault см. в разделе [Azure Key Vault Security: Identity and access management](../articles/key-vault/general/security-overview.md#identity-management) (Безопасность Azure Key Vault: управление удостоверениями и доступом).
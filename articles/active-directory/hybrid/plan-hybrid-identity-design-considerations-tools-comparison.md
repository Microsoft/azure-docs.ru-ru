---
title: 'Гибридная идентификация: сравнение инструментов интеграции каталогов | Документация Майкрософт'
description: На этой странице приведено исчерпывающее сравнение различных инструментов, которые можно использовать для интеграции каталогов.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 1e62a4bd-4d55-4609-895e-70131dedbf52
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/07/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 70e5c7e5fa428d5a71ca1a7468bbaab2fc94078e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101643850"
---
# <a name="hybrid-identity-directory-integration-tools-comparison"></a>Сравнение инструментов интеграции каталогов гибридной идентификации
На протяжении многих лет инструменты интеграции каталогов развивались и расширялись.  


- [FIM](/previous-versions/windows/desktop/forefront-2010/ff182370(v=vs.100)) и [MIM](/microsoft-identity-manager/microsoft-identity-manager-2016) по-прежнему поддерживаются и в основном обеспечивают синхронизацию между локальными системами.   [Соединитель FIM Windows Azure AD](/previous-versions/mim/dn511001(v=ws.10)) поддерживается как в FIM, так и в MIM, но не рекомендуется для новых развертываний. клиенты с локальными источниками, такими как примечания или SAP HCM, должны использовать MIM для заполнения служб домен Active Directory Services (AD DS), а затем использовать либо Azure AD Connect синхронизацию, либо Azure AD Connect подготовки облака для синхронизации из AD DS в Azure AD.
- [Azure AD Connect синхронизация](how-to-connect-sync-whatis.md) включает компоненты и функции, ранее выпущенные в DirSync и Azure AD Sync, для синхронизации между лесами AD DS и Azure AD.  
- [Azure AD Connect подготовка облака](../cloud-sync/what-is-cloud-sync.md) — это новый агент Майкрософт для синхронизации из AD DS с Azure AD. он полезен для таких сценариев, как слияние и приобретение, когда приобретенные леса AD изолированы от леса AD родительской компании.

Дополнительные сведения о различиях между Azure AD Connect синхронизацией и Azure AD Connect подготовки облачных ресурсов см. в статье [что такое Azure AD Connect подготовки облака?](../cloud-sync/what-is-cloud-sync.md)

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
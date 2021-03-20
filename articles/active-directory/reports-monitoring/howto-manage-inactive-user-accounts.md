---
title: Управление неактивными учетными записями пользователей в Azure AD | Документация Майкрософт
description: Узнайте, как обнаружить устаревшие учетные записи в Azure AD и управлять ими
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: ada19f69-665c-452a-8452-701029bf4252
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 01/21/2021
ms.author: markvi
ms.reviewer: besiler
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8fb517f8c50ad2c32f23542e60069a0e0a496a2d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98660670"
---
# <a name="how-to-manage-inactive-user-accounts-in-azure-ad"></a>Практическое руководство. Управление неактивными учетными записями пользователей в Azure AD

В больших средах учетные записи пользователей не всегда удаляются, когда сотрудники уходят из организации. IT-администратору нужно обнаружить устаревшие учетные записи и разобраться с ними, поскольку они представляют угрозу безопасности.

В этой статье описывается метод управления устаревшими учетными записями в Azure AD. 

## <a name="what-are-inactive-user-accounts"></a>Что такое неактивные учетные записи пользователей?

Неактивные учетные записи — это учетные записи пользователей, которые сотрудники больше не используют для доступа к ресурсам организации. Если вы *долгое время* не входили в учетную запись вашей системы — это один из основных признаков неактивной учетной записи. Поскольку неактивные учетные записи тесно связаны с частотой входа в систему, для их обнаружения можно использовать временную метку последнего успешного входа. 

Задача этого метода заключается в том, чтобы определить, что значит *некоторое время* конкретно для вашей среды. Например, пользователи могут не входить в систему *некоторое время*, поскольку они находятся в отпуске. При определении характеристик неактивной учетной записи следует учитывать все возможные причины отсутствия входа в систему. Для многих организаций характерное для неактивной учетной записи отсутствие пользователей в сети составляет 90–180 дней. 

По последнему успешному входу в систему можно ориентировочно судить о том, понадобится ли пользователю в дальнейшем доступ к ресурсам организации.  Это поможет определить, требуется ли еще членство в группе или доступ к приложениям или их можно удалить. Если речь идет об управлении внешним пользователем, это поможет понять, активен ли он в клиенте или его данные следует очистить. 

    
## <a name="how-to-detect-inactive-user-accounts"></a>Как обнаружить неактивные учетные записи?

Вы можете обнаружить неактивные учетные записи с помощью свойства **lastSignInDateTime**, предоставленного типом ресурса **signInActivity** в API **Microsoft Graph**. Этот параметр можно использовать для отслеживания по следующим принципам:

- **По имени пользователя**. В этой ситуации вы находите пользователя по имени, вычисляя его через lastSignInDateTime: `https://graph.microsoft.com/beta/users?$filter=startswith(displayName,'markvi')&$select=displayName,signInActivity`

- **По дате, связанной с пользователем**. В этом варианте вы запрашиваете список пользователей с lastSignInDateTime до определенной даты: `https://graph.microsoft.com/beta/users?filter=signInActivity/lastSignInDateTime le 2019-06-01T00:00:00Z`






## <a name="what-you-need-to-know"></a>Это важно знать

В этом разделе перечислены необходимые сведения о свойстве lastSignInDateTime.

### <a name="how-can-i-access-this-property"></a>Как получить доступ к этому параметру?

Свойство **lastSignInDateTime** предоставляется [типом ресурса signInActivity](/graph/api/resources/signinactivity?view=graph-rest-beta) в [REST API Microsoft Graph](/graph/overview?view=graph-rest-beta#whats-in-microsoft-graph).   

### <a name="is-the-lastsignindatetime-property-available-through-the-get-azureaduser-cmdlet"></a>Доступен ли параметр lastSignInDateTime в командлете Get-AzureAdUser?

Нет.

### <a name="what-edition-of-azure-ad-do-i-need-to-access-the-property"></a>Какой выпуск Azure AD следует использовать, чтобы получить доступ к этому свойству?

Это свойство доступно для всех выпусков Azure AD.

### <a name="what-permission-do-i-need-to-read-the-property"></a>Какие права необходимы для чтения свойства?

Для чтения этого свойства следует предоставить следующие права: 

- AuditLogs.Read.All
- Organisation.Read.All  


### <a name="when-does-azure-ad-update-the-property"></a>Когда Azure AD обновляет свойство?

Каждый успешный интерактивный вход приводит к обновлению базового хранилища данных. Как правило, успешные попытки входа отображаются в соответствующем отчете в течение 10 минут.
 

### <a name="what-does-a-blank-property-value-mean"></a>Что означает пустое значение свойства?

Для создания временной метки lastSignInDateTime требуется выполнить успешный вход в систему. Поскольку параметр lastSignInDateTime является новой функцией, значение свойства lastSignInDateTime может быть пустым, если:

- Последний успешный вход пользователя выполнялся до апреля 2020.
- эта учетная запись пользователя никогда не использовалась для успешного входа.

## <a name="next-steps"></a>Дальнейшие действия

* [Получение данных с помощью API отчетов Azure Active Directory с сертификатами](tutorial-access-api-with-certificates.md)
* [Справочник по API аудита](/graph/api/resources/directoryaudit?view=graph-rest-beta) 
* [Справочник по API отчетов о действиях при входе](/graph/api/resources/signin?view=graph-rest-beta)
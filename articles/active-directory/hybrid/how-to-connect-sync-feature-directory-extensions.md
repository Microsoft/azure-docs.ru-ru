---
title: 'Синхронизация Azure AD Connect: расширения каталогов | Документация Майкрософт'
description: В этой статье описывается функция расширений каталогов в Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 995ee876-4415-4bb0-a258-cca3cbb02193
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/12/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 25d4152783129fa1c5950d6cf6287332bf90d32a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97976883"
---
# <a name="azure-ad-connect-sync-directory-extensions"></a>Синхронизация Azure AD Connect: расширения каталогов
Расширения каталогов можно использовать для расширения схемы в Azure Active Directory (Azure AD) с помощью собственных атрибутов из локального каталога Active Directory. Эта функция позволяет создавать бизнес-приложения с помощью атрибутов, которыми вы по-прежнему можете управлять локально. Эти атрибуты можно использовать с помощью [расширений](/graph/extensibility-overview
). Доступные атрибуты можно просмотреть с помощью [обозревателя Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer). Эту функцию также можно использовать для создания динамических групп в Azure AD.

В настоящее время никакие Microsoft 365 рабочие нагрузки не используют эти атрибуты.

## <a name="customize-which-attributes-to-synchronize-with-azure-ad"></a>Настройка атрибутов для синхронизации с Azure AD

Дополнительные атрибуты для синхронизации выбираются в разделе пользовательских параметров мастера установки.

> [!NOTE]
> В Azure AD Connect версиях, предшествующих 1.2.65.0, в поле поиска для **доступных атрибутов** учитывается регистр.

![Мастер расширения схемы](./media/how-to-connect-sync-feature-directory-extensions/extension2.png)  

 При установке отображаются следующие допустимые атрибуты:

* Типы объектов пользователей и групп
* Однозначные атрибуты: строка, логическое значение, целое число, двоичное значение.
* Многозначные атрибуты: строка, двоичное значение.


>[!NOTE]
> После Azure AD Connect синхронизации многозначного Active Directory атрибута в Azure AD в качестве расширения атрибута с несколькими значениями можно включить атрибут в утверждение SAML. Однако невозможно использовать эти данные через вызов API.

Список атрибутов считывается из кэша схемы, созданного во время установки Azure AD Connect. Если в схему Active Directory добавлены дополнительные атрибуты, они будут отображаться только после [обновления схемы](how-to-connect-installation-wizard.md#refresh-directory-schema).

Объект в Azure AD может иметь до 100 атрибутов расширений каталога. Максимальная длина составляет 250 символов. Если значение атрибута больше, он будет усечен модулем синхронизации.

## <a name="configuration-changes-in-azure-ad-made-by-the-wizard"></a>Изменения конфигурации в Azure AD, сделанные мастером

Во время установки Azure AD Connect приложение регистрируется при условии наличия этих атрибутов. Это приложение отображается на портале Azure. Его имя всегда является **приложением расширения схемы клиента**.

![Приложение расширения схемы](./media/how-to-connect-sync-feature-directory-extensions/extension3new.png)

Убедитесь, что выбраны **все приложения** для просмотра этого приложения.

Атрибуты имеют префикс с **расширением \_ {applicationId} \_**. ApplicationId имеет одинаковое значение для всех атрибутов в клиенте Azure AD. Это значение потребуется для всех других сценариев в этом разделе.

## <a name="viewing-attributes-using-the-microsoft-graph-api"></a>Просмотр атрибутов с помощью API Microsoft Graph

Теперь эти атрибуты доступны через Microsoft Graph API с помощью [обозревателя Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer#).

>[!NOTE]
> В Microsoft Graph API необходимо запросить возвращаемые атрибуты. Явно выберите атрибуты следующим образом: `https://graph.microsoft.com/beta/users/abbie.spencer@fabrikamonline.com?$select=extension_9d98ed114c4840d298fad781915f27e4_employeeID,extension_9d98ed114c4840d298fad781915f27e4_division` .
>
> Дополнительные сведения см. в разделе [Параметр select](/graph/query-parameters#select-parameter).

>[!NOTE]
> Не поддерживается синхронизация значений атрибутов из AADConnect с атрибутами расширения, которые не создаются AADConnect. Это может привести к проблемам с производительностью и непредвиденным результатам. Для синхронизации поддерживаются только те атрибуты расширения, которые были созданы, как показано выше.

## <a name="use-the-attributes-in-dynamic-groups"></a>Использование атрибутов в динамических группах

Одним из наиболее полезных сценариев является использование этих атрибутов в динамической безопасности или группах Microsoft 365.

1. Создайте новую группу в Azure AD. Присвойте ему хорошее имя и убедитесь, что **Тип членства** является **динамическим пользователем**.

   ![Снимок экрана с новой группой](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup1.png)

2. Выберите, чтобы **добавить динамический запрос**. Если взглянуть на свойства, эти расширенные атрибуты отображаться не будут. Их необходимо добавить первыми. Щелкните **получить пользовательские свойства расширения**, введите идентификатор приложения и нажмите кнопку **обновить свойства**.

   ![Снимок экрана с добавленными расширениями каталогов](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup2.png) 

3. Откройте раскрывающийся список свойств и обратите внимание, что добавленные атрибуты теперь видимы.

   ![Снимок экрана с новыми атрибутами, отображаемыми в пользовательском интерфейсе](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup3.png)

   Заполните выражение в соответствии с вашими требованиями. В нашем примере правило имеет значение **(User.extension_9d98ed114c4840d298fad781915f27e4_division-EQ "продажи и маркетинг")**.

4. После создания группы предоставьте Azure AD некоторое время для заполнения участников, а затем просмотрите участников.

   ![Снимок экрана с элементами в динамической группе](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup4.png)  

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше о настройке [службы синхронизации Azure AD Connect](how-to-connect-sync-whatis.md) .

Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).

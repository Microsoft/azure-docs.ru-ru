---
title: Назначение ролей именам субъектов-служб Соглашения Enterprise Azure
description: Эта статья поможет вам назначить роли именам субъектов-служб с помощью PowerShell и REST API.
author: bandersmsft
ms.reviewer: ruturajd
tags: billing
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 03/07/2021
ms.author: banders
ms.openlocfilehash: e7f5370e1e387947d196959fef31043ea8f4d3bd
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102508526"
---
# <a name="assign-roles-to-azure-enterprise-agreement-service-principal-names"></a>Назначение ролей именам субъектов-служб Соглашения Enterprise Azure

Вы можете управлять регистрацией Соглашения Enterprise (EA) на [портале Azure Enterprise](https://ea.azure.com/). Вы можете создавать различные роли для управления организацией, просматривать затраты и создавать подписки. Эта статья поможет автоматизировать некоторые из этих задач с помощью Azure PowerShell и REST API с именами субъекта-службы Azure (SPN).

Прежде чем приступать к работе, убедитесь, что вы ознакомлены со следующими статьями.

- [Роли для Соглашения Enterprise](understand-ea-roles.md)
- [Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps)
- [Вызов REST API с помощью Postman](/rest/api/azure/#how-to-call-azure-rest-apis-with-postman)

## <a name="create-and-authenticate-your-service-principal"></a>Создание и проверка подлинности субъекта-службы

Чтобы автоматизировать действия EA с помощью имени субъекта-службы, необходимо создать приложение Azure Active Directory (Azure AD). Оно может выполнять проверку подлинности автоматически. Ознакомьтесь со следующими статьями и выполните действия, описанные в этой статье, чтобы создать и проверить подлинность субъекта-службы.

1. [Создание субъекта-службы](../../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal)
2. [Получение значений идентификатора клиента и приложения для входа](../../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in)

Ниже приведен пример снимка экрана, показывающий регистрацию приложения.

:::image type="content" source="./media/assign-roles-azure-service-principals/register-an-application.png" alt-text="Снимок экрана, показывающий регистрацию приложения." lightbox="./media/assign-roles-azure-service-principals/register-an-application.png" :::

### <a name="find-your-spn-and-tenant-id"></a>Поиск имени субъекта-службы и идентификатора клиента

Кроме того, необходим идентификатор объекта SPN и идентификатор клиента приложения. Сведения об операциях назначения разрешений см. в следующих разделах.

Идентификатор клиента приложения Azure AD можно найти на странице обзора приложения. Чтобы найти его на портале Azure, перейдите к Azure Active Directory и выберите **Корпоративные приложения**. Найдите приложение.

:::image type="content" source="./media/assign-roles-azure-service-principals/enterprise-application.png" alt-text="Снимок экрана, показывающий пример корпоративного приложения." lightbox="./media/assign-roles-azure-service-principals/enterprise-application.png" :::

Выберите приложение. Ниже приведен пример идентификатора приложения и идентификатора объекта.

:::image type="content" source="./media/assign-roles-azure-service-principals/application-id-object-id.png" alt-text="Снимок экрана, показывающий идентификатор приложения и идентификатор объекта для корпоративного приложения." lightbox="./media/assign-roles-azure-service-principals/application-id-object-id.png" :::

Идентификатор арендатора можно найти на странице обзора Microsoft Azure AD.

:::image type="content" source="./media/assign-roles-azure-service-principals/tenant-id.png" alt-text="Снимок экрана, показывающий, где можно найти свой идентификатор арендатора." lightbox="./media/assign-roles-azure-service-principals/tenant-id.png" :::

Идентификатор основного клиента также называется идентификатором участника, именем субъекта-службы и идентификатором объекта в различных расположениях. Значение идентификатора клиента Azure AD выглядит как GUID в таком формате: `11111111-1111-1111-1111-111111111111`.

## <a name="permissions-that-can-be-assigned-to-the-spn"></a>Разрешения, которые могут быть назначены имени субъекта-службы

Для следующих шагов вы предоставляете приложению Azure AD разрешение на действия с помощью роли EA. Имени субъекта-службы можно назначить только следующие роли. Идентификатор определения роли, точно такой, как показан, используется далее на этапах назначения.

| Роль | Разрешенные действия | Идентификатор определения роли |
| --- | --- | --- |
| EnrollmentReader | Может просматривать сведения об использовании и платежах во всех учетных записях и подписках. Может просматривать предоплату Azure (ранее называемую денежным обязательством), связанную с регистрацией. | 24f8edb6-1668-4659-b5e2-40bb5f3a7d7e |
| DepartmentReader | Загрузите сведения об использовании для отдела, который они администрируют. Может просматривать сведения об использовании и расходах, связанных с их отделом. | db609904-a47f-4794-9be8-9bd86fbffd8a |
| SubscriptionCreator | Создание новых подписок в заданной области учетной записи. | a0bcee42-bf30-4d1b-926a-48d21664ef71 |

- Модуль чтения регистрации может быть назначен имени субъекта-службы только пользователем с ролью для записи в регистрации.
- Читатель отдела может быть назначен имени субъекта-службы только пользователем с ролью для записи в регистрации или ролью для записи в отделе.
- Роль создателя подписки может быть назначена имени субъекта-службы только пользователем, который является владельцем учетной записи регистрации.

## <a name="assign-enrollment-account-role-permission-to-the-spn"></a>Назначение разрешения роли учетной записи регистрации имени субъекта-службы

См. статью REST API [Назначение ролей – Put](/rest/api/billing/2019-10-01-preview/roleassignments/put).

Во время чтения статьи выберите **Попробовать**, чтобы приступить к работе с именем субъекта-службы.

:::image type="content" source="./media/assign-roles-azure-service-principals/put-try-it.png" alt-text="Снимок экрана, показывающий пункт &quot;Попробовать&quot; в статье Put." lightbox="./media/assign-roles-azure-service-principals/put-try-it.png" :::

Войдите с помощью своей учетной записи в клиент, имеющий доступ к регистрации, в которой вы хотите назначить доступ.

Укажите следующие параметры в запросе API.

**billingAccountName**

Параметр является идентификатором учетной записи выставления счетов. Его можно найти на портале Azure на странице "Управление затратами и выставление счетов".

:::image type="content" source="./media/assign-roles-azure-service-principals/billing-account-id.png" alt-text="Снимок экрана, показывающий идентификатор учетной записи выставления счетов." lightbox="./media/assign-roles-azure-service-principals/billing-account-id.png" :::

**billingRoleAssignmentName**

Этот параметр — это уникальный идентификатор GUID, который необходимо указать. Глобально уникальный идентификатор можно создать с помощью команды PowerShell [New-Guid](/powershell/module/microsoft.powershell.utility/new-guid).

Можно также использовать веб-сайт [генератора GUID/UUID](https://guidgenerator.com/) для создания уникального идентификатора GUID.

**api-version**

Используйте версию **2019-10-01-(предварительная версия)** .

Текст запроса содержит код JSON, который необходимо использовать.

Используйте образец текста запроса в статье [Назначение ролей — Put — Примеры](/rest/api/billing/2019-10-01-preview/roleassignments/put#examples).

Существует три параметра, которые необходимо использовать в составе JSON.

| Параметр | Где ее найти |
| --- | --- |
| properties.principalId | См. раздел [Поиск имени субъекта-службы и идентификатора клиента](#find-your-spn-and-tenant-id). |
| properties.principalTenantId | См. раздел [Поиск имени субъекта-службы и идентификатора клиента](#find-your-spn-and-tenant-id). |
| properties.roleDefinitionId | "/providers/Microsoft.Billing/billingAccounts/{BillingAccountName}/billingRoleDefinitions/24f8edb6-1668-4659-b5e2-40bb5f3a7d7e" |

Имя учетной записи выставления счетов — это тот же параметр, который использовался в параметрах API. Это идентификатор регистрации, который вы видите на портале EA и на портале Azure.

Обратите внимание, что `24f8edb6-1668-4659-b5e2-40bb5f3a7d7e` является идентификатором определения роли выставления счетов для EnrollmentReader.

Щелкните **Выполнить**, чтобы выполнить приложение.

:::image type="content" source="./media/assign-roles-azure-service-principals/roleassignments-put-try-it-run.png" alt-text="Снимок экрана, показывающий пример пункта &quot;Попробовать&quot; при назначении роли с примером информации, готовой к запуску." lightbox="./media/assign-roles-azure-service-principals/roleassignments-put-try-it-run.png" :::

Ответ `200 OK` показывает, что имя субъекта-службы успешно добавлено.

Теперь можно использовать имя субъекта-службы (приложение Azure AD с идентификатором объекта) для автоматического доступа к API EA. У имени субъекта-службы есть роль EnrollmentReader.

## <a name="assign-the-department-reader-role-to-the-spn"></a>Назначение роли для чтения в отделе для имени субъекта-службы

Перед началом ознакомьтесь со статьей REST API [Назначение ролей в отделе регистрации — Put](/rest/api/billing/2019-10-01-preview/enrollmentdepartmentroleassignments/put).

При чтении статьи выберите **Попробовать**.

:::image type="content" source="./media/assign-roles-azure-service-principals/enrollment-department-role-assignments-put-try-it.png" alt-text="Снимок экрана, показывающий пункт &quot;Попробовать&quot; в статье о назначении ролей в отделе регистрации." lightbox="./media/assign-roles-azure-service-principals/enrollment-department-role-assignments-put-try-it.png" :::

Войдите с помощью своей учетной записи в клиент, имеющий доступ к регистрации, в которой вы хотите назначить доступ.

Укажите следующие параметры в запросе API.

**billingAccountName**

Это идентификатор учетной записи выставления счетов. Его можно найти на портале Azure на странице "Управление затратами и выставление счетов".

:::image type="content" source="./media/assign-roles-azure-service-principals/billing-account-id.png" alt-text="Снимок экрана, показывающий идентификатор учетной записи выставления счетов." lightbox="./media/assign-roles-azure-service-principals/billing-account-id.png" :::

**billingRoleAssignmentName**

Этот параметр — это уникальный идентификатор GUID, который необходимо указать. Глобально уникальный идентификатор можно создать с помощью команды PowerShell [New-Guid](/powershell/module/microsoft.powershell.utility/new-guid).

Можно также использовать веб-сайт [генератора GUID/UUID](https://guidgenerator.com/) для создания уникального идентификатора GUID.

**departmentName**

Это идентификатор отдела. Идентификаторы отделов можно просмотреть на портале Azure. Перейдите в "Управление затратами и выставление счетов" > **Отделы**.

В этом примере использован отдел ACE. Идентификатором в этом примере является `84819`.

:::image type="content" source="./media/assign-roles-azure-service-principals/department-id.png" alt-text="Снимок экрана, на котором показан пример идентификатора отдела." lightbox="./media/assign-roles-azure-service-principals/department-id.png" :::

**api-version**

Используйте версию **2019-10-01-(предварительная версия)** .

Текст запроса содержит код JSON, который необходимо использовать.

Используйте образец из статьи [Назначение ролей в отделе регистрации — Put](/billing/2019-10-01-preview/enrollmentdepartmentroleassignments/put). Существует три параметра, которые необходимо использовать в составе JSON.

| Параметр | Где ее найти |
| --- | --- |
| properties.principalId | См. раздел [Поиск имени субъекта-службы и идентификатора клиента](#find-your-spn-and-tenant-id). |
| properties.principalTenantId | См. раздел [Поиск имени субъекта-службы и идентификатора клиента](#find-your-spn-and-tenant-id). |
| properties.roleDefinitionId | "/providers/Microsoft.Billing/billingAccounts/{BillingAccountName}/billingRoleDefinitions/db609904-a47f-4794-9be8-9bd86fbffd8a" |

Имя учетной записи выставления счетов — это тот же параметр, который использовался в параметрах API. Это идентификатор регистрации, который вы видите на портале EA и на портале Azure.

Идентификатор определения роли выставления счетов `db609904-a47f-4794-9be8-9bd86fbffd8a` предназначен для читателя отдела.

Щелкните **Выполнить**, чтобы выполнить приложение.

:::image type="content" source="./media/assign-roles-azure-service-principals/enrollment-department-role-assignments-put-try-it-run.png" alt-text="Снимок экрана, показывающий пример назначения ролей в отделе регистрации: пункт &quot;Попробовать&quot; в REST Put с примером информации, готовой к запуску." lightbox="./media/assign-roles-azure-service-principals/enrollment-department-role-assignments-put-try-it-run.png" :::

Ответ `200 OK` показывает, что имя субъекта-службы успешно добавлено.

Теперь можно использовать имя субъекта-службы (приложение Azure AD с идентификатором объекта) для автоматического доступа к API EA. Имя субъекта-службы имеет роль DepartmentReader.

## <a name="assign-the-subscription-creator-role-to-the-spn"></a>Назначение роли создателя подписки имени субъекта-службы

См. статью [Назначение ролей для учетной записи регистрации — Put](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put).

При чтении этой статьи выберите пункт **Попробовать**, чтобы назначить роль создателя подписки имени субъекта-службы.

:::image type="content" source="./media/assign-roles-azure-service-principals/enrollment-department-role-assignments-put-try-it.png" alt-text="Снимок экрана, показывающий пункт &quot;Попробовать&quot; в статье &quot;Назначение ролей для учетной записи регистрации — Put&quot;." lightbox="./media/assign-roles-azure-service-principals/enrollment-department-role-assignments-put-try-it.png" :::

Войдите с помощью своей учетной записи в клиент, имеющий доступ к регистрации, в которой вы хотите назначить доступ.

Укажите следующие параметры в запросе API. Дополнительные сведения см. в статье [Назначение ролей для учетной записи регистрации — Put — параметры URI](/rest/api/billing/2019-10-01-preview/enrollmentaccountroleassignments/put#uri-parameters).

**billingAccountName**

Параметр является идентификатором учетной записи выставления счетов. Его можно найти на портале Azure на странице "Управление затратами и выставление счетов".

:::image type="content" source="./media/assign-roles-azure-service-principals/billing-account-id.png" alt-text="Снимок экрана, показывающий идентификатор учетной записи выставления счетов." lightbox="./media/assign-roles-azure-service-principals/billing-account-id.png" :::

**billingRoleAssignmentName**

Этот параметр — это уникальный идентификатор GUID, который необходимо указать. Глобально уникальный идентификатор можно создать с помощью команды PowerShell [New-Guid](/powershell/module/microsoft.powershell.utility/new-guid).

Можно также использовать веб-сайт [генератора GUID/UUID](https://guidgenerator.com/) для создания уникального идентификатора GUID.
**enrollmentAccountName**

Параметр является идентификатором учетной записи. Найдите идентификатор учетной записи для имени учетной записи на портале Azure на странице "Управление затратами и выставление счетов" в области "Развертывание" и в области "Отдел".

В этом примере мы использовали тестовую учетную запись GTM. Идентификатор: `196987`.

:::image type="content" source="./media/assign-roles-azure-service-principals/account-id.png" alt-text="Снимок экрана, показывающий идентификатор учетной записи." lightbox="./media/assign-roles-azure-service-principals/account-id.png" :::

**api-version**

Используйте версию **2019-10-01-(предварительная версия)** .

Текст запроса содержит код JSON, который необходимо использовать.

Используйте образец из статьи [Назначение ролей в отделе регистрации — Put — Примеры](/rest/api/billing/2019-10-01-preview/enrollmentdepartmentroleassignments/put#putenrollmentdepartmentadministratorroleassignment).

Существует три параметра, которые необходимо использовать в составе JSON.

| Параметр | Где ее найти |
| --- | --- |
| properties.principalId | См. раздел [Поиск имени субъекта-службы и идентификатора клиента](#find-your-spn-and-tenant-id). |
| properties.principalTenantId | См. раздел [Поиск имени субъекта-службы и идентификатора клиента](#find-your-spn-and-tenant-id). |
| properties.roleDefinitionId | "/providers/Microsoft.Billing/billingAccounts/{BillingAccountID}/enrollmentAccounts/196987/billingRoleDefinitions/a0bcee42-bf30-4d1b-926a-48d21664ef71" |

Имя учетной записи выставления счетов — это тот же параметр, который использовался в параметрах API. Это идентификатор регистрации, который вы видите на портале EA и на портале Azure.

Идентификатор определения роли выставления счетов `a0bcee42-bf30-4d1b-926a-48d21664ef71` предназначен для роли создателя подписки.

Щелкните **Выполнить**, чтобы выполнить приложение.

:::image type="content" source="./media/assign-roles-azure-service-principals/enrollment-account-role-assignments-put-try-it.png" alt-text="Снимок экрана, показывающий пункт &quot;Попробовать&quot; в статье &quot;Назначение ролей для учетной записи регистрации — Put&quot;" lightbox="./media/assign-roles-azure-service-principals/enrollment-account-role-assignments-put-try-it.png" :::

Ответ `200 OK` показывает, что имя субъекта-службы успешно добавлено.

Теперь можно использовать имя субъекта-службы (приложение Azure AD с идентификатором объекта) для автоматического доступа к API EA. Имя субъекта-службы имеет роль SubscriptionCreator.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [администрировании портала Azure EA](ea-portal-administration.md).
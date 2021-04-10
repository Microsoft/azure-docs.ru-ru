---
title: Разрешения приложений для пользовательских ролей в Azure Active Directory | Документация Майкрософт
description: Обзорные сведения о настройке разрешений корпоративных приложений для пользовательских ролей Azure AD с помощью портала Azure, PowerShell или API Graph.
services: active-directory
author: rolyon
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: overview
ms.date: 11/04/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.openlocfilehash: d2159540c688a63082efb792fd4d261062ef65d9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103466799"
---
# <a name="enterprise-application-permissions-for-custom-roles-in-azure-active-directory"></a>Разрешения корпоративных приложений для пользовательских ролей в Azure Active Directory

Эта статья содержит описание всех существующих на этот момент разрешений корпоративных приложений для определений пользовательских ролей в Azure Active Directory (Azure AD). В ней вы найдете списки разрешений для нескольких распространенных сценариев, а также полный список разрешений корпоративных приложений. Разрешения Application Proxy в этот выпуск пока не включены.

## <a name="required-license-plan"></a>Требуемый план лицензирования

Для использования этой функции вашей организации Azure AD требуется лицензия Azure AD Premium P1. Чтобы найти подходящую лицензию, ознакомьтесь с разделом [Сравнение общедоступных функций выпусков Free, Basic и Premium](https://azure.microsoft.com/pricing/details/active-directory/).

## <a name="enterprise-application-permissions"></a>Разрешения корпоративных приложений

Дополнительные сведения об использовании этих разрешений см. в статье [Назначение пользовательских ролей для управления корпоративными приложениями](custom-enterprise-apps.md).

### <a name="assigning-users-or-groups-to-an-application"></a>Назначение пользователей или групп приложению

Позволяет делегировать назначение пользователей или групп, которые смогут использовать приложения с единым входом на основе SAML. Необходимые разрешения

- microsoft.directory/servicePrincipals/appRoleAssignedTo/update

### <a name="creating-gallery-applications"></a>Создание приложений в коллекции

Позволяет делегировать создание приложений коллекции Azure AD, например ServiceNow, F5 и Salesforce. Необходимые разрешения:

- microsoft.directory/applicationTemplates/instantiate

### <a name="configuring-basic-saml-urls"></a>Настройка базовых URL-адресов SAML

Позволяет делегировать обновление и чтение базовых конфигураций SAML для приложений с единым входом на основе SAML. Необходимые разрешения:

- microsoft.directory/servicePrincipals/authentication/update
- microsoft.directory/applications.myOrganization/authentication/update

### <a name="rolling-over-or-creating-signing-certs"></a>Смена или создание сертификатов для подписи

Позволяет делегировать управление сертификатами для подписи для приложений с единым входом на основе SAML. Необходимые разрешения:

microsoft.directory/applications/credentials/update

### <a name="update-expiring-sign-in-cert-notification-email-address"></a>Изменение адреса электронной почты для уведомлений об истечении срока действия сертификата

Позволяет делегировать изменение адреса электронной почты для уведомлений об истечении срока действия сертификата для приложений с единым входом на основе SAML. Необходимые разрешения:

- microsoft.directory/applications.myOrganization/authentication/update
- microsoft.directory/applications.myOrganization/permissions/update
- microsoft.directory/servicePrincipals/authentication/update
- microsoft.directory/servicePrincipals/basic/update

### <a name="manage-saml-token-signature-and-sign-in-algorithm"></a>Управление алгоритмом подписи и входа для токенов SAML

Позволяет делегировать изменение алгоритма подписи и входа для токенов SAML для приложений с единым входом на основе SAML. Необходимые разрешения:

- microsoft.directory/applicationPolicies/basic/update
- microsoft.directory/applications/authentication/update
- microsoft.directory/servicePrincipals/policies/update

### <a name="manage-user-attributes-and-claims"></a>Управление атрибутами и утверждениями пользователя

Позволяет делегировать создание, удаление и обновление атрибутов и утверждений пользователя для приложений с единым входом на основе SAML. Необходимые разрешения:

- microsoft.directory/applicationPolicies/basic/update
- microsoft.directory/applications/authentication/update
- microsoft.directory/servicePrincipals/policies/update

## <a name="app-provisioning-permissions"></a>Разрешения на подготовку приложения к работе

Выполнение любых операций записи, таких как управление заданием, схемой или учетными данными через пользовательский интерфейс, требует также разрешений на чтение для просмотра страницы подготовки к работе.

Чтобы указать всех или назначенных пользователей или группы в качестве области действия, в настоящее время необходимо присвоить одновременно разрешения synchronizationJob и synchronizationCredentials.

### <a name="turn-on-or-restart-provisioning-jobs"></a>Включение или перезапуск заданий подготовки к работе

Позволяет делегировать возможность включать, выключать и перезапускать задания подготовки к работе. Необходимые разрешения:

- microsoft.directory/servicePrincipals/synchronizationJobs/manage  

### <a name="configure-the-provisioning-schema"></a>Настройка схемы подготовки к работе  

Позволяет делегировать изменение схемы сопоставления атрибутов. Необходимые разрешения:

- microsoft.directory/servicePrincipals/synchronizationSchema/manage  

### <a name="read-provisioning-settings-associated-with-the-application-object"></a>Чтение параметров подготовки к работе, связанных с объектом приложения

Позволяет делегировать возможность читать параметры подготовки к работе, связанные с объектом. Необходимые разрешения:

- microsoft.directory/applications/synchronization/standard/read

### <a name="read-provisioning-settings-associated-with-your-service-principal"></a>Чтение параметров подготовки к работе, связанных с субъектом-службой

Позволяет делегировать возможность читать параметры подготовки к работе, связанные с объектом. Необходимые разрешения:

- microsoft.directory/servicePrincipals/synchronization/standard/read

### <a name="authorize-application-access-for-provisioning"></a>Авторизация доступа к приложениям для подготовки к работе  

Позволяет делегировать возможность разрешать доступ к приложениям для подготовки к работе. Пример входящего маркера носителя OAuth. Необходимые разрешения:

- microsoft.directory/servicePrincipals/synchronizationCredentials/manage

## <a name="full-list-of-permissions"></a>Полный список разрешений

Разрешение | Описание
---------- | -----------
microsoft.directory/applicationPolicies/allProperties/read | Чтение всех свойств в политиках приложений.
microsoft.directory/applicationPolicies/allProperties/update | Обновление всех свойств в политиках приложений.
microsoft.directory/applicationPolicies/basic/update | Изменение стандартных свойств в политиках приложений.
microsoft.directory/applicationPolicies/create | Создание политик приложений.
microsoft.directory/applicationPolicies/createAsOwner | Создание политик приложений. Первым владельцем назначается создатель.
microsoft.directory/applicationPolicies/delete | Удаление политик приложений.
microsoft.directory/applicationPolicies/owners/read | Чтение сведений о владельцах для политик приложений.
microsoft.directory/applicationPolicies/owners/update | Обновление свойства владельца для политик приложений.
microsoft.directory/applicationPolicies/policyAppliedTo/read | Чтение списка политик приложений, назначенных объекту.
microsoft.directory/applicationPolicies/standard/read | Чтение стандартных свойств в политиках приложений.
microsoft.directory/servicePrincipals/allProperties/allTasks | Создание и удаление объектов servicePrincipal, а также чтение и изменение всех свойств в Azure Active Directory.
microsoft.directory/servicePrincipals/allProperties/read | Чтение всех свойств субъектов-служб.
microsoft.directory/servicePrincipals/allProperties/update | Изменение всех свойств субъектов-служб.
microsoft.directory/servicePrincipals/appRoleAssignedTo/read | Чтение назначенных субъекту-службе ролей.
microsoft.directory/servicePrincipals/appRoleAssignedTo/update | Изменение назначенных субъекту-службе ролей.
microsoft.directory/servicePrincipals/appRoleAssignments/read | Чтение назначений ролей, назначенных субъекту-службе.
microsoft.directory/servicePrincipals/audience/update | Изменение свойств целевой аудитории для субъектов-служб.
microsoft.directory/servicePrincipals/authentication/update | Изменение свойств проверки подлинности для субъектов-служб.
microsoft.directory/servicePrincipals/basic/update | Изменение базовых свойств для субъектов-служб.
microsoft.directory/servicePrincipals/create | Создание субъектов-служб.
microsoft.directory/servicePrincipals/createAsOwner | Создание субъектов-служб. Первым владельцем назначается создатель.
microsoft.directory/servicePrincipals/credentials/update | Изменение свойств учетных данных для субъектов-служб.
microsoft.directory/servicePrincipals/delete | Удаление субъектов-служб.
microsoft.directory/servicePrincipals/disable | Отключение субъектов-служб.
microsoft.directory/servicePrincipals/enable | Включение субъектов-служб.
microsoft.directory/servicePrincipals/getPasswordSingleSignOnCredentials | Чтение учетных данных для единого входа на основе пароля в субъектах-службах.
microsoft.directory/servicePrincipals/managePasswordSingleSignOnCredentials | Управление учетными данными для единого входа на основе пароля в субъектах-службах.
microsoft.directory/servicePrincipals/oAuth2PermissionGrants/read | Чтение делегированных разрешений для субъектов-служб.
microsoft.directory/servicePrincipals/owners/read | Чтение сведений о владельцах субъектов-служб.
microsoft.directory/servicePrincipals/owners/update | Изменение владельцев субъектов-служб.
microsoft.directory/servicePrincipals/permissions/update |  
microsoft.directory/servicePrincipals/policies/read | Чтение политик для субъектов-служб.
microsoft.directory/servicePrincipals/policies/update | Изменение политик для субъектов-служб.
microsoft.directory/servicePrincipals/standard/read | Чтение стандартных свойств субъектов-служб.
microsoft.directory/servicePrincipals/synchronization/standard/read | Чтение параметров подготовки к работе, связанных с субъектом-службой.
microsoft.directory/servicePrincipals/tag/update | Изменение свойства тегов для субъектов-служб.
microsoft.directory/applicationTemplates/instantiate | Создание экземпляров приложений коллекции из шаблонов приложений.
microsoft.directory/auditLogs/allProperties/read | Чтение журналов аудита.
microsoft.directory/signInReports/allProperties/read | Чтение отчетов о входах.
microsoft.directory/applications/synchronization/standard/read | Чтение параметров подготовки к работе, связанных с объектом приложения.
microsoft.directory/servicePrincipals/synchronizationJobs/manage | Управление всеми аспектами синхронизации заданий для ресурсов субъектов-служб.
microsoft.directory/servicePrincipals/synchronization/standard/read | Чтение параметров подготовки к работе, связанных с субъектами-службами.
microsoft.directory/servicePrincipals/synchronizationSchema/manage | Управление всеми аспектами синхронизации схемы для ресурсов субъектов-служб.
microsoft.directory/provisioningLogs/allProperties/read | Чтение всех свойств журналов подготовки к работе.

## <a name="next-steps"></a>Дальнейшие действия

- [Создание настраиваемых ролей с помощью портала Azure, Azure AD PowerShell или API Graph](custom-create.md)
- [Список назначений ролей](view-assignments.md)

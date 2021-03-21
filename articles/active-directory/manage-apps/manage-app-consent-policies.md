---
title: Управление политиками согласия приложений в Azure AD
description: Узнайте, как управлять встроенными и пользовательскими политиками согласия приложений, чтобы контролировать, когда можно предоставить согласие.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 06/01/2020
ms.author: kenwith
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: 9c269e2ab37a08e48eedd3ee468080a382f9a8e3
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102558735"
---
# <a name="manage-app-consent-policies"></a>Управление политиками согласия для приложений

С помощью Azure AD PowerShell можно просматривать политики согласия приложений и управлять ими.

Политика согласия приложения состоит из одного или нескольких наборов условий "включает" и нуля или нескольких наборов условий "исключения". Чтобы событие рассматривалось в политике согласия приложения, оно должно соответствовать *хотя бы* одному набору *условий "включено* " и не должно совпадать с набором условий "исключить".

Каждый набор условий состоит из нескольких условий. Чтобы событие соответствовало заданному условию, должно выполняться *все* условия в наборе условий.

Политики согласия приложений, в которых идентификатор начинается с "Microsoft-", являются встроенными политиками. Некоторые из этих встроенных политик используются в существующих встроенных ролях каталогов. Например, `microsoft-application-admin` политика согласия приложения описывает условия, при которых администратор приложения и роли администратора облачного приложения могут предоставить согласие администратора на уровне клиента. Встроенные политики могут использоваться в пользовательских ролях каталога и для настройки параметров согласия пользователя, но не могут быть изменены или удалены.

## <a name="pre-requisites"></a>Предварительные требования

1. Убедитесь, что используете модуль [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Этот важно, если вы установили модуль [AzureAD](/powershell/module/azuread/) и модуль [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview).

    ```powershell
    Remove-Module AzureAD -ErrorAction SilentlyContinue
    Import-Module AzureADPreview
    ```

1. Подключитесь к Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

## <a name="list-existing-app-consent-policies"></a>Перечисление существующих политик согласия приложений

Рекомендуется начать с того, чтобы ознакомиться с существующими политиками согласия приложений в вашей организации:

1. Список всех политик согласия на приложения:

   ```powershell
   Get-AzureADMSPermissionGrantPolicy | ft Id, DisplayName, Description
   ```

1. Просмотр наборов условий политики "включает":

    ```powershell
    Get-AzureADMSPermissionGrantConditionSet -PolicyId "microsoft-application-admin" `
                                             -ConditionSetType "includes"
    ```

1. Просмотр наборов условий "исключения":

    ```powershell
    Get-AzureADMSPermissionGrantConditionSet -PolicyId "microsoft-application-admin" `
                                             -ConditionSetType "excludes"
    ```

## <a name="create-a-custom-app-consent-policy"></a>Создание настраиваемой политики согласия приложения

Чтобы создать настраиваемую политику согласия приложения, выполните следующие действия.

1. Создайте пустую политику согласия приложений.

   ```powershell
   New-AzureADMSPermissionGrantPolicy `
       -Id "my-custom-policy" `
       -DisplayName "My first custom consent policy" `
       -Description "This is a sample custom app consent policy."
   ```

1. Добавить наборы условий "включено".

   ```powershell
   # Include delegated permissions classified "low", for apps from verified publishers
   New-AzureADMSPermissionGrantConditionSet `
       -PolicyId "my-custom-policy" `
       -ConditionSetType "includes" `
       -PermissionType "delegated" `
       -PermissionClassification "low" `
       -ClientApplicationsFromVerifiedPublisherOnly $true
   ```

   Повторите этот шаг, чтобы добавить дополнительные наборы условий "включить".

1. При необходимости добавьте наборы условий "исключить".

   ```powershell
   # Retrieve the service principal for the Azure Management API
   $azureApi = Get-AzureADServicePrincipal -Filter "servicePrincipalNames/any(n:n eq 'https://management.azure.com/')"

   # Exclude delegated permissions for the Azure Management API
   New-AzureADMSPermissionGrantConditionSet `
       -PolicyId "my-custom-policy" `
       -ConditionSetType "excludes" `
       -PermissionType "delegated" `
       -ResourceApplication $azureApi.AppId
   ```

   Повторите этот шаг, чтобы добавить дополнительные наборы условий "исключить".

После создания политики согласия приложения можно [Разрешить согласие пользователя](configure-user-consent.md?tabs=azure-powershell#allow-user-consent-subject-to-an-app-consent-policy) в соответствии с этой политикой.

## <a name="delete-a-custom-app-consent-policy"></a>Удаление политики согласия пользовательского приложения

1. Ниже показано, как можно удалить политику согласия пользовательского приложения. **Это действие невозможно отменить.**

   ```powershell
   Remove-AzureADMSPermissionGrantPolicy -Id "my-custom-policy"
   ```

> [!WARNING]
> Не удается восстановить удаленные политики согласия приложений. Если вы случайно удалили политику согласия пользовательского приложения, необходимо будет повторно создать политику.

---

### <a name="supported-conditions"></a>Поддерживаемые условия

В следующей таблице приведен список поддерживаемых условий для политик согласия приложения.

| Условие | Описание|
|:---------------|:----------|
| пермиссионклассификатион | [Классификация разрешений](configure-permission-classifications.md) для предоставленного разрешения или "все" для сопоставления с любой классификацией разрешений (включая разрешения, которые не классифицируются). Значение по умолчанию — "все". |
| пермиссионтипе | Тип разрешения предоставляемого разрешения. Используйте "приложение" для разрешений приложения (например, ролей приложений) или "делегировано" для делегированных разрешений. <br><br>**Примечание**. значение "делегатедусерконсентабле" указывает делегированные разрешения, которые не были настроены издателем API для обязательного согласия администратора. это значение может использоваться во встроенных политиках предоставления разрешений, но не может использоваться в политиках предоставления пользовательских разрешений. Обязательный. |
| ресаурцеаппликатион | **Идентификатор** приложения ресурса (например, API), для которого предоставляется разрешение, или "Any" для сопоставления с любым приложением или API ресурсов. Значение по умолчанию — Any. |
| Разрешения | Список идентификаторов разрешений для конкретных разрешений для сопоставления или список с одним значением "все" для сопоставления с любым разрешением. По умолчанию используется единственное значение "ALL". <ul><li>Идентификаторы делегированных разрешений можно найти в свойстве **OAuth2Permissions** объекта ServicePrincipal API.</li><li>Идентификаторы разрешений приложения можно найти в свойстве **AppRoles** объекта ServicePrincipal API.</li></ol> |
| клиентаппликатионидс | Список значений **AppID** для клиентских приложений, которые необходимо сопоставить с, или список с единственным значением "ALL" для соответствия любому клиентскому приложению. По умолчанию используется единственное значение "ALL". |
| клиентаппликатионтенантидс | Список идентификаторов клиентов Azure Active Directory, в которых зарегистрировано клиентское приложение, или список с одним значением "все" для сопоставления с клиентскими приложениями, зарегистрированными в любом клиенте. По умолчанию используется единственное значение "ALL". |
| клиентаппликатионпублишеридс | Список идентификаторов Microsoft Partner Network (MPN) для [проверенных издателей](../develop/publisher-verification-overview.md) клиентского приложения или список с одним значением "все" для сопоставления с клиентскими приложениями от любого издателя. По умолчанию используется единственное значение "ALL". |
| клиентаппликатионсфромверифиедпублишеронли | Присвойте параметру значение `$true` , чтобы соответствовать только клиентским приложениям с [проверенными издателями](../develop/publisher-verification-overview.md). Задайте для параметра значение `$false` , чтобы соответствовать любому клиентскому приложению, даже если у него нет проверенного издателя. По умолчанию — `$false`. |

## <a name="next-steps"></a>Дальнейшие действия

Чтобы получить дополнительные сведения, обратитесь к разделу

* [Настройка параметров согласия пользователя](configure-user-consent.md)
* [Настройка рабочего процесса согласия администратора](configure-admin-consent-workflow.md)
* [Узнайте, как управлять согласием для приложений и оценивать запросы на согласие](manage-consent-requests.md)
* [Предоставление приложению согласия администратора на уровне арендатора](grant-admin-consent.md)
* [Разрешения и согласие для платформы удостоверений Майкрософт](../develop/v2-permissions-and-consent.md)

Получение справки или ответов на вопросы:
* [Azure AD в Microsoft Q&A](/answers/products/)
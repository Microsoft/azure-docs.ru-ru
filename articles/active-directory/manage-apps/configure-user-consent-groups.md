---
title: Настройка согласия владельца группы на приложения, обращающиеся к данным группы с помощью Azure AD
description: Узнайте, могут ли владельцы групп и команд получать согласие на приложения, которые будут иметь доступ к данным группы или группы.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 05/19/2020
ms.author: kenwith
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: 8d8604a1dd54ed819bb9e27c46d61a46466bf3da
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102548807"
---
# <a name="configure-group-owner-consent-to-apps-accessing-group-data"></a>Настройка согласия владельца группы на приложения, обращающиеся к данным группы

Владельцы групп и групп могут авторизовать приложения, такие как приложения, опубликованные сторонними поставщиками, для доступа к данным Организации, связанным с группой. Например, владелец группы в Microsoft Teams может разрешить приложению читать все сообщения групп в Teams или выводить базовые профили участников группы. Дополнительные сведения см. [в разделе согласие на использование ресурсов в Microsoft Teams](/microsoftteams/resource-specific-consent) .

## <a name="manage-group-owner-consent-to-apps"></a>Управление согласием владельца группы для приложений

Можно настроить, каким пользователям разрешено предоставлять разрешения приложениям, обращающимся к их группам или данным рабочих групп, или отключить для всех пользователей.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Выполните следующие действия, чтобы управлять согласием владельца группы на приложения, обращающиеся к данным группы:

1. Войдите на [портал Azure](https://portal.azure.com) как [глобальный администратор](../roles/permissions-reference.md#global-administrator).
2. Выберите **Azure Active Directory** > **Корпоративные приложения** > **Согласия и разрешения** > **Параметры согласия пользователя**.
3. В разделе **Согласие владельца группы для приложений, обращающихся к данным** выберите параметр, который вы хотите включить.
4. Нажмите кнопку **Save** (Сохранить), чтобы сохранить настройки.

В этом примере всем владельцам групп разрешено давать согласие на доступ приложений к данным группы:

:::image type="content" source="media/configure-user-consent-groups/group-owner-consent.png" alt-text="Параметры согласия владельца группы":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Вы можете использовать модуль предварительной версии Azure AD PowerShell, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), чтобы разрешить или запретить владельцам групп предоставлять согласие приложениям, обращающимся к данным организации, для групп, которыми они владеют.

1. Убедитесь, что используете модуль [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview). Этот важно, если вы установили модуль [AzureAD](/powershell/module/azuread/) и модуль [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview).

    ```powershell
    Remove-Module AzureAD
    Import-Module AzureADPreview
    ```

1. Подключитесь к Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

1. Получите текущее значение параметров каталога **Параметры политики согласия** в клиенте. Для этого необходимо проверить, были ли созданы параметры каталога для этой функции, и если нет, использовать значения из соответствующего шаблона параметров каталога.

    ```powershell
    $consentSettingsTemplateId = "dffd5d46-495d-40a9-8e21-954ff55e198a" # Consent Policy Settings
    $settings = Get-AzureADDirectorySetting -All $true | Where-Object { $_.TemplateId -eq $consentSettingsTemplateId }

    if (-not $settings) {
        $template = Get-AzureADDirectorySettingTemplate -Id $consentSettingsTemplateId
        $settings = $template.CreateDirectorySetting()
    }

    $enabledValue = $settings.Values | ? { $_.Name -eq "EnableGroupSpecificConsent" }
    $limitedToValue = $settings.Values | ? { $_.Name -eq "ConstrainGroupSpecificConsentToMembersOfGroupId" }
    ```

1. Изучите значения параметров. Есть два значения параметров, которые определяют, какие пользователи смогут предоставлять приложению доступ к данным своей группы:

    | Параметр       | Type         | Описание  |
    | ------------- | ------------ | ------------ |
    | _EnableGroupSpecificConsent_   | Логическое | Флаг, указывающий, разрешено ли владельцам групп предоставлять разрешения для конкретной группы. |
    | _ConstrainGroupSpecificConsentToMembersOfGroupId_ | Guid | Если параметр _EnableGroupSpecificConsent_ имеет значение "true", а для этого значения задан идентификатор объекта группы, члены этой группы будут иметь права на предоставление разрешений, относящихся к их группе. |

1. Обновите значения параметров для требуемой конфигурации:

    ```powershell
    # Disable group-specific consent entirely
    $enabledValue.Value = "False"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for all users
    $enabledValue.Value = "True"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for users in a given group
    $enabledValue.Value = "True"
    $limitedToValue.Value = "{group-object-id}"
    ```

1. Сохраните параметры.

    ```powershell
    if ($settings.Id) {
        # Update an existing directory settings
        Set-AzureADDirectorySetting -Id $settings.Id -DirectorySetting $settings
    } else {
        # Create a new directory settings to override the default setting 
        New-AzureADDirectorySetting -DirectorySetting $settings
    }
    ```

---

## <a name="next-steps"></a>Дальнейшие действия

Чтобы получить дополнительные сведения, обратитесь к разделу

* [Настройка параметров согласия пользователя](configure-user-consent.md)
* [Настройка рабочего процесса согласия администратора](configure-admin-consent-workflow.md)
* [Узнайте, как управлять согласием для приложений и оценивать запросы на согласие](manage-consent-requests.md)
* [Предоставление приложению согласия администратора на уровне арендатора](grant-admin-consent.md)
* [Разрешения и согласие для платформы удостоверений Майкрософт](../develop/v2-permissions-and-consent.md)

Получение справки или ответов на вопросы:
* [Azure AD в Microsoft Q&A ](/answers/topics/azure-active-directory.html)
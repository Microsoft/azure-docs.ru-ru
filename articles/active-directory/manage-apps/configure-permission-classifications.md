---
title: Настройка классификаций разрешений с помощью Azure AD
description: Узнайте, как управлять классификациями делегированных разрешений.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 06/01/2020
ms.author: phsignor
ms.reviewer: arvindh, luleon, phsignor
ms.custom: contperf-fy21q2
ms.openlocfilehash: 12a4ffb311e01ebb78b1ae392d1243c5d67eff6b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101644570"
---
# <a name="configure-permission-classifications"></a>Настройка классификации разрешений

Классификация разрешений позволяет определить влияние различных разрешений в соответствии с политиками и оценками рисков вашей организации. Например, можно использовать классификацию разрешений в политиках согласия, чтобы определить набор разрешений, на которые пользователи могут предоставлять согласие.

## <a name="manage-permission-classifications"></a>Управление классификациями разрешений

В настоящее время поддерживается только категория разрешений "Низкое влияние". Только делегированные разрешения, не требующие согласия администратора, можно отнести к категории "Низкое влияние".

> [!TIP]
> Минимальные разрешения, необходимые для базового входа, —, `openid` , `profile` `email` `User.Read` и `offline_access` , которые являются делегированными разрешениями на Microsoft Graph. С этими разрешениями приложение может считывать полные данные профиля пользователя, выполнившего вход, и поддерживать этот доступ, даже если пользователь больше не использует приложение.

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы классифицировать разрешения с помощью портал Azure, выполните следующие действия.

1. Войдите в [портал Azure](https://portal.azure.com) в качестве [глобального администратора](../roles/permissions-reference.md#global-administrator), [администратора приложения](../roles/permissions-reference.md#application-administrator)или [администратора облачных приложений](../roles/permissions-reference.md#cloud-application-administrator) .
1. Выберите **Azure Active Directory** > **Корпоративные приложения** > **Согласия и разрешения** > **Классификация разрешений**.
1. Выберите **Добавить разрешения**, чтобы отнести разрешение в категорию "Низкое влияние".
1. Выберите API и делегированные разрешения.

В этом примере мы классифицировали минимальный набор разрешений, необходимых для единого входа:

:::image type="content" source="media/configure-permission-classifications/permission-classifications.png" alt-text="Классификации разрешений":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Вы можете использовать последний модуль предварительной версии Azure AD PowerShell, [AzureADPreview](/powershell/module/azuread/?preserve-view=true&view=azureadps-2.0-preview), чтобы классифицировать разрешения. Классификации разрешений настраиваются на объекте **ServicePrincipal** интерфейса API, который публикует разрешения.

#### <a name="list-the-current-permission-classifications-for-an-api"></a>Перечисление текущих классификаций разрешений для API

1. Извлеките объект **ServicePrincipal** для API. Здесь мы извлекаем объект ServicePrincipal для API Microsoft Graph:

   ```powershell
   $api = Get-AzureADServicePrincipal `
       -Filter "servicePrincipalNames/any(n:n eq 'https://graph.microsoft.com')"
   ```

1. Чтение классификации делегированных разрешений для API:

   ```powershell
   Get-AzureADMSServicePrincipalDelegatedPermissionClassification `
       -ServicePrincipalId $api.ObjectId | Format-Table Id, PermissionName, Classification
   ```

#### <a name="classify-a-permission-as-low-impact"></a>Классификация разрешения как "низкого влияния"

1. Извлеките объект **ServicePrincipal** для API. Здесь мы извлекаем объект ServicePrincipal для API Microsoft Graph:

   ```powershell
   $api = Get-AzureADServicePrincipal `
       -Filter "servicePrincipalNames/any(n:n eq 'https://graph.microsoft.com')"
   ```

1. Найдите делегированное разрешение, которому вы хотите присвоить категорию:

   ```powershell
   $delegatedPermission = $api.OAuth2Permissions | Where-Object { $_.Value -eq "User.ReadBasic.All" }
   ```

1. Задайте категорию разрешения с помощью его имени и идентификатора:

   ```powershell
   Add-AzureADMSServicePrincipalDelegatedPermissionClassification `
      -ServicePrincipalId $api.ObjectId `
      -PermissionId $delegatedPermission.Id `
      -PermissionName $delegatedPermission.Value `
      -Classification "low"
   ```

#### <a name="remove-a-delegated-permission-classification"></a>Удаление классификации делегированных разрешений

1. Извлеките объект **ServicePrincipal** для API. Здесь мы извлекаем объект ServicePrincipal для API Microsoft Graph:

   ```powershell
   $api = Get-AzureADServicePrincipal `
       -Filter "servicePrincipalNames/any(n:n eq 'https://graph.microsoft.com')"
   ```

1. Найдите категорию делегированных разрешений, которую вы хотите удалить:

   ```powershell
   $classifications = Get-AzureADMSServicePrincipalDelegatedPermissionClassification `
       -ServicePrincipalId $api.ObjectId
   $classificationToRemove = $classifications | Where-Object {$_.PermissionName -eq "User.ReadBasic.All"}
   ```

1. Удаление категории разрешений:

   ```powershell
   Remove-AzureADMSServicePrincipalDelegatedPermissionClassification `
       -ServicePrincipalId $api.ObjectId `
       -Id $classificationToRemove.Id
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
* [Azure AD в Microsoft Q&A](/answers/topics/azure-active-directory.html)
---
title: Создание приложения Azure AD для доступа к API служб мультимедиа Azure с помощью PowerShell | Документация Майкрософт
description: Узнайте, как использовать PowerShell, чтобы создать приложение Azure Active Directory (Azure AD) и настроить его для доступа к API служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: d16c2bbb16a19e5cb22b2b2b0378880ec9aa48b5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103009663"
---
# <a name="use-powershell-to-create-an-azure-ad-app-to-use-with-the-azure-media-services-api"></a>Создание приложения Azure AD для работы с API служб мультимедиа Azure с помощью PowerShell

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Узнайте, как с помощью сценария PowerShell создать приложение Azure Active Directory (Azure AD) и участник-службу для доступа к ресурсам служб мультимедиа Azure.  

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure. Если у вас нет учетной записи Azure, начните с получения [бесплатной пробной версии](https://azure.microsoft.com/pricing/free-trial/). 
- Учетная запись служб мультимедиа. Дополнительные сведения см. в статье [Создание учетной записи служб мультимедиа Azure с помощью портала Azure](media-services-portal-create-account.md).

- Установите Azure PowerShell. Дополнительные сведения см. в разделе [об использовании Azure PowerShell](/powershell/azure/).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="create-an-azure-ad-app-by-using-powershell"></a>Создание приложения Azure AD с помощью PowerShell  

```powershell
Connect-AzAccount
Import-Module Az.Resources
Set-AzContext -SubscriptionId $SubscriptionId
$ServicePrincipal = New-AzADServicePrincipal -DisplayName $ApplicationDisplayName -Password $Password

Get-AzADServicePrincipal -ObjectId $ServicePrincipal.Id 
$NewRole = $null
$Scope = "/subscriptions/your subscription id/resourceGroups/userresourcegroup/providers/microsoft.media/mediaservices/your media account"

$Retries = 0;While ($NewRole -eq $null -and $Retries -le 6)
{
    # Sleep here for a few seconds to allow the service principal application to become active (usually, it will take only a couple of seconds)
    Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $ServicePrincipal.ApplicationId -Scope $Scope | Write-Verbose -ErrorAction SilentlyContinue
    $NewRole = Get-AzRoleAssignment -ServicePrincipalName $ServicePrincipal.ApplicationId -ErrorAction SilentlyContinue
    $Retries++;
}
```

Дополнительные сведения см. в следующих статьях:

- [Использование Azure PowerShell для создания субъекта-службы и доступа к ресурсам](../../active-directory/develop/howto-authenticate-service-principal-powershell.md)
- [Добавление и удаление назначений ролей Azure с помощью Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
- [Authenticating to Azure AD in daemon apps with certificates - manual configuration steps](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) (Ручная настройка аутентификации приложений управляющей программы в Azure AD с помощью сертификатов)

## <a name="next-steps"></a>Дальнейшие действия

Приступите к [передаче файлов в учетную запись](media-services-portal-upload-files.md).

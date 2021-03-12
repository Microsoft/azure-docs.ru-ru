---
title: Краткое руководство. Развертывание Azure API для FHIR с помощью PowerShell
description: В этом кратком руководстве показано, как развернуть Azure API для FHIR с помощью PowerShell.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 10/15/2019
ms.author: cavoeg
ms.openlocfilehash: ca20346f4c2968eac5e7308cb05b59f968a1c63e
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103019607"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-powershell"></a>Краткое руководство. Развертывание Azure API для FHIR с помощью PowerShell

В этом кратком руководстве показано, как развернуть Azure API для FHIR с помощью PowerShell.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="register-the-azure-api-for-fhir-resource-provider"></a>Регистрация поставщика ресурсов Azure API для FHIR

Если поставщик ресурсов `Microsoft.HealthcareApis` еще не зарегистрирован для вашей подписки, его можно зарегистрировать, сделав следующее:

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

## <a name="create-azure-resource-group"></a>Создание группы ресурсов Azure

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroupName" -Location westus2
```

## <a name="deploy-azure-api-for-fhir"></a>Развертывание Azure API для FHIR

```azurepowershell-interactive
New-AzHealthcareApisService -Name nameoffhirservice -ResourceGroupName myResourceGroupName -Location westus2 -Kind fhir-R4
```

> [!NOTE]
> В зависимости от версии установленного модуля `Az` PowerShell подготовленный сервер FHIR можно настроить для использования [локального механизма RBAC](configure-local-rbac.md), чтобы вошедший в систему пользователь PowerShell был задан в списке допустимых идентификаторов объектов удостоверений для развернутой службы FHIR. В дальнейшем рекомендуется [использовать Azure RBAC](configure-azure-rbac.md) для назначения ролей плоскости данных. Возможно, после развертывания нужно будет удалить идентификатор объекта этого пользователя, чтобы включить режим Azure RBAC.


## <a name="fetch-capability-statement"></a>Получение выписки о возможностях

Вы сможете проверить, работает ли учетная запись API Azure для FHIR, получив выписку о возможностях FHIR:

```azurepowershell-interactive
$metadata = Invoke-WebRequest -Uri "https://nameoffhirservice.azurehealthcareapis.com/metadata"
$metadata.RawContent
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите группу ресурсов, выполнив следующие действия:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как развернуть Azure API для FHIR в своей подписке. Чтобы задать дополнительные параметры в Azure API для FHIR, см. соответствующее руководство. Если вы уже готовы к использованию API Azure для FHIR, ознакомьтесь с процессом регистрации приложений.

>[!div class="nextstepaction"]
>[Дополнительные параметры в Azure API для FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Общие сведения о регистрации приложений](fhir-app-registration.md)

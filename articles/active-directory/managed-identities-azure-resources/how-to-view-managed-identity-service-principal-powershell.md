---
title: Просмотр субъекта-службы управляемого удостоверения с помощью PowerShell — Azure AD
description: Пошаговое руководство по просмотру субъекта-службы управляемого удостоверения с помощью PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/30/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 15ccc0faa4d74a2ef95aca00a6257f27b9a209c3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91611951"
---
# <a name="view-the-service-principal-of-a-managed-identity-using-powershell"></a>Просмотр субъекта-службы управляемого удостоверения с помощью PowerShell

Функция управляемых удостоверений для ресурсов Azure предоставляет службам Azure автоматически управляемые удостоверения в Azure Active Directory. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде. 

В этой статье описано, как просмотреть субъект-службу управляемого удостоверения с помощью PowerShell.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

- Если вы не работали с управляемыми удостоверениями для ресурсов Azure, изучите [общие сведения](overview.md).
- Если у вас нет учетной записи Azure, зарегистрируйтесь для получения [бесплатной пробной версии](https://azure.microsoft.com/free/).
- [Включите назначенное системой удостоверение в виртуальной машине](./qs-configure-portal-windows-vm.md#system-assigned-managed-identity) или [приложении](../../app-service/overview-managed-identity.md#add-a-system-assigned-identity).
- Выполнить примеры скриптов можно двумя способами:
    - используйте службу [Azure Cloud Shell](../../cloud-shell/overview.md), которую можно открыть с помощью кнопки **Попробовать** в правом верхнем углу блоков кода.
    - Выполните скрипты локально, установив последнюю версию [Azure PowerShell](/powershell/azure/install-az-ps), а затем войдите в Azure с помощью команды `Connect-AzAccount`.

## <a name="view-the-service-principal"></a>Просмотр субъекта-службы

Следующая команда позволяет просмотреть субъект-службу виртуальной машины или приложения с включенным управляемым удостоверением, назначаемым системой. Замените `<Azure resource name>` собственными значениями.

```azurepowershell-interactive
Get-AzADServicePrincipal -DisplayName <Azure resource name>
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о просмотре субъектов-служб Azure AD с помощью PowerShell см. в разделе [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal).

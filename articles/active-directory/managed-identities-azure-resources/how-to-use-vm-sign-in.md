---
title: Использование управляемых удостоверений на виртуальной машине Azure для входа в Azure
description: Пошаговые инструкции и примеры использования удостоверений, управляемых виртуальной машиной Azure для субъекта-службы ресурсов Azure, для входа клиента в скрипт и доступа к ресурсам.
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
ms.date: 01/29/2021
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 61e83bd27c9434c4222e0161e3b643b183d1aa84
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99090966"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-for-sign-in"></a>Использование управляемых удостоверений для ресурсов Azure на виртуальной машине Azure для входа 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  
Эта статья содержит примеры сценариев PowerShell и интерфейса командной строки, позволяющих войти в систему с использованием субъекта-службы управляемых удостоверений для ресурсов Azure, а также рекомендации по некоторым важным вопросам, таким как обработка ошибок.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

Если вы планируете использовать примеры Azure PowerShell или CLI из этой статьи, убедитесь, что у вас установлена последняя версия [Azure PowerShell](/powershell/azure/install-az-ps) или [Azure CLI](/cli/azure/install-azure-cli). 

> [!IMPORTANT]
> - Во всех примерах сценариев в этой статье предполагается, что клиент командной строки выполняется на виртуальной машине с включенными управляемыми удостоверениями для ресурсов Azure. Используйте функцию подключения виртуальной машины на портале Azure для удаленного подключения к своей виртуальной машине. Дополнительные сведения о включении управляемых удостоверений для ресурсов Azure на виртуальной машине см. в статье [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью Azure CLI](qs-configure-portal-windows-vm.md) или в одном из вариантов статьи (с использованием PowerShell, интерфейса командной строки, шаблона или пакета SDK для Azure). 
> - Чтобы избежать ошибок во время доступа к ресурсам, для управляемого удостоверения виртуальной машины необходимо предоставить по меньшей мере роль "Читатель" в соответствующей области (на уровне виртуальной машины или выше), то есть разрешить операции Azure Resource Manager на виртуальной машине. Дополнительные сведения см. в статье [Назначение доступа на основе управляемого удостоверения службы для ресурса с помощью портала Azure](howto-assign-access-portal.md).

## <a name="overview"></a>Обзор

Управляемые удостоверения для ресурсов Azure предоставляют [объект субъекта-службы](../develop/developer-glossary.md#service-principal-object) , который [создается при включении управляемых удостоверений для ресурсов Azure](overview.md) на виртуальной машине. Субъекту-службе можно предоставить доступ к определенным ресурсам Azure. Также субъект-служба может использоваться в качестве удостоверения скриптами и клиентами командной строки для входа в систему и доступа к ресурсам. Чтобы клиентский скрипт мог обращаться к защищенным ресурсам с собственным удостоверением, обычно нужно выполнить следующие условия:  

   - зарегистрировать скрипт в качестве конфиденциального клиента приложения или приложения веб-клиента в Azure AD и получить соответствующее согласие;
   - выполнить вход из скрипта с соответствующим субъектом-службой, используя учетные данные приложения (обычно они встраиваются в скрипт).

Благодаря управляемым удостоверениям для ресурсов Azure вашему клиенту сценария больше не требуется делать ни то, ни другое, так как он может войти с помощью субъекта-службы управляемых удостоверений для ресурсов Azure. 

## <a name="azure-cli"></a>Azure CLI

Следующий сценарий демонстрирует, как:

1. войти в Azure AD с помощью субъекта-службы управляемых удостоверений для ресурсов Azure на виртуальной машине;  
2. вызвать Azure Resource Manager и получить идентификатор субъекта-службы виртуальной машины. CLI автоматически управляет получением и использованием маркера. Не забудьте указать правильное имя виртуальной машины вместо `<VM-NAME>`.  

   ```azurecli
   az login --identity
   
   spID=$(az resource list -n <VM-NAME> --query [*].identity.principalId --out tsv)
   echo The managed identity for Azure resources service principal ID is $spID
   ```

## <a name="azure-powershell"></a>Azure PowerShell

Следующий сценарий демонстрирует, как:

1. войти в Azure AD с помощью субъекта-службы управляемых удостоверений для ресурсов Azure на виртуальной машине;  
2. вызвать командлет Azure Resource Manager для получения сведений о виртуальной машине. PowerShell автоматически управляет использованием маркера.  

   ```azurepowershell
   Add-AzAccount -identity

   # Call Azure Resource Manager to get the service principal ID for the VM's managed identity for Azure resources. 
   $vmInfoPs = Get-AzVM -ResourceGroupName <RESOURCE-GROUP> -Name <VM-NAME>
   $spID = $vmInfoPs.Identity.PrincipalId
   echo "The managed identity for Azure resources service principal ID is $spID"
   ```

## <a name="resource-ids-for-azure-services"></a>Идентификаторы ресурсов для служб Azure

Список ресурсов, которые поддерживают Azure AD и были протестированы с управляемыми удостоверениями для ресурсов Azure и соответствующими идентификаторами ресурсов, приведен в разделе [Службы Azure, поддерживающие аутентификацию Azure AD](services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

## <a name="error-handling-guidance"></a>Руководство по обработке ошибок 

Такие ответы могут обозначать, что управляемые удостоверения для ресурсов Azure на виртуальной машины настроены неправильно:

- PowerShell: *Invoke-WebRequest : Unable to connect to the remote server* (Invoke-WebRequest: Unable to connect to the remote server)
- CLI: *MSI: не удалось получить маркер из- `http://localhost:50342/oauth2/token` за ошибки "HTTPConnectionPool (узел =" localhost ", порт = 50342)* 

Если вы получаете одну из этих ошибок, вернитесь на виртуальную машину Azure в [портал Azure](https://portal.azure.com) и перейдите на страницу **удостоверение** и убедитесь, что для параметра **назначено системой** задано значение "Да".

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы включить управляемые удостоверения для ресурсов Azure на виртуальной машине Azure, обратитесь к статье [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью PowerShell](qs-configure-powershell-windows-vm.md) или [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью Azure CLI](qs-configure-cli-windows-vm.md).
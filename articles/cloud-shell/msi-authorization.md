---
title: Использование управляемых удостоверений для ресурсов в Azure Cloud Shell
description: Аутентификация кода с помощью управляемых удостоверений службы в Azure Cloud Shell
services: azure
author: maertendMSFT
ms.author: damaerte
tags: azure-resource-manager
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 04/14/2018
ms.openlocfilehash: 0fb19524079f84e92e1ddbc98a61917026492663
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89469904"
---
# <a name="use-managed-identities-for-azure-resources-in-azure-cloud-shell"></a>Использование управляемых удостоверений для ресурсов Azure в Azure Cloud Shell

Azure Cloud Shell поддерживает авторизацию с помощью управляемых удостоверений службы для ресурсов Azure. Используя этот метод, вы можете получить маркеры доступа, чтобы безопасно подключаться к службам Azure.

## <a name="about-managed-identities-for-azure-resources"></a>Сведения об управляемых удостоверениях для ресурсов Azure
Распространенная проблема при создании облачных приложений — безопасное управление учетными данными, которые необходимы в коде для аутентификации в облачных службах. В Cloud Shell может возникнуть необходимость аутентифицировать получение учетных данных из Key Vault, которые могут требоваться в скрипте.

Управляемые удостоверения для ресурсов Azure упрощают решение этой задачи, предоставляя службам Azure автоматически управляемое удостоверение в Azure Active Directory (Azure AD). Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, включая Key Vault, не храня какие-либо учетные данные в коде.

## <a name="acquire-access-token-in-cloud-shell"></a>Получение маркера доступа в Cloud Shell

Чтобы задать маркер доступа MSI в качестве переменной среды, `access_token`, выполните такие команды:
```
response=$(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s)
access_token=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["access_token"])')
echo The MSI access token is $access_token
```

## <a name="handling-token-expiration"></a>Обработка срока действия маркера

Локальная подсистема MSI кэширует маркеры. Таким образом их можно вызывать столько, сколько нужно, а сетевой вызов Azure AD выполняется, только если:
- произошел промах кэша из-за отсутствия маркера в кэше;
- истек срок действия маркера.

Если маркер кэшируется в коде, необходимо подготовиться к обработке сценариев, в которых ресурс указывает, что срок действия маркера истек.

Сведения об обработке ошибок маркера см. [здесь](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md#error-handling).

## <a name="next-steps"></a>Дальнейшие действия
[Управляемое удостоверение службы (MSI) для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md)  
[Использование управляемого удостоверения службы (MSI) виртуальной машины Azure для получения маркера](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)
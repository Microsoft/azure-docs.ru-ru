---
title: Выполнение команд PowerShell с учетными данными Azure AD для доступа к данным большого двоичного объекта
titleSuffix: Azure Storage
description: PowerShell поддерживает вход с использованием учетных данных Azure AD для выполнения команд в данных BLOB-объектов в службе хранилища Azure. Маркер доступа предоставляется для сеанса и позволяет авторизовать вызов операций. Разрешения зависят от роли Azure, назначенной субъекту безопасности Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/10/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: blobs
ms.openlocfilehash: 45fa5cf4c76577cb5e8ba9bf482f4aab7301e3c6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100391496"
---
# <a name="run-powershell-commands-with-azure-ad-credentials-to-access-blob-data"></a>Выполнение команд PowerShell с учетными данными Azure AD для доступа к данным большого двоичного объекта

Служба хранилища Azure предоставляет расширения для PowerShell, которые позволяют входить в систему и выполнять команды сценариев с учетными данными Azure Active Directory (Azure AD). При входе в PowerShell с учетными данными Azure AD возвращается маркер доступа OAuth 2,0. Этот маркер автоматически используется PowerShell для авторизации последующих операций с данными в хранилище BLOB-объектов. Для поддерживаемых операций больше не требуется передавать ключ учетной записи или маркер SAS с помощью команды.

Вы можете назначать разрешения для данных больших двоичных объектов субъекту безопасности Azure AD через Управление доступом на основе ролей Azure (Azure RBAC). Дополнительные сведения о ролях Azure в службе хранилища Azure см. в статье [Управление правами доступа к данным службы хранилища Azure с помощью Azure RBAC](../common/storage-auth-aad-rbac-portal.md).

## <a name="supported-operations"></a>Поддерживаемые операции

Расширения службы хранилища Azure поддерживаются для операций с данными больших двоичных объектов. Действия, которые можно вызывать, зависят от разрешений, предоставленных субъекту безопасности Azure AD, с помощью которого вы входите в PowerShell. Разрешения для контейнеров службы хранилища Azure назначаются через Azure RBAC. Например, если вы назначили роль **читателя данных большого двоичного объекта** , то можете выполнять команды сценариев, считывающие данные из контейнера. Если вы назначили роль **участника данных большого двоичного объекта** , то можете выполнять команды сценариев, которые считывают, записывают или удаляют контейнер или содержащиеся в них данные.

Дополнительные сведения о разрешениях, необходимых для каждой операции службы хранилища Azure в контейнере, см. в разделе [операции хранилища вызовов с токенами OAuth](/rest/api/storageservices/authorize-with-azure-active-directory#call-storage-operations-with-oauth-tokens).  

> [!IMPORTANT]
> Если учетная запись хранения заблокирована с блокировкой Azure Resource Manager **только для чтения** , операция " [список ключей](/rest/api/storagerp/storageaccounts/listkeys) " не разрешена для этой учетной записи хранения. **Список ключей** является операцией POST, и все операции POST предотвращаются, если для учетной записи настроена блокировка **только для чтения** . По этой причине, если учетная запись заблокирована с блокировкой **только для чтения** , пользователи, которым не назначены ключи учетной записи, должны использовать учетные данные Azure AD для доступа к данным BLOB-объектов. В PowerShell включите параметр, `-UseConnectedAccount` чтобы создать объект **AzureStorageContext** с учетными данными Azure AD.

## <a name="call-powershell-commands-using-azure-ad-credentials"></a>Вызов команд PowerShell с использованием учетных данных Azure AD

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Чтобы использовать Azure PowerShell для входа и выполнения последующих операций в службе хранилища Azure с помощью учетных данных Azure AD, создайте контекст хранилища для ссылки на учетную запись хранения и включите `-UseConnectedAccount` параметр.

В следующем примере показано, как создать контейнер в новой учетной записи хранения из Azure PowerShell с помощью учетных данных Azure AD. Не забудьте заменить значения заполнителей в угловых скобках собственными значениями.

1. Войдите в свою учетную запись Azure с помощью команды [Connect-азаккаунт](/powershell/module/az.accounts/connect-azaccount) :

    ```powershell
    Connect-AzAccount
    ```

    Дополнительные сведения о входе в Azure с помощью PowerShell см. [в разделе Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).

1. Создайте группу ресурсов Azure, вызвав командлет [New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup). 

    ```powershell
    $resourceGroup = "sample-resource-group-ps"
    $location = "eastus"
    New-AzResourceGroup -Name $resourceGroup -Location $location
    ```

1. Создайте учетную запись хранения, вызвав командлет [New-азсторажеаккаунт](/powershell/module/az.storage/new-azstorageaccount).

    ```powershell
    $storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroup `
      -Name "<storage-account>" `
      -SkuName Standard_LRS `
      -Location $location `
    ```

1. Получите контекст учетной записи хранения, указывающий новую учетную запись хранения, вызвав [New-азсторажеконтекст](/powershell/module/az.storage/new-azstoragecontext). При работе с учетной записью хранения можно ссылаться на контекст, а не повторять передачу учетных данных. Включите `-UseConnectedAccount` параметр для вызова любых последующих операций с данными с использованием учетных данных Azure AD:

    ```powershell
    $ctx = New-AzStorageContext -StorageAccountName "<storage-account>" -UseConnectedAccount
    ```

1. Прежде чем создавать контейнер, назначьте себе роль [Участник данных в хранилище BLOB-объектов](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor). Несмотря на то, что вы являетесь владельцем учетной записи, вам нужны явные разрешения на выполнение операций с данными с учетной записью хранения. Дополнительные сведения о назначении ролей Azure см. в статье [использование портал Azure для назначения роли Azure доступа к данным BLOB-объектов и очередей](../common/storage-auth-aad-rbac-portal.md).

    > [!IMPORTANT]
    > Назначение ролей Azure может занимать несколько минут.

1. Создайте контейнер, вызвав [New-азсторажеконтаинер](/powershell/module/az.storage/new-azstoragecontainer). Так как этот вызов использует контекст, созданный на предыдущих шагах, контейнер создается с использованием учетных данных Azure AD.

    ```powershell
    $containerName = "sample-container"
    New-AzStorageContainer -Name $containerName -Context $ctx
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Назначение роли Azure для доступа к данным BLOB-объектов и очередей с помощью PowerShell](../common/storage-auth-aad-rbac-powershell.md)
- [Авторизация доступа к данным BLOB-объектов и очередей с помощью управляемых удостоверений для ресурсов Azure](../common/storage-auth-aad-msi.md)
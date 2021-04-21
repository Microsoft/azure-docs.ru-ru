---
title: Получение маркеров подписанного URL-адреса в коде | Azure Key Vault
description: Функция управляемой учетной записи хранения обеспечивает простую интеграцию между Azure Key Vault и учетной записью хранения Azure. В этом примере для управления маркерами SAS используется пакет Azure SDK для .NET.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/10/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 8c8a7b357efc2e30d1ca93d0b45fd2563c980045
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752756"
---
# <a name="create-sas-definition-and-fetch-shared-access-signature-tokens-in-code"></a>Создание определения SAS и получение маркеров SAS в коде

Вы можете управлять учетной записью хранения с помощью маркеров подписанного URL-адреса (SAS), хранящихся в хранилище ключей. Дополнительные сведения см. в статье [Предоставление ограниченного доступа к ресурсам службы хранилища Azure с помощью подписанных URL-адресов (SAS)](../../storage/common/storage-sas-overview.md).

> [!NOTE]
> Мы рекомендуем защитить учетную запись хранения с помощью [управления доступом на основе ролей Azure (Azure RBAC)](../../storage/common/storage-auth-aad.md), чтобы усилить защиту и упростить авторизацию с использованием общего ключа.

В этой статье представлены примеры кода .NET, который создает определение SAS и извлекает маркеры SAS. Полные сведения, включая созданный клиент для учетных записей хранения, управляемых Key Vault, см. в нашем примере [ShareLink](/samples/azure/azure-sdk-for-net/share-link/). Сведения о создании и хранении маркеров SAS см. в статьях об управлении ключами учетной записи хранения с помощью [Key Vault и Azure CLI](overview-storage-keys.md) или с помощью [Key Vault и Azure PowerShell](overview-storage-keys-powershell.md).

## <a name="code-samples"></a>Примеры кода

В следующем примере показано, как создать шаблон SAS:

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="91-97":::

Используя этот шаблон, мы можем создать определение SAS с помощью 

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="137-156":::

Создав определение SAS, вы сможете получать маркеры SAS, например секреты, с помощью `SecretClient`. Перед именем секрета необходимо указать имя учетной записи хранения и тире:

:::code language="csharp" source="~/azure-sdk-for-net/sdk/keyvault/samples/sharelink/Program.cs" range="52-58":::

Если срок действия маркера подписанного URL-адреса истекает, можно снова получить тот же секрет, чтобы создать новый.

Сведения об использовании полученных данных из маркера SAS Key Vault для доступа к службам хранилища Azure см. в разделе [Использование SAS учетной записи из клиента](../../storage/common/storage-account-sas-create-dotnet.md#use-an-account-sas-from-a-client).

> [!NOTE]
> Приложение должно быть подготовлено к обновлению SAS на тот случай, если оно получит ошибку 403 от службы хранилища. Это требуется для тех ситуаций, когда ключ скомпрометирован и вам нужно сменить его быстрее, чем обычно. 

## <a name="next-steps"></a>Дальнейшие действия
- [Предоставление ограниченного доступа к ресурсам службы хранилища Azure с помощью подписанных URL-адресов (SAS)](../../storage/common/storage-sas-overview.md)
- Управление ключами учетной записи хранения с помощью [Key Vault и Azure CLI](overview-storage-keys.md) или с помощью [Key Vault и Azure PowerShell](overview-storage-keys-powershell.md)
- [Примеры ключей управляемой учетной записи хранения](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)
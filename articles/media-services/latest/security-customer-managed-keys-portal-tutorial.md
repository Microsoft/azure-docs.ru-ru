---
title: Использование ключей, управляемых клиентом, или портала BYOK
description: В рамках этого учебника используйте портал Azure для включения ключей, управляемых клиентом, или создавайте собственные ключи (BYOK) для учетной записи хранения Служб мультимедиа.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: tutorial
ms.date: 10/18/2020
ms.openlocfilehash: e34649162c7a30d4ab43aa068d2c864b5c2d90e0
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106281097"
---
# <a name="tutorial-use-the-azure-portal-to-use-customer-managed-keys-or-byok-with-media-services"></a>Руководство по использованию портала Azure для использования ключей, управляемых клиентом, или BYOK для Служб мультимедиа

API версии от 01 мая 2020 года позволяет использовать управляемый клиентом ключ RSA для учетной записи Служб мультимедиа Azure, у которой есть управляемое системой удостоверение. В этом руководстве рассматриваются действия, описанные в портале Azure.

Используемые службы:

- Хранилище Azure
- Azure Key Vault
- Службы мультимедиа Azure

Из этого руководства вы узнаете, как с помощью портала Azure выполнять следующие операции.

> [!div class="checklist"]
> - Создайте группу ресурсов.
> - Создание учетной записи с управляемым системой удостоверением:
> - создание учетной записи Служб мультимедиа с удостоверением, управляемым системой;
> - создание Key Vault для хранения управляемого пользователем ключа RSA;

## <a name="prerequisites"></a>Предварительные требования

Подписка Azure.

Если у вас еще нет подписки Azure, создайте [бесплатную пробную учетную запись](https://azure.microsoft.com/free/).

## <a name="system-managed-keys"></a>Ключи, управляемые системой

<!-- Create a resource group -->
[!INCLUDE [create a resource group in the portal](./includes/task-create-resource-group-portal.md)]

> [!IMPORTANT]
> Для следующих шагов создания учетной записи хранения необходимо выбрать управляемый системой ключ, в разделе "Дополнительные параметры".

<!-- Create a media services account -->

[!INCLUDE [create a media services account in the portal](./includes/task-create-media-services-account-portal.md)]

<!-- Create a key vault -->

[!INCLUDE [create a key vaultl](./includes/task-create-key-vault-portal.md)]

<!-- Enable CMK BYOK on the account -->
[!INCLUDE [enable CMK](./includes/task-enable-cmk-byok-portal.md)]

> [!IMPORTANT]
> Для следующих действий по шифрованию хранилища, необходимо выбрать **ключ, управляемый клиентом**.

<!-- Set encryption for storage account -->
[!INCLUDE [Set encryption for storage account](./includes/task-set-storage-encryption-portal.md)]

## <a name="change-the-key"></a>Изменение ключа

Службы мультимедиа автоматически обнаруживают, что ключ изменен. НЕОБЯЗАТЕЛЬНО. Чтобы проверить это, создайте версию того же ключа. Службы мультимедиа должны обнаружить, что ключ изменен.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать (и *оплачивать*) созданные ресурсы, удалите их.

## <a name="next-steps"></a>Дальнейшие шаги

Дополнительные сведения см. в следующей статье:
> [!div class="nextstepaction"]
> [Кодирование удаленного файла на основе URL-адреса и потоковой передачи видео с помощью REST](stream-files-tutorial-with-rest.md)

---
title: Content Moderator шифрование неактивных данных
titleSuffix: Azure Cognitive Services
description: Content Moderator шифрование неактивных данных.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/13/2020
ms.author: egeaney
ms.openlocfilehash: 1401108a594e30790e842ec379724603f11d493f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100524552"
---
# <a name="content-moderator-encryption-of-data-at-rest"></a>Content Moderator шифрование неактивных данных

Content Moderator автоматически шифрует данные при сохранении в облаке, помогая обеспечить безопасность и соответствие требованиям Организации.

[!INCLUDE [cognitive-services-about-encryption](../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> Ключи, управляемые клиентом, доступны только в ценовой категории E0. Чтобы запросить возможность использования ключей, управляемых клиентом, заполните и отправьте [Content Moderator Customer-Managed форму запроса ключа](https://aka.ms/cogsvc-cmk). Для получения сведений о состоянии вашего запроса потребуется около 3-5 рабочих дней. В зависимости от спроса вы можете поместить в очередь и утвердить, как только пространство станет доступным. После утверждения для использования CMK со службой Content Moderator необходимо создать новый ресурс Content Moderator и выбрать в качестве ценовой категории значение E0. После создания ресурса Content Moderator с ценовой категорией "E0" можно использовать Azure Key Vault для настройки управляемого удостоверения.

[!INCLUDE [cognitive-services-cmk](../includes/cognitive-services-cmk-regions.md)]

[!INCLUDE [cognitive-services-cmk](../includes/configure-customer-managed-keys.md)]

## <a name="enable-data-encryption-for-your-content-moderator-team"></a>Включение шифрования данных для команды Content Moderator

Чтобы включить шифрование данных для группы Content Moderatorной проверки, см. раздел Краткое руководство по началу работы [Content Moderator в Интернете](quick-start.md#create-a-review-team).  

> [!NOTE]
> Вам потребуется указать _идентификатор ресурса_ в ценовой категории Content Moderator E0.

## <a name="next-steps"></a>Дальнейшие действия

* Полный список служб, поддерживающих CMK, см. в разделе [управляемые клиентом ключи для Cognitive Services](../encryption/cognitive-services-encryption-keys-portal.md)
* [Об Azure Key Vault](../../key-vault/general/overview.md)
* [Форма запроса ключа Cognitive Services Customer-Managed](https://aka.ms/cogsvc-cmk)
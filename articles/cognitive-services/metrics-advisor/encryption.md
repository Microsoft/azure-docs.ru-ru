---
title: Шифрование службы советника по метрикам
titleSuffix: Azure Cognitive Services
description: Шифрование неактивных данных в службе советника по метрикам.
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: conceptual
ms.date: 09/10/2020
ms.author: mbullwin
ms.openlocfilehash: 5d41500a9c53e38cd36f0feba602e0e1baa5da2c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92909749"
---
# <a name="metrics-advisor-service-encryption-of-data-at-rest"></a>Шифрование неактивных данных в службе советника по метрикам

Служба советника по метрикам автоматически шифрует данные при их сохранении в облаке. Шифрование службы Advisor обеспечивает защиту данных и помогает удовлетворить ваши обязательства по обеспечению безопасности и соответствия требованиям Организации.

[!INCLUDE [cognitive-services-about-encryption](../includes/cognitive-services-about-encryption.md)]

> [!IMPORTANT]
> Ключи, управляемые клиентом, доступны только в ценовой категории E0. Чтобы запросить возможность использования ключей, управляемых клиентом, заполните и отправьте [форму запроса ключа службы Advisor Customer-Managed ключ](https://aka.ms/cogsvc-cmk). Для получения сведений о состоянии вашего запроса потребуется около 3-5 рабочих дней. В зависимости от спроса вы можете поместить в очередь и утвердить, как только пространство станет доступным. После утверждения для использования CMK со службой "помощник по метрикам" необходимо создать новый ресурс советника по метрикам и выбрать параметр "E0" в качестве ценовой категории. После создания ресурса советника по метрикам с ценовой категорией E0 можно использовать Azure Key Vault для настройки управляемого удостоверения.

[!INCLUDE [cognitive-services-cmk](../includes/cognitive-services-cmk-regions.md)]

[!INCLUDE [cognitive-services-cmk](../includes/configure-customer-managed-keys.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Форма запроса ключа Customer-Managed службы Advisor](https://aka.ms/cogsvc-cmk)
* [Дополнительные сведения об Azure Key Vault](../../key-vault/general/overview.md)
---
title: включить файл
description: включить файл
author: rothja
ms.service: azure-resource-manager
ms.topic: include
ms.date: 05/18/2018
ms.author: jroth
ms.custom: include file
ms.openlocfilehash: 74f9c5304237ec77a629bc914d95c345882b784f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95560709"
---
| Ресурс | Ограничение по умолчанию | Максимальное ограничение |
| --- | --- | --- |
| Число виртуальных ЦП на [подписку](https://azure.microsoft.com/pricing/)<sup>1</sup> |20 |10 000 |
| [Соадминистраторы](../articles/cost-management-billing/manage/add-change-subscription-administrator.md) на подписку |200 |200 |
| [Учетных записей хранения](../articles/storage/common/storage-account-create.md) на подписку<sup>2</sup> |100 |100 |
| [Облачные службы](../articles/cloud-services/cloud-services-choose-me.md) на подписку |20 |200 |
| [Локальные сети](/previous-versions/azure/reference/jj157100(v=azure.100)) на подписку |10 |500 |
| Серверы DNS на подписку |9 |100 |
| Зарезервированные IP на подписку |20 |100 |
| [Территориальные группы](/previous-versions/azure/virtual-network/virtual-networks-migrate-to-regional-vnet) на подписку |256 |256 |
| Длина имени подписки (в символах) | 64 | 64 |

<sup>1</sup> Сверхмалые экземпляры считаются как один виртуальный ЦП в контексте ограничения числа виртуальных ЦП, даже если ядро ЦП загружено не полностью.

<sup>2</sup>Ограничение учетной записи хранения распространяются на учетные записи хранения класса "Стандартный" и "Премиум".
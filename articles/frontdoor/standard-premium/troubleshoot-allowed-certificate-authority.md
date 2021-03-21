---
title: Разрешенные центры сертификации для передней дверцы Azure уровня "Стандартный" или "Премиум" (Предварительная версия)
description: В этой статье перечислены все центры сертификации, разрешенные при создании собственного сертификата.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: troubleshooting
ms.date: 02/18/2021
ms.author: qixwang
ms.openlocfilehash: e31ad0734630fbd0b4960c26f8d562cea66ae675
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102434872"
---
# <a name="allowed-certificate-authorities-for-azure-front-door-standardpremium-preview"></a>Разрешенные центры сертификации для передней дверцы Azure уровня "Стандартный" или "Премиум" (Предварительная версия)

При включении функции HTTPS с помощью собственного сертификата для пользовательского домена уровня "Стандартный" или "Премиум" основной дверцы Azure. Для создания сертификата TLS/SSL требуется разрешенный центр сертификации (ЦС). В противном случае при использовании недопустимого ЦС или самозаверяющего сертификата ваш запрос будет отклонен.

[!INCLUDE [cdn-front-door-allowed-ca](../../../includes/cdn-front-door-allowed-ca.md)]

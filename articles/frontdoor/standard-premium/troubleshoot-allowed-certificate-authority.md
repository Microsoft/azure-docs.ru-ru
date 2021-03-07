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
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102434872"
---
# <a name="allowed-certificate-authorities-for-azure-front-door-standardpremium-preview"></a>Разрешенные центры сертификации для передней дверцы Azure уровня "Стандартный" или "Премиум" (Предварительная версия)

При включении функции HTTPS с помощью собственного сертификата для пользовательского домена уровня "Стандартный" или "Премиум" основной дверцы Azure. Для создания сертификата TLS/SSL требуется разрешенный центр сертификации (ЦС). В противном случае при использовании недопустимого ЦС или самозаверяющего сертификата ваш запрос будет отклонен.

[!INCLUDE [cdn-front-door-allowed-ca](../../../includes/cdn-front-door-allowed-ca.md)]

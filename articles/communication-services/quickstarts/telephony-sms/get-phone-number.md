---
title: Краткое руководство. Управление номерами телефонов с помощью Служб коммуникации Azure
description: Узнайте, как управлять номерами телефонов с помощью Служб коммуникации Azure.
author: prakulka
manager: nmurav
services: azure-communication-services
ms.author: prakulka
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.custom: references_regions
zone_pivot_groups: acs-azp-java-net-python-csharp-js
ms.openlocfilehash: 19bb79f9a4deaebfacc75918c46a5516d2d398be
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107498205"
---
# <a name="quickstart-manage-phone-numbers"></a>Краткое руководство. Управление номерами телефонов

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

[!INCLUDE [Bulk Acquisition Instructions](../../includes/phone-number-special-order.md)]

::: zone pivot="platform-azp"
[!INCLUDE [Azure portal](./includes/phone-numbers-portal.md)]
::: zone-end

::: zone pivot="programming-language-csharp"
[!INCLUDE [Azure portal](./includes/phone-numbers-net.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/phone-numbers-java.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/phone-numbers-python.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/phone-numbers-js.md)]
::: zone-end

## <a name="troubleshooting"></a>Диагностика

Распространенные вопросы и проблемы.

- Покупка телефона поддерживается только в США. Чтобы приобрести номера телефонов, убедитесь, что:
  - Соответствующий адрес для выставления счетов подписки Azure находится в США. Сейчас вы не можете переместить ресурс в другую подписку.
  - Ресурс служб коммуникации Azure подготавливается в расположении данных в США. Сейчас вы не можете переместить ресурс в другое расположение данных.

- При удалении номер телефона не будет освобожден или доступен для повторного приобретения до окончания периода выставления счетов.

- При удалении ресурса Служб коммуникации связанные с ним номера телефонов немедленно автоматически освобождаются.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * приобрести номер телефона;
> * управлять номером телефона;
> * освободить номер телефона.

> [!div class="nextstepaction"]
> [Отправка SMS](../telephony-sms/send.md)
> [Начало работы с функцией вызова](../voice-video-calling/getting-started-with-calling.md)

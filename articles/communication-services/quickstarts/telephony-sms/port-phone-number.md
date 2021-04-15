---
title: Краткое руководство. Перенос номера телефона в Службы коммуникации Azure
description: Сведения о переносе номера телефон в ресурс Служб коммуникации
author: mikben
manager: mikben
services: azure-communication-services
ms.author: mikben
ms.date: 03/20/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.custom: references_regions
ms.openlocfilehash: b4357b8d4fe1d7184fc2dcfab2008dc6e0f334fe
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729910"
---
# <a name="quickstart-port-a-phone-number"></a>Краткое руководство. Перенос номера телефона

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Приступите к работе со Службами коммуникации Azure путем переноса номера телефона в ресурс Служб коммуникации Azure. Для переноса подходят бесплатные и географические номера, зарегистрированные в США. Дополнительные сведения о типах телефонных номеров см. в [основной документации по номерам телефона](../../concepts/telephony-sms/plan-solution.md).

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Активный ресурс Служб коммуникации.](../create-communication-resource.md)

## <a name="gather-your-azure-resource-details"></a>Сбор сведений о ресурсах Azure

Перед инициированием запроса на перенос перейдите на портал Azure и выберите ресурс Служб коммуникации. Выберите область `Overview` и щелкните ссылку `JSON View` в правом верхнем углу:

:::image type="content" source="./media/number-port/json-view.png" alt-text="Снимок экрана: выбор представления JSON.":::

Запишите **Идентификатор Azure** и **Неизменяемый идентификатор ресурса** для своего ресурса:

:::image type="content" source="./media/number-port/json-properties.png" alt-text="Снимок экрана: выбор свойств JSON.":::

## <a name="initiate-the-port-request"></a>Инициирование запроса на перенос

Для переноса подходят бесплатные и географические номера, зарегистрированные в США. Чтобы отправить запрос на перенос, используйте одну из следующих форм:

- Для бесплатных номеров: [Запрос на перенос бесплатного номера](https://aka.ms/acs-port-form-tollfree)
- Для географических номеров, зарегистрированных в США: [Запрос на перенос географического номера](https://aka.ms/acs-port-form-geographic)

После завершения отправьте заполненную форму запроса на перенос по адресу acsporting@microsoft.com. Убедитесь, что строка темы электронного письма начинается с "ACS Port-In Request" (Запрос на перенос в ACS).

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как выполнить следующие действия:

> [!div class="checklist"]
> * Получение метаданных для ресурса Служб коммуникации
> * Отправьте запрос на перенос номера телефона

> [!div class="nextstepaction"]
> [Отправка SMS](../telephony-sms/send.md)
> [Начало работы с функцией вызова](../voice-video-calling/getting-started-with-calling.md)

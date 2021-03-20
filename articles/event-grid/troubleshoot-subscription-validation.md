---
title: Сетка событий Azure. Устранение неполадок при проверке подписки
description: В этой статье описывается, как устранять неполадки при проверках подписок.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 857760182675d5673a3b09495c2faaf7372a4164
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94592946"
---
# <a name="troubleshoot-azure-event-grid-subscription-validations"></a>Устранение неполадок при проверках подписок
Если при создании подписки на событие отображается сообщение об ошибке `The attempt to validate the provided endpoint https://your-endpoint-here failed. For more details, visit https://aka.ms/esvalidation` , например, это означает, что в подтверждении проверки произошла ошибка. Чтобы устранить эту ошибку, проверьте следующие аспекты:

- Выполните запрос HTTP POST к URL-адресу перехватчика с примером текста запроса [SubscriptionValidationEvent](webhook-event-delivery.md#validation-details) с помощью инструкции POST, или аналогичного средства.
- Если веб-перехватчик реализует механизм подтверждения синхронной проверки, убедитесь, что ValidationCode возвращается как часть ответа.
- Если веб-перехватчик реализует механизм подтверждения по асинхронной проверке, убедитесь, что HTTP POST возвращает значение 200 OK.
- Если веб-перехватчик возвращает `403 (Forbidden)` ответ, проверьте, находится ли веб-перехватчик за шлюзом приложений Azure или брандмауэром веб-приложения. Если это так, вам нужно отключить эти правила брандмауэра и выполнить HTTP-запрос POST еще раз:
    - 920300 (в запросе отсутствует заголовок Accept)
    - 942430 (ограничение на обнаружение аномалий символов SQL (args): Превышено число специальных символов (12))
    - 920230 (обнаружено несколько кодировок URL-адресов)
    - 942130 (атака путем внедрения кода SQL: обнаружен SQL таутологи.)
    - 931130 (возможная атака с удаленными файлами (RFI) = не является ссылкой на домен или ссылкой)

> [!IMPORTANT]
> Подробные сведения о проверке конечных точек для веб-перехватчиков см. в разделе [Доставка событий веб-перехватчика](webhook-event-delivery.md).

В следующих разделах показано, как проверить подписки на события с помощью POST и перелистывания.  

## <a name="validate-event-grid-event-subscription-using-postman"></a>Проверка подписки на события Сетки событий с помощью Postman
Ниже приведен пример использования Postman для проверки подписки веб-перехватчика на события Сетки событий. 

![Проверка подписки на события Сетки событий с помощью Postman](./media/troubleshoot-subscription-validation/event-subscription-validation-postman.png)

Ниже приведен пример кода JSON **SubscriptionValidationEvent**.

```json
[
  {
    "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
    "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "subject": "",
    "data": {
      "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6",
    },
    "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
    "eventTime": "2018-01-25T22:12:19.4556811Z",
    "metadataVersion": "1",
    "dataVersion": "1"
  }
]
```

Вот пример успешного ответа.

```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

Чтобы узнать больше о проверке событий Сетки событий для веб-перехватчиков, ознакомьтесь с разделом [Проверка конечной точки с событиями сетки событий](webhook-event-delivery.md#endpoint-validation-with-event-grid-events).


## <a name="validate-event-grid-event-subscription-using-curl"></a>Проверка подписки на события Сетки событий с помощью Curl 
Ниже приведен пример использования Curl для проверки подписки веб-перехватчика на события Сетки событий. 

```bash
curl -X POST -d '[{"id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66","topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx","subject": "","data": {"validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"},"eventType": "Microsoft.EventGrid.SubscriptionValidationEvent","eventTime": "2018-01-25T22:12:19.4556811Z", "metadataVersion": "1","dataVersion": "1"}]' -H 'Content-Type: application/json' https://{your-webhook-url.com}
```

## <a name="validate-cloud-event-subscription-using-postman"></a>Проверка подписки на облачные события с помощью Postman
Ниже приведен пример использования Postman для проверки подписки веб-перехватчика на облачные события. 

![Проверка подписки на облачные события с помощью Postman](./media/troubleshoot-subscription-validation/cloud-event-subscription-validation-postman.png)

Используйте метод **HTTP OPTIONS** для проверки облачных событий. Чтобы узнать больше о проверке облачных событий для веб-перехватчиков, ознакомьтесь с разделом о [проверке конечных точек с облачными событиями](webhook-event-delivery.md#endpoint-validation-with-event-grid-events).

## <a name="troubleshoot-event-subscription-validation"></a>Устранение неполадок при проверке подписки на события

## <a name="next-steps"></a>Дальнейшие действия
Если вам нужна дополнительная помощь, опубликуйте свой вопрос на [форуме Stack Overflow](https://stackoverflow.com/questions/tagged/azure-eventgrid) или отправьте [запрос в службу поддержки](https://azure.microsoft.com/support/options/). 

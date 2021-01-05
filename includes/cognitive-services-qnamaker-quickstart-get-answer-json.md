---
title: включить файл
description: включить файл
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: include
ms.custom: include file
ms.date: 12/29/2020
ms.author: mrbullwin
ms.openlocfilehash: 8269cd3905ec7ea3ad0ff2ab6bfa31cc95da5b4f
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/30/2020
ms.locfileid: "97820580"
---
## <a name="request-and-response-json"></a>Запрос и ответ в формате JSON

Пример вопроса для REST API в формате JSON:

```json
{
    "question": "Is the QnA Maker Service free?",
    "top": 3
}
```

Вопрос включает свойство, позволяющее вернуть первые три ответа. 

Ответ возвращается в объекте JSON:

```json
{
  "answers": [
    {
      "questions": [
        "How do I embed the QnA Maker service in my website?"
      ],
      "answer": "Follow these steps to embed the QnA Maker service as a web-chat control in your website:\n\n\n1.  Create your FAQ bot by following the instructions [here](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/create-qna-bot).\n2.  Enable the web chat by following the steps [here](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat)",
      "score": 70.95,
      "id": 16,
      "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
      "metadata": []
    },
    {
      "questions": [
        "Do I need to use Bot Framework in order to use QnA Maker?"
      ],
      "answer": "No, you do not need to use the Bot Framework with QnA Maker. However, QnA Maker is offered as one of several templates in Azure Bot Service. Bot Service enables rapid intelligent bot development through Microsoft Bot Framework, and it runs in a server-less environment.",
      "score": 46.94,
      "id": 14,
      "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
      "metadata": []
    },
    {
      "questions": [
        "How can I create a bot with QnA Maker?"
      ],
      "answer": "Follow the instructions in [this](https://docs.microsoft.com/azure/cognitive-services/qnamaker/tutorials/create-qna-bot)documentation to create your Bot with Azure Bot Service.",
      "score": 43.25,
      "id": 15,
      "source": "https://docs.microsoft.com/azure/cognitive-services/qnamaker/faqs",
      "metadata": []
    }
  ]
}
```

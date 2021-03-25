---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/23/2019
ms.author: glenga
ms.openlocfilehash: 8bdd8b9d900cc50fdeb34ff7d233ac4d7e17a45c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78191080"
---
Обновите *HttpExample\\\_\_init\_\_.py* в соответствии с кодом ниже, добавив параметр `msg` в определение функции и `msg.set(name)` для оператора `if name:`.

:::code language="python" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/__init__.py":::

Параметр `msg` — это экземпляр [`azure.functions.InputStream class`](/python/api/azure-functions/azure.functions.httprequest). Его метод `set` записывает строковое сообщение в очередь. В этом случае имя передается функции в строке запроса URL-адреса.

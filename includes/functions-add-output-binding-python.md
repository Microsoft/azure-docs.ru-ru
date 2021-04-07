---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/23/2019
ms.author: glenga
ms.openlocfilehash: 8bdd8b9d900cc50fdeb34ff7d233ac4d7e17a45c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "78191080"
---
Обновите *HttpExample\\\_\_init\_\_.py* в соответствии с кодом ниже, добавив параметр `msg` в определение функции и `msg.set(name)` для оператора `if name:`.

:::code language="python" source="~/functions-docs-python/functions-add-output-binding-storage-queue-cli/HttpExample/__init__.py":::

Параметр `msg` — это экземпляр [`azure.functions.InputStream class`](/python/api/azure-functions/azure.functions.httprequest). Его метод `set` записывает строковое сообщение в очередь. В этом случае имя передается функции в строке запроса URL-адреса.

---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 10/18/2020
ms.author: glenga
ms.openlocfilehash: d1931614356a313334d712713965346e843a403d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93424927"
---
## <a name="invoke-the-function-on-azure"></a>Вызов функции в Azure

Функция использует триггер HTTP, поэтому ее необходимо вызывать через HTTP-запрос по URL-адресу в браузере или с помощью такого средства, как cURL. 

# <a name="browser"></a>[Браузер](#tab/browser)

Скопируйте полный URL-адрес вызова **Invoke URL**, показанный в выходных данных команды publish, в адресную строку браузера, добавив параметр запроса `&name=Functions`. В браузере должны отображаться выходные данные, аналогичные данным при локальном запуске функции.

![Выходные данные функции, выполняемой в Azure в браузере](./media/functions-run-remote-azure-cli/function-test-cloud-browser.png)

# <a name="curl"></a>[curl](#tab/curl)

Запустите [`curl`](https://curl.haxx.se/), используя **Invoke URL** и добавив параметр `&name=Functions`. Результатом выполнения этой команды должен быть текст Hello Functions.

![Выходные данные функции, выполненной в Azure в cURL](./media/functions-run-remote-azure-cli/function-test-cloud-curl.png)

---

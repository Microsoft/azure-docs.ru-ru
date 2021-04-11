---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/06/2020
ms.author: glenga
ms.openlocfilehash: bff2f05a95faf9c475189cb5a8003cb7fd9f69be
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101701408"
---
1. Чтобы запустить функцию, нажмите клавишу <kbd>F5</kbd> в Visual Studio. Возможно, вам потребуется включить исключение брандмауэра, чтобы инструменты могли обрабатывать HTTP-запросы. Уровни авторизации никогда не применяются при запуске функции в локальной среде.

2. Скопируйте URL-адрес функции из выходных данных среды выполнения функций Azure.

    ![Локальная среда выполнения Azure](./media/functions-run-function-test-local-vs/functions-debug-local-vs.png)

3. Вставьте URL-адрес для HTTP-запроса в адресной строке браузера. Добавьте строку запроса `?name=<YOUR_NAME>` в этот URL-адрес и выполните запрос. Ниже показан ответ в браузере на локальный запрос GET, возвращенный функцией: 

    ![Ответ функции localhost в браузере](./media/functions-run-function-test-local-vs/functions-run-browser-local-vs.png)

4. Чтобы остановить отладку, нажмите клавиши <kbd>Shift</kbd>+<kbd>F5</kbd> в Visual Studio.

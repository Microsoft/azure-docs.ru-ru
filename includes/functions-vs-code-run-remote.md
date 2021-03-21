---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 01/28/2021
ms.author: glenga
ms.openlocfilehash: 121b10cc568cce089c90e66b9c6f292c74f4acbe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99809575"
---
## <a name="run-the-function-in-azure"></a>Запуск функции в Azure

1. Вернитесь в область **Azure: Functions** (Azure: Функции) на боковой панели, разверните подписку, выберите новое приложение-функцию и **Функции**. Щелкните правой кнопкой мыши (в Windows) или используйте <kbd>Ctrl-</kbd>щелчок (в macOS) на функции `HttpExample` и выберите **Выполнить функцию...** .

    :::image type="content" source="media/functions-vs-code-run-remote/execute-function-now.png" alt-text="Вариант &quot;Выполнить функцию&quot; в Azure из Visual Studio Code":::

1. В поле **Ввести текст запроса** вы увидите значение текста запроса `{ "name": "Azure" }`. Нажмите клавишу ВВОД, чтобы отправить это сообщение запроса в свою функцию.  

1. При выполнении функции в Azure и возврате ответа в Visual Studio Code отобразится уведомление.

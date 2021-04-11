---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 01/28/2021
ms.author: glenga
ms.openlocfilehash: eae828d03431dd339c5399d8db8c6e46141ab11b
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106075357"
---
## <a name="run-the-function-locally"></a>Локальное выполнение функции

Visual Studio Code интегрируется с [Azure Functions Core Tools](../articles/azure-functions/functions-run-local.md), чтобы перед публикацией в Azure можно было запустить этот проект на локальном компьютере разработки.

1. Чтобы вызвать функцию, нажмите клавишу <kbd>F5</kbd>, чтобы запустить проект приложения-функции. Выходные данные основных инструментов отображаются на панели **Terminal** (Терминал). Ваше приложение запускается в панели **Терминал**. Отобразится URL-адрес конечной точки активируемой HTTP-запросом функции, которая выполняется локально.

    ![Выходные данные VS Code локальной функции](./media/functions-run-function-test-local-vs-code/functions-vscode-f5.png)

    При возникновении проблем с запуском в Windows убедитесь, что в качестве терминала по умолчанию для Visual Studio Code не используется **оболочка WSL**.

1. При запущенных Core Tools перейдите в область **Azure: Функции**. В разделе **Функции** разверните **Локальный проект** > **Функции**. Щелкните правой кнопкой мыши (в Windows) или используйте <kbd>Ctrl-</kbd>щелчок (в macOS) на функции `HttpExample` и выберите **Выполнить функцию...** .

    :::image type="content" source="media/functions-run-function-test-local-vs-code/execute-function-now.png" alt-text="Вариант &quot;Выполнить функцию&quot; из Visual Studio Code":::
    
1. В поле **Ввести текст запроса** вы увидите значение текста запроса `{ "name": "Azure" }`. Нажмите клавишу ВВОД, чтобы отправить это сообщение запроса в свою функцию. 

   Вместо этого можно отправить запрос HTTP GET по адресу `http://localhost:7071/api/HttpExample` в веб-браузере.

1. При выполнении функции локально и возврате ответа в Visual Studio Code отобразится уведомление. Сведения о выполнении функции отображаются на панели **Терминал**.

1. Нажмите клавиши <kbd>CTRL+C</kbd>, чтобы остановить Core Tools и отключить отладчик.

---
title: Тестирование Azure Stream Analytics запросов локально для входа в динамический поток с помощью Visual Studio Code
description: В этой статье описывается, как выполнить локальное тестирование запросов для входных данных в динамическом потоке с помощью средств Azure Stream Analytics для Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/14/2019
ms.topic: how-to
ms.openlocfilehash: dd4966ee75e9dc0ff401823e4291f8d299c8893c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93122900"
---
# <a name="test-stream-analytics-queries-locally-against-live-stream-input-by-using-visual-studio-code"></a>Тестирование Stream Analytics запросов локально для входа в динамический поток с помощью Visual Studio Code

Azure Stream Analytics инструменты можно использовать для Visual Studio Code для локального тестирования заданий Stream Analytics в потоке ввода в режиме реального времени. Входные данные могут поступать из источника, такого как концентраторы событий Azure или центр Интернета вещей Azure. Выходные результаты отправляются в виде JSON-файлов в папку в проекте с именем **локалрунаутпутс**.

## <a name="prerequisites"></a>Предварительные условия

* Установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download) и перезапустите Visual Studio Code.

* Используйте [это краткое руководство](quick-create-visual-studio-code.md) , чтобы узнать, как создать Stream Analytics задание с помощью Visual Studio Code.

## <a name="define-a-live-stream-input"></a>Определение входного потока в реальном времени

1. Щелкните правой кнопкой мыши папку **Inputs** в проекте Stream Analytics. Теперь выберите **ASA: добавить входные данные** в контекстном меню.

   ![Добавление входных данных из папки Inputs](./media/quick-create-visual-studio-code/add-input-from-inputs-folder.png)

   Можно также нажать **клавиши CTRL + SHIFT + P** , чтобы открыть палитру команд, и ввести **ASA: Add input**.

   ![Добавление входа Stream Analytics в Visual Studio Code](./media/quick-create-visual-studio-code/add-input.png)

2. Выберите тип источника входных данных из раскрывающегося списка.

   ![Выбор центра Интернета вещей в качестве источника входных данных](./media/quick-create-visual-studio-code/iot-hub.png)

3. Если вы добавили источник входных данных с помощью палитры команд, выберите скрипт запроса Stream Analytics, который будет использовать этот источник. Здесь будет автоматически указан путь к файлу **myASAproj.asaql**.

   ![Выбор скрипта Stream Analytics в Visual Studio Code](./media/quick-create-visual-studio-code/asa-script.png)

4. Щелкните **Выбрать подписку Azure** в раскрывающемся списке.

    ![Выбор подписки в списке](./media/quick-create-visual-studio-code/add-input-select-subscription.png)

5. Настройте созданный JSON файл. Функция CodeLens помогает вводить строки, выбирать значения в раскрывающихся списках и редактировать текст непосредственно в файле. На следующем снимке экрана показано, как **выбрать подписку в списке**.

   ![Настройка входного файла в Visual Studio Code](./media/quick-create-visual-studio-code/configure-input.png)

## <a name="preview-input"></a>Предварительный просмотр входных данных

Чтобы убедиться, что входные данные получены, выберите **Предварительный просмотр данных** в динамическом входном файле конфигурации в верхней строке. Некоторые входные данные поступают из центра Интернета вещей и отображаются в окне предварительного просмотра. Просмотр может занять несколько секунд.

 ![Просмотр динамических входных данных](./media/quick-create-visual-studio-code/preview-live-input.png)

## <a name="run-queries-locally"></a>Локальное выполнение запросов.

Вернитесь в редактор запросов и выберите **запустить локально**. Затем выберите **использовать динамический вход** из раскрывающегося списка.

![Выберите "выполнить локально" в редакторе запросов.](./media/vscode-local-run/run-locally.png)

![Выберите "использовать Live input"](./media/vscode-local-run-live-input/run-locally-use-live-input.png)

Результат отображается в правом окне и обновляется каждые 3 секунды. Можно выбрать **Запуск** для повторного тестирования. Можно также выбрать пункт **Открыть в папке** , чтобы просмотреть файлы результатов в проводнике и открыть их с помощью Visual Studio Code или такого средства, как Excel. Обратите внимание, что файлы результатов доступны только в формате JSON.

По умолчанию задано время начала создания выходных данных для задания, равное **Now**. Время можно настроить, нажав кнопку **время начала вывода** в окне результатов.

![Просмотреть результат локального запуска](./media/vscode-local-run-live-input/vscode-livetesting.gif)

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр Azure Stream Analytics заданий с помощью Visual Studio Code (Предварительная версия)](visual-studio-code-explore-jobs.md)

* [Настройка конвейеров CI/CD с использованием пакета npm](./cicd-overview.md)
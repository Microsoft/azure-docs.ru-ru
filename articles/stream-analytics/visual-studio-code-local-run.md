---
title: Локальное тестирование задания Azure Stream Analytics с использованием примеров данных с помощью Visual Studio Code
description: В этой статье описывается, как проверить запросы локально с использованием примеров данных с помощью средств Azure Stream Analytics для Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/10/2019
ms.topic: how-to
ms.openlocfilehash: bbd83fb3ef3225fc19c48bb4c5962d6559cf32f8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97708455"
---
# <a name="test-stream-analytics-queries-locally-with-sample-data-using-visual-studio-code"></a>Локальное тестирование запросов Stream Analytics с использованием примера данных и Visual Studio Code

Вы можете использовать средства Azure Stream Analytics для Visual Studio Code для локального тестирования заданий Stream Analytics с помощью примеров данных. Результаты запроса можно найти в файлах JSON в папке **локалрунаутпутс** проекта.

## <a name="prerequisites"></a>Предварительные условия

* Установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download) и перезапустите Visual Studio Code.

* Используйте это [Краткое руководство](quick-create-visual-studio-code.md) , чтобы узнать, как создать задание Stream Analytics с помощью Visual Studio Code.

## <a name="prepare-sample-data"></a>Подготовка примера данных

Сначала необходимо подготовить примеры файлов входных данных. Если на компьютере уже есть примеры файлов данных, этот шаг можно пропустить и перейти к следующему.

1. Щелкните **Предварительный просмотр данных** в файле входной конфигурации в верхней строке. Некоторые входные данные будут получены из центра Интернета вещей и показаны в окне предварительного просмотра. Обратите внимание, что это может занять некоторое время.

2. После отображения данных нажмите кнопку **Сохранить как** , чтобы сохранить данные в локальный файл.

 ![Просмотр динамических входных данных](./media/quick-create-visual-studio-code/preview-live-input.png)

## <a name="define-a-local-input"></a>Определить локальный вход

1. Щелкните **input.js** в разделе Папка входных данных в проекте Stream Analytics. Затем выберите **Добавить локальный вход** в верхней строке.

    ![Добавить локальный вход из проекта](./media/quick-create-visual-studio-code/add-input-from-project.png)

    Можно также нажать **клавиши CTRL + SHIFT + P** , чтобы открыть палитру команд и ввести **ASA: Add input**.

   ![Добавление входных данных Stream Analytics в VS Code](./media/quick-create-visual-studio-code/add-input.png)

2. Выберите **локальный вход**.

    ![Добавление местных входных данных ASA в Visual Studio Code](./media/vscode-local-run/add-local-input.png)

3. Выберите **+ новый локальный вход**.

    ![Добавление нового локального ввода ASA в Visual Studio Code](./media/vscode-local-run/add-new-local-input.png)

4. Введите тот же псевдоним ввода, который использовался в запросе.

    ![Добавление нового локального псевдонима ASA](./media/vscode-local-run/new-local-input-alias.png)

5. В созданном **LocalInput_Input.jsв** файле введите путь к файлу, в котором находится локальный файл данных.

    ![Введите путь к локальному файлу в Visual Studio](./media/vscode-local-run/local-file-path.png)

6. Выберите **Предварительный просмотр данных** , чтобы просмотреть входные данные. Тип сериализации для данных автоматически определяется в формате JSON или CSV. Используйте селектор для просмотра данных в **таблице** или **необработанном** формате. В таблице ниже приведен пример данных в **формате таблицы**.

     ![Предварительный просмотр локальных данных в формате таблицы](./media/vscode-local-run/local-file-preview-table.png)

    В следующей таблице приведен пример данных в **формате RAW**:

    ![Предварительный просмотр локальных данных в необработанном формате](./media/vscode-local-run/local-file-preview-raw.png)

## <a name="run-queries-locally"></a>Локальное выполнение запросов.

Вернитесь в редактор запросов и выберите **запустить локально**. Затем выберите **использовать локальный вход** из раскрывающегося списка.

![Выберите выполнить локально в редакторе запросов.](./media/vscode-local-run/run-locally.png)

![Использовать локальный вход](./media/vscode-local-run/run-locally-use-local-input.png)

Результат отображается в правом окне. Для повторного тестирования можно нажать кнопку **выполнить** . Можно также выбрать пункт **Открыть в папке** , чтобы просмотреть файлы результатов в проводнике и затем открыть их с помощью других средств. Обратите внимание, что файлы результатов доступны только в формате JSON.

![Просмотреть результат локального запуска](./media/vscode-local-run/run-locally-result.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Тестирование Azure Stream Analytics заданий локально с входными данными в режиме реального времени с помощью Visual Studio Code](visual-studio-code-local-run-live-input.md)

* [Просмотр Azure Stream Analytics заданий с помощью Visual Studio Code (Предварительная версия)](visual-studio-code-explore-jobs.md)

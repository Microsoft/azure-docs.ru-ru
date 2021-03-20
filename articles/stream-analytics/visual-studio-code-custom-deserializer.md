---
title: Создание пользовательских десериализаторов .NET для Azure Stream Analytics облачных заданий с помощью Visual Studio Code
description: В этом руководстве показано, как создать настраиваемый десериализатор .NET для Azure Stream Analytics облачного задания с помощью Visual Studio Code.
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/22/2020
ms.openlocfilehash: 1813fb222bca74f355fec52252ce3d77fef06e5d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98013929"
---
# <a name="create-custom-net-deserializers-for-azure-stream-analytics-in-visual-studio-code"></a>Создание пользовательских десериализаторов .NET для Azure Stream Analytics в Visual Studio Code

Azure Stream Analytics имеет [встроенную поддержку трех форматов данных](stream-analytics-parsing-json.md): JSON, CSV и Avro. С помощью пользовательских десериализаторов .NET можно считывать данные из других форматов, таких как [буфер протокола](https://developers.google.com/protocol-buffers/), [облигаций](https://github.com/Microsoft/bond) и другие пользовательские форматы для облачных заданий.

## <a name="custom-net-deserializers-in-visual-studio-code"></a>Пользовательские десериализаторы .NET в Visual Studio Code

Вы можете создать, протестировать и отладить пользовательский десериализатор .NET для Azure Stream Analytics облачного задания с помощью Visual Studio Code.

### <a name="prerequisites"></a>Предварительные условия

* Установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download) и перезапустите Visual Studio Code.

* Используйте это [Краткое руководство](quick-create-visual-studio-code.md) , чтобы узнать, как создать задание Stream Analytics с помощью Visual Studio Code.

### <a name="create-a-custom-deserializer"></a>Создание настраиваемого десериализатора

1. Откройте терминал и выполните следующую команду, чтобы создать библиотеку классов .NET в Visual Studio Code для пользовательского десериализатора с именем **протобуфдесериализер**.

   ```dotnetcli
   dotnet new classlib -o ProtobufDeserializer
   ```

2. Перейдите в каталог проекта **протобуфдесериализер** и установите пакеты NuGet [Microsoft. Azure. StreamAnalytics](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics/) и [Google. protobuf](https://www.nuget.org/packages/Google.Protobuf/) .

   ```dotnetcli
   dotnet add package Microsoft.Azure.StreamAnalytics
   ```

   ```dotnetcli
   dotnet add package Google.Protobuf
   ```

3. Добавьте в проект [класс MessageBodyProto](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/MessageBodyProto.cs) и [класс MessageBodyDeserializer](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/MessageBodyDeserializer.cs).

4. Создайте проект **протобуфдесериализер** .

### <a name="add-an-azure-stream-analytics-project"></a>Добавление проекта Azure Stream Analytics

1. Откройте Visual Studio Code и нажмите **клавиши CTRL + SHIFT + P** , чтобы открыть палитру команд. Затем введите ASA и выберите **ASA: создать проект**. Назовите его **протобуфклауддесериализер**.

### <a name="configure-a-stream-analytics-job"></a>Настройка задания Stream Analytics

1. Дважды щелкните **JobConfig.json**. Используйте конфигурации по умолчанию, за исключением следующих параметров:

   |Параметр|Рекомендуемое значение|
   |-------|---------------|
   |Ресурс параметров глобального хранилища|Выберите источник данных из текущей учетной записи|
   |Подписка на параметры глобального хранилища| < your subscription >|
   |Учетная запись хранения параметров глобального хранилища| < ваша учетная запись хранения >|
   |Учетная запись хранения параметров Кустомкодестораже|< ваша учетная запись хранения >|
   |Контейнер параметров Кустомкодестораже|< ваш контейнер хранилища >|

2. В разделе **входные данные** откройте **input.jsна**. Выберите **добавить динамический вход** и добавьте входные данные из Azure Data Lake Storage 2-го поколения или хранилища BLOB-объектов, а затем выберите **выбрать из подписки Azure**. Используйте конфигурации по умолчанию, за исключением следующих параметров:

   |Параметр|Рекомендуемое значение|
   |-------|---------------|
   |Имя|Входные данные|
   |Подписка|< your subscription >|
   |Учетная запись хранения|< ваша учетная запись хранения >|
   |Контейнер|< ваш контейнер хранилища >|
   |Тип сериализации|Выбор **настраиваемого**|
   |сериализатионпрожектпас|Выберите выбрать **путь к проекту библиотеки** из CodeLens и выберите проект библиотеки **протобуфдесериализер** , созданный в предыдущем разделе. Выбор **проекта сборки** для сборки проекта|
   |сериализатионкласснаме|Выберите выбрать **класс десериализации** из CodeLens, чтобы автоматически заполнить имя класса и путь библиотеки DLL.|
   |Имя класса|MessageBodyProto.MessageBodyDeserializer|

   :::image type="content" source="./media/custom-deserializer/create-input-vscode.png" alt-text="Добавьте входные данные для пользовательского десериализатора.":::

3. Добавьте следующий запрос в файл **протобуфклауддесериализер. asaql** .

   ```sql
   SELECT * FROM Input
   ```

4. Скачайте [пример входного файла protobuf](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/SimulatedTemperatureEvents.protobuf). В папке **входные данные** щелкните правой кнопкой мыши **input.jsв** и выберите команду **Добавить локальный вход**. Затем дважды щелкните **local_input1.jsна** и используйте конфигурации по умолчанию, за исключением следующих параметров.

   |Параметр|Рекомендуемое значение|
   |-------|---------------|
   |Выбор локального пути к файлу|Щелкните CodeLens, чтобы выбрать < путь к файлу скачанного примера входного файла protobuf>|

### <a name="execute-the-stream-analytics-job"></a>Выполнение задания Stream Analytics

1. Откройте **протобуфклауддесериализер. asaql** и выберите **запустить локально** из CodeLens, а затем выберите **использовать локальный вход** из раскрывающегося списка.

2. Просмотрите результаты на вкладке **результаты** в представлении схемы задания справа. Можно также щелкнуть шаги на схеме задания, чтобы просмотреть промежуточный результат. Дополнительные сведения см. [в статье локальная отладка с помощью схемы заданий](debug-locally-using-job-diagram-vs-code.md).

   :::image type="content" source="./media/custom-deserializer/check-local-run-result-vscode.png" alt-text="Проверьте результат локального запуска.":::

Вы успешно внедрили настраиваемый десериализатор в задание Stream Analytics! В этом руководстве вы проверили пользовательский десериализатор локально с помощью локальных входных данных. Вы также можете протестировать его [с помощью динамического ввода данных в облаке](visual-studio-code-local-run-live-input.md). Для запуска задания в облаке необходимо правильно настроить входные и выходные данные. Затем отправьте задание в Azure с Visual Studio Code для запуска задания в облаке с помощью только что реализованного пользовательского десериализатора.

### <a name="debug-your-deserializer"></a>Отладка десериализатора

Десериализатор .NET можно отлаживать локально так же, как и стандартный код .NET. 

1. Добавьте точки останова в функцию .NET.

2. Щелкните **запустить** из панели действий Visual Studio Code и выберите **создать launch.jsв файле**.
   :::image type="content" source="./media/custom-deserializer/create-launch-file-vscode.png" alt-text="Создайте файл запуска.":::

   Выберите **протобуфклауддесериализер** , а затем **Azure Stream Analytics** из раскрывающегося списка.
   :::image type="content" source="./media/custom-deserializer/create-launch-file-vscode-2.png" alt-text="Создайте файл запуска 2.":::

   Измените **launch.jsв** файле, чтобы заменить <ASAScript> . asaql на протобуфклауддесериализер. asaql.
   :::image type="content" source="./media/custom-deserializer/configure-launch-file-vscode.png" alt-text="Настройка файла запуска.":::

3. Нажмите клавишу **F5**, чтобы запустить отладку. Программа будет останавливаться в точках останова. Это работает и для локальных входных, и для входных данных в реальном времени.

   :::image type="content" source="./media/custom-deserializer/debug-vscode.png" alt-text="Отладка пользовательского десериализатора.":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, задание потоковой передачи и все связанные ресурсы можно удалить. При удалении задания будет прекращена тарификация за единицы потоковой передачи, потребляемые заданием. Если вы планируете использовать это задание в будущем, вы можете остановить и перезапустить его позже. Если вы не собираетесь использовать это задание дальше, удалите все ресурсы, созданные в ходе работы с этим руководством, выполнив следующие шаги:

1. В меню слева на портале Azure выберите **Группы ресурсов**, а затем выберите имя созданного ресурса.  

2. На странице группы ресурсов выберите **Удалить**, в текстовом поле введите имя ресурса для удаления и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как внедрить настраиваемый десериализатор .NET для входной сериализации буфера протокола. Дополнительные сведения о создании настраиваемых десериализаторов, см. в следующей статье:

> [!div class="nextstepaction"]
> * [Использование десериализаторов .NET для заданий Azure Stream Analytics](custom-deserializer-examples.md)
> * [Тестирование Azure Stream Analytics заданий локально с входными данными в режиме реального времени с помощью Visual Studio Code](visual-studio-code-local-run-live-input.md)
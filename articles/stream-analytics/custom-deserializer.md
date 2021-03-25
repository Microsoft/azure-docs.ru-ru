---
title: Учебник по настраиваемым десериализаторам .NET для облачных заданий Azure Stream Analytics
description: В этом руководстве показано, как создать настраиваемый десериализатор .NET для облачного задания Azure Stream Analytics с помощью Visual Studio.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: tutorial
ms.date: 12/17/2020
ms.openlocfilehash: 6db65d85b5d5d72e7b4764949abcffed004cae85
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98018213"
---
# <a name="tutorial-custom-net-deserializers-for-azure-stream-analytics"></a>Руководство по Пользовательские десериализаторы .NET для Azure Stream Analytics

Azure Stream Analytics имеет [встроенную поддержку трех форматов данных](stream-analytics-parsing-json.md): JSON, CSV и Avro. С помощью настраиваемых десериализаторов .NET для облачных и граничных заданий можно считывать данные из форматов, таких как[буфер протоколов](https://developers.google.com/protocol-buffers/), [облигации](https://github.com/Microsoft/bond) и других пользовательских форматов.

В этом руководстве показано, как создать настраиваемый десериализатор .NET для облачного задания Azure Stream Analytics с помощью Visual Studio. Сведения о создании десериализаторов .NET в Visual Studio Code см. в статье [Создание десериализаторов .NET для заданий Azure Stream Analytics в Visual Studio Code](visual-studio-code-custom-deserializer.md).

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание настраиваемого десериализатора для буфера протокола.
> * Создание задания Stream Analytics с использованием инструментов Azure Stream Analytics в Visual Studio.
> * Настройка задания Stream Analytics для использования настраиваемого десериализатора.
> * Выполните локальный запуск задания Stream Analytics для тестирования и отладки настраиваемого десериализатора.


## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Установите [Visual Studio 2019 (рекомендуется)](https://www.visualstudio.com/downloads/) или [Visual Studio 2017](https://www.visualstudio.com/vs/older-downloads/). Поддерживаются выпуски Enterprise (Ultimate/Premium), Professional и Community. Выпуск Express не поддерживается. 

* [Установите инструменты Stream Analytics для Visual Studio](stream-analytics-tools-for-visual-studio-install.md) или обновите до последней версии. 

* Откройте **Cloud Explorer** в Visual Studio и войдите в свою подписку Azure.

* Создайте контейнер в учетной записи хранения.
Созданный контейнер будет использоваться для хранения ресурсов, связанных с заданием Stream Analytics. Если у вас уже есть учетная запись хранения с существующими контейнерами, вы можете их использовать. Если нет, [создайте новый контейнер](../storage/blobs/storage-quickstart-blobs-portal.md).

## <a name="create-a-custom-deserializer"></a>Создание настраиваемого десериализатора

1. Откройте Visual Studio и выберите **Файл > Создать > Проект**. Выполните поиск **Stream Analytics** и выберите **Проект настраиваемого десериализатора Azure Stream Analytics (.NET)**. Присвойте проекту имя, например **Protobuf Deserializer**.

   ![Создание проекта библиотеки стандартных классов dotnet Visual Studio](./media/custom-deserializer/create-dotnet-library-project.png)

2. В обозревателе решений щелкните правой кнопкой мыши проект **Protobuf Deserializer** и выберите пункт **Manage NuGet Packages** (Управление пакетами NuGet) в меню. Затем установите пакеты NuGet **Microsoft.Azure.StreamAnalytics** и **Google.Protobuf**.

3. Добавьте в проект [класс MessageBodyProto](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/MessageBodyProto.cs) и [класс MessageBodyDeserializer](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/MessageBodyDeserializer.cs).

4. Создайте проект **Protobuf Deserializer**.

## <a name="add-an-azure-stream-analytics-project"></a>Добавление проекта Azure Stream Analytics

1. В обозревателе решений щелкните правой кнопкой мыши решение **Protobuf Deserializer** и выберите **Добавить > Новый проект**. В разделе **Azure Stream Analytics > Stream Analytics** выберите **Приложение Azure Stream Analytics**. Присвойте ему имя **ProtobufCloudDeserializer** и выберите **ОК**. 

2. Щелкните правой кнопкой мыши **Ссылки** в проекте Azure Stream Analytics **ProtobufCloudDeserializer**. В пункт **Проекты** добавьте **Protobuf Deserializer**. Оно должно заполнится автоматически.

## <a name="configure-a-stream-analytics-job"></a>Настройка задания Stream Analytics

1. Дважды щелкните **JobConfig.json**. Используйте конфигурации по умолчанию, за исключением следующих параметров:

   |Параметр|Рекомендуемое значение|
   |-------|---------------|
   |Ресурс параметров глобального хранилища|Выберите источник данных из текущей учетной записи|
   |Подписка на параметры глобального хранилища| < your subscription >|
   |Учетная запись хранения параметров глобального хранилища| < ваша учетная запись хранения >|
   |Ресурс параметров хранилища настраиваемого кода|Выберите источник данных из текущей учетной записи|
   |Учетная запись хранения параметров хранилища настраиваемого кода|< ваша учетная запись хранения >|
   |Контейнер параметров хранилища настраиваемого кода|< ваш контейнер хранилища >|

2. В разделе **Входные данные** дважды щелкните **Input.json**. Используйте конфигурации по умолчанию, за исключением следующих параметров:

   |Параметр|Рекомендуемое значение|
   |-------|---------------|
   |Источник|Хранилище BLOB-объектов|
   |Ресурс|Выберите источник данных из текущей учетной записи|
   |Подписка|< your subscription >|
   |Учетная запись хранения|< ваша учетная запись хранения >|
   |Контейнер|< ваш контейнер хранилища >|
   |Формат сериализации событий|Другие (Protobuf, XML, Частный...)|
   |Ресурс|Загрузка из ссылки на проект ASA или CodeBehind|
   |Имя сборки CSharp|ProtobufDeserializer.dll|
   |Имя класса|MessageBodyProto.MessageBodyDeserializer|
   |Тип сжатия событий|None|

3. Добавьте следующий запрос в файл **script. asaql**.

   ```sql
   SELECT * FROM Input
   ```

4. Скачайте [пример входного файла protobuf](https://github.com/Azure/azure-stream-analytics/blob/master/CustomDeserializers/Protobuf/SimulatedTemperatureEvents.protobuf). В папке **Входные данные**, щелкните правой кнопкой мыши **Input.json** и выберите команду **Добавить локальный ввод**. Затем дважды щелкните **local_Input.json** и настройте следующие параметры:

   |Параметр|Рекомендуемое значение|
   |-------|---------------|
   |Псевдоним входных данных|Входные данные|
   |Тип источника|Поток данных|
   |Формат сериализации событий|Другие (Protobuf, XML, Частный...)|
   |Имя сборки CSharp|ProtobufDeserializer.dll|
   |Имя класса|MessageBodyProto.MessageBodyDeserializer|
   |Путь к локальному входному файлу|< путь к файлу скачанного примера входного файла protobuf >|

## <a name="execute-the-stream-analytics-job"></a>Выполнение задания Stream Analytics

1. Откройте **Script.asaql** и выберите **Запустить локально**.

2. Просмотрите результаты в модуле **результаты локального выполнения Stream Analytics**.

Вы успешно внедрили настраиваемый десериализатор в задание Stream Analytics! В этом руководстве вы локально проверили настраиваемый десериализатор. Для реальных данных необходимо правильно настроить входные и выходные данные. Затем отправьте задание в Azure из Visual Studio, чтобы запустить задание в облаке, используя только что внедренный настраиваемый десериализатор.

## <a name="debug-your-deserializer"></a>Отладка десериализатора

Десериализатор .NET можно отлаживать локально так же, как и стандартный код .NET.

1. Щелкните правой кнопкой мыши имя проекта **ProtobufCloudDeserializer** и задайте его в качестве запускаемого проекта.

2. Добавьте точки останова в функцию.

3. Нажмите клавишу **F5**, чтобы запустить отладку. Программа будет останавливаться в точках останова.

## <a name="clean-up-resources"></a>Очистка ресурсов

Ставшие ненужными группу ресурсов, задание потоковой передачи и все связанные ресурсы можно удалить. При удалении задания будет прекращена тарификация за единицы потоковой передачи, потребляемые заданием. Если вы планируете использовать это задание в будущем, вы можете остановить и перезапустить его позже. Если вы не собираетесь использовать это задание дальше, удалите все ресурсы, созданные в ходе работы с этим руководством, выполнив следующие шаги:

1. В меню слева на портале Azure выберите **Группы ресурсов**, а затем выберите имя созданного ресурса.  

2. На странице группы ресурсов выберите **Удалить**, в текстовом поле введите имя ресурса для удаления и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как внедрить настраиваемый десериализатор .NET для входной сериализации буфера протокола. Дополнительные сведения о создании настраиваемых десериализаторов, см. в следующей статье:

> [!div class="nextstepaction"]
> [Использование десериализаторов .NET для заданий Azure Stream Analytics](custom-deserializer-examples.md)
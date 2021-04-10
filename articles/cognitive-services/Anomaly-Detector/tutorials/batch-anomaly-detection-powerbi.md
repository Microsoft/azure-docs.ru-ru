---
title: Руководство по визуализации аномалий c помощью пакетного обнаружения и Power BI
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать API Детектор аномалий и Power BI для визуализации аномалий в данных временных рядов.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: mbullwin
ms.openlocfilehash: fc2667dfddf4442acaf39f6783190e8640b22a89
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105561735"
---
# <a name="tutorial-visualize-anomalies-using-batch-detection-and-power-bi"></a>Руководство по визуализации аномалий c помощью пакетного обнаружения и Power BI

В этом учебнике показано, как найти аномалии в наборе данных временных рядов в пакетном режиме. В Power BI Desktop в файле Excel подготовьте данные для API Детектора аномалий и визуализируйте статистические аномалии в нем.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Импорт и преобразование набора данных временных рядов в Power BI Desktop.
> * Интеграция Power BI Desktop с API Детектора аномалий для пакетного обнаружения аномалий.
> * Визуализация аномалий, которые находятся в данных, включая ожидаемые и видимые значения, а также границы обнаружения аномалий.

## <a name="prerequisites"></a>Предварительные требования
* [Подписка Azure](https://azure.microsoft.com/free/cognitive-services)
* [Microsoft Power BI Desktop](https://powerbi.microsoft.com/get-started/) предоставляется бесплатно.
* Файл Excel (XLSX) содержит точки данных временных рядов. Пример данных для этого краткого руководства можно найти на сайте [GitHub](https://go.microsoft.com/fwlink/?linkid=2090962).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector"  title="Создание ресурса Детектора аномалий"  target="_blank">созданию ресурса Детектора аномалий </a> на портале Azure, чтобы получить ключ и конечную точку.
    * Для подключения приложения к API "Детектор аномалий" потребуется ключ и конечная точка из созданного ресурса. Вы сделаете это позже в этом кратком руководстве.

[!INCLUDE [cognitive-services-anomaly-detector-data-requirements](../../../../includes/cognitive-services-anomaly-detector-data-requirements.md)]

## <a name="load-and-format-the-time-series-data"></a>Загрузка и форматирование данных временных рядов

Чтобы начать работу, откройте Power BI Desktop и загрузите данные временных рядов, которые вы скачали при выполнении предварительных требований. Этот файл Excel содержит ряд пар меток времени в формате UTC и значений.  

> [!NOTE]
> Power BI использует данные из разнообразных источников, например из CSV-файлов, баз данных SQL, хранилища BLOB-объектов Azure и других.  

В главном окне Power BI Desktop щелкните ленту **Главная**. В группе **Внешние данные** на ленте откройте раскрывающееся меню **Получить данные** и выберите **Excel**.

![Изображение кнопки "Получить данные" в Power BI](../media/tutorials/power-bi-get-data-button.png)

В появившемся диалоговом окне перейдите в папку, в которую вы скачали пример файла XLSX, и выберите его. В открывшемся диалоговом окне **Навигатор** щелкните **Лист1**, а затем **Изменить**.

![Изображение экрана "Навигатор" с источником данных в Power BI](../media/tutorials/navigator-dialog-box.png)

Power BI преобразует отметки времени в первом столбце в тип данных `Date/Time`. Эти метки времени должны быть преобразованы в текст для отправки в API Детектора аномалий. Если редактор Power Query не открывается автоматически, щелкните **Изменить запросы** на вкладке "Главная".

Выберите ленту **Преобразование** в редакторе Power Query. В группе **Любой столбец** откройте раскрывающееся меню **Тип данных:** и выберите **Текстовый**.

![Изображение раскрывающегося списка типа данных](../media/tutorials/data-type-drop-down.png)

При получении уведомления об изменении типа столбца щелкните **Заменить текущее**. После этого щелкните **Закрыть и применить** или **Применить** на ленте **Главная**.

## <a name="create-a-function-to-send-the-data-and-format-the-response"></a>Создание функции для отправки данных и форматирования ответа

Чтобы отформатировать и отправить файл данных в API Детектора аномалий, вы можете вызвать запрос к созданной ранее таблице. В редакторе Power Query на ленте **Главная** откройте раскрывающееся меню **Создать источник** и выберите **Пустой запрос**.

Убедитесь, что новый запрос выбран, а затем щелкните **Расширенный редактор**.

![Изображение экрана "Расширенный редактор"](../media/tutorials/advanced-editor-screen.png)

В Расширенном редакторе используйте следующий фрагмент кода Power Query M для извлечения столбцов из таблицы и отправки их в API. Запрос создаст таблицу на основе ответа JSON и вернет ее. Замените переменную `apiKey` на допустимый ключ API Детектора аномалий, а `endpoint` на вашу конечную точку. После ввода запроса в Расширенном редакторе щелкните **Готово**.

```M
(table as table) => let

    apikey      = "[Placeholder: Your Anomaly Detector resource access key]",
    endpoint    = "[Placeholder: Your Anomaly Detector resource endpoint]/anomalydetector/v1.0/timeseries/entire/detect",
    inputTable = Table.TransformColumnTypes(table,{{"Timestamp", type text},{"Value", type number}}),
    jsontext    = Text.FromBinary(Json.FromValue(inputTable)),
    jsonbody    = "{ ""Granularity"": ""daily"", ""Sensitivity"": 95, ""Series"": "& jsontext &" }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Content-Type" = "application/json", #"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody, ManualStatusHandling={400}]),
    jsonresp    = Json.Document(bytesresp),

    respTable = Table.FromColumns({

                     Table.Column(inputTable, "Timestamp")
                     ,Table.Column(inputTable, "Value")
                     , Record.Field(jsonresp, "IsAnomaly") as list
                     , Record.Field(jsonresp, "ExpectedValues") as list
                     , Record.Field(jsonresp, "UpperMargins")as list
                     , Record.Field(jsonresp, "LowerMargins") as list
                     , Record.Field(jsonresp, "IsPositiveAnomaly") as list
                     , Record.Field(jsonresp, "IsNegativeAnomaly") as list

                  }, {"Timestamp", "Value", "IsAnomaly", "ExpectedValues", "UpperMargin", "LowerMargin", "IsPositiveAnomaly", "IsNegativeAnomaly"}
               ),

    respTable1 = Table.AddColumn(respTable , "UpperMargins", (row) => row[ExpectedValues] + row[UpperMargin]),
    respTable2 = Table.AddColumn(respTable1 , "LowerMargins", (row) => row[ExpectedValues] -  row[LowerMargin]),
    respTable3 = Table.RemoveColumns(respTable2, "UpperMargin"),
    respTable4 = Table.RemoveColumns(respTable3, "LowerMargin"),

    results = Table.TransformColumnTypes(

                respTable4,
                {{"Timestamp", type datetime}, {"Value", type number}, {"IsAnomaly", type logical}, {"IsPositiveAnomaly", type logical}, {"IsNegativeAnomaly", type logical},
                 {"ExpectedValues", type number}, {"UpperMargins", type number}, {"LowerMargins", type number}}
              )

 in results
```

Вызовите запрос на листе данных, выбрав в разделе **Введите параметр** значение `Sheet1` и нажав кнопку **Вызвать**.

![Изображение функции вызова](../media/tutorials/invoke-function-screenshot.png)

## <a name="data-source-privacy-and-authentication"></a>Конфиденциальность источника данных и аутентификация

> [!NOTE]
> Ознакомьтесь с политиками своей организации в отношении конфиденциальности данных и доступа к ним. Дополнительные сведения см. в статье об [уровнях конфиденциальности Power BI Desktop](/power-bi/desktop-privacy-levels).

При попытке выполнить запрос может появиться предупреждающее сообщение, так как в запросе используется внешний источник данных.

![Изображение, демонстрирующее предупреждение в Power BI](../media/tutorials/blocked-function.png)

Чтобы устранить эту проблему, последовательно выберите пункты **Файл**, **Параметры и настройки**. Затем щелкните **Параметры**. Под пунктом **Текущий файл** выберите **Конфиденциальность** и **Игнорировать уровни конфиденциальности для возможного улучшения производительности**.

Кроме того, может появиться сообщение с предложением указать способ подключения к API.

![Изображение, демонстрирующее запрос для указания учетных данных для доступа](../media/tutorials/edit-credentials-message.png)

Чтобы устранить эту проблему, щелкните **Изменить учетные данные** в сообщении. Когда появится диалоговое окно, выберите **Анонимно** для анонимного подключения к API. Щелкните **Подключить**.

После этого щелкните **Закрыть и применить** на ленте **Главная**, чтобы применить изменения.

## <a name="visualize-the-anomaly-detector-api-response"></a>Визуализация ответа API Детектора аномалий

На главном экране Power BI начните использовать созданные ранее запросы для визуализации данных. Сначала выберите в разделе **Визуализации** тип **График**. Затем добавьте на **ось** графика метку времени из вызванной функции. Щелкните значение правой кнопкой мыши и выберите **Метка времени**.

![Щелкните значение метки времени правой кнопкой мыши](../media/tutorials/timestamp-right-click.png)

Добавьте следующие поля из раздела **Вызванная функция** в поле **Значения** графика. Создайте график, используя параметры в приведенном ниже снимке экрана.

* Значение
* UpperMargins
* LowerMargins
* ExpectedValues

![Изображение параметров диаграммы](../media/tutorials/chart-settings.png)

После добавления полей щелкните график и измените его размер так, чтобы отображались все точки данных. График будет выглядеть примерно так, как на снимке экрана ниже:

![Изображение визуализации диаграммы](../media/tutorials/chart-visualization.png)

### <a name="display-anomaly-data-points"></a>Отображение точек данных с аномалиями

В правой части окна Power BI под областью **ПОЛЯ** в разделе **запроса вызванной функции** щелкните пункт **Значение** правой кнопкой мыши и выберите **Новая быстрая мера**.

![Изображение экрана новой быстрой меры](../media/tutorials/new-quick-measure.png)

На появившемся экране выберите **Отфильтрованное значение** в качестве вычисления. В поле **Базовое значение** выберите значение `Sum of Value`. Затем перетащите значение `IsAnomaly` из полей **Вызванная функция** в поле **Фильтр**. В раскрывающемся меню **Фильтр** выберите значение `True`.

![Второе изображение экрана новой быстрой меры](../media/tutorials/new-quick-measure-2.png)

После нажатия кнопки **ОК** в нижней части списка полей появится поле `Value for True`. Щелкните его правой кнопкой мыши и переименуйте на **Аномалии**. Добавьте его в **значения** графика. Затем выберите средство **Формат** и задайте для оси X тип **Категориальный**.

![Изображение формата оси X](../media/tutorials/format-x-axis.png)

Примените цвета к графику, щелкнув средство **Формат** и **Цвета данных**. График должен выглядеть примерно следующим образом:

![Изображение конечной диаграммы](../media/tutorials/final-chart.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
>[Обнаружение аномалий при потоковой передаче данных с помощью Azure Databricks](../overview.md)
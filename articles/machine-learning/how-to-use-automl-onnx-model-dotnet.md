---
title: Создание прогнозов с помощью модели Аутомл ONNX в .NET
description: Узнайте, как создавать прогнозы с помощью модели Аутомл ONNX в .NET с помощью ML.NET
titleSuffix: Azure Machine Learning
author: luisquintanilla
ms.author: luquinta
ms.date: 10/30/2020
ms.topic: conceptual
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.custom: how-to, automl
ms.openlocfilehash: 4fb147dc5c57c3a98607a025f566fa583bf87460
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93358819"
---
# <a name="make-predictions-with-an-automl-onnx-model-in-net"></a>Создание прогнозов с помощью модели Аутомл ONNX в .NET

Из этой статьи вы узнаете, как использовать модель автоматизированного ML (Аутомл) Open нейронной сети (ONNX) для создания прогнозов в консольном приложении .NET Core на C# с помощью ML.NET.

[Ml.NET](/dotnet/machine-learning/) — это платформа машинного обучения с открытым исходным кодом, предназначенная для экосистемы .NET, которая позволяет обучать и использовать пользовательские модели машинного обучения, используя подход с использованием кода в C# или F #, а также с помощью средств с небольшим кодом, таких как [построитель моделей](/dotnet/machine-learning/automate-training-with-model-builder) и интерфейс [командной строки ML.NET](/dotnet/machine-learning/automate-training-with-cli). Платформа также является расширяемой и позволяет использовать другие популярные платформы машинного обучения, такие как TensorFlow и ONNX.

ONNX — это формат с открытым исходным кодом для моделей AI. ONNX поддерживает взаимодействие между разными платформами. Это означает, что модель можно обучить в одной из многих популярных платформ машинного обучения, таких как PyTorch, преобразовать ее в формат ONNX и использовать модель ONNX в другой инфраструктуре, такой как ML.NET. Дополнительные сведения см. на [веб-сайте ONNX](https://onnx.ai/).

## <a name="prerequisites"></a>Предварительные условия

- [Пакет SDK для .NET Core 3,1 или выше](https://dotnet.microsoft.com/download)
- Текстовый редактор или интегрированная среда разработки (например, [Visual Studio](https://visualstudio.microsoft.com/vs/) или [Visual Studio Code](https://code.visualstudio.com/Download))
- Модель ONNX. Сведения о том, как обучить модель Аутомл ONNX, см. в следующей [записной книжке по классификации для маркетинга в банке](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/classification-bank-marketing-all-features/auto-ml-classification-bank-marketing-all-features.ipynb).
- [Netron](https://github.com/lutzroeder/netron) (необязательно)

## <a name="create-a-c-console-application"></a>Создание консольного приложения на C#

В этом примере для сборки приложения используется .NET Core CLI, но вы можете выполнять те же задачи с помощью Visual Studio. Дополнительные сведения о [.NET Core CLI](/dotnet/core/tools/).

1. Откройте терминал и создайте новое консольное приложение C# .NET Core. В этом примере имя приложения — `AutoMLONNXConsoleApp` . Каталог создается с тем же именем, что и содержимое приложения.

    ```dotnetcli
    dotnet new console -o AutoMLONNXConsoleApp
    ```

1. В окне терминала перейдите в каталог *аутомлоннксконсолеапп* .

    ```bash
    cd AutoMLONNXConsoleApp
    ```

## <a name="add-software-packages"></a>Добавление пакетов программного обеспечения

1. Установите пакеты NuGet **Microsoft.ml**, **Microsoft. ml. оннксрунтиме** и **Microsoft. ml. оннкстрансформер** с помощью .NET Core CLI.

    ```dotnetcli
    dotnet add package Microsoft.ML
    dotnet add package Microsoft.ML.OnnxRuntime
    dotnet add package Microsoft.ML.OnnxTransformer
    ```

    Эти пакеты содержат зависимости, необходимые для использования модели ONNX в приложении .NET. ML.NET предоставляет API, который использует [среду выполнения ONNX](https://github.com/Microsoft/onnxruntime) для прогнозов.

1. Откройте файл *Program. CS* и добавьте следующие `using` инструкции в начало для ссылки на соответствующие пакеты.

    ```csharp
    using System.Linq;
    using Microsoft.ML;
    using Microsoft.ML.Data;
    using Microsoft.ML.Transforms.Onnx;
    ```

## <a name="add-a-reference-to-the-onnx-model"></a>Добавление ссылки на модель ONNX

Чтобы консольное приложение может получить доступ к модели ONNX, добавьте ее в выходной каталог сборки.  Дополнительные сведения об общих элементах MSBuild см. в разделе с [руководством по MSBuild](/visualstudio/msbuild/common-msbuild-project-items).

Добавление ссылки на файл модели ONNX в приложении

1. Скопируйте модель ONNX в корневой каталог *аутомлоннксконсолеапп* приложения.
1. Откройте файл *аутомлоннксконсолеапп. csproj* и добавьте следующее содержимое в `Project` узел.

    ```xml
    <ItemGroup>
        <None Include="automl-model.onnx">
            <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
        </None>
    </ItemGroup>
    ```

    В этом случае имя файла модели ONNX — *аутомл-Model. ONNX*.

1. Откройте файл *Program. CS* и добавьте в класс следующую строку `Program` .

    ```csharp
    static string ONNX_MODEL_PATH = "automl-model.onnx";
    ```

## <a name="initialize-mlcontext"></a>Инициализация Млконтекст

В `Main` методе `Program` класса создайте новый экземпляр [`MLContext`](xref:Microsoft.ML.MLContext) .

```csharp
MLContext mlContext = new MLContext();
```

[`MLContext`](xref:Microsoft.ML.MLContext)Класс является отправной точкой для всех операций ml.NET, а инициализация `mlContext` создает новую среду ml.NET, которая может использоваться в жизненном цикле модели. Он похож на DbContext в Entity Framework.

## <a name="define-the-model-data-schema"></a>Определение схемы данных модели

Ваша модель предполагала входные и выходные данные в определенном формате. ML.NET позволяет определить формат данных с помощью классов. Иногда вы уже умеете узнать, как выглядит этот формат. В случаях, когда формат данных неизвестен, можно использовать такие средства, как нетрон, для проверки модели ONNX.

Модель, используемая в этом образце, использует данные из набора данных Нью ТЛК такси. Пример данных можно увидеть ниже:

|vendor_id|rate_code|passenger_count|trip_time_in_secs|trip_distance|payment_type|fare_amount|
|---|---|---|---|---|---|---|
|VTS|1|1|1140|3,75 %|CRD|15.5|
|VTS|1|1|480|2.72|CRD|10.0|
|VTS|1|1|1680|7.8|кш|26,5|

### <a name="inspect-the-onnx-model-optional"></a>Проверка модели ONNX (необязательно)

Используйте такой инструмент, как нетрон, для проверки входных и выходных данных модели.

1. Откройте нетрон.
1. В верхней строке меню выберите **файл > открыть** и используйте обозреватель файлов для выбора модели.
1. Откроется модель. Например, структура модели *аутомл-Model. onnx* выглядит следующим образом:

    :::image type="content" source="media/how-to-use-automl-onnx-model-dotnet/netron-automl-onnx-model.png" alt-text="Нетрон Аутомл ONNX Model":::

1. Выберите последний узел в нижней части диаграммы ( `variable_out1` в данном случае), чтобы отобразить метаданные модели. Входные и выходные данные на боковой панели показывают ожидаемые входные, выходные и типы данных модели. Используйте эти сведения, чтобы определить входную и выходную схему модели.

### <a name="define-model-input-schema"></a>Определение входной схемы модели

Создайте новый класс с именем `OnnxInput` со следующими свойствами в файле *Program. CS* .

```csharp
public class OnnxInput
{
    [ColumnName("vendor_id")]
    public string VendorId { get; set; }

    [ColumnName("rate_code"),OnnxMapType(typeof(Int64),typeof(Single))]
    public Int64 RateCode { get; set; }

    [ColumnName("passenger_count"), OnnxMapType(typeof(Int64), typeof(Single))]
    public Int64 PassengerCount { get; set; }

    [ColumnName("trip_time_in_secs"), OnnxMapType(typeof(Int64), typeof(Single))]
    public Int64 TripTimeInSecs { get; set; }

    [ColumnName("trip_distance")]
    public float TripDistance { get; set; }

    [ColumnName("payment_type")]
    public string PaymentType { get; set; }
}
```

Каждое из свойств сопоставляется со столбцом в наборе данных. Свойства добавлены с атрибутами.

[`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute)Атрибут позволяет указать, как ml.NET должен ссылаться на столбец при работе с данными. Например, несмотря на то, что `TripDistance` свойство соответствует стандартным соглашениям об именовании .NET, в модели известен только столбец или функция, известные как `trip_distance` . Чтобы устранить несоответствие имен, [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) атрибут сопоставляет `TripDistance` свойство со столбцом или функцией по имени `trip_distance` .
  
Для числовых значений ML.NET работает только с [`Single`](xref:System.Single) типами значений. Однако исходные типы данных некоторых столбцов являются целыми числами. [`OnnxMapType`](xref:Microsoft.ML.Transforms.Onnx.OnnxMapTypeAttribute)Атрибут сопоставляет типы между ONNX и ml.NET.

Дополнительные сведения об атрибутах данных см. в разделе [ml.NET Load Data Guide](/dotnet/machine-learning/how-to-guides/load-data-ml-net).

### <a name="define-model-output-schema"></a>Определение выходной схемы модели

После обработки данные создают выходные данные определенного формата. Определите схему вывода данных. Создайте новый класс с именем `OnnxOutput` со следующими свойствами в файле *Program. CS* .

```csharp
public class OnnxOutput
{
    [ColumnName("variable_out1")]
    public float[] PredictedFare { get; set; }
}
```

Аналогично `OnnxInput` , используйте [`ColumnName`](xref:Microsoft.ML.Data.ColumnNameAttribute) атрибут, чтобы отобразить `variable_out1` выходные данные в более описательное имя `PredictedFare` .

## <a name="define-a-prediction-pipeline"></a>Определение конвейера прогнозирования

Конвейер в ML.NET обычно представляет собой последовательность связанных преобразований, которые работают с входными данными для получения выходных данных. Дополнительные сведения о преобразовании данных см. в разделе [ml.NET Data Transform Guide](/dotnet/machine-learning/resources/transforms).

1. Создание нового метода, вызываемого `GetPredictionPipeline` внутри `Program` класса

    ```csharp
    static ITransformer GetPredictionPipeline(MLContext mlContext)
    {

    }
    ```

1. Определите имя входных и выходных столбцов. Добавьте следующий код в метод `GetPredictionPipeline`.

    ```csharp
    var inputColumns = new string []
    {
        "vendor_id", "rate_code", "passenger_count", "trip_time_in_secs", "trip_distance", "payment_type"
    };

    var outputColumns = new string [] { "variable_out1" };
    ```

1. Определите свой конвейер. [`IEstimator`](xref:Microsoft.ML.IEstimator%601)Предоставляет схему операций, входных и выходных схем конвейера.

    ```csharp
    var onnxPredictionPipeline =
        mlContext
            .Transforms
            .ApplyOnnxModel(
                outputColumnNames: outputColumns,
                inputColumnNames: inputColumns,
                ONNX_MODEL_PATH);
    ```

    В этом случае [`ApplyOnnxModel`](xref:Microsoft.ML.OnnxCatalog.ApplyOnnxModel%2A) является единственным преобразованием в конвейере, которое принимает имена входных и выходных столбцов, а также путь к файлу модели ONNX.

1. [`IEstimator`](xref:Microsoft.ML.IEstimator%601)Определяет только набор операций, применяемых к данным. Что работает с данными, называется [`ITransformer`](xref:Microsoft.ML.ITransformer) . Используйте `Fit` метод, чтобы создать его из `onnxPredictionPipeline` .

    ```csharp
    var emptyDv = mlContext.Data.LoadFromEnumerable(new OnnxInput[] {});

    return onnxPredictionPipeline.Fit(emptyDv);
    ```

    [`Fit`](xref:Microsoft.ML.IEstimator%601.Fit%2A)Метод принимает [`IDataView`](xref:Microsoft.ML.IDataView) входные данные для выполнения операций в. [`IDataView`](xref:Microsoft.ML.IDataView)— Это способ представления данных в ml.NET с помощью табличного формата. Поскольку в этом случае конвейер используется только для прогнозов, можно предоставить пустое значение, [`IDataView`](xref:Microsoft.ML.IDataView) чтобы предоставить [`ITransformer`](xref:Microsoft.ML.ITransformer) необходимые входные и выходные данные схемы. [`ITransformer`](xref:Microsoft.ML.ITransformer)Затем для дальнейшего использования в приложении возвращается размещается.

    > [!TIP]
    > В этом примере конвейер определен и используется в том же приложении. Однако рекомендуется использовать отдельные приложения для определения и использования конвейера для создания прогнозов. В ML.NET конвейеры можно сериализовать и сохранить для дальнейшего использования в других приложениях конечных пользователей .NET. ML.NET поддерживает различные целевые объекты развертывания, такие как классические приложения, веб-службы, приложения сборки * и многие другие. Дополнительные сведения о сохранении конвейеров см. в [ml.NET сохранение и загрузка обученных моделей](/dotnet/machine-learning/how-to-guides/save-load-machine-learning-models-ml-net).
    >
    > * Сборка поддерживается только в .NET Core 5 или более поздней версии

1. Внутри `Main` метода вызовите `GetPredictionPipeline` метод с необходимыми параметрами.

    ```csharp
    var onnxPredictionPipeline = GetPredictionPipeline(mlContext);
    ```

## <a name="use-the-model-to-make-predictions"></a>Использование модели для прогнозирования

Теперь, когда у вас есть конвейер, пришло время использовать его для создания прогнозов. ML.NET предоставляет удобный API для создания прогнозов на одном экземпляре данных с именем [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602) .

1. Внутри `Main` метода создайте с [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602) помощью [`CreatePredictionEngine`](xref:Microsoft.ML.ModelOperationsCatalog.CreatePredictionEngine%2A) метода.

    ```csharp
    var onnxPredictionEngine = mlContext.Model.CreatePredictionEngine<OnnxInput, OnnxOutput>(onnxPredictionPipeline);
    ```

1. Создание входных данных теста.

    ```csharp
    var testInput = new OnnxInput
    {
        VendorId = "CMT",
        RateCode = 1,
        PassengerCount = 1,
        TripTimeInSecs = 1271,
        TripDistance = 3.8f,
        PaymentType = "CRD"
    };
    ```

1. Используйте `predictionEngine` для создания прогнозов на основе новых `testInput` данных с помощью [`Predict`](xref:Microsoft.ML.PredictionEngineBase%602.Predict%2A) метода.

    ```csharp
    var prediction = onnxPredictionEngine.Predict(testInput);
    ```

1. Выводит результат прогноза на консоль.

    ```csharp
    Console.WriteLine($"Predicted Fare: {prediction.PredictedFare.First()}");
    ```

1. Используйте .NET Core CLI для запуска приложения.

    ```dotnetcli
    dotnet run
    ```

    Результат должен выглядеть так же, как и следующие выходные данные:

    ```text
    Predicted Fare: 15.621523
    ```

Дополнительные сведения о создании прогнозов в ML.NET см. в разделе [Использование модели для создания прогнозов](/dotnet/machine-learning/how-to-guides/machine-learning-model-predictions-ml-net).

## <a name="next-steps"></a>Дальнейшие действия

- [Развертывание модели в качестве ASP.NET Core веб-API](/dotnet/machine-learning/how-to-guides/serve-model-web-api-ml-net)
- [Развертывание модели в качестве бессерверной функции Azure .NET](/dotnet/machine-learning/how-to-guides/serve-model-serverless-azure-functions-ml-net)
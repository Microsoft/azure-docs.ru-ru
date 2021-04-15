---
title: Краткое руководство. Клиентская библиотека Распознавателя документов для .NET
description: Использование клиентской библиотеки Распознавателя документов для .NET для создания приложения для обработки форм, которое извлекает из документов пары "ключ — значение" и табличные данные.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 03/19/2021
ms.author: lajanuar
ms.openlocfilehash: 6407f6af4b142333d2a52f60eb0b05024e64d88e
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104761102"
---
<!-- markdownlint-disable MD024 -->
> [!IMPORTANT]
> В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных.

[Справочная документация](/dotnet/api/overview/azure/ai.formrecognizer-readme) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/src) | [Пакет (NuGet)](https://www.nuget.org/packages/Azure.AI.FormRecognizer) | [Примеры](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [IDE Visual Studio](https://visualstudio.microsoft.com/vs/) или текущая версия [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Набор данных для обучения в хранилище BLOB-объектов Azure. Советы и варианты для объединения данных для обучения см. в статье о [создании обучающего набора данных для пользовательской модели](../../build-training-data-set.md). При работе с этим кратким руководством можно использовать файлы в папке **Train** из [примера набора данных](https://go.microsoft.com/fwlink/?linkid=2090451) (скачайте и разархивируйте файл *sample_data.zip*).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Создание ресурса Распознавателя документов"  target="_blank">созданию ресурса Распознавателя документов </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API Распознавателя документов потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `formrecognizer-quickstart`. Эта команда создает простой проект "Hello World" на языке C# с одним файлом исходного кода: *program.cs*.

```console
dotnet new console -n formrecognizer-quickstart
```

Измените каталог на созданную папку приложения. Чтобы создать приложение, выполните следующую команду:

```console
dotnet build
```

Выходные данные сборки не должны содержать предупреждений или ошибок. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

В каталоге приложения установите клиентскую библиотеку Распознавателя документов для .NET с помощью следующей команды:

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

```console
dotnet add package Azure.AI.FormRecognizer --version 3.1.0-beta.1
```

> [!NOTE]
> В пакете SDK 3.1.0 Распознавателя документов используется _API версии 2.1, предварительная версия 2_. Используйте [**REST API**](../../quickstarts/client-library.md) для _API версии 2.1, предварительная версия 3_.

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

```console
dotnet add package Azure.AI.FormRecognizer --version 3.0.0
```

> [!NOTE]
> В пакете SDK 3.0.0 Распознавателя документов используется API версии 2.0

---

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md), где размещены примеры кода для этого краткого руководства.

В каталоге проекта откройте файл *Program.cs* в предпочитаемом редакторе или интегрированной среде разработки. Затем добавьте следующие `using` директивы:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_using)]

В классе приложения **Program** создайте переменные для ключа и конечной точки вашего ресурса.

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Распознавателя документов, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_creds)]

В методе **Main** приложения добавьте вызов асинхронных задач, используемых в этом руководстве. Вы реализуете их позже.

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_main)]

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_main)]


---


## <a name="object-model"></a>Объектная модель 

С помощью Распознавателя документов можно создавать клиенты двух разных типов. Первый, `FormRecognizerClient`, используется для запроса из службы распознанных полей и содержимого форм. Второй, `FormTrainingClient`, используется для создания настраиваемых моделей и управления ими. Эти модели можно использовать для улучшения распознавания. 

### <a name="formrecognizerclient"></a>FormRecognizerClient

`FormRecognizerClient` предоставляет операции для перечисленных ниже целей.

* Распознавание полей форм и содержимого с помощью настраиваемых моделей, обученных для анализа пользовательских форм.  Эти значения возвращаются в коллекцию объектов `RecognizedForm`. См. пример в разделе об [анализе пользовательских форм](#analyze-forms-with-a-custom-model).
* Распознавание содержимого формы, в том числе таблиц, строк и слов, без необходимости обучения модели.  Содержимое форм возвращается в коллекцию объектов `FormPage`. См. пример [анализа макета](#analyze-layout).
* Распознавание общих полей в квитанциях для США с помощью предварительно обученной модели для обработки квитанций в службе "Распознаватель документов". Эти поля и метаданные возвращаются в коллекцию объектов `RecognizedForm`. См. пример [анализа квитанций](#analyze-receipts).

### <a name="formtrainingclient"></a>FormTrainingClient

`FormTrainingClient` предоставляет операции для перечисленных ниже целей.

* Обучение настраиваемых моделей для анализа всех полей и значений в пользовательских формах.  Возвращаемый объект `CustomFormModel` задает типы форм, которые будет анализировать модель, и поля, которые она будет извлекать для каждого из этих типов.
* Обучение пользовательских моделей для анализа конкретных полей и значений, указываемых путем добавления меток к пользовательским формам.  Возвращаемый объект `CustomFormModel` задает поля, которые будут извлечены моделью, а также предполагаемую точность для каждого поля.
* Управление моделями, созданными в учетной записи.
* Копирование настраиваемой модели из одного ресурса Распознавателя документов в другой.

См. примеры для [обучения модели](#train-a-custom-model) и [управления настраиваемыми моделями](#manage-custom-models).

> [!NOTE]
> Модели можно также обучить с помощью графического пользовательского интерфейса, например [средства маркировки Распознавателя документов](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Распознавателя документов для .NET:
<!-- markdownlint-disable MD001 -->

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

* [аутентификация клиента](#authenticate-the-client);
* [анализ макета](#analyze-layout);
* [анализ квитанций](#analyze-receipts);
* [анализ визитных карточек](#analyze-business-cards);
* [анализ счетов](#analyze-invoices);
* [обучение пользовательской модели](#train-a-custom-model);
* [анализ документов с помощью пользовательской модели](#analyze-forms-with-a-custom-model);
* [управление пользовательскими моделями](#manage-your-custom-models).

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

* [аутентификация клиента](#authenticate-the-client);
* [анализ макета](#analyze-layout);
* [анализ квитанций](#analyze-receipts);
* [обучение пользовательской модели](#train-a-custom-model);
* [анализ документов с помощью пользовательской модели](#analyze-forms-with-a-custom-model);
* [управление пользовательскими моделями](#manage-your-custom-models).

---

## <a name="authenticate-the-client"></a>Аутентификация клиента

Под методом **Main** создайте метод с именем `AuthenticateClient`. Он будет использоваться в других задачах для проверки подлинности запросов, направляемых в службу "Распознаватель документов". Этот метод использует объект `AzureKeyCredential`, чтобы при необходимости ключ API можно было обновить без создания новых клиентских объектов.

> [!IMPORTANT]
> Получите ключ и конечную точку на портале Azure. Если ресурс Распознавателя документов, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_auth)]

Повторите шаги выше для нового метода, который аутентифицирует клиент обучения.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_auth_training)]

## <a name="get-assets-for-testing"></a>Получение ресурсов для тестирования 

Также потребуется добавить URL-адреса данных для обучения и тестирования. Добавьте их в корень класса **Program**.

* [!INCLUDE [get SAS URL](../sas-instructions.md)]

   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Получение подписанного URL-адреса":::
* Чтобы получить подписанный URL-адрес отдельного документа в контейнере Хранилища BLOB-объектов, повторите описанные выше действия. Также сохраните его во временном расположении.
* Наконец, сохраните URL-адрес примеров изображений квитанции, приведенных ниже (также доступны на [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms)).

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_urls)]

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_urls)]

---

## <a name="analyze-layout"></a>Анализ макета

С помощью Распознавателя документов можно анализировать таблицы, строки и слова в документах без необходимости обучать модель. Возвращаемое значение представляет собой коллекцию объектов **FormPage**: по одному для каждой страницы обработанного документа. Дополнительные сведения об извлечении макетов см. в статье [Служба макета Распознавателя документов](../../concept-layout.md).

Для анализа содержимого файла по указанному URL-адресу используйте метод `StartRecognizeContentFromUri`.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_getcontent_call)]

> [!TIP]
> Вы можете также получить содержимое из локального файла. Изучите информацию о методах класса [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient), например о **StartRecognizeContent**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) для сценариев, включающих использование локальных изображений.

Оставшаяся часть этой задачи выводит на консоль сведения о содержимом.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_getcontent_print)]

### <a name="output"></a>Выходные данные

```console
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    Line 5 has 3 words, and text: '6000 Redmond, WA'.
    Line 6 has 3 words, and text: 'Sunnayvale, CA 87659'.
    Line 7 has 1 word, and text: '99243'.
    Line 8 has 2 words, and text: 'Invoice Number'.
    Line 9 has 2 words, and text: 'Invoice Date'.
    Line 10 has 3 words, and text: 'Invoice Due Date'.
    Line 11 has 1 word, and text: 'Charges'.
    Line 12 has 2 words, and text: 'VAT ID'.
    Line 13 has 1 word, and text: '34278587'.
    Line 14 has 1 word, and text: '6/18/2017'.
    Line 15 has 1 word, and text: '6/24/2017'.
    Line 16 has 1 word, and text: '$56,651.49'.
    Line 17 has 1 word, and text: 'PT'.
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    Cell (0, 3) contains text: 'Charges'.
    Cell (0, 5) contains text: 'VAT ID'.
    Cell (1, 0) contains text: '34278587'.
    Cell (1, 1) contains text: '6/18/2017'.
    Cell (1, 2) contains text: '6/24/2017'.
    Cell (1, 3) contains text: '$56,651.49'.
    Cell (1, 5) contains text: 'PT'.
```

## <a name="analyze-invoices"></a>Анализ счетов

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

В этом разделе показано, как с помощью предварительно обученной модели анализировать счета на продажу и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе счета см. в статье [Служба макета Распознавателя документов](../../concept-invoices.md).

Для анализа счетов по URL-адресу используйте метод `StartRecognizeInvoicesFromUriAsync`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_invoice_call)]

> [!TIP]
> Можно также проанализировать локальные изображения счетов. Изучите информацию о методах класса [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient), например о **StartRecognizeInvoices**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов `RecognizedForm`: по одному для каждого счета в отправленном документе. В приведенном ниже блоке кода обрабатывается счет с указанным универсальным кодом ресурса (URI) и все ее основные поля с соответствующими значениями выводятся в консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_invoice_print)]

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

---

## <a name="train-a-custom-model"></a>Обучение пользовательской модели

В этом разделе демонстрируется, как обучить модель на основе собственных данных. Обученная модель выводит структурированные данные, которые содержат связи типа "ключ/значение" из исходного документа формы. После обучения модели вы можете проверить ее, повторно обучить и, в конечном итоге, применить с целью надежного извлечения данных из нескольких форм в соответствии со своими потребностями.

> [!NOTE]
> Также можно обучить модель с графическим пользовательским интерфейсом, такой как [пример средства разметки Распознавателя документов](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Обучение моделей без меток

Вы можете обучить свою модель анализировать все поля и значения в ваших формах без необходимости вручную помечать документы, которые вы будете использовать для обучения. Приведенный ниже метод обучает модель на основе заданного набора документов и выводит ее статус в консоль. 


[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_train)]


Возвращаемый объект `CustomFormModel` содержит сведения о типах форм, которые может анализировать модель, и полях, которые она способна извлечь из формы каждого типа. Приведенный ниже блок кода выводит эту информацию в консоль.


[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_train_response)]

Наконец, возвратите идентификатор обученной модели для использования на следующих шагах.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_train_return)]

### <a name="output"></a>Выходные данные

Этот ответ усечен для удобства чтения.

```console
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    ...
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    Cell (0, 3) contains text: 'Charges'.
    ... 
Custom Model Info:
    Model Id: 95035721-f19d-40eb-8820-0c806b42798b
    Model Status: Ready
    Training model started on: 8/24/2020 6:36:44 PM +00:00
    Training model completed on: 8/24/2020 6:36:50 PM +00:00
Submodel Form Type: form-95035721-f19d-40eb-8820-0c806b42798b
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    ... 
Custom Model Info:
    Model Id: e7a1181b-1fb7-40be-bfbe-1ee154183633
    Model Status: Ready
    Training model started on: 8/24/2020 6:36:44 PM +00:00
    Training model completed on: 8/24/2020 6:36:52 PM +00:00
Submodel Form Type: form-0
    FieldName: field-0, FieldLabel: Additional Notes:
    FieldName: field-1, FieldLabel: Address:
    FieldName: field-2, FieldLabel: Company Name:
    FieldName: field-3, FieldLabel: Company Phone:
    FieldName: field-4, FieldLabel: Dated As:
    FieldName: field-5, FieldLabel: Details
    FieldName: field-6, FieldLabel: Email:
    FieldName: field-7, FieldLabel: Hero Limited
    FieldName: field-8, FieldLabel: Name:
    FieldName: field-9, FieldLabel: Phone:
    ...
```

### <a name="train-a-model-with-labels"></a>Обучение моделей с метками

Модели также можно обучать на документах, которые размечены вручную. В некоторых сценариях обучение по такой схеме приводит к лучшим результатам. Для обучения с метками в контейнере хранилища BLOB-объектов вместе с документами, используемыми для обучения, должны находиться особые файлы с информацией о метках (`\<filename\>.pdf.labels.json`). Эти файлы можно создать в пользовательском интерфейсе [средства разметки Распознавателя документов](../../quickstarts/label-tool.md). Создав файлы, вызовите метод `StartTrainingAsync` и задайте для параметра `uselabels` значение `true`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_trainlabels)]

Возвращаемый объект `CustomFormModel` содержит поля, которые модель способна извлечь из документа, с указанием ориентировочной точности для каждого из них. Приведенный ниже блок кода выводит эту информацию в консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_trainlabels_response)]

### <a name="output"></a>Выходные данные

Этот ответ усечен для удобства чтения.

```console
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    Line 5 has 3 words, and text: '6000 Redmond, WA'.
    ...
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    ... 
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
Custom Model Info:
    Model Id: 63c013e3-1cab-43eb-84b0-f4b20cb9214c
    Model Status: Ready
    Training model started on: 8/24/2020 6:42:54 PM +00:00
    Training model completed on: 8/24/2020 6:43:01 PM +00:00
Submodel Form Type: form-63c013e3-1cab-43eb-84b0-f4b20cb9214c
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    FieldName: DatedAs
    FieldName: Email
    FieldName: Merchant
    ... 
```

## <a name="analyze-forms-with-a-custom-model"></a>анализ документов с помощью пользовательской модели;

В этом разделе показано, как извлекать данные вида "ключ/значение" и другое содержимое из пользовательских форм с помощью обученных на них моделей.

> [!IMPORTANT]
> Для реализации этого сценария у вас уже должна быть обученная модель, идентификатор которой нужно будет передать указанному ниже методу.

Используйте метод `StartRecognizeCustomFormsFromUri`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_analyze)]

> [!TIP]
> Вы можете также проанализировать локальные файлы. Изучите информацию о методах класса [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient), например о **StartRecognizeCustomForms**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов `RecognizedForm`: по одному для каждой страницы обработанного документа. Следующий код выводит в консоль результаты анализа. Он отображает каждое из распознанных полей с соответствующим значением и коэффициентом достоверности.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_analyze_response)]

### <a name="output"></a>Выходные данные

Этот ответ усечен для удобства чтения.

```console
Custom Model Info:
    Model Id: 9b0108ee-65c8-450e-b527-bb309d054fc4
    Model Status: Ready
    Training model started on: 8/24/2020 7:00:31 PM +00:00
    Training model completed on: 8/24/2020 7:00:32 PM +00:00
Submodel Form Type: form-9b0108ee-65c8-450e-b527-bb309d054fc4
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    ...
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    ...

Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    ...
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
Custom Model Info:
    Model Id: dc115156-ce0e-4202-bbe4-7426e7bee756
    Model Status: Ready
    Training model started on: 8/24/2020 7:00:31 PM +00:00
    Training model completed on: 8/24/2020 7:00:41 PM +00:00
Submodel Form Type: form-0
    FieldName: field-0, FieldLabel: Additional Notes:
    FieldName: field-1, FieldLabel: Address:
    FieldName: field-2, FieldLabel: Company Name:
    FieldName: field-3, FieldLabel: Company Phone:
    FieldName: field-4, FieldLabel: Dated As:
    ... 
Form of type: custom:form
Field 'Azure.AI.FormRecognizer.Models.FieldValue:
    Value: '$56,651.49
    Confidence: '0.249
Field 'Azure.AI.FormRecognizer.Models.FieldValue:
    Value: 'PT
    Confidence: '0.245
Field 'Azure.AI.FormRecognizer.Models.FieldValue:
    Value: '99243
    Confidence: '0.114
   ...
```

## <a name="analyze-receipts"></a>Анализ квитанций

В этом разделе объясняется, как с помощью предварительно обученной модели можно анализировать используемые в США квитанции и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе квитанций см. в статье [Готовая модель получения Распознавателя форм](../../concept-receipts.md).

Для анализа чеков по URL-адресу используйте метод `StartRecognizeReceiptsFromUri`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_receipt_call)]

> [!TIP]
> Можно также проанализировать локальные изображения квитанций. Изучите информацию о методах класса [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient), например о **StartRecognizeReceipts**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов `RecognizedReceipt`: по одному для каждой страницы обработанного документа. В следующем блоке кода обрабатывается квитанция с указанным универсальным кодом ресурса (URI) и все ее основные поля с соответствующими значениями выводятся в консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_receipt_print)]

### <a name="output"></a>Выходные данные 

```console
Form Page 1 has 18 lines.
    Line 0 has 1 word, and text: 'Contoso'.
    Line 1 has 1 word, and text: 'Address:'.
    Line 2 has 3 words, and text: 'Invoice For: Microsoft'.
    Line 3 has 4 words, and text: '1 Redmond way Suite'.
    Line 4 has 3 words, and text: '1020 Enterprise Way'.
    Line 5 has 3 words, and text: '6000 Redmond, WA'.
    Line 6 has 3 words, and text: 'Sunnayvale, CA 87659'.
    Line 7 has 1 word, and text: '99243'.
    Line 8 has 2 words, and text: 'Invoice Number'.
    Line 9 has 2 words, and text: 'Invoice Date'.
    Line 10 has 3 words, and text: 'Invoice Due Date'.
    Line 11 has 1 word, and text: 'Charges'.
    Line 12 has 2 words, and text: 'VAT ID'.
    Line 13 has 1 word, and text: '34278587'.
    Line 14 has 1 word, and text: '6/18/2017'.
    Line 15 has 1 word, and text: '6/24/2017'.
    Line 16 has 1 word, and text: '$56,651.49'.
    Line 17 has 1 word, and text: 'PT'.
Table 0 has 2 rows and 6 columns.
    Cell (0, 0) contains text: 'Invoice Number'.
    Cell (0, 1) contains text: 'Invoice Date'.
    Cell (0, 2) contains text: 'Invoice Due Date'.
    Cell (0, 3) contains text: 'Charges'.
    Cell (0, 5) contains text: 'VAT ID'.
    Cell (1, 0) contains text: '34278587'.
    Cell (1, 1) contains text: '6/18/2017'.
    Cell (1, 2) contains text: '6/24/2017'.
    Cell (1, 3) contains text: '$56,651.49'.
    Cell (1, 5) contains text: 'PT'.
Merchant Name: 'Contoso Contoso', with confidence 0.516
Transaction Date: '6/10/2019 12:00:00 AM', with confidence 0.985
Item:
    Name: '8GB RAM (Black)', with confidence 0.916
    Total Price: '999', with confidence 0.559
Item:
    Name: 'SurfacePen', with confidence 0.858
    Total Price: '99.99', with confidence 0.386
Total: '1203.39', with confidence '0.774'
```

## <a name="analyze-business-cards"></a>Анализ визитных карточек

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

В этом разделе показано, как с помощью предварительно обученной модели анализировать визитные карточки на английском языке и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе визитных карточек см. в статье [Готовая модель для анализа визитных карточек с помощью Распознавателя форм](../../concept-business-cards.md).

Для анализа визитных карточек по URL-адресу используйте метод `StartRecognizeBusinessCardsFromUriAsync`. 

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_bc_call)]

> [!TIP]
> Можно также проанализировать локальные изображения квитанций. Изучите информацию о методах класса [FormRecognizerClient](/dotnet/api/azure.ai.formrecognizer.formrecognizerclient), например о **StartRecognizeBusinessCards**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов `RecognizedForm`: по одному для каждой карточки в документе. В следующем блоке кода обрабатывается визитная карточка с указанным универсальным кодом ресурса (URI) и все ее основные поля с соответствующими значениями выводятся в консоль.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart-preview.cs?name=snippet_bc_print)]

---

## <a name="manage-custom-models"></a>Управление пользовательскими моделями

В этом разделе показано, как управлять пользовательскими моделями, которые хранятся в вашей учетной записи. В следующем методе вы выполните несколько операций:

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage)]


### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Проверка числа моделей в учетной записи ресурсов FormRecognizer

Приведенный ниже блок кода проверяет, сколько моделей сохранено в вашей учетной записи Распознавателя документов, и сравнивает это значение с ограничением для данной учетной записи.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_count)]

### <a name="output"></a>Выходные данные 

```console
Account has 20 models.
It can have at most 5000 models.
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Вывод списка моделей, которые хранятся в учетной записи ресурсов

Приведенный ниже блок кода выводит в консоль список текущих моделей в вашей учетной записи с подробными сведениями о них.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_list)]


### <a name="output"></a>Выходные данные 

Этот ответ усечен для удобства чтения.

```console
Custom Model Info:
    Model Id: 05932d5a-a2f8-4030-a2ef-4e5ed7112515
    Model Status: Creating
    Training model started on: 8/24/2020 7:35:02 PM +00:00
    Training model completed on: 8/24/2020 7:35:02 PM +00:00
Custom Model Info:
    Model Id: 150828c4-2eb2-487e-a728-60d5d504bd16
    Model Status: Ready
    Training model started on: 8/24/2020 7:33:25 PM +00:00
    Training model completed on: 8/24/2020 7:33:27 PM +00:00
Custom Model Info:
    Model Id: 3303e9de-6cec-4dfb-9e68-36510a6ecbb2
    Model Status: Ready
    Training model started on: 8/24/2020 7:29:27 PM +00:00
    Training model completed on: 8/24/2020 7:29:36 PM +00:00
```

### <a name="get-a-specific-model-using-the-models-id"></a>Получение определенной модели по ее идентификатору

Приведенный ниже блок кода обучает новую модель (как описано в разделе [Обучение модели](#train-a-model-without-labels)), а затем получает вторую ссылку на нее по ее идентификатору.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_get)]

### <a name="output"></a>Выходные данные 

Этот ответ усечен для удобства чтения.

```console
Custom Model Info:
    Model Id: 150828c4-2eb2-487e-a728-60d5d504bd16
    Model Status: Ready
    Training model started on: 8/24/2020 7:33:25 PM +00:00
    Training model completed on: 8/24/2020 7:33:27 PM +00:00
Submodel Form Type: form-150828c4-2eb2-487e-a728-60d5d504bd16
    FieldName: CompanyAddress
    FieldName: CompanyName
    FieldName: CompanyPhoneNumber
    FieldName: DatedAs
    FieldName: Email
    FieldName: Merchant
    FieldName: PhoneNumber
    FieldName: PurchaseOrderNumber
    FieldName: Quantity
    FieldName: Signature
    FieldName: Subtotal
    FieldName: Tax
    FieldName: Total
    FieldName: VendorName
    FieldName: Website
...
```

### <a name="delete-a-model-from-the-resource-account"></a>Удаление модели из учетной записи ресурсов

Модель можно удалить из учетной записи ресурсов по ее идентификатору. В рамках этого шага мы также закроем метод.

[!code-csharp[](~/cognitive-services-quickstart-code/dotnet/FormRecognizer/FormRecognizerQuickstart.cs?name=snippet_manage_model_delete)]


## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```dotnet
dotnet run
```


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Устранение неполадок

Ошибки, возвращаемые службой при взаимодействии с клиентской библиотекой Распознавателя документов Cognitive Services с помощью .NET SDK, приводят к исключению `RequestFailedException`. В сообщении об ошибке указывается тот же код состояния HTTP, что был бы возвращен запросом API REST.

Например, если отправить изображение квитанции с недействительным URI, служба вернет ошибку `400` "Недопустимый запрос".

```csharp
try
{
    RecognizedReceiptCollection receipts = await client.StartRecognizeReceiptsFromUri(new Uri(receiptUri)).WaitForCompletionAsync();
}
catch (RequestFailedException e)
{
    Console.WriteLine(e.ToString());
}
```

В журнале вы также найдете дополнительную информацию, такую как идентификатор запроса клиента данной операции.

```
Message:
    Azure.RequestFailedException: Service request failed.
    Status: 400 (Bad Request)

Content:
    {"error":{"code":"FailedToDownloadImage","innerError":
    {"requestId":"8ca04feb-86db-4552-857c-fde903251518"},
    "message":"Failed to download image from input URL."}}

Headers:
    Transfer-Encoding: chunked
    x-envoy-upstream-service-time: REDACTED
    apim-request-id: REDACTED
    Strict-Transport-Security: REDACTED
    X-Content-Type-Options: REDACTED
    Date: Mon, 20 Apr 2020 22:48:35 GMT
    Content-Type: application/json; charset=utf-8
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано использование клиентской библиотеки .NET Распознавателя документов для обучения модели и анализа документов различными способами. Теперь следует изучить советы по созданию лучшего набора данных для обучения и созданию более точных моделей.

> [!div class="nextstepaction"]
> [Создание набора данных для обучения](../../build-training-data-set.md)

* [Что такое Распознаватель документов?](../../overview.md)
* Пример кода из этого руководства, а также другие ресурсы можно найти на [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md).

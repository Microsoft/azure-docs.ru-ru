---
title: Краткое руководство. Клиентская библиотека Распознавателя документов для Python
description: Использование клиентской библиотеки Распознавателя документов для Python для создания приложения для обработки форм, которое извлекает из документов пары "ключ — значение" и табличные данные.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 03/19/2021
ms.author: lajanuar
ms.openlocfilehash: f28fba451a5fbad9efe76e0d6bd39488beebb103
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104761167"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
> [!IMPORTANT]
>
> * В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных. См. справочную документацию ниже. 

[Справочная документация](/python/api/azure-ai-formrecognizer) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/azure/ai/formrecognizer) | [Пакет (PyPi)](https://pypi.org/project/azure-ai-formrecognizer/) | [Примеры](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [Python 3.x](https://www.python.org/)
  * Установка Python должна включать [pip](https://pip.pypa.io/en/stable/). Чтобы проверить, установлен ли pip, выполните команду `pip --version` в командной строке. Чтобы использовать pip, установите последнюю версию Python.
* Набор данных для обучения в хранилище BLOB-объектов Azure. Советы и варианты для объединения данных для обучения см. в статье о [создании обучающего набора данных для пользовательской модели](../../build-training-data-set.md). При работе с этим кратким руководством можно использовать файлы в папке **Train** из [примера набора данных](https://go.microsoft.com/fwlink/?linkid=2090451) (скачайте и разархивируйте файл *sample_data.zip*).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Создание ресурса Распознавателя документов"  target="_blank">созданию ресурса Распознавателя документов </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API Распознавателя документов потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python можно установить последнюю версию клиентской библиотеки Распознавателя документов с помощью приведенной ниже команды.

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

```console
pip install azure-ai-formrecognizer --pre
```

> [!NOTE]
> В пакете SDK 3.1.0 Распознавателя документов используется _API версии 2.1, предварительная версия 2_. Используйте [**REST API**](../../quickstarts/client-library.md) для _API версии 2.1, предварительная версия 3_.

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

```console
pip install azure-ai-formrecognizer
```

> [!NOTE]
> В пакете SDK 3.0.0 Распознавателя документов используется API версии 2.0

---

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте приложение Python в предпочитаемом редакторе или среде интегрированной разработки. Затем импортируйте приведенные ниже библиотеки.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/FormRecognizerQuickstart.py), где размещены примеры кода для этого краткого руководства.

Создайте переменные для конечной точки Azure и ключа ресурса. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_creds)]

## <a name="object-model"></a>Объектная модель

С помощью Распознавателя документов можно создавать клиенты двух разных типов. Первый, `form_recognizer_client`, используется для запроса из службы распознанных полей и содержимого форм. Второй, `form_training_client`, используется для создания настраиваемых моделей и управления ими. Эти модели можно использовать для улучшения распознавания. 

### <a name="formrecognizerclient"></a>FormRecognizerClient

`form_recognizer_client` предоставляет операции для перечисленных ниже целей.

* Распознавание полей форм и содержимого с помощью настраиваемых моделей, обученных для анализа пользовательских форм.
* Распознавание содержимого формы, в том числе таблиц, строк и слов, без необходимости обучения модели.
* Распознавание общих полей в квитанциях с помощью предварительно обученной модели для обработки квитанций в службе "Распознаватель документов".

### <a name="formtrainingclient"></a>FormTrainingClient

`form_training_client` предоставляет операции для перечисленных ниже целей.

* Обучение настраиваемых моделей для анализа всех полей и значений в пользовательских формах. Более подробное описание процесса создания набора данных для обучения см. в [документации службы по обучению модели без добавления меток](#train-a-model-without-labels).
* Обучение пользовательских моделей для анализа конкретных полей и значений, указываемых путем добавления меток к пользовательским формам. Более подробное объяснение применения меток к набору данных для обучения см. в [документации службы по обучению модели с добавлением меток](#train-a-model-with-labels).
* Управление моделями, созданными в учетной записи.
* Копирование настраиваемой модели из одного ресурса Распознавателя документов в другой.

> [!NOTE]
> Модели можно также обучить с помощью графического пользовательского интерфейса, например [средства маркировки Распознавателя документов](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Распознавателя документов для Python:
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
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

В этом фрагменте выполняется проверка подлинности двух клиентских объектов с использованием переменных подписки, указанных выше. Здесь применяется объект **AzureKeyCredential**, чтобы при необходимости ключ API можно было обновить, не создавая новых клиентских объектов.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_auth)]

## <a name="get-assets-for-testing"></a>Получение ресурсов для тестирования

Потребуется добавить URL-адреса данных для обучения и тестирования.

* [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]
  
   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Получение подписанного URL-адреса":::
* Используйте пример формы и изображения квитанций, включенные в приведенные ниже примеры. Вы можете также скачать их с [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms) или выполнить описанные выше действия, чтобы получить URL-адрес SAS для отдельного документа в хранилище BLOB-объектов. 

> [!NOTE]
> Во фрагментах кода в этом руководстве используются удаленные формы, доступ к которым осуществляется по URL-адресам. Если вместо этого вы планируете работать с локальными документами, воспользуйтесь соответствующими методами, которые описаны в [справочной документации](/python/api/azure-ai-formrecognizer) и [образцах](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples).

## <a name="analyze-layout"></a>Анализ макета

С помощью Распознавателя документов можно анализировать таблицы, строки и слова в документах без необходимости обучать модель. Дополнительные сведения об извлечении макетов см. в статье [Служба макета Распознавателя документов](../../concept-layout.md).

Для анализа содержимого файла по указанному URL-адресу используйте метод `begin_recognize_content_from_url`. Возвращаемое значение представляет собой коллекцию объектов `FormPage`: по одному для каждой страницы обработанного документа. Приведенный ниже код обрабатывает эти объекты в цикле и выводит извлеченные из документа пары ключей и значений, а также табличные данные.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_getcontent)]

> [!TIP]
> Можно также получить содержимое из локальных изображений. Изучите информацию о методах класса [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), например о `begin_recognize_content`. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) для сценариев, включающих использование локальных изображений.

### <a name="output"></a>Выходные данные

```console
Table found on page 1:
Cell text: Invoice Number
Location: [Point(x=0.5075, y=2.8088), Point(x=1.9061, y=2.8088), Point(x=1.9061, y=3.3219), Point(x=0.5075, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Date
Location: [Point(x=1.9061, y=2.8088), Point(x=3.3074, y=2.8088), Point(x=3.3074, y=3.3219), Point(x=1.9061, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Due Date
Location: [Point(x=3.3074, y=2.8088), Point(x=4.7074, y=2.8088), Point(x=4.7074, y=3.3219), Point(x=3.3074, y=3.3219)]
Confidence score: 1.0

Cell text: Charges
Location: [Point(x=4.7074, y=2.8088), Point(x=5.386, y=2.8088), Point(x=5.386, y=3.3219), Point(x=4.7074, y=3.3219)]
Confidence score: 1.0
...

```

## <a name="analyze-invoices"></a>Анализ счетов

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

В этом разделе показано, как с помощью предварительно обученной модели анализировать счета на продажу и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе счета см. в статье [Служба макета Распознавателя документов](../../concept-invoices.md). Для анализа счетов по URL-адресу используйте метод `begin_recognize_invoices_from_url`. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py?name=snippet_invoice)]

> [!TIP]
> Можно также проанализировать локальные изображения счетов. Изучите информацию о методах класса [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), например о `begin_recognize_invoices`. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) для сценариев, включающих использование локальных изображений.

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

---

## <a name="train-a-custom-model"></a>Обучение пользовательской модели

В этом разделе демонстрируется, как обучить модель на основе собственных данных. Обученная модель выводит структурированные данные, которые содержат связи типа "ключ/значение" из исходного документа формы. После обучения модели вы можете проверить ее, повторно обучить и, в конечном итоге, применить с целью надежного извлечения данных из нескольких форм в соответствии со своими потребностями.

> [!NOTE]
> Также можно обучить модель с графическим пользовательским интерфейсом, такой как [пример средства разметки Распознавателя документов](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Обучение моделей без меток

Вы можете обучить свою модель анализировать все поля и значения в ваших формах без необходимости вручную помечать документы, которые вы будете использовать для обучения.

В следующем фрагменте кода с помощью клиента обучения с функцией `begin_training` выполняется обучение модели на основе заданного набора документов. Возвращаемый объект `CustomFormModel` содержит сведения о типах форм, которые может анализировать модель, и полях, которые она способна извлечь из формы каждого типа. Приведенный ниже блок кода выводит эту информацию в консоль.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_train)]


### <a name="output"></a>Выходные данные

Это выходные данные для модели, обученной по данным для обучения, доступным в [пакете SDK для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: 628739de-779c-473d-8214-d35c72d3d4f7
Status: ready
Training started on: 2020-08-20 23:16:51+00:00
Training completed on: 2020-08-20 23:16:59+00:00

Recognized fields:
The submodel with form type 'form-0' has recognized the following fields: Additional Notes:, Address:, Company Name:, Company Phone:, Dated As:, Details, Email:, Hero Limited, Name:, Phone:, Purchase Order, Purchase Order #:, Quantity, SUBTOTAL, Seattle, WA 93849 Phone:, Shipped From, Shipped To, TAX, TOTAL, Total, Unit Price, Vendor Name:, Website:
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

### <a name="train-a-model-with-labels"></a>Обучение моделей с метками

Модели также можно обучать на документах, которые размечены вручную. В некоторых сценариях обучение по такой схеме приводит к лучшим результатам. Возвращаемый объект `CustomFormModel` содержит поля, которые модель способна извлечь из документа, с указанием ориентировочной точности для каждого из них. Приведенный ниже блок кода выводит эту информацию в консоль.

> [!IMPORTANT]
> Для обучения с метками в контейнере хранилища BLOB-объектов вместе с документами, используемыми для обучения, должны находиться особые файлы с информацией о метках (`\<filename\>.pdf.labels.json`). Эти файлы можно создать в пользовательском интерфейсе [средства разметки Распознавателя документов](../../quickstarts/label-tool.md). Создав файлы, вызовите функцию `begin_training` и задайте для параметра *use_training_labels* значение `true`.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_trainlabels)]

### <a name="output"></a>Выходные данные

Это выходные данные для модели, обученной по данным для обучения, доступным в [пакете SDK для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91

Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00

Recognized fields:
The submodel with form type 'form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91' has recognized the following fields: CompanyAddress, CompanyName, CompanyPhoneNumber, DatedAs, Email, Merchant, PhoneNumber, PurchaseOrderNumber, Quantity, Signature, Subtotal, Tax, Total, VendorName, Website
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

## <a name="analyze-forms-with-a-custom-model"></a>анализ документов с помощью пользовательской модели;

В этом разделе показано, как извлекать данные вида "ключ/значение" и другое содержимое из пользовательских форм с помощью обученных на них моделей.

> [!IMPORTANT]
> Для реализации этого сценария у вас уже должна быть обученная модель, идентификатор которой нужно будет передать указанному ниже методу. См. раздел [Обучение модели](#train-a-model-without-labels).

Используйте метод `begin_recognize_custom_forms_from_url`. Возвращаемое значение представляет собой коллекцию объектов `RecognizedForm`: по одному для каждой страницы обработанного документа. Следующий код выводит в консоль результаты анализа. Он отображает каждое из распознанных полей с соответствующим значением и коэффициентом достоверности.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_analyze)]

> [!TIP]
> Можно также проанализировать локальные изображения. Изучите информацию о методах класса [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), например о `begin_recognize_custom_forms`. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) для сценариев, включающих использование локальных изображений.


### <a name="output"></a>Выходные данные

При использовании модели из предыдущего примера предоставляются приведенные ниже выходные данные.

```console
Form type: form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Field 'Merchant' has label 'Merchant' with value 'Invoice For:' and a confidence score of 0.116
Field 'CompanyAddress' has label 'CompanyAddress' with value '1 Redmond way Suite 6000 Redmond, WA' and a confidence score of 0.258
Field 'Website' has label 'Website' with value '99243' and a confidence score of 0.114
Field 'VendorName' has label 'VendorName' with value 'Charges' and a confidence score of 0.145
Field 'CompanyPhoneNumber' has label 'CompanyPhoneNumber' with value '$56,651.49' and a confidence score of 0.249
Field 'CompanyName' has label 'CompanyName' with value 'PT' and a confidence score of 0.245
Field 'DatedAs' has label 'DatedAs' with value 'None' and a confidence score of None
Field 'Email' has label 'Email' with value 'None' and a confidence score of None
Field 'PhoneNumber' has label 'PhoneNumber' with value 'None' and a confidence score of None
Field 'PurchaseOrderNumber' has label 'PurchaseOrderNumber' with value 'None' and a confidence score of None
Field 'Quantity' has label 'Quantity' with value 'None' and a confidence score of None
Field 'Signature' has label 'Signature' with value 'None' and a confidence score of None
Field 'Subtotal' has label 'Subtotal' with value 'None' and a confidence score of None
Field 'Tax' has label 'Tax' with value 'None' and a confidence score of None
Field 'Total' has label 'Total' with value 'None' and a confidence score of None
```

## <a name="analyze-receipts"></a>Анализ квитанций

В этом разделе объясняется, как с помощью предварительно обученной модели можно анализировать используемые в США квитанции и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе квитанций см. в статье [Готовая модель получения Распознавателя форм](../../concept-receipts.md). Для анализа чеков по URL-адресу используйте метод `begin_recognize_receipts_from_url`. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_receipts)]

> [!TIP]
> Можно также проанализировать локальные изображения квитанций. Изучите информацию о методах класса [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), например о `begin_recognize_receipts`. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) для сценариев, включающих использование локальных изображений.

### <a name="output"></a>Выходные данные

```console
ReceiptType: Itemized has confidence 0.659
MerchantName: Contoso Contoso has confidence 0.516
MerchantAddress: 123 Main Street Redmond, WA 98052 has confidence 0.986
MerchantPhoneNumber: None has confidence 0.99
TransactionDate: 2019-06-10 has confidence 0.985
TransactionTime: 13:59:00 has confidence 0.968
Receipt Items:
...Item #1
......Name: 8GB RAM (Black) has confidence 0.916
......TotalPrice: 999.0 has confidence 0.559
...Item #2
......Quantity: None has confidence 0.858
......Name: SurfacePen has confidence 0.858
......TotalPrice: 99.99 has confidence 0.386
Subtotal: 1098.99 has confidence 0.964
Tax: 104.4 has confidence 0.713
Total: 1203.39 has confidence 0.774
```

## <a name="analyze-business-cards"></a>Анализ визитных карточек

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

В этом разделе показано, как с помощью предварительно обученной модели анализировать визитные карточки на английском языке и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе визитных карточек см. в статье [Готовая модель для анализа визитных карточек с помощью Распознавателя форм](../../concept-business-cards.md). Для анализа визитных карточек по URL-адресу используйте метод `begin_recognize_business_cards_from_url`. 

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart-preview.py?name=snippet_bc)]

> [!TIP]
> Можно также проанализировать локальные изображения визитных карточек. Изучите информацию о методах класса [FormRecognizerClient](/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer.formrecognizerclient), например о `begin_recognize_business_cards`. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples) для сценариев, включающих использование локальных изображений.

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

---

## <a name="manage-your-custom-models"></a>Управление пользовательскими моделями

В этом разделе показано, как управлять пользовательскими моделями, которые хранятся в вашей учетной записи. 

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Проверка числа моделей в учетной записи ресурсов FormRecognizer

Приведенный ниже блок кода проверяет, сколько моделей сохранено в вашей учетной записи Распознавателя документов, и сравнивает это значение с ограничением для данной учетной записи.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_count)]


### <a name="output"></a>Выходные данные

```console
Our account has 5 custom models, and we can have at most 5000 custom models
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Вывод списка моделей, которые хранятся в учетной записи ресурсов

Приведенный ниже блок кода выводит в консоль список текущих моделей в вашей учетной записи с подробными сведениями о них. Он также сохраняет ссылку на первую модель.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_list)]


### <a name="output"></a>Выходные данные

Это пример выходных данных для тестовой учетной записи.

```console
We have models with the following ids:
453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
628739de-779c-473d-8214-d35c72d3d4f7
ae636292-0b14-4e26-81a7-a0bfcbaf7c91
b4b5df77-8538-4ffb-a996-f67158ecd305
c6309148-6b64-4fef-aea0-d39521452699
```

### <a name="get-a-specific-model-using-the-models-id"></a>Получение определенной модели по ее идентификатору

Следующий блок кода получает подробные сведения о модели по ее идентификатору, сохраненному в предыдущем разделе.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_getmodel)]


### <a name="output"></a>Выходные данные

Это пример выходных данных для настраиваемой модели, созданной в предыдущем примере.

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00
```

### <a name="delete-a-model-from-the-resource-account"></a>Удаление модели из учетной записи ресурсов

Модель можно удалить из учетной записи ресурсов по ее идентификатору. Этот фрагмент кода удаляет модель, которая использовалась в предыдущем разделе.

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerQuickstart.py?name=snippet_manage_delete)]


## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `python` для файла quickstart.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="general"></a>Общие сведения

Клиентская библиотека Распознавателя документов вызывает исключения, определенные в [Azure Core](https://aka.ms/azsdk-python-azure-core).

### <a name="logging"></a>Logging

Эта библиотека использует для ведения журнала [соответствующую стандартную библиотеку](https://docs.python.org/3/library/logging.html). На уровне информации (INFO) в журнал заносится основная информация о сеансах HTTP (URL-адреса, заголовки и т. д.).

С помощью аргумента-ключевого слова `logging_enable` можно включить в клиенте ведение журнала на уровне отладки (DEBUG), на котором фиксируются сведения о телах запросов и ответов, а также заголовки без изменений:

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerLogging.py?name=snippet_logging)]


Аналогичным образом с помощью параметра `logging_enable` можно включить подробное журналирование для отдельной операции (даже если этот режим не включен в клиенте):

[!code-python[](~/cognitive-services-quickstart-code/python/FormRecognizer/FormRecognizerLogging.py?name=snippet_example)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано использование клиентской библиотеки Распознавателя документов на языке Python для обучения модели и анализа документов различными способами. Теперь следует изучить советы по созданию лучшего набора данных для обучения и созданию более точных моделей.

> [!div class="nextstepaction"]
> [Создание набора данных для обучения](../../build-training-data-set.md)

* [Что такое Распознаватель документов?](../../overview.md)
* Пример кода из этого руководства можно найти на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/FormRecognizerQuickstart.py).

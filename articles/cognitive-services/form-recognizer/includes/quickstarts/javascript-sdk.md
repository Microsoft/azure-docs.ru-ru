---
title: Краткое руководство. Клиентская библиотека Распознавателя документов для JavaScript
description: Использование клиентской библиотеки Распознавателя документов для JavaScript для создания приложения для обработки форм, которое извлекает из документов пары "ключ — значение" и табличные данные.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 10/26/2020
ms.author: lajanuar
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 52acb6d0e99d54491dc1ce30545629ba246acc77
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102445669"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
> [!IMPORTANT]
>
> * В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных. См. справочную документацию ниже. 

[Справочная документация](../../index.yml) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/formrecognizer/ai-form-recognizer/) | [Пакет (npm)](https://www.npmjs.com/package/@azure/ai-form-recognizer) | [Примеры](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Текущая версия [Node.js](https://nodejs.org/)
* Набор данных для обучения в хранилище BLOB-объектов Azure. Советы и варианты для объединения данных для обучения см. в статье о [создании обучающего набора данных для пользовательской модели](../../build-training-data-set.md). При работе с этим кратким руководством можно использовать файлы в папке **Train** из [примера набора данных](https://go.microsoft.com/fwlink/?linkid=2090451) (скачайте и разархивируйте файл *sample_data.zip*).
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Создание ресурса Распознавателя документов"  target="_blank">созданию ресурса Распознавателя документов </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API Распознавателя документов потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `npm init`, чтобы создать приложение узла с помощью файла `package.json`. 

```console
npm init
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Установите пакет NPM `ai-form-recognizer`.

```console
npm install @azure/ai-form-recognizer
```

Файл `package.json` этого приложения будет дополнен зависимостями.

Создайте файл с именем `index.js`, откройте его и импортируйте следующие библиотеки:

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/FormRecognizer/FormRecognizerQuickstart.js), где размещены примеры кода для этого краткого руководства.

Создайте переменные для конечной точки Azure и ключа ресурса. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_creds)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Распознавателя документов, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

## <a name="object-model"></a>Объектная модель

С помощью Распознавателя документов можно создавать клиенты двух разных типов. Первый, `FormRecognizerClient`, используется для запроса из службы распознанных полей и содержимого форм. Второй, `FormTrainingClient`, используется для создания настраиваемых моделей и управления ими. Эти модели можно использовать для улучшения распознавания. 

### <a name="formrecognizerclient"></a>FormRecognizerClient

`FormRecognizerClient` предоставляет операции для перечисленных ниже целей.

* Распознавание полей форм и содержимого с помощью настраиваемых моделей, обученных для анализа пользовательских форм. Эти значения возвращаются в коллекцию объектов `RecognizedForm`.
* Распознавание содержимого формы, в том числе таблиц, строк и слов, без необходимости обучения модели. Содержимое форм возвращается в коллекцию объектов `FormPage`.
* Распознавание общих полей в квитанциях с помощью предварительно обученной модели для обработки квитанций в службе "Распознаватель документов". Эти поля и метаданные возвращаются в коллекцию `RecognizedReceipt`.

### <a name="formtrainingclient"></a>FormTrainingClient

`FormTrainingClient` предоставляет операции для перечисленных ниже целей.

* Обучение настраиваемых моделей для анализа всех полей и значений в пользовательских формах. Возвращаемый объект `CustomFormModel` задает типы форм, которые будет анализировать модель, и поля, которые она будет извлекать для каждого из этих типов. Более подробное описание процесса создания набора данных для обучения см. в [документации службы по обучению модели без добавления меток](#train-a-model-without-labels).
* Обучение пользовательских моделей для анализа конкретных полей и значений, указываемых путем добавления меток к пользовательским формам. Возвращаемый объект `CustomFormModel` задает поля, которые будут извлечены моделью, а также предполагаемую точность для каждого поля. Более подробное объяснение применения меток к набору данных для обучения см. в [документации службы по обучению модели с добавлением меток](#train-a-model-with-labels).
* Управление моделями, созданными в учетной записи.
* Копирование настраиваемой модели из одного ресурса Распознавателя документов в другой.

> [!NOTE]
> Модели можно также обучить с помощью графического пользовательского интерфейса, например [средства маркировки Распознавателя документов](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Распознавателя документов для JavaScript:

* [аутентификация клиента](#authenticate-the-client);
* [анализ макета](#analyze-layout);
* [анализ квитанций](#analyze-receipts);
* [обучение пользовательской модели](#train-a-custom-model);
* [анализ документов с помощью пользовательской модели](#analyze-forms-with-a-custom-model);
* [управление пользовательскими моделями](#manage-your-custom-models).

## <a name="authenticate-the-client"></a>Аутентификация клиента



Выполните проверку подлинности клиентского объекта с использованием переменных подписки, которые вы определили. Здесь применяется объект `AzureKeyCredential`, чтобы при необходимости ключ API можно было обновить, не создавая новых клиентских объектов. Также будет создан объект клиента бучения.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_auth)]


## <a name="get-assets-for-testing"></a>Получение ресурсов для тестирования

Также потребуется добавить URL-адреса данных для обучения и тестирования.
* [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]
   
   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Получение подписанного URL-адреса":::
* Используйте пример формы и изображения квитанций, включенные в приведенные ниже примеры. Вы можете также скачать их с [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/test-assets) или выполнить описанные выше действия, чтобы получить URL-адрес SAS для отдельного документа в хранилище BLOB-объектов. 


## <a name="analyze-layout"></a>Анализ макета

С помощью Распознавателя документов можно анализировать таблицы, строки и слова в документах без необходимости обучать модель. Дополнительные сведения об извлечении макетов см. в статье [Служба макета Распознавателя документов](../../concept-layout.md). Для анализа содержимого файла по указанному URI используйте метод `beginRecognizeContentFromUrl`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_getcontent)]


> [!TIP]
> Вы можете также получить содержимое из локального файла. Изучите информацию о методах класса [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), например о **beginRecognizeContent**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) для сценариев, включающих использование локальных изображений.

### <a name="output"></a>Выходные данные

```console
Page 1: width 8.5 and height 11 with unit inch
cell [0,0] has text Invoice Number
cell [0,1] has text Invoice Date
cell [0,2] has text Invoice Due Date
cell [0,3] has text Charges
cell [0,5] has text VAT ID
cell [1,0] has text 34278587
cell [1,1] has text 6/18/2017
cell [1,2] has text 6/24/2017
cell [1,3] has text $56,651.49
cell [1,5] has text PT
```



## <a name="train-a-custom-model"></a>Обучение пользовательской модели

В этом разделе демонстрируется, как обучить модель на основе собственных данных. Обученная модель выводит структурированные данные, которые содержат связи типа "ключ/значение" из исходного документа формы. После обучения модели вы можете проверить ее, повторно обучить и, в конечном итоге, применить с целью надежного извлечения данных из нескольких форм в соответствии со своими потребностями.

> [!NOTE]
> Также можно обучить модель с графическим пользовательским интерфейсом, такой как [пример средства разметки Распознавателя документов](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Обучение моделей без меток

Вы можете обучить свою модель анализировать все поля и значения в ваших формах без необходимости вручную помечать документы, которые вы будете использовать для обучения.

Приведенная ниже функция обучает модель на основе заданного набора документов и выводит ее статус в консоль. 

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_train)]


### <a name="output"></a>Выходные данные

Это выходные данные для модели, обученной по данным для обучения, доступным в [пакете SDK для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer). Данный пример выходных данных усечен для удобства чтения.

```console
training status: creating
training status: ready
Model ID: 9d893595-1690-4cf2-a4b1-fbac0fb11909
Status: ready
Training started on: Thu Aug 20 2020 20:27:26 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:27:37 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'field-0'
The model found field 'field-1'
The model found field 'field-2'
The model found field 'field-3'
The model found field 'field-4'
The model found field 'field-5'
The model found field 'field-6'
The model found field 'field-7'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: 
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: 
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: 
...
```

### <a name="train-a-model-with-labels"></a>Обучение моделей с метками

Модели также можно обучать на документах, которые размечены вручную. В некоторых сценариях обучение по такой схеме приводит к лучшим результатам. Для обучения с метками в контейнере хранилища BLOB-объектов вместе с документами, используемыми для обучения, должны находиться особые файлы с информацией о метках (`\<filename\>.pdf.labels.json`). Эти файлы можно создать в пользовательском интерфейсе [средства разметки Распознавателя документов](../../quickstarts/label-tool.md). Создав файлы, вызовите метод `beginTraining` и задайте для параметра `uselabels` значение `true`.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_trainlabels)]


### <a name="output"></a>Выходные данные 

Это выходные данные для модели, обученной по данным для обучения, доступным в [пакете SDK для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples). Данный пример выходных данных усечен для удобства чтения.

```console
training status: creating
training status: ready
Model ID: 789b1b37-4cc3-4e36-8665-9dde68618072
Status: ready
Training started on: Thu Aug 20 2020 20:30:37 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:30:43 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'CompanyAddress'
The model found field 'CompanyName'
The model found field 'CompanyPhoneNumber'
The model found field 'DatedAs'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
...
```

## <a name="analyze-forms-with-a-custom-model"></a>анализ документов с помощью пользовательской модели;

В этом разделе показано, как извлекать данные вида "ключ/значение" и другое содержимое из пользовательских форм с помощью обученных на них моделей.

> [!IMPORTANT]
> Для реализации этого сценария у вас уже должна быть обученная модель, идентификатор которой нужно будет передать указанному ниже методу. См. раздел [Обучение модели](#train-a-model-without-labels).

Используйте метод `beginRecognizeCustomFormsFromUrl`. Возвращаемое значение представляет собой коллекцию объектов `RecognizedForm`: по одному для каждой страницы обработанного документа.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_analyze)]

> [!TIP]
> Вы можете также проанализировать локальные файлы. Изучите информацию о методах класса [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), например о **beginRecognizeCustomForms**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) для сценариев, включающих использование локальных изображений.


### <a name="output"></a>Выходные данные

```console
status: notStarted
status: succeeded
Forms:
custom:form, page range: [object Object]
Pages:
Page number: 1
Tables
cell (0,0) Invoice Number
cell (0,1) Invoice Date
cell (0,2) Invoice Due Date
cell (0,3) Charges
cell (0,5) VAT ID
cell (1,0) 34278587
cell (1,1) 6/18/2017
cell (1,2) 6/24/2017
cell (1,3) $56,651.49
cell (1,5) PT
Fields:
Field Merchant has value 'Invoice For:' with a confidence score of 0.116
Field CompanyPhoneNumber has value '$56,651.49' with a confidence score of 0.249
Field VendorName has value 'Charges' with a confidence score of 0.145
Field CompanyAddress has value '1 Redmond way Suite 6000 Redmond, WA' with a confidence score of 0.258
Field CompanyName has value 'PT' with a confidence score of 0.245
Field Website has value '99243' with a confidence score of 0.114
Field DatedAs has value 'undefined' with a confidence score of undefined
Field Email has value 'undefined' with a confidence score of undefined
Field PhoneNumber has value 'undefined' with a confidence score of undefined
Field PurchaseOrderNumber has value 'undefined' with a confidence score of undefined
Field Quantity has value 'undefined' with a confidence score of undefined
Field Signature has value 'undefined' with a confidence score of undefined
Field Subtotal has value 'undefined' with a confidence score of undefined
Field Tax has value 'undefined' with a confidence score of undefined
Field Total has value 'undefined' with a confidence score of undefined
```

## <a name="analyze-receipts"></a>Анализ квитанций

В этом разделе объясняется, как с помощью предварительно обученной модели можно анализировать используемые в США квитанции и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе квитанций см. в статье [Готовая модель получения Распознавателя форм](../../concept-receipts.md).

Для анализа чеков по URI используйте метод `beginRecognizeReceiptsFromUrl`. В следующем блоке кода обрабатывается квитанция с указанным универсальным кодом ресурса (URI) и в консоль выводятся все ее основные поля с соответствующими значениями.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_receipts)]

> [!TIP]
> Можно также проанализировать локальные изображения квитанций. Изучите информацию о методах класса [FormRecognizerClient](/javascript/api/@azure/ai-form-recognizer/formrecognizerclient), например о **beginRecognizeReceipts**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples) для сценариев, включающих использование локальных изображений.

### <a name="output"></a>Выходные данные

```console
status: notStarted
status: running
status: succeeded
First receipt:
  Receipt Type: 'Itemized', with confidence of 0.659
  Merchant Name: 'Contoso Contoso', with confidence of 0.516
  Transaction Date: 'Sun Jun 09 2019 17:00:00 GMT-0700 (Pacific Daylight Time)', with confidence of 0.985
    Item Name: '8GB RAM (Black)', with confidence of 0.916
    Item Name: 'SurfacePen', with confidence of 0.858
  Total: '1203.39', with confidence of 0.774
```

## <a name="manage-your-custom-models"></a>Управление пользовательскими моделями

В этом разделе показано, как управлять пользовательскими моделями, которые хранятся в вашей учетной записи. Приведенный ниже код выполняет все задачи по управлению моделями в одной функции (в качестве примера).

### <a name="get-number-of-models"></a>Получение количества моделей

Следующий блок кода получает количество моделей, в настоящее время находящихся в вашей учетной записи.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_count)]

### <a name="get-list-of-models-in-account"></a>Получение списка моделей в учетной записи

Следующий блок кода предоставляет полный список доступных в учетной записи моделей, в том числе сведения о дате и времени их создания и их текущем состоянии.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_list)]

### <a name="output"></a>Выходные данные

```console
model 0:
{
  modelId: '453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e',
  status: 'invalid',
  trainingStartedOn: 2020-08-20T22:28:52.000Z,
  trainingCompletedOn: 2020-08-20T22:28:53.000Z
}
model 1:
{
  modelId: '628739de-779c-473d-8214-d35c72d3d4f7',
  status: 'ready',
  trainingStartedOn: 2020-08-20T23:16:51.000Z,
  trainingCompletedOn: 2020-08-20T23:16:59.000Z
}
model 2:
{
  modelId: '789b1b37-4cc3-4e36-8665-9dde68618072',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:30:37.000Z,
  trainingCompletedOn: 2020-08-21T03:30:43.000Z
}
model 3:
{
  modelId: '9d893595-1690-4cf2-a4b1-fbac0fb11909',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:27:26.000Z,
  trainingCompletedOn: 2020-08-21T03:27:37.000Z
}
```

### <a name="get-list-of-model-ids-by-page"></a>Получение списка идентификаторов моделей с разбиением на страницы

Этот блок кода предоставляет список моделей и их идентификаторов с разбиением на страницы.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_listpages)]

### <a name="output"></a>Выходные данные

```console
model 1: 453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
model 2: 628739de-779c-473d-8214-d35c72d3d4f7
model 3: 789b1b37-4cc3-4e36-8665-9dde68618072
```

### <a name="get-model-by-id"></a>Получение модели по идентификатору

Следующая функция принимает идентификатор модели и получает соответствующий объект модели. Эта функция не вызывается по умолчанию.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_getmodel)]

### <a name="delete-a-model-from-the-resource-account"></a>Удаление модели из учетной записи ресурсов

Модель можно удалить из учетной записи ресурсов по ее идентификатору. Эта функция удаляет модель с заданным идентификатором. Эта функция не вызывается по умолчанию.

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/FormRecognizer/FormRecognizerQuickstart.js?name=snippet_manage_delete)]

### <a name="output"></a>Выходные данные

```console
Model with id 789b1b37-4cc3-4e36-8665-9dde68618072 has been deleted
```

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `node` для файла quickstart.

```console
node index.js
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="enable-logs"></a>Включение журналов

При использовании этой библиотеки с помощью приведенной ниже переменной среды можно настроить ведение журналов отладки.

```console
export DEBUG=azure*
```

Дополнительные сведения о том, как включить журналы, см. в [документации к пакету @azure/logger](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/core/logger).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано использование клиентской библиотеки Распознавателя документов на языке JavaScript для обучения модели и анализа документов различными способами. Теперь следует изучить советы по созданию лучшего набора данных для обучения и созданию более точных моделей.

> [!div class="nextstepaction"]
> [Создание набора данных для обучения](../../build-training-data-set.md)

## <a name="see-also"></a>См. также

* [Что такое Распознаватель документов?](../../overview.md)
* Пример кода из этого руководства можно найти на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/FormRecognizer/FormRecognizerQuickstart.js).

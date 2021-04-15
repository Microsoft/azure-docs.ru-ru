---
title: Краткое руководство. Клиентская библиотека Распознавателя документов для Java
description: Узнайте, как использовать клиентскую библиотеку Распознавателя документов для Java для создания приложения для обработки форм, которое извлекает из пользовательских документов пары "ключ — значение" и табличные данные.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 03/19/2021
ms.custom: devx-track-java
ms.author: lajanuar
ms.openlocfilehash: a709f82b04ed5c1fe70f6927b33605cfff20ed6b
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104761104"
---
<!-- markdownlint-disable MD001 -->
<!-- markdownlint-disable MD024 -->
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD034 -->
> [!IMPORTANT]
> В коде, приведенном в этой статье, для простоты используются синхронные методы и незащищенное хранилище учетных данных.

[Справочная документация](/java/api/overview/azure/ai-formrecognizer-readme) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src) | [Пакет (Maven)](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer) | [Примеры](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Текущая версия [пакета средств разработки Java (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* [Средство сборки Gradle](https://gradle.org/install/) или другой диспетчер зависимостей.
* Получив подписку Azure, перейдите к <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="Создание ресурса Распознавателя документов"  target="_blank">созданию ресурса Распознавателя документов </a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
  * Для подключения приложения к API Распознавателя документов потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
  * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.
* Набор данных для обучения в хранилище BLOB-объектов Azure. Советы и варианты для объединения данных для обучения см. в статье о [создании обучающего набора данных для пользовательской модели](../../build-training-data-set.md). При работе с этим кратким руководством можно использовать файлы в папке **Train** из [примера набора данных](https://go.microsoft.com/fwlink/?linkid=2090451) (скачайте и разархивируйте файл *sample_data.zip*).

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-gradle-project"></a>Создание проекта Gradle

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него. 

```console
mkdir myapp && cd myapp
```

Выполните команду `gradle init` из рабочей папки. Эта команда создает необходимые файлы сборки для Gradle, включая *build.gradle.kts*, который используется во время выполнения для создания и настройки приложения.

```console
gradle init --type basic
```

Когда появится запрос на выбор **предметно-ориентированного языка**, выберите **Kotlin**.

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

В этом кратком руководстве используется диспетчер зависимостей Gradle. Клиентскую библиотеку и информацию для других диспетчеров зависимостей можно найти в [центральном репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer).

В файле проекта *build.gradle.kts* включите клиентскую библиотеку в качестве оператора `implementation` наряду с требуемыми подключаемыми модулями и параметрами.

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.azure", name = "azure-ai-formrecognizer", version = "3.1.0-beta.1")
}
```

> [!NOTE]
> В пакете SDK 3.1.0 Распознавателя документов используется _API версии 2.1, предварительная версия 2_. Используйте [**REST API**](../../quickstarts/client-library.md) для _API версии 2.1, предварительная версия 3_.

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.azure", name = "azure-ai-formrecognizer", version = "3.0.0")
}
```

> [!NOTE]
> В пакете SDK 3.0.0 Распознавателя документов используется API версии 2.0

---

### <a name="create-a-java-file"></a>Создание файла Java


Выполните следующие команды из рабочего каталога:

```console
mkdir -p src/main/java
```

Перейдите к новой папке и создайте файл с именем *FormRecognizer.java*. Откройте его в предпочитаемом редакторе или интегрированной среде разработки и добавьте следующие операторы `import`:

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/FormRecognizer/FormRecognizer.java), где размещены примеры кода для этого краткого руководства.


В классе **FormRecognizer** приложения создайте переменные для ключа и конечной точки ресурса.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_creds)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Распознавателя документов, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Дополнительные сведения см. в статье о [безопасности в Cognitive Services](../../../cognitive-services-security.md).

В методе **main** приложения добавьте вызовы методов, используемых в этом кратком руководстве. Они будут определены позже. Также потребуется добавить URL-адреса данных для обучения и тестирования.

* [!INCLUDE [get SAS URL](../../includes/sas-instructions.md)]
  
   :::image type="content" source="../../media/quickstarts/get-sas-url.png" alt-text="Получение подписанного URL-адреса":::
* Получить URL-адрес формы для тестирования (отдельного документа в хранилище больших двоичных объектов) можно с помощью описанных выше действий. Также можно взять URL-адрес документа, расположенного в другом месте.
* С помощью описанного выше метода также получите URL-адрес изображения квитанции.
<!-- markdownlint-disable MD024 -->
#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_mainvars)]

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_maincalls)]

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_mainvars)]

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_maincalls)]

---

## <a name="object-model"></a>Объектная модель

С помощью Распознавателя документов можно создавать клиенты двух разных типов. Первый, `FormRecognizerClient`, используется для запроса из службы распознанных полей и содержимого форм. Второй, `FormTrainingClient`, используется для создания настраиваемых моделей и управления ими. Эти модели можно использовать для улучшения распознавания.

### <a name="formrecognizerclient"></a>FormRecognizerClient

`FormRecognizerClient` предоставляет операции для перечисленных ниже целей.

* Распознавание полей форм и содержимого с помощью настраиваемых моделей, обученных для анализа пользовательских форм.  Эти значения возвращаются в коллекцию объектов `RecognizedForm`. См. пример в разделе об [анализе пользовательских форм](#analyze-forms-with-a-custom-model).
* Распознавание содержимого формы, в том числе таблиц, строк и слов, без необходимости обучения модели.  Содержимое форм возвращается в коллекцию объектов `FormPage`. См. пример [анализа макета](#analyze-layout).
* Распознавание общих полей в квитанциях для США с помощью предварительно обученной модели для обработки квитанций в службе "Распознаватель документов".  Эти поля и метаданные возвращаются в коллекцию объектов `RecognizedForm`. См. пример [анализа квитанций](#analyze-receipts).

### <a name="formtrainingclient"></a>FormTrainingClient

`FormTrainingClient` предоставляет операции для перечисленных ниже целей.

* Обучение настраиваемых моделей для анализа всех полей и значений в пользовательских формах.  Возвращаемый объект `CustomFormModel` задает типы форм, которые будет анализировать модель, и поля, которые она будет извлекать для каждого из этих типов.
* Обучение пользовательских моделей для анализа конкретных полей и значений, указываемых путем добавления меток к пользовательским формам.  Возвращаемый объект `CustomFormModel` задает поля, которые будут извлечены моделью, а также предполагаемую точность для каждого поля.
* Управление моделями, созданными в учетной записи.
* Копирование настраиваемой модели из одного ресурса Распознавателя документов в другой.

> [!NOTE]
> Модели можно также обучить с помощью графического пользовательского интерфейса, например [средства маркировки Распознавателя документов](../../quickstarts/label-tool.md).

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Распознавателя документов для Java:
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

В начале метода **main** добавьте следующий код. В этом фрагменте выполняется проверка подлинности двух клиентских объектов с использованием переменных подписки, указанных выше. Здесь применяется объект **AzureKeyCredential**, чтобы при необходимости ключ API можно было обновить, не создавая новых клиентских объектов.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_auth)]

## <a name="analyze-layout"></a>Анализ макета

С помощью Распознавателя документов можно анализировать таблицы, строки и слова в документах без необходимости обучать модель. Дополнительные сведения об извлечении макетов см. в статье [Служба макета Распознавателя документов](../../concept-layout.md).

Для анализа содержимого файла по указанному URL-адресу используйте метод **beginRecognizeContentFromUrl**.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_getcontent_call)]

> [!TIP]
> Вы можете также получить содержимое из локального файла. Изучите информацию о методах класса [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), например о **beginRecognizeContent**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов **FormPage**: по одному для каждой страницы обработанного документа. Приведенный ниже код обрабатывает эти объекты в цикле и выводит извлеченные из документа пары ключей и значений, а также табличные данные.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_getcontent_print)]
### <a name="output"></a>Выходные данные

```console
Get form content...
----Recognizing content ----
Has width: 8.500000 and height: 11.000000, measured with unit: inch.
Table has 2 rows and 6 columns.
Cell has text Invoice Number.
Cell has text Invoice Date.
Cell has text Invoice Due Date.
Cell has text Charges.
Cell has text VAT ID.
Cell has text 458176.
Cell has text 3/28/2018.
Cell has text 4/16/2018.
Cell has text $89,024.34.
Cell has text ET.
```

## <a name="analyze-invoices"></a>Анализ счетов

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

В этом разделе показано, как с помощью предварительно обученной модели анализировать счета на продажу и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе счета см. в статье [Служба макета Распознавателя документов](../../concept-invoices.md).

Для анализа счетов по URL-адресу используйте метод `beginRecognizeInvoicesFromUrl`. 

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_invoice_call)]

> [!TIP]
> Можно также проанализировать счета, сохраненные локально. Изучите информацию о методах класса [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), например о **beginRecognizeInvoices**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов **RecognizedForm**: по одному для каждого счета в документе. В приведенном ниже блоке кода обрабатывается счет с указанным универсальным кодом ресурса (URI) и все ее основные поля с соответствующими значениями выводятся в консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_invoice_print)]

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

Приведенный ниже метод обучает модель на основе заданного набора документов и выводит ее статус в консоль. 

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_train_call)]

Возвращаемый объект **CustomFormModel** содержит сведения о типах форм, которые может анализировать модель, и полях, которые она способна извлечь из формы каждого типа. Приведенный ниже блок кода выводит эту информацию в консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_train_print)]

Наконец, этот метод возвращает уникальный идентификатор модели.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_train_return)]


### <a name="output"></a>Выходные данные

```console
Train Model with training data...
Model Id: 20c3544d-97b4-49d9-b39b-dc32d85f1358
Model Status: ready
Training started on: 2020-08-31T16:52:09Z
Training completed on: 2020-08-31T16:52:23Z

Recognized Fields:
The subModel has form type form-0
The model found field 'field-0' with label: Address:
The model found field 'field-1' with label: Charges
The model found field 'field-2' with label: Invoice Date
The model found field 'field-3' with label: Invoice Due Date
The model found field 'field-4' with label: Invoice For:
The model found field 'field-5' with label: Invoice Number
The model found field 'field-6' with label: VAT ID
```

### <a name="train-a-model-with-labels"></a>Обучение моделей с метками

Модели также можно обучать на документах, которые размечены вручную. В некоторых сценариях обучение по такой схеме приводит к лучшим результатам. Для обучения с метками в контейнере больших двоичных объектов вместе с документами, используемыми для обучения, должны находиться особые файлы с информацией о метках ( *\<filename\>.pdf.labels.json*). Эти файлы можно создать в пользовательском интерфейсе [средства разметки Распознавателя документов](../../quickstarts/label-tool.md). Создав файлы, вызовите метод **beginTraining** и задайте для параметра *useTrainingLabels* значение `true`.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_trainlabels_call)]


Возвращаемый объект **CustomFormModel** содержит поля, которые модель способна извлечь из документа, с указанием ориентировочной точности для каждого из них. Приведенный ниже блок кода выводит эту информацию в консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_trainlabels_print)]


### <a name="output"></a>Выходные данные

```console
Train Model with training data...
Model Id: 20c3544d-97b4-49d9-b39b-dc32d85f1358
Model Status: ready
Training started on: 2020-08-31T16:52:09Z
Training completed on: 2020-08-31T16:52:23Z

Recognized Fields:
The subModel has form type form-0
The model found field 'field-0' with label: Address:
The model found field 'field-1' with label: Charges
The model found field 'field-2' with label: Invoice Date
The model found field 'field-3' with label: Invoice Due Date
The model found field 'field-4' with label: Invoice For:
The model found field 'field-5' with label: Invoice Number
The model found field 'field-6' with label: VAT ID
```

## <a name="analyze-forms-with-a-custom-model"></a>анализ документов с помощью пользовательской модели;

В этом разделе показано, как извлекать данные вида "ключ/значение" и другое содержимое из пользовательских форм с помощью обученных на них моделей.

> [!IMPORTANT]
> Для реализации этого сценария у вас уже должна быть обученная модель, идентификатор которой нужно будет передать указанному ниже методу. См. раздел [Обучение модели](#train-a-model-without-labels).

Используйте метод **beginRecognizeCustomFormsFromUrl**. 

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_analyze_call)]

> [!TIP]
> Вы можете также проанализировать локальные файлы. Изучите информацию о методах класса [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), например о **beginRecognizeCustomForms**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов **RecognizedForm**: по одному для каждой страницы обработанного документа. Следующий код выводит в консоль результаты анализа. Он отображает каждое из распознанных полей с соответствующим значением и коэффициентом достоверности.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_analyze_print)]


### <a name="output"></a>Выходные данные

```console
Analyze PDF form...
----------- Recognized custom form info for page 0 -----------
Form type: form-0
Field 'field-0' has label 'Address:' with a confidence score of 0.91.
Field 'field-1' has label 'Invoice For:' with a confidence score of 1.00.
Field 'field-2' has label 'Invoice Number' with a confidence score of 1.00.
Field 'field-3' has label 'Invoice Date' with a confidence score of 1.00.
Field 'field-4' has label 'Invoice Due Date' with a confidence score of 1.00.
Field 'field-5' has label 'Charges' with a confidence score of 1.00.
Field 'field-6' has label 'VAT ID' with a confidence score of 1.00.
```

## <a name="analyze-receipts"></a>Анализ квитанций

В этом разделе объясняется, как с помощью предварительно обученной модели можно анализировать используемые в США квитанции и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе квитанций см. в статье [Готовая модель получения Распознавателя форм](../../concept-receipts.md).

Для распознавания чеков по указанному URI используйте метод **beginRecognizeReceiptsFromUrl**. 

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_receipts_call)]

> [!TIP]
> Можно также проанализировать локальные изображения квитанций. Изучите информацию о методах класса [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), например о **beginRecognizeReceipts**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов **RecognizedReceipt**: по одному для каждой страницы обработанного документа. Следующий блок кода обрабатывает квитанции в цикле и выводит их данные в консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_receipts_print)]

Следующий блок кода обрабатывает в цикле отдельные объекты, обнаруженные в квитанции, и выводит их данные в консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_receipts_print_items)]

### <a name="output"></a>Выходные данные 

```console
Analyze receipt...
----------- Recognized Receipt page 0 -----------
Merchant Name: Contoso Contoso, confidence: 0.62
Merchant Address: 123 Main Street Redmond, WA 98052, confidence: 0.99
Transaction Date: 2020-06-10, confidence: 0.90
Receipt Items:
Name: Cappuccino, confidence: 0.96s
Quantity: null, confidence: 0.957s]
Total Price: 2.200000, confidence: 0.95
Name: BACON & EGGS, confidence: 0.94s
Quantity: null, confidence: 0.927s]
Total Price: null, confidence: 0.93
```

## <a name="analyze-business-cards"></a>Анализ визитных карточек

#### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/preview)

В этом разделе показано, как с помощью предварительно обученной модели анализировать визитные карточки на английском языке и извлекать из них содержимое стандартных полей. Дополнительные сведения об анализе визитных карточек см. в статье [Готовая модель для анализа визитных карточек с помощью Распознавателя форм](../../concept-business-cards.md).

Для анализа визитных карточек по URL-адресу используйте метод `beginRecognizeBusinessCardsFromUrl`. 

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_bc_call)]

> [!TIP]
> Можно также проанализировать локальные изображения визитных карточек. Изучите информацию о методах класса [FormRecognizerClient](/java/api/com.azure.ai.formrecognizer.formrecognizerclient), например о **beginRecognizeBusinessCards**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples/README.md) для сценариев, включающих использование локальных изображений.

Возвращаемое значение представляет собой коллекцию объектов **RecognizedForm**: по одному для каждой карточки в документе. В следующем блоке кода обрабатывается визитная карточка с указанным универсальным кодом ресурса (URI) и все ее основные поля с соответствующими значениями выводятся в консоль.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer-preview.java?name=snippet_bc_print)]

#### <a name="v20"></a>[Версия 2.0](#tab/ga)

> [!IMPORTANT]
> Эта функция недоступна в выбранной версии API.

---

## <a name="manage-custom-models"></a>Управление пользовательскими моделями

В этом разделе показано, как управлять пользовательскими моделями, которые хранятся в вашей учетной записи. Приведенный ниже код выполняет все задачи по управлению моделями в одном методе (в качестве примера). Сначала скопируйте сигнатуру метода ниже:

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage)]


### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Проверка числа моделей в учетной записи ресурсов FormRecognizer

Приведенный ниже блок кода проверяет, сколько моделей сохранено в вашей учетной записи Распознавателя документов, и сравнивает это значение с ограничением для данной учетной записи.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage_count)]


#### <a name="output"></a>Выходные данные 

```console
The account has 12 custom models, and we can have at most 250 custom models
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Вывод списка моделей, которые хранятся в учетной записи ресурсов

Приведенный ниже блок кода выводит в консоль список текущих моделей в вашей учетной записи с подробными сведениями о них.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage_list)]


#### <a name="output"></a>Выходные данные 

Этот ответ усечен для удобства чтения.

```console
We have following models in the account:
Model Id: 0b048b60-86cc-47ec-9782-ad0ffaf7a5ce
Model Id: 0b048b60-86cc-47ec-9782-ad0ffaf7a5ce
Model Status: ready
Training started on: 2020-06-04T18:33:08Z
Training completed on: 2020-06-04T18:33:10Z
Custom Model Form type: form-0b048b60-86cc-47ec-9782-ad0ffaf7a5ce
Custom Model Accuracy: 1.00
Field Text: invoice date
Field Accuracy: 1.00
Field Text: invoice number
Field Accuracy: 1.00
...
```

### <a name="delete-a-model-from-the-resource-account"></a>Удаление модели из учетной записи ресурсов

Модель можно удалить из учетной записи ресурсов по ее идентификатору.

[!code-java[](~/cognitive-services-quickstart-code/java/FormRecognizer/FormRecognizer.java?name=snippet_manage_delete)]

## <a name="run-the-application"></a>Выполнение приложения

Вернитесь к главному каталогу проекта. Затем создайте приложение с помощью следующей команды:

```console
gradle build
```

Запустите приложение, выполнив цель `run`:

```console
gradle run
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Устранение неполадок

Клиенты Распознавателя документов вызывают исключения `ErrorResponseException`. Например, если вы попытаетесь указать URL-адрес недействительного исходного файла, будет вызвано исключение `ErrorResponseException` с описанием причины ошибки. В следующем фрагменте кода ошибка корректно обрабатывается путем перехвата исключения с выводом дополнительной информации.

```java Snippet:FormRecognizerBadRequest
try {
    formRecognizerClient.beginRecognizeContentFromUrl("invalidSourceUrl");
} catch (ErrorResponseException e) {
    System.out.println(e.getMessage());
}
```

### <a name="enable-client-logging"></a>Включение ведения журнала клиента

Пакеты SDK Azure для Java поддерживают возможность ведения подробных журналов, которые помогают находить ошибки в приложениях и ускоряют их устранение. Такие журналы содержат сведения о работе приложения до момента сбоя, что позволяет определить его первопричину. Инструкции, которые помогут включить ведение журналов, см. на [вики-странице, посвященной журналам](https://github.com/Azure/azure-sdk-for-java/wiki/Logging-with-Azure-SDK).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве описано использование клиентской библиотеки Распознавателя документов на языке Java для обучения модели и анализа документов различными способами. Теперь следует изучить советы по созданию лучшего набора данных для обучения и созданию более точных моделей.

> [!div class="nextstepaction"]
> [Создание набора данных для обучения](../../build-training-data-set.md)

* [Что такое Распознаватель документов?](../../overview.md)
* Пример кода из этого руководства, а также другие ресурсы можно найти на [GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/src/samples).

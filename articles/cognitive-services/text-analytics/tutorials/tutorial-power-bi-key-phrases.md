---
title: Руководство по Интеграция Power BI с Анализом текста в Cognitive Services
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать API службы "Анализ текста" для извлечения ключевых фраз из текста, хранимого в Power BI.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: tutorial
ms.date: 02/09/2021
ms.author: aahi
ms.openlocfilehash: 47feddb88fd7ddae1f8be54709019b4c339d177d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599176"
---
# <a name="tutorial-integrate-power-bi-with-the-text-analytics-cognitive-service"></a>Руководство по Интеграция Power BI с Анализом текста в Cognitive Services

Microsoft Power BI Desktop — бесплатное приложение, позволяющее подключиться к данным, а затем преобразовать и визуализировать их. Служба "Анализ текста", входящая в набор служб Microsoft Azure Cognitive Services, обеспечивает обработку естественного языка. Она может извлечь из необработанного и неструктурированного текста самые важные фразы, выполнить анализ тональности и определить в нем известные сущности, например торговые марки. Оба инструмента помогают быстро узнать, о чем говорят ваши пользователи и что они при этом испытывают.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Использование Power BI Desktop для импорта и преобразования данных.
> * Создание пользовательской функции в Power BI Desktop.
> * Интеграция Power BI Desktop с API ключевых фраз службы "Анализ текста".
> * Использование API ключевых фраз службы "Анализ текста" для извлечения наиболее важных фраз из отзывов клиентов.
> * Создание облака слов на основе отзывов клиентов.

## <a name="prerequisites"></a>Предварительные требования
<a name="Prerequisites"></a>

- Microsoft Power BI Desktop ([скачать бесплатно](https://powerbi.microsoft.com/get-started/)).
- Учетная запись Microsoft Azure. [Создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/) или [войдите](https://portal.azure.com/) в существующую.
- Учетная запись API Cognitive Services с API анализа текста. Если у вас ее нет, вы можете [зарегистрироваться](../../cognitive-services-apis-create-account.md) и выбрать ценовую категорию "Бесплатный", обеспечивающую 5000 транзакций в месяц (ознакомьтесь со [сведениями о ценах](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/) для работы с этим руководством).
- [Ключ доступа к Анализу текста](../../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource), созданный автоматически во время регистрации.
- Отзывы пользователей. Можно использовать [наш пример данных](https://aka.ms/cogsvc/ta) или собственные данные. В этом руководстве предполагается, что вы используете наш пример данных.

## <a name="load-customer-data"></a>Загрузка данных пользователей
<a name="LoadingData"></a>

Чтобы начать работу, откройте Power BI Desktop и загрузите файл данных с разделителями-запятыми `FabrikamComments.csv`, скачанный при выполнении [предварительных требований](#Prerequisites). В этом файле представлены возможные действия пользователей за сутки на форуме поддержки небольшой вымышленной компании.

> [!NOTE]
> Power BI использует данные из разнообразных источников, например из Facebook или базы данных SQL. См. дополнительные сведения об [интеграции Facebook с Power BI](https://powerbi.microsoft.com/integrations/facebook/) и [интеграции SQL Server с Power BI](https://powerbi.microsoft.com/integrations/sql-server/).

В главном окне Power BI Desktop выберите ленту **Главная**. В группе **Внешние данные** на ленте откройте раскрывающееся меню **Получить данные** и выберите **Текстовый или CSV-файл**.

![[Кнопка "Получить данные"]](../media/tutorials/power-bi/get-data-button.png)

Появится диалоговое окно "Открытие". Перейдите к папке "Загрузки" или к папке, в которую вы скачали файл `FabrikamComments.csv`. Щелкните `FabrikamComments.csv` и нажмите кнопку **Открыть**. Появится диалоговое окно "Импорт CSV-файла".

![[Диалоговое окно "Импорт CSV-файла"]](../media/tutorials/power-bi/csv-import.png)

В диалоговом окне "Импорт CSV-файла" можно проверить кодировку, разделитель, строки заголовка и типы столбцов, определенные в Power BI Desktop. Если все указано правильно, щелкните **Загрузить**.

Чтобы просмотреть загруженные данные, нажмите кнопку **Просмотр данных** в левой части рабочей области Power BI. Откроется таблица с данными, как в Microsoft Excel.

![[Исходное представление импортированных данных]](../media/tutorials/power-bi/initial-data-view.png)

## <a name="prepare-the-data"></a>Подготовка данных
<a name="PreparingData"></a>

Может потребоваться преобразовать данные в Power BI Desktop перед их обработкой API ключевых фраз службы "Анализ текста".

Пример данных содержит столбцы `subject` и `comment`. С помощью функции слияния столбцов в Power BI Desktop вы можете извлечь ключевые фразы из данных в обоих столбцах, а не только в столбце `comment`.

В Power BI Desktop выберите ленту **Главная**. В группе **Внешние данные** щелкните **Изменить запросы**.

![[Группа "Внешние данные" на ленте "Главная"]](../media/tutorials/power-bi/edit-queries.png)

В списке **Запросы** слева выберите `FabrikamComments`, если это еще не сделано.

Затем выделите столбцы `subject` и `comment` в таблице. Может потребоваться горизонтально прокрутить документ, чтобы увидеть эти столбцы. Сначала щелкните заголовок столбца `subject`, а затем, удерживая клавишу CTRL, щелкните заголовок столбца `comment`.

![[Выбор полей для объединения]](../media/tutorials/power-bi/select-columns.png)

Выберите ленту **Преобразование**. В группе **Текстовые столбцы** на ленте щелкните **Объединить столбцы**. Появится диалоговое окно "Объединение столбцов".

![[Объединение полей в диалоговом окне "Объединение столбцов"]](../media/tutorials/power-bi/merge-columns.png)

В диалоговом окне "Объединение столбцов" выберите `Tab` в качестве разделителя и щелкните **ОК**.

Кроме того, можно отфильтровать пустые сообщения с помощью фильтра "Удалить пустые", а также удалить непечатаемые знаки с помощью очистки. Если в ваших данных содержится столбец `spamscore`, как в примере, с помощью фильтра "Число" можно игнорировать нежелательные комментарии.

## <a name="understand-the-api"></a>Общие сведения об API
<a name="UnderstandingAPI"></a>

[API ключевых фраз](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-V3-0/operations/KeyPhrases) службы "Анализ текста" может обрабатывать до тысячи текстовых документов за один HTTP-запрос. Power BI обрабатывает одну запись за раз, поэтому в данном руководстве вызовы API будут содержать по одному документу. В каждом обрабатываемом документе для API ключевых фраз требуются приведенные ниже поля.

| Поле | Описание |
| - | - |
| `id`  | Уникальный идентификатор документа в пределах запроса. Это поле также содержится в ответе, так что, если вы обрабатываете несколько документов, извлеченные ключевые фразы можно легко сопоставить с исходным документом. В этом руководстве, так как обрабатывается только один документ на запрос, можно жестко закодировать одно значение `id` для каждого запроса.|
| `text`  | Текст для обработки. Значение этого поля извлекается из столбца `Merged`, созданного в [предыдущем разделе](#PreparingData). Он содержит строку темы и текст комментария. API ключевых фраз поддерживает данные длиной максимум 5120 символов.|
| `language` | Код естественного языка документа. Все сообщения в примере данных написаны на английском языке, поэтому можно жестко закодировать значение `en` для этого поля.|

## <a name="create-a-custom-function"></a>Создание пользовательской функции
<a name="CreateCustomFunction"></a>

Теперь вы готовы к созданию настраиваемой функции, позволяющей интегрировать Power BI и Анализ текст. Этот текст служит параметром функции. Она преобразовывает данные в требуемый формат JSON (и из него) и выполняет HTTP-запрос к API ключевых фраз. Затем эта функция анализирует ответ API и возвращает строку со списком извлеченных ключевых фраз с разделителями-запятыми.

> [!NOTE]
> Настраиваемые функции Power BI Desktop написаны на [языке формул Power Query M](/powerquery-m/power-query-m-reference) (М для краткости). М — язык функционального программирования на основе [F#](/dotnet/fsharp/). Чтобы завершить работу с этим руководством, не нужно быть программистом, — весь необходимый код приведен ниже.

Убедитесь, что вы работаете с окном редактора запросов в Power BI Desktop. Если это не так, выберите ленту **Главная**, а затем в группе **Внешние данные** щелкните **Изменить запросы**.

Теперь на ленте **Главная** в группе **Новый запрос** откройте раскрывающееся меню **Новый источник** и выберите **Пустой запрос**. 

В списке "Запросы" появится новый запрос с именем `Query1` по умолчанию. Дважды щелкните эту запись и переименуйте ее в `KeyPhrases`.

Теперь на ленте **Главная** в группе **Запрос** щелкните **Расширенный редактор**, чтобы открыть окно "Расширенный редактор". Удалите находящийся в окне код и вставьте следующий. 

> [!NOTE]
> Замените приведенный ниже пример конечной точки (с `<your-custom-subdomain>`) конечной точкой, созданной для вашего ресурса Анализа текста. Эту конечную точку можно найти, войдя на [портал Azure](https://azure.microsoft.com/features/azure-portal/), выбрав подписку, использующую Анализ текста, и затем выбрав `Quick start`.


```fsharp
// Returns key phrases from the text in a comma-separated list
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics" & "/v3.0/keyPhrases",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    keyphrases  = Text.Lower(Text.Combine(jsonresp[documents]{0}[keyPhrases], ", "))
in  keyphrases
```

Замените `YOUR_API_KEY_HERE` ключом доступа к своей службе "Анализ текста". Этот ключ можно также найти, войдя на [портал Azure](https://azure.microsoft.com/features/azure-portal/), выбрав подписку, использующую Анализ текста, и затем выбрав страницу "Обзор". (Не удаляйте кавычки вокруг ключа.) Нажмите **Готово**.

## <a name="use-the-custom-function"></a>Использование пользовательской функции
<a name="UseCustomFunction"></a>

Теперь можно использовать пользовательскую функцию, чтобы извлекать ключевые фразы из комментариев пользователей и сохранять их в новом столбце таблицы. 

В окне редактора запросов Power BI Desktop вернитесь к запросу `FabrikamComments`. Выберите ленту **Добавление столбца**. В группе **Общие** щелкните **Вызвать настраиваемую функцию**.

![[Кнопка "Вызвать настраиваемую функцию"]](../media/tutorials/power-bi/invoke-custom-function-button.png)<br><br>

Откроется диалоговое окно "Вызвать настраиваемую функцию". В поле **Новое имя столбца** введите `keyphrases`. Из списка **Запрос функции** выберите пользовательскую функцию, которую вы создали, `KeyPhrases`.

В диалоговом окне появится новое поле **Text (optional)** (Текст (необязательный)). Для этого поля будет предложено указать, какой столбец будет использоваться для предоставления значений параметра `text` API ключевых фраз. (Помните, что вы уже жестко запрограммировали значения параметров `language` и `id`.) В раскрывающемся меню выберите `Merged` (столбец, созданный [ранее](#PreparingData) путем объединения полей темы и сообщения).

![[Вызов настраиваемой функции]](../media/tutorials/power-bi/invoke-custom-function.png)

В конце нажмите кнопку **ОК**.

Если все готово, Power BI вызовет пользовательскую функцию по одному разу для каждой строки в таблице. Она отправляет запросы к API ключевых фраз и добавляет новый столбец в таблицу для сохранения результатов. Но перед этим может потребоваться указать параметры проверки подлинности и конфиденциальности.

## <a name="authentication-and-privacy"></a>Проверка подлинности и конфиденциальность
<a name="Authentication"></a>

После закрытия диалогового окна "Вызвать настраиваемую функцию" может появиться баннер с предложением указать способ подключения к API ключевых фраз.

![[Баннер учетных данных]](../media/tutorials/power-bi/credentials-banner.png)

Щелкните **Изменить учетные данные**, убедитесь, что выбран параметр `Anonymous`, затем выберите **Подключить**. 

> [!NOTE]
> Следует выбрать `Anonymous`, так как Служба "Анализ текста" выполнит аутентификацию с помощью ключа доступа, поэтому Power BI не нужно предоставлять учетные данные для самого HTTP-запроса.

> [!div class="mx-imgBorder"]
> ![[Выбор анонимного доступа для проверки подлинности]](../media/tutorials/power-bi/access-web-content.png)

Если баннер "Изменить учетные данные" все еще активен после выбора анонимного доступа, вероятно, вы забыли вставить ключ доступа к службе "Анализ текста" в коде `KeyPhrases` в [пользовательской функции](#CreateCustomFunction).

Затем появится баннер с предложением указать сведения об уровне конфиденциальности источника данных. 

![[Баннер с запросом конфиденциальных сведений]](../media/tutorials/power-bi/privacy-banner.png)

Щелкните **Продолжить** и выберите `Public` для всех источников данных в диалоговом окне. Нажмите **Сохранить**.

![[Настройка уровня конфиденциальности источника данных]](../media/tutorials/power-bi/privacy-dialog.png)

## <a name="create-the-word-cloud"></a>Создание облака слов
<a name="WordCloud"></a>

После завершения работы с баннерами на ленте "Главная" щелкните **Закрыть и применить**, чтобы закрыть редактор запросов.

Power BI Desktop требуется несколько секунд, чтобы выполнить необходимые HTTP-запросы. В новом столбце `keyphrases` будут ключевые фразы, обнаруженные API ключевых фраз в тексте каждой строки. 

Теперь с помощью этого столбца вы создадите облако слов. Сначала нажмите кнопку **Отчет** в главном окне Power BI Desktop, слева от рабочей области.

> [!NOTE]
> Зачем использовать извлеченные ключевые фразы вместо текста всего комментария? Ключевые фразы содержат *важные* слова из комментариев пользователей, а не только *самые распространенные*. Кроме того, размер слова в полученном облаке не изменяется в соответствии с частотой его использования в относительно небольшом числе комментариев.

Если у вас нет настраиваемого визуального элемента облака Word, установите его. На панели "Визуализации" справа щелкните многоточие ( **...** ) и выберите **Import From Market** (Импорт из Market). Если слово "облако" не отображается в названиях отображаемых средств визуализации в списке, используйте слово "облако" для поиска и нажмите кнопку **Добавить** рядом с визуальным элементом "Облако слов". Power BI установит визуальный элемент облака слов и сообщит об этом.

![[Добавление настраиваемого визуального элемента]](../media/tutorials/power-bi/add-custom-visuals.png)<br><br>

Сначала щелкните значок облака Word на панели "Визуализации".

![[Значок облака Word на панели визуализаций]](../media/tutorials/power-bi/visualizations-panel.png)

В рабочей области появится новый отчет. Перетащите поле `keyphrases` с панели "Поля" в поле "Категория" на панели "Визуализации". Облако Word появится внутри отчета.

Перейдите на страницу "Формат" на панели "Визуализации". В категории "Исключения" включите **Стоп-слова по умолчанию**, чтобы не добавлять в облако короткие распространенные слова, например "of". Но так как мы будем визуализировать ключевые фразы, они могут не содержать стоп-слова.

![[Активация стоп-слов по умолчанию]](../media/tutorials/power-bi/default-stop-words.png)

Далее отключите **Поворот текста** и **Заголовок**.

![[Активация режима фокусировки]](../media/tutorials/power-bi/word-cloud-focus-mode.png)

Нажмите кнопку "Режим фокусировки", чтобы улучшить отображение облака. Инструмент развернет облако на всю рабочую область, как показано ниже.

![[Облако Word]](../media/tutorials/power-bi/word-cloud.png)

## <a name="more-text-analytics-services"></a>Другие возможности службы "Анализ текста"
<a name="MoreServices"></a>

Служба "Анализ текста", как одна из служб Cognitive Services от Microsoft Azure, также поддерживает анализ тональности и распознавание языка. Распознавание языка особенно удобно использовать, когда комментарий пользователя содержит не только английские слова.

Оба этих API схожи с API ключевых фраз. Это означает, что вы можете интегрировать их с Power BI Desktop с помощью пользовательских функций, которые почти идентичны той, что вы создали в этом руководстве. Создайте пустой запрос и вставьте в Расширенный редактор код ниже, как вы делали раньше. (Не забудьте про ключ доступа!) Затем аналогичным способом используйте функцию, чтобы добавить новый столбец в таблицу.

Функция анализа тональности ниже возвращает оценку с указанием степени положительности тона текста.

```fsharp
// Returns the sentiment score of the text, from 0.0 (least favorable) to 1.0 (most favorable)
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/sentiment",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    sentiment   = jsonresp[documents]{0}[detectedLanguage][confidenceScore] in  sentiment
```

Ниже приведены две версии функции распознавания языка. Первая версия возвращает код языка ISO (например, `en` для английского), а вторая — его более привычное название (например, `English`). Обе версии отличаются лишь последней строкой.

```fsharp
// Returns the two-letter language code (for example, 'en' for English) of the text
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/languages",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    language    = jsonresp [documents]{0}[detectedLanguage] [iso6391Name] in language 
```
```fsharp
// Returns the name (for example, 'English') of the language in which the text is written
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/languages",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    language    jsonresp [documents]{0}[detectedLanguage] [iso6391Name] in language 
```

Ниже представлен вариант функции ключевых фраз, который возвращает фразы в виде списка, а не отдельной строки с разделителями-запятыми. 

> [!NOTE]
> Возврат отдельной строки упрощает наш пример облака Word. В свою очередь список — более гибкий формат работы с фразами в Power BI. Power BI Desktop позволяет управлять объектами списка с помощью группы "Структурированный столбец" в редакторе запросов на ленте "Преобразование".

```fsharp
// Returns key phrases from the text as a list object
(text) => let
    apikey      = "YOUR_API_KEY_HERE",
    endpoint    = "https://<your-custom-subdomain>.cognitiveservices.azure.com" & "/text/analytics/v3.0/keyPhrases",
    jsontext    = Text.FromBinary(Json.FromValue(Text.Start(Text.Trim(text), 5000))),
    jsonbody    = "{ documents: [ { language: ""en"", id: ""0"", text: " & jsontext & " } ] }",
    bytesbody   = Text.ToBinary(jsonbody),
    headers     = [#"Ocp-Apim-Subscription-Key" = apikey],
    bytesresp   = Web.Contents(endpoint, [Headers=headers, Content=bytesbody]),
    jsonresp    = Json.Document(bytesresp),
    keyphrases  = jsonresp[documents]{0}[keyPhrases]
in  keyphrases
```

## <a name="next-steps"></a>Дальнейшие действия
<a name="NextSteps"></a>

Дополнительные сведения о службе текстовой аналитики, языке формул Power Query M и Power BI см. в следующих статьях.

> [!div class="nextstepaction"]
> [Справочник по API анализа текста](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0)

> [!div class="nextstepaction"]
> [Справочник по Power Query M](/powerquery-m/power-query-m-reference)

> [!div class="nextstepaction"]
> [Документация по Power BI](https://powerbi.microsoft.com/documentation/powerbi-landing-page/)

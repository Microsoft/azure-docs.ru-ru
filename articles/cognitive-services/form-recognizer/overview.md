---
title: Что такое Распознаватель документов?
titleSuffix: Azure Cognitive Services
description: Служба "Распознаватель документов Azure" позволяет выявлять и извлекать пары "ключ-значение" и табличные данные из документов форм, а также извлекать основные сведения из квитанций о продажах и визитных карточек.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: overview
ms.date: 03/15/2021
ms.author: lajanuar
ms.custom: cog-serv-seo-aug-2020
keywords: автоматическая обработка данных, обработка документов, автоматическая запись данных, обработка форм
ms.openlocfilehash: 680bb612546aaffc167970c1c48a44159ef9af6f
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/15/2021
ms.locfileid: "107518242"
---
# <a name="what-is-form-recognizer"></a>Что такое Распознаватель документов?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Распознаватель документов Azure — это служба Cognitive Service, которая позволяет создавать автоматизированное программное обеспечение для обработки данных с помощью технологии машинного обучения. Эта служба идентифицирует и извлекает пары "ключ — значение", отметки выбора, таблицы и структуру из документов&mdash;она выводит структурированные данные, включая связи в исходном файле, ограничивающие прямоугольники, сведения о доверительном уровне и пр. Вы можете быстро получать точные результаты с учетом специфики содержимого, не прибегая к сложному ручному вмешательству или продолжительной обработке и анализу данных. Распознаватель документов позволяет автоматизировать ввод данных в приложения и расширяет возможности поиска документов.

Распознаватель документов состоит из пользовательских моделей обработки документов, предварительно созданных моделей для обработки счетов, квитанций, удостоверяющих личность документов и визитных карточек, а также модели макета. Вы можете вызывать модели Распознавателя документов с помощью REST API или пакетов SDK клиентских библиотек, чтобы упростить систему и интегрировать модель в рабочий процесс или приложение.

Эта документация включает статьи следующих типов:  

* [**Краткие руководства**](quickstarts/client-library.md) — инструкции по началу работы и отправке запросов в службу.  
* [**Руководства**](build-training-data-set.md) — содержат инструкции для более специфического или специализированного использования службы.  
* [**Статьи с основными понятиями**](concept-layout.md) — здесь подробно описываются функциональность и возможности службы.  
* [**Руководства**](tutorial-bulk-processing.md) — объемные статьи, в которых описываются способы использования службы в качестве компонента расширенных бизнес-решений.  

## <a name="form-recognizer-features"></a>Функции Распознавателя документов

С помощью Распознавателя документов можно легко извлечь и проанализировать данные формы, используя следующие функции:

* **[API макета](#layout-api)**  — извлекает из документа текста, отметки выбора и структуры таблиц наряду с координатами ограничивающих прямоугольников для них.
* **[Пользовательские модели](#custom-models)**  — извлекают из форм текст, табличные данные и пары "ключ — значение". Эти модели обучаются по вашим данным, а значит хорошо адаптированы для работы с вашими формами.

* **[Предварительно созданные модели](#prebuilt-models)**  — с их помощью вы можете извлекать данные из документов уникальных типов. Сейчас доступны указанные ниже предварительно созданные модели

  * [Счета](./concept-invoices.md)
  * [Квитанции продажи](./concept-receipts.md)
  * [Визитные карточки](./concept-business-cards.md)
  * [Удостоверяющие личность документы](./concept-identification-cards.md)


## <a name="get-started"></a>Начало работы

С помощью предоставленного примера Распознавателя документов изучите возможности макета и предварительно созданных моделей, а также обучите пользовательскую модель для своих документов. Чтобы опробовать Распознаватель документов, вам потребуется подписка Azure ([**создайте ее бесплатно**](https://azure.microsoft.com/free/cognitive-services)) и конечная точка или ключ [**ресурса Распознавателя документов**](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer).

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

> [!div class="nextstepaction"]
> [Опробовать Распознаватель документов](https://fott-preview.azurewebsites.net/)

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

> [!div class="nextstepaction"]
> [Опробовать Распознаватель документов](https://fott.azurewebsites.net/)

---
Чтобы приступить к извлечению данных из документов, следуйте инструкциям в [кратком руководстве по клиентской библиотеке и REST API](./quickstarts/client-library.md). При изучении технологии мы рекомендуем использовать бесплатную версию службы. Имейте в виду, что количество бесплатных страниц ограничено до 500 страниц в месяц.

Чтобы приступить к работе, можно также воспользоваться примерами REST с сайта GitHub: 

* Извлечение текста, отметок выбора и структуры таблиц из документов.
  * [Извлечение данных макета — Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-layout.md)
* Обучение пользовательских моделей и извлечение данных форм
  * [Обучение без использования меток — Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-train-extract.md)
  * [Обучение с использованием меток — Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-labeled-data.md)
* Извлечение данных из счетов
  * [Извлечение данных счета: Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-invoices.md)
* Извлечение данных из товарных чеков
  * [Извлечение данных товарного чека — Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-receipts.md)
* Извлечение данных из визитных карточек
  * [Извлечение данных визитной карточки — Python](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/FormRecognizer/rest/python-business-cards.md)

### <a name="review-the-rest-apis"></a>Ознакомьтесь с интерфейсами REST API.

Воспользуйтесь следующими API для обучения моделей и извлечения структурированных данных из форм.

|Имя |Описание |
|---|---|
| **Анализ макета** | Анализ документа, переданного в виде потока, для извлечения текста, отметок выбора, таблиц и структуры. |
| **Обучение пользовательской модели**| Обучите новую модель для анализа своих форм с помощью пяти форм такого же типа. Задайте для параметра _useLabelFile_ значение `true`, чтобы обучить данные с вручную проставленными метками. |
| **Анализ формы** |Проанализируйте форму, переданную в виде потока, чтобы извлечь текст, пары "ключ — значение" и таблицы из формы с помощью настраиваемой модели.  |
| **Анализ счета** | Анализ счета для извлечения основных сведений, таблиц и другого текста.|
| **Анализ квитанции** | Анализ документа квитанции для извлечения основных сведений и другого текста.|
| **Анализ удостоверяющего личность документа** | Анализ удостоверяющих личность документов для извлечения основных сведений и другого текста.|
| **Анализ визитной карточки** | Выполните анализ визитной карточки, чтобы извлечь основные сведения и текст.|

### <a name="v21-preview"></a>[Предварительная версия 2.1](#tab/v2-1)

Ознакомьтесь со [справочной документацией по REST API](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-3/operations/AnalyzeWithCustomForm), чтобы узнать больше. Если вы уже знакомы с предыдущей версией API, обратитесь к статье [о новых возможностях и изменениях](./whats-new.md).

### <a name="v20"></a>[Версия 2.0](#tab/v2-0)

Ознакомьтесь со [справочной документацией по REST API](https://westus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeLayoutAsync), чтобы узнать больше. Если вы уже знакомы с предыдущей версией API, обратитесь к статье [о новых возможностях и изменениях](./whats-new.md).

---

## <a name="layout-api"></a>API макета

Распознаватель документов может извлекать из документов текст, отметки выбора и структуру таблиц (номера строк и столбцов, связанные с текстом) с помощью технологии оптического распознавания символов (OCR) и расширенной модели глубокого обучения. Дополнительные сведения см. в концептуальном руководстве по [использованию макета](./concept-layout.md).

:::image type="content" source="./media/tables-example.jpg" alt-text="Пример таблиц" lightbox="./media/tables-example.jpg":::

## <a name="custom-models"></a>Настраиваемые модели

Настраиваемая модель Распознавателя документов обучается на основе ваших данных. Для начала работы достаточно указать пять примеров форм ввода. Обученная модель обработки документов выводит структурированные данные, которые содержат связи из исходного документа формы. После обучения модели вы можете проверить ее, повторно обучить и, в конечном итоге, применить с целью надежного извлечения данных из нескольких форм в соответствии со своими потребностями.

При обучении настраиваемых моделей доступны следующие варианты: обучение с помеченными данными и без помеченных данных.

### <a name="train-without-labels"></a>Обучение без меток

Распознаватель документов использует неконтролируемое обучение, чтобы изучить макет и связи между полями и записями формы. При получении входных форм алгоритм кластеризует их по типам, обнаруживает имеющиеся ключи и таблицы, а также связывает значения с ключами и записями в таблицах. Обучение без меток не требует маркировки данных вручную и больших усилий для написания кода и обслуживания, поэтому мы рекомендуем всегда начинать с него.

Советы и варианты для сбора документов для обучения см. в [этой статье](./build-training-data-set.md).

### <a name="train-with-labels"></a>Обучение с метками

При обучении с помеченными данными модель использует контролируемое обучение для извлечения важных значений из предоставленных форм с метками. Данные с метками повышают эффективность моделей и позволяют получить модели для достаточно сложных форм и (или) форм со значениями без ключей.

Распознаватель документов использует [API макета](#layout-api) для изучения ожидаемых размеров и положений печатных и рукописных элементов текста и для извлечения таблиц. Затем он применяет заданные пользователем метки для изучения связей "ключ — значение" и таблиц в предоставленных документах. Мы рекомендуем использовать не менее пяти форм одного типа (одной структуры) с проставленными вручную метками, чтобы начать обучение новой модели, и постепенно добавлять данные с метками для повышения ее точности. Распознаватель документов позволяет обучить модель для извлечения пар "ключ — значение" и таблиц с помощью контролируемого обучения. 

[Начало работы с функцией обучения с использованием меток](./quickstarts/label-tool.md)

> [!VIDEO https://channel9.msdn.com/Shows/Docs-Azure/Azure-Form-Recognizer/player]

## <a name="prebuilt-models"></a>Предварительно созданные модели

Распознаватель документов также включает предварительно созданные модели для автоматической обработки данных уникальных типов форм.

### <a name="prebuilt-invoice-model"></a>Предварительно созданная модель счетов

Предварительно созданная модель счетов извлекает данные из счетов в разных форматах и возвращает структурированные данные. Эта модель извлекает ключевые сведения, такие как идентификатор счета, сведения о клиенте, поставщике, доставке, выставлении счета, сумме, налоге, промежуточной сумме, элементах строк и т. д. Кроме того, предварительно созданная модель для обработки счетов обучается анализировать и возвращать весь текст и таблицы в счете. Дополнительные сведения см. в концептуальном руководстве по [обработке счетов](./concept-invoices.md).

:::image type="content" source="./media/overview-invoices.jpg" alt-text="Пример счета" lightbox="./media/overview-invoices.jpg":::

### <a name="prebuilt-receipt-model"></a>Предварительно созданная модель для обработки квитанций

Предварительно созданная модель для обработки квитанций используется для чтения типичных для Австралии, Канады, Великобритании, Индии и США квитанций на английском языке &mdash; таких, которые используются ресторанами, заправочными станциями, розничными магазинами и т. д. Эта модель извлекает основные сведения, например сведения о времени и дате транзакции, о продавце, сумме налогов, позиции заказа, общей сумме и многое другое. Кроме того, предварительно созданная модель для обработки квитанций обучается анализировать и возвращать весь текст в чеке. Дополнительные сведения см. в концептуальном руководстве по [обработке квитанций](./concept-receipts.md).

:::image type="content" source="./media/overview-receipt.jpg" alt-text="Пример квитанции" lightbox="./media/overview-receipt.jpg":::

### <a name="prebuilt-identification-id-cards-model"></a>Предварительно созданная модель удостоверяющих личность документов

Модель удостоверяющих личность документов позволяет извлекать основные сведения из международных паспортов и водительских прав. Она извлекает такие данные, как идентификатор документа, дата рождения, окончание срока действия, имя, страна, регион, машиночитаемая зона и многое другое. Подробнее см. в руководстве по [удостоверяющим личность документам](./concept-identification-cards.md).

:::image type="content" source="./media/overview-id.jpg" alt-text="Пример удостоверяющего личность документа" lightbox="./media/overview-id.jpg":::

### <a name="prebuilt-business-cards-model"></a>Предварительно созданная модель для обработки визитных карточек

Модель для обработки визитных карточек позволяет извлекать такие сведения, как имя лица, должность, адрес, адрес электронной почты, название компании и номера телефонов, из визитных карточек на английском языке. Дополнительные сведения см. в концептуальном руководстве по [обработке визитных карточек](./concept-business-cards.md).

:::image type="content" source="./media/overview-business-card.jpg" alt-text="Пример визитной карточки" lightbox="./media/overview-business-card.jpg":::

## <a name="input-requirements"></a>Требования к входным данным

[!INCLUDE [input requirements](./includes/input-requirements.md)]

## <a name="deploy-on-premises-using-docker-containers"></a>Развертывание в локальной среде с помощью контейнеров Docker

Для развертывания функций API в локальной среде [используйте контейнеры Распознавателя документов (предварительная версия)](form-recognizer-container-howto.md). Этот контейнер Docker позволяет разместить службу ближе к данным для обеспечения безопасности, соответствия требованиям и других эксплуатационных преимуществ.

## <a name="service-availability-and-redundancy"></a>Доступность и избыточность службы

### <a name="is-form-recognizer-service-zone-resilient"></a>Является ли служба "Распознаватель документов" устойчивой в пределах зоны?

Да. Для службы "Распознаватель документов" по умолчанию обеспечивается устойчивость в пределах зоны.

### <a name="how-do-i-configure-the-form-recognizer-service-to-be-zone-resilient"></a>Как настроить службу "Распознаватель документов" для устойчивости в пределах зоны?

Чтобы включить устойчивость зоны, настройка со стороны клиента не требуется. Устойчивость в пределах зоны для ресурсов Распознавателя документов доступна по умолчанию и управляется самой службой.

## <a name="data-privacy-and-security"></a>Конфиденциальность и безопасность данных

Как и в случае со всеми другими службами Cognitive Services, разработчикам, использующим службу Распознавателя документов, следует учитывать политику корпорации Майкрософт касательно клиентских данных. Дополнительные сведения см. на [странице о Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) Центра управления безопасностью Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия

Воспользуйтесь нашим веб-инструментом и кратким руководством для получения дополнительных сведений о службе Распознавателя документов.

* [**Инструмент "Распознаватель документов"**](https://fott-preview.azurewebsites.net/)
* [**Краткое руководство. Использование клиентской библиотеки или REST API Распознавателя документов**](quickstarts/client-library.md)

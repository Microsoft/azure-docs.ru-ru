---
title: Краткое руководство по использованию клиентской библиотеки Content Moderator для Python
titleSuffix: Azure Cognitive Services
description: Из этого краткого руководства вы узнайте, как приступить к работе с клиентской библиотекой Content Moderator для Python. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: include
ms.date: 09/15/2020
ms.custom: cog-serv-seo-aug-2020
ms.author: pafarley
ms.openlocfilehash: 06ff04d8615b7ebdda07e993a3fc560d44fbf702
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105641"
---
Приступайте к работе с клиентской библиотекой Azure Content Moderator для Python. Выполните приведенные здесь действия, чтобы установить пакет PiPy и протестировать пример кода для выполнения базовых задач. 

Content Moderator — это служба ИИ, позволяющая управлять содержимым, которое может носить оскорбительный характер или представлять опасность, либо нежелательным содержимым иного рода. Используйте службу модерации содержимого на основе ИИ, чтобы проверить текст, изображения и видео, а также автоматически применить флаги содержимого. Затем интегрируйте свое приложение со средством проверки, онлайн-средой модерации для команды специалистов-рецензентов. Встройте в свое приложение программное обеспечение для фильтрации содержимого, чтобы обеспечить соответствие нормативным требованиям и настроить надлежащую среду для пользователей.

Клиентскую библиотеку Content Moderator для Python можно использовать для следующих задач:

* Модерация текста
* Использование пользовательского списка терминов
* Модерация изображений
* Использование пользовательского списка изображений
* Создание проверки

[Справочная документация](/python/api/overview/azure/cognitiveservices/contentmoderator) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-contentmoderator) | [Пакет (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-contentmoderator/) | [Примеры](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [Python 3.x](https://www.python.org/)
  * Установка Python должна включать [pip](https://pip.pypa.io/en/stable/). Чтобы проверить, установлен ли pip, выполните команду `pip --version` в командной строке. Чтобы использовать pip, установите последнюю версию Python.
* Получив подписку Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator"  title="Создание ресурса Content Moderator"  target="_blank">создайте ресурс Content Moderator</a> на портале Azure, чтобы получить ключ и конечную точку. Дождитесь, пока закончится развертывание, и нажмите кнопку **Перейти к ресурсу**.
    * Для подключения приложения к Content Moderator потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.


## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python вы можете установить клиентскую библиотеку Content Moderator с помощью следующей команды:

```console
pip install --upgrade azure-cognitiveservices-vision-contentmoderator
```

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте скрипт Python и откройте его в предпочитаемом редакторе или интегрированной среде разработки. В начало файла добавьте следующие инструкции `import`.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/ContentModerator/ContentModeratorQuickstart.py), где размещены примеры кода для этого краткого руководства.

Затем создайте переменные для ключа и расположения конечной точки ресурса.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_vars)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс Content Moderator, созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

## <a name="object-model"></a>Объектная модель

Следующие классы обрабатывают некоторые основные функции клиентской библиотеки Content Moderator для Python.

|Имя|Описание|
|---|---|
|[ContentModeratorClient](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.content_moderator_client.contentmoderatorclient)|Этот класс требуется для всех функций Content Moderator. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов.|
|[ImageModerationOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.imagemoderationoperations)|Этот класс предоставляет функции для анализа изображений на наличие содержимого для взрослых, личной информации или лиц людей.|
|[TextModerationOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.textmoderationoperations)|Этот класс предоставляет функции для анализа текста с целью определения языка, наличия ненормативной лексики, ошибок и личной информации.|
[ReviewsOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.reviewsoperations)|Этот класс предоставляет функциональные возможности API-интерфейсов проверки, в том числе методы создания заданий, настраиваемых рабочих процессов и пользовательских проверок.|

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки Content Moderator для Python:

* [аутентификация клиента](#authenticate-the-client);
* [модерация текста](#moderate-text);
* [использование пользовательского списка терминов](#use-a-custom-terms-list);
* [модерация изображений](#moderate-images);
* [использование пользовательского списка изображений](#use-a-custom-image-list);
* [создание проверки](#create-a-review).

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр клиента с конечной точкой и ключом. Создайте объект [CognitiveServicesCredentials](/python/api/msrest/msrest.authentication.cognitiveservicescredentials) с ключом и используйте его с конечной точкой, чтобы создать объект [ContentModeratorClient](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.content_moderator_client.contentmoderatorclient).

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_client)]

## <a name="moderate-text"></a>Модерация текста

В следующем коде используется клиент Content Moderator для анализа текста и вывода результатов в консоль. Сначала создайте папку **text_files/** в корне проекта и добавьте в нее файл *content_moderator_text_moderation.txt*. Добавьте в этот файл свой текст или используйте текст из этого примера:

```
Is this a grabage email abcdef@abcd.com, phone: 4255550111, IP: 255.255.255.255, 1234 Main Boulevard, Panapolis WA 96555.
Crap is the profanity here. Is this information PII? phone 2065550111
```

Добавьте ссылку на новую папку.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_textfolder)]

Затем добавьте в скрипт Python следующий код.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_textmod)]

## <a name="use-a-custom-terms-list"></a>Использование пользовательского списка терминов

Следующий код демонстрирует, как модерировать текст с помощью пользовательского списка терминов. Класс [ListManagementTermListsOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.listmanagementtermlistsoperations) предназначен для создания списка терминов, управления отдельными терминами и проверки текста на наличие терминов из списка.

### <a name="get-sample-text"></a>Создание файла с текстом

Для работы с текстом сначала создайте папку **text_files/** в корне проекта и добавьте в нее файл *content_moderator_term_list.txt*. Этот файл должен содержать текст, который будет проверяться по списку терминов. Вы можете использовать следующий пример текста:

```
This text contains the terms "term1" and "term2".
```

Добавьте ссылку на папку, если она еще не определена.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_textfolder)]

### <a name="create-a-list"></a>Создать список

Добавьте следующий код в скрипт Python, чтобы создать пользовательский список терминов и сохранить значение его идентификатора.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_create)]

### <a name="define-list-details"></a>Определение сведений о списке

Идентификатор списка можно использовать для изменения его имени и описания.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_details)]

### <a name="add-a-term-to-the-list"></a>Добавление термина в список

Следующий код добавляет в список термины `"term1"` и `"term2"`.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_add)]

### <a name="get-all-terms-in-the-list"></a>Получение всех терминов из списка

Идентификатор списка можно использовать для получения всех терминов из списка.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_getterms)]

### <a name="refresh-the-list-index"></a>Обновление индекса списка

При добавлении терминов в список или их удалении оттуда необходимо обновить индекс, прежде чем можно будет использовать обновленный список.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_refresh)]

### <a name="screen-text-against-the-list"></a>Проверка текста на наличие терминов из списка

Основная задача пользовательского списка терминов заключается в проверке текста на наличие терминов, содержащихся в списке. 

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_screen)]

### <a name="remove-a-term-from-a-list"></a>Удаление термина из списка

Следующий код удаляет термин `"term1"` из списка.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_remove)]

### <a name="remove-all-terms-from-a-list"></a>Удаление всех терминов из списка

Используйте следующий код, чтобы удалить все термины из списка.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_removeall)]

### <a name="delete-a-list"></a>Удаление списка

Используйте следующий код, чтобы удалить пользовательский список терминов.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_termslist_deletelist)]

## <a name="moderate-images"></a>Модерация изображений

В следующем коде используется клиент Content Moderator и объект [ImageModerationOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.imagemoderationoperations) для анализа изображений на наличие содержимого для взрослых и содержимого непристойного характера.

### <a name="get-sample-images"></a>Получить образцы изображений

Определите ссылку на изображения для анализа.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemodvars)]

Затем добавьте следующий код для последовательного анализа изображений. Все остальные примеры кода из этого раздела будут добавляться в этот цикл.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_iterate)]

### <a name="check-for-adultracy-content"></a>Проверка на наличие содержимого для взрослых и содержимого непристойного характера

Следующий код проверяет изображение по указанному URL-адресу на наличие содержимого для взрослых и содержимого непристойного характера, а затем выводит результаты в консоль. См. также о [принципах модерации изображений](../../image-moderation-api.md).

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_ar)]

### <a name="check-for-visible-text"></a>Проверка на наличие распознаваемого текста

Следующий код проверяет изображение на наличие распознаваемого текста, а затем выводит результаты в консоль.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_text)]

### <a name="check-for-faces"></a>Проверка на наличие лиц

Следующий код проверяет изображение на наличие лиц, а затем выводит результаты в консоль.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagemod_face)]

## <a name="use-a-custom-image-list"></a>Использование пользовательского списка изображений

Следующий код демонстрирует, как модерировать изображения с помощью настраиваемого списка изображений. Эта функция полезна, если вам нужно проверять часто поступающие экземпляры одного и того же набора изображений. Список таких изображений позволит повысить производительность. Класс [ListManagementImageListsOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.listmanagementimagelistsoperations) предназначен для создания списка изображений, управления отдельными изображениями списка и сравнения других изображений с изображениями из списка.

Создайте следующие текстовые переменные, чтобы указать URL-адреса изображений, которые будут использоваться в этом сценарии.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelistvars)]

> [!NOTE]
> Выше показан не полноценный список изображений, а пример списка изображений, которые будут добавлены в раздел кода `add images`.


### <a name="create-an-image-list"></a>Создание списка изображений

Добавьте следующий код, чтобы создать список изображений и сохранить ссылку на его идентификатор.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_create)]

### <a name="add-images-to-a-list"></a>Добавление изображений в список

Следующий код добавляет все изображения в список.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_add)]

Определите вспомогательную функцию **add_images** в любом месте скрипта.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_addhelper)]

### <a name="get-images-in-list"></a>Получение изображений из списка

Следующий код выводит имена всех изображений, содержащихся в списке.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_getimages)]

### <a name="update-list-details"></a>Обновление сведений о списке

Идентификатор списка можно использовать для обновления имени и описания списка.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_updatedetails)]

### <a name="get-list-details"></a>Получение сведений о списке

Используйте следующий код, чтобы получить текущие сведения о списке.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_getdetails)]

### <a name="refresh-the-list-index"></a>Обновление индекса списка

При добавлении изображений в список или их удалении оттуда необходимо обновить индекс списка, прежде чем его можно будет использовать для проверки других изображений.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_refresh)]

### <a name="match-images-against-the-list"></a>Сопоставление изображений с изображениями из списка

Основная задача списка изображений заключается в сопоставлении анализируемых изображений с изображениями, содержащимися в списке.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_match)]

### <a name="remove-an-image-from-the-list"></a>Удаление изображения из списка

Следующий код удаляет элемент из списка. В нашем примере это изображение, которое не соответствует категории списка.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_remove)]

### <a name="remove-all-images-from-a-list"></a>Удаление всех изображений из списка

Используйте следующий код, чтобы очистить список изображений.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_removeall)]

### <a name="delete-the-image-list"></a>Удаление списка изображений

Используйте следующий код, чтобы удалить указанный список изображений.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagelist_delete)]

## <a name="create-a-review"></a>Создание проверки

С помощью клиентской библиотеки Content Moderator для Python можно передать содержимое в [средство проверки](https://contentmoderator.cognitive.microsoft.com) для его проверки человеком. Чтобы узнать больше о средстве проверки, ознакомьтесь с разделом [Средство проверки Content Moderator](../../review-tool-user-guide/human-in-the-loop.md).

В следующем коде класс [ReviewsOperations](/python/api/azure-cognitiveservices-vision-contentmoderator/azure.cognitiveservices.vision.contentmoderator.operations.reviewsoperations) используется для создания проверки, получения ее идентификатора и уточнения сведений о ней после выполнения действий пользователем-модератором на веб-портале средства проверки.

### <a name="get-review-credentials"></a>Получение учетных данных проверки

Сначала войдите в средство проверки, чтобы узнать имя своей группы проверки. Затем назначьте его соответствующей переменной в коде. Вы можете также настроить конечную точку обратного вызова для получения сведений о действиях проверки.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagereview_vars)]

### <a name="create-an-image-review"></a>Создание проверки изображения

Добавьте следующий код, чтобы создать проверку изображения по указанному URL-адресу и передать результаты проверки. Код сохраняет ссылку на идентификатор проверки. 
[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagereview_create)]

### <a name="get-review-details"></a>Получение сведений о проверке

Используйте следующий код для уточнении сведений о проверке. После создания проверки вы можете открыть средство проверки для непосредственного взаимодействия с содержимым. Дополнительные сведения о том, как это сделать, приведены в [практическом руководстве по проверкам](../../review-tool-user-guide/review-moderated-images.md). По завершении вы можете снова запустить этот код, чтобы получить результаты проверки.

[!code-python[](~/cognitive-services-quickstart-code/python/ContentModerator/ContentModeratorQuickstart.py?name=snippet_imagereview_getdetails)]

Если в этом сценарии использовалась конечная точка обратного вызова, в ней должно быть получено событие в таком формате:

```console
{'callback_endpoint': 'https://requestb.in/qmsakwqm',
 'content': '',
 'content_id': '3ebe16cb-31ed-4292-8b71-1dfe9b0e821f',
 'created_by': 'cspythonsdk',
 'metadata': [{'key': 'sc', 'value': 'True'}],
 'review_id': '201901i14682e2afe624fee95ebb248643139e7',
 'reviewer_result_tags': [{'key': 'a', 'value': 'True'},
                          {'key': 'r', 'value': 'True'}],
 'status': 'Complete',
 'sub_team': 'public',
 'type': 'Image'}
```

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `python` для файла quickstart.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководстве вы узнали, как модерировать содержимое с помощью библиотеки Python для Content Moderator. Теперь узнайте больше о модерации изображений или другого мультимедиа, прочитав концептуальное руководство.

> [!div class="nextstepaction"]
>[Принципы модерации изображений](../../image-moderation-api.md)

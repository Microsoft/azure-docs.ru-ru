---
title: Краткое руководство по использованию клиентской библиотеки API "Распознавание лиц" (Python)
description: Клиентская библиотека API Распознавания лиц для Python позволяет обнаруживать лица, находить похожие лица (поиск лиц по изображению) и идентифицировать лица (поиск с распознаванием лиц).
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: include
ms.date: 11/10/2020
ms.author: pafarley
ms.openlocfilehash: 0939216e1d96e898d7c45b9e7a3ec72453b263d6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105958358"
---
В этом руководстве показано, как начать работу с клиентской библиотекой API Распознавания лиц для Python. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач. В службе "Распознавание лиц" доступны передовые алгоритмы обнаружения и распознавания лиц на изображениях.

Клиентская библиотека Распознавания лиц для Python позволяет выполнять такие задачи:

* [Определение лиц на изображении](#detect-faces-in-an-image)
* [поиск похожих лиц](#find-similar-faces);
* [Создание и обучение PersonGroup](#create-and-train-a-persongroup)
* [опознание лица](#identify-a-face);
* [Проверка лиц](#verify-faces)

[Справочная документация](/python/api/azure-cognitiveservices-vision-face/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-face) | [Пакет (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-face/) | [Примеры](/samples/browse/?products=azure&term=face)

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [Python 3.x](https://www.python.org/)
  * Установка Python должна включать [pip](https://pip.pypa.io/en/stable/). Чтобы проверить, установлен ли pip, выполните команду `pip --version` в командной строке. Чтобы использовать pip, установите последнюю версию Python.
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="создайте ресурс "Распознавание лиц""  target="_blank">Создание ресурса "Распознавание лиц"</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.
    * Для подключения приложения к API Распознавания лиц потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка
 
### <a name="install-the-client-library"></a>Установка клиентской библиотеки

После установки Python вы можете установить клиентскую библиотеку с помощью следующей команды:

```console
pip install --upgrade azure-cognitiveservices-vision-face
```

### <a name="create-a-new-python-application"></a>Создание приложения Python

Создайте скрипт Python с произвольным именем, например&mdash;*quickstart-file.py*. Откройте его в редакторе или интегрированной среде разработки и импортируйте следующие библиотеки.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_imports)]

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py), где размещены примеры кода для этого краткого руководства.

Затем создайте переменные для конечной точки и ключа ресурса Azure.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_subvars)]

> [!IMPORTANT]
> Перейдите на портал Azure. Если ресурс "Распознавание лиц", созданный в соответствии с указаниями в разделе **Предварительные требования**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ и конечная точка располагаются на странице **ключа и конечной точки** ресурса в разделе **управления ресурсами**. 
>
> Не забудьте удалить ключ из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы воплощают некоторые основные функции клиентской библиотеки API "Распознавание лиц" для Python.

|Имя|Описание|
|---|---|
|[FaceClient](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient) | Этот класс реализует авторизацию для использования Распознавания лиц и требуется для реализации всех ее функций. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов. |
|[FaceOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations)|Этот класс обрабатывает основные задачи по обнаружению и распознаванию лиц. |
|[DetectedFace](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface)|Этот класс представляет все данные об отдельном лице, обнаруженном на изображении. Его можно использовать для получения подробных сведений о лице.|
|[FaceListOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.facelistoperations)|Этот класс управляет хранимыми в облаке конструкциями **FaceList**, которые включают систематизированную коллекцию лиц. |
|[PersonGroupPersonOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations)| Этот класс управляет хранимыми в облаке конструкциями **Person**, в которых хранится коллекция лиц одного человека.|
|[PersonGroupOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongroupoperations)| Этот класс управляет хранимыми в облаке конструкциями **PersonGroup**, в которых хранится систематизированная коллекция объектов **Person**. |
|[SnapshotOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.snapshotoperations)|Этот класс управляет функцией создания моментального снимка. Его можно использовать для временного сохранения всех хранимых в облаке данных о лицах и переноса этих данных в новую подписку Azure. |

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода демонстрируют выполнение следующих действий с помощью клиентской библиотеки API Распознавания лиц для Python:

* [аутентификация клиента](#authenticate-the-client);
* [Определение лиц на изображении](#detect-faces-in-an-image)
* [поиск похожих лиц](#find-similar-faces);
* [Создание и обучение PersonGroup](#create-and-train-a-persongroup)
* [опознание лица](#identify-a-face);
* [Проверка лиц](#verify-faces)

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр клиента с конечной точкой и ключом. Создайте объект [CognitiveServicesCredentials](/python/api/msrest/msrest.authentication.cognitiveservicescredentials) с ключом и используйте его с конечной точкой, чтобы создать объект [FaceClient](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.faceclient).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_auth)]

## <a name="detect-faces-in-an-image"></a>Определение лиц на изображении

Следующий код обнаруживает лицо на удаленно хранящемся изображении. Он выводит идентификатор обнаруженного лица в консоль, а также сохраняет его в памяти программы. Затем он обнаруживает лица на изображении с несколькими людьми и выводит их идентификаторы в консоль. Изменяя параметры в методе [detect_with_url](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations#detect-with-url-url--return-face-id-true--return-face-landmarks-false--return-face-attributes-none--recognition-model--recognition-01---return-recognition-model-false--detection-model--detection-01---custom-headers-none--raw-false----operation-config-), можно получать разные сведения о каждом объекте [DetectedFace](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.models.detectedface).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_detect)]

> [!TIP]
> Обнаружение лиц можно также выполнить, используя локальное изображение. Изучите информацию о методах класса [FaceOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations), например о **detect_with_stream**.

### <a name="display-and-frame-faces"></a>Отображение лиц и их выделение рамкой

Следующий код выводит заданное изображение на экран и отмечает рамкой присутствующие на нем лица с помощью свойства DetectedFace.faceRectangle.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_frame)]

![Молодая женщина, вокруг лица которой нарисован красный прямоугольник](../../images/face-rectangle-result.png)

## <a name="find-similar-faces"></a>Поиск похожих лиц

Следующий код принимает одно обнаруженное лицо (источник) и выполняет поиск совпадений (поиск лиц по изображению) в коллекции других лиц (цель). При обнаружении совпадения в консоль выводится идентификатор лица, с которым найдено совпадение.

### <a name="find-matches"></a>Поиск совпадений

Сначала выполните код из предыдущего раздела ([Определение лиц на изображении](#detect-faces-in-an-image)), чтобы сохранить ссылку на отдельное лицо. Затем выполните следующий код, чтобы получить ссылки на несколько лиц на изображении группы людей.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_detectgroup)]

Затем добавьте следующий блок кода, чтобы найти экземпляры первого лица в группе. Сведения о том, как изменить это поведение, см. в описании метода [find_similar](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.faceoperations#find-similar-face-id--face-list-id-none--large-face-list-id-none--face-ids-none--max-num-of-candidates-returned-20--mode--matchperson---custom-headers-none--raw-false----operation-config-).

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_findsimilar)]

### <a name="print-matches"></a>Вывод совпадений

Используйте следующий код, чтобы вывести сведения о совпадениях в консоль.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_findsimilar_print)]

## <a name="create-and-train-a-persongroup"></a>Создание и обучение PersonGroup

Следующий код создает объект **PersonGroup** с тремя объектами **Person**. Он соотносит каждый объект **Person** с коллекцией примеров изображений, а затем походит обучение, чтобы распознать каждое лицо. 

### <a name="create-persongroup"></a>Создание PersonGroup

Для выполнения этого сценария необходимо сохранить следующие изображения в корневом каталоге проекта https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

Эта группа изображений содержит три коллекции изображений лиц, соотносящихся с лицами трех разных людей. Код определит три объекта **Person** и соотнесет их с файлами изображений, которые начинаются с префикса `woman`, `man` и `child`.

После настройки изображений определите метку в верхней части скрипта для создаваемого объекта **PersonGroup**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroupvars)]

Добавьте следующий код в нижнюю часть скрипта. Этот код создает объект **PersonGroup** и три объекта **Person**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_create)]

### <a name="assign-faces-to-persons"></a>Назначение лиц объектам Person

Следующий код сортирует изображения по префиксу, обнаруживает лица и назначает лица каждому объекту **Person**.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_assign)]

> [!TIP]
> Объект **PersonGroup** можно также создать на основе удаленно размещенных изображений, используя их URL-адреса. Изучите информацию о методах класса [PersonGroupPersonOperations](/python/api/azure-cognitiveservices-vision-face/azure.cognitiveservices.vision.face.operations.persongrouppersonoperations), например о **add_face_from_url**.

### <a name="train-persongroup"></a>Обучение PersonGroup

После назначения лиц необходимо обучить объект **PersonGroup**, чтобы он мог опознавать визуальные черты, связанные с каждым из его объектов **Person**. Следующий код вызывает асинхронный метод **train**, запрашивает результат и выводит состояние в консоль.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_persongroup_train)]

> [!TIP]
> API службы "Распознавание лиц" выполняется на наборе предварительно созданных моделей, которые по своей природе являются статическими (производительность модели не ухудшается и не улучшается при выполнении службы). Результаты, предоставляемые моделью, могут измениться, если корпорация Майкрософт обновит внутреннюю структуру модели без перехода на полностью новую версию модели. Чтобы воспользоваться преимуществами новой версии модели, вы можете повторно обучить **PersonGroup**, указав более новую модель в качестве параметра с такими же изображениями для регистрации.

## <a name="identify-a-face"></a>опознание лиц;

Операция идентификации (Identify) принимает изображение человека или нескольких людей и пытается опознать каждое лицо на этом изображении (поиск с распознаванием лиц). Он сравнивает каждое обнаруженное лицо с **PersonGroup**, которая является базой данных объектов **Person** с известными характеристиками лиц.

> [!IMPORTANT]
> Чтобы выполнить этот пример, сначала необходимо выполнить код из раздела [Создание и обучение PersonGroup](#create-and-train-a-persongroup).

### <a name="get-a-test-image"></a>Получение тестового изображения

Следующий код ищет в корневом каталоге проекта изображение _test-image-person-group.jpg_ и обнаруживает на нем лица. Это изображение есть в коллекции изображений, использовавшихся для создания и обучения **PersonGroup**. Оно доступно по адресу https://github.com/Azure-Samples/cognitive-services-sample-data-files/tree/master/Face/images.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_identify_testimage)]

### <a name="identify-faces"></a>Идентификация лиц

Метод **identify** принимает массив обнаруженных лиц и сравнивает их с **PersonGroup**. Если он находит совпадение с **Person**, он сохраняет результат. Этот код выводит в консоль подробные сведения о совпадении.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_identify)]

## <a name="verify-faces"></a>Проверка лиц

Операция проверки принимает идентификатор лица, идентификатор другого лица или объекта **Person**, а затем определяет, связаны ли они с одним и тем же человеком.

Следующий код определяет лица на двух исходных изображениях, а затем сопоставляет их с лицом, определенным на целевом изображении.

### <a name="get-test-images"></a>Получение тестовых изображений

Следующий код блокирует объявление переменных, которые указывают на исходное и целевое изображения для операции проверки.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_baseurl)]

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_photos)]

### <a name="detect-faces-for-verification"></a>Определение лиц для проверки

Следующий код определяет лица на исходном и целевом изображениях и сохраняет их в переменных.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify_detect)]

### <a name="get-verification-results"></a>Получение результатов проверки

Следующий код сравнивает все исходные изображения с целевым изображением и выводит сообщение, которое указывает, связаны ли они с одним и тем же человеком.

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_verify)]

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение распознавания лиц из каталога приложения с помощью команды `python`.

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

Если при изучении этого краткого руководства вы создали объект **PersonGroup** и хотите его удалить, выполните в скрипте следующий код:

[!code-python[](~/cognitive-services-quickstart-code/python/Face/FaceQuickstart.py?name=snippet_deletegroup)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как с помощью клиентской библиотеки API Распознавания лиц для Python выполнять базовые задачи по распознаванию лиц. Далее ознакомьтесь со справочной документацией, чтобы узнать больше о библиотеке.

> [!div class="nextstepaction"]
> [Справочник по API Распознавания лиц (Python)](/python/api/azure-cognitiveservices-vision-face/)

* [Что представляет собой служба "Распознавание лиц"?](../../overview.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/Face/FaceQuickstart.py).

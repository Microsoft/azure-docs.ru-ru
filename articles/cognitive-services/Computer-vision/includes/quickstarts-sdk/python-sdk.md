---
title: Краткое руководство. Клиентская библиотека оптического распознавания символов для Python
description: В этом кратком руководстве описано, как приступить к работе с клиентской библиотекой оптического распознавания символов для Python.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 12/15/2020
ms.author: pafarley
ms.openlocfilehash: d0b2a854391097cc7d95c4286ba581f3660d397e
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284827"
---
<a name="HOLTop"></a>

Используйте клиентскую библиотеку оптического распознавания символов для чтения печатных и рукописных текстов с помощью API чтения.

[Справочная документация](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cognitiveservices/azure-cognitiveservices-vision-computervision) | [Пакет (PiPy)](https://pypi.org/project/azure-cognitiveservices-vision-computervision/) | [Примеры](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>Предварительные требования

* подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/).
* [Python 3.x](https://www.python.org/)
  * Установка Python должна включать [pip](https://pip.pypa.io/en/stable/). Чтобы проверить, установлен ли pip, выполните команду `pip --version` в командной строке. Чтобы использовать pip, установите последнюю версию Python.
* Получив подписку Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Создание ресурса Компьютерного зрения"  target="_blank">создайте ресурс Компьютерного зрения</a> на портале Azure, чтобы получить ключ и конечную точку. После развертывания щелкните **Перейти к ресурсам**.

    * Для подключения приложения к API Компьютерного зрения потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

Клиентскую библиотеку можно установить с помощью следующей команды:

```console
pip install --upgrade azure-cognitiveservices-vision-computervision
```

Также установите библиотеку Pillow.

```console
pip install pillow
```

### <a name="create-a-new-python-application"></a>Создание приложения Python

> [!TIP]
> Хотите просмотреть готовый файл с кодом для этого краткого руководства? Его можно найти [на сайте GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/ComputerVision/ComputerVisionQuickstart.py), где размещены примеры кода для этого краткого руководства.

Создайте файл Python, например *quickstart-file.py*. Затем откройте его в предпочитаемом редакторе или интегрированной среде разработки.

### <a name="find-the-subscription-key-and-endpoint"></a>Поиск ключа подписки и конечной точки

Перейдите на портал Azure. Если ресурс Компьютерного зрения, созданный с учетом **предварительных требований**, успешно развернут, нажмите кнопку **Перейти к ресурсу** в разделе **Дальнейшие действия**. Ключ подписки и конечную точку можно найти на странице **Ключи и конечная точка** ресурса в разделе **Управление ресурсами**. 

Создайте переменные для конечной точки и ключа подписки на службу "Компьютерное зрение". Вставьте ключ подписки и конечную точку в следующий код, где указано. Конечная точка службы "Компьютерное зрение" имеет такой формат: `https://<your_computer_vision_resource_name>.cognitiveservices.azure.com/`.

[!code-python[](~/cognitive-services-quickstart-code/python/ComputerVision/ComputerVisionQuickstart.py?name=snippet_imports_and_vars)]

> [!IMPORTANT]
> Не забудьте удалить ключ подписки из кода, когда закончите, и никогда не публикуйте его в открытом доступе. Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [хранилище ключей Azure](../../../../key-vault/general/overview.md).

> [!div class="nextstepaction"]
> [Мной настроен клиент](?success=set-up-client#object-model) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Python&Section=set-up-client)

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывают некоторые основные функции пакета SDK OCR для Python.

|Имя|Описание|
|---|---|
|[ComputerVisionClientOperationsMixin](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision.operations.computervisionclientoperationsmixin)| Этот класс напрямую обрабатывает все операции с изображениями, такие как анализ изображений, обнаружение текста и создание эскизов.|
| [ComputerVisionClient](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision.computervisionclient) | Этот класс требуется для всех функций Компьютерного зрения. Вы создаете его экземпляр с информацией о подписке и используете его для создания экземпляров других классов. Он реализует **ComputerVisionClientOperationsMixin**.|
|[VisualFeatureTypes](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision.models.visualfeaturetypes)| Это перечисление определяет различные типы анализа изображений, которые можно выполнить в стандартной операции анализа. Набор значений **VisualFeatureTypes** указывается в зависимости от потребностей. |

## <a name="code-examples"></a>Примеры кода

В следующих фрагментах кода показано выполнение следующих действий с помощью клиентской библиотеки OCR для Python:

* [аутентификация клиента](#authenticate-the-client);
* [Чтение печатного и рукописного текста](#read-printed-and-handwritten-text).

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр клиента с конечной точкой и ключом. Создайте объект [CognitiveServicesCredentials](/python/api/msrest/msrest.authentication.cognitiveservicescredentials) с помощью своего ключа и используйте его с вашей конечной точкой для создания объекта [ComputerVisionClient](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision.computervisionclient).

[!code-python[](~/cognitive-services-quickstart-code/python/ComputerVision/ComputerVisionQuickstart.py?name=snippet_client)]

> [!div class="nextstepaction"]
> [Мной аутентифицирован клиент](?success=authenticate-client#read-printed-and-handwritten-text) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Python&Section=authenticate-client)

## <a name="read-printed-and-handwritten-text"></a>Чтение печатного и рукописного текста

Служба OCR может считывать видимый текст в образе и преобразовывать его в поток символов. Это необходимо выполнять двумя частями.

### <a name="call-the-read-api"></a>Вызов API чтения

Сначала выполните следующий код, чтобы вызвать метод **read** для предоставленного изображения. Он возвращает идентификатор операции и запускает асинхронный процесс чтения содержимого образа.

[!code-python[](~/cognitive-services-quickstart-code/python/ComputerVision/ComputerVisionQuickstart.py?name=snippet_read_call)]

> [!TIP]
> Кроме того, вы можете прочитать текст из локального образа. Изучите информацию о методах класса [ComputerVisionClientOperationsMixin](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision.operations.computervisionclientoperationsmixin), например **read_in_stream**. Либо просмотрите пример кода на [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/ComputerVision/ComputerVisionQuickstart.py) для сценариев, включающих использование локальных изображений.

### <a name="get-read-results"></a>Получение результатов чтения

Затем получите идентификатор операции, возвращенный из вызова **read**, и получите от службы результаты операции по этому идентификатору. Следующий код проверяет операцию с интервалами в одну секунду, пока не будут возвращены результаты. После этого извлеченные текстовые данные выводятся на консоль.

[!code-python[](~/cognitive-services-quickstart-code/python/ComputerVision/ComputerVisionQuickstart.py?name=snippet_read_response)]

> [!div class="nextstepaction"]
> [Мной считан текст](?success=read-printed-handwritten-text#run-the-application) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Python&Section=read-printed-handwritten-text)

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение, выполнив команду `python` для файла quickstart.

```console
python quickstart-file.py
```

> [!div class="nextstepaction"]
> [Мной запущено приложение](?success=run-the-application#clean-up-resources) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Python&Section=run-the-application)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку Cognitive Services, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней.

* [Портал](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

> [!div class="nextstepaction"]
> [Мной очищены ресурсы](?success=clean-up-resources#next-steps) [Возникла проблема](https://microsoft.qualtrics.com/jfe/form/SV_0Cl5zkG3CnDjq6O?PLanguage=Python&Section=clean-up-resources)

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как решать базовые задачи с помощью библиотеки OCR для Python. Далее ознакомьтесь со справочной документацией, чтобы узнать больше о библиотеке.

> [!div class="nextstepaction"]
>[Справочник по API OCR (Python)](/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision)

* [Общие сведения об OCR](../../overview-ocr.md)
* Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/python/ComputerVision/ComputerVisionQuickstart.py).

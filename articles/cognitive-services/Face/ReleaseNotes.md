---
title: Новые возможности службы "Распознавание лиц"
titleSuffix: Azure Cognitive Services
description: Заметки о выпуске для службы распознавания лиц содержат историю изменений для разных версий.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: overview
ms.date: 03/30/2021
ms.author: pafarley
ms.custom: contperf-fy21q3
ms.openlocfilehash: c580828d29e92ecef7ecc73b8f3e5843c3ecd23d
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106078886"
---
# <a name="whats-new-in-face-service"></a>Новые возможности службы "Распознавание лиц"

Служба распознавания лиц Azure обновляется регулярно. В этой статье содержатся сведения о дополнительных возможностях, исправлениях проблем и обновлениях документации.

## <a name="february-2021"></a>Февраль 2021 года

### <a name="new-face-api-detection-model"></a>Новая модель обнаружения API распознавания лиц
* Новая модель обнаружения 03 — это наиболее точная модель обнаружения, доступная в настоящее время. Если вы являетесь новым клиентом, рекомендуем использовать именно эту модель. Обнаружение 03 улучшает как уровень полноты, так и точность для мелких лиц, найденных в изображениях (64 x 64 пикселей). Среди дополнительных возможностей следует отметить общее снижение числа ложноположительных результатов и более качественное обнаружение на повернутых ориентациях лиц. Сочетание модели обнаружения 03 с новой моделью распознавания 04 также обеспечит более высокую точность распознавания. Дополнительные сведения см. в разделе [Указание модели обнаружения лиц](./face-api-how-to-topics/specify-detection-model.md).
### <a name="new-detectable-face-attributes"></a>Новые обнаруживаемые атрибуты лиц
* Атрибут `faceMask` доступен в последней модели 03 вместе с дополнительным атрибутом `"noseAndMouthCovered"`, который определяет, надета ли маска лица должным образом, то есть закрывает ли она как нос, так и рот. Чтобы использовать новую функцию обнаружения маски, пользователям необходимо указать модель обнаружения в запросе API: назначить версию модели с параметром _detectionModel_, для которого задано значение `detection_03`. Дополнительные сведения см. в разделе [Указание модели обнаружения лиц](./face-api-how-to-topics/specify-detection-model.md).
### <a name="new-face-api-recognition-model"></a>Новая модель API распознавания лиц
* Новая модель распознавания 04 — это самая точная модель распознавания, доступная в настоящее время. Если вы являетесь новым клиентом, рекомендуем использовать именно эту модель для проверки и идентификации. Она повышает точность распознавания модели распознавания 03, включая улучшенное распознавание для зарегистрированных пользователей с масками на лице (хирургические маски, маски N95, тканевые маски). Теперь клиенты могут создавать надежные и удобные возможности работы для пользователей, позволяющие определить, носит ли зарегистрированный пользователь маску на лице, с помощью последней модели обнаружения 03, а также распознавать лица, используя новую модель распознавания 04. Дополнительные сведения см. в разделе [Указание модели распознавания лиц](./face-api-how-to-topics/specify-recognition-model.md).


## <a name="january-2021"></a>Январь 2021 г.
### <a name="mitigate-latency"></a>Уменьшение задержки
* Команда Face опубликовала новую статью, где подробно описываются возможные причины задержки при использовании службы и возможные способы уменьшения ее уровня. См. статью [Уменьшение задержки при использовании службы распознавания лиц](./face-api-how-to-topics/how-to-mitigate-latency.md).

## <a name="december-2020"></a>Декабрь 2020 г.
### <a name="customer-configuration-for-face-id-storage"></a>Настройка хранилища ИД лиц на стороне клиента
* Несмотря на то, что служба распознавания лиц не сохраняет изображения клиентов, извлеченные характерные черты лиц будут храниться на сервере. ИД лица — это идентификатор характерной черты лица, который будет использоваться в методах [Face — Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239), [Face — Verify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) и [Face — Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237). Срок хранения характерных черт лиц истекает через 24 часа после исходного запроса на обнаружение, после чего они будут удалены. Теперь клиенты могут определить период времени, в течение которого такие ИД лиц будут находиться в кэше. Максимальное значение по-прежнему равно 24 часам, но теперь можно установить минимальное значение в 60 секунд. Новые диапазоны времени для ИД лиц, которые помещаются в кэш, допускают значения от 60 секунд до 24 часов. Дополнительные сведения можно найти в справочнике по API [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) (параметр *faceIdTimeToLive*).

## <a name="november-2020"></a>Ноябрь 2020 г.
### <a name="sample-face-enrollment-app"></a>Пример приложения регистрации лиц
* Команда опубликовала пример приложения регистрации лиц, чтобы продемонстрировать рекомендации по определению обоснованно запрашиваемого согласия и созданию систем высокоточного распознавания лиц благодаря высококачественной регистрации. Пример с открытым исходным кодом можно найти в руководстве [Создание приложения регистрации](build-enrollment-app.md) и в [GitHub](https://github.com/Azure-Samples/cognitive-services-FaceAPIEnrollmentSample), который разработчики могут использовать при развертывании или настройке. 

## <a name="august-2020"></a>Август 2020 г.
### <a name="customer-managed-encryption-of-data-at-rest"></a>Управляемое клиентом шифрование неактивных данных
* Служба распознавания лиц автоматически шифрует данные перед их сохранением в облаке. Такое шифрование защищает данные и помогает соблюдать корпоративные обязательства по обеспечению безопасности и соответствия требованиям. По умолчанию в подписке используются ключи шифрования, управляемые корпорацией Майкрософт. Подпиской также можно управлять с помощью собственных ключей, которые называются управляемыми клиентом ключами (CMK). Дополнительные сведения см. в статье [Ключи, управляемые клиентом](./encrypt-data-at-rest.md).

## <a name="april-2020"></a>Апрель 2020 г.
### <a name="new-face-api-recognition-model"></a>Новая модель API распознавания лиц
* Новая модель распознавания 03 — это самая точная модель, доступная в настоящее время. Если вы являетесь новым клиентом, рекомендуем использовать именно эту модель. Модель распознавания 03 обеспечит более высокую точность при определении как сходства, так совпадающих черт на лицах людей. Дополнительные сведения см. в статье [Указание модели распознавания лиц](./face-api-how-to-topics/specify-recognition-model.md).

## <a name="june-2019"></a>Июнь 2019 г.

### <a name="new-face-api-detection-model"></a>Новая модель обнаружения API распознавания лиц
* Новая модель обнаружения 02 обеспечивает более высокую точность для изображений с небольшими лицами, лицами при виде сбоку, частично перекрытыми и размытыми лицами. Для использования этой модели в методах [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250), [LargeFaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3), [PersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) и [LargePersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42) нужно указать имя новой модели обнаружения лиц `detection_02` в параметре `detectionModel`. Дополнительные сведения см. в статье [Указание модели обнаружения](Face-API-How-to-Topics/specify-detection-model.md).

## <a name="april-2019"></a>Апрель 2019 г.

### <a name="improved-attribute-accuracy"></a>Повышенная точность атрибутов
* Повышен уровень общей точности атрибутов `age` и `headPose`. Атрибут `headPose` также обновлен и поддерживает значение `pitch`. Эти атрибуты можно указать в параметре `returnFaceAttributes` для параметра `returnFaceAttributes` в методе [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236). 
### <a name="improved-processing-speeds"></a>Повышенная скорость обработки
* Увеличена скорость выполнения операций для методов [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250), [LargeFaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3), [PersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) и [LargePersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42).

## <a name="march-2019"></a>Март 2019 г.

### <a name="new-face-api-recognition-model"></a>Новая модель API распознавания лиц
* Модель распознавания 02 имеет более высокую точность. Используйте ее с помощью методов [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b), [LargeFaceList — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc), [PersonGroup — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) и [LargePersonGroup — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d). Для этого укажите имя новой модели распознавания лиц `recognition_02` в параметре `recognitionModel`. Дополнительные сведения см. в статье [Указание модели распознавания](Face-API-How-to-Topics/specify-recognition-model.md).

## <a name="january-2019"></a>Январь 2019 г.

### <a name="face-snapshot-feature"></a>Функция Face Snapshot
* Эта функция позволяет службе поддерживать перенос данных между подписками: [Snapshot](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/snapshot-get). Дополнительные сведения см. в статье [Перенос данных распознавания лиц в другую подписку распознавания лиц](Face-API-How-to-Topics/how-to-migrate-face-data.md).

## <a name="october-2018"></a>Октябрь 2018 г.

### <a name="api-messages"></a>Сообщения API
* Четкое описание для `status`, `createdDateTime`, `lastActionDateTime` и `lastSuccessfulTrainingDateTime` см. в [PersonGroup — Get Training Status](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395247) (PersonGroup — получение состояния обучения), [LargePersonGroup — Get Training Status](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae32c6ac60f11b48b5aa5) (LargePersonGroup — получение состояния обучения) и [LargeFaceList — Get Training Status](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a1582f8d2de3616c086f2cf) (LargeFaceList — получение состояния обучения).

## <a name="may-2018"></a>Май 2018 г.

### <a name="improved-attribute-accuracy"></a>Повышенная точность атрибутов
* Значительно улучшено поведение атрибута `gender`, а также улучшены атрибуты `age`, `glasses`, `facialHair`, `hair` и `makeup`. Вы можете использовать их в параметре `returnFaceAttributes` метода [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).
### <a name="increased-file-size-limit"></a>Увеличен предельный размер файла
* Предельный размер файла изображения увеличен с 4 МБ до 6 МБ для методов [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250), [LargeFaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3), [PersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) и [LargePersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42).

## <a name="march-2018"></a>Март 2018 г.

### <a name="new-data-structure"></a>Новая структура данных
* [LargeFaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc) и [LargePersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d). Дополнительные сведения см. в статье [How to use the large-scale feature](Face-API-How-to-Topics/how-to-use-large-scale.md) (Использование функции для увеличения масштаба).
* Для параметра `maxNumOfCandidatesReturned` в методе [Face — Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) расширен диапазон с [1, 5] до [1, 100], а значение по умолчанию — 10.

## <a name="may-2017"></a>Май 2017 г.

### <a name="new-detectable-face-attributes"></a>Новые обнаруживаемые атрибуты лиц
* Добавлены атрибуты `hair`, `makeup`, `accessory`, `occlusion`, `blur`, `exposure`, и `noise` в параметре `returnFaceAttributes` метода [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).
* Для PersonGroup в методе [Face — Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) поддерживаются до 10 000 пользователей.
* Добавлено разбиение на страницы в методе [PersonGroup Person — List](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395241) с использованием необязательных параметров `start` и `top`.
* Добавлена поддержка параллелизма при добавлении и удалении лиц для разных FaceLists и разных людей в PersonGroup.

## <a name="march-2017"></a>Март 2017 г.

### <a name="new-detectable-face-attribute"></a>Новый обнаруживаемый атрибут лиц
* Добавлен атрибут `emotion` в параметре `returnFaceAttributes` метода [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).
### <a name="fixed-issues"></a>Устраненные проблемы
* Не удавалось повторно обнаружить лицо с наложенным прямоугольником, который возвращается методом [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) в качестве `targetFace` в [FaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) и [PersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b).
* Размер обнаруживаемого лица находится строго в диапазоне от 36×36 до 4096×4096 пикселей.

## <a name="november-2016"></a>Ноябрь 2016 г.
### <a name="new-subscription-tier"></a>Новый уровень подписки
* Добавлена подписка категории "Стандартный" для Хранилища изображений лиц, которая позволяет сохранять между сеансами дополнительные изображения лиц и использовать их в [PersonGroup Person — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) или [FaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) для идентификации людей или поиска похожих. За хранение изображений взимается плата: 0,5 долл. США за 1000 изображений лиц. Плата распределяется пропорционально за каждый день. Для подписки категории "Бесплатный" сохраняется ограничение на общее количество людей — до 1000 человек.

## <a name="october-2016"></a>Октябрь 2016 г.
### <a name="api-messages"></a>Сообщения API
* Изменено сообщение об ошибке, возникающее при передаче сведений о нескольких лицах в метод `targetFace`. Вместо "There are more than one face in the image" теперь возвращается строка "There is more than one face in the image" в методах [FaceList — Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) и [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b).

## <a name="july-2016"></a>Июль 2016 г.
### <a name="new-features"></a>новые функции;
* Добавлена поддержка аутентификации путем сравнения идентификатора лица с идентификатором объекта Person в методе [Face — Verify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a).
* Добавлен необязательный параметр `mode`, который позволяет выбрать один из двух режимов работы (`matchPerson` или `matchFace`) в методе [Face — Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237). По умолчанию используется значение `matchPerson`.
* Добавлен необязательный параметр `confidenceThreshold`, с помощью которого пользователь может задать порог привязки лица к объекту Person в методе [Face — Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).
* Добавлены необязательные параметры `start` и `top` в методе [PersonGroup — List](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248), с помощью которых пользователь может указать начальную точку и общее число отображаемых групп PersonGroup.

## <a name="v10-changes-from-v0"></a>Отличия версии 1.0 от версии 0

* Корневая конечная точка службы изменена с ```https://westus.api.cognitive.microsoft.com/face/v0/``` на ```https://westus.api.cognitive.microsoft.com/face/v1.0/```. Внесены изменения в методы [Face — Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [Face — Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239), [Face — Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) и [Face — Group](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238).
* Минимальный размер обнаруживаемого лица изменен на 36×36 пикселей. Теперь лица размером менее 36×36 не обнаруживаются.
* Данные PersonGroup и Person в интерфейсе распознавания лиц версии 0 объявлены нерекомендуемыми. Эти данные недоступны в службе распознавания лиц версии 1.0.
* Конечная точка API распознавания лиц версии 0 объявлена нерекомендуемой с 30 июня 2016 г.
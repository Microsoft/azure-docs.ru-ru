---
title: Все предварительно созданные сущности — LUIS
titleSuffix: Azure Cognitive Services
description: В этой статье содержатся списки готовых сущностей, включенных в службу "Распознавание речи" (LUIS).
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 07/20/2020
ms.openlocfilehash: cb3c74a2176ee7fcac53afb5185e8c62e66f4dfb
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104798805"
---
# <a name="entities-per-culture-in-your-luis-model"></a>Сущности для языков и региональных параметров в модели LUIS

Служба "Распознавание речи" (LUIS) включает готовые сущности.

## <a name="entity-resolution"></a>Разрешение сущностей
Когда в приложение включается предварительно созданная сущность, LUIS включает соответствующее разрешение сущности в ответе на конечную точку. Все примеры фраз также помечаются с помощью сущности.

Поведение предварительно созданных сущностей нельзя изменить, но можно улучшить разрешение, [добавив предварительно созданную сущность в качестве функции в сущность машинного обучения или подсущность](luis-concept-entity-types.md#effective-prebuilt-entities).

## <a name="availability"></a>Доступность
Если не указано иное, предварительно созданные сущности доступны во всех языковых стандартах (языках и региональных параметрах) приложений LUIS. В следующей таблице показаны предварительно созданные сущности, которые поддерживаются для каждого языка и региональных параметров.

|culture|Подкатегории языков и региональных параметров|Примечания|
|--|--|--|
|Китайский|[zh-CN](#chinese-entity-support)||
|Нидерландский|[nl-NL](#dutch-entity-support)||
|Английский|[en-US (США)](#english-american-entity-support)||
|Французский|[fr-CA (Канада)](#french-canadian-entity-support), [fr-FR (Франция)](#french-france-entity-support), ||
|Немецкий|[de-DE](#german-entity-support)||
|Итальянский|[it-IT](#italian-entity-support)||
|Японский|[ja-JP](#japanese-entity-support)||
|Корейский|[ko-KR](#korean-entity-support)||
|Португальский|[pt-BR (Бразилия)](#portuguese-brazil-entity-support)||
|Испанский|[es-ES (Испания)](#spanish-spain-entity-support), [es-MX (Мексика)](#spanish-mexico-entity-support)||
|Турецкий|[турецкий](#turkish-entity-support)||

## <a name="prediction-endpoint-runtime"></a>Среда выполнения прогнозирующих точек

Доступность предварительно созданной сущности на определенном языке определяется версией среды выполнения прогнозной конечной точки.

## <a name="chinese-entity-support"></a>Поддержка сущностей для китайского языка

Поддерживаются следующие сущности:

| Предварительно созданная сущность | zh-CN |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    -   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    V2, V3   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="dutch-entity-support"></a>Поддержка сущностей для голландского языка

Поддерживаются следующие сущности:

| Предварительно созданная сущность | nl-NL |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DateTime](luis-reference-prebuilt-deprecated.md)   |    -   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="english-american-entity-support"></a>Поддержка сущностей для английского языка (США)

Поддерживаются следующие сущности:

| Предварительно созданная сущность | en-US |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    V2, V3   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    V2, V3   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    V2, V3   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="french-france-entity-support"></a>Поддержка сущностей для французского языка (Франция)

Поддерживаются следующие сущности:

| Предварительно созданная сущность | fr-FR |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |   -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="french-canadian-entity-support"></a>Поддержка сущностей для французского языка (Канада)

Поддерживаются следующие сущности:

| Предварительно созданная сущность | fr-CA |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="german-entity-support"></a>Поддержка сущностей для немецкого языка

Поддерживаются следующие сущности:

|Предварительно созданная сущность | de-DE |
| -------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="italian-entity-support"></a>Поддержка сущностей для итальянского языка

Итальянский предварительно сформированный возраст, валюта, размер, число, _разрешение_ в процентах изменились с версии v2 и V3 Preview.

Поддерживаются следующие сущности:

| Предварительно созданная сущность | it-IT |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DateTime](luis-reference-prebuilt-deprecated.md)   |    -   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="japanese-entity-support"></a>Поддержка сущностей для японского языка

Поддерживаются следующие сущности:

|Предварительно созданная сущность | ja-JP |
| -------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2,-   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2,-   |
[DateTime](luis-reference-prebuilt-deprecated.md)   |    -   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2,-   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2,-   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2,-   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2,-   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2,-   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="korean-entity-support"></a>Поддержка сущностей для корейского языка

Поддерживаются следующие сущности:

| Предварительно созданная сущность | ko-KR |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    -   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    -   |
[DateTime](luis-reference-prebuilt-deprecated.md)   |    -   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    -   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    -   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    -   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    -   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    -   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="portuguese-brazil-entity-support"></a>Поддержка сущностей для португальского языка (Бразилия)

Поддерживаются следующие сущности:

| Предварительно созданная сущность | pt-BR |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

KeyPhrase недоступна во всех подкатегориях португальского языка и региональных параметров (Бразилия) — ```pt-BR```.

## <a name="spanish-spain-entity-support"></a>Поддержка сущностей для испанского языка (Испания)

Поддерживаются следующие сущности:

| Предварительно созданная сущность | es-ES |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    V2, V3   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    V2, V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    V2, V3   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    V2, V3   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    V2, V3   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    V2, V3   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    V2, V3   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

## <a name="spanish-mexico-entity-support"></a>Поддержка сущностей для испанского языка (Мексика)

Поддерживаются следующие сущности:

| Предварительно созданная сущность | es-MX |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    -   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    -   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    -   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    -   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    V2, V3   |
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    V2, V3   |
[Число](luis-reference-prebuilt-number.md)   |    V2, V3   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    -   |
[OrdinalV2](luis-reference-prebuilt-ordinal-v2.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    -   |
[персоннаме](luis-reference-prebuilt-person.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    V2, V3   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    -   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    V2, V3   |

См. примечания об [устаревших предварительно созданных сущностях](luis-reference-prebuilt-deprecated.md).

## <a name="turkish-entity-support"></a>Поддержка сущностей для турецкого языка

| Предварительно созданная сущность | tr-tr |
| --------------- | :---: |
[Возраст](luis-reference-prebuilt-age.md):<br>year<br>month<br>week<br>day   |    -   |
[Currency (денежная единица)](luis-reference-prebuilt-currency.md):<br>доллар<br>дробная единица (например, пенни)  |    -   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md):<br>Дата<br>daterange<br>time<br>timerange   |    -   |
[Измерение](luis-reference-prebuilt-dimension.md):<br>том<br>площадь<br>вес<br>информация (например, бит/байт)<br>длина (например, метр)<br>скорость (например, миль в час)  |    -   |
[Электронная почта](luis-reference-prebuilt-email.md)   |    -   |
[кэйфрасе](luis-reference-prebuilt-keyphrase.md)   |    -   |
[Число](luis-reference-prebuilt-number.md)   |    -   |
[Порядковый номер](luis-reference-prebuilt-ordinal.md)   |    -   |
[Percentage](luis-reference-prebuilt-percentage.md)   |    -   |
[PhoneNumber](luis-reference-prebuilt-phonenumber.md)   |    -   |
[Температура](luis-reference-prebuilt-temperature.md):<br>Фаренгейта<br>Кельвина<br>Ранкина<br>Делиля<br>Цельсия   |    -   |
[URL-адрес](luis-reference-prebuilt-url.md)   |    -   |

<!---
See notes on [Deprecated prebuilt entities](luis-reference-prebuilt-deprecated.md)
KeyPhrase is not available.
-->

## <a name="contribute-to-prebuilt-entity-cultures"></a>Участие в создании предварительно созданных сущностей для языков и региональных параметров
Предварительно созданные сущности разрабатываются в проекте с открытым кодом Recognizers-Text. Приглашаем к [участию](https://github.com/Microsoft/Recognizers-Text) в проекте. Этот проект содержит примеры единиц валюты для различных языков и региональных параметров.

GeographyV2 и PersonName не включаются в проект Recognizers-Text. Чтобы устранить проблемы с этими предварительно созданными сущностями, отправьте [запрос в службу поддержки](../../azure-portal/supportability/how-to-create-azure-support-request.md).

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения о сущностях [number](luis-reference-prebuilt-number.md), [datetimeV2](luis-reference-prebuilt-datetimev2.md) и [currency](luis-reference-prebuilt-currency.md).

---
title: Нерекомендуемые профессиональные навыки
titleSuffix: Azure Cognitive Search
description: На этой странице приведен список признанных навыков, которые считаются устаревшими и не будут поддерживаться в ближайшем будущем в Azure Когнитивный поиск навыков.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 85f3b9862bd8155c1a4b11860dc82d92a2f9e810
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88936101"
---
# <a name="deprecated-cognitive-skills-in-azure-cognitive-search"></a>Нерекомендуемые профессиональные навыки в Azure Когнитивный поиск

В этом документе описываются когнитивные навыки, которые считаются устаревшими. Ознакомьтесь со следующими понятиями:

* Имя навыка. Имя навыка, который станет устаревшим. Оно сопоставляется с атрибутом @odata.type.
* Последняя доступная версия API. Последняя версия общедоступного API Когнитивный поиск Azure, с помощью которой можно создать или обновить навыков, содержащий соответствующий устаревший навык.
* Прекращение поддержки. Последний день, после которого соответствующий навык не будет поддерживаться. Ранее созданные наборы навыков по-прежнему будут работать, но пользователям рекомендуется прекратить их использование.
* Рекомендации. Способ перехода для использования поддерживаемого навыка. Пользователям следует придерживаться этих рекомендаций, чтобы продолжать получать поддержку.

## <a name="microsoftskillstextnamedentityrecognitionskill"></a>Microsoft.Skills.Text.NamedEntityRecognitionSkill

### <a name="last-available-api-version"></a>Последняя доступная версия API

2017-11-11-Preview

### <a name="end-of-support"></a>Дата окончания поддержки

15 февраля 2019 г.

### <a name="recommendations"></a>Рекомендации 

Теперь следует использовать тип [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md). Он предоставляет большинство функций NamedEntityRecognitionSkill более высокого качества. Он также содержит более полные сведения в составных полях выходных данных.

Для перехода на [навык распознавания сущностей](cognitive-search-skill-entity-recognition.md) необходимо внести одно или несколько следующих изменений в определение навыка. Вы можете обновить определение навыков с помощью [API обновления набора навыков](/rest/api/searchservice/update-skillset).

> [!NOTE]
> Сейчас оценка достоверности не поддерживается в качестве концепции. Параметр `minimumPrecision` содержится в `EntityRecognitionSkill` для использования в будущем, а также для обеспечения обратной совместимости.

1. *(Обязательно.)* Измените `@odata.type` с `"#Microsoft.Skills.Text.NamedEntityRecognitionSkill"` на `"#Microsoft.Skills.Text.EntityRecognitionSkill"`.

2. *(Необязательно.)* Если вы используете выходные данные `entities`, лучше укажите сложные выходные данные коллекции `namedEntities` из `EntityRecognitionSkill`. В определении навыка можно задать `targetName`, чтобы сопоставить его с заметкой с именем `entities`.

3. *(Необязательно.)* Если вы явно не укажете `categories`, `EntityRecognitionSkill` может возвратить другой тип категорий, кроме тех, которые поддерживались `NamedEntityRecognitionSkill`. Если такое поведение нежелательно, явно задайте для параметра `categories` значение `["Person", "Location", "Organization"]`.

    _Примеры изменения определений_

    * Простое изменение

        _Раньше — определение навыка NamedEntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
            "categories": [ "Person"],
            "defaultLanguageCode": "en",
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            }
            ]
        }
        ```
        _Теперь — определение навыка EntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": [ "Person"],
            "defaultLanguageCode": "en",
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            }
            ]
        }
        ```
    
    * Немного сложное изменение

        _Раньше — определение навыка NamedEntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
            "defaultLanguageCode": "en",
            "minimumPrecision": 0.1,
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            },
            {
                "name": "entities"
            }
            ]
        }
        ```
        _Теперь — определение навыка EntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": [ "Person", "Location", "Organization" ],
            "defaultLanguageCode": "en",
            "minimumPrecision": 0.1,
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            },
            {
                "name": "namedEntities",
                "targetName": "entities"
            }
            ]
        }
        ```

## <a name="see-also"></a>См. также раздел

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [Навык распознавания сущностей](cognitive-search-skill-entity-recognition.md)
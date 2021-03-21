---
title: Условный навык для переприятия
titleSuffix: Azure Cognitive Search
description: Условный навык в Azure Когнитивный поиск включает фильтрацию, создание значений по умолчанию и слияние в определении набора навыков.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: f47ca56fa1b40422edeb0d4e11c24be6f60e49e5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101666368"
---
# <a name="conditional-cognitive-skill"></a>Условный навык для переприятия

**Условный** навык позволяет использовать сценарии когнитивный Поиск Azure, для которых требуется логическая операция, чтобы определить данные для назначения выходу. Эти сценарии включают фильтрацию, назначение значения по умолчанию и слияние данных на основе условия.

В следующем псевдокоде показано, что делает условный навык:

```
if (condition) 
    { output = whenTrue } 
else 
    { output = whenFalse } 
```

> [!NOTE]
> Этот навык не привязан к Azure Cognitive Services API, и вы не платите за его использование. Однако вы по-прежнему должны [присоединить ресурс Cognitive Services](cognitive-search-attach-cognitive-services.md) , чтобы переопределить параметр "Free" (бесплатный), ограничивающий небольшое количество дополнений в день.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ConditionalSkill


## <a name="evaluated-fields"></a>Вычисляемые поля

Этот навык является специальным, так как его входные данные являются вычисляемыми полями.

Следующие элементы являются допустимыми значениями выражения:

-   Пути к заметкам (пути в выражениях должны быть разделены символами "$ (" и ")")
 <br/>
    Примеры:
    ```
        "= $(/document)"
        "= $(/document/content)"
    ```

-  Литералы (строки, числа, true, false, null) <br/>
    Примеры:
    ```
       "= 'this is a string'"   // string (note the single quotation marks)
       "= 34"                   // number
       "= true"                 // Boolean
       "= null"                 // null value
    ```

-  Выражения, использующие операторы сравнения (= =,! =, >=, >, <=, <) <br/>
    Примеры:
    ```
        "= $(/document/language) == 'en'"
        "= $(/document/sentiment) >= 0.5"
    ```

-   Выражения, использующие логические операторы (&&, | |,!, ^) <br/>
    Примеры:
    ```
        "= $(/document/language) == 'en' && $(/document/sentiment) > 0.5"
        "= !true"
    ```

-   Выражения, использующие числовые операторы (+,-, \* ,/,%) <br/>
    Примеры: 
    ```
        "= $(/document/sentiment) + 0.5"         // addition
        "= $(/document/totalValue) * 1.10"       // multiplication
        "= $(/document/lengthInMeters) / 0.3049" // division
    ```

Поскольку условный навык поддерживает оценку, его можно использовать в сценариях незначительного преобразования. Например, см. [Определение навыка 4](#transformation-example).

## <a name="skill-inputs"></a>Входные данные навыков
Во входных данных учитывается регистр.

| Входные данные   | Описание |
|-------------|-------------|
| condition   | Это [вычисляемое поле](#evaluated-fields) , представляющее условие для оценки. Это условие должно возвращать логическое значение (*true* или *false*).   <br/>  Примеры: <br/> "= true" <br/> "= $ (/документ/лангуаже) = = ' fr '" <br/> "= $ (/документ/Пажес/ \* /Language) = = $ (/документ/експектедлангуаже)" <br/> |
| вхентруе    | Это [вычисляемое поле](#evaluated-fields) , представляющее возвращаемое значение, если условие оценивается как *истинное*. Константные строки должны возвращаться в одинарных кавычках ("и"). <br/>Примеры значений: <br/> "=" контракт ""<br/>"= $ (/документ/контракттипе)" <br/> "= $ (/документ/ентитиес/ \* )" <br/> |
| вхенфалсе   | Это [вычисляемое поле](#evaluated-fields) , представляющее возвращаемое значение, если условие оценивается как *false*. <br/>Примеры значений: <br/> "=" контракт ""<br/>"= $ (/документ/контракттипе)" <br/> "= $ (/документ/ентитиес/ \* )" <br/>

## <a name="skill-outputs"></a>Выходные данные навыка
Существует один выход, который просто называется Output. Он возвращает значение *вхенфалсе* , если условие имеет значение false, или *вхентруе* , если условие имеет значение true.

## <a name="examples"></a>Примеры

### <a name="sample-skill-definition-1-filter-documents-to-return-only-french-documents"></a>Пример определения навыка 1. фильтрация документов для возврата только документов на французском языке

Следующие выходные данные возвращают массив предложений ("/документ/френчсентенцес"), если язык документа является французским. Если язык не является французским, для него задается значение *null*.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == 'fr'" },
        { "name": "whenTrue", "source": "/document/sentences" },
        { "name": "whenFalse", "source": "= null" }
    ],
    "outputs": [ { "name": "output", "targetName": "frenchSentences" } ]
}
```
Если в качестве *контекста* другого навыка используется "/документ/френчсентенцес", этот навык выполняется только в том случае, если значение "/документ/френчсентенцес" не равно *null*.


### <a name="sample-skill-definition-2-set-a-default-value-for-a-value-that-doesnt-exist"></a>Пример определения навыка 2. Задание значения по умолчанию для несуществующего значения

Следующий результат создает аннотацию ("/документ/лангуажевисдефаулт"), которая задана для языка документа, или "ES", если язык не задан.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == null" },
        { "name": "whenTrue", "source": "= 'es'" },
        { "name": "whenFalse", "source": "= $(/document/language)" }
    ],
    "outputs": [ { "name": "output", "targetName": "languageWithDefault" } ]
}
```

### <a name="sample-skill-definition-3-merge-values-from-two-fields-into-one"></a>Пример определения навыка 3. объединение значений из двух полей в одно

В этом примере некоторые предложения имеют свойство *френчсентимент* . Если свойство *френчсентимент* имеет значение null, мы хотим использовать значение *енглишсентимент* . Мы присваиваем выходные данные члену, который называется *тональности* ("/документ/сентенцес/*/сентимент").

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= $(/document/sentences/*/frenchSentiment) == null" },
        { "name": "whenTrue", "source": "/document/sentences/*/englishSentiment" },
        { "name": "whenFalse", "source": "/document/sentences/*/frenchSentiment" }
    ],
    "outputs": [ { "name": "output", "targetName": "sentiment" } ]
}
```

## <a name="transformation-example"></a>Пример преобразования
### <a name="sample-skill-definition-4-data-transformation-on-a-single-field"></a>Пример определения навыка 4. Преобразование данных для одного поля

В этом примере мы получаем *тональности* от 0 до 1. Мы хотим преобразовать его в значение от-1 до 1. Для этого незначительного преобразования можно использовать условный навык.

В этом примере мы не используем условный аспект навыка, так как условие всегда имеет *значение true*.

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document/sentences/*",
    "inputs": [
        { "name": "condition", "source": "= true" },
        { "name": "whenTrue", "source": "= $(/document/sentences/*/sentiment) * 2 - 1" },
        { "name": "whenFalse", "source": "= 0" }
    ],
    "outputs": [ { "name": "output", "targetName": "normalizedSentiment" } ]
}
```

## <a name="special-considerations"></a>Специальные рекомендации
Некоторые параметры оцениваются, поэтому необходимо особо осторожно следовать документированному шаблону. Выражения должны начинаться со знака равенства. Путь должен быть отделен символами "$ (" и ")". Убедитесь, что строки заключены в одинарные кавычки. Это помогает оценщику различать строки и фактические пути и операторы. Кроме того, не забудьте поставить пробел вокруг операторов (например, "*" в пути означает нечто, отличное от умножения).


## <a name="next-steps"></a>Дальнейшие действия

+ [Встроенные навыки](cognitive-search-predefined-skills.md)
+ [Определение набора навыков](cognitive-search-defining-skillset.md)

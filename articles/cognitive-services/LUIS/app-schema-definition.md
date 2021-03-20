---
title: Определение схемы приложения
description: Приложение LUIS представляется в `.json` или, а также `.lu` включает все объекты, сущности, пример фразы продолжительностью, функции и параметры.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 08/22/2020
ms.openlocfilehash: db4fd52dad82542f20e58ebb3b8526c5be7f2f88
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91327329"
---
# <a name="app-schema-definition"></a>Определение схемы приложения

Приложение LUIS представляется в `.json` или, а также `.lu` включает все объекты, сущности, пример фразы продолжительностью, функции и параметры.

## <a name="format"></a>Формат

При импорте и экспорте приложения выберите либо `.json` `.lu` .

|Формат|Сведения|
|--|--|
|`.json`| Стандартный формат программирования|
|`.lu`|Поддерживается [инструментами Bot Builder](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md)платформы Bot.|

## <a name="version-7x"></a>Версия 7. x

* При переходе к версии 7. x сущности представляются как вложенные сущности машинного обучения.
* Поддержка создания вложенных сущностей машинного обучения с помощью `enableNestedChildren` свойства для следующих API-интерфейсов разработки:
    * [Добавление метки](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c08)
    * [Добавить метку пакета](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c09)
    * [Метки проверки](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c0a)
    * [Предложить запросы к конечной точке для сущностей](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2e)
    * [Предложить запросы к конечной точке для целей](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2d)

```json
{
  "luis_schema_version": "7.0.0",
  "intents": [
    {
      "name": "None",
      "features": []
    }
  ],
  "entities": [],
  "hierarchicals": [],
  "composites": [],
  "closedLists": [],
  "prebuiltEntities": [],
  "utterances": [],
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "patternAnyEntities": [],
  "regex_entities": [],
  "phraselists": [
  ],
  "regex_features": [],
  "patterns": [],
  "settings": []
}
```

| element                  | Комментировать                              |
|--------------------------|--------------------------------------|
| "иерархии": [],     | Не рекомендуется, использовать [сущности машинного обучения](luis-concept-entity-types.md).   |
| "Composite": [],        | Не рекомендуется, использовать [сущности машинного обучения](luis-concept-entity-types.md). Ссылка на [составной объект](reference-entity-composite.md) . |
| "Клоседлистс": [],       | [Список](reference-entity-list.md) ссылок на сущности в основном используется в качестве компонентов для сущностей.    |
| "versionId": "0,1",      | Версия приложения LUIS.|
| "Name": "example-App",   | Имя приложения LUIS. |
| "DESC": "",              | Необязательное описание приложения LUIS.  |
| "Culture": "en-US",      | [Язык](luis-language-support.md) приложения, который влияет на базовые функции, такие как предварительно подготовленные сущности, машинное обучение и лексический анализатор.  |
| "Токенизерверсион": "1.0.0", | [Разметчика](luis-language-support.md#tokenization)  |
| "Паттернанентитиес": [],   | [Сущность Pattern.any](reference-entity-pattern-any.md)    |
| "regex_entities": [],    |  [Сущность регулярного выражения](reference-entity-regular-expression.md)   |
| "фраселистс": [],       |  [Списки фраз (функция)](luis-concept-feature.md#create-a-phrase-list-for-a-concept)   |
| "regex_features": [],    |  Не рекомендуется, использовать [сущности машинного обучения](luis-concept-entity-types.md). |
| "Patterns": [],          |  [Шаблоны улучшают точность прогнозов](luis-concept-patterns.md) с помощью [синтаксиса шаблона](reference-pattern-syntax.md)   |
| "Параметры": []           | [Параметры приложения](luis-reference-application-settings.md)|

## <a name="version-6x"></a>Версия 6. x

* Переходя к версии 6. x, используйте новую [сущность машинного обучения](reference-entity-machine-learned-entity.md) для представления сущностей.

```json
{
  "luis_schema_version": "6.0.0",
  "intents": [
    {
      "name": "None",
      "features": []
    }
  ],
  "entities": [],
  "hierarchicals": [],
  "composites": [],
  "closedLists": [],
  "prebuiltEntities": [],
  "utterances": [],
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "patternAnyEntities": [],
  "regex_entities": [],
  "phraselists": [],
  "regex_features": [],
  "patterns": [],
  "settings": []
}
```

## <a name="version-4x"></a>Версия 4. x

```json
{
  "luis_schema_version": "4.0.0",
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "intents": [
    {
      "name": "None"
    }
  ],
  "entities": [],
  "composites": [],
  "closedLists": [],
  "patternAnyEntities": [],
  "regex_entities": [],
  "prebuiltEntities": [],
  "model_features": [],
  "regex_features": [],
  "patterns": [],
  "utterances": [],
  "settings": []
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Переход к [API-интерфейсам разработки версии 3](luis-migration-authoring-entities.md)
---
title: Пользовательский навык AML в навыков
titleSuffix: Azure Cognitive Search
description: Расширьте возможности Azure Когнитивный поиск навыков с помощью моделей Машинное обучение Azure.
manager: nitinme
author: mattmsft
ms.author: magottei
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: 6cefe543ea8ba992b028448070bf041a77bfec64
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97630281"
---
# <a name="aml-skill-in-an-azure-cognitive-search-enrichment-pipeline"></a>Навык AML в конвейере Когнитивный поиск обогащения Azure

> [!IMPORTANT] 
> Этот навык сейчас находится на этапе общедоступной предварительной версии. Для предварительной версии функции соглашение об уровне обслуживания не предусмотрено. Мы не рекомендуем использовать ее в рабочей среде. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Пакет SDK для .NET сейчас не поддерживается.

Навык **AML** позволяет расширить возможности искусственного интеллекта с помощью настраиваемой модели [машинное обучение Azure](../machine-learning/overview-what-is-azure-ml.md) (AML). После того как модель AML будет [обучена и развернута](../machine-learning/concept-azure-machine-learning-architecture.md#workspace), навык **AML** интегрирует его в обогащение искусственного интеллекта.

Как и в случае со встроенными навыками, навык **AML** имеет входные и выходные данные. Входные данные отправляются в развернутую службу AML в виде объекта JSON, который выводит полезные данные JSON в качестве ответа, а также код состояния успеха. Ожидается, что ответ будет иметь выходные данные, заданные навыком **AML** . Любой другой ответ считается ошибкой, и никакие обогащения не выполняются.

> [!NOTE]
> Индексатор повторит попытку дважды для определенных стандартных кодов состояния HTTP, возвращаемых службой AML. Это такие коды состояния HTTP:
> * `503 Service Unavailable`
> * `429 Too Many Requests`

## <a name="prerequisites"></a>Предварительные условия

* [Рабочая область AML](../machine-learning/concept-workspace.md)
* [Целевой объект вычислений AML службы Azure Kubernetes](../machine-learning/concept-compute-target.md) в этой рабочей области с [развернутой моделью](../machine-learning/how-to-deploy-azure-kubernetes-service.md)
  * В [целевом объекте вычислений должен быть включен протокол SSL](../machine-learning/how-to-secure-web-service.md#deploy-on-azure-kubernetes-service). Когнитивный поиск Azure разрешает доступ только к конечным точкам **HTTPS**
  * Самозаверяющие сертификаты использовать нельзя.

## <a name="odatatype"></a>@odata.type  
Microsoft. Skills. Custom. Амлскилл

## <a name="skill-parameters"></a>Параметры навыков

Параметры зависят от регистра. Параметры, которые вы выбираете для использования, зависят от того [, какая проверка подлинности требует служба AML, если таковая имеется](#WhatSkillParametersToUse) .

| Имя параметра | Описание |
|--------------------|-------------|
| `uri` | (Требуется для [без проверки подлинности или проверки подлинности ключа](#WhatSkillParametersToUse)) [URI оценки службы AML](../machine-learning/how-to-consume-web-service.md) , в которую будут отправлены полезные данные _JSON_ . Допускается только схема URI **HTTPS** . |
| `key` | (Требуется для [проверки подлинности с помощью ключа](#WhatSkillParametersToUse)) [Ключ для службы AML](../machine-learning/how-to-consume-web-service.md#authentication-with-keys). |
| `resourceId` | (Требуется для [проверки подлинности маркера](#WhatSkillParametersToUse)). Azure Resource Manager идентификатор ресурса службы AML. Он должен быть в формате "подписки/{GUID}/resourceGroups/{ресурс-группа-имя}/Microsoft. Мачинелеарнингсервицес/рабочие области/{Workspace-Name}/Services/{service_name}. |
| `region` | (Необязательно для [проверки подлинности маркера](#WhatSkillParametersToUse)). [Регион](https://azure.microsoft.com/global-infrastructure/regions/) , в котором развернута служба AML. |
| `timeout` | (Необязательно.) Если указано, означает время ожидания вызова API HTTP-клиента. Значение должно быть отформатировано как значение dayTimeDuration XSD (ограниченное подмножество значения [продолжительности ISO 8601](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration) ). Например, `PT60S` для 60 секунд. Если не задано, выбирается значение по умолчанию — 30 секунд. Время ожидания может быть не более 230 секунд и не менее 1 секунды. |
| `degreeOfParallelism` | Используемых Указывает число вызовов, которые индексатор будет выполнять параллельно с указанной конечной точкой. Это значение можно уменьшить, если конечная точка завершается неудачей при загрузке запроса или если ваша конечная точка может принимать больше запросов, и вы хотите увеличить производительность индексатора.  Если не задано, используется значение по умолчанию 5. Для параметра degreeOfParallelism можно задать не более 10 и не менее 1.

<a name="WhatSkillParametersToUse"></a>

## <a name="what-skill-parameters-to-use"></a>Какие параметры квалификации использовать

Требуемые параметры навыков AML зависят от того, какая проверка подлинности использует служба AML, если таковая имеется. Службы AML предоставляют три варианта проверки подлинности:

* [Проверка подлинности на основе ключей](../machine-learning/how-to-authenticate-web-service.md#key-based-authentication). Для проверки подлинности запросов оценки от навыков AML предоставляется статический ключ
  * Использование _URI_ и параметров _ключа_
* [Проверка подлинности на основе маркеров](../machine-learning/how-to-authenticate-web-service.md#token-based-authentication). Служба AML [развертывается с использованием проверки подлинности на основе маркеров](../machine-learning/how-to-authenticate-web-service.md#token-based-authentication). [Управляемому удостоверению](../active-directory/managed-identities-azure-resources/overview.md) службы когнитивный Поиск Azure предоставляется [роль читателя](../machine-learning/how-to-assign-roles.md) в рабочей области службы AML. Затем навык AML использует управляемое удостоверение службы Когнитивный поиск Azure для проверки подлинности в службе AML, не требующей статических ключей.
  * Используйте параметр _resourceId_ .
  * Если служба Когнитивный поиск Azure находится в другом регионе в рабочей области AML, используйте параметр _Region_ , чтобы задать регион, в котором развернута служба AML.
* Без поверки подлинности. Для использования службы AML не требуется проверка подлинности
  * Использование параметра _URI_

## <a name="skill-inputs"></a>Входные данные навыков

Предварительно заданные входные данные для этого навыка отсутствуют. Можно выбрать одно или несколько полей, которые уже будут доступны во время выполнения этого навыка в качестве входных данных, а полезные данные _JSON_ , отправляемые в службу AML, будут иметь разные поля.

## <a name="skill-outputs"></a>Выходные данные навыка

Предварительно заданные выходные данные для этого навыка отсутствуют. В зависимости от ответа, возвращаемого службой AML, добавьте поля выходных данных, чтобы их можно было получить из ответа _JSON_ .

## <a name="sample-definition"></a>Пример определения

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
    "description": "A sample model that detects the language of sentence",
    "uri": "https://contoso.count-things.com/score",
    "context": "/document",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "detected_language_code"
      }
    ]
  }
```

## <a name="sample-input-json-structure"></a>Пример структуры входных данных JSON

Эта структура _JSON_ представляет полезные данные, которые будут отправлены в службу AML. Поля верхнего уровня структуры будут соответствовать именам, указанным в `inputs` разделе определения навыка. Значения этих полей будут взяты из источника (`source`) этих полей (например, из поля в документе или, возможно, из другого навыка).

```json
{
  "text": "Este es un contrato en Inglés"
}
```

## <a name="sample-output-json-structure"></a>Пример структуры выходных данных JSON

Выходные данные соответствуют ответу, возвращенному службой AML. Служба AML должна возвращать только полезные данные _JSON_ (проверяются в `Content-Type` заголовке ответа) и должна быть объектом, где поля являются дополнением, соответствующим "именам" в `output` и, значение которых считается дополнением.

```json
{
    "detected_language_code": "es"
}
```

## <a name="inline-shaping-sample-definition"></a>Определение образца определения для встроенной формы

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
    "description": "A sample model that detects the language of sentence",
    "uri": "https://contoso.count-things.com/score",
    "context": "/document",
    "inputs": [
      {
        "name": "shapedText",
        "sourceContext": "/document",
        "inputs": [
            {
              "name": "content",
              "source": "/document/content"
            }
        ]
      }
    ],
    "outputs": [
      {
        "name": "detected_language_code"
      }
    ]
  }
```

## <a name="inline-shaping-input-json-structure"></a>Входная структура JSON для встроенной формы

```json
{
  "shapedText": { "content": "Este es un contrato en Inglés" }
}
```

## <a name="inline-shaping-sample-output-json-structure"></a>Структура выходных данных образца формирования кода JSON

```json
{
    "detected_language_code": "es"
}
```

## <a name="error-cases"></a>Варианты ошибок
В дополнение к тому, что AML недоступен или отправляет неуспешные коды состояния, ниже рассматриваются ошибочные случаи.

* Если служба AML возвращает код состояния Success, но ответ указывает, что это не так `application/json` , ответ считается недопустимым и никакие дополнения не будут выполняться.
* Если служба AML возвращает недопустимый код JSON

В случаях, когда служба AML недоступна или возвращает ошибку HTTP, в журнал выполнения индексатора будет добавлена понятная ошибка с любыми доступными сведениями об ошибке HTTP.

## <a name="see-also"></a>См. также раздел

+ [Определение набора навыков](cognitive-search-defining-skillset.md)
+ [Устранение неполадок службы AML](../machine-learning/how-to-troubleshoot-deployment.md)
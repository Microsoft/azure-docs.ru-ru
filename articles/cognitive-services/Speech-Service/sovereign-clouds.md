---
title: Облака независимых — служба речи
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать облака независимых
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.custom: references_regions
ms.date: 01/07/2021
ms.author: alexeyo
ms.openlocfilehash: f30b1f0f14bba54b8b4fcd7c5190f3c533f199a6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98061758"
---
# <a name="speech-services-in-sovereign-clouds"></a>Речевые службы в независимых облаках

## <a name="azure-government-united-states"></a>Azure для государственных организаций (США)

Доступно только для государственных организаций США и их партнеров. Дополнительные сведения о Azure [для государственных организаций](../../azure-government/documentation-government-welcome.md) см [. здесь.](../../azure-government/compare-azure-government-global-azure.md)

- **портал Azure:**
  - [https://portal.azure.us/](https://portal.azure.us/)
- **Регионах**
  - US Gov (Аризона)
  - US Gov (Вирджиния)
- **Доступные ценовые категории:**
  - Free (F0) и Standard (S0). Дополнительные сведения см. [здесь](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/)
- **Поддерживаемые функции**
  - Преобразование речи в текст
    - Пользовательское распознавание речи (акустическая модель (AM) и модель языка (LM))
      - [Speech Studio](https://speech.azure.us/)
  - Преобразование текста в речь
    - Стандартный Voice
    - Нейронный Voice
  - Переводчик речи
- **Неподдерживаемые функции:**
  - Custom Voice (Пользовательские голосовые модели)
- **Поддерживаемые языки:**
  - Список поддерживаемых языков см. [здесь](language-support.md) .

### <a name="endpoint-information"></a>Сведения о конечных точках

В этом разделе содержатся сведения о конечной точке речевых служб для использования с [речевым пакетом SDK](speech-sdk.md), [REST APIом преобразования речи в текст](rest-speech-to-text.md)и [REST API преобразования текста в речь](rest-text-to-speech.md).

#### <a name="speech-services-rest-api"></a>Службы речи REST API

Службы речи REST API конечные точки в Azure для государственных организаций имеют следующий формат:

|  Тип или операция REST API | Формат конечной точки |
|--|--|
| Маркер доступа | `https://<REGION_IDENTIFIER>.api.cognitive.microsoft.us/sts/v1.0/issueToken`
| [Преобразование речи в текст REST API v 3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30) | `https://<REGION_IDENTIFIER>.api.cognitive.microsoft.us/<URL_PATH>` |
| [REST API речи в текст для коротких аудио](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) | `https://<REGION_IDENTIFIER>.stt.speech.azure.us/<URL_PATH>` |
| [REST API преобразования текста в речь.](rest-text-to-speech.md) | `https://<REGION_IDENTIFIER>.tts.speech.azure.us/<URL_PATH>` |

Замените `<REGION_IDENTIFIER>` идентификатором, соответствующим региону подписки, из следующей таблицы:

|                     | Идентификатор региона |
|--|--|
| **US Gov (Аризона)**  | `usgovarizona` |
| **US Gov (Вирджиния)** | `usgovvirginia` |

#### <a name="speech-sdk"></a>Пакет SDK для службы "Речь"

Для речевого пакета SDK в независимых облаках необходимо использовать экземпляр `SpeechConfig` класса или `--host` параметр [интерфейса командной строки для распознавания](spx-overview.md)речи. (Вы также можете использовать создание экземпляров "из конечной точки" и `--endpoint` Параметр речевого интерфейса командной строки).

`SpeechConfig` класс должен быть создан следующим образом:
```csharp
var config = SpeechConfig.FromHost(usGovHost, subscriptionKey);
```
Речевой интерфейс командной строки следует использовать следующим образом (Обратите внимание на `--host` параметр):
```dos
spx recognize --host "usGovHost" --file myaudio.wav
```
Замените на `subscriptionKey` свой ключ ресурса речи. Замените на `usGovHost` выражение, совпадающее с требуемым предложением службы и регионом подписки из этой таблицы:

|  Предложение региона или услуги | Выражение узла |
|--|--|
| **US Gov (Аризона)** | |
| Преобразование речи в текст | `wss://usgovarizona.stt.speech.azure.us` |
| Преобразование текста в речь | `https://usgovarizona.tts.speech.azure.us` |
| **US Gov (Вирджиния)** | |
| Преобразование речи в текст | `wss://usgovvirginia.stt.speech.azure.us` |
| Преобразование текста в речь | `https://usgovvirginia.tts.speech.azure.us` |


## <a name="azure-china"></a>Azure для Китая

Доступ к организациям с бизнес-присутствием в Китае. Дополнительные сведения об Azure для Китая см [. здесь.](/azure/china/overview-operations) 


- **портал Azure:**
  - [https://portal.azure.cn/](https://portal.azure.cn/)
- **Регионах**
  - Восточный Китай 2
- **Доступные ценовые категории:**
  - Free (F0) и Standard (S0). Дополнительные сведения см. [здесь](https://www.azure.cn/pricing/details/cognitive-services/index.html)
- **Поддерживаемые функции**
  - Преобразование речи в текст
    - Пользовательское распознавание речи (акустическая модель (AM) и модель языка (LM))
      - [Speech Studio](https://speech.azure.cn/)
  - Преобразование текста в речь
    - Стандартный Voice
    - Нейронный Voice
  - Переводчик речи
- **Неподдерживаемые функции:**
  - Custom Voice (Пользовательские голосовые модели)
- **Поддерживаемые языки:**
  - Список поддерживаемых языков см. [здесь](language-support.md) .

### <a name="endpoint-information"></a>Сведения о конечных точках

В этом разделе содержатся сведения о конечной точке речевых служб для использования с [речевым пакетом SDK](speech-sdk.md), [REST APIом преобразования речи в текст](rest-speech-to-text.md)и [REST API преобразования текста в речь](rest-text-to-speech.md).

#### <a name="speech-services-rest-api"></a>Службы речи REST API

Службы речи REST API конечные точки в Azure для Китая имеют следующий формат:

|  Тип или операция REST API | Формат конечной точки |
|--|--|
| Маркер доступа | `https://<REGION_IDENTIFIER>.api.cognitive.azure.cn/sts/v1.0/issueToken`
| [Преобразование речи в текст REST API v 3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30) | `https://<REGION_IDENTIFIER>.api.cognitive.azure.cn/<URL_PATH>` |
| [REST API речи в текст для коротких аудио](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) | `https://<REGION_IDENTIFIER>.stt.speech.azure.cn/<URL_PATH>` |
| [REST API преобразования текста в речь.](rest-text-to-speech.md) | `https://<REGION_IDENTIFIER>.tts.speech.azure.cn/<URL_PATH>` |

Замените `<REGION_IDENTIFIER>` идентификатором, соответствующим региону подписки, из следующей таблицы:

|                     | Идентификатор региона |
|--|--|
| **Восточный Китай 2**  | `chinaeast2` |

#### <a name="speech-sdk"></a>Пакет SDK для службы "Речь"

Для речевого пакета SDK в независимых облаках необходимо использовать экземпляр `SpeechConfig` класса или `--host` параметр [интерфейса командной строки для распознавания](spx-overview.md)речи. (Вы также можете использовать создание экземпляров "из конечной точки" и `--endpoint` Параметр речевого интерфейса командной строки).

`SpeechConfig` класс должен быть создан следующим образом:
```csharp
var config = SpeechConfig.FromHost(azCnHost, subscriptionKey);
```
Речевой интерфейс командной строки следует использовать следующим образом (Обратите внимание на `--host` параметр):
```dos
spx recognize --host "azCnHost" --file myaudio.wav
```
Замените на `subscriptionKey` свой ключ ресурса речи. Замените на `azCnHost` выражение, совпадающее с требуемым предложением службы и регионом подписки из этой таблицы:

|  Предложение региона или услуги | Выражение узла |
|--|--|
| **Восточный Китай 2** | |
| Преобразование речи в текст | `wss://chinaeast2.stt.speech.azure.cn` |
| Преобразование текста в речь | `https://chinaeast2.tts.speech.azure.cn` |
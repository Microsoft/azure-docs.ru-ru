---
author: wolfma61
ms.service: cognitive-services
ms.topic: include
ms.date: 05/06/2019
ms.author: wolfma
ms.openlocfilehash: 1eefb7097f672552956e90a19e0a7b411cae8a24
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102455703"
---
### <a name="standard-and-neural-voices"></a>Стандартные и нейронные голоса

Используйте эту таблицу, чтобы определить **доступность стандартных голосов** по регионам и конечным точкам:

| Регион | Конечная точка |
|--------|----------|
| Восточная Австралия | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/v1` |
| Южная Бразилия | `https://brazilsouth.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центральная Канада | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центральная часть США | `https://centralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Восточная Азия | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/v1` |
| Восточная часть США | `https://eastus.tts.speech.microsoft.com/cognitiveservices/v1` |
| восточная часть США 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центральная Франция | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центральная Индия | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/v1` |
| Восточная Япония | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/v1` |
| Западная Япония | `https://japanwest.tts.speech.microsoft.com/cognitiveservices/v1` |
| Республика Корея, центральный регион | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центрально-северная часть США | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Северная Европа | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центрально-южная часть США | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Юго-Восточная Азия | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/v1` |
| южная часть Соединенного Королевства | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/v1` |
| Западная Европа | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/v1` |
| западная часть США | `https://westus.tts.speech.microsoft.com/cognitiveservices/v1` |
| западная часть США 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/v1` |

> [!TIP]
> [Голоса в предварительной версии](../articles/cognitive-services/Speech-Service/language-support.md#neural-voices-in-preview) доступны только в следующих трех регионах: Восточная часть США, Западная Европа и Юго-Восточная Азия.

Используйте эту таблицу для определения **доступности нейронных голосов** по регионам или конечным точкам:

| Регион | Конечная точка |
|--------|----------|
| Восточная Австралия | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/v1` | 
| Центральная Канада | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/v1` |
| Восточная часть США | `https://eastus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центральная Индия | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/v1` |
| Центрально-южная часть США | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/v1` |
| Юго-Восточная Азия | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/v1` |
| южная часть Соединенного Королевства | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/v1` |
| Западная Европа | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/v1` |
| западная часть США 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/v1` |

### <a name="custom-voices"></a>Настраиваемое синтезирование голоса

Если вы создали пользовательский шрифт голоса, используйте созданную конечную точку. Вы также можете использовать перечисленные ниже конечные точки, заменив параметр на `{deploymentId}` идентификатор развертывания для модели голоса.

| Регион | Конечная точка |
|--------|----------|
| Восточная Австралия | `https://australiaeast.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Южная Бразилия | `https://brazilsouth.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Центральная Канада | `https://canadacentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Центральная часть США | `https://centralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Восточная Азия | `https://eastasia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Восточная часть США | `https://eastus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| восточная часть США 2 | `https://eastus2.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Центральная Франция | `https://francecentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Центральная Индия | `https://centralindia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Восточная Япония | `https://japaneast.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Западная Япония | `https://japanwest.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Республика Корея, центральный регион | `https://koreacentral.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Центрально-северная часть США | `https://northcentralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Северная Европа | `https://northeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Центрально-южная часть США | `https://southcentralus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Юго-Восточная Азия | `https://southeastasia.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| южная часть Соединенного Королевства | `https://uksouth.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| Западная Европа | `https://westeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| западная часть США | `https://westus.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |
| западная часть США 2 | `https://westus2.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId={deploymentId}` |

### <a name="custom-neural-voice"></a>Что такое Пользовательский нейронный голос

В следующей таблице приведены сведения о региональной поддержке пользовательских функций нейронной речи.

| Компонент | Поддерживаемые регионы |
|---|---|
| Размещение модели речи | Восточная часть США, Западная часть США 2, Юго-Центральная часть США, Юго-Восточная Азия, южная часть Соединенного Королевства, Западная Европа, Восточная Австралия |
| Символы реального времени | Восточная часть США, Западная часть США 2, Юго-Центральная часть США, Юго-Восточная Азия, южная часть Соединенного Королевства, Западная Европа, Восточная Австралия |
| Длинные звуковые символы | Восточная часть США, Западная Европа, южная часть Соединенного Королевства, Юго-Восточная Азия, Центральная Индия |
| Пользовательское нейронное обучение | Восточная часть США, южная часть Соединенного Королевства |

### <a name="long-audio-api"></a>Длинный аудио API

Длинный аудио API доступен в нескольких регионах с уникальными конечными точками.

| Регион | Конечная точка |
|--------|----------|
| Восточная часть США | `https://eastus.customvoice.api.speech.microsoft.com` |
| Центральная Индия | `https://centralindia.customvoice.api.speech.microsoft.com` |
| Юго-Восточная Азия | `https://southeastasia.customvoice.api.speech.microsoft.com` |
| южная часть Соединенного Королевства | `https://uksouth.customvoice.api.speech.microsoft.com` |
| Западная Европа | `https://westeurope.customvoice.api.speech.microsoft.com` |

---
title: Регионы публикации & конечные точки — LUIS
description: Регион, указанный в портал Azure, совпадает с областью, в которой будет опубликовано приложение LUIS, а для этого же региона создается URL-адрес конечной точки.
ms.service: cognitive-services
ms.subservice: language-understanding
author: aahill
ms.author: aahi
ms.topic: reference
ms.date: 01/21/2021
ms.custom: references_regions
ms.openlocfilehash: 8b43fc472f3247a93414a0b18d9098c6dfb94917
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98681613"
---
# <a name="authoring-and-publishing-regions-and-the-associated-keys"></a>Регионы создания и публикации и связанные ключи

LUIS области разработки поддерживаются порталом LUIS. Чтобы опубликовать приложение LUIS в несколько регионов, требуется по меньшей мере один ключ для каждого региона.

<a name="luis-website"></a>

## <a name="luis-authoring-regions"></a>LUIS области разработки

[!INCLUDE [portal consolidation](includes/portal-consolidation.md)]

LUIS содержит один портал, который можно использовать независимо от региона, [www.Luis.AI](https://www.luis.ai). По-прежнему необходимо создавать и публиковать в одном и том же регионе.

Области разработки имеют [парные регионы отработки отказа](../../best-practices-availability-paired-regions.md)

<a name="regions-and-azure-resources"></a>

## <a name="publishing-regions-and-azure-resources"></a>Регионы публикации и ресурсы Azure

Приложения можно публиковать во всех регионах, которые связаны с ресурсами LUIS, добавленными на портале LUIS. Например, для приложения, созданного на [www.Luis.AI][www.luis.ai], при создании ресурса Luis или службы в **westus** и [добавлении его в приложение в качестве ресурса](luis-how-to-azure-subscription.md)приложение публикуется в этом регионе.

## <a name="public-apps"></a>Общедоступные приложения
Общедоступное приложение можно публиковать во всех регионах, чтобы пользователь с ключом ресурса LUIS для определенного региона смог получить доступ к приложению в том регионе, который связан с ключом ресурса.

<a name="publishing-regions"></a>

## <a name="publishing-regions-are-tied-to-authoring-regions"></a>Регионы публикации привязаны к областям разработки

Приложение региона разработки можно публиковать только в соответствующий регион публикации. Если приложение находится в неправильном регионе разработки, экспортируйте его, а затем импортируйте в правильный регион разработки для региона публикации.

> [!NOTE]
> Приложения LUIS, созданные на https://www.luis.ai , теперь могут быть опубликованы во всех конечных точках, включая [европейские](#publishing-to-europe) и [австралийские](#publishing-to-australia) регионы.

## <a name="publishing-to-europe"></a>Публикация в Европе

 Глобальный регион | Область API создания | Регион публикации и запроса<br>`API region name`   |  Формат URL-адреса конечной точки   |
|-----|------|------|------|
| Европа | `westeurope`| Центральная Франция<br>`francecentral`     | `https://francecentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Европа | `westeurope`| Северная Европа<br>`northeurope`     | `https://northeurope.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Европа | `westeurope`| Западная Европа<br>`westeurope`    |  `https://westeurope.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Европа | `westeurope`| южная часть Соединенного Королевства<br>`uksouth`    |  `https://uksouth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="publishing-to-australia"></a>Публикация в Австралии

 Глобальный регион | Область API создания | Регион публикации и запроса<br>`API region name`   |  Формат URL-адреса конечной точки   |
|-----|------|------|------|
| Австралия | `australiaeast` | Восточная Австралия<br>`australiaeast`     |  `https://australiaeast.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="other-publishing-regions"></a>Другие регионы публикации

 Глобальный регион | Область API создания | Регион публикации и запроса<br>`API region name`   |  Формат URL-адреса конечной точки   |
|-----|------|------|------|
| Африка | `westus`<br>[www.luis.ai][www.luis.ai]| Северная часть ЮАР;<br>`southafricanorth` |  `https://southafricanorth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Центральная Индия<br>`centralindia` |  `https://centralindia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Восточная Азия<br>`eastasia`     |  `https://eastasia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Japan East<br>`japaneast`     |   `https://japaneast.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Западная Япония<br>`japanwest`     |   `https://japanwest.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Республика Корея, центральный регион<br>`koreacentral`     |   `https://koreacentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Southeast Asia<br>`southeastasia`     |   `https://southeastasia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Северная ОАЭ<br>`northuae`     |   `https://northuae.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Центральная Канада<br>`canadacentral`     |   `https://canadacentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Центральная часть США<br>`centralus`     |   `https://centralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Восточная часть США<br>`eastus`      |  `https://eastus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | восточная часть США 2<br>`eastus2`     |  `https://eastus2.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Центрально-северная часть США<br>`northcentralus`  |  `https://northcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Центрально-южная часть США<br>`southcentralus`  |  `https://southcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | центрально-западная часть США<br>`westcentralus`    |  `https://westcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Западная часть США<br>`westus`  |   `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Западная часть США 2<br>`westus2`    |  `https://westus2.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |
| Южная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Brazil South<br>`brazilsouth`    |  `https://brazilsouth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY` |

## <a name="endpoints"></a>Конечные точки

Узнайте больше о [конечных точках создания и прогнозирования](developer-reference-resource.md).

## <a name="failover-regions"></a>Регионы отработки отказа

Каждый регион имеет дополнительный регион для отработки отказа. При отработки отказа Европы в Европе и Австралии происходит сбой в Австралии.

Области разработки имеют [парные регионы отработки отказа](../../best-practices-availability-paired-regions.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Справочник по предварительно созданным сущностям](./luis-reference-prebuilt-entities.md)

 [www.luis.ai]: https://www.luis.ai
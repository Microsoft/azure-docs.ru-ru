---
title: Регионы публикации & конечные точки — LUIS
description: Регион, указанный в портал Azure, совпадает с областью, в которой будет опубликовано приложение LUIS, а для этого же региона создается URL-адрес конечной точки.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 11/09/2020
ms.openlocfilehash: bb2aa2954ab4aed5466858226e385155b9da8ec4
ms.sourcegitcommit: 8f0803d3336d8c47654e119f1edd747180fe67aa
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/07/2021
ms.locfileid: "97977036"
---
# <a name="authoring-and-publishing-regions-and-the-associated-keys"></a>Регионы создания и публикации и связанные ключи

[!INCLUDE [LUIS Free account](includes/luis-portal-note.md)]

Соответствующие порталы LUIS поддерживают три области разработки. Чтобы опубликовать приложение LUIS в несколько регионов, требуется по меньшей мере один ключ для каждого региона.

<a name="luis-website"></a>

## <a name="luis-authoring-regions"></a>LUIS области разработки
На основе региона существует три LUIS портала для создания порталов. Разработку и публикацию следует выполнять в одном регионе.

|LUIS|Регион разработки|Имя региона Azure|
|--|--|--|
|[www.luis.ai][www.luis.ai] |США<br>не Европа<br>не Австралия| `westus`|
|[au.luis.ai][au.luis.ai] |Австралия| `australiaeast`|
|[eu.luis.ai][eu.luis.ai] |Европа|`westeurope`|

Области разработки имеют [парные регионы отработки отказа](../../best-practices-availability-paired-regions.md).

<a name="regions-and-azure-resources"></a>

## <a name="publishing-regions-and-azure-resources"></a>Регионы публикации и ресурсы Azure
Приложения можно публиковать во всех регионах, которые связаны с ресурсами LUIS, добавленными на портале LUIS. Например, для приложения, созданного на [www.Luis.AI][www.luis.ai], при создании ресурса Luis или службы в **westus** и [добавлении его в приложение в качестве ресурса](luis-how-to-azure-subscription.md)приложение публикуется в этом регионе.

## <a name="public-apps"></a>Общедоступные приложения
Общедоступное приложение можно публиковать во всех регионах, чтобы пользователь с ключом ресурса LUIS для определенного региона смог получить доступ к приложению в том регионе, который связан с ключом ресурса.

<a name="publishing-regions"></a>

## <a name="publishing-regions-are-tied-to-authoring-regions"></a>Регионы публикации привязаны к областям разработки

Приложение региона разработки можно публиковать только в соответствующий регион публикации. Если приложение находится в неправильном регионе разработки, экспортируйте его, а затем импортируйте в правильный регион разработки для региона публикации.

Приложения LUIS, созданные в https://www.luis.ai, могут быть опубликованы во всех конечных точках, за исключением [европейских](#publishing-to-europe) и [австралийских](#publishing-to-australia) регионов.

## <a name="publishing-to-europe"></a>Публикация в Европе

Для публикации в европейские регионы приложения LUIS нужно создавать только по адресу https://eu.luis.ai. При попытке выполнить публикацию в другое место с помощью ключа в европейском регионе LUIS отображает предупреждающее сообщение. Вместо этого используйте https://eu.luis.ai. Приложения LUIS, созданные в, [https://eu.luis.ai][eu.luis.ai] не переносятся автоматически в другие регионы. Чтобы перенести приложение LUIS, экспортируйте, а затем импортируйте его.

## <a name="europe-publishing-regions"></a>Регионы публикации в Европе

 Глобальный регион | Регион разработки API и разработки веб-сайта| Регион публикации и запроса<br>`API region name`   |  Формат URL-адреса конечной точки   |
|-----|------|------|------|
| [Европа](#publishing-to-europe)| `westeurope`<br>[eu.luis.ai][eu.luis.ai]| Центральная Франция<br>`francecentral`     | `https://francecentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| [Европа](#publishing-to-europe)| `westeurope`<br>[eu.luis.ai][eu.luis.ai]| Северная Европа<br>`northeurope`     | `https://northeurope.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| [Европа](#publishing-to-europe) | `westeurope`<br>[eu.luis.ai][eu.luis.ai]| Западная Европа<br>`westeurope`    |  `https://westeurope.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| [Европа](#publishing-to-europe) | `westeurope`<br>[eu.luis.ai][eu.luis.ai]| южная часть Соединенного Королевства<br>`uksouth`    |  `https://uksouth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="publishing-to-australia"></a>Публикация в Австралии

Для публикации в австралийские регионы приложения LUIS нужно создавать только по адресу https://au.luis.ai. При попытке выполнить публикацию в другое место с помощью ключа в австралийском регионе LUIS отображает предупреждающее сообщение. Вместо этого используйте https://au.luis.ai. Приложения LUIS, созданные в, [https://au.luis.ai][au.luis.ai] не переносятся автоматически в другие регионы. Чтобы перенести приложение LUIS, экспортируйте, а затем импортируйте его.

## <a name="australia-publishing-regions"></a>Регионы публикации Австралии

 Глобальный регион | Регион разработки API и разработки веб-сайта| Регион публикации и запроса<br>`API region name`   |  Формат URL-адреса конечной точки   |
|-----|------|------|------|
| [Австралия](#publishing-to-australia) | `australiaeast`<br>[au.luis.ai][au.luis.ai]| Восточная Австралия<br>`australiaeast`     |  `https://australiaeast.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="publishing-to-other-regions"></a>Публикация в других регионах

Для публикации в других регионах вы создаете приложения LUIS только по адресу [https://www.luis.ai](https://www.luis.ai) .

## <a name="other-publishing-regions"></a>Другие регионы публикации

 Глобальный регион | Регион разработки API и разработки веб-сайта| Регион публикации и запроса<br>`API region name`   |  Формат URL-адреса конечной точки   |
|-----|------|------|------|
| Африка | `westus`<br>[www.luis.ai][www.luis.ai]| Северная часть ЮАР;<br>`southafricanorth` |  `https://southafricanorth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Центральная Индия<br>`centralindia` |  `https://centralindia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Восточная Азия<br>`eastasia`     |  `https://eastasia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Восточная Япония<br>`japaneast`     |   `https://japaneast.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Западная Япония<br>`japanwest`     |   `https://japanwest.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Республика Корея, центральный регион<br>`koreacentral`     |   `https://koreacentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Юго-Восточная Азия<br>`southeastasia`     |   `https://southeastasia.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Азия | `westus`<br>[www.luis.ai][www.luis.ai]| Северная ОАЭ<br>`northuae`     |   `https://northuae.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Центральная Канада<br>`canadacentral`     |   `https://canadacentral.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Центральная часть США<br>`centralus`     |   `https://centralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Восточная часть США<br>`eastus`      |  `https://eastus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | восточная часть США 2<br>`eastus2`     |  `https://eastus2.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Центрально-северная часть США<br>`northcentralus`  |  `https://northcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Центрально-южная часть США<br>`southcentralus`  |  `https://southcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | центрально-западная часть США<br>`westcentralus`    |  `https://westcentralus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |
| Северная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | западная часть США<br>`westus`  |   `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`  |
| Северная Америка |`westus`<br>[www.luis.ai][www.luis.ai] | Западная часть США 2<br>`westus2`    |  `https://westus2.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`  |
| Южная Америка | `westus`<br>[www.luis.ai][www.luis.ai] | Южная Бразилия<br>`brazilsouth`    |  `https://brazilsouth.api.cognitive.microsoft.com/luis/v2.0/apps/YOUR-APP-ID?subscription-key=YOUR-SUBSCRIPTION-KEY`   |

## <a name="endpoints"></a>Конечные точки

Узнайте больше о [конечных точках создания и прогнозирования](developer-reference-resource.md).

## <a name="failover-regions"></a>Регионы отработки отказа

Каждый регион имеет дополнительный регион для отработки отказа. При отработки отказа Европы в Европе и Австралии происходит сбой в Австралии.

Области разработки имеют [парные регионы отработки отказа](../../best-practices-availability-paired-regions.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Справочник по предварительно созданным сущностям](./luis-reference-prebuilt-entities.md)

 [www.luis.ai]: https://www.luis.ai
 [au.luis.ai]: https://au.luis.ai
 [eu.luis.ai]: https://eu.luis.ai

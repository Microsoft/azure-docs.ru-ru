---
title: Часто задаваемые вопросы о Cognitive Services контейнерах
titleSuffix: Azure Cognitive Services
description: Часто задаваемые вопросы и ответы на них.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: 71b57eae1a66e6966f61123e638c4790410ef445
ms.sourcegitcommit: aeba98c7b85ad435b631d40cbe1f9419727d5884
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2021
ms.locfileid: "97862535"
---
# <a name="azure-cognitive-services-containers-frequently-asked-questions-faq"></a>Часто задаваемые вопросы о контейнерах Azure Cognitive Services

## <a name="general-questions"></a>Общие вопросы

**Вопрос. что доступно?**

Ответ **.** Контейнеры Azure Cognitive Services позволяют разработчикам использовать те же Интеллектуальные API, которые доступны в Azure, но с [преимуществами](../cognitive-services-container-support.md#features-and-benefits) использования контейнеров. Некоторые контейнеры доступны в виде предварительной версии, для доступа к которым может потребоваться приложение. Другие контейнеры общедоступны в режиме предварительной версии или являются общедоступными. Полный список контейнеров и их доступности можно найти в статье [Поддержка контейнеров в Cognitive Services Azure](../cognitive-services-container-support.md) . Вы также можете просмотреть контейнеры в [центре DOCKER](https://hub.docker.com/_/microsoft-azure-cognitive-services).

**Вопрос. Существует ли разница между Cognitive Services облаком и контейнерами?**

Ответ **.** Cognitive Services контейнеры являются альтернативой Cognitive Services облаку. Контейнеры предлагают те же возможности, что и соответствующие облачные службы. Клиенты могут развертывать контейнеры локально или в Azure. Базовая технология искусственного интеллекта, ценовые категории, ключи API и сигнатуры API одинаковы между контейнером и соответствующими облачными службами. Ниже приведены [функции и преимущества](../cognitive-services-container-support.md#features-and-benefits) выбора контейнеров в эквиваленте облачной службы.

**Вопрос. Разделы справки получить доступ к контейнеру предварительной версии или использовать его?**

Ответ **.** Ранее контейнеры с условным просмотром были размещены в `containerpreview.azurecr.io` репозитории. Начиная с сентября 22 2020, эти контейнеры размещаются в реестре контейнеров Майкрософт, и для их загрузки не требуется использовать команду DOCKER Login. Вы сможете запустить контейнер с условным просмотром, если ваш ресурс Azure создан с утвержденным ИДЕНТИФИКАТОРом подписки Azure. Вы не сможете запустить контейнер, если подписка Azure не была утверждена после завершения [формы запроса](https://aka.ms/csgate).


**Вопрос. будут ли контейнеры доступны для всех Cognitive Services и каковы следующие наборы контейнеров, которые следует рассчитывать?**

Ответ **.** Мы хотим сделать больше Cognitive Services доступными в качестве предложений контейнеров. Обратитесь к локальному диспетчеру учетная запись Майкрософт, чтобы получать обновления для новых выпусков контейнеров и других Cognitive Services объявлений.

**Вопрос. что будет Service-Levelным соглашением (SLA) для Cognitive Services контейнеров?**

Ответ **.** Cognitive Services контейнеры не имеют соглашения об уровне обслуживания.

Cognitive Services конфигурации контейнеров ресурсов контролируются клиентами, поэтому корпорация Майкрософт не будет предлагать соглашение об уровне обслуживания для общедоступной версии. Клиенты могут свободно развертывать контейнеры в локальной среде, поэтому они определяют среды размещения.

> [!IMPORTANT]
> Дополнительные сведения о соглашениях Cognitive Services Service-Level см. на [странице соглашения об уровне обслуживания](https://azure.microsoft.com/support/legal/sla/cognitive-services/v1_1/).

**Вопрос. доступны ли эти контейнеры в облаках независимых?**

Ответ **.** Не все знакомы с термином «независимых Cloud», поэтому начнем с определения:

> "Независимых Cloud" состоит из облаков [Azure для государственных организаций](../../azure-government/documentation-government-welcome.md), [Azure для Германии](../../germany/germany-welcome.md)и [Azure для Китая](/azure/china/overview-operations) .

К сожалению, контейнеры Cognitive Services изначально *не* поддерживаются в облаках независимых. Контейнеры могут быть запущены в этих облаках, но они будут извлечены из общедоступного облака и необходимо будет отправить данные об использовании в общедоступную конечную точку.

### <a name="versioning"></a>Управление версиями

**Вопрос. как контейнеры обновляются до последней версии?**

Ответ **.** Клиенты могут выбрать время обновления развернутых контейнеров. Контейнеры будут помечены стандартными [тегами DOCKER](https://docs.docker.com/engine/reference/commandline/tag/) , например, `latest` для указания самой последней версии. Мы советуем клиентам извлечь последнюю версию контейнеров по мере их выпуска, а также извлечь [веб-перехватчики реестра контейнеров Azure](../../container-registry/container-registry-webhook.md) , чтобы получить сведения о том, как получать уведомления об обновлении образа.
 
**Вопрос. какие версии будут поддерживаться?**

Ответ **.** Будет поддерживаться текущая и последняя основная версия контейнера. Однако мы рекомендуем клиентам оставаться актуальными, чтобы получить новейшие технологии.
 
**Вопрос. как обновляются версии обновлений?**

Ответ **.** Изменения основной версии указывают на то, что в сигнатуре API имеется критическое изменение. Мы ожидаем, что это будет в целом совпадать с основными изменениями версии соответствующего предложения по обприятию. Изменения дополнительных версий указывают на исправления ошибок, обновления модели или новые функции, которые не применяют критические изменения к сигнатуре API.

## <a name="technical-questions"></a>Технические вопросы

**Вопрос. как запускать контейнеры Cognitive Services на устройствах IoT?**

Ответ **.** Если у вас нет надежного подключения к Интернету или вы хотите сэкономить на стоимости пропускной способности. Если у вас есть требования с малой задержкой или работа с конфиденциальными данными, которые необходимо проанализировать на сайте, [Azure IOT Edge с контейнерами Cognitive Services](https://azure.microsoft.com/blog/running-cognitive-services-on-iot-edge/) обеспечивает согласованность с облаком.

**Вопрос. являются ли эти контейнеры совместимыми с OpenShift?** 

Мы не тестируем контейнеры с помощью OpenShift, но обычно контейнеры Cognitive Services должны выполняться на любой платформе, поддерживающей образы DOCKER. Если вы используете OpenShift, рекомендуется запустить контейнеры как `root-user` .

**Вопрос. Разделы справки предоставить отзыв о продукте и рекомендации по функциям?**

Ответ **.** Клиентам рекомендуется накладывать [свои вопросы](https://cognitive.uservoice.com/) в общедоступном виде и проголосовать за них, которые сделали то же самое, когда проблемы перекрываются. Средство User Voice Tool можно использовать как для отзывов продуктов, так и для рекомендаций по функциям.

**Вопрос. какие сообщения о состоянии и ошибки возвращаются Cognitive Services контейнерами?**

Ответ **.** Список сообщений о состоянии и ошибок см. в следующей таблице.

|Состояние  | Описание  |
|---------|---------|
| `Valid` | Ключ API действителен, никаких действий не требуется. |
| `Invalid` |   Недопустимый ключ API. Для запуска контейнера необходимо указать допустимый ключ API. Найдите ключ API и регион службы в разделе **ключи и конечная точка** для ресурса Azure Cognitive Services в портал Azure. |
| `Mismatch` | Вы указали ключ или конечную точку API для другого типа ресурса "службы". Найдите ключ API и регион службы в разделе **ключи и конечная точка** для ресурса Azure Cognitive Services. |
| `CouldNotConnect` | Контейнеру не удалось подключиться к конечной точке выставления счетов. Проверьте `Retry-After` значение и дождитесь окончания этого периода перед выполнением дополнительных запросов. |
| `OutOfQuota` | Ключ API не входит в квоту. Можно либо обновить ценовую категорию, либо дождаться доступности дополнительной квоты. Найдите свой уровень в разделе **ценовая** Категория в ресурсе службы "Поиск в Azure" в портал Azure. |
| `BillingEndpointBusy` | Конечная точка выставления счетов в данный момент занята. Проверьте `Retry-After` значение и дождитесь окончания этого периода перед выполнением дополнительных запросов. |
| `ContainerUseUnauthorized` | Указанный ключ API не разрешен для использования с этим контейнером. Вероятно, вы используете контейнер с условным обозначением, поэтому убедитесь, что идентификатор подписки Azure утвержден, отправив [запрос в сети](https://aka.ms/csgate). |
| `Unknown` | В настоящее время сервер не может обрабатывать запросы на выставление счетов. |


**Вопрос. кто поддается за поддержку?**

Ответ **.** Каналы поддержки клиентов совпадают с Cognitive Servicesным предложением облака. Все контейнеры Cognitive Services включают функции ведения журнала, которые помогут нам и клиентам службы поддержки сообщества. Дополнительные сведения о поддержке см. в следующих параметрах.

### <a name="customer-support-plan"></a>План поддержки клиентов

Клиенты должны обратиться к [плану поддержки Azure](https://azure.microsoft.com/support/plans/) , чтобы узнать, кто будет обращаться за поддержкой.

### <a name="azure-knowledge-center"></a>Центр знаний Azure

Клиент свободен для изучения [центра знаний Azure](https://azure.microsoft.com/resources/knowledge-center/) , чтобы ответить на вопросы и проблемы поддержки.

### <a name="stack-overflow"></a>Stack Overflow

> [Stack overflow](https://en.wikipedia.org/wiki/Stack_Overflow) — это сайт вопросов и ответов для профессиональных программистов и энтузиастов.

Изучите следующие теги для потенциальных вопросов и ответов, которые отвечают вашим потребностям.

* [Azure Cognitive Services](https://stackoverflow.com/questions/tagged/azure-cognitive-services)
* [Microsoft для себя](https://stackoverflow.com/questions/tagged/microsoft-cognitive)

**Вопрос. как работает выставление счетов?**

Ответ **.** Клиентам выставляются счета на основе потребления, как в Cognitive Services облаке. Необходимо настроить контейнеры для отправки данных измерения в Azure, и счета за транзакции будут выставляться соответствующим образом. Ресурсы, используемые в размещенных и локальных службах, будут добавляться в одну квоту с многоуровневыми ценами, подсчитанными по обоим расходам. Дополнительные сведения см. на странице цен соответствующего предложения.

* [Детектор аномалий][ad-containers-billing]
* [Компьютерное зрение][cv-containers-billing]
* [Распознавание лиц][fa-containers-billing]
* [Распознаватель документов][fr-containers-billing]
* [Language Understanding (LUIS)][lu-containers-billing]
* [API службы "Речь"][sp-containers-billing]
* [Анализ текста][ta-containers-billing]

> [!IMPORTANT]
> Контейнеры Cognitive Services не лицензируются для запуска без подключения к Azure для отслеживания использования. Клиенты должны разрешить контейнерам непрерывную передачу данных для выставления счетов в службу контроля потребления. Контейнеры Cognitive Services не отправляют данные клиента в корпорацию Майкрософт.
 
**Вопрос. Какова текущая гарантия на поддержку контейнеров?**

Ответ **.** Для предварительных версий отсутствует гарантия. Стандартная гарантия Майкрософт для корпоративного программного обеспечения будет применяться, когда контейнеры формально объявлены как общедоступные.
 
**Вопрос. что происходит с Cognitive Services контейнерами при потере подключения к Интернету?**

Ответ **.** Cognitive Services контейнеры *не лицензированы* для запуска без подключения к Azure для измерения. Клиентам необходимо включить контейнеры для взаимодействия со службой измерения.

**Вопрос. как долго контейнер может действовать без подключения к Azure?**

Ответ **.** Cognitive Services контейнеры *не лицензированы* для запуска без подключения к Azure для измерения. Клиентам необходимо включить контейнеры для взаимодействия со службой измерения.
 
**Вопрос. Каково текущее оборудование, необходимое для выполнения этих контейнеров?**

Ответ **.** Контейнеры Cognitive Services — это основанные на x64 контейнеры, которые могут запускать любой совместимый узел Linux, виртуальную машину и пограничное устройство, поддерживающее контейнеры DOCKER x64. Все они нуждаются в процессорах ЦП. Ниже приведены минимальные и рекомендуемые конфигурации для каждого предложения контейнера.

* [Детектор аномалий][ad-containers-recommendations]
* [Компьютерное зрение][cv-containers-recommendations]
* [Распознавание лиц][fa-containers-recommendations]
* [Распознаватель документов][fr-containers-recommendations]
* [Language Understanding (LUIS)][lu-containers-recommendations]
* [API службы "Речь"][sp-containers-recommendations]
* [Анализ текста][ta-containers-recommendations]
 
**Вопрос. являются ли эти контейнеры в настоящее время поддерживаемыми в Windows?**

Ответ **.** Контейнеры Cognitive Services — это контейнеры Linux, однако существует поддержка контейнеров Linux в Windows. Дополнительные сведения о контейнерах Linux в Windows см. в [документации по DOCKER](https://blog.docker.com/2017/09/preview-linux-containers-on-windows/).
 
**Вопрос. Разделы справки обнаружить контейнеры?**

Ответ **.** Контейнеры Cognitive Services доступны в различных местах, таких как портал Azure, концентратор DOCKER и реестры контейнеров Azure. Сведения о последних расположениях контейнеров см. в разделе [образы контейнеров](container-image-tags.md).

**Вопрос. как Cognitive Services контейнеры сравниваются с предложениями AWS и Google?**

Ответ **.** Корпорация Майкрософт является первым поставщиком облачных решений для перемещения предварительно обученных моделей искусственного интеллекта в контейнеры с простой оплатой за каждую транзакцию, как если бы клиенты использовали облачную службу. Корпорация Майкрософт считает, что гибридное облако предоставляет клиентам дополнительные возможности выбора.

**Вопрос. какие сертификаты соответствуют контейнерам?**

Ответ **.** В контейнерах служб не предусмотрены сертификаты соответствия требованиям

**Вопрос. какие регионы доступны в Cognitive Services контейнерах?**

Ответ **.** Контейнеры можно запускать где угодно в любом регионе, однако им требуется ключ и обратная связь с Azure для контроля использования. Все поддерживаемые регионы для облачной службы поддерживаются для вызова контроля использования контейнеров.

[!INCLUDE [Containers next steps](includes/containers-next-steps.md)]

[ad-containers]: ../anomaly-Detector/anomaly-detector-container-howto.md
[cv-containers]: ../computer-vision/computer-vision-how-to-install-containers.md
[fa-containers]: ../face/face-how-to-install-containers.md
[fr-containers]: ../form-recognizer/form-recognizer-container-howto.md
[lu-containers]: ../luis/luis-container-howto.md
[sp-containers]: ../speech-service/speech-container-howto.md
[ta-containers]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md

[ad-containers-billing]: ../anomaly-Detector/anomaly-detector-container-howto.md#billing
[cv-containers-billing]: ../computer-vision/computer-vision-how-to-install-containers.md#billing
[fa-containers-billing]: ../face/face-how-to-install-containers.md#billing
[fr-containers-billing]: ../form-recognizer/form-recognizer-container-howto.md#billing
[lu-containers-billing]: ../luis/luis-container-howto.md#billing
[sp-containers-billing]: ../speech-service/speech-container-howto.md#billing
[ta-containers-billing]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md#billing

[ad-containers-recommendations]: ../anomaly-Detector/anomaly-detector-container-howto.md#container-requirements-and-recommendations
[cv-containers-recommendations]: ../computer-vision/computer-vision-how-to-install-containers.md#container-requirements-and-recommendations
[fa-containers-recommendations]: ../face/face-how-to-install-containers.md#container-requirements-and-recommendations
[fr-containers-recommendations]: ../form-recognizer/form-recognizer-container-howto.md#container-requirements-and-recommendations
[lu-containers-recommendations]: ../luis/luis-container-howto.md#container-requirements-and-recommendations
[sp-containers-recommendations]: ../speech-service/speech-container-howto.md#container-requirements-and-recommendations
[ta-containers-recommendations]: ../text-analytics/how-tos/text-analytics-how-to-install-containers.md#container-requirements-and-recommendations
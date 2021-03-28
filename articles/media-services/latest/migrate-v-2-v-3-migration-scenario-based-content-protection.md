---
title: Руководство по миграции защиты содержимого
description: В этой статье приводятся рекомендации по сценариям защиты содержимого, которые помогут вам выполнить миграцию из служб мультимедиа Azure версии 2 на v3.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: conceptual
ms.workload: media
ms.date: 03/26/2021
ms.author: inhenkel
ms.openlocfilehash: 7ef41b76f343d8997feebc4a366deda7ce6a2afa
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105644062"
---
# <a name="content-protection-scenario-based-migration-guidance"></a>Руководство по миграции на основе сценариев защиты содержимого

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![шаги миграции 2](./media/migration-guide/steps-4.svg)

Эта статья содержит подробные сведения и рекомендации по переносу вариантов использования защиты содержимого из API v2 в новый API служб мультимедиа Azure v3.

## <a name="protect-content-in-v3-api"></a>Защита содержимого в API V3

Используйте поддержку [многоключовых](design-multi-drm-system-with-access-control.md) функций в новом API V3.

Подробные инструкции см. в разделе Основные понятия защиты содержимого, руководства и инструкции.

## <a name="visibility-of-v2-assets-streaminglocators-and-properties-in-the-v3-api-for-content-protection-scenarios"></a>Видимость ресурсов v2, Стреаминглокаторс и свойств в API V3 для сценариев защиты содержимого

Во время миграции на API V3 вы обнаружите, что вам нужно получить доступ к некоторым свойствам или ключам содержимого из ресурсов v2. Одно из ключевых различий заключается в том, что API v2 будет использовать **AssetId** в качестве первичного ключа идентификации, а новый API V3 использует имя управления ресурсами Azure сущности в качестве основного идентификатора.  Свойство **Asset.Name** v2 обычно не используется в качестве уникального идентификатора, поэтому при переходе на v3 вы увидите, что имена ресурсов v2 теперь отображаются в поле **Asset. Description** .

Например, если ранее имелся ресурс версии 2 с ИДЕНТИФИКАТОРом **«NetBIOS: CID: UUID: 8cb39104-122c-496e-9ac5-7f9e2c2547b8»**, то при перечислении старых ресурсов v2 через API V3 имя теперь будет частью идентификатора GUID в конце (в данном случае **«8cb39104-122c-496e-9ac5-7f9e2c2547b8»**).

Вы можете запросить **стреаминглокаторс** , связанный с ресурсами, созданными в API v2, с помощью нового метода v3 [Листстреаминглокаторс](https://docs.microsoft.com/rest/api/media/assets/liststreaminglocators) в сущности Asset.  Также см. ссылку на версию пакета SDK для клиента .NET [листстреаминглокаторсасинк](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.assetsoperationsextensions.liststreaminglocatorsasync?view=azure-dotnet)

Результаты метода **листстреаминглокаторс** будут предоставлять **имя** и **Стреаминглокаторид** локатора вместе с **стреамингполицинаме**.

Чтобы найти **contentkey** , используемые в **стреаминглокаторс** для защиты содержимого, можно вызвать метод [стреаминглокатор. листконтенткэйсасинк](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.media.streaminglocatorsoperationsextensions.listcontentkeysasync?view=azure-dotnet) .  

Все **активы** , созданные и опубликованные с помощью API v2, будут иметь как [политику ключа содержимого](https://docs.microsoft.com/azure/media-services/latest/content-key-policy-concept) , так и ключ содержимого, определенные для них в API V3 вместо использования политики по умолчанию ключа содержимого в [политике потоковой передачи](https://docs.microsoft.com/azure/media-services/latest/streaming-policy-concept).

Дополнительные сведения о защите содержимого в API V3 см. в статье [Защита содержимого с помощью динамического шифрования служб мультимедиа.](https://docs.microsoft.com/azure/media-services/latest/content-protection-overview)

## <a name="how-to-list-your-v2-assets-and-content-protection-settings-using-the-v3-api"></a>Как получить список ресурсов v2 и параметров защиты содержимого с помощью API V3

В API v2 для защиты потокового содержимого обычно используются **активы**, **стреаминглокаторс** и **contentkey** .
При переходе на API V3 ресурсы API v2, Стреаминглокаторс и Contentkey предоставляются автоматически в API V3, и все данные на них доступны для доступа.

## <a name="can-i-update-v2-properties-using-the-v3-api"></a>Можно ли обновить свойства v2 с помощью API V3?

Нет, обновить свойства сущностей v2 через API V3, созданные с помощью Стреаминглокаторс, СтреамингполиЦиес, политик ключей содержимого и ключей содержимого в v2, нельзя.
Если необходимо обновлять, изменять или изменять содержимое, хранящееся в сущностях v2, необходимо обновить его через API v2 или создать новые сущности API V3, чтобы перенести их вперед.

## <a name="how-do-i-change-the-contentkeypolicy-used-for-a-v2-asset-that-is-published-and-keep-the-same-content-key"></a>Разделы справки изменить Контенткэйполици, используемый для публикуемого ресурса v2 и сохранив тот же ключ содержимого?

В этом случае необходимо сначала отменить публикацию (удалить все указатели потоковой передачи) в ресурсе с помощью пакета SDK v2 (удалите указатель, удалить связь с политикой авторизации ключа содержимого, удалить связь с ключом содержимого, удалите ключ содержимого), а затем создать новый **[стреаминглокатор](https://docs.microsoft.com/azure/media-services/latest/streaming-locators-concept)** в v3 с помощью v3 [стреамингполици](https://docs.microsoft.com/azure/media-services/latest/streaming-policy-concept) и [контенткэйполици](https://docs.microsoft.com/azure/media-services/latest/content-key-policy-concept).

При создании **[стреаминглокатор](https://docs.microsoft.com/azure/media-services/latest/streaming-locators-concept)** необходимо указать конкретный идентификатор ключа содержимого и значение ключа.

Обратите внимание, что можно удалить локатор v2 с помощью API V3, но это не приведет к удалению ключа содержимого или политики ключей содержимого, которые использовались, если они были созданы в API v2.  

## <a name="using-amse-v2-and-amse-v3-side-by-side"></a>Параллельное использование AMSE v2 и AMSE v3

При переносе содержимого из v2 в версии 3 рекомендуется установить [средство обозревателя служб мультимедиа Azure версии 2](https://github.com/Azure/Azure-Media-Services-Explorer/releases/tag/v4.3.15.0) вместе с [инструментом обозревателя служб мультимедиа v3 Azure](https://github.com/Azure/Azure-Media-Services-Explorer) , чтобы сравнить данные, которые они показывают рядом, для ресурса, созданного и опубликованного с помощью API v2. Свойства должны быть видимыми, но в несколько других расположениях.  


## <a name="content-protection-concepts-tutorials-and-how-to-guides"></a>Основные понятия защиты содержимого, руководства и инструкции

### <a name="concepts"></a>Основные понятия

- [Защита содержимого с помощью динамического шифрования служб мультимедиа](content-protection-overview.md)
- [Проектирование системы для защиты содержимого с несколькими подсистемами DRM и управлением доступом](design-multi-drm-system-with-access-control.md)
- [Шаблон лицензии служб мультимедиа v3 с PlayReady](playready-license-template-overview.md)
- [Обзор служб мультимедиа v3 с использованием шаблона лицензии Widevine](widevine-license-template-overview.md)
- [Требования и конфигурация лицензии Apple FairPlay](fairplay-license-overview.md)
- [Политики потоковой передачи](streaming-policy-concept.md)
- [Политики ключей содержимого](content-key-policy-concept.md)

### <a name="tutorials"></a>Учебники

[Краткое руководство. Шифрование содержимого с помощью портала](encrypt-content-quickstart.md)

### <a name="how-to-guides"></a>Практические руководства

- [Получение ключа подписи из существующей политики](get-content-key-policy-dotnet-howto.md)
- [Автономная потоковая передача FairPlay для iOS с помощью служб мультимедиа v3](offline-fairplay-for-ios.md)
- [Автономная потоковая передача Widevine для Android с помощью служб мультимедиа v3](offline-widevine-for-android.md)
- [Автономная потоковая передача PlayReady для Windows 10 с помощью служб мультимедиа v3](offline-plaready-streaming-for-windows-10.md)

## <a name="samples"></a>Примеры

Кроме того, можно [сравнить код v2 и v3 в примерах кода](migrate-v-2-v-3-migration-samples.md).

## <a name="tools"></a>Инструменты

- [v3 инструмент "Обозреватель служб мультимедиа Azure"](https://github.com/Azure/Azure-Media-Services-Explorer)
- [Версия 2 средства обозревателя служб мультимедиа Azure](https://github.com/Azure/Azure-Media-Services-Explorer/releases/tag/v4.3.15.0)

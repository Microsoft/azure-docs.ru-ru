---
title: Доступ к службам мультимедиа версии 2 и V3 API
description: В этой статье описываются различия в доступе к API между службами мультимедиа Azure версии 2 и 3.
services: media-services
documentationcenter: na
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.devlang: multiple
ms.topic: conceptual
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 03/25/2021
ms.author: inhenkel
ms.openlocfilehash: 5f3c6526139389da3bfdbc3c43cf8b6d2a1dbccf
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105567973"
---
# <a name="api-access-differences-between-azure-media-services-v2-to-v3-api"></a>Различия в доступе API между службами мультимедиа Azure версии 2 и V3 API

![логотип с руководством по миграции](./media/migration-guide/azure-media-services-logo-migration-guide.svg)

<hr color="#5ea0ef" size="10">

![шаги миграции 2](./media/migration-guide/steps-2.svg)

В этой статье описываются различия в доступе к API между службами мультимедиа Azure версии 2 и 3.

## <a name="api-access"></a>Доступ через API

Все учетные записи служб мультимедиа будут иметь доступ к API V3. Однако настоятельно рекомендуется использовать новую учетную запись для разработки перед применением обновленного кода к существующей учетной записи v2. Это связано с тем, что сущности v3 не поддерживают обратную совместимость с v2. Некоторые сущности v2, такие как активы, имеют прямую совместимость с v3.
Можно продолжать использовать существующие учетные записи, если вы не смешивать API v2 и v3, а затем пытаетесь вернуться к версии 2, но это не рекомендуется.

Доступ к API v2 будет доступен до тех пор, пока не будет снят с учета в 2024.

## <a name="create-a-v3-account"></a>Создание учетной записи v3

Во время миграции можно создать учетную запись v3, которая по-прежнему имеет доступ к v2.  Создание учетной записи может осуществляться с помощью следующих действий.

- REST API и более ранние версии
- Установите флажок на портале.

> [!div class="mx-imgBorder"]
> [![Создание учетной записи на портале ](./media/migration-guide/v-3-v-2-access-account-creation-small.png)](./media/migration-guide/v-3-v-2-access-account-creation.png#lightbox)

Для всех .NET, CLI и других пакетов SDK будет использоваться последняя версия API 2020-05-01, поэтому найдите или настройте более старые версии API.

> [!NOTE]
> Новые учетные записи, созданные с помощью API 2020-05-01, не могут использовать API v2.

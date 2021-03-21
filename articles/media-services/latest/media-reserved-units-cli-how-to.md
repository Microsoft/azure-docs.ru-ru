---
title: Масштабирование зарезервированных единиц мультимедиа (Мрус) CLI
description: В этом разделе показано, как использовать CLI для масштабирования обработки мультимедиа с использованием служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/30/2020
ms.author: inhenkel
ms.openlocfilehash: a07c4a20b854e09daf3b320b8c99757ca99b2578
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102213816"
---
# <a name="how-to-scale-media-reserved-units"></a>Как масштабировать зарезервированные единицы мультимедиа

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье показано, как масштабировать зарезервированные единицы мультимедиа (Мрсс) для ускорения кодирования.

## <a name="prerequisites"></a>Предварительные требования

[Создание учетной записи Служб мультимедиа](./create-account-howto.md).

Общие сведения о [зарезервированных единицах мультимедиа](concept-media-reserved-units.md).

## <a name="scale-media-reserved-units-with-cli"></a>Масштабирование зарезервированных единиц кодирования с помощью интерфейса командной строки

Выполните команду `mru`.

Следующая команда [az ams account mru](/cli/azure/ams/account/mru) настраивает зарезервированные единицы мультимедиа в учетной записи amsaccount с использованием параметров **count** и **type**.

```azurecli
az ams account mru set -n amsaccount -g amsResourceGroup --count 10 --type S3
```

## <a name="billing"></a>Выставление счетов

Вы платите за количество минут, в течение которых зарезервированные единицы мультимедиа подготавливаются в вашей учетной записи. Это происходит независимо от того, выполняются ли в вашей учетной записи какие-либо задания. Подробное описание см. в разделе часто задаваемых вопросов на странице [цен на службы мультимедиа](https://azure.microsoft.com/pricing/details/media-services/).   

## <a name="next-step"></a>Следующий шаг

[Анализ видео](analyze-videos-tutorial-with-api.md)

## <a name="see-also"></a>См. также раздел

* [Квоты и ограничения](limits-quotas-constraints.md)

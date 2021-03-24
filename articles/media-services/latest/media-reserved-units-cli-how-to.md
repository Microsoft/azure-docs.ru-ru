---
title: Масштабирование зарезервированных единиц мультимедиа (Мрус) CLI
description: В этом разделе показано, как использовать CLI для масштабирования обработки мультимедиа с использованием служб мультимедиа Azure.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: how-to
ms.date: 03/22/2021
ms.author: inhenkel
ms.openlocfilehash: 06c0c6333b84697415ef598d4c5e853d5c006f08
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870166"
---
# <a name="how-to-scale-media-reserved-units"></a>Как масштабировать зарезервированные единицы мультимедиа

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье показано, как масштабировать зарезервированные единицы мультимедиа (Мрсс) для ускорения кодирования.

> [!WARNING]
> Эта команда больше не будет работать для учетных записей служб мультимедиа, созданных с помощью версии API 2020-05-01 или более поздней. Для этих учетных записей зарезервированные единицы мультимедиа больше не нужны, так как система будет автоматически масштабироваться в соответствии с нагрузкой. Если вы не видите параметр для управления Мрус в портал Azure, вы используете учетную запись, созданную с помощью API 2020-05-01 или более поздней версии.

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

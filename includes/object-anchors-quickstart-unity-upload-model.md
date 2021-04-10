---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 03/02/2021
ms.author: crtreasu
ms.openlocfilehash: d06a6ecd8af16da3e6df21e984fbf6a727fbc27e
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105105358"
---
### <a name="upload-your-model"></a>Добавление модели

Если у вас еще нет модели объектных привязок, создайте ее, следуя инструкциям в статье [Создание модели](../articles/object-anchors/quickstarts/get-started-model-conversion.md). Потом возвращайтесь к этому разделу.

После подключения HoloLens к порталу устройств Windows выполните указанные ниже действия, чтобы добавить модель для использования приложением.

1. На портале устройств Windows перейдите в раздел **Система > Проводник > LocalAppData**. Там вы увидите приложение в соответствующем списке.

    :::image type="content" source="./media/object-anchors-quickstarts-unity/portal-localappdata.png" alt-text="проводник":::

2. Откройте приложение и щелкните папку `LocalState`.

    :::image type="content" source="./media/object-anchors-quickstarts-unity/portal-localstate.png" alt-text="Открывание папки LocalState":::

3. Отправьте файл модели в папку`LocalState`.

    :::image type="content" source="./media/object-anchors-quickstarts/portal-upload-model.png" alt-text="отправка модели на портал":::

    Снова запустите приложение на устройстве HoloLens. Теперь вы можете обнаруживать объекты, соответствующие модели.
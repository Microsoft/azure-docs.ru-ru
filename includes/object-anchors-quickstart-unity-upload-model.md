---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 03/02/2021
ms.author: crtreasu
ms.openlocfilehash: d8dfc3d4b7a8447250481b98c1adadc865a29da1
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102532701"
---
### <a name="upload-your-model"></a>Добавление модели

Если у вас еще нет модели объектных привязок, создайте ее, следуя инструкциям в статье [Создание модели](/azure/object-anchors/quickstarts/get-started-model-conversion). Потом возвращайтесь к этому разделу.

После подключения HoloLens к порталу устройств Windows выполните указанные ниже действия, чтобы добавить модель для использования приложением.

1. На портале устройств Windows перейдите в раздел **Система > Проводник > LocalAppData**. Там вы увидите приложение в соответствующем списке.

    :::image type="content" source="./media/object-anchors-quickstarts-unity/portal-localappdata.png" alt-text="проводник":::

2. Откройте приложение и щелкните папку `LocalState`.

    :::image type="content" source="./media/object-anchors-quickstarts-unity/portal-localstate.png" alt-text="Открывание папки LocalState":::

3. Отправьте файл модели в папку`LocalState`.

    :::image type="content" source="./media/object-anchors-quickstarts/portal-upload-model.png" alt-text="отправка модели на портал":::

    Снова запустите приложение на устройстве HoloLens. Теперь вы можете обнаруживать объекты, соответствующие модели.

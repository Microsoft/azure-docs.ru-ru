---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 04/03/2020
ms.author: crtreasu
ms.openlocfilehash: 43c8c578587c65b6e0317caf77ff2d0604cb4fa3
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "102044672"
---
### <a name="build-and-deploy-the-app"></a>Сборка и развертывание приложения

Откройте файл `.sln`, созданный Unity. Измените конфигурацию сборки на указанную ниже.

:::image type="content" source="./media/object-anchors-quickstarts-unity/aoa-unity-vs-build-config.png" alt-text="конфигурация сборки":::

Далее нужно настроить **IP-адрес удаленного компьютера** для развертывания и отладки приложения.

Щелкните правой кнопкой мыши проект приложения и выберите **Свойства**. На странице свойств выберите **Свойства конфигурации -> Отладка**. Измените значение параметра **Имя компьютера** на IP-адрес устройства HoloLens и нажмите кнопку **Применить**.

:::image type="content" source="./media/object-anchors-quickstarts-unity/aoa-unity-vs-remote-debug.png" alt-text="удаленная отладка":::

Закройте страницу свойств. Нажмите **Удаленный компьютер**. Приложение начнет сборку и развертывание на удаленном устройстве. Убедитесь, что устройство активно.

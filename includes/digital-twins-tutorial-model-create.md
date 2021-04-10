---
author: baanders
description: Include-файл для учебника с инструкциями по созданию модели Azure Digital Twins с помощью командной строки.
ms.topic: include
ms.date: 3/5/2021
ms.author: baanders
ms.openlocfilehash: a94b9304ecd6c6630f6ad45652e76d2879bbc1b8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103463925"
---
1. **Обновите номер версии**, чтобы указать, что вы предоставляете более новую версию этой модели. Для этого измените *1* в конце значения `@id` на *2*. Можно указать любое число больше текущего номера версии.
1. **Измените свойство**. Измените имя свойства `Humidity` на *HumidityLevel* или на другое имя, которое вам больше нравится. Однако не забывайте, что если вы используете имя, отличное от *HumidityLevel*, то должны будете указывать это имя вместо *HumidityLevel* на протяжении всего учебника).
1. **Добавьте свойство**. Под свойством `HumidityLevel`, которое заканчивается на строке 15, вставьте следующий код, чтобы добавить свойство `RoomName` к объекту room.

    :::code language="json" source="~/digital-twins-docs-samples/models/Room.json" range="16-20":::

1. **Добавьте связь**. Под только что добавленным свойством `RoomName` вставьте следующий код, чтобы добавить для этого типа двойника возможность формировать связи *contains* с другими двойниками.

    :::code language="json" source="~/digital-twins-docs-samples/models/Room.json" range="21-24":::

Когда вы закончите, обновленная модель должна выглядеть следующим образом:

:::code language="json" source="~/digital-twins-docs-samples/models/Room.json":::

Обязательно сохраните этот файл, прежде чем двигаться дальше.
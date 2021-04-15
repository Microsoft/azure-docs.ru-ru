---
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.topic: include
ms.date: 10/26/2018
ms.openlocfilehash: 239ea2eb4d5bc8d326d5ca503a18b149252dc1be
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "69558767"
---
> [!div class="op_single_selector"]
> * [Node.js](../articles/iot-hub/iot-hub-node-node-device-management-get-started.md)
> * [.NET](../articles/iot-hub/iot-hub-csharp-csharp-device-management-get-started.md)
> * [Java](../articles/iot-hub/iot-hub-java-java-device-management-getstarted.md)
> * [Python](../articles/iot-hub/iot-hub-python-python-device-management-get-started.md)

Внутренние приложения могут использовать примитивы Центра Интернета вещей, а именно [двойники устройств][lnk-devtwin] и [прямые методы][lnk-c2dmethod], чтобы удаленно запускать и отслеживать на устройствах действия по управлению ими. В этом руководстве показано, как серверное приложение и приложение для устройства взаимодействуют для запуска и отслеживания процесса удаленной перезагрузки устройства с помощью Центра Интернета вещей.

[!INCLUDE [iot-hub-basic](iot-hub-basic-whole.md)]

Прямой метод используется для запуска действий по управлению устройствами (таких как перезагрузка, сброс к параметрам по умолчанию и обновление встроенного ПО) из внутреннего приложения в облаке. Устройство отвечает за следующие действия:

* Обработка запроса метода, отправленного из Центра Интернета вещей.

* Инициализация соответствующего действия на определенном устройстве.

* Предоставление сведений об обновлении состояния в Центр Интернета вещей в *сообщаемых свойствах*.

Вы можете воспользоваться серверным приложением в облаке, чтобы выполнить запросы двойника устройства для получения сведений о ходе выполнения ваших действий по управлению устройствами.

[lnk-devtwin]: ../articles/iot-hub/iot-hub-devguide-device-twins.md
[lnk-c2dmethod]: ../articles/iot-hub/iot-hub-devguide-direct-methods.md

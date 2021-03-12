---
title: Встроенные прямые методы edgeAgent — Azure IoT Edge
description: Мониторинг развертывания IoT Edge и управление им с помощью встроенных прямых методов в модуле среды выполнения агента IoT Edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/02/2020
ms.topic: conceptual
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 52cd7817594c5c2a1d4e3a4ca9c56891df594cd6
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103201113"
---
# <a name="communicate-with-edgeagent-using-built-in-direct-methods"></a>Обмен данными с edgeAgent с помощью встроенных прямых методов

[!INCLUDE [iot-edge-version-all-supported](../../includes/iot-edge-version-all-supported.md)]

Отслеживайте развертывания IoT Edge и управляйте ими с помощью прямых методов, входящих в модуль агента IoT Edge. Прямые методы реализуются на устройстве, а затем могут быть вызваны из облака. Агент IoT Edge содержит прямые методы, помогающие удаленно отслеживать IoT Edge устройства и управлять ими.

Дополнительные сведения о прямых методах, способах их использования и их реализации в собственных модулях см. в разделе [изучение и вызов прямых методов из центра Интернета вещей](../iot-hub/iot-hub-devguide-direct-methods.md).

Имена этих прямых методов обрабатываются без учета регистра.

## <a name="ping"></a>Проверка связи

Метод **ping** полезен для проверки того, работает ли IOT EDGE на устройстве, а также имеет ли устройство открытое подключение к центру Интернета вещей. Используйте этот прямой метод для проверки связи с агентом IoT Edge и получения его состояния. Успешный запрос проверки связи возвращает пустую полезную нагрузку и **"Status": 200**.

Например:

```azurecli
az iot hub invoke-module-method --method-name 'ping' -n <hub name> -d <device name> -m '$edgeAgent'
```

В портал Azure вызовите метод с именем метода `ping` и пустой полезной нагрузкой JSON `{}` .

![Вызов прямого метода "ping" в портал Azure](./media/how-to-edgeagent-direct-method/ping-direct-method.png)

## <a name="restart-module"></a>Перезапустить модуль

Метод **рестартмодуле** позволяет удаленно управлять модулями, работающими на IOT Edge устройстве. Если модуль сообщает о состоянии сбоя или другое неработоспособное поведение, можно запустить агент IoT Edge для его перезапуска. Успешная команда перезапуска возвращает пустую полезную нагрузку и **"Status": 200**.

Метод Рестартмодуле доступен в IoT Edge версии 1.0.9 и более поздних. 

Прямой метод Рестартмодуле можно использовать для любого модуля, работающего на IoT Edge устройстве, включая сам модуль edgeAgent. Однако если вы используете этот прямой метод для завершения работы edgeAgent, вы не получите результат успешного выполнения, так как подключение будет разорвано во время перезапуска модуля.

Например:

```azurecli
az iot hub invoke-module-method --method-name 'RestartModule' -n <hub name> -d <device name> -m '$edgeAgent' --method-payload \
'
    {
        "schemaVersion": "1.0",
        "id": "<module name>"
    }
'
```

В портал Azure вызовите метод с именем метода `RestartModule` и следующими полезными данными JSON:

```json
{
    "schemaVersion": "1.0",
    "id": "<module name>"
}
```

![Вызов прямого метода "Рестартмодуле" в портал Azure](./media/how-to-edgeagent-direct-method/restartmodule-direct-method.png)

## <a name="diagnostic-direct-methods"></a>Прямые методы диагностики

* [Жетмодулелогс](how-to-retrieve-iot-edge-logs.md#retrieve-module-logs): получение журналов модулей в ответе прямого метода.
* [Уплоадмодулелогс](how-to-retrieve-iot-edge-logs.md#upload-module-logs): получение журналов модулей и их отправка в хранилище BLOB-объектов Azure.
* [Уплоадсуппортбундле](how-to-retrieve-iot-edge-logs.md#upload-support-bundle-diagnostics): получение журналов модулей с помощью пакета поддержки и передача ZIP-файла в хранилище BLOB-объектов Azure.
* [Жеттаскстатус](how-to-retrieve-iot-edge-logs.md#get-upload-request-status): Проверьте состояние отправки журналов или запроса поддержки.

Эти прямые методы диагностики доступны в 1.0.10 выпуске.

## <a name="next-steps"></a>Дальнейшие действия

[Свойства двойников модулей агента IoT Edge и центра IoT Edge](module-edgeagent-edgehub.md)

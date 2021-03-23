---
title: Устранение неполадок с микроагентом Defender для Интернета вещей (предварительная версия)
description: Сведения об обработке непредвиденных или необъясненных ошибок.
ms.date: 1/24/2021
ms.topic: reference
ms.openlocfilehash: 51550a4d3e5042fed7cadc4eac10a0074e954f19
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104782458"
---
# <a name="defender-iot-micro-agent-troubleshooting-preview"></a>Устранение неполадок с микроагентом Defender для Интернета вещей (предварительная версия)

В случае непредвиденных или необъясненных ошибок используйте следующие методы устранения неполадок, чтобы попытаться устранить проблемы. При необходимости вы также можете обратиться в службу "защитник Azure для Интернета вещей".   

## <a name="service-status"></a>Состояние службы 

Чтобы просмотреть состояние службы, выполните следующие действия. 

1. Выполните следующую команду.

    ```azurecli
    systemctl status defender-iot-micro-agent.service 
    ```

1. Убедитесь, что служба является стабильной, убедившись в том, что это так `active` , а также в том, что время работоспособности процесса соответствует.

    :::image type="content" source="media/troubleshooting/active-running.png" alt-text="Убедитесь в стабильной работе службы, убедившись, что она активна и подходящая работоспособность.":::

Если служба указана как `inactive` , используйте следующую команду для запуска службы:

```azurecli
systemctl start defender-iot-micro-agent.service 
```

Если время простоя процесса слишком мало, вы узнаете, что происходит сбой службы. Чтобы устранить эту проблему, необходимо проверить журналы.

## <a name="review-logs"></a>Просмотр журналов 

Используйте следующую команду, чтобы убедиться, что служба "Micro Agent IoT" защитника запущена с правами root.

```azurecli
ps -aux | grep " defender-iot-micro-agent"
```

:::image type="content" source="media/troubleshooting/root-privileges.png" alt-text="Убедитесь, что служба защитник для Micro Agent для IoT запущена с правами root.":::

Чтобы просмотреть журналы, используйте следующую команду:  

```azurecli
sudo journalctl -u defender-iot-micro-agent | tail -n 200 
```

Чтобы перезапустить службу, используйте следующую команду: 

```azurecli
sudo systemctl restart defender-iot-micro-agent  
```

## <a name="next-steps"></a>Следующие шаги

Ознакомьтесь с [поддержкой функций и выбытие](edge-security-module-deprecation.md).

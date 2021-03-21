---
title: Аутентификация агентов безопасности (предварительная версия)
titleSuffix: Azure Defender for IoT
description: Выполните проверку подлинности Micro Agent с двумя возможными методами.
author: shhazam-ms
manager: rkarlin
ms.author: shhazam
ms.date: 1/20/2021
ms.topic: conceptual
ms.service: azure
ms.openlocfilehash: 018da32b90c7730f82eaa5aa2cd2b5c7a64719a6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102124590"
---
# <a name="micro-agent-authentication-methods-preview"></a>Методы проверки подлинности Micro Agent (Предварительная версия)

Существует два варианта проверки подлинности с помощью защитника для Micro Agent для IoT. 

- Строка подключения 

- Certificate 

## <a name="authentication-using-a-connection-string"></a>Проверка подлинности с помощью строки подключения 

Чтобы использовать строку подключения, необходимо добавить файл, который использует строку подключения в кодировке UTF-8 в каталоге агента защитника в файле с именем `connection_string.txt` . Например,

```azurecli
echo “<connection string>” > connection_string.txt 
```

После этого перезапустите службу с помощью этой команды.

```azurecli
sudo systemctl restart defender-iot-micro-agent.service
``` 

## <a name="authentication-using-a-certificate"></a>Проверка подлинности с помощью сертификата 


Чтобы выполнить проверку подлинности с помощью сертификата, выполните следующие действия. 

1. Поместите общедоступную часть сертификата с кодировкой PEM в каталог агента защитника в файл с именем `certificate_public.pem` .
1. Поместите закрытый ключ в кодировке PEM в каталог агента защитника в файл с именем `certificate_private.pem` .
1. Поместите соответствующую строку подключения в файл с именем `connection_string.txt` . Например,

    ```azurecli
    HostName=<the host name of the iot hub>;DeviceId=<the id of the device>;ModuleId=<the id of the module>;x509=true 
    ```

    Это действие означает, что агент защитника ожидает, что сертификат будет предоставлен для проверки подлинности. 

1. Перезапустите службу, используя следующий код: 

    ```azurecli
    sudo systemctl restart defender-iot-micro-agent.service 
    ```

## <a name="ensure-the-micro-agent-is-running-correctly"></a>Убедитесь, что Micro Agent работает правильно. 

1. Выполните следующую команду. 
    ```azurecli
    systemctl status defender-iot-micro-agent.service 
    ```
1. Убедитесь, что служба является стабильной, убедившись в том, что она **активна** и что время работоспособности процесса подходит. 

    :::image type="content" source="media/concept-security-agent-authentication/active.png" alt-text="Убедитесь, что служба является стабильной, убедившись в ее активном состоянии.":::

## <a name="next-steps"></a>Дальнейшие действия

Проверьте [уровень безопасности — тесты производительности CIS](concept-security-posture.md).

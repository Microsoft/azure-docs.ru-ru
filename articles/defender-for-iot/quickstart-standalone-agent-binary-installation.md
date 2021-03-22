---
title: Установка микроагента Defender для Интернета вещей
titleSuffix: Azure Defender for IoT
description: Сведения об установке и проверке подлинности микроагента Defender.
author: shhazam-ms
manager: rkarlin
ms.author: shhazam
ms.date: 3/3/2021
ms.topic: quickstart
ms.service: azure
ms.openlocfilehash: ccf28c47e2e1438a141e2497da70d32c1832ddb9
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102120442"
---
# <a name="install-defender-for-iot-micro-agent"></a>Установка микроагента Defender для Интернета вещей 

В этой статье объясняется, как установить микроагент Defender и реализовать для него проверку подлинности.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем установить модуль Defender для Интернета вещей, создайте удостоверение модуля в Центре Интернета вещей. Дополнительные сведения о создании удостоверения модуля см. в статье [Создание двойника модуля микроагента Defender для Интернета вещей](quickstart-create-micro-agent-module-twin.md).

## <a name="install-the-package"></a>Установка пакета

Установите и настройте репозиторий пакетов Майкрософт, выполнив [эти инструкции](/windows-server/administration/linux-package-repository-for-microsoft-software). 

Инструкции для Debian 9 не включают добавление репозитория, поэтому используйте следующие команды: 

```azurecli
curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add - 

sudo apt-get install software-properties-common

sudo apt-add-repository https://packages.microsoft.com/debian/9/multiarch/prod

sudo apt-get update
```

Чтобы установить пакет микроагента Defender в Debian и дистрибутивы Linux на базе Ubuntu, используйте следующую команду:

```azurecli
sudo apt-get install defender-iot-micro-agent 
```

## <a name="micro-agent-authentication-methods"></a>Способы проверки подлинности микроагента 

Существует два способа проверки подлинности микроагента Defender для Интернета вещей: 

- Строка подключения. 

- Сертификат.

### <a name="authenticate-using-a-connection-string"></a>Проверка подлинности с помощью строки подключения

Чтобы реализовать проверку подлинности с помощью строки подключения, сделайте следующее:

1. Добавьте файл с именем `connection_string.txt`, содержащий строку подключения в кодировке UTF-8, в путь каталога агента Defender `/var/defender_iot_micro_agent`, введя следующую команду:

    ```azurecli
    sudo bash -c 'echo "<connection string" > /var/defender_iot_micro_agent/connection_string.txt' 
    ```

    Теперь файл `connection_string.txt` должен находиться по такому пути: `/var/defender_iot_micro_agent/connection_string.txt`.

1. Перезапустите службу с помощью следующей команды:  

    ```azurecli
    sudo systemctl restart defender-iot-micro-agent.service 
    ```

### <a name="authenticate-using-a-certificate"></a>Проверка подлинности с помощью сертификата

Чтобы реализовать проверку подлинности с помощью сертификата, сделайте следующее:

1. Получите сертификат, следуя [этим инструкциям](../iot-hub/iot-hub-security-x509-get-started.md).

1. Поместите открытую часть сертификата в кодировке PEM и закрытый ключ в файлы `certificate_public.pem` и `certificate_private.pem` в каталоге агента Defender. 

1. Поместите соответствующую строку подключения в файл `connection_string.txt`. Строка подключения должна выглядеть следующим образом: 

    `HostName=<the host name of the iot hub>;DeviceId=<the id of the device>;ModuleId=<the id of the module>;x509=true` 

    Эта строка используется для оповещения агента Defender о том, что требуется предоставить сертификат для проверки подлинности. 

1. Перезапустите службу, используя следующую команду:  

    ```azurecli
    sudo systemctl restart defender-iot-micro-agent.service
    ```

### <a name="validate-your-installation"></a>Проверка установки

Чтобы проверить установку, сделайте следующее:

1. Проверьте, правильно ли работает микроагент, выполнив следующую команду:  

    ```azurecli
    systemctl status defender-iot-micro-agent.service
    ```
1. Убедитесь, что служба работает стабильно (состояние службы должно быть `active`) и что время доступности процесса является соответствующим.

    :::image type="content" source="media/quickstart-standalone-agent-binary-installation/active-running.png" alt-text="Проверка того, что служба является стабильной и активной.":::
 
## <a name="testing-the-system-end-to-end"></a>Комплексное тестирование системы 

Вы можете выполнить комплексное тестирование системы, создав файл триггера на устройстве. При базовом сканировании в агенте файл триггера будет определяться как нарушение базового уровня. 

Создайте файл в файловой системе с помощью следующей команды:

```azurecli
sudo touch /tmp/DefenderForIoTOSBaselineTrigger.txt 
```
В центре появится рекомендация касательно сбоя базовой проверки с идентификатором `CceId` CIS-debian-9-DEFENDER_FOR_IOT_TEST_CHECKS-0.0: 

:::image type="content" source="media/quickstart-standalone-agent-binary-installation/validation-failure.png" alt-text="Рекомендация касательно сбоя базовой проверки, возникшего в центре." lightbox="media/quickstart-standalone-agent-binary-installation/validation-failure-expanded.png":::

Рекомендация в центре должна появиться в течение часа. 

## <a name="micro-agent-versioning"></a>Управление версиями микроагента 

Чтобы установить определенную версию микроагента Defender для Интернета вещей, выполните следующую команду: 

```azurecli
sudo apt-get install defender-iot-micro-agent=<version>
```

## <a name="next-steps"></a>Дальнейшие действия

[Создание микроагента Defender с использованием исходного кода](quickstart-building-the-defender-micro-agent-from-source.md)
---
title: Устранение общих проблем с Azure Перцепт DK и IoT Edge
description: Получите советы по устранению некоторых из наиболее распространенных проблем с Azure Перцепт DK.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/18/2021
ms.custom: template-how-to
ms.openlocfilehash: 93812cf2b0db7fc3557e31c8d9e8053831c7b90f
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103011006"
---
# <a name="azure-percept-dk-dev-kit-troubleshooting"></a>Устранение неполадок в Azure Перцепт DK (комплект разработчика)

Общие советы по устранению неполадок в Azure Перцепт DK см. в приведенных ниже руководствах.

## <a name="general-troubleshooting-commands"></a>Общие команды устранения неполадок

Чтобы выполнить эти команды, 
1. Подключение к [Wi-Fi AP пакета разработчика](./quickstart-percept-dk-set-up.md)
1. [Подключение по протоколу SSH к комплекту разработки](./how-to-ssh-into-percept-dk.md)
1. Введите команды в терминале SSH

Чтобы перенаправить выходные данные в txt-файл для дальнейшего анализа, используйте следующий синтаксис:

```console
sudo [command] > [file name].txt
```

После перенаправления выходных данных в txt-файл скопируйте файл на главный компьютер с помощью SCP:

```console
scp [remote username]@[IP address]:[remote file path]/[file name].txt [local host file path]
```

```[local host file path]``` относится к расположению на главном компьютере, куда нужно скопировать txt файл. ```[remote username]``` — Это имя пользователя SSH, выбранное во время [установки](./quickstart-percept-dk-set-up.md). Если вы не настроили вход SSH во время OOBE, имя удаленного пользователя — ```root``` .

Дополнительные сведения о Azure IoT Edge командах см. в [документации по устранению неполадок устройств Azure IOT Edge](https://docs.microsoft.com/azure/iot-edge/troubleshoot).

|Категория:         |Команда:                    |Функция:                  |
|------------------|----------------------------|---------------------------|
|ОС                |```cat /etc/os-release```         |Проверка версии образа Маринер |
|ОС                |```cat /etc/os-subrelease```      |проверить версию производного образа |
|ОС                |```cat /etc/adu-version```        |Проверка версии аду |
|температура;       |```cat /sys/class/thermal/thermal_zone0/temp``` |Проверка температуры DevKit |
|Wi-Fi             |```sudo journalctl -u hostapd.service``` |Проверка журналов Софтап|
|Wi-Fi             |```sudo journalctl -u wpa_supplicant.service``` |Проверка журналов Wi-Fi Services |
|Wi-Fi             |```sudo journalctl -u ztpd.service```  |Проверка Wi-Fi журналов службы подготовки для нулевого сенсорного ввода |
|Wi-Fi             |```sudo journalctl -u systemd-networkd``` |Проверка журналов сетевого стека Маринер |
|Wi-Fi             |```sudo cat /etc/hostapd/hostapd-wlan1.conf``` |Проверка сведений о конфигурации точки доступа WiFi |
|OOBE              |```sudo journalctl -u oobe -b```       |Проверка журналов OOBE |
|Телеметрия         |```sudo azure-device-health-id```      |Поиск уникальных HW_ID телеметрии |
|Azure IoT Edge          |```sudo iotedge check```          |Выполните проверку конфигурации и подключения для распространенных проблем. |
|Azure IoT Edge          |```sudo iotedge logs [container name]``` |Проверка журналов контейнеров, например модулей распознавания речи и концепций |
|Azure IoT Edge          |```sudo iotedge support-bundle --since 1h``` |получение журналов модулей, Azure IoT Edge журналов диспетчера безопасности, журналов обработчика контейнеров, ```iotedge check``` выходных данных JSON и других полезных сведений об отладке за последний час |
|Azure IoT Edge          |```sudo journalctl -u iotedge -f``` |Просмотр журналов Azure IoT Edge диспетчера безопасности |
|Azure IoT Edge          |```sudo systemctl restart iotedge``` |перезапуск управляющей программы Azure IoT Edge Security |
|Azure IoT Edge          |```sudo iotedge list```           |Вывод списка развернутых модулей Azure IoT Edge |
|Другое             |```df [option] [file]```          |Отображение сведений о доступном или общем пространстве в указанных файловых системах |
|Другое             |`ip route get 1.1.1.1`        |Отображение сведений об IP-адресе устройства и интерфейсе |
|Другое             |<code>ip route get 1.1.1.1 &#124; awk '{print $7}'</code> <br> `ifconfig [interface]` |отображать только IP-адрес устройства |


```journalctl```Wi-Fi команды могут быть объединены в следующую одну команду:

```console
sudo journalctl -u hostapd.service -u wpa_supplicant.service -u ztpd.service -u systemd-networkd -b
```

## <a name="docker-troubleshooting-commands"></a>Команды устранения неполадок DOCKER

|Команда:                        |Функция:                  |
|--------------------------------|---------------------------|
|```sudo docker ps``` |[показывает, какие контейнеры выполняются](https://docs.docker.com/engine/reference/commandline/ps/) |
|```sudo docker images``` |[показывает, какие образы находятся на устройстве](https://docs.docker.com/engine/reference/commandline/images/)|
|```sudo docker rmi [image id] -f``` |[Удаляет образ с устройства](https://docs.docker.com/engine/reference/commandline/rmi/) |
|```sudo docker logs -f edgeAgent``` <br> ```sudo docker logs -f [module_name]``` |[принимает журналы контейнеров указанного модуля](https://docs.docker.com/engine/reference/commandline/logs/) |
|```sudo docker image prune``` |[Удаляет все висячие образы](https://docs.docker.com/engine/reference/commandline/image_prune/) |
|```sudo watch docker ps``` <br> ```watch ifconfig [interface]``` |Проверка состояния скачивания контейнера DOCKER |

## <a name="usb-updating"></a>Обновление USB

|Ошибка:                                    |Решение.                                               |
|------------------------------------------|--------------------------------------------------------|
|LIBUSB_ERROR_XXX во время USB флэш-памяти через ууу |Эта ошибка возникает вследствие сбоя подключения USB во время обновления УУУ. Если USB-кабель неправильно подключен к портам USB на компьютере или 10 раз PE, произойдет ошибка этой формы. Попробуйте отключить и повторно подключить оба конца USB-кабеля и жигглинг кабель, чтобы обеспечить безопасное подключение. Это почти всегда решает проблему. |

## <a name="azure-percept-dk-carrier-board-led-states"></a>Индикаторы на плате перевозчика Azure Перцепт DK

На корпусе платной платы имеется три мелких индикатора. Рядом с индикатором 1 печатается значок облака, значок Wi-Fi печатается рядом с ИНДИКАТОРом 2, а рядом с ИНДИКАТОРом 3 выводится значок восклицательного знака. Сведения о каждом состоянии СВЕТОИНДИКАТОРА см. в следующей таблице.

|Светодиодный индикатор             |Область      |Описание                      |
|----------------|-----------|---------------------------------|
|СВЕТОИНДИКАТОР 1 (центр Интернета вещей) |Вкл. (сплошной) |Устройство подключено к центру Интернета вещей. |
|Индикатор 2 (Wi-Fi)   |Замедлит мерцание |Устройство готово к настройке с помощью Wi-Fi Easy Connect и уведомляет его о присутствии с конфигуратором. |
|Индикатор 2 (Wi-Fi)   |Быстрая мигающий |Проверка подлинности прошла успешно, выполняется сопоставление устройства. |
|Индикатор 2 (Wi-Fi)   |Вкл. (сплошной) |Проверка подлинности и сопоставление успешно выполнены; устройство подключено к сети Wi-Fi. |
|СВЕТОИНДИКАТОР 3           |Н/Д         |СВЕТОИНДИКАТОР не используется. |



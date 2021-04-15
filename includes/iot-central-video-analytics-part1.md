---
title: Включить файл
description: включить файл
services: iot-central
author: dominicbetts
ms.service: iot-central
ms.topic: include
ms.date: 10/06/2020
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 2f3e4bf640b8da31a7fa4d818b94b0372d3026b8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96763439"
---
Пример приложения включает в себя два имитированных устройства и один шлюз IoT Edge. В следующих учебниках описаны два подхода, с которыми можно поэкспериментировать, а также сведения о возможностях шлюза.

* Создайте шлюз IoT Edge на виртуальной машине Azure и подключите имитируемую камеру.
* Создайте шлюз IoT Edge на реальном устройстве, например Intel NUC, и подключите реальную камеру.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Создание приложения розничного магазина с помощью шаблона видеоаналитики Azure IoT Central.
> * Настройка параметров приложения
> * Создание шаблона для устройства шлюза IoT Edge
> * Добавление имитированного устройства шлюза в приложение IoT Central.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этой серией учебников вам потребуется следующее:

* Подписка Azure. Если у вас еще нет подписки, создайте ее на [странице регистрации в Azure](https://aka.ms/createazuresubscription);
* Если вы используете реальную камеру, вам потребуется иметь подключение между устройством IoT Edge и камерой, а также канал **с потоковой передачей по протоколу RTSP**.

## <a name="initial-setup"></a>Начальная настройка

В этих учебниках вы обновите и используете несколько файлов конфигурации. Исходные версии этих файлов доступны в репозитории [LVA-gateway](https://github.com/Azure/live-video-analytics/tree/master/ref-apps/lva-edge-iot-central-gateway) на GitHub. Репозиторий содержит текстовый файл [scratchpad](https://github.com/Azure/live-video-analytics/blob/master/ref-apps/lva-edge-iot-central-gateway/setup/Scratchpad.txt), который можно скачать и использовать для записи значений конфигурации из развертываемых служб. Этот файл поможет выполнить дальнейшие действия в учебниках.

Создайте папку с именем *lva-configuration* на локальном компьютере, чтобы сохранить копии этих файлов. Затем щелкните правой кнопкой мыши каждую из следующих ссылок и выберите **Сохранить как**, чтобы сохранить файл в папку *lva-configuration*.

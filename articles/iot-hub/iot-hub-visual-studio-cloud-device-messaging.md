---
title: Использование VS Cloud Explorer для управления обменом сообщениями устройств центра Интернета вещей Azure
description: Узнайте, как использовать Cloud Explorer для Visual Studio для мониторинга сообщений между облаком и устройством в Центре Интернета вещей Azure.
author: shizn
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: xshi
ms.openlocfilehash: c56bb7030b2ebc12e3afc24e2d8cb29ce2dda0bf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74079478"
---
# <a name="use-cloud-explorer-for-visual-studio-to-send-and-receive-messages-between-your-device-and-iot-hub"></a>Использование Cloud Explorer для Visual Studio для обмена сообщениями между устройством и Центром Интернета вещей

![Комплексная схема](./media/iot-hub-visual-studio-cloud-device-messaging/e-to-e-diagram.png)

[Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS) — это полезное расширение для Visual Studio. Оно позволяет просматривать ресурсы Azure, проверять их свойства и выполнять основные действия разработчика в среде Visual Studio. Эта статья посвящена использованию Cloud Explorer для отправки и получения сообщений между устройством и центром.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="what-you-learn"></a>Что вы узнаете

Из этой статьи вы узнаете, как использовать Cloud Explorer для Visual Studio для отслеживания сообщений, отправляемых с устройства в облако, и отправки сообщений из облака на устройство. В сообщениях, отправляемых с устройства в облако, могут содержаться данные датчиков, которые устройство собирает и отправляет в Центр Интернета вещей. Сообщения, отправляемые из облака на устройство, могут содержать команды, которые Центр Интернета вещей отправляет на устройство. Например, для активации светодиодного индикатора, подключенного к устройству.

## <a name="what-you-do"></a>Что нужно сделать

В этой статье вы выполните следующие задачи:

- Мониторинг сообщений, отправляемых с устройства в облако, с помощью Cloud Explorer для Visual Studio.

- Мониторинг сообщений, отправляемых из облака на устройство, с помощью Cloud Explorer для Visual Studio.

## <a name="what-you-need"></a>Что нужно

Вам потребуется сделать следующее:

- Активная подписка Azure.

- Центр Интернета вещей Azure в подписке.

- Microsoft Visual Studio 2017 с обновлением 9 или более поздней версии. В этой статье используется [Visual Studio 2019](https://www.visualstudio.com/vs/).

- Компонент Cloud Explorer из Visual Studio Installer, который по умолчанию выбран в рабочей нагрузке Azure.

## <a name="update-cloud-explorer-to-latest-version"></a>Обновление Cloud Explorer до последней версии

Компонент Cloud Explorer из Visual Studio Installer для Visual Studio 2017 поддерживает мониторинг сообщений, отправляемых с устройства в облако и из облака на устройство. Чтобы использовать Visual Studio 2017, скачайте и установите последнюю версию [Cloud Explorer](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.CloudExplorerForVS).

## <a name="sign-in-to-access-your-hub"></a>Войдите, чтобы получить доступ к концентратору

Чтобы получить доступ к концентратору, выполните следующие действия.

1. В Visual Studio выберите **Просмотреть**  >  **Cloud Explorer** , чтобы открыть Cloud Explorer.

1. Щелкните значок управления учетной записью, чтобы отобразить подписки.

    ![Значок управления учетной записью](media/iot-hub-visual-studio-cloud-device-messaging/account-management-icon.png)

1. Если вы вошли в Azure, ваши учетные записи будут отображаться. Чтобы войти в Azure в первый раз, выберите **Добавить учетную запись**.

1. Выберите подписки Azure, которые вы хотите использовать, и нажмите кнопку **Применить**.

1. Разверните подписку, а затем — **центры Интернета вещей**.  В каждом концентраторе можно увидеть устройства для этого центра.

    ![Список устройств](media/iot-hub-visual-studio-cloud-device-messaging/hub-device-list.png)

## <a name="monitor-device-to-cloud-messages"></a>Отслеживание сообщений, отправляемых с устройства в облако

Чтобы отслеживать сообщения, отправляемые в Центр Интернета вещей с устройства, выполните следующие действия.

1. Щелкните правой кнопкой мыши Центр Интернета вещей или устройство и выберите **Start Monitoring D2C Message** (Начать мониторинг сообщений D2C).

    ![Действие Start Monitoring D2C Message (Начать мониторинг сообщений D2C)](media/iot-hub-visual-studio-cloud-device-messaging/start-monitoring-d2c-message-vs2019.png)

1. Наблюдаемые сообщения отображаются в разделе **выходные данные**.

    ![Результаты мониторинга сообщений D2C](media/iot-hub-visual-studio-cloud-device-messaging/monitor-d2c-message-result-vs2019.png)

1. Чтобы остановить мониторинг, щелкните правой кнопкой мыши любой Центр Интернета вещей или любое устройство и выберите **Stop Monitoring D2C Message** (Остановить мониторинг сообщений D2C).

## <a name="send-cloud-to-device-messages"></a>Отправка сообщений из облака на устройство

Чтобы отправить сообщение из Центра Интернета вещей на устройство, выполните следующие действия.

1. Щелкните правой кнопкой мыши свое устройство и выберите **Send C2D Message** (Отправить сообщение C2D).

1. Введите сообщение в поле ввода.

    ![Действие Send C2D Message (Отправить сообщение C2D)](media/iot-hub-visual-studio-cloud-device-messaging/send-c2d-message-test.png)

    Результаты отображаются в разделе **выходные данные**.

    ![Результаты отправки сообщения C2D](media/iot-hub-visual-studio-cloud-device-messaging/send-c2d-message-result-vs2019.png)

## <a name="next-steps"></a>Дальнейшие действия

Вы узнали, как отслеживать сообщения, отправляемые из устройства Интернета вещей в облако Центра Интернета вещей, и отправлять сообщения из этого облака на устройство.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
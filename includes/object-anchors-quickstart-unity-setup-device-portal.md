---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 04/03/2020
ms.author: crtreasu
ms.openlocfilehash: 81c4e95660eb2875cd1b03bdfa670aeabadedc3f
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105104739"
---
### <a name="set-up-the-windows-device-portal"></a>Настройка портала устройств Windows

Чтобы подключиться к HoloLens по сети Wi-Fi, выполните указанные ниже действия.

1. Сначала [подключите HoloLens к сети Wi-Fi](/hololens/hololens-network).

2. Затем найдите IP-адрес устройства в разделе **Параметры > Сеть и Интернет > Wi-Fi > Дополнительные параметры**.

3. В веб-браузере вашего ПК откройте `https://<YOUR_HOLOLENS_IP_ADDRESS>`. В браузере отобразится следующее сообщение: "Возникла проблема с сертификатом безопасности этого веб-сайта". Это сообщение появляется потому, что выданный порталу устройств сертификат является самозаверяющим. Вы можете проигнорировать эту ошибку сертификата и продолжить работу.

Дополнительные сведения о настройке портала устройств Windows см. [в этой статье](/windows/mixed-reality/using-the-windows-device-portal).
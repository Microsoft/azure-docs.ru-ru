---
author: craigktreasure
ms.service: azure-object-anchors
ms.topic: include
ms.date: 04/03/2020
ms.author: crtreasu
ms.openlocfilehash: 44fd3b2bc89ac53e7a7c0293961098509d3f5c76
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102044692"
---
### <a name="set-up-the-windows-device-portal"></a>Настройка портала устройств Windows

Чтобы подключиться к HoloLens по сети Wi-Fi, выполните указанные ниже действия.

1. Сначала [подключите HoloLens к сети Wi-Fi](https://docs.microsoft.com/hololens/hololens-network).

2. Затем найдите IP-адрес устройства в разделе **Параметры > Сеть и Интернет > Wi-Fi > Дополнительные параметры**.

3. В веб-браузере вашего ПК откройте `https://<YOUR_HOLOLENS_IP_ADDRESS>`. В браузере отобразится следующее сообщение: "Возникла проблема с сертификатом безопасности этого веб-сайта". Это сообщение появляется потому, что выданный порталу устройств сертификат является самозаверяющим. Вы можете проигнорировать эту ошибку сертификата и продолжить работу.

Дополнительные сведения о настройке портала устройств Windows см. [в этой статье](https://docs.microsoft.com/windows/mixed-reality/using-the-windows-device-portal).

---
title: 'Виртуальная глобальная сеть Azure: профили VPN-клиента пользователя'
description: Описывается, как использовать файл профиля клиента.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 02/08/2021
ms.author: cherylmc
ms.openlocfilehash: d83b6ed2ae83db569d3c61e3cf4cd887f875eb25
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99980921"
---
# <a name="working-with-user-vpn-client-profile-files"></a>Работа с файлами профилей VPN-клиента пользователя

Файлы профиля содержат сведения, необходимые для настройки VPN-подключения. Эта статья поможет вам получить и понять сведения, необходимые для профиля клиента VPN пользователя.

## <a name="download-the-profile"></a>Скачивание профиля

Чтобы скачать ZIP-файл профиля клиента, можно выполнить действия, описанные в статье [Загрузка профилей](global-hub-profile.md) .

[!INCLUDE [client profiles](../../includes/vpn-gateway-vwan-vpn-profile-download.md)]

* Папка **OpenVPN** содержит профиль *ovpn*, который необходимо изменить, чтобы добавить ключ и сертификат. Дополнительные сведения см. в разделе [Configure опенвпн Clients](../virtual-wan/howto-openvpn-clients.md#windows).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о VPN-подключении пользователей виртуальной глобальной сети см. [в статье Создание VPN-подключения пользователя](virtual-wan-point-to-site-portal.md).

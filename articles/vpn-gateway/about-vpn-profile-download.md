---
title: VPN-шлюз Azure. О профилях VPN-клиента "точка — сеть"
description: Используйте эту статью, чтобы найти сведения, необходимые для профиля VPN-клиента.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 02/08/2021
ms.author: cherylmc
ms.openlocfilehash: 9bb66363d525648df08f32451842402ad1d0d93b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99979086"
---
# <a name="working-with-p2s-vpn-client-profile-files"></a>Работа с файлами профиля VPN-клиента P2S

Файлы профиля содержат сведения, необходимые для настройки VPN-подключения. Эта статья поможет вам получить и понять сведения, необходимые для профиля VPN-клиента.

## <a name="generate-and-download-profile"></a>Создать и скачать профиль

Вы можете создать файлы конфигурации клиента с помощью PowerShell или с помощью портала Azure. И в первом, и во втором случае возвращается один и тот же ZIP-файл.

### <a name="portal"></a>Портал

1. На портале Azure перейдите в шлюз виртуальной сети, к которой необходимо подключиться.
1. На странице шлюз виртуальной сети выберите Конфигурация " **точка — сеть**".
1. В верхней части страницы Конфигурация "точка — сеть" выберите **скачать VPN-клиент**. Пакет конфигурации клиента создается несколько минут.
1. В браузере появится сообщение о том, что ZIP-файл конфигурации клиента доступен. Он имеет то же имя, что и шлюз. Распакуйте файл. После этого отобразятся папки.

### <a name="powershell"></a>PowerShell

Чтобы создать с помощью PowerShell, можно использовать следующий пример:

1. При создании файлов конфигурации VPN-клиента установите для параметра AuthenticationMethod значение EapTls. Создайте конфигурацию VPN-клиента с помощью следующей команды:

   ```azurepowershell-interactive
   $profile=New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapTls"

   $profile.VPNProfileSASUrl
   ```

1. Скопируйте URL-адрес в браузер, чтобы скачать ZIP-файл, а затем распакуйте файл для просмотра папок.

[!INCLUDE [client profiles](../../includes/vpn-gateway-vwan-vpn-profile-download.md)]

* Папка **OpenVPN** содержит профиль *ovpn*, который необходимо изменить, чтобы добавить ключ и сертификат. Дополнительные сведения см. в статье [Настройка клиентов OpenVPN для VPN-шлюза Azure](vpn-gateway-howto-openvpn-clients.md#windows). Если на VPN-шлюзе выбрана аутентификация Azure AD, то эта папка отсутствует в ZIP-файле. В этом случае перейдите в папку AzureVPN и найдите файл azurevpnconfig.xml.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о [подключениях "точка — сеть"](point-to-site-about.md).

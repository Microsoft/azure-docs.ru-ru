---
title: 'Создание & установка файлов конфигурации VPN-клиента P2S: проверка подлинности сертификата'
titleSuffix: Azure VPN Gateway
description: Создание и установка файлов конфигурации VPN-клиента Windows, Linux, Linux (strongSwan) и macOS X для проверки подлинности с помощью сертификата P2S.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 11/11/2020
ms.author: cherylmc
ms.openlocfilehash: c7b186aa1a6f63b1bc3e9dbefa5001faac967762
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94556170"
---
# <a name="create-and-install-vpn-client-configuration-files-for-native-azure-certificate-authentication-p2s-configurations"></a>Создание и установка файлов конфигурации VPN-клиента для настройки подключений типа "точка — сеть" с использованием собственной аутентификации Azure на основе сертификата

Файлы конфигурации VPN-клиента содержатся в ZIP-файле. Файлы конфигурации предоставляют параметры, необходимые для собственных клиентов Windows, VPN-подключений по протоколу IKEv2 или Linux, для подключения к виртуальной сети по подключениям типа "точка — сеть", которые используют собственную проверку подлинности на основе сертификата Azure.

Файлы конфигурации клиента относятся к конфигурации VPN для виртуальной сети. Если вы создали файлы конфигурации VPN-клиента, а затем внесли изменения в конфигурацию VPN-подключения типа "точка — сеть" (например, изменили тип VPN-протокола или аутентификации), создайте новую конфигурацию VPN-клиента на устройствах пользователей.

* См. дополнительные сведения о [подключениях типа "точка — сеть"](point-to-site-about.md).
* Инструкции касательно OpenVPN см. в статье [Настройка OpenVPN для подключения "точка — сеть" VPN-шлюза Azure (предварительная версия)](vpn-gateway-howto-openvpn.md) и [Настройка клиентов OpenVPN для VPN-шлюза Azure (предварительная версия)](vpn-gateway-howto-openvpn-clients.md).

>[!IMPORTANT]
>[!INCLUDE [TLS](../../includes/vpn-gateway-tls-change.md)]
>

## <a name="generate-vpn-client-configuration-files"></a><a name="generate"></a>Создание файлов конфигурации VPN-клиента

Перед подключением убедитесь, что на устройствах всех пользователей установлены действительные сертификаты. См. инструкции по [установке сертификата клиента](point-to-site-how-to-vpn-client-install-azure-cert.md).

Вы можете создать файлы конфигурации клиента с помощью PowerShell или с помощью портала Azure. И в первом, и во втором случае возвращается один и тот же ZIP-файл. Распакуйте файл. Отобразятся следующие папки:

* **WindowsAmd64** и **WindowsX86**. Эти папки содержат пакеты установщика 32- и 64-разрядной версий Windows. Пакет установщика **WindowsAmd64** предназначен для всех поддерживаемых 64-разрядных клиентов Windows, не только Amd.
* **Generic**. Эта папка содержит общие сведения для создания собственной конфигурации VPN-клиента. Эта папка доступна, если для шлюза настроены протоколы IKEv2 и SSTP+IKEv2. Если настроен только протокол SSTP, эта папка отсутствует.

### <a name="generate-files-using-the-azure-portal"></a><a name="zipportal"></a>Создание файлов с помощью портала Azure

1. На портале Azure перейдите в шлюз виртуальной сети, к которой необходимо подключиться.
1. На странице шлюз виртуальной сети выберите Конфигурация " **точка — сеть**".

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/download-client.png" alt-text="Скачивание VPN-клиента":::
1. В верхней части страницы Конфигурация "точка — сеть" выберите **скачать VPN-клиент**. Пакет конфигурации клиента создается несколько минут.
1. В браузере появится сообщение о том, что ZIP-файл конфигурации клиента доступен. Он имеет то же имя, что и шлюз. Распакуйте файл. После этого отобразятся папки.

### <a name="generate-files-using-powershell"></a><a name="zipps"></a>Создание файлов с помощью PowerShell

1. При создании файлов конфигурации VPN-клиента установите для параметра AuthenticationMethod значение EapTls. Создайте конфигурацию VPN-клиента с помощью следующей команды:

   ```azurepowershell-interactive
   $profile=New-AzVpnClientConfiguration -ResourceGroupName "TestRG" -Name "VNet1GW" -AuthenticationMethod "EapTls"

   $profile.VPNProfileSASUrl
   ```

1. Скопируйте URL-адрес в браузер, чтобы скачать ZIP-файл, а затем распакуйте файл для просмотра папок.

## <a name="windows"></a><a name="installwin"></a>Windows

[!INCLUDE [Windows instructions](../../includes/vpn-gateway-p2s-client-configuration-windows.md)]

## <a name="mac-os-x"></a><a name="installmac"></a>Mac (OS X)

 Необходимо вручную настроить VPN-клиент IKEv2 на каждом компьютере Mac, который будет подключаться к Azure. Azure не предоставляет файл mobileconfig для собственной аутентификации Azure на основе сертификата. В папке **Generic** содержатся все необходимые для конфигурации сведения. Если папки Generic нет в скачанных файлах, вполне вероятно, что IKEv2 не был выбран в качестве типа туннеля. Обратите внимание, что базовый SKU VPN-шлюза не поддерживает IKEv2. Выбрав IKEv2, создайте ZIP-файл еще раз для получения папки Generic.<br>Эта папка содержит следующие файлы:

* **VpnSettings.xml**, который содержит важные параметры, такие как адрес сервера и тип туннеля. 
* **VpnServerRoot. cer**, содержащий корневой сертификат, необходимый для проверки VPN-шлюза Azure во время установки подключения P2S.

Чтобы настроить на устройстве Mac собственный VPN-клиент для аутентификации на основе сертификата, сделайте следующее: Выполните следующие действия на каждом компьютере Mac, который будет подключаться к Azure:

1. Импортируйте корневой сертификат **VpnServerRoot** на компьютер Mac. Для этого скопируйте файл на устройство Mac и дважды щелкните его. Выберите **Добавить** к импорту.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/add-certificate.png" alt-text="Снимок экрана: страница &quot;сертификаты&quot;":::
  
    >[!NOTE]
    >При двойном щелчке сертификата диалоговое окно **добавления** может не отобразиться, но сертификат устанавливается в надлежащее хранилище. Вы можете проверить наличие сертификата в цепочке ключей входа по категории сертификатов.
    >
  
1. Убедитесь, что вы установили сертификат клиента, выданный с использованием корневого сертификата, который вы передали в Azure при настройке параметров подключения типа "точка — сеть". Это не сертификат VPNServerRoot, который вы установили на предыдущем шаге. Этот сертификат клиента требуется для аутентификации. Дополнительную информацию о создании сертификатов см. в разделе [Настройка подключения "точка — сеть" к виртуальной сети с использованием собственной аутентификации Azure на основе сертификата и портала Azure](vpn-gateway-howto-point-to-site-resource-manager-portal.md#generatecert). Инструкции по установке сертификата клиента см. в разделе [Установка сертификата клиента для аутентификации Azure на основе сертификата при подключениях типа "точка — сеть"](point-to-site-how-to-vpn-client-install-azure-cert.md).
1. Откройте диалоговое окно **сети** в разделе " **сетевые настройки** " и выберите **"+"** , чтобы создать профиль подключения VPN-клиента для подключения P2S к виртуальной сети Azure.

   Установите следующие значения: для параметра **Interface** (Интерфейс) — VPN, для **VPN Type** (Тип VPN) — IKEv2. Укажите имя профиля в поле **имя службы** , а затем щелкните **создать** , чтобы создать профиль подключения VPN-клиента.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/network.png" alt-text="На снимке экрана показано окно сети с возможностью выбора интерфейса, выбора типа VPN и ввода имени службы.":::
1. В папке **Generic** из файла **VpnSettings.xml** скопируйте значение тега **VpnServer**. Вставьте это значение в поля профиля **Server Address** (Адрес сервера) и **Remote ID** (Удаленный ИД).

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/server.png" alt-text="На снимке экрана показаны сведения о сервере.":::
1. Выберите **Параметры проверки подлинности** и выберите **сертификат**. Для **Catalina** выберите **нет**, а затем — **сертификат**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/authentication-settings.png" alt-text="На снимке экрана показаны параметры проверки подлинности.":::

   Для Catalina выберите **нет** и **сертификат**. **Выберите** правильный сертификат:
   
   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/catalina.png" alt-text="На снимке экрана показано окно сети, для которого выбран параметр Проверка подлинности и сертификат выбрано.":::

1. Нажмите кнопку **выбрать...** чтобы выбрать сертификат клиента, который будет использоваться для аутентификации. Это сертификат, установленный на шаге 2.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/certificate.png" alt-text="На снимке экрана показано окно сети с параметрами проверки подлинности, где можно выбрать сертификат.":::
1. В окне **Choose An Identity** (Выбор удостоверения) отобразится список доступных сертификатов. Выберите правильный сертификат, а затем нажмите кнопку **продолжить**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/identity.png" alt-text="На снимке экрана показано диалоговое окно Выбор удостоверения, в котором можно выбрать нужный сертификат.":::

1. В поле **Local ID** (Локальный идентификатор) укажите имя сертификата (из шага 6). В нашем примере поисковый запрос будет выглядеть так: `ikev2Client.com`. Затем нажмите кнопку **Применить** , чтобы сохранить изменения.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/apply-connect.png" alt-text="На снимке экрана показано применение.":::
1. В диалоговом окне **сеть** нажмите кнопку **Применить** , чтобы сохранить все изменения. Затем выберите **Connect** (Подключиться), чтобы установить подключение "точка — сеть" к виртуальной сети Azure.

## <a name="linux-strongswan-gui"></a><a name="linuxgui"></a>Linux (графический пользовательский интерфейс strongSwan)

### <a name="install-strongswan"></a><a name="installstrongswan"></a>Установка strongSwan

[!INCLUDE [install strongSwan](../../includes/vpn-gateway-strongswan-install-include.md)]

### <a name="generate-certificates"></a><a name="genlinuxcerts"></a>Создание сертификатов

Если вы еще не создали сертификаты, выполните следующие действия.

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

### <a name="install-and-configure"></a><a name="install"></a>Установка и настройка.

Следующие инструкции были созданы в Ubuntu 18.0.4. Ubuntu 16.0.10 не поддерживает графический пользовательский интерфейс strongSwan. Использовать Ubuntu 16.0.10 можно только с помощью [командной строки](#linuxinstallcli). В зависимости от версии Linux и strongSwan указанные ниже примеры могут отличаться от экранов, которые вы видите.

1. Откройте приложение **Terminal**, чтобы установить **strongSwan** и его Network Manager, выполнив команду из примера.

   ```
   sudo apt install network-manager-strongswan
   ```
1. Выберите **Параметры**, а затем выберите **сеть**. Нажмите **+** кнопку, чтобы создать новое соединение.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/edit-connections.png" alt-text="На снимке экрана показана страница &quot;Сетевые подключения&quot;.":::

1. В меню выберите **IPSec/IKEv2 (strongSwan)** и дважды щелкните.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/add-connection.png" alt-text="На снимке экрана показана страница Добавление VPN.":::

1. На странице **Добавление VPN** добавьте имя VPN-подключения.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/choose-type.png" alt-text="На снимке экрана показан выбор типа подключения.":::
1. Откройте файл **VpnSettings.xml** из папки **Generic** в скачанных файлах конфигурации клиента. Найдите тег с именем **VpnServer** и скопируйте имя, начинающееся с "сертификат azuregateway" и заканчивая на ". cloudapp.NET".

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/vpn-server.png" alt-text="На снимке экрана показана копия данных.":::
1. Вставьте имя в поле **адрес** нового VPN-подключения в разделе **шлюз** . Затем щелкните значок папки в конце поля **Certificate** (Сертификат), перейдите в папку **Generic** и выберите файл **VpnServerRoot**.
1. В разделе **Client** (Клиент) подключения для параметра **Authentication** (Аутентификация) выберите **Certificate/private key** (Сертификат/закрытый ключ). Для параметров **Certificate** (Сертификат ) и **Private key** (Закрытый ключ) выберите созданные ранее сертификат и закрытый ключ. В разделе **Options** (Параметры) выберите **Request an inner IP address** (Запросить внутренний IP-адрес). Затем выберите **Добавить**.

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/ip-request.png" alt-text="На снимке экрана показан запрос внутреннего IP-адреса.":::

1. **Включите подключение.**

   :::image type="content" source="./media/point-to-site-vpn-client-configuration-azure-cert/turn-on.png" alt-text="На снимке экрана показана копия.":::

## <a name="linux-strongswan-cli"></a><a name="linuxinstallcli"></a>Linux (strongSwan CLI)

### <a name="install-strongswan"></a>Установка strongSwan

[!INCLUDE [install strongSwan](../../includes/vpn-gateway-strongswan-install-include.md)]

### <a name="generate-certificates"></a>Создайте сертификаты.

Если вы еще не создали сертификаты, выполните следующие действия.

[!INCLUDE [strongSwan certificates](../../includes/vpn-gateway-strongswan-certificates-include.md)]

### <a name="install-and-configure"></a>Установка и настройка.

1. Загрузите пакет VPN-клиента с портала Azure.
1. Извлеките файл.
1. В **общей** папке скопируйте или переместите **VpnServerRoot. cer** в **/ЕТК/ипсек.д/кацертс**.
1. Скопируйте или переместите **CP Client. p12** в **/ЕТК/ипсек.д/привате/**. Этот файл является сертификатом клиента для VPN-шлюза.
1. Откройте файл **VpnSettings.xml** и скопируйте `<VpnServer>` значение. Это значение потребуется на следующем шаге.
1. Измените значения в приведенном ниже примере, а затем добавьте пример в конфигурацию **/ЕТК/ипсек.конф** .
  
   ```
   conn azure
         keyexchange=ikev2
         type=tunnel
         leftfirewall=yes
         left=%any
         leftauth=eap-tls
         leftid=%client # use the DNS alternative name prefixed with the %
         right= Enter the VPN Server value here# Azure VPN gateway address
         rightid=% # Enter the VPN Server value here# Azure VPN gateway FQDN with %
         rightsubnet=0.0.0.0/0
         leftsourceip=%config
         auto=add
   ```
1. Добавьте следующие значения в **/ЕТК/ипсек.Секретс**.

   ```
   : P12 client.p12 'password' # key filename inside /etc/ipsec.d/private directory
   ```

1. Выполните следующие команды:

   ```
   # ipsec restart
   # ipsec up azure
   ```

## <a name="next-steps"></a>Дальнейшие действия

Вернитесь к исходной статье, из которой вы работали, а затем завершите настройку P2S.

* [Действия по настройке PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md).
* [Портал Azure действия по настройке](vpn-gateway-howto-point-to-site-resource-manager-portal.md).

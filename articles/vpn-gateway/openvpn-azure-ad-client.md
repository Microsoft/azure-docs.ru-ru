---
title: 'VPN-шлюз: VPN-клиент для подключений по протоколу P2S Опенвпн: проверка подлинности Azure AD'
description: Узнайте, как настроить VPN-клиент для подключения к виртуальной сети с помощью VPN типа "точка — сеть" и Azure Active Directory проверки подлинности.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 10/15/2020
ms.author: alzam
ms.openlocfilehash: 02ce8e1809c5dd404e7afa25178acf37e7346cab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102548416"
---
# <a name="azure-active-directory-authentication-configure-a-vpn-client-for-p2s-openvpn-protocol-connections"></a>Azure Active Directory проверка подлинности: Настройка VPN-клиента для подключений по протоколу P2S Опенвпн

Эта статья поможет настроить VPN-клиент для подключения к виртуальной сети с помощью VPN типа "точка — сеть" и Azure Active Directory проверки подлинности. Прежде чем можно будет подключиться и пройти проверку подлинности с помощью Azure AD, необходимо сначала настроить клиент Azure AD. Дополнительные сведения см. в статье [Настройка клиента Azure AD](openvpn-azure-ad-tenant.md).

[!INCLUDE [Windows 10 and OpenVPN note](../../includes/vpn-gateway-openvpn-auth-include.md)]

## <a name="working-with-client-profiles"></a><a name="profile"></a>Работа с профилями клиентов

Для подключения необходимо скачать VPN-клиент Azure и настроить профиль VPN-клиента на каждом компьютере, который хочет подключиться к виртуальной сети. Можно создать клиентский профиль на компьютере, экспортировать его, а затем импортировать на дополнительные компьютеры.

### <a name="to-download-the-azure-vpn-client"></a>Загрузка VPN-клиента Azure

Используйте эту [ссылку](https://go.microsoft.com/fwlink/?linkid=2117554), чтобы скачать VPN-клиент Azure. Убедитесь, что VPN-клиент Azure имеет разрешение на запуск в фоновом режиме. Чтобы проверить или включить разрешение, выполните следующие действия:

1. Последовательно выберите пункты Пуск и параметры > конфиденциальность > фоновые приложения.
2. В разделе фоновые приложения убедитесь, что **приложение разрешить запуск в фоновом режиме** включено.
3. В разделе Выберите приложения **, которые могут** выполняться в фоновом режиме, а затем включите параметры для VPN-клиента Azure.

  ![разрешение](./media/openvpn-azure-ad-client/backgroundpermission.png)

### <a name="to-create-a-certificate-based-client-profile"></a><a name="cert"></a>Создание профиля клиента на основе сертификата

При работе с профилем на основе сертификата убедитесь, что на клиентском компьютере установлены соответствующие сертификаты. Дополнительные сведения о сертификатах см. в разделе [Установка сертификатов клиента](point-to-site-how-to-vpn-client-install-azure-cert.md).

  ![cert](./media/openvpn-azure-ad-client/create/create-cert1.jpg)

### <a name="to-create-a-radius-client-profile"></a><a name="radius"></a>Создание профиля клиента RADIUS

  ![radius](./media/openvpn-azure-ad-client/create/create-radius1.jpg)
  
> [!NOTE]
> Секрет сервера можно экспортировать в профиле VPN-клиента P2S.  Инструкции по экспорту клиентского профиля можно найти [здесь](about-vpn-profile-download.md).
>

### <a name="to-export-and-distribute-a-client-profile"></a><a name="export"></a>Экспорт и распространение клиентского профиля

Если у вас есть рабочий профиль и его необходимо распространить другим пользователям, его можно экспортировать, выполнив следующие действия.

1. Выделите профиль VPN-клиента, который требуется экспортировать, щелкните.. **.**, а затем выберите **Экспорт**.

    ![Снимок экрана, на котором показана страница "клиент VPN Azure" с выбранным многоточием и выделено "Export".](./media/openvpn-azure-ad-client/export/export1.jpg)

2. Выберите расположение, в которое нужно сохранить этот профиль, оставьте имя файла, а затем нажмите кнопку **сохранить** , чтобы сохранить XML-файл.

    ![экспорт](./media/openvpn-azure-ad-client/export/export2.jpg)

### <a name="to-import-a-client-profile"></a><a name="import"></a>Импорт профиля клиента

1. На странице выберите **Import** (Импорт).

    ![Снимок экрана, на котором показана выбранная кнопка "Добавить" и действие "Импорт", выделенное в нижней левой части окна.](./media/openvpn-azure-ad-client/import/import1.jpg)

2. Перейдите к XML-файлу профиля и выберите его. Выбрав файл, выберите **Open** (Открыть).

    ![Снимок экрана, на котором показан выбранный файл профиля x m l.](./media/openvpn-azure-ad-client/import/import2.jpg)

3. Укажите имя профиля и выберите **Save** (Сохранить).

    ![Снимок экрана, на котором показаны выделенные "имя подключения" и выбрана кнопка "Сохранить".](./media/openvpn-azure-ad-client/import/import3.jpg)

4. Выберите **Connect** (Подключиться), чтобы подключиться к VPN.

    ![Снимок экрана, на котором отображается выбранная виртуальная частная сеть и кнопка "подключиться".](./media/openvpn-azure-ad-client/import/import4.jpg)

5. После подключения значок станет зеленым и выдаст **Connected** (Подключено).

    ![импорт](./media/openvpn-azure-ad-client/import/import5.jpg)

### <a name="to-delete-a-client-profile"></a><a name="delete"></a>Удаление профиля клиента

1. Щелкните многоточие рядом с удаляемым клиентским профилем. Затем щелкните **Remove** (Удалить).

    ![Снимок экрана, на котором показаны многоточие и флажок "Удалить".](./media/openvpn-azure-ad-client/delete/delete1.jpg)

2. Выберите **Remove** (Удалить), чтобы выполнить удаление.

    ![удалить](./media/openvpn-azure-ad-client/delete/delete2.jpg)

## <a name="create-a-connection"></a><a name="connection"></a>Создание подключения

1. На странице выберите **+** , а затем **+ Добавить**.

    ![Снимок экрана, на котором показана выбранная кнопка "Добавить".](./media/openvpn-azure-ad-client/create/create1.jpg)

2. Заполните сведения о подключении. Если вы не уверены в этих значениях, обратитесь к администратору. После заполнения значений выберите **сохранить**.

    ![Снимок экрана, на котором показаны выделенные свойства VPN-подключения и выбрана кнопка "Сохранить".](./media/openvpn-azure-ad-client/create/create2.jpg)

3. Выберите **Connect** (Подключиться), чтобы подключиться к VPN.

    ![Снимок экрана, на котором отображается выбранная кнопка "подключить".](./media/openvpn-azure-ad-client/create/create3.jpg)

4. Выберите правильные учетные данные, а затем нажмите кнопку **продолжить**.

    ![Снимок экрана, на котором показаны примеры учетных данных и выбрана кнопка "продолжить".](./media/openvpn-azure-ad-client/create/create4.jpg)

5. После успешного подключения значок будет зеленым и, например, **подключенным**.

    ![подключение](./media/openvpn-azure-ad-client/create/create5.jpg)

### <a name="to-connect-automatically"></a><a name="autoconnect"></a>Автоматическое подключение

Эти действия помогут настроить подключение для автоматического подключения с параметром Always on.

1. На домашней странице VPN-клиента выберите **параметры VPN**.

    ![Снимок экрана домашней страницы VPN с выбранным параметром "параметры VPN".](./media/openvpn-azure-ad-client/auto/auto1.jpg)

2. Выберите **Да** в диалоговом окне переключение приложений.

    ![Снимок экрана: "вы хотели переключать приложения?" диалоговое окно с выбранной кнопкой "Да".](./media/openvpn-azure-ad-client/auto/auto2.jpg)

3. Убедитесь, что подключение, которое вы хотите установить, еще не подключено, затем выделите профиль и установите флажок " **автоматически устанавливать подключение** ".

    ![Снимок экрана окна "Параметры" с установленным флажком "подключить автоматически".](./media/openvpn-azure-ad-client/auto/auto3.jpg)

4. Выберите **Подключиться** , чтобы инициировать VPN-подключение.

    ![auto](./media/openvpn-azure-ad-client/auto/auto4.jpg)

## <a name="diagnose-connection-issues"></a><a name="diagnose"></a>Diagnose connection issues (Диагностика проблем с подключением)

1. Для диагностики проблем с подключением можно использовать средство **Diagnose** (Диагностика). Щелкните **...** рядом с VPN-подключением, которое нужно диагностировать, чтобы открыть меню. Затем выберите **Diagnose** (Диагностика).

    ![Снимок экрана с многоточием и "Диагностика выбрана".](./media/openvpn-azure-ad-client/diagnose/diagnose1.jpg)

2. На странице **Connection Properties** (Свойства подключения) выберите **Run Diagnosis** (Выполнить диагностику).

    ![Снимок экрана, на котором показана страница "Свойства подключения" с выбранным параметром "Диагностика запуска".](./media/openvpn-azure-ad-client/diagnose/diagnose2.jpg)

3. Войдите с помощью своих учетных данных.

    ![Снимок экрана, на котором показано диалоговое окно "вход в системе" с выбранной рабочей или учебной учетной записью.](./media/openvpn-azure-ad-client/diagnose/diagnose3.jpg)

4. Просмотр результатов диагностики.

    ![diagnose](./media/openvpn-azure-ad-client/diagnose/diagnose4.jpg)

## <a name="faq"></a>Вопросы и ответы

### <a name="is-the-azure-vpn-client-supported-with-windows-fips-mode"></a>Поддерживается ли VPN-клиент Azure в режиме Windows FIPS?

Да, с исправлением [KB4577063](https://support.microsoft.com/help/4577063/windows-10-update-kb4577063) .

### <a name="how-do-i-add-dns-suffixes-to-the-vpn-client"></a>Разделы справки добавить DNS-суффиксы к VPN-клиенту?

Вы можете изменить скачанный XML-файл профиля и добавить **\<dnssuffixes> \<dnssufix> \</dnssufix> \</dnssuffixes>** Теги.

```
<azvpnprofile>
<clientconfig>

    <dnssuffixes>
          <dnssuffix>.mycorp.com</dnssuffix>
          <dnssuffix>.xyz.com</dnssuffix>
          <dnssuffix>.etc.net</dnssuffix>
    </dnssuffixes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-add-custom-dns-servers-to-the-vpn-client"></a>Разделы справки добавить пользовательские DNS-серверы в VPN-клиент?

Вы можете изменить скачанный XML-файл профиля и добавить **\<dnsservers> \<dnsserver> \</dnsserver> \</dnsservers>** Теги.

```
<azvpnprofile>
<clientconfig>

    <dnsservers>
        <dnsserver>x.x.x.x</dnsserver>
        <dnsserver>y.y.y.y</dnsserver>
    </dnsservers>
    
</clientconfig>
</azvpnprofile>
```

> [!NOTE]
> Клиент Azure AD Опенвпн использует записи DNS таблица политики разрешения имен (NRPT). Это означает, что DNS-серверы не будут перечислены в выходных данных `ipconfig /all` . Чтобы подтвердить использование параметров DNS, обратитесь к разделу [Get-днсклиентнрптполици](/powershell/module/dnsclient/get-dnsclientnrptpolicy) в PowerShell.
>

### <a name="how-do-i-add-custom-routes-to-the-vpn-client"></a>Разделы справки добавить настраиваемые маршруты к VPN-клиенту?

Вы можете изменить скачанный XML-файл профиля и добавить **\<includeroutes> \<route> \<destination> \<mask> \</destination> \</mask> \</route> \</includeroutes>** Теги.

```
<azvpnprofile>
<clientconfig>

    <includeroutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </includeroutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="how-do-i-block-exclude-routes-from-the-vpn-client"></a>Разделы справки блокировать (исключить) маршруты от VPN-клиента?

Вы можете изменить скачанный XML-файл профиля и добавить **\<excluderoutes> \<route> \<destination> \<mask> \</destination> \</mask> \</route> \</excluderoutes>** Теги.

```
<azvpnprofile>
<clientconfig>

    <excluderoutes>
        <route>
            <destination>x.x.x.x</destination><mask>24</mask>
        </route>
    </excluderoutes>
    
</clientconfig>
</azvpnprofile>
```

### <a name="can-i-import-the-profile-from-a-command-line-prompt"></a>Можно ли импортировать профиль из командной строки?

Вы можете импортировать профиль из командной строки, поместив скачанный файл **azurevpnconfig.xml** в папку **%усерпрофиле%\аппдата\локал\паккажес\ Microsoft.AzureVpn_8wekyb3d8bbwe \локалстате** и выполнив следующую команду:

```
azurevpn -i azurevpnconfig.xml 
```
чтобы принудительно выполнить импорт, используйте параметр **-f** .


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Создание клиента Azure Active Directory для P2S открытых VPN-подключений, использующих проверку подлинности Azure AD](openvpn-azure-ad-tenant.md).
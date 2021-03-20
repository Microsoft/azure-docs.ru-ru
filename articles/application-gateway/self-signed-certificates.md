---
title: Создание самозаверяющего сертификата с помощью пользовательского корневого ЦС
titleSuffix: Azure Application Gateway
description: Узнайте, как создать самозаверяющий сертификат Шлюза приложений Azure с помощью пользовательского корневого ЦС.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 07/23/2019
ms.author: victorh
ms.openlocfilehash: e60aa9f072a447af97aa7cc66534e6e893fdbcf6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93396946"
---
# <a name="generate-an-azure-application-gateway-self-signed-certificate-with-a-custom-root-ca"></a>Создание самозаверяющего сертификата Шлюза приложений Azure с помощью пользовательского корневого ЦС

В SKU Шлюза приложений версии 2 используются доверенные корневые сертификаты для поддержки внутренних серверов. То есть сертификаты аутентификации, которые были необходимы в SKU версии 1, больше не нужны. *Корневой сертификат* — это закодированный в Base64 корневой сертификат формата X.509(.CER) из внутреннего сервера сертификатов. Он определяет корневой центр сертификации (ЦС), выдавший сертификат сервера. Затем сертификат сервера используется для обмена данными по протоколу TLS/SSL.

Шлюз приложений доверяет сертификату вашего веб-сайта по умолчанию, если он подписан известным центром сертификации (например, GoDaddy или DigiCert). В этом случае вам не нужно явно загружать корневой сертификат. Дополнительные сведения см. в статье [Общие сведения о завершении TLS и сквозном подключении TLS с помощью Шлюза приложений](ssl-overview.md). Но если у вас есть среда разработки и/или тестирования и вы не хотите приобретать проверенный сертификат, подписанный центром сертификации, можно создать собственный пользовательский ЦС и создать самозаверяющий сертификат. 

> [!NOTE]
> Самозаверяющие сертификаты не являются доверенными по умолчанию и могут быть сложными в обслуживании. Кроме того, они могут использовать устаревшие хэши и комплекты шифров, которые могут быть ненадежными. Для повышения безопасности приобретите сертификат, подписанный известным центром сертификации.

В этой статье раскрываются следующие темы:

- создание пользовательского центра сертификации;
- создание самозаверяющего сертификата, подписанного пользовательским центром сертификации;
- передача самозаверяющего корневого сертификата в Шлюз приложений для аутентификации внутреннего сервера.

## <a name="prerequisites"></a>Предварительные требования

- **[OpenSSL](https://www.openssl.org/) на компьютере под управлением Windows или Linux.** 

   Хотя для управления сертификатами могут быть доступны и другие инструменты, в этом учебнике используется OpenSSL. OpenSSL поставляется со многими дистрибутивами Linux, например Ubuntu.
- **Веб-сервер.**

   Например, Apache, IIS или NGINX для тестирования сертификатов.

- **SKU Шлюза приложений версии 2.**
   
  Если у вас нет шлюза приложений, см. статью [Краткое руководство. Направление веб-трафика с помощью Шлюза приложений Azure на портале Azure](quick-create-portal.md).

## <a name="create-a-root-ca-certificate"></a>Создание сертификата корневого ЦС

Создайте сертификат корневого ЦС с помощью OpenSSL.

### <a name="create-the-root-key"></a>Создание корневого ключа

1. Войдите на компьютер, где установлен OpenSSL, и выполните следующую команду. В результате будет создан ключ, защищенный паролем.

   ```
   openssl ecparam -out contoso.key -name prime256v1 -genkey
   ```
1. При появлении запроса введите надежный пароль. Например, по меньшей мере девять символов с прописными буквами, строчными буквами, числами и символами.

### <a name="create-a-root-certificate-and-self-sign-it"></a>Создание корневого сертификата и его самозаверение

1. Используйте следующие команды для создания запроса на подпись сертификата (CSR) и сертификата.

   ```
   openssl req -new -sha256 -key contoso.key -out contoso.csr

   openssl x509 -req -sha256 -days 365 -in contoso.csr -signkey contoso.key -out contoso.crt
   ```
   Предыдущие команды создают корневой сертификат. Вы будете использовать его для подписи сертификата сервера.

1. При появлении запроса введите пароль для корневого ключа и сведения об организации для пользовательского ЦС, такие как страна или регион, штат, организация, подразделение и полное доменное имя (домен издателя).

   ![Создание корневого сертификата](media/self-signed-certificates/root-cert.png)

## <a name="create-a-server-certificate"></a>Создание сертификата сервера

Далее предстоит создать сертификат сервера с помощью OpenSSL.

### <a name="create-the-certificates-key"></a>Создание ключа сертификата

Используйте следующую команду, чтобы создать ключ для сертификата сервера.

   ```
   openssl ecparam -out fabrikam.key -name prime256v1 -genkey
   ```

### <a name="create-the-csr-certificate-signing-request"></a>Создание запроса на подпись сертификата (CSR)

CSR — это открытый ключ, предоставляемый ЦС при запросе сертификата. ЦС выдает сертификат для этого конкретного запроса.

> [!NOTE]
> Общее имя (CN) для сертификата сервера должно отличаться от имени домена издателя. Например, в данном случае общее имя издателя `www.contoso.com`, а сертификата сервера — `www.fabrikam.com`.


1. Используйте следующую команду, чтобы создать CSR:

   ```
   openssl req -new -sha256 -key fabrikam.key -out fabrikam.csr
   ```

1. При появлении запроса введите пароль для корневого ключа и сведения об организации для пользовательского ЦС: страна или регион, штат, организация, подразделение и полное доменное имя. Это домен веб-сайта — он должен отличаться от издателя.

   ![Сертификат сервера](media/self-signed-certificates/server-cert.png)

### <a name="generate-the-certificate-with-the-csr-and-the-key-and-sign-it-with-the-cas-root-key"></a>Создание сертификата с использованием CSR и ключа, а также подписывание сертификата с помощью корневого ключа ЦС

1. Используйте следующую команду, чтобы создать сертификат:

   ```
   openssl x509 -req -in fabrikam.csr -CA  contoso.crt -CAkey contoso.key -CAcreateserial -out fabrikam.crt -days 365 -sha256
   ```
### <a name="verify-the-newly-created-certificate"></a>Проверка созданного сертификата

1. Используйте следующую команду, чтобы напечатать выходные данные CRT-файла и проверить его содержимое:

   ```
   openssl x509 -in fabrikam.crt -text -noout
   ```

   ![Проверка сертификата](media/self-signed-certificates/verify-cert.png)

1. Проверьте свой каталог и убедитесь, что в нем есть следующие файлы:

   - contoso.crt;
   - contoso.key;
   - fabrikam.crt;
   - fabrikam.key.

## <a name="configure-the-certificate-in-your-web-servers-tls-settings"></a>Настройка сертификата в параметрах TLS веб-сервера

На веб-сервере настройте TLS с помощью файлов fabrikam.crt и brikam.key. Если веб-сервер не может принять два файла, их можно объединить в один файл PEM или PFX с помощью команд OpenSSL.

### <a name="iis"></a>IIS

Инструкции по импорту сертификата и его передаче в качестве сертификата сервера в службах IIS см. в статье [Практическое руководство. Импорт и установка сертификата на веб-сервере в Windows Server 2003](https://support.microsoft.com/help/816794/how-to-install-imported-certificates-on-a-web-server-in-windows-server).

Инструкции по привязке TLS см. в статье [Как настроить SSL для IIS 7](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#create-an-ssl-binding-1).

### <a name="apache"></a>Apache

Ниже приведен пример конфигурации [виртуального узла, настроенного для SSL](https://cwiki.apache.org/confluence/display/HTTPD/NameBasedSSLVHosts) в Apache:

```
<VirtualHost www.fabrikam:443>
      DocumentRoot /var/www/fabrikam
      ServerName www.fabrikam.com
      SSLEngine on
      SSLCertificateFile /home/user/fabrikam.crt
      SSLCertificateKeyFile /home/user/fabrikam.key
</VirtualHost>
```

### <a name="nginx"></a>NGINX

Ниже приведен пример конфигурации [серверного блока NGINX](https://nginx.org/docs/http/configuring_https_servers.html) с конфигурацией TLS:

![NGINX с использованием TLS](media/self-signed-certificates/nginx-ssl.png)

## <a name="access-the-server-to-verify-the-configuration"></a>Доступ к серверу для проверки конфигурации

1. Добавьте корневой сертификат в доверенное корневое хранилище компьютера. Войдя на веб-сайт, убедитесь, что в браузере отображается вся цепочка сертификатов.

   ![Доверенные корневые сертификаты](media/self-signed-certificates/trusted-root-cert.png)

   > [!NOTE]
   > Предполагается, что служба доменных имен (DNS) настроена таким образом, чтобы сопоставлять имя веб-сервера (в данном примере — www.fabrikam.com) с IP-адресом вашего сервера. В противном случае можно изменить файл [hosts](https://answers.microsoft.com/en-us/windows/forum/all/how-to-edit-host-file-in-windows-10/7696f204-2aaf-4111-913b-09d6917f7f3d), чтобы разрешить имя.
1. Перейдите на свой веб-сайт и щелкните значок замка в поле адреса браузера, чтобы проверить сведения о сайте и сертификате.

## <a name="verify-the-configuration-with-openssl"></a>Проверка конфигурации с помощью OpenSSL

Для проверки сертификата можно также использовать OpenSSL.

```
openssl s_client -connect localhost:443 -servername www.fabrikam.com -showcerts
```

![Проверка сертификата с помощью OpenSSL](media/self-signed-certificates/openssl-verify.png)

## <a name="upload-the-root-certificate-to-application-gateways-http-settings"></a>Передача корневого сертификата в параметры HTTP Шлюза приложений

Чтобы передать сертификат в Шлюз приложений, необходимо экспортировать CRT-сертификат в формат CER с кодировкой Base-64. Поскольку CRT-файл уже содержит открытый ключ в формате Base-64, просто измените расширение файла с CRT на CER. 

### <a name="azure-portal"></a>Портал Azure

Чтобы отправить доверенный корневой сертификат с портала, выберите **Параметры HTTP**, а затем выберите протокол **HTTPS**.

![Добавление сертификата с помощью портала](media/self-signed-certificates/portal-cert.png)

### <a name="azure-powershell"></a>Azure PowerShell

Для отправки корневого сертификата можно также использовать Azure CLI или Azure PowerShell. Следующий код является примером для Azure PowerShell.

> [!NOTE]
> В следующем примере в шлюз приложений добавляется доверенный корневой сертификат, создается новый параметр HTTP и добавляется новое правило (предполагается, что серверный пул и прослушиватель уже существуют).

```azurepowershell
## Add the trusted root certificate to the Application Gateway

$gw=Get-AzApplicationGateway -Name appgwv2 -ResourceGroupName rgOne

Add-AzApplicationGatewayTrustedRootCertificate `
   -ApplicationGateway $gw `
   -Name CustomCARoot `
   -CertificateFile "C:\Users\surmb\Downloads\contoso.cer"

$trustedroot = Get-AzApplicationGatewayTrustedRootCertificate `
   -Name CustomCARoot `
   -ApplicationGateway $gw

## Get the listener, backend pool and probe

$listener = Get-AzApplicationGatewayHttpListener `
   -Name basichttps `
   -ApplicationGateway $gw

$bepool = Get-AzApplicationGatewayBackendAddressPool `
  -Name testbackendpool `
  -ApplicationGateway $gw

Add-AzApplicationGatewayProbeConfig `
  -ApplicationGateway $gw `
  -Name testprobe `
  -Protocol Https `
  -HostName "www.fabrikam.com" `
  -Path "/" `
  -Interval 15 `
  -Timeout 20 `
  -UnhealthyThreshold 3

$probe = Get-AzApplicationGatewayProbeConfig `
  -Name testprobe `
  -ApplicationGateway $gw

## Add the configuration to the HTTP Setting and don't forget to set the "hostname" field
## to the domain name of the server certificate as this will be set as the SNI header and
## will be used to verify the backend server's certificate. Note that TLS handshake will
## fail otherwise and might lead to backend servers being deemed as Unhealthy by the probes

Add-AzApplicationGatewayBackendHttpSettings `
  -ApplicationGateway $gw `
  -Name testbackend `
  -Port 443 `
  -Protocol Https `
  -Probe $probe `
  -TrustedRootCertificate $trustedroot `
  -CookieBasedAffinity Disabled `
  -RequestTimeout 20 `
  -HostName www.fabrikam.com

## Get the configuration and update the Application Gateway

$backendhttp = Get-AzApplicationGatewayBackendHttpSettings `
  -Name testbackend `
  -ApplicationGateway $gw

Add-AzApplicationGatewayRequestRoutingRule `
  -ApplicationGateway $gw `
  -Name testrule `
  -RuleType Basic `
  -BackendHttpSettings $backendhttp `
  -HttpListener $listener `
  -BackendAddressPool $bepool

Set-AzApplicationGateway -ApplicationGateway $gw 
```

### <a name="verify-the-application-gateway-backend-health"></a>Проверка работоспособности серверной части шлюза приложений

1. Выберите представление **Оценка работоспособности серверной части** вашего шлюза приложений, чтобы проверить работоспособность пробы.
1. Для пробы HTTPS должно отображаться состояние **Работоспособна**.

![Проба HTTPS](media/self-signed-certificates/https-probe.png)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше о протоколе SSL/TLS в Шлюзе приложений, см. статью [Общие сведения о завершении TLS и сквозном подключении TLS с помощью Шлюза приложений](ssl-overview.md).
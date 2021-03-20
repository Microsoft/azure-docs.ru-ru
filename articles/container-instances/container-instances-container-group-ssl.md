---
title: Включение TLS с помощью контейнера расширения
description: Создайте конечную точку SSL или TLS для группы контейнеров, работающей в службе "экземпляры контейнеров Azure", запустив nginx в контейнере расширения.
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: 6587a84e7cbe655c509f74e9e39e93010e7058be
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96558085"
---
# <a name="enable-a-tls-endpoint-in-a-sidecar-container"></a>Включение конечной точки TLS в контейнере расширения

В этой статье показано, как создать [группу контейнеров](container-instances-container-groups.md) с контейнером приложения и контейнером расширения, в котором работает поставщик TLS/SSL. Настроив группу контейнеров с отдельной конечной точкой TLS, вы включите TLS-подключения для приложения, не изменяя код приложения.

Вы настраиваете пример группы контейнеров, состоящей из двух контейнеров:
* Контейнер приложения, в котором выполняется простое веб-приложение, использующее общедоступное изображение Microsoft [ACI-HelloWorld](https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld) . 
* Контейнер расширения, в котором выполняется общедоступный образ [nginx](https://hub.docker.com/_/nginx) , настроенный для использования TLS. 

В этом примере группа контейнеров предоставляет только порт 443 для nginx с его общедоступным IP-адресом. Nginx направляет запросы HTTPS в сопутствующее веб-приложение, которое прослушивает внутренний порт 80. Вы можете адаптировать пример для приложений контейнера, прослушиваемых другими портами. 

Другие подходы к включению TLS в группе контейнеров см. в разделе [дальнейшие действия](#next-steps) .

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется версия 2.0.55 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата

Чтобы настроить nginx в качестве поставщика TLS, требуется сертификат TLS/SSL. В этой статье показано, как создать и настроить самозаверяющий сертификат TLS/SSL. В рабочих сценариях следует получить сертификат из центра сертификации.

Чтобы создать самозаверяющий сертификат TLS/SSL, используйте средство [OpenSSL](https://www.openssl.org/) , доступное в Azure Cloud Shell и многих дистрибутивах Linux, или используйте сравнимое клиентское средство в вашей операционной системе.

Сначала создайте запрос на сертификат (CSR-файл) в локальном рабочем каталоге:

```console
openssl req -new -newkey rsa:2048 -nodes -keyout ssl.key -out ssl.csr
```

Следуйте инструкциям на экране, чтобы добавить идентификационные данные. В поле Common Name (общее имя) введите имя узла, связанное с сертификатом. При запросе пароля нажмите клавишу ВВОД без ввода текста, чтобы пропустить Добавление пароля.

Выполните следующую команду, чтобы создать самозаверяющий сертификат (CRT-файл) из запроса на сертификат. Пример:

```console
openssl x509 -req -days 365 -in ssl.csr -signkey ssl.key -out ssl.crt
```

Теперь в каталоге должны отображаться три файла: запрос сертификата ( `ssl.csr` ), закрытый ключ ( `ssl.key` ) и самозаверяющий сертификат ( `ssl.crt` ). Вы используете `ssl.key` и `ssl.crt` в последующих шагах.

## <a name="configure-nginx-to-use-tls"></a>Настройка Nginx для использования TLS

### <a name="create-nginx-configuration-file"></a>Создать файл конфигурации nginx

В этом разделе вы создадите файл конфигурации для nginx, чтобы использовать TLS. Начните с копирования следующего текста в новый файл с именем `nginx.conf` . В Azure Cloud Shell для создания файла в рабочей папке можно применить Visual Studio Code.

```console
code nginx.conf
```

В `location` Убедитесь, что для `proxy_pass` приложения задан правильный порт. В этом примере мы устанавливаем порт 80 для `aci-helloworld` контейнера.

```console
# nginx Configuration File
# https://wiki.nginx.org/Configuration

# Run as a less privileged user for security reasons.
user nginx;

worker_processes auto;

events {
  worker_connections 1024;
}

pid        /var/run/nginx.pid;

http {

    #Redirect to https, using 307 instead of 301 to preserve post data

    server {
        listen [::]:443 ssl;
        listen 443 ssl;

        server_name localhost;

        # Protect against the BEAST attack by not using SSLv3 at all. If you need to support older browsers (IE6) you may need to add
        # SSLv3 to the list of protocols below.
        ssl_protocols              TLSv1.2;

        # Ciphers set to best allow protection from Beast, while providing forwarding secrecy, as defined by Mozilla - https://wiki.mozilla.org/Security/Server_Side_TLS#Nginx
        ssl_ciphers                ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:ECDHE-RSA-RC4-SHA:ECDHE-ECDSA-RC4-SHA:AES128:AES256:RC4-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!3DES:!MD5:!PSK;
        ssl_prefer_server_ciphers  on;

        # Optimize TLS/SSL by caching session parameters for 10 minutes. This cuts down on the number of expensive TLS/SSL handshakes.
        # The handshake is the most CPU-intensive operation, and by default it is re-negotiated on every new/parallel connection.
        # By enabling a cache (of type "shared between all Nginx workers"), we tell the client to re-use the already negotiated state.
        # Further optimization can be achieved by raising keepalive_timeout, but that shouldn't be done unless you serve primarily HTTPS.
        ssl_session_cache    shared:SSL:10m; # a 1mb cache can hold about 4000 sessions, so we can hold 40000 sessions
        ssl_session_timeout  24h;


        # Use a higher keepalive timeout to reduce the need for repeated handshakes
        keepalive_timeout 300; # up from 75 secs default

        # remember the certificate for a year and automatically connect to HTTPS
        add_header Strict-Transport-Security 'max-age=31536000; includeSubDomains';

        ssl_certificate      /etc/nginx/ssl.crt;
        ssl_certificate_key  /etc/nginx/ssl.key;

        location / {
            proxy_pass http://localhost:80; # TODO: replace port if app listens on port other than 80
            
            proxy_set_header Connection "";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $remote_addr;
        }
    }
}
```

### <a name="base64-encode-secrets-and-configuration-file"></a>Секреты кодировки Base64 и файла конфигурации

Base64 кодирует файл конфигурации nginx, сертификат TLS/SSL и ключ TLS. В следующем разделе вы вводите закодированное содержимое в файл YAML, используемый для развертывания группы контейнеров.

```console
cat nginx.conf | base64 > base64-nginx.conf
cat ssl.crt | base64 > base64-ssl.crt
cat ssl.key | base64 > base64-ssl.key
```

## <a name="deploy-container-group"></a>Развернуть группу контейнеров

Теперь разверните группу контейнеров, указав конфигурации контейнеров в [файле YAML](container-instances-multi-container-yaml.md).

### <a name="create-yaml-file"></a>Создание файла YAML

Скопируйте следующий YAML в новый файл с именем `deploy-aci.yaml` . В Azure Cloud Shell для создания файла в рабочей папке можно применить Visual Studio Code.

```console
code deploy-aci.yaml
```

Введите содержимое файлов в кодировке Base64, как указано в разделе `secret` . Например, для `cat` просмотра содержимого каждого из файлов в кодировке Base64. Во время развертывания эти файлы добавляются в [секретный том](container-instances-volume-secret.md) в группе контейнеров. В этом примере секретный том монтируется в контейнер nginx.

```YAML
api-version: 2019-12-01
location: westus
name: app-with-ssl
properties:
  containers:
  - name: nginx-with-ssl
    properties:
      image: nginx
      ports:
      - port: 443
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - name: nginx-config
        mountPath: /etc/nginx
  - name: my-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  volumes:
  - secret:
      ssl.crt: <Enter contents of base64-ssl.crt here>
      ssl.key: <Enter contents of base64-ssl.key here>
      nginx.conf: <Enter contents of base64-nginx.conf here>
    name: nginx-config
  ipAddress:
    ports:
    - port: 443
      protocol: TCP
    type: Public
  osType: Linux
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

### <a name="deploy-the-container-group"></a>Развертывание группы контейнеров

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create).

```azurecli-interactive
az group create --name myResourceGroup --location westus
```

Разверните группу контейнеров с помощью команды [AZ Container Create](/cli/azure/container#az-container-create) , ПЕРЕДАВ файл YAML в качестве аргумента.

```azurecli
az container create --resource-group <myResourceGroup> --file deploy-aci.yaml
```

### <a name="view-deployment-state"></a>Просмотр состояния развертывания

Чтобы просмотреть состояние развертывания, используйте следующую команду [az container show](/cli/azure/container#az-container-show).

```azurecli
az container show --resource-group <myResourceGroup> --name app-with-ssl --output table
```

Для успешного развертывания выходные данные похожи на следующие:

```console
Name          ResourceGroup    Status    Image                                                    IP:ports             Network    CPU/Memory       OsType    Location
------------  ---------------  --------  -------------------------------------------------------  -------------------  ---------  ---------------  --------  ----------
app-with-ssl  myresourcegroup  Running   nginx, mcr.microsoft.com/azuredocs/aci-helloworld        52.157.22.76:443     Public     1.0 core/1.5 gb  Linux     westus
```

## <a name="verify-tls-connection"></a>Проверка подключения TLS

С помощью браузера перейдите по общедоступному IP-адресу группы контейнеров. IP-адрес, показанный в этом примере, — это `52.157.22.76` , поэтому URL-адрес имеет значение **https://52.157.22.76** . Для просмотра работающего приложения необходимо использовать протокол HTTPS из-за конфигурации сервера nginx. Сбой при попытке подключения по протоколу HTTP.

![Снимок экрана браузера: приложение, выполняющееся в экземпляре контейнера Azure](./media/container-instances-container-group-ssl/aci-app-ssl-browser.png)

> [!NOTE]
> Так как в этом примере используется самозаверяющий сертификат, а не один из центра сертификации, браузер отображает предупреждение системы безопасности при подключении к сайту по протоколу HTTPS. Может потребоваться принять предупреждение или настроить параметры браузера или сертификата, чтобы перейти к странице. Это ожидаемое поведение.

>

## <a name="next-steps"></a>Дальнейшие действия

В этой статье мы рассмотрели, как настроить контейнер Nginx для включения TLS-подключений к веб приложению, работающему в группе контейнеров. Этот пример можно адаптировать для приложений, прослушиваемых через порты, отличные от порта 80. Кроме того, можно обновить файл конфигурации nginx, чтобы автоматически перенаправлять подключения сервера через порт 80 (HTTP) для использования протокола HTTPS.

Хотя в этой статье используется nginx в расширения, можно использовать другой поставщик TLS, например [Кадди](https://caddyserver.com/).

При развертывании группы контейнеров в [виртуальной сети Azure](container-instances-vnet.md)можно рассмотреть другие варианты включения КОНЕЧНОЙ точки TLS для внутреннего экземпляра контейнера, в том числе:

* [Прокси-серверы Функций Azure](../azure-functions/functions-proxies.md)
* [Управление API Azure](../api-management/api-management-key-concepts.md)
* [Шлюз приложений Azure](../application-gateway/overview.md) — см. пример [шаблона развертывания](https://github.com/Azure/azure-quickstart-templates/tree/master/201-aci-wordpress-vnet).

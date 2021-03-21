---
title: Пример запуска контейнера команды запуска DOCKER
titleSuffix: Azure Cognitive Services
description: Команда запуска DOCKER для контейнера работоспособности
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/12/2020
ms.author: aahi
ms.openlocfilehash: af8fec56c32b52e2af584e59f08db6cc7129c9c5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98948345"
---
## <a name="install-the-container"></a>Установка контейнера

Существует несколько способов установки и запуска Анализ текста для контейнера работоспособности. 

- Используйте [портал Azure](../how-tos/text-analytics-how-to-install-containers.md?tabs=healthcare) , чтобы создать анализ текста ресурс, и используйте DOCKER для получения контейнера.
- Используйте следующие сценарии PowerShell и Azure CLI для автоматизации развертывания ресурсов и конфигурации контейнеров.

### <a name="run-the-container-locally"></a>Локальный запуск контейнера

Чтобы запустить контейнер в собственной среде после загрузки образа контейнера, найдите его идентификатор образа:
 
```bash
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

Выполните следующую `docker run` команду. Замените заполнители ниже собственными значениями:

| Заполнитель | Значение | Формат или пример |
|-------------|-------|---|
| **{API_KEY}** | Ключ для ресурса Анализ текста. Его можно найти на странице **ключа и конечной точки** ресурса на портал Azure. |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{ENDPOINT_URI}** | Конечная точка для доступа к API анализа текста. Его можно найти на странице **ключа и конечной точки** ресурса на портал Azure. | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |
| **{IMAGE_ID}** | Идентификатор образа для контейнера. | `1.1.011300001-amd64-preview` |
| **{INPUT_DIR}** | Входной каталог для контейнера. | Windows: `C:\healthcareMount` <br> Linux и MacOS: `/home/username/input` |

```bash
docker run --rm -it -p 5000:5000 --cpus 6 --memory 12g \
--mount type=bind,src={INPUT_DIR},target=/output {IMAGE_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Disk:Format=json
```

Команда:

- Предполагается, что входной каталог существует на размещающем компьютере
- Запускает Анализ текста для контейнера работоспособности из образа контейнера.
- Выделяет 6 ядер ЦП и 12 гигабайт (ГБ) памяти.
- предоставляет TCP-порт 5000 и выделяет псевдотелетайп для контейнера;
- автоматически удаляет контейнер после завершения его работы. Образ контейнера остается доступным на главном компьютере.

### <a name="demo-ui-to-visualize-output"></a>Демонстрационный пользовательский интерфейс для визуализации выходных данных

> [!NOTE]
> Демонстрация доступна только в Анализ текста контейнере работоспособности.

Контейнер предоставляет интерфейсы REST API конечной точки прогнозирования запросов.  Мы также предоставили средство визуализации в контейнере, доступ к которому осуществляется путем добавления `/demo` к конечной точке контейнера. Пример:

```
http://<serverURL>:5000/demo
```

Используйте приведенный ниже запрос в виде фигурной скобки, чтобы отправить запрос в развернутый контейнер, заменив `serverURL` переменную соответствующим значением.

```bash
curl -X POST 'http://<serverURL>:5000/text/analytics/v3.2-preview.1/entities/health' --header 'Content-Type: application/json' --header 'accept: application/json' --data-binary @example.json

```

### <a name="install-the-container-using-azure-web-app-for-containers"></a>Установка контейнера с помощью Azure Веб-приложение для контейнеров

Azure [веб-приложение для контейнеров](https://azure.microsoft.com/services/app-service/containers/) — это ресурс Azure, предназначенный для выполнения контейнеров в облаке. Она предоставляет готовые возможности, такие как автоматическое масштабирование, поддержка контейнеров DOCKER и создание прикрепления DOCKER, Поддержка HTTPS и многое другое.

> [!NOTE]
> С помощью веб-приложения Azure вы автоматически получите домен в виде `<appservice_name>.azurewebsites.net`

Запустите этот скрипт PowerShell с помощью Azure CLI, чтобы создать Веб-приложение для контейнеров, используя подписку и образ контейнера по протоколу HTTPS. Дождитесь завершения сценария (приблизительно 25-30 минут) перед отправкой первого запроса.

```azurecli
$subscription_name = ""                    # THe name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           #    and AppSerivce to be attached to.
$resources_location = ""                   # This is the location you wish the AppServicePlan to be deployed to.
                                           #    You can use the "az account list-locations -o table" command to
                                           #    get the list of available locations and location code names.
$appservice_plan_name = ""                 # This is the AppServicePlan name you wish to have.
$appservice_name = ""                      # This is the AppService resource name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_REGISTRY_SERVER_PASSWORD = ""      # This will be provided separately.
$DOCKER_REGISTRY_SERVER_USERNAME = ""      # This will be provided separately.
$DOCKER_IMAGE_NAME = "containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest"

az login
az account set -s $subscription_name
az appservice plan create -n $appservice_plan_name -g $resource_group_name --is-linux -l $resources_location --sku P3V2
az webapp create -g $resource_group_name -p $appservice_plan_name -n $appservice_name -i $DOCKER_IMAGE_NAME -s $DOCKER_REGISTRY_SERVER_USERNAME -w $DOCKER_REGISTRY_SERVER_PASSWORD
az webapp config appsettings set -g $resource_group_name -n $appservice_name --settings Eula=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: https://<appservice_name>.azurewebsites.net
```

### <a name="install-the-container-using-azure-container-instance"></a>Установка контейнера с помощью экземпляра контейнера Azure

Вы также можете использовать экземпляр контейнера Azure (ACI), чтобы упростить развертывание. ACI — это ресурс, который позволяет запускать контейнеры DOCKER по запросу в управляемой среде Azure, не поддерживающей работу сервера. 

Инструкции по развертыванию ресурса ACI с помощью портал Azure см. в статье Использование службы " [экземпляры контейнеров Azure](../../containers/azure-container-instance-recipe.md) ". Вы также можете использовать приведенный ниже сценарий PowerShell с помощью Azure CLI, который создаст ACI в подписке с помощью образа контейнера.  Дождитесь завершения сценария (приблизительно 25-30 минут) перед отправкой первого запроса.  Из-за ограничения на максимальное число ЦП на ресурс ACI не выбирайте этот параметр, если предполагается отправлять более 5 больших документов (примерно 5000 символов каждый) для каждого запроса.
Сведения о доступности см. в статье [ACI региональные службы поддержки](../../../container-instances/container-instances-region-availability.md) . 

> [!NOTE] 
> В службе "экземпляры контейнеров Azure" не включена поддержка HTTPS для встроенных доменов. Если требуется протокол HTTPS, необходимо настроить его вручную, включая создание сертификата и регистрацию домена. Инструкции для этого можно найти в NGINX ниже.

```azurecli
$subscription_name = ""                    # The name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           # and AppService to be attached to.
$resources_location = ""                   # This is the location you wish the web app to be deployed to.
                                           # You can use the "az account list-locations -o table" command to
                                           # Get the list of available locations and location code names.
$azure_container_instance_name = ""        # This is the AzureContainerInstance name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_REGISTRY_SERVER_PASSWORD = ""      # This will be provided separately.
$DOCKER_REGISTRY_SERVER_USERNAME = ""      # This will be provided separately.
$DNS_LABEL = ""                            # This is the DNS label name you wish your ACI will have
$DOCKER_REGISTRY_LOGIN_SERVER = "containerpreview.azurecr.io"
$DOCKER_IMAGE_NAME = "containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest"

az login
az account set -s $subscription_name
az container create --resource-group $resource_group_name --name $azure_container_instance_name --image $DOCKER_IMAGE_NAME --cpu 4 --memory 12 --registry-login-server $DOCKER_REGISTRY_LOGIN_SERVER --registry-username $DOCKER_REGISTRY_SERVER_USERNAME --registry-password $DOCKER_REGISTRY_SERVER_PASSWORD --port 5000 --dns-name-label $DNS_LABEL --environment-variables Eula=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: http://<unique_dns_label>.<resource_group_region>.azurecontainer.io:5000
```

### <a name="secure-aci-connectivity"></a>Безопасное подключение ACI

По умолчанию при использовании ACI с API контейнера безопасность не предоставляется. Это связано с тем, что обычно контейнеры будут выполняться как часть модуля, который защищен от внешней сети сетевым мостом. Однако можно изменить контейнер с интерфейсным компонентом, сохранив конечную точку контейнера закрытой. В следующих примерах используется [nginx](https://www.nginx.com) в качестве входящего шлюза для поддержки проверки подлинности по протоколам HTTPS/SSL и Client-Certificate.

> [!NOTE]
> NGINX — это высокопроизводительный и прокси-сервер HTTP с открытым исходным кодом. Контейнер NGINX можно использовать для завершения подключения TLS для одного контейнера. Также возможны более сложные решения завершения TLS на основе NGINX.

#### <a name="set-up-nginx-as-an-ingress-gateway"></a>Настройка NGINX в качестве входящего шлюза

NGINX использует [файлы конфигурации](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/) для включения компонентов во время выполнения. Чтобы включить завершение TLS для другой службы, необходимо указать SSL-сертификат для завершения подключения TLS и  `proxy_pass` указать адрес службы. Пример приведен ниже.


> [!NOTE]
> `ssl_certificate` в локальной файловой системе контейнера NGINX требуется указать путь. Адрес, указанный для, `proxy_pass` должен быть доступен в сети контейнера nginx.

Контейнер NGINX загрузит все файлы в `_.conf_` , которые будут подключены к `/etc/nginx/conf.d/` пути конфигурации HTTP.

```nginx
server {
  listen              80;
  return 301 https://$host$request_uri;
}
server {
  listen              443 ssl;
  # replace with .crt and .key paths
  ssl_certificate     /cert/Local.crt;
  ssl_certificate_key /cert/Local.key;

  location / {
    proxy_pass http://cognitive-service:5000;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP  $remote_addr;
  }
}
```

#### <a name="example-docker-compose-file"></a>Пример файла создания DOCKER

В приведенном ниже примере показано, как можно создать файл- [сокомпозицию DOCKER](https://docs.docker.com/compose/reference/overview) для развертывания NGINX и анализ текста для контейнеров работоспособности.

```yaml
version: "3.7"
services:
  cognitive-service:
    image: {IMAGE_ID}
    ports:
      - 5000:5000
    environment:
      - eula=accept
      - billing={ENDPOINT_URI}
      - apikey={API_KEY}
      - Logging:Disk:Format=json
    volumes:
        # replace with path to logs folder
      - <path-to-logs-folder>:/output
  nginx:
    image: nginx
    ports:
      - 443:443
    volumes:
        # replace with paths for certs and conf folders
      - <path-to-certs-folder>:/cert
      - <path-to-conf-folder>:/etc/nginx/conf.d/
```

Чтобы инициировать этот файл создания DOCKER, выполните следующую команду в консоли на корневом уровне файла:

```bash
docker-compose up
```

Дополнительные сведения см. в документации по NGINX на [nginx SSL](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/).
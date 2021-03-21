---
title: Правила доступа брандмауэра
description: Настройте правила для доступа к реестру контейнеров Azure из защищенного брандмауэра, разрешив доступ к REST API и доменным именам конечных точек данных или диапазонам IP-адресов, зависящим от службы.
ms.topic: article
ms.date: 05/18/2020
ms.openlocfilehash: 548d64632c1d726111770dfb49f705d31f5ca714
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97935994"
---
# <a name="configure-rules-to-access-an-azure-container-registry-behind-a-firewall"></a>Настройка правил для доступа к реестру контейнеров Azure за брандмауэром

В этой статье объясняется, как настроить правила в брандмауэре, чтобы разрешить доступ к реестру контейнеров Azure. Например, для извлечения образа контейнера устройству Azure IoT Edge, которое находится за брандмауэром или прокси-сервером, может потребоваться доступ к реестру контейнеров. Или заблокированному серверу в локальной сети может потребоваться доступ для отправки образа.

Если вместо этого вы хотите настроить входящий сетевой доступ к реестру контейнеров только в виртуальной сети Azure, обратитесь к статье [Настройка Приватного канала Azure для реестра контейнеров Azure](container-registry-private-link.md).

## <a name="about-registry-endpoints"></a>Сведения о конечных точках реестра

Чтобы извлечь или отправить образы или другие артефакты в реестр контейнеров Azure, клиент (например, управляющая программа Docker) должен взаимодействовать по протоколу HTTPS с двумя разными конечными точками. Для клиентов, обращающихся к реестру из-за пределов брандмауэра, необходимо настроить правила доступа для обеих конечных точек. Обе конечные точки доступны через порт 443.

* **Конечная точка REST API реестра** — операции проверки подлинности и управления реестром обрабатываются через общедоступную конечную точку REST API реестра. Эта конечная точка является именем входа на сервер для реестра. Например, `myregistry.azurecr.io`.

* **Конечная точка хранилища (данные)**  — Azure [выделяет хранилище BLOB-объектов](container-registry-storage.md) в учетных записях хранения Azure от имени каждого реестра для управления данными для образов контейнеров и других артефактов. При работе со слоями образов в реестре контейнеров Azure клиент выполняет запросы с помощью конечной точки учетной записи хранения, предоставляемой реестром.

Если реестр [геореплицированный](container-registry-geo-replication.md), клиенту может потребоваться взаимодействовать с конечной точкой данных в определенном регионе или в нескольких реплицируемых регионах.

## <a name="allow-access-to-rest-and-data-endpoints"></a>Разрешение доступа к REST и конечным точкам данных

* **Конечная точка REST** — разрешение доступа к полному имени входа на сервер для реестра, `<registry-name>.azurecr.io`, или к соответствующему диапазону IP-адресов.
* **Конечная точка хранилища (данные)**  — разрешение доступа ко всем учетным записям хранилища BLOB-объектов Azure с помощью подстановочных знаков `*.blob.core.windows.net` или к соответствующему диапазону IP-адресов.
> [!NOTE]
> В Реестре контейнеров Azure имеются [выделенные конечные точки данных](#enable-dedicated-data-endpoints), что позволяет жестко ограничить правила брандмауэра клиента для хранилища реестра. При необходимости включите конечные точки данных во всех регионах, где размещается или реплицируется реестр, используя формат `<registry-name>.<region>.data.azurecr.io`.

## <a name="allow-access-by-ip-address-range"></a>Разрешение доступа по диапазону IP-адресов

Если в организации применяются политики разрешения доступа только к определенным IP-адресам или диапазонам адресов, скачайте [диапазоны IP-адресов Azure и теги службы — общедоступное облако](https://www.microsoft.com/download/details.aspx?id=56519).

Чтобы найти диапазоны IP-адресов конечной точки ACR REST, доступ к которым необходимо разрешить, найдите **AzureContainerRegistry** в файле JSON.

> [!IMPORTANT]
> Диапазоны IP-адресов для служб Azure могут измениться, а обновления публикуются еженедельно. Регулярно загружайте файл JSON и вносите необходимые обновления в правила доступа. Если сценарий предполагает настройку правил группы безопасности сети в виртуальной сети Azure или если вы используете Брандмауэр Azure, используйте [тег службы](#allow-access-by-service-tag) **AzureContainerRegistry**.
>

### <a name="rest-ip-addresses-for-all-regions"></a>IP-адреса REST для всех регионов

```json
{
  "name": "AzureContainerRegistry",
  "id": "AzureContainerRegistry",
  "properties": {
    "changeNumber": 10,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.66.140.72/29",
    [...]
```

### <a name="rest-ip-addresses-for-a-specific-region"></a>IP-адреса REST для определенного региона

Выполните поиск конкретного региона, например **AzureContainerRegistry.AustraliaEast**.

```json
{
  "name": "AzureContainerRegistry.AustraliaEast",
  "id": "AzureContainerRegistry.AustraliaEast",
  "properties": {
    "changeNumber": 1,
    "region": "australiaeast",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.70.72.136/29",
    [...]
```

### <a name="storage-ip-addresses-for-all-regions"></a>IP-адреса хранилища для всех регионов

```json
{
  "name": "Storage",
  "id": "Storage",
  "properties": {
    "changeNumber": 19,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "13.65.107.32/28",
    [...]
```

### <a name="storage-ip-addresses-for-specific-regions"></a>IP-адреса хранилища для определенного региона

Выполните поиск конкретного региона, например **Storage.AustraliaCentral**.

```json
{
  "name": "Storage.AustraliaCentral",
  "id": "Storage.AustraliaCentral",
  "properties": {
    "changeNumber": 1,
    "region": "australiacentral",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "52.239.216.0/23"
    [...]
```

## <a name="allow-access-by-service-tag"></a>Разрешение доступа по тегу службы

В виртуальной сети Azure используйте правила безопасности сети для фильтрации трафика от ресурса, например виртуальной машины, в реестр контейнеров. Чтобы упростить создание правил сети Azure, используйте [тег службы](../virtual-network/network-security-groups-overview.md#service-tags) **AzureContainerRegistry**. Тег службы представляет группу префиксов IP-адресов для глобального доступа к службе Azure или в определенном регионе Azure. Тег автоматически обновляется при изменении адресов. 

Например, создайте правило группы безопасности сети для исходящего трафика с назначением **AzureContainerRegistry**, чтобы разрешить трафик в реестр контейнеров Azure. Чтобы разрешить доступ к тегу службы только в определенном регионе, укажите регион в следующем формате: **AzureContainerRegistry**.[*название региона*].

## <a name="enable-dedicated-data-endpoints"></a>Включение выделенных конечных точек данных 

> [!WARNING]
> Если вы ранее настроили брандмауэр клиента на доступ к существующим конечным точкам `*.blob.core.windows.net`, переключение на выделенные конечные точки данных повлияет на подключение клиентов, что приведет к сбоям при извлечении. Чтобы обеспечить стабильный доступ для клиентов, добавьте новые правила конечной точки данных в правила брандмауэра клиента. После завершения включите выделенные конечные точки данных для реестров с помощью Azure CLI или других средств.

Выделенные конечные точки данных являются необязательным компонентом службы реестра контейнеров уровня **Премиум**. Ознакомиться с информацией об уровнях служб реестра и ограничениях можно в статье [Уровни служб Реестра контейнеров Azure](container-registry-skus.md). 

Включить выделенные конечные точки данных можно с помощью портала Azure или Azure CLI. Конечные точки данных соответствуют региональным шаблонам, `<registry-name>.<region>.data.azurecr.io`. В геореплицированном реестре включение конечных точек данных позволяет использовать конечные точки во всех регионах реплик.

### <a name="portal"></a>Портал

Порядок включения конечных точек данных с помощью портала

1. Перейдите к реестру контейнеров.
1. Выберите **Сеть** > **Общий доступ**.
1. Установите флажок **Включить выделенную конечную точку данных**.
1. Щелкните **Сохранить**.

Конечная точка данных или конечные точки отобразятся на портале.

:::image type="content" source="media/container-registry-firewall-access-rules/dedicated-data-endpoints-portal.png" alt-text="Выделенные конечные точки данных на портале":::

### <a name="azure-cli"></a>Azure CLI

Чтобы включить конечные точки данных с помощью Azure CLI, используйте Azure CLI версии 2.4.0 или более поздней. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Следующая команда [az acr update][az-acr-update] позволяет включить выделенные конечные точки данных в реестре *myregistry*. 

```azurecli
az acr update --name myregistry --data-endpoint-enabled
```

Чтобы просмотреть конечные точки данных, используйте команду [az acr show-endpoints][az-acr-show-endpoints].

```azurecli
az acr show-endpoints --name myregistry
```

В выходных данных для демонстрационных целей показаны две региональные конечные точки.

```
{
    "loginServer": "myregistry.azurecr.io",
    "dataEndpoints": [
        {
            "region": "eastus",
            "endpoint": "myregistry.eastus.data.azurecr.io",
        },
        {
            "region": "westus",
            "endpoint": "myregistry.westus.data.azurecr.io",
        }
    ]
}
```

После настройки выделенных конечных точек данных для реестра можно включить правила доступа брандмауэра клиента для конечных точек данных. Включите правила доступа к конечной точке данных для всех необходимых разделов реестра.

## <a name="configure-client-firewall-rules-for-mcr"></a>Настройка правил брандмауэра клиента для MCR

Если вам требуется доступ к реестру контейнеров Microsoft (MCR) из-за пределов брандмауэра, обратитесь к руководству по настройке [правил брандмауэра клиента MCR](https://github.com/microsoft/containerregistry/blob/master/client-firewall-rules.md). MCR — это основной реестр для всех опубликованных корпорацией Майкрософт образов Docker, таких как образы Windows Server.

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте подробнее о [рекомендациях Azure по обеспечению сетевой безопасности](../security/fundamentals/network-best-practices.md).

* Узнайте подробнее о [группах безопасности](../virtual-network/network-security-groups-overview.md) в виртуальной сети Azure.

* Узнайте подробнее о настройке [Приватного канала](container-registry-private-link.md) для реестра контейнеров.

* Узнайте подробнее о [выделенных конечных точках данных](https://azure.microsoft.com/blog/azure-container-registry-mitigating-data-exfiltration-with-dedicated-data-endpoints/) для реестра контейнеров Azure.



<!-- IMAGES -->

<!-- LINKS - External -->

<!-- LINKS - Internal -->

[az-acr-update]: /cli/azure/acr#az-acr-update
[az-acr-show-endpoints]: /cli/azure/acr#az-acr-show-endpoints
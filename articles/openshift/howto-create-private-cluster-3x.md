---
title: Создание частного кластера с помощью Azure Red Hat OpenShift 3.11 | Документация Майкрософт
description: Создание частного кластера с помощью Azure Red Hat OpenShift 3.11
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 03/02/2020
keywords: aro, openshift, частный кластер, red hat
ms.openlocfilehash: 28a21635da770a990a21e0da2a8034c754f4f7aa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "100633246"
---
# <a name="create-a-private-cluster-with-azure-red-hat-openshift-311"></a>Создание частного кластера с помощью Azure Red Hat OpenShift 3.11

> [!IMPORTANT]
> Azure Red Hat OpenShift 3,11 будет прекращена 30 июня 2022. Поддержка для создания новых кластеров Azure Red Hat OpenShift 3,11 продолжится до 30 ноября 2020. После выхода из эксплуатации оставшиеся кластеры Azure Red Hat OpenShift 3,11 будут закрыты для предотвращения уязвимости системы безопасности.
> 
> Следуйте указаниям этого руководством, чтобы [создать кластер Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Если у вас есть определенные вопросы, [свяжитесь с нами](mailto:arofeedback@microsoft.com).

Частные кластеры обеспечивают следующие преимущества.

* Частные кластеры не предоставляют компоненты уровня управления кластером (такие как серверы API) через общедоступный IP-адрес.
* Виртуальная сеть частного кластера настраивается клиентами, что позволяет настроить сеть на поддержку пиринга с другими виртуальными сетями, включая среды ExpressRoute. Вы также можете настроить пользовательскую службу DNS в виртуальной сети для интеграции с внутренними службами.

## <a name="before-you-begin"></a>Перед началом

Поля в следующем фрагменте конфигурации являются новыми и должны быть включены в конфигурацию кластера. `managementSubnetCidr` должен находиться в виртуальной сети кластера и использоваться Azure для управления кластером.

```json
properties:
 networkProfile:
   managementSubnetCidr: 10.0.1.0/24
 masterPoolProfile:
   apiProperties:
     privateApiServer: true
```

Частный кластер можно развернуть с помощью примеров сценариев, приведенных ниже. После развертывания кластера выполните команду `cluster get` и просмотрите свойство `properties.FQDN`, чтобы определить частный IP-адрес сервера API OpenShift.

Виртуальная сеть кластера будет создана с разрешениями, чтобы ее можно было изменить. После этого можно настроить сеть для доступа к виртуальной сети (ExpressRoute, VPN, пиринг виртуальной сети) в соответствии с вашими потребностями.

Если изменить серверы доменных имен DNS в виртуальной сети кластера, необходимо будет выполнить обновление в кластере со свойством `properties.RefreshCluster`, имеющим значение `true`, чтобы можно было повторно создать образ виртуальных машин. Это обновление позволит им выбрать новые серверы доменных имен.

## <a name="sample-configuration-scripts"></a>Примеры сценариев конфигурации

Используйте примеры сценариев в этом разделе, чтобы настроить и развернуть частный кластер.

### <a name="environment"></a>Среда

Заполните переменные среды ниже, используя собственные значения.

> [!NOTE]
> В качестве расположения следует указать `eastus2`, поскольку в настоящее время это единственное поддерживаемое расположение для частных кластеров.

``` bash
export CLUSTER_NAME=
export LOCATION=eastus2
export TOKEN=$(az account get-access-token --query 'accessToken' -o tsv)
export SUBID=
export TENANT_ID=
export ADMIN_GROUP=
export CLIENT_ID=
export SECRET=
```

### <a name="private-clusterjson"></a>private-cluster.json

Ниже приведен пример конфигурации кластера с включенным частным кластером с использованием переменных среды, определенных выше.

```json
{
   "location": "$LOCATION",
   "name": "$CLUSTER_NAME",
   "properties": {
       "openShiftVersion": "v3.11",
       "networkProfile": {
           "vnetCIDR": "10.0.0.0/8",
           "managementSubnetCIDR" : "10.0.1.0/24"
       },
       "authProfile": {
           "identityProviders": [
               {
                   "name": "Azure AD",
                   "provider": {
                       "kind": "AADIdentityProvider",
                       "clientId": "$CLIENT_ID",
                       "secret": "$SECRET",
                       "tenantId": "$TENANT_ID",
                       "customerAdminGroupID": "$ADMIN_GROUP"
                   }
               }
           ]
       },
       "masterPoolProfile": {
           "name": "master",
           "count": 3,
           "vmSize": "Standard_D4s_v3",
           "osType": "Linux",
           "subnetCIDR": "10.0.0.0/24",
           "apiProperties": {
                "privateApiServer": true
            }
       },
       "agentPoolProfiles": [
           {
               "role": "compute",
               "name": "compute",
               "count": 1,
               "vmSize": "Standard_D4s_v3",
               "osType": "Linux",
               "subnetCIDR": "10.0.0.0/24"
           },
           {
               "role": "infra",
               "name": "infra",
               "count": 3,
               "vmSize": "Standard_D4s_v3",
               "osType": "Linux",
               "subnetCIDR": "10.0.0.0/24"
           }
       ],
       "routerProfiles": [
           {
               "name": "default"
           }
       ]
   }
}
```

## <a name="deploy-a-private-cluster"></a>Развертывание частного кластера

После настройки частного кластера с помощью приведенных выше примеров сценариев выполните следующую команду, чтобы развернуть частный кластер.

``` bash
az group create --name $CLUSTER_NAME --location $LOCATION
cat private-cluster.json | envsubst | curl -v -X PUT \
-H 'Content-Type: application/json; charset=utf-8' \
-H 'Authorization: Bearer '$TOKEN'' -d @- \
 https://management.azure.com/subscriptions/$SUBID/resourceGroups/$CLUSTER_NAME/providers/Microsoft.ContainerService/openShiftManagedClusters/$CLUSTER_NAME?api-version=2019-10-27-preview
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о доступе к консоли OpenShift см. в [пошаговом руководстве по веб-консоли](https://docs.openshift.com/container-platform/3.11/getting_started/developers_console.html).

---
title: Развертывание платформы контейнеров OpenShift 3,11 в Azure
description: Развертывание платформы контейнеров OpenShift 3,11 в Azure.
author: haroldwongms
manager: mdotson
ms.service: virtual-machines
ms.subservice: openshift
ms.collection: linux
ms.topic: how-to
ms.workload: infrastructure
ms.date: 04/05/2020
ms.author: haroldw
ms.openlocfilehash: ce3f56530a7bad6eecd4c2edd3b21debe8c75a24
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102214156"
---
# <a name="deploy-openshift-container-platform-311-in-azure"></a>Развертывание платформы контейнеров OpenShift 3,11 в Azure

Вы можете использовать один из нескольких методов для развертывания платформы контейнеров OpenShift 3,11 в Azure:

- Вы можете вручную развернуть необходимые компоненты инфраструктуры Azure, а затем следовать [документации по платформе контейнеров OpenShift](https://docs.openshift.com/container-platform).
- Также можно использовать существующий [шаблон Resource Manager](https://github.com/Microsoft/openshift-container-platform/), упрощающий развертывание кластера платформы OpenShift Container Platform.
- Другой вариант — использовать [предложение Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/osatesting.open-shift-azure-proxy).

Для всех вариантов подписка Red Hat является обязательной. Во время развертывания экземпляр Red Hat Enterprise Linux регистрируется в подписке Red Hat и связывается с идентификатором пула, который содержит права доступа к платформе OpenShift Container Platform.
Убедитесь, что у вас есть действительное имя пользователя, пароль и идентификатор пула диспетчера подписки Red Hat (RHSM). Можно также использовать ключ активации, идентификатор организации и идентификатор пула. Данные можно проверить, войдя на сайт https://access.redhat.com.


## <a name="deploy-using-the-openshift-container-platform-resource-manager-311-template"></a>Развертывание с помощью шаблона диспетчер ресурсов 3,11 платформы контейнеров OpenShift

### <a name="private-clusters"></a>Частные кластеры

Для развертывания частных кластеров OpenShift требуется больше, чем просто отсутствие общедоступного IP-адреса, связанного с основной подсистемой балансировки нагрузки (веб-консоль) или с подсистемой балансировки нагрузки (маршрутизатор).  Частный кластер обычно использует пользовательский DNS-сервер (а не Azure DNS по умолчанию), имя пользовательского домена (например, contoso.com) и предварительно определенные виртуальные сети.  Для частных кластеров необходимо заранее настроить виртуальную сеть с использованием всех подходящих подсетей и параметров DNS-сервера.  Затем используйте **ексистингмастерсубнетреференце**, **ексистингинфрасубнетреференце**, **ексистингкнссубнетреференце** и **existingNodeSubnetReference** , чтобы указать существующую подсеть для использования кластером.

Если выбрана частная Главная реплика (**мастерклустертипе**= Private), для **мастерприватеклустерип** необходимо указать статический частный IP-адрес.  Этот IP-адрес будет назначен внешнему интерфейсу основной подсистемы балансировки нагрузки.  IP-адрес должен быть в пределах CIDR для главной подсети и не используется.  для **мастерклустерднстипе** необходимо задать значение Custom, а для **мастерклустерднс** должно быть указано имя главного DNS-сервера.  DNS-имя должно быть сопоставлено со статическим частным IP-адресом и будет использоваться для доступа к консоли на главных узлах.

Если выбран частный маршрутизатор (**раутерклустертипе**= Private), для **раутерприватеклустерип** необходимо указать статический частный IP-адрес.  Этот IP-адрес будет назначен внешнему интерфейсу подсистемы балансировки нагрузки.  IP-адрес должен быть в пределах CIDR для подсети инфраструктуры и не используется.  для параметра **раутингсубдомаинтипе** должно быть задано значение Custom, а для **раутингсубдомаин** необходимо указать DNS-имя с подстановочными знаками.  

Если выбраны частные главные и частные маршрутизаторы, для **имя_домена** также необходимо указать имя личного домена.

После успешного развертывания узел бастиона является единственным узлом с общедоступным IP-адресом, к которому можно подключаться по протоколу SSH.  Даже если главные узлы настроены для общего доступа, они не предоставляются для доступа по протоколу SSH.

Для развертывания с использованием шаблона Resource Manager используется файл параметров, в котором указываются входные параметры. Для дополнительной настройки развертывания создайте вилку репозитория GitHub и измените нужные элементы.

Например, достаточно часто возникает потребность изменить следующие параметры:

- размер виртуальной машины-бастиона (переменная в azuredeploy.json);
- соглашения об именовании (переменные в azuredeploy.json);
- особенности кластера OpenShift, изменяемые в файле hosts (deployOpenShift.sh).

### <a name="configure-the-parameters-file"></a>Настройка файла параметров

[Шаблон платформы контейнеров OpenShift](https://github.com/Microsoft/openshift-container-platform) содержит несколько ветвей для разных версий платформы контейнеров OpenShift.  В соответствии с потребностями вы можете выполнять развертывание непосредственно из репозитория или создать вилку репозитория, чтобы вносить нужные изменения в шаблоны и скрипты перед развертыванием.

Используйте значение `appId` созданной ранее субъект-службы для параметра `aadClientId`.

В примере ниже представлен файл параметров с именем azuredeploy.parameters.json, который содержит все необходимые входные данные.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "_artifactsLocation": {
            "value": "https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master"
        },
        "location": {
            "value": "eastus"
        },
        "masterVmSize": {
            "value": "Standard_E2s_v3"
        },
        "infraVmSize": {
            "value": "Standard_D4s_v3"
        },
        "nodeVmSize": {
            "value": "Standard_D4s_v3"
        },
        "cnsVmSize": {
            "value": "Standard_E4s_v3"
        },
        "osImageType": {
            "value": "defaultgallery"
        },
        "marketplaceOsImage": {
            "value": {
                "publisher": "RedHat",
                "offer": "RHEL",
                "sku": "7-RAW",
                "version": "latest"
            }
        },
        "storageKind": {
            "value": "changeme"
        },
        "openshiftClusterPrefix": {
            "value": "changeme"
        },
        "minorVersion": {
            "value": "69"
        },
        "masterInstanceCount": {
            "value": 3
        },
        "infraInstanceCount": {
            "value": 3
        },
        "nodeInstanceCount": {
            "value": 3
        },
        "cnsInstanceCount": {
            "value": 3
        },
        "osDiskSize": {
            "value": 64
        },
        "dataDiskSize": {
            "value": 64
        },
        "cnsGlusterDiskSize": {
            "value": 128
        },
        "adminUsername": {
            "value": "changeme"
        },
        "enableMetrics": {
            "value": "false"
        },
        "enableLogging": {
            "value": "false"
        },
        "enableCNS": {
            "value": "false"
        },
        "rhsmUsernameOrOrgId": {
            "value": "changeme"
        },
        "rhsmPoolId": {
            "value": "changeme"
        },
        "rhsmBrokerPoolId": {
            "value": "changeme"
        },
        "sshPublicKey": {
            "value": "GEN-SSH-PUB-KEY"
        },
        "keyVaultSubscriptionId": {
            "value": "255a325e-8276-4ada-af8f-33af5658eb34"
        },
        "keyVaultResourceGroup": {
            "value": "changeme"
        },
        "keyVaultName": {
            "value": "changeme"
        },
        "enableAzure": {
            "value": "true"
        },
        "aadClientId": {
            "value": "changeme"
        },
        "domainName": {
            "value": "contoso.com"
        },
        "masterClusterDnsType": {
            "value": "default"
        },
        "masterClusterDns": {
            "value": "console.contoso.com"
        },
        "routingSubDomainType": {
            "value": "nipio"
        },
        "routingSubDomain": {
            "value": "apps.contoso.com"
        },
        "virtualNetworkNewOrExisting": {
            "value": "new"
        },
        "virtualNetworkName": {
            "value": "changeme"
        },
        "addressPrefixes": {
            "value": "10.0.0.0/14"
        },
        "masterSubnetName": {
            "value": "changeme"
        },
        "masterSubnetPrefix": {
            "value": "10.1.0.0/16"
        },
        "infraSubnetName": {
            "value": "changeme"
        },
        "infraSubnetPrefix": {
            "value": "10.2.0.0/16"
        },
        "nodeSubnetName": {
            "value": "changeme"
        },
        "nodeSubnetPrefix": {
            "value": "10.3.0.0/16"
        },
        "existingMasterSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/mastersubnet"
        },
        "existingInfraSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/infrasubnet"
        },
        "existingCnsSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/cnssubnet"
        },
        "existingNodeSubnetReference": {
            "value": "/subscriptions/abc686f6-963b-4e64-bff4-99dc369ab1cd/resourceGroups/vnetresourcegroup/providers/Microsoft.Network/virtualNetworks/openshiftvnet/subnets/nodesubnet"
        },
        "masterClusterType": {
            "value": "public"
        },
        "masterPrivateClusterIp": {
            "value": "10.1.0.200"
        },
        "routerClusterType": {
            "value": "public"
        },
        "routerPrivateClusterIp": {
            "value": "10.2.0.200"
        },
        "routingCertType": {
            "value": "selfsigned"
        },
        "masterCertType": {
            "value": "selfsigned"
        }
    }
}
```

Замените указанные параметры соответствующими данными для своей системы.

Разные выпуски могут иметь разные параметры, поэтому проверьте необходимые параметры для используемой ветви.

### <a name="azuredeployparametersjson-file-explained"></a>azuredeploy.Parameters.jsс описанием файла

| Свойство. | Описание | Допустимые параметры | Значение по умолчанию |
|----------|-------------|---------------|---------------|
| `_artifactsLocation`  | URL-адрес для артефактов (JSON, сценарии и т. д.) |  |  HTTPS: \/ /RAW.githubusercontent.com/Microsoft/openshift-Container-Platform/Master  |
| `location` | Регион Azure для развертывания ресурсов |  |  |
| `masterVmSize` | Размер главной виртуальной машины. Выберите один из разрешенных размеров виртуальных машин, перечисленных в azuredeploy.jsв файле |  | Standard_E2s_v3 |
| `infraVmSize` | Размер инфраструктуры виртуальной машины. Выберите один из разрешенных размеров виртуальных машин, перечисленных в azuredeploy.jsв файле |  | Standard_D4s_v3 |
| `nodeVmSize` | Размер виртуальной машины узла приложения. Выберите один из разрешенных размеров виртуальных машин, перечисленных в azuredeploy.jsв файле |  | Standard_D4s_v3 |
| `cnsVmSize` | Размер виртуальной машины узла собственного хранилища (CNS) контейнера. Выберите один из разрешенных размеров виртуальных машин, перечисленных в azuredeploy.jsв файле |  | Standard_E4s_v3 |
| `osImageType` | Используемый образ RHEL. дефаултгаллери: по запросу; Marketplace: стороннее изображение | дефаултгаллери <br> магазин | дефаултгаллери |
| `marketplaceOsImage` | Если `osImageType` параметр — Marketplace, введите соответствующие значения для параметров "издатель", "предложение", "SKU", "версия" предложения Marketplace. Этот параметр является типом объекта |  |  |
| `storageKind` | Тип используемого хранилища  | Управляемые<br> unmanaged | Управляемые |
| `openshiftClusterPrefix` | Префикс кластера, используемый для настройки имен узлов для всех узлов.  От 1 до 20 символов |  | mycluster |
| `minoVersion` | Дополнительный номер версии OpenShift контейнера платформы 3,11 для развертывания |  | 69 |
| `masterInstanceCount` | Количество узлов образцов для развертывания | 1, 3, 5 | 3 |
| `infraInstanceCount` | Число развертываемых узлов для развертывания | 1, 2, 3 | 3 |
| `nodeInstanceCount` | Число развертываемых узлов | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30 | 2 |
| `cnsInstanceCount` | Число развертываемых узлов CNS | 3, 4 | 3 |
| `osDiskSize` | Размер диска операционной системы для виртуальной машины (в ГБ) | 64, 128, 256, 512, 1024, 2048 | 64 |
| `dataDiskSize` | Размер диска данных для присоединения к узлам для тома DOCKER (в ГБ) | 32, 64, 128, 256, 512, 1024, 2048 | 64 |
| `cnsGlusterDiskSize` | Размер диска данных для присоединения к узлам CNS для использования глустерфс (в ГБ | 32, 64, 128, 256, 512, 1024, 2048 | 128 |
| `adminUsername` | Имя администратора для входа в ОС (ВМ) и начального пользователя OpenShift |  | окпадмин |
| `enableMetrics` | Включите метрики. Метрики нуждаются в дополнительных ресурсах, поэтому выберите подходящий размер для инфраструктуры виртуальной машины | true <br> false | false |
| `enableLogging` | Включить ведение журнала. для модуля elasticsearch требуется 8 ГБ ОЗУ, поэтому выберите подходящий размер для инфраструктуры виртуальной машины. | true <br> false | false |
| `enableCNS` | Включение собственного хранилища контейнера | true <br> false | false |
| `rhsmUsernameOrOrgId` | Имя пользователя или идентификатор организации для диспетчера подписки Red Hat |  |  |
| `rhsmPoolId` | Идентификатор пула диспетчера подписки Red Hat, который содержит ваши права OpenShift для вычислений узлов. |  |  |
| `rhsmBrokerPoolId` | Идентификатор пула диспетчера подписок Red Hat, который содержит ваши права на OpenShift для главных и бесузелных узлов. Если у вас нет других идентификаторов пула, укажите идентификатор пула "Рхсмпулид". |  |
| `sshPublicKey` | Скопируйте открытый ключ SSH |  |  |
| `keyVaultSubscriptionId` | Идентификатор подписки, которая содержит Key Vault |  |  |
| `keyVaultResourceGroup` | Имя группы ресурсов, которая содержит Key Vault |  |  |
| `keyVaultName` | Имя созданного Key Vault |  |  |
| `enableAzure` | Включение поставщика облачных служб Azure | true <br> false | true |
| `aadClientId` | Идентификатор клиента Azure Active Directory также известен как идентификатор приложения для субъекта-службы |  |  |
| `domainName` | Имя пользовательского доменного имени, которое будет использоваться (если применимо). Если не развертывать полностью частный кластер, задайте значение "нет". |  | нет |
| `masterClusterDnsType` | Тип домена для веб-консоли OpenShift. "default" будет использовать DNS-метку для основного общедоступного IP-адреса. "Custom" позволяет определить собственное имя | default <br> пользовательский | default |
| `masterClusterDns` | Настраиваемое DNS-имя, которое будет использоваться для доступа к Web Console OpenShift, если вы выбрали "Custom" для `masterClusterDnsType` |  | console.contoso.com |
| `routingSubDomainType` | Если задано значение "нипио", `routingSubDomain` будет использоваться NIP.IO.  Используйте "Custom", если у вас есть собственный домен, который вы хотите использовать для маршрутизации. | нипио <br> пользовательский | нипио |
| `routingSubDomain` | DNS-имя с подстановочными знаками, которое вы хотите использовать для маршрутизации, если вы выбрали "Custom" для `routingSubDomainType` |  | apps.contoso.com |
| `virtualNetworkNewOrExisting` | Укажите, следует ли использовать существующую виртуальную сеть или создать виртуальную сеть | existing <br> new | new |
| `virtualNetworkResourceGroupName` | Имя группы ресурсов для новой виртуальной сети, если вы выбрали "создать" для `virtualNetworkNewOrExisting` |  | resourceGroup (). имя |
| `virtualNetworkName` | Имя создаваемой виртуальной сети, если вы выбрали "создать" для `virtualNetworkNewOrExisting` |  | опеншифтвнет |
| `addressPrefixes` | Префикс адреса новой виртуальной сети |  | 10.0.0.0/14 |
| `masterSubnetName` | Имя главной подсети |  | мастерсубнет |
| `masterSubnetPrefix` | CIDR, используемый для главной подсети, должен быть подмножеством addressPrefix |  | 10.1.0.0/16 |
| `infraSubnetName` | Имя подсети инфраструктуры |  | инфрасубнет |
| `infraSubnetPrefix` | CIDR, используемый для инфраструктуры подсети, должен быть подмножеством addressPrefix |  | 10.2.0.0/16 |
| `nodeSubnetName` | Имя подсети узла |  | нодесубнет |
| `nodeSubnetPrefix` | CIDR, используемый для подсети узла, должен быть подмножеством addressPrefix |  | 10.3.0.0/16 |
| `existingMasterSubnetReference` | Полная ссылка на существующую подсеть для главных узлов. Не требуется при создании новой виртуальной сети или подсетей |  |  |
| `existingInfraSubnetReference` | Полная ссылка на существующую подсеть для узлов. Не требуется при создании новой виртуальной сети или подсетей |  |  |
| `existingCnsSubnetReference` | Полная ссылка на существующую подсеть для узлов CNS. Не требуется при создании новой виртуальной сети или подсетей |  |  |
| `existingNodeSubnetReference` | Полная ссылка на существующую подсеть для вычислений узлов. Не требуется при создании новой виртуальной сети или подсетей |  |  |
| `masterClusterType` | Укажите, использует ли кластер частные или общедоступные главные узлы. Если выбран параметр частный, главные узлы не будут доступны в Интернете через общедоступный IP-адрес. Вместо этого он будет использовать частный IP-адрес, указанный в `masterPrivateClusterIp` | public <br> private | public |
| `masterPrivateClusterIp` | Если выбраны частные главные узлы, необходимо указать частный IP-адрес для использования внутренней подсистемой балансировки нагрузки для главных узлов. Этот статический IP-адрес должен находиться в пределах блока CIDR для главной подсети и не используется. Если выбраны общедоступные главные узлы, это значение не будет использоваться, но по-прежнему должно быть указано |  | 10.1.0.200 |
| `routerClusterType` | Укажите, использует ли кластер частные или общедоступные узлы. Если выбран параметр частный, то узлы инфраструктуры не будут доступны через общедоступный IP-адрес. Вместо этого он будет использовать частный IP-адрес, указанный в `routerPrivateClusterIp` | public <br> private | public |
| `routerPrivateClusterIp` | Если выбраны частные узлы, необходимо указать частный IP-адрес, который будет использоваться внутренней подсистемой балансировки нагрузки для кластерных узлов. Этот статический IP-адрес должен находиться в пределах блока CIDR для подсети инфраструктуры и не используется. Если выбраны общедоступные инфраструктуры, это значение не будет использоваться, но по-прежнему должно быть указано |  | 10.2.0.200 |
| `routingCertType` | Использовать пользовательский сертификат для домена маршрутизации или самозаверяющий сертификат по умолчанию — следуйте инструкциям в разделе " **настраиваемые сертификаты** ". | селфсигнед <br> пользовательский | селфсигнед |
| `masterCertType` | Использовать пользовательский сертификат для главного домена или самозаверяющий сертификат по умолчанию — следуйте инструкциям в разделе " **настраиваемые сертификаты** ". | селфсигнед <br> пользовательский | селфсигнед |

<br>

### <a name="deploy-using-azure-cli"></a>Развертывание с помощью Azure CLI

> [!NOTE] 
> Для выполнения следующей команды необходим интерфейс командной строки версии не ниже 2.0.8. Узнать свою версию CLI можно с помощью команды `az --version`. Чтобы обновить версию CLI, ознакомьтесь со статьей [Установка Azure CLI](/cli/azure/install-azure-cli).

В примере ниже кластер OpenShift и все связанные ресурсы развертываются в группу ресурсов openshiftrg с именем развертывания myOpenShiftCluster. Репозиторий GitHub ссылается непосредственно на шаблон, при этом используется локальный файл параметров с именем azuredeploy.parameters.json.

```azurecli 
az deployment group create -g openshiftrg --name myOpenShiftCluster \
      --template-uri https://raw.githubusercontent.com/Microsoft/openshift-container-platform/master/azuredeploy.json \
      --parameters @./azuredeploy.parameters.json
```

Для завершения развертывания требуется не менее 60 минут в зависимости от общего числа развернутых и настроенных узлов. После завершения развертывания в терминале отобразятся полное доменное имя DNS узла-бастиона и URL-адрес консоли OpenShift.

```json
{
  "Bastion DNS FQDN": "bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com",
  "OpenShift Console URL": "http://openshiftlb.eastus.cloudapp.azure.com/console"
}
```

Если вы не хотите, чтобы окно командной строки оставалось ожидать завершения развертывания, добавьте `--no-wait` в число параметров при развертывании группы. Выходные данные из развертывания можно получить на портале Azure в разделе развертывания для группы ресурсов.

## <a name="connect-to-the-openshift-cluster"></a>Подключение к кластеру OpenShift

Когда завершится развертывание, получите данные для подключения из раздела выходных данных развертывания. Подключитесь к консоли OpenShift в браузере с помощью **URL-адреса консоли OpenShift**. Вы также можете SSH-подключение к узлу бастиона. В следующем примере в качестве имени администратора указано clusteradmin, а для полного доменного имени DNS с общедоступным IP-адресом узла-бастиона — bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com.

```bash
$ ssh clusteradmin@bastiondns4hawllzaavu6g.eastus.cloudapp.azure.com
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если группа ресурсов, кластер OpenShift и все связанные ресурсы больше не нужны, их можно удалить с помощью команды [az group delete](/cli/azure/group).

```azurecli 
az group delete --name openshiftrg
```

## <a name="next-steps"></a>Дальнейшие действия

- [Задачи, выполняемые после развертывания](./openshift-container-platform-3x-post-deployment.md)
- [Устранение неполадок с развертыванием OpenShift в Azure](./openshift-container-platform-3x-troubleshooting.md)
- [Overview](https://docs.openshift.com) (Обзор)

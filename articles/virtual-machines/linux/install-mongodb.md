---
title: Установка MongoDB на виртуальной машине Linux с помощью Azure CLI
description: Узнайте, как установить и настроить MongoDB на виртуальной машине Linux с помощью Azure CLI.
author: cynthn
manager: gwallace
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/15/2017
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: e3bc8ed2745e06096e05f17319a8f7896f87f80f
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/20/2020
ms.locfileid: "97702044"
---
# <a name="how-to-install-and-configure-mongodb-on-a-linux-vm"></a>Как установить и настроить MongoDB на виртуальной машине Linux

[MongoDB](https://www.mongodb.org) — это популярная высокопроизводительная база данных NoSQL с открытым кодом. В этой статье показано, как установить и настроить MongoDB на виртуальной машине Linux с помощью Azure CLI. Изучив представленные примеры, вы узнаете, как:

* [установить и настроить базовый экземпляр MongoDB вручную](#manually-install-and-configure-mongodb-on-a-vm);
* [создать базовый экземпляр MongoDB на основе шаблона Resource Manager](#create-basic-mongodb-instance-on-centos-using-a-template);
* [создать сложный сегментированный кластер MongoDB с набором реплик с использованием шаблона Resource Manager](#create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template).


## <a name="manually-install-and-configure-mongodb-on-a-vm"></a>Установка и настройка MongoDB на виртуальной машине вручную
База данных MongoDB [содержит инструкции по установке](https://docs.mongodb.com/manual/administration/install-on-linux/) для дистрибутивов Linux, в том числе Red Hat, CentOS, SUSE, Ubuntu и Debian. В следующем примере создается виртуальная машина *CentOS*. Для создания этой среды необходимо установить последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войти в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index).

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). В следующем примере создается виртуальная машина с именем *myVM* и именем пользователя *azureuser*, использующая аутентификацию с открытым ключом SSH.

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image CentOS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Подключитесь к виртуальной машине по протоколу SSH с помощью имени пользователя и адреса `publicIpAddress`, указанного в результатах, полученных на предыдущем шаге.

```bash
ssh azureuser@<publicIpAddress>
```

Чтобы добавить источники установки для MongoDB, создайте файл репозитория **yum**, как показано ниже:

```bash
sudo touch /etc/yum.repos.d/mongodb-org-3.6.repo
```

Откройте файл репозитория MongoDB для редактирования, например с помощью `vi` или `nano`. Добавьте следующие строки.

```sh
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

Установите MongoDB, используя **yum**, как показано ниже:

```bash
sudo yum install -y mongodb-org
```

По умолчанию в образах CentOS принудительно используется SELinux. Это препятствует доступу к MongoDB. Установите средства управления политиками и настройте SELinux так, чтобы база данных MongoDB могла использовать TCP-порт 27017 по умолчанию:

```bash
sudo yum install -y policycoreutils-python
sudo semanage port -a -t mongod_port_t -p tcp 27017
```

Запустите службу MongoDB следующим образом:

```bash
sudo service mongod start
```

Проверьте установленную базы данных MongoDB, подключившись к ней с помощью локального клиента `mongo`:

```bash
mongo
```

Теперь проверьте экземпляр MongoDB, добавив некоторые данные и выполнив их поиск:

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```

При необходимости настройте автоматический запуск MongoDB при перезагрузке системы:

```bash
sudo chkconfig mongod on
```


## <a name="create-basic-mongodb-instance-on-centos-using-a-template"></a>Создание базового экземпляра MongoDB на виртуальной машине CentOS с использованием шаблона
На виртуальной машине CentOS можно создать базовый экземпляр MongoDB, используя следующий шаблон быстрого запуска Azure из GitHub. В этом шаблоне используется расширение настраиваемых скриптов для Linux, что позволяет добавить в созданную виртуальную машину CentOS репозиторий **yum**, а затем установить MongoDB.

* [Базовый экземпляр MongoDB в CentOS](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-on-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json

Для создания этой среды необходимо установить последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войти в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index). Сначала создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

Затем разверните шаблон MongoDB с помощью команды [AZ Deployment Group Create](/cli/azure/deployment/group). При появлении запроса введите свои уникальные значения для *newStorageAccountName*, *dnsNameForPublicIP*, а также имя пользователя и пароль администратора:

```azurecli
az deployment group create --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json
```

Войдите на виртуальную машину с помощью ее общедоступного DNS-адреса. Его можно просмотреть с помощью команды [az vm show](/cli/azure/vm).

```azurecli
az vm show -g myResourceGroup -n myLinuxVM -d --query [fqdns] -o tsv
```

Подключитесь к виртуальной машине по протоколу SSH с помощью имени пользователя и общедоступного DNS-адреса.

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

Проверьте установку базы данных MongoDB, подключившись к ней с помощью локального клиента `mongo`, как показано ниже:

```bash
mongo
```

Теперь проверьте экземпляр, добавив некоторые данные и выполнив их поиск, как показано ниже:

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```


## <a name="create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template"></a>Создание сложного сегментированного кластера MongoDB на виртуальной машине CentOS с использованием шаблона
Используя следующий шаблон быстрого запуска Azure из GitHub, можно создать сложный сегментированный кластер MongoDB. Этот шаблон соответствует [рекомендациям для сегментированного кластера MongoDB](https://docs.mongodb.com/manual/core/sharded-cluster-components/) в отношении избыточности и высокой доступности. Он предусматривает создание двух сегментов с тремя узлами в каждом наборе реплик. Кроме того, он создает набор реплик сервера конфигурации и два сервера маршрутизации **mongos**. Это позволяет обеспечить согласованность приложений из разных сегментов.

* [MongoDB сегментированный кластер на CentOS](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json

> [!WARNING]
> Для развертывания сложного сегментированного кластера MongoDB требуется более 20 ядер. Обычно 20 ядер — это количество по умолчанию для региона, выделяемое на одну подписку. Отправьте запрос в службу поддержки Azure, чтобы увеличить количество ядер.

Для создания этой среды необходимо установить последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войти в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index). Сначала создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli
az group create --name myResourceGroup --location eastus
```

Затем разверните шаблон MongoDB с помощью команды [AZ Deployment Group Create](/cli/azure/deployment/group). Определите необходимые имена и размеры ресурсов, например *mongoAdminUsername*, *sizeOfDataDiskInGB* и *configNodeVmSize*:

```azurecli
az deployment group create --resource-group myResourceGroup \
  --parameters '{"adminUsername": {"value": "azureuser"},
    "adminPassword": {"value": "P@ssw0rd!"},
    "mongoAdminUsername": {"value": "mongoadmin"},
    "mongoAdminPassword": {"value": "P@ssw0rd!"},
    "dnsNamePrefix": {"value": "mypublicdns"},
    "environment": {"value": "AzureCloud"},
    "numDataDisks": {"value": "4"},
    "sizeOfDataDiskInGB": {"value": 20},
    "centOsVersion": {"value": "7.0"},
    "routerNodeVmSize": {"value": "Standard_DS3_v2"},
    "configNodeVmSize": {"value": "Standard_DS3_v2"},
    "replicaNodeVmSize": {"value": "Standard_DS3_v2"},
    "zabbixServerIPAddress": {"value": "Null"}}' \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json \
  --name myMongoDBCluster \
  --no-wait
```

На то, чтобы развернуть и настроить все экземпляры виртуальной машины, может потребоваться более одного часа. Флаг `--no-wait` в конце предыдущей команды используется для возвращения управления командной строке после того, как развертывание шаблона будет принято платформой Azure. Затем можно просмотреть состояние развертывания с помощью команды [AZ Deployment Group Показать](/cli/azure/deployment/group). Приведенный ниже пример позволяет просмотреть состояние развернутой службы *myMongoDBCluster* в группе ресурсов *myResourceGroup*:

```azurecli
az deployment group show \
    --resource-group myResourceGroup \
    --name myMongoDBCluster \
    --query [properties.provisioningState] \
    --output tsv
```

## <a name="next-steps"></a>Дальнейшие действия
В этих примерах подключение к экземпляру MongoDB выполняется локально с помощью виртуальной машины. Чтобы подключится к экземпляру MongoDB из другой виртуальной машины или сети, [создайте соответствующие правила группы безопасности сети](nsg-quickstart.md).

В этих примерах в целях разработки развертывается основная среда MongoDB. Примените необходимые параметры конфигурации безопасности для среды. Дополнительные сведения о безопасности MongoDB см. на [этой странице](https://docs.mongodb.com/manual/security/).

Дополнительные сведения о создании с использованием шаблонов см. в статье [Общие сведения о диспетчере ресурсов Azure](../../azure-resource-manager/management/overview.md).

Для скачивания и выполнения скриптов на виртуальных машинах в шаблонах Azure Resource Manager используется расширение настраиваемых скриптов. Дополнительные сведения см. в статье [Использование расширения пользовательских сценариев Azure на виртуальных машинах Linux](../extensions/custom-script-linux.md).

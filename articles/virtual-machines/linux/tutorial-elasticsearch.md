---
title: Развертывание ElasticSearch на виртуальной машине разработки в Azure
description: Руководство. Установка Elastic Stack на виртуальной машине разработки Linux в Azure
services: virtual-machines
documentationcenter: virtual-machines
author: rloutlaw
manager: justhe
tags: azure-resource-manager
ms.service: virtual-machines
ms.collection: linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 10/11/2017
ms.author: routlaw
ms.openlocfilehash: b3213d7de9aee88486fa1db1388c51948fbde430
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102557817"
---
# <a name="install-the-elastic-stack-on-an-azure-vm"></a>Установка Elastic Stack на виртуальной машине в Azure

В этой статье рассматриваются способы развертывания [Elasticsearch](https://www.elastic.co/products/elasticsearch), [Logstash](https://www.elastic.co/products/logstash) и [Kibana](https://www.elastic.co/products/kibana) на виртуальной машине Ubuntu в Azure. Чтобы увидеть Elastic Stack в действии, вы можете подключиться к Kibana и работать с несколькими примерами данных журнала. 

Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины Ubuntu в группе ресурсов Azure.
> * Установка Elasticsearch, Logstash и Kibana на виртуальной машине.
> * Отправка примера данных Elasticsearch с помощью Logstash. 
> * Открытие портов и работа с данными в консоли Kibana.


 Это развертывание подходит для разработки с помощью Elastic Stack. Дополнительные сведения о стеке Elastic Stack, включая рекомендации для рабочей среды, см. в [документации по разработке в Elastic Stack](https://www.elastic.co/guide/index.html) и в статье [Run Elasticsearch on Azure](/azure/architecture/elasticsearch/) (Запуск Elasticsearch в Azure).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. 

В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm). 

В следующем примере создаются виртуальная машина *myVM* и ключи SSH, если они не существуют в расположении ключей по умолчанию. Чтобы использовать определенный набор ключей, используйте параметр `--ssh-key-value`.  

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

После создания виртуальной машины в Azure CLI отображается информация следующего вида. Запишите значение `publicIpAddress`. Этот адрес используется для доступа к виртуальной машине.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="ssh-into-your-vm"></a>SSH-подключение к виртуальной машине

Если вы не знаете общедоступный IP-адрес виртуальной машины, используйте команду [az network public-ip list](/cli/azure/network/public-ip):

```azurecli-interactive
az network public-ip list --resource-group myResourceGroup --query [].ipAddress
```

Используйте следующую команду для создания сеанса SSH с виртуальной машиной. Укажите правильный общедоступный IP-адрес виртуальной машины. В этом примере это IP-адрес *40.68.254.142*.

```bash
ssh azureuser@40.68.254.142
```

## <a name="install-the-elastic-stack"></a>Установка Elastic Stack

Импортируйте ключ подписи Elasticsearch и обновите список источников APT, чтобы включить репозиторий эластичных пакетов:

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

Установите виртуальную машину Java на виртуальную машину и настройте переменную JAVA_HOME. Это необходимо для работы компонентов Elastic Stack.

```bash
sudo apt update && sudo apt install openjdk-8-jre-headless
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Чтобы обновить источники пакетов Ubuntu и установить Elasticsearch, Kibana и Logstash, выполните команду ниже.

```bash
sudo apt update && sudo apt install elasticsearch kibana logstash   
```

> [!NOTE]
> Подробные инструкции по установке, включая макеты каталогов и первоначальную конфигурацию, содержатся в [документации по Elastic Stack](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html).

## <a name="start-elasticsearch"></a>Запуск Elasticsearch 

Запустите Elasticsearch на виртуальной машине с помощью следующей команды:

```bash
sudo systemctl start elasticsearch.service
```

Эта команда не возвращает выходные данные, поэтому убедитесь, что Elasticsearch запущен на виртуальной машине с помощью команды `curl`:

```bash
sudo curl -XGET 'localhost:9200/'
```

Если Elasticsearch запущен, результат будет выглядеть следующим образом:

```json
{
  "name" : "w6Z4NwR",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "SDzCajBoSK2EkXmHvJVaDQ",
  "version" : {
    "number" : "5.6.3",
    "build_hash" : "1a2f265",
    "build_date" : "2017-10-06T20:33:39.012Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

## <a name="start-logstash-and-add-data-to-elasticsearch"></a>Запуск Logstash и добавление данных в Elasticsearch

Запустите Logstash с помощью следующей команды:

```bash 
sudo systemctl start logstash.service
```

Протестируйте Logstash в интерактивном режиме, чтобы убедиться, что он работает правильно:

```bash
sudo /usr/share/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

Это базовый [конвейер](https://www.elastic.co/guide/en/logstash/5.6/pipeline.html) logstash, который выводит стандартные входные данные в стандартные выходные. 

```output
The stdin plugin is now waiting for input:
hello azure
2017-10-11T20:01:08.904Z myVM hello azure
```

Настройте Logstash для переадресации сообщений ядра с этой виртуальной машины в Elasticsearch. Создайте файл в пустом каталоге с именем `vm-syslog-logstash.conf` и вставьте следующую конфигурацию Logstash:

```Logstash
input {
    stdin {
        type => "stdin-type"
    }

    file {
        type => "syslog"
        path => [ "/var/log/*.log", "/var/log/*/*.log", "/var/log/messages", "/var/log/syslog" ]
        start_position => "beginning"
    }
}

output {

    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts  => "localhost:9200"
    }
}
```

Проверьте эту конфигурацию и отправьте данные системного журнала в Elasticsearch:

```bash
sudo /usr/share/logstash/bin/logstash -f vm-syslog-logstash.conf
```

Вы увидите, что записи системного журнала в вашем терминале выводятся по мере их отправки в Elasticsearch. Используйте `CTRL+C`, чтобы выйти из Logstash, как только вы отправили часть данных.

## <a name="start-kibana-and-visualize-the-data-in-elasticsearch"></a>Запуск Kibana и визуализация данных в Elasticsearch

Измените `/etc/kibana/kibana.yml` и IP-адрес, на котором Kibana ожидает передачи данных, чтобы получить к нему доступ из веб-браузера.

```bash
server.host: "0.0.0.0"
```

Запустите Kibana с помощью следующей команды:

```bash
sudo systemctl start kibana.service
```

Откройте порт 5601 с помощью Azure CLI, чтобы разрешить удаленный доступ к консоли Kibana:

```azurecli-interactive
az vm open-port --port 5601 --resource-group myResourceGroup --name myVM
```

Откройте консоль Kibana и нажмите кнопку **Создать**, чтобы создать индекс по умолчанию, основанный на данных системного журнала, отправленных ранее в Elasticsearch. 

![Снимок экрана: консоль Kibana, где выделена кнопка создания.](media/elasticsearch-install/kibana-index.png)

Щелкните вкладку **Поиск** на консоли Kibana для поиска, просмотра и фильтрации событий системного журнала.

![Обзор событий системного журнала в Kibana](media/elasticsearch-install/kibana-search-filter.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы развернули Elastic Stack на виртуальной машине разработки в Azure. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание виртуальной машины Ubuntu в группе ресурсов Azure.
> * Установка Elasticsearch, Logstash и Kibana на виртуальной машине.
> * Отправка примеров данных в Elasticsearch из Logstash 
> * Открытие портов и работа с данными в консоли Kibana.

---
title: Управление журналами потоков NSG с помощью Grafana
titleSuffix: Azure Network Watcher
description: Управляйте журналами потоков для групп безопасности сети и анализируйте их с помощью наблюдателя за сетями и Grafana.
services: network-watcher
documentationcenter: na
author: damendo
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/15/2017
ms.author: damendo
ms.openlocfilehash: d522d305c70214009b8aa2886d07d2d5403dd2b1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97656314"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-using-network-watcher-and-grafana"></a>Управление журналами потоков для групп безопасности сети и их анализ с помощью наблюдателя за сетями и Grafana

[Журналы потоков для групп безопасности сети (NSG)](network-watcher-nsg-flow-logging-overview.md) содержат сведения, которые могут быть полезны для анализа входящего и исходящего IP-трафика в сетевых интерфейсах. Эти журналы потоков отображают сведения о входящем и исходящем потоках на основе правила NSG, сетевой карте, к которой относится поток, 5 кортежах потока (IP-адрес назначения и исходный IP-адрес, порт назначения и исходный порт, протокол), а также сведения о разрешении и отклонении трафика.

Ведение журналов потоков можно включить для нескольких групп безопасности сети. Такой объем регистрируемых данных усложняет синтаксический анализ и получение аналитических сведений из журналов. В этой статье описывается решение для централизованного управления этими журналами потоков для NSG с помощью Grafana, графического средства с открытым исходным кодом, ElasticSearch, распределенного модуля поиска и аналитики, а также Logstash, конвейера обработки данных на стороне сервера с открытым исходным кодом.  

## <a name="scenario"></a>Сценарий

Журналы потоков для NSG можно включить с помощью наблюдателя за сетями. Они сохраняются в хранилище BLOB-объектов Azure. Подключаемый модуль Logstash подключается к этому хранилищу, обрабатывает журналы потоков и отправляет их в ElasticSearch.  Когда журналы потоков будут сохранены в ElasticSearch, вы сможете анализировать и визуализировать их на настраиваемых панелях мониторинга в Grafana.

![Наблюдатель за сетями NSG в Grafana](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig1.png)

## <a name="installation-steps"></a>Шаги установки

### <a name="enable-network-security-group-flow-logging"></a>Включение ведения журнала потоков для групп безопасности сети

В рамках данного сценария вам необходимо включить ведение журнала потоков по меньшей мере для одной группы безопасности сети в своей учетной записи. Инструкции по включению журналов потоков безопасности сети см. в следующей статье [Введение в ведение журнала потоков для групп безопасности сети](network-watcher-nsg-flow-logging-overview.md).

### <a name="setup-considerations"></a>Рекомендации по настройке

В этом примере Grafana, ElasticSearch и Logstash настраиваются на сервере Ubuntu 16.04 LTS, развернутом в Azure. Для запуска всех трех компонентов используется минимальная настройка — они выполняются на одной виртуальной машине. Такая конфигурация актуальна только для тестовых и некритических рабочих нагрузок. Можно разработать Logstash, Elasticsearch и Grafana для независимого масштабирования в нескольких экземплярах. Чтобы получить дополнительные сведения, мы рекомендуем ознакомиться с руководством для каждого из этих компонентов.

### <a name="install-logstash"></a>Установка Logstash

Logstash позволяет преобразовать формат журналов потоков из JSON в плоскую структуру на уровне кортежей потока.

1. Введите следующие команды, чтобы установить Logstash.

    ```bash
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```

2. Настройте в Logstash синтаксический анализ журналов потоков и их отправку в ElasticSearch. Создайте файл logstash.conf следующим образом:

    ```bash
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

3. Добавьте в этот файл содержимое ниже. Измените имя учетной записи хранения и ключ доступа, введя значения для своей учетной записи хранения:

   ```bash
    input {
      azureblob
      {
        storage_account_name => "mystorageaccount"
        storage_access_key => "VGhpcyBpcyBhIGZha2Uga2V5Lg=="
        container => "insights-logs-networksecuritygroupflowevent"
        codec => "json"
        # Refer https://docs.microsoft.com/azure/network-watcher/network-watcher-read-nsg-flow-logs
        # Typical numbers could be 21/9 or 12/2 depends on the nsg log file types
        file_head_bytes => 12
        file_tail_bytes => 2
        # Enable / tweak these settings when event is too big for codec to handle.
        # break_json_down_policy => "with_head_tail"
        # break_json_batch_count => 2
      }
    }
    filter {
      split { field => "[records]" }
      split { field => "[records][properties][flows]"}
      split { field => "[records][properties][flows][flows]"}
      split { field => "[records][properties][flows][flows][flowTuples]"}

      mutate {
        split => { "[records][resourceId]" => "/"}
        add_field => { "Subscription" => "%{[records][resourceId][2]}"
          "ResourceGroup" => "%{[records][resourceId][4]}"
          "NetworkSecurityGroup" => "%{[records][resourceId][8]}" 
        }
        convert => {"Subscription" => "string"}
        convert => {"ResourceGroup" => "string"}
        convert => {"NetworkSecurityGroup" => "string"}
        split => { "[records][properties][flows][flows][flowTuples]" => "," }
        add_field => {
          "unixtimestamp" => "%{[records][properties][flows][flows][flowTuples][0]}"
          "srcIp" => "%{[records][properties][flows][flows][flowTuples][1]}"
          "destIp" => "%{[records][properties][flows][flows][flowTuples][2]}"
          "srcPort" => "%{[records][properties][flows][flows][flowTuples][3]}"
          "destPort" => "%{[records][properties][flows][flows][flowTuples][4]}"
          "protocol" => "%{[records][properties][flows][flows][flowTuples][5]}"
          "trafficflow" => "%{[records][properties][flows][flows][flowTuples][6]}"
          "traffic" => "%{[records][properties][flows][flows][flowTuples][7]}"
    "flowstate" => "%{[records][properties][flows][flows][flowTuples][8]}"
    "packetsSourceToDest" => "%{[records][properties][flows][flows][flowTuples][9]}"
    "bytesSentSourceToDest" => "%{[records][properties][flows][flows][flowTuples][10]}"
    "packetsDestToSource" => "%{[records][properties][flows][flows][flowTuples][11]}"
    "bytesSentDestToSource" => "%{[records][properties][flows][flows][flowTuples][12]}"
        }
        add_field => {
          "time" => "%{[records][time]}"
          "systemId" => "%{[records][systemId]}"
          "category" => "%{[records][category]}"
          "resourceId" => "%{[records][resourceId]}"
          "operationName" => "%{[records][operationName}}"
          "Version" => "%{[records][properties][Version}}"
          "rule" => "%{[records][properties][flows][rule]}"
          "mac" => "%{[records][properties][flows][flows][mac]}"
        }
        convert => {"unixtimestamp" => "integer"}
        convert => {"srcPort" => "integer"}
        convert => {"destPort" => "integer"}
        add_field => { "message" => "%{Message}" }        
      }
 
      date {
        match => ["unixtimestamp" , "UNIX"]
      }
    }
    output {
      stdout { codec => rubydebug }
      elasticsearch {
        hosts => "localhost"
        index => "nsg-flow-logs"
      }
    }
   ```

Предоставленный файл конфигурации Logstash состоит из трех частей: данные входа, фильтр и данные выхода.
В разделе входа описан источник, из которого в Logstash поступают журналы для обработки. В нашем примере используется подключаемый модуль входа для "azureblob" (устанавливается на следующих этапах), который предоставляет доступ к файлам журнала потоков для групп безопасности сети. Файлы хранятся в формате JSON в хранилище BLOB-объектов. 

Затем в разделе фильтра каждый файл журнала потоков преобразовывается в плоскую структуру, чтобы каждый отдельный кортеж потока и связанные свойства стали отдельным событием Logstash.

В итоге раздел выходных данных перенаправляет каждое событие Logstash на сервер ElasticSearch. Вы можете изменить файл конфигурации Logstash в соответствии со своими потребностями.

### <a name="install-the-logstash-input-plugin-for-azure-blob-storage"></a>Установка подключаемого модуля ввода Logstash для хранилища BLOB-объектов Azure

Этот подключаемый модуль Logstash позволяет обращаться к журналам потоков непосредственно из настроенной для них учетной записи хранилища BLOB-объектов. Чтобы установить этот подключаемый модуль, из каталога установки Logstash по умолчанию (в этом случае — /usr/share/logstash/bin) выполните следующую команду:

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

Дополнительные сведения об этом подключаемом модуле Logstash для Azure Storage Blob см. [здесь](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob).

### <a name="install-elasticsearch"></a>Установка Elasticsearch

Чтобы установить ElasticSearch, можно использовать приведенный ниже скрипт. Сведения об установке ElasticSearch см. на [этой](https://www.elastic.co/guide/en/elastic-stack/current/index.html) странице.

```bash
apt-get install apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen -y
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://packages.elastic.co/elasticsearch/5.x/debian stable main" | tee -a /etc/apt/sources.list.d/elasticsearch-5.x.list
apt-get update && apt-get install elasticsearch
sed -i s/#cluster.name:.*/cluster.name:\ grafana/ /etc/elasticsearch/elasticsearch.yml
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```

### <a name="install-grafana"></a>Установка Grafana

Чтобы установить и запустить Grafana, выполните следующие команды:

```bash
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.5.1_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_4.5.1_amd64.deb
sudo service grafana-server start
```

Дополнительные сведения по установке см. на странице [Grafana Labs](https://docs.grafana.org/installation/debian/).

#### <a name="add-the-elasticsearch-server-as-a-data-source"></a>Добавление сервера ElasticSearch в качестве источника данных

Далее необходимо добавить индекс ElasticSearch, содержащий журналы потоков, в качестве источника данных. Источник данных можно добавить, выбрав параметр **Добавить источник данных** и заполнив соответствующую форму. Пример такой конфигурации приведен на снимке экрана ниже:

![Добавление источника данных](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig2.png)

#### <a name="create-a-dashboard"></a>Создание панели мониторинга

После успешной настройки Grafana для считывания индекса ElasticSearch, содержащего журналы потоков для NSG, вы можете создать и персонализировать панели мониторинга. Чтобы создать панель мониторинга, выберите команду **создания первой панели мониторинга**. В примере конфигурации графика ниже показаны потоки, сегментированные по правилу NSG.

![График панели мониторинга](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig3.png)

На снимке экрана ниже показаны график и диаграмма с основными потоками и их частотой. Потоки представлены по правилам NSG и по принятым решениям. Grafana имеет широкие возможности настройки, поэтому мы рекомендуем создавать панели мониторинга в соответствии со своими потребностями мониторинга. Стандартная панель мониторинга выглядит следующим образом:

![Снимок экрана, на котором показан пример конфигурации диаграммы с потоками, сегментированными по правилу NSG.](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig4.png)

## <a name="conclusion"></a>Заключение

Интеграция наблюдателя за сетями с ElasticSearch и Grafana обеспечивает удобное централизованное управление журналами потоков для групп безопасности сети и другими данными, а также возможность их визуализации. Grafana располагает рядом других мощных графических возможностей, которые можно использовать для дальнейшего управления журналами потоков, а также анализа сетевого трафика. Теперь, когда вы настроили экземпляр Grafana и подключили его к Azure, можно продолжить изучение других функций, предлагаемых Grafana.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об использовании [наблюдателя за сетями](network-watcher-monitoring-overview.md).


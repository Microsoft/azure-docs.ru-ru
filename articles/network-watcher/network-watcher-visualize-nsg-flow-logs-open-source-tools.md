---
title: Визуализация журналов потоков NSG — эластичный стек
titleSuffix: Azure Network Watcher
description: Управляйте журналами потоков для групп безопасности сети и анализируйте их в Azure с помощью Наблюдателя за сетями и Elastic Stack.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: aca8c75f262e472cbc770c052b86d6e760ee449a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026477"
---
# <a name="visualize-azure-network-watcher-nsg-flow-logs-using-open-source-tools"></a>Визуализация журналов потоков для групп безопасности сети Наблюдателя за сетями Azure с помощью инструментов с открытым кодом

Журналы потоков для групп безопасности сети содержат информацию о входящем и исходящем IP-трафике групп безопасности сети. Эти журналы потоков отображают сведения о входящем и исходящем потоках на основе правил, сетевой карте, к которой относится поток, 5 кортежах потока (исходные IP-адрес и порт, конечные IP-адрес и порт, тип протокола), а также информацию о том, был поток запрещен или разрешен.

Данные журналы потоков могут быть трудны для анализа вручную и получения информации. Однако существует несколько инструментов с открытым кодом, которые могут помочь визуализировать эти данные. В этой статье описывается решение для визуализации этих журналов с помощью Elastic Stack, что позволит быстро индексировать и визуализировать журналы потоков на панели мониторинга Kibana.

## <a name="scenario"></a>Сценарий

В этой статье мы настроим решение, которое позволит визуализировать журналы потоков для групп безопасности сети с помощью Elastic Stack.  Подключаемый модуль ввода Logstash будет получать журналы потоков непосредственно из большого двоичного объекта службы хранилища, настроенного для хранения журналов потоков. Затем с помощью Elastic Stack журналы потоков будут индексированы и использованы для создания панели мониторинга Kibana для отображения информации.

![На схеме показан сценарий, позволяющий визуализировать журналы потоков для групп безопасности сети с помощью эластичного стека.][scenario]

## <a name="steps"></a>Шаги

### <a name="enable-network-security-group-flow-logging"></a>Включение ведения журнала потоков для групп безопасности сети
В рамках данного сценария вам необходимо включить ведение журнала потоков по меньшей мере для одной группы безопасности сети в своей учетной записи. Инструкции по включению журналов потоков безопасности сети см. в следующей статье [Введение в ведение журнала потоков для групп безопасности сети](network-watcher-nsg-flow-logging-overview.md).

### <a name="set-up-the-elastic-stack"></a>Настройка Elastic Stack
Подключив журналы потоков NSG к Elastic Stack, мы сможем создать панель мониторинга Kibana, которая позволяет использовать журналы для поиска информации, создания диаграмм, анализа и изучения.

#### <a name="install-elasticsearch"></a>Установка Elasticsearch

1. Для Elastic Stack версии 5.0 и более поздних версий требуется Java 8. Выполните команду `java -version`, чтобы проверить установленную версию. Если компонент Java не установлен, обратитесь к документации [по поддерживаемым в Azure пакетам JDK](/azure/developer/java/fundamentals/java-jdk-long-term-support).
2. Скачайте правильный двоичный пакет для своей системы.

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.deb
   sudo dpkg -i elasticsearch-5.2.0.deb
   sudo /etc/init.d/elasticsearch start
   ```

   Другие методы установки можно найти в [документации по установке Elasticsearch](https://www.elastic.co/guide/en/beats/libbeat/5.2/elasticsearch-installation.html).

3. Убедитесь, что Elasticsearch выполняется, выполнив следующую команду.

    ```bash
    curl http://127.0.0.1:9200
    ```

    Вы должны увидеть ответ следующего вида.

    ```json
    {
    "name" : "Angela Del Toro",
    "cluster_name" : "elasticsearch",
    "version" : {
        "number" : "5.2.0",
        "build_hash" : "8ff36d139e16f8720f2947ef62c8167a888992fe",
        "build_timestamp" : "2016-01-27T13:32:39Z",
        "build_snapshot" : false,
        "lucene_version" : "6.1.0"
    },
    "tagline" : "You Know, for Search"
    }
    ```

Дополнительные инструкции по установке Elasticsearch можно найти на [странице установки](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/_installation.html).

### <a name="install-logstash"></a>Установка Logstash

1. Введите следующие команды, чтобы установить Logstash.

    ```bash
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```
2. Далее необходимо настроить Logstash для получения и анализа журналов потоков. Создайте файл logstash.conf следующим образом.

    ```bash
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

3. Добавьте в этот файл приведенное ниже содержимое.

   ```
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

     mutate{
      split => { "[records][resourceId]" => "/"}
      add_field => {"Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"}
      convert => {"Subscription" => "string"}
      convert => {"ResourceGroup" => "string"}
      convert => {"NetworkSecurityGroup" => "string"}
      split => { "[records][properties][flows][flows][flowTuples]" => ","}
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
      convert => {"unixtimestamp" => "integer"}
      convert => {"srcPort" => "integer"}
      convert => {"destPort" => "integer"}        
     }

     date{
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

Дополнительные инструкции по установке Logstash см. в [официальной документации](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html).

### <a name="install-the-logstash-input-plugin-for-azure-blob-storage"></a>Установка подключаемого модуля ввода Logstash для хранилища BLOB-объектов Azure

Этот подключаемый модуль Logstash позволит обращаться непосредственно к журналам потоков из заданной для них учетной записи хранения. Чтобы установить этот подключаемый модуль, из каталога установки Logstash по умолчанию (в этом случае — /usr/share/logstash/bin) выполните следующую команду.

```bash
logstash-plugin install logstash-input-azureblob
```

Чтобы запустить Logstash, выполните приведенную ниже команду.

```bash
sudo /etc/init.d/logstash start
```

Дополнительные сведения об этом подключаемом модуле см. в [документации](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob).

### <a name="install-kibana"></a>Установка Kibana

1. Введите следующие команды для установки Kibana.

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.2.0-linux-x86_64.tar.gz
   tar xzvf kibana-5.2.0-linux-x86_64.tar.gz
   ```

2. Запустите Kibana, выполнив следующие команды.

   ```bash
   cd kibana-5.2.0-linux-x86_64/
   ./bin/kibana
   ```

3. Чтобы открыть веб-интерфейс Kibana, перейдите по адресу `http://localhost:5601`.
4. В нашем сценарии для журналов потоков используется шаблон индексирования nsg-flow-logs. Его можно изменить в разделе output файла logstash.conf.
5. Чтобы получить возможность просматривать панель мониторинга Kibana удаленно, создайте правило NSG для доступа к **порту 5601**.

### <a name="create-a-kibana-dashboard"></a>Создание панели мониторинга Kibana

На следующем рисунке показан пример панели мониторинга, используемой для просмотра сведений и трендов в оповещениях.

![Рисунок 1][1]

Загрузка [файла панели мониторинга](https://aka.ms/networkwatchernsgflowlogdashboard), [файла визуализации](https://aka.ms/networkwatchernsgflowlogvisualizations) и [сохраненного файла поиска](https://aka.ms/networkwatchernsgflowlogsearch).

На вкладке **Management** (Управление) в Kibana перейдите к элементу **Saved Objects** (Сохраненные объекты) и импортируйте все три файла. Затем откройте вкладку **Dashboard** (Панель мониторинга) и загрузите пример панели мониторинга.

Вы можете также создать собственные визуализации и панели мониторинга, адаптированные к интересующим вас метрикам. Дополнительные сведения о визуализациях Kibana можно получить в [официальной документации по Kibana](https://www.tutorialspoint.com/kibana/kibana_create_visualization.htm).

### <a name="visualize-nsg-flow-logs"></a>Визуализация журналов потоков NSG

Пример панели мониторинга содержит несколько визуализаций для журналов потоков.

1. "Flows by Decision Over Time" (Потоки по решению с течением времени) и "Flows by Direction Over Time" (Потоки по направлению с течением времени) — диаграммы распределения во времени, показывающие число потоков за указанный период времени. Для этих визуализаций можно изменить единицу измерения времени и диапазон. Диаграмма "Flows by Decision Over Time" (Потоки по решению с течением времени) показывает соотношение принятых решений разрешить или запретить поток, а диаграмма "Flows by Direction Over Time" (Потоки по направлению с течением времени) — соотношение входящего и исходящего трафика. С помощью этих визуальных элементов можно проверять тенденции трафика с течением времени и искать какие-либо пики или нестандартные шаблоны.

   ![На снимке экрана показан пример панели мониторинга с последовательностью принятия решений и направлений с течением времени.][2]

2. "Flows by Destination" (Потоки по назначению) и "Flows by Source Port" (Потоки по сходному порту) — круговые диаграммы, показывающие распределение потоков по соответствующим портам. В этом представлении можно увидеть наиболее часто используемые порты. Если щелкнуть конкретный порт на круговой диаграмме, то на остальной части панели мониторинга отобразятся только потоки этого порта.

   ![На снимке экрана показан пример панели мониторинга с потоками по назначению и порту источника.][3]

3. "Number of Flows" (Число потоков) и "Earliest Log Time" (Время записи самого раннего журнала) — метрики, показывающие количество записанных потоков и дату записи самого раннего журнала.

   ![На снимке экрана показан пример панели мониторинга с числом потоков и самым ранним временем журнала.][4]

4. "Flows by NSG and Rule" (Потоки по NSG и правилу) — гистограмма, показывающая распределение потоков в каждой группе безопасности сети, а также распределение правил в каждой из этих групп. Здесь можно увидеть, какие NSG и правила создают большую часть трафика.

   ![На снимке экрана показан пример панели мониторинга с последовательностью по N S G и правилу.][5]

5. "Top 10 Source IPs" (10 наиболее используемых исходных IP-адресов) и "Top 10 Destination IPs" (10 наиболее используемых конечных IP-адресов) — гистограммы, показывающие 10 наиболее используемых исходных и конечных IP-адресов. Эти гистограммы можно настроить для отображения большего ли меньшего числа наиболее используемых IP-адресов. Здесь можно увидеть наиболее часто встречающиеся IP-адреса, а также решение по трафику (разрешить его или запретить), принятое для каждого IP-адреса.

   ![На снимке экрана показан пример панели мониторинга, показывающий первые десять адресов источника и назначения I P.][6]

6. "Flow Tuples" (Кортежи потоков) — в этой таблице показаны сведения, содержащиеся в каждом кортеже потока, а также соответствующие NSG и правило.

   ![На снимке экрана показаны кортежи потока в таблице.][7]

С помощью строки запроса в верхней части панели мониторинга можно отфильтровать ее содержимое по любому параметру потоков, включая идентификатор подписки, группы ресурсов, правило или любую другую интересующую вас переменную. Дополнительные сведения о запросах и фильтрах Kibana см. в [официальной документации](https://www.elastic.co/guide/en/beats/packetbeat/current/kibana-queries-filters.html).

## <a name="conclusion"></a>Заключение

Сочетая журналы потоков для групп безопасности сети с Elastic Stack, мы получили эффективный и настраиваемый способ визуализации сетевого трафика. Эти панели мониторинга позволяют быстро получить и предоставить информацию о сетевом трафике, а также отфильтровать ее и проверить на наличие каких-либо потенциальных нарушений. С помощью Kibana эти панели мониторинга можно адаптировать, чтобы создать определенные визуализации, соответствующие требованиям безопасности, аудита и соответствия.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со статьей [Visualizing Network Security Group flow logs with Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md) (Визуализация журналов потоков для групп безопасности сети с помощью Power BI).

<!--Image references-->

[scenario]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/scenario.png
[1]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure1.png
[2]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure2.png
[3]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure3.png
[4]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure4.png
[5]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure5.png
[6]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure6.png
[7]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure7.png
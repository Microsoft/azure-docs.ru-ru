---
title: Анализ журналов потоков для групп безопасности сети с помощью Graylog | Документация Майкрософт
description: Узнайте, как анализировать журналы потоков для групп безопасности сети и управлять ими в Azure с помощью Наблюдателя за сетями и Graylog.
services: network-watcher
documentationcenter: na
author: damendo
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2017
ms.author: damendo
ms.openlocfilehash: 8b363d90d05e95b017c3a655b57dbabc3712a155
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94965550"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-in-azure-using-network-watcher-and-graylog"></a>Анализ журналов потоков для групп безопасности сети и управление ими в Azure с помощью Наблюдателя за сетями и Graylog

[Журналы потоков для групп безопасности сети](network-watcher-nsg-flow-logging-overview.md) содержат информацию, которая позволяет анализировать входящий и исходящий IP-трафик сетевых интерфейсов Azure. Журналы потоков предоставляют следующие сведения о входящем и исходящем потоках: правила групп безопасности сети, данные сетевого интерфейса потока, 5 кортежей потока (IP-адрес и порт источника, IP-адрес и порт назначения, протокол) и сведения о разрешении или запрете трафика.

Ведение журналов потоков можно включить для нескольких групп безопасности сети. Если таких групп будет много, просмотр и анализ журналов сбудет сопровождаться трудностями. В этой статье предложено решение по централизованному управлению журналами потоков для групп безопасности сети на основе Graylog (средство с открытым кодом для анализа журналов и управления ими) и Logstash (серверный конвейер с открытым кодом для обработки данных).

> [!Warning]
> В следующих шагах используются журналы потоков версии 1. Дополнительные сведения см. в статье [Общие сведения о ведении журнала потоков для групп безопасности сети](network-watcher-nsg-flow-logging-overview.md). Приведенные ниже инструкции не будут работать с файлами журнала версии 2 без изменений.

## <a name="scenario"></a>Сценарий

Чтобы включить журналы потоков для группы безопасности сети, используется Наблюдатель за сетями. Журналы потоков передаются в хранилище BLOB-объектов Azure. Подключаемый модуль Logstash подключается к этому хранилищу, обрабатывает журналы потоков и отправляет их в Graylog. В Graylog вы сможете анализировать и визуализировать данные журналов на настраиваемых панелях мониторинга.

![Рабочий процесс Graylog](./media/network-watcher-analyze-nsg-flow-logs-graylog/workflow.png)

## <a name="installation-steps"></a>Процесс установки

### <a name="enable-network-security-group-flow-logging"></a>Включение журналов потоков для групп безопасности сети

В рамках этого сценария вам нужно включить ведение журнала потоков по меньшей мере для одной группы безопасности сети в учетной записи. Инструкции по включению журналов потоков для групп безопасности сети см. в статье [Introduction to flow logging for Network Security Groups](network-watcher-nsg-flow-logging-overview.md) (Общие сведения о журналах потоков для групп безопасности сети).

### <a name="setting-up-graylog"></a>Установка Graylog

В этом примере Graylog и Logstash настраиваются на сервере Ubuntu 14.04, развернутом в Azure.

- Просмотрите [документацию](https://docs.graylog.org/en/2.2/pages/installation/os/ubuntu.html) с пошаговыми инструкциями по установке Graylog в Ubuntu.
- Обязательно настройте веб-интерфейс Graylog, выполнив инструкции в [документации](https://docs.graylog.org/en/2.2/pages/configuration/web_interface.html#configuring-webif).

В нашем примере используется минимальная установка Graylog (т. е. один экземпляр Graylog). Но на основе Graylog можно создать масштабную систему для множества ресурсов, которая будет соответствовать конфигурации сети и производственным потребностям. Дополнительные сведения об архитектурных аспектах и подробные инструкции по разработке решений см. в [документации](https://docs.graylog.org/en/2.2/pages/architecture.html) Graylog и [руководстве по архитектуре](https://www.slideshare.net/Graylog/graylog-engineering-design-your-architecture).

Есть несколько способов установки Graylog. Их можно выбирать в соответствии с используемой платформой и предпочтениями. Полный список возможных методов см. в официальной [документации](https://docs.graylog.org/en/2.2/pages/installation.html) Graylog. Серверное приложение Graylog выполняется на дистрибутивах ОС Linux. Для его работы требуются следующие компоненты:

-  Java SE 8 или более поздней версии — [документация по JDK Azul для Azure](/azure/developer/java/fundamentals/java-jdk-long-term-support).
-  Elastic Search 2.x (2.1.0 или более поздней версии) (см. [документацию по установке Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/_installation.html));
-  MongoDB 2.4 или более поздней версии (см. [документацию по установке MongoDB](https://docs.mongodb.com/manual/administration/install-on-linux/)).

### <a name="install-logstash"></a>Установка Logstash

Logstash позволяет преобразовать формат журналов потоков из JSON в плоскую структуру на уровне кортежей потока. Такое преобразование упрощает размещение журналов в Graylog и поиск по ним.

1. Введите следующие команды, чтобы установить Logstash.

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
   sudo dpkg -i logstash-5.2.0.deb
   ```

2. Настройте в Logstash синтаксический анализ журналов потоков и их отправку в Graylog. Создайте файл logstash.conf.

   ```bash
   sudo touch /etc/logstash/conf.d/logstash.conf
   ```

3. Добавьте в этот файл содержимое ниже. Замените выделенные значения информацией об используемой учетной записи хранения.

   ```
    input {
        azureblob
        {
            storage_account_name => "mystorageaccount"
            storage_access_key => "NrUZmx7pJSKaRJzvQbeiZWi5nBRWOTr7Wwr9DrvK7YtDBrADYxT1y0oEExtSlkDnGRt7qcRiZzEBCCyRYND8SxSt"
            container => "insights-logs-networksecuritygroupflowevent"
            registry_create_policy => "start_over"
            codec => "json"
            file_head_bytes => 21
            file_tail_bytes => 9
            # Possible options: `do_not_break`, `with_head_tail`, `without_head_tail`
            break_json_down_policy  => 'with_head_tail'
            break_json_batch_count => 2
            interval => 5
        }
    }
    
    filter {
        split { field => "[records]" }
        split { field => "[records][properties][flows]"}
        split { field => "[records][properties][flows][flows]"}
        split { field => "[records][properties][flows][flows][flowTuples]"
    }
    
     mutate {
        split => { "[records][resourceId]" => "/"}
        add_field =>{
                    "Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"
        }
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
        udp {
            host => "127.0.0.1"
            port => 12201
        }
    }
    ```
   Предоставленный файл конфигурации Logstash состоит из трех частей: данные входа, фильтр и данные выхода. В разделе входа описан источник, из которого в Logstash поступают журналы для обработки. В нашем примере используется подключаемый модуль входа для блога Azure (устанавливается на следующих этапах), который предоставляет доступ к файлам журнала потоков для групп безопасности сети. Файлы хранятся в формате JSON в хранилище BLOB-объектов.

Затем в разделе фильтра каждый файл журнала потоков преобразовывается в плоскую структуру, чтобы каждый отдельный кортеж потока и связанные свойства стали отдельным событием Logstash.

Наконец, в разделе выходных данных каждое событие Logstash перенаправляется на сервер Graylog. Вы можете изменить файл конфигурации Logstash в соответствии со своими потребностями.

   > [!NOTE]
   > В предыдущем файле конфигурации предполагается, что сервер Graylog настроен с петлевым IP-адресом 127.0.0.1 локального узла. В противном случае измените параметр узла в разделе выходных данных, указав правильный IP-адрес.

Дополнительные инструкции по установке Logstash см. в [документации по Logstash](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html).

### <a name="install-the-logstash-input-plug-in-for-azure-blob-storage"></a>Установка подключаемого модуля входа Logstash для хранилища BLOB-объектов Azure

Подключаемый модуль Logstash позволяет обращаться к журналам потоков непосредственно из настроенной для них учетной записи хранения. Чтобы установить этот подключаемый модуль, из каталога установки Logstash по умолчанию (в нашем случае — /usr/share/logstash/bin) выполните следующую команду:

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

Просмотрите [документацию](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob) с дополнительными сведения об этом подключаемом модуле.

### <a name="set-up-connection-from-logstash-to-graylog"></a>Настройка подключения Logstash к Graylog

Теперь установлено подключение из Logstash к журналам потоков и настроен сервер Graylog. Осталось только настроить в Graylog прием входящих файлов журнала.

1. Откройте веб-интерфейс сервера Graylog по URL-адресу, который вы настроили для него ранее. Для доступа к интерфейсу просто откройте в веб-браузере страницу `http://<graylog-server-ip>:9000/`

2. Чтобы перейти к странице настройки, откройте раскрывающееся меню **System** (Система) на верхней панели навигации справа и нажмите кнопку **Inputs** (Входы).
   Также можно использовать элемент `http://<graylog-server-ip>:9000/system/inputs`

   ![Начало работы](./media/network-watcher-analyze-nsg-flow-logs-graylog/getting-started.png)

3. Чтобы активировать новый вход, выберите *GELF UDP* в раскрывающемся списке **Select Input** (Выбор входа) и заполните предложенную форму. GELF расшифровывается как "Graylog Extended Log Format" (расширенный формат журнала Greylog). Этот формат собственной разработки Graylog. Дополнительные сведения о преимуществах формата можно найти в [документации](https://docs.graylog.org/en/2.2/pages/gelf.html) Greylog.

   Обязательно привяжите вход к IP-адресу, на котором вы настроили сервер Greylog. Этот IP-адрес должен совпадать со значением в поле **host** (Узел), указанным в свойствах выхода UDP в файле конфигурации Logstash. По умолчанию используется порт *12201*. Убедитесь, что номер порта совпадает со значением в поле **port** (Порт), указанным в свойствах выхода UDP в файле конфигурации Logstash.

   ![На снимке экрана показаны входные данные Graylog с параметрами для запуска и поиска входных данных.](./media/network-watcher-analyze-nsg-flow-logs-graylog/inputs.png)

   Сразу после активации входа сведения о нем отобразятся в разделе **Local inputs** (Локальные входы), как показано на следующем изображении.

   ![Снимок экрана, на котором показан раздел локальных входных данных, содержащий входные данные, которые вы запустили.](./media/network-watcher-analyze-nsg-flow-logs-graylog/local-inputs.png)

   Просмотрите [документацию](https://docs.graylog.org/en/2.2/pages/sending_data.html#what-are-graylog-message-inputs) с дополнительными сведениями о входах для сообщений Graylog.

4. Чтобы начать передачу журналов потоков, после завершения настройки можно запустить Logstash с помощью команды `sudo systemctl start logstash.service`.

### <a name="search-through-graylog-messages"></a>Поиск по сообщениям Graylog

Подождите, пока сервер Graylog накопит сообщения, а затем попробуйте выполнить по ним поиск. Чтобы проверить, поступают ли сообщения на сервер Graylog, на странице настройки **Inputs** (Входы) нажмите кнопку **Show received messages** (Показать полученные сообщения) для созданного входа GELF UDP. Откроется примерно такой экран: 

![На снимке экрана показан сервер Graylog, отображающий результаты поиска, гистограмму и сообщения.](./media/network-watcher-analyze-nsg-flow-logs-graylog/histogram.png)

Щелкнув синюю ссылку %{Message}, можно развернуть любое сообщение и просмотреть сведения о кортежах этого потока, как показано на следующем изображении:

![На снимке экрана показаны подробные сведения о сообщении с сервера Graylog.](./media/network-watcher-analyze-nsg-flow-logs-graylog/messages.png)

Если вы не выберете конкретное поле, по умолчанию в поиск включаются данные из всех полей сообщения. Если вам нужны сложные параметры поиска (например, кортежи потоков с конкретного IP-адреса источника), можно использовать язык поисковых запросов Graylog, который [описан здесь](https://docs.graylog.org/en/2.2/pages/queries.html).

## <a name="analyze-network-security-group-flow-logs-using-graylog"></a>Анализ журналов потоков для групп безопасности сети с помощью Graylog

Когда система Graylog будет настроена и начнет работать, можно использовать ее функциональные возможности для анализа данных из журнала потоков. Один из методов такого анализа — создание определенного представления данных с помощью панели мониторинга.

### <a name="create-a-dashboard"></a>Создание панели мониторинга

1. В верхней панели навигации выберите **Dashboards** (Панели мониторинга) или перейдите к `http://<graylog-server-ip>:9000/dashboards/`.

2. Затем нажмите зеленую кнопку **Create dashboard** (Создать панель мониторинга) и заполните небольшую форму. В ней нужно ввести заголовок и описание панели мониторинга. Нажмите кнопку **Save** (Сохранить), чтобы создать новую панель мониторинга. Созданная панель мониторинга будет выглядеть так, как показано на следующем изображении:

    ![На снимке экрана показаны панели мониторинга сервера Graylog с параметрами для создания и изменения панелей мониторинга.](./media/network-watcher-analyze-nsg-flow-logs-graylog/dashboards.png)

### <a name="add-widgets"></a>Добавление мини-приложений

Вы можете открыть новую панель мониторинга, щелкнув ее заголовок. Пока она пуста. Давайте добавим на нее мини-приложения. Простым и удобным вариантом для начала работы с панелью мониторинга будет мини-приложение диаграмм **Quick Values** (Быстрые значения), в которых отображается список значений для выбранного поля и сведения об их распределении.

1. Вернитесь к результатам поиска по входу UDP, на который поступают журналы потоков, выбрав действие **Search** (Поиск) на верхней панели навигации.

2. На панели **Результаты поиска** в левой части экрана найдите вкладку **Поля**, где перечислены разные поля данных для каждого входящего сообщения кортежа в потоке.

3. Выберите любой параметр, который нужно визуализировать (здесь выбран IP-адрес источника). Щелкните синюю стрелку раскрывающегося списка слева от поля, чтобы отобразился полный список мини-приложений, и выберите элемент **Quick Values** (Быстрые значения) для создания мини-приложения. Результат будет выглядеть примерно так:

   ![Исходный IP-адрес](./media/network-watcher-analyze-nsg-flow-logs-graylog/srcip.png)

4. Здесь можно нажать кнопку **Add to dashboard** (Добавить на панель мониторинга), которая расположена в правом верхнем углу окна мини-приложения, и выбрать панель мониторинга, на которую нужно добавить этот элемент.

5. Вернитесь к панели мониторинга. Вы увидите на ней добавленное мини-приложение.

   Вы можете добавлять на панель мониторинга много других мини-приложений, например гистограммы и счетчики для отслеживания важных метрик. На следующем изображении представлен пример настроенной панели мониторинга.

   ![Панель мониторинга для журналов потоков](./media/network-watcher-analyze-nsg-flow-logs-graylog/flowlogs-dashboard.png)

    Дополнительные сведения о панелях мониторинга и других типах мини-приложений вы найдете в [документации](https://docs.graylog.org/en/2.2/pages/dashboards.html) Graylog.

Интеграция Наблюдателя за сетями с Graylog обеспечивает удобное централизованное управление журналами потоков для групп безопасности сети и позволяет визуализировать данные журналов. Graylog располагает рядом мощных возможностей, таких как потоки данных и предупреждения. Они помогают в управлении журналами потоков и анализе сетевого трафика. Мы завершили настройку системы Graylog и подключили ее к Azure. Теперь можно приступать к изучению ее функций.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в статье [Визуализация журналов потоков для групп безопасности сети с помощью Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md).
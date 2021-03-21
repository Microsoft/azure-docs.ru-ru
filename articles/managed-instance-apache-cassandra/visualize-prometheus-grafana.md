---
title: Настройка Grafana для визуализации метрик, выпущенных в Azure Управляемый экземпляр для Apache Cassandra
description: Узнайте, как установить и настроить Grafana на виртуальной машине, чтобы визуализировать метрики, выпущенные Управляемый экземпляр Azure для кластера Apache Cassandra.
author: TheovanKraay
ms.service: managed-instance-apache-cassandra
ms.topic: how-to
ms.date: 03/02/2021
ms.author: thvankra
ms.openlocfilehash: ed0ff343595429a4cb81fef280203f1180eeb098
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746586"
---
# <a name="configure-grafana-to-visualize-metrics-emitted-from-the-managed-instance-cluster"></a>Настройка Grafana для визуализации метрик, выпущенных из кластера управляемых экземпляров

> [!IMPORTANT]
> Служба "Управляемый экземпляр Azure для Apache Cassandra" сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

При развертывании Управляемый экземпляр Azure для кластера Apache Cassandra Служба подготавливает сервер, на котором размещены [Prometheus](https://prometheus.io/) , которые могут использоваться различными клиентскими средствами. Prometheus — это решение для мониторинга с открытым исходным кодом. Управляемый экземпляр будет выдавать метрики и хранить данные на 10 минут или 10 ГБ (при первом достижении порогового значения). В этой статье описывается, как настроить Grafana для визуализации метрик, выдаваемых из кластера управляемых экземпляров. Для визуализации метрик необходимы следующие задачи.

* Разверните виртуальную машину Ubuntu в виртуальной сети Azure, где имеется управляемый экземпляр.
* Установите [средство Grafana](https://grafana.com/grafana/) с открытым исходным кодом, чтобы создать панели мониторинга и визуализировать метрики, созданные из Prometheus.

## <a name="deploy-a-ubuntu-server"></a>Развертывание сервера Ubuntu

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Перейдите к группе ресурсов, в которой находится кластер управляемых экземпляров. Щелкните **Добавить** и выполните поиск образа **Ubuntu Server 18,04 LTS** :

   :::image type="content" source="./media/visualize-prometheus-grafana/select-ubuntu-image.png" alt-text="Найдите образ Ubuntu Server из портал Azure." border="true":::

1. Выберите образ и щелкните **создать**.

1. В колонке **Создание виртуальной машины** введите значения для следующих полей. для других полей можно оставить значения по умолчанию:

   * **Имя виртуальной машины** . Введите имя виртуальной машины.
   * **Регион** — выберите тот же регион, в котором развернута виртуальная сеть.

   :::image type="content" source="./media/visualize-prometheus-grafana/create-vm-ubuntu.png" alt-text="Заполните форму, чтобы создать виртуальную машину с помощью образа Ubuntu Server." border="true":::

1. На вкладке **сети** выберите виртуальную входящий сетевой трафик, в которой развернут управляемый экземпляр.

   :::image type="content" source="./media/visualize-prometheus-grafana/configure-networking-details.png" alt-text="Настройте сетевые параметры сервера Ubuntu." border="true":::

1. Чтобы создать сервер Grafana, нажмите кнопку **Обзор и создание** .

## <a name="install-grafana"></a>Установка Grafana

1. В портал Azure Откройте виртуальную сеть, в которой развернут управляемый экземпляр и сервер Grafana. Вы увидите экземпляр масштабируемого набора виртуальных машин с именем **Cassandra-переход (экземпляр 0)**. Эти метрики Prometheus размещаются в этом масштабируемом наборе виртуальных машин. Запишите IP-адрес этого экземпляра:

   :::image type="content" source="./media/visualize-prometheus-grafana/prometheus-instance-address.png" alt-text="Получение IP-адреса экземпляра Prometheus." border="true":::

1. Подключитесь к созданному серверу Ubuntu, используя [Azure CLI](../virtual-machines/linux/ssh-from-windows.md#ssh-clients) или предпочтительное клиентское средство для подключения по протоколу SSH.

1. После подключения к виртуальной машине необходимо установить и настроить Grafana для подключения к масштабируемому набору виртуальных машин, в котором расположены метрики. Откройте командную строку и введите `nano` команду, чтобы открыть текстовый редактор nano. Вставьте следующий скрипт в текстовый редактор, обязательно замените на `<prometheus IP address>` IP-адрес, записанный на предыдущем шаге:

   ```bash
   #!/bin/bash
   
   echo "Installing Grafana..."
   
   if ! $SSH dpkg -s grafana prometheus > /dev/null; then
       echo "Installing packages."
       echo 'deb https://packages.grafana.com/oss/deb stable main' | $SSH sudo tee /etc/apt/sources.list.d/grafana.list > /dev/null
       curl https://packages.grafana.com/gpg.key | $SSH sudo apt-key add -
       $SSH sudo apt-get update
       $SSH sudo apt-get install -y grafana prometheus
   else
       echo "Skipping package installation"
   fi
   
   echo "Configuring grafana"
   cat <<EOF | $SSH sudo tee /etc/grafana/provisioning/datasources/prometheus.yml
   apiVersion: 1
   datasources:
     - name: Prometheus
       type: prometheus
       url: https://<prometheus IP address>:9443
       jsonData:
         tlsSkipVerify: true
   EOF
   
   echo "Restarting Grafana"
   $SSH sudo systemctl enable grafana-server
   $SSH sudo systemctl restart grafana-server
   
   echo "Installing Grafana plugins"
   $SSH sudo grafana-cli plugins install natel-discrete-panel
   $SSH sudo grafana-cli plugins install grafana-polystat-panel
   $SSH sudo systemctl restart grafana-server
   ```

1. Введите, `ctrl + X` чтобы сохранить файл. Можно присвоить имя файлу `grafana.sh` .

1. Введите `./grafana.sh` команду в командной строке, чтобы установить Grafana.

1. После завершения установки Grafana будет доступен по **порту 3000** на IP-адресе сервера, как показано на следующем снимке экрана:

   :::image type="content" source="./media/visualize-prometheus-grafana/open-grafana-port.png" alt-text="Запустите Grafana по порту 3000." border="true":::

1. Вы можете выбрать из панелей мониторинга с открытым кодом, созданных для Apache Cassandra в Grafana, например JSON-файл с [обзором кластера](https://github.com/TheovanKraay/cassandra-exporter/blob/master/grafana/instaclustr/cluster-overview.json) . Скачайте и импортируйте определение JSON панели мониторинга в Grafana:

   :::image type="content" source="./media/visualize-prometheus-grafana/grafana-import.png" alt-text="Импорт определения JSON Grafana." border="true":::

   :::image type="content" source="./media/visualize-prometheus-grafana/grafana-upload-json.png" alt-text="Отправка определения JSON Grafana." border="true":::

1. Затем вы можете отслеживать кластер управляемого экземпляра Cassandra с помощью выбранной панели мониторинга:

   :::image type="content" source="./media/visualize-prometheus-grafana/monitor-cassandra-metrics.gif" alt-text="Просмотрите метрики управляемого экземпляра Cassandra на панели мониторинга." border="true":::

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как настроить панели мониторинга для визуализации метрик в Prometheus с помощью Grafana. Дополнительные сведения об Azure Управляемый экземпляр для Apache Cassandra см. в следующих статьях:

* [Обзор Управляемый экземпляр Azure для Apache Cassandra](introduction.md)
* [Развертывание управляемого кластера Apache Spark с помощью Azure Databricks (Предварительная версия)](deploy-cluster-databricks.md)

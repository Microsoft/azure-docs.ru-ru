---
title: Просмотр журналов и метрик с помощью Kibana и Grafana
description: Просмотр журналов и метрик с помощью Kibana и Grafana
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 12/08/2020
ms.topic: how-to
ms.openlocfilehash: cb53aba300b933c78d9ac2f5fc5cf8054f3413e3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104670007"
---
# <a name="view-logs-and-metrics-using-kibana-and-grafana"></a>Просмотр журналов и метрик с помощью Kibana и Grafana

Веб-панели мониторинга Kibana и Grafana предоставляются для получения аналитических сведений и ясности в пространствах имен Kubernetes, используемых службами данных с поддержкой Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]


## <a name="monitor-azure-sql-managed-instances-on-azure-arc"></a>Мониторинг управляемых экземпляров Azure SQL в службе "Дуга Azure"

Чтобы получить доступ к панелям мониторинга журналов и мониторинга для Управляемый экземпляр SQL с поддержкой Arc, выполните следующую `azdata` команду интерфейса командной строки.

```bash

azdata arc sql endpoint list -n <name of SQL instance>

```
Соответствующие панели мониторинга Grafana:

* "Метрики управляемого экземпляра Azure SQL"
* Метрики узла узла
* "Метрики главных модулей узлов"


> [!NOTE]
>  При появлении запроса на ввод имени пользователя и пароля введите имя пользователя и пароль, указанные во время создания контроллера данных Arc Azure.

> [!NOTE]
>  Появится сообщение с предупреждением о сертификате, поскольку сертификаты, используемые в предварительной версии, являются самозаверяющими сертификатами.


## <a name="monitor-azure-database-for-postgresql-hyperscale-on-azure-arc"></a>Мониторинг службы "база данных Azure для PostgreSQL" в службе "Дуга Azure"

Чтобы получить доступ к панелям мониторинга и мониторингу PostgreSQL для масштабирования, выполните следующую `azdata` команду интерфейса командной строки.

```bash

azdata arc postgres endpoint list -n <name of postgreSQL instance>

```

Соответствующие панели мониторинга postgreSQL:

* "Метрики postgres"
* "Метрики таблицы postgres"
* Метрики узла узла
* "Метрики главных модулей узлов"


## <a name="additional-firewall-configuration"></a>Дополнительная настройка брандмауэра

В зависимости от того, где развернут контроллер данных, может оказаться, что для доступа к конечным точкам Kibana и Grafana необходимо открыть порты в брандмауэре.

Ниже приведен пример того, как это сделать для виртуальной машины Azure. Это нужно сделать, если вы развернули Kubernetes с помощью скрипта.

Ниже описаны действия по созданию правила NSG для конечных точек Kibana и Grafana.

### <a name="find-the-name-of-the-nsg"></a>Поиск имени модели NSG.

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

### <a name="add-the-nsg-rule"></a>Добавление правила NSG

После получения имени NSG можно добавить правило с помощью следующей команды:

```azurecli
az network nsg rule create -n ports_30777 --nsg-name azurearcvmNSG --priority 600 -g azurearcvm-rg --access Allow --description 'Allow Kibana and Grafana ports' --destination-address-prefixes '*' --destination-port-ranges 30777 --direction Inbound --protocol Tcp --source-address-prefixes '*' --source-port-ranges '*'
```


## <a name="next-steps"></a>Дальнейшие действия
- Попробуйте [передать метрики и журналы в Azure Monitor](upload-metrics-and-logs-to-azure-monitor.md)
- Дополнительные сведения о Grafana:
   - [Начало работы](https://grafana.com/docs/grafana/latest/getting-started/getting-started)
   - [Основы Grafana](https://grafana.com/tutorials/grafana-fundamentals/#1)
   - [Учебники по Grafana](https://grafana.com/tutorials/grafana-fundamentals/#1)
- Дополнительные сведения о Kibana
   - [Введение](https://www.elastic.co/webinars/getting-started-kibana?baymax=default&elektra=docs&storm=top-video)
   - [Kibana Guide](https://www.elastic.co/guide/en/kibana/current/index.html)
   - [Общие сведения о панели мониторинга разверткой с визуализациями данных в Kibana](https://www.elastic.co/webinars/dashboard-drilldowns-with-data-visualizations-in-kibana/)
   - [Создание панелей мониторинга Kibana](https://www.elastic.co/webinars/how-to-build-kibana-dashboards/)


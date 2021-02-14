---
title: Создание группы серверов масштабирования базы данных Azure для PostgreSQL в службе "Дуга Azure"
description: Создание группы серверов масштабирования базы данных Azure для PostgreSQL в службе "Дуга Azure"
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 02/11/2021
ms.topic: how-to
ms.openlocfilehash: 4ff45eea8e07a282d8529c745344c11706bc27bb
ms.sourcegitcommit: d4734bc680ea221ea80fdea67859d6d32241aefc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2021
ms.locfileid: "100387994"
---
# <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Создание группы сервера с Гипермасштабированием PostgreSQL с поддержкой Azure Arc

В этом документе описаны шаги по созданию группы PostgreSQL Scale Server в службе "Дуга" Azure.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="getting-started"></a>Начало работы
Если вы уже знакомы с приведенными ниже разделами, этот абзац можно пропустить.
Прежде чем продолжить создание, необходимо ознакомиться с важными разделами.
- [Обзор служб данных с поддержкой дуги Azure](overview.md)
- [Режимы подключения и требования](connectivity.md)
- [Основные понятия конфигурации хранилища и хранилища Kubernetes](storage-configuration.md)
- [Модель ресурсов Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

Если вы предпочитаете работать без подготовки полной среды самостоятельно, начните работу с помощью [Azure Arc JumpStart](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) в службе Azure Kubernetes Service (AKS), AWS эластичной Kubernetes Service (ЕКС), Google Cloud Kubernetes Engine (гке) или на виртуальной машине Azure.


## <a name="login-to-the-azure-arc-data-controller"></a>Вход в контроллер данных ARC в Azure

Перед созданием экземпляра необходимо сначала войти в систему на контроллере данных ARC в Azure. Если вы уже вошли в контроллер данных, этот шаг можно пропустить.

```console
azdata login
```

Затем будет предложено ввести имя пользователя, пароль и пространство имен System.  

> Если вы использовали скрипт для создания контроллера данных, пространство имен должно быть **дугой** .

```console
Namespace: arc
Username: arcadmin
Password:
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Предварительный и временный шаг только для пользователей OpenShift

Реализуйте этот шаг перед переходом к следующему шагу. Чтобы развернуть группу серверов PostgreSQL в Red Hat OpenShift в проекте, отличном от используемого по умолчанию, необходимо выполнить следующие команды в кластере, чтобы обновить ограничения безопасности. Эта команда предоставляет необходимые привилегии учетным записям служб, которые будут выполнять группу серверов PostgreSQL. Ограничение контекста безопасности (SCC) **_Arc-Data-SCC_** добавляется при развертывании контроллера данных ARC в Azure.

```console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

_**Server-Group-Name** — это имя группы серверов, которая будет создана на следующем шаге._
   
Дополнительные сведения о SCC в OpenShift см. в [документации по OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html).
Теперь вы можете реализовать следующий шаг.

## <a name="create-an-azure-database-for-postgresql-hyperscale-server-group"></a>Создание группы серверов масштабирования базы данных Azure для PostgreSQL

Чтобы создать группу серверов "база данных Azure для PostgreSQL" в службе "Дуга" Azure, используйте следующую команду:

```console
azdata arc postgres server create -n <name> --workers <# worker nodes with #>=2> --storage-class-data <storage class name> --storage-class-logs <storage class name> --storage-class-backups <storage class name>

#Example
#azdata arc postgres server create -n postgres01 --workers 2
```

> [!IMPORTANT]
> - Класс хранения, используемый для резервного копирования (_--Storage-Class-Backups-СКБ_), по умолчанию использует класс хранения данных контроллера данных, если он не указан.
> - Чтобы восстановить группу серверов в отдельной группе серверов (например, восстановление до точки во времени), необходимо настроить группу серверов для использования постоянных виртуальных цепей с режимом доступа Реадвритемани. Это необходимо сделать при создании группы серверов. Его нельзя изменить после создания. Дополнительные сведения см. в статье:
>    - [Создание группы серверов, готовой к резервному копированию и восстановлению](backup-restore-postgresql-hyperscale.md#create-a-server-group-that-is-ready-for-backups-and-restores)
>    - [Ограничения PostgreSQL с поддержкой ARC в Azure](limitations-postgresql-hyperscale.md)


> [!NOTE]
> - **Доступны другие параметры командной строки.  Просмотрите полный список параметров, выполнив `azdata arc postgres server create --help` .**
>
> - Единица, принимаемая параметрами--Volume-Size-*, — это Kubernetes ресурсное количество (целое число, за которым следует одно из этих данных, достаточное (T, G, M, K, M) или их степень-два эквивалента (Ti, МВт, MI, KI)).
> - Длина имени не должна превышать 12 символов и соответствовать соглашениям об именовании DNS.
> - Вам будет предложено ввести пароль для пользователя с правами администратора _postgres_ Standard.  Интерактивную строку можно пропустить, задав `AZDATA_PASSWORD` переменную среды сеанса перед выполнением команды Create.
> - Если контроллер данных развернут с помощью AZDATA_USERNAME и AZDATA_PASSWORD переменных среды сеанса в одном сеансе терминала, то значения AZDATA_PASSWORD будут использоваться для развертывания группы серверов PostgreSQL. Если вы предпочитаете использовать другой пароль, то либо (1) обновите значение для AZDATA_PASSWORD или (2) удалите переменную среды AZDATA_PASSWORD или удалите ее значение, при создании группы серверов вам будет предложено ввести пароль в интерактивном режиме.
> - Имя пользователя-администратора по умолчанию для ядра СУБД PostgreSQL _postgres_ не может быть изменено на этом этапе.
> - Создание группы серверов PostgreSQL Scale не приводит к немедленной регистрации ресурсов в Azure. В рамках процесса отправки данных [инвентаризации](upload-metrics-and-logs-to-azure-monitor.md)  или [использования](view-billing-data-in-azure.md) ресурсов в Azure ресурсы будут созданы в Azure, а ресурсы будут доступны в портал Azure.



## <a name="list-your-azure-database-for-postgresql-server-groups-created-in-your-arc-setup"></a>Вывод списка групп серверов базы данных Azure для PostgreSQL, созданных в вашей настройке Arc

Чтобы просмотреть группы PostgreSQL Scale Server в службе "Дуга Azure", используйте следующую команду:

```console
azdata arc postgres server list
```


```output
Name        State     Workers
----------  --------  ---------
postgres01  Ready     2
```

## <a name="get-the-endpoints-to-connect-to-your-azure-database-for-postgresql-server-groups"></a>Получение конечных точек для подключения к группам серверов базы данных Azure для PostgreSQL

Чтобы просмотреть конечные точки для экземпляра PostgreSQL, выполните следующую команду:

```console
azdata arc postgres endpoint list -n <server group name>
```
Пример:
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

Конечную точку экземпляра PostgreSQL можно использовать для подключения к группе PostgreSQL Scale Server из избранного средства:  [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio), [Пгкли](https://www.pgcli.com/) PSQL, pgAdmin и т. д.

Если вы используете виртуальную машину Azure для тестирования, следуйте приведенным ниже инструкциям.

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Специальное примечание о развертывании виртуальных машин Azure

При использовании виртуальной машины Azure IP-адрес конечной точки не будет показывать _общедоступный_ IP-адрес. Чтобы узнать общедоступный IP-адрес, используйте следующую команду:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Затем можно объединить общедоступный IP-адрес с портом, чтобы установить соединение.

Также может потребоваться предоставить порт группы PostgreSQL Scale Server через шлюз сетевой безопасности (NSG). Чтобы разрешить трафик через (NSG), необходимо добавить правило, которое можно выполнить с помощью следующей команды:

Чтобы задать правило, необходимо знать имя NSG. Определить NSG можно с помощью следующей команды:

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

После получения имени NSG можно добавить правило брандмауэра, используя следующую команду. Приведенные здесь примеры значений создают правило NSG для порта 30655 и допускают подключение с **любого** исходного IP-адреса.  Это не рекомендуется по соображениям безопасности.  Вы можете лучше зафиксировать вещи, указав значение-Source-Address-префиксы, характерные для IP-адреса клиента или диапазон IP-адресов, который охватывает IP-адреса вашей команды или организации.

Замените значение параметра--destination-Port-Ranges на номер порта, полученный в приведенной выше команде "аздата Arc postgres Server List".

```azurecli
az network nsg rule create -n db_port --destination-port-ranges 30655 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="connect-with-azure-data-studio"></a>Подключение к Azure Data Studio

Откройте Azure Data Studio и подключитесь к экземпляру с помощью IP-адреса внешней конечной точки и номера порта выше, а также пароля, указанного во время создания экземпляра.  Если PostgreSQL недоступен в раскрывающемся списке *тип подключения* , можно установить расширение PostgreSQL, выполнив поиск по имени PostgreSQL на вкладке расширения.

> [!NOTE]
> Необходимо нажать кнопку [дополнительно] на панели подключения, чтобы ввести номер порта.

Помните, что при использовании виртуальной машины Azure потребуется _общедоступный_ IP-адрес, доступный с помощью следующей команды:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

## <a name="connect-with-psql"></a>Подключение с помощью psql

Чтобы получить доступ к группе серверов PostgreSQL Scale, передайте внешнюю конечную точку группы серверов PostgreSQL, полученной выше:

Теперь можно подключить любой из psql:

```console
psql postgresql://postgres:<EnterYourPassword>@10.0.0.4:30655
```

## <a name="next-steps"></a>Следующие шаги

- Ознакомьтесь с основными понятиями и руководствами по использованию службы "база данных Azure для PostgreSQL", чтобы распределить данные между несколькими узлами PostgreSQL, а также получить преимущества от всех возможностей службы "база данных Azure для PostgreSQL". :
    * [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
    * [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
    * [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
    * [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Проектирование базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Разработка панели мониторинга для аналитики в режиме реального времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* В приведенных выше документах пропустите разделы **Вход в портал Azure**& **создать базу данных Azure для PostgreSQL-Scale (Цитус)**. Реализуйте оставшиеся шаги в развертывании Azure ARC. Эти разделы относятся к службе "база данных Azure для PostgreSQL" (Цитус), предлагаемой в качестве службы PaaS в облаке Azure, но другие части документов напрямую применимы к вашей геомасштабированию Azure с включенной PostgreSQL.

- [Масштабирование группы серверов с Гипермасштабированием Базы данных Azure для PostgreSQL](scale-out-postgresql-hyperscale-server-group.md)
- [Основные понятия конфигурации хранилища и хранилища Kubernetes](storage-configuration.md)
- [Развертывание утверждений Постоянного тома](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Модель ресурсов Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

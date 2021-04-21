---
title: Создание группы сервера с Гипермасштабированием PostgreSQL с поддержкой Azure Arc
description: Создание группы сервера с Гипермасштабированием PostgreSQL с поддержкой Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 02/11/2021
ms.topic: how-to
ms.openlocfilehash: 046f9d80c034e1ac1f2e7ffe144b4f389861b043
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101687946"
---
# <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Создание группы сервера с Гипермасштабированием PostgreSQL с поддержкой Azure Arc

В этом документе описаны шаги по созданию группы серверов PostgreSQL с гипермасштабированием на основе Azure Arc.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="getting-started"></a>Начало работы
Если вы уже знакомы с нижеприведенными темами, этот абзац можно пропустить.
Прежде чем перейти к процессу создания, необходимо ознакомиться со следующими важными темами:
- [Обзор служб данных с поддержкой Azure Arc](overview.md)
- [Режимы подключения и требования](connectivity.md)
- [Основные понятия, относящиеся к конфигурации хранилища и хранилищу Kubernetes](storage-configuration.md)
- [Модель ресурса Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

Если вы предпочитаете действовать без самостоятельной подготовки полнофункциональной среды, начните работу с помощью статьи [Быстрый старт с Azure Arc](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) на основе Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) или в виртуальной машине Azure.


## <a name="login-to-the-azure-arc-data-controller"></a>Вход в систему контроллера данных Azure Arc

Перед созданием экземпляра сначала необходимо войти в систему контроллера данных Azure Arc. Если вы уже вошли в систему контроллера данных, этот шаг можно пропустить.

```console
azdata login
```

Вам будет предложено ввести имя пользователя, пароль и пространство имен системы.  

> Если для создания контроллера данных вы использовали скрипт, у пространства имен должно быть название **arc**

```console
Namespace: arc
Username: arcadmin
Password:
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Предварительный и временный шаг только для пользователей OpenShift
Выполните этот шаг, прежде чем переходить к следующему шагу. Для развертывания группы серверов с гипермасштабированием PostgreSQL на основе Red Hat OpenShift в проекте, отличном от используемого по умолчанию, необходимо выполнить следующие команды в кластере, чтобы обновить ограничения, касающиеся безопасности. Эта команда предоставляет необходимые привилегии учетным записям служб, на основе которых будет работать группа серверов с гипермасштабированием PostgreSQL. Ограничение контекста безопасности (SCC) arc-data-scc было добавлено при развертывании контроллера данных Azure Arc.

```Console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

**Server-group-name — это имя группы серверов, которая будет создана на следующем шаге.**

Дополнительные сведения об SCC в OpenShift см. в [Документации по OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html). Теперь вы можете выполнить следующий шаг.


## <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Создание группы сервера с Гипермасштабированием PostgreSQL с поддержкой Azure Arc

Чтобы создать группу серверов с гипермасштабированием PostgreSQL с поддержкой Azure Arc на контроллере данных Arc, используйте команду `azdata arc postgres server create`, в которую нужно добавить несколько параметров.

Дополнительные сведения обо всех параметрах, которые можно задавать во время создания, см. в выводе команды:
```console
azdata arc postgres server create --help
```

Ниже приведены основные параметры, которые следует учитывать.
- **Имя группы серверов**, которая развертывается. Укажите либо `--name`, либо `-n`, после чего должно следовать имя длиной не более 11 символов.

- **Версия подсистемы PostgreSQL**, которая развертывается: по умолчанию используется версия 12. Чтобы развернуть версию 12, можно либо опустить этот параметр, либо добавить один из следующих параметров: `--engine-version 12` или `-ev 12`. Чтобы развернуть версию 11, укажите `--engine-version 11` или `-ev 11`.

- **Количество рабочих узлов**, которые нужно развернуть до необходимого масштаба и обеспечить при этом для них оптимальную производительность. Прежде чем продолжить, ознакомьтесь с [основными понятиями гипермасштабирования Postgres](concepts-distributed-postgres-hyperscale.md). Чтобы указать количество рабочих узлов для развертывания, используйте параметр `--workers` или `-w`, за которым следует целое число, равное или больше 2. Например, если требуется развернуть группу серверов с двумя рабочими узлами, укажите `--workers 2` или `-w 2`. Это приведет к созданию трех модулей Pod: один для координирующего узла или экземпляра, и два для рабочих узлов и экземпляров (по одному для каждого из них).

- **Классы хранения**, которые должны использоваться группой серверов. Важно задать класс хранения непосредственно во время развертывания группы серверов, так как после развертывания эту настройку изменить нельзя. Если изменять класс хранения после развертывания, нужно будет извлечь данные, удалить группу серверов, создать новую группу серверов и импортировать данные. Вы можете указать классы хранения, предназначенные для данных, журналов и резервных копий. По умолчанию, если классы хранения не указаны, будут использоваться классы хранения контроллера данных.
    - Чтобы задать класс хранения для данных, укажите параметр `--storage-class-data` или `-scd`, за которым следует имя класса хранения.
    - Чтобы задать класс хранения для журналов, укажите параметр `--storage-class-logs` или `-scl`, за которым следует имя класса хранения.
    - В этой предварительной версии PostgreSQL с поддержкой Azure Arc и гипермасштабированием класс хранения для резервных копий можно задать двумя способами в зависимости от того, какие типы операций резервного копирования и восстановления нужно выполнять. Мы работаем над тем, чтобы упростить этот процесс. Вам нужно указать либо класс хранения, либо подключение с утверждением тома. Подключение с утверждением тома — это комбинация из существующего утверждения постоянного тома (в том же пространстве имен) и типа тома (и необязательные метаданные в зависимости от типа тома), которые разделены двоеточием. Постоянный том будет подключен к каждому модулю Pod для группы серверов PostgreSQL.
        - Если планируется выполнять только полное восстановление базы данных, задайте параметр `--storage-class-backups` или `-scb`, за которым следует имя класса хранения.
        - Если планируется выполнить полное восстановление базы данных и восстановление на определенный момент времени, задайте параметр `--volume-claim-mounts` или `-vcm`, после которого укажите имя утверждения тома и тип тома.

Обратите внимание, что при выполнении команды "create" вам будет предложено ввести пароль пользователя с правами администратора `postgres` по умолчанию. В этой предварительной версии имя этого пользователя изменить нельзя. Вы можете отключить интерактивную подсказку, задав переменную среды сеанса `AZDATA_PASSWORD` перед выполнением команды "create".

### <a name="examples"></a>Примеры

**Чтобы развернуть группу серверов Postgres версии 12 с именем "postgres01" и с 2 рабочими узлами, которые используют такие же классы хранения, что и контроллер данных, выполните следующую команду:**
```console
azdata arc postgres server create -n postgres01 --workers 2
```

**Чтобы развернуть группу серверов Postgres версии 12 с именем "postgres01" и с 2 рабочими узлами, которые для данных и журналов используют такие же классы хранения, что и контроллер данных, но при этом собственный определенный класс хранения используется для полного восстановления и для восстановления на конкретный момент времени, выполните следующие действия:**

 В этом примере предполагается, что группа серверов размещена в кластере Azure Kubernetes Service (AKS). В этом примере в качестве имени класса хранилища используется обозначение "azurefile-premium". Приведенный ниже пример можно изменить в соответствии с вашей средой. Обратите внимание, что для этой конфигурации **требуется настройка accessModes ReadWriteMany**.  

Создайте сначала файл YAML, содержащий приведенное ниже описание резервной копии PVC, и назовите его, например, CreateBackupPVC.yml:
```console
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
  namespace: arc
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 100Gi
  storageClassName: azurefile-premium
```

Затем создайте PVC, используя определение, содержащееся в файле YAML:

```console
kubectl create -f e:\CreateBackupPVC.yml -n arc
``` 

Далее создайте группу серверов:

```console
azdata arc postgres server create -n postgres01 --workers 2 -vcm backup-pvc:backup
```

> [!IMPORTANT]
> - Ознакомьтесь с [текущими ограничениями, относящимися к резервному копированию и восстановлению](limitations-postgresql-hyperscale.md#backup-and-restore).


> [!NOTE]  
> - Если вы развернули контроллер данных с помощью переменных среды сеанса `AZDATA_USERNAME` и `AZDATA_PASSWORD` в одном сеансе терминала, тогда эти значения для `AZDATA_PASSWORD` будут использоваться при развертывании группы серверов PostgreSQL с гипермасштабированием. Если вы хотите использовать другой пароль, то либо (1) обновите значение параметра `AZDATA_PASSWORD`, либо (2) удалите переменную среды `AZDATA_PASSWORD`, либо (3) удалите его значение, чтобы при создании группы серверов вам было предложено ввести пароль в интерактивном режиме.
> - Создание группы серверов PostgreSQL с гипермасштабированием не приводит к немедленной регистрации ресурсов в Azure. В рамках процесса отправки в Azure [данных инвентаризации ресурсов](upload-metrics-and-logs-to-azure-monitor.md)  или [данных об использовании](view-billing-data-in-azure.md) необходимые ресурсы будут создаваться в Azure, и эти ресурсы можно просматривать на портале Azure.



## <a name="list-the-postgresql-hyperscale-server-groups-deployed-in-your-arc-data-controller"></a>Вывод списка групп серверов PostgreSQL с гипермасштабированием, развернутых в контроллере данных Arc

Чтобы вывести список групп серверов PostgreSQL с гипермасштабированием, развернутых в контроллере данных Arc, выполните следующую команду:

```console
azdata arc postgres server list
```


```output
Name        State     Workers
----------  --------  ---------
postgres01  Ready     2
```

## <a name="get-the-endpoints-to-connect-to-your-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Получение конечных точек для подключения к группам серверов PostgreSQL с поддержкой Azure Arc и с гипермасштабируемостью

Чтобы посмотреть конечные точки для группы серверов PostgreSQL, выполните следующую команду:

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

Конечную точку экземпляра PostgreSQL можно использовать для подключения к группе серверов PostgreSQL с гипермасштабированием посредством предпочитаемого вами инструмента: [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio), [pgcli](https://www.pgcli.com/) psql, pgAdmin и т. д. При этом выполняется подключение к узлу или экземпляру координатора, который выполняет маршрутизацию запроса к соответствующим рабочим узлам или экземплярам, если были созданы распределенные таблицы. Дополнительные сведения см. в статье [Основные понятия по теме PostgreSQL с поддержкой Azure Arc и гипермасштабированием](concepts-distributed-postgres-hyperscale.md).

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Специальное примечание о развертываниях виртуальных машин Azure

При использовании виртуальной машины Azure в IP-адресе конечной точки не отображается _общедоступный_ IP-адрес. Чтобы получить общедоступный IP-адрес, используйте следующую команду:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Затем можно объединить общедоступный IP-адрес с портом, чтобы выполнить подключение.

Также может понадобиться сделать порт группы серверов PostgreSQL с гипермасштабированием, доступным через шлюз безопасности сети (NSG). Чтобы разрешить передачу трафика через (NSG), необходимо добавить соответствующее правило с помощью следующей команды:

Чтобы задать правило, нужно знать имя используемого NSG. Получить информацию об NSG можно с помощью следующей команды:

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

После получения имени NSG можно добавить правило для брандмауэра, используя следующую команду. В этом примере создается правило NSG для порта 30655 и разрешается подключение с **любого** исходного IP-адреса.  Делать это не рекомендуется по соображениям безопасности!  Вы можете более эффективно выполнять блокирование, указав значение -source-address-prefixes, уникальное для IP-адреса клиента, или диапазон IP-адресов, который охватывает IP-адреса вашей команды или организации.

Замените значение параметра --destination-port-ranges номером порта, полученным с помощью приведенной выше команды azdata arc postgres server list.

```azurecli
az network nsg rule create -n db_port --destination-port-ranges 30655 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="connect-with-azure-data-studio"></a>Подключение к Azure Data Studio

Откройте Azure Data Studio и подключитесь к вашему экземпляру с помощью IP-адреса внешней конечной точки и заданного ранее номера порта, а также пароля, указанного во время создания экземпляра.  Если PostgreSQL недоступен в раскрывающемся списке *Тип подключения*, расширение PostgreSQL можно установить, выполнив поиск по названию PostgreSQL на вкладке расширений.

> [!NOTE]
> Чтобы ввести номер порта, нужно нажать кнопку [Дополнительно] на панели подключения.

Следует иметь в виду, что при использовании виртуальной машины Azure потребуется _общедоступный_ IP-адрес, получить который можно с помощью следующей команды:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

## <a name="connect-with-psql"></a>Подключение с помощью psql

Чтобы получить доступ к группе серверов PostgreSQL с гипермасштабированием, нужно указать в настройках внешнюю конечную точку группы серверов PostgreSQL с гипермасштабированием, полученную на одном из предыдущих шагов:

Теперь можно подключить любой из клиентов psql:

```console
psql postgresql://postgres:<EnterYourPassword>@10.0.0.4:30655
```

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с основными понятиями и руководствами, касающимися службы "База данных Azure для PostgreSQL с гипермасштабированием", чтобы распределить данные между несколькими узлами PostgreSQL с гипермасштабированием и реализовать преимущества, связанные с лучшей производительностью:
    * [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
    * [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
    * [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
    * [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Разработка базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Разработка панели мониторинга для аналитики в реальном времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* В приведенных выше документах можно пропустить разделы **Вход на портал Azure** и **Создание базы данных Azure для PostgreSQL — гипермасштабирование (Citus)** . Выполните оставшиеся шаги по развертыванию Azure Arc. Эти разделы относятся к службе "База данных Azure для PostgreSQL с гипермасштабированием" (Citus), предоставляемой в качестве службы PaaS в облаке Azure, но другие части указанных документов напрямую применимы к технологии "PostgreSQL с поддержкой Azure Arc и с гипермасштабированием".

- [Масштабирование группы серверов PostgreSQL с поддержкой Azure Arc и с гипермасштабированием](scale-out-postgresql-hyperscale-server-group.md)
- [Основные понятия, относящиеся к конфигурации хранилища и хранилищу Kubernetes](storage-configuration.md)
- [Расширение утверждений постоянного тома](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Модель ресурса Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

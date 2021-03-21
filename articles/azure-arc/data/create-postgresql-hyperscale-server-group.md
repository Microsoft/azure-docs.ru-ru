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
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101687946"
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
Реализуйте этот шаг перед переходом к следующему шагу. Чтобы развернуть группу серверов PostgreSQL в Red Hat OpenShift в проекте, отличном от используемого по умолчанию, необходимо выполнить следующие команды в кластере, чтобы обновить ограничения безопасности. Эта команда предоставляет необходимые привилегии учетным записям служб, которые будут выполнять группу серверов PostgreSQL. Ограничение контекста безопасности (SCC) Arc-Data-SCC добавляется при развертывании контроллера данных ARC в Azure.

```Console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

**Server-Group-name — это имя группы серверов, которая будет создана на следующем шаге.**

Дополнительные сведения о SCC в OpenShift см. в [документации по OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html). Теперь вы можете реализовать следующий шаг.


## <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Создание группы сервера с Гипермасштабированием PostgreSQL с поддержкой Azure Arc

Чтобы создать в контроллере данных Arc группу серверов PostgreSQL, поддерживающую дугу Azure, используйте команду, `azdata arc postgres server create` в которую будут переданы несколько параметров.

Дополнительные сведения о параметрах, которые можно задать во время создания, см. в выходных данных команды:
```console
azdata arc postgres server create --help
```

Ниже приведены основные параметры, которые следует учитывать.
- **имя группы серверов** , которую требуется развернуть. Укажите либо `--name` `-n` , за которым следует имя, длина которого не должна превышать 11 символов.

- **Версия подсистемы PostgreSQL** , которую необходимо развернуть: по умолчанию используется версия 12. Чтобы развернуть версию 12, можно либо опустить этот параметр, либо передать один из следующих параметров: `--engine-version 12` или `-ev 12` . Чтобы развернуть версию 11, укажите `--engine-version 11` или `-ev 11` .

- **количество рабочих узлов** , которые требуется развернуть в масштабном развертывании и которые могут достигать лучшего производительность. Прежде чем перейти к этому представлению, прочитайте [Основные понятия postgres Scale](concepts-distributed-postgres-hyperscale.md). Чтобы указать количество рабочих узлов для развертывания, используйте параметр `--workers` или, `-w` за которым следует целое число больше или равное 2. Например, если требуется развернуть группу серверов с двумя рабочими узлами, укажите `--workers 2` или `-w 2` . Это приведет к созданию трех модулей Pod, по одному для узла или экземпляра координатора, а также двух рабочих узлов и экземпляров (по одному для каждого из рабочих ролей).

- **классы хранения** , которые должна использовать группа серверов. Важно задать класс хранения непосредственно во время развертывания группы серверов, так как это невозможно изменить после развертывания. Если вы изменили класс хранения после развертывания, потребуется извлечь данные, удалить группу серверов, создать новую группу серверов и импортировать данные. Вы можете указать классы хранения данных, журналы и резервные копии. По умолчанию, если не указать классы хранения, будут использоваться классы хранилища контроллера данных.
    - чтобы задать класс хранения данных, укажите параметр `--storage-class-data` или, `-scd` за которым следует имя класса хранения.
    - чтобы задать класс хранения для журналов, укажите параметр `--storage-class-logs` или, `-scl` за которым следует имя класса хранения.
    - чтобы задать класс хранения для резервных копий: в этой предварительной версии службы "Дуга Azure с включенным PostgreSQL" можно задать классы хранения двумя способами в зависимости от того, какие типы операций резервного копирования и восстановления требуется выполнить. Мы работаем над упрощением этого процесса. Вы указываете класс хранилища или подключение к подсистеме тома. Подключение с утверждением тома — это пара существующего постоянного тома (в том же пространстве имен) и типа тома (и необязательные метаданные в зависимости от типа тома), разделенные двоеточием. Постоянный том будет подключен к каждому Pod для группы серверов PostgreSQL.
        - Если планируется выполнять только полное восстановление базы данных, задайте параметр `--storage-class-backups` или, `-scb` за которым следует имя класса хранения.
        - Если планируется выполнить полное восстановление базы данных и восстановление на момент времени, задайте параметр `--volume-claim-mounts` или укажите `-vcm` имя утверждения тома и тип тома.

Обратите внимание, что при выполнении команды CREATE вам будет предложено ввести пароль пользователя с правами администратора по умолчанию `postgres` . Имя этого пользователя нельзя изменить в этой предварительной версии. Вы можете пропустить интерактивную подсказку, задав `AZDATA_PASSWORD` переменную среды сеанса перед выполнением команды Create.

### <a name="examples"></a>Примеры

**Чтобы развернуть группу серверов postgres версии 12 с именем postgres01 с 2 рабочими узлами, использующими те же классы хранения, что и контроллер данных, выполните следующую команду:**
```console
azdata arc postgres server create -n postgres01 --workers 2
```

**Чтобы развернуть группу серверов postgres версии 12 с именем postgres01 с 2 рабочими узлами, использующими те же классы хранения, что и для данных и журналов, но его конкретный класс хранения для полного восстановления и восстановления на момент времени, выполните следующие действия.**

 В этом примере предполагается, что группа серверов размещена в кластере Azure Kubernetes Service (AKS). В этом примере используется азурефиле-Premium в качестве имени класса хранилища. Приведенный ниже пример можно изменить в соответствии с вашей средой. Обратите внимание, что для этой конфигурации **требуется Акцессмодес реадвритемани** .  

Сначала создайте файл YAML, содержащий приведенное ниже описание резервной PVC и назовите его Креатебаккуппвк. yml, например:
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

Затем создайте PVC, используя определение, хранящееся в файле YAML:

```console
kubectl create -f e:\CreateBackupPVC.yml -n arc
``` 

Затем создайте группу серверов.

```console
azdata arc postgres server create -n postgres01 --workers 2 -vcm backup-pvc:backup
```

> [!IMPORTANT]
> - Прочтите [текущие ограничения, связанные с резервным копированием и восстановлением](limitations-postgresql-hyperscale.md#backup-and-restore).


> [!NOTE]  
> - Если вы развернули контроллер данных с помощью `AZDATA_USERNAME` `AZDATA_PASSWORD` переменных среды сеанса и в одном сеансе терминала, значения `AZDATA_PASSWORD` будут использоваться для развертывания группы серверов PostgreSQL в масштабе. Если вы предпочитаете использовать другой пароль, то либо (1) обновите значение параметра `AZDATA_PASSWORD` или (2) удалите `AZDATA_PASSWORD` переменную среды, либо (3) удалите его значение, чтобы при создании группы серверов было предложено ввести пароль в интерактивном режиме.
> - Создание группы серверов PostgreSQL Scale не приводит к немедленной регистрации ресурсов в Azure. В рамках процесса отправки данных [инвентаризации](upload-metrics-and-logs-to-azure-monitor.md)  или [использования](view-billing-data-in-azure.md) ресурсов в Azure ресурсы будут созданы в Azure, а ресурсы будут доступны в портал Azure.



## <a name="list-the-postgresql-hyperscale-server-groups-deployed-in-your-arc-data-controller"></a>Вывод списка групп PostgreSQL Scale Server, развернутых в контроллере данных Arc

Чтобы получить список групп PostgreSQL Scale Server, развернутых в контроллере данных Arc, выполните следующую команду:

```console
azdata arc postgres server list
```


```output
Name        State     Workers
----------  --------  ---------
postgres01  Ready     2
```

## <a name="get-the-endpoints-to-connect-to-your-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Получение конечных точек для подключения к вашей службе "Дуга Azure" с PostgreSQL масштабируемыми группами серверов

Чтобы просмотреть конечные точки для группы серверов PostgreSQL, выполните следующую команду:

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

Конечную точку экземпляра PostgreSQL можно использовать для подключения к группе PostgreSQL Scale Server из избранного средства:  [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio), [Пгкли](https://www.pgcli.com/) PSQL, pgAdmin и т. д. При этом выполняется подключение к узлу или экземпляру координатора, который выполняет маршрутизацию запроса к соответствующим рабочим узлам или экземплярам, если созданы распределенные таблицы. Дополнительные сведения см. в статье [Основные понятия PostgreSQLного масштабирования Azure](concepts-distributed-postgres-hyperscale.md).

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

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с основными понятиями и руководствами по использованию службы "база данных Azure для PostgreSQL", чтобы распределить данные между несколькими узлами PostgreSQL-Scale и воспользоваться преимуществами лучшей производительность.
    * [Узлы и таблицы](../../postgresql/concepts-hyperscale-nodes.md)
    * [Определение типа приложения](../../postgresql/concepts-hyperscale-app-type.md)
    * [Выбор столбца распределения](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Совместное размещение таблиц](../../postgresql/concepts-hyperscale-colocation.md)
    * [Распространение и изменение таблиц](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Проектирование базы данных с несколькими клиентами](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Разработка панели мониторинга для аналитики в режиме реального времени](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* В приведенных выше документах пропустите разделы **Вход в портал Azure**& **создать базу данных Azure для PostgreSQL-Scale (Цитус)**. Реализуйте оставшиеся шаги в развертывании Azure ARC. Эти разделы относятся к службе "база данных Azure для PostgreSQL" (Цитус), предлагаемой в качестве службы PaaS в облаке Azure, но другие части документов напрямую применимы к вашей геомасштабированию Azure с включенной PostgreSQL.

- [Масштабирование дуги Azure для группы серверов PostgreSQL Scale](scale-out-postgresql-hyperscale-server-group.md)
- [Основные понятия конфигурации хранилища и хранилища Kubernetes](storage-configuration.md)
- [Развертывание утверждений Постоянного тома](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Модель ресурсов Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

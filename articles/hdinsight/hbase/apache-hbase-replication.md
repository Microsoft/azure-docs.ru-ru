---
title: Репликация кластера HBase в виртуальных сетях — Azure HDInsight
description: Сведения о том, как настроить репликацию HBase с одной версии HDInsight на другую для балансировки нагрузки, обеспечения высокого уровня доступности, переноса или обновления без простоя и аварийного восстановления.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 12/06/2019
ms.openlocfilehash: cfcb3a5a601afadb9f3fcd71c24e18a9d7f27b9e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946413"
---
# <a name="set-up-apache-hbase-cluster-replication-in-azure-virtual-networks"></a>Настройка репликации кластера Apache HBase в виртуальных сетях Azure

Настройка репликации [Apache HBase](https://hbase.apache.org/) в пределах одной или двух виртуальных сетей в Azure.

Репликация кластера использует методологию source-push. Кластер HBase может быть исходным, кластером назначения или выполнять обе роли одновременно. Репликация выполняется асинхронно. Целью репликации в конечном итоге является согласованность. При получении источником изменения в семействе столбцов с включенной репликацией такое изменение распространяется на все кластеры назначения. При репликации данных с одного кластера на другой исходный кластер и все кластеры, которые уже потребили данные, отслеживаются для предотвращения циклических репликаций.

В этой статье описано, как настроить репликацию источника и назначения. Другие топологии кластеров см. в [справочном руководстве по Apache HBase](https://hbase.apache.org/book.html#_cluster_replication).

Примеры использования репликации HBase для одной виртуальной сети:

* Балансировка нагрузки. Например, выполнения проверки или заданий MapReduce в целевом кластере и приема данных на исходном кластере.
* Добавление высокого уровня доступности.
* Перенос данных из одного кластера HBase в другой.
* Обновление кластера Azure HDInsight до другой версии.

Примеры использования репликации HBase для двух виртуальных сетей:

* Настройка аварийного восстановления.
* Балансировка нагрузки и секционирование приложения.
* Добавление высокого уровня доступности.

Кластеры можно реплицировать с помощью скриптов [действий сценария](../hdinsight-hadoop-customize-cluster-linux.md), которые можно найти на [GitHub](https://github.com/Azure/hbase-utils/tree/master/replication).

## <a name="prerequisites"></a>Предварительные условия
Прежде чем приступать к этой статье, необходимо иметь подписку Azure. См. страницу о [получении бесплатной пробной версии Azure](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).

## <a name="set-up-the-environments"></a>Настройка сред

Доступны три варианта конфигурации:

- два кластера Apache HBase в одной виртуальной сети Azure;
- два кластера Apache HBase в двух виртуальных сетях в одном регионе;
- два кластера Apache HBase в двух виртуальных сетях в двух регионах (георепликация).

В этой статье описан сценарий георепликации.

Чтобы помочь вам в настройке сред мы создали некоторые [шаблоны Azure Resource Manager](../../azure-resource-manager/management/overview.md). Если вы предпочитаете настраивать среды с помощью других методов, см. следующие статьи:

- [Создание кластеров Apache Hadoop в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
- [Создание кластеров Apache HBase в виртуальной сети Azure](apache-hbase-provision-vnet.md)

### <a name="set-up-two-virtual-networks-in-two-different-regions"></a>Настройка двух виртуальных сетей в двух разных регионах

Для использования шаблона, который создает две виртуальные сети в двух разных регионах и соединяет их с помощью VPN-подключения, нажмите кнопку **Развернуть в Azure**. Определение шаблона хранится в [общедоступном хранилище BLOB-объектов](https://hditutorialdata.blob.core.windows.net/hbaseha/azuredeploy.json).

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhbaseha%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-replication/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

Ниже приведены некоторые жестко заданные значения в шаблоне.

**VNet 1**

| Свойство. | Значение |
|----------|-------|
| Расположение | западная часть США |
| Имя виртуальной сети | &lt;префикс_имени_кластера>-vnet1 |
| Address space prefix | 10.1.0.0/16 |
| Имя подсети | subnet 1 |
| Subnet prefix | 10.1.0.0/24 |
| Subnet (gateway) name | GatewaySubnet (не может быть изменено) |
| Префикс подсети (шлюза). | 10.1.255.0/27 |
| Имя шлюза. | vnet1gw |
| Тип шлюза | Vpn |
| Тип VPN шлюза. | RouteBased |
| Gateway SKU | Basic |
| Gateway IP | vnet1gwip |

**VNet 2**

| Свойство. | Значение |
|----------|-------|
| Расположение | Восточная часть США |
| Имя виртуальной сети | &lt;префикс_имени_кластера>-vnet2 |
| Address space prefix | 10.2.0.0/16 |
| Имя подсети | subnet 1 |
| Subnet prefix | 10.2.0.0/24 |
| Subnet (gateway) name | GatewaySubnet (не может быть изменено) |
| Префикс подсети (шлюза). | 10.2.255.0/27 |
| Имя шлюза. | vnet2gw |
| Тип шлюза | Vpn |
| Тип VPN шлюза. | RouteBased |
| Gateway SKU | Basic |
| Gateway IP | vnet1gwip |

## <a name="setup-dns"></a>Настройка службы доменных имен (DNS)

В последнем разделе шаблон создает виртуальную машину Ubuntu в каждой виртуальной сети.  В этом разделе вы установите Bind на две виртуальные машины DNS, а затем настроите на них переадресацию DNS.

Чтобы установить Bind, найдите общедоступные IP-адреса двух виртуальных машин DNS.

1. Откройте [портал Azure](https://portal.azure.com).
2. Откройте виртуальную машину DNS, выбрав **Группы ресурсов > [имя группы ресурсов] > [vnet1DNS]**.  Используйте имя группы ресурсов, созданной в последней процедуре. По умолчанию виртуальные машины DNS называются *vnet1DNS* и *vnet2NDS*.
3. Выберите **Панель свойств**. Откроется страница свойств виртуальной сети.
4. Запишите **общедоступный IP-адрес**, а также проверьте **частный IP-адрес**.  Для vnet1DNS должен быть указан IP-адрес **10.1.0.4**, а для vnet2DNS — **10.2.0.4**.  
5. Настройте использование в обеих виртуальных сетях DNS-серверов по умолчанию (предоставленных Azure). Это позволит разрешить входящий и исходящий доступ к пакетам, используемым при установке Bind на следующих шагах.

Чтобы установить Bind, выполните следующую процедуру.

1. Используйте SSH для подключения к __общедоступному IP-адресу__ виртуальной машины. В следующем примере устанавливается подключение к виртуальной машине по адресу 40.68.254.142:

    ```bash
    ssh sshuser@40.68.254.142
    ```

    Замените `sshuser` учетной записью пользователя SSH, указанной при создании виртуальной машины DNS.

    > [!NOTE]  
    > Есть несколько способов получить служебную программу `ssh`. В Linux, Unix и macOS она предоставляется как часть операционной системы. Если вы используете Windows, рассмотрите один из следующих вариантов:
    >
    > * [Azure Cloud Shell](../../cloud-shell/quickstart.md);
    > * [Bash на платформе Ubuntu в Windows 10](/windows/wsl/about);
    > * [Git (https://git-scm.com/)](https://git-scm.com/);
    > * [OpenSSH (https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH)](https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH).

2. Чтобы установить Bind, используйте следующие команды из сеанса SSH:

    ```bash
    sudo apt-get update -y
    sudo apt-get install bind9 -y
    ```

3. Настройте привязку, чтобы перенаправить запросы на разрешение имен на локальный DNS-сервер. Чтобы сделать это, в качестве содержимого файла `/etc/bind/named.conf.options` добавьте следующий текст:

    ```
    acl goodclients {
        10.1.0.0/16; # Replace with the IP address range of the virtual network 1
        10.2.0.0/16; # Replace with the IP address range of the virtual network 2
        localhost;
        localhost;
    };
    
    options {
        directory "/var/cache/bind";
        recursion yes;
        allow-query { goodclients; };

        forwarders {
            168.63.129.16; #This is the Azure DNS server
        };

        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
    };
    ```
    
    > [!IMPORTANT]  
    > Замените значения в разделе `goodclients` следующим диапазоном IP-адресов двух виртуальных сетей. Этот раздел определяет адреса, по которым этот DNS-сервер принимает запросы.

    Чтобы изменить этот файл, используйте следующую команду:

    ```bash
    sudo nano /etc/bind/named.conf.options
    ```

    Чтобы сохранить файл, нажмите клавиши __CTRL+X__, затем — __Y__ и __ВВОД__.

4. В сеансе SSH используйте следующую команду:

    ```bash
    hostname -f
    ```

    Эта команда возвращает значение следующего вида:

    ```output
    vnet1DNS.icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net
    ```

    Текст `icb0d0thtw0ebifqt0g1jycdxd.ex.internal.cloudapp.net` — это __DNS-суффикс__ для виртуальной сети. Сохраните это значение, так как оно будет использовано позже.

    Кроме того, необходимо найти на DNS-сервере DNS-суффикс. Он понадобится нам на следующем шаге.

5. Чтобы настроить Bind для разрешения DNS-имен ресурсов в виртуальной сети, в качестве содержимого файла `/etc/bind/named.conf.local` добавьте следующий текст:

    ```
    // Replace the following with the DNS suffix for your virtual network
    zone "v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net" {
            type forward;
            forwarders {10.2.0.4;}; # The Azure recursive resolver
    };
    ```

    > [!IMPORTANT]  
    > Замените значение `v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` DNS-суффиксом другой виртуальной сети. IP-адрес сервера пересылки представляет собой частный IP-адрес DNS-сервера в другой виртуальной сети.

    Чтобы изменить этот файл, используйте следующую команду:

    ```bash
    sudo nano /etc/bind/named.conf.local
    ```

    Чтобы сохранить файл, нажмите клавиши __CTRL+X__, затем — __Y__ и __ВВОД__.

6. Чтобы запустить Bind, используйте следующую команду:

    ```bash
    sudo service bind9 restart
    ```

7. Чтобы убедиться, что привязка может разрешать имена ресурсов в другой виртуальной сети, используйте следующие команды:

    ```bash
    sudo apt install dnsutils
    nslookup vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    ```

    > [!IMPORTANT]  
    > Замените значение `vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net` полным доменным именем (FQDN) виртуальной машины DNS в другой сети.
    >
    > Замените `10.2.0.4`__внутренним IP-адресом__ пользовательского DNS-сервера в другой виртуальной сети.

    Ответ будет выглядеть следующим образом:

    ```output
    Server:         10.2.0.4
    Address:        10.2.0.4#53
    
    Non-authoritative answer:
    Name:   vnet2dns.v5ant3az2hbe1edzthhvwwkcse.bx.internal.cloudapp.net
    Address: 10.2.0.4
    ```

    Пока вы не сможете искать IP-адрес из другой сети без указанного IP-адреса DNS-сервера.

### <a name="configure-the-virtual-network-to-use-the-custom-dns-server"></a>Настройка виртуальной сети для использования с пользовательским DNS-сервером

Чтобы настроить виртуальную сеть для использования с пользовательским DNS-сервером вместо рекурсивного сопоставителя Azure, сделайте следующее:

1. На [портале Azure](https://portal.azure.com) выберите виртуальную сеть, а затем — __DNS-серверы__.

2. Выберите __Custom__ (Пользовательский) и введите __внутренний IP-адрес__ пользовательского DNS-сервера. Наконец, щелкните __Сохранить__.

6. Откройте виртуальную машину DNS-сервера в виртуальной сети 1 и щелкните **Перезагрузить**.  Чтобы конфигурация DNS вступила в силу, необходимо перезагрузить все виртуальные машины в виртуальной сети.
7. Повторите шаги настройки пользовательского DNS-сервера для виртуальной сети 2.

Чтобы проверить конфигурацию DNS, подключитесь к двум виртуальным машинам DNS с помощью SSH и проверьте связь с DNS-сервером другой виртуальной сети с помощью имени узла. Если это не сработает, используйте следующую команду для проверки состояния DNS:

```bash
sudo service bind9 status
```

## <a name="create-apache-hbase-clusters"></a>Создание кластеров Apache HBase

В каждой виртуальной сети создайте кластер [Apache HBase](https://hbase.apache.org/) со следующей конфигурацией:

- **Имя группы ресурсов.** Используйте те же имена групп ресурсов, как при создании виртуальных сетей.
- **Тип кластера**: HBase
- **Версия.** HBase 1.1.2 (HDI 3.6).
- **Расположение.** Используйте то же расположение, что и у виртуальной сети.  По умолчанию для виртуальной сети 1 указано расположение *западная часть США*, а для виртуальной сети 2 — *восточная часть США*.
- **Хранилище.** Создайте учетную запись хранения для кластера.
- **Виртуальная сеть** (из дополнительных параметров на портале). Выберите виртуальную сеть 1, созданную в предыдущей процедуре.
- **Подсеть.** Имя по умолчанию, используемое в шаблоне, — **subnet1**.

Чтобы убедиться, что среда правильно настроена, проверьте связь FQDN головного узла между двумя кластерами.

## <a name="load-test-data"></a>Загрузка тестовых данных

При репликации кластера необходимо указать реплицируемые таблицы. В этом разделе вы загрузите данные в исходный кластер. В следующем разделе вы включите репликацию между двумя кластерами.

Чтобы создать таблицу [Contacts](apache-hbase-tutorial-get-started-linux.md) и вставить в нее некоторые данные, следуйте инструкциям по **началу работы с примером Apache HBase в HDInsight**.

> [!NOTE]
> Если необходимо реплицировать таблицы из пользовательского пространства имен, необходимо также убедиться, что в целевом кластере определены соответствующие пользовательские пространства имен.
>

## <a name="enable-replication"></a>Включение репликации

Ниже описано, как вызвать скрипт действия сценария на портале Azure. Дополнительные сведения о том, как выполнить действие сценария с помощью Azure PowerShell и классической версии Azure CLI, см. в статье [Настройка кластеров HDInsight под управлением Linux с помощью действий сценариев](../hdinsight-hadoop-customize-cluster-linux.md).

**Включение репликации HBase на портале Azure**

1. Войдите на [портал Azure](https://portal.azure.com).
2. Откройте исходный кластер HBase.
3. В меню кластера выберите **Действия скрипта**.
4. В верхней части страницы выберите **Отправить новое**.
5. Выберите или введите следующие сведения.

   1. **Имя:** введите **Включение репликации**.
   2. **URI bash-скрипта.** Введите **https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_enable_replication.sh**.
   3. **Головной узел**. Выберите этот тип узла. Отмените выбор других типов узлов.
   4. **Параметры**: Параметры в следующем примере позволяют включить репликацию для всех существующих таблиц, а затем копировать все данные из исходного кластера в целевой.

    `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -copydata`
    
      > [!NOTE]
      > Используйте имя узла вместо полного доменного имени (FQDN) для DNS-имени как исходного, так и целевого кластера.
      >
      > В этом пошаговом руководстве предполагается, что HN1 является активным головного узла. Проверьте кластер, чтобы определить активный головной узел.

6. Нажмите кнопку **создания**. Выполнение скрипта может занять некоторое время, особенно при использовании аргумента **-copydata**.

Ниже приведены обязательные аргументы.

|Имя|Описание|
|----|-----------|
|-s, --src-cluster | Указывает DNS-имя исходного кластера HBase. например -s hbsrccluster, --src-cluster=hbsrccluster. |
|-d, --dst-cluster | Указывает DNS-имя кластера назначения (реплики) HBase. например -s dsthbcluster, --src-cluster=dsthbcluster. |
|-sp, --src-ambari-password | Указывает пароль администратора для Ambari в исходном кластере HBase. |
|-dp, --dst-ambari-password | Указывает пароль администратора для Ambari в целевом кластере HBase.|

Необязательные аргументы для этой команды.

|Имя|Описание|
|----|-----------|
|-su, --src-ambari-user | Указывает имя пользователя-администратора для Ambari в исходном кластере HBase. Значение по умолчанию — **admin**. |
|-du, --dst-ambari-user | Указывает имя пользователя-администратора для Ambari в целевом кластере HBase. Значение по умолчанию — **admin**. |
|-t, --table-list | Указывает таблицы для репликации. например --table-list="table1;table2;table3". Если не указать таблицы, будут реплицированы все существующие таблицы HBase.|
|-m, --machine | Указывает головной узел, на котором будет выполняться действие сценария. Значение должно быть выбрано в зависимости от активного головного узла. Используйте этот параметр при запуске скрипта $0 как действия сценария из портала HDInsight или Azure PowerShell.|
|-cp, -copydata | Включает перенос существующих данных для таблиц, где включена репликация. |
|-rpm, -replicate-phoenix-meta | Включает репликацию для системных таблиц Phoenix. <br><br>*Используйте этот параметр с осторожностью.* Рекомендуется повторно создать таблицы Phoenix в реплицированных кластерах перед использованием этого скрипта. |
|-h, --help | Отображает сведения об использовании. |

Раздел `print_usage()`[скрипта](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_enable_replication.sh) содержит подробное описание параметров.

После успешного развертывания действия сценария можно использовать протокол SSH, чтобы подключиться к кластеру назначения HBase, а после убедиться, что данные реплицированы.

### <a name="replication-scenarios"></a>Сценарии репликации

Ниже перечислены некоторые общие способы применения и используемые параметры.

- **Включение репликации для всех таблиц между двумя кластерами.** В этом сценарии не требуется копирование или перенос существующих данных в таблицах и не используются таблицы Phoenix. Используйте следующие параметры:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password>`

- **Включение репликации для отдельных таблиц.** Чтобы включить репликацию для table1, table2 и table3, используйте следующие параметры:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3"`

- **Включите репликацию для конкретных таблиц и скопируйте существующие данные**. Чтобы включить репликацию для table1, table2 и table3, используйте следующие параметры:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -copydata`

- **Включение репликации для всех таблиц с репликацией метаданных Phoenix из источника в место назначения**. Репликация метаданных Phoenix работает не идеально. Ее следует использовать с осторожностью. Используйте следующие параметры:

  `-m hn1 -s <source hbase cluster name> -d <destination hbase cluster name> -sp <source cluster Ambari password> -dp <destination cluster Ambari password> -t "table1;table2;table3" -replicate-phoenix-meta`

## <a name="copy-and-migrate-data"></a>Копирование и перенос данных

Существует два отдельных скрипта действия сценария для копирования или переноса данных после включения репликации:

- [Скрипт для маленьких таблиц](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_copy_table.sh) (таблицы размером в несколько гигабайт, ожидаемое время их копирования — менее одного часа).

- [Скрипт для больших таблиц](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/nohup_hdi_copy_table.sh) (таблицы, копирование которых занимает больше одного часа).

Выполните ту же процедуру, описанную в разделе [Включение репликации](#enable-replication) для вызова действия сценария. Используйте следующие параметры:

`-m hn1 -t <table1:start_timestamp:end_timestamp;table2:start_timestamp:end_timestamp;...> -p <replication_peer> [-everythingTillNow]`

Раздел `print_usage()`[скрипта](https://github.com/Azure/hbase-utils/blob/master/replication/hdi_copy_table.sh) содержит подробное описание параметров.

### <a name="scenarios"></a>Сценарии

- **Копирование определенных таблиц (test1, test2 и test3) для всех строк, измененных до настоящего момента (текущая отметка времени):**

  `-m hn1 -t "test1::;test2::;test3::" -p "zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure" -everythingTillNow`

  Или сделайте так:

  `-m hn1 -t "test1::;test2::;test3::" --replication-peer="zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure" -everythingTillNow`

- **Копирование определенных таблиц с указанным интервалом времени**:

  `-m hn1 -t "table1:0:452256397;table2:14141444:452256397" -p "zk5-hbrpl2;zk1-hbrpl2;zk5-hbrpl2:2181:/hbase-unsecure"`

## <a name="disable-replication"></a>Отключение репликации

Чтобы отключить репликацию, используйте другой пример действия сценария из [GitHub](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh). Выполните ту же процедуру, описанную в разделе [Включение репликации](#enable-replication) для вызова действия сценария. Используйте следующие параметры:

`-m hn1 -s <source hbase cluster name> -sp <source cluster Ambari password> <-all|-t "table1;table2;...">`

Раздел `print_usage()`[скрипта](https://raw.githubusercontent.com/Azure/hbase-utils/master/replication/hdi_disable_replication.sh) содержит подробное описание параметров.

### <a name="scenarios"></a>Сценарии

- **Отключение репликации для всех таблиц:**

  `-m hn1 -s <source hbase cluster name> -sp Mypassword\!789 -all`

  или диспетчер конфигурации служб

  `--src-cluster=<source hbase cluster name> --dst-cluster=<destination hbase cluster name> --src-ambari-user=<source cluster Ambari user name> --src-ambari-password=<source cluster Ambari password>`

- **Отключение репликации для определенных таблиц (table1, table2 и table3):**

  `-m hn1 -s <source hbase cluster name> -sp <source cluster Ambari password> -t "table1;table2;table3"`

> [!NOTE]
> Если планируется удалить целевой кластер, убедитесь, что он удален из однорангового списка исходного кластера. Это можно сделать, выполнив команду remove_peer "1" в оболочке HBase в исходном кластере. Сбой в исходном кластере может работать неправильно.
>

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как настроить репликацию Apache HBase в виртуальной сети или между двумя виртуальными сетями. Дополнительные сведения об HDInsight и Apache HBase см.в следующих статьях:

* [Начало работы с примером Apache HBase в HDInsight](./apache-hbase-tutorial-get-started-linux.md)
* [Общие сведения об HDInsight Apache HBase](./apache-hbase-overview.md)
* [Создание кластеров Apache HBase в виртуальной сети Azure](./apache-hbase-provision-vnet.md)
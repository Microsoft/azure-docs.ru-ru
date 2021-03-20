---
title: Подключение или установка Apache Beeline в Azure HDInsight
description: Узнайте, как подключиться к клиенту Apache Beeline для выполнения запросов Hive с Hadoop в HDInsight. Beeline — это служебная программа для работы с HiveServer2 через JDBC.
ms.service: hdinsight
ms.topic: how-to
ms.custom: contperf-fy21q1
ms.date: 05/27/2020
ms.openlocfilehash: ab5dedf6718dad4f16fde59d905e2e59be5c495f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944464"
---
# <a name="connect-to-apache-beeline-on-hdinsight-or-install-it-locally"></a>Подключитесь к Apache Beeline в HDInsight или установите его локально

[Apache Beeline](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–NewCommandLineShell) — это клиент Hive, который входит в головные узлы кластера HDInsight. В этой статье описывается, как подключиться к клиенту Beeline, установленному в кластере HDInsight, по разным типам соединений. В нем также обсуждается [Установка клиента Beeline локально](#install-beeline-client). 

## <a name="types-of-connections"></a>Типы подключений

### <a name="from-an-ssh-session"></a>Из сеанса SSH

При подключении из сеанса SSH к кластеру головного узла можно подключиться к `headnodehost` адресу через порт `10001` :

```bash
beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http'
```

### <a name="over-an-azure-virtual-network"></a>Через виртуальную сеть Azure

При подключении клиента к HDInsight через виртуальную сеть Azure необходимо указать полное доменное имя головного узла кластера. Так как подключение устанавливается напрямую к узлам кластера, для подключения используется порт `10001`:

```bash
beeline -u 'jdbc:hive2://<headnode-FQDN>:10001/;transportMode=http'
```

Замените на `<headnode-FQDN>` полное доменное имя кластера головного узла. Чтобы найти полное доменное имя головного узла, используйте сведения из [этого раздела](../hdinsight-hadoop-manage-ambari-rest-api.md#get-the-fqdn-of-cluster-nodes).

### <a name="to-hdinsight-enterprise-security-package-esp-cluster-using-kerberos"></a>В кластер HDInsight Корпоративный пакет безопасности (ESP) по протоколу Kerberos

При подключении клиента к кластеру Корпоративный пакет безопасности (ESP), присоединенному к Azure Active Directory (AAD) — DS на компьютере в той же области кластера, необходимо также указать имя домена `<AAD-Domain>` и имя учетной записи пользователя домена с разрешениями на доступ к кластеру `<username>` :

```bash
kinit <username>
beeline -u 'jdbc:hive2://<headnode-FQDN>:10001/default;principal=hive/_HOST@<AAD-Domain>;auth-kerberos;transportMode=http' -n <username>
```

Замените `<username>` на имя учетной записи домена с разрешениями на доступ к кластеру. Замените `<AAD-DOMAIN>` именем домена Azure Active Directory (AAD), к которому присоединен кластер. Используйте строку в верхнем регистре для значения `<AAD-DOMAIN>`. В противном случае учетные данные не будут найдены. При необходимости проверьте `/etc/krb5.conf` на наличие имен областей определения приложения.

Чтобы найти URL-адрес JDBC из Ambari, сделайте следующее:

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/#/main/services/HIVE/summary`, где `CLUSTERNAME` — это имя вашего кластера. Убедитесь, что HiveServer2 работает.

1. Используйте буфер обмена для копирования URL-адреса JDBC HiveServer2.

### <a name="over-public-or-private-endpoints"></a>Через общедоступные или частные конечные точки

При подключении к кластеру с помощью общедоступных или частных конечных точек необходимо указать имя учетной записи для входа в кластер (по умолчанию `admin` ) и пароль. Например, при использовании Beeline из системы клиента для подключения к адресу `clustername.azurehdinsight.net`. Это подключение устанавливается через порт `443` и шифруется с помощью TLS/SSL.

Замените `clustername` на имя вашего кластера HDInsight. Замените `admin` учетной записью для входа в кластер. Для кластеров ESP используйте полное имя участника-пользователя (например, user@domain.com ). Замените `password` паролем этой учетной записи.

```bash
beeline -u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p 'password'
```

или для частной конечной точки:

```bash
beeline -u 'jdbc:hive2://clustername-int.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/hive2' -n admin -p 'password'
```

Частные конечные точки указывают на базовую подсистему балансировки нагрузки, доступ к которой можно получить только с виртуальных сетей пиринга в том же регионе. Дополнительные сведения см. [в статье ограничения для пиринга глобальной виртуальной сети и подсистем балансировки нагрузки](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) . Вы можете использовать `curl` команду с `-v` параметром для устранения проблем с подключением к общедоступным или частным конечным точкам перед использованием Beeline.

### <a name="use-beeline-with-apache-spark"></a>Использование Beeline с Apache Spark

Apache Spark предоставляет собственную реализацию HiveServer2, которая иногда называется сервером Thrift Spark. Эта служба использует Spark SQL для разрешения запросов, а не Hive. И могут повысить производительность в зависимости от запроса.

#### <a name="through-public-or-private-endpoints"></a>Через общедоступные или частные конечные точки

Используемая строка подключения немного отличается. Вместо содержащего `httpPath=/hive2` его использования `httpPath/sparkhive2` . Замените `clustername` на имя вашего кластера HDInsight. Замените `admin` учетной записью для входа в кластер. Для кластеров ESP используйте полное имя участника-пользователя (например, user@domain.com ). Замените `password` паролем этой учетной записи.

```bash
beeline -u 'jdbc:hive2://clustername.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/sparkhive2' -n admin -p 'password'
```

или для частной конечной точки:

```bash
beeline -u 'jdbc:hive2://clustername-int.azurehdinsight.net:443/;ssl=true;transportMode=http;httpPath=/sparkhive2' -n admin -p 'password'
```

Частные конечные точки указывают на базовую подсистему балансировки нагрузки, доступ к которой можно получить только с виртуальных сетей пиринга в том же регионе. Дополнительные сведения см. [в статье ограничения для пиринга глобальной виртуальной сети и подсистем балансировки нагрузки](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) . Вы можете использовать `curl` команду с `-v` параметром для устранения проблем с подключением к общедоступным или частным конечным точкам перед использованием Beeline.

#### <a name="from-cluster-head-or-inside-azure-virtual-network-with-apache-spark"></a>Из головного кластера или внутри виртуальной сети Azure с Apache Spark

При подключении непосредственно из головного узла кластера или из ресурса в той же виртуальной сети Azure, что и кластер HDInsight, вместо порта `10001` для сервера Spark Thrift нужно использовать порт `10002`. В следующем примере показано, как подключиться непосредственно к головному узлу:

```bash
/usr/hdp/current/spark2-client/bin/beeline -u 'jdbc:hive2://headnodehost:10002/;transportMode=http'
```

## <a name="install-beeline-client"></a>Установка клиента Beeline

Хотя Beeline входит в головные узлы, может потребоваться установить его локально.  Шаги установки для локального компьютера основаны на [подсистеме Windows для Linux](/windows/wsl/install-win10).

1. Обновление списков пакетов. Введите следующую команду в оболочке bash:

    ```bash
    sudo apt-get update
    ```

1. Установите Java, если он не установлен. Можно проверить с помощью `which java` команды.

    1. Если пакет Java не установлен, введите следующую команду:

        ```bash
        sudo apt install openjdk-11-jre-headless
        ```

    1. Откройте файл bashrc (часто находится в ~ файл/.bashrc): `nano ~/.bashrc` .

    1. Измените файл bashrc. В конце файла добавьте следующую строку:

        ```bash
        export JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64
        ```

        Затем нажмите клавиши **CTRL + X**, затем **Y**, а затем введите.

1. Скачайте архивы Hadoop и Beeline, введите следующие команды:

    ```bash
    wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.3/hadoop-2.7.3.tar.gz
    wget https://archive.apache.org/dist/hive/hive-1.2.1/apache-hive-1.2.1-bin.tar.gz
    ```

1. Распакуйте архивы и введите следующие команды:

    ```bash
    tar -xvzf hadoop-2.7.3.tar.gz
    tar -xvzf apache-hive-1.2.1-bin.tar.gz
    ```

1. Дополнительные уточнение файла bashrc. Необходимо будет задать путь, по которому будут распакованы архивы. Если используется [подсистема Windows для Linux](/windows/wsl/install-win10)и вы следовали инструкциям в точности, путь будет иметь вид `/mnt/c/Users/user/` , где `user` — это имя пользователя.

    1. Откройте файл: `nano ~/.bashrc`

    1. Измените приведенные ниже команды, указав соответствующий путь, а затем введите их в конце файла bashrc:

        ```bash
        export HADOOP_HOME=/path_where_the_archives_were_unpacked/hadoop-2.7.3
        export HIVE_HOME=/path_where_the_archives_were_unpacked/apache-hive-1.2.1-bin
        PATH=$PATH:$HIVE_HOME/bin
        ```

    1. Затем нажмите клавиши **CTRL + X**, затем **Y**, а затем введите.

1. Закройте и снова откройте сеанс bash.

1. Проверьте подключение. Используйте формат подключения [через общедоступные или частные конечные точки](#over-public-or-private-endpoints)выше.

## <a name="next-steps"></a>Дальнейшие действия

* Примеры использования клиента Beeline с Apache Hive см. в статье [использование Apache Beeline с Apache Hive](apache-hadoop-use-hive-beeline.md)
* Дополнительные общие сведения о Hive в HDInsight см. в статье [использование Apache Hive с Apache Hadoop в hdinsight](hdinsight-use-hive.md) .
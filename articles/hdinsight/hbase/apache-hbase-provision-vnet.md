---
title: Создание кластеров HBase в виртуальной сети в Azure
description: Приступите к работе с HBase в Azure HDInsight. Узнайте, как создать кластеры HDInsight HBase в виртуальной сети Azure.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/23/2019
ms.openlocfilehash: 9f179981aa39402681b4830d58a29f5b1259c7e2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946118"
---
# <a name="create-apache-hbase-clusters-on-hdinsight-in-azure-virtual-network"></a>Создание кластеров Apache HBase в HDInsight в виртуальной сети Azure

Узнайте, как создавать кластеры Azure HDInsight Apache HBase в [виртуальной сети Azure](https://azure.microsoft.com/services/virtual-network/).

Благодаря интеграции виртуальной сети кластеры Apache HBase можно развернуть в той же виртуальной сети, что и приложения. Это позволяет приложениям взаимодействовать с HBase напрямую. К преимуществам относятся:

* прямое подключение веб-приложения к узлам кластера HBase, который обеспечивает обмен данными с помощью интерфейсов API удаленного вызова процедур (RPC) Java для HBase;
* повышение производительности без необходимости организации пропуска трафика через множество шлюзов и подсистемы балансировки нагрузки;
* возможность обработки конфиденциальной информации более безопасным способом, без необходимости организации общедоступной конечной точки.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="create-apache-hbase-cluster-into-virtual-network"></a>Создание кластера Apache HBase в виртуальной сети

В этом разделе описывается, как создать кластер Apache HBase под управлением Linux с зависимой учетной записью службы хранилища Azure в виртуальной сети Azure c помощью [шаблона Azure Resource Manager](../../azure-resource-manager/templates/deploy-powershell.md). Сведения о других способах создания кластеров и их параметрах см. в статье [Создание кластеров Hadoop под управлением Linux в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Дополнительные сведения о создании в HDInsight кластеров Apache Hadoop с помощью шаблонов см. в [этой статье](../hdinsight-hadoop-create-linux-clusters-arm-templates.md).

> [!NOTE]  
> Некоторые свойства жестко заданы в шаблоне. Пример:
>
> * **Расположение**: восточная часть США 2.
> * **Версия кластера**: 3.6
> * **Число рабочих узлов кластера**: 2.
> * **Учетная запись хранения по умолчанию**: уникальная строка
> * **Имя виртуальной сети**: имя_кластера-vnet
> * **Адресное пространство виртуальной сети**: 10.0.0.0/16
> * **Имя подсети**: subnet1
> * **Диапазон адресов подсети**: 10.0.0.0/24.
>
> `CLUSTERNAME` заменяется именем кластера, которое вы задаете при использовании шаблона.

1. Выберите следующее изображение, чтобы открыть шаблон на портале Azure. Шаблон находится на странице [Шаблоны быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-linux-vnet/).

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-hbase-linux-vnet%2Fazuredeploy.json" target="_blank"><img src="./media/apache-hbase-provision-vnet/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

1. В диалоговом окне **Настраиваемое развертывание** выберите **изменить шаблон**.

1. В строке 165 измените значение `Standard_A3` на `Standard_A4_V2` . Затем нажмите кнопку **Save** (Сохранить).

1. Заполните оставшийся шаблон следующими сведениями:

    |Свойство. |Значение |
    |---|---|
    |Подписка|Выберите подписку Azure, которая использовалась для создания кластера HDInsight, зависимой учетной записи хранения и виртуальной сети Azure.|
    Группа ресурсов|Щелкните **Создать** и укажите имя новой группы ресурсов.|
    |Расположение|Выберите расположение группы ресурсов.|
    |Имя кластера,|Введите имя создаваемого кластера Hadoop.|
    |Имя пользователя и пароль для входа в кластер|Имя пользователя по умолчанию — **Admin**. Укажите пароль.|
    |Имя пользователя и пароль SSH|Имя пользователя по умолчанию — **sshuser**.  Введите пароль.|

    Установите флажок **я принимаю условия и условия**, указанные выше.

1. Щелкните **Приобрести**. Процесс создания кластера занимает около 20 минут. После создания кластера можно выбрать его на портале, чтобы открыть его.

После завершения статьи может потребоваться удалить кластер. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Поскольку стоимость кластера во много раз превышает стоимость хранилища, экономически целесообразно удалять неиспользуемые кластеры. Инструкции по удалению кластера см. в статье [Управление кластерами Apache Hadoop для Windows в HDInsight с помощью портала Azure](../hdinsight-administer-use-portal-linux.md#delete-clusters).

Чтобы начать работу с новым кластером HBase, выполните процедуры, которые представлены в статье [Начало работы с примером Apache HBase в HDInsight](./apache-hbase-tutorial-get-started-linux.md).

## <a name="connect-to-the-apache-hbase-cluster-using-apache-hbase-java-rpc-apis"></a>Подключение к кластеру Apache HBase с помощью API-интерфейсов удаленного вызова процедур Java Apache HBase

### <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину IaaS в той же виртуальной сети Azure и той же подсети. Инструкции по созданию виртуальной машины IaaS см. в статье [Создание первой виртуальной машины Windows на портале Azure](../../virtual-machines/windows/quick-create-portal.md). При выполнении действий, описанных в этом документе, необходимо использовать следующие значения для конфигурации сети.

* **Виртуальная сеть**: имя_кластера — vnet
* **Подсеть**: subnet1

> [!IMPORTANT]  
> Замените `CLUSTERNAME` именем, которое вы использовали при создании кластера HDInsight на предыдущих шагах.

Благодаря этим значениям виртуальная машина будет размещена в той же виртуальной сети и подсети, что и кластер HDInsight. Эта конфигурация позволит им напрямую взаимодействовать друг с другом. Существует возможность создания кластера HDInsight с пустым граничным узлом. Граничный узел можно использовать для управления кластером.  Подробные сведения см. в статье [Использование пустых граничных узлов в HDInsight](../hdinsight-apps-use-edge-node.md).

### <a name="obtain-fully-qualified-domain-name"></a>Получение полного доменного имени

При использовании Java-приложения для удаленного подключения к HBase необходимо использовать полное доменное имя (FQDN). Чтобы определить это, вам необходимо получить DNS-суффикс кластера HBase. Для этого используйте один из следующих методов.

* С помощью веб-браузера вызовите [Apache Ambari](https://ambari.apache.org/).

    Перейдите по адресу `https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/hosts?minimal_response=true`. Он возвращает JSON-файл с суффиксами DNS.

* Используйте веб-сайт Ambari.

    1. Перейдите по адресу `https://CLUSTERNAME.azurehdinsight.net`.
    2. В верхнем меню выберите **узлы** .

* Используйте Curl, чтобы выполнить вызовы REST:

    ```bash
    curl -u <username>:<password> -k https://CLUSTERNAME.azurehdinsight.net/ambari/api/v1/clusters/CLUSTERNAME.azurehdinsight.net/services/hbase/components/hbrest
    ```

В возвращенных данных JSON найдите запись «host_name». Она содержит полные доменные имена узлов в кластере. Пример:

```
"host_name" : "hn0-hbaseg.hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net"
```

Часть доменного имени, начинающаяся с имени кластера, является DNS-суффиксом. Например, `hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net`.

<!--
3.    Change the primary DNS suffix configuration of the virtual machine. This enables the virtual machine to automatically resolve the host name of the HBase cluster without explicit specification of the suffix. For example, the *workernode0* host name will be correctly resolved to workernode0 of the HBase cluster.

    To make the configuration change:

    1. RDP into the virtual machine.
    2. Open **Local Group Policy Editor**. The executable is gpedit.msc.
    3. Expand **Computer Configuration**, expand **Administrative Templates**, expand **Network**, and then click **DNS Client**.
    - Set **Primary DNS Suffix** to the value obtained in step 2:

        ![hdinsight.hbase.primary.dns.suffix](./media/apache-hbase-provision-vnet/hdi-primary-dns-suffix.png)
    4. Click **OK**.
    5. Reboot the virtual machine.
-->

### <a name="verify-communication-inside-virtual-network"></a>Проверка обмена данными в виртуальной сети

Чтобы проверить обмен данными между виртуальной машиной и кластером HBase, используйте следующую команду `ping headnode0.<dns suffix>` на виртуальной машине. Например, `ping hn0-hbaseg.hjfrnszlumfuhfk4pi1guh410c.bx.internal.cloudapp.net`.

Чтобы использовать эту информацию в Java-приложении, создайте приложение, выполнив шаги, представленные в статье [Создание приложений Java для Apache HBase](./apache-hbase-build-java-maven-linux.md). Чтобы приложение подключилось к удаленному серверу HBase, измените файл **hbase-site.xml** в этом примере, чтобы использовать полное доменное имя для ZooKeeper. Пример:

```xml
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>zookeeper0.<dns suffix>,zookeeper1.<dns suffix>,zookeeper2.<dns suffix></value>
</property>
```

> [!NOTE]  
> Чтобы получить дополнительную информацию о разрешении имен в виртуальных сетях Azure, а также об использовании своего​ собственного DNS-сервера, ознакомьтесь со статьей [Разрешение имен для ВМ и экземпляров ролей](../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как создать кластер Apache HBase. Дополнительные сведения см. на следующих ресурсах:

* [Приступая к работе с HDInsight](../hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Использование пустых граничных узлов в HDInsight](../hdinsight-apps-use-edge-node.md)
* [Настройка репликации кластера HBase в виртуальных сетях Azure](apache-hbase-replication.md)
* [Создание кластеров Apache Hadoop в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)
* [Начало работы с примером Apache HBase в HDInsight](./apache-hbase-tutorial-get-started-linux.md)
* [Обзор виртуальной сети](../../virtual-network/virtual-networks-overview.md)

---
title: Краткое руководство. Развертывание управляемого кластера Apache Spark с Azure Databricks
description: В этом кратком руководстве объясняется, как развернуть управляемый кластер Apache Spark с Azure Databricks с помощью портала Azure.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 03/02/2021
ms.openlocfilehash: fd0d5c5b73ae1fb1eae7f38a22913018555ebe11
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101747870"
---
# <a name="quickstart-deploy-a-managed-apache-spark-cluster-preview-with-azure-databricks"></a>Краткое руководство. Развертывание управляемого кластера Apache Spark (предварительная версия) с Azure Databricks

Управляемый экземпляр Azure для Apache Cassandra позволяет автоматизировать операции развертывания и масштабирования для управляемых центров обработки данных Apache Cassandra с открытым кодом, ускоряя работу в гибридных сценариях и сокращая необходимый объем текущего обслуживания.

> [!IMPORTANT]
> Служба "Управляемый экземпляр Azure для Apache Cassandra" сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этом кратком руководстве показано, как с помощью портала Azure создать полностью управляемый кластер Apache Spark в виртуальной сети Azure вашего Управляемого экземпляра Azure для кластера Apache Cassandra. Сначала вы создадите кластер Spark в Azure Databricks. А позже сможете создать для кластера записные книжки или подключить к нему существующие записные книжки, выполнять чтение данных из разных источников и анализировать аналитические сведения.

См. подробные инструкции по [развертыванию Azure Databricks в виртуальной сети Azure (внедрение в виртуальную сеть)](/azure/databricks/administration-guide/cloud-configurations/azure/vnet-inject).

## <a name="create-an-azure-databricks-cluster"></a>Создание кластера Azure Databricks.

Выполните следующие действия, чтобы создать кластер Azure Databricks в виртуальной сети с Управляемым экземпляром Azure для Apache Cassandra:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. На панели навигации слева найдите пункт **Группы ресурсов** и перейдите к группе ресурсов, содержащей виртуальную сеть, в которой развернут управляемый экземпляр.

1. Откройте ресурс **виртуальной сети** и запишите **диапазон адресов**:

    :::image type="content" source="./media/deploy-cluster-databricks/virtual-network-address-space.png" alt-text="Экран &quot;Получение диапазона адресов для виртуальной сети&quot;." border="true":::

1. В группе ресурсов выберите **Добавить** и введите в поле поиска **Azure Databricks**:

    :::image type="content" source="./media/deploy-cluster-databricks/databricks.png" alt-text="Экран &quot;Поиск Azure Databricks&quot;." border="true":::

1. Выберите **Создать**, чтобы создать учетную запись Azure Databricks:

    :::image type="content" source="./media/deploy-cluster-databricks/databricks-create.png" alt-text="Экран &quot;Создание учетной записи Azure Databricks&quot;." border="true":::

1. Введите следующие значения:

   * **Имя рабочей области**. Укажите имя для рабочей области Databricks.
   * **Регион**. Необходимо выбрать тот же регион, что и для виртуальной сети.
   * **Ценовая категория**. Выберите один из уровней "Стандартный" или "Премиум" или воспользуйтесь бесплатной пробной версией. Дополнительные сведения об этих ценовых категориях см. на [странице цен на Databricks](https://azure.microsoft.com/pricing/details/databricks/).

    :::image type="content" source="./media/deploy-cluster-databricks/select-name.png" alt-text="Экран &quot;Заполнение полей имени рабочей области, региона и ценовой категории для учетной записи Databricks&quot;." border="true":::

1. Затем перейдите на вкладку **Сеть** и введите следующие сведения:

   * **Развертывание рабочей области Azure Databricks в виртуальной сети**. Выберите **Да**.
   * **Виртуальная сеть**. В раскрывающемся списке выберите виртуальную сеть, в которой находится управляемый экземпляр.
   * **Имя общедоступной подсети**. Введите имя для общедоступной подсети.
   * **Диапазон CIDR общедоступной подсети**. Введите диапазон IP-адресов для общедоступной подсети.
   * **Имя частной подсети**. Введите имя для частной подсети.
   * **Диапазон CIDR частной подсети**. Введите диапазон IP-адресов для частной подсети.

   Чтобы избежать конфликтов диапазонов, убедитесь, что выбраны более высокие диапазоны. При необходимости используйте [визуальный калькулятор подсетей](https://www.fryguy.net/wp-content/tools/subnets.html), чтобы разделить диапазоны:

   :::image type="content" source="./media/deploy-cluster-databricks/subnet-calculator.png" alt-text="Экран &quot;Использование калькулятора подсетей виртуальной сети&quot;." border="true":::

   На следующем снимке экрана показан пример сведений на панели "Сеть".

   :::image type="content" source="./media/deploy-cluster-databricks/subnets.png" alt-text="Экран &quot;Указание имен общедоступных и частных подсетей&quot;." border="true":::

1. Выберите **Просмотр и создание**, а затем — **Создать**, чтобы развернуть рабочую область.

1. После ее создания выберите **Запуск рабочей области**.

1. Вы будете перенаправлены на портал Azure Databricks. На портале выберите **Создать кластер**.

1. На панели **Новый кластер** примите значения по умолчанию для всех полей, кроме следующих:

   * **Имя кластера**. Введите имя для кластера.
   * **Databricks Runtime Version** (Версия Databricks Runtime). Выберите Scala 2.11 или более раннюю версию, поддерживаемую соединителем Cassandra.

    :::image type="content" source="./media/deploy-cluster-databricks/spark-cluster.png" alt-text="Экран &quot;Выбор версии Databricks Runtime и кластера Spark&quot;." border="true":::

1. Разверните пункт **Дополнительные параметры** и добавьте следующую конфигурацию. Обязательно замените IP-адреса и учетные данные узла:

    ```java
    spark.cassandra.connection.host <node1 IP>,<node 2 IP>, <node IP>
    spark.cassandra.auth.password cassandra
    spark.cassandra.connection.port 9042
    spark.cassandra.auth.username cassandra
    spark.cassandra.connection.ssl.enabled true
    ```

1. На вкладке **Библиотеки** установите последнюю версию соединителя Spark для Cassandra (*spark-cassandra-connector*) и перезапустите кластер:

    :::image type="content" source="./media/deploy-cluster-databricks/connector.png" alt-text="Экран &quot;Установка соединителя Cassandra&quot;." border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь в дальнейшем использовать кластер с управляемым экземпляром, удалите его, выполнив следующие действия:

1. В меню слева на портале Azure выберите **Группы ресурсов**.
1. Выберите из списка группу ресурсов, созданную для этого краткого руководства.
1. На панели **Обзор** на странице группы ресурсов выберите **Удалить группу ресурсов**.
3. В следующем окне введите имя группы ресурсов, которую требуется удалить, и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать полностью управляемый кластер Apache Spark в виртуальной сети вашего Управляемого экземпляра Azure для кластера Apache Cassandra. Далее вы сможете узнать, как управлять ресурсами кластера и центра обработки данных: 

> [!div class="nextstepaction"]
> [Управление ресурсами службы "Управляемый экземпляр Azure для ресурсов Apache Cassandra" с помощью Azure CLI](manage-resources-cli.md)


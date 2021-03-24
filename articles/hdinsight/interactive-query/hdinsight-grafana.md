---
title: Использование Grafana в Azure HDInsight
description: Узнайте, как получить доступ к панели мониторинга Grafana с кластерами Apache Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/27/2019
ms.openlocfilehash: 81fd3b368f9405192c164ed7a0638caad0cd75fc
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104869758"
---
# <a name="access-grafana-in-azure-hdinsight"></a>Доступ к Grafana в Azure HDInsight

[Grafana](https://grafana.com/) — это популярное средство с открытым кодом для построения графов и панелей мониторинга. Grafana — многофункциональное средство, которое не только позволяет пользователям создавать настраиваемые панели мониторинга с общим доступом, но также предоставляет такие возможности, как шаблонны и скрипты панелей мониторинга, интеграция LDAP, использование нескольких источников данных и многое другое.

Сейчас в Azure HDInsight Grafana поддерживается с типами кластеров Spark, HBase, Kafka и Interactive Requery. Он не поддерживается для кластеров с включенным пакетом безопасности Enterprise.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="create-an-apache-hadoop-cluster"></a>Создание кластера Apache Hadoop

См. [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](../hdinsight-hadoop-create-linux-clusters-portal.md). Для **типа кластера** выберите **Spark**, **Kafka**, **HBase** или **Interactive Query**.

## <a name="access-the-grafana-dashboard"></a>Доступ к панели мониторинга Grafana

1. В веб-браузере перейдите в `https://CLUSTERNAME.azurehdinsight.net/grafana/` папку, где имя_кластера — имя кластера.

1. Введите учетные данные пользователя кластера Hadoop.

1. Появится панель мониторинга Grafana, которая выглядит следующим образом:

    :::image type="content" source="./media/hdinsight-grafana/hdinsight-grafana-dashboard.png " alt-text="Веб-панель мониторинга HDInsight Grafana" border="true":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите созданный кластер, сделав следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В поле **Поиск** в верхней части страницы введите **HDInsight**.

1. Выберите **Кластеры HDInsight** в разделе **Службы**.

1. В появившемся списке кластеров HDInsight щелкните **...** рядом с созданным кластером.

1. Выберите команду **Удалить**. Выберите **Да**.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об анализе данных с помощью HDInsight см. в следующих статьях:

* [Используйте Apache Hive с HDInsight](../hadoop/hdinsight-use-hive.md).

* [Использование MapReduce с HDInsight](../hadoop/hdinsight-use-mapreduce.md).

* Приступая [к работе с Visual Studio Hadoop Tools для HDInsight](../hadoop/apache-hadoop-visual-studio-tools-get-started.md).

---
title: Создание Azure HDInsight-Azure Data Lake Storage 2-го поколения-Portal
description: Узнайте, как использовать Azure Data Lake Storage 2-го поколения с кластерами Azure HDInsight с помощью портала.
author: guyhay
ms.author: guyhay
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020
ms.date: 09/17/2020
ms.openlocfilehash: 5eb6cfc7f0dbc0bc886550e1d601dc52dff66267
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104866375"
---
# <a name="create-a-cluster-with-data-lake-storage-gen2-using-the-azure-portal"></a>Создание кластера с Data Lake Storage 2-го поколения помощью портал Azure

Портал Azure — это веб-инструмент управления службами и ресурсами, размещенными в облаке Microsoft Azure. Из этой статьи вы узнаете, как создавать кластеры Azure HDInsight под управлением Linux с помощью портала Azure. Дополнительные сведения можно получить из [создания кластеров HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Чтобы создать кластер HDInsight, который использует Data Lake Storage 2-го поколения для хранилища, выполните следующие действия, чтобы настроить учетную запись хранения с иерархическим пространством имен.

## <a name="create-a-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем

Создайте управляемое удостоверение, назначаемое пользователем, если у вас его еще нет.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В левом верхнем углу щелкните **создать ресурс**.
1. В поле поиска введите " **пользователь назначен** " и щелкните " **управляемое удостоверение, назначенное пользователем**".
1. Нажмите кнопку **Создать**.
1. Введите имя управляемого удостоверения, выберите подходящую подписку, группу ресурсов и расположение.
1. Нажмите кнопку **Создать**.

Дополнительные сведения о работе управляемых удостоверений в Azure HDInsight см. [в статье управляемые удостоверения в Azure hdinsight](hdinsight-managed-identities.md).

:::image type="content" source="./media/hdinsight-hadoop-use-data-lake-storage-gen2/create-user-assigned-managed-identity-portal.png" alt-text="Создание управляемого удостоверения, назначаемого пользователем":::

## <a name="create-a-storage-account-to-use-with-data-lake-storage-gen2"></a>Создание учетной записи хранения для использования с Data Lake Storage 2-го поколения

Создайте учетную запись хранения для использования с Azure Data Lake Storage 2-го поколения.

1. Войдите на [портал Azure](https://portal.azure.com).
1. В левом верхнем углу щелкните **создать ресурс**.
1. В поле поиска введите **Storage** и щелкните **учетная запись хранения**.
1. Нажмите кнопку **Создать**.
1. На экране **Создание учетной записи хранения** выполните следующие действия.
    1. Выберите подходящую подписку и группу ресурсов.
    1. Введите имя учетной записи хранения с помощью Data Lake Storage 2-го поколения.
    1. Перейдите на вкладку **Дополнительно** .
    1. Щелкните **включено** рядом с **иерархическим пространством имен** в разделе **Data Lake Storage 2-го поколения**.
    1. Щелкните **Просмотреть и создать**.
    1. Нажмите кнопку **Создать**.

Дополнительные сведения о других параметрах во время создания учетной записи хранения см. в разделе [Краткое руководство. Создание учетной записи хранения для Azure Data Lake Storage 2-го поколения](../storage/blobs/create-data-lake-storage-account.md).

:::image type="content" source="./media/hdinsight-hadoop-use-data-lake-storage-gen2/azure-data-lake-storage-account-create-advanced.png" alt-text="Снимок экрана, на котором показано создание учетной записи хранения на портале Azure":::

## <a name="set-up-permissions-for-the-managed-identity-on-the-data-lake-storage-gen2"></a>Настройка разрешений для управляемого удостоверения на Data Lake Storage 2-го поколения

Назначьте управляемое удостоверение роли **владельца данных BLOB-объекта хранилища** в учетной записи хранения.

1. Войдите в свою учетную запись хранения на [портале Azure](https://portal.azure.com).
1. Выберите учетную запись хранения, а затем щелкните **Управление доступом (IAM)** , чтобы отобразить параметры управления доступом для учетной записи. Выберите вкладку **Назначения ролей**, чтобы просмотреть список назначений ролей.

    :::image type="content" source="./media/hdinsight-hadoop-use-data-lake-storage-gen2/portal-access-control.png" alt-text="Снимок экрана с параметрами управления доступом к хранилищу":::

1. Нажмите кнопку **+ добавить назначение роли** , чтобы добавить новую роль.
1. В окне **Добавление назначения ролей** выберите роль **владельца данных BLOB-объекта хранилища** . Затем выберите подписку с управляемым удостоверением и учетной записью хранения. Затем найдите управляемое удостоверение, назначаемое пользователем, которое было создано ранее. Наконец, выберите управляемое удостоверение, которое будет отображаться в разделе **Выбранные элементы**.

    :::image type="content" source="./media/hdinsight-hadoop-use-data-lake-storage-gen2/add-rbac-role3-window.png" alt-text="Снимок экрана, показывающий, как назначить роль Azure":::

1. Щелкните **Сохранить**. Выбранное пользователем удостоверение теперь отображается под выбранной ролью.
1. После завершения первоначальной настройки вы можете создать кластер на портале. Кластер должен быть расположен в том же регионе Azure, что и учетная запись хранения. На вкладке **хранилище** в меню Создание кластера выберите следующие параметры.

    * В качестве **основного типа хранилища** выберите **Azure Data Lake Storage 2-го поколения**.
    * В разделе **Основная учетная запись хранения** найдите и выберите только что созданную учетную запись хранения с хранилищем Data Lake Storage 2-го поколения.

    * В разделе **удостоверение** выберите только что созданное назначенное пользователем управляемое удостоверение.

        :::image type="content" source="./media/hdinsight-hadoop-use-data-lake-storage-gen2/azure-portal-cluster-storage-gentwo.png" alt-text="Параметры хранилища для Azure Data Lake Storage 2-го поколения с Azure HDInsight":::

    > [!NOTE]
    > * Чтобы добавить вторичную учетную запись хранения с Data Lake Storage 2-го поколения, на уровне учетной записи хранения просто назначьте созданное ранее управляемое удостоверение новому Data Lake Storage 2-го поколения, который вы хотите добавить. Рекомендуем добавить дополнительную учетную запись хранения с Data Lake Storage 2-го поколения через колонку "дополнительные учетные записи хранения" в HDInsight не поддерживается.
    > * Вы можете включить RA-GRS или RA-ZRS в учетной записи хранения BLOB-объектов Azure, используемой HDInsight. Однако создание кластера для вторичной конечной точки RA-GRS или RA-ZRS не поддерживается.

## <a name="delete-the-cluster"></a>Удаление кластера

См. раздел [Удаление кластера HDInsight с помощью браузера, PowerShell или Azure CLI](./hdinsight-delete-cluster.md).

## <a name="troubleshoot"></a>Диагностика

Если при создании кластеров HDInsight возникли проблемы, см. раздел [Создание кластеров](./hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Дальнейшие действия

Вы успешно создали кластер HDInsight. Теперь узнайте, как работать с кластером.

### <a name="apache-spark-clusters"></a>Кластеры Apache Spark

* [Настройка кластеров HDInsight под управлением Linux с помощью действий сценариев](hdinsight-hadoop-customize-cluster-linux.md)
* [Создание автономного приложения с использованием Scala](spark/apache-spark-create-standalone-application.md)
* [Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](spark/apache-spark-livy-rest-interface.md)
* [Использование Apache Spark со средствами бизнес-аналитики. Выполнение интерактивного анализа данных с использованием Spark в HDInsight с помощью средств бизнес-аналитики](spark/apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](spark/apache-spark-machine-learning-mllib-ipython.md)

### <a name="apache-hadoop-clusters"></a>Кластеры Apache Hadoop

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hadoop/hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Кластеры Apache HBase

* [Начало работы с Apache HBase в HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Разработка приложений Java для Apache HBase в HDInsight](hbase/apache-hbase-build-java-maven-linux.md)

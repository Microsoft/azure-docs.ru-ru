---
title: Краткое руководство. Apache Hadoop, Apache Hive и портал Azure HDInsight
description: В этом кратком руководстве для создания кластера Hadoop в HDInsight используется портал Azure
keywords: начало работы с hadoop, hadoop под управлением linux, краткое руководство по hadoop, начало работы с hive, краткое руководство по hive
ms.service: hdinsight
ms.topic: quickstart
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.date: 02/24/2020
ms.openlocfilehash: 92520be7e9d2bb83cb61856fe3df38487445ad9a
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104863893"
---
# <a name="quickstart-create-apache-hadoop-cluster-in-azure-hdinsight-using-azure-portal"></a>Краткое руководство. Создание кластера Apache Hadoop в Azure HDInsight с помощью портала Azure

Из этой статьи вы узнаете, как создать кластеры Apache Hadoop в HDInsight с помощью портала Azure, а затем запустить задания Apache Hive в HDInsight. Большинство заданий Hadoop — пакетные. Вы создаете кластер, выполняете несколько заданий, а затем удаляете кластер. В этой статье будут выполнены все три задачи. Подробные объяснения доступных конфигураций см. в статье об [установке кластеров в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md). Дополнительные сведения об использовании портала для создания кластеров см. в статье о [создании кластеров на портале](../hdinsight-hadoop-create-linux-clusters-portal.md).

В этом кратком руководстве для создания кластера Hadoop в HDInsight используется портал Azure. Создать кластер можно также с помощью [шаблона Azure Resource Manager](apache-hadoop-linux-tutorial-get-started.md).

Сейчас в HDInsight доступно [семь типов кластеров](../hdinsight-overview.md#cluster-types-in-hdinsight). Каждый тип кластера поддерживает свой набор компонентов. Все типы кластеров поддерживают инфраструктуру Hive. Дополнительные сведения о поддерживаемых компонентах в HDInsight см. в статье [Что представляют собой компоненты и версии Apache Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)  

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="create-an-apache-hadoop-cluster"></a>Создание кластера Apache Hadoop

В этом разделе вы создадите кластер Hadoop в HDInsight, используя портал Azure.

1. Войдите на [портал Azure](https://portal.azure.com).

1. В меню сверху выберите **+ Create a resource** (+ Создать ресурс).

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/azure-portal-create-resource.png" alt-text="Создание кластера HDInsight ресурса" border="true":::

1. Выберите элементы **Analytics** > **Azure HDInsight**, чтобы перейти на страницу **Создание кластера HDInsight**.

1. На вкладке **Основные сведения** укажите следующую информацию:

   |Свойство  |Описание  |
   |---------|---------|
   |Подписка    |  В раскрывающемся списке выберите подписку Azure, которая используется для кластера. |
   |Группа ресурсов     | В раскрывающемся списке выберите существующую группу ресурсов, а затем **Создать новую**.|
   |Имя кластера   | Введите глобально уникальное имя Имя может содержать до 59 знаков, включая буквы, цифры и дефисы. Первый и последний знаки в имени не могут быть дефисами. |
   |Регион    | В раскрывающемся списке выберите регион, в котором создается кластер.  Выберите ближайшее к себе расположение для повышения производительности. |
   |Тип кластера| Щелкните **Выберите тип кластера**. Затем выберите **Hadoop** в качестве типа кластера.|
   |Версия|В раскрывающемся списке выберите **версию**. Используйте версию по умолчанию, если не уверены, что именно нужно выбрать.|
   |Имя для входа и пароль для кластера    | Имя для входа по умолчанию — **admin**. Длина пароля должна составлять не менее 10 символов. Пароль должен содержать по меньшей мере одну цифру, одну прописную и одну строчную буквы, а также один специальный символ (кроме ' " ` \)). Ни в коем случае **не вводите** распространенные пароли, например Pass@word1.|
   |Имя пользователя для Secure Shell (SSH) | Имя пользователя по умолчанию — **sshuser**.  Можно указать другое имя пользователя SSH. |
   |Использование пароля для входа в кластер для SSH| Установите этот флажок, чтобы использовать одинаковый пароль для пользователя SSH и имени для входа для кластера.|

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/azure-portal-cluster-basics.png" alt-text="Начало работы с HDInsight в Linux. Предоставление базовых значений кластера" border="true":::

   Нажмите кнопку **Далее: Хранилище >>** для перехода к настройкам хранилища.

1. На вкладке **Хранилище** укажите следующие значения:

   |Свойство  |Описание  |
   |---------|---------|
   |Тип первичного хранилища|Используйте значение **службы хранилища Azure** по умолчанию.|
   |Метод выбора|Используйте значение **Выбрать в списке** по умолчанию.|
   |Основную учетную запись хранения|Используйте раскрывающийся список, чтобы выбрать имеющуюся учетную запись хранения, или создайте новую, щелкнув **Создать**. При создании учетной записи имя должно содержать от 3 до 24 символов, включая цифры и строчные буквы|
   |Контейнер|Используйте значение, предоставленное автоматически.|

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/azure-portal-cluster-storage.png" alt-text="Начало работы с HDInsight в Linux. Предоставление значений хранилища кластера" border="true":::

   Каждый кластер зависит от учетной записи [службы хранилища Azure](../hdinsight-hadoop-use-blob-storage.md), [Azure Data Lake Storage 1-го поколения](../hdinsight-hadoop-use-data-lake-storage-gen1.md) или [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md). Она называется учетной записью хранения по умолчанию. Кластер HDInsight должен находиться в том же регионе Azure, что и его учетная запись хранения, используемая по умолчанию. Удаление кластеров не приведет к удалению учетной записи хранения.

   Перейдите на вкладку **Review + create** (Просмотр и создание).

1. На вкладке **Просмотр и создание** проверьте все значения, выбранные на предыдущих шагах.

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/azure-portal-cluster-review-create-hadoop.png" alt-text="Начало работы с HDInsight в Linux. Сводка по кластеру" border="true":::

1. Нажмите кнопку **создания**. Процесс создания кластера занимает около 20 минут.

   После создания кластера на портале Azure отобразится страница с общими сведениями об этом кластере.

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/cluster-settings-overview.png" alt-text="Начало работы с HDInsight в Linux. Параметры кластера" border="true":::

## <a name="run-apache-hive-queries"></a>Выполнение запросов Apache Hive

[Apache Hive](hdinsight-use-hive.md) — это самый популярный компонент службы HDInsight. Существует множество способов выполнения заданий Hive в HDInsight. В этом кратком руководстве используется представление Ambari Hive на портале. Другие способы отправки заданий Hive описаны в статье [Использование Hive в HDInsight](hdinsight-use-hive.md).

> [!NOTE]
> В HDInsight 4.0 больше не используется представление Apache Hive.

1. Чтобы открыть Ambari, на предыдущем экране выберите **Панель мониторинга кластера**.  Вы можете также перейти по адресу `https://ClusterName.azurehdinsight.net`, где `ClusterName` — это кластер, созданный в предыдущем разделе.

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-get-started-open-cluster-dashboard.png" alt-text="Начало работы с HDInsight в Linux. Панель мониторинга кластера" border="true":::

2. Введите имя пользователя Hadoop и пароль, указанные при создании кластера. Имя пользователя по умолчанию — **admin**.

3. Откройте **представление Hive**, как показано на снимке экрана ниже:

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/hdi-select-hive-view.png" alt-text="Выбор представления Hive в Ambari" border="true":::

4. На вкладке **Запрос** вставьте следующие инструкции HiveQL:

   ```sql
   SHOW TABLES;
   ```

    :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/hdi-apache-hive-view1.png" alt-text="Редактор запросов представления Hive HDInsight" border="true":::

5. Нажмите кнопку **Выполнить**. Под вкладкой **Запрос** появится вкладка **Результаты** с информацией о задании.

   Когда запрос будет выполнен, на вкладке **Запрос** появятся результаты этой операции. Вы увидите одну таблицу с именем **hivesampletable**. Этот пример таблицы Hive входит в состав всех кластеров HDInsight.

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-hive-views.png" alt-text="Результаты просмотра Apache Hive HDInsight" border="true":::

6. Повторите шаги 4 и 5 и выполните следующий запрос:

   ```sql
   SELECT * FROM hivesampletable;
   ```

7. Вы также можете сохранить результаты запроса. Нажмите кнопку меню справа и укажите, как это следует сделать: скачать результаты в качестве CSF-файла или сохранить их в учетной записи хранения, связанной с кластером.

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-hive-view-save-results.png" alt-text="Сохранение результатов запроса Apache Hive" border="true":::

Когда задание Hive будет завершено, вы сможете [экспортировать результаты в Базу данных SQL Azure или базу данных SQL Server](apache-hadoop-use-sqoop-mac-linux.md) либо [визуализировать их с помощью Excel](apache-hadoop-connect-excel-power-query.md). Дополнительные сведения об использовании Hive в HDInsight см. в руководстве по [Apache Hive и HiveQL в Azure HDInsight](hdinsight-use-hive.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим кратким руководством кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Так как затраты на кластер во много раз превышают затраты на хранилище, экономически целесообразно удалять неиспользуемые кластеры.

> [!NOTE]  
> Если вы *сразу же* перейдете к следующей статье, чтобы узнать, как выполнять операции извлечения, преобразования и загрузки, то можете не прерывать работу кластера. Дело в том, что в этом руководстве вам придется повторно создать кластер. Но если вы не собираетесь немедленно приступать к изучению следующей статьи, то нужно удалить кластер.

### <a name="to-delete-the-cluster-andor-the-default-storage-account"></a>Удаление кластера и (или) учетной записи хранения по умолчанию

1. Вернитесь на вкладку браузера, на которой открыт портал Azure. Откройте страницу обзора кластера. Если требуется удалить кластер и сохранить учетную запись хранения по умолчанию, щелкните **Удалить**.

   :::image type="content" source="./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-delete-cluster.png" alt-text="Удаление кластера Azure HDInsight" border="true":::

2. Если вам нужно удалить кластер и учетную запись хранения по умолчанию, выберите имя группы ресурсов (выделено на предыдущем снимке экрана), чтобы открыть страницу группы ресурсов.

3. Выберите **Удалить группу ресурсов**, чтобы удалить группу ресурсов, которая содержит кластер и учетную запись хранения по умолчанию. Обратите внимание, что удаление группы ресурсов приводит к удалению учетной записи хранения. Если вы хотите сохранить учетную запись хранения, удалите только кластер.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как с помощью шаблона Resource Manager создать кластер HDInsight под управлением Linux и как выполнять базовые запросы Hive. В следующей статье вы узнаете, как выполнять операции извлечения, преобразования и загрузки с помощью Hadoop в HDInsight.

> [!div class="nextstepaction"]
> [Учебник: извлечение, преобразование и загрузка данных с помощью интерактивного запроса в HDInsight](../interactive-query/interactive-query-tutorial-analyze-flight-data.md)

---
title: Краткое руководство. Создание кластеров Apache Hadoop в Azure HDInsight с помощью шаблонов Resource Manager
description: В этом кратком руководстве вы создадите кластер Apache Hadoop в Azure HDInsight с помощью шаблонов Resource Manager
ms.service: hdinsight
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 03/13/2020
ms.openlocfilehash: 3d1860c629c492287d8caadcf79b0e136b8a3038
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104863825"
---
# <a name="quickstart-create-apache-hadoop-cluster-in-azure-hdinsight-using-arm-template"></a>Краткое руководство. Создание кластера Apache Hadoop в Azure HDInsight с помощью шаблона ARM

В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager (шаблона ARM) создать кластер [Apache Hadoop](./apache-hadoop-introduction.md) в Azure HDInsight. Первоначально технология Hadoop была платформой с открытым кодом для распределенной обработки и анализа наборов больших данных в кластерах. Экосистема Hadoop состоит из взаимосвязанного программного обеспечения и служебных программ, таких как Apache Hive, Apache HBase, Spark, Kafka и т. д.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]
  
В настоящее время в HDInsight доступно [семь типов кластеров](../hdinsight-overview.md#cluster-types-in-hdinsight). Каждый тип кластера поддерживает свой набор компонентов. Все типы кластеров поддерживают инфраструктуру Hive. Дополнительные сведения о поддерживаемых компонентах в HDInsight см. в статье [Что представляют собой различные компоненты Hadoop, доступные в HDInsight?](../hdinsight-component-versioning.md)  

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[:::image type="icon" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-ssh-password%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-ssh-password/).

:::code language="json" source="~/quickstart-templates/101-hdinsight-linux-ssh-password/azuredeploy.json":::

В шаблоне определено два ресурса Azure:

* С помощью [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) создается учетная запись хранения Azure.
* С помощью [Microsoft.HDInsight/cluster](/azure/templates/microsoft.hdinsight/clusters) создается кластер HDInsight.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Нажмите кнопку **Развертывание в Azure** ниже, чтобы войти в Azure и открыть шаблон ARM.

    [:::image type="icon" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-ssh-password%2Fazuredeploy.json)

1. Введите или выберите следующие значения:

    |Свойство  |Описание  |
    |---------|---------|
    |Подписка|В раскрывающемся списке выберите подписку Azure, которая используется для кластера.|
    |Группа ресурсов|В раскрывающемся списке выберите существующую группу ресурсов, а затем **Создать новую**.|
    |Расположение|В качестве значения будет автоматически указано расположение, используемое для группы ресурсов.|
    |Имя кластера,|Введите глобально уникальное имя Для этого шаблона вы можете использовать только строчные буквы и цифры.|
    |Тип кластера | Выберите **hadoop**. |
    |Имя пользователя для входа в кластер|Укажите имя пользователя, по умолчанию — **admin**.|
    |Пароль для входа в кластер|Введите пароль. Длина пароля должна составлять не менее 10 символов. Пароль должен содержать по меньшей мере одну цифру, одну прописную и одну строчную буквы, а также один специальный символ (кроме ' " ` ). |
    |Имя пользователя SSH|Укажите имя пользователя, по умолчанию — **sshuser**.|
    |Пароль SSH|Укажите пароль.|

    Некоторые свойства жестко заданы в шаблоне.  Эти значения можно настроить из шаблона. Дополнительные сведения об этих свойствах см. в инструкциях по [созданию кластеров Apache Hadoop в HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

    > [!NOTE]  
    > Необходимо указать уникальные значения, соответствующие правилам именования. Шаблон не выполняет проверки. Если введенные значения уже используются или не соответствуют правилам именования, после отправки шаблона произойдет ошибка.  

    :::image type="content" source="./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-get-started-arm-template-on-portal.png " alt-text="HDInsight в Linux. Шаблон Resource Manager на портале для начала работы" border="true":::

1. Ознакомьтесь с **УСЛОВИЯМИ ИСПОЛЬЗОВАНИЯ**. Затем установите флажок **Я принимаю указанные выше условия** и нажмите кнопку **Приобрести**. Вы получите уведомление о выполнении развертывания. Процесс создания кластера занимает около 20 минут.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

После создания кластера вы получите уведомление **Развертывание прошло успешно** со ссылкой **Перейти к ресурсу**. На странице группы ресурсов будет указан новый кластер HDInsight и хранилище по умолчанию, связанное с кластером. У каждого кластера есть зависимость учетной записи [службы хранилища Azure](../hdinsight-hadoop-use-blob-storage.md), [Azure Data Lake Storage 1-го поколения](../hdinsight-hadoop-use-data-lake-storage-gen1.md) или [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md). Она называется учетной записью хранения по умолчанию. Кластер HDInsight должен находиться в том же регионе Azure, что и его учетная запись хранения, используемая по умолчанию. Удаление кластеров не приведет к удалению учетной записи хранения.

> [!NOTE]  
> Сведения о других способах создания кластеров, а также о свойствах, используемых в этом кратком руководстве, см. в статье о [создании кластеров HDInsight](../hdinsight-hadoop-provision-linux-clusters.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим кратким руководством кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Так как затраты на кластер во много раз превышают затраты на хранилище, экономически целесообразно удалять неиспользуемые кластеры.

> [!NOTE]  
> Если вы *сразу же* перейдете к следующему руководству, чтобы узнать, как выполнять операции извлечения, преобразования и загрузки, то можете не прерывать работу кластера. Дело в том, что в этом руководстве вам придется повторно создать кластер. Но если вы не собираетесь немедленно приступать к изучению следующего руководства, то нужно удалить кластер.

На портале Azure перейдите в свой кластер и выберите **Удалить**.

:::image type="content" source="./media/apache-hadoop-linux-tutorial-get-started/hdinsight-delete-cluster.png " alt-text="Удаление кластера HDInsight на портале" border="true":::

Кроме того, можно выбрать имя группы ресурсов, чтобы открыть страницу группы ресурсов, а затем щелкнуть **Удалить группу ресурсов**. Вместе с группой ресурсов вы также удалите кластер HDInsight и учетную запись хранения по умолчанию.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать в HDInsight кластер Apache Hadoop с помощью шаблона ARM. В следующей статье вы узнаете, как выполнять операции извлечения, преобразования и загрузки с помощью Hadoop в HDInsight.

> [!div class="nextstepaction"]
> [Учебник: извлечение, преобразование и загрузка данных с помощью интерактивного запроса в HDInsight](../interactive-query/interactive-query-tutorial-analyze-flight-data.md)

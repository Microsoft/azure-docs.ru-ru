---
title: Краткое руководство. Создание кластера служб машинного обучения ML Services с помощью шаблона в Azure HDInsight
description: В этом кратком руководстве показано, как с помощью шаблона Azure Resource Manager создать кластер служб машинного обучения ML Services в Azure HDInsight.
ms.service: hdinsight
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 03/13/2020
ms.openlocfilehash: 4b4196a503bdc0fd455c5d731e11e5c099832c8e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104869538"
---
# <a name="quickstart-create-ml-services-cluster-in-azure-hdinsight-using-arm-template"></a>Краткое руководство. Создание кластера служб машинного обучения ML Services в Azure HDInsight с помощью шаблона ARM

Из этого краткого руководства вы узнаете, как с помощью шаблона Azure Resource Manager (ARM) создать кластер [служб ML](./r-server-overview.md) в Azure HDInsight. Microsoft Machine Learning Server доступен в качестве варианта развертывания при создании кластеров HDInsight в Azure. Тип кластера, предоставляющий этот вариант, называется Службы машинного обучения ML Services. Это позволяет специалистам по обработке и анализу данных, статистикам и R-программистам получать доступ к масштабируемым, распределенным методам аналитики в HDInsight по требованию.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[:::image type="icon" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-rserver%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="review-the-template"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-hdinsight-rserver/).

:::code language="json" source="~/quickstart-templates/101-hdinsight-rserver/azuredeploy.json":::

В шаблоне определено два ресурса Azure:

* С помощью [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts) создается учетная запись хранения Azure.
* С помощью [Microsoft.HDInsight/cluster](/azure/templates/microsoft.hdinsight/clusters) создается кластер HDInsight.

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Нажмите кнопку **Развертывание в Azure** ниже, чтобы войти в Azure и открыть шаблон ARM.

    [:::image type="icon" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Развертывание в Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-rserver%2Fazuredeploy.json)

1. Введите или выберите следующие значения:

    |Свойство |Описание |
    |---|---|
    |Подписка|В раскрывающемся списке выберите подписку Azure, которая используется для кластера.|
    |Группа ресурсов|В раскрывающемся списке выберите существующую группу ресурсов, а затем **Создать новую**.|
    |Расположение|В качестве значения будет автоматически указано расположение, используемое для группы ресурсов.|
    |Имя кластера,|Введите глобально уникальное имя Для этого шаблона вы можете использовать только строчные буквы и цифры.|
    |Имя пользователя для входа в кластер|Укажите имя пользователя, по умолчанию — **admin**.|
    |Пароль для входа в кластер|Введите пароль. Длина пароля должна составлять не менее 10 символов. Пароль должен содержать по меньшей мере одну цифру, одну прописную и одну строчную буквы, а также один специальный символ (кроме ' " ` ). |
    |Имя пользователя SSH|Укажите имя пользователя, по умолчанию — sshuser.|
    |Пароль SSH|Укажите пароль.|

    :::image type="content" source="./media/quickstart-resource-manager-template/resource-manager-template-rserver.png" alt-text="Развертывание шаблона Resource Manager HBase" border="true":::

1. Ознакомьтесь с **условиями использования**. Затем установите флажок **Я принимаю указанные выше условия** и нажмите кнопку **Приобрести**. Вы получите уведомление о выполнении развертывания. Процесс создания кластера занимает около 20 минут.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

После создания кластера вы получите уведомление **Развертывание прошло успешно** со ссылкой **Перейти к ресурсу**. На странице группы ресурсов будет указан новый кластер HDInsight и хранилище по умолчанию, связанное с кластером. У каждого кластера есть зависимость учетной записи [службы хранилища Azure](../hdinsight-hadoop-use-blob-storage.md), [Azure Data Lake Storage 1-го поколения](../hdinsight-hadoop-use-data-lake-storage-gen1.md) или [`Azure Data Lake Storage Gen2`](../hdinsight-hadoop-use-data-lake-storage-gen2.md). Она называется учетной записью хранения по умолчанию. Кластер HDInsight должен находиться в том же регионе Azure, что и его учетная запись хранения, используемая по умолчанию. Удаление кластеров не приведет к удалению учетной записи хранения.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этим кратким руководством кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Так как затраты на кластер во много раз превышают затраты на хранилище, экономически целесообразно удалять неиспользуемые кластеры.

На портале Azure перейдите в свой кластер и выберите **Удалить**.

[Удаление шаблона Resource Manager HBase](./media/quickstart-resource-manager-template/azure-portal-delete-rserver.png)

Кроме того, можно выбрать имя группы ресурсов, чтобы открыть страницу группы ресурсов, а затем щелкнуть **Удалить группу ресурсов**. Вместе с группой ресурсов вы также удалите кластер HDInsight и учетную запись хранения по умолчанию.

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как создать в HDInsight кластер служб ML с помощью шаблона ARM. В следующей статье вы узнаете, как запустить сценарий R с помощью сервера RStudio, демонстрирующий использование Spark для распределенных вычислений сценария R.

> [!div class="nextstepaction"]
> [Краткое руководство: выполнение сценария R в кластере служб машинного обучения в Azure HDInsight с помощью сервера RStudio](./machine-learning-services-quickstart-job-rstudio.md)

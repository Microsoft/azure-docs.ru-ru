---
title: Создание фабрики данных в службе "Фабрика данных Azure" с помощью шаблона Azure Resource Manager (ARM)
description: Создание примера конвейера в службе "Фабрика данных Azure" с помощью шаблона Azure Resource Manager (ARM)
ms.service: data-factory
tags: azure-resource-manager
author: dcstwh
ms.author: weetok
ms.reviewer: jburchel, jingwang
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 07/16/2020
ms.openlocfilehash: c579b6d723533e751e08a80a578195c03e945607
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104783342"
---
# <a name="quickstart-create-an-azure-data-factory-using-arm-template"></a>Краткое руководство. Создание фабрики данных в службе "Фабрика данных Azure" с помощью шаблона ARM

> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](v1/data-factory-build-your-first-pipeline-using-arm.md)
> * [Текущая версия](quickstart-create-data-factory-resource-manager-template.md)

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве объясняется, как создать фабрику данных в службе "Фабрика данных Azure" с использованием шаблона Azure Resource Manager (ARM). Конвейер, который вы создадите в этой фабрике данных, **копирует** данные из одной папки в другую в хранилище BLOB-объектов Azure. Инструкции по **преобразованию** данных с помощью Фабрики данных Azure см. в статье [Преобразование данных с помощью действия Spark в фабрике данных Azure](transform-data-using-spark.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

> [!NOTE]
> Эта статья не содержит подробный обзор службы фабрики данных. Общие сведения о службе фабрики данных Azure см. в статье [Введение в фабрику данных Azure](introduction.md).

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-factory-v2-blob-to-blob-copy%2Fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

### <a name="azure-subscription"></a>Подписка Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

### <a name="create-a-file"></a>Создание файла

Откройте текстовый редактор, например **Блокнот**, и создайте файл **emp.txt** с таким содержимым:

```emp.txt
John, Doe
Jane, Doe
```

Сохраните файл в папке **C:\ADFv2QuickStartPSH**. (Если папка не существует, создайте ее.)

## <a name="review-template"></a>Проверка шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-data-factory-v2-blob-to-blob-copy/).

:::code language="json" source="~/quickstart-templates/101-data-factory-v2-blob-to-blob-copy/azuredeploy.json":::

В шаблоне определено пять ресурсов Azure:

- [Microsoft.Storage/storageAccounts](/azure/templates/Microsoft.Storage/storageAccounts): определяет учетную запись хранения;
- [Microsoft.DataFactory/factories](/azure/templates/microsoft.datafactory/factories): создание Фабрики данных Azure;
- [Microsoft.DataFactory/factories/linkedServices](/azure/templates/microsoft.datafactory/factories/linkedservices): создание связанной службы Фабрики данных Azure;
- [Microsoft.DataFactory/factories/datasets](/azure/templates/microsoft.datafactory/factories/datasets): создание набора данных Фабрики данных Azure;
- [Microsoft.DataFactory/factories/pipelines](/azure/templates/microsoft.datafactory/factories/pipelines): создание конвейера Фабрики данных Azure.

Другие примеры шаблонов для службы "Фабрика данных Azure" можно найти в [коллекции шаблонов быстрого запуска](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datafactory&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Развертывание шаблона

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Этот шаблон создает учетную запись службы "Фабрика данных Azure", учетную запись хранения и контейнер BLOB-объектов.

    [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-data-factory-v2-blob-to-blob-copy%2Fazuredeploy.json)

2. Введите или выберите следующие значения.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-deploy-arm-template.png" alt-text="Развертывание шаблона ARM для службы &quot;Фабрика данных Azure&quot;":::.

    Если значение не указано, используйте значения по умолчанию для создания ресурсов службы "Фабрика данных Azure".

    - **Подписка**: Выберите подписку Azure.
    - **Группа ресурсов.** Щелкните **Создать**, введите уникальное имя группы ресурсов и нажмите кнопку **ОК**.
    - **Регион**. Выберите расположение.  Например, *Восточная часть США*.
    - **Имя фабрики данных.** Сохраните значение по умолчанию.
    - **Расположение.** Сохраните значение по умолчанию.
    - **Имя учетной записи хранения**. Сохраните значение по умолчанию.
    - **Контейнер BLOB-объектов.** Сохраните значение по умолчанию.

## <a name="review-deployed-resources"></a>Просмотр развернутых ресурсов

1. Выберите **Перейти к группе ресурсов**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-go-to-resource-group.png" alt-text="Группа ресурсов":::

2.  Убедитесь, что Фабрика данных Azure успешно создана.
    1. Имя фабрики данных в службе "Фабрика данных Azure" имеет формат datafactory\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-sample.png" alt-text="Пример фабрики данных":::

2. Убедитесь, что учетная запись хранения успешно создана.
    1. Имя учетной записи хранения имеет формат storage\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-storage-account.png" alt-text="Учетная запись хранения":::

3. Выберите созданную учетную запись хранения и щелкните **Контейнеры**.
    1. На странице **Контейнеры** выберите созданный контейнер больших двоичных объектов.
        1. Имя контейнера BLOB-объектов имеет формат blob\<uniqueid\>.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-blob-container.png" alt-text="Контейнер BLOB-объектов":::

### <a name="upload-a-file"></a>Отправка файла

1. На странице **Контейнеры** выберите **Отправить**.

2. На панели справа щелкните поле **Файлы**, а затем найдите и выберите ранее созданный файл **emp.txt**.

3. Разверните заголовок **Дополнительно**.

4. В поле **Отправить в папку** введите *input*.

5. Нажмите кнопку **Отправить**. В списке должен отобразиться файл **emp.txt** с состоянием отправки.

6. Щелкните **Закрыть** значок (**X**), чтобы закрыть страницу **Отправка BLOB-объектов**.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-upload-blob-file.png" alt-text="Отправка файла в папку входных данных":::

Не закрывайте страницу контейнера, так как здесь вы сможете проверить выходные данные в конце этого краткого руководства.

### <a name="start-trigger"></a>Триггер запуска

1. На странице **Фабрики данных** щелкните созданную фабрику данных. 

2. Щелкните плитку **Создание и мониторинг**. 

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-author-monitor-tile.png" alt-text="Создание и мониторинг":::

2. Перейдите на вкладку **Создание** :::image type="icon" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-author.png" border="false":::.

3. Выберите созданный конвейер (ArmtemplateSampleCopyPipeline).

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-pipelines.png" alt-text="Конвейер шаблона ARM":::

4. Выберите **Добавить триггер** > **Trigger Now** (Активировать сейчас).

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-trigger-now.png" alt-text="Триггер":::

5. На панели справа в разделе **Запуск конвейера** щелкните **ОК**.

### <a name="monitor-the-pipeline"></a>Мониторинг конвейера

1. Выберите вкладку **Мониторинг** :::image type="icon" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-monitor.png" border="false":::.

2. Вы увидите, что выполняется действие, связанное с выполнением конвейера. В этом кратком руководстве конвейер содержит только один тип действия: копирование. Поэтому вы видите выполнение именно этого действия.

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-successful-run.png" alt-text="Успешное выполнение":::

### <a name="verify-the-output-file"></a>Проверка файла выходных данных

Этот конвейер автоматически создает папку выходных данных output в контейнере BLOB-объектов. Затем он копирует файл emp.txt из входной папки в выходную. 

1. На портале Azure откройте страницу **Контейнеры** и нажмите кнопку **Обновить**, чтобы появилась папка output. 

2. В списке папок выберите **output**.

3. Убедитесь, что файл **emp.txt** скопирован в папку output. 

    :::image type="content" source="media/quickstart-create-data-factory-resource-manager-template/data-factory-arm-template-output.png" alt-text="Выходные данные":::

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ресурсы, созданные в ходе работы с этим руководством, двумя способами. Вы можете удалить [группу ресурсов Azure](../azure-resource-manager/management/delete-resource-group.md), которая содержит все связанные ресурсы. Если же вы хотите сохранить другие ресурсы, удалите только фабрику данных, созданную в этом руководстве.

При удалении группы ресурсов будут удалены все входящие в нее ресурсы, включая фабрики данных. Выполните следующую команду, чтобы удалить всю группу ресурсов: 

```azurepowershell-interactive
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

Если вам нужно удалить только фабрику данных, а не всю группу ресурсов, выполните следующую команду: 

```azurepowershell-interactive
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать фабрику данных в службе "Фабрика данных Azure" с помощью шаблона ARM и проверить это развертывание. Дополнительные сведения о службе "Фабрика данных Azure" и Azure Resource Manager см. в статьях ниже.

- [Документация по фабрике данных Azure](index.yml)
- Сведения об [Azure Resource Manager](../azure-resource-manager/management/overview.md)
- Изучите другие [шаблоны ARM для службы "Фабрика данных Azure"](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datafactory&pageNumber=1&sort=Popular)
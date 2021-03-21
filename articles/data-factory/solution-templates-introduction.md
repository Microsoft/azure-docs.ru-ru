---
title: Общие сведения о шаблонах
description: Узнайте, как использовать предварительно определенный шаблон, чтобы быстро начать работу с Фабрикой данных Azure.
ms.service: data-factory
ms.topic: conceptual
ms.author: daperlov
author: djpmsft
ms.custom: seo-lt-2019
ms.date: 01/04/2019
ms.openlocfilehash: 8c0e4db2bc686fff2bd718f45c63a0fc26f6cd55
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100375397"
---
# <a name="templates"></a>Шаблоны

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Шаблоны — это предварительно определенные конвейеры Фабрики данных Azure, которые позволяют быстро начать работу с этой службой. Шаблоны нужны в тех случаях, когда вы еще не знакомы с Фабрикой данных и хотите быстро начать работу с ней. Эти шаблоны позволяют сократить время разработки для создания проектов интеграции данных, тем самым улучшая производительность разработчика.

## <a name="create-data-factory-pipelines-from-templates"></a>Создание конвейеров Фабрики данных на основе шаблонов

Вы можете приступить к созданию конвейера Фабрики данных на основе шаблона двумя приведенными ниже способами:

1.  Выберите **Create pipeline from template** (Создать конвейер с помощью шаблона) на странице Overview (Обзор), чтобы открыть коллекцию шаблонов.

    ![Открытие коллекции шаблонов на странице Overview (Обзор)](media/solution-templates-introduction/templates-intro-image1.png)

1.  На вкладке автор в обозреватель ресурсов выберите **+** , затем выберите **конвейер из шаблона** , чтобы открыть галерею шаблонов.

    ![Открытие коллекции шаблонов на вкладке Author (Автор)](media/solution-templates-introduction/templates-intro-image2.png)

## <a name="template-gallery"></a>Коллекция шаблонов

![Коллекция шаблонов](media/solution-templates-introduction/templates-intro-image3.png)

### <a name="out-of-the-box-data-factory-templates"></a>Готовые шаблоны Фабрики данных

Фабрика данных использует шаблоны Azure Resource Manager для сохранения шаблонов конвейера фабрики данных. Вы можете просмотреть все шаблоны диспетчер ресурсов, а также файл манифеста, используемый для использования шаблонов фабрики данных, в [официальном репозитории GitHub фабрики данных Azure](https://github.com/Azure/Azure-DataFactory/tree/master/templates). Предварительно определенные шаблоны, предоставляемые корпорацией Майкрософт, включают, в частности, следующие элементы:

-   Шаблоны копирования:

    -   [Групповое копирование из базы данных](solution-template-bulk-copy-with-control-table.md)
    
    -   [Копирование новых файлов с использованием параметра LastModifiedDate](solution-template-copy-new-files-lastmodifieddate.md)

    -   [Копирование нескольких контейнеров файлов между файловыми хранилищами](solution-template-copy-files-multiple-containers.md)

    -   [Перемещение файлов](solution-template-move-files.md)

    -   [Разностное копирование из базы данных](solution-template-delta-copy-with-control-table.md)

    -   Копировать из \<source\> в \<destination\>

        -   [Из Amazon S3 в Azure Data Lake Store 2-го поколения](solution-template-migration-s3-azure.md)

        -   Из Google Big Query в Azure Data Lake Store 2-го поколения

        -   Из HDF в Azure Data Lake Store 2-го поколения

        -   Из Netezza в Azure Data Lake Store 1-го поколения

        -   Из локального экземпляра SQL Server в Базу данных SQL Azure

        -   Из SQL Server локальной среды в Azure синапсе Analytics

        -   Из Oracle локально в Azure синапсе Analytics

-   Шаблоны SSIS

    -   Планирование Azure-SSIS Integration Runtime для выполнения пакетов SSIS

-   Шаблоны преобразования

    -   [Извлечение, преобразование и загрузка с Azure Databricks](solution-template-databricks-notebook.md)

### <a name="my-templates"></a>Мои шаблоны

Конвейер можно также сохранить как шаблон, выбрав **Save as template** (Сохранить как шаблон) на вкладке Pipeline (Конвейер).

![Сохранение конвейера как шаблон](media/solution-templates-introduction/templates-intro-image4.png)

В разделе **My Templates** (Мои шаблоны) в коллекции шаблонов можно просмотреть конвейеры, сохраненные в виде шаблонов. Также их можно увидеть в разделе **Templates** (Шаблоны) обозревателя ресурсов.

![Мои шаблоны](media/solution-templates-introduction/templates-intro-image5.png)

> [!NOTE]
> Чтобы использовать функцию My Templates (Мои шаблоны), необходимо включить интеграцию с GIT. Поддерживаются GIT Azure DevOps и GitHub.

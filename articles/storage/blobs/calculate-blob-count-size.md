---
title: Расчет числа и размера BLOB-объектов с помощью инвентаризации службы хранилища Azure
description: Узнайте, как вычислить количество и общий размер больших двоичных объектов на контейнер.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 03/10/2021
ms.service: storage
ms.subservice: blobs
ms.topic: how-to
ms.openlocfilehash: 92e5b00cd655677cdc3096bc2142dfe1b704adf2
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102638678"
---
# <a name="calculate-blob-count-and-total-size-per-container-using-azure-storage-inventory"></a>Расчет числа BLOB-объектов и общего размера на контейнер с помощью инвентаризации хранилища Azure

В этой статье используется функция инвентаризации хранилища BLOB-объектов Azure и Azure синапсе для вычисления числа больших двоичных объектов и общего размера больших двоичных объектов на контейнер. Эти значения полезны при оптимизации использования большого двоичного объекта на контейнер.

Метаданные большого двоичного объекта не включаются в этот метод. Функция инвентаризации хранилища BLOB-объектов Azure использует [список больших двоичных объектов](/rest/api/storageservices/list-blobs) REST API с параметрами по умолчанию. Таким образом, в примере не поддерживаются моментальные снимки, контейнеры "$" и т. д.

> [!IMPORTANT]
> Инвентаризация BLOB-объектов сейчас доступна в **предварительной версии**. Ознакомьтесь с [дополнительными условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не включены в общедоступную версию.

## <a name="enable-inventory-reports"></a>Включение отчетов об инвентаризации

Первым шагом в этом методе является [Включение отчетов об инвентаризации](blob-inventory.md#enable-inventory-reports) в вашей учетной записи хранения. После включения отчетов об инвентаризации для создания первого отчета может потребоваться подождать до 24 часов.

При наличии отчета об инвентаризации для анализа предоставьте самому себе доступ на чтение BLOB-объекта к контейнеру, в котором находится CSV-файл отчета.

1. Перейдите к контейнеру с файлом отчета CSV инвентаризации.
1. Выберите **Управление доступом (IAM)**, а затем **добавьте назначения ролей** .

    :::image type="content" source="media/calculate-blob-count-size/access.png" alt-text="Выберите добавить назначения ролей.":::

1. В раскрывающемся списке **роль** выберите **модуль чтения данных BLOB-объекта хранилища** .

    :::image type="content" source="media/calculate-blob-count-size/add-role-assignment.png" alt-text="Добавление роли читателя данных BLOB-объекта хранилища из раскрывающегося списка":::

1. Введите адрес электронной почты учетной записи, которую вы используете для запуска отчета в поле **выбора** .

## <a name="create-an-azure-synapse-workspace"></a>Создание рабочей области Azure Synapse

Затем [Создайте рабочую область Azure синапсе](/azure/synapse-analytics/get-started-create-workspace) , в которой будет выполняться SQL-запрос для передачи результатов инвентаризации.

## <a name="create-the-sql-query"></a>Создание SQL запроса

После создания рабочей области Azure синапсе выполните следующие действия.

1. Перейдите по адресу [https://web.azuresynapse.net](https://web.azuresynapse.net).
1. Выберите вкладку **Разработка** на левой границе.
1. Выберите крупный знак "плюс" (+), чтобы добавить элемент.
1. Выберите **скрипт SQL**.

    :::image type="content" source="media/calculate-blob-count-size/synapse-sql-script.png" alt-text="Выберите скрипт SQL для создания нового запроса":::

## <a name="run-the-sql-query"></a>Выполнение SQL запроса

1. Добавьте следующий SQL-запрос в рабочую область Azure синапсе для [чтения CSV-файла инвентаризации](/azure/synapse-analytics/sql/query-single-csv-file#read-a-csv-file).

    В качестве `bulk` параметра используйте URL-адрес файла CSV отчета по инвентаризации, который требуется проанализировать.

    ```sql
    SELECT LEFT([Name], CHARINDEX('/', [Name]) - 1) AS Container, 
            COUNT(*) As TotalBlobCount,
            SUM([Content-Length]) As TotalBlobSize
    FROM OPENROWSET(
        bulk '<URL to your inventory CSV file>',
        format='csv', parser_version='2.0', header_row=true
    ) AS Source
    GROUP BY LEFT([Name], CHARINDEX('/', [Name]) - 1)
    ```

1. Назовите запрос SQL на панели свойств справа.

1. Опубликуйте запрос SQL, нажав клавиши CTRL + S или нажав кнопку **опубликовать все** .

1. Нажмите кнопку **выполнить** , чтобы выполнить SQL запрос. Количество больших двоичных объектов и общий размер каждого контейнера отображаются в области **результатов** .

    :::image type="content" source="media/calculate-blob-count-size/output.png" alt-text="Выходные данные выполнения скрипта для вычисления числа BLOB-объектов и общего размера.":::

## <a name="next-steps"></a>Дальнейшие действия

- [Использование инвентаризации BLOB-объектов службы хранилища Azure для управления данными больших двоичных объектов](blob-inventory.md)
- [Вычисление общего указываемого в счете размера контейнера больших двоичных объектов](../scripts/storage-blobs-container-calculate-billing-size-powershell.md)

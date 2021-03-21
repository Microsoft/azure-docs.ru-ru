---
title: Мониторинг Azure Data Lake Analytics-портал Azure
description: В этой статье описано, как с помощью портала Azure устранить неполадки с заданиями Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: troubleshooting
ms.date: 12/05/2016
ms.openlocfilehash: ddcf41a333d261bd9c5f669fde724a25eeba670e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220284"
---
# <a name="monitor-jobs-in-azure-data-lake-analytics-using-the-azure-portal"></a>Мониторинг заданий в Azure Data Lake Analytics с помощью портала Azure

## <a name="to-see-all-the-jobs"></a>Просмотр всех заданий

1. На портале Azure щелкните **Microsoft Azure** в левом верхнем углу.

2. Щелкните элемент с именем вашей учетной записи аналитики озера данных.  Сводные данные задания отображаются на элементе **Управление заданиями** .

   ![Аналитика озера данных Azure: управление заданиями](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-job-management.png)

    Элемент управления заданиями позволяет взглянуть на состояние заданий. Обратите внимание, что здесь имеется задание, завершившееся сбоем.
3. Щелкните элемент **Управление заданиями** для просмотра заданий. Задания делятся на три категории: **Выполняется**, **В очереди** и **Завершено**. Задание, завершившееся сбоем, будет отображено в разделе **Завершено** . Оно должно быть первым в списке. При наличии большого числа заданий можно щелкнуть **Фильтр** , чтобы упростить их поиск.

   ![Аналитика озера данных Azure: фильтрация заданий](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-filter-jobs.png)

4. Щелкните невыполненное задание из списка, чтобы открыть сведения о задании:

   ![Аналитика озера данных Azure: невыполненное задание](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-failed-job.png)

    Обратите внимание на кнопку **Отправить повторно** . После устранения проблемы можно отправить задание повторно.

5. Щелкните выделенную часть на предыдущем снимке экрана, чтобы открыть сведения об ошибке.  Отобразится примерно следующий текст:

   ![Аналитика озера данных Azure: сведения о невыполненном задании](./media/data-lake-analytics-monitor-and-troubleshoot-tutorial/data-lake-analytics-failed-job-details.png)

   В нем сообщается, что исходная папка не найдена.

6. Щелкните **Дублировать скрипт**.

7. Измените путь **с** на: `/Samples/Data/SearchLog.tsv`

8. Щелкните **Отправить задание**.

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор аналитики озера данных Azure](data-lake-analytics-overview.md)
* [Начало работы с аналитикой озера данных Azure с помощью Azure PowerShell](data-lake-analytics-get-started-powershell.md)
* [Управление Azure Data Lake Analytics с помощью портал Azure](data-lake-analytics-manage-use-portal.md)

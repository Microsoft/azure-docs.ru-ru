---
title: Azure Data Lake Analytics запросов — Visual Studio
description: Сведения об установке средств Data Lake для Visual Studio, разработке и тестировании скриптов U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 08/30/2019
ms.openlocfilehash: d3812cb27d2d9ea7a49038f566f8b4cc5d779172
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95241698"
---
# <a name="develop-u-sql-scripts-by-using-data-lake-tools-for-visual-studio"></a>Разработка скриптов U-SQL с помощью средств Data Lake для Visual Studio

[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Средства Azure Data Lake и Stream Analytics включают функции, близкие к функциям двух служб Azure: Azure Data Lake Analytics и Azure Stream Analytics. Дополнительные сведения о Azure Stream Analytics сценариях см. в разделе [средства Azure Stream Analytics для Visual Studio](../stream-analytics/stream-analytics-tools-for-visual-studio-install.md).

В этой статье описывается, как использовать Visual Studio для создания учетных записей Azure Data Lake Analytics. Задания можно определять в [U-SQL](data-lake-analytics-u-sql-get-started.md)и отправлять задания в службу Data Lake Analytics. Дополнительные сведения о Data Lake Analytics см. в [обзоре Azure Data Lake Analytics](data-lake-analytics-overview.md).

> [!IMPORTANT]
> Рекомендуется выполнить обновление до Средства Azure Data Lake для Visual Studio версии 2.3.3000.4 или более поздней. Предыдущие версии являются устаревшими и недоступными для скачивания.
>
> 1. Проверьте, не используете ли вы Средства Azure Data Lake для Visual Studio более ранней версии, чем 2.3.3000.4.
>
>    ![Проверка версии средств](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-about-data-lake.png)
>
> 1. Если у вас более ранняя версия, чем 2.3.3000.4, обновите Средства Azure Data Lake для Visual Studio, посетив сайт центра загрузки:
>    - [Для Visual Studio 2017 и 2019](https://marketplace.visualstudio.com/items?itemName=ADLTools.AzureDataLakeandStreamAnalyticsTools)
>    - [для Visual Studio 2013 и 2015.](https://www.microsoft.com/en-us/download/details.aspx?id=49504)

## <a name="prerequisites"></a>Предварительные условия

* **Visual Studio**: поддерживаются все выпуски, кроме Express.

  * Visual Studio 2019
  * Visual Studio 2017
  * Visual Studio 2015
  * Visual Studio 2013

* **Microsoft Azure SDK для .NET** (версии 2.7.1 или выше). Можно установить его с помощью [установщика веб-платформы](https://www.microsoft.com/web/downloads/platform.aspx).
* Учетная запись **Data Lake Analytics** . Чтобы создать учетную запись, ознакомьтесь со статьей [Руководство. Начало работы с Azure Data Lake Analytics с помощью портала Azure](data-lake-analytics-get-started-portal.md).

## <a name="install-azure-data-lake-tools-for-visual-studio"></a>Установка средств Azure Data Lake для Visual Studio

Для работы с этим руководством необходимо установить Data Lake Tools для Visual Studio. Дополнительные сведения см. в разделе [Install Data Lake Tools for Visual Studio](data-lake-analytics-data-lake-tools-install.md).

## <a name="connect-to-an-azure-data-lake-analytics-account"></a>Подключение к учетной записи Azure Data Lake Analytics

1. Запустите Visual Studio.

1. Откройте **Обозреватель Data Lake Analytics** , выбрав **вид**  >  **Data Lake Analytics Explorer**.

1. Щелкните правой кнопкой мыши **Azure**, а затем выберите **подключиться к Microsoft Azure подписке**. В поле **Вход в учетную запись** выполните инструкции.

1. В **Обозреватель сервера** выберите Data Lake Analytics **Azure**  >  . Отобразится список учетных записей Data Lake Analytics.

## <a name="write-your-first-u-sql-script"></a>Создание первого скрипта U-SQL

Ниже приводится простой скрипт U-SQL. Он определяет небольшой набор данных и по умолчанию записывает его в хранилище Data Lake Store как файл с именем `/data.csv`.

```sql
USE DATABASE master;
USE SCHEMA dbo;
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

## <a name="submit-a-data-lake-analytics-job"></a>Отправка задания аналитики озера данных

1. В Visual Studio выберите **Файл** > **Создать** > **Проект**.

1. Выберите тип **проекта U-SQL** и нажмите кнопку **Далее**. В окне **Настройка нового проекта** выберите **создать**.

   Visual Studio создает решение, содержащее файл **script. usql** .

1. Вставьте скрипт из [записи первого скрипта U-SQL](#write-your-first-u-sql-script) в окно **script. usql** .

1. В **Обозреватель решений** щелкните правой кнопкой мыши **script. Usql** и выберите **отправить сценарий**.

1. В списке **Отправить задание** выберите учетную запись Data Lake Analytics и нажмите кнопку **Отправить**.

   ![Отправка проекта U-SQL в Visual Studio](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-submit-job-vs2019.png)

После отправки задания откроется вкладка **Представление задания**, на которой отображается ход выполнения задания.

* В окне **Сводные данные задания** представлена сводка задания.
* В окне **Граф задания** визуализируется ход выполнения задания.
* В окне **Операции с метаданными** представлены сведения обо всех действиях, выполненных в каталоге U-SQL.
* В окне **Данные** отображаются все входные и выходные данные.
* **Состояние журнала** отображает сведения о временной шкале и состоянии.
* В ходе **анализа Au** показано, сколько использовано в задании, и изучите моделирование различных СТРАТЕГИЙ распределения Au.
* В окне **Диагностика** представлены данные расширенного анализа для выполнения задания и оптимизации производительности.

![График выполнения задания аналитики озера данных U-SQL в Visual Studio](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-analytics-data-lake-tools-performance-graph.png)

Чтобы просмотреть Последнее состояние задания и обновить экран, выберите **Обновить**.

## <a name="check-job-status"></a>Проверка состояния задания

1. В **Обозреватель сервера** выберите Data Lake Analytics **Azure**  >  .

1. Разверните окно имени учетной записи Data Lake Analytics.

1. Дважды щелкните **Задания**.

1. Выберите задание, отправленное ранее.

## <a name="see-the-job-output"></a>Просмотр выходных данных задания

1. В **Обозреватель сервера** перейдите к отправленному заданию.

1. Перейдите на вкладку **Данные**.

1. На вкладке **Job Outputs** (Выходные данные задания) выберите файл `"/data.csv"`.

## <a name="next-steps"></a>Дальнейшие действия

* [Тестирование и отладка заданий U-SQL с помощью локального выполнения и пакета SDK U-SQL для Azure Data Lake](data-lake-analytics-data-lake-tools-local-run.md)
* [Локальный запуск и локальная отладка U-SQL в Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
* [Использование средств Azure Data Lake для Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)

---
title: Записные книжки синапсе Studio
description: Из этой статьи вы узнаете, как создавать и разрабатывать записные книжки Azure синапсе Studio для подготовки и визуализации данных.
services: synapse analytics
author: ruixinxu
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 10/19/2020
ms.author: ruxu
ms.reviewer: ''
ms.custom: devx-track-python
ms.openlocfilehash: c5dfd442bb52a5b1d319bd0a40b656d549134e7e
ms.sourcegitcommit: c94e282a08fcaa36c4e498771b6004f0bfe8fb70
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105612331"
---
# <a name="create-develop-and-maintain-synapse-studio-notebooks-in-azure-synapse-analytics"></a>Создание, разработка и обслуживание записных книжек синапсе Studio в Azure синапсе Analytics

Записная книжка синапсе Studio — это веб-интерфейс для создания файлов, содержащих код в реальном времени, визуализации и текст описания. Записные книжки отлично подходят для проверки идей и использования быстрых экспериментов, чтобы получить аналитические сведения по данным. Они также широко используются при подготовке и визуализации данных, машинном обучении и других сценариях с большими данными.

С помощью записной книжки Azure Synapse Studio можно выполнять следующие задачи.

* Начните работу, потратив минимум усилий на настройку.
* Обеспечьте безопасность данных благодаря встроенным функциям безопасности предприятия.
* Анализируйте данные в необработанных форматах (CSV, txt, JSON и т. д.), обработанных форматах файлов (parquet, Delta Lake, ORC и т. д.) и файлах табличных данных SQL с использованием Spark и SQL.
* Повысьте продуктивность труда с расширенными возможностями разработки и встроенной визуализацией данных.

В этой статье описывается использование записных книжек в Azure Synapse Studio.

## <a name="preview-of-the-new-notebook-experience"></a>Предварительная версия нового интерфейса записной книжки
Команда синапсе развернула новый компонент записных книжек в синапсе Studio, чтобы обеспечить единообразную работу с ноутбуками для клиентов корпорации Майкрософт и максимально увеличить возможности обнаружения, повышения производительности, совместного использования и совместной работы. Новый интерфейс записной книжки готов к предварительной версии. Нажмите кнопку **Предварительная версия компонентов** на панели инструментов записной книжки, чтобы включить ее. В приведенной ниже таблице отслеживается Сравнение характеристик существующей записной книжки (так называемое «классическая Записная книжка») с новой предварительной версией.  

|Функция|Классическая Записная книжка|Предварительный просмотр записной книжки|
|--|--|--|
|% выполнения| Не поддерживается | &#9745;|
|% журнала| Не поддерживается |&#9745;
|% загрузки| Не поддерживается |&#9745;|
|%% HTML| Не поддерживается |&#9745;|
|Перетаскивание для перемещения ячейки| Не поддерживается |&#9745;|
|Постоянные выходные данные вывода ()|&#9745;| Недоступно |
|Форматирование текстовой ячейки с помощью кнопок панели инструментов|&#9745;| Недоступно |
|Отменить операцию ячейки| &#9745;| Недоступно |


## <a name="create-a-notebook"></a>Создание записной книжки

Существует два способа создания записной книжки. Вы можете создать записную книжку или импортировать имеющуюся записную книжку в рабочую область Azure Synapse из **обозревателя объектов**. Записные книжки Azure Synapse Studio могут распознавать стандартные IPYNB-файлы Jupyter Notebook.

![Создание записной книжки для импорта](./media/apache-spark-development-using-notebooks/synapse-create-import-notebook-2.png)

## <a name="develop-notebooks"></a>Разработка записных книжек

Записные книжки состоят из ячеек, являющихся отдельными блоками кода или текста, которые можно запускать независимо или в группе.

### <a name="add-a-cell"></a>Добавление ячейки

Существует несколько способов добавить в записную книжку новую ячейку.

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

1. Разверните верхнюю левую кнопку **+ Cell** (Добавить ячейку) и выберите **Добавить ячейку кода** или **Добавить текстовую ячейку**.

    ![add-cell-with-cell-button](./media/apache-spark-development-using-notebooks/synapse-add-cell-1.png)

2. Наведите указатель мыши между двумя ячейками и выберите **Добавить код** или **Добавьте текст**.

    ![add-cell-between-space](./media/apache-spark-development-using-notebooks/synapse-add-cell-2.png)

3. Используйте [сочетания клавиш в режиме команд](#shortcut-keys-under-command-mode). Нажмите клавишу **A**, чтобы вставить ячейку над текущей ячейкой. Нажмите клавишу **B**, чтобы вставить ячейку под текущей ячейкой.


# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

1. Разверните верхнюю левую **и кнопку ячейки** и выберите **ячейку кода** или **ячейку Markdown**.
    ![Кнопка "добавить-Azure-Записная книжка-ячейка"](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-1.png)
2. Щелкните знак "плюс" в начале ячейки и выберите **ячейку кода** или **ячейку Markdown**.

    ![Добавление-Azure-Notebook-ячейка в промежутке между пробелами](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-add-cell-2.png)

3. Используйте [сочетание клавиш азнб в режиме команд](#shortcut-keys-under-command-mode). Нажмите клавишу **A**, чтобы вставить ячейку над текущей ячейкой. Нажмите клавишу **B**, чтобы вставить ячейку под текущей ячейкой.

---

### <a name="set-a-primary-language"></a>Установка основного языка

Записные книжки Azure Synapse Studio поддерживают четыре языка Apache Spark:

* PySpark (Python)
* Spark (Scala)
* SparkSQL
* .NET для Apache Spark (C#)

Установить основной язык для новых добавленных ячеек можно из раскрывающегося списка на верхней панели команд.

   ![default-synapse-language](./media/apache-spark-development-using-notebooks/synapse-default-language.png)

### <a name="use-multiple-languages"></a>Использование нескольких языков

В одной записной книжке можно использовать несколько языков, указав магическую языковую команду в начале ячейки. В следующей таблице перечислены магические команды, которые используются для переключения языков ячеек.

|Магические команды |Язык | Описание |  
|---|------|-----|
|%%pyspark| Python | Выполните запрос **Python** к контексту Spark.  |
|%%spark| Scala | Выполните запрос **Scala** к контексту Spark.  |  
|%%sql| SparkSQL | Выполните запрос **SparkSQL** к контексту Spark.  |
|%%csharp | .NET для Spark C# | Выполните запрос **.NET для Spark C#** к контексту Spark. |

На следующем рисунке показан пример того, как можно написать запрос PySpark с помощью магической команды **%%pyspark** или запроса SparkSQL с магической командой **%%sql** в записной книжке **Spark(Scala)** . Обратите внимание, что основным языком для записной книжки является pySpark.

   ![Команды Magic синапсе Spark](./media/apache-spark-development-using-notebooks/synapse-spark-magics.png)

### <a name="use-temp-tables-to-reference-data-across-languages"></a>Использование временных таблиц для ссылки на данные на разных языках

В записной книжке Synapse Studio вы не можете ссылаться на данные или переменные напрямую на разных языках. В Spark на временную таблицу можно ссылаться на разных языках. Ниже приведен пример того, как прочитать `Scala` DataFrame в `PySpark` и `SparkSQL`, используя в качестве обходного решения временную таблицу Spark.

1. В ячейке 1 прочтите кадр данных из соединителя пула SQL с помощью Scala и создайте временную таблицу.

   ```scala
   %%scala
   val scalaDataFrame = spark.read.sqlanalytics("mySQLPoolDatabase.dbo.mySQLPoolTable")
   scalaDataFrame.createOrReplaceTempView( "mydataframetable" )
   ```

2. В ячейке 2 выполните запрос на данные с помощью SQL Spark.
   
   ```sql
   %%sql
   SELECT * FROM mydataframetable
   ```

3. В ячейке 3 используйте данные в PySpark.

   ```python
   %%pyspark
   myNewPythonDataFrame = spark.sql("SELECT * FROM mydataframetable")
   ```

### <a name="ide-style-intellisense"></a>Функция IntelliSense в IDE

Записные книжки Azure Synapse Studio интегрированы с редактором Monaco, что позволяет работать с функцией IntelliSense в IDE в редакторе ячеек. Выделение синтаксиса, маркер ошибок и автоматическое завершение кода позволяют быстрее создавать код и выделять проблемы.

Возможности IntelliSense находятся на разных уровнях зрелости для разных языков. Используйте следующую таблицу, чтобы узнать, что поддерживается.

|Языки| Выделение синтаксиса | Маркер синтаксических ошибок  | Завершение кода синтаксиса | Завершение кода переменной| Завершение кода системной функции| Завершение кода пользовательской функции| Автоматический отступ | Свертывание кода|
|--|--|--|--|--|--|--|--|--|
|PySpark (Python)|Да|Да|Да|Да|Да|Да|Да|Да|
|Spark (Scala)|Да|Да|Да|Да|-|-|-|Да|
|SparkSQL|Да|Да|-|-|-|-|-|-|
|.NET для Spark C#|Да|-|-|-|-|-|-|-|

### <a name="format-text-cell-with-toolbar-buttons"></a>Форматирование текстовой ячейки с помощью кнопок панели инструментов

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Для выполнения общих действий markdown можно использовать кнопки форматирования на панели инструментов текстовых ячеек. Они включают выделение текста полужирным шрифтом, выделение курсивом, вставку фрагментов кода, вставку неупорядоченного списка, вставку упорядоченного списка, а также вставку изображения из URL.

  ![Панель инструментов текстовой ячейки синапсе](./media/apache-spark-development-using-notebooks/synapse-text-cell-toolbar.png)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Панель инструментов кнопки формат пока недоступна для предварительной версии записной книжки. 

---

### <a name="undo-cell-operations"></a>Отмена операций с ячейками

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Нажмите кнопку **отменить** или сочетание клавиш **CTRL + Z** , чтобы отменить последнюю операцию с ячейкой. Теперь вы можете отменить до последних 20 исторических действий с ячейками. 

   ![Синапсе отменить ячейки](./media/apache-spark-development-using-notebooks/synapse-undo-cells.png)
# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Операция отмены ячейки недоступна для предварительной версии записной книжки. 

---

### <a name="move-a-cell"></a>Перемещение ячейки

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Нажмите кнопку с многоточием (…), чтобы открыть дополнительное меню действий с ячейкой в правом углу. Затем выберите **Move cell up** (Переместить ячейку верх) или **Move cell down** (Переместить ячейку вниз), чтобы переместить текущую ячейку. 

Вы также можете использовать [сочетания клавиш в режиме команд](#shortcut-keys-under-command-mode). Нажмите клавиши **Ctrl+Alt+↑**, чтобы переместить текущую ячейку вверх. Нажмите клавиши **Ctrl+Alt+↓**, чтобы переместить текущую ячейку вниз.

   ![move-a-cell](./media/apache-spark-development-using-notebooks/synapse-move-cells.png)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Щелкните левую часть ячейки и перетащите ее в нужное расположение. 
    ![Синапсе переместить ячейки](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-drag-drop-cell.gif)

---

### <a name="delete-a-cell"></a>Удаление ячейки

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Нажмите кнопку с многоточием (…), чтобы открыть дополнительное меню действий с ячейкой справа, а затем выберите **Удалить ячейку**. 

Вы также можете использовать [сочетания клавиш в режиме команд](#shortcut-keys-under-command-mode). Нажмите **D,D**, чтобы удалить текущую ячейку.
  
   ![delete-a-cell](./media/apache-spark-development-using-notebooks/synapse-delete-cell.png)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Чтобы удалить ячейку, нажмите кнопку Удалить в правой части ячейки. 

Вы также можете использовать [сочетания клавиш в режиме команд](#shortcut-keys-under-command-mode). Нажмите клавиши **SHIFT + D** , чтобы удалить текущую ячейку. 

   ![Azure-Notebook-удалить-a-Cell](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-delete-cell.png)

---

### <a name="collapse-a-cell-input"></a>Сворачивание входных данных ячейки

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Нажмите кнопку со стрелкой в нижней части текущей ячейки, чтобы свернуть ее. Чтобы развернуть его, нажмите кнопку со стрелкой, пока ячейка свернута.

   ![collapse-cell-input](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-input.gif)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

На панели инструментов ячейки выберите **больше команд** (...) и **введите** , чтобы свернуть входные данные текущей ячейки. Чтобы развернуть его, выберите **входные данные скрыты** , пока ячейка свернута.

   ![Azure-Notebook-сворачивать-Cell-input](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-input.gif)

---

### <a name="collapse-a-cell-output"></a>Сворачивание выходных данных ячейки

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Нажмите кнопку **Свернуть выходные** данные в левом верхнем углу текущей ячейки, чтобы свернуть ее. Чтобы развернуть его, установите флажок **Показывать выходные данные ячейки** , когда выводится свернутая ячейка.

   ![collapse-cell-output](./media/apache-spark-development-using-notebooks/synapse-collapse-cell-output.gif)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Чтобы свернуть выходные данные текущей ячейки, нажмите кнопку More (...) ( **другие команды** ) на панели инструментов ячейки и **выходные данные** . Чтобы развернуть его, выберите ту же кнопку, когда выходные данные ячейки скрыты.

   ![Azure-Notebook-свернуть-ячейка-вывод](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-collapse-cell-output.gif)


---

## <a name="run-notebooks"></a>Запуск записных книжек

Вы можете выполнять ячейки кода в записной книжке по отдельности или все сразу. Состояние и ход выполнения каждой ячейки представлены в записной книжке.

### <a name="run-a-cell"></a>Выполнение ячейки

Есть несколько способов запустить код в ячейке.

1. Наведите указатель мыши на необходимую ячейку и нажмите кнопку **Выполнить ячейку** или нажмите клавиши **CTRL+ВВОД**.

   ![run-cell-1](./media/apache-spark-development-using-notebooks/synapse-run-cell.png)
  
2. Используйте [сочетания клавиш в режиме команд](#shortcut-keys-under-command-mode). Нажмите клавиши **SHIFT+ВВОД**, чтобы выполнить текущую ячейку и выбрать ячейку ниже. Нажмите клавиши **ALT+ВВОД**, чтобы выполнить текущую ячейку и вставить новую ячейку ниже.

---

### <a name="run-all-cells"></a>Выполнение всех ячеек
Нажмите кнопку **запустить все** , чтобы выполнить последовательное выполнение всех ячеек в текущей записной книжке.

   ![run-all-cells](./media/apache-spark-development-using-notebooks/synapse-run-all.png)


### <a name="run-all-cells-above-or-below"></a>Выполнение всех ячеек выше или ниже

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Чтобы открыть дополнительное меню действий с ячейками справа, нажмите кнопку с многоточием ( **…** ). Затем выберите **Run cells above** (Выполнить ячейки выше), чтобы последовательно запустить все ячейки выше текущей. Выберите **Run cells below** (Выполнить ячейки ниже), чтобы последовательно запустить все ячейки ниже текущей.

   ![run-cells-above-or-below](./media/apache-spark-development-using-notebooks/synapse-run-cells-above-or-below.png)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Разверните раскрывающийся список на панели **запустить все** , затем выберите **выполнить ячейки выше** , чтобы выполнить все ячейки, расположенные выше текущей последовательности. Выберите **Run cells below** (Выполнить ячейки ниже), чтобы последовательно запустить все ячейки ниже текущей.

   ![Azure-Notebook-Run-Cells-выше или ниже](./media/apache-spark-development-using-notebooks/synapse-aznb-run-cells-above-or-below.png)

---

### <a name="cancel-all-running-cells"></a>Отменить все работающие ячейки

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)
Нажмите кнопку **Отмена всех** , чтобы отменить выполнение ячеек или ячеек, ожидающих в очереди. 
   ![Отмена-все ячейки](./media/apache-spark-development-using-notebooks/synapse-cancel-all.png) 

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Нажмите кнопку **Отмена всех** , чтобы отменить выполнение ячеек или ячеек, ожидающих в очереди. 
   ![Azure-Notebook-Cancel-Reall-Cells](./media/apache-spark-development-using-notebooks/synapse-aznb-cancel-all.png) 

---



### <a name="notebook-reference"></a>Ссылка на записную книжку

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Не поддерживается.

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Вы можете использовать ```%run <notebook path>``` команду Magic для ссылки на другую записную книжку в контексте текущей записной книжки. Все переменные, определенные в справочной записной книжке, доступны в текущей записной книжке. ```%run``` Команда Magic поддерживает вложенные вызовы, но не поддерживает рекурсивные вызовы. Если глубина инструкции больше пяти, вы получите исключение. ```%run``` в настоящее время команда поддерживается только для передачи пути к записной книжке в качестве параметра. 

Например, ``` %run /path/notebookA ```.

> [!NOTE]
> Ссылка на записную книжку не поддерживается в конвейере синапсе.
>
>

---


### <a name="cell-status-indicator"></a>Индикатор состояния ячейки

Пошаговое состояние выполнения ячейки отображается под ячейкой, чтобы можно было просматривать ее текущий ход выполнения. После завершения выполнения ячейки отображается сводная информация об итогах выполнения с общей длительностью и временем окончания и сохраняется там для дальнейшего использования.

![cell-status](./media/apache-spark-development-using-notebooks/synapse-cell-status.png)

### <a name="spark-progress-indicator"></a>Индикатор хода выполнения Spark

Записная книжка Azure Synapse Studio основана исключительно на Spark. Ячейки кода выполняются на бессерверном пуле Apache Spark удаленно. Индикатор хода выполнения задания Spark снабжен индикатором хода выполнения в реальном времени, который поможет вам понять состояние хода выполнения задания.
Количество задач для каждого задания или этапа помогает определить параллельный уровень задания Spark. Вы также можете глубже перейти к пользовательскому интерфейсу Spark определенного задания (или этапа), выбрав ссылку на имя задания (или этапа).


![spark-progress-indicator](./media/apache-spark-development-using-notebooks/synapse-spark-progress-indicator.png)

### <a name="spark-session-config"></a>Конфигурация сеанса Spark

Вы можете указать время ожидания, число и размер исполнителей, которые будут переданы текущему сеансу Spark в области **Настройка сеанса**. Перезапустите сеанс Spark, чтобы изменения конфигурации вступили в силу. Все кэшированные переменные записной книжки очищаются.

[![Управление сеансами](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png)](./media/apache-spark-development-using-notebooks/synapse-azure-notebook-spark-session-management.png#lightbox)

#### <a name="spark-session-config-magic-command"></a>Команда Magic для настройки сеанса Spark
Параметры сеанса Spark можно также указать с помощью команды Magic **%% Configure**. Чтобы параметры вступили в действие, необходимо перезапустить сеанс Spark. Мы рекомендуем запустить **%% Configure** в начале записной книжки. Ниже приведен пример, см. https://github.com/cloudera/livy#request-body полный список допустимых параметров. 

```
%%configure -f
{
    to config the session.
    "driverMemory":"2g",
    "driverCores":3,
    "executorMemory":"2g",
    "executorCores":2,
    "jars":["myjar1.jar","myjar.jar"],
    "conf":{
        "spark.driver.maxResultSize":"10g"
    }
}
```
> [!NOTE]
> Команда Magic для настройки сеанса Spark не поддерживается в конвейере синапсе.
>
>

## <a name="bring-data-to-a-notebook"></a>Перенесение данных в записную книжку

Вы можете загрузить данные из хранилища BLOB-объектов Azure, Azure Data Lake Store Gen 2 и пула SQL, как показано в приведенных ниже примерах кода.

### <a name="read-a-csv-from-azure-data-lake-store-gen2-as-a-spark-dataframe"></a>Считывание CSV из Azure Data Lake Store Gen2 в качестве Spark DataFrame

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import *
account_name = "Your account name"
container_name = "Your container name"
relative_path = "Your path"
adls_path = 'abfss://%s@%s.dfs.core.windows.net/%s' % (container_name, account_name, relative_path)

df1 = spark.read.option('header', 'true') \
                .option('delimiter', ',') \
                .csv(adls_path + '/Testfile.csv')

```

#### <a name="read-a-csv-from-azure-blob-storage-as-a-spark-dataframe"></a>Считывание CSV из хранилища BLOB-объектов Azure в качестве Spark DataFrame

```python

from pyspark.sql import SparkSession

# Azure storage access info
blob_account_name = 'Your account name' # replace with your blob name
blob_container_name = 'Your container name' # replace with your container name
blob_relative_path = 'Your path' # replace with your relative folder path
linked_service_name = 'Your linked service name' # replace with your linked service name

blob_sas_token = mssparkutils.credentials.getConnectionStringOrCreds(linked_service_name)

# Allow SPARK to access from Blob remotely

wasb_path = 'wasbs://%s@%s.blob.core.windows.net/%s' % (blob_container_name, blob_account_name, blob_relative_path)

spark.conf.set('fs.azure.sas.%s.%s.blob.core.windows.net' % (blob_container_name, blob_account_name), blob_sas_token)
print('Remote blob path: ' + wasb_path)

df = spark.read.option("header", "true") \
            .option("delimiter","|") \
            .schema(schema) \
            .csv(wasbs_path)
```

### <a name="read-data-from-the-primary-storage-account"></a>Считывание данных из основной учетной записи хранения

Доступ к данным можно получить в основной учетной записи хранения напрямую. Нет необходимости предоставлять секретные ключи. В Data Explorer щелкните файл правой кнопкой мыши и выберите **Создать записную книжку**, чтобы просмотреть новую записную книжку с автоматически созданным средством извлечения данных.

![data-to-cell](./media/apache-spark-development-using-notebooks/synapse-data-to-cell.png)

## <a name="save-notebooks"></a>Сохранение записных книжек

Вы можете сохранить одну или все записные книжки в рабочей области.

1. Чтобы сохранить изменения, сделанные в одной записной книжке, нажмите кнопку **Опубликовать** на панели команд записной книжки.

   ![publish-notebook](./media/apache-spark-development-using-notebooks/synapse-publish-notebook.png)

2. Чтобы сохранить все записные книжки в рабочей области, нажмите кнопку **Опубликовать все** на панели команд рабочей области. 

   ![publish-all](./media/apache-spark-development-using-notebooks/synapse-publish-all.png)

В свойствах записной книжки вы можете указать, следует ли при сохранении включать выходные данные ячейки.

   ![notebook-properties](./media/apache-spark-development-using-notebooks/synapse-notebook-properties.png)

## <a name="magic-commands"></a>Магические команды
Вы можете использовать привычные команды Magic Jupyter в записных книжках Azure синапсе Studio. Проверьте следующий список как текущие доступные волшебные команды. Расскажите нам [о вариантах использования на GitHub](https://github.com/MicrosoftDocs/azure-docs/issues/new) , чтобы мы могли продолжить создавать более волшебные команды в соответствии с вашими потребностями.

> [!NOTE]
> В конвейере синапсе поддерживаются только следующие команды Magic:%% pyspark,%% Spark,%% CSharp,%% SQL. 
>
>

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Доступные магические команды строки: [%lsmagic](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [%time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%timeit](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit).

Доступно магическых ячеек: [%% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%, запись](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%% WriteFile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%% SQL](#use-multiple-languages), [%% pyspark](#use-multiple-languages), [%% Spark](#use-multiple-languages), [%% CSharp](#use-multiple-languages),[%% Configure](#spark-session-config-magic-command)



# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Доступные магическые строки: [% лсмагик](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-lsmagic), [% времени](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [% History](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-history), " [% Run](#notebook-reference)", [% Load](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-load)

Доступно магическых ячеек: [%% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-time), [%% time](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit), [%%, запись](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-capture), [%% WriteFile](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-writefile), [%% SQL](#use-multiple-languages), [%% pyspark](#use-multiple-languages), [%% Spark](#use-multiple-languages), [%% CSharp](#use-multiple-languages), [%% HTML](https://ipython.readthedocs.io/en/stable/interactive/magics.html#cellmagic-html), [%% Configure](#spark-session-config-magic-command)

--- 

## <a name="integrate-a-notebook"></a>Интеграция записной книжки

### <a name="add-a-notebook-to-a-pipeline"></a>Добавление записной книжки в конвейер

Нажмите кнопку **Добавить в конвейер** в правом верхнем углу, чтобы добавить записную книжку в существующий конвейер или создать новый конвейер.

![Добавить записную книжку в конвейер](./media/apache-spark-development-using-notebooks/add-to-pipeline.png)

### <a name="designate-a-parameters-cell"></a>Назначение ячейки параметров

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

Чтобы параметризовать записную книжку, щелкните многоточие (...), чтобы открыть меню "действия" в правом углу. Затем выберите **параметр переключить ячейку параметра** , чтобы назначить ячейку в качестве ячейки параметров.

![Toggle-параметр](./media/apache-spark-development-using-notebooks/toggle-parameter-cell.png)

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

Чтобы параметризовать записную книжку, нажмите кнопку с многоточием (...), чтобы получить доступ к **командам more** на панели инструментов ячейки. Затем выберите **параметр переключить ячейку параметра** , чтобы назначить ячейку в качестве ячейки параметров.

![Azure-Notebook-переключатель-параметр](./media/apache-spark-development-using-notebooks/azure-notebook-toggle-parameter-cell.png)

---

Фабрика данных Azure ищет ячейку Parameters и обрабатывает ее как значения по умолчанию для параметров, передаваемых во время выполнения. Подсистема выполнения добавит новую ячейку под ячейкой параметров с входными параметрами, чтобы перезаписать значения по умолчанию. Если ячейка параметров не назначена, то вставленная ячейка вставляется в верхнюю часть записной книжки.


### <a name="assign-parameters-values-from-a-pipeline"></a>Назначение значений параметров из конвейера

Создав записную книжку с параметрами, вы можете выполнить ее из конвейера с помощью действия записной книжки Azure синапсе. После добавления действия на холст конвейера можно будет задать значения параметров в разделе **базовые параметры** на вкладке **Параметры** . 

![Назначение параметра](./media/apache-spark-development-using-notebooks/assign-parameter.png)

При назначении значений параметров можно использовать [язык выражений конвейера](../../data-factory/control-flow-expression-language-functions.md) или [системные переменные](../../data-factory/control-flow-system-variables.md).



## <a name="shortcut-keys"></a>Сочетания клавиш

Аналогично Jupyter Notebook в записных книжках Azure Synapse Studio есть модальный пользовательский интерфейс. Клавиатура выполняет разные действия в зависимости от того, в каком режиме находится ячейка записной книжки. Записные книжки Synapse Studio поддерживают для данной ячейки кода следующие два режима: режим команд и режим правки.

1. Ячейка находится в режиме команд, когда нет текстового курсора для ввода текста. Когда ячейка находится в режиме команд, вы можете редактировать записную книжку целиком, но не вводить ее в отдельные ячейки. Введите режим команд, нажав `ESC` или выполнив команду мыши вне области редактора ячейки.

   ![command-mode](./media/apache-spark-development-using-notebooks/synapse-command-mode-2.png)

2. Режим правки обозначается текстовым курсором для ввода текста в области редактора. Если ячейка находится в режиме редактирования, ее можно ввести в ячейку. Войдите в режим редактирования, нажав `Enter` или выполнив команду мыши в области редактора ячейки.
   
   ![edit-mode](./media/apache-spark-development-using-notebooks/synapse-edit-mode-2.png)

### <a name="shortcut-keys-under-command-mode"></a>Сочетания клавиш в режиме команд

# <a name="classical-notebook"></a>[Классическая Записная книжка](#tab/classical)

С помощью приведенных ниже сочетаний клавиш вы можете легче перемещаться и выполнять код в записных книжках Azure Synapse.

| Действие |Сочетания клавиш для записной книжки Synapse Studio  |
|--|--|
|Запуск текущей ячейки и выбор ниже | SHIFT+ВВОД |
|Запуск текущей ячейки и вставка ниже | ALT+ВВОД |
|Выбор ячейки выше| Вверх |
|Выбор ячейки ниже| Вниз |
|Вставка ячейки выше| Объект |
|Вставка ячейки ниже| B |
|Расширение выбранных ячеек выше| Shift+Up |
|Расширение выбранных ячеек ниже| Shift+ВНИЗ|
|Перемещение ячейки вверх| CTRL+ALT+↑ |
|Перемещение ячейки вниз| CTRL+ALT+↓ |
|Удаление выбранных ячеек| D, D |
|Переключение в режим правки| Введите: |

# <a name="preview-notebook"></a>[Предварительный просмотр записной книжки](#tab/preview)

| Действие |Сочетания клавиш для записной книжки Synapse Studio  |
|--|--|
|Запуск текущей ячейки и выбор ниже | SHIFT+ВВОД |
|Запуск текущей ячейки и вставка ниже | ALT+ВВОД |
|Выполнить текущую ячейку| Ctrl+ВВОД |
|Выбор ячейки выше| Вверх |
|Выбор ячейки ниже| Вниз |
|Выбрать предыдущую ячейку| K |
|Выбрать следующую ячейку| J |
|Вставка ячейки выше| Объект |
|Вставка ячейки ниже| B |
|Удаление выбранных ячеек| Shift + D |
|Переключение в режим правки| Введите: |

---

### <a name="shortcut-keys-under-edit-mode"></a>Сочетания клавиш в режиме правки


С помощью приведенных ниже сочетаний клавиш вы можете легче перемещаться и выполнять код в записных книжках Azure Synapse в режиме правки.

| Действие |Сочетания клавиш для записной книжки Synapse Studio  |
|--|--|
|Перемещение курсора вверх | Вверх |
|Перемещение курсора вниз|Вниз|
|Отмена|Ctrl+Z|
|Повторить|CTRL+Y|
|Добавление и удаление комментариев|Ctrl+/|
|Удаление слова перед|Ctrl+Backspace|
|Удаление слова после|Ctrl+Delete|
|Переход к началу ячейки|Ctrl+Home|
|Передох к концу ячейки |Ctrl+End|
|Переход на одно слово влево|Ctrl+Left|
|Переход на одно слово вправо|Ctrl+Right|
|Выбрать все|Ctrl+A|
|Отступ| Ctrl +]|
|Понижение уровня|Ctrl+[|
|Переключение в режим команд| ESC |

---

## <a name="next-steps"></a>Дальнейшие действия
- [Ознакомьтесь с примерами записных книжек Synapse](https://github.com/Azure-Samples/Synapse/tree/master/Notebooks)
- [Краткое руководство. Создание пула Apache Spark в Azure Synapse Analytics с помощью веб-инструментов](../quickstart-apache-spark-notebook.md)
- [Что такое Apache Spark в Azure Synapse Analytics](apache-spark-overview.md)
- [Использование .NET для Apache Spark с помощью Azure Synapse Analytics](spark-dotnet.md)
- [Документация по .NET для Apache Spark](/dotnet/spark)
- [Azure Synapse Analytics](../index.yml)
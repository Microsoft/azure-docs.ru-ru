---
title: Руководство по Spark & Hive Tools для VSCode (приложение Spark)
description: Руководство. Разработка приложений Spark на языке Python и их отправка в бессерверный пул Apache Spark с помощью Spark & Hive Tools.
services: synapse-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 09/03/2020
ms.openlocfilehash: e5d335ee14709ec330405419f5be8ac5fbd6ce75
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98943770"
---
# <a name="tutorial-create-an-apache-spark-applications-with-vscode-using-a-synapse-workspace"></a>Руководство по созданию приложений Apache Spark с помощью VSCode и рабочей области Synapse

Узнайте, как использовать Apache Spark & Hive Tools для Visual Studio Code. Эти средства используются для создания и отправки пакетных заданий Hive, интерактивных запросов Apache Hive и сценариев PySpark для Apache Spark. Сначала мы покажем, как установить Spark & Hive Tools в Visual Studio Code. Затем мы рассмотрим, как отправлять задания в Spark & Hive Tools.

Spark & Hive Tools можно установить на платформах, поддерживаемых Visual Studio Code. Ниже приведены предварительные требования для разных платформ.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения действий, описанных в этой статье, необходимо следующее:

- Бессерверный пул Apache Spark. Сведения о создании бессерверного пула Apache Spark см. в статье [Создание пула Apache Spark с помощью портала Azure](../../synapse-analytics/quickstart-create-apache-spark-pool-portal.md).
- [Visual Studio Code](https://code.visualstudio.com/).
- [Mono](https://www.mono-project.com/docs/getting-started/install/). Mono требуется только для Linux и macOS.
- [Интерактивная среда PySpark для Visual Studio Code](../../hdinsight/set-up-pyspark-interactive-environment.md).
- Локальный каталог. В этой статье используется **C:\HD\Synaseexample**.

## <a name="install-spark--hive-tools"></a>Установка Spark & Hive Tools

После выполнения предварительных требований можно установить Spark & Hive Tools для Visual Studio Code в соответствии с приведенными ниже инструкциями.

1. Откройте Visual Studio Code.

2. В строке меню выберите **Вид** > **Расширения**.

3. В поле поиска введите **Spark & Hive**.

4. Выберите **Spark & Hive Tools** в результатах поиска и щелкните **Установить**.

     ![Установка расширения Python Spark & Hive для Visual Studio Code](./media/vscode-tool-synapse/install-hdInsight-plugin.png)

5. При необходимости выберите **Перезагрузить**.

## <a name="open-a-work-folder"></a>Открытие рабочей папки

Чтобы открыть рабочую папку и создать файл в Visual Studio Code, выполните следующие действия.

1. В строке меню выберите **Файл** > **Открыть папку...**  > **C:\HD\Synaseexample**, а затем нажмите кнопку **Выбрать папку**. Папка отображается в представлении **проводника** слева.

2. В представлении **проводника** выберите папку **Synaseexample**, а затем — значок **Новый файл** рядом с рабочей папкой.

     ![Значок "Новый файл" в Visual Studio Code](./media/vscode-tool-synapse/visual-studio-code-new-file.png)

3. Присвойте новому файлу имя с расширением `.py` (скрипт Spark). В этом примере используется **HelloWorld.py**.

## <a name="connect-to-your-spark-pools"></a>Подключение к пулам Spark

Войдите в подписку Azure, чтобы подключиться к пулам Spark.

### <a name="sign-in-to-your-azure-subscription"></a>Войдите в свою подписку Azure.

Для подключения к Azure выполните следующие действия.

1. В строке меню выберите **Вид** > **Палитра команд...** и введите **Azure: Вход**:

     ![Вход в Spark & Hive Tools для Visual Studio Code](./media/vscode-tool-synapse/hdinsight-for-vscode-extension-login.png)

2. Следуйте инструкциям по входу, чтобы войти в Azure. После подключения имя учетной записи Azure будет отображаться в строке состояния внизу окна Visual Studio Code.

## <a name="set-the-default-spark-pool"></a>Задание стандартного пула Spark

1. Повторно откройте папку **Synaseexample**, которая упоминалась [ранее](#open-a-work-folder), если она была закрыта.  

2. Выберите файл **HelloWorld.py**, созданный [ранее](#open-a-work-folder). Он откроется в редакторе скриптов.

3. Щелкните редактор скриптов правой кнопкой мыши и выберите пункт **Synapse: Set default Spark pool** (Synapse. Задать пул стандртный пул Spark).  

4. [Подключитесь](#connect-to-your-spark-pools) к учетной записи Azure, если вы еще этого не сделали.

5. Выберите пул Spark в качестве стандартного для текущего файла скрипта. Средства автоматически обновляют файл конфигурации **.VSCode\settings.json**.

     ![Настройка конфигурации кластера по умолчанию](./media/vscode-tool-synapse/set-default-cluster-configuration.png)

## <a name="submit-interactive-synapse-pyspark-queries-to-spark-pool"></a>Отправка интерактивных запросов Synapse PySpark в пул Spark

Пользователи могут работать с Synapse PySpark в пуле Spark в интерактивном режиме, как описано ниже.

### <a name="using-the-synapse-pyspark-interactive-command-in-py-file"></a>Использование интерактивной команды Synapse PySpark в файле PY
Чтобы отправить запросы с помощью интерактивной команды PySpark, выполните следующие действия.

1. Повторно откройте папку **Synaseexample**, которая упоминалась [ранее](#open-a-work-folder), если она была закрыта.  

2. Создайте новый файл **HelloWorld.py**, выполнив [приведенные выше инструкции](#open-a-work-folder).

3. Скопируйте и вставьте в файл скрипта следующий код:

```python
import sys
from operator import add
from pyspark.sql import SparkSession, Row
 
spark = SparkSession\
 .builder\
 .appName("PythonWordCount")\
 .getOrCreate()
 
data = [Row(col1='pyspark and spark', col2=1), Row(col1='pyspark', col2=2), Row(col1='spark vs hadoop', col2=2), Row(col1='spark', col2=2), Row(col1='hadoop', col2=2)]
df = spark.createDataFrame(data)
lines = df.rdd.map(lambda r: r[0])
 
counters = lines.flatMap(lambda x: x.split(' ')) \
 .map(lambda x: (x, 1)) \
 .reduceByKey(add)
 
output = counters.collect()
sortedCollection = sorted(output, key = lambda r: r[1], reverse = True)
 
for (word, count) in sortedCollection:
 print("%s: %i" % (word, count))
```

4. В правом нижнем углу окна появится запрос на установку ядра PySpark или Synapse Pyspark. Можно нажать кнопку **Установить**, чтобы перейти к установке PySpark или Synapse Pyspark, или **Пропустить**, чтобы пропустить этот шаг.

     ![Установка ядра PySpark](./media/vscode-tool-synapse/install-the-pyspark-kernel.png)

5. Если вы хотите установить его позже, последовательно выберите **Файл** > **Настройки** > **Параметры**, а затем снимите флажок **Hdinsight: Enable Skip Pyspark Installation** (Разрешить пропуск установки Pyspark). 
    
     ![включение функции пропуска установки Pyspark](./media/vscode-tool-synapse/enable-skip-pyspark-installation.png)

6. Если на шаге 4 установка PySpark или Synapse Pyspark прошла успешно, в правом нижнем углу окна появится окно сообщения PySpark/Synapse Pyspark installed successfully (Расширение PySpark или Synapse Pyspark успешно установлено). Нажмите кнопку **Перезагрузить**, чтобы перезагрузить окно.

     ![расширение PySpark успешно установлено](./media/vscode-tool-synapse/pyspark-kernel-installed-successfully.png)

7. В строке меню последовательно выберите **Вид** > **Палитра команд...** или воспользуйтесь сочетанием клавиш **SHIFT+CTRL+P** и введите **Python: Select Interpreter to start Jupyter Server** (Выбрать интерпретатор для запуска сервера Jupyter).

     ![выбор интерпретатора для запуска сервера Jupyter](./media/vscode-tool-synapse/select-interpreter-to-start-jupyter-server.png)

8. Выберите параметр python ниже.

     ![выбор параметра ниже](./media/vscode-tool-synapse/choose-the-below-option.png)
    
9. В строке меню последовательно выберите **Вид** > **Палитра команд...** или воспользуйтесь сочетанием клавиш **SHIFT+CTRL+P** и введите **Developer: Reload Window** (Разработчик: перезагрузить окно).

     ![перезагрузка окна](./media/vscode-tool-synapse/reload-window.png)

10. [Подключитесь](#connect-to-your-spark-pools) к учетной записи Azure, если вы еще этого не сделали.

11. Выберите весь код, щелкните редактор скриптов правой кнопкой мыши и выберите пункт **Synapse: Pyspark Interactive** (Synapse. Интерактивный запрос PySpark), чтобы отправить запрос. 

     ![контекстное меню интерактивной среды pyspark](./media/vscode-tool-synapse/pyspark-interactive-right-click.png)

12. Выберите пул Spark, если вы не указали стандартный. Через несколько секунд на новой вкладке отображаются результаты **интерактивного окна Python**. Щелкните PySpark, чтобы переключить ядро на **Synapse PySpark**, затем снова отправьте выбранный код, и он будет выполнен успешно. Эти средства также позволяют отправить блок кода вместо целого файла скрипта с помощью контекстного меню.

     ![интерактивный](./media/vscode-tool-synapse/pyspark-interactive-python-interactive-window.png)

### <a name="perform-interactive-query-in-py-file-using-a--comment"></a>Выполнение интерактивного запроса в файле PY с помощью комментария #%%

1. Добавьте **#%%** перед кодом, чтобы работать с ним как с записной книжкой.

     ![добавление #%%](./media/vscode-tool-synapse/run-cell.png)

2. Щелкните **Выполнить ячейку**. Через несколько секунд на новой вкладке отображаются результаты интерактивного окна Python. Щелкните PySpark, чтобы переключить ядро на **Synapse PySpark**, затем щелкните **Выполнить ячейку** снова, и код будет выполнен успешно. 

     ![результаты выполнения ячейки](./media/vscode-tool-synapse/run-cell-get-results.png)

## <a name="leverage-ipynb-support-from-python-extension"></a>Использование поддержки IPYNB из расширения Python

1. Записную книжку Jupyter Notebook можно создать с помощью команды из палитры команд или путем создания нового IPYNB-файла в рабочей области. Дополнительные сведения см. в статье [Работа с записными книжками Jupyter Notebook в Visual Studio Code](https://code.visualstudio.com/docs/python/jupyter-support).

2. Нажмите кнопку **Выполнить ячейку**, выполните инструкции на экране, чтобы **задать стандартный пул Spark** (настоятельно рекомендуется перед каждым открытием записной книжки устанавливать стандартный кластер или пул), а затем **перезагрузите** окно.

     ![задание стандартного пула Spark и перезагрузка](./media/vscode-tool-synapse/set-the-default-spark-pool-and-reload.png)

3. Щелкните PySpark, чтобы переключить ядро на **Synapse Pyspark**, а затем щелкните **Выполнить ячейку**, после чего отобразится результат.

     ![результаты выполнения IPYNB](./media/vscode-tool-synapse/run-ipynb-file-results.png)


> [!NOTE]
>
>1. В этом расширении не поддерживается ms-python версии 2020.5.78807 и более новой. Это [известная проблема](#known-issues).
>  
>2. Рекомендуется переключиться на ядро Synapse Pyspark для отключения автоматической настройки параметров на портале Azure. В противном случае для активации кластера и установки ядра synapse при первом использовании может потребоваться много времени. 
>
>    ![автоматическая настройка параметров](./media/vscode-tool-synapse/auto-settings.png)

## <a name="submit-pyspark-batch-job-to-spark-pool"></a>Отправка пакетного задания PySpark в пул Spark

1. Повторно откройте папку **Synaseexample**, которая упоминалась [ранее](#open-a-work-folder), если она была закрыта.  

2. Создайте новый файл **BatchFile.py**, выполнив [приведенные выше инструкции](#open-a-work-folder).

3. Скопируйте и вставьте в файл скрипта следующий код:

    ```python
    from __future__ import print_function
    import sys
    from operator import add
    from pyspark.sql import SparkSession
    if __name__ == "__main__":
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
    
        lines = spark.read.text('/HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv').rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' '))\
                    .map(lambda x: (x, 1))\
                    .reduceByKey(add)
        output = counts.collect()
        for (word, count) in output:
            print("%s: %i" % (word, count))
        spark.stop()
    ```

4. [Подключитесь](#connect-to-your-spark-pools) к учетной записи Azure, если вы еще этого не сделали.

5. Щелкните редактор скриптов правой кнопкой мыши и выберите пункт **Synapse: Пакет PySpark**.

6. Выберите пул Spark, в который необходимо отправить задание PySpark.

     ![Вывод результатов задания Python](./media/vscode-tool-synapse/submit-pythonjob-result.png)

После отправки пакетного задания в пул Spark журналы отправки отображаются в окне **вывода** в Visual Studio Code. Также отображается URL-адрес пользовательского интерфейса **Spark** и **приложения для задания Spark**. Вы можете открыть этот URL-адрес в браузере для отслеживания состояния задания.

## <a name="access-and-manage-synapse-workspace"></a>Доступ к рабочей области Synapse и управление ею

В Azure Explorer можно выполнять различные операции в Spark & Hive Tools для VSCode. В Azure Explorer.

![изображение Azure](./media/vscode-tool-synapse/azure.png)

### <a name="launch-workspace"></a>Запуск рабочей области

1. В Azure Explorer перейдите к узлу **SYNAPSE** и разверните его, чтобы отобразить список подписок на Synapse.

     ![Обозреватель с выделенным пунктом Synapse](./media/vscode-tool-synapse/synapse-explorer.png)

2. Щелкните подписку рабочей области Synapse, разверните ее и отобразите список рабочих областей.

3. Щелкните правой кнопкой мыши рабочую область, а затем выберите **View Apache Spark applications** (Просмотр приложений Apache Spark). Откроется страница приложения Apache Spark на веб-сайте Synapse Studio.

     ![контекстное меню в рабочей области](./media/vscode-tool-synapse/right-click-on-workspace.png)

     ![приложение Synapse Studio](./media/vscode-tool-synapse/synapse-studio-application.png)

4. Разверните рабочую область. В ней отобразятся элементы **Хранилище по умолчанию** и **Пулы Spark**.

5. Щелкните правой кнопкой мыши **Хранилище по умолчанию**. Отобразятся пункты **Копировать полный путь** и **Открыть в Synapse Studio**. 

     ![контекстное меню для хранилища по умолчанию](./media/vscode-tool-synapse/right-click-on-default-storage.png)

     - Щелкните **Копировать полный путь**. Теперь скопированный основной URL-адрес учетной записи ADLS 2-го поколения можно вставить в любом месте.

     - Щелкните **Открыть в Synapse Studio**. Основная учетная запись хранения будет открыта в Synapse Studio.

     ![стандартное хранилище в Synapse Studio](./media/vscode-tool-synapse/default-storage-in-synapse-studio.png)

6. Разверните узел **Хранилище по умолчанию**. Отобразится основная учетная запись хранения.

7. Разверните узел **Пулы Spark**. Отобразятся все пулы Spark в рабочей области.

     ![развертывание списка пулов в хранилище](./media/vscode-tool-synapse/expand-storage-pool.png)


## <a name="known-issues"></a>Известные проблемы

### <a name="ms-python-2020578807-version-is-not-supported-on-this-extention"></a>В этом расширении не поддерживается ms-python версии 2020.5.78807 и более новой 

Сбой при подключении к записной книжке Jupyter Notebook является известной проблемой для Python версии 2020.5.78807 и более новых. Во избежание этой проблемы пользователям рекомендуется использовать ms-python версии **[2020.4.76186](https://github.com/microsoft/vscode-python/releases/download/2020.4.76186/ms-python-release.vsix)** .

![Известные проблемы](./media/vscode-tool-synapse/known-issue.png)


## <a name="next-steps"></a>Дальнейшие действия

- [Azure Synapse Analytics](../overview-what-is.md)
- [Создание пула Apache Spark для рабочей области Azure Synapse Analytics](../../synapse-analytics/quickstart-create-apache-spark-pool-studio.md)

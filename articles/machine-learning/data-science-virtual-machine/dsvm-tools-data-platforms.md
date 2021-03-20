---
title: Поддерживаемые платформы данных
titleSuffix: Azure Data Science Virtual Machine
description: Сведения о поддерживаемых платформах данных и средствах для виртуальной машины Azure для обработки и анализа данных.
keywords: средства анализа и обработки данных, виртуальная машина для анализа и обработки данных, средства для анализа и обработки данных, анализ и обработка данных Linux
services: machine-learning
ms.service: data-science-vm
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 12/12/2019
ms.openlocfilehash: 927e945a0d045abcd1caa2951dbd484224b2f425
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100519547"
---
# <a name="data-platforms-supported-on-the-data-science-virtual-machine"></a>Языки, поддерживаемые Виртуальной машиной для обработки и анализа данных

С помощью виртуальной машины для обработки и анализа данных (DSVM) можно создать аналитику для широкого спектра платформ данных. Кроме интерфейсов для платформ удаленных данных DSVM предоставляет локальный экземпляр для быстрой разработки и создания прототипов.

В DSVM поддерживаются следующие инструменты платформы данных.

## <a name="sql-server-developer-edition"></a>SQL Server Developer Edition

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Экземпляр локальной реляционной базы данных      |
| Поддерживаемые выпуски DSVM      | Windows 2016: SQL Server 2017, Windows 2019: SQL Server 2019      |
| Распространенные способы применения      | <ul><li>Быстрая локальная разработка приложений с меньшим набором данных</li><li>Запуск R внутри базы данных</li></ul> |
| Ссылки на примеры      | <ul><li>Небольшой пример набора данных о городе Нью-Йорка загружается в базу данных SQL:<br/>  `nyctaxi`</li><li>Образец Jupyter, отображающий Microsoft Machine Learning Server и аналитическую аналитику в базе данных, можно найти по адресу:<br/> `~notebooks/SQL_R_Services_End_to_End_Tutorial.ipynb`</li></ul> |
| Связанные средства на DSVM       | <ul><li>SQL Server Management Studio</li><li>Драйверы ODBC и JDBC</li><li>pyodbc, RODBC</li><li>Apache Drill</li></ul> |

> [!NOTE]
> SQL Server Developer Edition можно использовать только в целях разработки и тестирования. Для запуска в рабочей среде вам потребуется лицензия или одна из виртуальных машин SQL Server.


### <a name="setup"></a>Настройка

Сервер базы данных уже предварительно настроен, а службы Windows, связанные с SQL Server (например `SQL Server (MSSQLSERVER)` ,), настроены для автоматического запуска. Только ручной шаг включает в себя включение аналитической аналитики базы данных с помощью Microsoft Machine Learning Server. Вы можете включить аналитику, выполнив следующую команду в качестве однократного действия в SQL Server Management Studio (SSMS). Выполните эту команду после входа в систему от имени администратора компьютера, откройте новый запрос в SSMS и убедитесь, что выбранная база данных `master` :

```sql
CREATE LOGIN [%COMPUTERNAME%\SQLRUserGroup] FROM WINDOWS 
```

(Замените% COMPUTERNAME% на имя виртуальной машины.)

Для запуска SQL Server Management Studio можно выполнить поиск по слову "SQL Server Management Studio" в списке программ или воспользоваться поиском Windows для поиска и запуска. При появлении запроса на ввод учетных данных выберите **Проверка подлинности Windows** и используйте имя компьютера или ```localhost``` в поле **SQL Server имя** .

### <a name="how-to-use-and-run-it"></a>Использование и запуск

По умолчанию сервер базы данных с экземпляром базы данных по умолчанию запускается автоматически. Чтобы получить доступ к базе данных SQL Server локально, на виртуальной машине можно использовать такие средства, как SQL Server Management Studio. Учетные записи локального администратора имеют административный доступ к базе данных.

Кроме того, DSVM поставляется с драйверами ODBC и JDBC для взаимодействия с SQL Server, базами данных SQL Azure и Azure синапсе Analytics из приложений, написанных на нескольких языках, включая Python и Machine Learning Server.

### <a name="how-is-it-configured-and-installed-on-the-dsvm"></a>Как она настраивается и устанавливается на DSVM? 

 SQL Server устанавливается стандартным способом. Его можно найти в `C:\Program Files\Microsoft SQL Server`. Экземпляр Machine Learning Server в базе данных находится по адресу `C:\Program Files\Microsoft SQL Server\MSSQL13.MSSQLSERVER\R_SERVICES` . DSVM также имеет отдельный изолированный экземпляр Machine Learning Server, который устанавливается в `C:\Program Files\Microsoft\R Server\R_SERVER` . Эти два экземпляра Machine Learning Server не используют общие библиотеки.


## <a name="apache-spark-2x-standalone"></a>Экземпляр Apache Spark 2.x (изолированный)

| Категория | Значение |
| ------------- | ------------- |
| Что это такое?   | Изолированный экземпляр (один узел в процессе) популярной платформы Apache Spark; система для быстрой и крупномасштабной обработки данных и машинного обучения     |
| Поддерживаемые выпуски DSVM      | Linux     |
| Распространенные способы применения      | <ul><li>Быстрая разработка приложений Spark/PySpark на локальном компьютере с меньшим набором данных и более поздним развертыванием в крупных кластерах Spark, таких как Azure HDInsight</li><li>Тестирование контекста Spark Microsoft Machine Learning Server</li><li>Использование SparkML или библиотеки [MMLSpark](https://github.com/Azure/mmlspark) с открытым кодом Майкрософт для создания приложений ml</li></ul> |
| Ссылки на примеры      |    Пример Jupyter:<ul><li>~/Нотебукс/спаркмл/писпарк</li><li>~/Нотебукс/ммлспарк</li></ul><p>Microsoft Machine Learning Server (контекст Spark):/Дсвм/самплес/МРС/мрсспаркконтекстсампле.р</p> |
| Связанные средства на DSVM       | <ul><li>PySpark, Scala</li><li>Jupyter (ядра Spark/PySpark)</li><li>Microsoft Machine Learning Server, Spark, Sparklyr</li><li>Apache Drill</li></ul> |

### <a name="how-to-use-it"></a>Способ использования
Вы можете отправить задания Spark в командной строке, выполнив `spark-submit` команду или `pyspark` . Вы можете также создать записную книжку Jupyter с ядром Spark.

Вы можете использовать Spark из R с помощью таких библиотек, как Spark, Sparklyr и Microsoft Machine Learning Server, доступных в DSVM. См. указатели на примеры в предыдущей таблице.

### <a name="setup"></a>Настройка
Перед запуском в контексте Spark в Microsoft Machine Learning Server в Ubuntu Linux DSVM Edition необходимо выполнить однократную настройку, чтобы включить локальный одноузловой экземпляр Hadoop HDFS и Yarn. По умолчанию на виртуальной машине для обработки и анализа данных службы Hadoop установлены, но отключены. Чтобы включить их, выполните следующие команды с правами root в первый раз:

```bash
echo -e 'y\n' | ssh-keygen -t rsa -P '' -f ~hadoop/.ssh/id_rsa
cat ~hadoop/.ssh/id_rsa.pub >> ~hadoop/.ssh/authorized_keys
chmod 0600 ~hadoop/.ssh/authorized_keys
chown hadoop:hadoop ~hadoop/.ssh/id_rsa
chown hadoop:hadoop ~hadoop/.ssh/id_rsa.pub
chown hadoop:hadoop ~hadoop/.ssh/authorized_keys
systemctl start hadoop-namenode hadoop-datanode hadoop-yarn
```

Вы можете отключить службы, связанные с Hadoop, если они больше не нужны, выполнив ```systemctl stop hadoop-namenode hadoop-datanode hadoop-yarn``` .

Пример, демонстрирующий разработку и тестирование MRS в удаленном контексте Spark (который является автономным экземпляром Spark в DSVM) предоставляется и доступен в `/dsvm/samples/MRS` каталоге.


### <a name="how-is-it-configured-and-installed-on-the-dsvm"></a>Как она настраивается и устанавливается на DSVM? 
|Платформа|Размещение установки ($SPARK_HOME)|
|:--------|:--------|
|Linux   | /dsvm/tools/spark-X.X.X-bin-hadoopX.X|


Библиотеки для доступа к данным из хранилища BLOB-объектов Azure или Azure Data Lake Storage с помощью библиотек машинного обучения Microsoft MMLSpark, предустановленных в $SPARK _HOME/жарс. Эти JAR-файлы автоматически загружаются при запуске Spark. По умолчанию Spark использует данные на локальном диске. 

Чтобы экземпляр Spark в DSVM для доступа к данным, хранящимся в хранилище BLOB-объектов или Azure Data Lake Storage, необходимо создать и настроить `core-site.xml` файл на основе шаблона, найденного в $SPARK _HOME/конф/core-site.xml. Template. Кроме того, необходимо иметь соответствующие учетные данные для доступа к хранилищу BLOB-объектов и Azure Data Lake Storage. (Обратите внимание, что файлы шаблонов используют заполнители для хранилища BLOB-объектов и конфигураций Azure Data Lake Storage.)

Более подробные сведения о создании учетных данных Azure Data Lake Storage служб см. [в статье Проверка подлинности с помощью Azure Data Lake Storage 1-го поколения](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md). После ввода учетных данных для хранилища BLOB-объектов или Azure Data Lake Storage в файл core-site.xml можно ссылаться на данные, хранящиеся в этих источниках, с помощью префикса URI wasb://или adl://.
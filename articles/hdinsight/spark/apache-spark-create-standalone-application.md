---
title: Руководство по приложению Scala с Maven для Spark с IntelliJ в Azure HDInsight
description: Руководство по созданию приложения Spark на языке Scala с помощью Apache Maven в качестве системы сборки. В качестве основы используется существующий архетип Maven для Scala, предоставляемый IntelliJ IDEA.
ms.service: hdinsight
ms.topic: tutorial
ms.custom: contperf-fy21q1
ms.date: 08/21/2020
ms.openlocfilehash: 54738ebe45792bea70067383aaeeca99667f753f
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106068325"
---
# <a name="tutorial-create-a-scala-maven-application-for-apache-spark-in-hdinsight-using-intellij"></a>Руководство по Создание приложения Scala Maven для Apache Spark в HDInsight с помощью IntelliJ

Из этого руководства вы узнаете, как создать приложение Apache Spark на языке Scala в Apache Maven с помощью IntelliJ IDEA. В примере из этой статьи в качестве системы сборки используется Apache Maven, а в качестве основы используется существующий архетип Maven для Scala, предоставляемый IntelliJ IDEA.  Создание приложения Scala в IntelliJ IDEA включает в себя следующие этапы:

* использование Maven в качестве системы сборки;
* обновление файла объектной модели проектов для разрешения зависимостей модуля Spark;
* написание приложения на языке Scala;
* создание JAR-файла, который можно отправить в кластеры HDInsight Spark;
* запуск приложений с помощью Livy в кластере Spark.

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Установка подключаемого модуля Scala для IntelliJ IDEA
> * Разработка приложения Scala Maven с помощью IntelliJ.
> * Создание автономного проекта Scala

## <a name="prerequisites"></a>Предварительные требования

* Кластер Apache Spark в HDInsight. Инструкции см. в статье [Начало работы. Создание кластера Apache Spark в HDInsight на платформе Linux и выполнение интерактивных запросов с помощью SQL Spark](apache-spark-jupyter-spark-sql.md).

* [Комплект разработчика Oracle Java](https://www.azul.com/downloads/azure-only/zulu/).  В этом руководстве используется Java версии 8.0.202.

* Java IDE. В этой статье используется среда [IntelliJ IDEA версии  2018.3.4](https://www.jetbrains.com/idea/download/).

* Azure Toolkit for IntelliJ.  Дополнительные сведения см. в статье [Установка набора средств Azure для IntelliJ](/java/azure/intellij/azure-toolkit-for-intellij-create-hello-world-web-app#installation-and-sign-in).

## <a name="install-scala-plugin-for-intellij-idea"></a>Установка подключаемого модуля Scala для IntelliJ IDEA

Чтобы установить подключаемый модуль Scala, сделайте следующее:

1. Откройте IntelliJ IDEA.

2. На экране приветствия выберите **Configure** (Настройка) > **Plugins** (Подключаемые модули), чтобы открыть окно **подключаемых модулей**.

    :::image type="content" source="./media/apache-spark-create-standalone-application/enable-scala-plugin1.png" alt-text="Включение подключаемого модуля Scala в IntelliJ IDEA" border="true":::

3. Выберите **Install** (Установить) в области подключаемого модуля Scala в новом окне.  

    :::image type="content" source="./media/apache-spark-create-standalone-application/install-scala-plugin.png" alt-text="Установка подключаемого модуля Scala в IntelliJ IDEA" border="true":::

4. После успешной установки подключаемого модуля необходимо перезапустить интегрированную среду разработки.

## <a name="use-intellij-to-create-application"></a>Создание приложения с помощью IntelliJ

1. Запустите IntelliJ IDEA и выберите **Create New Project** (Создать проект), чтобы открыть окно **New Project** (Новый проект).

2. На панели слева выберите **Azure Spark/HDInsight**.

3. В главном окне выберите **Spark Project (Scala)** (Проект Spark (Scala)).

4. Из раскрывающегося списка **Build tool** (Инструмент сборки) выберите одно из следующих значений:
      * **Maven** для поддержки мастера создания проекта Scala.
      * **SBT** для управления зависимостями и создания проекта Scala.

   :::image type="content" source="./media/apache-spark-create-standalone-application/intellij-project-apache-spark.png" alt-text="Диалоговое окно нового проекта в IntelliJ" border="true":::

5. Выберите **Далее**.

6. В окне **New Project** (Новый проект) укажите следующую информацию:  

  	|  Свойство   | Описание   |  
  	| ----- | ----- |  
  	|Имя проекта| Введите имя.|  
  	|Project&nbsp;location (Расположение проекта)| Введите расположение для сохранения проекта.|
  	|Project SDK (Пакет SDK проекта)| При первом использовании IDEA это поле будет пустым.  Выберите **New...** (Создать...) и перейдите к JDK.|
  	|Версия Spark|Мастер создания интегрирует правильную версию пакетов SDK для Spark и Scala. Если используется версия кластера Spark более ранняя, чем 2.0, выберите **Spark 1.x**. В противном случае выберите **Spark 2.x**. В этом примере используется **Spark 2.3.0 (Scala 2.11.8)** .|

    :::image type="content" source="./media/apache-spark-create-standalone-application/hdi-scala-new-project.png" alt-text="Выбор пакета SDK Spark в IntelliJ IDEA" border="true":::

7. Нажмите кнопку **Готово**.

## <a name="create-a-standalone-scala-project"></a>Создание автономного проекта Scala

1. Запустите IntelliJ IDEA и выберите **Create New Project** (Создать проект), чтобы открыть окно **New Project** (Новый проект).

2. Выберите **Maven** в левой области.

3. Выберите пакет в поле **Project SDK** (Пакет SDK проекта). Если это поле пусто, щелкните **New...** (Создать...) и перейдите к каталогу установки Java.

4. Установите флажок **Create from archetype** (Создать на основе архетипа).  

5. В списке архетипов выберите **`org.scala-tools.archetypes:scala-archetype-simple`** . Этот архетип создает структуру каталога и скачивает зависимости по умолчанию, необходимые для написания программы Scala.

    :::image type="content" source="./media/apache-spark-create-standalone-application/intellij-project-create-maven.png" alt-text="Снимок экрана: выбранный архетип в окне создания проекта" border="true":::

6. Выберите **Далее**.

7. Разверните **Координаты артефакта**. Введите соответствующие значения для параметров **GroupId** и **ArtifactId**. Значения **Name** (Имя) и **Location** (Расположение) заполнятся автоматически. В этом руководстве используются представленные ниже значения.

    - **GroupId:** com.microsoft.spark.example;
    - **ArtifactId:** SparkSimpleApp.

    :::image type="content" source="./media/apache-spark-create-standalone-application/intellij-artifact-coordinates.png" alt-text="Снимок экрана: пункт &quot;Координаты артефакта&quot; в окне создания проекта" border="true":::

8. Выберите **Далее**.

9. Проверьте параметры и нажмите кнопку **Next** (Далее).

10. Проверьте имя и расположение проекта, а затем выберите **Finish** (Готово).  Импорт этого проекта займет несколько минут.

11. После импорта проекта в области слева выберите **SparkSimpleApp** > **src** > **test** > **scala** > **com** > **microsoft** > **spark** > **example**.  Щелкните **MySpec** правой кнопкой мыши и выберите **Delete...** (Удалить...). Этот файл не потребуется для этого приложения.  Нажмите кнопку **ОК** в диалоговом окне.
  
12. На следующих шагах описывается обновление файла **pom.xml** для определения зависимостей приложения Spark Scala. Чтобы автоматически скачать эти зависимости и разрешить их, необходимо настроить Maven.

13. В меню **File** (Файл) выберите **Settings** (Параметры), чтобы открыть окно **параметров**.

14. В окне **Settings** (Параметры) выберите **Build, Execution, Deployment** (Сборка, выполнение, развертывание) > **Build Tools** (Средства сборки) > **Maven** > **Importing** (Импорт).

15. Установите флажок **Import Maven projects automatically**(Импортировать проекты Maven автоматически).

16. Нажмите кнопку **Apply** (Применить), а затем нажмите кнопку **ОК**.  Затем вы вернетесь обратно в окно проекта.

    :::image type="content" source="./media/apache-spark-create-standalone-application/configure-maven-download.png" alt-text="Настройка автоматической загрузки Maven" border="true":::

17. В области слева выберите **src** > **main** > **scala** > **com.microsoft.spark.example**, а затем двойным щелчком выберите **App** (Приложение), чтобы открыть App.scala.

18. Замените имеющийся пример кода кодом ниже, а затем сохраните изменения. Этот код считывает данные из файла HVAC.csv (доступного для всех кластеров Spark в HDInsight). Извлекает строки, у которых в шестом столбце только одна цифра, и записывает выходные данные в **/HVACOut** в контейнере хранилища по умолчанию для кластера.

    ```scala
    package com.microsoft.spark.example

    import org.apache.spark.SparkConf
    import org.apache.spark.SparkContext

    /**
      * Test IO to wasb
      */
    object WasbIOTest {
        def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("WASBIOTest")
            val sc = new SparkContext(conf)
    
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
    
            //find the rows which have only one digit in the 7th column in the CSV
            val rdd1 = rdd.filter(s => s.split(",")(6).length() == 1)
    
            rdd1.saveAsTextFile("wasb:///HVACout")
        }
    }
    ```

19. В левой области дважды щелкните **pom.xml**.  

20. Добавьте следующие сегменты в файл `<project>\<properties>`:

    ```xml
    <scala.version>2.11.8</scala.version>
    <scala.compat.version>2.11.8</scala.compat.version>
    <scala.binary.version>2.11</scala.binary.version>
    ```

21. Добавьте следующие сегменты в файл `<project>\<dependencies>`:

    ```xml
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_${scala.binary.version}</artifactId>
        <version>2.3.0</version>
    </dependency>
    ```

    Сохраните изменения в файле pom.xml.

22. Создайте JAR-файл. IntelliJ IDEA позволяет создавать JAR-файлы в качестве артефактов проекта. Выполните следующие действия:

    1. В меню **File** (Файл) выберите **Project Structure...** (Структура проекта...).

    2. В окне **Project Structure** (Структура проекта) выберите **Artifacts** (Артефакты) > **символ "плюс" +**  > **JAR** > **From modules with dependencies...** (Из модулей с зависимостями...).

        :::image type="content" source="./media/apache-spark-create-standalone-application/hdinsight-create-jar1.png" alt-text="Добавление JAR-файла со структурой проекта в IntelliJ IDEA" border="true":::

    3. В окне **Create JAR from Modules** (Создание JAR-файла на основе модулей) выберите значок папки в текстовом поле **Main Class** (Основной класс).

    4. В окне **Select Main Class** (Выбор основного класса) выберите класс, который отображается по умолчанию, и щелкните **ОК**.

        :::image type="content" source="./media/apache-spark-create-standalone-application/hdinsight-create-jar2.png" alt-text="Выбор класса со структурой проекта в IntelliJ IDEA" border="true":::

    5. В окне **Create JAR from Modules** (Создание JAR-файла на основе модулей) выберите вариант **extract to the target JAR** (Извлечь в целевой JAR-файл) и щелкните **ОК**.  В результате будет создан один JAR-файл, содержащий все зависимости.

        :::image type="content" source="./media/apache-spark-create-standalone-application/hdinsight-create-jar3.png" alt-text="Добавление JAR-файла со структурой проекта из модуля в IntelliJ IDEA" border="true":::

    6. На вкладке **Макет выходных данных** содержится список всех JAR-файлов, которые включены в проект Maven. Здесь можно выбрать и удалить файлы, от которых не зависит работа приложения Scala. Из создаваемого приложения можно удалить все файлы, кроме последнего (**SparkSimpleApp compile output** (Выходные данные компиляции SparkSimpleApp)). Выберите JAR-файлы, которые нужно удалить, и щелкните значок минус **-** .

        :::image type="content" source="./media/apache-spark-create-standalone-application/hdi-delete-output-jars.png" alt-text="Удаление выходных данных со структурой проекта в IntelliJ IDEA" border="true":::

        Убедитесь, что установлен флажок **Include in project build** (Включить в сборку проекта). Это гарантирует, что JAR-файл будет создаваться при каждом создании и обновлении проекта. Нажмите кнопку **Apply** (Применить), а затем — **ОК**.

    7. Чтобы создать JAR-файл, выберите **Build** (Сборка) > **Build Artifacts** (Артефакты сборки)  > **Build** (Сборка). Компиляция проекта займет около 30 секунд.  Выходной JAR-файл будет создан в разделе **\out\artifacts**.

        :::image type="content" source="./media/apache-spark-create-standalone-application/hdi-artifact-output-jar.png" alt-text="Выходные данные артефакта проекта в IntelliJ IDEA" border="true":::

## <a name="run-the-application-on-the-apache-spark-cluster"></a>Запуск приложения в кластере Apache Spark

Чтобы запустить приложение в кластере, можно использовать следующие методы:

* **Скопируйте приложение JAR в большой двоичный объект службы хранилища Azure**, связанный с кластером. Вы можете использовать для этого служебную программу командной строки **AzCopy**. Кроме того, для отправки данных можно использовать множество других клиентов. Дополнительные сведения о них см. в статье [Отправка данных для заданий Apache Hadoop в HDInsight](../hdinsight-upload-data.md).

* **Используйте Apache Livy для удаленной отправки задания приложения** в кластер Spark. В кластерах HDInsight Spark есть сервер Livy, который использует конечные точки REST для удаленной отправки заданий Spark. Дополнительные сведения см. в статье об [удаленной отправке заданий Apache Spark с помощью Apache Livy и кластеров Spark в HDInsight ](apache-spark-livy-rest-interface.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите созданный кластер, сделав следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В поле **Поиск** в верхней части страницы введите **HDInsight**.

1. Выберите **Кластеры HDInsight** в разделе **Службы**.

1. В списке кластеров HDInsight, который отобразится, выберите **...** рядом с кластером, созданным при работе с этим учебником.

1. Выберите команду **Удалить**. Выберите **Да**.

:::image type="content" source="./media/apache-spark-create-standalone-application/hdinsight-azure-portal-delete-cluster.png " alt-text="Удаление кластера HDInsight на портале Azure" border="true":::

## <a name="next-step"></a>Следующий шаг

Из этой статьи вы узнали, как создать приложение Apache Spark на языке Scala. Из следующей статьи вы узнаете, как запустить это приложение на кластере HDInsight Spark, используя Livy.

> [!div class="nextstepaction"]
>[Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](./apache-spark-livy-rest-interface.md)
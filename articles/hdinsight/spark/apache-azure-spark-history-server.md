---
title: Использование расширенных функций на сервере журнала Apache Spark для отладки приложений в Azure HDInsight
description: Используйте расширенные функции на сервере журнала Apache Spark для отладки и диагностики приложений Spark в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 11/25/2019
ms.openlocfilehash: c6645bc605dbd60d331ac0de002c36384b2bbbc4
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864760"
---
# <a name="use-the-extended-features-of-the-apache-spark-history-server-to-debug-and-diagnose-spark-applications"></a>Использование расширенных функций сервера журнала Apache Spark для отладки и диагностики приложений Spark

В этой статье показано, как использовать расширенные функции сервера журнала Apache Spark для отладки и диагностики завершенных или выполняющихся приложений Spark. Расширение включает вкладку " **данные** ", вкладку **графа** и вкладку " **Диагностика** ". На вкладке **данные** можно проверить входные и выходные данные задания Spark. На вкладке **граф** можно проверить поток данных и воспроизвести граф задания. На вкладке **Диагностика** можно указать функции анализа **смещения данных**, **отклонения времени** и **использования исполнителя** .

## <a name="get-access-to-the-spark-history-server"></a>Получение доступа к серверу журнала Spark

Сервер журнала Spark — это веб-интерфейс для завершенных и выполняющихся приложений Spark. Его можно открыть либо из портал Azure, либо из URL-адреса.

### <a name="open-the-spark-history-server-web-ui-from-the-azure-portal"></a>Откройте пользовательский веб-интерфейс сервера журнала Spark из портал Azure

1. Откройте кластер Spark на [портале Azure](https://portal.azure.com/). Дополнительные сведения см. в разделе [Отображение кластеров](../hdinsight-administer-use-portal-linux.md#showClusters).
2. На **панели мониторинга кластера** выберите  **сервер журнала Spark**. При появлении запроса введите учетные данные администратора для кластера Spark.

    :::image type="content" source="./media/apache-azure-spark-history-server/azure-portal-dashboard-spark-history.png " alt-text="Запустите сервер журнала Spark из портал Azure." border="true"::: портал Azure ". border = "true":::

### <a name="open-the-spark-history-server-web-ui-by-url"></a>Открытие пользовательского веб-интерфейса сервера журнала Spark по URL-адресу

Откройте сервер журнала Spark, перейдя по `https://CLUSTERNAME.azurehdinsight.net/sparkhistory` адресу, где **имя_кластера** — это имя кластера Spark.

Веб-интерфейс сервера журнала Spark может выглядеть следующим образом:

:::image type="content" source="./media/apache-azure-spark-history-server/hdinsight-spark-history-server.png" alt-text="Страница сервера журнала Spark." border="true":::

## <a name="use-the-data-tab-in-the-spark-history-server"></a>Использование вкладки "данные" на сервере журнала Spark

Выберите идентификатор задания, а затем в меню инструментов выберите пункт **данные** , чтобы просмотреть представление данных.

+ Проверьте **входные данные**, **выходные данные** и **операции с таблицами** , выбрав отдельные вкладки.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-data-tabs.png" alt-text="Вкладки данных на странице приложения &quot;данные для Spark&quot;." border="true":::

+ Скопируйте все строки, нажав кнопку **Копировать** .

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-data-copy.png" alt-text="Копирование данных на странице приложения Spark." border="true":::

+ Сохраните все данные как. CSV-файл, нажав кнопку **CSV** .

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-data-save.png" alt-text="Сохранение данных в виде. CSV-файл на странице данных для приложения Spark." border="true":::

+ Выполните поиск данных, введя ключевые слова в поле **поиска** . Результаты поиска будут отображаться немедленно.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-data-search.png" alt-text="Поиск данных на странице приложения &quot;данные для Spark&quot;." border="true":::

+ Выберите заголовок столбца, чтобы отсортировать таблицу. Щелкните знак «плюс», чтобы развернуть строку, чтобы отобразить дополнительные сведения. Щелкните знак «минус», чтобы свернуть строку.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-data-table.png" alt-text="Таблица данных на странице приложения &quot;данные для Spark&quot;." border="true":::

+ Скачайте один файл, нажав кнопку **частичной загрузки** справа. Выбранный файл будет скачан локально. Если файл больше не существует, откроется новая вкладка для отображения сообщений об ошибках.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-data-download-row.png" alt-text="Строка загрузки данных на странице приложения &quot;данные для Spark&quot;." border="true":::

+ Скопируйте полный или относительный путь, выбрав параметр **Копировать полный путь** или **Копировать относительный путь** , который можно развернуть из меню загрузки. Для Azure Data Lake Storage файлов выберите **Открыть в обозреватель службы хранилища Azure** , чтобы запустить обозреватель службы хранилища Azure и перейдите к папке после входа.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-data-copy-path.png" alt-text="Параметры Копировать полный путь и копировать относительный путь на странице данные для приложения Spark." border="true":::

+ Если на одной странице слишком много строк для показа, выберите номера страниц в нижней части таблицы для перехода.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-data-page.png" alt-text="Номера страниц на странице данных для приложения Spark." border="true":::

+ Для получения дополнительных сведений наведите указатель мыши на или выберите знак вопроса рядом с полем **данные для приложения Spark** , чтобы отобразить подсказку.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-data-more-info.png" alt-text="Дополнительные сведения см. на странице &quot;данные для приложения Spark&quot;." border="true":::

+  Чтобы отправить отзыв о проблемах, выберите пункт **отправить нам отзыв**.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-feedback.png" alt-text="Оставить отзыв на странице данных для приложения Spark." border="true":::

## <a name="use-the-graph-tab-in-the-spark-history-server"></a>Использование вкладки граф на сервере журнала Spark

+ Выберите идентификатор задания, а затем выберите пункт **граф** в меню инструментов, чтобы просмотреть граф задания. По умолчанию на диаграмме отображаются все задания. Отфильтруйте результаты с помощью раскрывающегося меню **ИД задания** .

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-graph-jobid.png" alt-text="Раскрывающееся меню ИД задания на странице приложения Spark & графе заданий." border="true":::

+ По умолчанию выбран **ход выполнения** . Проверьте поток данных, выбрав в раскрывающемся меню **Отображение** пункт **Чтение** или **запись** .

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-display.png" alt-text="Проверьте поток данных на странице с графиком задания & приложения Spark." border="true":::

+ Цвет фона каждой задачи соответствует тепловой карте.

   :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-heatmap.png" alt-text="Тепловая схема на странице с графиком задания & приложения Spark." border="true":::


    |Color |Description |
    |---|---|
    |Зеленый|задание выполнено успешно.|
    |Оранжевый|Не удалось выполнить задачу, но это не повлияет на окончательный результат задания. Эти задачи имеют дубликаты или экземпляры повторных попыток, которые могут быть выполнены позже.|
    |Синий|задача выполняется.|
    |Белый|задача ожидает выполнения, либо этап пропущен.|
    |Красный|задачу выполнить не удалось.|

     :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-color-running.png" alt-text="Выполнение задачи на странице с графиком задания & приложения Spark." border="true":::

     Пропущенные этапы отображаются белым цветом.
    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-color-skip.png" alt-text="Пропущенная задача на странице с графиком задания & приложения Spark." border="true":::

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-color-failed.png" alt-text="Невыполненная задача на странице с графиком задания & приложения Spark." border="true":::

     > [!NOTE]  
     > Для завершенных заданий доступно воспроизведение. Нажмите кнопку **Воспроизведение** , чтобы вернуться к выполнению задания. В любое время прервите задание, нажав кнопку "Закрыть". При воспроизведении задания в каждой задаче отображается состояние по цвету. Воспроизведение не поддерживается для незавершенных заданий.

+ Прокрутите окно, чтобы увеличить или уменьшить граф задания, или выберите **масштаб в соответствии** с размерами экрана.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-zoom2fit.png" alt-text="Выберите масштаб в соответствии со страницей графика задания & приложения Spark." border="true":::

+ При сбое задач наведите указатель мыши на узел графа, чтобы увидеть подсказку, а затем выберите этап, чтобы открыть его на новой странице.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-tooltip.png" alt-text="Просмотрите подсказку на странице приложения Spark & графе заданий." border="true":::

+ На странице приложения Spark & график задания на этапах будут отображаться подсказки и мелкие значки, если задачи соответствуют следующим условиям.
  + Неравномерное распределение данных: размер чтения данных > средний размер считанных данных для всех задач на этом этапе * 2 *и* размер чтения данных > 10 МБ.
  + Отклонение по времени: время выполнения > среднее время выполнения всех задач на этом этапе * 2 *и* времени выполнения > 2 минуты.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-skew-icon.png" alt-text="Значок наклоненной задачи на странице приложения Spark & графе заданий." border="true":::

+ На узле граф задания отобразятся следующие сведения о каждом этапе.
  + ID
  + Имя или описание
  + Общий номер задачи
  + Чтение данных: сумма входного размера и случайный размер для чтения.
  + Запись данных: сумма размера вывода и случайный размер записи.
  + Время выполнения: время между временем начала первой попытки и временем выполнения последней попытки.
  + Число строк: сумма входных записей, выходных записей, случайный порядок чтения записей и случайного перезаписи записей.
  + Ход выполнения

    > [!NOTE]  
    > По умолчанию узел графа задания отображает сведения о последней попыток каждого этапа (за исключением времени выполнения этапа). Но во время воспроизведения узел графа задания будет отображать сведения о каждой попытки.

    > [!NOTE]  
    > Для объемов чтения и записи данных используется 1 МБ = 1000 КБ = 1000 * 1000 байт.

+ Отправьте отзыв о проблемах, выбрав параметр **отправить нам отзыв**.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-feedback.png" alt-text="Параметр feedback на странице приложения Spark & графе заданий." border="true":::

## <a name="use-the-diagnosis-tab-in-the-spark-history-server"></a>Использование вкладки "Диагностика" на сервере журнала Spark

Выберите идентификатор задания, а затем в меню инструментов выберите **Диагностика** , чтобы просмотреть представление диагностики заданий. Вкладка **Диагностика** включает в себя анализ **смещения данных**, **отклонения времени** и **использования исполнителя**.

+ Проверьте **отклонение данных**, **отклонения времени** и **Использование исполнителя** , выбрав вкладки соответственно.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-diagnosis-tabs.png" alt-text="Вкладка «неравномерность данных» на вкладке «Диагностика»." border="true":::

### <a name="data-skew"></a>Неравномерное распределение данных

Перейдите на вкладку « **отклонение данных** ». Соответствующие отклоненные задачи отображаются на основе указанных параметров.

#### <a name="specify-parameters"></a>Указание параметров

В разделе **Указание параметров** отображаются параметры, используемые для обнаружения смещения данных. Правило по умолчанию: прочитанные данные о задаче больше трех раз из числа считанных данных задачи, а чтение данных задачи превышает 10 МБ. Если вы хотите определить собственное правило для отклоненных задач, можно выбрать параметры. Секции **отклоненные** и **наклонные диаграммы** будут обновлены соответствующим образом.

#### <a name="skewed-stage"></a>Отклоненный этап

В разделе **отклоненный этап** отображаются этапы, имеющие отклоненные задачи, удовлетворяющие заданным критериям. Если в одном этапе имеется несколько отклоненных задач, в разделе **отклоненный этап** отображается только наиболее наклоненная задача (то есть наибольшие данные для смещения данных).

:::image type="content" source="./media/apache-azure-spark-history-server/sparkui-diagnosis-dataskew-section2.png" alt-text="Расширенное представление вкладки «неравномерное распределение данных» на вкладке «Диагностика»." border="true":::

##### <a name="skew-chart"></a>Диаграмма отклонений

При выборе строки в таблице « **стадия наклона** » на **диаграмме «неравномерность** » отображаются дополнительные сведения о распределении задач на основе времени чтения и выполнения данных. Отклоненные задачи помечаются красным цветом, и обычные задачи помечаются синим цветом. Для повышения производительности на диаграмме отображается до 100 образцов задач. Сведения о задаче отображаются в нижней правой панели.

:::image type="content" source="./media/apache-azure-spark-history-server/sparkui-diagnosis-dataskew-section3.png" alt-text="Диаграмма отклонений для этапа 10 в пользовательском интерфейсе Spark." border="true":::

### <a name="time-skew"></a>Неравномерное распределение времени

На вкладке **Неравномерное распределение времени** отображаются задачи с неравномерным распределением времени выполнения.

#### <a name="specify-parameters"></a>Указание параметров

В разделе **Указание параметров** отображаются параметры, которые используются для определения смещения времени. Правило по умолчанию: время выполнения задачи больше трех раз среднего времени выполнения, а время выполнения задачи превышает 30 секунд. Параметры можно изменить в соответствии с вашими потребностями. На **смещенной и** **наклонной диаграммах** отображаются соответствующие сведения о стадиях и задачах, как на вкладке « **отклонение данных** ».

При выборе параметра **отклонение по времени** отфильтрованный результат отображается в разделе **отклоненный этап** в соответствии с параметрами, заданными в разделе **Указание параметров** . При выборе одного элемента на смещенном **этапе** соответствующая диаграмма становится черновиком в третьем разделе, а сведения о задаче отображаются на нижней правой панели.

:::image type="content" source="./media/apache-azure-spark-history-server/sparkui-diagnosis-timeskew-section2.png" alt-text="Вкладка «отклонение по времени» на вкладке «Диагностика»." border="true":::

### <a name="executor-usage-analysis-graphs"></a>Графы анализа использования исполнителя

На **графике использования исполнителя** отображается фактическое выделение и состояние выполнения задания.  

При выборе **анализа использования исполнителя** четыре различные кривые об использовании исполнителя являются черновиками: **выделенные исполнители**, **выполняющиеся исполнители**, **простаивающие исполнители** и **Максимальное количество экземпляров исполнителя**. Каждый **добавленный исполнитель** или событие, **удаленное по исполнителю** , увеличивают или уменьшают выделенные исполнители. Для получения дополнительных сравнений можно проверить **временную шкалу событий** на вкладке **задания** .

:::image type="content" source="./media/apache-azure-spark-history-server/sparkui-diagnosis-executors.png" alt-text="Вкладка &quot;анализ использования исполнителя&quot; на вкладке Диагностика." border="true":::

Выберите значок цвета, чтобы выбрать или отменить выбор соответствующего содержимого во всех черновиках.

 :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-diagnosis-select-chart.png" alt-text="Выберите диаграмму на вкладке Анализ использования исполнителя." border="true":::

## <a name="faq"></a>ВОПРОСЫ И ОТВЕТЫ

### <a name="how-do-i-revert-to-the-community-version"></a>Разделы справки вернуться к версии сообщества?

Чтобы вернуться к версии сообщества, выполните следующие действия.

1. Откройте кластер в Ambari.
1. Перейдите к **Spark2**  >  **configs**.
1. Выберите **пользовательские spark2 — значения по умолчанию**.
1. Выберите **Добавить свойство...**.
1. Добавьте **Spark. UI. Улучшенный. Enabled = false**, а затем сохраните его.
1. Это свойство задает значение **false**.
1. Нажмите кнопку **сохранить** , чтобы сохранить конфигурацию.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-turn-off.png" alt-text="Отключение функции в Apache Ambari." border="true":::

1. Выберите **Spark2** на панели слева. Затем на вкладке **Сводка** выберите **Spark2 History Server (сервер журнала событий**).

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-restart1.png" alt-text="Представление сводки в Apache Ambari." border="true":::

1. Чтобы перезапустить сервер журнала Spark, нажмите кнопку " **Начало** " справа от **сервера журнала Spark2**, а затем выберите **перезапустить** в раскрывающемся меню.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-restart2.png" alt-text="Перезапустите сервер журнала Spark в Apache Ambari." border="true":::  

1. Обновите веб-интерфейс сервера журнала Spark. Она вернется к версии сообщества.

### <a name="how-do-i-upload-a-spark-history-server-event-to-report-it-as-an-issue"></a>Разделы справки отправить событие сервера журнала Spark, чтобы сообщить о нем как о проблемах?

При возникновении ошибки на сервере журнала Spark выполните следующие действия, чтобы сообщить о событии.

1. Скачайте событие, выбрав **скачать** в пользовательском веб-интерфейсе сервера журнала Spark.

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-download-event.png" alt-text="Скачайте событие в пользовательский интерфейс сервера журнала Spark." border="true":::

2. Выберите **отправить нам отзыв** на странице с **графиком задания & приложения Spark** .

    :::image type="content" source="./media/apache-azure-spark-history-server/sparkui-graph-feedback.png" alt-text="Оставить отзыв на странице &quot;график задания & приложения Spark&quot;" border="true":::

3. Укажите заголовок и описание ошибки. Затем перетащите ZIP-файл в поле редактирования и выберите **отправить новую ошибку**.

    :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-file-issue.png" alt-text="Отправьте и отправьте новую ошибку." border="true":::

### <a name="how-do-i-upgrade-a-jar-file-in-a-hotfix-scenario"></a>Разделы справки обновить JAR-файл в сценарии исправления?

Если вы хотите обновить исправление, используйте следующий скрипт, который будет обновлен `spark-enhancement.jar*` .

**upgrade_spark_enhancement.sh**:

   ```bash
    #!/usr/bin/env bash

    # Copyright (C) Microsoft Corporation. All rights reserved.

    # Arguments:
    # $1 Enhancement jar path

    if [ "$#" -ne 1 ]; then
        >&2 echo "Please provide the upgrade jar path."
        exit 1
    fi

    install_jar() {
        tmp_jar_path="/tmp/spark-enhancement-hotfix-$( date +%s )"

        if wget -O "$tmp_jar_path" "$2"; then
            for FILE in "$1"/spark-enhancement*.jar
            do
                back_up_path="$FILE.original.$( date +%s )"
                echo "Back up $FILE to $back_up_path"
                mv "$FILE" "$back_up_path"
                echo "Copy the hotfix jar file from $tmp_jar_path   to $FILE"
                cp "$tmp_jar_path" "$FILE"

                "Hotfix done."
                break
            done
        else    
            >&2 echo "Download jar file failed."
            exit 1
        fi
    }

    jars_folder="/usr/hdp/current/spark2-client/jars"
    jar_path=$1

    if ls ${jars_folder}/spark-enhancement*.jar 1>/dev/null 2>&1;   then
        install_jar "$jars_folder" "$jar_path"
    else
        >&2 echo "There is no target jar on this node. Exit with no action."
        exit 0
    fi
   ```

#### <a name="usage"></a>Использование

`upgrade_spark_enhancement.sh https://${jar_path}`

#### <a name="example"></a>Пример

`upgrade_spark_enhancement.sh https://${account_name}.blob.core.windows.net/packages/jars/spark-enhancement-${version}.jar`

#### <a name="use-the-bash-file-from-the-azure-portal"></a>Использование Bash файла из портал Azure

1. Запустите [портал Azure](https://ms.portal.azure.com), а затем выберите свой кластер.
2. Завершите [действие скрипта](../hdinsight-hadoop-customize-cluster-linux.md) со следующими параметрами.

    |Свойство |Значение |
    |---|---|
    |Тип скрипта|- Custom|
    |Имя|упградежар|
    |URI bash-скрипта|`https://hdinsighttoolingstorage.blob.core.windows.net/shsscriptactions/upgrade_spark_enhancement.sh`|
    |Типы узлов|Головной, Рабочий|
    |Параметры|`https://${account_name}.blob.core.windows.net/packages/jars/spark-enhancement-${version}.jar`|

     :::image type="content" source="./media/apache-azure-spark-history-server/apache-spark-upload1.png" alt-text="Действие &quot;отправить скрипт&quot; портал Azure" border="true":::

## <a name="known-issues"></a>Известные проблемы

+ В настоящее время сервер журнала Spark работает только для Spark 2,3 и 2,4.

+ Входные и выходные данные, использующие RDD, не будут отображаться на вкладке " **данные** ".

## <a name="next-steps"></a>Дальнейшие действия

+ [Управление ресурсами для кластера Apache Spark в HDInsight](apache-spark-resource-manager.md)
+ [Настройка параметров Apache Spark](apache-spark-settings.md)

## <a name="suggestions"></a>Предложения

Если у вас возникли какие-либо отзывы или возникли проблемы при использовании этого средства, отправьте сообщение электронной почты по адресу ( [hdivstool@microsoft.com](mailto:hdivstool@microsoft.com) ).

---
title: Краткое руководство. Анализ данных в Azure Data Lake Storage 2-го поколения с помощью Azure Databricks | Документация Майкрософт
description: Узнайте, как выполнить задание Spark в Azure Databricks с помощью портала Azure и учетной записи хранения Azure Data Lake Storage 2-го поколения.
author: normesta
ms.author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: quickstart
ms.date: 06/12/2020
ms.reviewer: jeking
ms.openlocfilehash: 908bf21d2fe101731b11e3a8ad783f17728c8ed3
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677336"
---
# <a name="quickstart-analyze-data-with-databricks"></a>Краткое руководство. Анализ данных с помощью Databricks

В рамках этого краткого руководства вы выполните задание Apache Spark с помощью Azure Databricks для анализа данных, хранимых в учетной записи хранения. В рамках задания Spark показано, как анализировать данные подписки на радиоканал, чтобы получить сведения о бесплатном или оплачиваемом использовании на основе демографических данных.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

* Имя учетной записи хранения Azure Data Lake Storage 2-го поколения. [Создайте учетную запись хранения Azure Data Lake Storage 2-го поколения](data-lake-storage-quickstart-create-account.md).

* Идентификатор клиента, идентификатор приложения и пароль субъекта-службы Azure с назначенной ролью **участника для данных большого двоичного объекта хранилища**. [Создайте субъект-службу](../../active-directory/develop/howto-create-service-principal-portal.md).

  > [!IMPORTANT]
  > Назначьте роль в учетной записи хранения Data Lake Storage 2-го поколения. Можно назначить роль родительской группе ресурсов или подписке, но вы будете получать ошибки, связанные с разрешениями, пока роль не будет назначена учетной записи хранения.

## <a name="create-an-azure-databricks-workspace"></a>Создание рабочей области Azure Databricks

В этом разделе вы создадите рабочую область Azure Databricks с помощью портала Azure.

1. На портале Azure выберите **Создать ресурс** > **Analytics** > **Azure Databricks**.

    ![Databricks на портале Azure](./media/data-lake-storage-quickstart-create-databricks-account/azure-databricks-on-portal.png "Databricks на портале Azure")

2. В разделе **службы Azure Databricks** укажите значения для создания рабочей области Databricks.

    ![Создайте рабочую область Azure Databricks](./media/data-lake-storage-quickstart-create-databricks-account/create-databricks-workspace.png "Создание рабочей области Azure Databricks").

    Укажите следующие значения.

    |Свойство  |Описание  |
    |---------|---------|
    |**Имя рабочей области**     | Укажите имя рабочей области Databricks.        |
    |**Подписка**     | Выберите подписку Azure в раскрывающемся списке.        |
    |**Группа ресурсов**     | Укажите, следует ли создать новую группу ресурсов или использовать имеющуюся. Группа ресурсов — это контейнер, содержащий связанные ресурсы для решения Azure. Дополнительные сведения см. в [обзоре группы ресурсов Azure](../../azure-resource-manager/management/overview.md). |
    |**Расположение**     | Выберите **Западная часть США 2**. Если хотите, выберите другую публичную область.        |
    |**Ценовая категория**     |  Вы можете выбрать уровень **Стандартный** или **Премиум**. Дополнительные сведения об этих ценовых категориях см. на [странице цен на Databricks](https://azure.microsoft.com/pricing/details/databricks/).       |

3. Создание учетной записи займет несколько минут. Чтобы отслеживать состояние операции, просмотрите индикатор выполнения вверху.

4. Установите флажок **Закрепить на панели мониторинга** и щелкните **Создать**.

## <a name="create-a-spark-cluster-in-databricks"></a>Создание кластера Spark в Databricks

1. На портале Azure перейдите к созданной рабочей области Databricks, а затем выберите **Launch Workspace** (Запуск рабочей области).

2. Вы будете перенаправлены на портал Azure Databricks. На портале выберите **Новый** > **Кластер**.

    ![Databricks в Azure](./media/data-lake-storage-quickstart-create-databricks-account/databricks-on-azure.png "Databricks в Azure")

3. На странице **создания кластера** укажите значения для создания кластера.

    ![Создание кластера Databricks Spark в Azure](./media/data-lake-storage-quickstart-create-databricks-account/create-databricks-spark-cluster.png "Создание кластера Databricks Spark в Azure")

    Заполните значения в следующих полях, и примите значения по умолчанию для других полей.

    - Введите имя кластера.
     
    - Убедитесь, что установлен флажок **Terminate after 120 minutes of activity** (Завершить работу после 120 минут отсутствия активности). Укажите длительность (в минутах) для завершения работы кластера, если тот не используется.

4. Выберите **Create cluster** (Создать кластер). После запуска кластера можно вложить записные книжки в кластер и запустить задания Spark.

Дополнительные сведения о создании кластеров см. в статье [о создании кластера Spark в Azure Databricks](https://docs.azuredatabricks.net/user-guide/clusters/create.html).

## <a name="create-notebook"></a>Создание записной книжки

В этом разделе показано создание записной книжки в рабочей области Azure Databricks и выполнение фрагментов кода для настройки учетной записи хранения.

1. На [портале Azure](https://portal.azure.com) перейдите к созданной рабочей области Azure Databricks, а затем выберите **Launch Workspace** (Запуск рабочей области).

2. В левой области выберите **Рабочая область**. В раскрывающемся списке **Рабочая область** выберите **Создать** > **Notebook** (Записная книжка).

    ![Снимок экрана, на котором показано создание записной книжки в Databricks и выделены пункты меню "Создать" и "Записная книжка".](./media/data-lake-storage-quickstart-create-databricks-account/databricks-create-notebook.png "Создание записной книжки в Databricks")

3. В диалоговом окне **создания записной книжки** введите имя записной книжки. Выберите **Scala** в качестве языка, а затем выберите созданный ранее кластер Spark.

    ![Создание записной книжки в Databricks](./media/data-lake-storage-quickstart-create-databricks-account/databricks-notebook-details.png "Создание записной книжки в Databricks")

    Нажмите кнопку **создания**.

4. Скопируйте и вставьте следующий блок кода в первую ячейку, но не запускайте этот код.

   ```scala
   spark.conf.set("fs.azure.account.auth.type.<storage-account-name>.dfs.core.windows.net", "OAuth")
   spark.conf.set("fs.azure.account.oauth.provider.type.<storage-account-name>.dfs.core.windows.net", "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider")
   spark.conf.set("fs.azure.account.oauth2.client.id.<storage-account-name>.dfs.core.windows.net", "<appID>")
   spark.conf.set("fs.azure.account.oauth2.client.secret.<storage-account-name>.dfs.core.windows.net", "<password>")
   spark.conf.set("fs.azure.account.oauth2.client.endpoint.<storage-account-name>.dfs.core.windows.net", "https://login.microsoftonline.com/<tenant-id>/oauth2/token")
   spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "true")
   dbutils.fs.ls("abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/")
   spark.conf.set("fs.azure.createRemoteFileSystemDuringInitialization", "false")

   ```
5. В этом блоке кода замените значения заполнителя `storage-account-name`, `appID`, `password` и `tenant-id` значениями, полученными в ходе создания субъекта-службы. Задайте значение заполнителя `container-name` для имени контейнера.

6. Нажмите клавиши **SHIFT + ВВОД** , чтобы запустить код в этом блоке.

## <a name="ingest-sample-data"></a>Принятие демонстрационных данных

Прежде чем приступить к работе с этим разделом, выполните следующие предварительные требования.

Введите следующий код в ячейку записной книжки:

```bash
%sh wget -P /tmp https://raw.githubusercontent.com/Azure/usql/master/Examples/Samples/Data/json/radiowebsite/small_radio_json.json
```

В ячейке нажмите клавиши **SHIFT+ВВОД** , чтобы выполнить код.

Теперь в новую ячейку под этой введите следующий код и замените значения, которые отображаются в скобках, использованными ранее.

```python
dbutils.fs.cp("file:///tmp/small_radio_json.json", "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/")
```

В ячейке нажмите клавиши **SHIFT+ВВОД** , чтобы выполнить код.

## <a name="run-a-spark-sql-job"></a>Выполнение задания SQL Spark

Выполните следующие задачи для выполнения задания SQL Spark на основе данных.

1. Выполните инструкцию SQL для создания временной таблицы, используя данные образца файла данных JSON, **small_radio_json.json**. В указанном ниже фрагменте кода замените значения заполнителя именем контейнера и учетной записи хранения. Используя созданные ранее заметки, вставьте фрагмент кода в ячейку нового кода в записной книжке, а затем нажмите сочетание клавиш SHIFT + ВВОД.

    ```sql
    %sql
    DROP TABLE IF EXISTS radio_sample_data;
    CREATE TABLE radio_sample_data
    USING json
    OPTIONS (
     path  "abfss://<container-name>@<storage-account-name>.dfs.core.windows.net/small_radio_json.json"
    )
    ```

    После успешного выполнения команды все данные из файла JSON будут отображаться в виде таблицы в кластере Databricks.

    Волшебная команда языка `%sql` позволяет выполнить код SQL из записной книжки, даже если записная книжка другого типа. Дополнительные сведения см. в статье об [объединении языков программирования в записной книжке](https://docs.azuredatabricks.net/user-guide/notebooks/index.html#mixing-languages-in-a-notebook).

2. Рассмотрим моментальный снимок примера данных JSON, чтобы лучше ознакомиться с запросом, который вы выполняете. Вставьте следующий фрагмент кода в ячейку кода и нажмите клавиши **SHIFT + ВВОД**.

    ```sql
    %sql
    SELECT * from radio_sample_data
    ```

3. Отобразятся табличные данные, как показано на следующем снимке экрана (показаны только некоторые столбцы).

    ![Пример данных JSON](./media/data-lake-storage-quickstart-create-databricks-account/databricks-sample-csv-data.png "Пример данных JSON")

    Среди прочего пример данных демонстрирует пол аудитории радиоканала (имя столбца **gender** ), а также категорию подписки (бесплатная или платная) (имя столбца **level** ).

4. Теперь можно создать визуальное представление данных по каждому полу, а также по числу пользователей с бесплатными и платными учетными записями. В нижней части выходных табличных данных щелкните значок **линейчатой диаграммы** , а затем — **параметры построения**.

    ![Создание линейчатой диаграммы](./media/data-lake-storage-quickstart-create-databricks-account/create-plots-databricks-notebook.png "Создание линейчатой диаграммы")

5. В разделе **настроек построения** перетащите значения, как показано на снимке экрана.

    ![Снимок экрана, на котором показана страница настройки графика и значения, которые можно перетащить.](./media/data-lake-storage-quickstart-create-databricks-account/databricks-notebook-customize-plot.png "Настройка линейчатой диаграммы")

    - Для параметра **Ключи** задайте значение **gender**.
    - Для параметра **Series groupings** (Группирование ряда) задайте значение **level**.
    - Для параметра **Значения** задайте значение **level**.
    - Для параметра **Агрегирование** задайте значение **COUNT**.

6. Нажмите кнопку **Применить**.

7. Выходные данные отображают визуальное представление, как показано на следующем снимке экрана:

     ![Настройка линейчатой диаграммы](./media/data-lake-storage-quickstart-create-databricks-account/databricks-sql-query-output-bar-chart.png "Настройка линейчатой диаграммы")

## <a name="clean-up-resources"></a>Очистка ресурсов

Изучив эту статью, вы можете завершить работу кластера. В рабочей области Azure Databricks выберите **Кластеры** и найдите кластер, работу которого необходимо завершить. Наведите указатель мыши на многоточие в столбце **Действия** и выберите значок **Завершить**.

![Остановка кластера Databricks](./media/data-lake-storage-quickstart-create-databricks-account/terminate-databricks-cluster.png "Остановка кластера Databricks")

Если не завершить работу кластера вручную, она завершится автоматически, если во время создания кластера вы установили флажок **Terminate after \_\_ minutes of inactivity** (Завершать работу после \_\_ мин бездействия). Если установить эту опцию, кластер завершит роботу после того, как не будет активен в течение заданного промежутка времени.

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как создать кластер Spark в Azure Databricks и запускать задание Spark с использованием данных, хранящихся в учетной записи хранения с поддержкой Data Lake Storage 2-го поколения.

Перейдите к следующей статье, чтобы узнать, как выполнять операции ETL (извлечения, преобразование и загрузка данных) с помощью Azure Databricks.

> [!div class="nextstepaction"]
>[Извлечение, преобразование и загрузка данных с помощью Azure Databricks](../../azure-databricks/databricks-extract-load-sql-data-warehouse.md)

- Чтобы узнать, как импортировать данные из других источников данных в Azure Databricks, см. статью об [источниках данных Spark](https://docs.azuredatabricks.net/spark/latest/data-sources/index.html).

- Дополнительные сведения о других способах доступа к Azure Data Lake Storage 2-го поколения из рабочей области Azure Databricks см. в [этой статье](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/azure-datalake-gen2.html).

---
title: 'Преобразование данных с помощью Spark в службе "Фабрика данных Azure" '
description: В этом руководстве представлены пошаговые инструкции по преобразованию данных с использованием действия Spark в фабрике данных Azure.
ms.service: data-factory
ms.topic: tutorial
author: nabhishek
ms.author: abnarain
ms.date: 01/10/2018
ms.openlocfilehash: 2e2a50a96402f01fe914c79d5257fc5bb4dc57a0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100377794"
---
# <a name="transform-data-in-the-cloud-by-using-a-spark-activity-in-azure-data-factory"></a>Преобразование данных в облаке с помощью действия Spark в фабрике данных Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве вы создадите конвейер фабрики данных Azure с помощью портала Azure. Конвейер преобразует данные с помощью действия Spark и связанной службы Azure HDInsight по запросу. 

В этом руководстве вы выполните следующие шаги:

> [!div class="checklist"]
> * Создали фабрику данных. 
> * Создание конвейера, использующего действие Spark.
> * Активация выполнения конвейера.
> * Осуществили мониторинг выполнения конвейера.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

* **Учетная запись хранения Azure**. Нужно создать входной файл и сценарий Python и передать их в службу хранилища Azure. Выходные данные программы Spark хранятся в этой учетной записи хранения. Кластер Spark по запросу использует ту же учетную запись хранения, что и его основное хранилище.  

> [!NOTE]
> HDInsight поддерживает только учетные записи хранения общего назначения с ценовой категорией "Стандартный". Убедитесь, что используете не учетную запись ценовой категории "Премиум" и не учетную запись только для большого двоичного объекта.

* **Azure PowerShell**. Следуйте инструкциям по [установке и настройке Azure PowerShell](/powershell/azure/install-Az-ps).


### <a name="upload-the-python-script-to-your-blob-storage-account"></a>Отправка скрипта Python в учетную запись хранилища BLOB-объектов
1. Создайте файл Python с именем **WordCount_Spark.py** со следующим содержимым: 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
1. Замените свойство *&lt;storageAccountName&gt;* именем своей учетной записи хранения Azure. Затем сохраните файл. 
1. В хранилище BLOB-объектов Azure создайте контейнер с именем **adftutorial**, если он не существует. 
1. Создайте папку с именем **spark**.
1. Создайте вложенную папку с именем **script** в папке **spark**. 
1. Отправьте файл **WordCount_Spark.py** во вложенную папку **script**. 


### <a name="upload-the-input-file"></a>Отправка входного файла
1. Создайте файл с определенным текстом и назовите его **minecraftstory.txt**. Программа Spark подсчитывает количество слов в этом тексте. 
1. Создайте вложенную папку с именем **inputfiles** в папке **spark**. 
1. Отправьте файл **minecraftstory.txt** во вложенную папку **inputfiles**. 

## <a name="create-a-data-factory"></a>Создание фабрики данных

1. Запустите веб-браузер **Microsoft Edge** или **Google Chrome**. Сейчас только эти браузеры поддерживают пользовательский интерфейс фабрики данных.
1. В меню слева выберите **Создать**, **Данные+аналитика**, **Фабрика данных**. 
   
   ![Выбор фабрики данных в области "Создать"](./media/tutorial-transform-data-spark-portal/new-azure-data-factory-menu.png)
1. На панели **Новая фабрика данных** введите **ADFTutorialDataFactory** в поле **Имя**. 
      
   ![Панель "Новая фабрика данных"](./media/tutorial-transform-data-spark-portal/new-azure-data-factory.png)
 
   Имя фабрики данных Azure должно быть *глобально уникальным*. Если появится следующая ошибка, измените имя фабрики данных. (Например, используйте **&lt;yourname&gt;ADFTutorialDataFactory**.) Правила именования для артефактов службы "Фабрика данных" приведены в статье [Фабрика данных Azure — правила именования](naming-rules.md).
  
   ![Ошибка "Имя недоступно"](./media/tutorial-transform-data-spark-portal/name-not-available-error.png)
1. В поле **Подписка** выберите подписку Azure, в рамках которой вы хотите создать фабрику данных. 
1. Для **группы ресурсов** выполните одно из следующих действий:
     
   - Выберите **Использовать существующую** и укажите существующую группу ресурсов в раскрывающемся списке. 
   - Выберите **Создать новую** и укажите имя группы ресурсов.   
         
   Некоторые действия, описанные в этом руководстве, предполагают, что для группы ресурсов используется имя **ADFTutorialResourceGroup**. Сведения о группах ресурсов см. в статье, где описывается [использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/overview.md).  
1. Укажите **V2** при выборе **версии**.
1. В поле **Расположение** выберите расположение фабрики данных. 

   Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (такие как служба хранилища Azure и база данных SQL Azure) и вычислительные среды (например, HDInsight), используемые фабрикой данных, могут находиться в других регионах.

1. Нажмите кнопку **создания**.

1. Когда создание завершится, откроется страница **Фабрика данных**. Выберите элемент **Author & Monitor** (Создание и мониторинг), чтобы открыть на отдельной вкладке приложение пользовательского интерфейса службы "Фабрика данных".

    ![Домашняя страница фабрики данных с элементом Author & Monitor (Создание и мониторинг)](./media/tutorial-transform-data-spark-portal/data-factory-home-page.png)

## <a name="create-linked-services"></a>Создание связанных служб
Создайте две связанные службы в этом разделе: 
    
- **Связанную службу хранилища Azure**, которая связывает учетную запись хранения Azure с фабрикой данных. Это хранилище используется кластером HDInsight по запросу. В нем также содержится скрипт Spark для выполнения. 
- **Связанную службу HDInsight по запросу**. Фабрика данных Azure автоматически создает кластер HDInsight и запускает программу Spark. Кластер Hadoop удаляется, если он не используется в течение заданного времени. 

### <a name="create-an-azure-storage-linked-service"></a>Создание связанной службы хранилища Azure

1. На странице **Let's get started** (Начало работы) переключитесь на вкладку **Edit** (Редактирование) на левой панели. 

   ![Страница Let's get started (Начало работы)](./media/tutorial-transform-data-spark-portal/get-started-page.png)

1. В нижней части окна выберите **Подключения**, а затем **+ Создать**. 

   ![Кнопки для создания подключения](./media/tutorial-transform-data-spark-portal/new-connection.png)
1. В окне **New Linked Service** (Новая связанная служба) выберите **Хранилище данных** > **Хранилище BLOB-объектов Azure** и щелкните **Продолжить**. 

   ![Выбор элемента "Хранилище BLOB-объектов Azure"](./media/tutorial-transform-data-spark-portal/select-azure-storage.png)
1. Выберите имя из списка в поле **Имя учетной записи хранения**, а затем щелкните **Сохранить**. 

   ![Поле для указания имени учетной записи хранения](./media/tutorial-transform-data-spark-portal/new-azure-storage-linked-service.png)


### <a name="create-an-on-demand-hdinsight-linked-service"></a>Создание связанной службы HDInsight по запросу

1. Снова нажмите кнопку **+ Создать**, чтобы создать еще одну связанную службу. 
1. В окне **New Linked Service** (Новая связанная служба) выберите **Среда выполнения приложений** > **Azure HDInsight**, а затем выберите **Continue** (Продолжить). 

   ![Выбор элемента Azure HDInsight](./media/tutorial-transform-data-spark-portal/select-azure-hdinsight.png)
1. В окне **New Linked Service** (Новая связанная служба) сделайте следующее: 

   а. В поле **Name** (Имя) введите **AzureHDInsightLinkedService**.
   
   b. Убедитесь, что в поле **Type** (Тип) выбран вариант **On-demand HDInsight** (HDInsight по запросу).
   
   c. Для параметра **Azure Storage Linked Service** (Связанная служба хранилища Azure) выберите **AzureStorage1**. Эта та связанная служба, которую вы создали ранее. Если было использовано другое имя, укажите его в этом поле. 
   
   d. В качестве **типа кластера** выберите **spark**.
   
   д) В поле **Идентификатор субъекта-службы** введите идентификатор субъекта-службы, имеющего права на создание кластера HDInsight. 
   
      Этому субъекту-службе должна быть назначена роль участника подписки или группы ресурсов, в которой создается кластер. Дополнительные сведения см. в статье [Создание приложения Azure Active Directory и субъекта-службы с доступом к ресурсам с помощью портала](../active-directory/develop/howto-create-service-principal-portal.md). **Идентификатор субъекта-службы** —это эквивалент *идентификатора приложения*, а **ключ субъекта-службы** — значения *секрета клиента*.
   
   е) Введите ключ в поле **Ключ субъекта-службы**. 
   
   ж. В качестве **группы ресурсов** выберите ту же группу ресурсов, которая была использована при создании фабрики данных. В этой группе ресурсов будет создан кластер HDInsight. 
   
   h. Разверните список **Тип ОС**.
   
   i. Введите **имя пользователя кластера**. 
   
   j. Введите **пароль кластера** для этого пользователя. 
   
   k. Нажмите кнопку **Готово**. 

   ![Параметры связанной службы HDInsight](./media/tutorial-transform-data-spark-portal/azure-hdinsight-linked-service-settings.png)

> [!NOTE]
> Azure HDInsight ограничивает общее количество ядер, которые можно использовать в каждом поддерживаемом регионе Azure. Для связанной службы HDInsight по требованию создается кластер HDInsight в расположении хранилища Azure, используемом в качестве основного хранилища. Убедитесь, что имеется достаточное количество квот ядра для успешного создания кластера. Дополнительные сведения см. в статье [Установка кластеров в HDInsight с использованием Hadoop, Spark, Kafka и других технологий](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md). 

## <a name="create-a-pipeline"></a>Создание конвейера

1. Нажмите кнопку **+** (плюс) и в меню выберите **Pipeline** (Конвейер).

   ![Кнопки для создания конвейера](./media/tutorial-transform-data-spark-portal/new-pipeline-menu.png)
1. На панели элементов **Действия** разверните узел **HDInsight**. Перетащите действие **Spark** с панели элементов **Действия** в область конструктора конвейера. 

   ![Перетаскивание действия Spark](./media/tutorial-transform-data-spark-portal/drag-drop-spark-activity.png)
1. В свойствах для окна действия **Spark** в нижней части страницы завершите следующие действия: 

   а. Перейдите на вкладку **HDI Cluster** (Кластер HDI).
   
   b. Выберите службу **AzureHDInsightLinkedService** (созданную в приведенной выше процедуре). 
        
   ![Определение связанной службы HDInsight](./media/tutorial-transform-data-spark-portal/select-hdinsight-linked-service.png)
1. Перейдите на вкладку **Script/Jar** (Скрипт или JAR-файл) и выполните следующие действия: 

   а. Для параметра **Job Linked Service** (Связанная служба задания) выберите значение **AzureStorage1**.
   
   b. Выберите **Поиск в хранилище**.

   ![Определение сценария Spark на вкладке Script/Jar (Скрипт или JAR-файл)](./media/tutorial-transform-data-spark-portal/specify-spark-script.png)
   
   c. Перейдите к папке **adftutorial/spark/script**, выберите в ней файл **WordCount_Spark.py**, а затем выберите **Готово**.      

1. Чтобы проверить работу конвейера, нажмите кнопку **Проверка** на панели инструментов. Чтобы закрыть окно проверки, нажмите кнопку **>>** Стрелка вправо. 
    
   ![Кнопка Validate (Проверка)](./media/tutorial-transform-data-spark-portal/validate-button.png)
1. Выберите **Опубликовать все**. Пользовательский интерфейс фабрики данных опубликует сущности (связанные службы и конвейер) в службе фабрики данных Azure. 
    
   ![Кнопка "Опубликовать все"](./media/tutorial-transform-data-spark-portal/publish-button.png)


## <a name="trigger-a-pipeline-run"></a>Активация выполнения конвейера
Выберите **Добавить триггер** на панели инструментов, а затем **Trigger Now** (Запустить сейчас). 

![Кнопки "Trigger" (Запустить) и "Trigger Now" (Запустить сейчас)](./media/tutorial-transform-data-spark-portal/trigger-now-menu.png)

## <a name="monitor-the-pipeline-run"></a>Мониторинг конвейера

1. Перейдите на вкладку **Мониторинг**. Убедитесь, что здесь отображается запуск конвейера. Создание кластера Spark занимает около 20 минут. 
   
1. Периодически нажимайте **Обновить**, чтобы контролировать состояние выполнения конвейера. 

   ![Вкладка для наблюдения за запуском конвейера с кнопкой "Обновить"](./media/tutorial-transform-data-spark-portal/monitor-tab.png)

1. Чтобы увидеть выполнение действий, связанных с выполнением конвейера, выберите **View Activity Runs** (Просмотр выполнения действий) в столбце **Действия**.

   ![Состояние выполнения конвейера](./media/tutorial-transform-data-spark-portal/pipeline-run-succeeded.png) 

   Чтобы вернуться в режим просмотра запусков конвейера, выберите ссылку **All Pipeline Runs** (Все запуски конвейера).

   ![Просмотр параметра "Выполнения действий"](./media/tutorial-transform-data-spark-portal/activity-runs.png)

## <a name="verify-the-output"></a>Проверка выходных данных
Убедитесь, что целевой файл создается в папке spark/otuputfiles/wordcount для контейнера adftutorial. 

![Расположение выходного файла](./media/tutorial-transform-data-spark-portal/verity-output.png)

В этом файле должны содержаться все слова из входного текстового файла и число, обозначающее количество таких слов в этом файле. Пример: 

```
(u'This', 1)
(u'a', 1)
(u'is', 1)
(u'test', 1)
(u'file', 1)
```

## <a name="next-steps"></a>Дальнейшие действия
Конвейер из этого примера преобразует данные с помощью действия Spark и связанной службы HDInsight по запросу. Вы ознакомились с выполнением следующих задач: 

> [!div class="checklist"]
> * Создали фабрику данных. 
> * Создание конвейера, использующего действие Spark.
> * Активация выполнения конвейера.
> * Осуществили мониторинг выполнения конвейера.

Чтобы узнать, как преобразовать данные, запустив сценарий Hive в кластере Azure HDInsight, который находится в виртуальной сети, ознакомьтесь со следующим руководством: 

> [!div class="nextstepaction"]
> [Преобразование данных в виртуальной сети Azure с помощью действия Hive в фабрике данных Azure](tutorial-transform-data-hive-virtual-network-portal.md).






---
title: Запуск записной книжки Databricks с помощью действия
description: Сведения об использовании действия Databricks Notebook в фабрике данных Azure для выполнения записной книжки Databricks в кластере заданий Databricks.
ms.service: data-factory
ms.topic: tutorial
ms.author: abnarain
author: nabhishek
ms.custom: seo-lt-2019
ms.date: 03/12/2018
ms.openlocfilehash: 20253954035798187f28c18c8207c114d27bc9c6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100374088"
---
# <a name="run-a-databricks-notebook-with-the-databricks-notebook-activity-in-azure-data-factory"></a>Запуск записной книжки Databricks с помощью действия Databricks Notebook в фабрике данных Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве рассматривается создание конвейера фабрики данных Azure, который выполняет записную книжку Databricks в кластере заданий Databricks, на портале Azure. Конвейер также передает параметры фабрики данных Azure в записную книжку Databricks во время выполнения.

В этом руководстве вы выполните следующие шаги:

  - Создали фабрику данных.

  - создадите конвейер, который использует действие Databricks Notebook.

  - Активация выполнения конвейера.

  - Осуществили мониторинг выполнения конвейера.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

Уделите 11 минут вашего времени, чтобы просмотреть следующее видео с кратким обзором и демонстрацией этой функции:

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/ingest-prepare-and-transform-using-azure-databricks-and-data-factory/player]

## <a name="prerequisites"></a>Предварительные требования

  - **Создайте рабочую область Azure Databricks**. [Создайте рабочую область Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal) или используйте существующую. Вы создадите записную книжку Python в рабочей области Azure Databricks. Затем запустите ее и передадите в нее параметры, используя фабрику данных Azure.

## <a name="create-a-data-factory"></a>Создание фабрики данных

1.  Запустите веб-браузер **Microsoft Edge** или **Google Chrome**. Сейчас только эти браузеры поддерживают пользовательский интерфейс фабрики данных.

1.  В меню слева выберите **Создать ресурс**, **Аналитика**, **Фабрика данных**.

    ![Создание фабрики данных](media/transform-data-using-databricks-notebook/new-azure-data-factory-menu.png)

1.  На панели **Новая фабрика данных** введите **ADFTutorialDataFactory** в поле **Имя**.

    Имя фабрики данных Azure должно быть *глобально уникальным*. Если появится следующая ошибка, измените имя фабрики данных. (Например, используйте **\<yourname\>ADFTutorialDataFactory**.) Правила именования для артефактов службы "Фабрика данных" приведены в статье [Фабрика данных Azure — правила именования](./naming-rules.md).

    ![Указание имени для новой фабрики данных](media/transform-data-using-databricks-notebook/new-azure-data-factory.png)

1.  В поле **Подписка** выберите подписку Azure, в рамках которой вы хотите создать фабрику данных.

1.  Для **группы ресурсов** выполните одно из следующих действий:
    
    - Выберите **Использовать существующий** и выберите существующую группу ресурсов из раскрывающегося списка.
    
    - Выберите **Создать** и введите имя группы ресурсов.

    Некоторые действия, описанные в этом руководстве, предполагают, что для группы ресурсов используется имя **ADFTutorialResourceGroup**. Сведения о группах ресурсов см. в статье, где описывается [использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/management/overview.md).

1.  Укажите **V2** при выборе **версии**.

1.  В поле **Расположение** выберите расположение фабрики данных.

    Чтобы получить список регионов Azure, в которых сейчас доступна Фабрика данных, выберите интересующие вас регионы на следующей странице, а затем разверните раздел **Аналитика**, чтобы найти пункт **Фабрика данных**: [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/). Хранилища данных (такие как служба хранилища Azure и база данных SQL Azure) и вычислительные среды (например, HDInsight), используемые фабрикой данных, могут находиться в других регионах.
1.  Нажмите кнопку **создания**.


1.  Когда создание завершится, откроется страница **Фабрика данных**. Выберите элемент **Author & Monitor** (Создание и мониторинг), чтобы открыть на отдельной вкладке приложение пользовательского интерфейса службы "Фабрика данных".

    ![Запуск приложения пользовательского интерфейса фабрики данных](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image4.png)

## <a name="create-linked-services"></a>Создание связанных служб

В этом разделе создается связанная служба Databricks. Эта связанная служба содержит сведения о подключении к кластеру Databricks.

### <a name="create-an-azure-databricks-linked-service"></a>Создание связанной службы Azure Databricks

1.  На странице **Let's get started** (Начало работы) переключитесь на вкладку **Edit** (Редактирование) на левой панели.

    ![Изменение новой связанной службы](media/transform-data-using-databricks-notebook/get-started-page.png)

1.  В нижней части окна выберите **Подключения**, а затем **+ Создать**.
    
    ![Создание подключения](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image6.png)

1.  В окне **New Linked Service** (Новая связанная служба) выберите **Вычисления** \> **Azure Databricks** и щелкните **Продолжить**.
    
    ![Указание связанной службы Databricks](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image7.png)

1.  В окне **New Linked Service** (Новая связанная служба) сделайте следующее:
    
    1.  В качестве **имени** введите **_AzureDatabricks\_LinkedService_**.
    
    1.  Выберите соответствующую **рабочую область Databricks**, которая выполняться в записной книжке.

    1.  В поле **Выберите кластер** выберите **Новый кластер заданий**.
    
    1.  Значение **домена и региона** должно заполниться автоматически.

    1.  В поле **Маркер доступа** укажите маркер, сгенерированный из рабочей области Azure Databricks. Инструкции можно найти [здесь](https://docs.databricks.com/api/latest/authentication.html#generate-token).

    1.  В поле **Версия кластера** выберите **4.2** (Apache Spark 2.3.1, Scala 2.11).

    1.  В поле **Тип узла кластера** выберите **Standard\_D3\_v2** в категории **Общего назначения (HDD)** . 
    
    1.  В поле **Рабочая роль** введите **2**.
    
    1.  Нажмите кнопку **Готово**.

        ![Завершение создания связанной службы](media/transform-data-using-databricks-notebook/new-databricks-linkedservice.png)

## <a name="create-a-pipeline"></a>Создание конвейера

1.  Нажмите кнопку **+** (плюс) и в меню выберите **Pipeline** (Конвейер).

    ![Кнопки для создания конвейера](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image9.png)

1.  Создайте **параметр** для использования в **конвейере**. Позже вы передадите этот параметр в действие Databricks Notebook. В пустом конвейере щелкните вкладку **Параметры**, а затем — **Создать** и присвойте параметру имя **name**.

    ![Создание параметра](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image10.png)

    ![Создание параметра name](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image11.png)

1.  На панели элементов **Действия** разверните узел **Databricks**. Перетащите действие **Notebook** с панели элементов **Действия** в область конструктора конвейера.

    ![Перетаскивание записной книжки в область конструктора](media/transform-data-using-databricks-notebook/new-adf-pipeline.png)

1.  В свойствах для окна действия **Databricks** **Notebook** в нижней части страницы выполните следующие действия:

    а. Откройте вкладку **Azure Databricks**.

    b. Выберите **AzureDatabricks\_LinkedService** (создано ранее).

    c. Откройте вкладку **Параметры**.

    c. Выберите **путь к записной книжке** Databricks. Давайте создадим записную книжку и укажем путь. Чтобы получить путь к записной книжке, выполните следующие действия.

       1. Запуск рабочей области Azure Databricks

       1. В рабочей области выберите команду **создания папки** и присвойте папке имя **adftutorial**.

          ![Создание папки](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image13.png)

       1. [Создайте записную книжку](https://docs.databricks.com/user-guide/notebooks/index.html#creating-a-notebook) (Python) с именем **mynotebook** в папке **adftutorial**, выбрав команду **Создать**.

          ![Создание записной книжки](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image14.png)

          ![Указание свойств новой записной книжки](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image15.png)

       1. В недавно созданной записной книжке mynotebook добавьте следующий код:

           ```
           # Creating widgets for leveraging parameters, and printing the parameters

           dbutils.widgets.text("input&quot;, &quot;&quot;,&quot;")
           y = dbutils.widgets.get("input")
           print ("Param -\'input':")
           print (y)
           ```

           ![Создание мини-приложений для параметров](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image16.png)

       1. В этом случае **путь к записной книжке** — **/adftutorial/mynotebook**.

1.  Вернитесь к **инструменту для создания пользовательского интерфейса фабрики данных**. Перейдите на вкладку **Параметры** в разделе **Действие Notebook1**.

    а.  **Добавьте параметр** в действие записной книжки. Это тот же параметр, который был добавлен ранее в **конвейер**.

       ![Добавление параметра](media/transform-data-using-databricks-notebook/new-adf-parameters.png)

    b.  Присвойте параметру имя **input** и укажите в качестве значения выражение **\@pipeline().parameters.name**.

1.  Чтобы проверить работу конвейера, нажмите кнопку **Проверка** на панели инструментов. Чтобы закрыть окно проверки, нажмите кнопку **\>\>** (стрелка вправо).

    ![Проверка конвейера](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image18.png)

1.  Выберите **Опубликовать все**. Пользовательский интерфейс фабрики данных опубликует сущности (связанные службы и конвейер) в службе фабрики данных Azure.

    ![Публикация новых сущностей фабрики данных](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image19.png)

## <a name="trigger-a-pipeline-run"></a>Активация выполнения конвейера

Выберите **Trigger** (Запустить) на панели инструментов, а затем **Trigger Now** (Запустить сейчас).

![Выбор команды Trigger Now (Запустить сейчас)](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image20.png)

В диалоговом окне **Pipeline Run** (Запуск конвейера) появится запрос на ввод параметра **name**. Укажите **/path/filename** в качестве параметра. Нажмите кнопку **Готово**.

![Указание значения для параметра name](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image21.png)

## <a name="monitor-the-pipeline-run"></a>Мониторинг конвейера

1.  Перейдите на вкладку **Мониторинг**. Убедитесь, что здесь отображается запуск конвейера. Для создания кластера заданий Databricks, где выполняется записная книжка, требуется приблизительно 5–8 минут.

    ![Мониторинг конвейера](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image22.png)

1.  Периодически нажимайте **Обновить**, чтобы контролировать состояние выполнения конвейера.

1.  Чтобы увидеть выполнение действий, связанных с выполнением конвейера, выберите **View Activity Runs** (Просмотр выполнения действий) в столбце **Действия**.

    ![Просмотр выполнений действий](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image23.png)

Чтобы вернуться в режим просмотра запусков конвейера, выберите ссылку **Конвейеры** вверху.

## <a name="verify-the-output"></a>Проверка выходных данных

Войдите в **рабочую область Azure Databricks**, перейдите в раздел **Кластеры**, и вы увидите одно из таких состояний **задания**: *в ожидании, выполняется или завершено*.

![Просмотр кластера заданий и задания](media/transform-data-using-databricks-notebook/databricks-notebook-activity-image24.png)

Щелкнув **имя задания**, можно перейти к дополнительным сведениям. При успешном запуске можно проверить переданные параметры и выходные данные записной книжки Python.

![Просмотр сведений о выполнении и выходных данных](media/transform-data-using-databricks-notebook/databricks-output.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом примере конвейер запускает действие Databricks Notebook и передает в него параметр. Вы ознакомились с выполнением следующих задач:

  - Создали фабрику данных.

  - создавать конвейер, который использует действие Databricks Notebook.

  - Активация выполнения конвейера.

  - Осуществили мониторинг выполнения конвейера.
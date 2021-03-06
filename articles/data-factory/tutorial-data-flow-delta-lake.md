---
title: Дельта Lake ETL с потоками данных
description: В этом учебнике приводятся пошаговые инструкции по использованию потоков данных для преобразования и анализа данных в Delta Lake.
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2021
ms.date: 02/09/2021
ms.openlocfilehash: cb8d44353e826df14ed3baab2c4ca66ffed4a569
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100418113"
---
# <a name="transform-data-in-delta-lake-using-mapping-data-flows"></a>Преобразование данных в Дельта Lake с помощью сопоставления потоков данных

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Если вы еще не работали с фабрикой данных Azure, ознакомьтесь со статьей [Введение в фабрику данных Azure](introduction.md).

В этом руководстве вы будете использовать холст потока данных для создания потоков данных, позволяющих анализировать и преобразовывать данные в Azure Data Lake Storage (ADLS) Gen2 и сохранять их в разностной.

## <a name="prerequisites"></a>Предварительные требования
* **Подписка Azure**. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/) Azure, прежде чем начинать работу.
* **Учетная запись хранения Azure**. Хранилище ADLS используется в качестве хранилища данных *источника* и *приемника* . Если у вас нет учетной записи хранения, создайте ее, следуя действиям в [этом разделе](../storage/common/storage-account-create.md).

Файл, который мы преобразовывать в этом учебнике, MoviesDB.csv, который можно найти [здесь](https://github.com/kromerm/adfdataflowdocs/blob/master/sampledata/moviesDB2.csv). Чтобы получить файл из GitHub, скопируйте его содержимое в текстовый редактор по своему усмотрению, чтобы сохранить его локально в виде CSV-файла. Сведения о передаче файла в учетную запись хранения см. [в разделе Отправка больших двоичных объектов с помощью портал Azure](../storage/blobs/storage-quickstart-blobs-portal.md). Примеры будут ссылаться на контейнер с именем Sample-Data.

## <a name="create-a-data-factory"></a>Создание фабрики данных

На этом шаге вы создадите фабрику данных и откроете интерфейс взаимодействия фабрики данных, чтобы создать конвейер в фабрике данных.

1. Откройте **Microsoft Edge** или **Google Chrome**. В настоящее время пользовательский интерфейс фабрики данных поддерживается только в веб-браузерах Microsoft ребр и Google Chrome.
1. В меню слева выберите **создать ресурс**  >  **Интеграция**  >  **фабрики данных** .
1. На странице **Новая фабрика данных** в поле **имя** введите **ADFTutorialDataFactory** .
1. Выберите **подписку** Azure, в рамках которой вы хотите создать фабрику данных.
1. Для **группы ресурсов** выполните одно из следующих действий:

    а. Выберите **Использовать существующую** и укажите существующую группу ресурсов в раскрывающемся списке.

    b. Выберите **Создать новую** и укажите имя группы ресурсов. 
         
    Сведения о группах ресурсов см. в статье [Общие сведения об Azure Resource Manager](../azure-resource-manager/management/overview.md). 
1. В качестве **версии** выберите **V2**.
1. В поле **Расположение** выберите расположение фабрики данных. В раскрывающемся списке отображаются только поддерживаемые расположения. Хранилища данных (например, служба хранилища Azure и база данных SQL) и расчеты (например, Azure HDInsight), используемые фабрикой данных, могут находиться в других регионах.
1. Нажмите кнопку **создания**.
1. После завершения создания вы увидите уведомление в центре уведомлений. Нажмите кнопку **Перейти к ресурсу**, чтобы открыть страницу фабрики данных.
1. Выберите **Создание и мониторинг**, чтобы запустить на отдельной вкладке пользовательский интерфейс фабрики данных.

## <a name="create-a-pipeline-with-a-data-flow-activity"></a>Создание конвейера с действием потока данных

На этом шаге вы создадите конвейер, содержащий действие потока данных.

1. На странице **Начало работы** выберите **Create pipeline** (Создать конвейер).

   ![Создание конвейера](./media/doc-common-process/get-started-page.png)

1. На вкладке **Общие** для конвейера введите **Делталаке** в поле **имя** конвейера.
1. На верхней панели фабрики подвиньте ползунок **Отладка потока данных** на. Режим отладки позволяет выполнять интерактивное тестирование логики преобразования в динамическом кластере Spark. Подготовка кластеров потоков данных занимает 5-7 минут, и пользователям рекомендуется включить отладку первыми, если планируется разработка потока данных. Дополнительные сведения см. в статье [Режим отладки](concepts-data-flow-debug-mode.md).

    ![Действие потока данных](media/tutorial-data-flow/dataflow1.png)
1. В области **действия** разверните элемент Гармошка **Перемещение и преобразование** . Перетащите действие **поток данных** из панели на холст конвейера.

    ![Снимок экрана, на котором показан холст конвейера, на котором можно удалить действие потока данных.](media/tutorial-data-flow/activity1.png)
1. Во всплывающем окне **Добавление потока данных** выберите **создать новый поток данных** и назовите **делталаке** потока данных. По завершении нажмите кнопку Готово.

    ![Снимок экрана, на котором показано, где вы назначите имя потока данных при создании нового потока данных.](media/tutorial-data-flow/activity2.png)

## <a name="build-transformation-logic-in-the-data-flow-canvas"></a>Логика преобразования "сборка" на холсте потока данных

В этом руководстве вы создадите два потока данных. Поток данных кулак является простым источником для приемника для создания новой разностной версии Lake из CSV-файла с фильмами. Наконец, вы создадите этот поток ниже для обновления данных в Delta Lake.

![Окончательный поток](media/data-flow/data-flow-tutorial-6.png "Окончательный поток")

### <a name="tutorial-objectives"></a>Цели учебника

1. Возьмем источник набора данных Мовиесксв выше, образуя новый разностный Lake из него
1. Создание логики с обновленными оценками для фильмов 1988 на "1"
1. Удалить все фильмы из 1950
1. Вставка новых фильмов для 2021 путем копирования фильмов из 1960

### <a name="start-from-a-blank-data-flow-canvas"></a>Запуск с пустого холста потока данных

1. Щелкните преобразование «источник»
1. Щелкните Создать рядом с набором данных в нижней панели 1 Создание новой связанной службы для ADLS 2-го поколения
1. Выбор текста с разделителями для типа набора данных
1. Назовите набор данных «Мовиесксв». 
1. Укажите файл Мовиесксв, отправленный в хранилище выше
1. Установите в качестве разделителя запятую и включите заголовок в первую строку 
1. Перейдите на вкладку проекция источника и нажмите кнопку "обнаружить типы данных".
1. После настройки проекции можно продолжить 
1. Добавление преобразования приемника
1. Дельта — это встроенный тип набора данных. Необходимо указать учетную запись хранения ADLS 2-го поколения.
   
   ![Встроенный набор данных](media/data-flow/data-flow-tutorial-5.png "Встроенный набор данных")

1. Выберите имя папки в контейнере хранилища, в котором ADF-файл должен создать разностную папку Lake
1. Вернитесь в конструктор конвейера и щелкните Отладка, чтобы выполнить конвейер в режиме отладки с использованием только этого действия потока данных на холсте. При этом создается новый разностный Lake ADLS 2-го поколения.
1. Из ресурсов фабрики щелкните Создать > поток данных. 
1. Снова используйте Мовиесксв в качестве источника и нажмите кнопку "обнаружить типы данных".
1. Добавление преобразования «фильтр» к преобразованию «источник» в графе
1. Допускаются только строки фильмов, которые соответствуют трем годам, с которыми вы будете работать (1950, 1988 и 1960).
1. После добавления преобразования «Производный столбец» в преобразование «фильтр» необходимо обновить оценки для каждого фильма 1988 в «1».
1. В том же производном столбце создайте фильмы для 2021, выполнив существующий год и изменив год на 2021. Давайте выберем 1960.
1. Вот как будут выглядеть три производных столбца

   ![Производный столбец](media/data-flow/data-flow-tutorial-2.png "Производный столбец")
   
1. ```Update, insert, delete, and upsert``` политики создаются в преобразовании «изменение строк». Добавление преобразования «изменение строки» после производного столбца.
1. Политики изменения строк должны выглядеть следующим образом.

   ![Изменение строк](media/data-flow/data-flow-tutorial-3.png "Изменение строк")
   
1. Теперь, когда вы установили правильную политику для каждого типа ALTER Row, убедитесь, что для преобразования приемника заданы правильные правила обновления.

   ![Приемник](media/data-flow/data-flow-tutorial-4.png "Приемник")
   
1. Здесь мы используем разностный приемник данных в ADLS 2-го поколения Data Lake и позволяя выполнять операции вставки, обновления и удаления. 
1. Обратите внимание, что ключевые столбцы представляют собой составной ключ, состоящие из столбца первичного ключевого фильма и года. Это связано с тем, что мы создали поддельные фильмы 2021, скопировав 1960 строк. Это позволяет избежать конфликтов при поиске существующих строк, обеспечивая уникальность.

### <a name="download-completed-sample"></a>Загрузка завершенного примера
[Ниже приведен пример решения для разностного конвейера с потоком данных для обновления или удаления строк в Lake:](https://github.com/kromerm/adfdataflowdocs/blob/master/sampledata/DeltaPipeline.zip)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [языке выражений потока данных](data-flow-expression-functions.md).

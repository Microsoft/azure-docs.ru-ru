---
title: Преобразование с помощью Azure Databricks
description: Сведения об использовании шаблона решения для преобразования данных с помощью записной книжки Databricks в Фабрике данных Azure.
ms.author: abnarain
author: nabhishek
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 04/27/2020
ms.openlocfilehash: ee663423071458605f37f07293693dbc91f592bb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100362120"
---
# <a name="transformation-with-azure-databricks"></a>Преобразование с помощью Azure Databricks

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этом руководстве вы создадите сквозной конвейер, содержащий **проверку**, **копирование данных** и действия **записной книжки** в фабрике данных Azure.

- **Проверка** гарантирует готовность исходного набора данных к выпуску перед запуском задания копирования и аналитики.

- **Копирование данных** дублирует исходный набор данных в хранилище приемников, которое монтируется как DBFS в записной книжке Azure Databricks. Таким образом, набор данных может быть напрямую использован Spark.

- **Записная книжка** активирует Блокнот, который преобразует набор данных. Он также добавляет набор данных в обработанную папку или Azure Azure синапсе Analytics.

Для простоты шаблон в этом учебнике не создает запланированный триггер. При необходимости его можно добавить.

![Схема конвейера](media/solution-template-Databricks-notebook/pipeline-example.png)

## <a name="prerequisites"></a>Предварительные условия

- Учетная запись хранилища BLOB-объектов Azure с контейнером, который вызывается `sinkdata` для использования в качестве приемника.

  Запишите имя учетной записи хранения, имя контейнера и ключ доступа. Эти значения понадобятся позже в шаблоне.

- Рабочая область Azure Databricks.

## <a name="import-a-notebook-for-transformation"></a>Импорт записной книжки для преобразования

Чтобы импортировать записную книжку **преобразования** в рабочую область "кирпичы", сделайте следующее:

1. Войдите в рабочую область Azure Databricks и выберите **Импорт**.
       ![Команда меню для импорта рабочей области ](media/solution-template-Databricks-notebook/import-notebook.png) путь к рабочей области может отличаться от отображаемого, но его следует запомнить для последующего использования.
1. Выберите **Импорт из: URL-адрес**. В текстовом поле введите `https://adflabstaging1.blob.core.windows.net/share/Transformations.html` .

   ![Параметры для импорта записной книжки](media/solution-template-Databricks-notebook/import-from-url.png)

1. Теперь изменим в записную книжку **Преобразование** сведения о подключении к хранилищу.

   В импортированной записной книжке перейдите к **команде 5** , как показано в следующем фрагменте кода.

   - Замените `<storage name>` и `<access key>` собственными сведениями о подключении к хранилищу.
   - Используйте учетную запись хранения с `sinkdata` контейнером.

    ```python
    # Supply storageName and accessKey values  
    storageName = "<storage name>"  
    accessKey = "<access key>"  

    try:  
      dbutils.fs.mount(  
        source = "wasbs://sinkdata\@"+storageName+".blob.core.windows.net/",  
        mount_point = "/mnt/Data Factorydata",  
        extra_configs = {"fs.azure.account.key."+storageName+".blob.core.windows.net": accessKey})  

    except Exception as e:  
      # The error message has a long stack track. This code tries to print just the relevant line indicating what failed.

    import re
    result = re.findall(r"\^\s\*Caused by:\s*\S+:\s\*(.*)\$", e.message, flags=re.MULTILINE)
    if result:
      print result[-1] \# Print only the relevant error message
    else:  
      print e \# Otherwise print the whole stack trace.  
    ```

1. Создайте **маркер доступа Databricks**, чтобы Фабрика данных могла получать доступ к Databricks.
   1. В рабочей области "кирпичы" выберите значок профиля пользователя в правом верхнем углу.
   1. Выберите **Параметры пользователя**.
    ![Команда меню для параметров пользователя](media/solution-template-Databricks-notebook/user-setting.png)
   1. Выберите **создать новый токен** на вкладке **маркеры доступа** .
   1. Выберите **Создать**.

    ![Кнопка "создать"](media/solution-template-Databricks-notebook/generate-new-token.png)

   *Сохраните маркер доступа* для последующего использования при создании связанной службы "кирпичы". Маркер доступа выглядит примерно так `dapi32db32cbb4w6eee18b7d87e45exxxxxx` .

## <a name="how-to-use-this-template"></a>Как использовать этот шаблон

1. Перейдите к **преобразованию с помощью шаблона Azure Databricks** и создайте новые связанные службы для следующих соединений.

   ![Параметры подключений](media/solution-template-Databricks-notebook/connections-preview.png)

    - **Подключение** к исходному BLOB-объекту — для доступа к исходным данным.

       В этом упражнении вы можете использовать общедоступное хранилище больших двоичных объектов, содержащее исходные файлы. Сослаться на следующий снимок экрана для конфигурации. Используйте следующий **URL-адрес SAS** для подключения к исходному хранилищу (доступ только для чтения):

       `https://storagewithdata.blob.core.windows.net/data?sv=2018-03-28&si=read%20and%20list&sr=c&sig=PuyyS6%2FKdB2JxcZN0kPlmHSBlD8uIKyzhBWmWzznkBw%3D`

        ![Параметры метода проверки подлинности и URL-адреса SAS](media/solution-template-Databricks-notebook/source-blob-connection.png)

    - **Подключение к целевому BLOB-объекту** — для хранения скопированных данных.

       В окне **Новая связанная служба** выберите BLOB-объект хранилища приемника.

       ![BLOB-объект хранилища приемника в качестве новой связанной службы](media/solution-template-Databricks-notebook/destination-blob-connection.png)

    - **Azure Databricks** — для подключения к кластеру кирпичей.

        Создайте связанную службу "кирпичы" с помощью созданного ранее ключа доступа. Вы можете выбрать *Интерактивный кластер* , если он есть. В этом примере используется параметр **создания кластера заданий** .

        ![Параметры для подключения к кластеру](media/solution-template-Databricks-notebook/databricks-connection.png)

1. Щелкните **Использовать этот шаблон**. Вы увидите созданный конвейер.

    ![Создание конвейера](media/solution-template-Databricks-notebook/new-pipeline.png)

## <a name="pipeline-introduction-and-configuration"></a>Введение и Настройка конвейера

В новом конвейере большинство параметров настраиваются автоматически со значениями по умолчанию. Проверьте конфигурации конвейера и внесите необходимые изменения.

1. В **флаге доступности** действия **проверки** убедитесь, что для параметра исходный **набор данных** задано значение `SourceAvailabilityDataset` , созданное ранее.

   ![Значение исходного набора данных](media/solution-template-Databricks-notebook/validation-settings.png)

1. В файле действия **копирование данных** в **BLOB-объект** проверьте вкладки **источник** и **приемник** . При необходимости измените параметры.

   - Вкладка источника на вкладке « **источник** » ![](media/solution-template-Databricks-notebook/copy-source-settings.png)

   -  ![ Вкладка приемника вкладки приемника](media/solution-template-Databricks-notebook/copy-sink-settings.png)

1. В **преобразовании**«действие **записной книжки** » проверьте и при необходимости обновите пути и параметры.

   Для **связанной службы "кирпичные** данные" необходимо предварительно заполнить значение из предыдущего шага, как показано: ![ заполненное значение для связанной службы "кирпичные данные"](media/solution-template-Databricks-notebook/notebook-activity.png)

   Чтобы проверить параметры **записной книжки** , сделайте следующее:
  
    1. Перейдите на вкладку **Параметры** . В поле **путь к записной книжке** Проверьте правильность пути по умолчанию. Возможно, потребуется просмотреть и выбрать правильный путь к записной книжке.

       ![Путь к записной книжке](media/solution-template-Databricks-notebook/notebook-settings.png)

    1. Разверните селектор **базовых параметров** и убедитесь, что параметры соответствуют приведенным на снимке экрана ниже. Эти параметры передаются в записную книжку "кирпичы" из фабрики данных.

       ![Базовые параметры](media/solution-template-Databricks-notebook/base-parameters.png)

1. Убедитесь, что **Параметры конвейера** соответствуют приведенным на следующем снимке экрана: ![ Параметры конвейера.](media/solution-template-Databricks-notebook/pipeline-parameters.png)

1. Подключитесь к наборам данных.

    >[!NOTE]
    >В приведенных ниже наборах данных путь к файлу был автоматически указан в шаблоне. Если требуются какие – либо изменения, убедитесь, что вы указали путь для **контейнера** и **каталога** в случае любой ошибки подключения.

   - **Саурцеаваилабилитидатасет** — проверка доступности исходных данных.

     ![Варианты выбора для связанной службы и пути к файлу для Саурцеаваилабилитидатасет](media/solution-template-Databricks-notebook/source-availability-dataset.png)

   - **Саурцефилесдатасет** — для доступа к исходным данным.

       ![Варианты выбора для связанной службы и пути к файлу для Саурцефилесдатасет](media/solution-template-Databricks-notebook/source-file-dataset.png)

   - **Дестинатионфилесдатасет** — копирование данных в целевое расположение приемника. Используйте следующие значения.

     - **Связанная служба**  -  `sinkBlob_LS` , созданная на предыдущем шаге.

     - **Путь к файлу**  -  `sinkdata/staged_sink` .

       ![Варианты выбора для связанной службы и пути к файлу для Дестинатионфилесдатасет](media/solution-template-Databricks-notebook/destination-dataset.png)

1. Выберите **Отладка** , чтобы запустить конвейер. Для получения более подробных журналов Spark можно найти ссылку на журналы кирпичей.

    ![Ссылка на журналы данных модуля](media/solution-template-Databricks-notebook/pipeline-run-output.png)

    Файл данных также можно проверить с помощью Обозреватель службы хранилища Azure.

    > [!NOTE]
    > Для сопоставления с запусками конвейера фабрики данных этот пример добавляет идентификатор запуска конвейера из фабрики данных в выходную папку. Это помогает отследить файлы, создаваемые каждым запуском.
    > ![Добавленный идентификатор запуска конвейера](media/solution-template-Databricks-notebook/verify-data-files.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения о службе фабрики данных Azure, службе интеграции данных в облаке](introduction.md)

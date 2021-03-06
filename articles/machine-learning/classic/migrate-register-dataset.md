---
title: 'ML Studio (классическая модель): миграция на Машинное обучение Azure набор данных'
description: Перестроение наборов данных Studio (классическая модель) в конструкторе Машинное обучение Azure
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: xiaoharper
ms.author: zhanxia
ms.date: 02/04/2021
ms.openlocfilehash: 4c04dd5a2b41b3db54b20c9e514767453951cc35
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103565300"
---
# <a name="migrate-a-studio-classic-dataset-to-azure-machine-learning"></a>Миграция набора данных Studio (классическая модель) в Машинное обучение Azure

Из этой статьи вы узнаете, как перенести набор данных Studio (классическая модель) в Машинное обучение Azure. Дополнительные сведения о миграции из Studio (классической) см. [в статье Обзор миграции](migrate-overview.md).

Существует три варианта переноса набора данных в Машинное обучение Azure. Ознакомьтесь с каждым из разделов, чтобы определить, какой вариант лучше подходит для вашего сценария.


|Где находятся данные? | Вариант миграции  |
|---------|---------|
|В студии (классическая модель)     |  Вариант 1. [Скачайте набор данных из Studio (классическая модель) и отправьте его в машинное обучение Azure](#download-the-dataset-from-studio-classic).      |
|Облачное хранилище     | Вариант 2. [Регистрация набора данных из облачного источника](#import-data-from-cloud-sources). <br><br>  Вариант 3. [Использование модуля "Импорт данных" для получения данных из облачного источника](#import-data-from-cloud-sources).        |

> [!NOTE]
> Машинное обучение Azure также поддерживает [рабочие процессы](../how-to-create-register-datasets.md) , создающие код, для создания наборов данных и управления ими. 

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Рабочая область машинного обучения Azure. [Создайте рабочую область Машинного обучения Azure](../how-to-manage-workspace.md#create-a-workspace).
- Набор данных Studio (классический) для миграции.


## <a name="download-the-dataset-from-studio-classic"></a>Загрузка набора данных из студии (классическая модель)

Самый простой способ переноса набора данных Studio (классической) в Машинное обучение Azure — скачать набор данных и зарегистрировать его в Машинное обучение Azure. При этом создается новая копия набора данных, которая загружается в хранилище Машинное обучение Azure.

Вы можете скачать следующие типы наборов данных (классические) Studio напрямую.

* Обычный текст (TXT)
* Текст с разделителями-запятыми с заголовком (CSV) или без заголовка (NH.CSV)
* Текст с разделителями-табуляциями с заголовком (TSV) или без заголовка (NH.TSV)
* Файл Excel
* ZIP-файл (ZIP)

Для непосредственного скачивания наборов данных:
1. Перейдите в рабочую область Studio (классическая модель) ( [https://studio.azureml.net](https://studio.azureml.net) ).
1. На панели навигации слева перейдите на вкладку **наборы данных** .
1. Выберите наборы данных, которые требуется скачать.
1. На нижней панели действий выберите **скачать**.

    ![Снимок экрана, показывающий, как скачать набор данных в Studio (классическая модель)](./media/migrate-register-dataset/download-dataset.png)

Для следующих типов данных необходимо использовать модуль **преобразовать в CSV** для скачивания наборов данных.

* Данные SVMLight (. SVMLight) 
* Данные в формате файла связи атрибутов (ARFF) (. ARFF) 
* Файл объекта или рабочей области R (RData)
* Тип набора данных (. Data). Тип набора данных — это внутренний тип данных Studio (классический) для выходных данных модуля.

Чтобы преобразовать набор данных в CSV-файл и скачать результаты:

1. Перейдите в рабочую область Studio (классическая модель) ( [https://studio.azureml.net](https://studio.azureml.net) ).
1. Создайте новый эксперимент.
1. Перетащите набор данных, который необходимо загрузить на холст.
1. Добавьте модуль **Convert в CSV** -файл.
1. Подключите входной порт **Convert to CSV** к порту вывода набора данных.
1. Запустите эксперимент.
1. Щелкните модуль **преобразовать в CSV** правой кнопкой мыши.
1. Выберите **загрузить набор данных результатов**  >  .

    ![Снимок экрана, показывающий, как настроить конвейер преобразования в CSV-файл](./media/migrate-register-dataset/csv-download-dataset.png)

### <a name="upload-your-dataset-to-azure-machine-learning"></a>Передача набора данных в Машинное обучение Azure

После загрузки файла данных можно зарегистрировать набор данных в Машинное обучение Azure:

1. Перейдите в Машинное обучение Azure Studio ([ml.Azure.com](https://ml.azure.com)).
1. В области навигации слева перейдите на вкладку **наборы данных** .
1. Выберите **создать набор данных**  >  **из локальных файлов**.
    ![Снимок экрана, на котором показана вкладка "наборы данных" и кнопка для создания локального файла](./media/migrate-register-dataset/register-dataset.png)
1. Введите название и описание.
1. В качестве **типа набора данных** выберите **табличный**.

    > [!NOTE]
    > ZIP-файлы также можно отправлять в виде наборов данных. Чтобы передать ZIP-файл, выберите **файл** для **типа набора данных**.

1. В поле " **хранилище данных и выбор файлов**" выберите хранилище, в которое нужно передать файл DataSet.

    По умолчанию Машинное обучение Azure сохраняет набор данных в рабочей области по умолчанию Blobstore. Дополнительные сведения о хранилищах данных см. [в статье подключение к службам хранилища](../how-to-access-data.md).

1. Задайте параметры анализа данных и схему для набора данных. Затем подтвердите параметры.

## <a name="import-data-from-cloud-sources"></a>Импорт данных из облачных источников

Если ваши данные уже находятся в облачной службе хранилища и вы хотите хранить их в собственном расположении. Можно использовать один из следующих вариантов.

|Метод приема|Описание|
|---| --- |
|Регистрация набора данных Машинное обучение Azure|Прием данных из локальных и сетевых источников данных (BLOB-объектов, ADLS 1-го поколения, ADLS 2-го поколения, общего файлового ресурса, SQL DB). <br><br>Создает ссылку на источник данных, который отложенно вычисляется во время выполнения. Используйте этот параметр, если вы постоянно обращаетесь к этому набору данных и хотите включить расширенные функции данных, такие как управление версиями и мониторинг данных.
|Модуль импорта данных|Прием данных из сетевых источников данных (BLOB-объектов, ADLS 1-го поколения, ADLS 2-го поколения, общего файлового ресурса, SQL DB). <br><br> Набор данных импортируется только в текущий запуск конвейера конструктора.


>[!Note]
> Пользователи Studio (классические) должны заметить, что следующие облачные источники изначально не поддерживаются в Машинное обучение Azure:
> - Запрос Hive
> - таблице Azure
> - Azure Cosmos DB
> - Локальная база данных SQL
>
> Мы рекомендуем пользователям переносить свои данные в Поддерживаемые службы хранилища с помощью фабрики данных Azure.  

### <a name="register-an-azure-machine-learning-dataset"></a>Регистрация набора данных Машинное обучение Azure

Чтобы зарегистрировать набор данных для Машинное обучение Azure из облачной службы, выполните следующие действия. 

1. [Создайте хранилище данных](../how-to-connect-data-ui.md#create-datastores), которое связывает службу облачного хранилища с рабочей областью машинное обучение Azure. 

1. [Зарегистрируйте набор данных](../how-to-connect-data-ui.md#create-datasets). Если выполняется миграция набора данных Studio (классическая модель), выберите параметр **табличный** набор данных.

После регистрации набора данных в Машинное обучение Azure его можно использовать в конструкторе:
 
1. Создайте новый черновик конвейера конструктора.
1. В палитре модулей слева разверните раздел **наборы данных** .
1. Перетащите зарегистрированный набор данных на холст. 

### <a name="use-the-import-data-module"></a>Использование модуля "Импорт данных"

Чтобы импортировать данные непосредственно в конвейер конструктора, выполните следующие действия.

1. [Создайте хранилище данных](https://github.com/MicrosoftDocs/azure-docs-pr/blob/master/articles/machine-learning/how-to-connect-data-ui.md#create-datastores), которое связывает службу облачного хранилища с рабочей областью машинное обучение Azure. 

После создания хранилища данных можно использовать модуль [**Импорт данных**](../algorithm-module-reference/import-data.md) в конструкторе, чтобы принимать данные из него:

1. Создайте новый черновик конвейера конструктора.
1. В палитре модулей слева найдите модуль **Импорт данных** и перетащите его на холст.
1. Выберите модуль **Импорт данных** и используйте параметры на панели справа, чтобы настроить источник данных.

## <a name="next-steps"></a>Следующие шаги

Из этой статьи вы узнали, как перенести набор данных Studio (классический) в Машинное обучение Azure. Следующим шагом является [Перестроение конвейера обучения студии (классический)](migrate-rebuild-experiment.md).


См. Другие статьи в серии миграции Studio (классическая модель):

1. [Обзор миграции](migrate-overview.md).
1. **Перенесите наборы данных**.
1. [Перестройте конвейер обучения Studio (классический)](migrate-rebuild-experiment.md).
1. [Перестройте веб-службу Studio (классическая)](migrate-rebuild-web-service.md).
1. [Интегрируйте веб-службу машинное обучение Azure с клиентскими приложениями](migrate-rebuild-integrate-with-client-app.md).
1. Выполните [миграцию выполнения скрипта R](migrate-execute-r-script.md).

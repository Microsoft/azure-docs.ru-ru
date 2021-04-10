---
title: Руководство по Использование мастера оценки моделей машинного обучения для выделенных пулов SQL
description: Руководство. Использование мастера оценки моделей машинного обучения для дополнения данных в выделенных пулах SQL.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 09/25/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: d8db9257ad6eed98b39cd2c9a52351f013453365
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98935223"
---
# <a name="tutorial-machine-learning-model-scoring-wizard-preview-for-dedicated-sql-pools"></a>Руководство по Использование мастера оценки моделей машинного обучения (предварительная версия) для выделенных пулов SQL

Узнайте, как легко дополнить данные в выделенных пулах SQL с помощью прогнозных моделей машинного обучения. Модели, создаваемые вашими данными, теперь легко доступны специалистам по анализу данных для прогнозной аналитики. Специалист по данным в Azure Synapse Analytics может просто выбрать модель в реестре моделей Машинного обучения Azure для развертывания в пулах Azure Synapse SQL и запустить прогнозирование, чтобы обогатить данные.

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
> - Обучить прогнозирующую модель машинного обучения и внести модель в реестр моделей Машинного обучения Azure.
> - Использовать мастер оценки SQL для запуска прогнозов в выделенном пуле SQL.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

- [Рабочая область Azure Synapse Analytics](../get-started-create-workspace.md) с учетной записью хранения Azure Data Lake Storage 2-го поколения, настроенной в качестве хранилища по умолчанию. При работе с файловой системой Data Lake Storage 2-го поколения вам нужно иметь роль *Участник для данных BLOB-объектов хранилища*.
- Выделенный пул SQL в рабочей области Azure Synapse Analytics. Дополнительные сведения о см. в статье [Создание выделенного пула SQL](../quickstart-create-sql-pool-studio.md).
- Связанная служба Машинного обучения Azure в рабочей области Azure Synapse Analytics. Дополнительные сведения см. в разделе [Создание связанной службы Машинного обучения Azure в Azure Synapse](quickstart-integrate-azure-machine-learning.md).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="train-a-model-in-azure-machine-learning"></a>Обучение модели в Машинном обучении Azure

Прежде чем начать, убедитесь, что используется версия sklearn 0.20.3.

Перед выполнением всех ячеек в записной книжке проверьте, выполняется ли вычислительный экземпляр.

![Снимок экрана: проверка вычислений Машинного обучения Azure.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-train-00b.png)

1. Перейдите в рабочую область Машинного обучения Azure.

1. Загрузите [Predict NYC Taxi Tips.ipynb](https://go.microsoft.com/fwlink/?linkid=2144301).

1. Откройте рабочую область Машинного обучения Azure в [Студии Машинного обучения Azure](https://ml.azure.com).

1. Выберите **Записные книжки** > **Отправить файлы**. Затем выберите скачанный файл **Predict NYC Taxi Tips.ipynb** и отправьте его.
   ![Снимок экрана: кнопка для отправки файла.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-train-00a.png)

1. После отправки и открытия записной книжки выберите **Выполнить все ячейки**.

   Выполнение одной из ячеек может завершиться ошибкой, после чего вы получите запрос на аутентификацию в Azure. Проследите за этим в выходных данных ячейки и выполните аутентификацию в браузере, перейдя по ссылке и введя код. Затем повторно запустите записную книжку.

1. Записная книжка обучит модель ONNX и зарегистрирует ее в MLflow. Перейдите в раздел **Модели**, чтобы проверить, правильно ли зарегистрирована новая модель.
   ![Снимок экрана: модель в реестре.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-train-00c.png)

1. При запуске записной книжки тестовые данные также экспортируются в CSV-файл. Загрузите CSV-файл в локальную систему. Позже вы импортируете CSV-файл в выделенный пул SQL и будете использовать эти данные для тестирования модели.

   CSV-файл создается в той же папке, что и файл записной книжки. Нажмите кнопку **Обновить** в проводнике, если файл не отображается сразу.

   ![Снимок экрана: CSV-файл.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-train-00d.png)

## <a name="launch-predictions-with-the-sql-scoring-wizard"></a>Запуск прогнозирований с помощью мастера оценки SQL

1. Откройте рабочую область Azure Synapse в Synapse Studio.

1. Выберите **Данные** > **Связанные** > **Учетные записи хранения**. Передайте `test_data.csv` в учетную запись хранения по умолчанию.

   ![Снимок экрана: варианты для отправки данных.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00a.png)

1. Перейдите в раздел **Разработка** > **Скрипты SQL**. Создайте новый скрипт SQL для загрузки `test_data.csv` в выделенный пул SQL.

   > [!NOTE]
   > Обновите URL-адрес файла в этом скрипте перед его запуском.

   ```SQL
   IF NOT EXISTS (SELECT * FROM sys.objects WHERE NAME = 'nyc_taxi' AND TYPE = 'U')
   CREATE TABLE dbo.nyc_taxi
   (
       tipped int,
       fareAmount float,
       paymentType int,
       passengerCount int,
       tripDistance float,
       tripTimeSecs bigint,
       pickupTimeBin nvarchar(30)
   )
   WITH
   (
       DISTRIBUTION = ROUND_ROBIN,
       CLUSTERED COLUMNSTORE INDEX
   )
   GO
   
   COPY INTO dbo.nyc_taxi
   (tipped 1, fareAmount 2, paymentType 3, passengerCount 4, tripDistance 5, tripTimeSecs 6, pickupTimeBin 7)
   FROM '<URL to linked storage account>/test_data.csv'
   WITH
   (
       FILE_TYPE = 'CSV',
       ROWTERMINATOR='0x0A',
       FIELDQUOTE = '"',
       FIELDTERMINATOR = ',',
       FIRSTROW = 2
   )
   GO
   
   SELECT TOP 100 * FROM nyc_taxi
   GO
   ```

   ![Загрузка данных в выделенный пул SQL](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00b.png)

1. Перейдите в раздел **Данные** > **Рабочая область**. Откройте мастер оценки SQL, щелкнув правой кнопкой мыши таблицу выделенного пула SQL. Выберите **Машинное обучение** > **Enrich with existing model** (Дополнить с помощью существующей модели).

   > [!NOTE]
   > Параметр машинного обучения не отображается, если для Машинного обучения Azure не создана связанная служба (см. раздел [Предварительные требования](#prerequisites) в начале этого руководства).

   ![Снимок экрана: параметр Машинного обучения.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00c.png)

1. Выберите связанную рабочую область Машинного обучения Azure в раскрывающемся списке. При этом будет загружен список моделей машинного обучения из реестра моделей выбранной рабочей области Машинного обучения Azure. В настоящее время поддерживаются только модели ONNX, поэтому на этом шаге будут отображаться только модели ONNX.

1. Выберите только что обученную модель и щелкните **Продолжить**.

   ![Снимок экрана: выбор модели Машинного обучения Azure.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00d.png)

1. Сопоставьте столбцы таблицы с входными данными модели и укажите выходные данные модели. Если модель сохранена в формате MLflow и подпись модели заполнена, сопоставление будет выполнено автоматически с использованием логики на основе сходства имен. Интерфейс также поддерживает сопоставление вручную.

   Выберите **Continue** (Продолжить).

   ![Снимок экрана: сопоставление таблицы и модели.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00e.png)

1. Созданный код T-SQL упаковывается внутри хранимой процедуры. Именно поэтому необходимо указать имя хранимой процедуры. Двоичные данные модели, включая метаданные (версия, описание и другая информация), будут физически скопированы из Машинного обучения Azure в таблицу выделенного пула SQL. Следовательно, необходимо указать, в какой таблице будет сохраняться модель. 

   Вы можете выбрать **существующую таблицу** или **создать новую**. По завершении выберите **Развернуть модель и открыть скрипт**, чтобы развернуть модель и создать T-SQL-скрипт прогнозирования.

   ![Снимок экрана: варианты создания хранимой процедуры.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00f.png)

1. После создания скрипта нажмите кнопку **Запустить**, чтобы выполнить оценку и получить прогнозы.

   ![Снимок экрана: выполнение оценки и получение прогнозов.](media/tutorial-sql-pool-model-scoring-wizard/tutorial-sql-scoring-wizard-00g.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Краткое руководство. Создание новой связанной службы Машинного обучения Azure в Azure Synapse](quickstart-integrate-azure-machine-learning.md)
- [Возможности машинного обучения в Azure Synapse Analytics](what-is-machine-learning.md)

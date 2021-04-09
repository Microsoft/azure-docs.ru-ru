---
title: Руководство по обнаружению аномалий с помощью Cognitive Services
description: Узнайте, как использовать Cognitive Services для обнаружения аномалий в Azure Synapse Analytics.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: c54300bf37f6f4526c525b1502d902e5f4336ed7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98943501"
---
# <a name="tutorial-anomaly-detection-with-cognitive-services-preview"></a>Руководство по обнаружению аномалий с помощью Cognitive Services (предварительная версия)

Из этого учебника вы узнаете, как обогатить данные в Azure Synapse с помощью [Azure Cognitive Services](../../cognitive-services/index.yml). Для поиска аномалий вы будете использовать [Детектор аномалий](../../cognitive-services/anomaly-detector/index.yml). Чтобы обнаруживать аномалии, пользователь в Azure Synapse может просто выбрать таблицу для обогащения.

Темы, рассматриваемые в этом руководстве:

> [!div class="checklist"]
> - Шаги для получения набора данных таблицы Spark, содержащего данные временных рядов.
> - Использование интерфейса мастера в Azure Synapse для обогащения данных с помощью Детектора аномалий в Cognitive Services.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись, прежде чем начинать работу](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Предварительные требования

- [Рабочая область Azure Synapse Analytics](../get-started-create-workspace.md) с учетной записью хранения Azure Data Lake Storage 2-го поколения, настроенной в качестве хранилища по умолчанию. При работе с файловой системой Data Lake Storage 2-го поколения вам нужно иметь права *участника для получения данных Хранилища BLOB-объектов*.
- Пул Spark в рабочей области Azure Synapse Analytics. Дополнительные сведения см. в статье [Создание пула Spark в Azure Synapse](../quickstart-create-sql-pool-studio.md).
- Выполнение действий, предшествующих настройке, указанных в [руководстве по необходимым компонентам для использования Cognitive Services в Azure Synapse](tutorial-configure-cognitive-services-synapse.md).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-spark-table"></a>Создание таблицы Spark

Для работы с этим руководством вам потребуется таблица Spark.

1. Загрузите следующий файл записной книжки, содержащий код для создания таблицы Spark: [prepare_anomaly_detector_data.ipynb](https://go.microsoft.com/fwlink/?linkid=2149577).

1. Отправьте файл в рабочую область Azure Synapse.

   ![Снимок экрана: варианты для отправки записной книжки.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00a.png)

1. Откройте файл записной книжки и выберите **Выполнить все**, чтобы выполнить все ячейки.

   ![Снимок экрана: варианты для создания таблицы Spark.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00b.png)

1. Таблица Spark с именем **anomaly_detector_testing_data** теперь должна отображаться в базе данных Spark по умолчанию.

## <a name="open-the-cognitive-services-wizard"></a>Открытие мастера Cognitive Services

1. Щелкните правой кнопкой мыши таблицу Spark, созданную на предыдущем шаге. Выберите **Машинное обучение** > **Enrich with existing model** (Дополнить с использованием существующей модели), чтобы открыть мастер.

   ![Снимок экрана: варианты для открытия мастера оценки.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00g.png)

2. Появляется панель конфигурации и запрос на выбор модели Cognitive Services. Выберите **Детектор аномалий**.

   ![Снимок экрана: выбор Детектора аномалий в качестве модели.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00c.png)

## <a name="provide-authentication-details"></a>Предоставление сведений для проверки подлинности

Для выполнения проверки подлинности в Cognitive Services нужно предоставить ссылку на секрет, размещенный в хранилище ключей. Указанные ниже входные данные зависят от [предварительных действий](tutorial-configure-cognitive-services-synapse.md), которые вы должны были выполнить до этого этапа.

- **Подписка Azure**: Выберите подписку Azure, к которой принадлежит ваше хранилище ключей.
- **Учетная запись Cognitive Services.** Введите ресурс "Анализ текста", к которому вы будете подключаться.
- **Связанная служба Azure Key Vault.** Во время выполнения предварительных действий вы создали связанную службу для ресурса "Анализ текста". Выберите ее.
- **Имя секрета.** Введите имя секрета в хранилище ключей, содержащего ключ для проверки подлинности в ресурсе Cognitive Services.

![Снимок экрана: сведения о проверке подлинности для хранилища ключей.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00d.png)

## <a name="configure-anomaly-detector"></a>Настройка Детектора аномалий

Чтобы настроить Детектор аномалий, укажите следующие сведения:

- **Granularity** (Степень детализации). Скорость выборки данных. Выберите **ежемесячно**. 

- **Столбец метки времени**. Столбец, представляющий временные ряды. Выберите **метки времени (строка)** .

- **Столбец значения временных рядов**. Столбец, представляющий значение ряда в момент времени, указанный в столбце "Метка времени". Выберите **значение (двойной)** .

- **Столбец группирования**. Столбец, группирующий ряды. То есть все строки, имеющие одно и то же значение в этом столбце, должны формировать один временной ряд. Выберите **группа (строка)** .

Когда все будет готово, выберите **Открыть записную книжку**. Будет создана записная книжка с кодом PySpark, который использует Azure Cognitive Services для обнаружения аномалий.

![Снимок экрана: подробные сведения о конфигурации для Детектора аномалий.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00e.png)

## <a name="run-the-notebook"></a>Запустите записную книжку

Записная книжка, которую вы только что открыли, использует [библиотеку mmlspark](https://github.com/Azure/mmlspark) для подключения к Cognitive Services. Предоставленные сведения об Azure Key Vault позволяют безопасно сослаться на секреты из этого интерфейса, не раскрывая их.

Теперь можно выполнить все ячейки, чтобы запустить обнаружение аномалий. Выберите **Запустить все**. [Дополнительные сведения о Детекторе аномалий в Cognitive Services](../../cognitive-services/anomaly-detector/index.yml).

![Снимок экрана: обнаружение аномалий.](media/tutorial-cognitive-services/tutorial-cognitive-services-anomaly-00f.png)

## <a name="next-steps"></a>Дальнейшие действия

- [Учебник. Анализ тональности с помощью Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Учебник. Оценка моделей машинного обучения в выделенных пулах Azure Synapse SQL](tutorial-sql-pool-model-scoring-wizard.md)
- [Возможности машинного обучения в Azure Synapse Analytics](what-is-machine-learning.md)

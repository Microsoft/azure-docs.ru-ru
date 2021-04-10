---
title: Руководство. Анализ тональности с помощью Cognitive Services
description: Сведения о том, как использовать Cognitive Services для анализа тональности в Azure Synapse Analytics
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: 08d5e53facce172c2287c2e341895f0ee38571f0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98943701"
---
# <a name="tutorial-sentiment-analysis-with-cognitive-services-preview"></a>Руководство по анализу тональности с помощью Cognitive Services (предварительная версия)

Из этого руководства вы узнаете, как дополнить данные в Azure Synapse с помощью [Azure Cognitive Services](../../cognitive-services/index.yml). Для анализа тональности вы будете использовать возможности [Анализа текста](../../cognitive-services/text-analytics/index.yml). 

Чтобы дополнить данные сведениями о тональности, пользователь в Azure Synapse может просто выбрать таблицу с текстовым столбцом. Эти тональности могут быть положительными, отрицательными, смешанными или нейтральными. С ними также возвращаются данные о вероятности.

Темы, рассматриваемые в этом руководстве:

> [!div class="checklist"]
> - получение набора данных из таблицы Spark, содержащей текстовый столбец, для анализа тональности;
> - использование возможностей мастера в Azure Synapse для дополнения данных с помощью Анализа текста в Cognitive Services.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись, прежде чем начинать работу](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Предварительные требования

- [Рабочая область Azure Synapse Analytics](../get-started-create-workspace.md) с учетной записью хранения Azure Data Lake Storage 2-го поколения, настроенной в качестве хранилища по умолчанию. При работе с файловой системой Data Lake Storage 2-го поколения вам нужно иметь права *участника для получения данных Хранилища BLOB-объектов*.
- Пул Spark в рабочей области Azure Synapse Analytics. Дополнительные сведения см. в статье [Создание пула Spark в Azure Synapse](../quickstart-create-sql-pool-studio.md).
- Предварительные действия описаны в руководстве по [настройке Cognitive Services в Azure Synapse](tutorial-configure-cognitive-services-synapse.md).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Войдите на [портал Azure](https://portal.azure.com/).

## <a name="create-a-spark-table"></a>Создание таблицы Spark

Для работы с этим руководством вам потребуется таблица Spark.

1. Скачайте файл [FabrikamComments.csv](https://github.com/Kaiqb/KaiqbRepo0731190208/blob/master/CognitiveServices/TextAnalytics/FabrikamComments.csv), который содержит набор данных для анализа текста. 

1. Отправьте этот файл в учетную запись хранения Azure Synapse в Data Lake Storage 2-го поколения.
  
   ![Снимок экрана: варианты для отправки данных.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00a.png)

1. Создайте таблицу Spark из CSV-файла. Для этого щелкните этот файл правой кнопкой мыши и последовательно выберите элементы **Создать записную книжку** > **Создать таблицу Spark**.

   ![Снимок экрана: варианты для создания таблицы Spark.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00b.png)

1. Присвойте имя таблице в ячейке кода и выполните записную книжку в пуле Spark. Обязательно задайте значение `header=True`.

   ![Снимок экрана: выполнение записной книжки.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00c.png)

   ```python
   %%pyspark
   df = spark.read.load('abfss://default@azuresynapsesa.dfs.core.windows.net/data/FabrikamComments.csv', format='csv'
   ## If a header exists, uncomment the line below
   , header=True
   )
   df.write.mode("overwrite").saveAsTable("default.YourTableName")
   ```

## <a name="open-the-cognitive-services-wizard"></a>Открытие мастера Cognitive Services

1. Щелкните правой кнопкой мыши таблицу Spark, созданную при выполнении предыдущей процедуры. Выберите элементы **Машинное обучение** > **Enrich with existing model (Дополнить с использованием существующей модели)** , чтобы открыть мастер.

   ![Снимок экрана: варианты для открытия мастера оценки.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00d.png)

2. Отобразится панель конфигурации и запрос на выбор модели Cognitive Services. Выберите элемент **Text analytics - Sentiment Analysis** (Анализ текста — анализ тональности).

   ![Снимок экрана: выбор модели Cognitive Services.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00e.png)

## <a name="provide-authentication-details"></a>Предоставление сведений для проверки подлинности

Для аутентификации в Cognitive Services нужно предоставить ссылку на секрет для хранилища ключей. Указанные ниже входные данные зависят от [предварительных действий](tutorial-configure-cognitive-services-synapse.md), которые должны быть выполнены до этого этапа.

- **Подписка Azure**: Выберите подписку Azure, к которой принадлежит ваше хранилище ключей.
- **Учетная запись Cognitive Services.** Введите ресурс "Анализ текста", к которому вы будете подключаться.
- **Связанная служба Azure Key Vault.** Во время выполнения предварительных действий вы создали связанную службу для ресурса "Анализ текста". Выберите ее.
- **Имя секрета.** Введите имя секрета в хранилище ключей, содержащего ключ для проверки подлинности в ресурсе Cognitive Services.

![Снимок экрана: сведения об аутентификации для хранилища ключей.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00f.png)

## <a name="configure-sentiment-analysis"></a>Настройка анализа тональности

Затем настройте анализ тональности. Выберите следующие данные:
- **Язык**: выберите вариант **Английский** в качестве языка текста, для которого нужно выполнить анализ тональности.
- **Текстовый столбец.** Выберите **комментарий (строка)** в качестве текстового столбца в наборе данных, который будет проанализирован для определения тональности.

Когда все будет готово, выберите элемент **Открыть записную книжку**. Будет создана записная книжка с кодом PySpark, который обеспечивает анализ тональности с помощью Azure Cognitive Services.

![Снимок экрана: параметры для настройки анализа тональности.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00g.png)

## <a name="run-the-notebook"></a>Запустите записную книжку

Записная книжка, которую вы только что открыли, использует [библиотеку mmlspark](https://github.com/Azure/mmlspark) для подключения к Cognitive Services. Предоставленные сведения об Azure Key Vault позволяют сослаться на секреты из этого интерфейса с поддержкой высокого уровня безопасности, не раскрывая их.

Теперь вы можете выполнить все ячейки, чтобы дополнить данные сведениями о тональности. Выберите элемент **Запустить все**. 

Тональности возвращаются как **положительные**, **отрицательные**, **нейтральные** или **смешанные**. Вы также получите данные о вероятности для каждой тональности. [Дополнительные сведения об анализе тональности в Cognitive Services.](../../cognitive-services/text-analytics/how-tos/text-analytics-how-to-sentiment-analysis.md)

![Снимок экрана: анализ тональности.](media/tutorial-cognitive-services/tutorial-cognitive-services-sentiment-00h.png)

## <a name="next-steps"></a>Дальнейшие действия
- [Учебник. Обнаружение аномалий с помощью Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Учебник. Оценка моделей машинного обучения в выделенных пулах Azure Synapse SQL](tutorial-sql-pool-model-scoring-wizard.md)
- [Возможности машинного обучения в Azure Synapse Analytics](what-is-machine-learning.md)
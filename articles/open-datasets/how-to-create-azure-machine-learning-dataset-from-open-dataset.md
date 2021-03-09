---
title: Создание наборов данных с помощью открытых наборов данных Azure
titleSuffix: Azure Open Datasets
description: Узнайте, как создать набор данных Машинное обучение Azure из набора открытых данных Azure.
ms.service: open-datasets
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.date: 08/05/2020
ms.custom: how-to, tracking-python
ms.openlocfilehash: dd1440cd7bda8d40a81290cd9f633264b9641dc5
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102502558"
---
# <a name="create-azure-machine-learning-datasets-from-azure-open-datasets"></a>Создание наборов данных Машинное обучение Azure из открытых наборов данных Azure

Из этой статьи вы узнаете, как перенести проверенные данные обогащения в локальные или удаленные эксперименты машинного обучения с помощью [машинное обучение Azure](../machine-learning/overview-what-is-azure-ml.md) наборов данных и [Azure Open DataSets](./index.yml). 

Создавая [набор данных машинное обучение Azure](../machine-learning/how-to-create-register-datasets.md), вы создаете ссылку на расположение источника данных вместе с копией его метаданных. Поскольку наборы данных отложенно оцениваются, и данные остаются в существующем расположении,
* Дополнительная плата за хранение не взимается.
* Не рискуйте непреднамеренное изменение исходных источников данных. 
* Улучшение производительности рабочих процессов ML.

Чтобы узнать, где находятся наборы данных в общем рабочем процессе доступа к данным Машинное обучение Azure, ознакомьтесь со статьей [безопасный доступ к](../machine-learning/concept-data.md#data-workflow) данным.

Открытые наборы данных Azure — это проверенные общедоступные наборы данных, которые можно использовать для добавления функций для конкретных сценариев, чтобы расширить возможности прогнозных решений и повысить их точность. Сведения об общедоступных доменах, которые могут помочь в обучении моделей машинного обучения, см. в разделе [Открытие каталога наборов](https://azure.microsoft.com/en-in/services/open-datasets/catalog/) данных.

* [сводка](https://azure.microsoft.com/services/open-datasets/catalog/noaa-integrated-surface-data/)
* [писью](https://azure.microsoft.com/services/open-datasets/catalog/us-decennial-census-zip/)
* [государственные](https://azure.microsoft.com/services/open-datasets/catalog/public-holidays/)
* [общедоступная безопасность](https://azure.microsoft.com/services/open-datasets/catalog/chicago-safety-data/)
* location

Открытые наборы данных находятся в облаке на Microsoft Azure и включены в [пакет SDK для машинное обучение Azure Python](#create-datasets-with-the-sdk) и в [машинное обучение Azure Studio](#create-datasets-with-the-studio).


## <a name="prerequisites"></a>Предварительные требования

Для этой статьи требуется:

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начать работу. Попробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree).

* [Рабочая область машинное обучение Azure](../machine-learning/how-to-manage-workspace.md).

* [Установленный пакет SDK машинное обучение Azure для Python](/python/api/overview/azure/ml/install), который включает в себя `azureml-datasets` пакеты.

    * Создание [машинное обучение Azure вычислительного экземпляра](../machine-learning/how-to-create-manage-compute-instance.md), который представляет собой полностью настроенную и управляемую среду разработки, включающую интегрированные записные книжки и уже установленный пакет SDK.

    **OR**

    * Вы можете работать с собственной средой Python и установить пакет SDK самостоятельно с помощью [этих инструкций](/python/api/overview/azure/ml/install).

> [!NOTE]
> Некоторые классы наборов данных имеют зависимости от пакета [azureml-](/python/api/azureml-dataprep/) DataMarket, который совместим только с 64-разрядным Python. Для пользователей Linux эти классы поддерживаются только в следующих дистрибутивах: Red Hat Enterprise Linux (7, 8), Ubuntu (14,04, 16,04, 18,04), Fedora (27, 28), Debian (8, 9) и CentOS (7).

## <a name="create-datasets-with-the-sdk"></a>Создание наборов данных с помощью пакета SDK

Чтобы создать Машинное обучение Azure наборы данных с помощью классов открытых наборов данных Azure в пакете SDK для Python, убедитесь, что пакет установлен с помощью `pip install azureml-opendatasets` . Каждый дискретный набор данных представлен собственным классом в пакете SDK, а некоторые классы доступны в виде машинное обучение Azure [ `TabularDataset` , `FileDataset` ](../machine-learning/how-to-create-register-datasets.md#dataset-types)или и того, и другого. Полный список классов см. в [справочной документации](/python/api/azureml-opendatasets/azureml.opendatasets) `opendatasets` .

Некоторые классы можно извлечь `opendatasets` как `TabularDataset` или `FileDataset` , что позволяет напрямую управлять файлами и/или скачивать их. Другие классы могут получить набор данных **только** с помощью `get_tabular_dataset()` функций или `get_file_dataset()` из `Dataset` класса в пакете SDK Python.

В следующем коде показано, что `opendatasets` класс MNIST может возвращать значение `TabularDataset` или `FileDataset` . 


```python
from azureml.core import Dataset
from azureml.opendatasets import MNIST

# MNIST class can return either TabularDataset or FileDataset
tabular_dataset = MNIST.get_tabular_dataset()
file_dataset = MNIST.get_file_dataset()
```

В этом примере `opendatasets` класс диабета доступен только в качестве `TabularDataset` , поэтому используется `get_tabular_dataset()` .

```python

from azureml.opendatasets import Diabetes
from azureml.core import Dataset

# Diabetes class can return ONLY TabularDataset and must be called from the static function
diabetes_tabular = Diabetes.get_tabular_dataset()
```
## <a name="register-datasets"></a>Регистрация наборов данных

Зарегистрируйте Машинное обучение Azure набор данных в рабочей области, чтобы вы могли поделиться ими с другими пользователями и использовать их в разных экспериментах в рабочей области. При регистрации Машинное обучение Azure набора данных, созданного из открытых наборов данных, данные сразу не загружаются, но при запросе к данным они будут доступны позже (например, во время обучения) из центрального хранилища.

Чтобы зарегистрировать наборы данных в рабочей области, используйте [`register()`](/python/api/azureml-core/azureml.data.abstract_dataset.abstractdataset#register-workspace--name--description-none--tags-none--create-new-version-false-) метод. 

```Python
titanic_ds = titanic_ds.register(workspace=workspace,
                                 name='titanic_ds',
                                 description='titanic training data')
```

## <a name="create-datasets-with-the-studio"></a>Создание наборов данных с помощью студии

Вы также можете создавать Машинное обучение Azure наборы данных из открытых наборов данных Azure с помощью [машинное обучение Azure Studio](https://ml.azure.com), Объединенного веб-интерфейса, включающего средства машинного обучения, для выполнения сценариев обработки и анализа данных для специалистов по обработке и анализу и обработке данных на всех уровнях навыков.

> [!Note]
> Наборы данных, созданные с помощью Машинное обучение Azure Studio, автоматически регистрируются в рабочей области.

1. В рабочей области выберите вкладку **наборы данных** в разделе **активы**. В раскрывающемся меню **CREATE DataSet (создание набора данных** ) выберите пункт **из открытых наборов**.

    ![Открытие набора данных с помощью пользовательского интерфейса](./media/how-to-create-dataset-from-open-dataset/open-datasets-1.png)

1. Выберите набор данных, выбрав его плитку. (Можно выполнить фильтрацию с помощью панели поиска.) Нажмите кнопку **Далее**.

    ![Выбор набора данных](./media/how-to-create-dataset-from-open-dataset/open-datasets-2.png)

1. Выберите имя, под которым будет зарегистрирован набор данных, и при необходимости отфильтруйте данные с помощью доступных фильтров. В этом случае для набора данных " **общедоступные праздники** " вы фильтруете период времени в один год и код страны только в США. Подробные сведения о данных, такие как, описания полей и диапазоны дат, см. в [каталоге открытых наборов данных Azure](https://azure.microsoft.com/services/open-datasets/catalog) . Нажмите кнопку **создания**.

    ![Установка параметров набора данных и создание набора данных](./media/how-to-create-dataset-from-open-dataset/open-datasets-3.png)

    Набор данных теперь доступен в рабочей области в разделе **наборы данных**. Его можно использовать точно так же, как и другие созданные наборы данных.


## <a name="access-datasets-for-your-experiments"></a>Доступ к наборам данных для экспериментов

Используйте свои наборы данных в экспериментах машинного обучения для обучения моделей ML. [Узнайте больше об обучении с наборами данных](../machine-learning/how-to-train-with-datasets.md).

## <a name="example-notebooks"></a>Примеры записных книжек

Примеры и демонстрации функций открытых наборов данных см. в этих [примерах записных книжек](samples.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Обучить первую модель машинного обучения](../machine-learning/tutorial-1st-experiment-sdk-train.md).

* [Обучение с наборами данных](../machine-learning/how-to-train-with-datasets.md).

* [Создайте набор данных машинного обучения Azure](../machine-learning/how-to-create-register-datasets.md).

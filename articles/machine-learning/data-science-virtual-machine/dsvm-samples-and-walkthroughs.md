---
title: Примеры программ & примеры машинного обучения
titleSuffix: Azure Data Science Virtual Machine
description: С помощью этих примеров и пошаговых руководств вы узнаете, как выполнять типичные задачи и сценарии с помощью виртуальной машины для обработки и анализа данных.
keywords: средства анализа и обработки данных, виртуальная машина для анализа и обработки данных, средства для анализа и обработки данных, анализ и обработка данных Linux
services: machine-learning
ms.service: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: cda5dfd936243602775e1f4f965032b9d746b0b7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100519768"
---
# <a name="samples-on-azure-data-science-virtual-machines"></a>Примеры на виртуальных машинах Azure для обработки и анализа данных

Виртуальные машины Azure для обработки и анализа данных (Dsvm) включают в себя полный набор примеров кода. Эти примеры включают в себя Jupyterные записные книжки и сценарии на языках, таких как Python и R.
> [!NOTE]
> Дополнительные сведения о том, как запускать записные книжки Jupyter на виртуальных машинах для обработки и анализа данных, см. в разделе [Access Jupyter](#access-jupyter) .

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этих примеров необходимо подготовить [виртуальную машину Ubuntu](./dsvm-ubuntu-intro.md)для обработки и анализа данных.

## <a name="available-samples"></a>Доступные примеры
| Категории примеров | Описание | Расположения |
| ------------- | ------------- | ------------- |
| Язык R  | Примеры иллюстрируют такие сценарии, как подключение к облачным хранилищам данных на основе Azure и сравнение кода R и Microsoft Machine Learning Server с открытым кодом. Они также объясняют, как эксплуатацию модели на Microsoft Machine Learning Server и SQL Server. <br/> [Язык R](#r-language) | <br/>`~notebooks` <br/> <br/> `~samples/MicrosoftR` <br/> <br/> `~samples/RSqlDemo` <br/> <br/> `~samples/SQLRServices`<br/> <br/>|
| Язык Python  | Примеры иллюстрируют такие сценарии, как подключение к облачным хранилищам данных на основе Azure и работа с Машинное обучение Azure.  <br/> [Язык Python](#python-language) | <br/>`~notebooks` <br/><br/>|
| Язык Julia  | Содержит подробное описание построения и глубокого обучения в Julia. Также объясняется, как вызывать C и Python из Julia. <br/> [Язык Julia](#julia-language) |<br/> Windows:<br/> `~notebooks/Julia_notebooks`<br/><br/> Linux:<br/> `~notebooks/julia`<br/><br/> |
| Машинное обучение Azure  | Демонстрирует создание моделей машинного обучения и глубокого обучения с помощью Машинное обучение. Модели можно развертывать в любом расположении. Используйте автоматическое машинное обучение и интеллектуальную настройку гиперпараметров. Вы также можете использовать управление моделями и распределенное обучение. <br/> [Машинное обучение](#azure-machine-learning) | <br/>`~notebooks/AzureML`<br/> <br/>|
| Записные книжки PyTorch  | Примеры глубокого обучения, в которых используются нейронные сети на основе PyTorch. Записные книжки есть как для базовых, так и для расширенных сценариев.  <br/> [Записные книжки PyTorch](#pytorch) | <br/>`~notebooks/Deep_learning_frameworks/pytorch`<br/> <br/>|
| TensorFlow  |  Разнообразные примеры и методики нейронной сети, реализованные с помощью платформы TensorFlow. <br/> [TensorFlow](#tensorflow); | <br/>`~notebooks/Deep_learning_frameworks/tensorflow`<br/><br/> |
| Microsoft Cognitive Toolkit <br/>   | Примеры глубокого обучения, опубликованные Группой Cognitive Toolkit в корпорации Майкрософт.  <br/> [Cognitive Toolkit](#cntk) | <br/> `~notebooks/DeepLearningTools/CNTK/Tutorials`<br/><br/> Linux:<br/> `~notebooks/CNTK`<br/> <br/>|
| Caffe2 | Примеры глубокого обучения, в которых используются нейронные сети на основе Caffe2. Несколько записных книжек посвящены пользователям с Caffe2 и их эффективному использованию. Примеры включают предварительную обработку изображений и создание набора данных. Они также включают регрессию и использование предварительно обученных моделей. <br/> [Caffe2](#caffe2) | <br/>`~notebooks/Deep_learning_frameworks/caffe2`<br/><br/> |
| H2O   | Примеры на основе Python, использующие H2O для реальных сценариев проблем. <br/> [H2O](#h2o) | <br/>`~notebooks/h2o`<br/><br/> |
| Язык SparkML  | Примеры, в которых используются функции Apache Spark MLLib Toolkit через pySpark и MMLSpark: Microsoft Машинное обучение для Apache Spark на Apache Spark 2. x.  <br/> [Язык SparkML](#sparkml) | <br/>`~notebooks/SparkML/pySpark`<br/>`~notebooks/MMLSpark`<br/><br/>  |
| XGBoost; | Стандартные примеры машинного обучения в XGBoost для таких сценариев, как классификация и регрессия. <br/> [XGBoost](#xgboost) | <br/>Windows:<br/>`\dsvm\samples\xgboost\demo`<br/><br/> |

<br/>

## <a name="access-jupyter"></a>Доступ к Jupyter 

Чтобы получить доступ к Jupyter, выберите значок **Jupyter** в меню Рабочий стол или приложение. Вы также можете получить доступ к Jupyter в выпуске Linux DSVM. Для удаленного доступа из веб-браузера перейдите `https://<Full Domain Name or IP Address of the DSVM>:8000` на Ubuntu.

Чтобы добавить исключения и предоставить доступ к Jupyter через браузер, используйте следующие рекомендации.


![Добавление исключения Jupyter](./media/ubuntu-jupyter-exception.png)


Войдите в систему, используя тот же пароль, который вы используете для входа на виртуальную машину для обработки и анализа данных.
<br/>

**Домашняя страница Jupyter**
<br/>![Домашняя страница Jupyter](./media/jupyter-home.png)<br/>

## <a name="r-language"></a>Язык R 
<br/>![Примеры для R](./media/r-language-samples.png)<br/>

## <a name="python-language"></a>Язык Python
<br/>![Примеры для Python](./media/python-language-samples.png)<br/>

## <a name="julia-language"></a>Язык Julia 
<br/>![Примеры для Julia](./media/julia-samples.png)<br/>

## <a name="azure-machine-learning"></a>Машинное обучение Azure 
<br/>![Образцы Машинного обучения Azure](./media/azureml-samples.png)<br/>

## <a name="pytorch"></a>PyTorch
<br/>![Примеры PyTorch](./media/pytorch-samples.png)<br/>

## <a name="tensorflow"></a>TensorFlow 
<br/>![Примеры TensorFlow](./media/tensorflow-samples.png)<br/>


## <a name="cntk"></a>CNTK 
<br/>![Примеры CNTK](./media/cntk-samples.png)<br/>


## <a name="caffe2"></a>Caffe2 
<br/>![Примеры caffe2](./media/caffe2-samples.png)<br/>

## <a name="h2o"></a>H2O 
<br/>![Примеры H2O](./media/h2o-samples.png)<br/>

## <a name="sparkml"></a>SparkML 
<br/>![Примеры SparkML](./media/sparkml-samples.png)<br/>

## <a name="xgboost"></a>XGBoost; 
<br/>![Примеры XGBoost](./media/xgboost-samples.png)<br/>


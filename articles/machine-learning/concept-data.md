---
title: Защита доступа к данным в облаке
titleSuffix: Azure Machine Learning
description: Узнайте, как безопасно подключаться к хранилищу данных в Azure с помощью Машинное обучение Azure хранилищ и данных.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: nibaccam
author: nibaccam
ms.author: nibaccam
ms.date: 08/31/2020
ms.custom: devx-track-python, data4ml
ms.openlocfilehash: 601be8409db22162a410d481e6609d378718a7b4
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102503595"
---
# <a name="secure-data-access-in-azure-machine-learning"></a>Защита доступа к данным в Машинное обучение Azure

Машинное обучение Azure позволяет легко подключаться к данным в облаке. Он предоставляет уровень абстракции по сравнению с базовой службой хранилища, что позволяет безопасно получать доступ к данным и работать с ними без необходимости написания кода, относящегося к типу хранилища. Машинное обучение Azure также предоставляет следующие возможности данных:

*    Взаимодействие с кадрами данных Pandas и Spark
*    Управление версиями и отслеживание преобразований данных
*    Маркировка данных 
*    Мониторинг смещения данных
    
## <a name="data-workflow"></a>Рабочий процесс данных

Когда вы будете готовы к использованию данных в облачном решении для хранения, мы рекомендуем использовать следующий рабочий процесс доставки данных. В этом рабочем процессе предполагается, что у вас есть [учетная запись хранения Azure](../storage/common/storage-account-create.md?tabs=azure-portal) и данные в облачной службе хранилища Azure. 

1. Создайте [хранилище данных машинное обучение Azure](#datastores) для хранения сведений о подключении к службе хранилища Azure.

2. На основе этого хранилища [данных создайте машинное обучение Azure DataSet](#datasets) , указывающий на определенные файлы в базовом хранилище. 

3. Чтобы использовать этот набор данных в эксперименте машинного обучения, можно либо
    1. Подключите его к целевому объекту вычислений эксперимента для обучения модели.

        **OR** 

    1. Его следует использовать непосредственно в Машинное обучение Azureных решениях, таких как Автоматизированные запуски экспериментов машинного обучения, конвейеры машинного обучения или [конструктор машинное обучение Azure](concept-designer.md).

4. Создание [мониторов набора данных](#drift) для выходного набора данных модели для обнаружения смещения данных. 

5. Если будет обнаружено смещение данных, обновите входной набор данных и соответствующим образом выполните обучение модели.

На следующей схеме показана визуальная демонстрация этого рекомендуемого рабочего процесса.

![На схеме показана служба хранилища Azure, которая помещается в хранилище данных, которое передается в DataSet. Набор данных передается в обучение модели, которое передается обратно в набор данных.](./media/concept-data/data-concept-diagram.svg)

<a name="datastores"></a>
## <a name="connect-to-storage-with-datastores"></a>Подключение к хранилищу данных с использованием хранилищ данных

Машинное обучение Azure хранилища данных безопасно сохраняют информацию о подключении к хранилищу в Azure, поэтому вам не нужно кодировать их в своих сценариях. [Зарегистрируйте и создайте хранилище](how-to-access-data.md) данных, чтобы легко подключиться к учетной записи хранения и получить доступ к данным в базовой службе хранилища. 

Поддерживаемые облачные службы хранилища в Azure, которые можно зарегистрировать как хранилища данных:

+ Контейнер больших двоичных объектов Azure
+ Общая папка Azure
+ Azure Data Lake
+ Azure Data Lake 2-го поколения
+ База данных SQL Azure
+ База данных Azure для PostgreSQL
+ Файловая система Databricks
+ База данных Azure для MySQL

>[!TIP]
> Общедоступная функциональность для создания хранилищ данных требует проверки подлинности на основе учетных данных для доступа к службам хранилища, таким как субъект-служба или маркер подписанного URL-доступа (SAS). Доступ к этим учетным данным можно получить у пользователей, имеющих доступ к рабочей области с правами *читателя* . <br><br>Если это проблема,  [Создайте хранилище данных, которое использует доступ к данным на основе удостоверений к службам хранилища (Предварительная версия)](how-to-identity-based-data-access.md). Эта возможность является [экспериментальной](/python/api/overview/azure/ml/#stable-vs-experimental) функцией предварительной версии и может быть изменена в любое время.

<a name="datasets"></a>
## <a name="reference-data-in-storage-with-datasets"></a>Справочные данные в хранилище с помощью наборов данных

Машинное обучение Azure наборы данных не копируются. Создавая набор данных, вы создаете ссылку на данные в своей службе хранилища вместе с копией его метаданных. 

Поскольку наборы данных отложенно оцениваются, и данные остаются в существующем расположении,

* Дополнительная плата за хранение не взимается.
* Не рискуйте непреднамеренное изменение исходных источников данных.
* Улучшение производительности рабочих процессов ML.

Чтобы взаимодействовать с данными в хранилище, [Создайте набор данных](how-to-create-register-datasets.md) для упаковки данных в объект, который можно использовать для задач машинного обучения. Зарегистрируйте набор данных в рабочей области, чтобы поделиться им и повторно использовать его в разных экспериментах без сложностей приема данных.

Наборы данных можно создавать из локальных файлов, общедоступных URL-адресов, [открытых наборов данных Azure](https://azure.microsoft.com/services/open-datasets/)или служб хранилища Azure через хранилища данных. 

Существует два типа наборов данных: 

+ [Филедатасет](/python/api/azureml-core/azureml.data.file_dataset.filedataset) ссылается на один или несколько файлов в хранилищах данных или общедоступных URL-адресах. Если данные уже очищены и готовы к использованию в учебных экспериментах, можно [загрузить или подключить](how-to-train-with-datasets.md#mount-files-to-remote-compute-targets) к целевому объекту вычислений файлы, на которые ссылается филедатасетс.

+ [Табулардатасет](/python/api/azureml-core/azureml.data.tabulardataset) представляет данные в табличном формате путем синтаксического анализа указанного файла или списка файлов. Вы можете загрузить Табулардатасет в кадр данных Pandas или Spark для дальнейшей обработки и очистки. Полный список форматов данных, которые можно создать Табулардатасетс из, см. в разделе [класс табулардатасетфактори](/python/api/azureml-core/azureml.data.dataset_factory.tabulardatasetfactory).

Дополнительные возможности наборов данных можно найти в следующей документации:

+ [Версия и отслеживание](how-to-version-track-datasets.md) журнала преобразований данных.
+ [Отслеживайте набор данных](how-to-monitor-datasets.md) , чтобы упростить обнаружение отклонения данных.    

## <a name="work-with-your-data"></a>Работа с данными

С помощью наборов данных можно выполнять ряд задач машинного обучения благодаря простой интеграции с Машинное обучение Azureными функциями. 

+ Создайте [проект меток данных](#label).
+ Обучение моделей машинного обучения:
     + [автоматические эксперименты ML](how-to-use-automated-ml-for-ml-models.md)
     + [конструктор](tutorial-designer-automobile-price-train-score.md#import-data)
     + [Компьютеры](how-to-train-with-datasets.md)
     + [Конвейеры Машинное обучение Azure](./how-to-create-machine-learning-pipelines.md)
+ Доступ к наборам данных для оценки с помощью [вывода пакетов](./tutorial-pipeline-batch-scoring-classification.md) в [конвейерах машинного обучения](./how-to-create-machine-learning-pipelines.md).
+ Настройте монитор набора данных для обнаружения [несмещенных данных](#drift) .

<a name="label"></a>

## <a name="label-data-with-data-labeling-projects"></a>Пометка данных с помощью проектов меток данных

Добавление меток к большим объемам данных часто усложняет проекты машинного обучения. Для таких компонентов концепции компьютера, как классификация изображений или обнаружение объектов, обычно требуются тысячи изображений и соответствующие метки.

Решение "Машинное обучение Azure" служит централизованным расположением для создания, администрирования и мониторинга проектов маркировки. Проекты маркировки помогают упорядочивать данные, метки и координировать работу команды, то есть более эффективно управлять задачами присвоения меток. Сейчас поддерживаются такие задачи: классификация изображений (с несколькими метками или классами) и идентификация объектов с использованием ограничивающих прямоугольников.

Создайте [проект меток данных](how-to-create-labeling-projects.md)и выводите набор данных для использования в экспериментах машинного обучения.

<a name="drift"></a>

## <a name="monitor-model-performance-with-data-drift"></a>Мониторинг производительности модели с помощью смещения данных

В контексте машинного обучения смещение данных — это изменение входных данных модели, которое приводит к снижению производительности модели. Это одна из основных причин, с которой точность снижается с течением времени, поэтому отслеживание смещения данных помогает выявить проблемы с производительностью модели.

Дополнительные сведения о способах обнаружения и оповещения о смещении данных для новых данных в наборе данных см. в статье [Создание монитора набора данных](how-to-monitor-datasets.md) .

## <a name="next-steps"></a>Дальнейшие действия 

+ Создайте набор данных в Машинное обучение Azure Studio или с помощью пакета SDK для Python, [выполнив следующие действия.](how-to-create-register-datasets.md)
+ Испытайте примеры обучения наборов данных с помощью [примеров записных книжек](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).
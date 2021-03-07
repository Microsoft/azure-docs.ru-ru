---
title: Разработка с помощью Аутомл & Azure Databricks
titleSuffix: Azure Machine Learning
description: Научитесь настраивать среду разработки в Машинное обучение Azure и Azure Databricks. Используйте пакеты SDK машинного обучения Azure для кирпичей и кирпичей Аутомл.
services: machine-learning
author: rastala
ms.author: roastala
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.date: 10/21/2020
ms.topic: conceptual
ms.custom: how-to, devx-track-python
ms.openlocfilehash: 43e503105eb9d59de5de2fe26e024cc473c61af6
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102430502"
---
# <a name="set-up-a-development-environment-with-azure-databricks-and-automl-in-azure-machine-learning"></a>Настройка среды разработки с помощью Azure Databricks и Аутомл в Машинное обучение Azure 

Узнайте, как настроить среду разработки в Машинное обучение Azure, которая использует Azure Databricks и автоматизированное ML.

Azure Databricks идеально подходит для выполнения крупномасштабных рабочих процессов машинного обучения на масштабируемой платформе Apache Spark в облаке Azure. Она предоставляет среду для совместной работы на основе записных книжек с вычислительным кластером на основе ЦП или GPU.

Сведения о других средах разработки машинного обучения см. в разделе [Настройка среды разработки Python](how-to-configure-environment.md).


## <a name="prerequisite"></a>Предварительное требование

Рабочая область Машинное обучение Azure. Если у вас ее нет, можно создать рабочую область Машинное обучение Azure с помощью шаблонов [портал Azure](how-to-manage-workspace.md), [Azure CLI](how-to-manage-workspace-cli.md#create-a-workspace)и [Azure Resource Manager](how-to-create-workspace-template.md).


## <a name="azure-databricks-with-azure-machine-learning-and-automl"></a>Azure Databricks с Машинное обучение Azure и Аутомл

Azure Databricks интегрируется с Машинное обучение Azure и его возможностями Аутомл. 

Можно использовать Azure Databricks:

+ Для обучения модели с помощью Spark MLlib и развертывания модели в ACI/AKS.
+ С [автоматизированными возможностями машинного обучения](concept-automated-ml.md) с помощью пакета Azure ML SDK.
+ В качестве целевого объекта вычислений из [конвейера машинное обучение Azure](concept-ml-pipelines.md).

## <a name="set-up-a-databricks-cluster"></a>Настройка кластера кирпичей данных

Создание [кластера кирпичей](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal). Некоторые параметры применяются только в том случае, если пакет SDK устанавливается для автоматического машинного обучения в модулях.

**Создание кластера занимает несколько минут.**

Используйте следующие параметры.

| Параметр |Область применения| Значение |
|----|---|---|
| Имя кластера, |always| yourclustername |
| Версия Databricks Runtime |always| Среда выполнения 7,1 (Scala 2,21, Spark 3.0.0) — не МАШИНное обучение|
| Версия Python |always| 3 |
| Тип рабочего процесса <br>(определяет максимальное число одновременных итераций) |Автоматическое Машинное обучение<br>только| Предпочитается виртуальная машина, оптимизированная для операций в памяти |
| Рабочие роли |always| 2 или больше |
| Включение автомасштабирования |Автоматическое Машинное обучение<br>только| Снять пометку |

Дождитесь запуска кластера, прежде чем продолжать.

## <a name="add-the-azure-ml-sdk-to-databricks"></a>Добавление пакета SDK для машинного обучения Azure в кирпичи

После запуска кластера [Создайте библиотеку](https://docs.databricks.com/user-guide/libraries.html#create-a-library) , чтобы присоединить соответствующий пакет SDK машинное обучение Azure к кластеру. 

Чтобы использовать автоматизированное ML, перейдите к [добавлению пакета SDK для машинного обучения Azure с помощью аутомл](#add-the-azure-ml-sdk-with-automl-to-databricks).


1. Щелкните правой кнопкой мыши текущую папку рабочей области, в которой нужно сохранить библиотеку. Выберите **создать**  >  **библиотеку**.
    
    > [!TIP]
    > Если у вас старая версия пакета SDK, отмените ее выбор из установленных библиотек кластера и перейдите в корзину. Установите новую версию пакета SDK и перезапустите кластер. Если после перезагрузки возникла ошибка, отключите и повторно подключите кластер.

1. Выберите следующий параметр (другие установки пакета SDK не поддерживаются).

   |&nbsp;Дополнительные возможности пакета SDK &nbsp;|Источник|&nbsp;Имя PyPi&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
   |----|---|---|
   |Для кирпичей| Отправка Python Egg или PyPI | azureml-sdk[databricks]|

   > [!WARNING]
   > Другие дополнительные компоненты SDK устанавливаться не могут. Выберите только параметр [ `databricks` ].

   * Не выбирайте **Автоматическое присоединение ко всем кластерам**.
   * Выберите  **присоединить** рядом с именем кластера.

1. Отслеживать ошибки до тех пор, пока состояние не изменится на " **подключено**", что может занять несколько минут.  Если этот шаг завершается ошибкой:

   Попробуйте перезапустить кластер, выполнив следующие действия.
   1. В левой области щелкните **Кластеры**.
   1. Выберите имя кластера в таблице.
   1. На вкладке **Библиотеки** щелкните **Перезапустить**.

   Успешная установка выглядит следующим образом: 

  ![Пакет SDK Машинное обучение Azure для кирпичей](./media/how-to-configure-environment/amlsdk-withoutautoml.jpg) 

## <a name="add-the-azure-ml-sdk-with-automl-to-databricks"></a>Добавление пакета SDK машинного обучения Azure с Аутомл в кирпичи
Если кластер был создан с Databricks Runtime 7,1 или более поздней версии (*не* в машинном коде), выполните следующую команду в первой ячейке записной книжки, чтобы установить пакет SDK для AML.

```
%pip install --upgrade --force-reinstall -r https://aka.ms/automl_linux_requirements.txt
```
Для Databricks Runtime 7,0 и более низкого уровня установите пакет SDK для Машинное обучение Azure с помощью [скрипта init](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks/automl/README.md).

### <a name="automl-config-settings"></a>Параметры конфигурации Аутомл

В Аутомл config при использовании Azure Databricks добавьте следующие параметры:

- ```max_concurrent_iterations``` основано на количестве рабочих узлов в кластере.
- ```spark_context=sc``` основывается на контексте Spark по умолчанию.

## <a name="ml-notebooks-that-work-with-azure-databricks"></a>Записные книжки ML, работающие с Azure Databricks

Попробуйте продукт:
+ Несмотря на то что доступны многие примеры записных книжек, **с Azure Databricks работают только [эти образцы записных книжек](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/azure-databricks) .**

+ Импортируйте эти примеры непосредственно из рабочей области. См. ниже: ![ выберите Импорт ](./media/how-to-configure-environment/azure-db-screenshot.png)
 ![ панели импорта](./media/how-to-configure-environment/azure-db-import.png)

+ Узнайте, как [создать конвейер с модулями обработки в качестве обучающего вычислений](./how-to-create-machine-learning-pipelines.md).

## <a name="troubleshooting"></a>Устранение неполадок

* Для **отмены автоматического запуска машинного обучения отменяются**: при использовании средств автоматического машинного обучения в Azure Databricks для отмены запуска и запуска нового эксперимента перезапустите кластер Azure Databricks.

* **Модуль данных >10 итераций для автоматического машинного обучения**: в автоматических параметрах машинного обучения при наличии более 10 итераций задайте значение `show_output` `False` при отправке выполнения.

* **Мини-приложение "кирпичи" для машинное обучение Azure пакета SDK и автоматизированного машинного обучения**. в записной книжке для модуля "приложения" не поддерживается приложение машинное обучение Azure SDK, так как записные книжки не могут анализировать элементы HTML. Вы можете просмотреть мини-приложение на портале с помощью этого кода Python в Azure Databricks записной книжке:

    ```
    displayHTML("<a href={} target='_blank'>Azure Portal: {}</a>".format(local_run.get_portal_url(), local_run.id))
    ```

* **Сбой при установке пакетов**

    Сбой установки пакета SDK Машинное обучение Azure на Azure Databricks при установке дополнительных пакетов. Некоторые пакеты, такие как `psutil`, могут приводить к конфликтам. Чтобы избежать ошибок установки, установите пакеты, зафиксировать версию библиотеки. Эта проблема связана с модулями связи, а не с пакетом SDK для Машинное обучение Azure. Эта проблема также может возникнуть и в других библиотеках. Пример
    
    ```python
    psutil cryptography==1.5 pyopenssl==16.0.0 ipython==2.2.0
    ```

    Кроме того, можно использовать скрипты init, если возникают проблемы с установкой библиотек Python. Этот подход официально не поддерживается. Дополнительные сведения см. в разделе [сценарии инициализации с областью действия кластера](/azure/databricks/clusters/init-scripts#cluster-scoped-init-scripts).

* **Ошибка импорта: не удается импортировать `Timedelta` имя `pandas._libs.tslibs` из**: Если вы видите эту ошибку при использовании автоматического машинного обучения, выполните в записной книжке две следующие строки:
    ```
    %sh rm -rf /databricks/python/lib/python3.7/site-packages/pandas-0.23.4.dist-info /databricks/python/lib/python3.7/site-packages/pandas
    %sh /databricks/python/bin/pip install pandas==0.23.4
    ```

* **Ошибка импорта: отсутствует модуль с именем "Pandas. Core. indexes"**: Если вы видите эту ошибку при использовании автоматического машинного обучения, выполните следующие действия.

    1. Выполните следующую команду, чтобы установить два пакета в кластере Azure Databricks:
    
       ```bash
       scikit-learn==0.19.1
       pandas==0.22.0
       ```
    
    1. Отсоедините и снова подключите кластер к записной книжке.
    
    Если эти действия не устранят проблему, попробуйте перезапустить кластер.

* **Фаилтосендфеасер**. Если при `FailToSendFeather` чтении данных в кластере Azure Databricks возникает ошибка, см. следующие решения.
    
    * Обновите `azureml-sdk[automl]` пакет до последней версии.
    * Добавьте `azureml-dataprep` версию 1.1.8 или более позднюю.
    * Добавьте `pyarrow` версию 0,11 или более позднюю.
  

## <a name="next-steps"></a>Дальнейшие действия

- [Обучение модели](tutorial-train-models-with-aml.md) на машинное обучение Azure с помощью набора данных MNIST.
- См. [Справочник по машинное обучение Azure SDK для Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py).
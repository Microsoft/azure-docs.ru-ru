---
title: Запуск Машинное обучение Azure рабочих нагрузок на Apache Spark в HDInsight
description: Узнайте, как выполнять рабочие нагрузки Машинного обучения Azure с помощью автоматического машинного обучения в Apache Spark в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/13/2019
ms.openlocfilehash: 4087341a9a96ae56c00972f886ce3cc8891750a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929728"
---
# <a name="run-azure-machine-learning-workloads-with-automated-machine-learning-on-apache-spark-in-hdinsight"></a>Запуск Машинное обучение Azure рабочих нагрузок с помощью автоматического машинного обучения на Apache Spark в HDInsight

Машинное обучение Azure упрощает и ускоряет создание, обучение и развертывание моделей машинного обучения. В автоматизированном машинном обучении (Аутомл) вы начинаете с обучающих данных, которые имеют определенную целевую функцию, а затем просматриваете сочетания алгоритмов и выбора компонентов, чтобы автоматически выбирать оптимальную модель для данных на основе оценок обучения. HDInsight позволяет клиентам подготавливать кластеры с сотнями узлов. Аутомл, работающий в Spark в кластере HDInsight, позволяет пользователям использовать ресурсы вычислений на этих узлах для выполнения заданий обучения в масштабном режиме и для параллельного выполнения нескольких заданий обучения. Это позволяет пользователям выполнять эксперименты Аутомл при совместном использовании вычислений с другими рабочими нагрузками больших данных.

## <a name="install-azure-machine-learning-on-an-hdinsight-cluster"></a>Установка Машинного обучения Azure в кластере HDInsight

Общие руководства по автоматическому машинному обучению см. [в разделе Учебник. Использование автоматизированного машинного обучения для создания модели регрессии](../../machine-learning/tutorial-auto-train-models.md).
Все новые кластеры HDInsight-Spark предварительно установлены с помощью пакета SDK для AzureML-AutoML.

> [!Note]
> Пакеты Машинного обучения Azure устанавливаются в среду conda Python3. Установленный Jupyter Notebook следует запускать с помощью ядра PySpark3.

Для использования Аутомл также можно использовать записные книжки Zeppelin.

> [!Note]
> В Zeppelin есть [известная проблема](https://community.hortonworks.com/content/supportkb/207822/the-livypyspark3-interpreter-uses-python-2-instead.html) , из-за которой PySpark3 не выбирает нужную версию Python. Используйте документированные обходные решения.

## <a name="authentication-for-workspace"></a>Аутентификация для рабочей области

Для создания рабочей области и отправки эксперимента требуется маркер аутентификации. Этот маркер можно создать с помощью [приложения Azure AD](../../active-directory/develop/app-objects-and-service-principals.md). Для создания требуемого маркера аутентификации также можно использовать [пользователя Azure AD](/azure/python/python-sdk-azure-authenticate), если многофакторная проверка подлинности не включена в учетной записи.  

Следующий фрагмент кода создает маркер аутентификации, используя **приложение Azure AD**.

```python
from azureml.core.authentication import ServicePrincipalAuthentication
auth_sp = ServicePrincipalAuthentication(
    tenant_id='<Azure Tenant ID>',
    service_principal_id='<Azure AD Application ID>',
    service_principal_password='<Azure AD Application Key>'
)
```

Следующий фрагмент кода создает маркер аутентификации, используя **пользователя Azure AD**.

```python
from azure.common.credentials import UserPassCredentials
credentials = UserPassCredentials('user@domain.com', 'my_smart_password')
```

## <a name="loading-dataset"></a>Загрузка набора данных

Автоматическое машинное обучение в Spark использует **потоки данных**, которые являются неизменяемыми операциями с данными с отложенным вычислением.  Поток данных может загрузить набор данных из большого двоичного объекта с общедоступным доступом на чтение или с URL-адреса большого двоичного объекта с маркером SAS.

```python
import azureml.dataprep as dprep

dataflow_public = dprep.read_csv(
    path='https://commonartifacts.blob.core.windows.net/automl/UCI_Adult_train.csv')

dataflow_with_token = dprep.read_csv(
    path='https://dpreptestfiles.blob.core.windows.net/testfiles/read_csv_duplicate_headers.csv?st=2018-06-15T23%3A01%3A42Z&se=2019-06-16T23%3A01%3A00Z&sp=r&sv=2017-04-17&sr=b&sig=ugQQCmeC2eBamm6ynM7wnI%2BI3TTDTM6z9RPKj4a%2FU6g%3D')
```

Вы также можете зарегистрировать хранилище данных в рабочей области с использованием однократной регистрации.

## <a name="experiment-submission"></a>Отправка эксперимента

В [конфигурации автоматического машинного обучения](/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig)свойство `spark_context` должно быть установлено для запуска пакета в распределенном режиме. Свойство `concurrent_iterations`, которое является максимальным числом параллельно выполняемых итераций, должно быть меньше количества ядер исполнителя для приложения Spark.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о мотивации автоматизированного машинного обучения см. в разделе [модели выпуска с помощью автоматизированного машинного обучения Майкрософт.](https://azure.microsoft.com/blog/release-models-at-pace-using-microsoft-s-automl/)
* Дополнительные сведения об использовании автоматизированных возможностей ML в машинном [обучении Azure см. в разделе новые возможности автоматического машинного обучения в машинное обучение Azure](https://azure.microsoft.com/blog/new-automated-machine-learning-capabilities-in-azure-machine-learning-service/)
* [Проект AutoML от Microsoft Research](https://www.microsoft.com/research/project/automl/)

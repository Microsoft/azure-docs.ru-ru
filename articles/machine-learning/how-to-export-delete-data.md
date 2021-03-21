---
title: Экспорт или удаление данных рабочей области
titleSuffix: Azure Machine Learning
description: Узнайте, как экспортировать или удалить рабочую область с помощью Машинное обучение Azure Studio, интерфейса командной строки, пакета SDK и прошедших проверку подлинности интерфейсов API RESTFUL.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
author: lobrien
ms.author: laobri
ms.date: 04/24/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 030e09f0bb1514e8d3828da52604d751a2a8a11a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102520090"
---
# <a name="export-or-delete-your-machine-learning-service-workspace-data"></a>Экспорт или удаление данных рабочей области Службы машинного обучения

В Машинное обучение Azure можно экспортировать или удалить данные рабочей области с помощью графического интерфейса портала или пакета SDK для Python. В этой статье описываются оба варианта.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## <a name="control-your-workspace-data"></a>Управление данными рабочей области

Данные в продукте, хранящиеся Машинное обучение Azure, доступны для экспорта и удаления. Экспортировать и удалить можно с помощью Машинное обучение Azure Studio, CLI и пакета SDK. Доступ к данным телеметрии может осуществляться через портал конфиденциальности Azure. 

В Машинное обучение Azure личные данные состоят из сведений о пользователе в документах журнала выполнения. 

## <a name="delete-high-level-resources-using-the-portal"></a>Удаление высокоуровневого ресурса с помощью портала

При создании рабочей области Azure создает ряд ресурсов в группе ресурсов:

- Сама Рабочая область
- Учетная запись хранения
- Реестр контейнеров
- Экземпляр Application Insights
- Хранилище ключей

Эти ресурсы можно удалить, выбрав их в списке и выбрав **Удалить** . 

:::image type="content" source="media/how-to-export-delete-data/delete-resource-group-resources.png" alt-text="Снимок экрана портала с выделенным значком удаления":::

Документы журнала выполнения, которые могут содержать личные сведения о пользователе, хранятся в учетной записи хранения в хранилище BLOB-объектов во вложенных папках `/azureml` . Вы можете скачать и удалить данные с портала.

:::image type="content" source="media/how-to-export-delete-data/storage-account-folders.png" alt-text="Снимок экрана: Каталог azureml в учетной записи хранения на портале":::

## <a name="export-and-delete-machine-learning-resources-using-azure-machine-learning-studio"></a>Экспорт и удаление ресурсов машинного обучения с помощью Машинное обучение Azure Studio

Машинное обучение Azure Studio предоставляет единое представление ресурсов машинного обучения, таких как записные книжки, наборы данных, модели и эксперименты. Машинное обучение Azure Studio подчеркивает сохранение данных и экспериментов. Вычислительные ресурсы, такие как конвейеры и вычислительные ресурсы, можно удалить с помощью браузера. Для этих ресурсов перейдите к интересующему ресурсу и нажмите кнопку **Удалить**. 

Для наборов данных можно выполнить отмену регистрации, а эксперименты можно заархивировать, но эти операции не удаляют данные. Чтобы полностью удалить данные, необходимо удалить наборы данных и данные запуска на уровне хранилища. Удаление на уровне хранилища выполняется с помощью портала, как описано выше.

Вы можете скачать обучающие артефакты из экспериментальных запусков с помощью студии. Выберите **эксперимент** и **запустите** его. Выберите **выходные данные + журналы** и перейдите к конкретным артефактам, которые вы хотите скачать. Выберите **...** и **Скачайте**.

Чтобы скачать зарегистрированную модель, перейдите к нужной **модели** и выберите **загрузить**. 

:::image type="contents" source="media/how-to-export-delete-data/model-download.png" alt-text="Снимок экрана: страница модели Studio с выделенным параметром загрузки":::

## <a name="export-and-delete-resources-using-the-python-sdk"></a>Экспорт и удаление ресурсов с помощью пакета SDK для Python

Вы можете скачать выходные данные определенного запуска, используя: 

```python
# Retrieved from Azure Machine Learning web UI
run_id = 'aaaaaaaa-bbbb-cccc-dddd-0123456789AB'
experiment = ws.experiments['my-experiment']
run = next(run for run in ex.get_runs() if run.id == run_id)
metrics_output_port = run.get_pipeline_output('metrics_output')
model_output_port = run.get_pipeline_output('model_output')

metrics_output_port.download('.', show_progress=True)
model_output_port.download('.', show_progress=True)
```

С помощью пакета SDK для Python можно удалить следующие ресурсы машинного обучения: 

| Type | Вызов функции | Примечания | 
| --- | --- | --- |
| `Workspace` | [`delete`](/python/api/azureml-core/azureml.core.workspace.workspace#delete-delete-dependent-resources-false--no-wait-false-) | Используйте `delete-dependent-resources` для каскадного удаления |
| `Model` | [`delete`](/python/api/azureml-core/azureml.core.model%28class%29#delete--) | | 
| `ComputeTarget` | [`delete`](/python/api/azureml-core/azureml.core.computetarget#delete--) | |
| `WebService` | [`delete`](/python/api/azureml-core/azureml.core.webservice%28class%29) | |
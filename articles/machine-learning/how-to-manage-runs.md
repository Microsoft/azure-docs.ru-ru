---
title: Запуск, отслеживание и отмена обучающих запусков в Python
titleSuffix: Azure Machine Learning
description: Узнайте, как запускать, отслеживать и отслеживать запуск эксперимента машинного обучения с помощью пакета SDK для Машинное обучение Azure Python.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: roastala
author: rastala
manager: cgronlun
ms.reviewer: nibaccam
ms.date: 03/04/2021
ms.topic: conceptual
ms.custom: how-to, devx-track-python, devx-track-azurecli
ms.openlocfilehash: 26880fd6e3688dd95cc9f16072a35d5c4ce7c31e
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105110276"
---
# <a name="start-monitor-and-track-run-history"></a>Запуск, отслеживание и отслеживание журнала выполнения 

[Машинное обучение Azure SDK для Python](/python/api/overview/azure/ml/intro), [Машинное обучение CLI](reference-azure-machine-learning-cli.md)и [машинное обучение Azure Studio](https://ml.azure.com) предоставляют различные методы мониторинга, упорядочения и отслеживания запусков для обучения и экспериментирования. Журнал выполнения машинного обучения является важной частью подробного и повторяемого процесса разработки машинного обучения.

В этой статье показано, как выполнить следующие задачи.

* Мониторинг производительности выполнения.
* Отслеживание состояния выполнения по электронной почте.
* Теги и поиск запусков.
* Добавьте описание запуска. 
* Выполните поиск по журналу выполнения. 
* Отмена или сбой выполнения.
* Создание дочерних запусков.
 

> [!TIP]
> Дополнительные сведения о мониторинге службы Машинное обучение Azure и связанных служб Azure см. [в разделе мониторинг машинное обучение Azure](monitor-azure-machine-learning.md).
> Сведения о мониторинге моделей, развернутых в виде веб-служб или модулей IoT Edge, см. в разделе [сбора данных модели](how-to-enable-data-collection.md) и [мониторинга с помощью Application Insights](how-to-enable-app-insights.md).

## <a name="prerequisites"></a>Предварительные требования

Вам потребуются следующие элементы:

* Подписка Azure. Если у вас еще нет подписки Azure, создайте бесплатную учетную запись, прежде чем начинать работу. Опробуйте [бесплатную или платную версию Машинного обучения Azure](https://aka.ms/AMLFree) уже сегодня.

* [Рабочая область машинное обучение Azure](how-to-manage-workspace.md).

* Машинное обучение Azure SDK для Python (версия 1.0.21 или более поздняя). Чтобы установить или обновить последнюю версию пакета SDK, см. статью [Установка или обновление пакета SDK](/python/api/overview/azure/ml/install).

    Чтобы проверить версию пакета SDK для Машинное обучение Azure, используйте следующий код:

    ```python
    print(azureml.core.VERSION)
    ```

* Расширение [Azure CLI](/cli/azure/?preserve-view=true&view=azure-cli-latest) и [CLI для машинное обучение Azure](reference-azure-machine-learning-cli.md).


## <a name="monitor-run-performance"></a>Мониторинг производительности выполнения

* Запуск запуска и процесса ведения журнала

    # <a name="python"></a>[Python](#tab/python)
    
    1. Настройте эксперимент, импортировав классы [рабочей области](/python/api/azureml-core/azureml.core.workspace.workspace), [эксперимент](/python/api/azureml-core/azureml.core.experiment.experiment), [Run](/python/api/azureml-core/azureml.core.run%28class%29)и [скриптрунконфиг](/python/api/azureml-core/azureml.core.scriptrunconfig) из пакета [azureml. Core](/python/api/azureml-core/azureml.core) .
    
        ```python
        import azureml.core
        from azureml.core import Workspace, Experiment, Run
        from azureml.core import ScriptRunConfig
        
        ws = Workspace.from_config()
        exp = Experiment(workspace=ws, name="explore-runs")
        ```
    
    1. Запустите запуск и процесс ведения журнала с помощью [`start_logging()`](/python/api/azureml-core/azureml.core.experiment%28class%29#start-logging--args----kwargs-) метода.
    
        ```python
        notebook_run = exp.start_logging()
        notebook_run.log(name="message", value="Hello from run!")
        ```
        
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    Чтобы начать выполнение эксперимента, выполните следующие действия.
    
    1. В оболочке или командной строке используйте Azure CLI для проверки подлинности в подписке Azure:
    
        ```azurecli-interactive
        az login
        ```
        
        [!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 
    
    1. Подключите конфигурацию рабочей области к папке, содержащей сценарий обучения. Замените `myworkspace` рабочей областью машинное обучение Azure. Замените на `myresourcegroup` группу ресурсов Azure, содержащую рабочую область:
    
        ```azurecli-interactive
        az ml folder attach -w myworkspace -g myresourcegroup
        ```
    
        Эта команда создает подкаталог `.azureml`, содержащий примеры файлов runconfig и среды Conda. Он также содержит файл `config.json`, который используется для взаимодействия с рабочей областью Машинного обучения Azure.
    
        См. дополнительные сведения о команде [az ml folder attach](/cli/azure/ext/azure-cli-ml/ml/folder?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-folder-attach).
    
    2. Чтобы начать выполнение, используйте следующую команду. При использовании этой команды укажите имя файла runconfig (текст до \* . runconfig, если вы ищете файловую систему) в параметре-c.
    
        ```azurecli-interactive
        az ml run submit-script -c sklearn -e testexperiment train.py
        ```
    
        > [!TIP]
        > `az ml folder attach`Команда создала `.azureml` подкаталог, который содержит два примера файлов runconfig.
        >
        > При наличии скрипта Python, который программно создает объект конфигурации запуска, вы можете использовать метод [RunConfig.Save()](/python/api/azureml-core/azureml.core.runconfiguration#save-path-none--name-none--separate-environment-yaml-false-), чтобы сохранить его как файл runconfig.
        >
        > Дополнительные примеры файлов runconfig см. в разделе [https://github.com/MicrosoftDocs/pipelines-azureml/](https://github.com/MicrosoftDocs/pipelines-azureml/) .
    
        См. дополнительные сведения о команде [az ml run submit-script](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-submit-script).

    # <a name="studio"></a>[Студия](#tab/azure-studio)

    Пример обучения модели в конструкторе Машинное обучение Azure см. в разделе [учебник. Прогнозирование цен автомобилей с помощью конструктора](tutorial-designer-automobile-price-train-score.md).

    ---

* Наблюдение за состоянием запуска

    # <a name="python"></a>[Python](#tab/python)
    
    * Получение состояния выполнения с помощью [`get_status()`](/python/api/azureml-core/azureml.core.run%28class%29#get-status--) метода.
    
        ```python
        print(notebook_run.get_status())
        ```
    
    * Чтобы получить идентификатор выполнения, время выполнения и другие сведения о выполнении, используйте [`get_details()`](/python/api/azureml-core/azureml.core.workspace.workspace#get-details--) метод.
    
        ```python
        print(notebook_run.get_details())
        ```
    
    * Когда выполнение завершится успешно, используйте [`complete()`](/python/api/azureml-core/azureml.core.run%28class%29#complete--set-status-true-) метод, чтобы пометить его как завершенный.
    
        ```python
        notebook_run.complete()
        print(notebook_run.get_status())
        ```
    
    * Если вы используете `with...as` шаблон разработки Python, выполнение автоматически помечается как завершенное, если выполнение выходит за пределы области. Не нужно вручную помечать запуск как завершенный.
        
        ```python
        with exp.start_logging() as notebook_run:
            notebook_run.log(name="message", value="Hello from run!")
            print(notebook_run.get_status())
        
        print(notebook_run.get_status())
        ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    * Чтобы просмотреть список запусков для эксперимента, используйте следующую команду. Замените на `experiment` имя своего эксперимента:
    
        ```azurecli-interactive
        az ml run list --experiment-name experiment
        ```
    
        Эта команда возвращает документ JSON, в котором перечисляются сведения о выполнении этого эксперимента.
    
        См. дополнительные сведения о команде [az ml experiment list](/cli/azure/ext/azure-cli-ml/ml/experiment?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-experiment-list).
    
    * Чтобы просмотреть сведения о конкретном выполнении, используйте следующую команду. Замените на `runid` идентификатор запуска:
    
        ```azurecli-interactive
        az ml run show -r runid
        ```
    
        Эта команда возвращает документ JSON, в котором перечисляются сведения о выполнении.
    
        Дополнительные сведения см. в разделе [AZ ML Run показ](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-show).
    
    
    # <a name="studio"></a>[Студия](#tab/azure-studio)
    
    Чтобы просмотреть свои запуски в студии, сделайте следующее: 
    
    1. Перейдите на вкладку **эксперименты** .
    
    1. Выберите любой из **экспериментов** , чтобы просмотреть все запуски в эксперименте, или выберите **все запуски** , чтобы просмотреть все запуски, отправленные в рабочей области.
    
        На странице **все запуски** можно отфильтровать список запусков по тегам, экспериментам, целевым параметрам вычислений и многое другое для лучшего упорядочения и определения области работы.  
    
    1. Внесите настройки на странице, выбрав запуски для сравнения, добавляя диаграммы или применяя фильтры. Эти изменения можно сохранить в виде **пользовательского представления** , чтобы можно было легко вернуться к работе. Пользователи с разрешениями рабочей области могут изменять или просматривать пользовательское представление. Кроме того, предоставьте общий доступ к пользовательскому представлению членам группы для расширенной совместной работы, выбрав **представление общего доступа**.   
    
        :::image type="content" source="media/how-to-manage-runs/custom-views.gif" alt-text="Снимок экрана: создание пользовательского представления":::
    
    1. Чтобы просмотреть журналы выполнения, выберите конкретный запуск и на вкладке **выходные данные + журналы** можно найти журналы диагностики и ошибок для выполнения.
    
    ---

## <a name="monitor-the-run-status-by-email-notification"></a>Отслеживание состояния выполнения по электронной почте уведомление

1. В [портал Azure](https://ms.portal.azure.com/)на панели навигации слева выберите вкладку **монитор** . 

1. Выберите **параметры диагностики** , а затем выберите **+ Добавить параметр диагностики**.

    ![Снимок экрана параметров диагностики для уведомления по электронной почте](./media/how-to-manage-runs/diagnostic-setting.png)

1. В параметре диагностики 
    1. в разделе **сведения о категории** выберите **амлрунстатусчанжедевент**. 
    1. В **сведениях о назначении** выберите **рабочую область отправить в log Analytics**  и укажите **подписку** и **log Analytics рабочую область**. 

    > [!NOTE]
    > **Рабочая область azure log Analytics** — это ресурс Azure другого типа, отличный от **рабочей области службы машинное обучение Azure**. Если в этом списке нет параметров, можно [создать рабочую область log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/quick-create-workspace). 
    
    ![Где сохранить уведомление по электронной почте](./media/how-to-manage-runs/log-location.png)

1. На вкладке **журналы** добавьте **новое правило генерации оповещений**. 

    ![Новое правило генерации оповещений](./media/how-to-manage-runs/new-alert-rule.png)

1. Узнайте, [как создавать оповещения журналов и управлять ими с помощью Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-log).

## <a name="run-description"></a>Описание запуска 

Описание запуска можно добавить в выполнение, чтобы предоставить больше контекста и сведений для выполнения. Можно также выполнить поиск по этим описаниям из списка запусков и добавить описание запуска в качестве столбца в список запусков. 

Перейдите на страницу **сведений о запуске** для выполнения и щелкните значок редактирования или карандаша, чтобы добавить, изменить или удалить описания для выполнения. Чтобы сохранить изменения в списке запуски, сохраните изменения в существующем настраиваемом представлении или в новом пользовательском представлении. Формат Markdown поддерживается для описаний запуска, что позволяет внедрять и глубокое связывание изображений, как показано ниже.

:::image type="content" source="media/how-to-manage-runs/run-description.gif" alt-text="Снимок экрана: создание описания запуска"::: 

## <a name="tag-and-find-runs"></a>Теги и поиск запусков

В Машинное обучение Azure можно использовать свойства и теги для упорядочения и запроса своих запусков для получения важной информации.

* Добавление свойств и тегов

    # <a name="python"></a>[Python](#tab/python)
    
    Чтобы добавить метаданные для поиска в запуски, используйте [`add_properties()`](/python/api/azureml-core/azureml.core.run%28class%29#add-properties-properties-) метод. Например, следующий код добавляет `"author"` свойство в Run:
    
    ```Python
    local_run.add_properties({"author":"azureml-user"})
    print(local_run.get_properties())
    ```
    
    Свойства являются неизменяемыми, поэтому они создают постоянную запись для целей аудита. В следующем примере кода возникает ошибка, так как мы уже добавили в `"azureml-user"` качестве `"author"` значения свойства в приведенном выше коде:
    
    ```Python
    try:
        local_run.add_properties({"author":"different-user"})
    except Exception as e:
        print(e)
    ```
    
    В отличие от свойств, теги являются изменяемыми. Чтобы добавить доступную для поиска и осмысленную информацию для потребителей вашего эксперимента, используйте [`tag()`](/python/api/azureml-core/azureml.core.run%28class%29#tag-key--value-none-) метод.
    
    ```Python
    local_run.tag("quality", "great run")
    print(local_run.get_tags())
    
    local_run.tag("quality", "fantastic run")
    print(local_run.get_tags())
    ```
    
    Можно также добавить простые строковые Теги. Если эти теги отображаются в словаре тегов в качестве ключей, они имеют значение `None` .
    
    ```Python
    local_run.tag("worth another look")
    print(local_run.get_tags())
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    > [!NOTE]
    > С помощью интерфейса командной строки можно добавлять или обновлять только теги.
    
    Чтобы добавить или обновить тег, используйте следующую команду:
    
    ```azurecli-interactive
    az ml run update -r runid --add-tag quality='fantastic run'
    ```
    
    Дополнительные сведения см. в разделе [AZ ML Run Update](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-update).
    
    # <a name="studio"></a>[Студия](#tab/azure-studio)
    
    В студии можно добавлять, изменять и удалять теги Run. Перейдите на страницу **сведений о выполнении** для выполнения и щелкните значок редактирования или карандаша, чтобы добавить, изменить или удалить теги для выполнения. Вы также можете выполнять поиск и фильтрацию по этим тегам со страницы списка запусков.
    
    :::image type="content" source="media/how-to-manage-runs/run-tags.gif" alt-text="Снимок экрана: Добавление, изменение и удаление тегов запуска":::
    
    ---

* Свойства и Теги запроса

    Вы можете запросить запуски в эксперименте, чтобы получить список запусков, соответствующих указанным свойствам и тегам.

    # <a name="python"></a>[Python](#tab/python)
    
    ```Python
    list(exp.get_runs(properties={"author":"azureml-user"},tags={"quality":"fantastic run"}))
    list(exp.get_runs(properties={"author":"azureml-user"},tags="worth another look"))
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    Azure CLI поддерживает запросы [JMESPath](http://jmespath.org) , которые можно использовать для фильтрации запусков на основе свойств и тегов. Чтобы использовать запрос JMESPath с Azure CLI, укажите его с помощью `--query` параметра. В следующих примерах показаны некоторые запросы, использующие свойства и теги.
    
    ```azurecli-interactive
    # list runs where the author property = 'azureml-user'
    az ml run list --experiment-name experiment [?properties.author=='azureml-user']
    # list runs where the tag contains a key that starts with 'worth another look'
    az ml run list --experiment-name experiment [?tags.keys(@)[?starts_with(@, 'worth another look')]]
    # list runs where the author property = 'azureml-user' and the 'quality' tag starts with 'fantastic run'
    az ml run list --experiment-name experiment [?properties.author=='azureml-user' && tags.quality=='fantastic run']
    ```
    
    Дополнительные сведения о запросах Azure CLI результатов см. в разделе [запрос Azure CLI команды Output](/cli/azure/query-azure-cli?preserve-view=true&view=azure-cli-latest).
    
    # <a name="studio"></a>[Студия](#tab/azure-studio)
    
    Чтобы найти конкретные запуски, перейдите к списку  **все запуски** . У вас есть два варианта:
    
    1. Используйте кнопку **Добавить фильтр** и выберите фильтр по тегам, чтобы отфильтровать выполнений по тегу, который был назначен запускам. <br><br>
    OR
    
    1. Используйте панель поиска для быстрого поиска запусков, выполнив поиск по метаданным запуска, таким как состояние выполнения, описания, имена экспериментов и имя отправителя. 
    
## <a name="cancel-or-fail-runs"></a>Отмена или неудача выполнения

Если вы заметили ошибку или если выполнение занимает слишком много времени, можно отменить запуск.

# <a name="python"></a>[Python](#tab/python)

Чтобы отменить запуск с помощью пакета SDK, используйте [`cancel()`](/python/api/azureml-core/azureml.core.run%28class%29#cancel--) метод:

```python
src = ScriptRunConfig(source_directory='.', script='hello_with_delay.py')
local_run = exp.submit(src)
print(local_run.get_status())

local_run.cancel()
print(local_run.get_status())
```

Если выполнение завершается, но содержит ошибку (например, был использован неверный сценарий обучения), можно использовать [`fail()`](/python/api/azureml-core/azureml.core.run%28class%29#fail-error-details-none--error-code-none---set-status-true-) метод, чтобы пометить его как неудачный.

```python
local_run = exp.submit(src)
local_run.fail()
print(local_run.get_status())
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы отменить запуск с помощью интерфейса командной строки, используйте следующую команду. Замените на `runid` идентификатор запуска

```azurecli-interactive
az ml run cancel -r runid -w workspace_name -e experiment_name
```

Дополнительные сведения см. в статье [AZ ML Run отмена](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest#ext-azure-cli-ml-az-ml-run-cancel).

# <a name="studio"></a>[Студия](#tab/azure-studio)

Чтобы отменить запуск в студии, выполните следующие действия.

1. Перейдите к выполняющемуся конвейеру в разделе **эксперименты** или **конвейеры** . 

1. Выберите номер выполнения конвейера, который нужно отменить.

1. На панели инструментов нажмите **кнопку Отмена** .

---

## <a name="create-child-runs"></a>Создание дочерних запусков

Создайте дочерние выполнения для объединения связанных запусков, например для различных итераций настройки параметров.

> [!NOTE]
> Дочерние запуски можно создать только с помощью пакета SDK.

В этом примере кода используется `hello_with_children.py` скрипт для создания пакета из пяти дочерних запусков из отправленного запуска с помощью [`child_run()`](/python/api/azureml-core/azureml.core.run%28class%29#child-run-name-none--run-id-none--outputs-none-) метода:

```python
!more hello_with_children.py
src = ScriptRunConfig(source_directory='.', script='hello_with_children.py')

local_run = exp.submit(src)
local_run.wait_for_completion(show_output=True)
print(local_run.get_status())

with exp.start_logging() as parent_run:
    for c,count in enumerate(range(5)):
        with parent_run.child_run() as child:
            child.log(name="Hello from child run", value=c)
```

> [!NOTE]
> При выходе из области действия дочерние тесты автоматически помечаются как завершенные.

Чтобы эффективно создать множество дочерних запусков, используйте [`create_children()`](/python/api/azureml-core/azureml.core.run.run#create-children-count-none--tag-key-none--tag-values-none-) метод. Поскольку каждое создание приводит к сетевому вызову, создание пакета выполняется более эффективно, чем создание их по одному.

### <a name="submit-child-runs"></a>Отправить дочерние выполнения

Дочерние запуски также могут быть отправлены из родительского запуска. Это позволяет создавать иерархии родительских и дочерних запусков. Невозможно создать родительский дочерний элемент. даже если родительский запуск не выполняет никаких действий, но запускает дочерний, все равно необходимо создать иерархию. Состояния всех запусков являются независимыми: родительский элемент может находиться в `"Completed"` успешном состоянии, даже если один или несколько дочерних запусков были отменены или завершились сбоем.  

Может потребоваться, чтобы ваш ребенок мог использовать конфигурацию запуска, отличную от конфигурации родительского запуска. Например, вы можете использовать менее мощную конфигурацию на основе ЦП для родительского элемента, а также использовать конфигурации на основе GPU для своих детей. Другим распространенным желанием является передача каждого дочернего различных аргументов и данных. Чтобы настроить дочерний запуск, создайте `ScriptRunConfig` объект для дочернего запуска. Приведенный ниже код:

- Извлекает ресурс вычислений с именем `"gpu-cluster"` из рабочей области. `ws`
- Выполняет перебор различных значений аргументов для передачи дочерним `ScriptRunConfig` объектам
- Создает и отправляет новый дочерний запуск с использованием настраиваемого ресурса вычислений и аргумента.
- Блокируется до завершения всех дочерних запусков

```python
# parent.py
# This script controls the launching of child scripts
from azureml.core import Run, ScriptRunConfig

compute_target = ws.compute_targets["gpu-cluster"]

run = Run.get_context()

child_args = ['Apple', 'Banana', 'Orange']
for arg in child_args: 
    run.log('Status', f'Launching {arg}')
    child_config = ScriptRunConfig(source_directory=".", script='child.py', arguments=['--fruit', arg], compute_target=compute_target)
    # Starts the run asynchronously
    run.submit_child(child_config)

# Experiment will "complete" successfully at this point. 
# Instead of returning immediately, block until child runs complete

for child in run.get_children():
    child.wait_for_completion()
```

Чтобы создать множество дочерних запусков с одинаковыми конфигурациями, аргументами и входными данными, используйте [`create_children()`](/python/api/azureml-core/azureml.core.run.run#create-children-count-none--tag-key-none--tag-values-none-) метод. Поскольку каждое создание приводит к сетевому вызову, создание пакета выполняется более эффективно, чем создание их по одному.

В дочернем выполнении можно просмотреть идентификатор родительского запуска:

```python
## In child run script
child_run = Run.get_context()
child_run.parent.id
```

### <a name="query-child-runs"></a>Дочерние выполнения запросов

Чтобы запросить дочерние выполнения определенного родителя, используйте [`get_children()`](/python/api/azureml-core/azureml.core.run%28class%29#get-children-recursive-false--tags-none--properties-none--type-none--status-none---rehydrate-runs-true-) метод. ``recursive = True``Аргумент позволяет запрашивать вложенное дерево дочерних элементов и внуками.

```python
print(parent_run.get_children())
```

### <a name="log-to-parent-or-root-run"></a>Выполнить вход в родительский или корневой запуск

С помощью поля можно `Run.parent` получить доступ к запуску, который запустил текущий дочерний запуск. Распространенным вариантом использования для использования `Run.parent` является объединение результатов журнала в одном месте. Обратите внимание, что дочерние операции запускаются асинхронно, и нет никакой гарантии, что порядок или синхронизация выходят за пределы способности родительского элемента ожидать завершения его дочерних запусков.

```python
# in child (or even grandchild) run

def root_run(self : Run) -> Run :
    if self.parent is None : 
        return self
    return root_run(self.parent)

current_child_run = Run.get_context()
root_run(current_child_run).log("MyMetric", f"Data from child run {current_child_run.id}")

```

## <a name="example-notebooks"></a>Примеры записных книжек

В следующих записных книжках демонстрируются концепции, описанные в этой статье.

* Дополнительные сведения об API ведения журнала см. в [записной книжке API для ведения журнала](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/logging-api/logging-api.ipynb).

* Дополнительные сведения об управлении запусками с помощью пакета SDK для Машинное обучение Azure см. в разделе [Управление запуском записной книжки](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/track-and-monitor-experiments/manage-runs/manage-runs.ipynb).

## <a name="next-steps"></a>Следующие шаги

* Сведения о том, как регистрировать метрики для экспериментов, см. в статье [метрики журнала во время учебных запусков](how-to-track-experiments.md).
* Сведения о мониторинге ресурсов и журналов с Машинное обучение Azure см. в разделе [мониторинг машинное обучение Azure](monitor-azure-machine-learning.md).

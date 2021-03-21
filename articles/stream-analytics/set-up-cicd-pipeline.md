---
title: Создание конвейера CI/CD для Stream Analytics задания с помощью Azure DevOps
description: В этой статье описывается, как настроить конвейер непрерывной интеграции и развертывания (CI/CD) для задания Azure Stream Analytics в Azure DevOps.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 09/10/2020
ms.openlocfilehash: f62b4c354ffa90bf1a03651fccf8780074344e46
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103466428"
---
# <a name="use-azure-devops-to-create-a-cicd-pipeline-for-a-stream-analytics-job"></a>Создание конвейера CI/CD для Stream Analytics задания с помощью Azure DevOps

Из этой статьи вы узнаете, как создавать конвейеры [сборки](/azure/devops/pipelines/get-started/pipelines-get-started) и [выпуска](/azure/devops/pipelines/release/define-multistage-release-process) Azure DevOps с помощью средств Azure Stream Analytics CI/CD.

## <a name="commit-your-stream-analytics-project"></a>Фиксация проекта Stream Analytics

Прежде чем начать, Зафиксируйте все Stream Analytics проекты как исходные файлы в репозитории [Azure DevOps](/azure/devops/user-guide/source-control) . Вы можете ссылаться на этот [Пример репозитория](https://dev.azure.com/wenyzou/azure-streamanalytics-cicd-demo) и [Stream Analytics исходный код проекта](https://dev.azure.com/wenyzou/_git/azure-streamanalytics-cicd-demo?path=%2FmyASAProject) в Azure pipelines.

В действиях, описанных в этой статье, используется проект Stream Analytics Visual Studio Code. Если вы используете проект Visual Studio, выполните действия, описанные в разделе [Автоматизация сборок, тестов и развертываний задания Azure Stream Analytics с помощью средств CI/CD](cicd-tools.md).

## <a name="create-a-build-pipeline"></a>Создание конвейера сборки

В этом разделе вы узнаете, как создать конвейер сборки. Вы можете ссылаться на этот пример [автоматического создания и тестирования конвейера](https://dev.azure.com/wenyzou/_git/azure-streamanalytics-cicd-demo?path=%2FmyASAProject) в Azure DevOps.

1. Откройте веб-браузер и перейдите к своему проекту в Azure DevOps.  

1. В разделе **конвейеры** в меню навигации слева выберите **сборки**. Затем выберите **создать конвейер**.

   :::image type="content" source="media/set-up-cicd-pipeline/new-pipeline.png" alt-text="Создание нового конвейера Azure":::

1. Выберите **использовать классический редактор** для создания конвейера без YAML.

1. Выберите тип источника, командный проект и репозиторий. Затем выберите **продолжить**.

   :::image type="content" source="media/set-up-cicd-pipeline/select-repo.png" alt-text="Выбор проекта Azure Stream Analytics":::

1. На странице **Выбор шаблона** выберите **пустое задание**.

## <a name="install-npm-package"></a>Установка пакета npm

1. На странице **задачи** щелкните знак «плюс» рядом с **заданием агента 1**. Введите *NPM* в поле поиска задач и выберите **NPM**.

   :::image type="content" source="media/set-up-cicd-pipeline/search-npm.png" alt-text="Выбор задачи NPM":::

2. Присвойте задаче **Отображаемое имя**. Измените параметр **команды** на *Custom* и введите следующую команду в **команде и аргументах**. Оставьте остальные параметры по умолчанию.

   ```bash
   install -g azure-streamanalytics-cicd
   ```

   :::image type="content" source="media/set-up-cicd-pipeline/npm-config.png" alt-text="Введите конфигурации для задачи NPM":::

Если необходимо использовать агент Hosted-Linux, выполните следующие действия.
1.  Выберите **спецификацию агента**
   
    :::image type="content" source="media/set-up-cicd-pipeline/select-linux-agent.png" alt-text="Снимок экрана: Выбор спецификации агента.":::

2.  На странице **задачи** щелкните знак «плюс» рядом с **заданием агента 1**. Введите *командную строку* в области поиска задач и выберите **Командная строка**.
   
    :::image type="content" source="media/set-up-cicd-pipeline/cmd-search.png" alt-text="Снимок экрана: задача поиска командной строки. ":::

3.  Присвойте задаче **Отображаемое имя**. Введите следующую команду в **Скрипт**. Оставьте остальные параметры по умолчанию.

      ```bash
      sudo npm install -g azure-streamanalytics-cicd --unsafe-perm=true --allow-root
      ```
      :::image type="content" source="media/set-up-cicd-pipeline/cmd-scripts.png" alt-text="Снимок экрана: ввод скрипта для задачи cmd.":::

## <a name="add-a-build-task"></a>Добавление задачи сборки

1. На странице **переменные** выберите **+ Добавить** в **переменных конвейера**. Добавьте следующие переменные. Задайте следующие значения в соответствии с вашими предпочтениями.

   |Имя переменной|Значение|
   |-|-|
   |прожектрутпас|Yourprojectname собственным значением|
   |outputPath|Выходные данные|
   |деплойпас|Развернуть|

2. На странице **задачи** щелкните знак «плюс» рядом с **заданием агента 1**. Найдите **командную строку**.

3. Присвойте задаче **Отображаемое имя** и введите следующий скрипт. Измените сценарий, указав имя репозитория и имя проекта.

   ```bash
   azure-streamanalytics-cicd build -project $(projectRootPath)/asaproj.json -outputpath $(projectRootPath)/$(outputPath)/$(deployPath)
   ```

   На приведенном ниже рисунке в качестве примера используется проект Stream Analytics Visual Studio Code.

   :::image type="content" source="media/set-up-cicd-pipeline/command-line-config-build.png" alt-text="Ввод конфигураций для задачи командной строки Visual Studio Code":::

## <a name="add-a-test-task"></a>Добавление тестовой задачи

1. На странице **переменные** выберите **+ Добавить** в **переменных конвейера**. Добавьте следующие переменные. Измените значения, указав путь к выходу и имя репозитория.

   |Имя переменной|Значение|
   |-|-|
   |тестпас|Проверить|

   :::image type="content" source="media/set-up-cicd-pipeline/pipeline-variables-test.png" alt-text="Добавить переменные конвейера":::

2. На странице **задачи** щелкните знак «плюс» рядом с **заданием агента 1**. Найдите **командную строку**.

3. Присвойте задаче **Отображаемое имя** и введите следующий скрипт. Измените скрипт, указав имя файла проекта и путь к файлу конфигурации теста. 

   Дополнительные сведения о добавлении и настройке тестовых случаев см. в разделе [инструкции по автоматическому тестированию](cicd-tools.md#automated-test) .

   ```bash
   azure-streamanalytics-cicd test -project $(projectRootPath)/asaproj.json -outputpath $(projectRootPath)/$(outputPath)/$(testPath) -testConfigPath $(projectRootPath)/test/testConfig.json 
   ```

   :::image type="content" source="media/set-up-cicd-pipeline/command-line-config-test.png" alt-text="Введите настройки для задачи командной строки":::

## <a name="add-a-copy-files-task"></a>Задача "Добавление файлов копирования"

Необходимо добавить задачу «копирование файла», чтобы скопировать файл сводки теста и Azure Resource Manager файлы шаблонов в папку артефактов. 

1. На странице **задачи** выберите **+** рядом с **заданием агента 1**. Поиск **копий файлов**. Затем введите следующие конфигурации. При назначении `**` **содержимого** все файлы результатов теста копируются.

   |Параметр|Входные данные|
   |-|-|
   |Отображаемое имя|Копировать файлы в: $ (Build. artifactstagingdirectory)|
   |Исходная папка|`$(system.defaultworkingdirectory)/$(outputPath)/`|
   |Содержимое| `**` |
   |Целевая папка| `$(build.artifactstagingdirectory)`|

2. Разверните узел **Параметры управления**. Выберите **, даже если предыдущая задача завершилась сбоем, если только сборка не была отменена** в ходе **выполнения этой задачи**.

   :::image type="content" source="media/set-up-cicd-pipeline/copy-config.png" alt-text="Введите конфигурации для задачи копирования":::

## <a name="add-a-publish-build-artifacts-task"></a>Добавление задачи «публикация артефактов сборки»

1. На странице **задачи** щелкните знак «плюс» рядом с **заданием агента 1**. Выполните поиск по запросу **Публикация артефактов сборки** и выберите параметр со значком с черной стрелкой.

2. Разверните узел **Параметры управления**. Выберите **, даже если предыдущая задача завершилась сбоем, если только сборка не была отменена** в ходе **выполнения этой задачи**.

   :::image type="content" source="media/set-up-cicd-pipeline/publish-config.png" alt-text="Введите конфигурации для задачи публикации":::

## <a name="save-and-run"></a>Сохранение и запуск

Завершив добавление задач пакет NPM, Командная строка, копирование файлов и публикация артефактов сборки, выберите **сохранить & очередь**. При появлении запроса введите комментарий и выберите **сохранить и выполнить**. Вы можете скачать результаты тестирования со страницы **сводки** в конвейере.

## <a name="check-the-build-and-test-results"></a>Проверка результатов сборки и теста

Файл сводки теста и файлы шаблонов Azure Resource Manager можно найти в **опубликованной** папке.

   :::image type="content" source="media/set-up-cicd-pipeline/check-build-test-result.png" alt-text="Проверка результатов сборки и теста":::

   :::image type="content" source="media/set-up-cicd-pipeline/check-drop-folder.png" alt-text="Проверить артефакты":::

## <a name="release-with-azure-pipelines"></a>Выпуск с Azure Pipelines

В этом разделе вы узнаете, как создать конвейер выпуска. Вы можете ссылаться на этот пример [конвейера выпуска](https://dev.azure.com/wenyzou/azure-streamanalytics-cicd-demo/_release?_a=releases&view=mine&definitionId=2&preserve-view=true) в Azure DevOps.

Откройте веб-браузер и перейдите к Azure Stream Analytics Visual Studio Code проекта.

1. В разделе **конвейеры** в меню навигации слева выберите **выпуски**. Затем выберите **создать конвейер**.

2. Выберите **начать с пустого задания**.

3. В поле **артефакты** выберите **+ Добавить артефакт**. В разделе **источник** выберите созданный конвейер сборки и нажмите кнопку **Добавить**.

   :::image type="content" source="media/set-up-cicd-pipeline/build-artifact.png" alt-text="Введите артефакт конвейера сборки":::

4. Измените имя **этапа 1** , чтобы **развернуть задание в тестовую среду**.

5. Добавьте новый этап и присвойте ему имя **Развертывание задания в рабочей среде**.

### <a name="add-deploy-tasks"></a>Добавление задач развертывания

1. В раскрывающемся списке задачи выберите **развернуть задание в тестовой среде**.

2. Выберите **+** рядом с **заданием агента** и выполните поиск по запросу **Развертывание шаблона ARM**. Задайте следующие параметры:

   |Параметр|Значение|
   |-|-|
   |Отображаемое имя| *Развертывание Мясапрожект*|
   |Подписка Azure.| Выберите свою подписку.|
   |Действие| *Создание или изменение группы ресурсов*|
   |Группа ресурсов| Выберите имя для тестовой группы ресурсов, которая будет содержать задание Stream Analytics.|
   |Расположение|Выберите расположение группы тестовых ресурсов.|
   |Расположение шаблона| Связанный артефакт|
   |Шаблон| $ (System. DefaultWorkingDirectory)/_azure-streamanalytics-cicd-Demo-CI-Deploy/Drop/myASAProject.JobTemplate.json |
   |Параметры шаблона|$ (System. DefaultWorkingDirectory)/_azure-streamanalytics-cicd-Demo-CI-Deploy/Drop/myASAProject.JobTemplate.parameters.json |
   |Переопределение параметров шаблона|-<arm_template_parameter> "ваше значение". Параметры можно определить с помощью **переменных**.|
   |Режим развертывания|Добавочный|

3. В раскрывающемся списке задачи выберите **развернуть задание в рабочей среде**.

4. Выберите **+** рядом с **заданием агента** и выполните поиск по запросу *Развертывание шаблона ARM*. Задайте следующие параметры:

   |Параметр|Значение|
   |-|-|
   |Отображаемое имя| *Развертывание Мясапрожект*|
   |Подписка Azure.| Выберите свою подписку.|
   |Действие| *Создание или изменение группы ресурсов*|
   |Группа ресурсов| Выберите имя рабочей группы ресурсов, которая будет содержать задание Stream Analytics.|
   |Расположение|Выберите расположение рабочей группы ресурсов.|
   |Расположение шаблона| *Связанный артефакт*|
   |Шаблон| $ (System. DefaultWorkingDirectory)/_azure-streamanalytics-cicd-Demo-CI-Deploy/Drop/myASAProject.JobTemplate.json |
   |Параметры шаблона|$ (System. DefaultWorkingDirectory)/_azure-streamanalytics-cicd-Demo-CI-Deploy/Drop/myASAProject.JobTemplate.parameters.json |
   |Переопределение параметров шаблона|-<arm_template_parameter> "ваше значение"|
   |Режим развертывания|Добавочный|

### <a name="create-a-release"></a>Создание выпуска

Чтобы создать выпуск, выберите **создать выпуск** в правом верхнем углу.

:::image type="content" source="media/set-up-cicd-pipeline/create-release.png" alt-text="Создание выпуска с помощью Azure Pipelines":::

## <a name="next-steps"></a>Следующие шаги

* [Непрерывная интеграция и непрерывное развертывание для Azure Stream Analytics](cicd-overview.md)
* [Автоматизация сборки, тестирования и развертывания задания Azure Stream Analytics с помощью средств CI/CD](cicd-tools.md)
---
title: Непрерывное развертывание с помощью Azure DevOps (Предварительная версия)
titleSuffix: Azure Cognitive Services
description: Из этой статьи вы узнаете, как настроить непрерывное развертывание для приложений с пользовательскими командами. Скрипты создаются для поддержки рабочих процессов непрерывного развертывания.
services: cognitive-services
author: xiaojul
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: xiaojul
ms.openlocfilehash: 686e342eb68e2e5c97bdbae9cd11bdff5819b5d6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98927600"
---
# <a name="continuous-deployment-with-azure-devops"></a>Непрерывное развертывание с помощью Azure DevOps

Из этой статьи вы узнаете, как настроить непрерывное развертывание для приложений с пользовательскими командами. Скрипты для поддержки рабочего процесса CI/CD предоставлены вам.

## <a name="prerequisite"></a>Предварительное требование
> [!div class = "checklist"]
> * Приложение настраиваемых команд для разработки (DEV)
> * Приложение пользовательских команд для рабочей среды (Production)
> * Регистрация в [Azure pipelines](/azure/devops/pipelines/get-started/pipelines-sign-up)

## <a name="exportimportpublish"></a>Экспорт, импорт и публикация

Скрипты размещаются в [Cognitive Services голосовым помощником — пользовательских командах](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/master/custom-commands). Клонировать скрипты из каталога Bash в репозиторий. Убедитесь, что вы сохраняете тот же путь.

### <a name="set-up-a-pipeline"></a>Настройка конвейера 

1. Перейдите в **Azure DevOps — конвейеры** и щелкните "создать конвейер".
1. В разделе **Подключение** выберите расположение репозитория, в котором находятся эти сценарии.
1. В разделе **Выбор** выберите свой репозиторий.
1. В разделе **Настройка** выберите "начальный конвейер".
1. Далее вы получите редактор с файлом YAML, замените раздел "шаги" этим сценарием.

    ```YAML
    steps:
    - task: Bash@3
      displayName: 'Export source app'
      inputs:
        targetType: filePath
        filePath: ./bash/export.sh
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(SourceAppId) -f ExportedDialogModel.json'
        workingDirectory: bash
        failOnStderr: true
    
    - task: Bash@3
      displayName: 'Import to target app'
      inputs:
        targetType: filePath
        filePath: ./bash/import.sh
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId) -f ExportedDialogModel.json'
        workingDirectory: bash
        failOnStderr: true
    
    - task: Bash@3
      displayName: 'Train and Publish target app'
      inputs:
        targetType: filePath
        filePath: './bash/train-and-publish.sh'
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId)'
        workingDirectory: bash
        failOnStderr: true
    ```
    
1. Обратите внимание, что в этих сценариях предполагается, что вы используете регион `westus2` , если это не так, обновите аргументы задач соответствующим образом.

    > [!div class="mx-imgBorder"]
    > ![Снимок экрана, который выделяет значение региона в аргументах.](media/custom-commands/cicd-new-pipeline-yaml.png)

1. В кнопке "сохранить и выполнить" откройте раскрывающийся список и нажмите кнопку "Сохранить".

### <a name="hook-up-the-pipeline-with-your-application"></a>Подключение конвейера к приложению

1. Перейдите на главную страницу конвейера.
1. В раскрывающемся списке в правом верхнем углу выберите **изменить конвейер**. Он получает редактор YAML. 
1. В правом верхнем углу рядом с кнопкой "выполнить" выберите " **переменные**". Нажмите кнопку **создать переменную**.
1. Добавьте следующие переменные:
    
    | Переменная | Описание |
    | ------- | --------------- | ----------- |
    | саурцеаппид | Идентификатор приложения для разработки |
    | таржетаппид | Идентификатор ПРОИЗВОДСТВЕННОго приложения |
    | субскриптионкэй | Ключ подписки, используемый для обоих приложений |
    | culture | Язык и региональные параметры приложений (например, EN-US) |

    > [!div class="mx-imgBorder"]
    > ![Отправка полезных данных действия](media/custom-commands/cicd-edit-pipeline-variables.png)

1. Щелкните "выполнить" и выберите "задание" выполняется. 

    Вы увидите список задач, которые выполняются с: "экспорт исходного приложения", "Импорт в целевое приложение" & "обучение и публикация целевого приложения".

## <a name="deploy-from-source-code"></a>Развертывание из исходного кода

Если вы хотите оставить определение приложения в репозитории, мы предоставляем скрипты для развертывания из исходного кода. Так как скрипты находятся в bash, при использовании Windows необходимо установить [подсистему Linux](/windows/wsl/install-win10).

Скрипты размещаются в [Cognitive Services голосовым помощником — пользовательских командах](https://github.com/Azure-Samples/Cognitive-Services-Voice-Assistant/tree/master/custom-commands). Клонировать скрипты из каталога Bash в репозиторий. Убедитесь, что вы сохраняете тот же путь.

### <a name="prepare-your-repository"></a>Подготовка репозитория

1. Создайте каталог для приложения, в нашем примере создайте его под названием "приложения".
1. Обновите аргументы скрипта Bash ниже и выполните команду. Будет импортирована модель диалога приложения в файл myapp.js
    ```BASH
    bash/export.sh -r <region> -s <subscriptionkey> -c en-us -a <appid> -f apps/myapp.json
    ```
    | Аргументы | Описание |
    | ------- | --------------- | ----------- |
    | region | регион приложения, например westus2. |
    | субскриптионкэй | ключ подписки для своего ресурса речи. |
    | appid | Идентификатор приложения настраиваемых команд, который необходимо экспортировать. |

1. Отправьте эти изменения в репозиторий.

### <a name="set-up-a-pipeline"></a>Настройка конвейера 

1. Перейдите в **Azure DevOps — конвейеры** и щелкните "создать конвейер".
1. В разделе **Подключение** выберите расположение репозитория, в котором находятся эти сценарии.
1. В разделе **Выбор** выберите свой репозиторий.
1. В разделе **Настройка** выберите "начальный конвейер".
1. Далее вы получите редактор с файлом YAML, замените раздел "шаги" этим сценарием.

    ```YAML
    steps:
    - task: Bash@3
      displayName: 'Import app'
      inputs:
        targetType: filePath
        filePath: ./bash/import.sh
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId) -f ../apps/myapp.json'
        workingDirectory: bash
        failOnStderr: true
    
    - task: Bash@3
      displayName: 'Train and Publish app'
      inputs:
        targetType: filePath
        filePath: './bash/train-and-publish.sh'
        arguments: '-r westus2 -s $(SubscriptionKey) -c $(Culture) -a $(TargetAppId)'
        workingDirectory: bash
        failOnStderr: true
    ```

    > [!NOTE]
    > в этих сценариях предполагается, что вы используете регион westus2, если это не так, обновите аргументы задач соответствующим образом.

1. В кнопке "сохранить и выполнить" откройте раскрывающийся список и нажмите кнопку "Сохранить".

### <a name="hook-up-the-pipeline-with-your-target-applications"></a>Подключение конвейера к целевым приложениям

1. Перейдите на главную страницу конвейера.
1. В раскрывающемся списке в правом верхнем углу выберите **изменить конвейер**. Он получает редактор YAML. 
1. В правом верхнем углу рядом с кнопкой "выполнить" выберите " **переменные**". Нажмите кнопку **создать переменную**.
1. Добавьте следующие переменные:

    | Переменная | Описание |
    | ------- | --------------- | ----------- |
    | таржетаппид | Идентификатор ПРОИЗВОДСТВЕННОго приложения |
    | субскриптионкэй | Ключ подписки, используемый для обоих приложений |
    | culture | Язык и региональные параметры приложений (например, EN-US) |

1. Щелкните "выполнить" и выберите "задание" выполняется.
    Вы увидите список задач, которые выполняются с: "Импорт приложения" & "обучение и публикация приложения".

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [См. примеры на GitHub](https://aka.ms/speech/cc-samples)

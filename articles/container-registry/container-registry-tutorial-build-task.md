---
title: Учебник. Сборка образа на основе фиксации кода
description: Из этого руководства вы узнаете, как настроить задачу Реестра контейнеров Azure, чтобы автоматически активировать сборки образов контейнера в облаке при фиксации исходного кода в репозитории Git.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: b4806ed30319ff058df6dfae0340a73ad4cb6132
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780791"
---
# <a name="tutorial-automate-container-image-builds-in-the-cloud-when-you-commit-source-code"></a>Руководство по Автоматизация создания образов контейнеров в облаке при фиксации исходного кода

Помимо [быстрых задач](container-registry-tutorial-quick-task.md), пакет "Задачи Реестра контейнеров Azure" поддерживает автоматическую сборку образов контейнеров Docker при фиксации исходного кода в репозитории Git. Поддерживаемые контексты Git для Задач ACR включают общедоступные или частные репозитории GitHub или Azure.

> [!NOTE]
> Сейчас Задачи ACR не поддерживают триггеры фиксации или запроса на вытягивание в репозиториях GitHub Enterprise.

В этом руководстве решение "Задачи Реестра контейнеров Azure" компилирует и отправляет один образ контейнера, указанный в Dockerfile, при фиксации исходного кода в репозитории Git. Чтобы создать [многошаговую задачу](container-registry-tasks-multi-step.md), в которой используется YAML-файл для определения действий по сборке, отправке и тестированию (при необходимости) нескольких контейнеров при фиксации кода, см. статью [Tutorial: Run a multi-step container workflow in the cloud when you commit source code](container-registry-tutorial-multistep-task.md) (Руководство. Запуск многошагового рабочего процесса в облаке при фиксации исходного кода). Обзор решения "Задачи ACR" см. в статье [Automate OS and framework patching with ACR Tasks](container-registry-tasks-overview.md) (Автоматизация установки исправлений для ОС и платформы с помощью службы "Задачи ACR").

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * Создание задачи
> * Тестирование задачи
> * Представление состояния задачи
> * Активация задачи с помощью фиксации кода

В этом руководстве предполагается, что вы уже выполнили шаги в [предыдущем руководстве](container-registry-tutorial-quick-task.md). Если вы еще этого не сделали, выполните действия, описанные в разделе [Предварительные требования](container-registry-tutorial-quick-task.md#prerequisites) в предыдущем руководстве, прежде чем перед продолжить.

[!INCLUDE [container-registry-task-tutorial-prereq.md](../../includes/container-registry-task-tutorial-prereq.md)]
[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="create-the-build-task"></a>Создание задачи сборки

Теперь, когда вы выполнили шаги, необходимые для включения в решении "Задачи ACR" чтения состояния фиксации и создания веб-перехватчиков в репозитории, вы можете создать задачу, которая запускает сборку образа контейнера при фиксации кода в репозитории.

Сначала заполните эти переменные среды оболочки значениями, подходящими для вашей среды. Этот шаг не является обязательным, но он упрощает выполнение многолинейных команд Azure CLI в этом руководстве. Если вы не заполните эти переменные среды, вам придется вручную заменять каждое значение в примерах команд.

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the previous section
```

Теперь создайте задачу, выполнив следующую команду [az acr task create][az-acr-task-create]:

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name taskhelloworld \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file Dockerfile \
    --git-access-token $GIT_PAT
```


Эта задача указывает, что каждый раз при фиксации кода в *главной* ветви репозитория, указанной в `--context`, решение "Задачи ACR" создает образ контейнера из кода в этой ветви. Для сборки образа используется Dockerfile, определяемый с помощью `--file` из корня репозитория. Аргумент `--image` указывает параметризованное значение `{{.Run.ID}}` для части с версией в теге образа, гарантируя, что образ коррелирует с конкретной сборкой и имеет уникальный тег.

Выходные данные успешно выполненной команды [az acr task create][az-acr-task-create] имеют следующий вид:

```output
{
  "agentConfiguration": {
    "cpu": 2
  },
  "creationDate": "2010-11-19T22:42:32.972298+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myregistry/providers/Microsoft.ContainerRegistry/registries/myregistry/tasks/taskhelloworld",
  "location": "westcentralus",
  "name": "taskhelloworld",
  "platform": {
    "architecture": "amd64",
    "os": "Linux",
    "variant": null
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "myregistry",
  "status": "Enabled",
  "step": {
    "arguments": [],
    "baseImageDependencies": null,
    "contextPath": "https://github.com/gituser/acr-build-helloworld-node#main",
    "dockerFilePath": "Dockerfile",
    "imageNames": [
      "helloworld:{{.Run.ID}}"
    ],
    "isPushEnabled": true,
    "noCache": false,
    "type&quot;: &quot;Docker"
  },
  "tags": null,
  "timeout": 3600,
  "trigger": {
    "baseImageTrigger": {
      "baseImageTriggerType": "Runtime",
      "name": "defaultBaseimageTriggerName",
      "status&quot;: &quot;Enabled"
    },
    "sourceTriggers": [
      {
        "name": "defaultSourceTriggerName",
        "sourceRepository": {
          "branch": "main",
          "repositoryUrl": "https://github.com/gituser/acr-build-helloworld-node#main",
          "sourceControlAuthProperties": null,
          "sourceControlType&quot;: &quot;GitHub"
        },
        "sourceTriggerEvents": [
          "commit"
        ],
        "status&quot;: &quot;Enabled"
      }
    ]
  },
  "type&quot;: &quot;Microsoft.ContainerRegistry/registries/tasks"
}
```

## <a name="test-the-build-task"></a>Тестирование задачи сборки

Теперь у вас есть задача, которая определяет сборку. Чтобы протестировать конвейер сборки, запустите сборку вручную, выполнив команду [​​az acr task run][az-acr-task-run]:

```azurecli
az acr task run --registry $ACR_NAME --name taskhelloworld
```

По умолчанию при выполнении команда `az acr task run` передает выходные данные журнала в консоль. В сокращенном примере выходных данных показаны только основные шаги.

```output
2020/11/19 22:51:00 Using acb_vol_9ee1f28c-4fd4-43c8-a651-f0ed027bbf0e as the home volume
2020/11/19 22:51:00 Setting up Docker configuration...
2020/11/19 22:51:02 Successfully set up Docker configuration
2020/11/19 22:51:02 Logging in to registry: myregistry.azurecr.io
2020/11/19 22:51:03 Successfully logged in
2020/11/19 22:51:03 Executing step: build
2020/11/19 22:51:03 Obtaining source code and scanning for dependencies...
2020/11/19 22:51:05 Successfully obtained source code and scanned for dependencies
Sending build context to Docker daemon  23.04kB
Step 1/5 : FROM node:15-alpine
[...]
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in 7382eea2a56a
Removing intermediate container 7382eea2a56a
 ---> e33cd684027b
Successfully built e33cd684027b
Successfully tagged myregistry.azurecr.io/helloworld:da2
2020/11/19 22:51:11 Executing step: push
2020/11/19 22:51:11 Pushing image: myregistry.azurecr.io/helloworld:da2, attempt 1
The push refers to repository [myregistry.azurecr.io/helloworld]
4a853682c993: Preparing
[...]
4a853682c993: Pushed
[...]
da2: digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419 size: 1366
2020/11/19 22:51:21 Successfully pushed image: myregistry.azurecr.io/helloworld:da2
2020/11/19 22:51:21 Step id: build marked as successful (elapsed time in seconds: 7.198937)
2020/11/19 22:51:21 Populating digests for step id: build...
2020/11/19 22:51:22 Successfully populated digests for step id: build
2020/11/19 22:51:22 Step id: push marked as successful (elapsed time in seconds: 10.180456)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloworld
    tag: da2
    digest: sha256:c24e62fd848544a5a87f06ea60109dbef9624d03b1124bfe03e1d2c11fd62419
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 9-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 68cdf2a37cdae0873b8e2f1c4d80ca60541029bf


Run ID: ca6 was successful after 27s
```

## <a name="trigger-a-build-with-a-commit"></a>Активация сборки с помощью фиксации

Теперь, когда вы протестировали задачу, выполнив ее вручную, запустите ее автоматически, изменив исходный код.

Сначала перейдите в каталог, содержащий локальный клон вашего [репозитория][sample-repo]:

```console
cd acr-build-helloworld-node
```

Затем выполните следующие команды для создания, фиксации и передачи нового файла в вилку вашего репозитория GitHub:

```console
echo "Hello World!" > hello.txt
git add hello.txt
git commit -m "Testing ACR Tasks"
git push origin main
```

Может потребоваться предоставить ваши учетные данные GitHub при выполнении команды `git push`. Введите свое имя пользователя GitHub и укажите созданный ранее личный маркер доступа в качестве пароля.

```azurecli
Username for 'https://github.com': <github-username>
Password for 'https://githubuser@github.com': <personal-access-token>
```

После того как вы передали фиксацию в ваш репозиторий, веб-перехватчик, созданный решением "Задачи ACR", запускается и выполняет сборку в службе "Реестр контейнеров Azure". Отобразите журналы текущей задачи, чтобы проверить ее и отслеживать ход ее выполнения:

```azurecli
az acr task logs --registry $ACR_NAME
```

Выходные данные выглядят следующим образом. Они показывают выполняемую (или последнюю выполненную) задачу:

```output
Showing logs of the last created run.
Run ID: ca7

[...]

Run ID: ca7 was successful after 38s
```

## <a name="list-builds"></a>Вывод списка сборок

Чтобы просмотреть список выполняемых задач, которые решение "Задачи ACR" выполнило в реестре, запустите команду [az acr task list-runs][az-acr-task-list-runs]:

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Результат этой команды должен выглядеть примерно следующим образом: Отобразятся сборки, созданные решением "Задачи ACR", а также команда "Git Commit" в столбце TRIGGER для самой последней задачи:

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------------  ----------  ---------  ---------  --------------------  ----------
ca7       taskhelloworld  linux       Succeeded  Commit     2020-11-19T22:54:34Z  00:00:29
ca6       taskhelloworld  linux       Succeeded  Manual     2020-11-19T22:51:47Z  00:00:24
ca5                       linux       Succeeded  Manual     2020-11-19T22:23:42Z  00:00:23
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать задачу, чтобы автоматически активировать сборки образов контейнера в Azure при фиксации исходного кода в репозитории Git. Перейдите к следующему руководству, чтобы узнать, как создавать задачи, которые активируют сборку при обновлении базового образа контейнера.

> [!div class="nextstepaction"]
> [Автоматизация сборки образа при обновлении базового образа](container-registry-tutorial-base-image-update.md)

<!-- LINKS - External -->
[sample-repo]: https://github.com/Azure-Samples/acr-build-helloworld-node

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-login]: /cli/azure/reference-index#az_login

---
title: Учебник. Многоэтапная задача ACR
description: Из этого руководства вы узнаете, как настроить задачу Реестра контейнеров Azure, чтобы автоматически активировать многошаговый рабочий процесс для сборки, запуска и отправки в облако образов контейнера при фиксации исходного кода в репозитории Git.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 6d6ca8c41ce808df4b32bc10a7b537ffae809b3a
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780683"
---
# <a name="tutorial-run-a-multi-step-container-workflow-in-the-cloud-when-you-commit-source-code"></a>Руководство по Запуск многошагового рабочего процесса в облаке при фиксации исходного кода

Помимо [быстрых задач](container-registry-tutorial-quick-task.md), решение "Задачи ACR" поддерживает многошаговые рабочие процессы для нескольких контейнеров, которые могут выполняться автоматически при фиксации исходного кода в репозитории Git. 

Из этого руководства вы узнаете, как на основе примеров файлов YAML определить многошаговые задачи, которые компилируют, запускают и отправляют в реестр один или несколько образов контейнеров при фиксации исходного кода. Чтобы создать задачу, которая автоматизирует только сборку одного образа при фиксации кода, воспользуйтесь руководством [ Автоматизация создания образов контейнеров в облаке при фиксации исходного кода](container-registry-tutorial-build-task.md). Обзор решения "Задачи ACR" см. в статье [Automate OS and framework patching with ACR Tasks](container-registry-tasks-overview.md) (Автоматизация установки исправлений для ОС и платформы с помощью решения "Задачи ACR").

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * определение многошаговой задачи с помощью файла YAML;
> * Создание задачи
> * (необязательно) добавление к задаче учетных данных для доступа к другому реестру;
> * Тестирование задачи
> * Представление состояния задачи
> * Активация задачи с помощью фиксации кода

В этом руководстве предполагается, что вы уже выполнили шаги в [предыдущем руководстве](container-registry-tutorial-quick-task.md). Если вы еще этого не сделали, выполните действия, описанные в разделе [Предварительные требования](container-registry-tutorial-quick-task.md#prerequisites) в предыдущем руководстве, прежде чем перед продолжить.

[!INCLUDE [container-registry-task-tutorial-prereq.md](../../includes/container-registry-task-tutorial-prereq.md)]
[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="create-a-multi-step-task"></a>Создание многошаговой задачи

Теперь, когда вы выполнили шаги, которые позволяют решению "Задачи ACR" получать данные о состоянии фиксации и создавать веб-перехватчики в репозитории, переходите к созданию многошаговой задачи, которая запускает сборку, выполнение и отправку образа контейнера.

### <a name="yaml-file"></a>Файл YAML

Шаги для многошаговой задачи определяются в [YAML-файле](container-registry-tasks-reference-yaml.md). Первый пример многошаговой задачи для этого руководства определен в файле `taskmulti.yaml`, который размещен в корне клонированного ранее репозитория GitHub:

```yml
version: v1.1.0
steps:
# Build target image
- build: -t {{.Run.Registry}}/hello-world:{{.Run.ID}} -f Dockerfile .
# Run image 
- cmd: -t {{.Run.Registry}}/hello-world:{{.Run.ID}}
  id: test
  detach: true
  ports: ["8080:80"]
- cmd: docker stop test
# Push image
- push:
  - {{.Run.Registry}}/hello-world:{{.Run.ID}}
```

Эта многошаговая задача выполняет следующие действия:

1. Выполняет шаг `build` для сборки образа из файла Dockerfile в рабочем каталоге. Этот образ предназначен для реестра `Run.Registry`, в котором запускается задача, и ему присваивается тег с уникальным идентификатором выполнения, созданным решением "Задачи ACR". 
1. Выполняет шаг `cmd` для запуска образа во временном контейнере. Этот пример запускает длительно выполняющийся контейнер в фоновом режиме и возвращает идентификатор этого контейнера, а затем останавливает контейнер. Реальный сценарий может включать шаги для тестирования запущенного контейнера, которые позволяют проверить правильность его работы.
1. На шаге `push` отправляет скомпилированный образ в реестр выполнения.

### <a name="task-command"></a>Командная строка задачи

Сначала заполните эти переменные среды оболочки значениями, подходящими для вашей среды. Этот шаг не является обязательным, но он упрощает выполнение многолинейных команд Azure CLI в этом руководстве. Если вы не заполните эти переменные среды, вам придется вручную заменять каждое значение в примерах команд.

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the previous section
```

Теперь создайте задачу, выполнив следующую команду [az acr task create][az-acr-task-create]:

```azurecli-interactive
az acr task create \
    --registry $ACR_NAME \
    --name example1 \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file taskmulti.yaml \
    --git-access-token $GIT_PAT
```

Эта задача указывает, что каждый раз при фиксации кода в *главной* ветви репозитория, указанной в `--context`, решение "Задачи ACR" выполняет многошаговую задачу из кода в этой ветви. Шаги этой задачи определяются в YAML-файле `--file`, расположенном в корне репозитория. 

Выходные данные успешно выполненной команды [az acr task create][az-acr-task-create] имеют следующий вид:

```output
{
  "agentConfiguration": {
    "cpu": 2
  },
  "creationDate": "2020-11-20T03:14:31.763887+00:00",
  "credentials": null,
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myregistry/providers/Microsoft.ContainerRegistry/registries/myregistry/tasks/taskmulti",
  "location": "westus",
  "name": "example1",
  "platform": {
    "architecture": "amd64",
    "os": "linux",
    "variant": null
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "status": "Enabled",
  "step": {
    "baseImageDependencies": null,
    "contextAccessToken": null,
    "contextPath": "https://github.com/gituser/acr-build-helloworld-node.git#main",
    "taskFilePath": "taskmulti.yaml",
    "type": "FileTask",
    "values": [],
    "valuesFilePath": null
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
          "repositoryUrl": "https://github.com/gituser/acr-build-helloworld-node.git#main",
          "sourceControlAuthProperties": null,
          "sourceControlType&quot;: &quot;Github"
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

## <a name="test-the-multi-step-workflow"></a>Тестирование многошагового рабочего процесса

Чтобы протестировать многошаговую задачу, запустите ее вручную, выполнив команду [​​az acr task run][az-acr-task-run]:

```azurecli-interactive
az acr task run --registry $ACR_NAME --name example1
```

По умолчанию при выполнении команда `az acr task run` передает выходные данные журнала в консоль. В выходных данных отображается ход выполнения каждого шага задачи. Ниже представлен сокращенный пример выходных данных, только с основными шагами.

```output
Queued a run with ID: cab
Waiting for an agent...
2020/11/20 00:03:31 Downloading source code...
2020/11/20 00:03:33 Finished downloading source code
2020/11/20 00:03:33 Using acb_vol_cfe6bd55-3076-4215-8091-6a81aec3d1b1 as the home volume
2020/11/20 00:03:33 Creating Docker network: acb_default_network, driver: 'bridge'
2020/11/20 00:03:34 Successfully set up Docker network: acb_default_network
2020/11/20 00:03:34 Setting up Docker configuration...
2020/11/20 00:03:34 Successfully set up Docker configuration
2020/11/20 00:03:34 Logging in to registry: myregistry.azurecr.io
2020/11/20 00:03:35 Successfully logged into myregistry.azurecr.io
2020/11/20 00:03:35 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
2020/11/20 00:03:35 Scanning for dependencies...
2020/11/20 00:03:36 Successfully scanned dependencies
2020/11/20 00:03:36 Launching container with name: acb_step_0
Sending build context to Docker daemon  24.06kB
[...]
Successfully built f669bfd170af
Successfully tagged myregistry.azurecr.io/hello-world:cf19
2020/11/20 00:03:43 Successfully executed container: acb_step_0
2020/11/20 00:03:43 Executing step ID: acb_step_1. Working directory: '', Network: 'acb_default_network'
2020/11/20 00:03:43 Launching container with name: acb_step_1
279b1cb6e092b64c8517c5506fcb45494cd5a0bd10a6beca3ba97f25c5d940cd
2020/11/20 00:03:44 Successfully executed container: acb_step_1
2020/11/20 00:03:44 Executing step ID: acb_step_2. Working directory: '', Network: 'acb_default_network'
2020/11/20 00:03:44 Pushing image: myregistry.azurecr.io/hello-world:cf19, attempt 1
[...]
2020/11/20 00:03:46 Successfully pushed image: myregistry.azurecr.io/hello-world:cf19
2020/11/20 00:03:46 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 7.425169)
2020/11/20 00:03:46 Populating digests for step ID: acb_step_0...
2020/11/20 00:03:47 Successfully populated digests for step ID: acb_step_0
2020/11/20 00:03:47 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 0.827129)
2020/11/20 00:03:47 Step ID: acb_step_2 marked as successful (elapsed time in seconds: 2.112113)
2020/11/20 00:03:47 The following dependencies were found:
2020/11/20 00:03:47
- image:
    registry: myregistry.azurecr.io
    repository: hello-world
    tag: cf19
    digest: sha256:6b981a8ca8596e840228c974c929db05c0727d8630465de536be74104693467a
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 15-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 1a3065388a0238e52865db1c8f3e97492a43444c

Run ID: cab was successful after 18s
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

```azurecli-interactive
Username for 'https://github.com': <github-username>
Password for 'https://githubuser@github.com': <personal-access-token>
```

После отправки фиксации в репозиторий запускается веб-перехватчик, созданный решением "Задачи ACR", который выполняет задачу в Реестре контейнеров Azure. Отобразите журналы текущей задачи, чтобы проверить ее и отслеживать ход ее выполнения:

```azurecli-interactive
az acr task logs --registry $ACR_NAME
```

Выходные данные выглядят следующим образом. Они показывают выполняемую (или последнюю выполненную) задачу:

```output
Showing logs of the last created run.
Run ID: cad

[...]

Run ID: cad was successful after 37s
```

## <a name="list-builds"></a>Вывод списка сборок

Чтобы просмотреть список выполняемых задач, которые решение "Задачи ACR" выполнило в реестре, запустите команду [az acr task list-runs][az-acr-task-list-runs]:

```azurecli-interactive
az acr task list-runs --registry $ACR_NAME --output table
```

Результат этой команды должен выглядеть примерно следующим образом: Отобразятся сборки, созданные решением "Задачи ACR", а также команда "Git Commit" в столбце TRIGGER для самой последней задачи:

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------------  ----------  ---------  ---------  --------------------  ----------
cad       example1        linux       Succeeded  Commit     2020-11-20T00:22:15Z  00:00:35
cac       taskhelloworld  linux       Succeeded  Commit     2020-11-20T00:22:15Z  00:00:22
cab       example1        linux       Succeeded  Manual     2020-11-20T00:18:36Z  00:00:47
```

## <a name="create-a-multi-registry-multi-step-task"></a>Создание многошаговой задачи для нескольких реестров

Решение "Задачи ACR" по умолчанию имеет разрешения на отправку образов в реестр, в котором выполняется задача, и извлечение образов из него. Может потребоваться, чтобы многошаговая задача работала с одним или несколькими дополнительными реестрами, помимо реестра выполнения. Например, вы хотите компилировать образы в одном реестре и сохранять их с новыми тегами в другом реестре, который используется для рабочей системы. В этом примере показано, как создать такую задачу и указать учетные данные для другого реестра.

Если у вас еще нет второго реестра, создайте его для этого примера. Если вам нужен реестр, см. сведения в [предыдущем руководстве](container-registry-tutorial-quick-task.md) или в статье [Краткое руководство. Создание реестра контейнеров с использованием Azure CLI](container-registry-get-started-azure-cli.md).

Чтобы создать задачу, нужно получить для реестра имя сервера входа в формате *mycontainerregistrydate.azurecr.io* (все символы в нижнем регистре). В нашем примере второй реестр используется для хранения образов с тегами по дате сборки.

### <a name="yaml-file"></a>Файл YAML

Второй пример многошаговой задачи для этого руководства определен в файле `taskmulti-multiregistry.yaml`, который размещен в корне клонированного ранее репозитория GitHub:

```yml
version: v1.1.0
steps:
# Build target images
- build: -t {{.Run.Registry}}/hello-world:{{.Run.ID}} -f Dockerfile .
- build: -t {{.Values.regDate}}/hello-world:{{.Run.Date}} -f Dockerfile .
# Run image 
- cmd: -t {{.Run.Registry}}/hello-world:{{.Run.ID}}
  id: test
  detach: true
  ports: ["8080:80"]
- cmd: docker stop test
# Push images
- push:
  - {{.Run.Registry}}/hello-world:{{.Run.ID}}
  - {{.Values.regDate}}/hello-world:{{.Run.Date}}
```

Эта многошаговая задача выполняет следующие действия:

1. Выполняет два шага `build` для сборки образов из файла Dockerfile в рабочем каталоге:
    * Первый шаг предназначен для реестра `Run.Registry`, в котором запускается задача, и ему присваивается тег с уникальным идентификатором выполнения, созданным решением "Задачи ACR". 
    * Второй предназначен для реестра, определяемого значением `regDate`, которое вы задаете при создании задачи (или указываете через внешний файл `values.yaml`, переданный в `az acr task create`). Этот образ получает тег с датой выполнения.
1. Выполняет шаг `cmd`, чтобы запустить один из скомпилированных контейнеров. Этот пример запускает длительно выполняющийся контейнер в фоновом режиме и возвращает идентификатор этого контейнера, а затем останавливает контейнер. В реальной ситуации может выполняться тестирование запущенного контейнера, которое позволяет проверить правильность его работы.
1. На шаге `push` отправляет скомпилированные образы: первый — в реестр выполнения, второй — в реестр, указанный в значении `regDate`.

### <a name="task-command"></a>Командная строка задачи

Используя определенные ранее переменные среды, создайте задачу, выполнив следующую команду [az acr task create][az-acr-task-create]. Замените значение *mycontainerregistrydate* именем реального реестра.

```azurecli-interactive
az acr task create \
    --registry $ACR_NAME \
    --name example2 \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file taskmulti-multiregistry.yaml \
    --git-access-token $GIT_PAT \
    --set regDate=mycontainerregistrydate.azurecr.io
```

### <a name="add-task-credential"></a>Добавление учетных данных задачи

Чтобы передать образы в реестр, указанный в значении `regDate`, выполните команду [az acr task credential add][az-acr-task-credential-add], которая добавляет в задачу учетные данные для входа в этот реестр.

В этом примере мы рекомендуем создать [субъект-службу](container-registry-auth-service-principal.md) с доступом к реестру, ограниченным ролью *AcrPush*, и наличием разрешений для отправки образов. Чтобы создать субъект-службу, воспользуйтесь [этим скриптом Azure CLI](https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/service-principal-create/service-principal-create.sh).

Передайте идентификатор приложения и пароль для субъекта-службы с помощью команды `az acr task credential add`. Обязательно замените имя сервера для входа *mycontainerregistrydate* именем второго реестра:

```azurecli-interactive
az acr task credential add --name example2 \
    --registry $ACR_NAME \
    --login-server mycontainerregistrydate.azurecr.io \
    --username <service-principal-application-id> \
    --password <service-principal-password>
```

Интерфейс командной строки вернет имя добавленного сервера входа для реестра.

### <a name="test-the-multi-step-workflow"></a>Тестирование многошагового рабочего процесса

Как и в предыдущем примере, чтобы протестировать многошаговую задачу, запустите ее вручную, выполнив команду [​​az acr task run][az-acr-task-run]. Чтобы задача запускалась при фиксации в репозитории Git, выполните инструкции из раздела [Trigger a build with a commit](#trigger-a-build-with-a-commit) (Активация сборки с помощью фиксации).

```azurecli-interactive
az acr task run --registry $ACR_NAME --name example2
```

По умолчанию при выполнении команда `az acr task run` передает выходные данные журнала в консоль. Как и в предыдущем примере, в выходных данных отображается ход выполнения каждого шага задачи. В сокращенном примере выходных данных показаны только основные шаги.

Выходные данные:

```output
Queued a run with ID: cf1g
Waiting for an agent...
2020/11/20 04:33:39 Downloading source code...
2020/11/20 04:33:41 Finished downloading source code
2020/11/20 04:33:42 Using acb_vol_4569b017-29fe-42bd-83b2-25c45a8ac807 as the home volume
2020/11/20 04:33:42 Creating Docker network: acb_default_network, driver: 'bridge'
2020/11/20 04:33:43 Successfully set up Docker network: acb_default_network
2020/11/20 04:33:43 Setting up Docker configuration...
2020/11/20 04:33:44 Successfully set up Docker configuration
2020/11/20 04:33:44 Logging in to registry: mycontainerregistry.azurecr.io
2020/11/20 04:33:45 Successfully logged into mycontainerregistry.azurecr.io
2020/11/20 04:33:45 Logging in to registry: mycontainerregistrydate.azurecr.io
2020/11/20 04:33:47 Successfully logged into mycontainerregistrydate.azurecr.io
2020/11/20 04:33:47 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
2020/11/20 04:33:47 Scanning for dependencies...
2020/11/20 04:33:47 Successfully scanned dependencies
2020/11/20 04:33:47 Launching container with name: acb_step_0
Sending build context to Docker daemon  25.09kB
[...]
Successfully tagged mycontainerregistry.azurecr.io/hello-world:cf1g
2020/11/20 04:33:55 Successfully executed container: acb_step_0
2020/11/20 04:33:55 Executing step ID: acb_step_1. Working directory: '', Network: 'acb_default_network'
2020/11/20 04:33:55 Scanning for dependencies...
2020/11/20 04:33:56 Successfully scanned dependencies
2020/11/20 04:33:56 Launching container with name: acb_step_1
Sending build context to Docker daemon  25.09kB
[...]
Successfully tagged mycontainerregistrydate.azurecr.io/hello-world:20190503-043342z
2020/11/20 04:33:57 Successfully executed container: acb_step_1
2020/11/20 04:33:57 Executing step ID: acb_step_2. Working directory: '', Network: 'acb_default_network'
2020/11/20 04:33:57 Launching container with name: acb_step_2
721437ff674051b6be63cbcd2fa8eb085eacbf38d7d632f1a079320133182101
2020/11/20 04:33:58 Successfully executed container: acb_step_2
2020/11/20 04:33:58 Executing step ID: acb_step_3. Working directory: '', Network: 'acb_default_network'
2020/11/20 04:33:58 Launching container with name: acb_step_3
test
2020/11/20 04:34:09 Successfully executed container: acb_step_3
2020/11/20 04:34:09 Executing step ID: acb_step_4. Working directory: '', Network: 'acb_default_network'
2020/11/20 04:34:09 Pushing image: mycontainerregistry.azurecr.io/hello-world:cf1g, attempt 1
The push refers to repository [mycontainerregistry.azurecr.io/hello-world]
[...]
2020/11/20 04:34:12 Successfully pushed image: mycontainerregistry.azurecr.io/hello-world:cf1g
2020/11/20 04:34:12 Pushing image: mycontainerregistrydate.azurecr.io/hello-world:20190503-043342z, attempt 1
The push refers to repository [mycontainerregistrydate.azurecr.io/hello-world]
[...]
2020/11/20 04:34:19 Successfully pushed image: mycontainerregistrydate.azurecr.io/hello-world:20190503-043342z
2020/11/20 04:34:19 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 8.125744)
2020/11/20 04:34:19 Populating digests for step ID: acb_step_0...
2020/11/20 04:34:21 Successfully populated digests for step ID: acb_step_0
2020/11/20 04:34:21 Step ID: acb_step_1 marked as successful (elapsed time in seconds: 2.009281)
2020/11/20 04:34:21 Populating digests for step ID: acb_step_1...
2020/11/20 04:34:23 Successfully populated digests for step ID: acb_step_1
2020/11/20 04:34:23 Step ID: acb_step_2 marked as successful (elapsed time in seconds: 0.795440)
2020/11/20 04:34:23 Step ID: acb_step_3 marked as successful (elapsed time in seconds: 11.446775)
2020/11/20 04:34:23 Step ID: acb_step_4 marked as successful (elapsed time in seconds: 9.734973)
2020/11/20 04:34:23 The following dependencies were found:
2020/11/20 04:34:23
- image:
    registry: mycontainerregistry.azurecr.io
    repository: hello-world
    tag: cf1g
    digest: sha256:75354e9edb995e8661438bad9913deed87a185fddd0193811f916d684b71a5d2
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 15-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 9d9023473c46a5e2c315681b11eb4552ef0faccc
- image:
    registry: mycontainerregistrydate.azurecr.io
    repository: hello-world
    tag: 20190503-043342z
    digest: sha256:75354e9edb995e8661438bad9913deed87a185fddd0193811f916d684b71a5d2
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 15-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git:
    git-head-revision: 9d9023473c46a5e2c315681b11eb4552ef0faccc

Run ID: cf1g was successful after 46s
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создавать многошаговые задачи для нескольких контейнеров, которые автоматически запускаются при фиксации исходного кода в репозитории Git. Расширенные возможности многошаговых задач, включая параллельное и зависимое выполнение шагов, описаны в [справочнике по YAML для решения "Задачи ACR"](container-registry-tasks-reference-yaml.md). Перейдите к следующему руководству, чтобы узнать, как создавать задачи, которые активируют сборку при обновлении базового образа контейнера.

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
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az_acr_task_credential_add    
[az-login]: /cli/azure/reference-index#az_login

<!-- IMAGES -->
[build-task-01-new-token]: ./media/container-registry-tutorial-build-tasks/build-task-01-new-token.png
[build-task-02-generated-token]: ./media/container-registry-tutorial-build-tasks/build-task-02-generated-token.png

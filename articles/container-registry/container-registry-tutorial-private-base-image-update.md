---
title: Руководство. Активация сборки образов при обновлении частного базового образа
description: С помощью этого руководства вы настроите в Реестре контейнеров Azure автоматическую активацию сборки образа контейнера в облаке при обновлении базового образа в другом реестре контейнеров Azure.
ms.topic: tutorial
ms.date: 11/20/2020
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 27ab7c3fc0f04023c32cfac181d8f8650de23560
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107772367"
---
# <a name="tutorial-automate-container-image-builds-when-a-base-image-is-updated-in-another-private-container-registry"></a>Руководство по Автоматизация сборки образов контейнера при обновлении базового образа в другом частном реестре контейнеров 

Решение [Задачи ACR](container-registry-tasks-overview.md) поддерживает автоматическую сборку образа при [обновлении базового образа](container-registry-tasks-base-images.md) контейнера, например, когда вы исправляете ОС или исполняющую среду в одном из базовых образов. 

Из этого руководства вы узнаете, как создать задачу ACR, которая запускает сборку в облаке при отправке базового образа контейнера в другой реестр контейнеров Azure. В другом руководстве описано, как создать задачу ACR, которая запускает сборку образа при отправке базового образа контейнера в [тот же реестр контейнеров Azure](container-registry-tutorial-base-image-update.md).

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание базового образа в базовом реестре;
> * создание задачи сборки приложения в другом реестре, которая отслеживает базовый образ; 
> * обновление базового образа для запуска задачи образа приложения;
> * отображение активированной задачи;
> * проверка обновленного образа приложения.

## <a name="prerequisites"></a>Предварительные требования

### <a name="complete-the-previous-tutorials"></a>Завершение работы с предыдущими руководствами

В этом руководстве предполагается, что вы уже настроили среду и выполнили следующие шаги, описанные в первых двух руководствах в серии:

* Создание реестра контейнеров Azure
* Создание вилки примера репозитория
* Клонировали пример репозитория.
* Создали личный маркер доступа GitHub.

Если вы еще этого не сделали, выполните инструкции из следующих учебников, прежде чем продолжить:

[Руководство. Создание образов контейнера в облаке с помощью службы "Задачи Реестра контейнеров Azure"](container-registry-tutorial-quick-task.md)

[Руководство. Автоматизация сборок образов контейнера с помощью задач службы "Реестр контейнеров Azure"](container-registry-tutorial-build-task.md)

Помимо реестра контейнеров, который вы создали для работы с предыдущими руководствами этой серии, вам потребуется еще один реестр для хранения базовых образов. При желании второй реестр можно создать в расположении, отличающемся от расположения исходного реестра.

### <a name="configure-the-environment"></a>Настройка среды

Заполните эти переменные среды оболочки значениями, подходящими для вашей среды. Этот шаг не является обязательным, но он упрощает выполнение многолинейных команд Azure CLI в этом руководстве. Если вы не заполните эти переменные среды, вам придется вручную заменять каждое значение в примерах команд.

```azurecli
BASE_ACR=<base-registry-name>   # The name of your Azure container registry for base images
ACR_NAME=<registry-name>        # The name of your Azure container registry for application images
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```

### <a name="base-image-update-scenario"></a>Сценарий обновления базового образа

В этом руководстве рассматривается сценарий обновления базового образа. Этот сценарий соответствует рабочему процессу разработки, в котором базовые образы совместно размещаются в частном реестре контейнеров, а образы приложения создаются в других реестрах. Базовые образы могут ссылаться на операционные системы и платформы, совместно используемые командой разработчиков, или даже на общие компоненты-службы.

Например, разработчики, которые создают образы приложения в собственных реестрах, могут обращаться к набору базовых образов, размещенных в общем базовом реестре. Базовый реестр может располагаться в другом регионе и к нему можно применить георепликацию.

[Пример кода][code-sample] включает два файла Docker: образ приложения и образ, который определяется как базовый. В следующих разделах вы создадите задачу ACR, которая автоматически запустит сборку образа приложения, когда новая версия базового образа будет отправлена в другой реестр контейнеров Azure.

* [Dockerfile-app][dockerfile-app]: небольшое веб-приложение Node.js, которое преображает для просмотра статическую веб-страницу с версией Node.js, на которой она основана. Строка версии моделируется: она отображает содержимое переменной среды (`NODE_VERSION`), которая определена в базовом образе.

* [Dockerfile-base][dockerfile-base]: образ, который приложение `Dockerfile-app` определяет как основной. Сам он основан на образе [узла][base-node] и включает переменную среды `NODE_VERSION`.

В следующих разделах вы создадите задачу, обновите значение `NODE_VERSION` в файле Docker базового образа, а затем используете решение "Задачи ACR" для создания базового образа. Когда задача ACR принудительно отправит новый базовый образ в реестр, он автоматически запустит сборку образа приложения. При желании можно запустить образ контейнера приложения локально, чтобы увидеть различные строки версии в созданных образах.

В этом руководстве создается задача ACR и отправляется образ приложения-контейнера, указанного в Dockerfile. С помощью решения "Задачи ACR" можно также запускать [многошаговые задачи](container-registry-tasks-multi-step.md), используя файл YAML для определения действий по созданию, отправке и тестированию (при необходимости) нескольких контейнеров.

## <a name="build-the-base-image"></a>создание базового образа;

Для начала создайте базовый образ с помощью функции *быстрой задачи* в решении "Задачи ACR", выполнив команду [az acr build][az-acr-build]. Как обсуждалось в [первом руководстве](container-registry-tutorial-quick-task.md) этой серии, этот процесс позволит не только создать образ, но и отправить его в реестр контейнеров (при успешном выполнении сборки). В нашем примере образ передается в реестр базовых образов.

```azurecli
az acr build --registry $BASE_ACR --image baseimages/node:15-alpine --file Dockerfile-base .
```

## <a name="create-a-task-to-track-the-private-base-image"></a>Создание задачи для отслеживания частного базового образа

Теперь создайте задание в реестре образов приложений с помощью команды [az acr task create][az-acr-task-create], чтобы подключить [управляемое удостоверение](container-registry-tasks-authentication-managed-identity.md). Это управляемое удостоверение пригодится вам на последующих шагах для проверки подлинности задания в реестре базовых образов. 

В нашем примере используется удостоверение, назначаемое системой, но для некоторых сценариев вам будет лучше создать и подключить управляемое удостоверение, назначаемое пользователем. Дополнительные сведения см. в статье о [проверке подлинности в разных реестрах в рамках задачи контроля доступа с помощью удостоверения, управляемого Azure](container-registry-tasks-cross-registry-authentication.md).

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name baseexample2 \
    --image helloworld:{{.Run.ID}} \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file Dockerfile-app \
    --git-access-token $GIT_PAT \
    --arg REGISTRY_NAME=$BASE_ACR.azurecr.io \
    --assign-identity
```

Эта задача аналогична той, которую вы создали при работе с [предыдущим учебником](container-registry-tutorial-build-task.md). Она инструктирует решение "Задачи ACR" активировать сборку образа, когда фиксации отправляются в репозиторий, указанный с помощью `--context`. Они отличаются тем, что Dockerfile для создания образа в предыдущем руководстве определяет общедоступный базовый образ (`FROM node:15-alpine`), а в этой задаче [Dockerfile-app][dockerfile-app] задает базовый образ в реестре базовых образов.

```Dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:15-alpine
```

Позже в этом руководстве эта конфигурация позволяет моделировать исправление платформы в базовом образе.

## <a name="give-identity-pull-permissions-to-base-registry"></a>Предоставление удостоверению разрешений на извлечение данных из базового реестра

Чтобы подключенное к задаче управляемое удостоверение могло извлекать образы из реестра базовых образов, ему нужно предоставить разрешения. Для начала получите идентификатор субъекта-службы для этого удостоверения, выполнив команду [az acr task show][az-acr-task-show]. Затем получите идентификатор ресурса для базового реестра, выполнив команду [az acr show][az-acr-show].

```azurecli
# Get service principal ID of the task
principalID=$(az acr task show --name baseexample2 --registry $ACR_NAME --query identity.principalId --output tsv) 

# Get resource ID of the base registry
baseregID=$(az acr show --name $BASE_ACR --query id --output tsv) 
```
 
Присвойте управляемому удостоверению разрешения на извлечение данных из этого реестра, выполнив команду [az role assignment create][az-role-assignment-create].

```azurecli
az role assignment create \
  --assignee $principalID \
  --scope $baseregID --role acrpull 
```

## <a name="add-target-registry-credentials-to-the-task"></a>Добавление учетных данных целевого реестра в задачу

Запустите [az acr task credential add][az-acr-task-credential-add], чтобы добавить в задачу учетные данные. Передайте параметр `--use-identity [system]`, чтобы назначаемое системой управляемое удостоверение задачи получило доступ к этим учетным данным.

```azurecli
az acr task credential add \
  --name baseexample2 \
  --registry $ACR_NAME \
  --login-server $BASE_ACR.azurecr.io \
  --use-identity [system] 
```

## <a name="manually-run-the-task"></a>Выполнение задачи вручную

Используйте команду [az acr task run][az-acr-task-run], чтобы вручную запустить задачу и создать образ приложения. Этот шаг нужно выполнить для того, чтобы задача отслеживала зависимость образа приложения от базового образа.

```azurecli
az acr task run --registry $ACR_NAME --name baseexample2
```

По завершении сборки запишите **ИД запуска** (например, "da6"), если нужно выполнить следующий необязательный шаг.

### <a name="optional-run-application-container-locally"></a>Необязательное действие: Локальный запуск контейнера приложения

Если вы работаете локально (а не в облачной среде Cloud Shell) и у вас установлен Docker, запустите контейнер, чтобы увидеть приложение, отображаемое в веб-браузере, а затем восстановите его базовый образ. Если вы используете Cloud Shell, пропустите этот раздел (Cloud Shell не поддерживает `az acr login` или `docker run`).

Сначала выполните проверку подлинности реестра контейнеров с помощью команды [az acr login][az-acr-login]:

```azurecli
az acr login --name $ACR_NAME
```

Затем локально запустите контейнер с помощью `docker run`. Замените **\<run-id\>** идентификатором запуска, полученным в выходных данных на предыдущем шаге (например, "da6"). В этом примере задается имя контейнера `myapp` и включен параметр `--rm` для удаления контейнера при его остановке.

```bash
docker run -d -p 8080:80 --name myapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Перейдите по адресу `http://localhost:8080` в браузере, и вы увидите номер версии Node.js, преобразованный для просмотра на веб-странице, аналогичный приведенному ниже. На более позднем этапе вы активируете версию, добавив "a" в строку версии.

:::image type="content" source="media/container-registry-tutorial-base-image-update/base-update-01.png" alt-text="Снимок экрана: пример приложения в браузере":::

Чтобы остановить и удалить контейнер, выполните следующую команду:

```bash
docker stop myapp
```

## <a name="list-the-builds"></a>Список сборок

Далее перечислите запуски задачи, которые Задачи ACR выполнили для реестра, с помощью команды [az acr task list-run][az-acr-task-list-runs]:

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Если вы закончили работу с предыдущим руководством (и не удалили реестр), вы должны увидеть выходные данные, похожие на приведенные ниже. Обратите внимание на количество запусков задач и последний идентификатор выполнения, чтобы сравнить результаты после обновления базового образа в следующем разделе.

```console
$ az acr task list-runs --registry $ACR_NAME --output table

UN ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
ca12      baseexample2    linux       Succeeded  Manual        2020-11-21T00:00:56Z  00:00:36
ca11      baseexample1    linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:34
ca10      taskhelloworld  linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:24
cay                       linux       Succeeded  Manual        2020-11-20T23:38:08Z  00:00:22
cax       baseexample1    linux       Succeeded  Manual        2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:29
```

## <a name="update-the-base-image"></a>Обновление базового образа

Здесь вы моделируете исправление платформы в базовом образе. Измените файл **Dockerfile-base**, добавив "a" после номера версии, определенного в `NODE_VERSION`:

```Dockerfile
ENV NODE_VERSION 15.2.1a
```

Запустите функцию быстрой задачи, чтобы создать измененный базовый образ. Запишите **ИД запуска** в выходных данных.

```azurecli
az acr build --registry $BASE_ACR --image baseimages/node:15-alpine --file Dockerfile-base .
```

После завершения сборки и отправки решением "Задача ACR" нового базового образа в реестр оно запустит сборку образа приложения. Задаче, созданной ранее, может потребоваться несколько минут, чтобы запустить сборку образа приложения, так как она должна обнаружить недавно созданный и переданный базовый образ.

## <a name="list-updated-build"></a>Список обновленной сборки

Теперь, когда вы обновили базовый образ, получите список выполнения задач снова, чтобы сравнить их со списком, полученным ранее. Если сначала выходные данные не отличаются, периодически запускайте команду, пока новое выполнение задачи не появится в списке.

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Результат аналогичен приведенному ниже. Триггером для последней выполненной сборки должно быть "обновление образа", указывающее, что задача была запущена с помощью функции быстрой задачи базового образа.

```console
$ az acr task list-runs --registry $ACR_NAME --output table

         PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
ca13      baseexample2    linux       Succeeded  Image Update  2020-11-21T00:06:00Z  00:00:43
ca12      baseexample2    linux       Succeeded  Manual        2020-11-21T00:00:56Z  00:00:36
ca11      baseexample1    linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:34
ca10      taskhelloworld  linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:24
cay                       linux       Succeeded  Manual        2020-11-20T23:38:08Z  00:00:22
cax       baseexample1    linux       Succeeded  Manual        2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:29
```

Если вы хотите выполнить следующий необязательный шаг (запуск только что созданного контейнера), чтобы отобразился обновленный номер версии, запишите значение **ИД запуска** для сборки образа, активируемой при обновлении (в предыдущих выходных данных — ca13).

### <a name="optional-run-newly-built-image"></a>Необязательное действие: Запуск только что созданного образа

Если вы работаете локально (не в облачной среде Cloud Shell) и у вас установлен Docker, запустите новый образ приложения после завершения его сборки. Замените код `<run-id>` идентификатором запуска, полученным на предыдущем шаге. Если вы используете Cloud Shell, пропустите этот раздел (Cloud Shell не поддерживает `docker run`).

```bash
docker run -d -p 8081:80 --name updatedapp --rm $ACR_NAME.azurecr.io/helloworld:<run-id>
```

Перейдите по адресу http://localhost:8081 в браузере, и вы увидите обновленный номер версии Node.js (с добавленным значением "а") на веб-странице:

:::image type="content" source="media/container-registry-tutorial-base-image-update/base-update-02.png" alt-text="Снимок экрана: обновленный пример приложения в браузере":::

Важно отметить, что вы обновили **базовый** образ с помощью нового номера версии, но новая версия отображается в последнем созданном **образе приложения**. Решение "Задачи ACR" обнаружило изменение базового образа и автоматически перестроило образ приложения.

Чтобы остановить и удалить контейнер, выполните следующую команду:

```bash
docker stop updatedapp
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать задачу, чтобы автоматически активировать сборки образа контейнера при обновлении базового образа. Теперь перейдите к следующему учебнику, чтобы узнать, как активировать задачи по определенному расписанию.

> [!div class="nextstepaction"]
> [Run an ACR task on a defined schedule](container-registry-tasks-scheduled.md) (Выполнение задачи записи контроля доступа по определенному расписанию)

<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-update]: /cli/azure/acr/task#az_acr_task_update
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-show]: /cli/azure/acr/task#az_acr_task_show
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az_acr_task_credential_add
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-task-list-runs]: /cli/azure/acr/task#az_acr_task_list_runs
[az-acr-task]: /cli/azure/acr#az_acr_task
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create

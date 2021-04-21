---
title: Учебник. Активация сборки образов при обновлении базового образа
description: Из этого учебника вы узнаете, как в службе "Реестр контейнеров Azure" настроить задачу для автоматического активирования сборки образов контейнера в облаке при обновлении базового образа в том же реестре.
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 6d83fb7796cecab686d52d561266bd41daef9e01
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107780809"
---
# <a name="tutorial-automate-container-image-builds-when-a-base-image-is-updated-in-an-azure-container-registry"></a>Руководство по автоматизации сборок образов контейнера при обновлении базового образа в службе "Реестр контейнеров Azure" 

Решение [Задачи ACR](container-registry-tasks-overview.md) поддерживает автоматическую сборку образа контейнера при [обновлении базового образа](container-registry-tasks-base-images.md) контейнера, например, когда вы исправляете ОС или исполняющую среду в одном из базовых образов. 

Из этого учебника вы узнаете, как создать задачу ACR, которая запускает сборку в облаке при отправке базового образа контейнера в тот же реестр. В другом учебнике описано, как создать задачу ACR, которая запускает сборку образа при отправке базового образа контейнера в [другой реестр контейнеров Azure](container-registry-tutorial-private-base-image-update.md). 

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание базового образа;
> * Создание образа приложения в том же реестре, который отслеживает базовый образ 
> * обновление базового образа для запуска задачи образа приложения;
> * отображение активированной задачи;
> * проверка обновленного образа приложения.

## <a name="prerequisites"></a>Предварительные требования

### <a name="complete-the-previous-tutorials"></a>Завершение работы с предыдущими руководствами

В этом руководстве предполагается, что вы уже настроили среду и выполнили следующие шаги, описанные в первых двух руководствах в серии:

- Создание реестра контейнеров Azure
- Создание вилки примера репозитория
- Клонировали пример репозитория.
- Создали личный маркер доступа GitHub.

Если вы еще этого не сделали, выполните инструкции из следующих учебников, прежде чем продолжить:

[Руководство. Создание образов контейнера в облаке с помощью службы "Задачи Реестра контейнеров Azure"](container-registry-tutorial-quick-task.md)

[Руководство. Автоматизация сборок образов контейнера с помощью задач службы "Реестр контейнеров Azure"](container-registry-tutorial-build-task.md)

### <a name="configure-the-environment"></a>Настройка среды

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]
- Для работы с этой статьей требуется Azure CLI версии 2.0.46 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

Заполните эти переменные среды оболочки значениями, подходящими для вашей среды. Этот шаг не является обязательным, но он упрощает выполнение многолинейных команд Azure CLI в этом руководстве. Если вы не заполните эти переменные среды, вам придется вручную заменять каждое значение в примерах команд.

```console
ACR_NAME=<registry-name>        # The name of your Azure container registry
GIT_USER=<github-username>      # Your GitHub user account name
GIT_PAT=<personal-access-token> # The PAT you generated in the second tutorial
```


### <a name="base-image-update-scenario"></a>Сценарий обновления базового образа

В этом учебнике описывается базовый сценарий обновления образа, в котором базовый образ и образ приложения хранятся в одном реестре. 

[Пример кода][code-sample] включает два файла Docker: образ приложения и образ, который определяется как базовый. В следующих разделах вы создадите задачу ACR, которая автоматически запустит сборку образа приложения, когда новая версия базового образа будет отправлена в тот же реестр контейнеров.

* [Dockerfile-app][dockerfile-app]: небольшое веб-приложение Node.js, которое преображает для просмотра статическую веб-страницу с версией Node.js, на которой она основана. Строка версии моделируется: она отображает содержимое переменной среды (`NODE_VERSION`), которая определена в базовом образе.

* [Dockerfile-base][dockerfile-base]: образ, который приложение `Dockerfile-app` определяет как основной. Сам он основан на образе [узла][base-node] и включает переменную среды `NODE_VERSION`.

В следующих разделах вы создадите задачу, обновите значение `NODE_VERSION` в файле Docker базового образа, а затем используете решение "Задачи ACR" для создания базового образа. Когда задача ACR принудительно отправит новый базовый образ в реестр, он автоматически запустит сборку образа приложения. При желании можно запустить образ контейнера приложения локально, чтобы увидеть различные строки версии в созданных образах.

В этом руководстве создается задача ACR и отправляется образ приложения-контейнера, указанного в Dockerfile. С помощью решения "Задачи ACR" можно также запускать [многошаговые задачи](container-registry-tasks-multi-step.md), используя файл YAML для определения действий по созданию, отправке и тестированию (при необходимости) нескольких контейнеров.

## <a name="build-the-base-image"></a>создание базового образа;

Для начала создайте базовый образ с помощью функции *быстрой задачи* в решении "Задачи ACR", выполнив команду [az acr build][az-acr-build]. Как обсуждалось в [первом руководстве](container-registry-tutorial-quick-task.md) этой серии, этот процесс позволит не только создать образ, но и отправить его в реестр контейнеров (при успешном выполнении сборки).

```azurecli
az acr build --registry $ACR_NAME --image baseimages/node:15-alpine --file Dockerfile-base .
```

## <a name="create-a-task"></a>Создание задачи

Далее с помощью команды [az acr task create][az-acr-task-create] создайте задачу:

```azurecli
az acr task create \
    --registry $ACR_NAME \
    --name baseexample1 \
    --image helloworld:{{.Run.ID}} \
    --arg REGISTRY_NAME=$ACR_NAME.azurecr.io \
    --context https://github.com/$GIT_USER/acr-build-helloworld-node.git#main \
    --file Dockerfile-app \
    --git-access-token $GIT_PAT
```

Эта задача аналогична той, которую вы создали при работе с [предыдущим учебником](container-registry-tutorial-build-task.md). Она инструктирует решение "Задачи ACR" активировать сборку образа, когда фиксации отправляются в репозиторий, указанный с помощью `--context`. В то время как Dockerfile, использованный для создания образа в предыдущем руководстве, определяет общедоступный базовый образ (`FROM node:15-alpine`), Dockerfile в этой задаче [Dockerfile-app][dockerfile-app] задает базовый образ в том же реестре:

```dockerfile
FROM ${REGISTRY_NAME}/baseimages/node:15-alpine
```

Позже в этом руководстве эта конфигурация позволяет моделировать исправление платформы в базовом образе.

## <a name="build-the-application-container"></a>Сборка контейнера приложения

Используйте команду [az acr task run][az-acr-task-run], чтобы вручную запустить задачу и создать образ приложения. Этот шаг нужно выполнить для того, чтобы задача отслеживала зависимость образа приложения от базового образа.

```azurecli
az acr task run --registry $ACR_NAME --name baseexample1
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

```output
RUN ID    TASK            PLATFORM    STATUS     TRIGGER    STARTED               DURATION
--------  --------------  ----------  ---------  ---------  --------------------  ----------
cax       baseexample1    linux       Succeeded  Manual     2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit     2020-11-20T23:16:07Z  00:00:29
cav       example2        linux       Succeeded  Commit     2020-11-20T23:16:07Z  00:00:55
cau       example1        linux       Succeeded  Commit     2020-11-20T23:16:07Z  00:00:40
cat       taskhelloworld  linux       Succeeded  Manual     2020-11-20T23:07:29Z  00:00:27
```

## <a name="update-the-base-image"></a>Обновление базового образа

Здесь вы моделируете исправление платформы в базовом образе. Измените файл **Dockerfile-base**, добавив "a" после номера версии, определенного в `NODE_VERSION`:

```dockerfile
ENV NODE_VERSION 15.2.1a
```

Запустите функцию быстрой задачи, чтобы создать измененный базовый образ. Запишите **ИД запуска** в выходных данных.

```azurecli
az acr build --registry $ACR_NAME --image baseimages/node:15-alpine --file Dockerfile-base .
```

После завершения сборки и отправки решением "Задача ACR" нового базового образа в реестр оно запустит сборку образа приложения. Задаче, созданной ранее, может потребоваться несколько минут, чтобы запустить сборку образа приложения, так как она должна обнаружить недавно созданный и переданный базовый образ.

## <a name="list-updated-build"></a>Список обновленной сборки

Теперь, когда вы обновили базовый образ, получите список выполнения задач снова, чтобы сравнить их со списком, полученным ранее. Если сначала выходные данные не отличаются, периодически запускайте команду, пока новое выполнение задачи не появится в списке.

```azurecli
az acr task list-runs --registry $ACR_NAME --output table
```

Результат аналогичен приведенному ниже. Триггером для последней выполненной сборки должно быть "обновление образа", указывающее, что задача была запущена с помощью функции быстрой задачи базового образа.

```output
Run ID    TASK            PLATFORM    STATUS     TRIGGER       STARTED               DURATION
--------  --------------  ----------  ---------  ------------  --------------------  ----------
ca11      baseexample1    linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:34
ca10      taskhelloworld  linux       Succeeded  Image Update  2020-11-20T23:38:24Z  00:00:24
cay                       linux       Succeeded  Manual        2020-11-20T23:38:08Z  00:00:22
cax       baseexample1    linux       Succeeded  Manual        2020-11-20T23:33:12Z  00:00:30
caw       taskhelloworld  linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:29
cav       example2        linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:55
cau       example1        linux       Succeeded  Commit        2020-11-20T23:16:07Z  00:00:40
cat       taskhelloworld  linux       Succeeded  Manual        2020-11-20T23:07:29Z  00:00:27
```

Если вы хотите выполнить следующий необязательный шаг (запуск только что созданного контейнера), чтобы отобразился обновленный номер версии, запишите значение **ИД запуска** для сборки образа, активируемой при обновлении (в предыдущих выходных данных — ca11).

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
[base-node]: https://hub.docker.com/_/node/
[code-sample]: https://github.com/Azure-Samples/acr-build-helloworld-node
[dockerfile-app]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-app
[dockerfile-base]: https://github.com/Azure-Samples/acr-build-helloworld-node/blob/master/Dockerfile-base

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-update]: /cli/azure/acr/task#az_acr_task_update
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-task-list-runs]: /cli/azure/acr
[az-acr-task]: /cli/azure/acr

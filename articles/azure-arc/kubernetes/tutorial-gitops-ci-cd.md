---
title: Учебник по реализации CI/CD с помощью GitOps при использовании кластеров Kubernetes с поддержкой Azure Arc
description: В этом учебнике описывается настройка решения CI/CD при использовании GitOps с кластерами Kubernetes с поддержкой Azure Arc. Чтобы изучить концептуальные основы этого рабочего процесса, ознакомьтесь со статьей о рабочем процессе CI/CD при использовании GitOps с Kubernetes с поддержкой Azure Arc.
author: tcare
ms.author: tcare
ms.service: azure-arc
ms.topic: tutorial
ms.date: 03/03/2021
ms.custom: template-tutorial
ms.openlocfilehash: 72caca47cde960eb7298ec2cf0c6994755cb3159
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102121615"
---
# <a name="tutorial-implement-cicd-with-gitops-using-azure-arc-enabled-kubernetes-clusters"></a>Учебник по реализации CI/CD с помощью GitOps при использовании кластеров Kubernetes с поддержкой Azure Arc


В этом учебнике описывается настройка решения CI/CD при использовании GitOps с кластерами Kubernetes с поддержкой Azure Arc. Вы узнаете, как выполнить следующие задачи с помощью примера приложения Azure для голосования.

> [!div class="checklist"]
> * Создание кластера Kubernetes с поддержкой Azure Arc.
> * Подключение репозиториев приложений и GitOps к Azure Repos.
> * Импорт конвейеров CI/CD.
> * Подключение Реестра контейнеров Azure (ACR) к Azure DevOps и Kubernetes.
> * Создание группы переменных среды.
> * Развертывание сред `dev` и `stage`.
> * Тестирование сред приложений.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="before-you-begin"></a>Перед началом

В этом учебнике предполагается, что вы знакомы с Azure DevOps, Azure Repos, Azure Pipelines и Azure CLI.

* Войдите в [Azure DevOps Services](https://dev.azure.com/).
* Выполните инструкции из [предыдущего учебника](https://docs.microsoft.com/azure/azure-arc/kubernetes/tutorial-use-gitops-connected-cluster), чтобы узнать, как развернуть GitOps для среды CI/CD.
* Проанализируйте [преимущества и архитектуру](https://docs.microsoft.com/azure/azure-arc/kubernetes/conceptual-configurations) этого компонента.
* Убедитесь, что у вас есть следующее:
  * [Подключенный кластер Kubernetes с поддержкой Azure Arc](https://docs.microsoft.com/azure/azure-arc/kubernetes/quickstart-connect-cluster#connect-an-existing-kubernetes-cluster) с именем **arc-cicd-cluster**.
  * Подключенный Реестр контейнеров Azure (ACR) с [интеграцией AKS](https://docs.microsoft.com/azure/aks/cluster-container-registry-integration) или [проверкой подлинности кластера без использования AKS](https://docs.microsoft.com/azure/container-registry/container-registry-auth-kubernetes).
  * Разрешения "Администратор сборки" и "Администратор проекта" для [Azure Repos](https://docs.microsoft.com/azure/devops/repos/get-started/what-is-repos) и [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/get-started/pipelines-get-started).
* Установите следующие расширения CLI для Kubernetes с поддержкой Azure Arc версии не ниже 1.0.0:

  ```azurecli
  az extension add --name connectedk8s
  az extension add --name k8s-configuration
  ```
  * Чтобы обновить эти расширения до последних версий, выполните следующие команды:

    ```azurecli
    az extension update --name connectedk8s
    az extension update --name k8s-configuration
    ```

## <a name="import-application-and-gitops-repos-into-azure-repos"></a>Импорт репозиториев приложений и GitOps в Azure Repos

Импортируйте [репозиторий приложений](https://docs.microsoft.com/azure/azure-arc/kubernetes/conceptual-gitops-cicd#application-repo) и [репозиторий GitOps](https://docs.microsoft.com/azure/azure-arc/kubernetes/conceptual-gitops-cicd#gitops-repo) в Azure Repos. Для работы с этим учебником используйте следующие примеры репозиториев:

* Репозиторий приложений **arc-cicd-demo-src**
   * URL-адрес: https://github.com/Azure/arc-cicd-demo-src
   * Содержит пример приложения Azure для голосования, которое будет развернуто с помощью GitOps.
* Репозиторий GitOps **arc-cicd-demo-gitops**
   * URL-адрес: https://github.com/Azure/arc-cicd-demo-gitops
   * Используется в качестве основы для ресурсов кластера, в которых размещено приложение Azure для голосования.

См. дополнительные сведения об [импорте репозиториев Git](https://docs.microsoft.com/azure/devops/repos/git/import-git-repository).

>[!NOTE]
> Импорт и использование двух отдельных репозиториев для приложений и GitOps может повысить безопасность и упростить работу. Разрешения и видимость репозиториев приложений и GitOps можно настраивать по отдельности.
> Например, изменения в коде приложения могут показаться администратору кластера ненужными с точки зрения требуемого состояния кластера. И наоборот, разработчику приложения не нужно знать конкретные параметры для каждой среды. Ему достаточно набора тестовых значений, которые обеспечивают необходимый охват этих параметров.

## <a name="connect-the-gitops-repo"></a>Подключение репозитория GitOps

Для непрерывного развертывания приложения подключите репозиторий приложений к кластеру, в котором используется GitOps. Репозиторий GitOps под названием **arc-cicd-demo-gitops** содержит основные ресурсы, позволяющие запустить приложение в кластере **arc-cicd-cluster**.

Исходный репозиторий GitOps содержит только [манифест](https://github.com/Azure/arc-cicd-demo-gitops/blob/master/arc-cicd-cluster/manifests/namespaces.yml), с помощью которого создаются пространства имен для **разработки** и **подготовки**, соответствующие средам развертывания.

Созданное подключение GitOps будет автоматически выполнять следующие действия:
* синхронизация манифестов в каталоге манифестов;
* обновление состояния кластера.

Рабочий процесс CI/CD заполнит каталог манифеста дополнительными манифестами для развертывания приложения.


1. [Создайте новое подключение GitOps](https://docs.microsoft.com/azure/azure-arc/kubernetes/tutorial-use-gitops-connected-cluster) к только что импортированному репозиторию **arc-cicd-demo-gitops** в Azure Repos.

   ```azurecli
   az k8sconfiguration create \
      --name cluster-config \
      --cluster-name arc-cicd-cluster \
      --resource-group myResourceGroup \
      --operator-instance-name cluster-config \
      --operator-namespace cluster-config \
      --repository-url https://dev.azure.com/<Your organization>/arc-cicd-demo-gitops \
      --https-user <Azure Repos username> \
      --https-key <Azure Repos PAT token> \
      --scope cluster \
      --cluster-type connectedClusters \
      --operator-params='--git-readonly --git-path=arc-cicd-cluster/manifests'
   ```

1. Убедитесь, что Flux использует в качестве базового пути *только* каталог `arc-cicd-cluster/manifests`. Определите путь с помощью следующего параметра оператора:

   `--git-path=arc-cicd-cluster/manifests`

   > [!NOTE]
   > Если используется строка подключения HTTPS и возникают проблемы с подключением, убедитесь, что в URL-адресе не указан префикс имени пользователя. Например, из `https://alice@dev.azure.com/contoso/arc-cicd-demo-gitops` необходимо удалить `alice@`. Вместо этого укажите пользователя с помощью `--https-user`, например `--https-user alice`.

1. Проверить состояние развертывания можно на портале Azure.
   * В случае успеха в кластере будут созданы пространства имен `dev` и `stage`.

## <a name="import-the-cicd-pipelines"></a>Импорт конвейеров CI/CD

Теперь, после синхронизации подключения GitOps, необходимо импортировать конвейеры CI/CD, создающие манифесты.

Репозиторий приложений содержит папку `.pipeline` с конвейерами, которые будут использоваться для запросов PR, CI и CD. Импортируйте и переименуйте три конвейера, предоставленные в примере репозитория.

| Имя файла конвейера | Описание |
| ------------- | ------------- |
| [`.pipelines/az-vote-pr-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-pr-pipeline.yaml)  | Конвейер PR для приложения с именем **arc-cicd-demo-src PR** |
| [`.pipelines/az-vote-ci-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-ci-pipeline.yaml) | Конвейер CI для приложения с именем **arc-cicd-demo-src CI** |
| [`.pipelines/az-vote-cd-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-cd-pipeline.yaml) | Конвейер CD для приложения с именем **arc-cicd-demo-src CD** |



## <a name="connect-your-acr"></a>Подключение к ACR
Как конвейеры, так и кластер будут использовать ACR для хранения и извлечения образов Docker.

### <a name="connect-acr-to-azure-devops"></a>Подключение ACR к Azure DevOps
В процессе непрерывной интеграции вы развернете контейнеры приложений в реестре. Для начала создайте подключение к службе Azure.

1. В Azure DevOps перейдите на страницу настроек проекта и откройте страницу **Подключения службы**. На TFS откройте страницу **Службы**, воспользовавшись значком **Параметры** в верхней строке меню.
2. Выберите **+ Новое подключение службы** и выберите необходимый тип подключения службы.
3. Заполните поля параметров для подключения службы. В данном руководстве выберите следующие поля.
   * Присвойте подключению службы имя **arc-demo-acr**. 
   * Выберите **myResourceGroup** в качестве группы ресурсов.
4. Выберите **Предоставить разрешение на доступ для всех конвейеров**. 
   * Этот параметр позволяет авторизовать файлы конвейера YAML для подключений служб. 
5. Нажмите кнопку **ОК**, чтобы создать подключение.

### <a name="connect-acr-to-kubernetes"></a>Подключение ACR к Kubernetes
Разрешите на кластере Kubernetes извлечение образов из ACR. Если он частный, потребуется проверка подлинности.

#### <a name="connect-acr-to-existing-aks-clusters"></a>Подключение ACR к имеющимся кластерам AKS

Интегрируйте имеющуюся службу ACR с существующими кластерами AKS с помощью следующей команды:

```azurecli
az aks update -n arc-cicd-cluster -g myResourceGroup --attach-acr arc-demo-acr
```

#### <a name="create-an-image-pull-secret"></a>Создание секрета для извлечения образа

Чтобы подключить кластеры без AKS и локальные кластеры к AKS, создайте секрет извлечения образа. Kubernetes использует секреты извлечения образа для хранения сведений, необходимых для проверки подлинности реестра.

Создайте секрет извлечения образа с помощью следующей команды `kubectl`. Повторите эти действия для пространств имен `dev` и `stage`.
```console
kubectl create secret docker-registry <secret-name> \
    --namespace <namespace> \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```

> [!TIP]
> Чтобы не задавать imagePullSecret для каждого объекта Pod, рекомендуется добавить imagePullSecret в учетную запись службы в пространствах имен `dev` и `stage`. Дополнительные сведения см. в [учебнике по Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account).

## <a name="create-environment-variable-groups"></a>Создание групп переменных среды

### <a name="app-repo-variable-group"></a>Группа переменных репозитория приложений
[Создайте группу переменных](https://docs.microsoft.com/azure/devops/pipelines/library/variable-groups) с именем **az-vote-app-dev**. Задайте следующие значения:

| Переменная | Значение |
| -------- | ----- |
| AZ_ACR_NAME | (ваш экземпляр ACR, например azurearctest.azurecr.io) |
| AZURE_SUBSCRIPTION | (ваше подключение к службе Azure, у которого, согласно предыдущим инструкциям в учебнике, должно быть имя **arc-demo-acr**) |
| AZURE_VOTE_IMAGE_REPO | Полный путь к репозиторию приложения Azure для голосования, например azurearctest.azurecr.io/azvote |
| ENVIRONMENT_NAME | Разработка |
| MANIFESTS_BRANCH | `master` |
| MANIFESTS_REPO | Строка подключения Git для репозитория GitOps |
| PAT | [Созданный личный маркер доступа](https://docs.microsoft.com/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?#create-a-pat) с разрешениями на чтение и запись. Сохраните его для дальнейшего использования при создании группы переменных `stage`. |
| SRC_FOLDER | `azure-vote` | 
| TARGET_CLUSTER | `arc-cicd-cluster` |
| TARGET_NAMESPACE | `dev` |

> [!IMPORTANT]
> Пометьте личный маркер доступа как тип секрета. В приложениях рекомендуется связывать секреты из [Azure KeyVault](https://docs.microsoft.com/azure/devops/pipelines/library/variable-groups#link-secrets-from-an-azure-key-vault).
>
### <a name="stage-environment-variable-group"></a>Подготовка группы переменных среды

1. Клонируйте группу переменных **az-vote-app-dev**.
1. Измените имя на **az-vote-app-stage**.
1. Убедитесь, что для соответствующих переменных имеются следующие значения:

| Переменная | Значение |
| -------- | ----- |
| ENVIRONMENT_NAME | Этап |
| TARGET_NAMESPACE | `stage` |

Теперь все готово для развертывания в средах `dev` и `stage`.

## <a name="deploy-the-dev-environment-for-the-first-time"></a>Первое развертывание среды разработки
После создания конвейеров CI и CD запустите конвейер CI, чтобы развернуть приложение в первый раз.

### <a name="ci-pipeline"></a>Конвейер CI

Во время первоначального выполнения конвейера CI может произойти ошибка авторизации ресурсов при чтении имени подключения службы.
1. Убедитесь, что доступ осуществляется именно к переменной AZURE_SUBSCRIPTION.
1. Авторизуйте использование.
1. Перезапустите конвейер.

Конвейер CI выполняет следующие действия:
* Убеждается, что изменение в приложении проходит все автоматические проверки качества для развертывания.
* Выполняет все дополнительные проверки, которые невозможно выполнить в конвейере PR.
    * В GitOps конвейер публикует также артефакты для фиксации, которая будет развернута конвейером CD.
* Убеждается, что образ Docker изменен и отправлен новый образ.

### <a name="cd-pipeline"></a>Конвейер CD
При успешном запуске конвейера CI активирует конвейер CD для завершения процесса развертывания. Развертывание в каждой среде выполняется с приращением.

> [!TIP]
> Если конвейер CD не активируется автоматически, выполните следующие действия.
> 1. Убедитесь, что имя соответствует триггеру ветви в [`.pipelines/az-vote-cd-pipeline.yaml`](https://github.com/Azure/arc-cicd-demo-src/blob/master/.pipelines/az-vote-cd-pipeline.yaml).
>    * Вместо этого она должна иметь вид `arc-cicd-demo-src CI`.
> 1. Перезапустите конвейер CI.

После создания изменений шаблона и манифеста в репозитории GitOps конвейер CD создаст фиксацию, отправит ее и создаст PR для утверждения.
1. Откройте ссылку на PR, указанную в выходных данных задачи `Create PR`.
1. Проверьте изменения в репозитории GitOps. Вы должны увидеть следующее:
   * Изменение шаблона Helm верхнего уровня.
   * Низкоуровневые манифесты Kubernetes, отражающие базовые изменения в требуемом состоянии. Эти манифесты развертывает Flux.
1. Если все верно, утвердите и завершите PR.

1. Через несколько минут Flux подхватит изменения и начнет развертывание.
1. Перенаправьте порт локально с помощью `kubectl` и убедитесь, что приложение работает правильно, с помощью этой команды:

   `kubectl port-forward -n dev svc/azure-vote-front 8080:80`

1. Просмотрите приложение Azure для голосования в браузере по адресу `http://localhost:8080/`.

1. Проголосуйте за избранные варианты и приготовьтесь к внесению ряда изменений в приложение.

## <a name="set-up-environment-approvals"></a>Настройка утверждений среды
При развертывании приложения можно не только внести изменения в код или шаблоны, но и непреднамеренно привести кластер к неисправному состоянию.

Если работа среды разработки прерывается после развертывания, не следует переводить его в последующие среды с помощью утверждений среды.

1. В проекте Azure DevOps перейдите в среду, которую необходимо защитить.
1. Перейдите в раздел **утверждения и проверки** для ресурса.
1. Нажмите кнопку **создания**.
1. Укажите утверждающих и необязательное сообщение.
1. Щелкните **Создать** еще раз, чтобы завершить добавление проверки утверждения вручную.

Дополнительные сведения см. в учебнике по [определению утверждений и проверок](https://docs.microsoft.com/azure/devops/pipelines/process/approvals).

При следующем выполнении конвейера CD конвейер будет приостановлен после создания запроса на вытягивание GitOps. Убедитесь, что изменение синхронизировано правильно и передает базовые функциональные возможности. Утвердите проверку в конвейере, чтобы разрешить перенос изменений в следующую среду.

## <a name="make-an-application-change"></a>Внесение изменений в приложение

С помощью этого базового набора шаблонов и манифестов, представляющих состояние в кластере, вы внесете небольшое изменение в приложение.

1. В репозитории **arc-cicd-demo-src** измените файл [`azure-vote/src/azure-vote-front/config_file.cfg`](https://github.com/Azure/arc-cicd-demo-src/blob/master/azure-vote/src/azure-vote-front/config_file.cfg).

2. Так как вопрос "Кто вам больше нравится: кошки или собаки?" не получил достаточно голосов, чтобы увеличить их число, измените его на вопрос "Что лучше: вкладки или пространства".

3. Зафиксируйте изменения в новой ветви, отправьте ее и создайте запрос на вытягивание.
   * Это типичный процесс разработки, который запустит жизненный цикл CI/CD.

## <a name="pr-validation-pipeline"></a>Конвейер проверки PR

Конвейер PR — это первая линия защиты от неисправности при внесении изменений. Обычно проверки качества кода приложения включают анализ кода и статический анализ. С точки зрения GitOps необходимо также обеспечить такой же уровень качества для развертываемой итоговой инфраструктуры.

Файлы Dockerfile и Helm диаграммы Helm приложения могут использовать анализ кода так же, как и приложение.

Ошибки, обнаруженные при анализе кода, могут варьироваться:
* от неправильно отформатированных файлов YAML
* до несоблюдения рекомендаций, например по настройке ограничений на использование приложением ресурсов ЦП и памяти.

> [!NOTE]
> Чтобы извлечь максимальную пользу из анализа кода Helm в реальном приложении, необходимо подставить значения, похожие на те, которые используются в реальной среде.

Ошибки, обнаруженные во время выполнения конвейера, отображаются в разделе результатов теста среды выполнения. Здесь можно выполнять следующие действия.
* Отслеживание полезной статистики по типам ошибок.
* Поиск первой фиксации, для которой они были обнаружены.
* Выполнение трассировки стека для стилевых ссылок на разделы кода, которые вызывают ошибку.

После завершения выполнения конвейера вы будете уверены в качестве кода приложения и шаблона, который будет его развертывать. Теперь можно утвердить и завершить запрос на вытягивание. Перед активацией конвейера CD будет выполнен повторный запуск CI, при котором будут повторно созданы шаблоны и манифесты.

> [!TIP]
> В реальной среде не забудьте установить политики ветви, которые позволят убедиться, что в PR передаются проверки качества. Дополнительные сведения см. в статье [Установка политик ветви](https://docs.microsoft.com/azure/devops/repos/git/branch-policies).

## <a name="cd-process-approvals"></a>Утверждения процессов CD

При успешной активации выполнения конвейера CI запускается конвейер CD для завершения процесса развертывания. Так же как и при первом выполнении конвейера CD, развертывание в каждой среде выполняется с приращением. В этот раз вам потребуется утвердить конвейер в каждой среде развертывания.

1. Утвердите развертывание в среде `dev`.
1. После создания изменений шаблона и манифеста в репозитории GitOps конвейер CD создаст фиксацию, отправит ее и создаст PR для утверждения.
1. Откройте ссылку на PR, указанную в задаче.
1. Проверьте изменения в репозитории GitOps. Вы должны увидеть следующее:
   * Изменение шаблона Helm верхнего уровня.
   * Низкоуровневые манифесты Kubernetes, отражающие базовые изменения в требуемом состоянии.
1. Если все верно, утвердите и завершите PR.
1. Дождитесь завершения развертывания.
1. В рамках базового теста проверки сборки перейдите на страницу приложения и убедитесь, что в приложении для голосования теперь отображается вопрос о том, что лучше: вкладки или пространства.
   * Перенаправьте порт локально с помощью `kubectl` и убедитесь, что приложение работает правильно, с помощью этой команды: `kubectl port-forward -n dev svc/azure-vote-front 8080:80`.
   * Просмотрите приложение Azure для голосования в браузере по адресу http://localhost:8080/ и убедитесь, что варианты голосования изменились на вкладки и пространства. 
1. Повторите шаги 1–7 для среды `stage`.

Развертывание завершено. На этом рабочий процесс CI/CD заканчивается.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь использовать это приложение в дальнейшем, удалите все его ресурсы, выполнив следующие действия.

1. Удалите подключение в конфигурации GitOps для Azure Arc.
   ```azurecli
   az k8sconfiguration delete \
   --name cluster-config \
   --cluster-name arc-cicd-cluster \
   --resource-group myResourceGroup \
   --cluster-type connectedClusters
   ```

2. Удалите пространство имен `dev`.
   * `kubectl delete namespace dev`

3. Удалите пространство имен `stage`.
   * `kubectl delete namespace stage`

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как настроить полный рабочий процесс CI/CD, который реализует DevOps от разработки приложений до его развертывания. Изменения в приложении автоматически активируют проверку и развертывание, которые контролируются утверждениями вручную.

Перейдите к нашей концептуальной статье, чтобы узнать больше о GitOps и конфигурациях в Kubernetes с поддержкой Azure Arc.

> [!div class="nextstepaction"]
> [Рабочий процесс CI/CD при использовании GitOps — Kubernetes с поддержкой Azure Arc](https://docs.microsoft.com/azure/azure-arc/kubernetes/conceptual-gitops-cicd)

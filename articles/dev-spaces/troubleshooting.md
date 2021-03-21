---
title: Устранение неполадок
services: azure-dev-spaces
ms.date: 09/25/2019
ms.topic: troubleshooting
description: Узнайте, как устранять распространенные неполадки при включении и использовании Azure Dev Spaces
keywords: 'Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, containers, Helm, service mesh, service mesh routing, kubectl, k8s '
ms.openlocfilehash: 14ced0c66b42b6f18c946d0c75091be1af5598f5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102197798"
---
# <a name="azure-dev-spaces-troubleshooting"></a>Устранение неполадок с Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Это руководство содержит сведения о распространенных проблемах, которые могут возникнуть при использовании Azure Dev Spaces.

Если при использовании Azure Dev Spaces возникла проблема, [создайте соответствующее сообщение в репозитории Azure Dev Spaces на GitHub](https://github.com/Azure/dev-spaces/issues).

## <a name="before-you-begin"></a>Перед началом

Чтобы более эффективно устранять неполадки, можно создать более подробные журналы для проверки.

Для Visual Studio задайте для переменной среды `MS_VS_AZUREDEVSPACES_TOOLS_LOGGING_ENABLED` значение "1". Обязательно перезапустите Visual Studio, чтобы изменения переменной среды вступили в силу. После включения подробные журналы будут записаны в каталог `%TEMP%\Microsoft.VisualStudio.Azure.DevSpaces.Tools`.

В CLI вы можете выводить больше сведений во время выполнения команды, используя параметр `--verbose`. Вы также можете просмотреть более подробные журналы в `%TEMP%\Azure Dev Spaces`. На компьютере Mac каталог *TEMP* можно найти, выполнив команду `echo $TMPDIR` в окне терминала. На компьютере с ОС Linux каталог *TEMP* обычно размещается здесь: `/tmp`. Кроме того, убедитесь, что в [файле конфигурации Azure CLI](/cli/azure/azure-cli-configuration#cli-configuration-values-and-environment-variables) включено ведение журнала.

Azure Dev Spaces лучше всего использовать для отладки одного экземпляра или pod. Файл `azds.yaml` содержит параметр *replicaCount*, который определяет количество pod, которые будут выполняться в Kubernetes для службы. Если вы измените значение *replicaCount* для выполнения в приложении нескольких pod для указанной службы, отладчик присоединится к первой группе pod (в алфавитном порядке имен). Отладчик присоединяется к другому модулю pod при перезапуске оригинального модуля pod, что может привести к непредвиденному поведению.

## <a name="common-issues-when-enabling-azure-dev-spaces"></a>Распространенные проблемы при включении Azure Dev Spaces

### <a name="error-failed-to-create-azure-dev-spaces-controller"></a>Ошибка Failed to create Azure Dev Spaces controller (Ошибка создания контроллера Azure Dev Spaces)

Вы можете увидеть эту ошибку, если что-то пойдет не так с созданием контроллера. Если это временная ошибка, ее можно устранить, удалив контроллер, а затем повторно создав его.

Можно попытаться удалить контроллер.

```bash
azds remove -g <resource group name> -n <cluster name>
```

Чтобы удалить контроллер, используйте интерфейс командной строки Azure Dev Spaces. Контроллер невозможно удалить через Visual Studio. Кроме того, интерфейс командной строки Azure Dev Spaces невозможно установить в Azure Cloud Shell, как и удалить контроллер через Azure Cloud Shell.

Если у вас еще не установлен интерфейс командной строки Azure Dev Spaces, сначала установите его с помощью следующей команды, а затем удалите контроллер.

```azurecli
az aks use-dev-spaces -g <resource group name> -n <cluster name>
```

### <a name="controller-create-failing-because-of-controller-name-length"></a>Ошибка создания контроллера из-за длины имени контроллера

Длина имени контроллера в Azure Dev Spaces не может превышать 31 символ. Если имя контроллера превышает 31 символ, при включении Dev Spaces в кластере AKS или при создании контроллера появится сообщение об ошибке. Пример:

```console
Failed to create a Dev Spaces controller for cluster 'a-controller-name-that-is-way-too-long-aks-east-us': Azure Dev Spaces Controller name 'a-controller-name-that-is-way-too-long-aks-east-us' is invalid. Constraint(s) violated: Azure Dev Spaces Controller names can only be at most 31 characters long*
```

Чтобы устранить эту проблему, создайте контроллер с другим именем. Пример:

```cmd
azds controller create --name my-controller --target-name MyAKS --resource-group MyResourceGroup
```

### <a name="enabling-dev-spaces-failing-when-windows-node-pools-are-added-to-an-aks-cluster"></a>Сбой включения Dev Spaces, если пулы узлов Windows добавлены в кластер AKS

В настоящее время Azure Dev Spaces используется только для работы в pod и на узлах Linux. Если у вас есть кластер AKS с пулом узлов Windows, обеспечьте, чтобы все pod для Azure Dev Spaces назначались только на узлах Linux. Если группу pod для Azure Dev Spaces назначить для выполнения на узле Windows, она не сможет запуститься и включение Azure Dev Spaces завершится сбоем.

Чтобы устранить эту проблему, [примените отметку](../aks/operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations) к кластеру AKS, чтобы группы pod для Linux не назначались для выполнения на узле Windows.

### <a name="error-found-no-untainted-linux-nodes-in-ready-state-on-the-cluster-there-needs-to-be-at-least-one-untainted-linux-node-in-ready-state-to-deploy-pods-in-azds-namespace"></a>Ошибка Found no untainted Linux nodes in Ready state on the cluster. There needs to be at least one untainted Linux node in Ready state to deploy pods in 'azds' namespace (В кластере не найдены узлы Linux без отметки в состоянии готовности. Для развертывания pod в пространстве имен azds должен быть хотя бы один узел Linux без отметки в состоянии готовности)

Azure Dev Spaces не может создать контроллер в кластере AKS, так как не находит узел без отметки в состоянии *готовности*, чтобы назначить для него pod. Для Azure Dev Spaces требуется хотя бы один узел Linux в состоянии *готовности*, чтобы для него можно было назначить группы pod без настройки допусков.

Чтобы устранить эту проблему, [измените конфигурацию отметок](../aks/operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations) для кластера AKS, чтобы хотя бы один узел Linux был доступен для назначения групп pod без настройки допусков. Кроме того, должен присутствовать хотя бы один узел Linux в состоянии *готовности*, допускающий назначение групп pod без настройки допусков. Если узлу требуется много времени для перехода в состояние *готовности*, попробуйте перезапустить такой узел.

### <a name="error-azure-dev-spaces-cli-not-installed-properly-when-running-az-aks-use-dev-spaces"></a>Ошибка Azure Dev Spaces CLI not installed properly (Azure Dev Spaces CLI установлен неправильно) при выполнении команды az aks use-dev-spaces

При обновлении Azure Dev Spaces CLI изменился путь установки. Такая ошибка может появиться, если вы используете версию Azure CLI старше, чем 2.0.63. Чтобы отобразить текущую версию Azure CLI, выполните `az --version`.

```azurecli
az --version
```

```output
azure-cli                         2.0.60 *
...
```

При выполнении `az aks use-dev-spaces` с версией Azure CLI старше, чем 2.0.63, возникает сообщение об ошибке, но установка проходит успешно. Вы можете спокойно продолжать работу с `azds`.

Чтобы устранить эту проблему, обновите установку [Azure CLI](/cli/azure/install-azure-cli) до версии 2.0.63 или более поздней. Так сообщение об ошибке, возникающее при выполнении `az aks use-dev-spaces`, больше не будет появляться. Также можно просто продолжать использовать текущую версию Azure CLI и Azure Dev Spaces CLI.

### <a name="error-unable-to-reach-kube-apiserver"></a>Ошибка Unable to reach kube-apiserver (Не удается получить доступ к kube-apiserver)

Такая ошибка может возникать, когда Azure Dev Spaces не может подключиться к серверу API кластера AKS.

Если доступ к серверу API кластера AKS заблокирован или у вас есть [Разрешенные диапазоны IP-адресов сервера API](../aks/api-server-authorized-ip-ranges.md) для кластера AKS, необходимо также [создать](../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled) или [Обновить](../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges) кластер, чтобы [разрешить дополнительные диапазоны на основе вашего региона](configure-networking.md#aks-cluster-network-requirements) .

Убедитесь, что сервер API доступен, выполнив команды kubectl. Если сервер API недоступен, обратитесь в службу поддержки AKS и повторите попытку, когда сервер API будет работать.

## <a name="common-issues-when-preparing-your-project-for-azure-dev-spaces"></a>Распространенные проблемы при подготовке проекта для Azure Dev Spaces

### <a name="warning-dockerfile-could-not-be-generated-due-to-unsupported-language"></a>Предупреждение Dockerfile could not be generated due to unsupported language (Невозможно создать Dockerfile из-за неподдерживаемого языка)
Azure Dev Spaces предоставляет встроенную поддержку C# и Node.js. При запуске команды `azds prep` в каталоге, содержащем код на одном из этих языков, Azure Dev Spaces автоматически создает соответствующий файл Dockerfile.

Вы можете спокойно использовать Azure Dev Spaces с кодом на других языках, но Dockerfile в этом случае нужно будет создать вручную перед первым запуском `azds up`.

Если ваше приложение написано на языке без встроенной поддержки в Azure Dev Spaces, необходимо предоставить соответствующий файл Dockerfile для создания образа контейнера, в котором выполняется ваш код. Необходимые сведения для написания файла Docker, который отвечает вашим потребностям, см. в статье с [рекомендациями по написанию файлов Docker](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) и справочнике по [файлам Docker](https://docs.docker.com/engine/reference/builder/).

Разместив соответствующий файл Dockerfile, выполните `azds up` для запуска приложения в Azure Dev Spaces.

## <a name="common-issues-when-starting-or-stopping-services-with-azure-dev-spaces"></a>Распространенные проблемы при запуске или остановке служб в Azure Dev Spaces

### <a name="error-config-file-not-found"></a>Ошибка Config file not found (Файл конфигурации не найден)

Такая ошибка может возникать при выполнении `azds up`. `azds up` и `azds prep` нужно выполнять из корневого каталога проекта, который вы хотите запустить в пространстве разработки.

Чтобы устранить эту проблему:
1. Измените текущий каталог на корневую папку, содержащую код службы. 
1. Если у вас нет файла _azds.yaml_ в папке кода, запустите `azds prep` для создания ресурсов Docker, Kubernetes и Azure Dev Spaces.

### <a name="timeout-at-waiting-for-container-image-build-step-with-aks-virtual-nodes"></a>При использовании виртуальных узлов AKS истекает время ожидания на шаге Waiting for container image build (Ожидание сборки образа контейнера)

Превышение времени ожидания происходит при попытке использовать Dev Spaces для запуска службы, которая настроена для выполнения на [виртуальном узле AKS](../aks/virtual-nodes-portal.md). Служба Dev Spaces в настоящее время не поддерживает создание служб или их отладку на виртуальных узлах.

При запуске `azds up` с параметром `--verbose` или включении подробного ведения журнала в Visual Studio можно просмотреть дополнительные сведения:

```cmd
azds up --verbose

Installed chart in 2s
Waiting for container image build...
pods/mywebapi-76cf5f69bb-lgprv: Scheduled: Successfully assigned default/mywebapi-76cf5f69bb-lgprv to virtual-node-aci-linux
Streaming build container logs for service 'mywebapi' failed with: Timed out after 601.3037572 seconds trying to start build logs streaming operation. 10m 1s
Container image build failed
```

Предложенная выше команда демонстрирует, что группа pod службы была назначена виртуальному узлу *virtual-node-aci-linux*.

Чтобы исправить эту проблему, обновите чарт Helm для этой службы, чтобы удалить все значения *nodeSelector* или *tolerations*, которые разрешают службе выполняться на виртуальном узле. Обычно эти значения определяются в файле `values.yaml` диаграммы.

Вы можете по-прежнему использовать кластер AKS с включенной возможностью виртуальных узлов, если служба, которую вы создаете или отлаживаете в Dev Spaces, выполняется на узле виртуальной машины. Выполнение службы на узле виртуальной машины в Dev Spaces считается стандартной конфигурацией.

### <a name="error-could-not-find-a-ready-tiller-pod-when-launching-dev-spaces"></a>Ошибка Could not find a ready Tiller pod (Не удалось найти готовую группу pod Tiller) при запуске Dev Spaces

Эта ошибка возникает, если клиент Helm больше не может обратиться к модулю pod Tiller, который выполняется в кластере.

Чтобы устранить эту проблему, перезапустите узлы агента в кластере.

### <a name="error-release-azds-identifier-spacename-servicename-failed-services-servicename-already-exists-or-pull-access-denied-for-servicename-repository-does-not-exist-or-may-require-docker-login"></a>Ошибка "Release аздс- \<identifier\> - \<spacename\> - \<servicename\> Fail: службы" \<servicename\> "уже существует" или "запрос на принудительный доступ запрещен для \<servicename\> , репозиторий не существует или может требовать" DOCKER Login "

Такие ошибки могут возникать при смешении собственных команд Helm (таких как `helm install`, `helm upgrade` или `helm delete`) с командами Dev Spaces (например, `azds up` и `azds down`) в пределах одного пространства разработки. Это происходит, так как в Dev Spaces есть собственный экземпляр Tiller, который конфликтует с вашим экземпляром Tiller, выполняющимся в этом же пространстве разработки.

В одном кластере AKS можно без проблем использовать команды Helm и команды Dev Spaces, но для каждого пространства имен с поддержкой Dev Spaces можно использовать только один из этих форматов.

Предположим, что вы используете команду Helm для запуска всего приложения в родительском пространстве разработки. Вы можете создать дочерние пространства разработки из этого родителя, с помощью команд Dev Spaces выполнить отдельные служб в дочерних пространствах разработки, и протестировать совместную работу служб. Когда вы будете готовы синхронизировать изменения, выполните команду Helm для развертывания обновленного кода в родительском пространстве разработки. Не используйте `azds up` для запуска обновленной службы в родительском пространстве разработки, так как она будет конфликтовать со службой, изначально запущенной с помощью Helm.

### <a name="existing-dockerfile-not-used-to-build-a-container"></a>Существующие Dockerfile не используются для сборки контейнера

Настройте службу Azure Dev Spaces так, чтобы она указывала на конкретный файл _Dockerfile_ в вашем проекте. Если Azure Dev Spaces не использует файл _Dockerfile_, который вы хотели использовать для сборки своих контейнеров, то можно явно указать Azure Dev Spaces, какой файл Dockerfile нужно использовать. 

Чтобы исправить эту проблему, откройте файл _azds.yaml_, который служба Azure Dev Spaces создала в проекте. Обновите строку *configurations: develop: build: dockerfile*, чтобы она указывала на нужный файл Dockerfile. Пример:

```yaml
...
configurations:
  develop:
    build:
      dockerfile: Dockerfile.develop
```

### <a name="error-unauthorized-authentication-required-when-trying-to-use-a-docker-image-from-a-private-registry"></a>Ошибка Unauthorized: authentication required (Не авторизовано: требуется проверка подлинности) при попытке использовать образ Docker из частного реестра

Вы используете образ Docker из частного реестра, который требует проверки подлинности.

Чтобы устранить эту проблему, можно разрешить Dev Spaces выполнять проверку подлинности и извлечение образов из этого частного реестра с помощью [imagePullSecrets](https://kubernetes.io/docs/concepts/configuration/secret/#using-imagepullsecrets). Чтобы использовать imagePullSecrets, [создайте секрет Kubernetes](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod) в пространстве имен, где вы используете этот образ. Затем укажите секрет в качестве значения для imagePullSecrets в `azds.yaml`.

Следующий пример демонстрирует определение imagePullSecrets в `azds.yaml`.

```yaml
kind: helm-release
apiVersion: 1.1
build:
  context: $BUILD_CONTEXT$
  dockerfile: Dockerfile
install:
  chart: $CHART_DIR$
  values:
  - values.dev.yaml?
  - secrets.dev.yaml?
  set:
    # Optional, specify an array of imagePullSecrets. These secrets must be manually created in the namespace.
    # This will override the imagePullSecrets array in values.yaml file.
    # If the dockerfile specifies any private registry, the imagePullSecret for the registry must be added here.
    # ref: https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
    #
    # This uses credentials from secret "myRegistryKeySecretName".
    imagePullSecrets:
      - name: myRegistryKeySecretName
```

> [!IMPORTANT]
> Значение imagePullSecrets, указанное в `azds.yaml`, переопределяет значение imagePullSecrets в `values.yaml`.

### <a name="error-service-cannot-be-started"></a>Ошибка Service cannot be started (Не удается запустить службу)

Эта ошибка может возникать, когда не удается запустить код службы. Чаще всего причина в пользовательском коде. Чтобы получить дополнительные диагностические сведения, включите более подробное ведение журнала при запуске службы.

Для этого в командной строке выполните `--verbose`. Можно также указать формат вывода с помощью `--output`. Пример:

```cmd
azds up --verbose --output json
```

В Visual Studio:

1. Откройте меню **Сервис > Параметры** и в разделе **Проекты и решения** выберите **Сборка и запуск**.
2. Измените значение параметра **Степень подробности сообщений при сборке проекта MSBuild** на **Подробно** или **Диагностика**.

    ![Снимок экрана диалогового окна "Сервис > Параметры"](media/common/VerbositySetting.PNG)

### <a name="rerunning-a-service-after-controller-re-creation"></a>Повторный запуск службы после повторного создания контроллера

Ошибка *Не удается запустить службу* может появиться при попытке перезапустить службу после удаления и повторного создания контроллера Azure Dev Spaces, связанного с этим кластером. В этом случае подробный вывод содержит следующий текст:

```output
Installing Helm chart...
Release "azds-33d46b-default-webapp1" does not exist. Installing it now.
Error: release azds-33d46b-default-webapp1 failed: services "webapp1" already exists
Helm install failed with exit code '1': Release "azds-33d46b-default-webapp1" does not exist. Installing it now.
Error: release azds-33d46b-default-webapp1 failed: services "webapp1" already exists
```

Эта ошибка связана с тем, что удаление контроллера Dev Spaces не приводит к удалению служб, ранее установленных этим контроллером. Воссоздание контроллера и последующая попытка запустить службы с использованием нового контроллера завершаются неудачно, поскольку старые службы все еще находятся на месте.

Чтобы решить эту проблему, используйте команду `kubectl delete`, чтобы вручную удалить старые службы из кластера, а затем повторно запустите Dev Spaces для установки новых служб.

### <a name="error-service-cannot-be-started-when-using-multi-stage-dockerfiles"></a>Ошибка Service cannot be started (Не удается запустить службу) при использовании многоэтапных файлов Dockerfile

Вы получите сообщение об ошибке *Не удается запустить службу* при использовании многоэтапного Dockerfile. В этом случае подробный вывод содержит следующий текст:

```cmd
$ azds up -v
Using dev space 'default' with target 'AksClusterName'
Synchronizing files...6s
Installing Helm chart...2s
Waiting for container image build...10s
Building container image...
Step 1/12 : FROM [imagename:tag] AS base
Error parsing reference: "[imagename:tag] AS base" is not a valid repository/tag: invalid reference format
Failed to build container image.
Service cannot be started.
```

Эта ошибка возникает из-за того, что Azure Dev Spaces пока не поддерживает многоэтапные сборки. Чтобы избежать многоэтапных сборок, повторно создайте Dockerfile.

### <a name="network-traffic-is-not-forwarded-to-your-aks-cluster-when-connecting-your-development-machine"></a>Сетевой трафик не перенаправляется в кластер AKS при подключении компьютера разработки

При использовании [Azure Dev Spaces для подключения кластера AKS к компьютеру разработки](https://code.visualstudio.com/docs/containers/bridge-to-kubernetes) может возникнуть ситуация, когда сетевой трафик не пересылается между компьютером разработки и кластером AKS.

При подключении кластера AKS к компьютеру разработки Azure Dev Spaces перенаправляет сетевой трафик между кластером AKS и компьютером разработки, изменяя файл `hosts` на компьютере разработки. Azure Dev Spaces создает в `hosts` запись с адресом заменяемой службы Kubernetes в качестве имени узла. Эта запись используется для перенаправления портов, чтобы перенаправлять сетевой трафик между компьютером разработки и кластером AKS. Если служба на компьютере разработки конфликтует с портом службы Kubernetes, которую вы заменяете, Azure Dev Spaces не сможет перенаправить сетевой трафик для службы Kubernetes. Например, служба *Windows BranchCache* обычно привязана к адресу *0.0.0.0:80*, что приводит к конфликту порта 80 на всех локальных IP-адресах.

Чтобы устранить эту проблему, вам нужно отключить все службы или процессы, конфликтующие с портом заменяемой службы Kubernetes. Для поиска служб или процессов на компьютере разработки, которые конфликтуют, можно использовать *netstat* или другие сетевые средства.

Например, чтобы отменить и отключить службу *Windows BranchCache*, сделайте следующее:
* Выполните `services.msc` в командной строке.
* Щелкните правой кнопкой мыши *BranchCache* и выберите *Свойства*.
* Нажмите *Остановить*.
* При желании вы можете отключить службу, установив для параметра *Тип запуска* значение *Отключено*.
* Нажмите кнопку *ОК*.

### <a name="error-no-azureassignedidentity-found-for-podazdsazds-webhook-deployment-id-in-assigned-state"></a>Ошибка "не найдено Азуреассигнедидентити для Pod: аздс/аздс-веб-перехватчика — развертывание- \<id\> в назначенном состоянии"

При запуске службы с Azure Dev Spaces в кластере AKS с установленным [управляемым удостоверением](../aks/use-managed-identity.md) и [управляемыми удостоверениями Pod](../aks/developer-best-practices-pod-security.md#use-pod-managed-identities) процесс может перестать отвечать после шага *установки диаграммы* . Проверив значение *azds-injector-webhook* в пространстве имен *azds*, вы можете столкнуться с такой ошибкой.

Службы, которые Azure Dev Spaces выполняет в кластере, используют управляемое удостоверение кластера для взаимодействия с внутренними службами Azure Dev Spaces, расположенными за пределами кластера. При установке управляемого удостоверения pod на узлах кластера настраиваются сетевые правила для перенаправления всех вызовов учетных данных управляемого удостоверения в [кластерную установку Node Managed Identity (NMI)](https://github.com/Azure/aad-pod-identity#node-managed-identity). Этот контроллер DaemonSet NMI определяет вызывающую группу pod и проверяет, что она отмечена для получения доступа к запрошенному управляемому удостоверению. Azure Dev Spaces не может определить, установлено ли в кластере управляемое удостоверение pod, и не может выполнить необходимую настройку, чтобы разрешить службам Azure Dev Spaces доступ к управляемому удостоверению кластера. Так как службы Azure Dev Spaces не настроены для доступа к управляемому удостоверению кластера, Демон NMI не позволит им получить маркер Azure AD для управляемого удостоверения и не сможет взаимодействовать с Azure Dev Spaces серверными службами.

Чтобы устранить эту проблему, примените [AzurePodIdentityException](https://azure.github.io/aad-pod-identity/docs/configure/application_exception) к *azds-injector-webhook* и обновите все группы pod, инструментированные Azure Dev Spaces, предоставив им доступ к управляемому удостоверению.

Создайте файл с именем *webhookException.yaml* и скопируйте в него следующее определение YAML.

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: azds-infrastructure-exception
  namespace: azds
spec:
  PodLabels:
    azds.io/uses-cluster-identity: "true"
```

Приведенный выше файл создает объект *AzurePodIdentityException* для веб-перехватчика *azds-injector-webhook*. Чтобы развернуть этот объект, используйте `kubectl`.

```cmd
kubectl apply -f webhookException.yaml
```

Чтобы обновить группы pod, инструментированные Azure Dev Spaces, предоставляя им доступ к управляемому удостоверению, обновите *пространство имен* в приведенном ниже определении YAML и выполните `kubectl`, чтобы применить его к каждому пространству разработки.

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: azds-infrastructure-exception
  namespace: myNamespace
spec:
  PodLabels:
    azds.io/instrumented: "true"
```

Кроме того, можно создать объекты *AzureIdentity* и *AzureIdentityBinding* и обновить метки pod для рабочих нагрузок, выполняемых в инструментированных Azure Dev Spaces пространствах разработки, чтобы разрешить доступ к управляемому удостоверению, созданному кластером AKS.

Чтобы получить список сведений об управляемом удостоверении, выполните следующую команду для кластера AKS.

```azurecli
az aks show -g <resourcegroup> -n <cluster> -o json --query "{clientId: identityProfile.kubeletidentity.clientId, resourceId: identityProfile.kubeletidentity.resourceId}"
```

Приведенная выше команда выводит значения *clientId* и *resourceId* для управляемого удостоверения. Пример:

```json
{
  "clientId": "<clientId>",
  "resourceId": "/subscriptions/<subid>/resourcegroups/<resourcegroup>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>"
}
```

Чтобы создать объект *AzureIdentity*, создайте файл с именем *clusteridentity.yaml* и примените следующее определение YAML, включив в него сведения об управляемом удостоверении, полученные из предыдущей команды.

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentity
metadata:
  name: my-cluster-mi
spec:
  type: 0
  ResourceID: /subscriptions/<subid>/resourcegroups/<resourcegroup>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<name>
  ClientID: <clientId>
```

Чтобы создать объект *AzureIdentityBinding*, создайте файл с именем *clusteridentitybinding.yaml* и включите в него следующее определение YAML.

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentityBinding
metadata:
  name: my-cluster-mi-binding
spec:
  AzureIdentity: my-cluster-mi
  Selector: my-label-value
```

Чтобы развернуть объекты *AzureIdentity* и *AzureIdentityBinding*, используйте `kubectl`.

```cmd
kubectl apply -f clusteridentity.yaml
kubectl apply -f clusteridentitybinding.yaml
```

После развертывания объектов *AzureIdentity* и *AzureIdentityBinding* любая рабочая нагрузка с меткой *aadpodidbinding: my-label-value* сможет получить доступ к управляемому удостоверению кластера. Добавьте эту метку и повторно разверните все рабочие нагрузки, которые выполняются в любом пространстве разработки. Пример:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sample
        aadpodidbinding: my-label-value
    spec:
      [...]
```

### <a name="error-cannot-get-connection-details-for-azure-dev-spaces-controller-abc-because-it-is-in-the-failed-state-something-wrong-might-have-happened-with-your-controller"></a>Ошибка "не удается получить сведения о подключении для контроллера Azure Dev Spaces" ABC ", так как он находится в состоянии" сбой ". Возможно, произошла какая-то проблема с контроллером».

Чтобы устранить эту проблему, попробуйте удалить контроллер Azure Dev Spaces из кластера и переустановить его.

```bash
azds remove -g <resource group name> -n <cluster name>
azds controller create --name <cluster name> -g <resource group name> -tn <cluster name>
```

Кроме того, по мере прекращения использования Azure Dev Spaces рассмотрите возможность [перехода на мост в Kubernetes](migrate-to-bridge-to-kubernetes.md) , который обеспечивает лучшую работу.

## <a name="common-issues-using-visual-studio-and-visual-studio-code-with-azure-dev-spaces"></a>Распространенные проблемы с использованием Visual Studio и Visual Studio Code в Azure Dev Spaces

### <a name="error-required-tools-and-configurations-are-missing"></a>Ошибка Required tools and configurations are missing (Отсутствуют необходимые средства и конфигурации)

Эта ошибка может возникать при запуске VS Code: "[Azure Dev Spaces] Отсутствуют необходимые средства и конфигурации для сборки и отладки "[имя проекта]"".
Ошибка означает, что это средство azds.exe не находится в переменной среды PATH, как показано в VS Code.

Запустите VS Code из командной строки, в которой переменная среды PATH задана должным образом.

### <a name="error-required-tools-to-build-and-debug-projectname-are-out-of-date"></a>Ошибка "Required tools to build and debug 'projectname' are out of date " ("Необходимые инструменты для сборки и отладки "projectname" устарели")

Вы увидите эту ошибку в Visual Studio Code, если у вас есть более новая версия расширения VS Code для Azure Dev Spaces, но более старая версия Azure Dev Spaces CLI.

Попробуйте скачать и установить последнюю версию интерфейса командной строки Azure Dev Spaces.

* [Windows](https://aka.ms/get-azds-windows)
* [Mac](https://aka.ms/get-azds-mac)
* [Linux](https://aka.ms/get-azds-linux)

### <a name="error-failed-to-find-debugger-extension-for-typecoreclr"></a>Ошибка: Failed to find debugger extension for type:coreclr (Не удалось найти расширение отладчика для типа coreclr)

Такая ошибка может возникнуть при запуске отладчика Visual Studio Code. Возможно, на компьютере разработки не установлено расширение VS Code для C#. Расширение C# включает поддержку отладки для .NET Core (CoreCLR).

Чтобы устранить эту проблему, установите [расширение VS Code для C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp).

### <a name="error-configured-debug-type-coreclr-is-not-supported"></a>Ошибка Configured debug type 'coreclr' is not supported (Настроенный тип отладки coreclr не поддерживается)

Такая ошибка может возникнуть при запуске отладчика Visual Studio Code. Возможно, на компьютере разработки не установлено расширение VS Code для Azure Dev Spaces.

Чтобы устранить эту проблему, установите расширение VS Code для Azure Dev Spaces.

### <a name="error-invalid-cwd-value-src-the-system-cannot-find-the-file-specified-or-launch-program-srcpath-to-project-binary-does-not-exist"></a>Ошибка Invalid 'cwd' value '/src' (Недопустимое для cwd значение /src) The system cannot find the file specified" ("Недопустимое значение "/src" каталога "cwd". Системе не удается найти указанный файл") или "launch: program '/src/[path to project binary]' does not exist" ("запуск: программа "/src/[путь к двоичному файлу проекта]" не существует")

Такая ошибка может возникнуть при запуске отладчика Visual Studio Code. По умолчанию расширение VS Code использует `src` в качестве рабочего каталога для проекта в контейнере. Эта ошибка может возникать, если вы обновили свой `Dockerfile`, указав другой рабочий каталог.

Чтобы устранить эту проблему, обновите файл `launch.json` в подкаталоге `.vscode` папки проекта. Измените каталог `configurations->cwd`, чтобы он указывал на тот же каталог, что и `WORKDIR`, определенный в `Dockerfile` проекта. Возможно, вам также потребуется обновить директиву `configurations->program`.

### <a name="error-the-pipe-program-azds-exited-unexpectedly-with-code-126"></a>Ошибка The pipe program 'azds' exited unexpectedly with code 126 (Программа канала azds неожиданно завершила работу с кодом 126)

Такая ошибка может возникнуть при запуске отладчика Visual Studio Code.

Чтобы устранить эту проблему, закройте и снова откройте Visual Studio. Перезапустите отладчик.

### <a name="error-internal-watch-failed-watch-enospc-when-attaching-debugging-to-a-nodejs-application"></a>Ошибка Internal watch failed: watch ENOSPC (Сбой внутреннего контрольного значения: контрольное значение ENOSPC) при присоединении отладчика к приложению Node.js

Такая ошибка возникает, если для узла, на котором выполняется pod с приложением Node.js, к которому вы пытаетесь присоединить отладчик, превышено значение *fs.inotify.max_user_watches*. В некоторых случаях [значение *fs.inotify.max_user_watches* по умолчанию будет слишком маленьким для обработки подключения отладчика непосредственно к pod](https://github.com/Azure/AKS/issues/772).

Временное решение для этой проблемы — увеличить значение *fs.inotify.max_user_watches* на каждом узле в кластере и перезапустить узел, чтобы изменения вступили в силу.

## <a name="other-common-issues"></a>Другие распространенные проблемы

### <a name="error-azds-is-not-recognized-as-an-internal-or-external-command-operable-program-or-batch-file"></a>Ошибка Azds is not recognized as an internal or external command, operable program, or batch file (Azds не распознается как внутренняя или внешняя команда, исполняемая программа или пакетный файл)

Такая ошибка может возникать, если средство `azds.exe` неправильно установлено или настроено.

Чтобы устранить эту проблему:

1. Проверьте, есть ли `azds.exe` в расположении %ProgramFiles%/Microsoft SDKs\Azure\Azure Dev Spaces CLI. Если файл есть в этой папке, добавьте это расположение в переменную среды PATH.
2. Если `azds.exe` отсутствует, выполните следующую команду.

    ```azurecli
    az aks use-dev-spaces -n <cluster-name> -g <resource-group>
    ```

### <a name="authorization-error-microsoftdevspacesregisteraction"></a>Ошибка авторизации Microsoft.DevSpaces/register/action

Вам потребуется доступ *владельца* или *участника* в подписке Azure, чтобы управлять Azure Dev Spaces. Эта ошибка авторизации может возникать, если вы пытаетесь управлять Dev Spaces, но у вас нет прав *владельца* или *участника* на доступ к связанной подписке Azure. Пример:

```output
The client '<User email/Id>' with object id '<Guid>' does not have authorization to perform action 'Microsoft.DevSpaces/register/action' over scope '/subscriptions/<Subscription Id>'.
```

Чтобы устранить эту проблему, зарегистрируйте пространство имен `Microsoft.DevSpaces` вручную, используя учетную запись с правами *владельца* или *участника* на доступ к подписке Azure.

```azurecli
az provider register --namespace Microsoft.DevSpaces
```

### <a name="new-pods-arent-starting"></a>Не запускаются новые pod

Инициализатор Kubernetes не может применить Подспек для новых модулей Pod из-за изменения разрешений RBAC Kubernetes на роль *администратора кластера* в кластере. Также возможно, что новая группа pod имеет недопустимый PodSpec (например, учетная запись службы, связанная с этой группой pod, больше не существует). Чтобы получить список групп pod в состоянии *ожидания* из-за проблемы с инициализатором, выполните команду `kubectl get pods`.

```bash
kubectl get pods --all-namespaces --include-uninitialized
```

Эта проблема может повлиять на группы pod во *всех пространствах имен* кластера, даже в пространствах имен без поддержки Azure Dev Spaces.

Чтобы устранить эту проблему, [обновите интерфейс командной строки Dev Spaces до последней версии](./how-to/upgrade-tools.md#update-the-dev-spaces-cli-extension-and-command-line-tools) а затем удалите *azds InitializerConfiguration* из контроллера Azure Dev Spaces.

```azurecli
az aks get-credentials --resource-group <resource group name> --name <cluster name>
```

```bash
kubectl delete InitializerConfiguration azds
```

После удаления *azds InitializerConfiguration* из контроллера Azure Dev Spaces выполните `kubectl delete`, чтобы удалить все группы pod в состоянии *ожидания*. После удаления всех таких групп pod повторите их развертывание.

Если новые группы pod после повторного развертывания снова переходят в состояние *ожидания*,выполните `kubectl delete`, чтобы удалить все группы pod в состоянии *ожидания*. После удаления всех таких групп pod удалите контроллер из кластера и заново установите его.

```bash
azds remove -g <resource group name> -n <cluster name>
azds controller create --name <cluster name> -g <resource group name> -tn <cluster name>
```

После переустановки контроллера повторно разверните все группы pod.

### <a name="incorrect-azure-rbac-permissions-for-calling-dev-spaces-controller-and-apis"></a>Неверные разрешения RBAC Azure для вызова контроллера и API-интерфейсов dev Spaces

Пользователь, который обращается к контроллеру Azure Dev Spaces, должен иметь доступ на чтение *kubeconfig* для администратора в кластере AKS. Например, такое разрешение предоставляется для [встроенной роли администратора кластера Службы Azure Kubernetes](../aks/control-kubeconfig-access.md#available-cluster-roles-permissions). Пользователь, обращающийся к контроллеру Azure Dev Spaces, должен также иметь роль *участника* или *владельца* Azure для контроллера. Дополнительные сведения об изменении разрешений для пользователя в кластере AKS см. [здесь](../aks/control-kubeconfig-access.md#assign-role-permissions-to-a-user-or-group).

Чтобы обновить роль пользователя Azure для контроллера, выполните следующие действия.

1. Войдите на портал Azure по адресу https://portal.azure.com.
1. Перейдите к группе ресурсов, которая содержит нужный контроллер. Обычно это та же группа ресурсов, в которой находится кластер AKS.
1. Установите флажок *Показывать скрытые типы*.
1. Щелкните контроллер.
1. Откройте панель *Управление доступом (IAM)* .
1. Перейдите на вкладку *Назначения ролей*.
1. Щелкните *Добавить* и *Добавить назначение роли*.
    * В поле *Роль* выберите *Участник* или *Владелец*.
    * В поле *Назначение доступа к* выберите *Пользователь, группа или субъект-служба Azure AD*.
    * *Выберите* пользователя, которому вы хотите предоставить разрешения.
1. Выберите команду *Сохранить*.

### <a name="dns-name-resolution-fails-for-a-public-url-associated-with-a-dev-spaces-service"></a>Сбой разрешения DNS-имен для общедоступных URL-адресов, связанных со службой Dev Spaces

Вы можете настроить общедоступную конечную точку URL-адреса для службы, указав параметр `--enable-ingress` в команде `azds prep` или установив флажок `Publicly Accessible` в Visual Studio. Общедоступное DNS-имя автоматически регистрируется при запуске службы в Dev Spaces. Если DNS-имя не зарегистрировано, в веб-браузере во время подключения к общедоступному URL-адресу вы увидите ошибки *Не удается отобразить страницу* или *Этот сайт недоступен*.

Чтобы устранить эту проблему:

* Проверьте состояние всех URL-адресов, связанных со службами Dev Spaces, выполнив следующий код.

  ```console
  azds list-uris
  ```

* Если URL-адрес находится в состоянии *ожидания*, значит Dev Spaces по-прежнему ожидает завершения регистрации DNS. Иногда для выполнения регистрации требуется несколько минут. Для каждой службы Dev Spaces также открывается туннель localhost, который можно использовать во время ожидания регистрации DNS.
* Если URL-адрес остается в состоянии *Ожидание* более 5 минут, это может указывать на проблему с внешним модулем DNS pod, который создает общедоступную конечную точку или контроллер nginx-входа, который приобретает общедоступную конечную точку. Чтобы удалить такие группы pod, можно использовать следующие команды, после чего AKS автоматически восстановит их.
  ```console
  kubectl delete pod -n kube-system -l app=addon-http-application-routing-external-dns
  kubectl delete pod -n kube-system -l app=addon-http-application-routing-nginx-ingress
  ```

### <a name="error-upstream-connect-error-or-disconnectreset-before-headers"></a>Ошибка Upstream connect error or disconnect/reset before headers (Ошибка вышестоящего подключения или отключение/сброс до прохождения заголовков)

Вы можете увидеть эту ошибку при попытке получить доступ к службе. Например, при переходе по URL-адресу службы в браузере. Такая ошибка означает, что порт контейнера недоступен. Это может происходить по следующим причинам:

* Процесс сборки или развертывания контейнера еще не завершен. Это может случиться, если вы запускаете `azds up` или отладчик, а затем пытаетесь получить доступ к контейнеру до его успешного развертывания.
* Конфигурация порта не согласуется в _Dockerfile_, диаграмме Helm и серверном коде, который открывает порт.

Чтобы устранить эту проблему:

1. Если процесс сборки или развертывания еще не завершен, вы можете подождать 2–3 секунды и повторить попытку доступа к службе. 
1. Проверьте конфигурацию порта для следующих ресурсов.
    * **[Диаграмма Helm](https://docs.helm.sh):** Задается значениями `service.port` и `deployment.containerPort` в файле values.yaml, созданному с помощью команды `azds prep`.
    * Все порты, открытые в коде приложения, например в Node.js: `var server = app.listen(80, function () {...}`

### <a name="the-type-or-namespace-name-mylibrary-couldnt-be-found"></a>The type or namespace name "MyLibrary" couldn't be found (Не удалось найти имя типа или пространства имен MyLibrary)

Не удается найти проект библиотеки, который вы используете. По умолчанию контекст сборки в Dev Spaces находится на уровне проекта или службы.  

Чтобы устранить эту проблему:

1. Измените файл `azds.yaml`, указав для контекста сборки уровень решения.
2. Измените файлы `Dockerfile` и `Dockerfile.develop`, чтобы они правильно указывали на файлы проекта, например `.csproj`, относительно нового контекста сборки.
3. Добавьте `.dockerignore` в тот же каталог, где расположен файл `.sln`.
4. Внесите в `.dockerignore` дополнительные записи по мере необходимости.

Пример см. [здесь](https://github.com/sgreenmsft/buildcontextsample).

### <a name="horizontal-pod-autoscaling-not-working-in-a-dev-space"></a>Горизонтальное автомасштабирование pod в пространстве разработки не работает

Когда вы запускаете службу в пространстве разработки, в группу pod этой службы [внедряются дополнительные контейнеры для инструментирования](how-dev-spaces-works-cluster-setup.md#prepare-your-aks-cluster), и для всех контейнеров в этой группе pod должны быть установлены ограничения и запросы ресурсов для горизонтального автомасштабирования pod.

Чтобы устранить эту проблему, примените запрос и ограничение ресурсов к внедренным контейнерам Dev Spaces. Запросы и ограничения ресурсов могут применяться к внедренному контейнеру (devspaces-proxy) путем добавления заметки `azds.io/proxy-resources` в спецификацию pod. Значение должно являться объектом JSON, который представляет раздел resources в спецификации контейнера для proxy.

Ниже приведен пример заметки proxy-resources, которую нужно применить к спецификации pod.
```
azds.io/proxy-resources: "{\"Limits\": {\"cpu\": \"300m\",\"memory\": \"400Mi\"},\"Requests\": {\"cpu\": \"150m\",\"memory\": \"200Mi\"}}"
```

### <a name="enable-azure-dev-spaces-on-an-existing-namespace-with-running-pods"></a>Включение Azure Dev Spaces в существующем пространстве имен с выполняющимися группами pod

Предположим, что у вас уже есть кластер AKS и пространство имен, где выполняются группы pod, и вы хотите включить для него Azure Dev Spaces.

Для этого выполните `use-dev-spaces` и примените `kubectl`, чтобы перезапустить все группы pod в этом пространстве имен.

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKS
az aks use-dev-spaces -g MyResourceGroup -n MyAKS --space my-namespace --yes
```

```console
kubectl -n my-namespace delete pod --all
```

После перезапуска групп pod вы можете сразу использовать это пространство имен с Azure Dev Spaces.

### <a name="enable-azure-dev-spaces-on-aks-cluster-with-restricted-egress-traffic-for-cluster-nodes"></a>Включение Azure Dev Spaces для кластера AKS с ограниченным исходящим трафиком для узлов кластера

Чтобы включить Azure Dev Spaces для кластера AKS, в котором ограничен исходящий трафик от узлов кластера, необходимо разрешить следующие полные доменные имена.

| Полное доменное имя.                                    | Порт      | Использование      |
|-----------------------------------------|-----------|----------|
| cloudflare.docker.com | HTTPS:443 | Для извлечения образов Linux Alpine и других образов Azure Dev Spaces |
| gcr.io | HTTP:443 | Для извлечения образов Helm/Tiller|
| storage.googleapis.com | HTTP:443 | Для извлечения образов Helm/Tiller|

Обновите брандмауэр или конфигурацию безопасности, чтобы разрешить сетевой трафик для всех указанных выше доменных имен и [служб Azure dev Spaces инфраструктуры](../dev-spaces/configure-networking.md#virtual-network-or-subnet-configurations).

### <a name="error-could-not-find-the-cluster-cluster-in-subscription-subscriptionid"></a>Ошибка "не удалось найти кластер \<cluster\> в подписке \<subscriptionId\> "

Такая ошибка может возникнуть, если в файле kubeconfig указаны не те кластер или подписка, которые вы пытаетесь использовать с клиентскими средствами Azure Dev Spaces. Средства Azure Dev Spaces на стороне клиента имитируют поведение *kubectl*, то есть используют [один или несколько файлов kubeconfig](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) для выбора кластера и взаимодействия с ним.

Чтобы устранить эту проблему:

* Измените текущий контекст с помощью `az aks use-dev-spaces -g <resource group name> -n <cluster name>`. Эта команда также включает Azure Dev Spaces в кластере AKS, если это еще не сделано. Вы также можете использовать `kubectl config use-context <cluster name>`, чтобы изменить текущий контекст.
* Выполните `az account show`, чтобы отобразить текущую целевую подписку Azure, и убедитесь, что она указана правильно. Вы можете изменить целевую подписку с помощью `az account set`.

### <a name="error-using-dev-spaces-after-rotating-aks-certificates"></a>Ошибка при использовании Dev Spaces после смены сертификатов AKS

После [смены сертификатов в кластере AKS](../aks/certificate-rotation.md) некоторые операции, например `azds space list` и `azds up`, будут завершаться сбоем. Кроме того, после смены сертификатов в кластере необходимо обновить сертификаты в контроллере Azure Dev Spaces.

Чтобы устранить эту проблему, предоставьте обновленные сертификаты в *kubeconfig* с помощью `az aks get-credentials`, а затем выполните команду `azds controller refresh-credentials`. Пример:

```azurecli
az aks get-credentials -g <resource group name> -n <cluster name>
```

```console
azds controller refresh-credentials -g <resource group name> -n <cluster name>
```

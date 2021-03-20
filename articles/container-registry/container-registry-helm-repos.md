---
title: Хранение Helm диаграмм
description: Узнайте, как хранить диаграммы Helm для приложений Kubernetes с помощью репозиториев в реестре контейнеров Azure.
ms.topic: article
ms.date: 06/12/2020
ms.openlocfilehash: 69b16f35589586787e1c31a0e9755b9030af755d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86537873"
---
# <a name="push-and-pull-helm-charts-to-an-azure-container-registry"></a>Отправка и извлечение Helm диаграмм в реестр контейнеров Azure

Для быстрого развертывания приложений и управления ими в Kubernetes можно использовать [диспетчер пакетов Helm с открытым исходным кодом][helm]. При использовании Helm пакеты приложений определяются как [диаграммы](https://helm.sh/docs/topics/charts/), которые собираются и сохраняются в [репозитории диаграммы Helm](https://helm.sh/docs/topics/chart_repository/).

В этой статье показано, как разместить репозитории диаграмм Helm в реестре контейнеров Azure с помощью команд Helm 3. Во многих случаях для разрабатываемых приложений можно создавать и передавать собственные диаграммы. Дополнительные сведения о создании собственных диаграмм Helm см. в разделе [Руководство разработчика шаблона диаграммы][develop-helm-charts]. Можно также сохранить существующую диаграмму Helm из другого репозитория Helm.

> [!IMPORTANT]
> Поддержка диаграмм Helm в реестре контейнеров Azure сейчас доступна в предварительной версии. Предварительные версии предоставляются при условии, что вы соглашаетесь с дополнительными [условиями использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.

## <a name="helm-3-or-helm-2"></a>Helm 3 или Helm 2?

Для хранения, управления и установки диаграмм Helm используется клиент Helm и интерфейс командной строки Helm. Основные выпуски клиента Helm включают Helm 3 и Helm 2. Дополнительные сведения о различиях версий см. в разделе [часто задаваемые вопросы о версии](https://helm.sh/docs/faq/). 

Helm 3 следует использовать для размещения диаграмм Helm в реестре контейнеров Azure. С Helm 3 вы:

* Может создать один или несколько репозиториев Helm в реестре контейнеров Azure
* Храните Helm 3 диаграммы в реестре как [артефакты OCI](container-registry-image-formats.md#oci-artifacts). В настоящее время Helm 3 поддержка OCI является *экспериментальной*.
* Проверьте подлинность в реестре с помощью `helm registry login` команды.
* Использование `helm chart` команд в интерфейсе командной строки Helm для отправки, извлечения и управления диаграммами Helm в реестре
* Используйте `helm install` для установки диаграмм в кластер Kubernetes из локального кэша репозитория.
> [!NOTE]
> Начиная с Helm 3 команды [AZ контроля доступа Helm][az-acr-helm] для использования с клиентом Helm 2 являются устаревшими. См. [план развития продукта](https://github.com/Azure/acr/blob/master/docs/acr-roadmap.md#acr-helm-ga). Если вы ранее развернули Helm 2 диаграммы, см. статью [Миграция Helm v2 на v3](https://helm.sh/docs/topics/v2_v3_migration/).

## <a name="prerequisites"></a>Предварительные требования

Для сценария, описанного в этой статье, необходимы следующие ресурсы:

- **Реестр контейнеров Azure** в подписке Azure. При необходимости создайте реестр с помощью [портал Azure](container-registry-get-started-portal.md) или [Azure CLI](container-registry-get-started-azure-cli.md).
- **Helm Client версии 3.1.0 или более поздней** — выполните команду `helm version` , чтобы найти текущую версию. Дополнительные сведения о том, как установить и обновить Helm, см. [здесь][helm-install].
- **Кластер Kubernetes** , в котором будет установлена диаграмма Helm. При необходимости создайте [кластер службы Azure Kubernetes][aks-quickstart]. 
- **Azure CLI версии 2.0.71 или более поздней** — выполните команду, `az --version` чтобы найти версию. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli-install].

## <a name="enable-oci-support"></a>Включить поддержку OCI

Используйте `helm version` команду, чтобы убедиться, что установлен Helm 3:

```console
helm version
```

Задайте следующую переменную среды, чтобы включить поддержку OCI в клиенте Helm 3. В настоящее время эта поддержка экспериментальна. 

```console
export HELM_EXPERIMENTAL_OCI=1
```

## <a name="create-a-sample-chart"></a>Создание образца диаграммы

Создайте тестовую диаграмму с помощью следующих команд:

```console
mkdir helmtest

cd helmtest
helm create hello-world
```

В качестве базового примера измените каталог на `templates` папку и сначала удалите содержимое:

```console
cd hello-world/templates
rm -rf *
```

В `templates` папке создайте файл `configmap.yaml` с именем, выполнив следующую команду:

```console
cat <<EOF > configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world-configmap
data:
  myvalue: "Hello World"
EOF
```

Дополнительные сведения о создании и выполнении этого примера см. в разделе [Начало работы](https://helm.sh/docs/chart_template_guide/getting_started/) в документах Helm.

## <a name="save-chart-to-local-registry-cache"></a>Сохранить диаграмму в локальном кэше реестра

Измените каталог на `hello-world` подкаталог. Затем выполните команду, `helm chart save` чтобы сохранить копию диаграммы локально и создать псевдоним с полным именем реестра (все строчные), а также с целевым репозиторием и тегом. 

В следующем примере имя реестра — *миконтаинеррегистри*, целевой репозиторий — *Hello-World*, а целевой тег диаграммы — *v1*, но значения для вашей среды заменяются:

```console
cd ..
helm chart save . hello-world:v1
helm chart save . mycontainerregistry.azurecr.io/helm/hello-world:v1
```

Выполните команду `helm chart list` , чтобы убедиться, что диаграммы сохранены в локальном кэше реестра. Она выводит выходные данные следующего вида:

```console
REF                                                      NAME            VERSION DIGEST  SIZE            CREATED
hello-world:v1                                           hello-world       0.1.0   5899db0 3.2 KiB        2 minutes 
mycontainerregistry.azurecr.io/helm/hello-world:v1       hello-world       0.1.0   5899db0 3.2 KiB        2 minutes
```

## <a name="authenticate-with-the-registry"></a>Проверка подлинности в реестре

Выполните `helm registry login` команду в CLI Helm 3, чтобы выполнить [аутентификацию в реестре](container-registry-authentication.md) , используя учетные данные, подходящие для вашего сценария.

Например, создайте [субъект-службу Azure Active Directory с разрешениями Pull и Push](container-registry-auth-service-principal.md#create-a-service-principal) (роль акрпуш) в реестре. Затем укажите учетные данные субъекта-службы в `helm registry login` . В следующем примере пароль предоставляется с помощью переменной среды:

```console
echo $spPassword | helm registry login mycontainerregistry.azurecr.io \
  --username <service-principal-id> \
  --password-stdin
```

## <a name="push-chart-to-registry"></a>Принудительная отправка диаграммы в реестр

Выполните `helm chart push` команду в CLI Helm 3, чтобы передать диаграмму в полный целевой репозиторий:

```console
helm chart push mycontainerregistry.azurecr.io/helm/hello-world:v1
```

После успешной принудительной отправки выходные данные выглядят примерно так:

```output
The push refers to repository [mycontainerregistry.azurecr.io/helm/hello-world]
ref:     mycontainerregistry.azurecr.io/helm/hello-world:v1
digest:  5899db028dcf96aeaabdadfa5899db025899db025899db025899db025899db02
size:    3.2 KiB
name:    hello-world
version: 0.1.0
```

## <a name="list-charts-in-the-repository"></a>Вывод списка диаграмм в репозитории

Как и в случае с изображениями, хранящимися в реестре контейнеров Azure, можно использовать команды [AZ контроля][az-acr-repository] учетных записей, чтобы отобразить репозитории, в которых размещены диаграммы, а также теги и манифесты диаграммы. 

Например, выполните команду [AZ контроля доступа к репозиторию][az-acr-repository-show] , чтобы просмотреть свойства репозитория, созданного на предыдущем шаге.

```azurecli
az acr repository show \
  --name mycontainerregistry \
  --repository helm/hello-world
```

Она выводит выходные данные следующего вида:

```output
{
  "changeableAttributes": {
    "deleteEnabled": true,
    "listEnabled": true,
    "readEnabled": true,
    "writeEnabled": true
  },
  "createdTime": "2020-03-20T18:11:37.6701689Z",
  "imageName": "helm/hello-world",
  "lastUpdateTime": "2020-03-20T18:11:37.7637082Z",
  "manifestCount": 1,
  "registry": "mycontainerregistry.azurecr.io",
  "tagCount": 1
}
```

Чтобы просмотреть сведения о диаграмме, хранящейся в репозитории, выполните команду AZ запись в [репозитории: Показать-манифесты][az-acr-repository-show-manifests] . Пример:

```azurecli
az acr repository show-manifests \
  --name mycontainerregistry \
  --repository helm/hello-world --detail
```

Выходные данные, сокращенные в этом примере, показывают `configMediaType` `application/vnd.cncf.helm.config.v1+json` :

```output
[
  {
    [...]
    "configMediaType": "application/vnd.cncf.helm.config.v1+json",
    "createdTime": "2020-03-20T18:11:37.7167893Z",
    "digest": "sha256:0c03b71c225c3ddff53660258ea16ca7412b53b1f6811bf769d8c85a1f0663ee",
    "imageSize": 3301,
    "lastUpdateTime": "2020-03-20T18:11:37.7167893Z",
    "mediaType": "application/vnd.oci.image.manifest.v1+json",
    "tags": [
      "v1"
    ]
```

## <a name="pull-chart-to-local-cache"></a>Извлечь диаграмму в локальный кэш

Чтобы установить Helm диаграмму в Kubernetes, эта диаграмма должна находиться в локальном кэше. В этом примере сначала выполните команду, `helm chart remove` чтобы удалить существующую локальную диаграмму с именем `mycontainerregistry.azurecr.io/helm/hello-world:v1` :

```console
helm chart remove mycontainerregistry.azurecr.io/helm/hello-world:v1
```

Выполните команду `helm chart pull` , чтобы скачать диаграмму из реестра контейнеров Azure в локальный кэш:

```console
helm chart pull mycontainerregistry.azurecr.io/helm/hello-world:v1
```

## <a name="export-helm-chart"></a>Экспортировать Helm диаграмму

Для дальнейшей работы с диаграммой экспортируйте ее в локальный каталог с помощью `helm chart export` . Например, экспортируйте диаграмму, которая была извлечена в `install` Каталог:

```console
helm chart export mycontainerregistry.azurecr.io/helm/hello-world:v1 \
  --destination ./install
```

Чтобы просмотреть сведения о экспортированной диаграмме в репозитории, выполните `helm show chart` команду в каталоге, в котором была экспортирована диаграмма.

```console
cd install
helm show chart hello-world
```

Helm возвращает подробные сведения о последней версии диаграммы, как показано в следующем примере выходных данных:

```output
apiVersion: v2
appVersion: 1.16.0
description: A Helm chart for Kubernetes
name: hello-world
type: application
version: 0.1.0    
```

## <a name="install-helm-chart"></a>Установить Helm диаграмму

Выполните команду `helm install` , чтобы установить диаграмму Helm, которая была извлечена в локальный кэш и экспортирована. Укажите имя выпуска, например *михелмтест*, или передайте `--generate-name` параметр. Пример:

```console
helm install myhelmtest ./hello-world
```

Выходные данные после успешной установки диаграммы похожи на следующие:

```console
NAME: myhelmtest
LAST DEPLOYED: Fri Mar 20 14:14:42 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

Чтобы проверить установку, выполните `helm get manifest` команду. 

```console
helm get manifest myhelmtest
```

Команда возвращает данные YAML в `configmap.yaml` файле шаблона.

Выполните команду `helm uninstall` , чтобы удалить выпуск диаграммы в кластере:

```console
helm uninstall myhelmtest
```

## <a name="delete-chart-from-the-registry"></a>Удалить диаграмму из реестра

Чтобы удалить диаграмму из реестра контейнеров, используйте команду [AZ запись репозитория Delete][az-acr-repository-delete] . Выполните следующую команду и подтвердите операцию при появлении запроса:

```azurecli
az acr repository delete --name mycontainerregistry --image helm/hello-world:v1
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о создании и развертывании собственных диаграмм Helm см. в разделе документации, посвященном [разработке диаграмм Helm][develop-helm-charts].
* Дополнительные сведения об установке приложений с помощью Helm в [службе Kubernetes Azure (AKS)](../aks/kubernetes-helm.md).
* Диаграммы Helm можно использовать в процессе создания контейнеров. Дополнительные сведения см. в статье [Использование задач реестра контейнеров Azure][acr-tasks].

<!-- LINKS - external -->
[helm]: https://helm.sh/
[helm-install]: https://helm.sh/docs/intro/install/
[develop-helm-charts]: https://helm.sh/docs/chart_template_guide/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[aks-quickstart]: ../aks/kubernetes-walkthrough.md
[acr-bestpractices]: container-registry-best-practices.md
[az-configure]: /cli/azure/reference-index#az-configure
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-helm]: /cli/azure/acr/helm
[az-acr-repository]: /cli/azure/acr/repository
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
[az-acr-repository-show-tags]: /cli/azure/acr/repository#az-acr-repository-show-tags
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests
[acr-tasks]: container-registry-tasks-overview.md

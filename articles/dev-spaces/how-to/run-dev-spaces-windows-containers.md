---
title: Взаимодействие с контейнерами Windows
services: azure-dev-spaces
ms.date: 01/16/2020
ms.topic: conceptual
description: Узнайте, как запускать Azure Dev Spaces в существующем кластере с контейнерами Windows
keywords: Azure Dev Spaces, пространства разработки, Docker, Kubernetes, Azure, AKS, служба Kubernetes Azure, контейнеры, контейнеры Windows
ms.openlocfilehash: 682af37b79083fb360a17f87624f69239c03549d
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102198189"
---
# <a name="interact-with-windows-containers-using-azure-dev-spaces"></a>Взаимодействие с контейнерами Windows с помощью Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Можно включить Azure Dev Spaces как для новых, так и для существующих пространств имен Kubernetes. Azure Dev Spaces будут запускать и инструментированные службы, работающие в контейнерах Linux. Эти службы также могут взаимодействовать с приложениями, которые работают в контейнерах Windows, в одном пространстве имен. В этой статье показано, как использовать пространства разработки для запуска служб в пространстве имен с существующими контейнерами Windows. В настоящее время нельзя выполнять отладку или присоединение к контейнерам Windows с Azure Dev Spaces.

## <a name="set-up-your-cluster"></a>Настройка кластера

В этой статье предполагается, что у вас уже есть кластер с пулами узлов Windows и Linux. Если необходимо создать кластер с пулами узлов Windows и Linux, следуйте приведенным [здесь][windows-container-cli]инструкциям.

Подключитесь к кластеру с помощью [kubectl][kubectl], клиента командной строки Kubernetes. Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][az-aks-get-credentials]. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Чтобы проверить подключение к кластеру, используйте команду [kubectl get][kubectl-get] для получения списка узлов кластера.

```azurecli-interactive
kubectl get nodes
```

В следующем примере выходных данных показан кластер с узлом Windows и Linux. Перед продолжением убедитесь, что состояние *Готово* для каждого узла.

```console
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmss000000   Ready    agent   13m    v1.14.8
aks-nodepool1-12345678-vmss000001   Ready    agent   13m    v1.14.8
aksnpwin000000                      Ready    agent   108s   v1.14.8
```

Примените [таинт][using-taints] к узлам Windows. Таинт на узлах Windows не позволяет пространствам разработки планировать работу контейнеров Linux на узлах Windows. Следующая команда примера команды применяет таинт к узлу Windows *aksnpwin987654* из предыдущего примера.

```azurecli-interactive
kubectl taint node aksnpwin987654 sku=win-node:NoSchedule
```

> [!IMPORTANT]
> При применении таинт к узлу необходимо настроить сопоставление в шаблоне развертывания службы для запуска службы на этом узле. В примере приложения уже настроено [сопоставление][sample-application-toleration-example] с таинт, настроенным в предыдущей команде.

## <a name="run-your-windows-service"></a>Запуск службы Windows

Запустите службу Windows в кластере AKS и убедитесь, что она находится в *запущенном* состоянии. В этой статье используется [пример приложения][sample-application] для демонстрации службы Windows и Linux, работающей в кластере.

Клонировать пример приложения из GitHub и переходить к `dev-spaces/samples/existingWindowsBackend/mywebapi-windows` каталогу:

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/existingWindowsBackend/mywebapi-windows
```

Пример приложения использует [Helm 3][helm-installed] для запуска службы Windows в кластере. Перейдите в `charts` каталог и используйте Helm для запуска службы Windows:

```console
cd charts/
kubectl create ns dev
helm install windows-service . --namespace dev
```

Приведенная выше команда использует Helm для запуска службы Windows в пространстве имен *dev* . Если у вас нет пространства имен с именем *dev*, оно будет создано.

Используйте `kubectl get pods` команду, чтобы проверить, работает ли служба Windows в кластере. 

```console
$ kubectl get pods --namespace dev --watch
NAME                     READY   STATUS              RESTARTS   AGE
myapi-4b9667d123-1a2b3   0/1     ContainerCreating   0          47s
...
myapi-4b9667d123-1a2b3   1/1     Running             0          98s
```

## <a name="enable-azure-dev-spaces"></a>Включить Azure Dev Spaces

Включите пространства разработки в том же пространстве имен, которое использовалось для запуска службы Windows. Следующая команда включает пространства разработки в пространстве имен *dev* :

```console
az aks use-dev-spaces -g myResourceGroup -n myAKSCluster --space dev --yes
```

## <a name="update-your-windows-service-for-dev-spaces"></a>Обновление службы Windows для пространств разработки

При включении пространств разработки в существующем пространстве имен с уже запущенными контейнерами пространства разработки по умолчанию будут пытаться использовать все новые контейнеры, выполняемые в этом пространстве имен. Пространства разработки также попытаются и инструментировать все новые контейнеры, созданные для службы, уже запущенной в пространстве имен. Чтобы пространства разработки не выполняли инструментирование контейнера, работающего в пространстве имен, добавьте заголовок *без прокси-сервера* в `deployment.yaml` .

Добавьте `azds.io/no-proxy: "true"` в `existingWindowsBackend/mywebapi-windows/charts/templates/deployment.yaml` файл:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  ...
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    ...
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mywebapi.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
        azds.io/no-proxy: "true"
```

Используйте `helm list` для перечисления развертывания службы Windows:

```cmd
$ helm list --namespace dev
NAME             REVISION   UPDATED                    STATUS    CHART            APP VERSION    NAMESPACE
windows-service    1        Wed Jul 24 15:45:59 2019   DEPLOYED  mywebapi-0.1.0   1.0            dev
```

В приведенном выше примере имя развертывания — *Windows-Service*. Обновите службу Windows с помощью новой конфигурации, используя `helm upgrade` :

```cmd
helm upgrade windows-service . --namespace dev
```

Так как вы обновили `deployment.yaml` , модули разработки не будут пытаться выполнять инструментирование службы.

## <a name="run-your-linux-application-with-azure-dev-spaces"></a>Запуск приложения Linux с Azure Dev Spaces

Перейдите в `webfrontend` каталог и используйте `azds prep` `azds up` команды и для запуска приложения Linux в кластере.

```console
cd ../../webfrontend-linux/
azds prep --enable-ingress
azds up
```

`azds prep --enable-ingress`Команда создает диаграмму Helm и файлы dockerfile для приложения.

> [!TIP]
> Azure Dev Spaces использует [Dockerfile и диаграмму Helm](../how-dev-spaces-works-prep.md#prepare-your-code) проекта для сборки и выполнения кода. Но вы можете изменить эти файлы, если нужно определить другой способ сборки и запуска проекта.

`azds up`Команда запускает службу в пространстве имен.

```console
$ azds up
Using dev space 'dev' with target 'myAKSCluster'
Synchronizing files...4s
Installing Helm chart...11s
Waiting for container image build...6s
Building container image...
Step 1/12 : FROM mcr.microsoft.com/dotnet/core/sdk:2.2
...
Step 12/12 : ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
Built container image in 36s
Waiting for container...2s
Service 'webfrontend' port 'http' is available at http://dev.webfrontend.abcdef0123.eus.azds.io/
Service 'webfrontend' port 80 (http) is available via port forwarding at http://localhost:57648
```

Вы можете увидеть, как работает служба, открыв общедоступный URL-адрес, который отображается в выходных данных команды аздс up. В этом примере используется такой общедоступный URL-адрес: `http://dev.webfrontend.abcdef0123.eus.azds.io/`. Перейдите к службе в браузере и щелкните *About (сведения* ) вверху. Убедитесь, что вы видите сообщение от службы *mywebapi* , содержащей версию Windows, используемую контейнером.

![Пример приложения, в котором отображается версия Windows из mywebapi](../media/run-dev-spaces-windows-containers/sample-app.png)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о принципах работы Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Принцип работы Azure Dev Spaces](../how-dev-spaces-works.md)

[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-installed]: https://helm.sh/docs/intro/install/
[sample-application]: https://github.com/Azure/dev-spaces/tree/master/samples/existingWindowsBackend
[sample-application-toleration-example]: https://github.com/Azure/dev-spaces/blob/master/samples/existingWindowsBackend/mywebapi-windows/charts/templates/deployment.yaml#L24-L27
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[using-taints]: ../../aks/use-multiple-node-pools.md#setting-nodepool-taints
[windows-container-cli]: ../../aks/windows-container-cli.md

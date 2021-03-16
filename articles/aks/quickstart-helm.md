---
title: Разработка в Azure Kubernetes Service (AKS) с помощью Helm
description: Используйте Helm с AKS и реестром контейнеров Azure для упаковки и запуска контейнеров приложений в кластере.
services: container-service
author: zr-msft
ms.topic: article
ms.date: 03/15/2021
ms.author: zarhoads
ms.openlocfilehash: 4f5232920853908aa5ad714313ead201494caa0d
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103493088"
---
# <a name="quickstart-develop-on-azure-kubernetes-service-aks-with-helm"></a>Краткое руководство. Разработка в службе Azure Kubernetes Service (AKS) с помощью Helm

[Helm][helm] — это средство упаковки с открытым исходным кодом, помогающее в установке и управлении жизненным циклом приложений Kubernetes. Как и диспетчеры пакетов Linux, такие как *APT* и *Yum*, Helm управляет Kubernetes диаграммами, которые являются пакетами предварительно настроенных ресурсов Kubernetes.

В этом кратком руководстве вы будете использовать Helm для упаковки и запуска приложения в AKS. Дополнительные сведения об установке существующего приложения с помощью Helm см. в инструкции по [установке существующих приложений с использованием Helm в AKS][helm-existing] .

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free).
* [Установленный Azure CLI](/cli/azure/install-azure-cli).
* [Установлен Helm v3][helm-install].

## <a name="create-an-azure-container-registry"></a>Создание реестра контейнеров Azure
Чтобы запустить приложение в кластере AKS с помощью Helm, необходимо сохранить образы контейнеров в реестре контейнеров Azure (запись контроля доступа). Укажите уникальное имя реестра в Azure и состоящий из 5-50 буквенно-цифровых символов. SKU *Базовый* — это оптимизированная по стоимости точка входа для целей разработки, обеспечивающая баланс ресурсов хранения и пропускной способности.

В приведенном ниже примере с помощью команды [AZ запись контроля][az-acr-create] доступа создается для создания записи контроля доступа с именем *михелмакр* в *MyResourceGroup* с номером SKU *Basic* .

```azurecli
az group create --name MyResourceGroup --location eastus
az acr create --resource-group MyResourceGroup --name MyHelmACR --sku Basic
```

Выходные данные будут похожи на приведенный ниже пример. Запишите значение *loginServer* для записи контроля доступа, так как вы будете использовать его на более позднем этапе. В приведенном ниже примере *myhelmacr.azurecr.IO* является *loginServer* для *михелмакр*.

```console
{
  "adminUserEnabled": false,
  "creationDate": "2019-06-11T13:35:17.998425+00:00",
  "id": "/subscriptions/<ID>/resourceGroups/MyResourceGroup/providers/Microsoft.ContainerRegistry/registries/MyHelmACR",
  "location": "eastus",
  "loginServer": "myhelmacr.azurecr.io",
  "name": "MyHelmACR",
  "networkRuleSet": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "MyResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

## <a name="create-an-aks-cluster"></a>Создание кластера AKS

Новому кластеру AKS требуется доступ к записи контроля доступа для извлечения образов контейнеров и их запуска. Используйте эту команду для следующих задач:
* Создайте кластер AKS с именем *мякс* и подключите *михелмакр*.
* Предоставьте кластеру *мякс* доступ к записи контроля доступа *михелмакр* .


```azurecli
az aks create -g MyResourceGroup -n MyAKS --location eastus  --attach-acr MyHelmACR --generate-ssh-keys
```

## <a name="connect-to-your-aks-cluster"></a>Подключение к кластеру AKS

Чтобы подключить кластер Kubernetes локально, используйте клиент командной строки Kubernetes, [kubectl][kubectl]. `kubectl` уже установлен, если используется Azure Cloud Shell. 

1. Установите `kubectl` локально с помощью `az aks install-cli` команды:

    ```azurecli
    az aks install-cli
    ```

2. Настройте `kubectl` для подключения к кластеру Kubernetes с помощью `az aks get-credentials` команды. Следующий пример команды получает учетные данные для кластера AKS с именем *мякс* в *MyResourceGroup*:  

    ```azurecli
    az aks get-credentials --resource-group MyResourceGroup --name MyAKS
    ```

## <a name="download-the-sample-application"></a>Загрузка примера приложения

В этом кратком руководстве используется [пример Node.js приложения из примера репозитория Azure dev Spaces][example-nodejs]. Клонирование приложения из GitHub и переход к `dev-spaces/samples/nodejs/getting-started/webfrontend` каталогу.

```console
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/nodejs/getting-started/webfrontend
```

## <a name="create-a-dockerfile"></a>Создание файла Dockerfile

Создайте новый файл *Dockerfile* с помощью следующих команд:

```dockerfile
FROM node:latest

WORKDIR /webfrontend

COPY package.json ./

RUN npm install

COPY . .

EXPOSE 80
CMD ["node","server.js"]
```

## <a name="build-and-push-the-sample-application-to-the-acr"></a>Создание и отправка примера приложения в запись контроля доступа

Используя предыдущий Dockerfile, выполните команду [AZ запись контроля][az-acr-build] доступа, чтобы собрать и отправить образ в реестр. В `.` конце команды задается расположение Dockerfile (в данном случае — текущий каталог).

```azurecli
az acr build --image webfrontend:v1 \
  --registry MyHelmACR \
  --file Dockerfile .
```

## <a name="create-your-helm-chart"></a>Создание диаграммы Helm

Создайте диаграмму Helm с помощью `helm create` команды.

```console
helm create webfrontend
```

Обновите внешний *интерфейс/значения. YAML*:
* Замените loginServer реестра, записанный на предыдущем шаге, например *myhelmacr.azurecr.IO*.
* Измените `image.repository` на `<loginServer>/webfrontend`.
* Измените `service.type` на `LoadBalancer`.

Пример:

```yml
# Default values for webfrontend.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: myhelmacr.azurecr.io/webfrontend
  pullPolicy: IfNotPresent
...
service:
  type: LoadBalancer
  port: 80
...
```

Обновите `appVersion` `v1` в в *интерфейсе YAML*. Например:

```yml
apiVersion: v2
name: webfrontend
...
# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application.
appVersion: v1
```

## <a name="run-your-helm-chart"></a>Запуск диаграммы Helm

Установите приложение с помощью диаграммы Helm с помощью `helm install` команды.

```console
helm install webfrontend webfrontend/
```

Возвращение общедоступного IP-адреса службы займет несколько минут. Отслеживайте ход выполнения с помощью `kubectl get service` команды с `--watch` аргументом.

```console
$ kubectl get service --watch

NAME                TYPE          CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
webfrontend         LoadBalancer  10.0.141.72   <pending>     80:32150/TCP   2m
...
webfrontend         LoadBalancer  10.0.141.72   <EXTERNAL-IP> 80:32150/TCP   7m
```

Перейдите к подсистеме балансировки нагрузки приложения в браузере, используя `<EXTERNAL-IP>` для просмотра примера приложения.

## <a name="delete-the-cluster"></a>Удаление кластера

Используйте команду [AZ Group Delete][az-group-delete] , чтобы удалить группу ресурсов, кластер AKS, реестр контейнеров, образы контейнеров, хранящиеся в записи контроля доступа, и все связанные ресурсы.

```azurecli-interactive
az group delete --name MyResourceGroup --yes --no-wait
```

> [!NOTE]
> Когда вы удаляете кластер, субъект-служба Azure Active Directory, используемый в кластере AKS, не удаляется. Инструкции по удалению субъекта-службы см. в разделе с [дополнительными замечаниями][sp-delete].
> 
> Управляемые удостоверения администрируются платформой, и их не нужно удалять.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании Helm см. в документации по Helm.

> [!div class="nextstepaction"]
> [Документация по Helm][helm-documentation]

[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-group-delete]: /cli/azure/group#az-group-delete
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[example-nodejs]: https://github.com/Azure/dev-spaces/tree/master/samples/nodejs/getting-started/webfrontend
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[helm]: https://helm.sh/
[helm-documentation]: https://helm.sh/docs/
[helm-existing]: kubernetes-helm.md
[helm-install]: https://helm.sh/docs/intro/install/
[sp-delete]: kubernetes-service-principal.md#additional-considerations

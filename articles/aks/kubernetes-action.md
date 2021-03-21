---
title: Создание, тестирование и развертывание контейнеров в службе Kubernetes Azure с помощью действий GitHub
description: Узнайте, как использовать действия GitHub для развертывания контейнера в Kubernetes
services: container-service
author: azooinmyluggage
ms.topic: article
ms.date: 11/06/2020
ms.author: atulmal
ms.custom: github-actions-azure
ms.openlocfilehash: 94134360de49a066f825cbb0c85712995d90b37f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98761466"
---
# <a name="github-actions-for-deploying-to-kubernetes-service"></a>Действия GitHub для развертывания в службе Kubernetes

[GitHub Actions](https://docs.github.com/en/actions) предоставляет гибкие возможности для создания автоматизированных рабочих процессов жизненного цикла разработки программного обеспечения. Вы можете использовать несколько действий Kubernetes для развертывания в контейнерах из реестра контейнеров Azure в службе Kubernetes Azure с помощью действий GitHub. 

## <a name="prerequisites"></a>Предварительные требования 

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Учетная запись GitHub. Если у вас ее нет, зарегистрируйтесь [бесплатно](https://github.com/join).  
- Рабочий кластер Kubernetes
    - [Руководство. Подготовка приложения для службы Kubernetes Azure](tutorial-kubernetes-prepare-app.md)

## <a name="workflow-file-overview"></a>Общие сведения о файле рабочего процесса

Рабочий процесс определяется файлом YAML (.yml) по пути `/.github/workflows/` в вашем репозитории. Это определение содержит разные шаги и параметры рабочего процесса.

Для рабочего процесса, предназначенного для AKS, файл содержит три раздела:

|Section  |Задания  |
|---------|---------|
|**Аутентификация** | Вход в частный реестр контейнеров (запись контроля доступа) |
|**Сборка** | Сборка & принудительная отправка образа контейнера  |
|**Развертывание** | 1. Настройка целевого кластера AKS |
| |2. Создание секрета Generic/DOCKER-Registry в кластере Kubernetes  |
||3. Развертывание в кластере Kubernetes|

## <a name="create-a-service-principal"></a>Создание субъекта-службы

Вы можете создать [субъект-службу](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) в [Azure CLI](/cli/azure/). Эту команду можно выполнить в [Azure Cloud Shell](https://shell.azure.com/) на портале Azure или с помощью кнопки **Попробовать**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```

В приведенной выше команде замените заполнители ИДЕНТИФИКАТОРом подписки и группой ресурсов. Выходные данные — это учетные данные назначения роли, которые обеспечивают доступ к ресурсу. Команда должна вывести объект JSON, аналогичный этому.

```json
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```
Скопируйте этот объект JSON, который затем можно использовать для выполнения аутентификации из GitHub.

## <a name="configure-the-github-secrets"></a>Настройка секретов GitHub

Выполните действия по настройке секретов:

1. В [GitHub](https://github.com/)перейдите к репозиторию, выберите **параметры > секреты > добавить новый секрет**.

    ![На снимке экрана показана ссылка "добавить новый секрет" для репозитория.](media/kubernetes-action/secrets.png)

2. Вставьте содержимое приведенной выше `az cli` команды в качестве значения переменной Secret. Например, `AZURE_CREDENTIALS`.

3. Аналогичным образом определите следующие дополнительные секреты для учетных данных реестра контейнеров и настройте их в действии входа DOCKER. 

    - REGISTRY_USERNAME
    - REGISTRY_PASSWORD

4. Вы увидите секреты, как показано ниже, после определения.

    ![На снимке экрана показаны существующие секреты для репозитория.](media/kubernetes-action/kubernetes-secrets.png)

##  <a name="build-a-container-image-and-deploy-to-azure-kubernetes-service-cluster"></a>Создание образа контейнера и развертывание в кластере службы Kubernetes Azure

Сборка и отправка образов контейнеров выполняются с помощью `Azure/docker-login@v1` действия. 


```yml
env:
  REGISTRY_NAME: {registry-name}
  CLUSTER_NAME: {cluster-name}
  CLUSTER_RESOURCE_GROUP: {resource-group-name}
  NAMESPACE: {namespace-name}
  APP_NAME: {app-name}
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    
    # Connect to Azure Container registry (ACR)
    - uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY_NAME }}.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }} 
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    # Container build and push to a Azure Container registry (ACR)
    - run: |
        docker build . -t ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}

```

### <a name="deploy-to-azure-kubernetes-service-cluster"></a>Развертывание в кластере службы Kubernetes Azure

Чтобы развернуть образ контейнера в AKS, необходимо будет использовать `Azure/k8s-deploy@v1` действие. Это действие имеет пять параметров:

| **Параметр**  | **Пояснение**  |
|---------|---------|
| **namespace** | Используемых Выберите целевое пространство имен Kubernetes. Если пространство имен не указано, команды будут выполняться в пространстве имен по умолчанию. | 
| **манифесты** |  Необходимости Путь к файлам манифеста, который будет использоваться для развертывания |
| **images** | Используемых Полный URL-адрес ресурса для образов, используемых для подстановок в файлах манифеста |
| **имажепуллсекретс** | Используемых Имя секрета DOCKER-Registry, который уже был настроен в кластере. Каждое из этих имен секретов добавляется в поле Имажепуллсекретс для рабочих нагрузок, найденных во входных файлах манифеста. |
| **kubectl — версия** | Используемых Устанавливает определенную версию двоичного файла kubectl |


Прежде чем можно будет выполнить развертывание в AKS, необходимо задать целевое пространство имен Kubernetes и создать секрет опрашивающего образа. Дополнительные сведения о том, как работают образы, см. в разделе [Извлечение образов из реестра контейнеров Azure в кластер Kubernetes](../container-registry/container-registry-auth-kubernetes.md). 

```yaml
  # Create namespace if doesn't exist
  - run: |
      kubectl create namespace ${{ env.NAMESPACE }} --dry-run -o json | kubectl apply -f -
  
  # Create image pull secret for ACR
  - uses: azure/k8s-create-secret@v1
    with:
      container-registry-url: ${{ env.REGISTRY_NAME }}.azurecr.io
      container-registry-username: ${{ secrets.REGISTRY_USERNAME }}
      container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
      secret-name: ${{ env.SECRET }}
      namespace: ${{ env.NAMESPACE }}
      force: true
```


Завершите развертывание с помощью `k8s-deploy` действия. Замените переменные среды значениями своего приложения. 

```yaml

on: [push]

# Environment variables available to all jobs and steps in this workflow
env:
  REGISTRY_NAME: {registry-name}
  CLUSTER_NAME: {cluster-name}
  CLUSTER_RESOURCE_GROUP: {resource-group-name}
  NAMESPACE: {namespace-name}
  SECRET: {secret-name}
  APP_NAME: {app-name}
  
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    
    # Connect to Azure Container registry (ACR)
    - uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY_NAME }}.azurecr.io
        username: ${{ secrets.REGISTRY_USERNAME }} 
        password: ${{ secrets.REGISTRY_PASSWORD }}
    
    # Container build and push to a Azure Container registry (ACR)
    - run: |
        docker build . -t ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        docker push ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
    
    # Set the target Azure Kubernetes Service (AKS) cluster. 
    - uses: azure/aks-set-context@v1
      with:
        creds: '${{ secrets.AZURE_CREDENTIALS }}'
        cluster-name: ${{ env.CLUSTER_NAME }}
        resource-group: ${{ env.CLUSTER_RESOURCE_GROUP }}
    
    # Create namespace if doesn't exist
    - run: |
        kubectl create namespace ${{ env.NAMESPACE }} --dry-run -o json | kubectl apply -f -
    
    # Create image pull secret for ACR
    - uses: azure/k8s-create-secret@v1
      with:
        container-registry-url: ${{ env.REGISTRY_NAME }}.azurecr.io
        container-registry-username: ${{ secrets.REGISTRY_USERNAME }}
        container-registry-password: ${{ secrets.REGISTRY_PASSWORD }}
        secret-name: ${{ env.SECRET }}
        namespace: ${{ env.NAMESPACE }}
        force: true
    
    # Deploy app to AKS
    - uses: azure/k8s-deploy@v1
      with:
        manifests: |
          manifests/deployment.yml
          manifests/service.yml
        images: |
          ${{ env.REGISTRY_NAME }}.azurecr.io/${{ env.APP_NAME }}:${{ github.sha }}
        imagepullsecrets: |
          ${{ env.SECRET }}
        namespace: ${{ env.NAMESPACE }}
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если кластер Kubernetes, реестр контейнеров и репозиторий больше не требуются, очистите развернутые ресурсы, удалив группу ресурсов и репозиторий GitHub. 

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Дополнительные сведения о службе Kubernetes Azure](/azure/architecture/reference-architectures/containers/aks-start-here)

### <a name="more-kubernetes-github-actions"></a>Другие действия Kubernetes GitHub

* [Установщик инструмента Kubectl](https://github.com/Azure/setup-kubectl) ( `azure/setup-kubectl` ): устанавливает определенную версию Kubectl в средстве выполнения.
* [Kubernetes Set context](https://github.com/Azure/k8s-set-context) ( `azure/k8s-set-context` ): укажите целевой контекст кластера Kubernetes, который будет использоваться другими действиями, или выполните любые команды kubectl.
* [AKS Set context](https://github.com/Azure/aks-set-context) ( `azure/aks-set-context` ): укажите целевой контекст кластера Azure Kubernetes Service.
* [Kubernetes создание секрета](https://github.com/Azure/k8s-create-secret) ( `azure/k8s-create-secret` ): Создайте общий секрет или секрет реестра DOCKER-Registry в кластере Kubernetes.
* [Kubernetes Deploy](https://github.com/Azure/k8s-deploy) ( `azure/k8s-deploy` ): внедрить и развертывать манифесты в кластерах Kubernetes.
* [Setup Helm](https://github.com/Azure/setup-helm) ( `azure/setup-helm` ): установите определенную версию двоичного файла Helm в средстве выполнения.
* [Kubernetes внедрить](https://github.com/Azure/k8s-bake) ( `azure/k8s-bake` ): файл манифеста внедрить, который будет использоваться для развертываний с помощью helm2, кустомизе или компосе.
* [Kubernetes Lint](https://github.com/azure/k8s-lint) ( `azure/k8s-lint` ): Проверьте или Lint файлы манифеста.

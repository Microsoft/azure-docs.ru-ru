---
title: Руководство. Развертывание конфигураций с помощью GitOps в кластерах Kubernetes с поддержкой Azure Arc
description: В этом руководстве показано, как применять конфигурации к кластеру Kubernetes с поддержкой Azure Arc. Чтобы изучить концептуальные основы этого процесса, ознакомьтесь со статьей "Конфигурации и GitOps — Kubernetes с поддержкой Azure Arc".
author: shashankbarsin
ms.author: shasb
ms.service: azure-arc
ms.topic: tutorial
ms.date: 03/02/2021
ms.custom: template-tutorial , devx-track-azurecli
ms.openlocfilehash: 0f1172ffa0d29734e7ec005bf2812eeca215aa0d
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484172"
---
# <a name="tutorial-deploy-configurations-using-gitops-on-an-azure-arc-enabled-kubernetes-cluster"></a>Руководство. Развертывание конфигураций с помощью GitOps в кластерах Kubernetes с поддержкой Azure Arc 

В этом руководстве показано, как применить конфигурации с помощью GitOps в кластерах Kubernetes с поддержкой Azure Arc. Вы узнаете, как:

> [!div class="checklist"]
> * создать конфигурацию для кластера Kubernetes с поддержкой Azure Arc на основе примера репозитория Git;
> * убедиться, что конфигурация успешно создана;
> * применить конфигурацию из частного репозитория Git;
> * проверить конфигурацию Kubernetes.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Существующий подключенный кластер Kubernetes с поддержкой Azure Arc.
    - Если вы еще не подключили кластер, выполните инструкции из пошагового руководства [Подключение кластера Kubernetes с поддержкой Azure Arc](quickstart-connect-cluster.md).
- Понимание преимуществ и архитектуры этой возможности. Дополнительные сведения см. в статье [Конфигурации и GitOps — Kubernetes с поддержкой Azure Arc](conceptual-configurations.md).
- Установите расширение Azure CLI `k8s-configuration` версии 1.0.0 или более поздней версии:
  
  ```azurecli
  az extension add --name k8s-configuration
  ```

    >[!TIP]
    > Если расширение `k8s-configuration` уже установлено, его можно обновить до последней версии, выполнив следующую команду: `az extension update --name k8s-configuration`

## <a name="create-a-configuration"></a>Создание конфигурации

[Пример репозитория](https://github.com/Azure/arc-k8s-demo) для этой статьи создан на основе роли оператора кластера. Манифесты в этом репозитории подготавливают несколько пространств имен, развертывают рабочие нагрузки и предоставляют некоторую конфигурацию для команды. Применение этого репозитория в GitOps создает в кластере следующие ресурсы:

* Пространства имен: `cluster-config`, `team-a`, `team-b`
* развертывание `cluster-config/azure-vote`;
* ConfigMap `team-a/endpoints`.

`config-agent` получает из Azure новые и обновленные конфигурации. Этот процесс занимает до 30 секунд.

Если вы связываете с конфигурацией частный репозиторий, обязательно выполните действия из раздела [Применение конфигурации из частного репозитория Git](#apply-configuration-from-a-private-git-repository).

## <a name="use-azure-cli"></a>Использование Azure CLI
Используя расширение Azure CLI для `k8s-configuration`, свяжите подключенный кластер с [примером репозитория Git](https://github.com/Azure/arc-k8s-demo). 
1. Присвойте этой конфигурации имя `cluster-config`.
1. Поручите агенту развертывание оператора в пространстве имен `cluster-config`.
1. Присвойте этому оператору разрешения `cluster-admin`.

    ```azurecli
    az k8s-configuration create --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --operator-instance-name cluster-config --operator-namespace cluster-config --repository-url https://github.com/Azure/arc-k8s-demo --scope cluster --cluster-type connectedClusters
    ```

    ```output
    {
      "complianceStatus": {
      "complianceState": "Pending",
      "lastConfigApplied": "0001-01-01T00:00:00",
      "message": "{\"OperatorMessage\":null,\"ClusterState\":null}",
      "messageLevel": "3"
      },
      "configurationProtectedSettings": {},
      "enableHelmOperator": false,
      "helmOperatorProperties": null,
      "id": "/subscriptions/<sub id>/resourceGroups/<group name>/providers/Microsoft.Kubernetes/connectedClusters/<cluster name>/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/cluster-config",
      "name": "cluster-config",
      "operatorInstanceName": "cluster-config",
      "operatorNamespace": "cluster-config",
      "operatorParams": "--git-readonly",
      "operatorScope": "cluster",
      "operatorType": "Flux",
      "provisioningState": "Succeeded",
      "repositoryPublicKey": "",
      "repositoryUrl": "https://github.com/Azure/arc-k8s-demo",
      "resourceGroup": "MyRG",
      "sshKnownHostsContents": "",
      "systemData": {
        "createdAt": "2020-11-24T21:22:01.542801+00:00",
        "createdBy": null,
        "createdByType": null,
        "lastModifiedAt": "2020-11-24T21:22:01.542801+00:00",
        "lastModifiedBy": null,
        "lastModifiedByType": null
      },
      "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
      ```

### <a name="use-a-public-git-repository"></a>Использование общедоступного репозитория Git

| Параметр | Формат |
| ------------- | ------------- |
| `--repository-url` | http[s]://server/repo[.git] или git://server/repo[.git]

### <a name="use-a-private-git-repository-with-ssh-and-flux-created-keys"></a>Использование частного репозитория Git с SSH и ключами, созданными с помощью Flux

Добавьте открытый ключ, созданный с помощью Flux, в учетную запись пользователя в поставщике службы Git. Если ключ добавляется в репозиторий, а не в учетную запись пользователя, в URL-адресе укажите `git@` вместо `user@`. 

Дополнительные сведения см. в статье [Применение конфигурации из частного репозитория Git](#apply-configuration-from-a-private-git-repository).


| Параметр | Формат | Примечания
| ------------- | ------------- | ------------- |
| `--repository-url` | ssh://user@server/repo[.git] или user@server:repo[.git] | Можно использовать `git@` вместо `user@`

### <a name="use-a-private-git-repository-with-ssh-and-user-provided-keys"></a>Использование частного репозитория Git с SSH и предоставленными пользователем ключами

Предоставьте собственный закрытый ключ напрямую или через файл. Этот ключ должен иметь [формат PEM](https://aka.ms/PEMformat) и заканчиваться символом перевода строки (\n). 

Добавьте связанный открытый ключ в учетную запись пользователя в поставщике службы Git. Если ключ добавляется в репозиторий, а не в учетную запись пользователя, укажите `git@` вместо `user@`. 

Дополнительные сведения см. в статье [Применение конфигурации из частного репозитория Git](#apply-configuration-from-a-private-git-repository).  

| Параметр | Формат | Примечания |
| ------------- | ------------- | ------------- |
| `--repository-url`  | ssh://user@server/repo[.git] или user@server:repo[.git] | Можно использовать `git@` вместо `user@` |
| `--ssh-private-key` | Ключ в кодировке base64 и [формате PEM](https://aka.ms/PEMformat) | Предоставьте ключ напрямую |
| `--ssh-private-key-file` | Полный путь к локальному файлу | Укажите полный путь к локальному файлу, который содержит ключ в формате PEM

### <a name="use-a-private-git-host-with-ssh-and-user-provided-known-hosts"></a>Использование закрытого узла Git с SSH с предоставленным пользователем списком известных узлов

Оператор Flux сохраняет список общих узлов Git в известном файле hosts для проверки подлинности репозитория Git перед установкой SSH-подключения. Если вы используете *нестандартный* репозиторий Git или собственный узел Git, предоставьте собственный ключ узла, чтобы Flux смог опознать репозиторий. 

Как и закрытые ключи, содержимое known_hosts можно предоставить напрямую или через файл. Для предоставления собственного содержимого соблюдайте [спецификации формата для содержимого known_hosts](https://aka.ms/KnownHostsFormat) и используйте любой из описанных выше сценариев с ключами SSH.

| Параметр | Формат | Примечания |
| ------------- | ------------- | ------------- |
| `--repository-url`  | ssh://user@server/repo[.git] или user@server:repo[.git] | Можно использовать `git@` вместо `user@` |
| `--ssh-known-hosts` | Данные в кодировке base64 | Непосредственное предоставление содержимого known_hosts |
| `--ssh-known-hosts-file` | Полный путь к локальному файлу | Предоставление содержимого known_hosts в локальном файле |

### <a name="use-a-private-git-repository-with-https"></a>Использование частного репозитория Git с HTTPS

| Параметр | Формат | Примечания |
| ------------- | ------------- | ------------- |
| `--repository-url` | https://server/repo [.git] | HTTPS с обычной проверкой подлинности |
| `--https-user` | Необработанные данные или данные в кодировке base64 | Имя пользователя HTTPS |
| `--https-key` | Необработанные данные или данные в кодировке base64 | Личный маркер доступа или пароль HTTPS

>[!NOTE]
>* Чарт для оператора Helm версии 1.2.0 и выше поддерживает закрытую проверку подлинности выпуска HTTPS Helm.
>* Выпуск HTTPS Helm не поддерживается для кластеров, управляемых AKS.
>* Если вам нужно, чтобы Flux обращался к репозиторию Git через прокси-сервер, потребуется настроить для агентов Azure Arc параметры прокси-сервера. Дополнительные сведения см. в разделе [Подключение с использованием исходящего прокси-сервера](./quickstart-connect-cluster.md#connect-using-an-outbound-proxy-server).


## <a name="additional-parameters"></a>Дополнительные параметры

Настройте в конфигурации следующие необязательные параметры.

| Параметр | Описание |
| ------------- | ------------- |
| `--enable-helm-operator`| Параметр для включения поддержки развертываний чартов Helm. |
| `--helm-operator-params` | Значения чарта для оператора Helm (если включено). Например, `--set helm.versions=v3`. |
| `--helm-operator-chart-version` | Версия чарта для оператора Helm (если включено). Используйте версию 1.2.0 или выше. По умолчанию: 1.2.0. |
| `--operator-namespace` | Имя для пространства имен оператора. По умолчанию: default. Длина не более 23 символов. |
| `--operator-params` | Параметры для оператора. Должны быть заданы в одинарных кавычках. Например ```--operator-params='--git-readonly --sync-garbage-collection --git-branch=main'```. 

### <a name="options-supported-in----operator-params"></a>Поддерживаемые параметры в `--operator-params`

| Параметр | Описание |
| ------------- | ------------- |
| `--git-branch`  | Ветвь репозитория Git для манифестов Kubernetes. Значение по умолчанию — master. В новых репозиториях есть корневая ветвь с именем `main`, и тогда здесь нужно задать значение `--git-branch=main`. |
| `--git-path`  | Относительный путь в репозитории Git, где Flux будет размещать манифесты Kubernetes. |
| `--git-readonly` | Этот репозиторий Git будет считаться доступным только для чтения. Flux не будет выполнять попытки записи в него. |
| `--manifest-generation`  | Если параметр включен, Flux будет искать .flux.yaml и запустит Kustomize или другие генераторы манифестов. |
| `--git-poll-interval`  | Период опроса на наличие новых фиксаций в репозитории Git. По умолчанию используется значение `5m` (5 минут). |
| `--sync-garbage-collection`  | Если этот флажок установлен, Flux удалит ресурсы, которые были созданы, но теперь отсутствуют в Git. |
| `--git-label`  | Метка для отслеживания хода синхронизации. Используется для маркировки ветви Git.  Значение по умолчанию — `flux-sync`. |
| `--git-user`  | Имя пользователя для фиксации Git. |
| `--git-email`  | Электронная почта для фиксации Git. 

Если вы не хотите, чтобы Flux записывал данные в репозиторий, а значения `--git-user` и `--git-email` не заданы, значение для `--git-readonly` будет присвоено автоматически.

Дополнительные сведения см. в [документации по Flux](https://aka.ms/FluxcdReadme).

> [!TIP]
> Вы можете создать конфигурацию на портале Azure, используя вкладку **GitOps** для ресурса Kubernetes с поддержкой Azure Arc.

## <a name="validate-the-configuration"></a>Проверка конфигурации

Проверка успешного создания конфигурации выполняется с помощью Azure CLI.

```azurecli
az k8s-configuration show --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

В ресурс конфигурации будет записываться актуальная информация о состояния соответствия, сообщения и отладочные сведения.

```output
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2020-12-10T18:26:52.801000+00:00",
    "message": "...",
    "messageLevel": "Information"
  },
  "configurationProtectedSettings": {},
  "enableHelmOperator": false,
  "helmOperatorProperties": {
    "chartValues": "",
    "chartVersion": ""
  },
  "id": "/subscriptions/<sub id>/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/cluster-config",
  "name": "cluster-config",
  "operatorInstanceName": "cluster-config",
  "operatorNamespace": "cluster-config",
  "operatorParams": "--git-readonly",
  "operatorScope": "cluster",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "...",
  "repositoryUrl": "git://github.com/Azure/arc-k8s-demo.git",
  "resourceGroup": "AzureArcTest",
  "sshKnownHostsContents": null,
  "systemData": {
    "createdAt": "2020-12-01T03:58:56.175674+00:00",
    "createdBy": null,
    "createdByType": null,
    "lastModifiedAt": "2020-12-10T18:30:56.881219+00:00",
    "lastModifiedBy": null,
    "lastModifiedByType": null
},
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

При создании или обновлении конфигурации происходит следующее.

1. Azure Arc `config-agent` проверяет наличие новых или обновленных конфигураций в Azure Resource Manager (`Microsoft.KubernetesConfiguration/sourceControlConfigurations`) и обнаруживает новую конфигурацию `Pending`.
1. `config-agent` считывает свойства конфигурации и создает целевое пространство имен.
1. Служба Azure Arc `controller-manager` создает учетную запись службы Kubernetes и сопоставляет ее с [ClusterRoleBinding или RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) для соответствующих разрешений (область `cluster` или `namespace`). Затем она развертывает экземпляр `flux`.
1. При использовании варианта SSH с ключами, созданными Flux, `flux` создает ключ SSH и записывает открытый ключ в журнал.
1. `config-agent` возвращает сведения о состоянии в ресурс конфигурации в Azure.

В ходе подготовки состояние ресурса конфигурации будет несколько раз меняться. Отслеживайте ход выполнения с помощью приведенной выше команды `az k8s-configuration show ...`:

| Изменение состояния | Описание |
| ------------- | ------------- |
| `complianceStatus`-> `Pending` | Представляет исходное состояние и состояние хода выполнения. |
| `complianceStatus` -> `Installed`  | `config-agent` удалось настроить кластер и развернуть `flux` без ошибок. |
| `complianceStatus` -> `Failed` | У `config-agent` возникла ошибка при развертывании `flux`. Подробные сведения приведены в тексте ответа `complianceStatus.message`. |

## <a name="apply-configuration-from-a-private-git-repository"></a>Применение конфигурации из частного репозитория Git

Если вы используете частный репозиторий Git, нужно настроить открытый ключ SSH в репозитории. Открытый ключ SSH вы можете предоставить самостоятельно, или его создаст Flux. Открытый ключ можно настроить для определенного репозитория Git или для пользователе Git, который имеет доступ к нужному репозиторию. 

### <a name="get-your-own-public-key"></a>Получение собственного открытого ключа

Если вы создали собственные ключи SSH, значит у вас уже есть закрытый и открытый ключи.

#### <a name="get-the-public-key-using-azure-cli"></a>Получение открытого ключа с помощью Azure CLI 

Используйте следующую команду в Azure CLI, если ключи будет создавать Flux.

```console
$ az k8s-configuration show --resource-group <resource group name> --cluster-name <connected cluster name> --name <configuration name> --cluster-type connectedClusters --query 'repositoryPublicKey' 
"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAREDACTED"
```

#### <a name="get-the-public-key-from-the-azure-portal"></a>Получение открытого ключа на портале Azure

Выполните следующие шаги на портале Azure, если ключи будет создавать Flux.

1. На портале Azure перейдите к подключенному кластерному ресурсу.
2. На странице ресурсов выберите GitOps и просмотрите список конфигураций для этого кластера.
3. Выберите конфигурацию, которая использует частный репозиторий Git.
4. В открывшемся окне контекста в нижней части окна скопируйте **открытый ключ репозитория**.

#### <a name="add-public-key-using-github"></a>Добавление открытого ключа с помощью GitHub

Используйте один из следующих методов.

* Вариант 1. Добавьте открытый ключ в учетную запись пользователя (применяется ко всем репозиториям в учетной записи).  
    1. Откройте GitHub и щелкните значок профиля в правом верхнем углу страницы.
    2. Щелкните **Параметры**.
    3. Щелкните **SSH and GPG keys** (Ключи SSH и GPG).
    4. Щелкните **New SSH key** (Новый ключ SSH).
    5. Укажите заголовок.
    6. Вставьте открытый ключ без кавычек.
    7. Щелкните **Add SSH key** (Добавить ключ SSH).

* Вариант 2. Добавьте открытый ключ в качестве ключа развертывания в репозиторий Git (применяется только к этому репозиторию).  
    1. Откройте GitHub и перейдите в свой репозиторий.
    1. Щелкните **Параметры**.
    1. Щелкните **Deploy keys** (Ключи развертывания).
    1. Щелкните **Add deploy key** (Добавить ключ развертывания).
    1. Укажите заголовок.
    1. Поставьте флажок **Allow write access** (Разрешить доступ на запись).
    1. Вставьте открытый ключ без кавычек.
    1. Щелкните **Add New** (Добавить новый).

#### <a name="add-public-key-using-an-azure-devops-repository"></a>Добавление открытого ключа с помощью репозитория Azure DevOps

Чтобы добавить ключ в список ключей SSH, сделайте следующее.

1. В разделе **User Settings** (Параметры пользователя) в правом верхнем углу (рядом с изображением профиля) щелкните **SSH public keys** (Открытые ключи SSH).
1. Выберите **+ New Key** (+ Новый ключ).
1. Укажите имя.
1. Вставьте открытый ключ без кавычек.
1. Нажмите кнопку **Добавить**.

## <a name="validate-the-kubernetes-configuration"></a>Проверка конфигурации Kubernetes

Когда `config-agent` установит экземпляр `flux`, сохраненные в репозитории Git ресурсы станут доступными для кластера. С помощью следующей команды убедитесь, что пространства имен, развертывания и ресурсы успешно созданы.

```console
kubectl get ns --show-labels
```

```output
NAME              STATUS   AGE    LABELS
azure-arc         Active   24h    <none>
cluster-config    Active   177m   <none>
default           Active   29h    <none>
itops             Active   177m   fluxcd.io/sync-gc-mark=sha256.9oYk8yEsRwWkR09n8eJCRNafckASgghAsUWgXWEQ9es,name=itops
kube-node-lease   Active   29h    <none>
kube-public       Active   29h    <none>
kube-system       Active   29h    <none>
team-a            Active   177m   fluxcd.io/sync-gc-mark=sha256.CS5boSi8kg_vyxfAeu7Das5harSy1i0gc2fodD7YDqA,name=team-a
team-b            Active   177m   fluxcd.io/sync-gc-mark=sha256.vF36thDIFnDDI2VEttBp5jgdxvEuaLmm7yT_cuA2UEw,name=team-b
```

Мы видим, что пространства имен `team-a`, `team-b`, `itops` и `cluster-config` созданы.

Оператор `flux` был развернут в пространство имен `cluster-config`, как указано в ресурсе конфигурации:

```console
kubectl -n cluster-config get deploy  -o wide
```

```output
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
cluster-config   1/1     1            1           3h    flux         docker.io/fluxcd/flux:1.16.0   instanceName=cluster-config,name=flux
memcached        1/1     1            1           3h    memcached    memcached:1.5.15               name=memcached
```

## <a name="further-exploration"></a>Дальнейшее исследование

Вы можете изучить другие ресурсы, развернутые в составе репозитория конфигурации, сделав следующее.

```console
kubectl -n team-a get cm -o yaml
kubectl -n itops get all
```
## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите конфигурацию с помощью Azure CLI или портала Azure. После выполнения команды удаления ресурс конфигурации будет сразу же удален из Azure. Полное удаление связанных объектов из кластера должно произойти в течение 10 минут. Если на момент удаления конфигурация находится в состоянии сбоя, полное удаление связанных объектов может занять до часа.

Когда удаляется конфигурация с областью действия `namespace`, Azure Arc не удаляет соответствующее пространство имен, чтобы предотвратить нарушение работы существующих рабочих нагрузок. При необходимости это пространство имен можно удалить вручную с помощью `kubectl`.

```azurecli
az k8s-configuration delete --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

> [!NOTE]
> Никакие изменения в кластере, произошедшие в результате развертываний из отслеживаемого репозитория Git, не удаляются при удалении конфигурации.

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующему учебнику, чтобы узнать, как реализовать непрерывную поставку и непрерывную интеграцию с помощью GitOps.
> [!div class="nextstepaction"]
> [Реализация CI/CD с помощью GitOps](./tutorial-gitops-ci-cd.md)

---
title: Обработку обновлений узла AKS с помощью действий GitHub
titleSuffix: Azure Kubernetes Service
description: Сведения об обновлении узлов AKS с помощью действий GitHub
services: container-service
ms.topic: article
ms.date: 11/27/2020
ms.openlocfilehash: 6876cf1e5044246492e249d8a61060cbeac46f96
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98217963"
---
# <a name="apply-security-updates-to-azure-kubernetes-service-aks-nodes-automatically-using-github-actions"></a>Автоматическое применение обновлений безопасности к узлам Azure Kubernetes Service (AKS) с помощью действий GitHub

Обновления для системы безопасности являются ключевой частью поддержания безопасности и соответствия кластера AKS с последними исправлениями для базовой ОС. Учитываются такие обновления, как исправления безопасности для операционной системы и обновления ядра. Для завершения процесса некоторых обновлений требуется перезагрузка узла.

При выполнении этой команды `az aks upgrade` вы получите простой способ применения обновлений. Команда обрабатывает последние обновления для всех узлов кластера, кордонинг и сток трафика на узлах, а также перезапускает узлы, а затем разрешает трафик на обновленные узлы. При обновлении узлов с помощью другого метода AKS не будет автоматически перезапускать узлы.

> [!NOTE]
> Основное различие между, `az aks upgrade` когда используется с `--node-image-only` флагом, заключается в том, что при его использовании будут обновлены только изображения узла. Если этот параметр не задан, будут обновлены образы узла и версия плоскости управления Kubernetes. Вы можете проверить [документы на наличие управляемых обновлений на узлах][managed-node-upgrades-article] и [документы для обновления кластера][cluster-upgrades-article] , чтобы получить более подробные сведения.

Все узлы Kubernetes работают на стандартной виртуальной машине Azure (ВМ). Эти виртуальные машины могут быть основаны на Windows или Linux. Виртуальные машины под управлением Linux используют образ Ubuntu, при этом ОС настроена на автоматическую проверку наличия обновлений каждую ночь.

При использовании `az aks upgrade` команды Azure CLI создает всплеск новых узлов с последними обновлениями безопасности и ядра, изначально эти узлы блокируются, чтобы предотвратить запланированное выполнение всех приложений до завершения обновления. После завершения Azure кордонс (делает узел недоступным для планирования новых рабочих нагрузок) и выполняет сток (перемещает существующие рабочие нагрузки в другой узел) старых узлов и ункордон новые, эффективно передавая все запланированные приложения на новые узлы.

Этот процесс лучше, чем обновление ядер на основе Linux вручную, так как Linux требует перезагрузки при установке нового обновления ядра. Если вы обновите ОС вручную, потребуется также перезагрузить виртуальную машину, вручную кордонинг и застокть все приложения.

В этой статье показано, как автоматизировать процесс обновления узлов AKS. Вы будете использовать действия GitHub и Azure CLI для создания задачи обновления на основе `cron` автоматических запусков.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Кроме того, нужно установить и настроить Azure CLI версии 2.0.59 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

В этой статье также предполагается, что у вас есть учетная запись [GitHub][github] для создания действий в.

## <a name="create-a-timed-github-action"></a>Создание действия по времени GitHub

`cron` — это служебная программа, которая позволяет выполнять набор команд или заданий по расписанию автоматически. Чтобы создать задание по автоматическому обновлению узлов AKS, вам потребуется репозиторий для размещения своих действий. Как правило, действия GitHub настраиваются в том же репозитории, что и приложение, но можно использовать любой репозиторий. В этой статье мы будем использовать ваш [репозиторий профилей][profile-repository]. Если у вас ее нет, создайте новый репозиторий с тем же именем, что и у имени пользователя GitHub.

1. Перейдите к репозиторию на сайте GitHub.
1. Щелкните вкладку **действия** в верхней части страницы.
1. Если вы уже настроили рабочий процесс в этом репозитории, вы будете перенаправлены в список завершенных запусков, в этом случае нажмите кнопку **создать рабочий процесс** . Если вы первый рабочий процесс в репозитории, GitHub предложит вам несколько шаблонов проектов, щелкните ссылку **настроить рабочий процесс самостоятельно** под текстом описания.
1. Измените рабочий процесс `name` и `on` теги, как показано ниже. В действиях GitHub используется тот же [синтаксис POSIX cron][cron-syntax] , что и для любой системы под управлением Linux. В этом расписании мы сообщаем, что рабочий процесс будет выполняться каждые 15 дней в 03:00.

    ```yml
    name: Upgrade cluster node images
    on:
      schedule:
        - cron: '0 3 */15 * *'
    ```

1. Создайте новое задание, используя приведенные ниже инструкции. Это задание называется `upgrade-node` , выполняется на агенте Ubuntu и подключается к учетной записи Azure CLI для выполнения необходимых действий по обновлению узлов.

    ```yml
    name: Upgrade cluster node images

    on:
      schedule:
        - cron: '0 3 */15 * *'

    jobs:
      upgrade-node:
        runs-on: ubuntu-latest
    ```

## <a name="set-up-the-azure-cli-in-the-workflow"></a>Настройка Azure CLI в рабочем процессе

В этом `steps` разделе вы определите всю работу, которую будет выполнять рабочий процесс для обновления узлов.

Скачайте и войдите в Azure CLI.

1. В правой части экрана действия GitHub найдите *строку поиска Marketplace* и введите **Azure login**.
1. В результате вы получите действие, именуемое именем **входа Azure** , опубликованным **Azure**:

      :::image type="content" source="media/node-upgrade-github-actions/azure-login-search.png" alt-text="В результатах поиска, где показаны две строки, первое действие называется Azure Login, а второй — &quot;реестр контейнеров Azure login":::

1. Щелкните **имя входа Azure**. На следующем экране щелкните **значок копирования** в правом верхнем углу примера кода.

    :::image type="content" source="media/node-upgrade-github-actions/azure-login.png" alt-text="Панель результатов действия входа Azure с образцом кода ниже, красный квадрат вокруг значка копирования выделяет место для щелчка":::

1. Вставьте следующий `steps` код в раздел:

      ```yml
      name: Upgrade cluster node images

      on:
        schedule:
          - cron: '0 3 */15 * *'

      jobs:
        upgrade-node:
          runs-on: ubuntu-latest

          steps:
            - name: Azure Login
              uses: Azure/login@v1.1
              with:
                creds: ${{ secrets.AZURE_CREDENTIALS }}
      ```

1. В Azure CLI выполните следующую команду, чтобы создать новое имя пользователя и пароль.

    ```azurecli-interactive
    az ad sp create-for-rbac -o json
    ```

    Выходные данные должны быть похожи на следующий код JSON:

    ```output
    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "azure-cli-xxxx-xx-xx-xx-xx-xx",
      "name": "http://azure-cli-xxxx-xx-xx-xx-xx-xx",
      "password": "xXxXxXxXx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. **В новом окне браузера** перейдите к репозиторию GitHub и откройте вкладку **Параметры** в репозитории. Щелкните **секреты** , а затем щелкните **новый секрет репозитория**.
1. Для параметра *Имя* используйте значение `AZURE_CREDENTIALS`.
1.  В поле *значение* введите все содержимое из выходных данных предыдущего шага, в котором было создано новое имя пользователя и пароль.

    :::image type="content" source="media/node-upgrade-github-actions/azure-credential-secret.png" alt-text="Форма, показывающая AZURE_CREDENTIALS как заголовок секрета, и выходные данные выполненной команды, вставленной в JSON":::

1. Нажмите кнопку **Добавить секрет**.

Интерфейс командной строки, используемый вашим действием, будет зарегистрирован в учетной записи Azure и готов к выполнению команд.

Чтобы создать шаги для выполнения Azure CLI команд.

1. Перейдите на **страницу поиска** в *GitHub Marketplace* в правой части экрана и выполните поиск *Azure CLI действия*. Выберите *действие Azure CLI в Azure*.

    :::image type="content" source="media/node-upgrade-github-actions/azure-cli-action.png" alt-text="Результат поиска &quot;Azure CLI действие&quot; с первым результатом, отображаемым в Azure":::

1. Нажмите кнопку копирования в *результатах GitHub Marketplace* и вставьте содержимое действия в основной редактор под действием *входа в Azure* , как показано ниже.

    ```yml
    name: Upgrade cluster node images

    on:
      schedule:
        - cron: '0 3 */15 * *'

    jobs:
      upgrade-node:
        runs-on: ubuntu-latest

        steps:
          - name: Azure Login
            uses: Azure/login@v1.1
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
          - name: Upgrade node images
            uses: Azure/cli@v1.0.0
            with:
              inlineScript: az aks upgrade -g {resourceGroupName} -n {aksClusterName} --node-image-only --yes
    ```

    > [!TIP]
    > Вы можете отделить `-g` `-n` Параметры и от команды, добавив их в секреты, аналогичные описанным выше. Замените `{resourceGroupName}` `{aksClusterName}` заполнители и своими секретными аналогами, например `${{secrets.RESOURCE_GROUP_NAME}}` и `${{secrets.AKS_CLUSTER_NAME}}`

1. Переименуйте файл на `upgrade-node-images`.
1. Щелкните **начать фиксацию**, добавьте заголовок сообщения и сохраните рабочий процесс.

После создания фиксации рабочий процесс будет сохранен и готов к выполнению.

> [!NOTE]
> Чтобы обновить пул с одним узлом вместо всех пулов узлов в кластере, добавьте `--name` параметр в `az aks nodepool upgrade` команду, чтобы указать имя пула узлов. Пример:
> ```azurecli-interactive
> az aks nodepool upgrade -g {resourceGroupName} --cluster-name {aksClusterName} --name {{nodePoolName}} --node-image-only
> ```

## <a name="run-the-github-action-manually"></a>Выполнение действия GitHub вручную

Рабочий процесс можно запустить вручную, в дополнение к запланированному выполнению, добавив новый `on` триггер с именем `workflow_dispatch` . Готовый файл должен выглядеть так, как показано ниже YAML:

```yml
name: Upgrade cluster node images

on:
  schedule:
    - cron: '0 3 */15 * *'
  workflow_dispatch:

jobs:
  upgrade-node:
    runs-on: ubuntu-latest

    steps:
      - name: Azure Login
        uses: Azure/login@v1.1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Code for upgrading one or more node pools
```

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о последних образах узлов см. в [заметках о выпуске AKS](https://github.com/Azure/AKS/releases) .
- Узнайте, как обновить версию Kubernetes с помощью [обновления кластера AKS][cluster-upgrades-article].
- Узнайте больше о нескольких пулах узлов и об обновлении пулов узлов с помощью [создания нескольких пулов узлов и управления ими][use-multiple-node-pools].
- Дополнительные сведения о [пулах системных узлов][system-pools]
- Сведения о том, как сэкономить ресурсы с помощью плашечных экземпляров, см. в разделе [Добавление пула узлов смесевых цветов в AKS][spot-pools] .

<!-- LINKS - external -->
[github]: https://github.com
[profile-repository]: https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-github-profile/about-your-profile
[cron-syntax]: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html#tag_20_25_07

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[managed-node-upgrades-article]: node-image-upgrade.md
[cluster-upgrades-article]: upgrade-cluster.md
[system-pools]: use-system-pools.md
[spot-pools]: spot-node-pool.md
[use-multiple-node-pools]: use-multiple-node-pools.md

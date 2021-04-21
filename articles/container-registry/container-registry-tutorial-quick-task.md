---
title: Учебник. Быстрая сборка образов контейнера
description: В этом руководстве описывается, как создать образ контейнеров Docker в Azure с помощью службы "Задачи Реестра контейнеров Azure" ("Задачи ACR"), а затем развернуть его в службу "Экземпляры контейнеров Azure".
ms.topic: tutorial
ms.date: 11/24/2020
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 282e6ea56835fba679510a29af936c1fbcb3ead2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775355"
---
# <a name="tutorial-build-and-deploy-container-images-in-the-cloud-with-azure-container-registry-tasks"></a>Руководство по Создание и развертывание образов контейнера в облаке с помощью службы "Задачи Реестра контейнеров Azure"

[Задачи ACR](container-registry-tasks-overview.md) — это набор компонентов в Реестре контейнеров Azure, предоставляющий упрощенные и эффективные сборки образов контейнера Docker в Azure. В этой статье вы узнаете, как использовать функцию *быстрой задачи* решения "Задачи ACR".

Цикл разработки "внутреннего цикла" — это итеративный процесс написания кода, сборки и тестирования приложения перед фиксацией в системе управления версиями. Функция "Быстрая задача" расширяет внутренний цикл разработки в облако, предоставляя успешную проверку сборки и автоматическую отправку успешно созданных образов в реестр контейнеров. Образы изначально создаются в облаке, ближе к вашему реестру, что обеспечивает быстрое развертывание.

Все знания о файле Dockerfile можно использовать в решении "Задачи ACR". Вам не нужно менять файлы Dockerfile для выполнения сборки в облаке с помощью решения "Задачи ACR", а только выполняемые команды. 

В этом руководстве, которое является первой частью серии, вы:

> [!div class="checklist"]
> * получите исходный код примера приложения;
> * создадите образ контейнера в Azure;
> * Развертывание контейнера в службе "Экземпляры контейнеров Azure"

В последующих руководствах вы узнаете, как использовать задачи сборки решения "Задачи ACR" для автоматической сборки образов контейнеров при фиксации кода и обновлении базового образа. С помощью решения "Задачи ACR" можно также запускать [многошаговые задачи](container-registry-tasks-multi-step.md), используя файл YAML для определения действий по созданию, отправке и тестированию (при необходимости) нескольких контейнеров.

## <a name="prerequisites"></a>Предварительные требования

### <a name="github-account"></a>Учетная запись GitHub

Создайте учетную запись на сайте https://github.com, если у вас ее еще нет. В этой серии руководств репозиторий GitHub используется для демонстрации автоматического создания образов в решении "Задачи ACR".

### <a name="fork-sample-repository"></a>Создание вилки примера репозитория

Затем с помощью пользовательского интерфейса GitHub создайте вилку примера хранилища в своей учетной записи GitHub. В этом руководстве создается образ контейнера из источника в репозитории, а в следующем выполняется отправка фиксации в вилку репозитория для запуска автоматической задачи.

Создайте вилку этого репозитория: https://github.com/Azure-Samples/acr-build-helloworld-node.

![Снимок экрана с выделенной кнопкой "Fork" (Создать вилку) в GitHub][quick-build-01-fork]

### <a name="clone-your-fork"></a>Клонирование вилки

После создания вилки репозитория клонируйте свою вилку и войдите в каталог, содержащий локальный клон.

Клонируйте репозиторий с помощью `git`, заменив **\<your-github-username\>** своим именем пользователя GitHub:

```console
git clone https://github.com/<your-github-username>/acr-build-helloworld-node
```

Введите каталог, содержащий исходный код:

```console
cd acr-build-helloworld-node
```

### <a name="bash-shell"></a>Оболочка Bash

Команды в данной серии руководств отформатированы для оболочки Bash. Если вы предпочитаете использовать PowerShell, командную строку или другую оболочку, может потребоваться скорректировать продолжение строки и формат переменной среды соответствующим образом.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="build-in-azure-with-acr-tasks"></a>Сборка в Azure с использованием решения "Задачи ACR"

Теперь, когда исходный код извлечен на компьютер, выполните следующие действия, чтобы создать реестр контейнеров и образ контейнера с помощью решения "Задачи ACR".

Чтобы упростить выполнение примеров команд, руководства из этой серии используют переменные среды оболочки. Выполните следующую команду, чтобы задать переменную `ACR_NAME`. Замените **\<registry-name\>** уникальным именем нового реестра контейнеров. Имя реестра должно быть уникальным в пределах Azure и состоять только из строчных букв и содержать от 5 до 50 буквенно-цифровых знаков. Другие ресурсы, создаваемые в руководстве, основаны на этом имени, поэтому вам требуется изменить только первую переменную.

```console
ACR_NAME=<registry-name>
```

Теперь, когда переменная среды реестра контейнеров заполнена, можно копировать оставшиеся команды и вставлять их в руководство без изменения каких-либо значений. Для создания группы ресурсов и реестра контейнеров выполните следующие команды:

```azurecli
RES_GROUP=$ACR_NAME # Resource Group name

az group create --resource-group $RES_GROUP --location eastus
az acr create --resource-group $RES_GROUP --name $ACR_NAME --sku Standard --location eastus
```

Теперь, когда у вас есть реестр, используйте решение "Задачи ACR" для создания образа контейнера из примера кода. Выполните команду [az acr build][az-acr-build], чтобы запустить *быструю задачу*:

```azurecli
az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
```

Результат выполнения команды [az acr build][az-acr-build] будет аналогичен следующему примеру. Вы можете просмотреть сведения об отправке исходного кода ("контекста") в Azure и дополнительные сведения об операции `docker build`, которую решение "Задача ACR" выполняет в облаке. Так как решение "Задачи ACR" использует `docker build` для создания образов, чтобы начать немедленно использовать это решение, изменения в файлах Dockerfiles не требуются.

```output
Packing source code into tar file to upload...
Sending build context (4.813 KiB) to ACR...
Queued a build with build ID: da1
Waiting for build agent...
2020/11/18 18:31:42 Using acb_vol_01185991-be5f-42f0-9403-a36bb997ff35 as the home volume
2020/11/18 18:31:42 Setting up Docker configuration...
2020/11/18 18:31:43 Successfully set up Docker configuration
2020/11/18 18:31:43 Logging in to registry: myregistry.azurecr.io
2020/11/18 18:31:55 Successfully logged in
Sending build context to Docker daemon   21.5kB
Step 1/5 : FROM node:15-alpine
15-alpine: Pulling from library/node
Digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
Status: Image is up to date for node:15-alpine
 ---> a56170f59699
Step 2/5 : COPY . /src
 ---> 88087d7e709a
Step 3/5 : RUN cd /src && npm install
 ---> Running in e80e1263ce9a
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN helloworld@1.0.0 No repository field.

up to date in 0.1s
Removing intermediate container e80e1263ce9a
 ---> 26aac291c02e
Step 4/5 : EXPOSE 80
 ---> Running in 318fb4c124ac
Removing intermediate container 318fb4c124ac
 ---> 113e157d0d5a
Step 5/5 : CMD ["node", "/src/server.js"]
 ---> Running in fe7027a11787
Removing intermediate container fe7027a11787
 ---> 20a27b90eb29
Successfully built 20a27b90eb29
Successfully tagged myregistry.azurecr.io/helloacrtasks:v1
2020/11/18 18:32:11 Pushing image: myregistry.azurecr.io/helloacrtasks:v1, attempt 1
The push refers to repository [myregistry.azurecr.io/helloacrtasks]
6428a18b7034: Preparing
c44b9827df52: Preparing
172ed8ca5e43: Preparing
8c9992f4e5dd: Preparing
8dfad2055603: Preparing
c44b9827df52: Pushed
172ed8ca5e43: Pushed
8dfad2055603: Pushed
6428a18b7034: Pushed
8c9992f4e5dd: Pushed
v1: digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1 size: 1366
2020/11/18 18:32:43 Successfully pushed image: myregistry.azurecr.io/helloacrtasks:v1
2020/11/18 18:32:43 Step ID acb_step_0 marked as successful (elapsed time in seconds: 15.648945)
The following dependencies were found:
- image:
    registry: myregistry.azurecr.io
    repository: helloacrtasks
    tag: v1
    digest: sha256:b038dcaa72b2889f56deaff7fa675f58c7c666041584f706c783a3958c4ac8d1
  runtime-dependency:
    registry: registry.hub.docker.com
    repository: library/node
    tag: 15-alpine
    digest: sha256:8dafc0968fb4d62834d9b826d85a8feecc69bd72cd51723c62c7db67c6dec6fa
  git: {}

Run ID: da1 was successful after 1m9.970148252s
```

В конце выходных данных решения "Задачи ACR" отображаются зависимости, обнаруженные для образа. Таким образом решение может автоматизировать создание образов при обновлении базовых образов, например, когда базовый образ обновляется в связи с установкой исправлений ОС или платформы. Вы узнаете о том, как решение "Задачи ACR" поддерживает обновления базовых образов, позже в этом цикле руководств.

## <a name="deploy-to-azure-container-instances"></a>Развертывание в службу "Экземпляры контейнеров Azure"

Решение "Задачи ACR" по умолчанию автоматически отправляет успешно созданные образы в реестр, позволяя сразу развертывать их из реестра.

В этом разделе создается Azure Key Vault и субъект-служба, после чего в службу "Экземпляры контейнеров Azure" развертывается контейнер с использованием учетных данных субъекта-службы.

### <a name="configure-registry-authentication"></a>Настройка проверки подлинности в реестре

Все производственные сценарии должны использовать [субъекты-службы][service-principal-auth] для получения доступа к реестру контейнеров Azure. Субъекты-службы позволяют вам предоставлять контроль доступа на основе ролей к образам вашего контейнера. Например, вы можете настроить субъект-службу с доступом только на извлечение к реестру.

#### <a name="create-a-key-vault"></a>Создайте хранилище ключей.

Если у вас еще нет хранилища в [Azure Key Vault](../key-vault/index.yml), создайте его с помощью Azure CLI, используя следующие команды.

```azurecli
AKV_NAME=$ACR_NAME-vault

az keyvault create --resource-group $RES_GROUP --name $AKV_NAME
```

#### <a name="create-a-service-principal-and-store-credentials"></a>Создание учетной записи субъекта-службы и сохранение учетных данных

Теперь вам нужно создать субъект-службу и сохранить его учетные данные в хранилище ключей.

Используйте команду [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] для создания субъекта-службы и [az keyvault secret set][az-keyvault-secret-set] для сохранения **пароля** субъекта-службы в хранилище:

```azurecli
# Create service principal, store its password in AKV (the registry *password*)
az keyvault secret set \
  --vault-name $AKV_NAME \
  --name $ACR_NAME-pull-pwd \
  --value $(az ad sp create-for-rbac \
                --name $ACR_NAME-pull \
                --scopes $(az acr show --name $ACR_NAME --query id --output tsv) \
                --role acrpull \
                --query password \
                --output tsv)
```

Аргумент `--role` в предыдущей команде настраивает субъект-службу с ролью *acrpull*, которая предоставляет доступ только для извлечения к реестру. Чтобы предоставить доступ для отправки и извлечения данных, измените аргумент `--role` на *acrpush*.

Затем сохраните в хранилище *appId* субъекта-службы, который является **именем пользователя**, переданным в реестр контейнеров Azure для проверки подлинности:

```azurecli
# Store service principal ID in AKV (the registry *username*)
az keyvault secret set \
    --vault-name $AKV_NAME \
    --name $ACR_NAME-pull-usr \
    --value $(az ad sp show --id http://$ACR_NAME-pull --query appId --output tsv)
```

Вы создали хранилище ключей Azure и сохранили в нем два секрета:

* `$ACR_NAME-pull-usr`: идентификатор субъекта-службы для использования в качестве **имени пользователя** реестра контейнеров.
* `$ACR_NAME-pull-pwd`: пароль субъекта-службы для использования в качестве **пароля** реестра контейнеров.

Теперь вы можете ссылаться на эти секреты по имени, когда вы или ваши приложения и службы извлекают образы из реестра.

### <a name="deploy-a-container-with-azure-cli"></a>Развертывание контейнера с помощью Azure CLI

Теперь, когда учетные данные субъекта-службы хранятся в секретах Аzure Key Vault, ваши приложения и службы могут использовать их для доступа к вашему частному реестру.

Выполните следующую команду [az container create][az-container-create], чтобы развернуть экземпляр контейнера. Команда использует учетные данные субъекта-службы, хранящиеся в Azure Key Vault, для проверки подлинности в вашем реестре контейнеров.

```azurecli
az container create \
    --resource-group $RES_GROUP \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --registry-username $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-usr --query value -o tsv) \
    --registry-password $(az keyvault secret show --vault-name $AKV_NAME --name $ACR_NAME-pull-pwd --query value -o tsv) \
    --dns-name-label acr-tasks-$ACR_NAME \
    --query "{FQDN:ipAddress.fqdn}" \
    --output table
```

Значение `--dns-name-label` должно быть уникальным в пределах Azure, поэтому предыдущая команда добавляет имя реестра контейнеров к метке DNS-имени контейнера. В выходных данных команды содержится полное доменное имя (FQDN) контейнера, например:

```output
FQDN
----------------------------------------------
acr-tasks-myregistry.eastus.azurecontainer.io
```

Запишите имя FQDN контейнера. Оно будет использоваться в следующем разделе.

### <a name="verify-the-deployment"></a>Проверка развертывания

Для контроля процесса запуска контейнера используйте команду [az container attach][az-container-attach]:

```azurecli
az container attach --resource-group $RES_GROUP --name acr-tasks
```

Выходные данные `az container attach` сначала отображают состояние контейнера, когда он извлекает образ и запускается, а затем связывает STDOUT и STDERR вашей локальной консоли с аналогичным содержимым контейнера.

```output
Container 'acr-tasks' is in state 'Running'...
(count: 1) (last timestamp: 2020-11-18 18:39:10+00:00) pulling image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2020-11-18 18:39:15+00:00) Successfully pulled image "myregistry.azurecr.io/helloacrtasks:v1"
(count: 1) (last timestamp: 2020-11-18 18:39:17+00:00) Created container
(count: 1) (last timestamp: 2020-11-18 18:39:17+00:00) Started container

Start streaming logs:
Server running at http://localhost:80
```

После появления `Server running at http://localhost:80` перейдите к FQDN контейнера в браузере, чтобы просмотреть работающее приложение. Полное доменное имя должно появиться в выходных данных команды `az container create`, которую вы выполнили в предыдущем разделе.

:::image type="content" source="media/container-registry-tutorial-quick-build/quick-build-02-browser.png" alt-text="Пример приложения, работающего в браузере":::

Чтобы окончательно отсоединить консоль от контейнера, нажмите клавиши `Control+C`.

## <a name="clean-up-resources"></a>Очистка ресурсов

Остановите экземпляр контейнера с помощью команды [az container delete][az-container-delete]:

```azurecli
az container delete --resource-group $RES_GROUP --name acr-tasks
```

Чтобы удалить *все* ресурсы, созданные в этом руководстве, включая реестр контейнеров, хранилище ключей и субъект-службу, выполните следующие команды. Однако эти ресурсы используются в [следующем руководстве](container-registry-tutorial-build-task.md) цикла, поэтому их можно оставить, если вы хотите перейти напрямую к следующему руководству.

```azurecli
az group delete --resource-group $RES_GROUP
az ad sp delete --id http://$ACR_NAME-pull
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы протестировали внутренний цикл с быстрой задачей, настройте **задачу сборки** для активации сборок образов контейнера при фиксации исходного кода в репозитории Git:

> [!div class="nextstepaction"]
> [Руководство. Автоматизация сборок образов контейнера с помощью службы "Сборка Реестра контейнеров Azure"](container-registry-tutorial-build-task.md)

<!-- LINKS - External -->
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az_acr_build
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az_ad_sp_create_for_rbac
[az-container-attach]: /cli/azure/container#az_container_attach
[az-container-create]: /cli/azure/container#az_container_create
[az-container-delete]: /cli/azure/container#az_container_delete
[az-keyvault-create]: /cli/azure/keyvault/secret#az_keyvault_create
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az_keyvault_secret_set
[az-login]: /cli/azure/reference-index#az_login
[service-principal-auth]: container-registry-auth-service-principal.md

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png

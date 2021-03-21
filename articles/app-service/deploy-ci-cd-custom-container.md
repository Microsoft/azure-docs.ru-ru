---
title: CI/CD в пользовательские контейнеры
description: Настройка непрерывного развертывания в пользовательском контейнере Windows или Linux в службе приложений Azure.
keywords: azure app service, linux, docker, acr,oss
author: msangapu-msft
ms.assetid: a47fb43a-bbbd-4751-bdc1-cd382eae49f8
ms.topic: article
ms.date: 03/12/2021
ms.author: msangapu
ms.custom: seodec18
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: bc36325b55f049eebef823d836768fccc39a7615
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103472176"
---
# <a name="continuous-deployment-with-custom-containers-in-azure-app-service"></a>Непрерывное развертывание с пользовательскими контейнерами в службе приложений Azure

В этом руководстве описывается настройка непрерывного развертывания для настраиваемого образа контейнера из управляемых репозиториев [реестра контейнеров Azure](https://azure.microsoft.com/services/container-registry/) или [Docker Hub](https://hub.docker.com).

## <a name="1-go-to-deployment-center"></a>1. Перейдите в центр развертывания

В [портал Azure](https://portal.azure.com)перейдите на страницу управления для приложения службы приложений.

В меню слева щелкните Параметры **центра развертывания**  >  . 

::: zone pivot="container-linux"
## <a name="2-choose-deployment-source"></a>2. Выбор источника развертывания

**Выбор** источника развертывания зависит от сценария.
- **Реестр контейнеров** настраивает CI/CD между реестром контейнеров и службой приложений.
- Параметр **действия GitHub** предназначен для вас, если вы сохраняете исходный код для образа контейнера в GitHub. Действие развертывания, активируемое новыми фиксациями в репозитории GitHub, можно запустить `docker build` и `docker push` непосредственно в реестре контейнеров, а затем обновить приложение службы приложений для запуска нового образа. Дополнительные сведения см. в статье [как CI/CD работает с действиями GitHub](#how-cicd-works-with-github-actions).
- Сведения о настройке CI/CD с **Azure pipelines** см. в статье [развертывание контейнера веб-приложения Azure из Azure pipelines](/devops/pipelines/targets/webapp-on-container-linux).

> [!NOTE]
> Для Docker Compose приложения выберите **Реестр контейнеров**.

Если выбраны действия GitHub, **щелкните** **авторизовать** и следуйте инструкциям на экране авторизации. Если вы уже проверили подлинность с помощью GitHub, можно выполнить развертывание из репозитория другого пользователя, щелкнув **изменить учетную запись**.

После авторизации учетной записи Azure с помощью GitHub **выберите** **организацию**, **репозиторий** и **ветвь** для развертывания.
::: zone-end  

::: zone pivot="container-windows"
## <a name="2-configure-registry-settings"></a>2. Настройка параметров реестра
::: zone-end  
::: zone pivot="container-linux"
## <a name="3-configure-registry-settings"></a>3. Настройка параметров реестра

Чтобы развернуть приложение с несколькими контейнерами (Docker Compose), **выберите** **DOCKER Compose** в списке **тип контейнера**.

Если раскрывающийся список **тип контейнера** не отображается, прокрутите страницу назад до **источника** и **выберите** **Реестр контейнеров**.
::: zone-end

В разделе " **источник реестра**" **выберите** расположение реестра контейнеров. Если он не является ни реестром контейнеров Azure, ни концентратором DOCKER, **выберите** **частный реестр**.

::: zone pivot="container-linux"
> [!NOTE]
> Если приложение с несколькими контейнерами (Docker Compose) использует более одного частного образа, убедитесь, что частные образы находятся в одном частном реестре и доступны с теми же учетными данными пользователя. Если в приложении с несколькими контейнерами используются только общедоступные образы, **выберите** **DOCKER Hub**, даже если некоторые образы не находятся в центре DOCKER.
::: zone-end  

Выполните следующие действия, выбрав вкладку, соответствующую вашему выбору.

# <a name="azure-container-registry"></a>[Реестр контейнеров Azure](#tab/acr)

В раскрывающемся списке **реестра** отображаются реестры в той же подписке, что и ваше приложение. **Выберите** нужный реестр.

> [!NOTE]
> Чтобы выполнить развертывание из реестра в другой подписке, **выберите** в разделе " **источник реестра** " **частный реестр** .

::: zone pivot="container-windows"
**Выберите** **изображение** и **тег** для развертывания. При необходимости **введите** команду запуска в **файле запуска**. 
::: zone-end
::: zone pivot="container-linux"
Выполните следующий шаг в зависимости от **типа контейнера**:
- Для **DOCKER Compose** **выберите** реестр для частных образов. **Щелкните** **выбрать файл** , чтобы отправить [файл DOCKER Compose](https://docs.docker.com/compose/compose-file/), или просто **вставьте** содержимое файла DOCKER Compose в файл **config**.
- Для **одного контейнера** **выберите** **изображение** и **тег** для развертывания. При необходимости **введите** команду запуска в **файле запуска**. 
::: zone-end

Служба приложений добавляет строку в **файл запуска** в [конец `docker run` команды (в качестве `[COMMAND] [ARG...]` сегмента)](https://docs.docker.com/engine/reference/run/) при запуске контейнера.

# <a name="docker-hub"></a>[Docker Hub](#tab/dockerhub)

::: zone pivot="container-windows"
В поле **доступ к репозиторию** **Укажите** , является ли развертываемое изображение открытым или закрытым.
::: zone-end
::: zone pivot="container-linux"
В поле **доступ к репозиторию** **Укажите** , является ли развертываемое изображение открытым или закрытым. Для Docker Compose приложения с одним или несколькими частными образами **выберите** **частный**.
::: zone-end

Если вы выбрали частный образ, **Укажите** имя для **входа** (имя пользователя) и **пароль** учетной записи DOCKER.

::: zone pivot="container-windows"
**Укажите** образ и имя тега в **полном имени образа и теге**, разделенном символом `:` (например, `nginx:latest` ). При необходимости **введите** команду запуска в **файле запуска**. 
::: zone-end
::: zone pivot="container-linux"
Выполните следующий шаг в зависимости от **типа контейнера**:
- Для **DOCKER Compose** **выберите** реестр для частных образов. **Щелкните** **выбрать файл** , чтобы отправить [файл DOCKER Compose](https://docs.docker.com/compose/compose-file/), или просто **вставьте** содержимое файла DOCKER Compose в файл **config**.
- Для **одного контейнера** **Укажите** образ и имя тега в **полном имени образа и теге**, разделенные символом `:` (например, `nginx:latest` ). При необходимости **введите** команду запуска в **файле запуска**. 
::: zone-end

Служба приложений добавляет строку в **файл запуска** в [конец `docker run` команды (в качестве `[COMMAND] [ARG...]` сегмента)](https://docs.docker.com/engine/reference/run/) при запуске контейнера.

# <a name="private-registry"></a>[Частный реестр](#tab/private)

В поле **URL-адрес сервера** **введите** URL-адрес сервера, начиная с **https://**.

В разделе **имя входа** и **пароль** **введите** учетные данные для входа в частный реестр.

::: zone pivot="container-windows"
**Укажите** образ и имя тега в **полном имени образа и теге**, разделенном символом `:` (например, `nginx:latest` ). При необходимости **введите** команду запуска в **файле запуска**. 
::: zone-end
::: zone pivot="container-linux"
Выполните следующий шаг в зависимости от **типа контейнера**:
- Для **DOCKER Compose** **выберите** реестр для частных образов. **Щелкните** **выбрать файл** , чтобы отправить [файл DOCKER Compose](https://docs.docker.com/compose/compose-file/), или просто **вставьте** содержимое файла DOCKER Compose в файл **config**.
- Для **одного контейнера** **Укажите** образ и имя тега в **полном имени образа и теге**, разделенные символом `:` (например, `nginx:latest` ). При необходимости **введите** команду запуска в **файле запуска**. 
::: zone-end

Служба приложений добавляет строку в **файл запуска** в [конец `docker run` команды (в качестве `[COMMAND] [ARG...]` сегмента)](https://docs.docker.com/engine/reference/run/) при запуске контейнера.

-----

::: zone pivot="container-windows"
## <a name="3-enable-cicd"></a>3. Включение CI/CD
::: zone-end
::: zone pivot="container-linux"
## <a name="4-enable-cicd"></a>4. Включение CI/CD
::: zone-end

Служба приложений поддерживает интеграцию CI/CD с реестром контейнеров Azure и DOCKER Hub. Чтобы включить его, **выберите** **включено** в **непрерывном развертывании**.

::: zone pivot="container-linux"
> [!NOTE]
> Если выбрать **действия GitHub** в **источнике**, этот параметр не будет выделяться, так как непрерывная обработка CI/CD выполняется действиями GitHub. Вместо этого вы увидите раздел **конфигурации рабочего процесса** , в котором можно **щелкнуть** **файл предварительного просмотра** , чтобы проверить файл рабочего процесса. Azure фиксирует этот файл в выбранном исходном репозитории GitHub для управления задачами сборки и развертывания. Дополнительные сведения см. в статье [как CI/CD работает с действиями GitHub](#how-cicd-works-with-github-actions).
::: zone-end

При включении этого параметра служба приложений добавляет веб-перехватчик в репозиторий в реестре контейнеров Azure или DOCKER Hub. Ваш репозиторий отправляется в этот веб-перехватчик при каждом обновлении выбранного образа с помощью `docker push` . Веб-перехватчик приводит к перезапуску и запуску приложения службы приложений `docker pull` для получения обновленного образа.

**Для других частных реестров** можно отправить веб-перехватчик вручную или как шаг в конвейере CI/CD. Для получения URL-адреса веб-перехватчика **нажмите кнопку** " **Копировать** " в поле " **URL**".

::: zone pivot="container-linux"
> [!NOTE]
> Поддержка приложений с несколькими контейнерами (Docker Compose) ограничена: 
> - Для реестра контейнеров Azure служба приложений создает веб-перехватчик в выбранном реестре с реестром в качестве области. A `docker push` в любой репозиторий в реестре (включая те, на которые не ссылается файл DOCKER Compose) запускает перезапуск приложения. Может потребоваться [Изменить веб-перехватчик](../container-registry/container-registry-webhook.md) до более узких областей.
> - Центр DOCKER не поддерживает веб-перехватчики на уровне реестра. Веб-перехватчики необходимо **Добавить** вручную в образы, указанные в файле DOCKER Compose.
::: zone-end

::: zone pivot="container-windows"
## <a name="4-save-your-settings"></a>4. Сохраните параметры
::: zone-end
::: zone pivot="container-linux"
## <a name="5-save-your-settings"></a>5. Сохраните параметры
::: zone-end

**Нажмите кнопку** **сохранить**.

::: zone pivot="container-linux"

## <a name="how-cicd-works-with-github-actions"></a>Как CI/CD работает с действиями GitHub

Если выбрать **действия GitHub** в **источнике** (см. раздел [Выбор источника развертывания](#2-choose-deployment-source)), служба приложений настроит CI/CD следующими способами:

- Придепозитайте файл рабочего процесса действий GitHub в репозиторий GitHub для обработки задач сборки и развертывания в службе приложений.
- Добавляет учетные данные частного реестра в виде секретов GitHub. Созданный файл рабочего процесса выполняет действие [Azure/DOCKER-login](https://github.com/Azure/docker-login) для входа с помощью частного реестра, а затем запускает `docker push` его для развертывания.
- Добавляет профиль публикации приложения в качестве секрета GitHub. Созданный файл рабочего процесса использует этот секрет для проверки подлинности в службе приложений, а затем запускает действие [Azure/WebDeploy-Deploy](https://github.com/Azure/webapps-deploy) для настройки обновленного образа, который запускает перезапуск приложения для извлечения обновленного образа.
- Захватывает сведения из [журналов выполнения рабочего процесса](https://docs.github.com/actions/managing-workflow-runs/using-workflow-run-logs) и отображает их на вкладке **журналы** в **центре развертывания** приложения.

Настроить поставщик сборки действий GitHub можно следующими способами.

- Настройте файл рабочего процесса после его создания в репозитории GitHub. Дополнительные сведения см. в разделе [синтаксис рабочего процесса для действий GitHub](https://docs.github.com/actions/reference/workflow-syntax-for-github-actions). Просто убедитесь, что рабочий процесс завершается действием [Azure/WebDeploy-Deploy](https://github.com/Azure/webapps-deploy) , чтобы запустить перезапуск приложения.
- Если выбранная ветвь защищена, вы по-прежнему можете просмотреть файл рабочего процесса без сохранения конфигурации, а затем добавить его и необходимые секреты GitHub в репозиторий вручную. Этот метод не обеспечивает интеграцию журналов с портал Azure.
- Вместо профиля публикации разверните его с помощью [субъекта-службы](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) в Azure Active Directory.

#### <a name="authenticate-with-a-service-principal"></a>Проверка подлинности с помощью субъекта-службы

Эта необязательная конфигурация заменяет проверку подлинности по умолчанию профилями публикации в созданном файле рабочего процесса.

**Создание** субъекта-службы с помощью команды [AZ AD SP Create-for-RBAC](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) в [Azure CLI](/cli/azure/). В следующем примере замените *\<subscription-id>* , *\<group-name>* и *\<app-name>* собственными значениями. **Сохраните** все выходные данные JSON для следующего шага, включая верхний уровень `{}` .

```azurecli-interactive
az ad sp create-for-rbac --name "myAppDeployAuth" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

> [!IMPORTANT]
> Для обеспечения безопасности предоставьте минимально необходимый доступ к субъекту-службе. Область в предыдущем примере ограничена конкретным приложением службы приложений, а не всей группой ресурсов.

В [GitHub](https://github.com/) **перейдите** к репозиторию, а затем **выберите** **Параметры > секреты > добавить новый секрет**. **Вставьте** все выходные данные JSON из команды Azure CLI в поле значения секрета. **Присвойте** секрету имя, например `AZURE_CREDENTIALS` .

В файле рабочего процесса, созданном **центром развертывания**, **измените** `azure/webapps-deploy` Шаг с помощью кода, как показано в следующем примере:

```yaml
- name: Sign in to Azure 
# Use the GitHub secret you added
- uses: azure/login@v1
    with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
- name: Deploy to Azure Web App
# Remove publish-profile
- uses: azure/webapps-deploy@v2
    with:
    app-name: '<app-name>'
    slot-name: 'production'
    images: '<registry-server>/${{ secrets.AzureAppService_ContainerUsername_... }}/<image>:${{ github.sha }}'
    - name: Sign out of Azure
    run: |
    az logout
```

::: zone-end

## <a name="automate-with-cli"></a>Автоматизация с помощью CLI

Чтобы настроить реестр контейнеров и образ DOCKER, **выполните команду** [AZ webapp config Container Set](/cli/azure/webapp/config/container#az-webapp-config-container-set).

# <a name="azure-container-registry"></a>[Реестр контейнеров Azure](#tab/acr)

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name '<image>:<tag>' --docker-registry-server-url 'https://<registry-name>.azurecr.io' --docker-registry-server-user '<username>' --docker-registry-server-password '<password>'
```

# <a name="docker-hub"></a>[Docker Hub](#tab/dockerhub)

```azurecli-interactive
# Public image
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name>

# Private image
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

# <a name="private-registry"></a>[Частный реестр](#tab/private)

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name '<image>:<tag>' --docker-registry-server-url <private-repo-url> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

-----

::: zone pivot="container-linux"
Чтобы настроить многоконтейнерное приложение (Docker Compose), **Подготовьте** DOCKER Compose файл локально, а затем **выполните** команду [AZ webapp config Container Set](/cli/azure/webapp/config/container#az-webapp-config-container-set) с `--multicontainer-config-file` параметром. Если файл Docker Compose содержит частные образы, **добавьте** `--docker-registry-server-*` Параметры, как показано в предыдущем примере.

```azurecli-interactive
az webapp config container set --resource-group <group-name> --name <app-name> --multicontainer-config-file <docker-compose-file>
```
::: zone-end

Чтобы настроить CI/CD из реестра контейнеров для приложения, **выполните команду** [AZ webapp Deployment Container Configuration](/cli/azure/webapp/deployment/container#az-webapp-deployment-container-config) с `--enable-cd` параметром. Команда выводит URL-адрес веб перехватчика, но вы должны вручную создать его в реестре на отдельном шаге. В следующем примере включается CI/CD для приложения, а затем для создания веб-перехватчика в реестре контейнеров Azure в выходных данных используется URL веб-перехватчика.

```azurecli-interactive
ci_cd_url=$(az webapp deployment container config --name <app-name> --resource-group <group-name> --enable-cd true --query CI_CD_URL --output tsv)

az acr webhook create --name <webhook-name> --registry <registry-name> --resource-group <group-name> --actions push --uri $ci_cd_url --scope '<image>:<tag>'
```

## <a name="more-resources"></a>Дополнительные ресурсы

* [Реестр контейнеров Azure](https://azure.microsoft.com/services/container-registry/)
* [Создание веб-приложения .NET Core в службе приложений на платформе Linux](quickstart-dotnetcore.md)
* [Краткое руководство. Запуск пользовательского контейнера в Службе приложений](quickstart-custom-container.md)
* [Служба приложений в Linux: вопросы и ответы](faq-app-service-linux.md)
* [Настройка пользовательских контейнеров](configure-custom-container.md)
* [Рабочие процессы действий для развертывания в Azure](https://github.com/Azure/actions-workflow-samples)

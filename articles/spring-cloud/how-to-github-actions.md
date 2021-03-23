---
title: Azure весны CI/CD в облаке с действиями GitHub
description: Создание рабочего процесса CI/CD для Azure Веснного облака с помощью действий GitHub
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/08/2020
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 8400fcacbfa4c76aceb079b788255e3d3b83ce33
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104878293"
---
# <a name="azure-spring-cloud-cicd-with-github-actions"></a>Azure весны CI/CD в облаке с действиями GitHub

Действия GitHub поддерживают автоматизированный рабочий процесс жизненного цикла разработки программного обеспечения. С помощью действий GitHub для Azure "пружинное облако" вы можете создавать рабочие процессы в репозитории для создания, тестирования, упаковки, выпуска и развертывания в Azure. 

## <a name="prerequisites"></a>Предварительные требования
Для этого примера требуется [Azure CLI](/cli/azure/install-azure-cli).

::: zone pivot="programming-language-csharp"
## <a name="set-up-github-repository-and-authenticate"></a>Настройка репозитория GitHub и проверка подлинности
Для авторизации действия входа в Azure требуются учетные данные субъекта-службы Azure. Чтобы получить учетные данные Azure, выполните следующие команды на локальном компьютере:

```
az login
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth 
```

Для доступа к определенной группе ресурсов можно уменьшить область:

```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```

Команда должна вывести объект JSON:

```JSON
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    ...
}
```

В этом примере используется [Пример стилтое на GitHub](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample).  Разветвление репозитория, откройте страницу репозитория GitHub для вилки и перейдите на вкладку **Параметры** . Откройте меню **секреты** и выберите **новый секрет**:

 ![Добавить новый секрет](./media/github-actions/actions1.png)

Задайте в качестве имени секрета `AZURE_CREDENTIALS` и его значение в строке JSON, которую вы нашли в заголовке, *настройте репозиторий GitHub и пройдите проверку подлинности*.

 ![Настройка секретных данных](./media/github-actions/actions2.png)

Вы также можете получить учетные данные для входа в Azure из Key Vault в действиях GitHub, как описано в статьях [Проверка подлинности Azure с помощью Key Vault в действиях GitHub](./spring-cloud-github-actions-key-vault.md).

## <a name="provision-service-instance"></a>Инициализация экземпляра службы
Для подготовки экземпляра облачной службы Azure весны выполните следующие команды с помощью Azure CLI.

```azurecli
az extension add --name spring-cloud
az group create --location eastus --name <resource group name>
az spring-cloud create -n <service instance name> -g <resource group name>
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/xxx/Azure-Spring-Cloud-Samples --label main --search-paths steeltoe-sample/config
```

## <a name="build-the-workflow"></a>Создание рабочего процесса
Рабочий процесс определяется с помощью следующих параметров.

### <a name="prepare-for-deployment-with-azure-cli"></a>Подготовка к развертыванию с помощью Azure CLI

В `az spring-cloud app create` настоящее время команда не идемпотентными. После однократного запуска вы получите сообщение об ошибке, если выполнить ту же команду еще раз. Этот рабочий процесс рекомендуется для существующих облачных приложений и экземпляров Azure весны.

Для подготовки используйте следующие Azure CLI команды:
```
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
az spring-cloud app create --name planet-weather-provider
az spring-cloud app create --name solar-system-weather
```

### <a name="deploy-with-azure-cli-directly"></a>Прямое развертывание с помощью Azure CLI

Создайте `.github/workflows/main.yml` файл в репозитории со следующим содержимым. Замените `<your resource group name>` и `<your service name>` правильными значениями.

```yaml
name: Steeltoe-CD

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the main branch
on:
  push:
    branches: [ main]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    env:
      working-directory: ./steeltoe-sample
      resource-group-name: <your resource group name>
      service-name: <your service name>
    
    # Supported .NET Core version matrix.
    strategy:
      matrix:
        dotnet: [ '3.1.x' ]

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Set up .NET Core 3.1 SDK
      - uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ matrix.dotnet }}
          
      # Set credential for az login
      - uses: azure/login@v1.1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: install Azure CLI extension
        run: |
          az extension add --name spring-cloud --yes
      
      - name: Build and package planet-weather-provider app
        working-directory: ${{env.working-directory}}/src/planet-weather-provider
        run: |
          dotnet publish
          az spring-cloud app deploy -n planet-weather-provider --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll --artifact-path ./publish-deploy-planet.zip -s ${{ env.service-name }} -g ${{ env.resource-group-name }}
      - name: Build solar-system-weather app
        working-directory: ${{env.working-directory}}/src/solar-system-weather
        run: |
          dotnet publish
          az spring-cloud app deploy -n solar-system-weather --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll --artifact-path ./publish-deploy-solar.zip -s ${{ env.service-name }} -g ${{ env.resource-group-name }}
```
::: zone-end

::: zone pivot="programming-language-java"
## <a name="set-up-github-repository-and-authenticate"></a>Настройка репозитория GitHub и проверка подлинности
Для авторизации действия входа в Azure требуются учетные данные субъекта-службы Azure. Чтобы получить учетные данные Azure, выполните следующие команды на локальном компьютере:
```
az login
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth 
```
Для доступа к определенной группе ресурсов можно уменьшить область:
```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```
Команда должна вывести объект JSON:
```JSON
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    ...
}
```

В этом примере используется пример [пиггиметрикс](https://github.com/Azure-Samples/piggymetrics) на GitHub.  Разветвление образца, откройте страницу репозитория GitHub и щелкните вкладку **Параметры** . Откройте меню **секреты** и выберите команду **Добавить новый секрет**:

 ![Добавить новый секрет](./media/github-actions/actions1.png)

Задайте в качестве имени секрета `AZURE_CREDENTIALS` и его значение в строке JSON, которую вы нашли в заголовке, *настройте репозиторий GitHub и пройдите проверку подлинности*.

 ![Настройка секретных данных](./media/github-actions/actions2.png)

Вы также можете получить учетные данные для входа в Azure из Key Vault в действиях GitHub, как описано в статьях [Проверка подлинности Azure с помощью Key Vault в действиях GitHub](./spring-cloud-github-actions-key-vault.md).

## <a name="provision-service-instance"></a>Инициализация экземпляра службы
Для подготовки экземпляра облачной службы Azure весны выполните следующие команды с помощью Azure CLI.
```
az extension add --name spring-cloud
az group create --location eastus --name <resource group name>
az spring-cloud create -n <service instance name> -g <resource group name>
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/xxx/piggymetrics --label config
```
## <a name="build-the-workflow"></a>Создание рабочего процесса
Рабочий процесс определяется с помощью следующих параметров.

### <a name="prepare-for-deployment-with-azure-cli"></a>Подготовка к развертыванию с помощью Azure CLI
В `az spring-cloud app create` настоящее время команда не идемпотентными.  Этот рабочий процесс рекомендуется для существующих облачных приложений и экземпляров Azure весны.

Для подготовки используйте следующие Azure CLI команды:
```
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
az spring-cloud app create --name gateway
az spring-cloud app create --name auth-service
az spring-cloud app create --name account-service
```

### <a name="deploy-with-azure-cli-directly"></a>Прямое развертывание с помощью Azure CLI
Создайте `.github/workflow/main.yml` файл в репозитории:

```
name: AzureSpringCloud
on: push

env:
  GROUP: <resource group name>
  SERVICE_NAME: <service instance name>

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@main
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
      
    - name: Install ASC AZ extension
      run: az extension add --name spring-cloud
   
    - name: Deploy with AZ CLI commands
      run: |
        az configure --defaults group=$GROUP
        az configure --defaults spring-cloud=$SERVICE_NAME
        az spring-cloud app deploy -n gateway --jar-path ${{ github.workspace }}/gateway/target/gateway.jar
        az spring-cloud app deploy -n account-service --jar-path ${{ github.workspace }}/account-service/target/account-service.jar
        az spring-cloud app deploy -n auth-service --jar-path ${{ github.workspace }}/auth-service/target/auth-service.jar
```
### <a name="deploy-with-azure-cli-action"></a>Развертывание с действием Azure CLI
Команда AZ `run` будет использовать последнюю версию Azure CLI. При наличии критических изменений можно также использовать определенную версию Azure CLI с помощью Azure или CLI `action` . 

> [!Note] 
> Эта команда будет выполняться в новом контейнере, поэтому `env` не будет работать, а доступ к файлам для кросс-действий может иметь дополнительные ограничения.

Создайте файл. GitHub/Workflow/Main. yml в репозитории:
```
name: AzureSpringCloud
on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@main
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
        
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
              
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.75
        inlineScript: |
          az extension add --name spring-cloud
          az configure --defaults group=<service group name>
          az configure --defaults spring-cloud=<service instance name>
          az spring-cloud app deploy -n gateway --jar-path $GITHUB_WORKSPACE/gateway/target/gateway.jar
          az spring-cloud app deploy -n account-service --jar-path $GITHUB_WORKSPACE/account-service/target/account-service.jar
          az spring-cloud app deploy -n auth-service --jar-path $GITHUB_WORKSPACE/auth-service/target/auth-service.jar
```

## <a name="deploy-with-maven-plugin"></a>Развертывание с помощью подключаемого модуля Maven
Другой вариант — использовать [подключаемый модуль Maven](./spring-cloud-quickstart.md) для развертывания JAR-файла и обновления параметров приложения. Команда `mvn azure-spring-cloud:deploy` идемпотентными и автоматически создаст приложения, если это необходимо. Создавать соответствующие приложения заранее не требуется.

```
name: AzureSpringCloud
on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@main
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
        
    # Maven plugin can cosume this authentication method automatically
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Maven deploy, make sure you have correct configurations in your pom.xml
    - name: deploy to Azure Spring Cloud using Maven
      run: |
        mvn azure-spring-cloud:deploy
```

::: zone-end

## <a name="run-the-workflow"></a>Запуск рабочего процесса

**Действия** GitHub должны быть включены автоматически после отправки `.github/workflow/main.yml` в GitHub. Действие будет активировано при отправке новой фиксации. При создании этого файла в браузере действие должно быть уже запущено.

Чтобы убедиться, что действие включено, щелкните вкладку **действия** на странице репозитория GitHub:

![Проверка включения действия](./media/github-actions/actions3.png)

Если действие выполняется в случае ошибки, например, если вы не задали учетные данные Azure, вы можете повторно выполнить проверку после исправления ошибки. На странице репозиторий GitHub щелкните **действия**, выберите конкретную задачу рабочего процесса, а затем нажмите кнопку **Повторное** выполнение проверок, чтобы повторно запустить проверки:

![Повторить проверку](./media/github-actions/actions4.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Key Vault для действий в облаке GitHub](./spring-cloud-github-actions-key-vault.md)
* [Субъекты-службы Azure Active Directory](/cli/azure/ad/sp#az-ad-sp-create-for-rbac)
* [GitHub Actions для Azure](https://github.com/Azure/actions/)

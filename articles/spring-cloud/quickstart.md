---
title: Краткое руководство. Развертывание первого приложения Azure Spring Cloud
description: В этом кратком руководстве описывается развертывание приложения Spring Cloud в Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 10/23/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 6ae56e4e1586a875d4d3d75d41580f375df0eebf
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105962855"
---
# <a name="quickstart-deploy-your-first-azure-spring-cloud-application"></a>Краткое руководство. Развертывание первого приложения Azure Spring Cloud

::: zone pivot="programming-language-csharp"
В этом кратком руководстве объясняется, как развернуть простое приложение микрослужбы Azure Spring Cloud для запуска в Azure.

>[!NOTE]
> В настоящее время поддержка Steeltoe для Azure Spring Cloud предлагается в качестве общедоступной предварительной версии. Предложения общедоступной предварительной версии позволяют клиентам экспериментировать с новыми функциями до официального выпуска.  Общедоступные предварительные версии функций и служб не предназначены для использования в рабочей среде.  Чтобы получить дополнительные сведения о поддержке на этапе использования предварительных версий, ознакомьтесь с разделом [Вопросы и ответы](https://azure.microsoft.com/support/faq/) или оформите [запрос на поддержку](../azure-portal/supportability/how-to-create-azure-support-request.md).

Из этого краткого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * создание базового проекта Steeltoe .NET Core;
> * подготовка экземпляра службы Azure Spring Cloud;
> * сборка и развертывание приложения с общедоступной конечной точкой;
> * Потоковая передача журналов в режиме реального времени

Код приложения, используемый в этом кратком руководстве, представляет собой простое приложение, созданное с помощью шаблона проекта веб-API .NET Core. После завершения работы с этим примером приложение будет доступно через Интернет, и вы сможете управлять им с помощью портала Azure и Azure CLI.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* [Пакет SDK для .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1). Служба Azure Spring Cloud поддерживает .NET Core 3.1 и более поздние версии.
* [Azure CLI](/cli/azure/install-azure-cli) версии 2.0.67 или более поздней.
* [Git](https://git-scm.com/).

## <a name="install-azure-cli-extension"></a>Установка расширения Azure CLI

Убедитесь в наличии Azure CLI версии 2.0.67 или более поздней.

```azurecli
az --version
```

Установите расширение Azure Spring Cloud для Azure CLI с помощью следующей команды:

```azurecli
az extension add --name spring-cloud
```

## <a name="log-in-to-azure"></a>Вход в Azure

1. Войдите в систему Azure CLI.

    ```azurecli
    az login
    ```

1. Если у вас есть несколько подписок, выберите ту, которую будете использовать для изучения этого краткого руководства.

   ```azurecli
   az account list -o table
   ```

   ```azurecli
   az account set --subscription <Name or ID of a subscription from the last step>
   ```

## <a name="generate-a-steeltoe-net-core-project"></a>Создание проекта Steeltoe .NET Core

В Visual Studio создайте веб-приложение ASP.NET Core с именем hello-world с помощью шаблона проекта API. Обратите внимание, что в дальнейшем WeatherForecastController будет создаваться автоматически и станет нашей конечной точкой для тестирования.

1. Создайте папку для исходного кода проекта и сам проект.
 
   ```console
   mkdir source-code
   ```

   ```console
   cd source-code
   ```

   ```dotnetcli
   dotnet new webapi -n hello-world --framework netcoreapp3.1
   ```

1. Перейдите в каталог проекта.

   ```console
   cd hello-world
   ```

1. Измените файл *appSettings.json*, чтобы добавить следующие параметры:

   ```json
   "spring": {
     "application": {
       "name": "hello-world"
     }
   },
   "eureka": {
     "client": {
       "shouldFetchRegistry": true,
       "shouldRegisterWithEureka": true
     }
   }
   ```

1. Кроме того, в файле *appsettings.json* измените уровень ведения журнала для категории `Microsoft` с `Warning` на `Information`. Это изменение гарантирует, что журналы будут создаваться при просмотре журналов потоковой передачи на более позднем этапе.

   Файл *appsettings.json* теперь выглядит как в следующем примере:

   ```json
   {
     "Logging": {
       "LogLevel": {
         "Default": "Information",
         "Microsoft": "Information",
         "Microsoft.Hosting.Lifetime": "Information"
       }
     },
     "AllowedHosts": "*",
     "spring": {
       "application": {
         "name": "hello-world"
       }
     },
     "eureka": {
       "client": {
         "shouldFetchRegistry": true,
         "shouldRegisterWithEureka": true
       }
     }
   }
   ```
   
1. Добавьте зависимости и задачу `Zip` в файл *.csproj*:

   ```xml
   <ItemGroup>
     <PackageReference Include="Steeltoe.Discovery.ClientCore" Version="3.0.0" />
     <PackageReference Include="Microsoft.Azure.SpringCloud.Client" Version="2.0.0-preview.1" />
   </ItemGroup>
   <Target Name="Publish-Zip" AfterTargets="Publish">
       <ZipDirectory SourceDirectory="$(PublishDir)" DestinationFile="$(MSBuildProjectDirectory)/deploy.zip" Overwrite="true" />
   </Target>
   ```

   Пакеты предназначены для обнаружения службы Steeltoe и клиентской библиотеки Azure Spring Cloud. Задача `Zip` предназначена для развертывания в Azure. При выполнении команды `dotnet publish` создаются двоичные файлы в папке *publish*, и эта задача архивирует папку *publish* в файл *.zip*, который вы отправляете в Azure.

3. В файле *Program.cs* добавьте директиву `using` и код, который использует клиентскую библиотеку Azure Spring Cloud:

   ```csharp
   using Microsoft.Azure.SpringCloud.Client;
   ```

   ```csharp
   public static IHostBuilder CreateHostBuilder(string[] args) =>
               Host.CreateDefaultBuilder(args)
                   .UseAzureSpringCloudService()
                   .ConfigureWebHostDefaults(webBuilder =>
                   {
                       webBuilder.UseStartup<Startup>();
                   });
   ```

4. В файле *Startup.cs* добавьте директиву `using` и код, который использует обнаружение службы Steeltoe, в конце методов `ConfigureServices` и `Configure`:

   ```csharp
   using Steeltoe.Discovery.Client;
   ```

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       // Template code not shown.

       services.AddDiscoveryClient(Configuration);
   }
   ```

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
       // Template code not shown.

       app.UseDiscoveryClient();
   }
   ```

1. Выполните сборку проекта и убедитесь в отсутствии ошибок компиляции.

   ```dotnetcli
   dotnet build
   ```
 
## <a name="provision-a-service-instance"></a>подготовка экземпляра службы к работе;

Следующая процедура создает экземпляр Azure Spring Cloud с помощью портала Azure.

1. Откройте [портал Azure](https://ms.portal.azure.com/). 

1. В поле поиска сверху введите *Azure Spring Cloud*.

1. Выберите пункт *Azure Spring Cloud* в списке результатов.

   ![Значок ASC — запуск](media/spring-cloud-quickstart-launch-app-portal/find-spring-cloud-start.png)

1. На странице Azure Spring Cloud выберите элемент **+ Добавить**.

   ![Значок ASC — добавление](media/spring-cloud-quickstart-launch-app-portal/spring-cloud-add.png)

1. Заполните форму на странице **создания** Azure Spring Cloud.  Ознакомьтесь со следующими рекомендациями:

   * **Подписка**: Выберите подписку, на которую будет выставляться счет за этот ресурс.
   * **Группа ресурсов.** Создайте новую группу ресурсов. Введенное здесь имя будет использоваться в дальнейших шагах как **\<resource group name\>** .
   * **Сведения о службе или ее название.** Укажите **\<service instance name\>** .  Его длина должна быть от 4 до 32 знаков. Имя может содержать только строчные буквы, цифры и дефисы.  Первым символом в имени службы должна быть буква, а последним — буква или цифра.
   * **Регион**. Выберите регион для экземпляра службы.

   ![Начальная страница портала ASC](media/spring-cloud-quickstart-launch-app-portal/portal-start.png)

6. Выберите **Просмотреть и создать**.

## <a name="build-and-deploy-the-app"></a>Создание и развертывание приложения

Следующая процедура используется для сборки и развертывания созданного ранее проекта.

1. Убедитесь, что командная строка все еще находится в папке проекта.

1. Запустите приведенную ниже команду, чтобы выполнить сборку проекта, опубликовать двоичные файлы и сохранить их в файле *.zip* в папке проекта.

   ```dotnetcorecli
   dotnet publish -c release -o ./publish
   ```

1. Создайте приложение в экземпляре Azure Spring Cloud с назначенной общедоступной конечной точкой. Используйте то же имя приложения hello-world, указанное в *appsettings.json*.

   ```console
   az spring-cloud app create -n hello-world -s <service instance name> -g <resource group name> --assign-endpoint --runtime-version NetCore_31
   ```

1. Разверните в приложении файл *.zip*.

   ```azurecli
   az spring-cloud app deploy -n hello-world -s <service instance name> -g <resource group name> --runtime-version NetCore_31 --main-entry hello-world.dll --artifact-path ./deploy.zip
   ```

   Параметр `--main-entry` определяет файл *.dll*, содержащий точку входа приложения. После того как служба загрузит файл *.zip*, она извлекает все файлы и папки и пытается выполнить точку входа в указанном `--main-entry` файле *.dll*.

   Для завершения развертывания приложения потребуется несколько минут. Чтобы убедиться, что оно развернуто, перейдите в колонку **Приложения** на портале Azure.

## <a name="test-the-app"></a>Тестирование приложения

По завершении развертывания получите доступ к приложению по следующему URL-адресу:

```http
https://<service instance name>-hello-world.azuremicroservices.io/weatherforecast
```

Приложение вернет данные JSON, как в следующем примере:

```json
[{"date":"2020-09-08T21:01:50.0198835+00:00","temperatureC":14,"temperatureF":57,"summary":"Bracing"},{"date":"2020-09-09T21:01:50.0200697+00:00","temperatureC":-14,"temperatureF":7,"summary":"Bracing"},{"date":"2020-09-10T21:01:50.0200715+00:00","temperatureC":27,"temperatureF":80,"summary":"Freezing"},{"date":"2020-09-11T21:01:50.0200717+00:00","temperatureC":18,"temperatureF":64,"summary":"Chilly"},{"date":"2020-09-12T21:01:50.0200719+00:00","temperatureC":16,"temperatureF":60,"summary":"Chilly"}]
```

## <a name="stream-logs-in-real-time"></a>Потоковая передача журналов в режиме реального времени

Используйте следующую команду, чтобы получать журналы в режиме реального времени из приложения.

```azurecli
az spring-cloud app logs -n hello-world -s <service instance name> -g <resource group name> --lines 100 -f
```

Журналы отображаются в выходных данных:

```output
[Azure Spring Cloud] The following environment variables are loaded:
2020-09-08 20:58:42,432 INFO supervisord started with pid 1
2020-09-08 20:58:43,435 INFO spawned: 'event-gather_00' with pid 9
2020-09-08 20:58:43,436 INFO spawned: 'dotnet-app_00' with pid 10
2020-09-08 20:58:43 [Warning] No managed processes are running. Wait for 30 seconds...
2020-09-08 20:58:44,843 INFO success: event-gather_00 entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2020-09-08 20:58:44,843 INFO success: dotnet-app_00 entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
←[40m←[32minfo←[39m←[22m←[49m: Steeltoe.Discovery.Eureka.DiscoveryClient[0]
      Starting HeartBeat
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://[::]:1025
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /netcorepublish/6e4db42a-b160-4b83-a771-c91adec18c60
2020-09-08 21:00:13 [Information] [10] Start listening...
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET http://asc-svc-hello-world.azuremicroservices.io/weatherforecast
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'hello_world.Controllers.WeatherForecastController.Get (hello-world)'
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[3]
      Route matched with {action = "Get", controller = "WeatherForecast"}. Executing controller action with signature System.Collections.Generic.IEnumerable`1[hello_world.WeatherForecast] Get() on controller hello_world.Controllers.WeatherForecastController (hello-world).
info: Microsoft.AspNetCore.Mvc.Infrastructure.ObjectResultExecutor[1]
      Executing ObjectResult, writing value of type 'hello_world.WeatherForecast[]'.
info: Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker[2]
      Executed action hello_world.Controllers.WeatherForecastController.Get (hello-world) in 1.8902ms
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'hello_world.Controllers.WeatherForecastController.Get (hello-world)'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 4.2591ms 200 application/json; charset=utf-8
```

> [!TIP]
> Используйте `az spring-cloud app logs -h` для просмотра дополнительных параметров и функциональных возможностей потоковой передачи журналов.

Чтобы получить дополнительные возможности анализа журналов, перейдите на вкладку **Журналы** в меню на [портале Azure](https://portal.azure.com/). Журналы отображаются с задержкой в несколько минут.
[ ![Аналитика журналов](media/spring-cloud-quickstart-java/logs-analytics.png) ](media/spring-cloud-quickstart-java/logs-analytics.png#lightbox)
::: zone-end

::: zone pivot="programming-language-java"
В этом кратком руководстве объясняется, как развернуть простое приложение микрослужбы Azure Spring Cloud для запуска в Azure. 

Код приложения, используемый в этом учебнике, — это простое приложение, созданное с помощью Spring Initializr. После завершения работы с этим примером приложение будет доступно через Интернет и им можно управлять с помощью портала Azure.

В этом кратком руководстве объясняются такие операции:

> [!div class="checklist"]
> * создание простого проекта Spring Cloud;
> * подготовка экземпляра службы к работе;
> * сборка и развертывание приложения с общедоступной конечной точкой;
> * Потоковая передача журналов в режиме реального времени

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством сделайте следующее:

* [установите JDK версии 8](/java/azure/jdk/);
* [Регистрация для получения подписки Azure](https://azure.microsoft.com/free/)
* (необязательно) [установите Azure CLI версии 2.0.67 или более поздней](/cli/azure/install-azure-cli) и расширение Azure Spring Cloud с помощью команды `az extension add --name spring-cloud`;
* (необязательно) [установите Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) и [выполните вход](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

## <a name="generate-a-spring-cloud-project"></a>Создание проекта Spring Cloud

Начните со [Spring Initializr](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.9.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin,cloud-config-client), чтобы создать пример проекта с рекомендуемыми зависимостями для Azure Spring Cloud. На рисунке ниже показан Initializr, настроенный для этого примера проекта.
```url
https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.4.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin,cloud-config-client
```
Обратите внимание, что в этом примере используется версия Java 8.  Если вы хотите использовать версию Java 11, измените параметр в разделе **Метаданные проекта**.

  ![Страница Initializr](media/spring-cloud-quickstart-java/initializr-page.png)

1. Щелкните **Создать**, если все зависимости заданы. Загрузите и распакуйте пакет, а затем создайте веб-контроллер для простого веб-приложения, добавив `src/main/java/com/example/hellospring/HelloController.java` следующим образом:

    ```java
    package com.example.hellospring;
    
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    @RestController
    public class HelloController {
    
        @RequestMapping("/")
        public String index() {
            return "Greetings from Azure Spring Cloud!";
        }
    
    }
    ```
## <a name="provision-an-instance-of-azure-spring-cloud"></a>Подготовка к работе экземпляра Azure Spring Cloud

Следующая процедура создает экземпляр Azure Spring Cloud с помощью портала Azure.

1. На новой вкладке откройте [портал Azure](https://ms.portal.azure.com/). 

2. В поле поиска сверху введите *Azure Spring Cloud*.

3. Выберите пункт *Azure Spring Cloud* в списке результатов.

    ![Значок ASC — запуск](media/spring-cloud-quickstart-launch-app-portal/find-spring-cloud-start.png)

4. На странице Azure Spring Cloud щелкните элемент **+ Добавить**.

    ![Значок ASC — добавление](media/spring-cloud-quickstart-launch-app-portal/spring-cloud-add.png)

5. Заполните форму на странице **создания** Azure Spring Cloud.  Ознакомьтесь со следующими рекомендациями:
    - **Подписка**: Выберите подписку, на которую будет выставляться счет за этот ресурс.
    - **Группа ресурсов.** Советуем создавать группы ресурсов для новых ресурсов. Она будет использоваться в дальнейших шагах как **\<resource group name\>** .
    - **Сведения о службе или ее название.** Укажите **\<service instance name\>** .  Его длина должна быть от 4 до 32 знаков. Имя может содержать только строчные буквы, цифры и дефисы.  Первым символом в имени службы должна быть буква, а последним — буква или цифра.
    - **Расположение.** Выберите регион для экземпляра службы.

    ![Начальная страница портала ASC](media/spring-cloud-quickstart-launch-app-portal/portal-start.png)

6. Щелкните **Проверка и создание**.

## <a name="build-and-deploy-the-app"></a>Создание и развертывание приложения
    
#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
Следующая процедура используется для сборки и развертывания приложения с помощью Azure CLI. Выполните следующую команду в корневом каталоге проекта.

1. Выполните сборку проекта с помощью Maven:

    ```console
    mvn clean package -DskipTests
    ```

1. Если вы еще не установили его, установите облачное расширение Spring Cloud для Azure CLI:

    ```azurecli
    az extension add --name spring-cloud
    ```
    
1. Создайте приложение с назначенной общедоступной конечной точкой. Если вы используете Java 11, добавьте переключатель `--runtime-version=Java_11`.

    ```azurecli
    az spring-cloud app create -n hellospring -s <service instance name> -g <resource group name> --assign-endpoint true
    ```

1. Разверните JAR-файл для приложения (`target\hellospring-0.0.1-SNAPSHOT.jar` в Windows):

    ```azurecli
    az spring-cloud app deploy -n hellospring -s <service instance name> -g <resource group name> --jar-path <jar file path>
    ```
    
1. Для завершения развертывания приложения потребуется несколько минут. Чтобы убедиться, что оно развернуто, перейдите в колонку **Приложения** на портале Azure. В ней должно отобразиться состояние приложения.

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

Следующая процедура использует подключаемый модуль IntelliJ для Azure Spring Cloud, чтобы развернуть пример приложения в IntelliJ IDEA.  

### <a name="import-project"></a>Импорт проекта

1. Откройте **диалоговое окно приветствия** IntelliJ и выберите **Import Project** (Импорт проекта), чтобы открыть мастер импорта.
1. Выберите папку `hellospring`.

    ![Импорт проекта](media/spring-cloud-quickstart-java/intellij-new-project.png)

### <a name="deploy-the-app"></a>Развертывание приложения
Чтобы выполнить развертывание в Azure, необходимо войти в учетную запись Azure и выбрать подписку.  Дополнительные сведения см. в статье [Установка и вход](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

1. Щелкните правой кнопкой мыши проект в обозревателе проектов IntelliJ и выберите **Azure** -> **Deploy to Azure Spring Cloud** (Развернуть в Azure Spring Cloud).

    [ ![Развертывание в Azure, шаг 1](media/spring-cloud-quickstart-java/intellij-deploy-azure-1.png) ](media/spring-cloud-quickstart-java/intellij-deploy-azure-1.png#lightbox)

1. Подтвердите имя для приложения, указанное в поле **Name** (Имя). **Имя** здесь обозначает конфигурацию, а не имя приложения. Обычно пользователям его не нужно изменять.
1. В текстовом поле **Artifact** (Артефакт) выберите *hellospring-0.0.1-SNAPSHOT.jar*.
1. В текстовом поле **Subscription** (Подписка) проверьте свою подписку.
1. В текстовом поле **Spring Cloud** выберите экземпляр Azure Spring Cloud, который вы создали при [подготовке к работе экземпляра Azure Spring Cloud](./spring-cloud-quickstart-provision-service-instance.md).
1. Задайте для параметра **Public Endpoint** (Общедоступная конечная точка) значение *Enable* (Включить).
1. В текстовом поле **App:** (Приложение:) выберите **Create app...** (Создать приложение...).
1. Введите *hellospring*, а затем нажмите кнопку **ОК**.

    [ ![Развертывание в Azure, подтверждение](media/spring-cloud-quickstart-java/intellij-deploy-to-azure.png) ](media/spring-cloud-quickstart-java/intellij-deploy-to-azure.png#lightbox)

1. Запустите развертывание, нажав кнопку **Run** (Выполнить) в нижней части диалогового окна **Deploy Azure Spring Cloud app** (Развертывание приложения Azure Spring Cloud). Подключаемый модуль выполнит команду `mvn package` для приложения `hellospring` и развернет JAR-файл, созданный командой `package`.
---

По завершении развертывания можно получить доступ к `https://<service instance name>-hellospring.azuremicroservices.io/`.

  [ ![Доступ к приложению из браузера](media/spring-cloud-quickstart-java/access-app-browser.png) ](media/spring-cloud-quickstart-java/access-app-browser.png#lightbox)

## <a name="streaming-logs-in-real-time"></a>Потоковая передача журналов в режиме реального времени

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

Используйте следующую команду, чтобы получать журналы в режиме реального времени из приложения.

```azurecli
az spring-cloud app logs -n hellospring -s <service instance name> -g <resource group name> --lines 100 -f

```
Внешний вид журналов в итоге:

[ ![Журналы потоковой передачи](media/spring-cloud-quickstart-java/streaming-logs.png) ](media/spring-cloud-quickstart-java/streaming-logs.png#lightbox)

>[!TIP]
> Используйте `az spring-cloud app logs -h` для просмотра дополнительных параметров и функциональных возможностей потоковой передачи журналов.

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

1. Выберите **Azure Explorer** и **Spring Cloud**.
1. Щелкните правой кнопкой мыши запущенное приложение.
1. Выберите **Streaming Logs** (Журналы потоковой передачи) из раскрывающегося списка.
1. Выберите экземпляр.

    [ ![Выбор журналов потоковой передачи](media/spring-cloud-quickstart-java/intellij-get-streaming-logs.png) ](media/spring-cloud-quickstart-java/intellij-get-streaming-logs.png)

1. Журнал потоковой передачи отобразится в окне выходных данных.

    [ ![Вывод журнала потоковой передачи](media/spring-cloud-quickstart-java/intellij-streaming-logs-output.png) ](media/spring-cloud-quickstart-java/intellij-streaming-logs-output.png)
---

Чтобы получить дополнительные возможности аналитики журналов, перейдите на вкладку **Журналы** в меню [портал Azure](https://portal.azure.com/). Журналы отображаются с задержкой в несколько минут.

[ ![Аналитика журналов](media/spring-cloud-quickstart-java/logs-analytics.png) ](media/spring-cloud-quickstart-java/logs-analytics.png#lightbox)
::: zone-end

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы создали ресурсы Azure, за которые будет взиматься плата, если вы оставите их в своей в подписке. Если эти ресурсы вам не понадобятся в будущем, вы можете удалить группу ресурсов на портале или с помощью приведенной ниже команды в Azure CLI:

```azurecli
az group delete --name <your resource group name; for example: hellospring-1558400876966-rg> --yes
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве рассматривались следующие темы:

> [!div class="checklist"]
> * создание базового проекта Azure Spring Cloud;
> * подготовка экземпляра службы к работе;
> * сборка и развертывание приложения с общедоступной конечной точкой;
> * Потоковая передача журналов в режиме реального времени

Чтобы узнать, как использовать дополнительные возможности Azure Spring, перейдите к серии кратких руководств, в которой описано развертывание примера приложения в Azure Spring Cloud:

> [!div class="nextstepaction"]
> [Создание и запуск микрослужб](spring-cloud-quickstart-sample-app-introduction.md)

Дополнительные примеры доступны на GitHub: [Примеры для Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples).

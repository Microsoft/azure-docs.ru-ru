---
title: Руководство по Сквозная проверка подлинности пользователей
description: Узнайте, как использовать проверку подлинности и авторизацию в Службе приложений для комплексной защиты приложений Службы приложений, включая доступ к удаленным API.
keywords: app service, azure app service, authN, authZ, secure, security, multi-tiered, azure active directory, azure ad
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/29/2020
ms.custom: devx-track-csharp, seodec18, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: e35a4abbcaf0cd2298d55c06de0ecfbef0dff354
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765815"
---
# <a name="tutorial-authenticate-and-authorize-users-end-to-end-in-azure-app-service"></a>Руководство по Сквозная проверка подлинности и авторизации в Службе приложений Azure

::: zone pivot="platform-windows"  

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. Кроме того, служба приложений имеет встроенную поддержку [проверки подлинности и авторизации пользователя](overview-authentication-authorization.md). В этом руководстве показано, как защитить ваши приложения с помощью проверки подлинности и авторизации в службе приложений. В качестве примера в нем используется приложение ASP.NET Core с интерфейсом Angular.js. При проверке подлинности и авторизации в службе приложений поддерживаются все языковые среды выполнения. Вы узнаете, как применить их к предпочитаемому языку, следуя руководству.

::: zone-end

::: zone pivot="platform-linux"

[Служба приложений Azure](overview.md) — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Linux. Кроме того, служба приложений имеет встроенную поддержку [проверки подлинности и авторизации пользователя](overview-authentication-authorization.md). В этом руководстве показано, как защитить ваши приложения с помощью проверки подлинности и авторизации в службе приложений. В качестве примера в нем используется приложение ASP.NET Core с интерфейсом Angular.js. При проверке подлинности и авторизации в службе приложений поддерживаются все языковые среды выполнения. Вы узнаете, как применить их к предпочитаемому языку, следуя руководству.

::: zone-end

![Простая проверка подлинности и авторизация](./media/tutorial-auth-aad/simple-auth.png)

В руководстве также показано, как защитить многоуровневое приложение, получив доступ к защищенному серверному API от имени прошедшего проверку подлинности пользователя, как [в коде сервера](#call-api-securely-from-server-code), так и [в коде браузера](#call-api-securely-from-browser-code).

![Расширенная проверка подлинности и авторизация](./media/tutorial-auth-aad/advanced-auth.png)

Это только некоторые из возможных сценариев проверки подлинности и авторизации в службе приложений. 

Ниже приведен полный список сведений, которые вы узнаете из этого руководства:

> [!div class="checklist"]
> * Включение встроенной проверки подлинности и авторизации.
> * Защита приложений от непроверенных запросов.
> * Использование Azure Active Directory в качестве поставщика удостоверений.
> * Доступ к удаленному приложению от имени выполнившего вход пользователя.
> * Безопасные вызовы между службами с помощью проверки подлинности с использованием токена.
> * Использование маркера доступа из серверного кода.
> * Использование маркера доступа из клиентского кода (браузер).

Шаги, описанные в этом руководстве, можно выполнить для macOS, Linux и Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

- <a href="https://git-scm.com/" target="_blank">установите Git</a>;
- <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Установите последний пакет SDK для .NET Core 3.1.</a>
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-local-net-core-app"></a>Создание локального приложения .NET Core

На этом шаге вы настроите локальный проект .NET Core. Используйте один и тот же проект для развертывания серверного приложения API и интерфейсного веб-приложения.

### <a name="clone-and-run-the-sample-application"></a>Клонирование и запуск примера приложения

Затем выполните следующие команды, чтобы клонировать репозиторий с примером и запустить его.

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
cd dotnet-core-api
dotnet run
```

Перейдите к `http://localhost:5000` и попробуйте добавить, изменить и удалить объекты списка дел. 

![Выполняющийся локально API ASP.NET Core](./media/tutorial-auth-aad/local-run.png)

Чтобы остановить ASP.NET Core в любое время, нажмите клавиши `Ctrl+C` в окне терминала.

## <a name="deploy-apps-to-azure"></a>Развертывание приложений в Azure

На этом шаге разверните проект в двух приложениях службы приложений. Одно из которых интерфейсное приложение, а второе — серверное.

### <a name="configure-a-deployment-user"></a>Настойка пользователя развертывания

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-azure-resources"></a>Создание ресурсов Azure

::: zone pivot="platform-windows"  

В Cloud Shell выполните следующие команды, чтобы создать два веб-приложения Windows: Замените _\<front-end-app-name>_ и _\<back-end-app-name>`-` двумя глобально уникальными именами приложений (допустимые символы:_ , `a-z` и `0-9`). Дополнительные сведения о каждой команде см. в статье [Размещение API-интерфейсов RESTful с поддержкой CORS в службе приложений Azure](app-service-web-tutorial-rest-api.md).

```azurecli-interactive
az group create --name myAuthResourceGroup --location "West Europe"
az appservice plan create --name myAuthAppServicePlan --resource-group myAuthResourceGroup --sku FREE
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <front-end-app-name> --deployment-local-git --query deploymentLocalGitUrl
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <back-end-app-name> --deployment-local-git --query deploymentLocalGitUrl
```

::: zone-end

::: zone pivot="platform-linux"

В Cloud Shell введите следующие команды, чтобы создать два веб-приложения: Замените _\<front-end-app-name>_ и _\<back-end-app-name>`-` двумя глобально уникальными именами приложений (допустимые символы:_ , `a-z` и `0-9`). Дополнительные сведения о каждой команде см. в статье [Создание приложения .NET Core в Службе приложений Azure](quickstart-dotnetcore.md).

```azurecli-interactive
az group create --name myAuthResourceGroup --location "West Europe"
az appservice plan create --name myAuthAppServicePlan --resource-group myAuthResourceGroup --sku FREE --is-linux
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <front-end-app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git --query deploymentLocalGitUrl
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <back-end-app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git --query deploymentLocalGitUrl
```

::: zone-end

> [!NOTE]
> Сохраните URL-адреса удаленных элементов Git вашего интерфейсного и серверного приложения, которые отображаются в выходных данных `az webapp create`.
>

### <a name="push-to-azure-from-git"></a>Публикация в Azure из Git

В _окне терминала на локальном компьютере_ выполните следующие команды Git, чтобы выполнить развертывание в серверное приложение. Замените _\<deploymentLocalGitUrl-of-back-end-app>_ URL-адресом удаленного репозитория Git, который вы сохранили на этапе [создания ресурсов Azure](#create-azure-resources). При появлении запроса на ввод учетных данных в диспетчере учетных данных Git введите [учетные данные развертывания](deploy-configure-credentials.md) (а не используемые для входа на портал Azure).

```bash
git remote add backend <deploymentLocalGitUrl-of-back-end-app>
git push backend master
```

В окне терминала на локальном компьютере выполните следующие команды Git, чтобы выполнить развертывание аналогичного кода в интерфейсное приложение. Замените _\<deploymentLocalGitUrl-of-front-end-app>_ URL-адресом удаленного репозитория Git, который вы сохранили на этапе [создания ресурсов Azure](#create-azure-resources).

```bash
git remote add frontend <deploymentLocalGitUrl-of-front-end-app>
git push frontend master
```

### <a name="browse-to-the-apps"></a>Переход к приложениям

Перейдите по следующим URL-адресам в браузере и посмотрите, как работают два приложения.

```
http://<back-end-app-name>.azurewebsites.net
http://<front-end-app-name>.azurewebsites.net
```

:::image type="content" source="./media/tutorial-auth-aad/azure-run.png" alt-text="Снимок экрана с примером REST API Службы приложений Azure в окне браузера, в котором отображается приложение списка дел.":::

> [!NOTE]
> Если приложение перезапускается, возможно, вы заметили, что новые данные были удалены. Это сделано намеренно, так как пример приложения ASP.NET Core использует базу данных в памяти.
>
>

## <a name="call-back-end-api-from-front-end"></a>Вызов серверного API из внешнего интерфейса

На этом шаге укажите код сервера интерфейсного приложения для доступа к серверному API. Позже вы включите доступ с проверкой подлинности от интерфейсной части к серверной.

### <a name="modify-front-end-code"></a>Изменение интерфейсного кода

В локальном репозитории откройте _Controllers/TodoController.cs_. В начале класса `TodoController` добавьте следующие строки и замените _\<back-end-app-name>_ именем вашего внутреннего приложения.

```cs
private static readonly HttpClient _client = new HttpClient();
private static readonly string _remoteUrl = "https://<back-end-app-name>.azurewebsites.net";
```

Найдите метод, помеченный `[HttpGet]`, и замените код внутри фигурных скобок следующим:

```cs
var data = await _client.GetStringAsync($"{_remoteUrl}/api/Todo");
return JsonConvert.DeserializeObject<List<TodoItem>>(data);
```

Первая строка осуществляет вызов `GET /api/Todo` к приложению серверного API.

Затем найдите метод, помеченный `[HttpGet("{id}")]`, и замените код внутри фигурных скобок следующим:

```cs
var data = await _client.GetStringAsync($"{_remoteUrl}/api/Todo/{id}");
return Content(data, "application/json");
```

Первая строка осуществляет вызов `GET /api/Todo/{id}` к приложению серверного API.

Затем найдите метод, помеченный `[HttpPost]`, и замените код внутри фигурных скобок следующим:

```cs
var response = await _client.PostAsJsonAsync($"{_remoteUrl}/api/Todo", todoItem);
var data = await response.Content.ReadAsStringAsync();
return Content(data, "application/json");
```

Первая строка осуществляет вызов `POST /api/Todo` к приложению серверного API.

Затем найдите метод, помеченный `[HttpPut("{id}")]`, и замените код внутри фигурных скобок следующим:

```cs
var res = await _client.PutAsJsonAsync($"{_remoteUrl}/api/Todo/{id}", todoItem);
return new NoContentResult();
```

Первая строка осуществляет вызов `PUT /api/Todo/{id}` к приложению серверного API.

Затем найдите метод, помеченный `[HttpDelete("{id}")]`, и замените код внутри фигурных скобок следующим:

```cs
var res = await _client.DeleteAsync($"{_remoteUrl}/api/Todo/{id}");
return new NoContentResult();
```

Первая строка осуществляет вызов `DELETE /api/Todo/{id}` к приложению серверного API.

Сохраните изменения. В окне терминала на локальном компьютере выполните следующие команды Git, чтобы выполнить развертывание аналогичного кода в интерфейсное приложение.

```bash
git add .
git commit -m "call back-end API"
git push frontend master
```

### <a name="check-your-changes"></a>Проверка изменений

Перейдите к `http://<front-end-app-name>.azurewebsites.net` и добавьте несколько элементов, например `from front end 1` и `from front end 2`.

Перейдите к `http://<back-end-app-name>.azurewebsites.net`, чтобы просмотреть элементы, добавленные из интерфейсного приложения. Кроме того, добавьте несколько элементов, например `from back end 1` и `from back end 2`, а затем обновите интерфейсное приложение, чтобы проверить изменения.

:::image type="content" source="./media/tutorial-auth-aad/remote-api-call-run.png" alt-text="Снимок экрана с примером REST API Службы приложений Azure в окне браузера, в котором отображается приложение списка дел с элементами, добавленными из интерфейсного приложения.":::

## <a name="configure-auth"></a>Настройка проверки подлинности

На этом шаге включите проверку подлинности и авторизацию для двух приложений. Кроме того, настройте создание маркера доступа в интерфейсном приложении, который вы можете использовать для осуществления прошедших проверку подлинности вызовов серверного приложения.

Azure Active Directory используется в качестве поставщика удостоверений. Дополнительные сведения см. в статье [Настройка приложения службы приложений для использования входа с помощью Azure Active Directory](configure-authentication-provider-aad.md).

### <a name="enable-authentication-and-authorization-for-back-end-app"></a>Включение проверки подлинности и авторизации в серверном приложении

В меню [портала Azure](https://portal.azure.com) выберите **Группы ресурсов** или выполните поиск по запросу *Группы ресурсов* на любой странице и выберите этот пункт.

В разделе **Группы ресурсов** найдите и выберите свою группу ресурсов. В разделе **Обзор** выберите страницу управления внутреннего приложения.

:::image type="content" source="./media/tutorial-auth-aad/portal-navigate-back-end.png" alt-text="Снимок экрана с окном группы ресурсов, в котором отображаются общие сведения о примере группы ресурсов и выбранная страница управления внутренним приложением.":::

В левом меню вашего внутреннего приложения выберите **Проверка подлинности или авторизация**, а затем включите проверку подлинности Службы приложений, выбрав **Включено**.

В раскрывающемся списке **Предпринимаемое действие, если проверка подлинности для запроса не выполнена** выберите **Войти с использованием Azure Active Directory**.

В разделе **Поставщики проверки подлинности** выберите **Azure Active Directory**.

:::image type="content" source="./media/tutorial-auth-aad/configure-auth-back-end.png" alt-text="Снимок экрана левого меню внутреннего приложения, в котором отображаются выбранные параметры проверки подлинности/авторизации, а также настройки, выбранные в меню справа.":::

Выберите **Экспресс**, затем примите настройки по умолчанию, чтобы создать приложение AD, и нажмите кнопку **ОК**.

На странице **Проверка подлинности или авторизация** нажмите кнопку **Сохранить**.

После того как вы увидите уведомление с сообщением `Successfully saved the Auth Settings for <back-end-app-name> App`, обновите страницу портала.

Щелкните **Azure Active Directory** еще раз, а затем выберите **Приложение Azure AD**.

Скопируйте **Идентификатор клиента** приложения Azure в Блокнот. Это значение понадобится позже.

:::image type="content" source="./media/tutorial-auth-aad/get-application-id-back-end.png" alt-text="Снимок экрана с окном параметров Azure Active Directory, в котором отображается Azure AD App, а также с окном &quot;Приложения Azure AD&quot;, в котором отображается идентификатор клиента для копирования.":::

На этот момент у вас уже есть автономное приложение, которое защищено функциями аутентификации и авторизации в службе приложений. В остальных разделах показано, как защитить решение с несколькими приложениями, "проводя" данные аутентифицированного пользователя от внешнего интерфейса к серверной части. 

### <a name="enable-authentication-and-authorization-for-front-end-app"></a>Включение проверки подлинности и авторизации в интерфейсном приложении

Выполните те же действия для внешнего приложения, но пропустите последний шаг. Для интерфейсного приложения идентификатор клиента не требуется.

При необходимости перейдите по ссылке `http://<front-end-app-name>.azurewebsites.net`. Теперь она должна направить вас на защищенную страницу входа. После входа *вы по-прежнему не можете получить доступ к данным из внутреннего приложения*, так как внутреннее приложение теперь требует входа в Azure Active Directory из интерфейсного приложения. Необходимо выполнить три действия:

- Предоставить доступ из интерфейсной части к серверной.
- Настроить службу приложений, чтобы вернуть доступный токен.
- Использовать токен в коде.

> [!TIP]
> Если вы столкнулись с ошибками и перенастроили параметры проверки подлинности или авторизации в своем приложении, токены в хранилище токенов не могут быть восстановлены из новых параметров. Чтобы убедиться, что ваши токены восстановлены, вам нужно выйти из приложения и войти в него. Для этого проще всего использовать браузер в режиме защищенного просмотра, затем закрыть и снова открыть браузер в режиме защищенного просмотра после изменения параметров в своих приложениях.

### <a name="grant-front-end-app-access-to-back-end"></a>Предоставление интерфейсному приложению доступа к серверной части

Теперь, когда вы включили проверку подлинности и авторизацию для обоих ваших приложений, каждое из них поддерживается приложением AD. На этом шаге предоставьте разрешения интерфейсному приложению для доступа к серверной части от имени пользователя. (Технически вы предоставляете интерфейсному _приложению AD_ разрешения на доступ к серверному _приложению AD_ от имени пользователя).

В меню [портала Azure](https://portal.azure.com) выберите **Azure Active Directory** или выполните поиск по запросу *Azure Active Directory* на любой странице и выберите этот пункт.

Поочередно выберите **Регистрация приложений** > **Собственные приложения** > **View all applications in this directory** (Просмотреть все приложения в этом каталоге). Щелкните имя внешнего приложения, а затем выберите **Разрешения API**.

:::image type="content" source="./media/tutorial-auth-aad/add-api-access-front-end.png" alt-text="Снимок экрана с окном регистраций приложений Microsoft, в котором выбраны собственные приложения, имя интерфейсного приложения и разрешения API.":::

Выберите **Добавить разрешение**, а затем — **Интерфейсы API, используемые моей организацией** >  **\<back-end-app-name>** .

На странице **Разрешения API запросов** серверного приложения выберите **Делегированные разрешения** и **user_impersonation**, а затем — **Добавить разрешения**.

:::image type="content" source="./media/tutorial-auth-aad/select-permission-front-end.png" alt-text="Снимок экрана со страницей запроса разрешений API, где выбраны элементы &quot;Делегированные разрешения&quot;, user_impersonation и кнопка &quot;Добавить разрешение&quot;.":::

### <a name="configure-app-service-to-return-a-usable-access-token"></a>Настройка службы приложений для возвращения используемых маркеров доступа

Теперь интерфейсное приложение имеет необходимые разрешения для доступа к серверному приложению в качестве вошедшего в систему пользователя. На этом шаге настройте проверку подлинности и авторизацию в службе приложений, чтобы получить доступный маркер доступа для доступа к серверной части. Для этого шага вам понадобится идентификатор клиента серверного приложения, который вы скопировали на шаге [Включение проверки подлинности и авторизации в серверном приложении](#enable-authentication-and-authorization-for-back-end-app).

Перейдите к [обозревателю ресурсов Azure](https://resources.azure.com) и с помощью дерева ресурсов найдите интерфейсное веб-приложение.

Теперь [обозреватель ресурсов Azure](https://resources.azure.com) открывается с помощью интерфейсного приложения, выбранного в дереве ресурсов. В верхней части страницы щелкните **Чтение и запись**, чтобы внести изменения в обозревателе ресурсов Azure.

:::image type="content" source="./media/tutorial-auth-aad/resources-enable-write.png" alt-text="Снимок экрана с кнопками &quot;Только для чтения&quot; и &quot;Чтение и запись&quot; в верхней части страницы обозревателя ресурсов Azure и выбранной кнопкой &quot;Чтение и запись&quot;.":::

В браузере слева перейдите к разделу **config** > **authsettings**.

В представлении **authsettings** щелкните **Изменить**. Установите значение `additionalLoginParams` в следующей строке JSON, используя идентификатор клиента, который вы скопировали. 

```json
"additionalLoginParams": ["response_type=code id_token","resource=<back-end-client-id>"],
```

:::image type="content" source="./media/tutorial-auth-aad/additional-login-params-front-end.png" alt-text="Снимок экрана с примером кода в представлении authsettings, в котором показана строка additionalLoginParams с примером идентификатора клиента.":::

Сохраните настройки, щелкнув **PUT**.

Теперь приложение настроено. Теперь интерфейсная часть готова к доступу к серверной части с помощью соответствующего маркера доступа.

Сведения о том, как настроить этот маркер доступа в других поставщиках, см. в разделе об [обновлении маркеров поставщиков удостоверений](app-service-authentication-how-to.md#refresh-identity-provider-tokens).

## <a name="call-api-securely-from-server-code"></a>Безопасный вызов API из кода сервера

На этом шаге активируйте ранее измененный код сервера, чтобы выполнять аутентифицированные вызовы серверного API.

Теперь ваше интерфейсное приложение имеет требуемое разрешение, а также добавляет идентификатор клиента серверного приложения к параметрам входа. Таким образом, оно может получить маркер доступа для проверки подлинности в серверном приложении. Служба приложений поставляет этот токен в код сервера, внедряя заголовок `X-MS-TOKEN-AAD-ACCESS-TOKEN` в каждый аутентифицированный запрос (см. раздел об [извлечении токена в коде приложения](app-service-authentication-how-to.md#retrieve-tokens-in-app-code)).

> [!NOTE]
> Эти заголовки внедряются для всех поддерживаемых языков. Доступ к ним осуществляется с помощью стандартного шаблона для каждого соответствующего языка.

В локальном репозитории снова откройте _Controllers/TodoController.cs_. В конструкторе `TodoController(TodoContext context)` добавьте следующий код.

```cs
public override void OnActionExecuting(ActionExecutingContext context)
{
    base.OnActionExecuting(context);

    _client.DefaultRequestHeaders.Accept.Clear();
    _client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Bearer", Request.Headers["X-MS-TOKEN-AAD-ACCESS-TOKEN"]);
}
```

Этот код добавляет стандартный HTTP-заголовок `Authorization: Bearer <access-token>` ко всем удаленным вызовам API. В конвейере выполните запрос ASP.NET Core MVC. `OnActionExecuting` выполняется непосредственно перед соответствующим действием, поэтому каждый исходящий вызов API теперь представляет маркер доступа.

Сохраните изменения. В окне терминала на локальном компьютере выполните следующие команды Git, чтобы выполнить развертывание аналогичного кода в интерфейсное приложение.

```bash
git add .
git commit -m "add authorization header for server code"
git push frontend master
```

Войдите в `https://<front-end-app-name>.azurewebsites.net` еще раз. На странице соглашения об использовании пользовательских данных нажмите кнопку **Принять**.

Теперь вы можете создавать, читать, обновлять и удалять данные из серверного приложения прямо в приложении. Единственное отличие теперь в том, что оба приложения защищены проверкой подлинности и авторизацией службы приложений, включая вызовы между службами.

Поздравляем! Код сервера теперь получает доступ к данным серверной части от имени пользователя, прошедшего проверку подлинности.

## <a name="call-api-securely-from-browser-code"></a>Безопасный вызов API из кода браузера

На этом шаге укажите внешний API для интерфейсного приложения Angular.js. Таким образом, вы узнаете, как получить маркер доступа и осуществлять с его помощью вызовы API к серверному приложению.

Код сервера имеет доступ к заголовкам запроса, код клиента может получить доступ к `GET /.auth/me`, чтобы получить те же токены доступа (см. раздел[Получение токенов в коде приложения](app-service-authentication-how-to.md#retrieve-tokens-in-app-code)).

> [!TIP]
> В этом разделе используются стандартные методы HTTP, чтобы продемонстрировать безопасные вызовы HTTP. Тем не менее вы можете использовать [библиотеку проверки подлинности Майкрософт для JavaScript](https://github.com/AzureAD/microsoft-authentication-library-for-js), чтобы упростить шаблон приложения Angular.js.
>

### <a name="configure-cors"></a>Настройка CORS

В Cloud Shell включите CORS для URL-адреса своего клиента с помощью команды [`az webapp cors add`](/cli/azure/webapp/cors#az_webapp_cors_add). Замените заполнители _\<back-end-app-name>_ и _\<front-end-app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myAuthResourceGroup --name <back-end-app-name> --allowed-origins 'https://<front-end-app-name>.azurewebsites.net'
```

Этот шаг не связан с проверкой подлинности и авторизацией. Тем не менее, это нужно, чтобы ваш браузер разрешал использовать междоменные вызовы API из вашего приложения Angular.js. Дополнительные сведения см. в разделе [Добавление функциональных возможностей CORS](app-service-web-tutorial-rest-api.md#add-cors-functionality).

### <a name="point-angularjs-app-to-back-end-api"></a>Указание серверного API для приложения Angular.js

В локальном репозитории откройте _wwwroot/index.html_.

В строке 51 задайте для переменной `apiEndpoint` URL-адрес HTTPS внутреннего приложения (`https://<back-end-app-name>.azurewebsites.net`). Замените _\<back-end-app-name>_ именем своего приложения в Службе приложений.

Откройте локальный репозиторий _wwwroot/app/scripts/todoListSvc.js_ и убедитесь, что `apiEndpoint` указан для всех вызовов API. Теперь приложение Angular.js вызывает серверные API-интерфейсы. 

### <a name="add-access-token-to-api-calls"></a>Добавление маркера доступа в вызовы API

В _wwwroot/app/scripts/todoListSvc.js_ над списком вызовов API (над строкой `getItems : function(){`) добавьте следующую функцию в список:

```javascript
setAuth: function (token) {
    $http.defaults.headers.common['Authorization'] = 'Bearer ' + token;
},
```

Эта функция вызывается для установки заголовка `Authorization` по умолчанию с маркером доступа. Он будет вызываться на следующем шаге.

В локальном репозитории _wwwroot/app/scripts/app.js_ найдите следующий код:

```javascript
$routeProvider.when("/Home", {
    controller: "todoListCtrl",
    templateUrl: "/App/Views/TodoList.html",
}).otherwise({ redirectTo: "/Home" });
```

Замените весь блок кода следующим кодом.

```javascript
$routeProvider.when("/Home", {
    controller: "todoListCtrl",
    templateUrl: "/App/Views/TodoList.html",
    resolve: {
        token: ['$http', 'todoListSvc', function ($http, todoListSvc) {
            return $http.get('/.auth/me').then(function (response) {
                todoListSvc.setAuth(response.data[0].access_token);
                return response.data[0].access_token;
            });
        }]
    },
}).otherwise({ redirectTo: "/Home" });
```

При новом изменении добавляется сопоставление `resolve`, которое вызывает `/.auth/me` и устанавливает маркер доступа. Это гарантирует, что у вас есть маркер доступа до создания экземпляра контроллера `todoListCtrl`. Таким образом, все вызовы API с помощью контроллера содержат токен.

### <a name="deploy-updates-and-test"></a>Развертывание и тестирование

Сохраните изменения. В окне терминала на локальном компьютере выполните следующие команды Git, чтобы выполнить развертывание аналогичного кода в интерфейсное приложение.

```bash
git add .
git commit -m "add authorization header for Angular"
git push frontend master
```

Перейдите к `https://<front-end-app-name>.azurewebsites.net` еще раз. Теперь вы можете создавать, читать, обновлять и удалять данные с серверного приложения прямо в приложении Angular.js.

Поздравляем! Код клиента теперь получает доступ к данным серверной части от имени пользователя, прошедшего проверку подлинности.

## <a name="when-access-tokens-expire"></a>Когда истекает срок действия маркеров доступа

Срок действия маркеров доступа истекает через некоторое время. Сведения об обновлении маркеров доступа без повторной проверки подлинности пользователей в приложении см. в разделе об [обновлении маркеров поставщиков удостоверений](app-service-authentication-how-to.md#refresh-identity-provider-tokens).

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущем шаге вы создали ресурсы Azure в группе ресурсов. Если эти ресурсы вам не понадобятся в будущем, вы можете удалить группу ресурсов, выполнив следующую команду в Cloud Shell:

```azurecli-interactive
az group delete --name myAuthResourceGroup
```

Ее выполнение может занять до минуты.

<a name="next"></a>
## <a name="next-steps"></a>Дальнейшие действия

Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * Включение встроенной проверки подлинности и авторизации.
> * Защита приложений от непроверенных запросов.
> * Использование Azure Active Directory в качестве поставщика удостоверений.
> * Доступ к удаленному приложению от имени выполнившего вход пользователя.
> * Безопасные вызовы между службами с помощью проверки подлинности с использованием токена.
> * Использование маркера доступа из серверного кода.
> * Использование маркера доступа из клиентского кода (браузер).

Перейдите к следующему руководству, чтобы научиться сопоставлять пользовательские DNS-имена с приложением.

> [!div class="nextstepaction"]
> [Сопоставление существующего настраиваемого DNS-имени со Службой приложений Azure](app-service-web-tutorial-custom-domain.md)

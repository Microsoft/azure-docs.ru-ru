---
title: Настройка параметров приложения функции в функциях Azure
description: Узнайте, как настроить параметры приложения функции в функциях Azure.
ms.assetid: 81eb04f8-9a27-45bb-bf24-9ab6c30d205c
ms.topic: conceptual
ms.date: 04/13/2020
ms.custom: cc996988-fb4f-47, devx-track-azurecli
ms.openlocfilehash: 5080d16a7b14506b24e07e2ee4ba862c645f83a8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98875455"
---
# <a name="manage-your-function-app"></a>Управление приложением функции 

В функциях Azure приложение-функция предоставляет контекст выполнения для отдельных функций. Поведение приложения-функции применяется ко всем содержащимся в нем функциям. Все функции в приложении-функции должны иметь один и тот же [язык](supported-languages.md). 

Отдельные функции в приложении-функции развертываются вместе и масштабируются вместе. Все функции в одном и том же приложении функции совместно используют ресурсы для каждого экземпляра, так как приложение функции масштабируется. 

Строки подключения, переменные среды и другие параметры приложения определяются отдельно для каждого приложения функции. Все данные, которые должны совместно использоваться приложениями-функциями, должны храниться извне в постоянном хранилище.

## <a name="get-started-in-the-azure-portal"></a>Начало работы на портале Azure

1. Для начала перейдите на [портал Azure] и войдите, используя свою учетную запись Azure. В строке поиска в верхней части портала введите имя приложения функции и выберите его из списка. 

2. В разделе **Параметры** в левой области выберите **Конфигурация**.

    :::image type="content" source="./media/functions-how-to-use-azure-function-app-settings/azure-function-app-main.png" alt-text="Обзор приложения-функции на портале Azure":::

Вы можете выбрать все необходимое для управления приложением-функцией на странице Обзор, в частности **[Параметры приложения](#settings)** и **[функции платформы](#platform-features)**.

## <a name="work-with-application-settings"></a><a name="settings"></a>Работа с параметрами приложения

Параметрами приложения можно управлять из [портал Azure](functions-how-to-use-azure-function-app-settings.md?tabs=portal#settings) и с помощью [Azure CLI](functions-how-to-use-azure-function-app-settings.md?tabs=azurecli#settings) и [Azure PowerShell](functions-how-to-use-azure-function-app-settings.md?tabs=powershell#settings). Вы также можете управлять параметрами приложения из [Visual Studio Code](functions-develop-vs-code.md#application-settings-in-azure) и из [Visual Studio](functions-develop-vs.md#function-app-settings). 

Эти параметры хранятся в зашифрованном виде. Дополнительные сведения см. в разделе [Application Settings Security](security-concepts.md#application-settings).

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы найти параметры приложения, см. раздел Начало [работы в портал Azure](#get-started-in-the-azure-portal). 

На вкладке **Параметры приложения** хранятся параметры, используемые приложением функции. Для просмотра значений на портале необходимо выбрать параметр **Показать значения** . Чтобы добавить параметр на портале, выберите **параметр создать приложение** и добавьте новую пару «ключ-значение».

![Параметры приложения функции в портал Azure.](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-settings-tab.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

[`az functionapp config appsettings list`](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-list)Команда возвращает существующие параметры приложения, как показано в следующем примере:

```azurecli-interactive
az functionapp config appsettings list --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME>
```

[`az functionapp config appsettings set`](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-set)Команда добавляет или обновляет параметр приложения. В следующем примере создается параметр с именем ключа `CUSTOM_FUNCTION_APP_SETTING` и значением `12345` :


```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings CUSTOM_FUNCTION_APP_SETTING=12345
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

[`Get-AzFunctionAppSetting`](/powershell/module/az.functions/get-azfunctionappsetting)Командлет возвращает существующие параметры приложения, как показано в следующем примере: 

```azurepowershell-interactive
Get-AzFunctionAppSetting -Name <FUNCTION_APP_NAME> -ResourceGroupName <RESOURCE_GROUP_NAME>
```

[`Update-AzFunctionAppSetting`](/powershell/module/az.functions/update-azfunctionappsetting)Команда добавляет или обновляет параметр приложения. В следующем примере создается параметр с именем ключа `CUSTOM_FUNCTION_APP_SETTING` и значением `12345` :

```azurepowershell-interactive
Update-AzFunctionAppSetting -Name <FUNCTION_APP_NAME> -ResourceGroupName <RESOURCE_GROUP_NAME> -AppSetting @{"CUSTOM_FUNCTION_APP_SETTING" = "12345"}
```

---

### <a name="use-application-settings"></a>Использовать параметры приложения

[!INCLUDE [functions-environment-variables](../../includes/functions-environment-variables.md)]

При разработке приложения-функции локально необходимо сохранить локальные копии этих значений в local.settings.jsв файле проекта. Дополнительные сведения см. в разделе [локальный файл параметров](functions-run-local.md#local-settings-file).

## <a name="hosting-plan-type"></a>Тип плана размещения

При создании приложения-функции также создается план размещения, в котором выполняется приложение. План может иметь одно или несколько приложений-функций. Функциональные возможности, масштабирование и цены на функции зависят от типа плана. Дополнительные сведения см. в статье [Параметры размещения функций Azure](functions-scale.md).

Вы можете определить тип плана, используемого приложением-функцией, из портал Azure или с помощью API-интерфейсов Azure CLI или Azure PowerShell. 

Тип плана указывается в следующих значениях:

| Тип плана | Портал | Azure CLI и PowerShell |
| --- | --- | --- |
| [Потребление](consumption-plan.md) | **Потребление** | `Dynamic` |
| [Премиальный](functions-premium-plan.md) | **еластикпремиум** | `ElasticPremium` |
| [Выделенная (служба приложений)](dedicated-plan.md) | Различные | Различные |

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы определить тип плана, используемого приложением функции, см. раздел **план службы приложений** на вкладке **Обзор** приложения функции в [портал Azure](https://portal.azure.com). Чтобы просмотреть ценовую категорию, выберите имя **плана службы приложений**, а затем в области слева выберите **свойства** .

![Просмотр плана масштабирования на портале](./media/functions-scale/function-app-overview-portal.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

Выполните следующую команду Azure CLI, чтобы получить тип плана размещения:

```azurecli-interactive
functionApp=<FUNCTION_APP_NAME>
resourceGroup=FunctionMonitoringExamples
appServicePlanId=$(az functionapp show --name $functionApp --resource-group $resourceGroup --query appServicePlanId --output tsv)
az appservice plan list --query "[?id=='$appServicePlanId'].sku.tier" --output tsv

```  

В предыдущем примере замените `<RESOURCE_GROUP>` и `<FUNCTION_APP_NAME>` соответствующими именами группы ресурсов и приложения функции. 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Выполните следующую команду Azure PowerShell, чтобы получить тип плана размещения:

```azurepowershell-interactive
$FunctionApp = '<FUNCTION_APP_NAME>'
$ResourceGroup = '<RESOURCE_GROUP>'

$PlanID = (Get-AzFunctionApp -ResourceGroupName $ResourceGroup -Name $FunctionApp).AppServicePlan
(Get-AzFunctionAppPlan -Name $PlanID -ResourceGroupName $ResourceGroup).SkuTier
```
В предыдущем примере замените `<RESOURCE_GROUP>` и `<FUNCTION_APP_NAME>` соответствующими именами группы ресурсов и приложения функции. 

---

## <a name="plan-migration"></a>Планирование миграции

Вы можете использовать команды Azure CLI, чтобы перенести приложение-функцию между планом потребления и планом Premium в Windows. Конкретные команды зависят от направления миграции. Прямая миграция на выделенный план (служба приложений) сейчас не поддерживается.

Эта миграция не поддерживается в Linux.

### <a name="consumption-to-premium"></a>Потребление в Premium

Используйте следующую процедуру, чтобы выполнить миграцию из плана потребления в план Premium в Windows.

1. Выполните следующую команду, чтобы создать новый план службы приложений (эластичный Premium) в том же регионе и группе ресурсов, что и имеющееся приложение-функция.  

    ```azurecli-interactive
    az functionapp plan create --name <NEW_PREMIUM_PLAN_NAME> --resource-group <MY_RESOURCE_GROUP> --location <REGION> --sku EP1
    ```

1. Выполните следующую команду, чтобы перенести существующее приложение функции в новый план Premium.

    ```azurecli-interactive
    az functionapp update --name <MY_APP_NAME> --resource-group <MY_RESOURCE_GROUP> --plan <NEW_PREMIUM_PLAN>
    ```

1. Если вы больше не хотите использовать предыдущий план приложения функции потребления, удалите исходный план приложения функции после подтверждения успешного переноса на новый. Выполните следующую команду, чтобы получить список всех планов потребления в группе ресурсов.

    ```azurecli-interactive
    az functionapp plan list --resource-group <MY_RESOURCE_GROUP> --query "[?sku.family=='Y'].{PlanName:name,Sites:numberOfSites}" -o table
    ```

    Вы можете безопасно удалить план с нулевыми сайтами, который был перенесен из.

1. Выполните следующую команду, чтобы удалить план потребления, перенесенный из.

    ```azurecli-interactive
    az functionapp plan delete --name <CONSUMPTION_PLAN_NAME> --resource-group <MY_RESOURCE_GROUP>
    ```

### <a name="premium-to-consumption"></a>Premium на потребление

Используйте следующую процедуру, чтобы выполнить миграцию из плана Premium в план потребления в Windows.

1. Выполните следующую команду, чтобы создать новое приложение-функцию (потребление) в том же регионе и группе ресурсов, что и существующее приложение-функция. Эта команда также создает новый план потребления, в котором выполняется приложение-функция.

    ```azurecli-interactive
    az functionapp create --resource-group <MY_RESOURCE_GROUP> --name <NEW_CONSUMPTION_APP_NAME> --consumption-plan-location <REGION> --runtime dotnet --functions-version 3 --storage-account <STORAGE_NAME>
    ```

1. Выполните следующую команду, чтобы перенести существующее приложение функции в новый план потребления.

    ```azurecli-interactive
    az functionapp update --name <MY_APP_NAME> --resource-group <MY_RESOURCE_GROUP> --plan <NEW_CONSUMPTION_PLAN>
    ```

1. Удалите приложение функции, созданное на шаге 1, так как вам понадобится только план, созданный для запуска существующего приложения функции.

    ```azurecli-interactive
    az functionapp delete --name <NEW_CONSUMPTION_APP_NAME> --resource-group <MY_RESOURCE_GROUP>
    ```

1. Если вы больше не хотите использовать предыдущий план приложения для расширенной функции, удалите исходный план приложения функции после подтверждения успешного переноса на новый. Обратите внимание, что если план не удаляется, вам по-прежнему будет выставляться плата за план Premium. Выполните следующую команду, чтобы получить список всех планов Premium в группе ресурсов.

    ```azurecli-interactive
    az functionapp plan list --resource-group <MY_RESOURCE_GROUP> --query "[?sku.family=='EP'].{PlanName:name,Sites:numberOfSites}" -o table
    ```

1. Выполните следующую команду, чтобы удалить план Premium, из которого был выполнен перенос.

    ```azurecli-interactive
    az functionapp plan delete --name <PREMIUM_PLAN> --resource-group <MY_RESOURCE_GROUP>
    ```

## <a name="platform-features"></a>Функции платформы

Приложения функций выполняются в и обслуживаются платформой службы приложений Azure. Поэтому они имеют доступ к большинству функций базовой платформы веб-хостинга Azure. В левой области вы можете получить доступ ко многим функциям платформы службы приложений, которые можно использовать в приложениях-функциях. 

> [!NOTE]
> Не все функции службы приложений доступны при выполнении приложения с планом размещения потребления.

Оставшаяся часть этой статьи посвящена следующим функциям службы приложений в портал Azure, которые полезны для функций:

+ [Редактор службы приложений](#editor)
+ [Консоль](#console)
+ [Дополнительные инструменты (Kudu)](#kudu)
+ [Варианты развертывания](#deployment)
+ [CORS](#cors)
+ [Аутентификация](#auth)

Дополнительные сведения о работе с параметрами службы приложений см. в статье [Настройка параметров в службе приложений Azure](../app-service/configure-common.md).

### <a name="app-service-editor"></a><a name="editor"></a>Редактор службы приложений

![Редактор службы приложений](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-appservice-editor.png)

Редактор службы приложений — это расширенный редактор на портале, который можно использовать для изменения JSON-файлов конфигурации и файлов с кодом. При выборе этого параметра откроется отдельная вкладка браузера с базовым редактором. Он позволяет выполнять интеграцию с репозиторием Git, запускать и отлаживать код и изменять параметры приложения-функции. Этот редактор предоставляет улучшенную среду разработки для функций по сравнению с встроенным редактором функций.  

Рекомендуется разрабатывать функции на локальном компьютере. При разработке локально и публикации в Azure файлы проекта доступны только для чтения на портале. Дополнительные сведения см. в статье Использование [кода и тестирование функций Azure в локальной](functions-develop-local.md)среде.

### <a name="console"></a><a name="console"></a>Консоль

![Консоль приложения-функции](./media/functions-how-to-use-azure-function-app-settings/configure-function-console.png)

Консоль на портале оптимально подходит разработчикам, желающим взаимодействовать с приложением-функцией из командной строки. Стандартные команды включают создание каталогов и файлов и навигацию по ним, а также выполнение пакетных файлов и сценариев. 

При разработке локально рекомендуется использовать [Azure functions Core Tools](functions-run-local.md) и [Azure CLI].

### <a name="advanced-tools-kudu"></a><a name="kudu"></a>Дополнительные инструменты (Kudu)

![Настройка Kudu](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-kudu.png)

Дополнительные средства для службы приложений (которые также называются Kudu) предоставляют доступ к расширенным административным функциям для приложения-функции. С помощью Kudu можно управлять системными сведениями, параметрами приложения, переменными среды, заголовками HTTP и переменными сервера. Кроме того, можно также запустить **Kudu**, перейдя на конечную точку SCM для приложения-функции, например `https://<myfunctionapp>.scm.azurewebsites.net/`. 


### <a name="deployment-center"></a><a name="deployment"></a>Центр развертывания

При использовании решения системы управления версиями для разработки и сопровождения кода функций центр развертывания позволяет выполнять сборку и развертывание из системы управления версиями. Проект будет создан и развернут в Azure при внесении обновлений. Дополнительные сведения см. [в статье технологии развертывания в службе "функции Azure](functions-deployment-technologies.md)".

### <a name="cross-origin-resource-sharing"></a><a name="cors"></a>Предоставление общего доступа к ресурсам независимо от источника

Чтобы предотвратить выполнение вредоносного кода на клиенте, современные браузеры блокируют запросы от веб-приложений к ресурсам, выполняемым в отдельном домене. [Общий доступ к ресурсам между источниками (CORS)](https://developer.mozilla.org/docs/Web/HTTP/CORS) позволяет `Access-Control-Allow-Origin` заголовку объявить, какие источники могут вызывать конечные точки в приложении-функции.

#### <a name="portal"></a>Портал

При настройке списка **разрешенных источников** для приложения `Access-Control-Allow-Origin` -функции заголовок автоматически добавляется ко всем ответам из конечных точек HTTP в приложении функции. 

![Настройка списка CORS приложения функции](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-cors.png)

При использовании подстановочного знака ( `*` ) все остальные домены игнорируются. 

Используйте [`az functionapp cors add`](/cli/azure/functionapp/cors#az-functionapp-cors-add) команду, чтобы добавить домен в список разрешенных источников. В следующем примере добавляется домен contoso.com:

```azurecli-interactive
az functionapp cors add --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--allowed-origins https://contoso.com
```

Используйте [`az functionapp cors show`](/cli/azure/functionapp/cors#az-functionapp-cors-show) команду для вывода списка текущих разрешенных источников.

### <a name="authentication"></a><a name="auth"></a>Аутентификация

![Настройка проверки подлинности для приложения-функции](./media/functions-how-to-use-azure-function-app-settings/configure-function-app-authentication.png)

Если функции используют триггер HTTP, можно настроить обязательную предварительную проверку подлинности для вызовов. Служба приложений поддерживает Azure Active Directory проверку подлинности и вход с помощью поставщиков социальных сетей, таких как Facebook, Microsoft и Twitter. Дополнительные сведения о настройке определенных поставщиков аутентификации см. в разделе [Проверка подлинности и авторизация в службе приложений Azure](../app-service/overview-authentication-authorization.md). 


## <a name="next-steps"></a>Дальнейшие действия

+ [Настройка параметров службы приложений Azure](../app-service/configure-common.md)
+ [Непрерывное развертывание для Функций Azure](functions-continuous-deployment.md)

[Azure CLI]: /cli/azure/
[Портал Azure]: https://portal.azure.com

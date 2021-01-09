---
title: Настройка параметров приложения функции в функциях Azure
description: Узнайте, как настроить параметры приложения функции в функциях Azure.
ms.assetid: 81eb04f8-9a27-45bb-bf24-9ab6c30d205c
ms.topic: conceptual
ms.date: 04/13/2020
ms.custom: cc996988-fb4f-47, devx-track-azurecli
ms.openlocfilehash: 2526fd60d6e07ecf43864945f2b05858b41ca567
ms.sourcegitcommit: c4c554db636f829d7abe70e2c433d27281b35183
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2021
ms.locfileid: "98035212"
---
# <a name="manage-your-function-app"></a>Управление приложением функции 

В функциях Azure приложение-функция предоставляет контекст выполнения для отдельных функций. Поведение приложения-функции применяется ко всем содержащимся в нем функциям. Все функции в приложении-функции должны иметь один и тот же [язык](supported-languages.md). 

Отдельные функции в приложении-функции развертываются вместе и масштабируются вместе. Все функции в одном и том же приложении функции совместно используют ресурсы для каждого экземпляра, так как приложение функции масштабируется. 

Строки подключения, переменные среды и другие параметры приложения определяются отдельно для каждого приложения функции. Все данные, которые должны совместно использоваться приложениями-функциями, должны храниться извне в постоянном хранилище.

В этой статье описывается настройка приложений функций и управление ими. 

> [!TIP]  
> Многие параметры конфигурации можно также управлять с помощью [Azure CLI]. 

## <a name="get-started-in-the-azure-portal"></a>Начало работы на портале Azure

1. Для начала перейдите на [портал Azure] и войдите, используя свою учетную запись Azure. В строке поиска в верхней части портала введите имя приложения функции и выберите его из списка. 

2. В разделе **Параметры** в левой области выберите **Конфигурация**.

    :::image type="content" source="./media/functions-how-to-use-azure-function-app-settings/azure-function-app-main.png" alt-text="Обзор приложения-функции на портале Azure":::

Вы можете выбрать все необходимое для управления приложением-функцией на странице Обзор, в частности **[Параметры приложения](#settings)** и **[функции платформы](#platform-features)**.

## <a name="work-with-application-settings"></a><a name="settings"></a>Работа с параметрами приложения

На вкладке **Параметры приложения** хранятся параметры, используемые приложением функции. Эти параметры хранятся в зашифрованном виде, поэтому необходимо выбрать параметр **Показать значения** , чтобы просмотреть значения на портале. Вы также можете получить доступ к параметрам приложения с помощью Azure CLI.

### <a name="portal"></a>Портал

Чтобы добавить параметр на портале, выберите **параметр создать приложение** и добавьте новую пару «ключ-значение».

![Параметры приложения функции в портал Azure.](./media/functions-how-to-use-azure-function-app-settings/azure-function-app-settings-tab.png)

### <a name="azure-cli"></a>Azure CLI

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

### <a name="use-application-settings"></a>Использовать параметры приложения

[!INCLUDE [functions-environment-variables](../../includes/functions-environment-variables.md)]

При разработке приложения-функции локально необходимо сохранить локальные копии этих значений в local.settings.jsв файле проекта. Дополнительные сведения см. в разделе [локальный файл параметров](functions-run-local.md#local-settings-file).

## <a name="hosting-plan-type"></a>Тип плана размещения

При создании приложения-функции также создается план размещения службы приложений, в котором выполняется приложение. План может иметь одно или несколько приложений-функций. Функциональные возможности, масштабирование и цены на функции зависят от типа плана. Дополнительные сведения см. на [странице с ценами на функции Azure](https://azure.microsoft.com/pricing/details/functions/).

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

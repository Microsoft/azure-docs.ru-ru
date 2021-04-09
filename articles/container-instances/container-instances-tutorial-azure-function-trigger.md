---
title: Учебник. Активация группы контейнеров по функциям Azure
description: Создание бессерверной функции PowerShell, которая активируется HTTP-запросами, для автоматического создания экземпляров контейнеров Azure
ms.topic: tutorial
ms.date: 06/10/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e7315796f2f7f89800b58f5fa607e69cd7ae3447
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98935407"
---
# <a name="tutorial-use-an-http-triggered-azure-function-to-create-a-container-group"></a>Руководство по применению функции Azure, активируемой HTTP-запросами, для создания группы контейнеров

[Функции Azure](../azure-functions/functions-overview.md) — это бессерверная служба вычислений, которая умеет выполнять скрипты или код в ответ на такие события, как HTTP-запрос, таймер или сообщение в очереди службы хранилища Azure.

Работая с этим руководством, вы создадите функцию Azure, которая принимает HTTP-запрос и активирует развертывание [группы контейнеров](container-instances-container-groups.md). В этом примере показано, как применить базовые возможности Функций Azure для автоматического создания ресурсов в службе "Экземпляры контейнеров Azure". Вы можете изменить или дополнить этот пример, чтобы создать более сложные сценарии или применить другие триггеры событий. 

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Использование Visual Studio Code в сочетании с [расширением Функций Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для создания простой функции PowerShell, активируемой по HTTP-запросу.
> * Создание удостоверения в приложении-функции и предоставление ему разрешений на создание ресурсов Azure.
> * Изменение и повторная публикация функции PowerShell для автоматизации развертывания группы контейнеров с одним контейнером.
> * Проверка развертывания контейнера, активируемого по HTTP-запросу.

## <a name="prerequisites"></a>Предварительные требования

Предварительные требования к установке Visual Studio Code с расширением Функций Azure в своей ОС см. в статье о [создании первой функции Azure с использованием Visual Studio Code](../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-powershell#configure-your-environment).

Для выполнения дополнительных инструкций, приведенных в этой статье, используется Azure PowerShell. Если вам необходимо выполнить установку или обновление, см. сведения в статье [Установка Azure PowerShell][azure-powershell-install] и разделе [Вход в Azure](/powershell/azure/get-started-azureps#sign-in-to-azure).

## <a name="create-a-basic-powershell-function"></a>Создание базовой функции PowerShell

Выполните инструкции из статьи [Создание первой функции PowerShell в Azure](../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-powershell), чтобы создать функцию PowerShell из шаблона Триггер HTTP. Присвойте этой функции Azure имя **HttpTrigger**, предложенное по умолчанию. Протестируйте эту функцию локально и опубликуйте проект в виде приложения-функции Azure, как описано в этом кратком руководстве. В этом примере создается простая функция, которая активируется HTTP-запросом и возвращает текстовую строку. Далее вы измените функцию, чтобы она создавала группу контейнеров.

В этой статье предполагается, что вы публикуете проект с именем *myfunctionapp* в группе ресурсов Azure, которой автоматически присваивается имя, совпадающее с именем приложения-функции (также *myfunctionapp*). При выполнении последующих инструкций замените их фактическими именами приложения-функции и группы ресурсов.

## <a name="enable-an-azure-managed-identity-in-the-function-app"></a>Настройка в приложении-функции удостоверения, управляемого Azure

С помощью приведенных ниже команд можно настроить в приложении-функции [управляемое удостоверение](../app-service/overview-managed-identity.md?toc=/azure/azure-functions/toc.json#add-a-system-assigned-identity), назначаемое системой. С его помощью выполняющий приложение узел PowerShell сможет автоматически проводить проверку подлинности в Azure, позволяя функциям выполнять действия со службами Azure, к которым предоставляет доступ это удостоверение. Далее вы предоставите управляемому удостоверению права на создание ресурсов в группе ресурсов для приложения-функции. 

[Добавьте удостоверение](../app-service/overview-managed-identity.md?tabs=dotnet#using-azure-powershell-1) в приложение-функцию:

```powershell
Update-AzFunctionApp -Name myfunctionapp `
    -ResourceGroupName myfunctionapp `
    -IdentityType SystemAssigned
```

Назначьте удостоверению роль "Участник" в области действия группы ресурсов:

```powershell
$SP=(Get-AzADServicePrincipal -DisplayName myfunctionapp).Id
$RG=(Get-AzResourceGroup -Name myfunctionapp).ResourceId
New-AzRoleAssignment -ObjectId $SP -RoleDefinitionName "Contributor" -Scope $RG
```

## <a name="modify-httptrigger-function"></a>Изменение функции HttpTrigger

Измените код PowerShell в функции **HttpTrigger**, чтобы она создавала группу контейнеров. Откройте файл функции `run.ps1` и найдите в нем следующий блок кода. Этот код отображает значение имени, если оно было передано как строка запроса в URL-адресе функции:

```powershell
[...]
if ($name) {
    $body = "Hello, $name. This HTTP triggered function executed successfully."
}
[...]
```

Замените эту строку следующим блоком кода. Если в строке запроса передано значение имени, оно будет использоваться для создания и именования группы контейнеров с помощью командлета [New-AzContainerGroup][new-azcontainergroup]. Не забудьте заменить имя *myfunctionapp* реальным именем группы ресурсов для приложения-функции.

```powershell
[...]
if ($name) {
    New-AzContainerGroup -ResourceGroupName myfunctionapp -Name $name `
        -Image alpine -OsType Linux `
        -Command "echo 'Hello from an Azure container instance triggered by an Azure function'" `
        -RestartPolicy Never
    if ($?) {
        $body = "This HTTP triggered function executed successfully. Started container group $name"
    }
    else  {
        $body = "There was a problem starting the container group."
    }
[...]
```

В этом примере создается группа контейнеров с одним контейнером, в котором выполняется образ `alpine`. Этот контейнер выполняет одну команду `echo` и завершает работу. В реальной системе можно точно так же запускать создание одной или нескольких групп контейнеров для выполнения пакетного задания.
 
## <a name="test-function-app-locally"></a>Локальное тестирование приложения-функции

Перед повторной публикацией проекта приложения-функции в Azure убедитесь, что функция выполняется в локальной среде. При локальном выполнении функции ресурсы Azure не создаются. Но можно проверить поток функции с передачей значения имени в строке запроса и без такой передачи. Ознакомьтесь со статьей [Локальная отладка функций Azure PowerShell](../azure-functions/functions-debug-powershell-local.md).

## <a name="republish-azure-function-app"></a>Повторная публикация приложения-функции Azure

Убедившись, что функция выполняется локально, опубликуйте проект в уже существующем приложении-функции Azure.

1. Откройте палитру команд в Visual Studio Code. Найдите и выберите элемент `Azure Functions: Deploy to Function App...`.
1. Выберите текущую рабочую папку для архивации и развертывания.
1. Выберите подписку, а затем имя существующего приложения-функции (в нашем примере это *myfunctionapp*). Подтвердите, что вы хотите перезаписать предыдущее развертывание.

После создания приложения-функции и применения пакета развертывания отобразится уведомление. Выберите **View Output** (Просмотреть выходные данные) в уведомлении, чтобы просмотреть результаты создания и развертывания обновленных ресурсов Azure.

## <a name="run-the-function-in-azure"></a>Запуск функции в Azure

После успешного завершения развертывания получите URL-адрес функции. Для этого можно открыть область **Azure: Functions** (Azure: Функции) в Visual Studio Code и скопировать URL-адрес функции **HttpTrigger** или получить этот URL-адрес на [портале Azure](../azure-functions/functions-get-started.md).

URL-адрес функции выглядит примерно так:

```
https://myfunctionapp.azurewebsites.net/api/HttpTrigger
```

### <a name="run-function-without-passing-a-name"></a>Запуск функции без передачи имени

Для первого теста выполните команду `curl` и передайте ей URL-адрес функции без добавления строки запроса `name`. 

```bash
curl --verbose "https://myfunctionapp.azurewebsites.net/api/HttpTrigger"
```

Функция возвращает код состояния 200 и текст `This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response`:

```
[...]
> GET /api/HttpTrigger? HTTP/1.1
> Host: myfunctionapp.azurewebsites.net
> User-Agent: curl/7.64.1
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/1.1 200 OK
< Content-Length: 135
< Content-Type: text/plain; charset=utf-8
< Request-Context: appId=cid-v1:d0bd0123-f713-4579-8990-bb368a229c38
< Date: Wed, 10 Jun 2020 17:50:27 GMT
< 
* Connection #0 to host myfunctionapp.azurewebsites.net left intact
This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.* Closing connection 0
```

### <a name="run-function-and-pass-the-name-of-a-container-group"></a>Запуск функции с передачей имени для группы контейнеров

Теперь еще раз выполните команду `curl` и добавьте имя группы контейнеров (в нашем примере это *mycontainergroup*) в качестве строки запроса `?name=mycontainergroup`:

```bash
curl --verbose "https://myfunctionapp.azurewebsites.net/api/HttpTrigger?name=mycontainergroup"
```

Функция возвращает код состояния 200 и активирует создание группы контейнеров:

```
[...]
> GET /api/HttpTrigger1?name=mycontainergroup HTTP/1.1
> Host: myfunctionapp.azurewebsites.net
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Content-Length: 92
< Content-Type: text/plain; charset=utf-8
< Request-Context: appId=cid-v1:d0bd0123-f713-4579-8990-bb368a229c38
< Date: Wed, 10 Jun 2020 17:54:31 GMT
< 
* Connection #0 to host myfunctionapp.azurewebsites.net left intact
This HTTP triggered function executed successfully. Started container group mycontainergroup* Closing connection 0
```

Убедитесь, что контейнер запущен, с помощью команды [Get-AzContainerInstanceLog][get-azcontainerinstancelog]:

```azurecli
Get-AzContainerInstanceLog -ResourceGroupName myfunctionapp `
  -ContainerGroupName mycontainergroup 
```

Образец вывода:

```console
Hello from an Azure container instance triggered by an Azure function
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы, созданные при работе с этим руководством, больше не требуются, можно выполнить команду [az group delete][az-group-delete], чтобы удалить группу ресурсов и все ресурсы, которые она содержит. Эта команда удаляет созданное вами приложение-функцию, а также запущенный контейнер и все связанные с ним ресурсы.

```azurecli-interactive
az group delete --name myfunctionapp
```

## <a name="next-steps"></a>Дальнейшие действия

В результате работы с этим руководством вы создали функцию Azure, которая принимает HTTP-запрос и активирует развертывание группы контейнеров. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Использование Visual Studio Code в сочетании с расширением Функций Azure для создания простой функции PowerShell, активируемой по HTTP-запросу.
> * Создание удостоверения в приложении-функции и предоставление ему разрешений на создание ресурсов Azure.
> * Изменение кода функции PowerShell для автоматизации развертывания группы контейнеров с одним контейнером.
> * Проверка развертывания контейнера, активируемого по HTTP-запросу.

Подробный пример запуска и мониторинга контейнерного задания см. в [этой публикации](https://dev.to/azure/event-driven-serverless-containers-with-powershell-azure-functions-and-azure-container-instances-e9b) и прилагаемом [примере кода](https://github.com/anthonychu/functions-powershell-run-aci).

Подробные инструкции по созданию функций Azure и публикации проекта функций см. в [документации по Функциям Azure](../azure-functions/index.yml). 

<!-- IMAGES -->


<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-powershell-install]: /powershell/azure/install-az-ps
[new-azcontainergroup]: /powershell/module/az.containerinstance/new-azcontainergroup
[get-azcontainerinstancelog]: /powershell/module/az.containerinstance/get-azcontainerinstancelog

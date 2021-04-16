---
title: Руководство. Маршрутизация событий изменения состояния политики в сетку событий с помощью Azure CLI
description: В этом руководстве вы настроите сетку событий для прослушивания событий изменения состояния политики и вызова веб-перехватчика.
ms.date: 03/29/2021
ms.topic: tutorial
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1fe87e4fd3349df7d8f5d57b2b2d95f95ed3fba8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105734884"
---
# <a name="tutorial-route-policy-state-change-events-to-event-grid-with-azure-cli"></a>Руководство. Маршрутизация событий изменения состояния политики в сетку событий с помощью Azure CLI

В этой статье вы узнаете, как настроить подписки на события политики Azure на отправку событий изменения состояния политики в конечную веб-точку. Пользователи политики Azure могут подписаться на отправку событий в случае изменения состояния политики в ресурсах. Эти события могут активировать веб-перехватчики, [функции Azure](../../../azure-functions/index.yml), [очереди службы хранилища Azure](../../../storage/queues/index.yml) или любой другой обработчик событий, который поддерживается [Сеткой событий Azure](../../../event-grid/index.yml). Как правило, события отправляются на конечную точку, которая обрабатывает данные событий и выполняет соответствующие действия. Но в этом руководстве для простоты события отправляются в веб-приложение, которое собирает и отображает сообщения.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

- Для работы с этим кратким руководством требуется запустить Azure CLI версии не ниже 2.0.76. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

- Даже если вы ранее использовали политику Azure или сетку событий, повторно зарегистрируйте соответствующих поставщиков ресурсов:

  ```azurecli-interactive
  # Log in first with az login if you're not using Cloud Shell

  # Provider register: Register the Azure Policy provider
  az provider register --namespace Microsoft.PolicyInsights

  # Provider register: Register the Azure Event Grid provider
  az provider register --namespace Microsoft.EventGrid
  ```

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Темами событий сетки являются ресурсы Azure, которые необходимо поместить в группу ресурсов Azure. Группа ресурсов Azure — это логическая коллекция, в которой выполняется развертывание и администрирование ресурсов Azure.

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). 

В следующем примере создается группа ресурсов с именем `<resource_group_name>` в расположении _westus_. Замените `<resource_group_name>` уникальным именем для группы ресурсов.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az group create --name <resource_group_name> --location westus
```

## <a name="create-an-event-grid-system-topic"></a>Создание системного раздела сетки событий

Теперь, когда у нас есть группа ресурсов, мы создаем [системный раздел](../../../event-grid/system-topics.md). Системный раздел в Сетке событий представляет одно или несколько событий, опубликованных службами Azure, такими как политика Azure и Центры событий Azure. В этом системном разделе для изменений состояния политики Azure используется тип раздела `Microsoft.PolicyInsights.PolicyStates`. Замените `<SubscriptionID>` в параметре **scope** идентификатором своей подписки, а `<resource_group_name>` в параметре **resource-group** — ранее созданной группой ресурсов.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az eventgrid system-topic create --name PolicyStateChanges --location global --topic-type Microsoft.PolicyInsights.PolicyStates --source "/subscriptions/<SubscriptionID>" --resource-group "<resource_group_name>"
```

## <a name="create-a-message-endpoint"></a>Создание конечной точки сообщения

Перед подпиской на раздел необходимо создать конечную точку для сообщения о событии. Обычно конечная точка выполняет действия на основе данных событий. Чтобы упростить работу с этим руководством, разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

Замените `<your-site-name>` уникальным именем для вашего веб-приложения. Имя веб-приложения должно быть уникальным, так как оно включается в запись DNS.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az deployment group create \
  --resource-group <resource_group_name> \
  --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/master/azuredeploy.json" \
  --parameters siteName=<your-site-name> hostingPlanName=viewerhost
```

Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

Вы увидите сайт, на котором сейчас не отображаются никакие сообщения.

## <a name="subscribe-to-the-system-topic"></a>Подписка на системный раздел

Подписка на раздел предоставляет Сетке событий Azure информацию о том, какие события вы намерены отслеживать и куда их следует отправлять. В следующем примере создается подписка на созданный системный раздел и передается URL-адрес из веб-приложения в качестве конечной точки для получения уведомлений о событиях. Замените `<event_subscription_name>` именем подписки на событие. Для `<resource_group_name>` и `<your-site-name>` используйте созданные ранее значения.

Конечная точка веб-приложения должна содержать суффикс `/api/updates/`.

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

# Create the subscription
az eventgrid system-topic event-subscription create \
  --name <event_subscription_name> \
  --resource-group <resource_group_name> \
  --system-topic-name PolicyStateChanges \
  --endpoint https://<your-site-name>.azurewebsites.net/api/updates
```

Теперь снова откройте веб-приложение и убедитесь, что оно успешно получило отправленное событие подтверждения подписки. Щелкните значок с изображением глаза, чтобы развернуть данные события. Сетка событий отправляет событие подтверждения, чтобы конечная точка могла подтвердить, что она готова получать данные события. Веб-приложение содержит код для проверки подписки.

:::image type="content" source="../media/route-state-change-events/view-subscription-event.png" alt-text="Снимок экрана: событие подтверждения подписки на сетку событий в предварительно созданном веб-приложении.":::

## <a name="create-a-policy-assignment"></a>Создание назначения политики

В этом кратком руководстве вы создадите назначение политики и присвоите определение **Требование тега в группах ресурсов**. Такое определение политики идентифицирует группы ресурсов, в которых отсутствует тег, настроенный во время назначения политики.

Чтобы создать назначение политики для группы ресурсов, созданной для хранения раздела сетки событий, выполните следующую команду:

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az policy assignment create --name 'requiredtags-events' --display-name 'Require tag on RG' --scope '<ResourceGroupScope>' --policy '<policy definition ID>' --params '{ "tagName": { "value": "EventTest" } }'
```

В указанной выше команде используются следующие сведения:

- **Name** — фактическое имя назначения. Для этого примера было использовано значение _requiredtags-events_.
- **DisplayName** — отображаемое имя назначения политики. В этом случае вы используете значение _Требовать тег для группы ресурсов_.
- **Scope.** Область определяет, к каким ресурсам или группе ресурсов принудительно применяется назначение политики. Политика может назначаться разным ресурсам: от подписки до групп ресурсов. Обязательно замените значение &lt;scope&gt; именем своей группы ресурсов. Для области группы ресурсов используется формат `/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>`.
- **Policy** — идентификатор определения политики, на основе которой вы создаете назначение. В этом случае это определение для идентификатора политики _Требование тега в группах ресурсов_. Чтобы получить идентификатор определения политики, выполните следующую команду: `az policy definition list --query "[?displayName=='Require a tag on resource groups']"`

После создания назначения политики дождитесь появления уведомления о событии **Microsoft.PolicyInsights.PolicyStateCreated** в веб-приложении. Созданная группа ресурсов показывает значение `data.complianceState` параметра _NonCompliant_ для запуска.

:::image type="content" source="../media/route-state-change-events/view-policystatecreated-event.png" alt-text="Снимок экрана: событие создания состояния политики подписки на сетку событий для группы ресурсов в предварительно созданном веб-приложении.":::

> [!NOTE]
> Если группа ресурсов наследует другие назначения политики из подписки или иерархии группы управления, для каждого из них также отображаются события. Проверьте значение свойства `data.policyDefinitionId` и убедитесь, что событие относится к назначению в этом руководстве.

## <a name="trigger-a-change-on-the-resource-group"></a>Активация изменения в группе ресурсов

Чтобы обеспечить соответствие группе ресурсов, требуется тег с именем **EventTest**. Добавьте тег в группу ресурсов с помощью следующей команды, заменив `<SubscriptionID>` идентификатором подписки и `<ResourceGroup>` именем группы ресурсов:

```azurecli-interactive
# Log in first with az login if you're not using Cloud Shell

az tag create --resource-id '/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup>' --tags EventTest=true
```

После добавления необходимого тега в группу ресурсов подождите, пока в веб-приложении не появится уведомление о событии **Microsoft.PolicyInsights.PolicyStateChanged**. Разверните событие. Теперь для значения `data.complianceState` отображается _Compliant_.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжать работу с этим веб-приложением и подпиской на события политики Azure, не очищайте ресурсы, созданные в рамках этой статьи. Если вы не планируете продолжать работу, используйте следующую команду, чтобы удалить все ресурсы, созданные в рамках этой статьи.

Замените `<resource_group_name>` именем группы ресурсов, созданной ранее.

```azurecli-interactive
az group delete --name <resource_group_name>
```

## <a name="next-steps"></a>Следующие шаги

Теперь, когда вы знаете, как создавать разделы и подписки на события для политики Azure, ознакомьтесь с дополнительными сведениями о событиях изменения состояния политики и возможностях службы "Сетка событий":

- [Реагирование на события изменения состояния политики Azure](../concepts/event-overview.md)
- [Сведения о схеме политики Azure для службы "Сетка событий"](../../../event-grid/event-schema-policy.md)
- [An introduction to Azure Event Grid](../../../event-grid/overview.md) (Общие сведения о службе "Сетка событий Azure")
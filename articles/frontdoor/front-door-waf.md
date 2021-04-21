---
title: Руководство по Масштабирование и защита веб-приложения с помощью Azure Front Door и Брандмауэра веб-приложений Azure (WAF)
description: В этом руководстве показано, как использовать Брандмауэр веб-приложений со службой Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/01/2020
ms.author: duau
ms.openlocfilehash: d315fa5b588c6e5f2e4643ca18626e400e6ca01b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107785655"
---
# <a name="tutorial-quickly-scale-and-protect-a-web-application-by-using-azure-front-door-and-azure-web-application-firewall-waf"></a>Руководство по Быстрое масштабирование и защита веб-приложения с помощью службы Azure Front Door и Брандмауэра веб-приложений Azure (WAF)

В последние недели на работу многих веб-приложений влияет быстрое увеличение объема трафика из-за пандемии COVID-19. Кроме того, увеличился объем вредоносного трафика, в том числе связанного с атаками типа "отказ в обслуживании". Но существует эффективный способ одновременно горизонтально увеличить масштаб приложения в соответствии с возросшим объемом трафика и обеспечить защиту от атак. Вам нужно просто настроить Azure Front Door с Azure WAF, чтобы ускорить работу, включить кэширование и создать дополнительный уровень защиты перед вашим приложением. В этой статье описывается, как быстро настроить Azure Front Door с Azure WAF для любых веб-приложений, которые работают в среде Azure и за ее пределами. 

В этом руководстве показано, как настроить WAF с помощью Azure CLI. То же самое можно выполнить на портале Azure, с помощью Azure PowerShell, Azure Resource Manager или Azure REST API. 

В этом руководстве вы узнаете, как:
> [!div class="checklist"]
> - создать профиль Front Door;
> - создать политику Azure WAF;
> - настроить набор правил для политики WAF;
> - связать политику WAF с Front Door;
> - настроить личный домен.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

- Описанные в этом руководстве действия выполняются через Azure CLI. Просмотрите это руководство по [началу работы с Azure CLI](/cli/azure/get-started-with-azure-cli).

  > [!TIP] 
  > Чтобы быстро начать работу с Azure CLI, можно использовать [Bash в Azure Cloud Shell](../cloud-shell/quickstart.md).

- Убедитесь, что в Azure CLI добавлено расширение `front-door`.

   ```azurecli-interactive 
   az extension add --name front-door
   ```

> [!NOTE] 
> Дополнительные сведения о командах, используемых в этом руководстве, см. в [справочных материалах по Azure CLI для Front Door](/cli/azure/ext/front-door).

## <a name="create-an-azure-front-door-resource"></a>Создание ресурса Azure Front Door

```azurecli-interactive 
az network front-door create --backend-address <>  --accepted-protocols <> --name <> --resource-group <>
```

`--backend-address`: полное доменное имя приложения, для которого нужно настроить защиту. Например, `myapplication.contoso.com`.

`--accepted-protocols`: протоколы, которые должна поддерживать служба Azure Front Door для веб-приложения. Например, `--accepted-protocols Http Https`.

`--name`: имя ресурса Azure Front Door.

`--resource-group`: группа ресурсов, в которую нужно поместить ресурс Azure Front Door. Дополнительные сведения о группах ресурсов см. в статье [Управление группами ресурсов в Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

В ответе от этой команды найдите ключ `hostName`. Это значение вам потребуется позже. `hostName` — это DNS-имя созданного ресурса Azure Front Door.

## <a name="create-an-azure-waf-profile-to-use-with-azure-front-door-resources"></a>Создание профиля Azure WAF, который будет использоваться с ресурсами Azure Front Door

```azurecli-interactive 
az network front-door waf-policy create --name <>  --resource-group <>  --disabled false --mode Prevention
```

`--name`: имя новой политики Azure WAF.

`--resource-group`: группа ресурсов, в которую нужно поместить ресурс WAF. 

Предыдущий код для командной строки создаст политику WAF, включит ее и установит в режим предотвращения. 

> [!NOTE] 
> Возможно, вы предпочтете создать политику WAF в режиме обнаружения, чтобы сначала проверить, как она обнаруживает и регистрирует вредоносные запросы (не блокируя их), прежде чем переходить в режим защиты.

В ответе от этой команды найдите ключ `ID`. Это значение вам потребуется позже. 

В поле `ID` должно быть значение в следующем формате:

/subscriptions/**subscription id**/resourcegroups/**resource group name**/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/**WAF policy name**

## <a name="add-managed-rule-sets-to-the-waf-policy"></a>Добавление управляемых наборов правил в политику WAF

В политику WAF вы можете добавить управляемые наборы правил. Управляемый набор создается и управляется корпорацией Майкрософт для защиты от определенного класса угроз. В нашем примере добавляются следующие два набора правил:
- Набор правил по умолчанию защищает от многих распространенных веб-угроз. 
- Набор правил защиты от ботов предотвращает деятельность вредоносных ботов.

Добавьте набор правил по умолчанию:

   ```azurecli-interactive 
   az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type DefaultRuleSet --version 1.0
   ```

Добавьте набор правил защиты от ботов:

   ```azurecli-interactive 
   az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type Microsoft_BotManagerRuleSet --version 1.0
   ```

`--policy-name`: имя, указанное ранее для ресурса Azure WAF.

`--resource-group`: группа ресурсов, в которую вы поместили ресурс WAF.

## <a name="associate-the-waf-policy-with-the-azure-front-door-resource"></a>Привязка политики WAF к ресурсу Azure Front Door

На этом шаге мы свяжем политику WAF, созданную для ресурса Azure Front Door, который размещен перед веб-приложением:

```azurecli-interactive 
az network front-door update --name <> --resource-group <> --set frontendEndpoints[0].webApplicationFirewallPolicyLink='{"id":"<>"}'
```

`--name`: имя, указанное ранее для ресурса Azure Front Door.

`--resource-group`: группа ресурсов, в которую вы поместили ресурс Azure Front Door.

`--set`: здесь вы сохраняете новую политику WAF в атрибуте `WebApplicationFirewallPolicyLink` для конечной точки `frontendEndpoint`, которая связана с ресурсом Azure Front Door. Идентификатор политики WAF вы получили из ответа, когда создавался профиль WAF.

 > [!NOTE] 
> Приведенный выше пример относится к сценарию без использования личного домена. Если вы не используете личные домены для доступа к веб-приложениям, следующий раздел можно пропустить. В этом случае передайте клиентам значение `hostName`, полученное при создании ресурса Azure Front Door. Они будут использовать это значение `hostName` для входа в веб-приложение.

## <a name="configure-the-custom-domain-for-your-web-application"></a>Настройка личного домена для веб-приложения

Имя личного домена для веб-приложения выполняет роль адреса приложения, по которому его находят клиенты. Например, www.contoso.com. Изначально это имя личного домена указывало на расположение, настроенное до включения Azure Front Door. После добавления Azure Front Door и WAF перед этим приложением запись DNS для личного домена должна указывать на новый ресурс Azure Front Door. Для этого измените привязку записи на DNS-сервере на имя `hostName` ресурса Azure Front Door, который вы получили ранее при создании этого ресурса.

Шаги по изменению записей DNS будут разными в зависимости от используемого поставщика услуг DNS. Если для размещения доменного имени вы используете службу Azure DNS, см. документацию [по изменению записей DNS](../dns/dns-operations-recordsets-cli.md) и укажите имя узла `hostName` ресурса Azure Front Door. 

Если вам нужно, чтобы ваши клиенты могли переходить на веб-сайт с помощью вершины зоны (например, contoso.com), необходимо обратить внимание на одну важную вещь. В этом сценарии необходимо использовать для размещения имени DNS службу Azure DNS и [тип записи "псевдоним" (alias)](../dns/dns-alias.md). 

Также в конфигурацию Azure Front Door нужно [добавить личный домен](./front-door-custom-domain.md), чтобы служба знала об этом сопоставлении.

Наконец, если вы используете для доступа к веб-приложению личный домен и хотите включить поддержку протокола HTTPS, вам нужно [настроить сертификаты для личного домена в Azure Front Door](./front-door-custom-domain-https.md). 

## <a name="lock-down-your-web-application"></a>Блокировка веб-приложения

Мы рекомендуем сделать так, чтобы только граничные узлы Azure Front Door могли обращаться к вашему веб-приложению. Так вы будете уверены, что никто не сможет обойти защиту Azure Front Door и напрямую обращаться к приложению. Для настройки блокировки воспользуйтесь разделом [Как сделать, чтобы к моему серверу имела доступ только служба Azure Front Door?](./front-door-faq.yml#how-do-i-lock-down-the-access-to-my-backend-to-only-azure-front-door-)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы, созданные при работе с этим руководством, больше не нужны, выполните команду [az group delete](/cli/azure/group#az_group_delete), чтобы удалить группу ресурсов, Front Door и политику WAF.

```azurecli-interactive
  az group delete \
    --name <>
```
`--name`: имя группы ресурсов для всех ресурсов, используемых в этом руководстве.

## <a name="next-steps"></a>Дальнейшие действия

Изучите следующие руководства по устранению неполадок с Front Door:

> [!div class="nextstepaction"]
> [Устранение распространенных проблем маршрутизации](front-door-troubleshoot-routing.md)
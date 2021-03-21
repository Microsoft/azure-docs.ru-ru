---
title: Увеличение квоты конечной точки — LUIS
titleSuffix: Azure Cognitive Services
description: Служба "Распознавание речи" (LUIS) позволяет увеличить квоту запросы конечной точки за пределы квоты для одного ключа. Для этого создайте дополнительные ключи для LUIS и добавьте их в приложение LUIS на странице **Публикация** в разделе **Ресурсы и ключи**.
manager: nitinme
ms.custom: seodec18, devx-track-js, devx-track-azurepowershell
services: cognitive-services
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 08/20/2019
ms.openlocfilehash: 37c4bd2af080a76e93bc9599f06e4d502985979f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102609657"
---
# <a name="use-microsoft-azure-traffic-manager-to-manage-endpoint-quota-across-keys"></a>Использование диспетчера трафика Microsoft Azure для распределения квоты конечной точки на несколько ключей
Служба "Распознавание речи" (LUIS) позволяет увеличить квоту запросы конечной точки за пределы квоты для одного ключа. Для этого создайте дополнительные ключи для LUIS и добавьте их в приложение LUIS на странице **Публикация** в разделе **Ресурсы и ключи**.

Клиентское приложение должно управлять трафиком между ключами. LUIS не делает этого.

В этой статье описано, как управлять трафиком между ключами с помощью [диспетчера трафика Azure][traffic-manager-marketing]. Вам потребуется обученное и опубликованное приложение LUIS. Если у вас его нет, воспользуйтесь [руководством по началу работы](luis-get-started-create-app.md) с предварительно созданными доменами.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="connect-to-powershell-in-the-azure-portal"></a>Подключение к PowerShell на портале Azure
На портале [Azure][azure-portal] откройте окно PowerShell. Значок окна PowerShell — **>_** в верхней панели навигации. Используя PowerShell на портале, вы получаете последнюю версию PowerShell и автоматически проходите проверку подлинности. Для использования PowerShell на портале требуется [учетная запись хранения Azure](https://azure.microsoft.com/services/storage/).

![Снимок экрана портала Azure с открытым окном PowerShell](./media/traffic-manager/azure-portal-powershell.png)

В следующих разделах используйте [командлеты PowerShell диспетчера трафика](/powershell/module/az.trafficmanager/#traffic_manager).

## <a name="create-azure-resource-group-with-powershell"></a>Создание группы ресурсов Azure с помощью PowerShell
Прежде чем создавать ресурсы Azure, создайте группу ресурсов, в которую будут входить все ресурсы. Укажите имя группы ресурсов `luis-traffic-manager` и регион `West US`. В регионе группы ресурсов хранятся метаданные о группе. Если ваши ресурсы находятся в другом регионе, это не скажется на скорости их работы.

Создайте группу ресурсов с помощью командлета **[New-азресаурцеграуп](/powershell/module/az.resources/new-azresourcegroup)** :

```powerShell
New-AzResourceGroup -Name luis-traffic-manager -Location "West US"
```

## <a name="create-luis-keys-to-increase-total-endpoint-quota"></a>Создание ключей LUIS для увеличения общей квоты конечной точки
1. На портале Azure создайте два ключа службы **Распознавание речи**: в `West US` и `East US`. Используйте существующую группу ресурсов `luis-traffic-manager`, созданную в предыдущем разделе.

    ![Снимок экрана портала Azure с двумя ключами LUIS в группе ресурсов luis-traffic-manager](./media/traffic-manager/luis-keys.png)

2. На веб-сайте [Luis][LUIS] в разделе **Управление** на странице **ресурсы Azure** назначьте ключи приложению и повторно опубликуйте приложение, нажав кнопку **опубликовать** в правом верхнем меню.

    В примере URL-адреса в столбце **Конечная точка** используется запрос GET с ключом конечной точки в качестве параметра запроса. Скопируйте URL-адреса конечной точки для двух новых ключей. Они используются для настройки диспетчера трафика далее в этой статье.

## <a name="manage-luis-endpoint-requests-across-keys-with-traffic-manager"></a>Распределение запросов к конечным точкам LUIS между ключами с помощью диспетчера трафика
Диспетчер трафика создает новую точку доступа DNS для ваших конечных точек. Она действует не на уровне шлюза или прокси, а исключительно на уровне DNS. В этом примере не изменяются записи DNS. Библиотека DNS используется для взаимодействия с диспетчером трафика, чтобы получить конечную точку для указанного запроса. Для выполнения _каждого_ запроса LUIS сначала необходимо отправить запрос к диспетчеру трафика, чтобы определить используемую конечную точку LUIS.

### <a name="polling-uses-luis-endpoint"></a>Опрос с использованием конечной точки LUIS
Диспетчер трафика периодически опрашивает конечные точки, чтобы проверить, что конечная точка по-прежнему доступна. URL-адрес, опрашиваемый диспетчером трафика, должен быть доступен с помощью запроса GET и должен возвращать ответ 200. Этим условиям отвечает URL-адрес конечной точки на странице **Публикация**. Поскольку у каждого ключа конечной точки есть свой маршрут и параметры строки запроса, для каждого ключа конечной точки требуется использовать собственный путь опроса. Каждый раз, когда диспетчер трафика выполняет опрос, он использует запрос стоимости квоты. Параметр строки запроса **q** конечной точки LUIS представляет собой высказывание, отправляемое в LUIS. Этот параметр используется не для отправки высказывания, а для добавления опроса диспетчера трафика в журнал конечной точки LUIS с целью отладки при настройке диспетчера трафика.

Так как у каждой конечной точки LUIS есть собственный путь, ей требуется собственный профиль Traffic Manager. Чтобы управлять профилями, создайте [_вложенную_ архитектуру диспетчера трафика](../../traffic-manager/traffic-manager-nested-profiles.md). Один родительский профиль указывает на дочерние профили и управляет трафиком между ними.

После настройки диспетчера трафика не забудьте изменить путь, указав параметр строки запроса logging=false, чтобы в журнал не записывались сведения об опросе.

## <a name="configure-traffic-manager-with-nested-profiles"></a>Настройка диспетчера трафика с использованием вложенных профилей
В следующих разделах будут созданы два дочерних профиля: один для ключа LUIS в регионе "Восточная часть США" и другой для ключа LUIS в регионе "Западная часть США". Затем будет создан родительский профиль, и оба дочерних профиля будут добавлены в родительский.

### <a name="create-the-east-us-traffic-manager-profile-with-powershell"></a>Создание профиля диспетчера трафика для региона "Восточная часть США" с помощью PowerShell
Чтобы создать профиль для региона "Восточная часть США", необходимо выполнить несколько действий: создать профиль, добавить конечную точку и настроить конечную точку. Профиль диспетчера трафика может включать несколько конечных точек, но у каждой конечной точки один и тот же путь проверки. Поскольку URL-адреса конечных точек LUIS подписок для регионов "Западная часть США" и "Восточная часть США" различаются по региону и ключу конечной точки, то каждая конечная точка LUIS должна быть единственной конечной точкой в профиле.

1. Создание профиля с помощью командлета **[New-азтраффикманажерпрофиле](/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile)**

    Для создания профиля воспользуйтесь указанным ниже командлетом. Не забудьте изменить значения параметров `appIdLuis` и `subscriptionKeyLuis`. Ключ подписки указан для региона LUIS "Восточная часть США". Если путь, включая идентификатор приложения и ключ конечной точки, является неправильным, то результат опроса диспетчера трафика получает состояние `degraded`, так как диспетчер трафика не может успешно опросить конечную точку LUIS. Убедитесь, что параметр `q` имеет значение `traffic-manager-east`, которое можно будет найти в журналах конечной точки LUIS.

    ```powerShell
    $eastprofile = New-AzTrafficManagerProfile -Name luis-profile-eastus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-eastus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appID>?subscription-key=<subscriptionKey>&q=traffic-manager-east"
    ```

    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-Name|luis-profile-eastus|Имя диспетчера трафика на портале Azure|
    |-ResourceGroupName|luis-traffic-manager|Создан в предыдущем разделе|
    |-TrafficRoutingMethod|Производительность|Дополнительные сведения см. в статье [Методы маршрутизации диспетчера трафика][routing-methods]. При использовании метода маршрутизации по производительности запрос URL-адреса для диспетчера трафика должен поступать из региона пользователя. Если запрос формируется с помощью чат-бота или другого приложения, то чат-бот или другое приложение должны установить соответствующий регион в вызове диспетчера трафика. |
    |-RelativeDnsName|luis-dns-eastus|Это дочерний домен службы: luis-dns-eastus.trafficmanager.net|
    |-Ttl|30|Интервал опроса, 30 секунд|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|Порт и протокол для LUIS — HTTPS-порт 443|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-east`|Замените `<appIdLuis>` и `<subscriptionKeyLuis>` собственными значениями.|

    Для успешного запроса ответ не возвращается.

2. Добавление конечной точки восточной части США с помощью командлета **[Add-азтраффикманажерендпоинтконфиг](/powershell/module/az.trafficmanager/add-aztrafficmanagerendpointconfig)**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName luis-east-endpoint -TrafficManagerProfile $eastprofile -Type ExternalEndpoints -Target eastus.api.cognitive.microsoft.com -EndpointLocation "eastus" -EndpointStatus Enabled
    ```
    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-EndpointName|luis-east-endpoint|Имя конечной точки в профиле|
    |-TrafficManagerProfile|$eastprofile|Используйте объект профиля, созданный на шаге 1|
    |-Type|ExternalEndpoints|Дополнительные сведения см. в статье [Конечная точка диспетчера трафика][traffic-manager-endpoints]. |
    |-Target|eastus.api.cognitive.microsoft.com|Домен LUIS конечной точки.|
    |-EndpointLocation|"eastus"|Регион конечной точки.|
    |-EndpointStatus|Активировано|Включить конечную точку при ее создании.|

    Успешный ответ выглядит следующим образом:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-eastus
    Name                             : luis-profile-eastus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-eastus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/<luis-app-id>?subscription-key=f0517d185bcf467cba5147d6260bb868&q=traffic-manager-east
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-east-endpoint}
    ```

3. Задание конечной точки восточной части США с помощью командлета **[Set-азтраффикманажерпрофиле](/powershell/module/az.trafficmanager/set-aztrafficmanagerprofile)**

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $eastprofile
    ```

    Успешный ответ аналогичен успешному ответу на шаге 2.

### <a name="create-the-west-us-traffic-manager-profile-with-powershell"></a>Создание профиля диспетчера трафика для региона "Западная часть США" с помощью PowerShell
Чтобы создать профиль для региона "Западная часть США", необходимо выполнить те же действия: создать профиль, добавить конечную точку и настроить конечную точку.

1. Создание профиля с помощью командлета **[New-азтраффикманажерпрофиле](/powershell/module/az.TrafficManager/New-azTrafficManagerProfile)**

    Для создания профиля воспользуйтесь указанным ниже командлетом. Не забудьте изменить значения параметров `appIdLuis` и `subscriptionKeyLuis`. Ключ подписки указан для региона LUIS "Восточная часть США". Если путь, включая идентификатор приложения и ключ конечной точки, является неправильным, то результат опроса диспетчера трафика получает состояние `degraded`, так как диспетчер трафика не может успешно опросить конечную точку LUIS. Убедитесь, что параметр `q` имеет значение `traffic-manager-west`, которое можно будет найти в журналах конечной точки LUIS.

    ```powerShell
    $westprofile = New-AzTrafficManagerProfile -Name luis-profile-westus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-westus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west"
    ```

    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-Name|luis-profile-westus|Имя диспетчера трафика на портале Azure|
    |-ResourceGroupName|luis-traffic-manager|Создан в предыдущем разделе|
    |-TrafficRoutingMethod|Производительность|Дополнительные сведения см. в статье [Методы маршрутизации диспетчера трафика][routing-methods]. При использовании метода маршрутизации по производительности запрос URL-адреса для диспетчера трафика должен поступать из региона пользователя. Если запрос формируется с помощью чат-бота или другого приложения, то чат-бот или другое приложение должны установить соответствующий регион в вызове диспетчера трафика. |
    |-RelativeDnsName|luis-dns-westus|Это дочерний домен службы: luis-dns-westus.trafficmanager.net|
    |-Ttl|30|Интервал опроса, 30 секунд|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|Порт и протокол для LUIS — HTTPS-порт 443|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west`|Замените `<appId>` и `<subscriptionKey>` собственными значениями. Помните, что этот ключ конечной точки отличается от ключа конечной точки в регионе "Восточная часть США"|

    Для успешного запроса ответ не возвращается.

2. Добавление конечной точки "Западная часть США" с помощью командлета **[Add-азтраффикманажерендпоинтконфиг](/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName luis-west-endpoint -TrafficManagerProfile $westprofile -Type ExternalEndpoints -Target westus.api.cognitive.microsoft.com -EndpointLocation "westus" -EndpointStatus Enabled
    ```

    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-EndpointName|luis-west-endpoint|Имя конечной точки в профиле|
    |-TrafficManagerProfile|$westprofile|Используйте объект профиля, созданный на шаге 1|
    |-Type|ExternalEndpoints|Дополнительные сведения см. в статье [Конечная точка диспетчера трафика][traffic-manager-endpoints]. |
    |-Target|westus.api.cognitive.microsoft.com|Домен LUIS конечной точки.|
    |-EndpointLocation|"westus"|Регион конечной точки.|
    |-EndpointStatus|Активировано|Включить конечную точку при ее создании.|

    Успешный ответ выглядит следующим образом:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-westus
    Name                             : luis-profile-westus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-westus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/c3fc5d1e-5187-40cc-af0f-fbde328aa16b?subscription-key=e3605f07e3cc4bedb7e02698a54c19cc&q=traffic-manager-west
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-west-endpoint}
    ```

3. Настройка конечной точки западной части США с помощью командлета **[Set-азтраффикманажерпрофиле](/powershell/module/az.TrafficManager/Set-azTrafficManagerProfile)**

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $westprofile
    ```

    Успешный ответ аналогичен успешному ответу на шаге 2.

### <a name="create-parent-traffic-manager-profile"></a>Создание родительского профиля диспетчера трафика
Создайте родительский профиль диспетчера трафика и свяжите два дочерних профиля диспетчера трафика с родительским.

1. Создание родительского профиля с помощью командлета **[New-азтраффикманажерпрофиле](/powershell/module/az.TrafficManager/New-azTrafficManagerProfile)**

    ```powerShell
    $parentprofile = New-AzTrafficManagerProfile -Name luis-profile-parent -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-parent -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/"
    ```

    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-Name|luis-profile-parent|Имя диспетчера трафика на портале Azure|
    |-ResourceGroupName|luis-traffic-manager|Создан в предыдущем разделе|
    |-TrafficRoutingMethod|Производительность|Дополнительные сведения см. в статье [Методы маршрутизации диспетчера трафика][routing-methods]. При использовании метода маршрутизации по производительности запрос URL-адреса для диспетчера трафика должен поступать из региона пользователя. Если запрос формируется с помощью чат-бота или другого приложения, то чат-бот или другое приложение должны установить соответствующий регион в вызове диспетчера трафика. |
    |-RelativeDnsName|luis-dns-parent|Это дочерний домен службы: luis-dns-parent.trafficmanager.net|
    |-Ttl|30|Интервал опроса, 30 секунд|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|Порт и протокол для LUIS — HTTPS-порт 443|
    |-MonitorPath|`/`|Этот путь не имеет значения, так как вместо него используются пути к дочерним конечным точкам.|

    Для успешного запроса ответ не возвращается.

2. Добавление дочернего профиля восточной части США в родительский профиль с помощью **[Add-азтраффикманажерендпоинтконфиг](/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** и типа **нестедендпоинтс**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint-useast -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $eastprofile.Id -EndpointStatus Enabled -EndpointLocation "eastus" -MinChildEndpoints 1
    ```

    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-EndpointName|child-endpoint-useast|Профиль для региона "Восточная часть США"|
    |-TrafficManagerProfile|$parentprofile|Профиль, которому будет назначена эта конечная точка|
    |-Type|NestedEndpoints|Дополнительные сведения см. в разделе [Add-азтраффикманажерендпоинтконфиг](/powershell/module/az.trafficmanager/Add-azTrafficManagerEndpointConfig). |
    |-TargetResourceId|$eastprofile.Id|Идентификатор дочернего профиля|
    |-EndpointStatus|Активировано|Состояние конечной точки после добавления в родительский профиль|
    |-EndpointLocation|"eastus"|[Имя региона Azure](https://azure.microsoft.com/global-infrastructure/regions/) ресурса.|
    |-MinChildEndpoints|1|Минимальное число дочерних конечных точек|

    Успешный ответ будет похож на приведенный ниже и будет включать новую конечную точку `child-endpoint-useast`:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast}
    ```

3. Добавление дочернего профиля "Западная часть США" в родительский с помощью командлета **[Add-азтраффикманажерендпоинтконфиг](/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** и типа **нестедендпоинтс**

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint-uswest -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $westprofile.Id -EndpointStatus Enabled -EndpointLocation "westus" -MinChildEndpoints 1
    ```

    В таблице ниже показаны значения каждой переменной в командлете:

    |Параметр конфигурации|Имя переменной или значение|Назначение|
    |--|--|--|
    |-EndpointName|child-endpoint-uswest|Профиль для региона "Западная часть США"|
    |-TrafficManagerProfile|$parentprofile|Профиль, которому будет назначена эта конечная точка|
    |-Type|NestedEndpoints|Дополнительные сведения см. в разделе [Add-азтраффикманажерендпоинтконфиг](/powershell/module/az.trafficmanager/Add-azTrafficManagerEndpointConfig). |
    |-TargetResourceId|$westprofile.Id|Идентификатор дочернего профиля|
    |-EndpointStatus|Активировано|Состояние конечной точки после добавления в родительский профиль|
    |-EndpointLocation|"westus"|[Имя региона Azure](https://azure.microsoft.com/global-infrastructure/regions/) ресурса.|
    |-MinChildEndpoints|1|Минимальное число дочерних конечных точек|

    Успешный ответ будет похож на приведенный ниже и будет включать как предыдущую конечную точку `child-endpoint-useast`, так и новую конечную точку `child-endpoint-uswest`:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast, child-endpoint-uswest}
    ```

4. Установка конечных точек с помощью командлета **[Set-азтраффикманажерпрофиле](/powershell/module/az.TrafficManager/Set-azTrafficManagerProfile)**

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $parentprofile
    ```

    Успешный ответ аналогичен успешному ответу на шаге 3.

### <a name="powershell-variables"></a>Переменные PowerShell
В предыдущих разделах были созданы три переменные PowerShell: `$eastprofile`, `$westprofile`, `$parentprofile`. Эти переменные используются на конечном этапе настройки диспетчера трафика. Если вы решили не создавать переменные или забыли, или окно PowerShell истечет, можно использовать командлет PowerShell **[Get-азтраффикманажерпрофиле](/powershell/module/az.TrafficManager/Get-azTrafficManagerProfile)**, чтобы снова получить профиль и назначить его переменной.

Замените элементы в угловых скобках (`<>`) на соответствующие значения для каждого из трех профилей.

```powerShell
$<variable-name> = Get-AzTrafficManagerProfile -Name <profile-name> -ResourceGroupName luis-traffic-manager
```

## <a name="verify-traffic-manager-works"></a>Проверка работы диспетчера трафика
О работоспособности профилей диспетчера трафика свидетельствует состояние `Online`. Это состояние определяется на основе пути опроса конечной точки.

### <a name="view-new-profiles-in-the-azure-portal"></a>Просмотр новых профилей на портале Azure
Чтобы убедиться, что все три профиля созданы, посмотрите на ресурсы в группе ресурсов `luis-traffic-manager`.

![Снимок экрана группы ресурсов Azure luis-traffic-manager](./media/traffic-manager/traffic-manager-profiles.png)

### <a name="verify-the-profile-status-is-online"></a>Проверка состояния профиля "В сети"
Диспетчер трафика опрашивает путь к каждой из конечных точек, чтобы убедиться, что она находится в рабочем режиме. Если она находится в рабочем режиме, дочерние профили находятся в состоянии `Online`. Это состояние отображается на странице **Обзор** каждого профиля.

![Снимок экрана страницы "Обзор" профиля диспетчера трафика с состоянием профиля "В сети"](./media/traffic-manager/profile-status-online.png)

### <a name="validate-traffic-manager-polling-works"></a>Проверка опроса диспетчера трафика
Еще один способ проверить работу опросов диспетчера трафика — журналы конечных точек LUIS. На странице со списком приложений веб-сайта [LUIS][LUIS] экспортируйте журнал конечной точки для приложения. Поскольку диспетчер трафика часто опрашивает две конечные точки, в журналах будут записи с опросом этих конечных точек, даже если они работали всего несколько минут. Найдите записи, запросы для которых начинаются с `traffic-manager-`.

```console
traffic-manager-west    6/7/2018 19:19  {"query":"traffic-manager-west","intents":[{"intent":"None","score":0.944767}],"entities":[]}
traffic-manager-east    6/7/2018 19:20  {"query":"traffic-manager-east","intents":[{"intent":"None","score":0.944767}],"entities":[]}
```

### <a name="validate-dns-response-from-traffic-manager-works"></a>Проверка ответа DNS от диспетчера трафика
Чтобы проверить, что ответ DNS возвращает конечную точку LUIS, отправьте запрос к родительскому профилю DNS диспетчера трафика с помощью клиентской библиотеки DNS. Имя DNS родительского профиля — `luis-dns-parent.trafficmanager.net`.

В следующем примере кода Node.js отправляется запрос к родительскому профилю, и возвращается конечная точка LUIS:

```javascript
const dns = require('dns');

dns.resolveAny('luis-dns-parent.trafficmanager.net', (err, ret) => {
  console.log('ret', ret);
});
```

Успешный ответ для конечной точки LUIS:

```json
[
    {
        value: 'westus.api.cognitive.microsoft.com',
        type: 'CNAME'
    }
]
```

## <a name="use-the-traffic-manager-parent-profile"></a>Использование родительского профиля диспетчера трафика
Чтобы управлять трафиком между конечными точками, необходимо добавить вызов DNS диспетчера трафика для поиска конечной точки LUIS. Этот вызов выполняется для каждого запроса LUIS к конечной точке и должен имитировать географическое расположение пользователя клиентского приложения LUIS. Добавьте код ответа DNS между клиентским приложением LUIS и запросом к LUIS для определения конечной точки.

## <a name="resolving-a-degraded-state"></a>Разрешение состояния пониженной функциональности

Включите [журналы диагностики](../../traffic-manager/traffic-manager-diagnostic-logs.md) для диспетчера трафика, чтобы узнать, почему конечная точка находится в состоянии пониженной функциональности.

## <a name="clean-up"></a>Очистка
Удалите два ключа конечной точки LUIS, три профиля диспетчера трафика и группу ресурсов, содержащую эти пять ресурсов. Это необходимо сделать на портале Azure. Удалите пять ресурсов из списка ресурсов. Затем удалите группу ресурсов.

## <a name="next-steps"></a>Следующие шаги

Чтобы узнать, как добавить код управления трафиком в бот BotFramework, ознакомьтесь с вариантами [ПО промежуточного слоя](/azure/bot-service/bot-builder-create-middleware?tabs=csaddmiddleware%252ccsetagoverwrite%252ccsmiddlewareshortcircuit%252ccsfallback%252ccsactivityhandler) в BotFramework версии 4.

[traffic-manager-marketing]: https://azure.microsoft.com/services/traffic-manager/
[traffic-manager-docs]: ../../traffic-manager/index.yml
[LUIS]: ./luis-reference-regions.md#luis-website
[azure-portal]: https://portal.azure.com/
[azure-storage]: https://azure.microsoft.com/services/storage/
[routing-methods]: ../../traffic-manager/traffic-manager-routing-methods.md
[traffic-manager-endpoints]: ../../traffic-manager/traffic-manager-endpoint-types.md
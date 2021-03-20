---
title: Развертывание служб управления API Azure в нескольких регионах Azure
titleSuffix: Azure API Management
description: Дополнительные сведения о развертывании экземпляра службы управления Azure API в различных регионах Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 427ebfe865002612be2f9aeb9db416f5c2f41e52
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88065460"
---
# <a name="how-to-deploy-an-azure-api-management-service-instance-to-multiple-azure-regions"></a>Развертывание экземпляра службы управления Azure API в различных регионах Azure

Служба управления API Azure поддерживает развертывание в нескольких регионах, что позволяет издателям API распространять единую службу управления API Azure через любое количество поддерживаемых регионов Azure. Функция с несколькими регионами позволяет сократить задержку запросов, воспринимаемую географически распространенными потребителями API, и повысить доступность службы, если один регион переходит в режим «вне сети».

Новая служба управления API Azure изначально содержит только одну [единицу][unit] в одном регионе Azure, основном регионе. Дополнительные единицы можно добавить в первичный или дополнительный регион. Компонент шлюза управления API развертывается в каждом выбранном основном и дополнительном регионе. Входящие запросы API автоматически направляются в ближайший регион. Если регион переходит в режим «вне сети», запросы API будут автоматически направляться по неисправному региону на следующий ближайший шлюз.

> [!NOTE]
> Во все регионы развертываются только компоненты шлюза управления API. Компонент управления службами и портал разработчика размещаются только в основном регионе. Поэтому в случае сбоя основного региона доступ к порталу разработчика и возможность изменения конфигурации (например, добавление API-интерфейсов, применение политик) будут ограничены до тех пор, пока основной регион не вернется в режим "в сети". В то время как основной регион доступен в автономном режиме, вторичные регионы продолжат обслуживать трафик API, используя последнюю доступную конфигурацию.

>[!IMPORTANT]
> Функция, позволяющая хранить данные клиентов в одном регионе, в настоящее время доступна только в регионе Юго-Восточной Азии (Сингапур) Азиатско-Тихоокеанский регион Geo. Для всех других регионов данные клиента хранятся в геообъектах.

[!INCLUDE [premium.md](../../includes/api-management-availability-premium.md)]

## <a name="deploy-api-management-service-to-a-new-region"></a><a name="add-region"> </a>Развертывание службы управления API в новом регионе

> [!NOTE]
> Если экземпляр службы управления API еще не создан, выполните инструкции в статье [Создание экземпляра службы управления API Azure][create an api management service instance].

1. В портал Azure перейдите к службе управления API и щелкните запись **Locations (расположения** ) в меню.
2. Щелкните **+ Добавить** на верхней панели.
3. Выберите расположение из раскрывающегося списка и задайте число единиц с помощью ползунка.
4. Нажмите кнопку **Добавить** , чтобы подтвердить.
5. Повторите эту процедуру, пока не будут настроены все расположения.
6. Нажмите кнопку **сохранить** на верхней панели, чтобы начать процесс развертывания.

## <a name="delete-an-api-management-service-location"></a><a name="remove-region"> </a>Удаление расположения службы управления API

1. В портал Azure перейдите к службе управления API и щелкните запись **Locations (расположения** ) в меню.
2. Для расположения, которое нужно удалить, откройте контекстное меню с помощью кнопки **…** в правой части таблицы. Щелкните **Удалить**.
3. Подтвердите удаление и нажмите кнопку **Сохранить**, чтобы применить изменения.

## <a name="route-api-calls-to-regional-backend-services"></a><a name="route-backend"> </a>Маршрутизация вызовов API в региональных серверных службах

Служба управления API Azure поддерживает только один URL-адрес внутренней службы. Несмотря на то что экземпляры службы управления API Azure существуют в разных регионах, шлюз API будет по-прежнему перенаправлять запросы к одной и той же внутренней службе, которая развертывается только в один регион. В этом случае выигрыш в производительности будет поступать только из ответов, кэшированных в службе управления API Azure в регионе, указанном для запроса, но обращение к серверной части по всему миру может по-прежнему вызывать большую задержку.

Чтобы полностью использовать географическое распределение системы, необходимо обладать серверными службами, развернутыми в тех же регионах, что и экземпляры службы управления API Azure. Затем с помощью политик и свойства `@(context.Deployment.Region)` можно перенаправлять трафик в локальные экземпляры серверной части.

1. Перейдите к экземпляру службы управления API Azure и в меню слева щелкните **Интерфейсы API**.
2. Выберите нужный API.
3. Щелкните **Редактор кода** в раскрывающемся меню со стрелкой в **Обработке входящих запросов**.

    ![Редактор кода API](./media/api-management-howto-deploy-multi-region/api-management-api-code-editor.png)

4. Используйте условие `set-backend` в сочетании с условными политиками `choose`, чтобы создать подходящую политику маршрутизации в разделе `<inbound> </inbound>` файла.

    Например, следующий файл XML будет работать как для региона "Западная часть США", так и для региона "Восточная Азия".

    ```xml
    <policies>
        <inbound>
            <base />
            <choose>
                <when condition="@("West US".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                    <set-backend-service base-url="http://contoso-us.com/" />
                </when>
                <when condition="@("East Asia".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                    <set-backend-service base-url="http://contoso-asia.com/" />
                </when>
                <otherwise>
                    <set-backend-service base-url="http://contoso-other.com/" />
                </otherwise>
            </choose>
        </inbound>
        <backend>
            <base />
        </backend>
        <outbound>
            <base />
        </outbound>
        <on-error>
            <base />
        </on-error>
    </policies>
    ```

> [!TIP]
> Вы также можете переключить серверные службы с помощью [диспетчера трафика Azure](https://azure.microsoft.com/services/traffic-manager/), направить вызовы API в диспетчер трафика и разрешить маршрутизацию автоматически.

## <a name="use-custom-routing-to-api-management-regional-gateways"></a><a name="custom-routing"> </a>Использование настраиваемой маршрутизации для региональных ШЛЮЗОВ управления API

Управление API направляет запросы на региональный _шлюз_ в соответствии с [наименьшей задержкой](../traffic-manager/traffic-manager-routing-methods.md#performance). Хотя этот параметр нельзя переопределить в службе управления API, можно использовать собственный диспетчер трафика с пользовательскими правилами маршрутизации.

1. Создайте собственный [диспетчер трафика Azure](https://azure.microsoft.com/services/traffic-manager/).
1. Если вы используете личный домен, [используйте его с диспетчером трафика](../traffic-manager/traffic-manager-point-internet-domain.md) вместо службы управления API.
1. [Настройте региональные конечные точки управления API в диспетчере трафика](../traffic-manager/traffic-manager-manage-endpoints.md). Региональные конечные точки соответствуют шаблону URL-адреса `https://<service-name>-<region>-01.regional.azure-api.net` , например `https://contoso-westus2-01.regional.azure-api.net` .
1. [Настройка конечных точек регионального состояния управления API в диспетчере трафика](../traffic-manager/traffic-manager-monitoring.md). Конечные точки регионального состояния следуют шаблону URL-адреса `https://<service-name>-<region>-01.regional.azure-api.net/status-0123456789abcdef` , например `https://contoso-westus2-01.regional.azure-api.net/status-0123456789abcdef` .
1. Укажите [метод маршрутизации](../traffic-manager/traffic-manager-routing-methods.md) диспетчера трафика.

[create an api management service instance]: get-started-create-service-instance.md
[get started with azure api management]: get-started-create-service-instance.md
[deploy an api management service instance to a new region]: #add-region
[delete an api management service instance from a region]: #remove-region
[unit]: https://azure.microsoft.com/pricing/details/api-management/
[premium]: https://azure.microsoft.com/pricing/details/api-management/

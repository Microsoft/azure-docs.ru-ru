---
title: 'Azure ExpressRoute: Настройка Direct для ExpressRoute: портал'
description: Эта страница поможет настроить ExpressRoute Direct с помощью портала.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/14/2020
ms.author: duau
ms.openlocfilehash: b133f1cce4af07d8d5e50e04670741fcf7c936a4
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102097080"
---
# <a name="create-expressroute-direct-using-the-azure-portal"></a>Создание ExpressRoute Direct с помощью портал Azure

В этой статье показано, как создать ExpressRoute Direct с помощью портал Azure.
Служба ExpressRoute Direct позволяет напрямую подключаться к глобальной сети Майкрософт в местах пиринга, стратегически распределенных по всему миру. См. дополнительные сведения об [ExpressRoute Direct](expressroute-erdirect-about.md).

## <a name="before-you-begin"></a><a name="before"></a>Подготовка к работе

Прежде чем использовать ExpressRoute Direct, сначала необходимо зарегистрировать подписку. Для регистрации выполните следующие действия с помощью Azure PowerShell.
1.  Войдите в Azure и выберите подписку, которую хотите зарегистрировать.

    ```azurepowershell-interactive
    Connect-AzAccount 

    Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
    ```

2. Зарегистрируйте подписку для общедоступной предварительной версии с помощью следующей команды:
    ```azurepowershell-interactive
    Register-AzProviderFeature -FeatureName AllowExpressRoutePorts -ProviderNamespace Microsoft.Network
    ```

После регистрации убедитесь, что поставщик ресурсов **Microsoft. Network** зарегистрирован в вашей подписке. Регистрация поставщика ресурсов настраивает подписку для работы с поставщиком ресурсов.

1. Получите доступ к параметрам подписки, как описано в разделе [поставщики и типы ресурсов Azure](../azure-resource-manager/management/resource-providers-and-types.md).
1. В подписке для **поставщиков ресурсов** убедитесь, что поставщик **Microsoft. Network** отображает **зарегистрированное** состояние. Если поставщик ресурсов Microsoft. Network отсутствует в списке зарегистрированных поставщиков, добавьте его.

## <a name="create-expressroute-direct"></a><a name="create-erdir"></a>Создание прямого канала ExpressRoute

1. В меню [портал Azure](https://portal.azure.com) или на **домашней** странице выберите **создать ресурс**.

1. На **новой** странице в **поле _Поиск в Marketplace_*_ введите _* ExpressRoute Direct**, а затем нажмите клавишу **Ввод** , чтобы получить результаты поиска.

1. В результатах выберите **ExpressRoute Direct**.

1. На странице **ExpressRoute Direct** нажмите кнопку **создать** , чтобы открыть страницу **Создание прямой страницы expressroute** .

1. Начните с заполнения полей на странице **Основные сведения** .

    :::image type="content" source="./media/how-to-expressroute-direct-portal/basics.png" alt-text="Страница &quot;Основные&quot;":::

    * **Подписка**. Подписка Azure, которую вы хотите использовать для создания нового канала ExpressRoute Direct. Ресурс ExpressRoute Direct и каналы ExpressRoute должны находиться в одной подписке.
    * **Группа ресурсов**. Группа ресурсов Azure, в которой будет создан новый прямой ресурс ExpressRoute. Создайте группу ресурсов, если у вас нет существующей группы ресурсов.
    * **Регион**. Общедоступный регион Azure, в котором будет создан ресурс.
    * **Имя**: имя нового прямого ресурса ExpressRoute.

1. Затем заполните поля на странице **Конфигурация** .

    :::image type="content" source="./media/how-to-expressroute-direct-portal/configuration.png" alt-text="Снимок экрана, на котором показана страница &quot;создание прямой ExpressRoute&quot; с выбранной вкладкой &quot;Конфигурация&quot;.":::

    * **Расположение пиринга**. расположение пиринга, в котором будет подключаться к ресурсу непосредственных подключений ExpressRoute. Дополнительные сведения о расположении пиринга см. в статье [расположения ExpressRoute](expressroute-locations-providers.md).
   * **Пропускная** способность. пропускная способность для пары портов, которую требуется зарезервировать. ExpressRoute Direct поддерживает параметры пропускной способности 10 ГБ и 100 ГБ. Если требуемая пропускная способность недоступна в указанном расположении пиринга, [откройте запрос в службу поддержки в портал Azure](https://aka.ms/azsupt).
   * **Инкапсуляция**. ExpressRoute Direct поддерживает инкапсуляцию QinQ и Dot1Q.
     * Если выбран QinQ, каждому каналу ExpressRoute динамически назначается S-тег и каналы будут уникальными в пределах ресурса ExpressRoute Direct.
     *  Каждый C-тег в канале должен быть уникальным в пределах канала, но не для всего ExpressRoute Direct.
     * Если выбрана инкапсуляция Dot1Q, необходимо достичь уникальности C-тега (VLAN) в пределах всего ресурса ExpressRoute Direct.
     >[!IMPORTANT]
     >ExpressRoute Direct может иметь только один тип инкапсуляции. После создания ExpressRoute Direct изменить инкапсуляцию невозможно.
     >

1. Укажите Теги ресурсов, а затем выберите **проверить и создать** , чтобы проверить параметры прямого ресурса ExpressRoute.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/validate.png" alt-text="Снимок экрана, на котором показана страница &quot;Создание ExpressRoute&quot; с выбранной вкладкой &quot;Проверка и создание&quot;.":::

1. Нажмите кнопку **Создать**. Отобразится сообщение о том, что развертывание выполняется. Состояние будет отображаться на этой странице при создании ресурсов. 

## <a name="generate-the-letter-of-authorization-loa"></a><a name="authorization"></a>Создание письма авторизации (определения гарантии)

В настоящее время создание письма авторизации на портале недоступно. Чтобы получить письмо авторизации, используйте **[Azure PowerShell](expressroute-howto-erdirect.md#authorization)** .

## <a name="change-admin-state-of-links"></a><a name="state"></a>Изменение параметра AdminState для ссылок

Этот процесс следует использовать для проведения тестирования уровня 1, что в свою очередь обеспечит правильное исправление перекрестных подключений в каждом маршрутизаторе для основного и дополнительного подключения.

1. На странице **Обзор** прямого ресурса ExpressRoute в разделе **ссылки** выберите **link1**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/link.png" alt-text="Link 1" lightbox="./media/how-to-expressroute-direct-portal/link-expand.png":::

1. Переключите параметр **состояния администратора** в значение **включено**, а затем нажмите кнопку **сохранить**.

    :::image type="content" source="./media/how-to-expressroute-direct-portal/state.png" alt-text="Состояние администратора":::

    >[!IMPORTANT]
    >Выставление счетов начнется, когда в любой из связей будет включено состояние администратора.
    >

1. Повторите тот же процесс для **Link2**.

## <a name="create-a-circuit"></a><a name="circuit"></a>Создание цепи

По умолчанию можно создать до 10 каналов в подписке, в которой есть ресурс ExpressRoute Direct. Это число можно увеличить по поддержке. Вы несете ответственность за отслеживание как подготовленной, так и используемой пропускной способности. Подготовленная пропускная способность — это сумма пропускной способности всех цепей в прямом ресурсе ExpressRoute. Используемая пропускная способность описывает фактическое потребление базовых физических интерфейсов.

* Существуют дополнительные пропускные способности каналов, которые могут использоваться на ExpressRoute Direct только для поддержки сценариев, описанных выше. Это: 40 Гбит/с 100 Гбит/с.

* Скутиер может быть локальным, стандартным или Premium.

* Скуфамили должен быть только MeteredData. Неограниченное число не поддерживается в ExpressRoute Direct.

Следующие шаги помогут создать канал ExpressRoute из потока операций ExpressRoute Direct. Если вы бы хотели, вы также можете создать канал с помощью потока операций по стандартному каналу, хотя в этом случае нет никаких преимуществ использования обычных действий потока операций для этой конфигурации. См. раздел [Создание и изменение канала ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md).

1. В разделе **Параметры** прямого подключения к ExpressRoute выберите **каналы**, а затем щелкните **+ добавить**. 

    :::image type="content" source="./media/how-to-expressroute-direct-portal/add.png" alt-text="На снимке экрана показаны параметры ExpressRoute с выбранными каналами и добавлены выделенными." lightbox="./media/how-to-expressroute-direct-portal/add-expand.png":::

1. Настройте параметры на странице **Конфигурация** .

   :::image type="content" source="./media/how-to-expressroute-direct-portal/configuration2.png" alt-text="Страница &quot;Конфигурация&quot; — Direct (ExpressRoute)":::

1. Чтобы проверить значения перед созданием ресурса, укажите все теги ресурсов, нажмите кнопку " **проверить и создать** ".

   :::image type="content" source="./media/how-to-expressroute-direct-portal/review.png" alt-text="Проверка и создание Direct для ExpressRoute":::

1. Нажмите кнопку **Создать**. Отобразится сообщение о том, что развертывание выполняется. Состояние будет отображаться на этой странице при создании ресурсов. 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании ExpressRoute Direct см. в [обзоре](expressroute-erdirect-about.md).

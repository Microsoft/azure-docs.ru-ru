---
title: Руководство по Настройка обработчика правила в Azure Front Door
description: В этой статье приводятся сведения о том, как настроить обработчик правил на портале Azure и в интерфейсе командной строки.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/09/2020
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4f81d8a75cab38bccbfe0ca0f6405baafeb83459
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106068189"
---
# <a name="tutorial-configure-your-rules-engine"></a>Руководство по Настройка обработчика правил

В этом учебнике описано, как создать конфигурацию обработчика правил и первое правило на портале Azure и в интерфейсе командной строки (CLI). 

В этом руководстве описано следующее:
> [!div class="checklist"]
> - настройка обработчика правил с помощью портала;
> - настройка обработчика правил с помощью Azure CLI.

## <a name="prerequisites"></a>Предварительные требования

* Прежде чем перейти к выполнению шагов в этом руководстве, создайте службу Front Door. Дополнительные сведения см. в [кратком руководстве Создание профиля Front Door для высокодоступного глобального веб-приложения](quickstart-create-front-door.md).

## <a name="configure-rules-engine-in-azure-portal"></a>Настройка обработчика правил на портале Azure
1. Во Front Door перейдите в раздел **Параметры** и выберите **Rule Engine configuration** (Конфигурация обработчика правил). Щелкните **Добавить**, присвойте имя конфигурации и приступите к созданию первой конфигурации обработчика правил.

    ![Меню параметров Front Door](./media/front-door-rules-engine/rules-engine-tutorial-1.png)

1. Щелкните **Добавить правило**, чтобы создать первое правило. Затем определите правило, щелкнув **Добавить условие** или **Добавить действие**.
    
    > [!NOTE]
    >- Чтобы удалить условие или действие из правила, используйте корзину в правой части определенного условия или действия.
    > - Чтобы создать правило, которое применяется ко всему входящему трафику, не указывайте никаких условий.
    > - Чтобы отключить обработку правил после выполнения первого условия соответствия, установите флажок **Stop evaluating remaining rule** (Остановить анализ оставшихся правил). Если флажок установлен и для какого-то правила выполнены все условия соответствия, то остальные правила в конфигурации выполняться не будут.  

    ![Конфигурации обработчика правил](./media/front-door-rules-engine/rules-engine-tutorial-4.png) 

1. Определите приоритет правил в конфигурации с помощью кнопок "Вверх", "Вниз" и "На самый верх". Приоритет устанавливается в порядке создания, то есть правило, указанное первым, является самым важным.

1. После создания одного или нескольких правил нажмите кнопку **Сохранить**. В результате будет создана конфигурация обработчика правил.

1. После создания одной или нескольких конфигураций свяжите конфигурацию обработчика правил с правилом маршрутизации. Хотя к нескольким правилам маршрутизации можно применить одну конфигурацию, правило маршрутизации может содержать только одну конфигурацию обработчика правил. Чтобы создать взаимосвязь, выберите **Front Door designer** (Конструктор Front Door) > **Route rules** (Правила маршрутизации). Выберите правило маршрутизации, к которому нужно добавить конфигурацию обработчика правил, выберите **Route details** (Сведения о маршрутизации) > **Rules engine configuration** (Конфигурация обработчика правил) и конфигурацию, которую нужно связать.

    ![Настройка правила маршрутизации](./media/front-door-rules-engine/rules-engine-tutorial-5.png)


## <a name="configure-rules-engine-in-azure-cli"></a>Настройка обработчика правил в Azure CLI

1. Установите [Azure CLI](/cli/azure/install-azure-cli), если это еще не сделано. Добавьте расширение "front-door":- az extension add --name front-door. Затем войдите в подписку az account set — <имя_или_ид> подписки.

1. Начните с создания обработчика правил. В этом примере показано одно правило с одним действием на основе заголовка и одно условие соответствия. 

    ```azurecli-interactive
    az network front-door rules-engine rule create -f {front_door} -g {resource_group} --rules-engine-name {rules_engine} --name {rule1} --priority 1 --action-type RequestHeader --header-action Overwrite --header-name Rewrite --header-value True --match-variable RequestFilenameExtension --operator Contains --match-values jpg png --transforms Lowercase
    ```

1. Перечисление всех правил. 

    ```azurecli-interactive
    az network front-door rules-engine rule list -f {front_door} -g {rg} --name {rules_engine}
    ```

1. Добавление перенаправляемого действия переопределения маршрутизации. 

    ```azurecli-interactive
    az network front-door rules-engine rule action add -f {front_door} -g {rg} --rules-engine-name {rules_engine} --name {rule1} --action-type ForwardRouteOverride --backend-pool {backend_pool_name} --caching Disabled
    ```

1. Перечисление всех действий в правиле. 

    ```azurecli-interactive
    az network front-door rules-engine rule action list -f {front_door} -g {rg} -r {rules_engine} --name {rule1}
    ```

1. Связывание конфигурации обработчика правил с правилом маршрутизации.  

    ```azurecli-interactive
    az network front-door routing-rule update -g {rg} -f {front_door} -n {routing_rule_name} --rules-engine {rules_engine}
    ```

1. Отмена связи с обработчиком правил. 

    ```azurecli-interactive
    az network front-door routing-rule update -g {rg} -f {front_door} -n {routing_rule_name} --remove rulesEngine # case sensitive word ‘rulesEngine’
    ```

Полный список команд обработчика правил AFD можно найти [здесь](/cli/azure/ext/front-door/network/front-door/rules-engine).   

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущих шагах вы настроили конфигурацию обработчика правил и связали ее с правилами маршрутизации. Если вам больше не нужна конфигурация обработчика правил, связанная с Front Door, ее можно удалить, выполнив приведенные ниже действия.

1. Отмените связывание всех правил маршрутизации с конфигурацией обработчика правил, щелкнув многоточие рядом с именем обработчика правил.

    :::image type="content" source="./media/front-door-rules-engine/front-door-rule-engine-routing-association.png" alt-text="Связать правило маршрутизации":::

1. Снимите флажки со всех правил маршрутизации, с которыми связана эта конфигурация обработчика правил, и нажмите "Сохранить".

    :::image type="content" source="./media/front-door-rules-engine/front-door-routing-rule-association.png" alt-text="Связывание правила маршрутизации":::

1. Теперь вы можете удалить конфигурацию обработчика правил с Front Door.

    :::image type="content" source="./media/front-door-rules-engine/front-door-delete-rule-engine-configuration.png" alt-text="Удаление конфигурации обработчика правил":::

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

* Создание конфигурации обработчика правил
* Свяжите конфигурацию с правилами маршрутизации Front Door.

Чтобы узнать, как добавить заголовки безопасности с помощью обработчика правил, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Добавление заголовков безопасности с помощью обработчика правил](front-door-security-headers.md)
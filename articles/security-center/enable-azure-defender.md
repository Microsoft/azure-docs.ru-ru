---
title: Включение интегрированной защиты рабочих нагрузок в Центре безопасности Azure
description: Узнайте, как включить Azure Defender для расширения защиты Центра безопасности Azure для гибридных и многооблачных ресурсов.
author: memildin
ms.author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: quickstart
ms.date: 02/24/2021
ms.openlocfilehash: 7124014821c79fa37aa04da8909e3b4ac3bcb4fb
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106492500"
---
# <a name="quickstart-enable-azure-defender"></a>Краткое руководство. Включение Azure Defender

Дополнительные сведения о преимуществах Azure Defender см. в статье [Сравнение бесплатной версии Центра безопасности с включенным средством Azure Defender](security-center-pricing.md).

## <a name="prerequisites"></a>Предварительные требования

Для работы с руководствами по Центру безопасности необходимо включить Azure Defender. 

С помощью Azure Defender вы можете защитить всю подписку Azure, и эта защита будет наследоваться всеми ресурсами в подписке.

Бесплатный пробный период составляет 30 дней. Цены для определенной валюты и региона см. на странице [цен на Центр безопасности](https://azure.microsoft.com/pricing/details/security-center/).

## <a name="enable-azure-defender-from-the-azure-portal"></a>Включение Azure Defender на портале Azure

Чтобы реализовать все возможности Центра безопасности, включая защиту от угроз, необходимо включить Azure Defender для подписки, которая содержит соответствующие рабочие нагрузки. Включение на уровне рабочей области не включает JIT-доступ к виртуальной машине, адаптивные элементы управления приложениями и возможности сетевого обнаружения для ресурсов Azure. Кроме того, единственные планы Azure Defender, доступные на уровне рабочей области, — это Azure Defender для серверов и Azure Defender для экземпляров SQL Server на компьютерах.

- Вы можете включить **Azure Defender для учетных записей хранения** на уровне подписки или ресурсов.
- Вы можете включить **Azure Defender для SQL** на уровне подписки или ресурсов.
- Для **Базы данных Azure для MariaDB, MySQL и PostgreSQL** защиту от угроз можно включить только на уровне ресурсов.

### <a name="to-enable-azure-defender-on-your-subscriptions-and-workspaces"></a>Включение Azure Defender для подписок и рабочих областей

- Чтобы включить Azure Defender в одной подписке, выполните следующие действия:

    1. В главном меню Центра безопасности выберите **Цены и параметры**.
    1. Выберите подписку или рабочую область, которую требуется защитить.
    1. Выберите **Включить Azure Defender**.
    1. Щелкните **Сохранить**.

    > [!TIP]
    > Здесь вы видите, что каждый план в Azure Defender оплачивается отдельно и может быть включен или отключен отдельно. Например, может потребоваться отключить Azure Defender для Службы приложений в подписках, у которых нет связанного плана Службы приложений Azure. 

    :::image type="content" source="./media/security-center-pricing/pricing-tier-page.png" alt-text="Страница цен Центра безопасности на портале":::

- Чтобы включить Azure Defender для нескольких подписок или рабочих областей, выполните следующие действия:

    1. На боковой панели Центра безопасности выберите **Начало работы**.

        На вкладке **Обновление** отображается список подписок и рабочих областей, доступных для подключения.

        :::image type="content" source="./media/enable-azure-defender/get-started-upgrade-tab.png" alt-text="Вкладка &quot;Обновление&quot; на странице начала работы"::: 

    1. В списке **Select subscriptions and workspaces to enable Azure Defender on** (Выберите подписки и рабочие области, для которых будет включен Azure Defender) выберите подписки и рабочие области, которые требуется обновить, а затем щелкните **Обновить**, чтобы включить Azure Defender.

       - Если вы выберете подписки и рабочие области, на которые не распространяются условия пробной версии, на следующем шаге они будут обновлены и станут платными.
       - Если вы выберете рабочую область, для которой доступна бесплатная пробная версия, следующий шаг включит ее.

        :::image type="content" source="./media/enable-azure-defender/upgrade-selected-workspaces-and-subscriptions.png" alt-text="Обновление всех выбранных рабочих областей и подписок на странице &quot;Начало работы&quot;":::


## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы включили Azure Defender, включите автоматический сбор данных с помощью необходимых агентов и расширений, описанных в статье [Автоматическая подготовка агентов и расширений из Центра безопасности Azure](security-center-enable-data-collection.md).
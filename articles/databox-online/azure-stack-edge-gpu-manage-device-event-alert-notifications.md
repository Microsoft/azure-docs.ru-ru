---
title: Управление уведомлениями о событиях устройств для Azure Stack пограничных ресурсов Pro | Документация Майкрософт
description: Описание использования портал Azure для управления оповещениями о событиях устройств на Azure Stackных ресурсах Pro.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 585343137a4a8fd8a1fb591c640e1183d71c0fd3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102443103"
---
# <a name="manage-device-event-alert-notifications-on-azure-stack-edge-pro-resources"></a>Управление уведомлениями о событиях устройств на Azure Stack пограничных ресурсах Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается создание правил действий в портал Azure для активации или подавления уведомлений о событиях устройств, происходящих в группе ресурсов, подписке Azure или отдельном ресурсе Azure Stackного периметра. Эта статья относится ко всем моделям Azure Stack пограничных устройств.  

## <a name="about-action-rules"></a>О правилах действий

Правило действия может активировать или подавлять уведомления об оповещениях. Правило действия добавляется в *группу действий* — набор параметров уведомлений, которые используются для уведомления пользователей, которым необходимо действовать с оповещениями, активированными в разных контекстах для ресурса или набора ресурсов.

Дополнительные сведения о правилах действий см. в разделе [Настройка правила действий](../azure-monitor/alerts/alerts-action-rules.md?tabs=portal#configuring-an-action-rule). Дополнительные сведения о группах действий см. [в разделе Создание групп действий и управление ими в портал Azure](../azure-monitor/alerts/action-groups.md).

> [!NOTE]
> Функция правил действий доступна в предварительной версии. Некоторые экраны и шаги могут измениться по мере уточнения процесса.


## <a name="create-an-action-rule"></a>Создание правила действия

Выполните следующие действия в портал Azure, чтобы создать правило действия для устройства Azure Stackного периметра.

> [!NOTE]
> Эти действия создают правило действия, которое отправляет уведомления группе действий. Дополнительные сведения о создании правила действий для подавления уведомлений см. в разделе [Настройка правила действий](../azure-monitor/alerts/alerts-action-rules.md?tabs=portal#configuring-an-action-rule).

1. Перейдите к Azure Stack пограничному устройству в портал Azure, а затем выберите **мониторинг > оповещения**. Выберите **Управление действиями**.

   ![Оповещения мониторинга, представление "Управление действиями"](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/action-rules-open-view-01.png)

2. Выберите **правила действий (Предварительная версия)**, а затем выберите **+ новое правило действия**.

   ![Управление действиями, параметрами правил действий](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/action-rules-open-view-02.png)

3. На экране **Создание правила действия** используйте **область** , чтобы выбрать подписку Azure, группу ресурсов или целевой ресурс. Правило действия будет действовать для всех предупреждений, созданных в этой области.

   1. Выберите **выбрать** по **области**.

      ![Выберите область для нового правила действия](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-scope-01.png)

   2. Выберите **подписку** для правила действия, при необходимости выполнив фильтрацию по типу **ресурса** . Чтобы отфильтровать Azure Stack граничные ресурсы, выберите **Data Box Edge устройства (датабокседже)**.

      В области **ресурсов** перечислены доступные ресурсы на основе сделанного выбора.
  
      ![Доступные ресурсы на экране "Выбор области"](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-scope-02.png)

   3. Установите флажок для каждого ресурса, к которому необходимо применить правило. Можно выбрать подписку, группы ресурсов или отдельные ресурсы. Затем выберите **Готово**.

      ![Примеры параметров для области действия правила](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-scope-03.png)

      На экране **создания правила действия** отображается выбранная область.

      ![Завершенная область для правила действия Azure Stack ребра](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-scope-04.png)

4. Используйте параметры **фильтра** , чтобы ограничить применение правила подмножеством предупреждений в выбранной области.

   1. Нажмите кнопку **Добавить** , чтобы открыть панель **Добавление фильтров** .

      ![Параметр для добавления фильтров к правилу действия](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-filter-01.png)

   2. В разделе **фильтры** добавьте каждый фильтр, который нужно применить. Для каждого фильтра выберите тип фильтра, **оператор** и **значение**.
   
      Список параметров фильтра см. в разделе [критерии фильтра](../azure-monitor/alerts/alerts-action-rules.md?tabs=portal#filter-criteria).

      Приведенные ниже примеры фильтров применяются ко всем оповещениям с уровнями серьезности 2, 3 и 4, которые служба мониторинга вызывает для Azure Stack граничных ресурсов.

      По завершении добавления фильтров выберите **Готово**.
   
      ![Пример фильтра для правила действия](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-filter-02.png)

5. На экране **Создание правила действия** выберите **Группа действий** , чтобы создать правило, отправляющее уведомления. Затем по **действиям** выберите **выбрать**.

   ![Параметр группы действий для создания правила действия, отправляющего уведомления](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-action-group-01.png)

   > [!NOTE]
   > Чтобы создать правило, которое подавляет уведомления, выберите **подавление**. Дополнительные сведения см. в разделе [Настройка правила действий](../azure-monitor/alerts/alerts-action-rules.md?tabs=portal#configuring-an-action-rule).

6. Выберите группу действий, которую вы хотите использовать с этим правилом действий. Затем щелкните **Выбрать**. Новое правило действия будет добавлено в параметры уведомлений выбранной группы действий.

   Если необходимо создать новую группу действий, выберите **+ создать группу действий** и выполните действия, описанные в разделе [Создание группы действий с помощью портал Azure](../azure-monitor/alerts/action-groups.md#create-an-action-group-by-using-the-azure-portal).

   ![Выберите группу действий для использования с правилом, а затем нажмите кнопку Выбрать.](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-action-group-02.png)

7. Укажите **имя** и **Описание** нового правила действия и назначьте правило группе ресурсов.

9. Новое правило включено по умолчанию. Если вы не хотите сразу приступить к использованию правила, выберите **нет** , чтобы **включить создание обновления правила**.

10. Завершив настройку, нажмите кнопку **создать**.

    ![Завершенные параметры для правила действия, которое будет отправлять уведомления об оповещениях](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-completed-settings.png)

    Откроется экран **правила действий (Предварительная версия)** , но вы можете не видеть новое правило действий немедленно. Фокус — это **все** группы ресурсов.

11. Чтобы просмотреть новое правило действия, выберите группу ресурсов для этого правила.

    ![Экран правил действий с отображаемым новым правилом](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/new-action-rule-displayed.png)


## <a name="view-notifications"></a>Просмотр уведомлений

Уведомления выступают, когда новое событие запускает оповещение для ресурса, который находится в пределах области действия правила.

Группа действий для наборов правил, которые получают уведомление и тип отправляемого уведомления, сообщения электронной почты, SMS или и того, и другого.

Получение уведомлений после активации оповещения может занять несколько минут.

Уведомление по электронной почте будет выглядеть примерно так.

![Образец уведомления по электронной почте для правила действия](media/azure-stack-edge-gpu-manage-device-event-alert-notifications/sample-action-rule-email-notification.png)


## <a name="next-steps"></a>Дальнейшие действия

<!-- - See [Create and manage action groups in the Azure portal](../azure-monitor/alerts/action-groups.md) for guidance on creating a new action group.
- See [Configure an action rule](../azure-monitor/alerts/alerts-action-rules.md?tabs=portal#configuring-an-action-rule) for more info about creating action rules that send or suppress alert notifications. -2 bullets referenced above. Making room for local tasks in "Next Steps." --> 
- Сведения о просмотре событий устройств, состояния оборудования и диаграмм метрик см. [в статье мониторинг Azure Stack пограничных Pro](azure-stack-edge-monitor.md) . 
- Дополнительные сведения об оптимизации Azure Monitor для устройств с графическим процессором Azure Stack с поддержкой ребра см. [в разделе использование Azure Monitor](azure-stack-edge-gpu-enable-azure-monitor.md) .
- Сведения об управлении отдельными предупреждениями см. [в статье Создание и Просмотр оповещений метрик и управление ими с помощью Azure Monitor ссылки](../azure-monitor/alerts/alerts-metric.md) .
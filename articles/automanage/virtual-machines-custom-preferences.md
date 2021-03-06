---
title: Создание настраиваемого предпочтения в управлении Azure для виртуальных машин
description: Узнайте, как настроить конфигурацию среды в службе "Управление Azure" и задать собственные предпочтения.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: how-to
ms.date: 02/22/2021
ms.author: jushiman
ms.openlocfilehash: 584a3503bf736fcf727a169611e6c79e0c374c90
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101647972"
---
# <a name="create-a-custom-preference-in-azure-automanage-for-vms"></a>Создание настраиваемого предпочтения в управлении Azure для виртуальных машин

В рекомендациях по управлению виртуальными машинами Azure предусмотрены среды по умолчанию, которые при необходимости можно изменить. В этой статье объясняется, как можно задать собственные предпочтения при включении автоуправления на новой или существующей виртуальной машине.

Сейчас мы поддерживаем настройки для [Azure Backup](..\backup\backup-azure-arm-vms-prepare.md#create-a-custom-policy) и [антивредоносного по Майкрософт](../security/fundamentals/antimalware.md#default-and-custom-antimalware-configuration).


> [!NOTE]
> Вы не можете изменить среду или предпочтение на виртуальной машине, пока включена поддержка автоматического управления. Для этой виртуальной машины необходимо отключить функцию автоуправления, а затем снова включить функцию управления, используя требуемую среду и настройки.


## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте учетную запись Azure](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/), прежде чем начинать работу.

> [!NOTE]
> Учетные записи бесплатной пробной версии не предоставляют доступа к виртуальным машинам, которые используются в этом руководстве. Перейдите на подписку с оплатой по мере использования.

> [!IMPORTANT]
> Для включения функции "Управление: **владелец** " или " **участник** " вместе с ролями " **администратор доступа пользователей** " требуется следующее разрешение Azure RBAC.


## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://portal.azure.com/).


## <a name="enable-automanage-for-vms-on-an-existing-vm"></a>Включение Автоматического управления для виртуальных машин на существующей виртуальной машине

1. В строке поиска найдите и выберите **Автоматическое управление — Рекомендации для компьютеров Azure**.

2. Выберите **Enable on existing VM** (Включить на существующей ВМ).

3. В колонке **Выбор компьютеров**:
    1. Отфильтруйте список виртуальных машин по полям **Подписка** и **Группа ресурсов**.
    1. Установите флажок для каждой виртуальной машины, которую требуется подключить.
    1. Нажмите кнопку **Выбрать**.

    :::image type="content" source="media\virtual-machine-custom-preferences\existing-vm-select-machine.png" alt-text="Выбор существующей виртуальной машины в списке доступных виртуальных машин.":::

    > [!NOTE]
    > Щелкните **Показать непригодные компьютеры** , чтобы просмотреть список неподдерживаемых компьютеров и причины. 

4. В разделе **Конфигурация** щелкните **сравнить среды**.

    :::image type="content" source="media\virtual-machine-custom-preferences\existing-vm-quick-create.png" alt-text="Сравнение сред.":::

5. В колонке **сведения о среде** выберите среду из раскрывающегося меню: *Разработка и тестирование* для тестирования, *произ* . для рабочей среды и нажмите кнопку **ОК** .

    :::image type="content" source="media\virtual-machine-custom-preferences\browse-production-profile.png" alt-text="Обзор рабочей среды.":::

6. После выбора среды можно выбрать **Параметры конфигурации**. По умолчанию будут использоваться предпочтительные методы Azure. Этот параметр содержит рекомендуемые параметры для каждой службы. Измените эти параметры, создав пользовательский параметр. 
    1. Щелкните **создать новые настройки**.
    1. В колонке **Создание предпочтения конфигурации** заполните вкладку Основные сведения:
        1. Подписка
        1. Группа ресурсов
        1. Имя предпочтения
        1. Регион

    :::image type="content" source="media\virtual-machine-custom-preferences\create-preference.png" alt-text="Заполните параметры конфигурации.":::

7. Настройте нужные параметры конфигурации.
        
    > [!NOTE]
    > При изменении конфигураций среды будут разрешены только те изменения, которые по-прежнему соответствуют рекомендациям верхней и нижней границ.

8. Проверьте сведения о конфигурации.
9. Нажмите кнопку **Создать** .

10. Нажмите кнопку **Включить**.


## <a name="disable-automanage-for-vms"></a>Отключение Автоматического управления для виртуальных машин

Быстро остановить использование службы Автоматического управления Azure для виртуальных машин можно, отключив Автоматическое управление.

:::image type="content" source="media\virtual-machine-custom-preferences\disable-step-1.png" alt-text="Отключение автоуправления для виртуальной машины.":::

1. Перейдите на страницу **Automanage — Azure virtual machine best practices** (Автоматическое управление — Рекомендации для виртуальных машин Azure), где перечислены все автоматически управляемые виртуальные машины.
1. Установите флажок рядом с виртуальной машиной, для которой необходимо отключить автоматическое управление.
1. Нажмите кнопку **Disable automanagent** (Отключить автоуправление).
1. Внимательно прочтите сообщения в появившемся всплывающем окне, прежде чем нажать кнопку **Отключить**.


## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы создали новую группу ресурсов, чтобы попробовать Автоматическое управление Azure для виртуальных машин, и больше не нуждаетесь в ней, вы можете удалить эту группу ресурсов. Удаление группы также приведет к удалению виртуальной машины и всех ресурсов в группе ресурсов.

Автоматическое управление Azure создает группы ресурсов по умолчанию для хранения ресурсов. Убедитесь, что имена группы ресурсов соответствуют формату "DefaultResourceGroupRegionName" и "AzureBackupRGRegionName", чтобы можно было удалить все ресурсы.

1. Выберите пункт **Группа ресурсов**.
1. На странице этой группы ресурсов нажмите **Удалить**.
1. При появлении запроса подтвердите имя группы ресурсов, которую необходимо удалить, и нажмите **Удалить**.


## <a name="next-steps"></a>Дальнейшие действия 

Получите ответы на наиболее часто задаваемые вопросы. 

> [!div class="nextstepaction"]
> [Часто задаваемые вопросы](faq.md)
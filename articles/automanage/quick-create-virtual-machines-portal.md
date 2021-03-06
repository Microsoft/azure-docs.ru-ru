---
title: Краткое руководство. Включение службы "Автоматическое управление Azure для виртуальных машин" на портале Azure
description: Узнайте, как быстро включить Автоматическое управление Azure для виртуальных машин на новой или существующей виртуальной машине на портале Azure.
author: ju-shim
ms.author: jushiman
ms.date: 02/17/2021
ms.topic: quickstart
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.custom:
- mode-portal
ms.openlocfilehash: 7121d83f9401fe985966324afe6a61cf8396b2bb
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534069"
---
# <a name="quickstart-enable-azure-automanage-for-virtual-machines-in-the-azure-portal"></a>Краткое руководство. Включение службы "Автоматическое управление Azure для виртуальных машин" на портале Azure

С помощью Автоматического управления Azure для виртуальных машин на портале Azure можно включить автоматическое управление для новой или существующей виртуальной машины.


## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте учетную запись Azure](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/), прежде чем начинать работу.

> [!NOTE]
> Учетные записи бесплатной пробной версии не предоставляют доступа к виртуальным машинам, которые используются в этом руководстве. Перейдите на подписку с оплатой по мере использования.

> [!IMPORTANT]
> Вы должны иметь роль **Участник** в группе ресурсов с вашими виртуальными машинами, чтобы включить службу "Автоматическое управление" с помощью существующей учетной записи для этой службы. Если вы включаете Автоматическое управление с помощью новой учетной записи, вам потребуются следующие разрешения: роль **Владелец** или **Участник** вместе с ролью **Администратор доступа пользователей** в вашей подписке.


## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите на [портал Azure](https://aka.ms/AutomanagePortal-Ignite21).

## <a name="enable-automanage-for-a-single-vm"></a>Включение Автоматического управления для отдельной виртуальной машины

1. Выберите виртуальную машину, для которой нужно включить эту службу.

2. Щелкните запись **Автоматическое управление (предварительная версия)** в содержании в разделе **Операции**.

3. Выберите **Начать**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\vmmanage-getstartedbutton.png" alt-text="Начало работы с отдельной виртуальной машиной.":::

4. Выберите параметры Автоматического управления (среда, настройки, учетная запись службы) и щелкните **Включить**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\vmmanage-enablepane.png" alt-text="Включение службы для отдельной виртуальной машины.":::

## <a name="enable-automanage-for-multiple-vms"></a>Включение Автоматического управления для нескольких виртуальных машин

1. В строке поиска найдите и выберите **Автоматическое управление — Рекомендации для компьютеров Azure**.

2. Выберите **Enable on existing VM** (Включить на существующей ВМ).

    :::image type="content" source="media\quick-create-virtual-machine-portal\zero-vm-list-view.png" alt-text="Enable on existing VM"::: (Включить на существующей ВМ).

3. В колонке **Выбор компьютеров**:
    1. Отфильтруйте список виртуальных машин по полям **Подписка** и **Группа ресурсов**.
    1. Установите флажок для каждой виртуальной машины, которую требуется подключить.
    1. Нажмите кнопку **Выбрать**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\existing-vm-select-machine.png" alt-text="Выбор существующей виртуальной машины в списке доступных виртуальных машин.":::

4. В разделе **Среда** выберите тип среды: **Разработка и тестирование** или **Рабочая среда**. 

    :::image type="content" source="media\quick-create-virtual-machine-portal\existing-vm-quick-create.png" alt-text="Выбор среды.":::

   Щелкните **Сравнение сведений о средах**, чтобы просмотреть различия между средами.
    1. Выберите среду из раскрывающегося списка: *Разработка и тестирование* для тестирования или *Рабочая среда* для рабочей среды.
    1. Нажмите кнопку **ОК** .

    :::image type="content" source="media\quick-create-virtual-machine-portal\browse-production-profile.png" alt-text="Обзор рабочей среды.":::

5. По умолчанию для параметров конфигурации выбраны параметры **Рекомендации для Azure**. Чтобы изменить это, создайте новые параметры или выберите существующие. 

    :::image type="content" source="media\quick-create-virtual-machine-portal\create-preference.png" alt-text="Создание параметров.":::

6. Нажмите кнопку **Включить**.


## <a name="enable-automanage-for-a-new-vm"></a>Включение Автоматического управления для новой виртуальной машины

Войдите на портал Azure [здесь](https://aka.ms/AzureAutomanagePreview), чтобы создать виртуальную машину и включить Автоматическое управление.

1. На вкладке **Общие сведения** введите данные о виртуальной машине.

> [!NOTE]
> Ознакомьтесь с поддерживаемыми [регионами](automanage-virtual-machines.md#supported-regions), [дистрибутивами Linux](automanage-linux.md#supported-linux-distributions-and-versions) и [версиями Windows Server](automanage-windows-server.md#supported-windows-server-versions) для Автоуправления.

2. Перейдите на вкладку **Управление** и выберите **среду Автоматического управления**.

    :::image type="content" source="media\quick-create-virtual-machine-portal\vmcreate-managementtab.png" alt-text="Включение Автоматического управления на вкладке &quot;Управление&quot;.":::

3. Оставьте остальные значения по умолчанию и нажмите кнопку **Просмотр и создание**, расположенную в нижней части страницы.

4. Когда появится сообщение об успешной проверке, выберите **Создать**.

## <a name="disable-automanage-for-vms"></a>Отключение Автоматического управления для виртуальных машин

Быстро остановить использование службы Автоматического управления Azure для виртуальных машин можно, отключив Автоматическое управление.

:::image type="content" source="media\automanage-virtual-machines\disable-step-1.png" alt-text="Отключение автоуправления для виртуальной машины.":::

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

В этом кратком руководстве описывается включение Автоматического управления Azure для виртуальных машин.

Узнайте, как можно создавать и применять пользовательские настройки при включении Автоматического управления на виртуальной машине.

> [!div class="nextstepaction"]
> [Автоматическое управление Azure для виртуальных машин: пользовательские параметры конфигурации](virtual-machines-custom-preferences.md)

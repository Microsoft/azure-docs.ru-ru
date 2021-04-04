---
title: Интеграция службы автоматизации Azure со службой "Сетка событий Azure" | Документация Майкрософт
description: Узнайте, как автоматически добавить тег при создании новой виртуальной машины и как отправить уведомление в Microsoft Teams.
keywords: automation, runbook, teams, event grid, virtual machine, VM
services: automation,event-grid
author: eamonoreilly
ms.service: automation
ms.topic: tutorial
ms.workload: infrastructure-services
ms.date: 07/07/2020
ms.author: eamono
ms.openlocfilehash: 3b9b49a4d38566891f442a3d2d7eac9bf1d36465
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87462009"
---
# <a name="tutorial-integrate-azure-automation-with-event-grid-and-microsoft-teams"></a>Руководство. Интеграция службы автоматизации Azure со службой "Сетка событий Azure" и Microsoft Teams

В этом руководстве описано следующее.

> [!div class="checklist"]
> * импорт примера runbook для Сетки событий;
> * создание перехватчика для Microsoft Teams (необязательно);
> * создание веб-перехватчика для runbook;
> * создание подписки в службе "Сетка событий";
> * создание виртуальной машины, которая запускает runbook.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

Чтобы выполнить инструкции из этого руководства, вам понадобится [учетная запись службы автоматизации Azure](../automation/index.yml), в которой будет храниться runbook для запуска из подписки служба "Сетка событий".

* Модуль `AzureRM.Tags` нужно загрузить в учетной записи службы автоматизации. См. инструкции по [импорту модулей в службе автоматизации Azure](../automation/automation-update-azure-modules.md).

## <a name="import-an-event-grid-sample-runbook"></a>Импорт примера runbook для службы "Сетка событий"

1. Выберите учетную запись службы автоматизации и перейдите на страницу **Runbooks**.

   ![Выбор модулей runbook](./media/ensure-tags-exists-on-new-virtual-machines/select-runbooks.png)

2. Нажмите кнопку **Просмотреть коллекцию**.

3. Выполните поиск по запросу **Сетка событий** и выберите элемент **Integrating Azure Automation with Event grid** (Интеграция службы автоматизации Azure с Сеткой событий).

    ![Импорт runbook из коллекции](media/ensure-tags-exists-on-new-virtual-machines/gallery-event-grid.png)

4. Выберите действие **Импорт** и присвойте ему имя **Watch-VMWrite**.

5. Когда импорт успешно завершится, выберите **Изменить**, чтобы просмотреть источник runbook. 
6. Обновите строку 74 в скрипте, чтобы использовать `Tag` вместо `Tags`.

    ```powershell
    Update-AzureRmVM -ResourceGroupName $VMResourceGroup -VM $VM -Tag $Tag | Write-Verbose
    ```
7. Нажмите кнопку **Опубликовать**.

## <a name="create-an-optional-microsoft-teams-webhook"></a>Создание перехватчика для Microsoft Teams (необязательно)

1. В Microsoft Teams выберите элемент **Дополнительные параметры** рядом с именем канала, а затем выберите **Соединители**.

    ![Подключения Microsoft Teams](media/ensure-tags-exists-on-new-virtual-machines/teams-webhook.png)

2. Прокрутите список соединителей до элемента **Входящий веб-перехватчик** и выберите для него действие **Добавить**.

3. Введите имя **AzureAutomationIntegration** и нажмите кнопку **Создать**.

4. Скопируйте URL-адрес веб-перехватчика в буфер обмена и сохраните его. URL-адрес веб-перехватчика используется для отправки данных в Microsoft Teams.

5. Выберите **Готово**, чтобы сохранить веб-перехватчик.

## <a name="create-a-webhook-for-the-runbook"></a>Создание веб-перехватчика для runbook

1. Откройте runbook с именем Watch-VMWrite.

2. Выберите **Веб-перехватчики** и нажмите кнопку **Добавить веб-перехватчик**.

3. Введите имя **WatchVMEventGrid**. Скопируйте URL-адрес в буфер обмена и сохраните его.

    ![Настройка имени веб-перехватчика](media/ensure-tags-exists-on-new-virtual-machines/copy-url.png)

4. Выберите действие **Настройка параметров и настроек запуска** и введите URL-адрес веб-перехватчика Microsoft Teams в поле **CHANNELURL**. Поле **WEBHOOKDATA** оставьте пустым.

    ![Настройка параметров веб-перехватчика](media/ensure-tags-exists-on-new-virtual-machines/configure-webhook-parameters.png)

5. Щелкните **Создать**, чтобы создать веб-перехватчик для модуля runbook службы автоматизации.

## <a name="create-an-event-grid-subscription"></a>Создание подписки Сетки событий

1. На странице **Учетная запись службы автоматизации** выберите элемент **Сетка событий**.

    ![Выбор службы "Сетка событий"](media/ensure-tags-exists-on-new-virtual-machines/select-event-grid.png)

2. Нажмите кнопку **+Подписка на события**.

3. Настройте для подписки следующие сведения.
    1. В списке **Тип раздела** выберите **Подписки Azure**.
    2. Снимите флажок **Подписка на все типы событий**.
    3. Введите имя **AzureAutomation**.
    4. В раскрывающемся списке **Определенные типы событий** снимите все флажки, кроме **Успешная запись ресурсов**.

        > [!NOTE] 
        > Для Azure Resource Manager сейчас не учитывается разница между созданием и обновлением. Поэтому реализация этого руководства для всех событий Microsoft.Resources.ResourceWriteSuccess в вашей подписке Azure может привести к большому количеству вызовов.
    1. Для **типа конечной точки** выберите **Веб-перехватчик**.
    2. Щелкните **Выбрать конечную точку**. На открывшейся странице **Выбор веб-перехватчика** вставьте URL-адрес веб-перехватчика, созданный для модуля runbook с именем Watch-VMWrite.
    3. В разделе **Фильтры** укажите подписку и группу ресурсов, в которых нужно искать новые виртуальные машины. Результат будет выглядеть так: `/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Compute/virtualMachines`.

4. Выберите действие **Создать**, чтобы сохранить подписку Сетки событий.

## <a name="create-a-vm-that-triggers-the-runbook"></a>Создание виртуальной машины, которая запускает runbook

1. Создайте новую виртуальную машину в той группе ресурсов, которую вы указали в фильтре префиксов для подписки Сетки событий.

2. Если все настроено правильно, runbook Watch-VMWrite сработает и добавит к этой виртуальной машине новый тег.

    ![Тег виртуальной машины](media/ensure-tags-exists-on-new-virtual-machines/vm-tag.png)

3. Новое сообщение отправляется в канал Microsoft Teams.

    ![Уведомление для Microsoft Teams](media/ensure-tags-exists-on-new-virtual-machines/teams-vm-message.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы настроили интеграцию между Сеткой событий и службой автоматизации. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * импорт примера runbook для Сетки событий;
> * создание перехватчика для Microsoft Teams (необязательно);
> * создание веб-перехватчика для runbook;
> * создание подписки в службе "Сетка событий";
> * создание виртуальной машины, которая запускает runbook.

> [!div class="nextstepaction"]
> [Создание и перенаправление пользовательского события при помощи службы "Сетка событий"](../event-grid/custom-event-quickstart.md)

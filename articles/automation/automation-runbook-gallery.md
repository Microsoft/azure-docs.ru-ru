---
title: Использование модулей runbook и модулей службы автоматизации Azure в коллекции PowerShell
description: В этой статье рассказывается, как использовать модули runbook и модули, разработанные Майкрософт и сообществом, в коллекции PowerShell.
services: automation
ms.subservice: process-automation
ms.date: 03/04/2021
ms.topic: conceptual
ms.openlocfilehash: afa782df8666413356fa334bf4e9dcb989b87c2f
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102441359"
---
# <a name="use-runbooks-and-modules-in-powershell-gallery"></a>Поиск последовательностей runbook и модулей в коллекции PowerShell

Вы можете не создавать собственные модули Runbook и другие модули в службе автоматизации Azure, а воспользоваться сценариями, уже созданными корпорацией Майкрософт и сообществом. Runbook и модули PowerShell можно получить из коллекция PowerShell и [модулей](#modules-in-powershell-gallery) [Runbook Python](#use-python-runbooks) в Организации Azure Automation GitHub. Вы также можете поделиться с сообществом [скриптами, которые вы разработали](#add-a-powershell-runbook-to-the-gallery).

> [!NOTE]
> В центре сценариев TechNet будет снято снятие с учета. Все модули Runbook из центра сценариев в коллекции Runbook были перемещены в нашу [организацию GitHub службы автоматизации](https://github.com/azureautomation) . Дополнительные сведения см. [здесь](https://techcommunity.microsoft.com/t5/azure-governance-and-management/azure-automation-runbooks-moving-to-github/ba-p/2039337).

## <a name="runbooks-in-powershell-gallery"></a>Модули runbook в коллекции PowerShell

В [коллекции PowerShell](https://www.powershellgallery.com/packages) содержатся разнообразные модули runbook, разработанные корпорацией Майкрософт и сообществом, которые вы можете импортировать в службу автоматизации Azure. Чтобы использовать модуль runbook, скачайте его из коллекции. Вы также можете напрямую импортировать модули runbook из этой коллекции или из своей учетной записи службы автоматизации на портале Azure.

> [!NOTE]
> Графические модули runbook в коллекции PowerShell не поддерживаются.

Их можно только импортировать напрямую из коллекции PowerShell через портал Azure. Эту функцию нельзя выполнить с помощью PowerShell.

> [!NOTE]
> Обязательно проверяйте содержимое всех модулей runbook, которые вы получаете из коллекции PowerShell, и соблюдайте предельную осторожность при их установке и запуске в рабочей среде.

## <a name="modules-in-powershell-gallery"></a>Модули в коллекции PowerShell

Модули PowerShell содержат командлеты, которые можно использовать в модулях Runbook. Существующие модули, которые можно установить в службе автоматизации Azure, доступны в [коллекция PowerShell](https://www.powershellgallery.com). Вы можете запустить эту галерею из портал Azure и установить модули непосредственно в службу автоматизации Azure. также можно скачать и установить их вручную.

## <a name="common-scenarios-available-in-powershell-gallery"></a>Типовые сценарии, доступные в коллекции PowerShell

В следующем списке перечислено несколько модулей runbook, которые поддерживают типовые сценарии. Полный список модулей runbook, созданных командой службы автоматизации Azure, см. в разделе [Профиль AzureAutomationTeam](https://www.powershellgallery.com/profiles/AzureAutomationTeam).

   * [Update-ModulesInAutomationToLatestVersion](https://www.powershellgallery.com/packages/Update-ModulesInAutomationToLatestVersion/) — импортирует последние версии всех модулей в учетной записи автоматизации из коллекции PowerShell.
   * [Enable-AzureDiagnostics](https://www.powershellgallery.com/packages/Enable-AzureDiagnostics/) — настраивает Диагностику Azure и Log Analytics для получения журналов службы автоматизации Azure, содержащих сведения о состоянии заданий и потоки заданий.
   * [Copy-ItemFromAzureVM](https://www.powershellgallery.com/packages/Copy-ItemFromAzureVM/) — копирует удаленный файл из виртуальной машины Windows Azure.
   * [Copy-итемтоазуревм](https://www.powershellgallery.com/packages/Copy-ItemToAzureVM/) — копирует локальный файл на виртуальную машину Azure.

## <a name="import-a-powershell-runbook-from-the-runbook-gallery-with-the-azure-portal"></a>Импорт модуля runbook из коллекции runbook с помощью портала Azure

1. На портале Azure выберите свою учетную запись службы автоматизации.
1. Выберите **Коллекция модулей runbook** в разделе **Автоматизация процессов**.
1. Выберите **Источник: коллекция PowerShell**. Отобразится список доступных модулей Runbook, которые можно просмотреть.
1. Можно использовать поле поиска над списком, чтобы уменьшить список, или использовать фильтры для сокращения отображаемых значений по издателю, типу и сортировке. Найдите нужный элемент коллекции и выберите его, чтобы просмотреть сведения о нем.

   :::image type="content" source="media/automation-runbook-gallery/browse-gallery-sm.png" alt-text="Просмотр коллекции модулей Runbook" lightbox="media/automation-runbook-gallery/browse-gallery-lg.png":::

1. Чтобы импортировать элемент, нажмите кнопку **Импорт** в колонке сведения.

   :::image type="content" source="media/automation-runbook-gallery/gallery-item-detail-sm.png" alt-text="Отображение сведений об элементе коллекции Runbook" lightbox="media/automation-runbook-gallery/gallery-item-detail-lg.png":::

1. Также для импорта модуля Runbook вы можете изменить его имя, а затем нажать кнопку **ОК** .
1. Модуль runbook появится на вкладке **Модули runbook** учетной записи службы автоматизации.

## <a name="import-a--powershell-runbook-from-github-with-the-azure-portal"></a>Импорт модуля Runbook PowerShell из GitHub с помощью портал Azure

1. На портале Azure выберите свою учетную запись службы автоматизации.
1. Выберите **Коллекция модулей runbook** в разделе **Автоматизация процессов**.
1. Выберите **Источник: GitHub**.
1. Фильтры, расположенные над списком, можно использовать для ограничения отображаемых значений по издателю, типу и сортировке. Найдите нужный элемент коллекции и выберите его, чтобы просмотреть сведения о нем.

   :::image type="content" source="media/automation-runbook-gallery/browse-gallery-github-sm.png" alt-text="Просмотр коллекции GitHub" lightbox="media/automation-runbook-gallery/browse-gallery-github-lg.png":::

1. Чтобы импортировать элемент, нажмите кнопку **Импорт** в колонке сведения.

   :::image type="content" source="media/automation-runbook-gallery/gallery-item-details-blade-github-sm.png" alt-text="Подробное представление модуля Runbook из коллекции GitHub" lightbox="media/automation-runbook-gallery/gallery-item-details-blade-github-lg.png":::

1. Также для импорта модуля Runbook вы можете изменить его имя, а затем нажать кнопку **ОК** .
1. Модуль runbook появится на вкладке **Модули runbook** учетной записи службы автоматизации.

## <a name="add-a-powershell-runbook-to-the-gallery"></a>Добавление модуля runbook PowerShell в коллекцию

Корпорация Майкрософт рекомендует добавлять в коллекцию PowerShell модули runbook, которые, на ваш взгляд, могут пригодиться другим клиентам. Коллекция PowerShell принимает модули PowerShell и скрипты PowerShell. Вы можете добавить модуль runbook, [отправив его в коллекцию PowerShell](/powershell/scripting/gallery/how-to/publishing-packages/publishing-a-package).

## <a name="import-a-module-from-the-module-gallery-with-the-azure-portal"></a>Импорт модуля из коллекции модулей с помощью портала Azure

1. На портале Azure выберите свою учетную запись службы автоматизации.
1. Выберите **Модули** в разделе **Общие ресурсы** чтобы открыть список модулей.
1. В верхней части страницы щелкните **Обзор коллекции**.

      :::image type="content" source="media/automation-runbook-gallery/modules-blade-sm.png" alt-text="Представление коллекции модулей" lightbox="media/automation-runbook-gallery/modules-blade-lg.png":::

1. На странице Обзор коллекции можно использовать поле поиска для поиска совпадений в любом из следующих полей:

   * Имя модуля
   * Теги
   * Автор
   * Командлет или имя ресурса DSC

1. Найдите интересующий вас модуль и выберите его, чтобы просмотреть сведения.

   Во время подробного рассмотрения конкретного модуля просмотрите дополнительную информацию. Данная информация включает ссылку на коллекцию PowerShell, все необходимые зависимости и все командлеты или ресурсы DSC, которые содержит модуль.

   :::image type="content" source="media/automation-runbook-gallery/gallery-item-details-blade-sm.png" alt-text="Подробное представление модуля из коллекции" lightbox="media/automation-runbook-gallery/gallery-item-details-blade-lg.png":::

1. Чтобы установить модуль непосредственно в службу автоматизации Azure, нажмите кнопку **Импорт**.
1. В панели импорта можно просмотреть имя модуля, который нужно импортировать. Если установлены все зависимости, кнопка **ОК** будет активна. Если какие-либо зависимости отсутствуют, их необходимо будет импортировать перед импортом данного модуля.
1. На странице импорта нажмите кнопку **ОК**, чтобы импортировать модуль. Когда служба автоматизации Azure импортирует модуль в вашу учетную запись, она извлекает метаданные о модуле и командлетах. Эта операция может занять несколько минут, так как каждое действие необходимо извлечь.
1. Вы получите два уведомления о начале и завершении развертывания модуля соответственно.
1. Доступные действия появятся после импорта модуля. Вы можете использовать ресурсы модуля в своих модулях runbook и ресурсах DSC.

> [!NOTE]
> Модули, которые поддерживают только PowerShell Core, не поддерживаются в службе автоматизации Azure. Их нельзя импортировать на портале Azure и развернуть непосредственно из коллекции PowerShell.

## <a name="use-python-runbooks"></a>Использование модулей runbook Python

Модули Runbook Python доступны в [Организации Azure Automation GitHub](https://github.com/azureautomation). При участии в репозитории GitHub добавьте тег **(раздел GitHub): Python3** при отправке публикации.

## <a name="request-a-runbook-or-module"></a>Запрос модуля runbook или другого модуля

Свой запрос можно отправить на сайте [User Voice](https://feedback.azure.com/forums/246290-azure-automation/).  Если вам нужна помощь в написании модуля runbook или освоении PowerShell, опубликуйте вопрос на нашей [странице вопросов и ответов Майкрософт](/answers/topics/azure-automation.html).

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы начать работу с модулем runbook PowerShell, изучите документ [Руководство. Создание модуля runbook PowerShell](learn/automation-tutorial-runbook-textual-powershell.md).
* Сведения о работе с модулями runbook см. в статье [Управление модулями runbook в службе автоматизации Azure](manage-runbooks.md).
* Дополнительные сведения о PowerShell см. в [документации PowerShell](/powershell/scripting/overview).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).

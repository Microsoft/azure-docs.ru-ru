---
title: Сведения о конфигурации сети службы автоматизации Azure
description: В этой статье содержатся сведения о сети, необходимые для настройки состояния службы автоматизации Azure, гибридной рабочей роли Runbook службы автоматизации Azure, Управление обновлениями, а также Отслеживание изменений и инвентаризации.
ms.author: magoedte
ms.topic: conceptual
ms.date: 01/26/2021
ms.openlocfilehash: 36331e9c07926d4d3ffff136aefa2f9a77d47cb4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101708889"
---
# <a name="azure-automation-network-configuration-details"></a>Сведения о конфигурации сети службы автоматизации Azure

На этой странице представлены сведения о сети, необходимые для [гибридной рабочей роли Runbook и конфигурации состояния](#hybrid-runbook-worker-and-state-configuration), а также для [Управление обновлениями и отслеживание изменений и инвентаризации](#update-management-and-change-tracking-and-inventory).

## <a name="hybrid-runbook-worker-and-state-configuration"></a>Гибридная Рабочая роль Runbook и конфигурация состояния

Следующие порты и URL-адреса необходимы для гибридной рабочей роли Runbook и для [настройки состояния автоматизации](automation-dsc-overview.md) для взаимодействия со службой автоматизации Azure.

* Порт: для исходящего доступа в Интернет требуется только 443
* Глобальный URL-адрес: `*.azure-automation.net`
* Глобальный URL-адрес US Gov (Вирджиния): `*.azure-automation.us`
* Служба агента: `https://<workspaceId>.agentsvc.azure-automation.net`

### <a name="network-planning-for-hybrid-runbook-worker"></a>Планирование сети для гибридной рабочей роли Runbook

Чтобы использовать гибридную рабочую роль Runbook для системы или пользователя для подключения и регистрации в службе автоматизации Azure, она должна иметь доступ к номеру порта и URL-адресам, описанным в этом разделе. Кроме того, Рабочая роль должна иметь доступ к [портам и URL-адресам, необходимым для подключения агента log Analytics](../azure-monitor/agents/agent-windows.md) к рабочей области Azure Monitor log Analytics.

При наличии учетной записи службы автоматизации, определенной для конкретного региона, можно ограничить обмен данными гибридной рабочей роли Runbook соответствующим региональным центром обработки данных. Проверьте [записи DNS, используемые службой автоматизации Azure](how-to/automation-region-dns-records.md) для необходимых записей DNS.

### <a name="configuration-of-private-networks-for-state-configuration"></a>Настройка частных сетей для настройки состояния

Если узлы находятся в частной сети, необходимо указать порт и URL-адреса, указанные выше. Эти ресурсы обеспечивают сетевое подключение для управляемого узла и позволяют DSC обмениваться данными со службой автоматизации Azure.

При использовании ресурсов DSC, взаимодействующих между узлами, например [ресурсов WaitFor*](/powershell/scripting/dsc/reference/resources/windows/waitForAllResource), необходимо также разрешить трафик между узлами. Сведения об этих требованиях к сети см. в документации по каждому ресурсу DSC.

Сведения о требованиях клиента к TLS 1,2 см. в разделе [Принудительная поддержка tls 1,2 для службы автоматизации Azure](automation-managing-data.md#tls-12-enforcement-for-azure-automation).

## <a name="update-management-and-change-tracking-and-inventory"></a>Управление обновлениями и Отслеживание изменений и Инвентаризация

Адреса в этой таблице необходимы как для Управление обновлениями, так и для Отслеживание изменений и инвентаризации. Абзац, который следует за таблицей, также применяется к обоим параметрам.

Для обмена данными с этими адресами используется **порт 443**.

|Azure Public  |Azure для государственных организаций  |
|---------|---------|
|\*.ods.opinsights.azure.com    | \*.ods.opinsights.azure.us         |
|\*.oms.opinsights.azure.com     | \*.oms.opinsights.azure.us        |
|\*.blob.core.windows.net | \*.blob.core.usgovcloudapi.net|
|\*.azure-automation.net | \*. azure-automation.us|

При создании правил безопасности сетевой группы или настройке брандмауэра Azure для разрешения трафика в службу автоматизации и Log Analytics рабочей области используйте [теги службы](../virtual-network/service-tags-overview.md#available-service-tags) **гуестандхибридманажемент** и **азуремонитор**. Это упрощает текущее управление правилами безопасности сети. Для безопасного и частного подключения к службе автоматизации виртуальных машин Azure см. раздел [Использование частной связи Azure](./how-to/private-link-security.md). Сведения о получении тега текущей службы и сведений о диапазоне для включения в локальные конфигурации брандмауэра см. в разделе [Загружаемые файлы JSON](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о службе [автоматизации Управление обновлениями](update-management\overview.md).
* Сведения о [гибридной рабочей роли Runbook](automation-hybrid-runbook-worker.md).
* Сведения о [Отслеживание изменений и инвентаризации](change-tracking\overview.md).
* Сведения о [настройке состояния службы автоматизации](automation-dsc-overview.md).
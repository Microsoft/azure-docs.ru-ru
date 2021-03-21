---
title: Записи DNS центра обработки данных Azure, используемые службой автоматизации Azure | Документация Майкрософт
description: В этой статье содержатся записи DNS, необходимые для функций службы автоматизации Azure при ограничении взаимодействия с определенным регионом Azure, на котором размещена эта учетная запись службы автоматизации.
services: automation
ms.subservice: process-automation
ms.date: 11/25/2020
ms.topic: conceptual
ms.openlocfilehash: 8630afa7410aad81a7a3c61540fc74702fc6481c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100575999"
---
# <a name="dns-records-for-azure-regions-used-by-azure-automation"></a>Записи DNS для регионов Azure, используемых службой автоматизации Azure

Служба [автоматизации Azure](../automation-intro.md) использует несколько записей DNS для подключения к службе. При наличии учетной записи службы автоматизации, определенной для конкретного региона, можно ограничить обмен данными с таким региональным центром обработки данных. Возможно, вам потребуется знать эти записи, чтобы обеспечить работу следующих функций автоматизации, если они размещены за брандмауэром:

* Гибридная рабочая роль Runbook
* State Configuration
* Веб-перехватчики

>[!NOTE]
>При регистрации гибридной рабочей роли Runbook Linux новые записи будут завершаться ошибкой, если она не имеет версию 1.6.10.2 или выше. Необходимо выполнить обновление до более новой версии [агента log Analytics для Linux](../../azure-monitor/agents/agent-linux.md) , чтобы компьютер мог получить обновленную версию рабочей роли и использовать эти новые записи. Существующие компьютеры будут продолжать работать без каких бы то ни было проблем.  

## <a name="dns-records-per-region"></a>Записи DNS для каждого региона

В таблице ниже содержатся записи DNS для каждого региона.

>[!NOTE]
>Хотя список предоставленных здесь записей DNS службы автоматизации был снят, они по-прежнему работают, чтобы позволить вам перейти на новые записи, перечисленные в разделе [Поддержка частной связи](#support-for-private-link) , и предотвратить сбои в процессах автоматизации.

| **Регион** | **Запись DNS** |
| --- | --- |
| Центральная Австралия |ac-jobruntimedata-prod-su1.azure-automation.net</br>ac-agentservice-prod-1.azure-automation.net |
| Восточная Австралия |ae-jobruntimedata-prod-su1.azure-automation.net</br>ae-agentservice-prod-1.azure-automation.net |
| Юго-Восточная Австралия |ase-jobruntimedata-prod-su1.azure-automation.net</br>ase-agentservice-prod-1.azure-automation.net |
| Центральная Канада |cc-jobruntimedata-prod-su1.azure-automation.net</br>cc-agentservice-prod-1.azure-automation.net |
| Центральная Индия |cid-jobruntimedata-prod-su1.azure-automation.net</br>cid-agentservice-prod-1.azure-automation.net |
| восточная часть США 2 |eus2-jobruntimedata-prod-su1.azure-automation.net</br>eus2-agentservice-prod-1.azure-automation.net |
| Восточная Япония |jpe-jobruntimedata-prod-su1.azure-automation.net</br>jpe-agentservice-prod-1.azure-automation.net |
| Северная Европа |ne-jobruntimedata-prod-su1.azure-automation.net</br>ne-agentservice-prod-1.azure-automation.net |
| Центрально-южная часть США |scus-jobruntimedata-prod-su1.azure-automation.net</br>scus-agentservice-prod-1.azure-automation.net |
| Юго-Восточная Азия |sea-jobruntimedata-prod-su1.azure-automation.net</br>sea-agentservice-prod-1.azure-automation.net|
| южная часть Соединенного Королевства | uks-jobruntimedata-prod-su1.azure-automation.net</br>uks-agentservice-prod-1.azure-automation.net |
| US Gov (Вирджиния) | usge-jobruntimedata-prod-su1.azure-automation.us<br>usge-agentservice-prod-1.azure-automation.us |
| центрально-западная часть США | wcus-jobruntimedata-prod-su1.azure-automation.net</br>wcus-agentservice-prod-1.azure-automation.net |
| Западная Европа |we-jobruntimedata-prod-su1.azure-automation.net</br>we-agentservice-prod-1.azure-automation.net |
| западная часть США 2 |wus2-jobruntimedata-prod-su1.azure-automation.net</br>wus2-agentservice-prod-su1.azure-automation.net |

### <a name="support-for-private-link"></a>Поддержка закрытой ссылки

Для поддержки [частной ссылки](../../private-link/private-link-overview.md) в службе автоматизации Azure были обновлены записи DNS для каждого поддерживаемого центра обработки данных. Вместо URL-адресов, зависящих от региона, URL-адреса зависят от конкретной учетной записи службы автоматизации.

| **Регион** | **Запись DNS** |
| --- | --- |
| центрально-западная часть США |`https://<accountId>.webhook.wcus.azure-automation.net`<br>`https://<accountId>.agentsvc.wcus.azure-automation.net`<br>`https://<accountId>.jrds.wcus.azure-automation.net` |
| западная часть США |`https://<accountId>.webhook.wus.azure-automation.net`<br>`https://<accountId>.agentsvc.wus.azure-automation.net`<br>`https://<accountId>.jrds.wus.azure-automation.net` |
| западная часть США 2 |`https://<accountId>.webhook.wus2.azure-automation.net`<br>`https://<accountId>.agentsvc.wus2.azure-automation.net`<br>`https://<accountId>.jrds.wus2.azure-automation.net` |
| Центральная часть США |`https://<accountId>.webhook.cus.azure-automation.net`<br>`https://<accountId>.agentsvc.cus.azure-automation.net`<br>`https://<accountId>.jrds.cus.azure-automation.net` |
| Центрально-южная часть США |`https://<accountId>.webhook.scus.azure-automation.net`<br>`https://<accountId>.agentsvc.scus.azure-automation.net`<br>`https://<accountId>.jrds.scus.azure-automation.net` |
| Центрально-северная часть США |`https://<accountId>.webhook.ncus.azure-automation.net`<br>`https://<accountId>.agentsvc.ncus.azure-automation.net`<br>`https://<accountId>.jrds.ncus.azure-automation.net` |
| Восточная часть США |`https://<accountId>.webhook.eus.azure-automation.net`<br>`https://<accountId>.agentsvc.eus.azure-automation.net`<br>`https://<accountId>.jrds.eus.azure-automation.net` |
| восточная часть США 2 |`https://<accountId>.webhook.eus2.azure-automation.net`<br>`https://<accountId>.agentsvc.eus2.azure-automation.net`<br>`https://<accountId>.jrds.eus2.azure-automation.net` |
| Центральная Канада |`https://<accountId>.webhook.cc.azure-automation.net`<br>`https://<accountId>.agentsvc.cc.azure-automation.net`<br>`https://<accountId>.jrds.cc.azure-automation.net` |
| Западная Европа |`https://<accountId>.webhook.we.azure-automation.net`<br>`https://<accountId>.agentsvc.we.azure-automation.net`<br>`https://<accountId>.jrds.we.azure-automation.net` |
| Северная Европа |`https://<accountId>.webhook.ne.azure-automation.net`<br>`https://<accountId>.agentsvc.ne.azure-automation.net`<br>`https://<accountId>.jrds.ne.azure-automation.net` |
| Юго-Восточная Азия |`https://<accountId>.webhook.sea.azure-automation.net`<br>`https://<accountId>.agentsvc.sea.azure-automation.net`<br>`https://<accountId>.jrds.sea.azure-automation.net` |
| Восточная Азия |`https://<accountId>.webhook.ea.azure-automation.net`<br>`https://<accountId>.agentsvc.ea.azure-automation.net`<br>`https://<accountId>.jrds.ea.azure-automation.net` |
| Центральная Индия |`https://<accountId>.webhook.cid.azure-automation.net`<br>`https://<accountId>.agentsvc.cid.azure-automation.net`<br>`https://<accountId>.jrds.cid.azure-automation.net` |
| Восточная Япония |`https://<accountId>.webhook.jpe.azure-automation.net`<br>`https://<accountId>.agentsvc.jpe.azure-automation.net`<br>`https://<accountId>.jrds.jpe.azure-automation.net` |
| Республика Корея, центральный регион |`https://<accountId>.webhook.kc.azure-automation.net`<br>`https://<accountId>.agentsvc.kc.azure-automation.net`<br>`https://<accountId>.jrds.kc.azure-automation.net` |
| Юго-Восточная Австралия |`https://<accountId>.webhook.ase.azure-automation.net`<br>`https://<accountId>.agentsvc.ase.azure-automation.net`<br>`https://<accountId>.jrds.ase.azure-automation.net` |
| Восточная Австралия |`https://<accountId>.webhook.ae.azure-automation.net`<br>`https://<accountId>.agentsvc.ae.azure-automation.net`<br>`https://<accountId>.jrds.ae.azure-automation.net` |
| Центральная Австралия |`https://<accountId>.webhook.ac.azure-automation.net`<br>`https://<accountId>.agentsvc.ac.azure-automation.net`<br>`https://<accountId>.jrds.ac.azure-automation.net` |
| южная часть Соединенного Королевства |`https://<accountId>.webhook.uks.azure-automation.net`<br>`https://<accountId>.agentsvc.uks.azure-automation.net`<br>`https://<accountId>.jrds.uks.azure-automation.net` |
| Центральная Франция |`https://<accountId>.webhook.fc.azure-automation.net`<br>`https://<accountId>.agentsvc.fc.azure-automation.net`<br>`https://<accountId>.jrds.fc.azure-automation.net` |
| Северная часть ЮАР; |`https://<accountId>.webhook.san.azure-automation.net`<br>`https://<accountId>.agentsvc.san.azure-automation.net`<br>`https://<accountId>.jrds.san.azure-automation.net` |
| Brazil South |`https://<accountId>.webhook.brs.azure-automation.net`<br>`https://<accountId>.agentsvc.brs.azure-automation.net`<br>`https://<accountId>.jrds.brs.azure-automation.net` |
| Северный Китай |`https://<accountId>.webhook.bjb.azure-automation.cn`<br>`https://<accountId>.agentsvc.bjb.azure-automation.cn`<br>`https://<accountId>.jrds.bjb.azure-automation.cn` |
| Северный Китай 2 |`https://<accountId>.webhook.bjs2.azure-automation.cn`<br>`https://<accountId>.agentsvc.bjs2.azure-automation.cn`<br>`https://<accountId>.jrds.bjs2.azure-automation.cn` |
| Восточный Китай 2 |`https://<accountId>.webhook.sha2.azure-automation.cn`<br>`https://<accountId>.agentsvc.sha2.azure-automation.cn`<br>`https://<accountId>.jrds.sha2.azure-automation.cn` |
| US Gov (Вирджиния) |`https://<accountId>.webhook.usge.azure-automation.us`<br>`https://<accountId>.agentsvc.usge.azure-automation.us`<br>`https://<accountId>.jrds.usge.azure-automation.us` |
| US Gov (Техас) |`https://<accountId>.webhook.ussc.azure-automation.us`<br>`https://<accountId>.agentsvc.ussc.azure-automation.us`<br>`https://<accountId>.jrds.ussc.azure-automation.us` |
| US Gov (Аризона) |`https://<accountId>.webhook.phx.azure-automation.us`<br>`https://<accountId>.agentsvc.phx.azure-automation.us`<br>`https://<accountId>.jrds.phx.azure-automation.us` |

Замените `<accountId>` в записи DNS на GUID, представляющий идентификатор учетной записи службы автоматизации, по **URL-адресу** значения. Идентификатор, необходимый для **ключей** , можно получить в разделе **Параметры учетной записи** в портал Azure.

![Страница первичного ключа учетной записи автоматизации](./media/automation-region-dns-records/automation-account-keys.png)

Скопируйте значение после *Accounts/* из поля **URL-адрес** . `https://<GUID>.agentsvc.<region>.azure-automation.net/accounts/<GUID>`

> [!NOTE]
> Все записи веб-перехватчиков и agentservice DNS были обновлены до новых записей DNS стиля для поддержки частной ссылки. Для записей DNS ЖРДС поддерживаются как старые, так и новые записи DNS в стиле. Если вы не используете частную ссылку, вы увидите записи DNS старого стиля, в то время как те, кто использует частную ссылку, увидят новый стиль записей DNS.

При определении [исключений](../automation-runbook-execution.md#exceptions) рекомендуем использовать адреса из списка. Чтобы получить список IP-адресов регионов вместо имен регионов, скачайте файл JSON из центра загрузки Майкрософт для следующих облачных сред:

* [Диапазоны IP-адресов и теги службы Azure — общедоступные](https://www.microsoft.com/download/details.aspx?id=56519)
* [Диапазоны IP-адресов и теги службы Azure — Azure для государственных организаций](https://www.microsoft.com/download/details.aspx?id=57063)
* [Диапазоны IP-адресов и теги службы Azure — Azure для Германии](https://www.microsoft.com/download/details.aspx?id=57064)
* [Диапазоны IP-адресов и теги службы Azure — Azure для Китая ВИАНЕТ 21](https://www.microsoft.com/download/details.aspx?id=57062)

В файле IP-адресов приводится список диапазонов IP-адресов, используемых в центрах обработки данных Microsoft Azure. Он включает диапазоны вычислений, SQL и хранения. Кроме того, в нем отражаются развернутые в настоящее время диапазоны и все предстоящие изменения диапазонов IP-адресов. Новые диапазоны, появившиеся в файле, не будут использоваться в центрах обработки данных по крайней мере одну неделю.

Скачивайте новый файл IP-адресов каждую неделю. и вносите соответствующие изменения на своем сайте, чтобы правильно определять службы, выполняемые в Azure.

> [!NOTE]
> Если вы используете Azure ExpressRoute, обратите внимание, что этот файл используется для того, чтобы обновлять объявление протокола BGP в пространстве Azure в первую неделю каждого месяца.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения см. в статье [Устранение неполадок с гибридными рабочими ролями Runbook](../troubleshoot/hybrid-runbook-worker.md#general).

* Сведения об устранении неполадок, связанных с конфигурацией состояния, см. в разделе [Устранение неполадок конфигурации состояния](../troubleshoot/desired-state-configuration.md).
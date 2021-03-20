---
title: Виртуальный рабочий стол RDP Шортпас (Предварительная версия)
titleSuffix: Azure
description: Как настроить RDP Шортпас (Предварительная версия) для виртуальных рабочих столов Windows.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: 295a46f6d1074ddf8422233ea3ccfa4d65c28fd8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100571584"
---
# <a name="windows-virtual-desktop-rdp-shortpath-preview"></a>Виртуальный рабочий стол RDP Шортпас (Предварительная версия)

> [!IMPORTANT]
> Протокол RDP Шортпас в настоящее время находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания, и мы не рекомендуем использовать ее для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

RDP Шортпас — это функция виртуального рабочего стола Windows, которая устанавливает прямой транспорт на основе UDP между удаленный рабочий стол клиентом и узлом сеансов. Протокол RDP использует этот транспорт для доставки удаленный рабочий стол и RemoteApp, обеспечивая более высокую надежность и постоянную задержку.

## <a name="key-benefits"></a>Основные преимущества

* Транспорт RDP Шортпас основан на высокоэффективном  [протоколе управления универсальной частотой (уркп)](https://www.microsoft.com/en-us/research/publication/urcp-universal-rate-control-protocol-for-real-time-communication-applications/). УРКП расширяет возможности протокола UDP с помощью активного мониторинга состояния сети и обеспечивает полное использование ссылок. УРКП работает с низкими задержками и уровнями потери по мере необходимости удаленный рабочий стол. УРКП обеспечивает наилучшую производительность, динамически изучая сетевые параметры и предоставляя протокол с помощью механизма управления скоростью.
* RDP Шортпас устанавливает прямое подключение между клиентом удаленный рабочий стол и узлом сеансов. Прямое подключение снижает зависимость от шлюзов виртуальных рабочих столов Windows, повышает надежность подключения и увеличивает пропускную способность, доступную для каждого сеанса пользователя.
* Удаление дополнительного ретранслятора сокращает время кругового пути, что повышает удобство работы пользователей с приложениями с учетом задержки и методами ввода.
* RDP Шортпас предоставляет поддержку для [настройки приоритета качества обслуживания (QoS)](./rdp-quality-of-service-qos.md) для подключений по протоколу RDP через метки Приоритизированные службы кода (DSCP).
* Транспорт Шортпас RDP позволяет [ограничить исходящий сетевой трафик](./rdp-bandwidth.md#limit-network-bandwidth-use-with-throttle-rate) , указав частоту регулирования для каждого сеанса.

## <a name="connection-security"></a>Безопасность подключения

RDP Шортпас расширяет возможности многотранспорта RDP. Он не заменяет транспорт обратных подключений, но дополняет его. Все начальные брокеры сеансов управляются с помощью инфраструктуры виртуальных рабочих столов Windows.

UDP-порт 3390 используется только для входящего трафика Шортпас, прошедшего проверку подлинности через транспорт с обратным соединением. Прослушиватель RDP Шортпас игнорирует все попытки подключения к прослушивателю, если они не соответствуют сеансу обратного подключения.

Протокол RDP Шортпас использует TLS-соединение между клиентом и узлом сеансов, используя сертификаты узла сеанса. По умолчанию сертификат, используемый для шифрования RDP, создается автоматически операционной системой во время развертывания. При необходимости клиенты могут развертывать централизованно управляемые сертификаты, выданные центром сертификации предприятия. Дополнительные сведения о конфигурациях сертификатов см. в [документации по Windows Server](/troubleshoot/windows-server/remote/remote-desktop-listener-certificate-configurations).

## <a name="rdp-shortpath-connection-sequence"></a>Последовательность подключений RDP Шортпас

После установки [транспорта с обратным соединением](./network-connectivity.md)клиент и узел сеансов устанавливают подключение RDP и согласовывают возможности множественного транспорта. Ниже приведены дополнительные действия.

1. Узел сеанса отправляет клиенту список частных и общедоступных адресов IPv4 и IPv6.
2. Клиент запускает фоновый поток для установки параллельного транспортного транспорта на основе UDP непосредственно на один из IP-адресов узла.
3. Пока клиент выполняет проверку предоставленных IP-адресов, он возобновляет начальное подключение через транспорт обратных подключений, чтобы не допустить задержки в подключении пользователя.
4. Если у клиента есть прямая линия анализа и конфигурация брандмауэра правильная, клиент устанавливает безопасное TLS-соединение с узлом сеансов.
5. После установки транспорта Шортпас протокол RDP перемещает все динамические виртуальные каналы (DVCs), включая удаленную графику, входные данные и перенаправление устройств в новый транспорт.
6. Если брандмауэр или топология сети не позволят клиенту устанавливать прямое подключение по протоколу UDP, протокол RDP продолжит работать с транспортом обратного подключения.

На приведенной ниже схеме представлен общий обзор сетевого подключения RDP Шортпас.

:::image type="content" source="media/rdp-shortpath-connections.svg" alt-text="Схема сетевых подключений RDP Шортпас" lightbox="media/rdp-shortpath-connections.svg":::

## <a name="requirements"></a>Требования

Для поддержки протокола RDP Шортпас клиенту виртуальных рабочих столов Windows требуется прямая информация об узле сеансов. Вы можете получить прямую информацию, используя одну из следующих технологий:

* [Частный пиринг ExpressRoute](../expressroute/expressroute-circuit-peerings.md)
* [VPN типа "сеть — сеть" (на основе IPsec)](../vpn-gateway/tutorial-site-to-site-portal.md)
* [VPN типа "точка — сеть" (на основе IPsec)](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)
* [Присвоение общедоступного IP-адреса](../virtual-network/virtual-network-public-ip-address.md)

Если вы используете другие типы VPN для подключения к виртуальной сети Azure, мы рекомендуем использовать VPN на основе UDP для получения наилучших результатов. Хотя большинство VPN-решений на основе TCP инкапсулируют все IP-пакеты, включая UDP, они добавляют наследуемые издержки на управление перегрузкой TCP, что замедляет производительность RDP.

Прямая линия анализа означает, что брандмауэры не блокируют UDP-порт 3390 и клиент может подключиться непосредственно к узлу сеансов.

## <a name="enabling-rdp-shortpath-preview"></a>Включение предварительной версии RDP Шортпас

Чтобы принять участие в предварительной версии RDP Шортпас, необходимо включить прослушиватель RDP Шортпас на узле сеансов. Вы можете включить RDP Шортпас на любом количестве узлов сеансов, используемых в вашей среде. Включать RDP Шортпас на всех узлах пула не требуется.
Чтобы включить прослушиватель Шортпас, необходимо настроить следующие параметры реестра:

> [!WARNING]
> При неправильном изменении реестра с помощью редактора реестра или другого метода могут возникнуть серьезные проблемы. Эти проблемы могут потребовать переустановки операционной системы. Корпорация Майкрософт не гарантирует, что такие неполадки могут быть устранены. Ответственность за изменение реестра лежит на пользователе.

1. На узле сеансов запустите Regedit.exe, а затем перейдите к следующему расположению:

    ```cmd
    HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations
    ```

2. Создать новое значение **DWORD** с именем **фусеудппортредиректор** и установить его в **1** (десятичное)
3. Создайте новое значение **DWORD** с именем **удппортнумбер** и установите его в **3390** (десятичное число).
4. Закройте редактор реестра.
5. Перезапуск узла сеансов

Чтобы задать следующие значения реестра, можно также выполнить следующие командлеты в окне PowerShell с повышенными привилегиями:

```powershell
$WinstationsKey = 'HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations'
New-ItemProperty -Path $WinstationsKey -Name 'fUseUdpPortRedirector' -ErrorAction:SilentlyContinue -PropertyType:dword -Value 1 -Force
New-ItemProperty -Path $WinstationsKey -Name 'UdpPortNumber' -ErrorAction:SilentlyContinue -PropertyType:dword -Value 3390 -Force
```

Для настройки групповой политики можно также использовать PowerShell.

```powershell
# Replace $domainName value with the name of your Active Directory domain
# Replace $policyName value with the name of existing Group Policy Object
$domainName = "contoso.com"
$policyName = "RDP Shortpath Policy"
Set-GPPrefRegistryValue -Key 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations' -ValueName 'fUseUdpPortRedirector' -Value 1 -Type:DWord  -Action:Create -Context:Computer -Name $policyName -Domain $domainName
Set-GPPrefRegistryValue -Key 'HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations' -ValueName 'UdpPortNumber' -Value 3390 -Type:DWord  -Action:Create -Context:Computer -Name $policyName -Domain $domainName
```

## <a name="configure-windows-defender-firewall-with-advanced-security"></a>Настройка брандмауэра защитника Windows в повышенной безопасности

Чтобы разрешить входящий сетевой трафик для RDP Шортпас, используйте узел Брандмауэр защитника Windows в области повышенной безопасности оснастки MMC "Управление групповая политика" для создания правил брандмауэра.

1. Откройте консоль управления групповыми политиками [брандмауэр защитника Windows в области повышенной безопасности](/windows/security/threat-protection/windows-firewall/open-the-group-policy-management-console-to-windows-firewall-with-advanced-security).
2. В области навигации выберите правила для **входящих подключений**.
3. Выберите **действие** и нажмите кнопку **создать правило**.
4. На странице **тип правила** в мастере создания правила для нового входящего подключения выберите **Настраиваемый** и нажмите кнопку **Далее**.
5. На странице **Программа** выберите **путь к этой программе** и введите "% SystemRoot% \system32\svchost.exe", а затем нажмите кнопку **Далее**.
6. На странице **протокол и порты** выберите тип протокола UDP. В поле **локальный порт** выберите определенные порты и введите 3390.
7. На странице **Область** можно указать, что правило применяется только к сетевому трафику, который направляется на IP-адреса, введенные на этой странице, или с них. Настройте в соответствии с разработкой, а затем нажмите кнопку **Далее**.
8. На странице **действие** выберите **Разрешить подключение**, а затем нажмите кнопку **Далее**.
9. На странице **профиль** выберите типы сетевых расположений, к которым применяется это правило, а затем нажмите кнопку **Далее**.
10. На странице **имя** введите имя и описание правила, а затем нажмите кнопку **Готово**.

Вы можете проверить, что новое правило соответствует снимкам экрана ниже: :::image type="content" source="media/rdp-shortpath-firewall-general-tab.png" alt-text="снимок экрана вкладки Общие для настройки брандмауэра для сетевых подключений RDP шортпас" lightbox="media/rdp-shortpath-firewall-general-tab.png"::: .

:::image type="content" source="media/rdp-shortpath-firewall-service-settings.png" alt-text="Снимок экрана: вкладка &quot;программы и службы&quot; для настройки брандмауэра для сетевых подключений RDP Шортпас" lightbox="media/rdp-shortpath-firewall-service-settings.png":::

:::image type="content" source="media/rdp-shortpath-firewall-protocol-and-ports.png" alt-text="Снимок экрана: вкладка &quot;протоколы и порты&quot; настройки брандмауэра для сетевых подключений RDP Шортпас" lightbox="media/rdp-shortpath-firewall-protocol-and-ports.png":::

Для настройки брандмауэра Windows можно также использовать PowerShell:

```powershell
New-NetFirewallRule -DisplayName 'Remote Desktop - Shortpath (UDP-In)'  -Action Allow -Description 'Inbound rule for the Remote Desktop service to allow RDP traffic. [UDP 3390]' -Group '@FirewallAPI.dll,-28752' -Name 'RemoteDesktop-UserMode-In-Shortpath-UDP'  -PolicyStore PersistentStore -Profile Domain, Private -Service TermService -Protocol udp -LocalPort 3390 -Program '%SystemRoot%\system32\svchost.exe' -Enabled:True
```

### <a name="using-powershell-to-configure-windows-defender-firewall"></a>Настройка брандмауэра защитника Windows с помощью PowerShell

Для настройки групповой политики можно также использовать PowerShell.

```powershell
# Replace $domainName value with the name of your Active Directory domain
# Replace $policyName value with the name of existing Group Policy Object
$domainName = "contoso.com"
$policyName = "RDP Shortpath Policy"
$gpoSession = Open-NetGPO -PolicyStore "$domainName\$policyName"
New-NetFirewallRule -DisplayName 'Remote Desktop - Shortpath (UDP-In)'  -Action Allow -Description 'Inbound rule for the Remote Desktop service to allow RDP traffic. [UDP 3390]' -Group '@FirewallAPI.dll,-28752' -Name 'RemoteDesktop-UserMode-In-Shortpath-UDP' -Profile Domain, Private -Service TermService -Protocol udp -LocalPort 3390 -Program '%SystemRoot%\system32\svchost.exe' -Enabled:True -GPOSession $gpoSession
Save-NetGPO -GPOSession $gpoSession
```

## <a name="configuring-azure-network-security-group"></a>Настройка группы безопасности сети Azure

Чтобы разрешить доступ к прослушивателю RDP Шортпас через границы безопасности сети, необходимо настроить группу безопасности сети Azure, разрешающую входящий UDP-порт 3390.
Следуйте указаниям в [документации по группе безопасности сети](../virtual-machines/windows/nsg-quickstart-portal.md) , чтобы создать правило безопасности для входящего трафика, разрешающее трафик со следующими параметрами:

* **Исходный код**  -  **Любой** или диапазон IP-адресов, в которых направляются клиенты
* **Диапазоны исходных портов** - **\***
* **Назначение**  -  **Любой**
* **Диапазоны**  -  целевых портов **3390**
* **Протокол**  -  **UDP-порт**
* **Действие**  -  **Разрешить**
* При необходимости измените **приоритет**. Приоритет влияет на порядок, в котором применяются правила: чем ниже числовое значение, тем раньше применяется правило.
* **Имя** -- **RDP шортпас**

### <a name="disabling-rdp-shortpath-for-a-specific-subnet"></a>Отключение RDP Шортпас для определенной подсети

Если необходимо заблокировать определенные подсети с помощью транспорта RDP Шортпас, можно настроить дополнительные группы безопасности сети, указав диапазоны исходных IP-адресов.

## <a name="verifying-the-connectivity"></a>Проверка подключения

### <a name="using-connection-information-dialog"></a>Диалоговое окно «использование сведений о соединении»

Чтобы убедиться, что подключения используют RDP Шортпас, откройте диалоговое окно "сведения о подключении", щелкнув значок антенны на панели инструментов подключения.

:::image type="content" source="media/rdp-shortpath-connection-bar.png" alt-text="Изображение панели подключение к удаленному рабочему столу":::

:::image type="content" source="media/rdp-shortpath-connection-info.png" alt-text="Изображение диалогового окна сведений о подключение к удаленному рабочему столу":::

### <a name="using-event-logs"></a>Использование журналов событий

Чтобы убедиться, что сеанс использует транспорт Шортпас RDP, выполните следующие действия.

1. Подключитесь к рабочему столу виртуальной машины с помощью клиента Виртуального рабочего стола Windows.
2. Запустите Просмотр событий и перейдите к следующему узлу: **журналы приложений и служб > microsoft > Windows > RemoteDesktopServices-RdpCoreCDV > Microsoft-Windows-ремотедесктопсервицес-рдпкорекдв/эксплуатация** .
3. Чтобы определить, используется ли транспорт RDP Шортпас, найдите событие с ИДЕНТИФИКАТОРом 131.

### <a name="using-log-analytics-to-verify-shortpath-connectivity"></a>Проверка подключения Шортпас с помощью Log Analytics

Если вы используете [Azure log Analytics](./diagnostics-log-analytics.md), вы можете отслеживать соединения, запрашивая [таблицу ввдконнектионс](/azure/azure-monitor/reference/tables/wvdconnections). Столбец с именем Удпусе указывает, использует ли стек RDP виртуальных рабочих столов Windows протокол UDP при подключении к текущему пользователю.
Допустимые значения:

* **0** — подключение пользователя не использует RDP шортпас
* **1** — подключение пользователя использует RDP шортпас
  
Следующий список запросов позволяет просматривать сведения о подключении. Этот запрос можно выполнить в [редакторе запросов log Analytics](../azure-monitor/logs/log-analytics-tutorial.md#write-a-query). Для каждого запроса замените `userupn` именем участника-пользователя, которого требуется найти.

```kusto
let Events = WVDConnections | where UserName == "userupn" ;
Events
| where State == "Connected"
| project CorrelationId , UserName, ResourceAlias , StartTime=TimeGenerated, UdpUse, SessionHostName, SessionHostSxSStackVersion
| join (Events
| where State == "Completed"
| project EndTime=TimeGenerated, CorrelationId, UdpUse)
on CorrelationId
| project StartTime, Duration = EndTime - StartTime, ResourceAlias, UdpUse,  SessionHostName, SessionHostSxSStackVersion
| sort by StartTime asc
```

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="verify-shortpath-listener"></a>Проверка прослушивателя Шортпас

Чтобы убедиться, что прослушиватель UDP включен, используйте следующую команду PowerShell на узле сеанса:

```powershell
Get-NetUDPEndpoint -OwningProcess ((Get-WmiObject win32_service -Filter "name = 'TermService'").ProcessId)  -LocalPort 3390
```

Если параметр включен, вы увидите выходные данные следующего вида:

```dos
LocalAddress                             LocalPort
------------                             ---------
::                                       3390
0.0.0.0                                  3390
```

При возникновении конфликта можно задать процесс, занимающий порт, с помощью следующей команды.

```powershell
Get-Process -id (Get-NetUDPEndpoint  -LocalPort 3390 -LocalAddress 0.0.0.0).OwningProcess
```

## <a name="disabling-rdp-shortpath"></a>Отключение RDP Шортпас

В некоторых случаях может потребоваться отключить транспорт RDP Шортпас. Вы можете отключить RDP Шортпас с помощью групповой политики.

### <a name="disabling-rdp-shortpath-on-the-client"></a>Отключение RDP Шортпас на клиенте

Чтобы отключить RDP Шортпас для конкретного клиента, можно отключить поддержку UDP с помощью следующих групповая политика.

1. На клиенте запустите **gpedit. msc**.
2. Последовательно выберите **Конфигурация компьютера > Шаблоны администрирования > компоненты Windows > службы удаленных рабочих столов > подключение к удаленному рабочему столу клиент**.
3. Установите для параметра **"отключить UDP на клиенте" значение "** **включено** ".

### <a name="disabling-rdp-shortpath-on-the-session-host"></a>Отключение RDP Шортпас на узле сеансов

Чтобы отключить RDP Шортпас для определенного узла сеансов, можно отключить поддержку UDP с помощью следующих групповая политика.

1. На узле сеансов запустите **gpedit. msc**.
2. Последовательно выберите **Конфигурация компьютера > Шаблоны администрирования > компоненты Windows > службы удаленных рабочих столов > подключение к удаленному рабочему столу узлов > подключения**.
3. Установите для параметра **"Выбор протоколов транспорта RDP"** значение **только TCP** .

## <a name="public-preview-feedback"></a>Отзывы о общедоступной предварительной версии

Мы хотели бы узнать о своих впечатлениях в этой общедоступной предварительной версии!
* Для вопросов, запросов, комментариев и других отзывов [Используйте эту форму обратной связи](https://aka.ms/RDPShortpathFeedback).

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о сетевом подключении к виртуальным рабочим столам Windows [см.](network-connectivity.md)
* Чтобы приступить к работе со службой QoS для виртуальных рабочих столов Windows, см. статью [Реализация качества обслуживания (QoS) для виртуальных рабочих столов Windows](rdp-quality-of-service-qos.md).
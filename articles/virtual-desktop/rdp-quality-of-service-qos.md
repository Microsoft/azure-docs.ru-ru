---
title: Реализация качества обслуживания (QoS) для виртуальных рабочих столов Windows (Предварительная версия)
titleSuffix: Azure
description: Как настроить QoS (Предварительная версия) для виртуальных рабочих столов Windows.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: b61faf74d96e2571e91f7bf9d10eac88cdbf8345
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94639373"
---
# <a name="implement-quality-of-service-qos-for-windows-virtual-desktop-preview"></a>Реализация качества обслуживания (QoS) для виртуальных рабочих столов Windows (Предварительная версия)

> [!IMPORTANT]
> Поддержка политики качества обслуживания для виртуальных рабочих столов Windows в настоящее время доступна в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания, и мы не рекомендуем использовать ее для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[RDP шортпас](./shortpath.md) предоставляет прямой транспорт на основе UDP между удаленный рабочий стол клиентом и узлом сеансов. RDP Шортпас включает настройку политик качества обслуживания для данных RDP.
Качество обслуживания в виртуальных рабочих столах Windows позволяет выполнять в режиме реального времени трафик RDP, чувствительный к задержкам в сети, для снижения трафика, который менее чувствителен к минимуму. Примером такого менее конфиденциального трафика будет загрузка нового приложения, где вторая секунда для загрузки не является большой. Служба QoS использует объекты Windows групповая политика для выявления и отметки всех пакетов в потоках в режиме реального времени и помогает сети предоставлять трафик RDP выделенной части пропускной способности.

Если вы поддерживаете большую группу пользователей, на которых возникли проблемы, описанные в этой статье, вам, вероятно, потребуется реализовать QoS. Для малого бизнеса с небольшим количеством пользователей может не потребоваться качество обслуживания, но оно должно быть полезным даже там.

Без какой бы то ни было формы качества обслуживания вы можете столкнуться со следующими проблемами:

* Нарушение — пакеты RDP поступают по разным тарифам, что может привести к невозможности визуальных и звуковых сбоев
* Потеря пакетов — пакеты отбрасываются, что приводит к повторной передаче, требующей дополнительного времени
* Отложенное время приема-передачи (RTT) — пакеты RDP занимают много времени, что приводит к заметным задержкам между вводом и реакцией удаленного приложения.

Самый сложный способ решения этих проблем заключается в увеличении размера подключения к данным как внутри, так и из Интернета. Так как это часто неэкономично, QoS предоставляет способ управления ресурсами, а не более эффективное использование пропускной способности. Для устранения проблем с качеством рекомендуется сначала использовать службу QoS, а затем добавить пропускную способность только там, где это необходимо.

Чтобы качество обслуживания вступило в силу, необходимо применить согласованные параметры QoS во всей Организации. Любая часть пути, которая не поддерживает приоритеты QoS, может привести к снижению качества сеанса RDP.

## <a name="introduction-to-qos-queues"></a>Общие сведения об очередях QoS

Для обеспечения качества обслуживания сетевые устройства должны иметь способ классификации трафика и должны иметь возможность отличать RDP от другого сетевого трафика.

Когда сетевой трафик входит в маршрутизатор, трафик помещается в очередь. Если политика качества обслуживания не настроена, то существует только одна очередь, и все данные рассматриваются как "первым помещается", первым — с тем же приоритетом. Это означает, что трафик RDP может накладываться за трафик, в котором задержка в несколько миллисекунд не будет проблемой.

При реализации QoS вы определяете несколько очередей с помощью одной из нескольких функций управления перегрузкой, таких как очередь приоритетов Cisco и [взвешенное справедливое помещение на основе классов (кбвфк)](https://www.cisco.com/en/US/docs/ios/12_0t/12_0t5/feature/guide/cbwfq.html#wp17641) и функции предотвращения перегрузки, такие как [взвешенное случайное обнаружение (вред)](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_conavd/configuration/15-mt/qos-conavd-15-mt-book/qos-conavd-cfg-wred.html).

Простой аналог заключается в том, что QoS создает виртуальные "carpoolные желобы" в сети данных. Поэтому некоторые типы данных никогда или редко сталкиваются с задержкой. После создания этих дорожек можно изменить их относительный размер и значительно повысить эффективность управления пропускной способностью подключения, сохраняя при этом бизнес-возможности для пользователей вашей организации.

## <a name="qos-implementation-checklist"></a>Контрольный список реализации QoS

На высоком уровне выполните следующие действия, чтобы реализовать качество обслуживания:

1. [Убедитесь, что сеть готова](#make-sure-your-network-is-ready)
2. [Убедитесь, что протокол RDP шортпас включен](./shortpath.md) — политики QoS не поддерживаются для транспорта с обратным подключением.
3. [Реализация вставки маркеров DSCP](#insert-dscp-markers) на узлах сеансов

При подготовке к реализации QoS учитывайте следующие рекомендации.

* Лучше использовать самый короткий путь к узлу сеансов
* Не рекомендуется использовать любые препятствия, такие как прокси или устройства проверки пакетов.

## <a name="make-sure-your-network-is-ready"></a>Убедитесь, что сеть готова

Если вы планируете реализовать качество обслуживания, вы должны заранее определить требования к пропускной способности и другие [требования к сети](/windows-server/remote/remote-desktop-services/network-guidance?context=/azure/virtual-desktop/context/context).
  
Перегрузка трафика по сети существенно повлияет на качество мультимедиа. Недостаток пропускной способности приводит к снижению производительности и плохому интерфейсу пользователя. По мере роста внедрения и использования виртуальных рабочих столов Windows используйте [log Analytics](./diagnostics-log-analytics.md) для выявления проблем, а затем вносите изменения с помощью QoS и выборочной пропускной способности.

### <a name="vpn-considerations"></a>Рекомендации по VPN

Служба QoS работает должным образом только при реализации по всем связям между клиентами и узлами сеансов. Если вы используете службу QoS во внутренней сети и пользователь входит в удаленное расположение, вы можете определить приоритеты только внутри внутренней управляемой сети. Хотя удаленные расположения могут получить управляемое подключение путем реализации виртуальной частной сети (VPN), VPN по сути добавляет нагрузку на пакеты и создает задержки в трафике в режиме реального времени.

В глобальной Организации с управляемыми связями, охватывающими континенты, настоятельно рекомендуется использовать QoS, так как пропускная способность для этих ссылок ограничена по сравнению с локальной сетью.

## <a name="insert-dscp-markers"></a>Вставить маркеры DSCP

Можно реализовать QoS с помощью объекта групповая политика (GPO), чтобы узлы сеансов могли вставить маркер DSCP в заголовки IP-пакетов, определяющие его как конкретный тип трафика. Маршрутизаторы и другие сетевые устройства могут быть настроены на распознавание этих пометок и помещают трафик в отдельную очередь с более высоким приоритетом.

Можно сравнить метки DSCP с метками почтовых отправок, которые показывают работникам, насколько срочно доставляется, и как лучше отсортировать их для ускорения доставки. После настройки сети для предоставления приоритета потокам удаленного рабочего стола, потерянные пакеты и пакеты с опозданием должны существенно уменьшиться.

После того как все сетевые устройства используют одни и те же классификации, маркировки и приоритеты, можно сократить или исключить задержки, пропущенные пакеты и нарушение. С точки зрения протокола RDP, необходимый этап настройки представляет собой классификацию и маркировку пакетов. Однако для успешного завершения качества обслуживания необходимо тщательно согласовать конфигурацию протокола удаленного рабочего стола с базовой конфигурацией сети.
Значение DSCP указывает соответствующей настроенной сети приоритет для предоставления пакета или потока.

Мы рекомендуем использовать значение DSCP 46, которое сопоставляется с классом DSCP для **ускоренной пересылки (EF)** .

### <a name="implement-qos-on-session-host-using-group-policy"></a>Реализация QoS на узле сеансов с помощью групповая политика

Для установки предопределенного значения DSCP можно использовать службу качества обслуживания на основе политик (QoS) в групповая политика.

Чтобы создать политику качества обслуживания для узлов сеансов, присоединенных к домену, сначала войдите на компьютер, на котором установлено групповая политика Management. Откройте управление групповая политика (нажмите кнопку Пуск, выберите пункт Администрирование, а затем выберите групповая политика управление) и выполните следующие действия.

1. В групповая политика управление выберите контейнер, в котором должна быть создана новая политика. Например, если весь сеанс содержит компьютеры, расположенные в подразделении с именем **"узлы сеансов"**, Новая политика должна быть создана в подразделении узла сеанса.

2. Щелкните правой кнопкой мыши соответствующий контейнер, а затем выберите **создать объект GPO в этом домене и свяжите его**.

3. В диалоговом окне **Создание объекта групповой политики** введите имя нового объекта Групповая политика в поле **имя** , а затем нажмите кнопку **ОК**.

4. Щелкните правой кнопкой мыши только что созданную политику и выберите **изменить**.

5. В редактор "Управление групповыми политиками" разверните узел **Конфигурация компьютера**, разверните узел **Параметры Windows**, щелкните правой кнопкой мыши элемент **QoS на основе политики** и выберите команду **создать новую политику**.

6. В диалоговом окне Служба **QoS на основе политик** на странице Открытие введите имя новой политики в поле **имя** . Выберите **указать значение DSCP** и установите значение **46**. Не устанавливайте флажок **задать частоту исходящих подключений** , а затем нажмите кнопку **Далее**.

7. На следующей странице выберите **только приложения с именем исполняемого файла** и введите имя **svchost.exe**, а затем нажмите кнопку **Далее**. Этот параметр указывает политике назначить только приоритет трафика, соответствующего службе удаленный рабочий стол.

8. На третьей странице убедитесь, что выбраны **все исходные IP-адреса** и **IP-адрес назначения** , а затем нажмите кнопку **Далее**. Эти два параметра гарантируют, что пакеты будут управляться независимо от того, на каком компьютере (IP-адрес) отправляются пакеты, а какой компьютер (IP-адрес) получит пакеты.

9. На четвертой странице выберите **UDP** в раскрывающемся списке **выберите протокол, к которому применяется политика QoS** .

10. Под заголовком **укажите номер исходного порта**, выберите **из этого исходного порта или диапазона**. В сопроводительном текстовом поле введите **3390**. Нажмите кнопку **Готово**.

Новые политики, которые вы создали, вступят в силу только после обновления групповая политика на компьютерах узла сеансов. Хотя групповая политика периодически обновляет, можно принудительно выполнить немедленное обновление, выполнив следующие действия.

1. На каждом узле сеансов, для которого требуется обновить групповая политика, откройте командную строку от имени администратора (*Запуск от имени администратора*).

1. В командной строке введите

   ```console
   gpupdate /force
   ```

### <a name="implement-qos-on-session-host-using-powershell"></a>Реализация QoS на узле сеансов с помощью PowerShell

Вы можете настроить QoS для RDP Шортпас с помощью следующего командлета PowerShell:

```powershell
New-NetQosPolicy -Name "RDP Shortpath" -AppPathNameMatchCondition "svchost.exe" -IPProtocolMatchCondition UDP -IPSrcPortStartMatchCondition 3390 -IPSrcPortEndMatchCondition 3390 -DSCPAction 46 -NetworkProfile All
```

## <a name="related-articles"></a>Похожие статьи

* [Политика качества обслуживания (QoS)](/windows-server/networking/technologies/qos/qos-policy-top)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о требованиях к пропускной способности виртуальных рабочих столов Windows см. в статье [Общие сведения о требованиях к пропускной способности протокол удаленного рабочего стола (RDP) для виртуальных рабочих](rdp-bandwidth.md)
* Дополнительные сведения о сетевом подключении к виртуальным рабочим столам Windows [см.](network-connectivity.md)

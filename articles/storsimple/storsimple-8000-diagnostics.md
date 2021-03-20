---
title: Средство диагностики для устранения неполадок устройства StorSimple 8000 | Документация Майкрософт
description: В этой статье представлен обзор средства диагностики и описание его использования с устройством StorSimple.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: bf0570cd62a2c329407cfb6dd087020f53cdc991
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94954075"
---
# <a name="use-the-storsimple-diagnostics-tool-to-troubleshoot-8000-series-device-issues"></a>Устранение неполадок устройства серии 8000 с помощью средства диагностики StorSimple

## <a name="overview"></a>Обзор

Средство диагностики StorSimple диагностирует проблемы, связанные с системой, производительностью, сетью и работоспособностью компонентов оборудования устройства StorSimple. Средство диагностики можно использовать в различных сценариях, таких как планирование рабочей нагрузки, развертывание устройства StorSimple, оценка сетевой среды и определение производительности работающего устройства. В этой статье представлен обзор средства диагностики и описание его использования с устройством StorSimple.

Средство диагностики в основном предназначено для локальных устройств StorSimple серии 8000 (8100 и 8600).

## <a name="run-diagnostics-tool"></a>Запуск средства диагностики

Это средство можно запустить из интерфейса Windows PowerShell устройства StorSimple. Доступ к локальному интерфейсу устройства можно получить двумя способами:

* [подключение к последовательной консоли устройства с помощью PuTTY](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console);
* [удаленное подключение к устройству через Windows PowerShell для StorSimple](storsimple-8000-remote-connect.md).

В этой статье предполагается, что вы подключились к последовательной консоли устройства с помощью PuTTY.

#### <a name="to-run-the-diagnostics-tool"></a>Запуск средства диагностики

Подключившись к интерфейсу Windows PowerShell устройства, выполните командлет, сделав следующее.
1. Войдите в последовательную консоль устройства, выполнив действия, описанные в разделе [Использование PuTTY для подключения к последовательной консоли устройства](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).

2. Введите следующую команду:

    `Invoke-HcsDiagnostics`

    Если параметр области не указан, командлет выполняет все диагностические тесты (проверка системы, работоспособности компонента оборудования, сети и производительности). 
    
    Чтобы выполнить определенный тест, укажите параметр области. Например, чтобы проверить только сеть, введите следующую команду:

    `Invoke-HcsDiagnostics -Scope Network`

3. Выберите и скопируйте выходные данные из окна PuTTY в текстовый файл для дальнейшего анализа.

## <a name="scenarios-to-use-the-diagnostics-tool"></a>Сценарии использования средства диагностики

Средство диагностики позволяет устранять неполадки сети, производительности, системы и работоспособности системного оборудования. Ниже приведено несколько сценариев.

* **Устройство не в сети.** Устройство StorSimple серии 8000 находится в автономном режиме. Однако в интерфейсе Windows PowerShell видно, что оба контроллера запущены и работают.
    * Это средство можно использовать для определения состояния сети.
         
         > [!NOTE]
         > Не используйте это средство для оценки параметров производительности и сетевых параметров на устройстве перед регистрацией (или настройкой с помощью мастера установки). Во время регистрации и работы с мастером настройки устройству назначается допустимый IP-адрес. Этот командлет можно выполнить на незарегистрированном устройстве для проверки работоспособности оборудования и системы. Используйте параметр области в следующих сценариях:
         >
         > `Invoke-HcsDiagnostics -Scope Hardware`
         >
         > `Invoke-HcsDiagnostics -Scope System`

* **Постоянные проблемы с устройством.** Вы столкнулись с проблемами устройства, которые невозможно устранить. Например, сбой регистрации. Кроме того, проблемы с устройством могут возникнуть после того, как оно будет зарегистрировано и поработает некоторое время.
    * В этом случае используйте это средство для предварительного устранения неполадок, прежде чем отправлять запрос на поддержку Майкрософт. Мы рекомендуем запустить это средство и сохранить его выходные данные. Затем их можно предоставить в службу поддержки, чтобы ускорить процесс устранения неполадок.
    * В случае сбоев компонентов оборудования или кластера необходимо отправить запрос на поддержку.

* **Низкий уровень производительности устройства.** Устройство Your StorSimple медленно работает.
    * В этом случае выполните командлет с параметром области производительности. Проанализируйте выходные данные. Вы получите сведения о задержке при выполнении операций чтения и записи в облаке. Используйте полученные значения задержки в качестве максимально достижимого целевого показателя, учтите запас на внутреннюю обработку данных и разверните рабочие нагрузки в системе. Дополнительные сведения см. в разделе [Проверка сети](#network-test).


## <a name="diagnostics-test-and-sample-outputs"></a>Диагностическая проверка и пример выходных данных

### <a name="hardware-test"></a>Проверка оборудования

В ходе этой проверки определяется состояние компонентов оборудования, встроенного ПО USM и встроенного ПО дисков в системе.

* Указываются компоненты оборудования, которые не прошли проверку или которых нет в системе.
* Определяются версии встроенного ПО USM и диска для контроллера 0 и 1 и общих компонентов в системе. Полный список компонентов оборудования см. в следующих разделах:

    * [Список компонентов для основного корпуса устройства StorSimple](storsimple-8000-monitor-hardware-status.md#component-list-for-primary-enclosure-of-storsimple-device)
    * [Список компонентов для корпуса EBOD устройства StorSimple](storsimple-8000-monitor-hardware-status.md#component-list-for-ebod-enclosure-of-storsimple-device)

> [!NOTE]
> Если в ходе проверки оборудования обнаружены неисправные компоненты, [отправьте запрос на поддержку Майкрософт](storsimple-8000-contact-microsoft-support.md).

#### <a name="sample-output-of-hardware-test-run-on-an-8100-device"></a>Пример выходных данных проверки оборудования устройства серии 8100

Ниже приведен пример выходных данных устройства StorSimple 8100. В модели устройства серии 8100 корпус EBOD не используется. Следовательно, компоненты контроллера EBOD не указываются.

```
Controller0>Invoke-HcsDiagnostics -Scope Hardware
Running hardware diagnostics ...
--------------------------------------------------
Hardware components failed or not present
----------------------

           Type           State      Controller           Index     EnclosureId
           ----           -----      ----------           -----     -----------
...rVipResource      NotPresent            None               1            None
...rVipResource      NotPresent            None               2            None
...rVipResource      NotPresent            None               3            None
...rVipResource      NotPresent            None               4            None
...rVipResource      NotPresent            None               5            None
...rVipResource      NotPresent            None               6            None
...rVipResource      NotPresent            None               7            None
...rVipResource      NotPresent            None               8            None
...rVipResource      NotPresent            None               9            None
...rVipResource      NotPresent            None              10            None
...rVipResource      NotPresent            None              11            None

Firmware information
----------------------
TalladegaController : ActiveBIOS:0.45.0010
                      BackupBIOS:0.45.0006
                      MainCPLD:17.0.000b
                      ActiveBMCRoot:2.0.001F
                      BackupBMCRoot:2.0.001F
                      BMCBoot:2.0.0002
                      LsiFirmware:20.00.04.00
                      LsiBios:07.37.00.00
                      Battery1Firmware:06.2C
                      Battery2Firmware:06.2C
                      DomFirmware:X231600
                      CanisterFirmware:3.5.0.56
                      CanisterBootloader:5.03
                      CanisterConfigCRC:0x9134777A
                      CanisterVPDStructure:0x06
                      CanisterGEMCPLD:0x19
                      CanisterVPDCRC:0x142F7DC2
                      MidplaneVPDStructure:0x0C
                      MidplaneVPDCRC:0xA6BD4F64
                      MidplaneCPLD:0x10
                      PCM1Firmware:1.00|1.05
                      PCM1VPDStructure:0x05
                      PCM1VPDCRC:0x41BEF99C
                      PCM2Firmware:1.00|1.05
                      PCM2VPDStructure:0x05
                      PCM2VPDCRC:0x41BEF99C

EbodController      :
DisksFirmware       : SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08

TalladegaController : ActiveBIOS:0.45.0010
                      BackupBIOS:0.45.0006
                      MainCPLD:17.0.000b
                      ActiveBMCRoot:2.0.001F
                      BackupBMCRoot:2.0.001F
                      BMCBoot:2.0.0002
                      LsiFirmware:20.00.04.00
                      LsiBios:07.37.00.00
                      Battery1Firmware:06.2C
                      Battery2Firmware:06.2C
                      DomFirmware:X231600
                      CanisterFirmware:3.5.0.56
                      CanisterBootloader:5.03
                      CanisterConfigCRC:0x9134777A
                      CanisterVPDStructure:0x06
                      CanisterGEMCPLD:0x19
                      CanisterVPDCRC:0x142F7DC2
                      MidplaneVPDStructure:0x0C
                      MidplaneVPDCRC:0xA6BD4F64
                      MidplaneCPLD:0x10
                      PCM1Firmware:1.00|1.05
                      PCM1VPDStructure:0x05
                      PCM1VPDCRC:0x41BEF99C
                      PCM2Firmware:1.00|1.05
                      PCM2VPDStructure:0x05
                      PCM2VPDCRC:0x41BEF99C

EbodController      :
DisksFirmware       : SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      SmrtStor:TXA2D20400GA6XYR:KZ50
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08
                      WD:WD4001FYYG-01SL3:VR08

--------------------------------------------------
```

### <a name="system-test"></a>Системная проверка

В ходе этой проверки предоставляются сведения о системе, доступных обновлениях, о кластере и служебные данные об устройстве.

* Сведения о системе включают модель, серийный номер устройства, часовой пояс, состояние контроллера и точную версию программного обеспечения, работающего в системе. Объяснение различных параметров системы, выводимых в качестве выходных данных, представлено в разделе [Интерпретация системных сведений](#appendix-interpreting-system-information).

* Из сведений о доступности обновлений можно узнать, доступен ли режим стандартного обновления и обновления для обслуживания, а также имена связанных пакетов. Если параметры `RegularUpdates` и `MaintenanceModeUpdates` имеют значение `false`, это означает, что обновления недоступны. Устройство находится в актуальном состоянии.
* Сведения о кластере содержат данные о различных логических компонентах всех групп кластеров HCS и их соответствующих состояниях. Если в этой части отчета указана группа кластеров в автономном состоянии, [обратитесь в службу поддержки Майкрософт](storsimple-8000-contact-microsoft-support.md).
* Сведения о службе включают имена и состояния всех служб HCS и CIS, работающих на устройстве. Они полезны для службы поддержки Майкрософт при устранении неполадок устройства.

#### <a name="sample-output-of-system-test-run-on-an-8100-device"></a>Пример выходных данных системной проверки устройства серии 8100

Ниже представлен пример выходных данных системной проверки устройства серии 8100.

```
Controller0>Invoke-HcsDiagnostics -Scope System
Running system diagnostics ...
--------------------------------------------------

System information
----------------------
Controller0:

InstanceId              : 7382407f-a56b-4622-8f3f-756fe04cfd38
Name                    : 8100-SHX0991003G467K
Model                   : 8100
SerialNumber            : SHX0991003G467K
TimeZone                : (UTC-08:00) Pacific Time (US & Canada)
CurrentController       : Controller0
ActiveController        : Controller0
Controller0Status       : Normal
Controller1Status       : Normal
SystemMode              : Normal
FriendlySoftwareVersion : StorSimple 8000 Series Update 4.0
HcsSoftwareVersion      : 6.3.9600.17820
ApiVersion              : 9.0.0.0
VhdVersion              : 6.3.9600.17759
OSVersion               : 6.3.9600.0
CisAgentVersion         : 1.0.9441.0
MdsAgentVersion         : 35.2.2.0
Lsisas2Version          : 2.0.78.0
Capacity                : 219902325555200
RemoteManagementMode    : Disabled
FipsMode                : Enabled

Controller1:
InstanceId              : 7382407f-a56b-4622-8f3f-756fe04cfd38
Name                    : 8100-SHX0991003G467K
Model                   : 8100
SerialNumber            : SHX0991003G467K
TimeZone                :
CurrentController       : Controller0
ActiveController        : Controller0
Controller0Status       : Normal
Controller1Status       : Normal
SystemMode              : Normal
FriendlySoftwareVersion : StorSimple 8000 Series Update 4.0
HcsSoftwareVersion      : 6.3.9600.17820
ApiVersion              : 9.0.0.0
VhdVersion              : 6.3.9600.17759
OSVersion               : 6.3.9600.0
CisAgentVersion         : 1.0.9441.0
MdsAgentVersion         : 35.2.2.0
Lsisas2Version          : 2.0.78.0
Capacity                : 219902325555200
RemoteManagementMode    : HttpsAndHttpEnabled
FipsMode                : Enabled

Update availability
----------------------
RegularUpdates              : False
MaintenanceModeUpdates      : False
RegularUpdatesTitle         : {}
MaintenanceModeUpdatesTitle : {}

Cluster information
----------------------
Name                          State OwnerGroup
----                          ----- ----------
ApplicationHostRLUA           Online HCS Cluster Group
Data0v4                       Online HCS Cluster Group
HCS Vnic Resource             Online HCS Cluster Group
hcs_cloud_connectivity_...    Online HCS Cluster Group
hcs_controller_replacement    Online HCS Cluster Group
hcs_datapath_service          Online HCS Cluster Group
hcs_management_service        Online HCS Cluster Group
hcs_nvram_service             Online HCS Cluster Group
hcs_passive_datapath          Online HCS Passive Cluster Group
hcs_platform_service          Online HCS Cluster Group
hcs_saas_agent_service        Online HCS Cluster Group
HddDataClusterDisk            Online HCS Cluster Group
HddMgmtClusterDisk            Online HCS Cluster Group
HddReplClusterDisk            Online HCS Cluster Group
iSCSI Target Server           Online HCS Cluster Group
NvramClusterDisk              Online HCS Cluster Group
SSAdminRLUA                   Online HCS Cluster Group
SsdDataClusterDisk            Online HCS Cluster Group
SsdNvramClusterDisk           Online HCS Cluster Group

Service information
----------------------
Name                                          Status DisplayName
----                                          ------ -----------
CiSAgentSvc                                   Stopped CiS Service Agent
hcs_cloud_connectivity_...                    Running hcs_cloud_connectivity...
hcs_controller_replacement                    Running HCS Controller Replace...
hcs_datapath_service                          Running HCS Datapath Service
hcs_management_service                        Running HCS Management Service
hcs_minishell                                 Running hcs_minishell
HCS_NVRAM_Service                             Running HCS NVRAM Service
hcs_passive_datapath                          Stopped HCS Passive Datapath S...
hcs_platform_service                          Running HCS Platform Monitor S...
hcs_saas_agent_service                        Running hcs_saas_agent_service
hcs_startup                                   Stopped hcs_startup
--------------------------------------------------
```

### <a name="network-test"></a>Проверка сети

Этот тест проверяет состояние сетевых интерфейсов, портов, подключений DNS и NTP-сервера, сертификата TLS/SSL, учетных данных учетной записи хранения, подключения к серверам обновлений и подключения веб-прокси на устройстве StorSimple.

#### <a name="sample-output-of-network-test-when-only-data0-is-enabled"></a>Пример выходных данных проверки сети при включении только DATA0

Ниже приведен пример выходных данных устройства серии 8100, по которому можно определить следующее:
* Включены и настроены только сетевые интерфейсы DATA0 и DATA1.
* Сетевые интерфейсы DATA2–5 не включены на портале.
* Конфигурация DNS-сервера допустимая, и устройство может подключаться через DNS-сервер.
* Вы также можете подключиться через NTP-сервер.
* Открыты порты 80 и 443. Тем не менее порт 9354 заблокирован. В соответствии с [требованиями системы к сети](./storsimple-8000-system-requirements.md) необходимо открыть этот порт для взаимодействия служебной шины.
* Сертификация TLS/SSL действительна.
* Устройство может подключиться к учетной записи хранения _myss8000storageacct_.
* Вы можете подключиться к серверам обновления.
* Веб-прокси не настроен на этом устройстве.

#### <a name="sample-output-of-network-test-when-data0-and-data1-are-enabled"></a>Пример выходных данных проверки сети при включении DATA0 и DATA1

```
Controller0>Invoke-HcsDiagnostics -Scope Network
Running network diagnostics ....
--------------------------------------------------
Validating networks .....
Name                Entity              Result              Details
----                ------              ------              -------
Network interface   Data0               Valid
Network interface   Data1               Valid
Network interface   Data2               Not enabled
Network interface   Data3               Not enabled
Network interface   Data4               Not enabled
Network interface   Data5               Not enabled
DNS                 10.222.118.154      Valid
NTP                 time.windows.com    Valid
Port                80                  Open
Port                443                 Open
Port                9354                Blocked
SSL certificate     https://myss8000... Valid
Storage account ... myss8000storageacct Valid
URL                 http://download.... Valid
URL                 http://download.... Valid
Web proxy                               Not enabled         Web proxy is not...
--------------------------------------------------
```

### <a name="performance-test"></a>тест производительности;

В ходе этой проверки определяется производительность облака по значениям задержки операций чтения и записи в облаке для устройства. С помощью этого средства можно задать базовые показатели производительности облака, которых можно достичь на устройстве StorSimple. Средство указывает максимальную производительность (самые низкие показатели задержки операций чтения и записи), которую можно получить для подключения.

Так как средство сообщает о максимально возможной достижимой производительности, полученные значения задержки операций чтения и записи можно использовать в качестве целевых при развертывании рабочей нагрузки.

В ходе теста моделируются размеры больших двоичных объектов, связанные с разными типами томов на устройстве. Обычные многоуровневые тома и их резервные копии, закрепленные локально, используют большие двоичные объекты на 64 КБ. Многоуровневые тома с архивацией используют большие двоичные объекты объемом 512 КБ. Если на устройстве настроены многоуровневые и локально закрепленные тома, выполняется проверка для больших двоичных объектов на 64 КБ.

Чтобы воспользоваться средством, сделайте следующее:

1.  Сначала создайте сочетание многоуровневых томов и многоуровневых томов с архивацией. Таким образом, средство выполнит тесты для больших двоичных объектов на 64 КБ и 512 КБ.

2. Создав и настроив тома, выполните командлет. Тип:

    `Invoke-HcsDiagnostics -Scope Performance`

3. Запишите значения задержек операций чтения и записи, указанных средством. Результаты проверки будут получены через несколько минут.

4. Если значения задержки подключения, сообщаемые средством, находятся в пределах ожидаемого диапазона значений, их можно использовать в качестве максимально достижимых целевых значений при развертывании рабочей нагрузки. Учитывайте некоторый запас на внутреннюю обработку данных.

    Если средство диагностики указало высокую задержку операций чтения и записи, сделайте следующее:

    1. Настройте аналитику хранилища для служб BLOB-объектов и проанализируйте выходные данные, чтобы определить задержки для учетной записи хранения Azure. Подробные инструкции см. в статье [Включение метрик хранилища и просмотр данных метрик](../storage/blobs/monitor-blob-storage.md). Если эти значения задержки так же высоки, как и значения, полученные от средства диагностики StorSimple, необходимо отправить запрос на поддержку в службе хранилища Azure.

    2. Если значения задержки в учетной записи хранения низкие, обратитесь к администратору сети, чтобы изучить любые проблемы с задержкой в сети.

#### <a name="sample-output-of-performance-test-run-on-an-8100-device"></a>Пример выходных данных проверки производительности устройства серии 8100

```
Controller0>Invoke-HcsDiagnostics -Scope Performance
Running performance diagnostics...
--------------------------------------------------
Cloud performance: writing blobs
Cloud write latency: 194 ms using credential 'myss8000storageacct', blob size '64KB'
Cloud performance: reading blobs..
Cloud read latency: 544 ms using credential 'myss8000storageacct', blob size '64KB'
Cloud performance: writing blobs.
Cloud write latency: 369 ms using credential 'myss8000storageacct', blob size '512KB'
Cloud performance: reading blobs...
Cloud read latency: 4924 ms using credential 'myss8000storageacct', blob size '512KB'
--------------------------------------------------
Controller0>
```

## <a name="appendix-interpreting-system-information"></a>Приложение: интерпретация системных сведений

Ниже представлена таблица со сведениями о сопоставлении различных параметров Windows PowerShell в сведениях о системе. 

| Параметр PowerShell    | Описание  |
|-------------------------|------------------|
| Instance ID             | С каждым контроллером связан уникальный идентификатор или GUID.|
| Имя                    | Понятное имя устройства, настроенное на портале Azure во время развертывания устройства. В качестве понятного имени по умолчанию используется серийный номер устройства. |
| Модель                   | Модель устройства StorSimple серии 8000 (8100 или 8600).|
| SerialNumber            | Серийный номер устройства назначается на заводе и состоит из 15 знаков. Например, 8600-SHX0991003G44HT:<br> 8600 — это модель устройства;<br>SHX — это регион производства;<br> 0991003 — определенный продукт; <br> G44HT — последние 5 цифр увеличиваются для получения уникального серийного номера. Набор может быть неупорядоченным.|
| TimeZone                | Часовой пояс устройства, настроенный на портале Azure во время развертывания устройства.|
| CurrentController       | Контроллер, к которому подключена система через интерфейс Windows PowerShell устройства StorSimple.|
| ActiveController        | Активный контроллер на устройстве, управляющий всеми сетевыми и дисковыми операциями. Это может быть контроллер 0 или 1.  |
| Controller0Status       | Состояние контроллера 0 на устройстве ("Обычное", "В режиме восстановления" или "Недоступно").|
| Controller1Status       | Состояние контроллера 1 на устройстве  ("Обычное", "В режиме восстановления" или "Недоступно").|
| SystemMode              | Общее состояние устройства StorSimple ("Обычное", "Обслуживание" или "Списан") (соответствует отключению на портале Azure).|
| FriendlySoftwareVersion | Понятная строка, соответствующая версии программного обеспечения устройства. Для системы с обновлением 4 будет использоваться понятная версия программного обеспечения StorSimple серии 8000 с обновлением 4.0.|
| HcsSoftwareVersion      | Версия программного обеспечения HCS, работающего на устройстве. Например, устройству StorSimple серии 8000 с обновлением 4.0 соответствует версия программного обеспечения HCS 6.3.9600.17820. |
| ApiVersion              | Версия программного обеспечения API Windows PowerShell устройства HCS.|
| VhdVersion              | Версия программного обеспечения заводского образа, поставляемого с устройством. При сбросе устройства к заводским настройкам оно запустится с этой версией программного обеспечения.|
| OSVersion               | Версия программного обеспечения операционной системы Windows Server, запущенной на устройстве. Устройство StorSimple работает на базе Windows Server 2012 R2, что соответствует версии 6.3.9600.|
| CisAgentVersion         | Версия агента CIS, запущенного на устройстве StorSimple. Этот агент позволяет взаимодействовать со службой StorSimple Manager, запущенной в Azure.|
| MdsAgentVersion         | Версия агента MDS, запущенного на устройстве StorSimple. Этот агент перемещает данные в службу MDS.|
| Lsisas2Version          | Версия драйверов LSI на устройстве StorSimple.|
| Capacity                | Общая емкость устройства в байтах.|
| RemoteManagementMode    | Указывает на то, можно ли управлять устройством удаленно через интерфейс Windows PowerShell. |
| FipsMode                | Указывает, включен ли на устройстве режим FIPS (федеральный стандарт обработки информации США). Стандарт FIPS 140 определяет криптографические алгоритмы, утвержденные для использования в компьютерных системах федеральной власти США для защиты конфиденциальных данных. Для устройств с обновлением версии 4 или более поздней режим FIPS включен по умолчанию. |

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте о [синтаксисе командлета Invoke-HcsDiagnostics](/previous-versions/windows/powershell-scripting/mt795371(v=wps.630)).

* Дополнительные сведения об [устранении неполадок развертывания](./storsimple-8000-troubleshoot-deployment.md) на устройстве StorSimple.
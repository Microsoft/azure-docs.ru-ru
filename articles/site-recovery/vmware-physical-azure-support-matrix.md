---
title: Матрица поддержки для VMware или физического аварийного восстановления в Azure Site Recovery
description: Содержит сводку по поддержке аварийного восстановления виртуальных машин VMware и физического сервера в Azure с помощью Azure Site Recovery.
ms.topic: conceptual
ms.date: 07/14/2020
ms.openlocfilehash: e025b1681306a3bb837a338655dc561e897226c2
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102449260"
---
# <a name="support-matrix-for-disaster-recovery--of-vmware-vms-and-physical-servers-to-azure"></a>Таблица поддержки аварийного восстановления виртуальных машин VMware и физических серверов в Azure

В этой статье перечислены поддерживаемые компоненты и параметры для аварийного восстановления виртуальных машин VMware и физических серверов в Azure с помощью [Azure Site Recovery](site-recovery-overview.md).

- [Узнайте больше](vmware-azure-architecture.md) об архитектуре аварийного восстановления виртуальных машин VMware или физических серверов.
- Следуйте нашим [руководствам](tutorial-prepare-azure.md) , чтобы испытать аварийное восстановление.

> [!NOTE]
> Site Recovery не будет перемещать или хранить данные клиента из целевого региона, в котором было настроено аварийное восстановление для исходных компьютеров. Клиенты могут выбрать хранилище служб восстановления из другого региона, если это необходимо. Хранилище служб восстановления содержит метаданные, но не имеет фактических данных клиента.

## <a name="deployment-scenarios"></a>Сценарии развертывания

**Сценарий** | **Сведения**
--- | ---
Аварийное восстановление виртуальных машин VMware | Репликация локальных виртуальных машин VMware в Azure. Этот сценарий можно развернуть в портал Azure или с помощью [PowerShell](vmware-azure-disaster-recovery-powershell.md).
Аварийное восстановление физических серверов | Репликация локальных физических серверов Windows или Linux в Azure. Этот сценарий можно развернуть с помощью портала Azure.

## <a name="on-premises-virtualization-servers"></a>Локальные серверы виртуализации

**Server** | **Requirements** | **Сведения**
--- | --- | ---
с сервером vCenter | Версия 7,0 & последующие обновления в этой версии, 6,7, 6,5, 6,0 или 5,5 | Рекомендуется использовать vCenter Server в развертывании аварийного восстановления.
Узлы vSphere | Версия 7,0 & последующие обновления в этой версии, 6,7, 6,5, 6,0 или 5,5 | Мы рекомендуем размещать узлы vSphere и серверы vCenter в той же сети, в которой находится сервер обработки. По умолчанию сервер обработки запускается на сервере конфигурации. [Подробнее.](vmware-physical-azure-config-process-server-overview.md)

## <a name="site-recovery-configuration-server"></a>Сервер конфигурации Site Recovery

Сервер конфигурации размещается на локальном компьютере, где выполняются компоненты Site Recovery, включая сервер конфигурации, сервер обработки и главный целевой сервер.

- Для виртуальных машин VMware необходимо настроить сервер конфигурации, загрузив шаблон OVF, чтобы создать виртуальную машину VMware.
- Для физических серверов Настройка компьютера сервера конфигурации выполняется вручную.

**Компонент** | **Requirements**
--- |---
Ядра ЦП | 8
ОЗУ | 16 Гб
Количество дисков | 3 диска<br/><br/> В состав дисков входят: диск ОС, диск кэша сервера обработки, диск хранения (для восстановления размещения).
Свободное место на диске | 600 ГБ пространства для кэша сервера обработки.
Свободное место на диске | 600 ГБ пространства для диска хранения.
Операционная система  | Windows Server 2012 R2 или Windows Server 2016 с возможностями рабочего стола <br/><br> Если вы планируете использовать встроенный главный целевой сервер для восстановления размещения, убедитесь, что версия ОС совпадает с версией реплицированных элементов или выше нее.|
Язык операционной системы | Английский (en-us)
[PowerCLI](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1) | Не требуется для сервера конфигурации [9,14](https://support.microsoft.com/help/4091311/update-rollup-23-for-azure-site-recovery) или более поздней версии.
Роли Windows Server | Не включайте службы домен Active Directory; Службы IIS (IIS) или Hyper-V.
Групповые политики| — запрет на использование командной строки; <br/> — запрет на использование инструментов редактирования реестра; <br/> — логика доверия для вложенных файлов; <br/> — включение выполнения скриптов. <br/> - [Подробнее](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))|
IIS | Не забудьте выполнить следующие действия.<br/><br/> -У вас нет предварительно существующего веб-сайта по умолчанию <br/> — включите [анонимную аутентификацию](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)); <br/> — включите параметр [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10));  <br/> — убедитесь, что отсутствует предварительно созданный веб-сайт или приложение, ожидающее передачи данных через порт 443.<br/>
Тип сетевой карты | VMXNET3 (при развертывании в качестве виртуальной машины VMware)
Тип IP-адреса | Статические
Порты | 443 используется для оркестрации канала управления<br/>9443 для транспорта данных

> [!NOTE]
Операционная система должна быть установлена с английским языком. Преобразование языкового стандарта после установки может привести к потенциальным проблемам.

## <a name="replicated-machines"></a>Реплицируемые компьютеры

Site Recovery поддерживает репликацию любой рабочей нагрузки, выполняемой на поддерживаемом компьютере.

**Компонент** | **Сведения**
--- | ---
Параметры компьютера | Компьютеры, которые реплицируются в Azure, должны соответствовать [требованиям Azure](#azure-vm-requirements).
Рабочая нагрузка компьютера | Site Recovery поддерживает репликацию любой рабочей нагрузки, выполняемой на поддерживаемом компьютере. [Подробнее.](./site-recovery-workload.md)
Имя компьютера | Убедитесь, что отображаемое имя компьютера не попадает в [зарезервированные имена ресурсов Azure](../azure-resource-manager/templates/error-reserved-resource-name.md) .<br/><br/> В именах логических томов регистр не учитывается. Убедитесь, что имена двух томов на устройстве не совпадают. Пример: тома с именами "voLUME1", "voLUME1" не могут быть защищены с помощью Azure Site Recovery.

### <a name="for-windows"></a>Для Windows

**Операционная система** | **Сведения**
--- | ---
Windows Server 2019 | Поддерживается из [накопительного пакета обновления 34](https://support.microsoft.com/help/4490016) (версия 9,22 службы Mobility Service), начиная с версии.
Windows Server 2016 64-bit | Поддерживается для основных серверных компонентов и сервера с возможностями рабочего стола.
Windows Server 2012 R2 или Windows Server 2012 | Поддерживается.
Windows Server 2008 R2 с пакетом обновления 1 (SP1) — назад. | Поддерживается.<br/><br/> В версии [9,30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) агента службы Mobility Service требуется [Обновление стека обслуживания (SSU)](https://support.microsoft.com/help/4490628) и [Обновление SHA-2](https://support.microsoft.com/help/4474419) , установленные на компьютерах под управлением Windows 2008 R2 с пакетом обновления 1 (SP1) или более поздней версии. SHA-1 не поддерживается с сентября 2019 г., а если подпись кода SHA-2 не включена, то расширение агента не будет устанавливаться и обновляться должным образом. Подробнее об [обновлении SHA-2 и требованиях](https://support.microsoft.com/topic/sha-2-code-signing-support-update-for-windows-server-2008-r2-windows-7-and-windows-server-2008-september-23-2019-84a8aad5-d8d9-2d5c-6d78-34f9aa5f8339).
Windows Server 2008 с пакетом обновления 2 (SP2) или более поздней версии (64-или 32-разрядная версия) |  Поддерживается только для миграции. [Подробнее.](migrate-tutorial-windows-server-2008.md)<br/><br/> В версии [9,30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) агента службы Mobility Service требуется [Обновление стека обслуживания (SSU)](https://support.microsoft.com/help/4493730) и [Обновление SHA-2](https://support.microsoft.com/help/4474419) , установленные на компьютерах под управлением Windows 2008 с пакетом обновления 2 (SP2). SHA-1 не поддерживается с сентября 2019 г., а если подпись кода SHA-2 не включена, то расширение агента не будет устанавливаться и обновляться должным образом. Подробнее об [обновлении SHA-2 и требованиях](https://support.microsoft.com/topic/sha-2-code-signing-support-update-for-windows-server-2008-r2-windows-7-and-windows-server-2008-september-23-2019-84a8aad5-d8d9-2d5c-6d78-34f9aa5f8339).
Windows 10, Windows 8.1, Windows 8 | Поддерживается только 64-разрядная система. 32-разрядная система не поддерживается.
Windows 7 с пакетом обновления 1 (SP1) 64-bit | Поддерживается из [накопительного пакета обновления 36](https://support.microsoft.com/help/4503156) (версия 9,22 службы Mobility Service), начиная с версии. </br></br> С [9,30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) агента службы Mobility Service требуется [Обновление стека обслуживания (SSU)](https://support.microsoft.com/help/4490628) и [Обновление SHA-2](https://support.microsoft.com/help/4474419) на компьютерах под управлением Windows 7 с пакетом обновления 1 (SP1).  SHA-1 не поддерживается с сентября 2019 г., а если подпись кода SHA-2 не включена, то расширение агента не будет устанавливаться и обновляться должным образом. Подробнее об [обновлении SHA-2 и требованиях](https://support.microsoft.com/topic/sha-2-code-signing-support-update-for-windows-server-2008-r2-windows-7-and-windows-server-2008-september-23-2019-84a8aad5-d8d9-2d5c-6d78-34f9aa5f8339).

### <a name="for-linux"></a>Для Linux

**Операционная система** | **Сведения**
--- | ---
Linux | Поддерживается только 64-разрядная система. 32-разрядная система не поддерживается.<br/><br/>На каждом сервере Linux должны быть установлены [компоненты Linux Integration Services (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) . После тестовой отработки отказа или отработки отказа необходимо загрузить сервер в Azure. Если в системе отсутствуют встроенные компоненты LIS, установите [их перед](https://www.microsoft.com/download/details.aspx?id=55106) включением репликации для загрузки виртуальных машин в Azure. <br/><br/> Site Recovery координирует отработку отказа для запуска серверов Linux в Azure. Но поставщики Linux могут поддерживать только те версии дистрибутивов, срок поддержки которых еще не окончился.<br/><br/> Поддерживаются только дистрибутивы Linux на основе номенклатурных ядер, которые являются частью выпуска или обновления дополнительной версии.<br/><br/> Обновление защищенных виртуальных машин с основной версией дистрибутивов Linux не поддерживается. Для обновления отключите репликацию, обновите операционную систему и включите репликацию снова.<br/><br/> Дополнительные [сведения](https://support.microsoft.com/help/2941892/support-for-linux-and-open-source-technology-in-azure) о поддержке Linux и технологии с открытым кодом в Azure.
Linux Red Hat Enterprise | от 5,2 до 5,11</b><br/> от 6,1 до 6,10</b> </br> 7,0, 7,1, 7,2, 7,3, 7,4, 7,5, 7,6, [7,7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7,8](https://support.microsoft.com/help/4564347/), [бета-версия 7,9](https://support.microsoft.com/help/4578241/), [7,9](https://support.microsoft.com/help/4590304/) </br> [8,0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8,1, [8,2](https://support.microsoft.com/help/4570609), [8,3](https://support.microsoft.com/help/4597409/) <br/> Несколько старых ядер на серверах, на которых работает Red Hat Enterprise Linux 5.2 — 5.11 & 6.1-6.10, не имеют предварительно установленных [компонентов Linux Integration Services (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) . Если в системе отсутствуют встроенные компоненты LIS, установите [их перед](https://www.microsoft.com/download/details.aspx?id=55106) включением репликации для загрузки виртуальных машин в Azure.
Linux: CentOS | от 5,2 до 5,11</b><br/> от 6,1 до 6,10</b><br/> </br> 7,0, 7,1, 7,2, 7,3, 7,4, 7,5, 7,6, [7,7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [7,8](https://support.microsoft.com/help/4564347/), [7,9](https://support.microsoft.com/help/4578241/) </br> [8,0](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), 8,1, [8,2](https://support.microsoft.com/help/4570609), [8,3](https://support.microsoft.com/help/4597409/) <br/><br/> Несколько старых ядер на серверах с CentOS 5.2 — 5.11 & 6.1-6.10 не имеют предварительно установленных  [компонентов Integration Services Linux (LIS)](https://www.microsoft.com/download/details.aspx?id=55106) . Если в системе отсутствуют встроенные компоненты LIS, установите [их перед](https://www.microsoft.com/download/details.aspx?id=55106) включением репликации для загрузки виртуальных машин в Azure.
Ubuntu | Ubuntu 14,04 * LTS Server [(Обзор поддерживаемых версий ядра)](#ubuntu-kernel-versions)<br/>Ubuntu 16,04 * LTS Server [(Обзор поддерживаемых версий ядра)](#ubuntu-kernel-versions) </br> Ubuntu 18,04 * LTS Server [(Обзор поддерживаемых версий ядра)](#ubuntu-kernel-versions) </br> Ubuntu 20,04 * LTS Server [(Обзор поддерживаемых версий ядра)](#ubuntu-kernel-versions) </br> (*включает поддержку для всех 14,04.* x *, 16,04.* x *, 18,04.* x *, 20,04.* версии x *)
Debian | Debian 7/Debian 8 (включает поддержку для всех 7. *x*, 8. *x* версии); Debian 9 (включает поддержку 9,1 – 9,13. Debian 9,0 не поддерживается.), Debian 10 [(ознакомьтесь с поддерживаемыми версиями ядра)](#debian-kernel-versions)
SUSE Linux | SUSE Linux Enterprise Server 12 SP1, SP2, SP3, SP4, [SP5](https://support.microsoft.com/help/4570609) [(Проверка поддерживаемых версий ядра)](#suse-linux-enterprise-server-12-supported-kernel-versions) <br/> SUSE Linux Enterprise Server 15, 15 с пакетом обновления 1 [(SP1) (Обзор поддерживаемых версий ядра)](#suse-linux-enterprise-server-15-supported-kernel-versions) <br/> SUSE Linux Enterprise Server 11 SP3. [Обязательно Скачайте последнюю версию установщика агента Mobility Service на сервере конфигурации](vmware-physical-mobility-service-overview.md#download-latest-mobility-agent-installer-for-suse-11-sp3-rhel-5-debian-7-server). </br> SUSE Linux Enterprise Server 11 SP4 </br> **Примечание**. обновление реплицированных компьютеров с SuSE Linux Enterprise Server 11 SP3 до SP4 не поддерживается. Чтобы выполнить обновление, отключите репликацию и снова включите ее после обновления. <br/>|
Oracle Linux | 6,4, 6,5, 6,6, 6,7, 6,8, 6,9, 6,10, 7,0, 7,1, 7,2, 7,3, 7,4, 7,5, 7,6, [7,7](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery), [7,8](https://support.microsoft.com/help/4573888/), [7,9](https://support.microsoft.com/help/4597409/), [8,0](https://support.microsoft.com/help/4573888/), [8,1](https://support.microsoft.com/help/4573888/), [8,3](https://support.microsoft.com/help/4597409/)  <br/> С ядром, совместимым с Red Hat, или с ядром Unbreakable Enterprise Kernel Release 3, 4 и 5 (UEK3, UEK4, UEK5)<br/><br/>8.1<br/>Для всех UEK ядер и RedHat kernel <= 3.10.0-1062. * поддерживаются в [9,35](https://support.microsoft.com/help/4573888/) . Поддержка остальных ядер RedHat доступна в [9,36](https://support.microsoft.com/help/4578241/) .

> [!Note]
>- Для каждой версии Windows Azure Site Recovery поддерживает только [долгосрочные сборки канала обслуживания (LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) .  В настоящее время не поддерживается [полугодовые](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel) выпуски канала.
>- Убедитесь, что для версий Linux Azure Site Recovery не поддерживает настраиваемые образы ОС. Поддерживаются только те ядра, которые являются частью выпуска или обновления дополнительных версий распространения.

### <a name="ubuntu-kernel-versions"></a>Версии ядра Ubuntu

**Поддерживаемый выпуск** | **Версия службы Mobility Service** | **Версия ядра** |
--- | --- | --- |
14.04 LTS | [9,37](https://support.microsoft.com/help/4582666/), [9,38](https://support.microsoft.com/help/4590304/), [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a), [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | с 3.13.0-24-generic по 3.13.0-170-generic,<br/>3.16.0-25-generic to 3.16.0-77-generic,<br/>3.19.0-18-generic to 3.19.0-80-generic,<br/>4.2.0-18-generic to 4.2.0-42-generic,<br/>с 4.4.0-21-generic по 4.4.0-148-generic,<br/>с 4.15.0-1023-azure по 4.15.0-1045-azure |
|||
16.04 LTS | [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | 4.4.0-21-Generic для 4.4.0-201-generic,<br/>с 4.8.0-34-generic по 4.8.0-58-generic<br/>с 4.10.0-14-generic по 4.10.0-42-generic,<br/>с 4.11.0-13-generic по 4.11.0-14-generic,<br/>с 4.13.0-16-generic по 4.13.0-45-generic,<br/>4.15.0-13-Generic — 4.15.0-133-Generic<br/>с 4.11.0-1009-azure по 4.11.0-1016-azure,<br/>с 4.13.0-1005-azure по 4.13.0-1018-azure. <br/>4.15.0-1012 — Azure — 4.15.0 — 1106 — Azure|
16.04 LTS | [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a) | 4.4.0-21-Generic для 4.4.0-197-generic,<br/>с 4.8.0-34-generic по 4.8.0-58-generic<br/>с 4.10.0-14-generic по 4.10.0-42-generic,<br/>с 4.11.0-13-generic по 4.11.0-14-generic,<br/>с 4.13.0-16-generic по 4.13.0-45-generic,<br/>4.15.0-13-Generic для 4.15.0-128-Generic<br/>с 4.11.0-1009-azure по 4.11.0-1016-azure,<br/>с 4.13.0-1005-azure по 4.13.0-1018-azure. <br/>4.15.0-1012 — Azure — 4.15.0 — 1102 — Azure |
16.04 LTS | [9,39](https://support.microsoft.com/help/4597409/) | 4.4.0-21-Generic для 4.4.0-194-generic,<br/>с 4.8.0-34-generic по 4.8.0-58-generic<br/>с 4.10.0-14-generic по 4.10.0-42-generic,<br/>с 4.11.0-13-generic по 4.11.0-14-generic,<br/>с 4.13.0-16-generic по 4.13.0-45-generic,<br/>4.15.0-13-Generic для 4.15.0-123-Generic<br/>с 4.11.0-1009-azure по 4.11.0-1016-azure,<br/>с 4.13.0-1005-azure по 4.13.0-1018-azure. <br/>4.15.0-1012 — Azure — 4.15.0 — 1098 — Azure|
16.04 LTS | [9,38](https://support.microsoft.com/help/4590304/) | 4.4.0-21-Generic — 4.4.0-190-generic,<br/>с 4.8.0-34-generic по 4.8.0-58-generic<br/>с 4.10.0-14-generic по 4.10.0-42-generic,<br/>с 4.11.0-13-generic по 4.11.0-14-generic,<br/>с 4.13.0-16-generic по 4.13.0-45-generic,<br/>4.15.0-13-Generic — 4.15.0-118-Generic<br/>с 4.11.0-1009-azure по 4.11.0-1016-azure,<br/>с 4.13.0-1005-azure по 4.13.0-1018-azure. <br/>4.15.0-1012 — Azure — 4.15.0 — 1096 — Azure|
16.04 LTS | [9,37](https://support.microsoft.com/help/4582666/) | 4.4.0-21-Generic для 4.4.0-189-generic,<br/>с 4.8.0-34-generic по 4.8.0-58-generic<br/>с 4.10.0-14-generic по 4.10.0-42-generic,<br/>с 4.11.0-13-generic по 4.11.0-14-generic,<br/>с 4.13.0-16-generic по 4.13.0-45-generic,<br/>4.15.0-13-Generic для 4.15.0-115-Generic<br/>с 4.11.0-1009-azure по 4.11.0-1016-azure,<br/>с 4.13.0-1005-azure по 4.13.0-1018-azure. <br/>4.15.0-1012 — Azure — 4.15.0 — 1093 — Azure |
|||
18.04 LTS | [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | 4.15.0-20-Generic для 4.15.0-135-Generic </br> с 4.18.0-13-generic по 4.18.0-25-generic </br> 5.0.0-15-Generic до 5.0.0-65-Generic </br> 5.3.0-19-Generic для 5.3.0-70-Generic </br> 5.4.0-37-Generic для 5.4.0-59-Generic</br> 5.4.0-60-Generic для 5.4.0-65-Generic </br> 4.15.0-1009-Azure в 4.15.0-1106-Azure </br> с 4.18.0-1006-azure по 4.18.0-1025-azure </br> 5.0.0-1012 — Azure — 5.0.0-1036 — Azure </br> 5.3.0-1007-Azure в 5.3.0-1035-Azure </br> 5.4.0-1020 — Azure — 5.4.0 — 1039 — Azure|
18.04 LTS | [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a) | 4.15.0-20-Generic для 4.15.0-129-Generic </br> с 4.18.0-13-generic по 4.18.0-25-generic </br> 5.0.0-15-Generic до 5.0.0-63-Generic </br> 5.3.0-19-Generic для 5.3.0-69-Generic </br> 5.4.0-37-Generic для 5.4.0-59-Generic</br> 4.15.0-1009-Azure в 4.15.0-1103-Azure </br> с 4.18.0-1006-azure по 4.18.0-1025-azure </br> 5.0.0-1012 — Azure — 5.0.0-1036 — Azure </br> 5.3.0-1007-Azure в 5.3.0-1035-Azure </br> 5.4.0-1020 — Azure — 5.4.0-1035-Azure|
18.04 LTS | [9,39](https://support.microsoft.com/help/4597409/) | 4.15.0-20-Generic для 4.15.0-123-Generic </br> с 4.18.0-13-generic по 4.18.0-25-generic </br> 5.0.0-15-Generic до 5.0.0-63-Generic </br> 5.3.0-19-Generic для 5.3.0-69-Generic </br> 5.4.0-37-Generic для 5.4.0-53-Generic</br> 4.15.0-1009-Azure до 4.15.0-1099 — Azure </br> с 4.18.0-1006-azure по 4.18.0-1025-azure </br> 5.0.0-1012 — Azure — 5.0.0-1036 — Azure </br> 5.3.0-1007-Azure в 5.3.0-1035-Azure </br> 5.4.0-1020 — Azure — 5.4.0-1031 — Azure|
18.04 LTS | [9,38](https://support.microsoft.com/help/4590304/) | 4.15.0-20-Generic — 4.15.0-118-Generic </br> с 4.18.0-13-generic по 4.18.0-25-generic </br> 5.0.0-15-Generic до 5.0.0-61-Generic </br> 5.3.0-19-Generic для 5.3.0-67-Generic </br> 5.4.0-37-Generic для 5.4.0-48-Generic</br> 4.15.0-1009-Azure в 4.15.0-1096-Azure </br> с 4.18.0-1006-azure по 4.18.0-1025-azure </br> 5.0.0-1012 — Azure — 5.0.0-1036 — Azure </br> 5.3.0-1007-Azure в 5.3.0-1035-Azure </br> 5.4.0-1020 — Azure — 5.4.0-1026 — Azure|
18.04 LTS | [9,37](https://support.microsoft.com/help/4582666/) | 4.15.0-20-Generic для 4.15.0-115-Generic </br> с 4.18.0-13-generic по 4.18.0-25-generic </br> 5.0.0-15-Generic – 5.0.0-60-Generic </br> 5.3.0-19-Generic для 5.3.0-66-Generic </br> 5.4.0-37-Generic для 5.4.0-45-Generic</br> 4.15.0-1009-Azure в 4.15.0-1093-Azure </br> с 4.18.0-1006-azure по 4.18.0-1025-azure </br> 5.0.0-1012 — Azure — 5.0.0-1036 — Azure </br> 5.3.0-1007-Azure в 5.3.0-1035-Azure </br> 5.4.0-1020 — Azure — 5.4.0-1023 — Azure|
|||
20,04 LTS |[9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533)| 5.4.0-26-Generic — 5.4.0-65 </br> -generic 5.4.0-1010-Azure в 5.4.0-1039-Azure </br> 5.8.0-29-Generic в 5.8.0-43-Generic </br>
20,04 LTS |[9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a)| 5.4.0-26-Generic — 5.4.0-59 </br> -generic 5.4.0-1010-Azure в 5.4.0-1035-Azure </br> 5.8.0-29-Generic — 5.8.0-34-Generic|
20,04 LTS |[9,39](https://support.microsoft.com/help/4597409/) | 5.4.0-26-Generic — 5.4.0-53 </br> -generic 5.4.0-1010-Azure в 5.4.0-1031-Azure
20,04 LTS |[9,38](https://support.microsoft.com/help/4590304/) | 5.4.0-26-Generic — 5.4.0-48 </br> -generic 5.4.0-1010-Azure в 5.4.0-1026-Azure
20,04 LTS |[9,37](https://support.microsoft.com/help/4582666/) | 5.4.0-26-Generic — 5.4.0-45 </br> -generic 5.4.0-1010-Azure в 5.4.0-1023-Azure

### <a name="debian-kernel-versions"></a>Версии ядра Debian


**Поддерживаемый выпуск** | **Версия службы Mobility Service** | **Версия ядра** |
--- | --- | --- |
Debian 7 | [9,37](https://support.microsoft.com/help/4582666/), [9,38](https://support.microsoft.com/help/4590304/), [9,39](https://support.microsoft.com/help/4597409/), [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a), [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | С 3.2.0-4-amd64 по 3.2.0-6-amd64, 3.16.0-0.bpo.4-amd64 |
|||
Debian 8 | [9,37](https://support.microsoft.com/help/4582666/), [9,38](https://support.microsoft.com/help/4590304/), [9,39](https://support.microsoft.com/help/4597409/), [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a), [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | 3.16.0-4-AMD64 – 3.16.0-11-AMD64, 4.9.0 -0. BPO. 4-AMD64 до 4.9.0 -0. BPO. 11 — AMD64 |
|||
Debian 9,1 | [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | 4.9.0-1-AMD64 – 4.9.0-14-AMD64 </br> 4.19.0 -0. BPO. 1 — AMD64 – 4.19.0 -0. BPO. 14-AMD64 </br> 4.19.0 -0. BPO. 1 — Cloud-AMD64 — 4.19.0 -0. BPO. 14-Cloud-AMD64
Debian 9,1 | [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a) | 4.9.0-1-AMD64 – 4.9.0-14-AMD64 </br> 4.19.0 -0. BPO. 1 — AMD64 – 4.19.0 -0. BPO. 13-AMD64 </br> 4.19.0 -0. BPO. 1 — Cloud-AMD64 — 4.19.0 -0. BPO. 13 — Cloud-AMD64 </br>
Debian 9,1 | [9,39](https://support.microsoft.com/help/4597409/) | 4.9.0-1-AMD64 – 4.9.0-14-AMD64 </br> 4.19.0 -0. BPO. 1 — AMD64 – 4.19.0 -0. BPO. 12 — AMD64 </br> 4.19.0 -0. BPO. 1 — Cloud-AMD64 — 4.19.0 -0. BPO. 12 — Cloud-AMD64 </br> 
Debian 9,1 | [9,38](https://support.microsoft.com/help/4590304/) | 4.9.0-1-AMD64 – 4.9.0-13-AMD64 </br> 4.19.0 -0. BPO. 1 — AMD64 – 4.19.0 -0. BPO. 11 — AMD64 </br> 4.19.0 -0. BPO. 1 — Cloud-AMD64 — 4.19.0 -0. BPO. 11 — Cloud-AMD64 </br> 
Debian 9,1 | [9,37](https://support.microsoft.com/help/4582666/) | 4.9.0-3-amd64 для 4.9.0-13-AMD64, 4.19.0 -0. BPO. 6-AMD64 до 4.19.0 -0. BPO. 10-AMD64, 4.19.0 -0. BPO. 6-Cloud-AMD64 до 4.19.0 -0. BPO. 10-Cloud-AMD64
|||
Debian 10 | [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | 4.19.0-5-AMD64 – 4.19.0-14-AMD64 </br> 4.19.0-6-Cloud-AMD64 до 4.19.0-14-Cloud-AMD64 </br> 5.8.0 -0. BPO. 2 — AMD64 </br> 5.8.0 -0. BPO. 2 — Cloud-AMD64
Debian 10 | [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a) | 4.19.0-5-AMD64 – 4.19.0-13-AMD64 </br> 4.19.0-6-Cloud-AMD64 до 4.19.0-13-Cloud-AMD64 </br> 5.8.0 -0. BPO. 2 — AMD64 </br> 5.8.0 -0. BPO. 2 — Cloud-AMD64

### <a name="suse-linux-enterprise-server-12-supported-kernel-versions"></a>Поддерживаемые версии ядра SUSE Linux Enterprise Server 12

**Выпуск** | **Версия службы Mobility Service** | **Версия ядра** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533) | Поддерживаются все [ядра версии SUSE 12 SP1, SP2, SP3, SP4 и SP5](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> с 4.4.138-4.7-azure по 4.4.180-4.31-azure,</br>4.12.14-6.3-Azure в 4.12.14-6.43-Azure </br> 4.12.14-16,7-Azure в 4.12.14-16.44-Azure |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a) | Поддерживаются все [ядра версии SUSE 12 SP1, SP2, SP3, SP4 и SP5](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> с 4.4.138-4.7-azure по 4.4.180-4.31-azure,</br>4.12.14-6.3-Azure в 4.12.14-6.43-Azure </br> 4.12.14-16,7-Azure в 4.12.14-16.38-Azure|
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9,39](https://support.microsoft.com/help/4597409/) | Поддерживаются все [ядра версии SUSE 12 SP1, SP2, SP3, SP4 и SP5](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> с 4.4.138-4.7-azure по 4.4.180-4.31-azure,</br>4.12.14-6.3-Azure в 4.12.14-6.43-Azure </br> 4.12.14-16,7-Azure в 4.12.14-16.34-Azure |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9,38](https://support.microsoft.com/help/4590304/) | Поддерживаются все [ядра версии SUSE 12 SP1, SP2, SP3, SP4 и SP5](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> с 4.4.138-4.7-azure по 4.4.180-4.31-azure,</br>4.12.14-6.3-Azure в 4.12.14-6.43-Azure </br> 4.12.14-16,7-Azure в 4.12.14-16.28-Azure |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4, SP5) | [9,37](https://support.microsoft.com/help/4582666/),  | Поддерживаются все [ядра версии SUSE 12 SP1, SP2, SP3, SP4 и SP5](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> с 4.4.138-4.7-azure по 4.4.180-4.31-azure,</br>4.12.14-6.3-Azure в 4.12.14-6.43-Azure </br> 4.12.14-16,7-Azure в 4.12.14-16.22-Azure  |

### <a name="suse-linux-enterprise-server-15-supported-kernel-versions"></a>SUSE Linux Enterprise Server 15 поддерживаемых версий ядра

**Выпуск** | **Версия службы Mobility Service** | **Версия ядра** |
--- | --- | --- |
SUSE Linux Enterprise Server 15, SP1, SP2 | [9,41](https://support.microsoft.com/en-us/topic/update-rollup-54-for-azure-site-recovery-50873c7c-272c-4a7a-b9bb-8cd59c230533)  | По умолчанию поддерживаются все [ядра версии SUSE 15, SP1 и SP2](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> 4.12.14-5.5 — Azure — 4.12.14 — 5.47 — Azure </br></br> 4.12.14-8.5 — Azure — 4.12.14 — 8.55 — Azure </br> 5.3.18-16 — Azure </br> 5.3.18-18.5 — Azure — 5.3.18 — 18.35 — Azure
SUSE Linux Enterprise Server 15, SP1, SP2 | [9,40](https://support.microsoft.com/en-us/topic/update-rollup-53-for-azure-site-recovery-060268ef-5835-bb49-7cbc-e8c1e6c6e12a)  | По умолчанию поддерживаются все [ядра версии SUSE 15, SP1 и SP2](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> 4.12.14-5.5 — Azure — 4.12.14 — 5.47 — Azure </br></br> 4.12.14-8.5 — Azure — 4.12.14 — 8.55 — Azure </br> 5.3.18-16 — Azure </br> 5.3.18-18.5 — Azure — 5.3.18 — 18.29 — Azure
SUSE Linux Enterprise Server 15, SP1, SP2 | [9,39](https://support.microsoft.com/help/4597409/)  | По умолчанию поддерживаются все [ядра версии SUSE 15, SP1 и SP2](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> 4.12.14-5.5 — Azure — 4.12.14 — 5.47 — Azure </br></br> 4.12.14-8.5 — Azure — 4.12.14 — 8.47 — Azure </br> 5.3.18-16 — Azure </br> 5.3.18-18.5 — Azure — 5.3.18 — 18.21 — Azure
SUSE Linux Enterprise Server 15, SP1, SP2 | [9,38](https://support.microsoft.com/help/4590304/)  | По умолчанию поддерживаются все [ядра версии SUSE 15, SP1 и SP2](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> 4.12.14-5.5 — Azure — 4.12.14 — 5.47 — Azure </br></br> 4.12.14-8.5 — Azure — 4.12.14 — 8.44 — Azure </br> 5.3.18-16 — Azure </br> 5.3.18-18.5 — Azure — 5.3.18 — 18.18 — Azure
SUSE Linux Enterprise Server 15 и 15 SP1 | [9,37](https://support.microsoft.com/help/4582666/)  | По умолчанию поддерживаются все [ядра версии SUSE 15, SP1 и SP2](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> 4.12.14-5.5 — Azure — 4.12.14 — 5.47 — Azure </br></br> 4.12.14-8.5 — Azure — 4.12.14 — 8.38 — Azure

## <a name="linux-file-systemsguest-storage"></a>Файловые системы Linux и гостевое хранилище

**Компонент** | **Поддерживается**
--- | ---
Файловые системы | ext3, ext4, XFS, БТРФС (условия, применимые в этой таблице)
Подготовка управления логическими томами (LVM)| Толстая инициализация — да <br></br> Тонкая инициализация — нет
Диспетчер томов | -LVM поддерживается.<br/> параметр-/Boot в LVM поддерживается начиная с [накопительного пакета обновления 31](https://support.microsoft.com/help/4478871/) (версия 9,20 службы Mobility Service). Она не поддерживается в предыдущих версиях службы Mobility Service.<br/> -Несколько дисков ОС не поддерживаются.
Паравиртуализированные запоминающие устройства | Устройства, экспортированные с помощью паравиртуализированных драйверов, не поддерживаются.
Блочные устройства ввода-вывода с несколькими очередями | Не поддерживается.
Физические серверы с контроллером хранилища HP CCISS | Не поддерживается.
Соглашение об именовании точки устройства/подключения | Имя устройства или точки подключения должно быть уникальным.<br/> Убедитесь, что имена двух устройств и точек подключения не содержат учет регистра. Например, имена устройств для одной виртуальной машины с *device1* и *device1* не поддерживаются.
Каталоги | Если вы используете версию службы Mobility Service, предшествующую версии 9,20 (выпущенную в [накопительном пакете обновления 31](https://support.microsoft.com/help/4478871/)), то действуют следующие ограничения.<br/><br/> — Эти каталоги (если они настроены как отдельные разделы и файловые системы) должны находиться на одном диске ОС на исходном сервере:/(root),/Boot,/usr,/usr/local,/var,/etc.</br> — Каталог/boot должен находиться в разделе диска, а не в томе LVM.<br/><br/> Начиная с версии 9,20 эти ограничения не применяются. 
Каталог загрузок | — Загрузочные диски не должны должны быть в формате раздела GPT. Это ограничение архитектуры Azure. Диски GPT поддерживаются в качестве дисков данных.<br/><br/> Несколько загрузочных дисков на виртуальной машине не поддерживаются.<br/><br/> -/Boot на томе LVM на нескольких дисках не поддерживается.<br/> — Невозможно реплицировать компьютер без загрузочного диска.
Требования к свободному месту| 2 ГБ в разделе /root <br/><br/> 250 МБ в папке установки
XFSv5 | Поддерживаются функции XFSv5 в файловых системах XFS, таких как контрольная сумма метаданных (служба Mobility Service версии 9,10).<br/> Используйте служебную программу xfs_info, чтобы проверить системный блок XFS для раздела. Если параметр `ftype` имеет значение 1, то используются XFSv5 компоненты.
бтрфс | БТРФС поддерживается из [накопительного пакета обновления 34](https://support.microsoft.com/help/4490016) (версия 9,22 службы Mobility Service). БТРФС не поддерживается, если:<br/><br/> — БТРФС файловая система файловой системы изменяется после включения защиты.</br> — Файловая система БТРФС распределена по нескольким дискам.</br> — Файловая система БТРФС поддерживает RAID.

## <a name="vmdisk-management"></a>Управление виртуальной машиной и диском

**Действие** | **Сведения**
--- | ---
Изменение размера диска на реплицируемой виртуальной машине | Поддерживается на исходной виртуальной машине перед отработкой отказа непосредственно в свойствах виртуальной машины. Отключение и повторное включение репликации не требуются.<br/><br/> При изменении исходной виртуальной машины после отработки отказа изменения не фиксируются.<br/><br/> При изменении размера диска на виртуальной машине Azure после отработки отказа при восстановлении после сбоя Site Recovery создает новую виртуальную машину с обновлениями.
Добавление диска к реплицируемой виртуальной машине | Не поддерживается.<br/> Отключите репликацию для виртуальной машины, добавьте диск, а затем снова включите репликацию.

> [!NOTE]
> Любые изменения в удостоверении диска не поддерживаются. Например, если секционирование диска было изменено с GPT на MBR или наоборот, это приведет к изменению удостоверения диска. В таком случае репликация будет нарушена, и потребуется новая установка. Для компьютеров Linux изменение имени устройства не поддерживается, так как оно влияет на удостоверение диска.

## <a name="network"></a>Сеть

**Компонент** | **Поддерживается**
--- | ---
Объединение сетевых карт в сети узла | Поддерживается для виртуальных машин VMware. <br/><br/>Не поддерживается для репликации физического компьютера.
Виртуальная локальная сеть узла | Да.
IPv4-адрес сети узла | Да.
IPv6-адрес сети узла | Нет.
Объединение гостевых сетевых карт в сети гостевой системы или сервера | Нет.
IPv4-адрес cети клиента или сервера | Да.
IPv6-адрес cети клиента или сервера | Нет.
Статический IP-адрес (Windows) сети клиента или сервера | Да.
Статический IP-адрес (Linux) cети клиента или сервера | Да. <br/><br/>Виртуальные машины настроены для использования DHCP при восстановлении размещения.
Несколько сетевых адаптеров cети клиента или сервера | Да.
Доступ к службе Site Recovery с закрытыми ссылками | Да. [Подробнее.](hybrid-how-to-enable-replication-private-endpoints.md)


## <a name="azure-vm-network-after-failover"></a>Сеть виртуальной машины Azure (после отработки отказа)

**Компонент** | **Поддерживается**
--- | ---
Azure ExpressRoute | Да
Внутренний балансировщик нагрузки | Да
Внешний балансировщик нагрузки | Да
Azure Traffic Manager | Да
Несколько сетевых адаптеров | Да
Зарезервированный IP-адрес | Да
IPv4 | Да
Сохранение исходного IP-адреса | Да
Конечные точки служб для виртуальной сети Azure<br/> | Да
Ускорение работы в сети | нет

## <a name="storage"></a>Память
**Компонент** | **Поддерживается**
--- | ---
Динамический диск | Диск ОС должен быть базовым диском. <br/><br/>Диски данных могут быть динамическими дисками.
Конфигурация дисков Docker | Нет
NFS узла | Да для VMware<br/><br/> Нет для физических серверов
SAN узла (ISCSI или FC) | Да
vSAN узла | Да для VMware<br/><br/> Недоступно для физических серверов.
Операции многопутевого ввода-вывода (MPIO) для узла | Да — протестировано с помощью Microsoft DSM, EMC PowerPath 5.7 SP4 и EMC PowerPath DSM для CLARiiON
Виртуальные тома узла (VVol) | Да для VMware<br/><br/> Недоступно для физических серверов.
VMDK клиента или сервера | Да
Общий диск кластера клиента или сервера | Нет
Зашифрованный диск клиента или сервера | Нет
NFS клиента или сервера | Нет
ISCSI гостя или сервера | Для миграции — да<br/>Для аварийного восстановления — нет, iSCSI выполнит восстановление размещения в качестве подключенного диска для виртуальной машины.
SMB 3.0 клиента или сервера | Нет
RDM клиента или сервера | Да<br/><br/> Недоступно для физических серверов.
Диск размером более 1 ТБ для клиента или сервера | Да, диск должен быть больше 1024 МБ<br/><br/>До 8 192 ГБ при репликации на управляемые диски (9,26 версии)<br></br> До 4 095 ГБ при репликации в учетные записи хранения
Диск клиента или сервера с физическими и логическими секторами с размером в 4 КБ | Нет
Диск гостя или сервера с логическим размером 4 КБ и 512-байтами физический сектор | Нет
Том с чередующимся диском размером более 4 ТБ для гостевой системы или сервера | Да
Управление логическими томами (LVM)| Толстая подготовка — да <br></br> Тонкая подготовка — нет
Дисковые пространства для гостя или сервера | Нет
Гость/сервер-интерфейс NVMe | Нет
"Горячее" добавление и удаление дисков для гостя или сервера | Нет
Исключение диска для гостя или сервера | Да
Поддержка операций многопутевого ввода-вывода (MPIO) для гостевой системы или сервера | Нет
Разделы GPT для гостевых и серверных систем | Из [накопительного пакета обновления 37](https://support.microsoft.com/help/4508614/) (версия 9,25 службы Mobility Service) поддерживаются пять секций. Поддерживались четыре ранее.
ReFS | Отказоустойчивая файловая система поддерживается в службе Mobility Service версии 9,23 или более поздней
Загрузка из виртуальной машины/сервера EFI/UEFI | — Поддерживается всеми [операционными системами UEFI Azure Marketplace](../virtual-machines/generation-2.md#generation-2-vm-images-in-azure-marketplace) с Site Recovery агента мобильности версии 9,30. <br/> -Безопасный тип загрузки UEFI не поддерживается. [Подробнее.](../virtual-machines/generation-2.md#on-premises-vs-azure-generation-2-vms)

## <a name="replication-channels"></a>Каналы репликации

|**Тип репликации**   |**Поддерживается**  |
|---------|---------|
|Разгрузка передачи данных (ODX)    |       Нет  |
|Автономное заполнение        |   Нет      |
| Azure Data Box | Нет

## <a name="azure-storage"></a>Хранилище Azure

**Компонент** | **Поддерживается**
--- | ---
Локально избыточное хранилище | Да
Геоизбыточное хранилище | Да
Геоизбыточное хранилище с доступом для чтения | Да
"Холодное" хранилище | Нет
"Горячее" хранилище| Нет
Blob-блоки | Нет
Шифрование неактивных (SSE)| Да
Шифрование неактивных (CMK)| Да (с помощью PowerShell AZ 3.3.0 Module)
Двойное шифрование при хранении | Да (с помощью PowerShell AZ 3.3.0 Module). Дополнительные сведения см. в статье Поддерживаемые регионы для [Windows](../virtual-machines/disk-encryption.md) и [Linux](../virtual-machines/disk-encryption.md).
Хранилище уровня "Премиум" | Да
Параметр безопасной пересылки | Да
Служба импорта и экспорта | Нет
Брандмауэры службы хранилища Azure для виртуальных сетей | Да.<br/> Настроена для целевой учетной записи хранения или кэша (используется для хранения данных репликации).
Учетные записи хранения общего назначения версии 2 (горячий и холодно-уровни) | Да (затраты на транзакции значительно выше для версии 2 по сравнению с v1)

## <a name="azure-compute"></a>Служба вычислений Azure

**Возможность** | **Поддерживается**
--- | ---
Группы доступности | Да
Зоны доступности | Нет
Концентратор | Да
Управляемые диски | Да

## <a name="azure-vm-requirements"></a>Требования для виртуальных машин Azure

Локальные виртуальные машины, реплицируемые в Azure, должны соответствовать требованиям к виртуальным машинам Azure, приведенным в этой таблице. Если Site Recovery выполняет проверку предварительных требований для репликации, проверка завершится ошибкой, если некоторые требования не будут выполнены.

**Компонент** | **Requirements** | **Сведения**
--- | --- | ---
Операционная система на виртуальной машине | [Поддерживаемые операционные системы для реплицируемых виртуальных машин](#replicated-machines) | Если не поддерживается, проверка завершается ошибкой.
Архитектура операционной системы на виртуальной машине | 64-разрядная | Если не поддерживается, проверка завершается ошибкой.
Размер диска операционной системы | До 2 048 ГБ для компьютеров поколения 1. <br> До 4 095 ГБ для компьютеров поколения 2. | Если не поддерживается, проверка завершается ошибкой.
Число дисков операционной системы | 1 </br> загрузочный и системный разделы на разных дисках не поддерживаются. | Если не поддерживается, проверка завершается ошибкой.
Число дисков с данными | 64 ГБ и меньше | Если не поддерживается, проверка завершается ошибкой.
Размер диска данных | До 32 767 ГБ при репликации на управляемый диск (9,41 версии)<br> До 4 095 ГБ при репликации в учетную запись хранения </br> Требования к минимальному размеру диска — не менее 1024 МБ| Если не поддерживается, проверка завершается ошибкой.
Сетевые адаптеры | Поддерживаются несколько адаптеров. |
Общий виртуальный жесткий диск | Не поддерживается. | Если не поддерживается, проверка завершается ошибкой.
Диск FC | Не поддерживается. | Если не поддерживается, проверка завершается ошибкой.
BitLocker | Не поддерживается. | Прежде чем включать репликацию для компьютера, необходимо отключить BitLocker. |
имя виртуальной машины; | От 1 до 63 символов.<br/><br/> при этом допустимы только буквы, цифры и дефисы<br/><br/> Имя компьютера должно начинаться и заканчиваться буквой или цифрой. |  Обновите значение в свойствах компьютера в службе Site Recovery.

## <a name="resource-group-limits"></a>Ограничения группы ресурсов

Сведения о количестве виртуальных машин, которые можно защитить в одной группе ресурсов, см. в статье [ограничения и квоты подписки](../azure-resource-manager/management/azure-subscription-service-limits.md#resource-group-limits).

## <a name="churn-limits"></a>Ограничения на обновление

В таблице ниже приведены ограничения Azure Site Recovery.
- Эти ограничения основаны на наших тестах, но не охватывают все возможные сочетания операций ввода-вывода в приложении.
- Фактические результаты зависят от сочетания операций ввода-вывода приложения.
- Для получения наилучших результатов настоятельно рекомендуется запустить [средство планировщик развертывания](site-recovery-deployment-planner.md)и выполнить расширенное тестирование приложений с помощью тестовой отработки отказа, чтобы получить истинную картину производительности приложения.

**Целевой объект репликации** | **Средний размер ввода-вывода исходного диска** |**Средняя скорость обработки данных исходного диска** | **Общий размер обработанных данных на исходном диске за день**
---|---|---|---
Хранилище уровня "Стандартный" | 8 КБ    | 2 МБ/с | 168 ГБ на диск
Диск P10 или P15 класса Premium | 8 КБ    | 2 МБ/с | 168 ГБ на диск
Диск P10 или P15 класса Premium | 16 КБ | 4 МБ/с |    336 ГБ на диск
Диск P10 или P15 класса Premium | 32 КБ или выше | 8 МБ/с | 672 ГБ на диск
Диск P20, P30, P40 или P50 класса Premium | 8 КБ    | 5 МБ/с | 421 ГБ на диск
Диск P20, P30, P40 или P50 класса Premium | 16 КБ или выше |20 МБ/с | 1684 ГБ на диск


**Исходная скорость обработки данных** | **Максимальное ограничение**
---|---
Пиковая скорость обработки данных на всех дисках виртуальной машины | 54 МБ/с
Максимальный объем обработки данных, поддерживаемый сервером обработки | 2 ТБ

- В указанных средних значениях предполагается 30-процентное перекрытие операций ввода-вывода.
- В зависимости от коэффициента перекрытия, размера записи и фактической рабочей нагрузки ввода-вывода Site Recovery может обрабатывать более высокую пропускную способность.
- Эти числа предполагают типичную невыполненную работу примерно через пять минут. то есть после передачи выполняется обработка данных, а созданная точка восстановления составляет 5 минут.

## <a name="storage-account-limits"></a>Ограничения учетной записи хранения

По мере увеличения среднего количества изменений на дисках количество дисков, которые учетная запись хранения может поддерживать, уменьшается. Приведенную ниже таблицу можно использовать в качестве рекомендации для принятия решений о количестве учетных записей хранения, которые необходимо подготовить.
 
**Тип учетной записи хранения**    |    **Обновление = 4 Мбит/с на диск**    |    **Обновление = 8 Мбит/с на диск**
---    |    ---    |    ---
Учетная запись хранения v1    |    диски 600    |    диски 300
V2 учетная запись хранения    |    диски 1500    |    диски 750

Обратите внимание, что указанные выше ограничения применимы только к сценариям гибридного аварийного восстановления.

## <a name="vault-tasks"></a>Задачи хранилища

**Действие** | **Поддерживается**
--- | ---
Перемещение хранилища между группами ресурсов | Нет
Перемещение хранилища между подписками и между ними | Нет
Перемещение хранилищ, сетей, виртуальных машин Azure между группами ресурсов | Нет
Перемещение хранилища, сети, виртуальных машин Azure в рамках и между подписками. | Нет


## <a name="obtain-latest-components"></a>Получение последних компонентов

**имя**; | **Описание** | **Сведения**
--- | --- | ---
Сервер конфигурации | Установлен локально.<br/> Координирует обмен данными между локальными серверами VMware или физическими компьютерами и Azure. | - [Сведения о](vmware-physical-azure-config-process-server-overview.md) сервере конфигурации.<br/> - [Сведения об](vmware-azure-manage-configuration-server.md#upgrade-the-configuration-server) обновлении до последней версии.<br/> - [Сведения о](vmware-azure-deploy-configuration-server.md) настройке сервера конфигурации.
Сервер обработки | По умолчанию устанавливается на сервере конфигурации.<br/> Получает данные репликации, оптимизирует их с помощью кэширования, сжатия и шифрования и отправляет в Azure.<br/> По мере роста развертывания можно добавить дополнительные серверы обработки для обработки больших объемов трафика репликации. | - [Сведения о](vmware-physical-azure-config-process-server-overview.md) сервере обработки.<br/> - [Сведения об](vmware-azure-manage-process-server.md#upgrade-a-process-server) обновлении до последней версии.<br/> - [Дополнительные сведения о](vmware-physical-large-deployment.md#set-up-a-process-server) настройке серверов обработки масштабирования.
Служба Mobility Service | Устанавливается на виртуальных машинах VMware или физических серверах, которые необходимо реплицировать.<br/> Координирует репликацию между локальными серверами VMware или физическими серверами и Azure.| - [Сведения о](vmware-physical-mobility-service-overview.md) службе Mobility Service.<br/> - [Сведения об](vmware-physical-manage-mobility-service.md#update-mobility-service-from-azure-portal) обновлении до последней версии.<br/>



## <a name="next-steps"></a>Дальнейшие действия
[Узнайте, как](tutorial-prepare-azure.md) подготовить Azure для аварийного восстановления виртуальных машин VMware.

[9.32 UR]: https://support.microsoft.com/en-in/help/4538187/update-rollup-44-for-azure-site-recovery
[9.31 UR]: https://support.microsoft.com/en-in/help/4537047/update-rollup-43-for-azure-site-recovery
[9.30 UR]: https://support.microsoft.com/en-in/help/4531426/update-rollup-42-for-azure-site-recovery
[9.29 UR]: https://support.microsoft.com/en-in/help/4528026/update-rollup-41-for-azure-site-recovery
[9.28 UR]: https://support.microsoft.com/en-in/help/4521530/update-rollup-40-for-azure-site-recovery
[9.27 UR]: https://support.microsoft.com/en-in/help/4517283/update-rollup-39-for-azure-site-recovery
[9.26 UR]: https://support.microsoft.com/en-in/help/4513507/update-rollup-38-for-azure-site-recovery
[9.25 UR]: https://support.microsoft.com/en-in/help/4508614/update-rollup-37-for-azure-site-recovery
[9.24 UR]: https://support.microsoft.com/en-in/help/4503156
[9.23 UR]: https://support.microsoft.com/en-in/help/4494485/update-rollup-35-for-azure-site-recovery
[9.22 UR]: https://support.microsoft.com/help/4489582/update-rollup-33-for-azure-site-recovery
[9.21 UR]: https://support.microsoft.com/help/4485985/update-rollup-32-for-azure-site-recovery
[9.20 UR]: https://support.microsoft.com/help/4478871/update-rollup-31-for-azure-site-recovery
[9.19 UR]: https://support.microsoft.com/help/4468181/azure-site-recovery-update-rollup-30
[9.18 UR]: https://support.microsoft.com/help/4466466/update-rollup-29-for-azure-site-recovery

---
title: Оценка виртуальных машин Azure в службе "миграция Azure"
description: Сведения об оценках в службе "миграция Azure"
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 05/27/2020
ms.openlocfilehash: 16c3b59bcfa14cc02f13dadd726e0380d934598b
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105023815"
---
# <a name="assessment-overview-migrate-to-azure-vms"></a>Обзор оценки (миграция на виртуальные машины Azure)

В этой статье представлен обзор оценок в средстве " [Миграция Azure: обнаружение и оценка](migrate-services-overview.md#azure-migrate-discovery-and-assessment-tool) ". Средство может оценивать локальные серверы в виртуальных средах VMware и в среде Hyper-V, а также физические серверы для миграции в Azure.

## <a name="whats-an-assessment"></a>Что такое оценка?

Оценка с помощью средства обнаружения и оценки измеряет готовность и оценивает воздействие миграции локальных серверов на Azure.

> [!NOTE]
> В Azure для государственных организаций ознакомьтесь с [поддерживаемыми целевыми](migrate-support-matrix.md#supported-geographies-azure-government) расположениями оценки. Обратите внимание, что рекомендации по размерам виртуальных машин в оценках будут использовать серию виртуальных машин специально для государственных облачных регионов. Дополнительные [сведения](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia&products=virtual-machines) о типах виртуальных машин.

## <a name="types-of-assessments"></a>Типы оценок

Существует три типа оценок, которые можно создать с помощью функции "миграция Azure": обнаружение и оценка.

***Тип оценки** | **Сведения**
--- | --- 
**Azure** | Оценки для переноса локальных серверов на виртуальные машины Azure. Вы можете оценивать локальные серверы в среде [VMware](how-to-set-up-appliance-vmware.md) и [Hyper-V](how-to-set-up-appliance-hyper-v.md) , а также [физические серверы](how-to-set-up-appliance-physical.md) для миграции на виртуальные машины Azure с помощью этого типа оценки.
**Azure SQL;** | Оценки для переноса локальных серверов SQL Server из среды VMware в базу данных SQL Azure или Azure SQL Управляемый экземпляр.
**Решение Azure VMware (AVS)** | Оценки для переноса локальных серверов в [Решение Azure VMware (AVS)](../azure-vmware/introduction.md). С помощью этого типа оценки вы можете оценить возможность миграции в Решение Azure VMware (AVS) для локальных [виртуальных машин VMware](how-to-set-up-appliance-vmware.md). [Дополнительные сведения](concepts-azure-vmware-solution-assessment-calculation.md)

Оценки, создаваемые с помощью службы "миграция Azure", — это моментальный снимок данных на момент времени. Оценка виртуальных машин Azure предоставляет два варианта критериев изменения размера:

**Тип оценки** | **Сведения** | **Data**
--- | --- | ---
**На основе производительности** | Оценки, которые предоставляют рекомендации на основе собранных данных о производительности | Рекомендуемый размер виртуальной машины основан на данных ЦП и использовании памяти.<br/><br/> Рекомендуемый тип диска зависит от числа операций ввода-вывода в секунду и пропускной способности локальных дисков. Типы дисков: Azure HDD (цен. категория "Стандартный"), Azure SSD (цен. категория "Стандартный") и Azure Premium.
**Как в локальной среде** | Оценки, которые не используют данные о производительности для создания рекомендаций | Рекомендуемый размер виртуальной машины зависит от размера локального сервера.<br/><br> Рекомендуемый тип диска зависит от выбранного типа хранилища для оценки.

## <a name="how-do-i-run-an-assessment"></a>Разделы справки выполнить оценку?

Существует несколько способов выполнения оценки.

- Оцените серверы с помощью метаданных сервера, собранных упрощенным устройством миграции Azure. Устройство обнаруживает локальные серверы. Затем он отправляет метаданные сервера и данные о производительности в службу "миграция Azure".
- Оценка серверов с помощью метаданных сервера, импортированных в формате значений с разделителями-запятыми (CSV).

## <a name="how-do-i-assess-with-the-appliance"></a>Разделы справки оценить с помощью устройства?

Если вы развертываете устройство миграции Azure для обнаружения локальных серверов, выполните следующие действия.

1. Настройте Azure и локальную среду для работы с миграцией Azure.
1. Для первой оценки создайте проект Azure и добавьте к нему средство обнаружения и оценки.
1. Разверните упрощенное устройство миграции Azure. Устройство постоянно обнаруживает локальные серверы и отправляет метаданные сервера и данные о производительности в службу "миграция Azure". Разверните устройство как виртуальную машину или физический сервер. Вам не нужно ничего устанавливать на серверах, которые вы хотите оценить.

После того как устройство начнет обнаружение серверов, вы можете собрать серверы, которые нужно оценить в группу, и выполнить оценку группы с помощью типа оценки **Виртуальная машина Azure**.

Чтобы выполнить эти действия, следуйте нашим руководствам по [VMware](./tutorial-discover-vmware.md), [Hyper-V](./tutorial-discover-hyper-v.md)или [физическим серверам](./tutorial-discover-physical.md) .

## <a name="how-do-i-assess-with-imported-data"></a>Разделы справки оценить с помощью импортированных данных?

Если вы оцениваете серверы с помощью CSV-файла, устройство не требуется. Вместо этого выполните следующие действия.

1. Настройка Azure для работы с миграцией Azure
1. Для первой оценки создайте проект Azure и добавьте к нему средство обнаружения и оценки.
1. Скачайте шаблон CSV-файла и добавьте в него данные сервера.
1. Импорт шаблона в службу "миграция Azure"
1. Откройте серверы, добавленные с помощью импорта, соберите их в группу и выполните оценку группы с типом оценки **Виртуальная машина Azure**.

## <a name="what-data-does-the-appliance-collect"></a>Какие данные собираются устройством?

Если вы используете устройство "миграция Azure" для оценки, ознакомьтесь с метаданными и данными о производительности, собранными для [VMware](migrate-appliance.md#collected-data---vmware) и [Hyper-V](migrate-appliance.md#collected-data---hyper-v).

## <a name="how-does-the-appliance-calculate-performance-data"></a>Как устройство вычисляет данные о производительности?

Если вы используете устройство для обнаружения, оно собирает данные о производительности для параметров вычислений, выполнив следующие действия.

1. Устройство собирает образец точки в режиме реального времени.

    - **Виртуальные машины VMware**. выборка собирается каждые 20 секунд.
    - **Виртуальные машины Hyper-V**. выборка собирается каждые 30 секунд.
    - **Физические серверы**. Пример сбора собираются каждые пять минут.

1. Устройство объединяет образцы точек для создания одной точки данных каждые 10 минут для серверов VMware и Hyper-V, а также каждые 5 минут для физических серверов. Чтобы создать точку данных, устройство выбирает пиковые значения из всех выборок. Затем она отправляет точку данных в Azure.
1. В ходе оценки хранятся все 10-минутные точки данных за прошлый месяц.
1. При создании оценки Оценка определяет соответствующую точку данных, которая будет использоваться для оптимизации дается задание. Идентификация основана на значениях процентиля для *журнала производительности* и *использования процентилей*.

    - Например, если журнал производительности составляет одну неделю, а заданный процентиль — 95 процентиль процентиль, то оценка сортирует 10-минутные точки выборки за последнюю неделю. Он сортирует их в возрастающем порядке и выбирает значение 95 процентиль процентиль для оптимизации дается задание.
    - Значение 95 процентиль процентиль гарантирует игнорирование любых выбросов, которые могут быть добавлены, если вы выбрали 99-м процентиль.
    - Если вы хотите выбрать пиковое использование для периода и не хотите пропускать выбросы, выберите 99-м процентиль для использования процентилей.

1. Это значение умножается на эффективный фактор, чтобы получить эффективные данные об использовании производительности для метрик, собираемых устройством:

    - загрузка ЦП;
    - Использование ОЗУ
    - Дисковые операции ввода-вывода (чтение и запись)
    - Пропускная способность диска (чтение и запись)
    - Пропускная способность сети (в и в)

## <a name="how-are-azure-vm-assessments-calculated"></a>Как рассчитываются оценки виртуальных машин Azure?

При оценке используются метаданные и данные производительности локальных серверов для вычисления оценок. При развертывании устройства "миграция Azure" для оценки используются данные, собираемые устройством. Но если вы выполняете оценку, импортированную с помощью CSV-файла, вы предоставляете метаданные для вычисления.

Вычисления выполняются в следующих трех этапах:

1. **Рассчитайте готовность Azure**: оцените, подходят ли серверы для миграции в Azure.
1. **Рассчитайте рекомендации по выбору размера**: Оценка вычислений, хранилища и размера сети.
1. **Вычисление месячных затрат**. Вычислите расчетные ежемесячные затраты на вычисления и хранение для запуска серверов в Azure после миграции.

Вычисления выполняются в предыдущем порядке. Сервер сервера переходит на более позднюю стадию, только если он проходит предыдущий. Например, если сервер не проходит этап готовности Azure, он помечается как неподходящий для Azure. Расчет размера и затрат для этого сервера не выполняется.

## <a name="whats-in-an-azure-vm-assessment"></a>Что такое оценка виртуальной машины Azure?

Вот именно, входящие в оценку виртуальной машины Azure:

**Свойство** | **Сведения**
--- | ---
**Целевое расположение** | Расположение, в которое необходимо выполнить миграцию. Сейчас Оценка поддерживает следующие целевые регионы Azure:<br/><br/> Восточная Австралия, юго-восток Австралии, Южная Бразилия, Центральная Канада, Восточная Канада, Центральная Индия, Центральная часть США, Восточный Китай, Северный Китай, Восточная Азия, Восточная часть США, Восточная часть США 2, Центральная Германия, северо-восток, Восточная Япония, Западная Япония, Центральная Европа, Юго-Центральный регион США, Юго-Восточная Азия, Южная Индия, Северная, Южная часть Соединенного Королевства , Западная Центральная часть США, Западная Европа, Западная Индия, Западная часть США и Западная часть США 2.
**Целевой диск хранилища (как и размер)** | Тип диска, используемый для хранения данных в Azure. <br/><br/> Укажите целевой диск хранилища в качестве управляемого с помощью уровня "Премиум", управляемого SSD (цен. категория "Стандартный") или управляемого HDD (цен. категория "Стандартный").
**Целевой диск хранилища (размер на основе производительности)** | Указывает тип целевого диска хранилища: Автоматический, управляемый с помощью уровня "Премиум", управляемый HDD (цен. категория "Стандартный") или управляемый SSD (цен. категория "Стандартный").<br/><br/> **Автоматически**. Рекомендуемый диск основан на данных производительности дисков, что означает количество операций ввода-вывода и пропускную способность.<br/><br/>**Premium или Standard**. Оценка рекомендует номер SKU диска в выбранном типе хранилища.<br/><br/> Если вы хотите использовать соглашение об уровне обслуживания виртуальных машин с одним экземпляром (SLA) 99,9%, рассмотрите возможность использования управляемых дисков уровня "Премиум". Это гарантирует, что все диски в оценке рекомендуются в качестве дисков, управляемых с помощью класса Premium.<br/><br/> Служба "миграция Azure" поддерживает только управляемые диски для оценки миграции.
**Зарезервированные экземпляры виртуальных машин Azure** | Указывает [зарезервированные экземпляры](https://azure.microsoft.com/pricing/reserved-vm-instances/) таким образом, чтобы оценки затрат в оценке проводились в расчете.<br/><br/> При выборе "зарезервированные экземпляры" скидка (%) и свойства "время работы виртуальной машины" неприменимы.<br/><br/> Сейчас служба "миграция Azure" поддерживает Azure Reserved VM Instances только для предложений с оплатой по мере использования.
**Критерии определения размера** | Используется для подобрать правильный размер виртуальной машины Azure.<br/><br/> Использование AS зависит от размера или размера, основанного на производительности.
**Журнал производительности** | Используется с изменением размера на основе производительности. Журнал производительности указывает продолжительность, используемую при оценке данных производительности.
**Использование процентиля** | Используется с изменением размера на основе производительности. Использование процентиля указывает значение процентиля образца производительности, используемого для оптимизации дается задание.
**Серия виртуальных машин** | Серия виртуальных машин Azure, которую вы хотите рассмотреть для оптимизации дается задание. Например, если у вас нет рабочей среды, которая требует виртуальных машин Azure серии А, эту серию можно исключить из списка серий.
**Фактор комфорта** | Буфер, используемый во время оценки. Он применяется к данным ЦП, ОЗУ, диску и сети для виртуальных машин. Они учетируют такие проблемы, как сезонное использование, короткий журнал производительности и, вероятнее всего, увеличение использования в будущем.<br/><br/> Например, 10-ядерная виртуальная машина с использованием 20% обычно приводит к использованию 2-ядерной виртуальной машины. В случае с незначительным фактором 2,0 в результате вместо этого используется 4-ядерная виртуальная машина.
**Предложение** | [Предложение Azure](https://azure.microsoft.com/support/legal/offer-details/) , в котором вы зарегистрированы. Оценка оценивает стоимость этого предложения.
**Денежный формат** | Валюта выставления счетов для вашей учетной записи.
**Скидка (%)** | Все скидки, относящиеся к подписке, полученные на основе предложения Azure. Значение по умолчанию — 0 %.
**Время доступности виртуальной машины** | Продолжительность в днях в месяц и часах в день для виртуальных машин Azure, которые не будут выполняться непрерывно. Оценка затрат основана на этой длительности.<br/><br/> Значения по умолчанию — 31 день в месяц и 24 часа в день.
**Преимущество гибридного использования Azure** | Указывает, есть ли у вас программа Software Assurance и что она подходит для [преимущество гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/). Если для параметра задано значение по умолчанию "Да", цены Azure для операционных систем, отличных от Windows, будут учитываться для виртуальных машин Windows.
**Подписка EA** | Указывает, что для оценки затрат используется подписка Соглашение Enterprise (EA). Учитывает скидку, применимую к подписке. <br/><br/> Оставьте параметры для зарезервированных экземпляров, скидка (%) и свойства времени работы виртуальной машины с их параметрами по умолчанию.


[Ознакомьтесь с рекомендациями](best-practices-assessment.md) по созданию оценки с помощью службы "миграция Azure".

## <a name="calculate-readiness"></a>Вычислить готовность

Не все серверы подходят для работы в Azure. Оценка виртуальных машин Azure оценивает все локальные серверы и назначает им категорию готовности.

- **Готово к работе с Azure**. сервер можно перенести "как есть" в Azure без каких-либо изменений. Он будет запущен в Azure с полной поддержкой Azure.
- **Условно готово к работе в Azure**. сервер может запускаться в Azure, но может не поддерживать полную поддержку Azure. Например, Azure не поддерживает сервер, на котором работает старая версия Windows Server. Перед миграцией этих серверов в Azure необходимо соблюдать осторожность. Чтобы устранить проблемы готовности, следуйте указаниям по исправлению, предлагаемому при оценке.
- **Не готово к работе в Azure**. сервер не будет запущен в Azure. Например, если на диске локального сервера хранится более 64 ТБ, в Azure невозможно разместить сервер. Следуйте указаниям по исправлению, чтобы устранить проблему перед миграцией.
- **Неизвестная готовность**: службе "миграция Azure" не удается определить готовность сервера из-за недостаточности метаданных.

Для вычисления готовности Оценка проверяет свойства сервера и параметры операционной системы, приведенные в следующих таблицах.

### <a name="server-properties"></a>Свойства сервера

Для оценки виртуальных машин Azure Оценка проверяет следующие свойства локальной виртуальной машины, чтобы определить, можно ли ее запустить на виртуальных машинах Azure.

Свойство | Сведения | Состояние готовности к работе в Azure
--- | --- | ---
**Тип загрузки** | Azure поддерживает виртуальные машины с типом загрузки BIOS, а не UEFI. | Условно готово, если тип загрузки — UEFI
**Ядра** | На каждом сервере должно быть не более 128 ядер. это максимальное число, поддерживаемое виртуальной машиной Azure.<br/><br/> Если доступен журнал производительности, служба "Миграция Azure" рассматривает используемые ядра для сравнения. Если параметры оценки указывают на неэффективное использование, количество используемых ядер умножается на коэффициент использования.<br/><br/> Если журнал производительности отсутствует, служба "миграция Azure" использует выделенные ядра для применения коэффициента отдыха. | Готово, если число ядер находится в пределах ограничения
**ОЗУ** | На каждом сервере должно быть не более 3 892 ГБ ОЗУ. это максимальный размер, поддерживаемый виртуальной машиной Azure серии M Standard_M128m &nbsp; <sup>2</sup> . [Подробнее](../virtual-machines/sizes.md).<br/><br/> Если доступен журнал производительности, служба "миграция Azure" считает использование ОЗУ для сравнения. Если указан отличный фактор, использование ОЗУ умножается на коэффициент использования.<br/><br/> Если журнал отсутствует, то для применения этого курса используется выделенная память.<br/><br/> | Готово, если объем ОЗУ в пределах ограничения
**Диск хранилища** | Выделенный размер диска не должен превышать 32 ТБ. Хотя Azure поддерживает диски 64-ТБ с дисками SSD (цен. категория "Ультра") Azure, оценка в настоящее время проверяется на наличие на диске 32 ТБ в качестве предельного размера диска, так как он еще не поддерживает SSD (цен. категория "Ультра"). <br/><br/> Число дисков, подключенных к серверу, включая диск ОС, должно быть не 65 или меньше. | Готово, если размер и число дисков преобразуются в пределы
**Сеть** | К серверу должно быть подключено не более 32 сетевых интерфейсов (NIC). | Готово, если число сетевых карт в пределах ограничения

### <a name="guest-operating-system"></a>Операционная система на виртуальной машине

Для оценки виртуальных машин Azure, а также для проверки свойств виртуальной машины Оценка просматривает операционную систему на сервере, чтобы определить, может ли он работать в Azure.

> [!NOTE]
> Для обработки гостевого анализа виртуальных машин VMware Оценка использует операционную систему, указанную для виртуальной машины в vCenter Server. Однако vCenter Server не предоставляет версию ядра для операционных систем Linux на виртуальных машинах. Чтобы найти версию, необходимо настроить [Обнаружение приложений](./how-to-discover-applications.md). Затем устройство обнаруживает сведения о версии, используя учетные данные гостя, указанные при настройке обнаружения приложений.


Для оценки готовности Azure на основе операционной системы Оценка использует следующую логику.

**Операционная система** | **Сведения** | **Состояние готовности к работе в Azure**
--- | --- | ---
Windows Server 2016 и все пакеты обновления | Azure обеспечивает полную поддержку. | Готово к работе в Azure.
Windows Server 2012 R2 и все пакеты обновления | Azure обеспечивает полную поддержку. | Готово к работе в Azure.
Windows Server 2012 и все пакеты обновления | Azure обеспечивает полную поддержку. | Готово к работе в Azure.
Windows Server 2008 R2 со всеми пакетами обновления | Azure обеспечивает полную поддержку.| Готово к работе в Azure.
Windows Server 2008 (32-разрядная и 64-разрядная) | Azure обеспечивает полную поддержку. | Готово к работе в Azure.
Windows Server 2003 и Windows Server 2003 R2 | Эти операционные системы прошли свои даты окончания поддержки и нуждаются в [настраиваемом соглашении о поддержке (CSA)](/troubleshoot/azure/virtual-machines/server-software-support) для поддержки в Azure. | Условно готово для Azure. Рекомендуется обновить операционную систему перед миграцией в Azure.
Windows 2000, Windows 98, Windows 95, Windows NT, Windows 3,1 и MS-DOS | Эти операционные системы прошли свои даты окончания поддержки. Сервер может запускаться в Azure, но Azure не поддерживает ОС. | Условно готово для Azure. Рекомендуется обновить операционную систему перед миграцией в Azure.
Windows 7, Windows 8 и Windows 10 | Azure обеспечивает поддержку только для [подписки Visual Studio.](../virtual-machines/windows/client-images.md) | Условно готово для Azure.
Windows 10 Pro | Azure обеспечивает поддержку с [правами на мультитенантное размещение.](../virtual-machines/windows/windows-desktop-multitenant-hosting-deployment.md) | Условно готово для Azure.
Windows Vista и Windows XP Professional | Эти операционные системы прошли свои даты окончания поддержки. Сервер может запускаться в Azure, но Azure не поддерживает ОС. | Условно готово для Azure. Рекомендуется обновить операционную систему перед миграцией в Azure.
Linux | См. [операционные системы Linux](../virtual-machines/linux/endorsed-distros.md) , одобренные Azure. Другие операционные системы Linux могут запускаться в Azure. Но рекомендуется обновить ОПЕРАЦИОНную систему до рекомендованной версии перед миграцией в Azure. | Устройство готово к работе в Azure, если использует рекомендуемую версию.<br/><br/>Условно готово, если версия не одобрена.
Другие операционные системы, такие как Oracle Solaris, Apple macOS и FreeBSD | Azure не рекомендует эти операционные системы. Сервер может запускаться в Azure, но Azure не поддерживает ОС. | Условно готово для Azure. Перед миграцией в Azure рекомендуется установить поддерживаемую ОС.  
ОС, указанные как **Другое** в vCenter Server | В этом случае служба "миграция Azure" не может опознать ОС. | Готовность неизвестна. Убедитесь, что Azure поддерживает операционную систему, работающую на виртуальной машине.
32-разрядные операционные системы | Сервер может запускаться в Azure, но Azure может не предоставлять полную поддержку. | Условно готово для Azure. Перед миграцией в Azure рассмотрите возможность обновления до 64-разрядной ОС.

## <a name="calculating-sizing"></a>Вычисление размера

После того как сервер помечается как готовый для Azure, оценка выдает рекомендации по выбору размера в оценке виртуальных машин Azure. Эти рекомендации указывают на виртуальную машину Azure и номер SKU диска. Вычисление размеров зависит от того, используется ли как-то локальное изменение размера или размера на основе производительности.

### <a name="calculate-sizing-as-is-on-premises"></a>Вычислить размер (как в локальной системе)

 Если вы используете "как есть" локальное изменение размера, оценка не учитывает журнал производительности виртуальных машин и дисков в оценке виртуальных машин Azure.

- **Вычисление размера**. Оценка выделяет номер SKU виртуальной машины Azure на основе размера, выделенного в локальной среде.
- **Размер хранилища и диска**. Оценка просматривает тип хранилища, указанный в свойствах оценки, и рекомендует соответствующий тип диска. Возможные типы хранилища: HDD (цен. категория "Стандартный"), SSD (цен. категория "Стандартный") и Premium. Тип хранилища по умолчанию — Premium.
- **Изменение размера сети**. при оценке сетевой адаптер рассматривается на локальном сервере.

### <a name="calculate-sizing-performance-based"></a>Вычисление размера (на основе производительности)

Если для оценки виртуальных машин Azure используется размер на основе производительности, то оценка выдает следующие рекомендации:

- Оценка оценивает журнал производительности сервера, определяя размер виртуальной машины и тип диска в Azure.
- При импорте серверов с помощью CSV-файла используются указанные значения. Этот метод особенно полезен, если вы переделили локальный сервер, заданный объем невелик и вы хотите подобрать правильный размер виртуальную машину Azure, чтобы сэкономить затраты.
- Если вы не хотите использовать данные о производительности, сбросьте критерии изменения размера в "как есть локально", как описано в предыдущем разделе.

#### <a name="calculate-storage-sizing"></a>Вычисление размера хранилища

При определении размера хранилища в оценке виртуальных машин Azure служба "миграция Azure" пытается сопоставлять каждый диск, подключенный к серверу, к диску Azure. Изменение размера работает следующим образом:

1. Оценка добавляет операции ввода-вывода для чтения и записи диска, чтобы получить общее число операций ввода/вывода в секунду. Аналогичным образом для получения общей пропускной способности каждого диска добавляются значения пропускной способности чтения и записи. В случае оценок на основе импорта можно указать общее число операций ввода-вывода в секунду, общую пропускную способность и общее число. дисков в импортированном файле без указания отдельных параметров диска. В этом случае размер отдельного диска будет пропущен, а предоставляемые данные будут использоваться непосредственно для расчета размера и выбрать соответствующий номер SKU виртуальной машины.

1. Если тип хранилища указан как автоматический, выбранный тип будет основываться на действующих значениях операций ввода-вывода и пропускной способности. Оценка определяет, следует ли сопоставлять диск с диском HDD (цен. категория "Стандартный"), SSD (цен. категория "Стандартный") или Premium в Azure. Если для типа хранилища задан один из этих типов дисков, то оценка пытается найти номер SKU диска в выбранном типе хранилища.
1. Диски выбираются следующим образом:
    - Если средство оценки не может найти диск с необходимыми операциями ввода-вывода и пропускной способностью, он помечает сервер как неподходящий для Azure.
    - Если средство оценки находит набор подходящих дисков, он выбирает диски, которые поддерживают расположение, указанное в параметрах оценки.
    - При наличии нескольких подходящих дисков оценка выбирает диск с наименьшей стоимостью.
    - Если данные о производительности для любого диска недоступны, размер диска конфигурации используется для поиска SSD (цен. категория "Стандартный") диска в Azure.

#### <a name="calculate-network-sizing"></a>Вычисление размера сети

При оценке виртуальных машин Azure Оценка пытается найти виртуальную машину Azure, которая поддерживает номер и необходимую производительность сетевых адаптеров, подключенных к локальному серверу.

- Чтобы получить эффективную производительность сети локального сервера, в ходе оценки происходит вычисление скорости передачи данных из сервера (сети) для всех сетевых адаптеров. Затем он применяет коэффициент отдыха. Он использует полученное значение для поиска виртуальной машины Azure, которая может поддерживать необходимую производительность сети.
- Помимо производительности сети, при оценке также учитывается, может ли виртуальная машина Azure поддерживать необходимое количество сетевых адаптеров.
- Если данные о производительности сети недоступны, при оценке для размера виртуальной машины учитывается только число сетевых адаптеров.

#### <a name="calculate-compute-sizing"></a>Вычисление размера вычислений

После вычисления требований к хранению и сети в процессе оценки учитываются требования к ЦП и ОЗУ, чтобы найти подходящий размер виртуальной машины в Azure.

- Служба "миграция Azure" рассматривает эффективные используемые ядра и оперативную память, чтобы найти подходящий размер виртуальной машины Azure.
- Если подходящий размер не найден, сервер помечается как неподходящий для Azure.
- Если найден подходящий размер, служба "Миграция Azure" применяет результаты вычисления параметров хранилища и сети. Затем он применяет расположение и параметры уровня ценообразования для окончательной рекомендации по размеру виртуальной машины.
- Если доступно несколько соответствующих размеров виртуальных машин, служба выбирает рекомендуемый, который обеспечивает наименьшие затраты.

## <a name="confidence-ratings-performance-based"></a>Оценка достоверности (на основе производительности)

Каждая оценка виртуальной машины Azure на основе производительности в службе "миграция Azure" связана с оценкой достоверности. Оценка находится в диапазоне от одной (самой низкой) до пяти звезд. Оценка достоверности помогает оценить надежность рекомендаций, предоставляемых службой Azure Migrate.

- Оценка достоверности назначается оценке. Оценка основана на доступности точек данных, необходимых для расчета оценки.
- Для определения размера на основе производительности необходимо:
    - Данные об использовании ЦП и ОЗУ.
    - Данные дискового ввода-вывода и пропускной способности для каждого диска, подключенного к серверу.
    - Сетевой ввод-вывод для управления размером на основе производительности для каждого сетевого адаптера, подключенного к серверу.

Если любое из этих номеров использования недоступно, рекомендации по размеру могут быть ненадежными.

> [!NOTE]
> Оценка достоверности не назначается для серверов, оцениваемых с помощью импортированного CSV-файла. Оценки также не применяются для оценки локальной среды "как есть".

### <a name="ratings"></a>Рейтинги

В этой таблице показаны оценки достоверности, которые зависят от процента доступных точек данных:

   **Уровень доступности точек данных** | **Оценка достоверности**
   --- | ---
   0-20% | 1 звезда
   21-40% | 2 звезды
   41-60% | 3 звезды
   61-80% | 4 звезды
   81-100% | 5 звезд

### <a name="low-confidence-ratings"></a>Низкая оценка достоверности

Ниже приведены несколько причин, по которым оценка может иметь низкую степень достоверности.

- Среда не была профилирована в период времени, для которого создается оценка. Например, если вы создаете оценку с длительностью производительности в один день, необходимо подождать по крайней мере день после начала обнаружения всех точек данных для сбора.
- Функция оценки не сможет собрать данные производительности для некоторых или всех серверов за период оценки. Для оценки высокой достоверности убедитесь, что: 
    - Серверы включены на время оценки
    - Исходящие подключения к портам 443 разрешены
    - Для серверов Hyper-V включена динамическая память 
    
    Пересчитайте оценку, чтобы отразить последние изменения оценки достоверности.

- Некоторые серверы были созданы в течение времени, в течение которого была рассчитана оценка. Например, предположим, что вы создали оценку для журнала производительности за прошлый месяц, но некоторые серверы были созданы только неделю назад. В этом случае данные о производительности для новых серверов будут недоступны для всей продолжительности, а Оценка достоверности будет низкой.

> [!NOTE]
> Если оценка достоверности не превышает пяти звезд, рекомендуется подождать по крайней мере один день, чтобы устройство было пропрофилировать среду, а затем повторно вычислить оценку. В противном случае размер на основе производительности может быть ненадежным. В этом случае рекомендуется переключить оценку на локальное изменение размера.

## <a name="calculate-monthly-costs"></a>Вычисление месячных затрат

После выполнения рекомендаций по изменению размера Оценка виртуальной машины Azure в службе "миграция Azure" вычисляет затраты на вычисления и хранение после миграции.

- **Стоимость вычислений**. Служба "миграция Azure" использует рекомендованный размер виртуальной машины Azure и API выставления счетов Azure для вычисления ежемесячных затрат на сервер.

    В расчете учитывается:
    - Операционная система
    - Software Assurance
    - Зарезервированные экземпляры
    - Время доступности виртуальной машины
    - Расположение
    - Параметры валюты

    Оценка вычисляет совокупную стоимость всех серверов, чтобы вычислить общую ежемесячную стоимость вычислений.

- **Стоимость хранилища**. ежемесячные затраты на хранение на сервере рассчитываются путем статистической обработки ежемесячных затрат на все диски, подключенные к серверу.

    Оценка вычисляет общие ежемесячные затраты на хранение, суммируя затраты на хранение всех серверов. В настоящее время вычисления не учитывают предложения, указанные в параметрах оценки.

Цены отображаются в валюте, заданной в настройках оценки.

## <a name="next-steps"></a>Дальнейшие действия

[Просмотрите](best-practices-assessment.md) рекомендации для создания оценок. 

- Узнайте о выполнении оценок для серверов, работающих в среде [VMware](./tutorial-discover-vmware.md) и [Hyper-V ](./tutorial-discover-hyper-v.md) , и [физических серверах](./tutorial-discover-physical.md).
- Узнайте, как оценивать серверы, [импортированные с помощью CSV-файла](./tutorial-discover-import.md).
- Дополнительные сведения о настройке [визуализации зависимостей](concepts-dependency-visualization.md).
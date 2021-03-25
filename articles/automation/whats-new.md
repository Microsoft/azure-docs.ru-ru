---
title: Новые возможности в службе автоматизации Azure
description: Важные обновления добавляются в службу автоматизации Azure каждый месяц.
ms.subservice: ''
ms.topic: overview
author: mgoedtel
ms.author: magoedte
ms.date: 02/23/2021
ms.custom: references_regions
ms.openlocfilehash: 899249c98c3ce0fdf061b1e689182f71c120aa13
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101729405"
---
# <a name="whats-new-in-azure-automation"></a>Новые возможности в службе автоматизации Azure

Служба автоматизации Azure совершенствуется на постоянной основе. Чтобы вы оставались в курсе последних разработок, в этой статье предоставлены такие сведения:

- Последние выпуски.
- Известные проблемы
- Исправления ошибок

Эта страница обновляется ежемесячно, поэтому регулярно пересматривайте ее.

## <a name="february-2021"></a>Февраль 2021 года

### <a name="support-for-automation-and-state-configuration-declared-ga-in-japan-west"></a>Объявлен этап общей доступности для поддержки службы автоматизации и State Configuration в регионе "Западная Япония"

**Тип.** Новая функция.

Учетная запись службы автоматизации и State Configuration доступны в регионе "Западная Япония". Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-in-japan-west-region/).

### <a name="introduced-custom-azure-policy-compliance-to-enforce-runbook-execution-on-hybrid-worker"></a>Представлена настраиваемая функция соответствия Политики Azure для принудительного выполнения runbook в гибридной рабочей роли

**Тип.** Новая функция.

Вы можете использовать новое правило соответствия Политики Azure, чтобы разрешить создание заданий, веб-перехватчиков и расписаний заданий только в группах гибридных рабочих ролей.

### <a name="update-management-availability-in-east-us-france-central-and-north-europe-regions"></a>Доступность Управления обновлениями в регионах "Восточная часть США", "Центральная Франция" и "Северная Европа"

**Тип.** Новая функция.

Функция Управления обновлениями службы автоматизации доступна в регионах "Восточная часть США", "Центральная Франция" и "Северная Европа". Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые сопоставления регионов](how-to/region-mappings.md).

## <a name="january-2021"></a>Январь 2021 г.

### <a name="support-for-automation-and-state-configuration-declared-ga-in-switzerland-west"></a>Объявлен этап общей доступности для поддержки службы автоматизации и State Configuration в регионе "Западная Швейцария"

**Тип.** Новая функция.

Учетная запись службы автоматизации и State Configuration доступны в регионе "Западная Швейцария". Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-in-switzerland-west-region/).

### <a name="added-python-3-script-to-import-module-with-multiple-dependencies"></a>Добавлен сценарий на Python 3 для импорта модуля с несколькими зависимостями

**Тип.** Новая функция.

Сценарий доступен для скачивания из нашего [репозитория GitHub](https://github.com/azureautomation/runbooks/blob/master/Utility/Python/import_py3package_from_pypi.py). 
 
### <a name="hybrid-runbook-worker-role-support-for-centos-8xrhel-8xsles-15"></a>Поддержка гибридной рабочей роли Runbook для Centos 8.x/RHEL 8.x/SLES 15

**Тип.** Новая функция

Гибридная рабочая роль Runbook поддерживает дистрибутивы CentOS 8.x, REHL 8.x и SLES 15 только для автоматизации процессов в гибридных рабочих ролях Runbook.  Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые операционные системы](automation-linux-hrw-install.md#supported-linux-operating-systems).

### <a name="update-management--change-tracking-availability-in-australia-east-east-asia-west-us--central-us-regions"></a>Доступность Управления обновлениями и Отслеживания изменений в регионах "Восточная Австралия", "Восточная Азия", "Западная часть США" и "Центральная часть США"

**Тип.** Новая функция.

Учетная запись автоматизации, Отслеживание изменений и инвентаризация и Управление обновлениями доступны в регионах "Восточная Австралия", "Восточная Азия", "Западная часть США" и "Центральная часть США". 

### <a name="introduced-public-preview-of-python-3-runbooks-in-us-government-cloud"></a>Представлена общедоступная предварительная версия модулей runbook для Python 3 в облаке для государственных организаций США

**Тип.** Новая функция службы автоматизации Azure предоставляет общедоступную предварительную версию поддержки облака Python 3 и гибридного выполнения runbook в регионах облака для государственных организаций США.  Дополнительные сведения см. в [объявлении о выпуске](https://azure.microsoft.com/updates/azure-automation-python-3-public-preview/).

### <a name="azure-automation-runbooks-moved-from-technet-script-center-to-github"></a>Последовательности runbook службы автоматизации Azure перенесены из центра скриптов TechNet в GitHub

**Тип.** Планирование изменений.

Центр скриптов TechNet больше не будет использоваться, и все последовательности runbook, размещенные в коллекции Runbook, были перенесены в нашу [организацию службы автоматизации на GitHub](https://github.com/azureautomation). Дополнительные сведения см. в статье, посвященной [переносу последовательностей runbook службы автоматизации Azure в GitHub](https://techcommunity.microsoft.com/t5/azure-governance-and-management/azure-automation-runbooks-moving-to-github/ba-p/2039337).

## <a name="december-2020"></a>Декабрь 2020 г.

### <a name="azure-automation-and-update-management-private-link-ga"></a>Общая доступность для службы автоматизации Azure и Приватного канала для Управления обновлениями

**Тип.** Новая функция.

Поддержка службы автоматизации Azure и Управления обновлениями предоставляется в общем доступе для глобальных и правительственных облаков Azure. В службе автоматизации Azure реализована поддержка Приватного канала для безопасного выполнения последовательности runbook в гибридной рабочей роли. При этом используется Управление обновлениями для установки исправлений на компьютерах, последовательность runbook вызывается через веб-перехватчик, а служба State Configuration обеспечивает соответствие. Дополнительные сведения см. в статье [Поддержка Приватного канала для службы автоматизации Azure](https://azure.microsoft.com/updates/azure-automation-private-link).

### <a name="azure-automation-classified-as-grade-c-certified-on-accessibility"></a>Служба автоматизации Azure получила категорию C по специальным возможностям

**Тип.** Новая функция.

Специальные возможности продуктов Майкрософт помогают агентствам удовлетворять соответствующие глобальные требования. На странице [объявления в блоге](https://cloudblogs.microsoft.com/industry-blog/government/2018/09/11/accessibility-conformance-reports/) выполните поиск по строке **Azure Automation**, чтобы прочитать отчет о соответствии специальных возможностей для службы автоматизации.

### <a name="support-for-automation-and-state-configuration-ga-in-uae-north"></a>Общая доступность поддержки службы автоматизации и State Configuration в регионе "Северная часть ОАЭ"

**Тип.** Новая функция.

Учетная запись службы автоматизации и State Configuration доступны в регионе "Северная часть ОАЭ". Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-in-uae-north-region/).

### <a name="support-for-automation-and-state-configuration-ga-in-germany-west-central"></a>Общая доступность поддержки службы автоматизации и State Configuration в регионе "Центрально-Западная Германия"

**Тип.** Новая функция.

Учетная запись службы автоматизации и State Configuration доступны в регионе "Центрально-Западная Германия". Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-in-germany-west-central-region/).

### <a name="dsc-support-for-oracle-6-and-7"></a>Поддержка DSC для Oracle 6 и 7

**Тип.** Новая функция.

Вы можете управлять компьютерами с Oracle Linux 6 и 7 с помощью State Configuration службы автоматизации. Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые дистрибутивы Linux](https://github.com/Azure/azure-linux-extensions/tree/master/DSC#4-supported-linux-distributions).

### <a name="public-preview-for-python3-runbooks-in-automation"></a>Общедоступная предварительная версия для последовательностей runbook Python3 в службе автоматизации

**Тип.** Новая функция.

Служба автоматизации Azure теперь поддерживает выполнение облачных и гибридных последовательностей runbook Python 3 в общедоступной предварительной версии во всех регионах в глобальном облаке Azure. Дополнительные сведения см. в [объявлении]((https://azure.microsoft.com/updates/azure-automation-python-3-public-preview/).

## <a name="november-2020"></a>Ноябрь 2020 г.

### <a name="dsc-support-for-ubuntu-1804"></a>Поддержка DSC для Ubuntu 18.04

**Тип.** Новая функция.

Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые дистрибутивы Linux](https://github.com/Azure/azure-linux-extensions/tree/master/DSC#4-supported-linux-distributions).

## <a name="october-2020"></a>Октябрь 2020 г.

### <a name="support-for-automation-and-state-configuration-ga-in-switzerland-north"></a>Общая доступность поддержки службы автоматизации и State Configuration в регионе "Северная Швейцария"

**Тип.** Новая функция.

Учетная запись службы автоматизации и State Configuration доступны в регионе "Северная Швейцария". Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-in-switzerland-north-region/).

### <a name="support-for-automation-and-state-configuration-ga-in-brazil-south-east"></a>Общая доступность поддержки службы автоматизации и State Configuration в регионе "Юго-Восточная Бразилия"

**Тип.** Новая функция.

Учетная запись службы автоматизации и State Configuration доступны в регионе "Юго-Восточная Бразилия". Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-in-brazil-southeast-region/).

### <a name="update-management-availability-in-south-central-us"></a>Доступность Управления обновлениями в регионе "Центрально-южная часть США"

**Тип.** Новая функция.

Сопоставление регионов службы автоматизации Azure обновлено для поддержки функции "Управление обновлениями" в регионе "Центрально-южная часть США". Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые сопоставления регионов](how-to/region-mappings.md#supported-mappings).

## <a name="september-2020"></a>Сентябрь 2020 г.

### <a name="startstop-vms-during-off-hours-runbooks-updated-to-use-azure-az-modules"></a>Последовательности runbook для Запуска и остановки виртуальных машин в нерабочее время обновлены для использования модулей Azure Az

**Тип.** Новая функция.

Последовательности runbook для запуска и остановки виртуальных машин были обновлены для использования модулей Az вместо модулей Azure Resource Manager. Общие сведения об обновлениях документации, отражающих это изменение, см. в статье [Запуск и остановка виртуальных машин в нерабочее время](automation-solution-vm-management.md).

## <a name="august-2020"></a>Август 2020 г.

### <a name="published-the-dsc-extension-to-support-azure-arc"></a>Опубликовано расширение DSC для поддержки Azure Arc

**Тип.** Новая функция.

Используйте State Configuration службы автоматизации Azure для централизованного хранения конфигураций и поддержки требуемого состояния гибридных подключенных компьютеров, реализуемого с помощью расширения виртуальной машины DSC на серверах с поддержкой Azure Arc. Дополнительные сведения см. в статье [Общие сведения о расширениях виртуальной машины на серверах с поддержкой Arc](../azure-arc/servers/manage-vm-extensions.md).

### <a name="july-2020"></a>Июль 2020 г.

### <a name="introduced-public-preview-of-private-link-support-in-automation"></a>Представлена общедоступная предварительная версия поддержки Приватного канала в службе автоматизации

**Тип.** Новая функция.

С помощью Приватного канала Azure вы можете безопасно подключать виртуальные сети к службе автоматизации Azure с использованием частных конечных точек. Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/public-preview-private-link-azure-automation-is-now-available/).

### <a name="hybrid-runbook-worker-support-for-windows-server-2008-r2"></a>Поддержка гибридных рабочих ролей runbook для Windows Server 2008 R2

**Тип.** Новая функция.

Гибридная рабочая роль runbook службы автоматизации поддерживает операционную систему Windows Server 2008 R2. Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые операционные системы](automation-windows-hrw-install.md#supported-windows-operating-system).

### <a name="update-management-support-for-windows-server-2008-r2"></a>Поддержка Управления обновлениями для Windows Server 2008 R2

**Тип.** Новая функция.

Управление обновлениями поддерживает оценку и установку исправлений для операционной системы Windows Server 2008 R2. Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые операционные системы](update-management/overview.md#clients).

### <a name="automation-diagnostic-logs-schema-update"></a>Обновление схемы журналов диагностики для службы автоматизации

**Тип.** Новая функция.

Схема данных журналов для службы автоматизации Azure в службе Log Analytics была изменена. Дополнительные сведения см. в разделе [Пересылка данных задания службы автоматизации Azure в журналы Azure Monitor](automation-manage-send-joblogs-log-analytics.md#filter-job-status-output-converted-into-a-json-object).

### <a name="azure-lighthouse-supports-automation-update-management"></a>Azure Lighthouse поддерживает Управление обновлениями в службе автоматизации

**Тип.** Новая функция.

Azure Lighthouse обеспечивает управление делегированными ресурсами с помощью Управления обновлениями для поставщиков служб и клиентов. Дополнительные сведения см. [здесь](https://azure.microsoft.com/blog/how-azure-lighthouse-enables-management-at-scale-for-service-providers/).

## <a name="june-2020"></a>Июнь 2020 г.

### <a name="automation-and-update-management-availability-in-the-us-gov-arizona-region"></a>Доступность службы автоматизации и Управления обновлениями в регионе US Gov (Аризона)

**Тип.** Новая функция.

Учетная запись службы автоматизации и Управление обновлениями уже доступны в регионе US Gov (Аризона). Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-generally-available-in-usgov-arizona-region/).

### <a name="hybrid-runbook-worker-onboarding-script-updated-to-use-az-modules"></a>Скрипт регистрации гибридной рабочей роли runbook обновлен для использования модулей Az

**Тип.** Новая функция.

Рабочая роль runbook New-OnPremiseHybridWorker обновлена для поддержки модулей Az. Дополнительные сведения см. в пакете в [галерее PowerShell](https://www.powershellgallery.com/packages/New-OnPremiseHybridWorker/1.7).

### <a name="update-management-availability-in-china-east-2"></a>Доступность Управления обновлениями в регионе "Восточный Китай 2"

**Тип.** Новая функция.

Сопоставление регионов службы автоматизации Azure обновлено для поддержки функции "Управление обновлениями" в регионе "Восточный Китай 2". Обновления документации, отражающие это изменение, см. в разделе [Поддерживаемые сопоставления регионов](how-to/region-mappings.md#supported-mappings).

## <a name="may-2020"></a>Май 2020 г.

### <a name="updated-automation-service-dns-records-from-region-specific-to-automation-account-specific-urls"></a>Записи DNS службы автоматизации обновлены с изменением URL-адресов с привязкой к региону на URL-адреса с привязкой к службе автоматизации

**Тип.** Новая функция.

Записи DNS службы автоматизации Azure обновлены для поддержки приватных каналов. Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-updateddns-records/).

### <a name="added-capability-to-keep-automation-runbooks--dsc-scripts-encrypted-by-default"></a>Добавлена возможность сохранения последовательностей runbook службы автоматизации и скриптов DSC зашифрованными по умолчанию

**Тип.** Новая функция.

В дополнение к повышению безопасности активов, последовательности runbook и скрипты DSC также шифруются для расширенной защиты службы автоматизации Azure.

## <a name="april-2020"></a>Апрель 2020 г.

### <a name="retirement-of-the-automation-watcher-task"></a>Прекращение использования задачи Наблюдателя за автоматизацией

**Тип.** Планирование изменений.

Azure Logic Apps теперь является рекомендуемым и поддерживаемым способом слежения за событиями, планирования повторяющихся задач и запуска действий. Функциональность задачи Наблюдателя больше не будет поддерживаться и улучшаться. Дополнительные сведения см. в статье [Планирование и запуск повторяющихся автоматизированных задач с помощью Logic Apps](../logic-apps/concepts-schedule-automated-recurring-tasks-workflows.md).

## <a name="march-2020"></a>Март 2020 г.

### <a name="support-for-impact-level-5-il5-compute-isolation-in-azure-commercial-and-government-cloud"></a>Поддержка изоляции вычислений с уровнем влияния 5 (IL5) в коммерческих и правительственных облаках Azure

**Тип**.

Гибридную рабочую роль runbook службы автоматизации Azure можно использовать на платформе "Azure для государственных организаций", чтобы обеспечить поддержку рабочих нагрузок с уровнем влияния 5. Дополнительные сведения см. в [документации](automation-hybrid-runbook-worker.md#support-for-impact-level-5-il5).

## <a name="february-2020"></a>Февраль 2020 г.

### <a name="introduced-support-for-azure-virtual-network-service-tags"></a>Добавлена поддержка тегов служб виртуальной сети Azure

**Тип.** Новая функция.

Поддержка тегов служб в службе автоматизации позволяет разрешать или запрещать трафик для службы автоматизации в ряде сценариев. Дополнительные сведения см. в [документации](automation-hybrid-runbook-worker.md#service-tags).

### <a name="enable-tls-12-support-for-azure-automation-service"></a>Включение поддержки TLS 1.2 для службы автоматизации Azure

**Тип.** Планирование изменений.

Служба автоматизации Azure полностью поддерживает TLS 1.2 и все вызовы клиентов (через веб-перехватчики, узлы DSC и гибридную рабочую роль). В TLS 1.1 и TLS 1.0 по-прежнему поддерживается обратная совместимость с более старыми клиентами до тех пор, пока клиенты не будут стандартизованы и полностью адаптированы для использования TLS 1.2.

## <a name="january-2020"></a>Январь 2020 г.

### <a name="introduced-public-preview-of-customer-managed-keys-for-azure-automation"></a>Представлена общедоступная предварительная версия управляемых клиентом ключей для службы автоматизации Azure.

**Тип.** Новая функция.

Клиенты могут управлять шифрованием активов службы автоматизации Azure и защищать их с помощью собственных управляемых ключей. Дополнительные сведения см. в разделе [Использование управляемых клиентом ключей](automation-secure-asset-encryption.md#use-of-customer-managed-keys-for-an-automation-account).

### <a name="retirement-of-azure-service-management-asm-rest-apis-for-azure-automation"></a>Прекращение использования REST API Управления службами Azure (ASM) для службы автоматизации Azure

**Тип:** прекращение использования

Использование REST API Управления службами Azure (ASM) для службы автоматизации Azure будет прекращено, а их поддержка будет прекращена после 30 января 2020 г. Дополнительные сведения см. в [объявлении](https://azure.microsoft.com/updates/azure-automation-service-management-rest-apis-are-being-retired-april-30-2019/).

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите внести свой вклад в разработку документации по службе автоматизации Azure, ознакомьтесь с [руководством для разработчиков документации](/contribute/).

---
title: Настройка Центра обновления Windows для Управления обновлениями службы автоматизации Azure
description: В этой статье описывается, как настроить Центр обновления Windows для работы с Управлением обновлениями службы автоматизации Azure.
services: automation
ms.subservice: update-management
ms.date: 05/04/2020
ms.topic: conceptual
ms.openlocfilehash: a1f95ca856223628974a9519b7c4811bde43965e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92222424"
---
# <a name="configure-windows-update-settings-for-azure-automation-update-management"></a>Настройка Центра обновления Windows для Управления обновлениями службы автоматизации Azure

Управление обновлениями службы автоматизации Azure использует [клиент Центра обновления Windows](/windows/deployment/update/windows-update-overview) для загрузки и установки обновлений Windows. Существуют определенные параметры, которые клиент Центра обновления Windows использует при подключении к Windows Server Update Services (WSUS) или Центру обновления Windows. Большинство их этих параметров можно контролировать с помощью следующих средств:

- Редактор локальных групповых политик
- Групповая политика
- PowerShell
- Прямое редактирование реестра

Управление обновлениями учитывает многие параметры, заданные для управления клиентом Центра обновления Windows. Если вы настроите параметры для установки обновлений, отличающихся от Windows, Управление обновлениями будет также обрабатывать такие обновления. Если вы настроите скачивание обновлений перед развертыванием обновлений, развертывание обновлений будет выполняться быстрее, эффективнее и с меньшей вероятностью превышения периода обслуживания.

Дополнительные рекомендации по настройке WSUS в подписке Azure и безопасному обновлению виртуальных машин Windows см. в статье [Планирование развертывания для обновления Виртуальных машин Windows в Azure](/azure/architecture/example-scenario/wsus/).

## <a name="pre-download-updates"></a>Предварительная загрузка обновлений

Чтобы настроить автоматическую загрузку обновлений без автоматической установки, можно настроить групповую политику, которая устанавливает для параметра [Автоматическое обновление](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates##configure-automatic-updates) значение 3. Этот вариант означает, что необходимые обновления скачиваются в фоновом режиме, и вы получите уведомление, когда они будут готовы к установке. Это позволяет Управлению обновлениями контролировать расписание обновлений, но скачивать данные не в период обслуживания. Такое поведение предотвращает появление ошибок `Maintenance window exceeded` в Управлении обновлениями.

Чтобы включить этот параметр с помощью PowerShell, сделайте следующее.

```powershell
$WUSettings = (New-Object -com "Microsoft.Update.AutoUpdate").Settings
$WUSettings.NotificationLevel = 3
$WUSettings.Save()
```

## <a name="configure-reboot-settings"></a>Настройка параметров перезагрузки

Применение ключей реестра, перечисленных в статьях [Управление дополнительными параметрами Центра обновления Windows](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry) и [Управление перезапусками устройства после обновлений](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart) может привести к перезагрузке компьютеров, даже если выбран вариант **Никогда не перезагружать** в параметрах **Развертывание обновлений**. Укажите для этих ключей реестра значения, которые лучше подходят в вашей среде.

## <a name="enable-updates-for-other-microsoft-products"></a>Включение обновлений для других продуктов Майкрософт

По умолчанию клиент Центра обновления Windows предоставляет обновления только для ОС Windows. Если вы включите параметр **При обновлении Windows предоставить обновления для других продуктов Майкрософт**, вы будете получать обновления и для других продуктов, в том числе исправления безопасности для SQL Server и другого программного обеспечения Майкрософт. Этот параметр можно настроить, если вы уже скачали и скопировали последние [файлы административных шаблонов](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra), которые доступны для Windows 2016 и более поздних версий.

Если у вас есть компьютеры под управлением Windows Server 2012 R2, вы не сможете настроить этот параметр в групповой политике. На таких компьютерах выполните следующую команду PowerShell:

```powershell
$ServiceManager = (New-Object -com "Microsoft.Update.ServiceManager")
$ServiceManager.Services
$ServiceID = "7971f918-a847-4430-9279-4a52d1efe18d"
$ServiceManager.AddService2($ServiceId,7,"")
```

## <a name="make-wsus-configuration-settings"></a>Настройки конфигурации WSUS

Управление обновлениями поддерживает параметры WSUS. Вы можете указать источники для сканирования и скачивания обновлений с помощью инструкций, указанных в статье [Управление дополнительными параметрами Центра обновления Windows](/windows/deployment/update/waas-wu-settings#specify-intranet-microsoft-update-service-location). По умолчанию клиент Центра обновления Windows скачивает обновления только из Центра обновления Windows. Если вы укажете сервер WSUS в качестве источника для своих компьютеров, развертывание будет завершаться сбоем, если обновления не утверждены в WSUS. 

Чтобы компьютеры использовали только внутреннюю службу обновления, настройте параметр [Не подключаться к расположениям Центра обновления Windows в Интернете](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates#do-not-connect-to-any-windows-update-internet-locations).

## <a name="next-steps"></a>Дальнейшие действия

Запланируйте развертывание обновлений, следуя инструкциям в статье [Управление обновлениями и исправлениями для виртуальных машин](manage-updates-for-vm.md).

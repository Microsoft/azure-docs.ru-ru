---
title: Интеграция журналов с ArcSight с помощью Azure Monitor | Документация Майкрософт
description: Узнайте, как интегрировать журналы Azure Active Directory с ArcSight с помощью Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: b37bef0d-982e-4e28-86b2-6c61ca524ae1
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/19/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: fc7bcab04da005fd0d46d18e7b708dcb1c9d58e5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89230523"
---
# <a name="integrate-azure-active-directory-logs-with-arcsight-using-azure-monitor"></a>Интеграция журналов Azure Active Directory с ArcSight с помощью Azure Monitor

[Micro Focus ArcSight](https://software.microfocus.com/products/siem-security-information-event-management/overview) — это решение SIEM (управление событиями и информацией в сфере безопасности) помогает обнаруживать угрозы безопасности на любой платформе и реагировать на них. Теперь вы можете направлять журналы Azure Active Directory (Azure AD) в ArcSight с помощью Azure Monitor и соединителя ArcSight для Azure AD. Эта функция позволяет отслеживать нарушения безопасности в клиенте с помощью ArcSight.  

В этой статье вы узнаете, как перенаправлять журналы Azure AD в ArcSight с помощью Azure Monitor. 

## <a name="prerequisites"></a>Предварительные условия

Для использования этой функции необходимо иметь следующее.
* Концентратор событий Azure, содержащий журналы действий Azure AD. Узнайте, как [настроить потоковую передачу журналов действий в концентратор событий](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 
* Настроенный экземпляр ArcSight Syslog NG Daemon SmartConnector (SmartConnector) или ArcSight Load Balancer. Если события отправляются в ArcSight Load Balancer, то он отправляет их в SmartConnector.

Скачайте и откройте [руководство по настройке ArcSight SmartConnector для концентратора событий Azure Monitor](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292). Это руководство содержит шаги, необходимые для установки и настройки ArcSight SmartConnector для Azure Monitor. 

## <a name="integrate-azure-ad-logs-with-arcsight"></a>Интеграция журналов Azure AD в ArcSight

1. Сначала выполните действия в разделе **предварительных требований** руководства по настройке. В этом разделе содержатся следующие шаги:
    * Установите разрешения пользователя в Azure, чтобы обеспечить наличие пользователя с ролью **Владелец** для развертывания и настройки соединителя.
    * Откройте порты на сервере с Syslog NG Daemon SmartConnector, чтобы к нему можно было получить доступ из Azure. 
    * Развертывание запускает сценарий Windows PowerShell, поэтому необходимо обеспечить в PowerShell запуск сценариев на компьютере, на котором вы хотите развернуть соединитель.

2. Выполните действия, описанные в разделе **Развертывание соединителя** руководства по настройке, чтобы развернуть соединитель. В этом разделе рассматриваются способы скачивания и извлечения соединителя, настройки свойств приложения и запуска скрипта развертывания из извлеченной папки. 

3. Следуйте инструкциям в разделе **Проверка развертывания в Azure**, чтобы убедиться, что соединитель настроен и работает правильно. Проверьте выполнение следующих условий.
    * Необходимые функции Azure созданы в подписке Azure.
    * Журналы Azure AD в потоковом режиме передаются в правильное местоположение. 
    * Параметры приложения из развертывания сохраняются в параметрах приложения в приложениях-функциях Azure. 
    * Группа ресурсов для ArcSight создается в Azure с приложением Azure AD для соединителя ArcSight и учетных записей хранения, содержащих сопоставленные файлы в формате CEF.

4. Наконец, выполните действия после развертывания, указанные в разделе **Настройка после развертывания** в руководстве по настройке. В этом разделе объясняется, как выполнить дополнительную настройку, если вы используете план службы приложений, чтобы предотвратить неактивность приложений-функций по истечении времени ожидания, настроить потоковую передачу журналов ресурсов из концентратора событий и обновить сертификат хранилища Смартконнектор для управляющей программы SysLog NG, чтобы связать его с созданной учетной записью хранения.

5. В руководстве по настройке также объясняется, как настроить свойства соединителя в Azure, а также обновить и удалить соединитель. Кроме того, имеется раздел о производительности, в котором говорится о переходе к [плану потребления Azure](https://azure.microsoft.com/pricing/details/functions) и настройке ArcSight Load Balancer, если объем нагрузки событий больше, чем тот, с которым может справиться Syslog NG Daemon SmartConnector.

## <a name="next-steps"></a>Дальнейшие действия

[Руководство по настройке ArcSight SmartConnector для концентратора событий Azure Monitor](https://community.microfocus.com/t5/ArcSight-Connectors/SmartConnector-for-Microsoft-Azure-Monitor-Event-Hub/ta-p/1671292)
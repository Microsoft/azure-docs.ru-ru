---
title: Azure AD Connect Health. Оповещение "Данные службы работоспособности неактуальны" | Документация Майкрософт
description: В этом документе описана причина отправки оповещения "Данные службы работоспособности неактуальны" и действия по устранению этой неполадки.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/26/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 00518eb91e57efaacb7abc63b6ad4531619be2ce
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98012875"
---
# <a name="health-service-data-is-not-up-to-date-alert"></a>Оповещение "Данные службы работоспособности неактуальны"

## <a name="overview"></a>Обзор

Агенты на локальных компьютерах, которые Azure AD Connect Health отслеживают периодическую передачу данных в службу Azure AD Connect Health. Если служба не получает данные от агента, сведения, представленные порталом, будут устаревшими. Чтобы выделить ошибку, служба выдаст **данные службы работоспособности — это не актуальное** оповещение. Это предупреждение создается, когда служба не получала полные данные за последние два часа.  

- Предупреждение о состоянии **предупреждения** срабатывает, если служба работоспособности получил только **частичные** типы данных, отправленные с сервера за последние два часа. Предупреждение о состоянии предупреждения не запускает уведомления по электронной почте для настроенных получателей. 
- Предупреждение о состоянии **ошибки** срабатывает, если служба работоспособности не получал ни одного типа данных с сервера за последние два часа. Оповещение о состоянии ошибки активирует уведомления по электронной почте для настроенных получателей.

Служба получает данные от агентов, работающих на локальных компьютерах, в зависимости от типа службы. В следующей таблице перечислены агенты, которые выполняются на компьютере, что они делают, и типы данных, создаваемые службой. В некоторых случаях в процессе участвует несколько служб, поэтому любой из них может быть определенным. 

## <a name="understanding-the-alert"></a>Основные сведения о предупреждении

В колонке **сведения о предупреждении** отображается время возникновения и последнего обнаружения предупреждения. Фоновый процесс, выполняемый каждые два часа, создает и повторно вычисляет предупреждение. В следующем примере исходное оповещение произошло в 03/10 в 9:59 AM. Предупреждение по-прежнему существовало на 03/12 в 10:00 AM после повторной оценки предупреждения. Колонка также содержит сведения о времени, когда служба работоспособности последний раз получал определенный тип данных. 
 
 ![Сведения об оповещении Azure AD Connect Health](./media/how-to-connect-health-data-freshness/data-freshness-details.png)
 
Следующая таблица сопоставляет типы служб с соответствующими обязательными типами данных:

| Service type (Тип службы) | Агент (имя службы Windows) | Назначение | Тип данных, созданный  |
| --- | --- | --- | --- |  
| Azure AD Connect (синхронизация) | Служба Sync Insights Azure AD Connect Health | Получение сведений о подключении AAD (соединители, правила синхронизации и т. д.) | — AadSyncService-SynchronizationRules <br />  — AadSyncService-Connectors <br /> — AadSyncService-GlobalConfigurations  <br />  — AadSyncService-RunProfileResults <br /> — AadSyncService-ServiceConfigurations <br /> — AadSyncService-ServiceStatus   |
|  | Служба Sync Monitoring Azure AD Connect Health | Получение счетчиков производительности, относящихся к AAD Connect, трассировки ETW, файлы | Счетчик производительности |
| AD DS | служба AD DS для Azure AD Connect Health; | Выполнение искусственных тестов, получение сведений о топологии, метаданные репликации |  -Adds-Топологинфо-JSON <br /> -Common-TestData-JSON (создание результатов теста)   | 
|  | служба наблюдения AD DS Azure AD Connect Health. | Собирайте счетчики производительности, относящиеся к ДОБАВЛЕНию, трассировки ETW, файлы | — Счетчик производительности  <br /> -Common-TestData-JSON (отправка результатов теста)  |
| AD FS | Служба диагностики AD FS Azure AD Connect Health | Выполнение искусственных тестов | TestResult (создание результатов теста) | 
| | Служба AD FS получения ценной информации из данных о работоспособности Azure AD Connect  | Получение метрик использования ADFS | Adfs-UsageMetrics |
| | Служба наблюдения AD FS Azure AD Connect Health | Получение счетчиков производительности, относящихся к ADFS, трассировок трассировки событий Windows, файлов | TestResult (отправка результатов теста) |

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок 

Шаги, необходимые для диагностики проблемы, приведены ниже. Первый — это набор основных проверок, общих для всех типов служб. 

> [!IMPORTANT] 
> Для этого оповещения соблюдается [политика хранения данных](reference-connect-health-user-privacy.md#data-retention-policy) Connect Health

* Убедитесь, что установлены последние версии агентов. Просмотрите [историю выпусков](reference-connect-health-version-history.md). 
* Убедитесь, что агенты службы Azure AD Connect Health **запущены** на компьютере. Например, Connect Health для AD FS должен состоять из трех служб.
  ![Проверка Azure AD Connect Health](./media/how-to-connect-health-agent-install/install5.png)

* Откройте [раздел требований](how-to-connect-health-agent-install.md#requirements) и убедитесь, что все они соблюдены.
* Используйте [средство проверки подключения](how-to-connect-health-agent-install.md#test-connectivity-to-azure-ad-connect-health-service) для обнаружения проблем с подключением.
* Если у вас есть прокси-сервер HTTP, выполните следующие [действия по настройке](how-to-connect-health-agent-install.md#configure-azure-ad-connect-health-agents-to-use-http-proxy). 


## <a name="next-steps"></a>Дальнейшие действия
Если какой либо из описанных выше шагов обнаружил проблему, исправьте ее и дождитесь, пока не будет устранено предупреждение. Фоновый процесс предупреждения выполняется каждые 2 часа, поэтому разрешение предупреждения займет до 2 часов. 

* [Azure AD Connect Health политики хранения данных](reference-connect-health-user-privacy.md#data-retention-policy)
* [Часто задаваемые вопросы об Azure AD Connect Health](reference-connect-health-faq.md)

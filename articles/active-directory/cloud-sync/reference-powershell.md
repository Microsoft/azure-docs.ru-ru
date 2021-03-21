---
title: Модуль Аадклаудсинктулс PowerShell для Azure AD Connect облачной синхронизации
description: В этой статье описывается, как установить агент подготовки облака Azure AD Connect.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/30/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: aa358b0c9d7747584deabe761160d3bcbcde8feb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99593186"
---
# <a name="aadcloudsynctools-powershell-module-for-azure-ad-connect-cloud-sync"></a>Модуль Аадклаудсинктулс PowerShell для Azure AD Connect облачной синхронизации

Модуль Аадклаудсинктулс предоставляет набор полезных средств, которые можно использовать для управления Azure AD Connect развертываниями облачной синхронизации.

## <a name="pre-requisites"></a>Предварительные требования
Требуются следующие предварительные требования:

- Все необходимые компоненты для этого модуля можно установить автоматически с помощью `Install-AADCloudSyncToolsPrerequisites`
- Этот модуль использует проверку подлинности MSAL, поэтому для него требуется установленный модуль MSAL.PS. Чтобы проверить, в окне PowerShell выполните процедуру `Get-module MSAL.PS -ListAvailable` . Если модуль установлен правильно, будет получен ответ. Вы можете использовать `Install-AADCloudSyncToolsPrerequisites` для установки последней версии MSAL.PS
- Несмотря на то, что модуль AzureAD PowerShell не является необходимым условием для каких-либо функций этого модуля, полезно установить его автоматически с помощью `Install-AADCloudSyncToolsPrerequisites` .
- Для установки модулей из PowerShell вручную требуется принудительное применение TLS 1,2. Чтобы обеспечить возможность установки модулей, в сеансе PowerShell установите следующие значения, прежде чем использовать
  ```
   Install-Module:
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 
  ```


## <a name="install-the-aadcloudsynctools-powershell-module"></a>Установка модуля PowerShell Аадклаудсинктулс
Чтобы установить и использовать модуль Аадклаудсинктулс, выполните следующие действия.

1. Откройте Windows PowerShell с правами администратора
2. Введите или скопируйте и вставьте следующий текст: `Import-module -Name "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Utility\AADCloudSyncTools"`
3. Нажмите клавишу ВВОД.
4. Чтобы проверить, был ли модуль импортирован, введите или скопируйте и вставьте следующий текст: `Get-module AADCloudSyncTools`
5. Теперь должны отобразиться сведения о модуле.
6. Далее, чтобы установить предварительные требования модуля Аадклаудсинктулс, выполните следующие действия. `Install-AADCloudSyncToolsPrerequisites`
7. При первом запуске будет установлен модуль Повешеллжет, если он отсутствует. Чтобы загрузить новый модуль PowershellGet, закройте окно PowerShell и откройте новый сеанс PowerShell с правами администратора. 
8. Снова Импортируйте модуль, используя шаг 3.
9. Выполните команду `Install-AADCloudSyncToolsPrerequisites` , чтобы установить модули MSAL и AzureAD
11. Все предварительные запросов секунду должны быть успешно установлены. ![ модуль установки](media/reference-powershell/install-1.png)


## <a name="aadcloudsynctools--cmdlets"></a>Командлеты Аадклаудсинктулс
### <a name="connect-aadcloudsynctools"></a>Connect-AADCloudSyncTools
Использует модуль MSAL.PS для запроса маркера для администратора Azure AD, чтобы получить доступ к Microsoft Graph 


### <a name="export-aadcloudsynctoolslogs"></a>Export-AADCloudSyncToolsLogs
Экспортирует и упаковывает все данные для устранения неполадок в сжатом файле следующим образом:
 1. Запускает подробную трассировку с помощью Start-Аадклаудсинктулсвербоселогс.  Эти журналы трассировки находятся в папке C:\ProgramData\Microsoft\Azure AD Connect Provisioning Agent\Trace.
 2. Собирает журнал трассировки в течение 3 минут.
   Можно указать другое время с параметром-ТраЦингдуратионминс или пропустить подробную трассировку с помощью-Скипвербосетраце.
 3. Останавливает детальную трассировку с помощью Stop-AADCloudSyncToolsVerboseLogs
 4. Сбор журналов Просмотр событий за последние 24 часа
 5. Сжимает все журналы агентов, подробные журналы и журналы просмотра событий в сжатом ZIP-файле в папке документов пользователя. 
 </br>Можно указать другую выходную папку с помощью-OutputPath. \<folder path\>

### <a name="get-aadcloudsynctoolsinfo"></a>Get-AADCloudSyncToolsInfo
Отображение сведений о клиенте Azure AD и состоянии внутренних переменных

### <a name="get-aadcloudsynctoolsjob"></a>Get-AADCloudSyncToolsJob
Использует Graph для получения субъектов-служб AD2AAD и возвращает сведения о задании синхронизации.
Может также вызываться с использованием конкретного идентификатора задания синхронизации в качестве параметра.

### <a name="get-aadcloudsynctoolsjobschedule"></a>Get-AADCloudSyncToolsJobSchedule
Использует Graph для получения субъектов-служб AD2AAD и возвращает расписание задания синхронизации.
Может также вызываться с использованием конкретного идентификатора задания синхронизации в качестве параметра.

### <a name="get-aadcloudsynctoolsjobschema"></a>Get-AADCloudSyncToolsJobSchema
Использует Graph для получения субъектов-служб AD2AAD и возвращает схему задания синхронизации.

### <a name="get-aadcloudsynctoolsjobscope"></a>Get-AADCloudSyncToolsJobScope
Использует Graph для получения схемы задания синхронизации для указанного идентификатора задания синхронизации и выводит все области группы фильтров.

### <a name="get-aadcloudsynctoolsjobsettings"></a>Get-AADCloudSyncToolsJobSettings
Использует Graph для получения субъектов-служб AD2AAD и возвращает параметры задания синхронизации.
Может также вызываться с использованием конкретного идентификатора задания синхронизации в качестве параметра.

### <a name="get-aadcloudsynctoolsjobstatus"></a>Get-AADCloudSyncToolsJobStatus
Использует Graph для получения субъектов-служб AD2AAD и возвращает состояние задания синхронизации.
Может также вызываться с использованием конкретного идентификатора задания синхронизации в качестве параметра.

### <a name="get-aadcloudsynctoolsserviceprincipal"></a>Get-AADCloudSyncToolsServicePrincipal
Использует Graph для получения субъектов-служб для AD2AAD и (или) Синкфабрик.
Без параметров будет возвращать только субъекты-службы AD2AAD.

### <a name="install-aadcloudsynctoolsprerequisites"></a>Install-AADCloudSyncToolsPrerequisites
Проверяет наличие модулей PowerShellGet v 2.2.4.1 или более поздней версии и Azure AD и MSAL.PS и устанавливает их, если они отсутствуют.

### <a name="invoke-aadcloudsynctoolsgraphquery"></a>Invoke-AADCloudSyncToolsGraphQuery
Вызывает веб-запрос для URI, метода и тела, указанные в качестве параметров

### <a name="repair-aadcloudsynctoolsaccount"></a>Repair-AADCloudSyncToolsAccount
Использует Azure AD PowerShell для удаления текущей учетной записи (если она есть) и сбрасывает проверку подлинности учетной записи синхронизации с помощью новой учетной записи синхронизации в Azure AD.

### <a name="restart-aadcloudsynctoolsjob"></a>Restart-AADCloudSyncToolsJob
Перезапускает полную синхронизацию.

### <a name="resume-aadcloudsynctoolsjob"></a>Resume-AADCloudSyncToolsJob
Продолжение синхронизации из предыдущего водяного знака.

### <a name="start-aadcloudsynctoolsverboselogs"></a>Start-AADCloudSyncToolsVerboseLogs
Изменяет "AADConnectProvisioningAgent.exe.config" для включения подробной трассировки и перезапуска службы Аадконнектпровисионингажент можно использовать параметр-Скипсервицерестарт для предотвращения перезапуска службы, но любые изменения конфигурации не вступят в силу.  Эти журналы трассировки находятся в папке C:\ProgramData\Microsoft\Azure AD Connect Provisioning Agent\Trace.

### <a name="stop-aadcloudsynctoolsverboselogs"></a>Stop-AADCloudSyncToolsVerboseLogs
Изменяет "AADConnectProvisioningAgent.exe.config" для отключения подробной трассировки и перезапуска службы Аадконнектпровисионингажент. Можно использовать параметр-Скипсервицерестарт, чтобы предотвратить перезапуск службы, но изменения конфигурации не вступят в силу.

### <a name="suspend-aadcloudsynctoolsjob"></a>Suspend-AADCloudSyncToolsJob
Приостанавливает синхронизацию.

## <a name="next-steps"></a>Дальнейшие действия 

- [Что собой представляет подготовка?](what-is-provisioning.md)
- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)


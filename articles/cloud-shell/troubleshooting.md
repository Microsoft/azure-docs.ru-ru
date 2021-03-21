---
title: Устранение неполадок Azure Cloud Shell | Документация Майкрософт
description: Устранение неполадок Azure Cloud Shell.
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: damaerte
ms.openlocfilehash: eea64520dd5440467c911b6de42d8c8c31fc1bde
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87543458"
---
# <a name="troubleshooting--limitations-of-azure-cloud-shell"></a>Устранение неполадок и ограничения в Azure Cloud Shell

Ниже описаны решения известных проблем в Azure Cloud Shell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="general-troubleshooting"></a>Общие действия по устранению неполадок

### <a name="error-running-azuread-cmdlets-in-powershell"></a>Ошибка при выполнении командлетов AzureAD в PowerShell

- **Сведения**: при запуске командлетов AzureAD `Get-AzureADUser` , например в Cloud Shell, может появиться сообщение об ошибке: `You must call the Connect-AzureAD cmdlet before calling any other cmdlets` . 
- **Решение**. выполните `Connect-AzureAD` командлет. Ранее Cloud Shell автоматически выполнял этот командлет во время запуска PowerShell. Чтобы ускорить время начала, командлет больше не будет выполняться автоматически. Вы можете восстановить предыдущее поведение, добавив `Connect-AzureAD` в файл $Profile в PowerShell.

### <a name="early-timeouts-in-firefox"></a>Преждевременное истечение времени ожидания в FireFox

- **Сведения.** Cloud Shell использует открытый WebSocket для передачи входящих и исходящих значений в браузер. FireFox включает предварительно заданные политики, которые могут преждевременно закрыть подключение по протоколу WebSocket, что может стать причиной преждевременного истечения времени ожидания в Cloud Shell.
- **Решение.** Откройте FireFox и перейдите в область about:config в поле URL-адреса. Введите в строку поиска network.websocket.timeout.ping.request и измените значение 0 на 10.

### <a name="disabling-cloud-shell-in-a-locked-down-network-environment"></a>Отключение Cloud Shell в заблокированной среде сети

- **Сведения**. Администраторам может потребоваться отключить доступ к Cloud Shell для своих пользователей. Cloud Shell использует доступ к `ux.console.azure.com` домену, который может быть запрещен, и останавливает любой доступ к Cloud Shell EntryPoint, включая Portal.Azure.com, Shell.Azure.com, расширение учетной записи azure Visual Studio Code и docs.Microsoft.com. В облаке для государственных организаций США точка входа имеет значение `ux.console.azure.us` ; нет соответствующего Shell.Azure.US.
- **Решение**. Ограничьте доступ к `ux.console.azure.com` `ux.console.azure.us` среде или через параметры сети. Значок Cloud Shell по-прежнему будет существовать в портал Azure, но не будет успешно подключаться к службе.

### <a name="storage-dialog---error-403-requestdisallowedbypolicy"></a>Диалоговое окно службы хранилища. Ошибка: 403 RequestDisallowedByPolicy

- **Сведения**. при создании учетной записи хранения с помощью Cloud Shell она не будет выполнена из-за назначения политики Azure администратором. Сообщение об ошибке будет содержать следующее: `The resource action 'Microsoft.Storage/storageAccounts/write' is disallowed by one or more policies.`
- **Решение**. обратитесь к администратору Azure, чтобы удалить или обновить назначение политики Azure, запрещающее создание хранилища.

### <a name="storage-dialog---error-400-disallowedoperation"></a>Диалоговое окно службы хранилища. Ошибка: 400 DisallowedOperation

- **Сведения.** Не удается создать хранилище, используя подписку Azure Active Directory.
- **Устранение.** Используйте подписку Azure, которая позволяет создавать ресурсы хранилища. Подписки Azure AD не поддерживают создание ресурсов Azure.

### <a name="terminal-output---error-failed-to-connect-terminal-websocket-cannot-be-established-press-enter-to-reconnect"></a>Выходные данные терминала. Ошибка "Failed to connect terminal: websocket cannot be established. Press `Enter` to reconnect" (Не удалось подключиться к терминалу. Невозможно установить подключение по протоколу WebSocket. Нажмите клавишу ВВОД, чтобы подключиться еще раз)
- **Сведения.** Для Cloud Shell требуется возможность установить подключение по протоколу WebSocket к инфраструктуре Cloud Shell.
- **Решение.** Убедитесь, что параметры сети настроены для разрешения отправки HTTP-запросов и запросов WebSocket к доменам по адресу *.console.azure.com.

### <a name="set-your-cloud-shell-connection-to-support-using-tls-12"></a>Настройка поддержки TLS 1.2 в подключении Cloud Shell
 - **Сведения**: чтобы определить версию протокола TLS для подключения к Cloud Shell, необходимо задать параметры, зависящие от браузера.
 - **Решение.** Перейдите к параметрам безопасности браузера и установите флажок "Использовать TLS 1.2".

## <a name="bash-troubleshooting"></a>Устранение неполадок в Bash

### <a name="cannot-run-the-docker-daemon"></a>Не удается запустить управляющую программу Docker

- **Сведения**. Cloud Shell использует контейнер для размещения оболочки среды, поэтому выполнение управляющей программы запрещено.
- **Устранение**. Используйте компонент [docker-machine](https://docs.docker.com/machine/overview/), установленный по умолчанию, для управления контейнерами Docker с удаленного узла Docker.

## <a name="powershell-troubleshooting"></a>Устранение неполадок в PowerShell

### <a name="gui-applications-are-not-supported"></a>Приложения с графическим пользовательским интерфейсом не поддерживаются

- **Сведения.** Если пользователь запускает приложение с графическим пользовательским интерфейсом, командная строка не отображается. Например, когда клонируется частный репозиторий GitHub с поддержкой двухфакторной проверки подлинности, отображается диалоговое окно выполнения двухфакторной проверки подлинности.
- **Разрешение.** Закройте и снова откройте оболочку.

### <a name="troubleshooting-remote-management-of-azure-vms"></a>Устранение неполадок удаленного управления виртуальными машинами Azure
> [!NOTE]
> Виртуальные машины Azure должны иметь общедоступный IP-адрес.

- **Сведения**. Из-за параметров брандмауэра Windows по умолчанию для WinRM может появиться следующее сообщение об ошибке: `Ensure the WinRM service is running. Remote Desktop into the VM for the first time and ensure it can be discovered.`.
- **Разрешение**. Запустите `Enable-AzVMPSRemoting`, чтобы включить на целевом компьютере все аспекты удаленного взаимодействия PowerShell.

### <a name="dir-does-not-update-the-result-in-azure-drive"></a>`dir` не обновляет результат на диске Azure

- **Сведения.** Чтобы оптимизировать взаимодействие с пользователем, по умолчанию результаты `dir` кэшируются на диск Azure.
- **Решение.** После создания, обновления или удаления ресурса Azure выполните команду `dir -force` для обновления результатов на диске Azure.

## <a name="general-limitations"></a>Общие ограничения

Azure Cloud Shell имеет следующие известные ограничения:

### <a name="quota-limitations"></a>Ограничения квот

Azure Cloud Shell имеет ограничение в 20 одновременных пользователей на одного клиента в каждом регионе. Если вы попытаетесь открыть более одновременные сеансы, чем это ограничение, вы увидите ошибку "клиент поверх квоты". Если у вас есть законная потребность в открытии большего числа сеансов (например, для учебных семинаров), обратитесь в службу поддержки, прежде чем запрашивать увеличение квоты.

Cloud Shell предоставляется в качестве бесплатной службы и предназначена для использования при настройке среды Azure, а не в качестве платформы общего назначения. Чрезмерное автоматизированное использование может считаться нарушением условий предоставления услуг Azure и может привести к блокировке доступа к Cloud Shell.

### <a name="system-state-and-persistence"></a>Состояние системы и сохраняемость

Компьютер, предоставляющий сеанс Cloud Shell, является временным и перезапускается, если сеанс не используется в течение 20 минут. Для Cloud Shell требуется подключение файлового ресурса Azure. Поэтому ваша подписка должна иметь возможность настраивать ресурсы хранилища для доступа к Cloud Shell. Дополнительные рекомендации:

- Если подключено хранилище, то сохраняются только изменения в каталоге `clouddrive`. Кроме того, в Bash сохраняется ваш каталог `$HOME`.
- Файловые ресурсы Azure можно подключить только в пределах [назначенного региона](persisting-shell-storage.md#mount-a-new-clouddrive).
  - В Bash выполните команду `env`, чтобы найти регион, заданный для `ACC_LOCATION`.
- Файлы Azure поддерживают только локально избыточное хранилище в геоизбыточные учетные записи.

### <a name="browser-support"></a>Поддержка браузеров

Cloud Shell поддерживает последние версии следующих браузеров:

- Microsoft Edge
- Microsoft Internet Explorer
- Google Chrome
- Mozilla Firefox;
- Apple Safari.
  - Использование Safari в режиме защищенного просмотра не поддерживается.

### <a name="copy-and-paste"></a>Копирование и вставка

[!INCLUDE [copy-paste](../../includes/cloud-shell-copy-paste.md)]

### <a name="usage-limits"></a>Ограничения использования

Cloud Shell предназначен для интерактивного использования. В результате любые длительные неинтерактивные сеансы завершаются без предупреждения.

### <a name="user-permissions"></a>Разрешения пользователя

Разрешения задаются как для обычных пользователей без доступа к sudo. Все, что устанавливается за пределами каталога `$Home`, не сохраняется.

## <a name="bash-limitations"></a>Ограничения Bash

### <a name="editing-bashrc"></a>Изменение файла .bashrc

Будьте внимательны при изменении файла .bashrc, так как некоторые изменения могут привести к непредвиденным ошибкам в Cloud Shell.

## <a name="powershell-limitations"></a>Ограничения PowerShell

### <a name="preview-version-of-azuread-module"></a>предварительная версия модуля AzureAD

В настоящее время доступен модуль `AzureAD.Standard.Preview`. Это предварительная версия на основе .NET Standard. Этот модуль предоставляет такую же функциональность, как `AzureAD`.

## <a name="personal-data-in-cloud-shell"></a>Персональные данные в Cloud Shell

Azure Cloud Shell серьезно относится к личным данным. Данные, собранные и хранимые службой Azure Cloud Shell, используются для предоставления значений по умолчанию при работе пользователя. Это могут быть последние использованные оболочки, предпочтительный размер шрифта, предпочтительный тип шрифта и сведения о файловом ресурсе для облачного диска. На случай, если вам понадобится экспортировать или удалить эти данные, используйте следующие инструкции.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

### <a name="export"></a>Экспорт
Чтобы **экспортировать** пользовательские настройки, сохраненные Cloud Shell, такие как предпочитаемая оболочка, размер и тип шрифта, выполните следующие команды.

1. [![Изображение, показывающее кнопку запуска Azure Cloud Shell.](https://shell.azure.com/images/launchcloudshell.png)](https://shell.azure.com)

2. В Bash или PowerShell выполните следующие команды.

Bash:

  ```
  token="Bearer $(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s | jq -r ".access_token")"
  curl https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -H Authorization:"$token" -s | jq
  ```

PowerShell.

  ```powershell
  $token= ((Invoke-WebRequest -Uri "$env:MSI_ENDPOINT`?resource=https://management.core.windows.net/" -Headers @{Metadata='true'}).content |  ConvertFrom-Json).access_token
  ((Invoke-WebRequest -Uri https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -Headers @{Authorization = "Bearer $token"}).Content | ConvertFrom-Json).properties | Format-List
```

### <a name="delete"></a>Удалить
Чтобы **удалить** пользовательские настройки, сохраненные Cloud Shell, такие как предпочитаемая оболочка, размер и тип шрифта, выполните следующие команды. При следующем запуске Cloud Shell вам будет предложено еще раз выставить файловый ресурс. 

>[!Note]
> При удалении параметров пользователя сам общий ресурс службы файлов Azure не будет удален. Перейдите к службе файлов Azure для выполнения этого действия.

1. [![Изображение, показывающее кнопку запуска Azure Cloud Shell.](https://shell.azure.com/images/launchcloudshell.png)](https://shell.azure.com)

2. В Bash или PowerShell выполните следующие команды.

Bash:

  ```
  token="Bearer $(curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true -s | jq -r ".access_token")"
  curl -X DELETE https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -H Authorization:"$token"
  ```

PowerShell.

  ```powershell
  $token= ((Invoke-WebRequest -Uri "$env:MSI_ENDPOINT`?resource=https://management.core.windows.net/" -Headers @{Metadata='true'}).content |  ConvertFrom-Json).access_token
  Invoke-WebRequest -Method Delete -Uri https://management.azure.com/providers/Microsoft.Portal/usersettings/cloudconsole?api-version=2017-12-01-preview -Headers @{Authorization = "Bearer $token"}
  ```
## <a name="azure-government-limitations"></a>Ограничения Azure для государственных организаций
Azure Cloud Shell в Azure для государственных организаций доступны только в портал Azure.

>[!Note]
> Подключение к GCC-Highным или государственным облакам с облаком для Exchange Online в настоящее время не поддерживается.

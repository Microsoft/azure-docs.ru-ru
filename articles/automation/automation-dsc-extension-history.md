---
title: Работа с журналом версий расширения Desired State Configuration в Azure
description: В этой статье содержатся сведения журнала версий для расширения Desired State Configuration (DSC) в Azure.
ms.date: 02/17/2021
keywords: dsc, powershell, azure, расширение
services: automation
ms.subservice: dsc
ms.topic: conceptual
ms.openlocfilehash: e51bce6e015ef7367625b010b74e5d2422b35051
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100651808"
---
# <a name="work-with-azure-desired-state-configuration-extension-version-history"></a>Работа с журналом версий расширения Desired State Configuration в Azure

[Расширение](../virtual-machines/extensions/dsc-overview.md) виртуальной машины Desired State Configuration (DSC) Azure обновляется по мере необходимости для поддержки улучшений и новых возможностей, предоставляемых Azure, Windows Server и Windows Management Framework (WMF), включающей Windows PowerShell.

В этой статье содержатся сведения о каждой версии расширения виртуальной машины Azure DSC, поддерживаемых средах, а также комментарии и примечания к новым возможностям или изменениям.

## <a name="latest-version"></a>Последняя версия

### <a name="version-283"></a>Версия 2,83

- **Дата выпуска:**
  - Февраль 2021 года
- **Поддержка ОС:**
  - Windows Server 2019
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 с пакетом обновления 1 (SP1)
  - Клиент Windows 7, 8.1, 10
  - Nano Server
- **Поддержка WMF:**
  - WMF 5.1
  - WMF 5.0 RTM
  - Обновление WMF 4.0
  - WMF 4.0
- **Среда**.
  - Azure
  - Azure для Китая (21Vianet)
  - Azure для государственных организаций
- **Примечания.** Этот выпуск включает исправление для неподписанных двоичных файлов с помощью расширения виртуальной машины Windows.

### <a name="version-280"></a>Версия 2,80

- **Дата выпуска:**
  - 26 сентября, Сен-2019 (Azure) | 6 июля 2020 г. (Azure Китая ВИАНЕТ 21) | 20 июля 2020 г. (Azure для государственных организаций)
- **Поддержка ОС:**
  - Windows Server 2019
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 с пакетом обновления 1 (SP1)
  - Клиент Windows 7, 8.1, 10
  - Nano Server
- **Поддержка WMF:**
  - WMF 5.1
  - WMF 5.0 RTM
  - Обновление WMF 4.0
  - WMF 4.0
- **Среда**.
  - Azure
  - Azure для Китая (21Vianet)
  - Azure для государственных организаций
- **Примечания.** В этот выпуск не включены новые функции.

### <a name="version-276"></a>Версия 2.76

- **Дата выпуска:**
  - 9 мая 2018 года (Azure) | 21 июня 2018 года (Azure для Китая (21Vianet), Azure для государственных организаций)
- **Поддержка ОС:**
  - Windows Server 2016
  - Windows Server 2012 R2
  - Windows Server 2012
  - Windows Server 2008 R2 с пакетом обновления 1 (SP1)
  - Клиент Windows 7, 8.1, 10
  - Nano Server
- **Поддержка WMF:**
  - WMF 5.1
  - WMF 5.0 RTM
  - Обновление WMF 4.0
  - WMF 4.0
- **Среда**.
  - Azure
  - Azure для Китая (21Vianet)
  - Azure для государственных организаций
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Улучшения в метаданных расширения для подсостояния и другие незначительные исправления ошибок.

## <a name="supported-versions"></a>Поддерживаемые версии

> [!WARNING]
> В версиях 2,4 – 2,13 для WMF 5,0 общедоступная Предварительная версия, срок действия сертификатов подписи которых истек до 2016 августа.
> Дополнительные сведения об этой ошибке см. в следующей [статье блога](https://devblogs.microsoft.com/powershell/azure-dsc-extension-versions-2-4-up-to-2-13-will-retire-in-august/).

### <a name="version-275"></a>Версия 2.75

- **Дата выпуска:** 5 марта 2018 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Windows Client 7, 8.1, 10, Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - После применения в GitHub протокола TLS 1,2 вы не сможете подключить виртуальную машину к конфигурации состояния службы автоматизации Azure с помощью шаблонов DIY диспетчер ресурсов, доступных в Azure Marketplace, или использовать расширение DSC для получения любых конфигураций, размещенных на GitHub. При развертывании расширения возвращается ошибка, аналогичная следующей:

    ```json
    {
        "code": "DeploymentFailed",
        "message": "At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.",
        "details": [{
            "code": "Conflict",
            "message": "{
                \"status\": \"Failed\",
                \"error\": {
                    \"code\": \"ResourceDeploymentFailure\",
                    \"message\": \"The resource operation completed with terminal provisioning state 'Failed'.\",
                    \"details\": [ {
                        \"code\": \"VMExtensionProvisioningError\",
                        \"message\": \"VM has reported a failure when processing extension 'Microsoft.Powershell.DSC'.
                        Error message: \\\"The DSC Extension failed to execute: Error downloading
                        https://github.com/Azure/azure-quickstart-templates/raw/master/dsc-extension-azure-automation-pullserver/UpdateLCMforAAPull.zip
                        after 29 attempts: The request was aborted: Could not create SSL/TLS secure channel..\\nMore information about the failure can
                        be found in the logs located under 'C:\\\\WindowsAzure\\\\Logs\\\\Plugins\\\\Microsoft.Powershell.DSC\\\\2.74.0.0' on the VM.\\\".\"
                    } ]
                }
            }"
        }]
    }
    ```

  - TLS 1.2 принудительно используется в новой версии расширения. При развертывании расширения, если вы уже указали `AutoUpgradeMinorVersion = true` в шаблоне диспетчер ресурсов, расширение будет обновляться по 2,75. Чтобы выполнить обновление вручную, укажите `TypeHandlerVersion = 2.75` в шаблоне Resource Manager.

### <a name="version-270---272"></a>Версии с 2.70 по 2.72

- **Дата выпуска:** 13 ноября 2017 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Windows Client 7, 8.1, 10, Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Исправления ошибок & усовершенствований, упрощающих использование конфигурации состояния службы автоматизации Azure на портале и с шаблоном диспетчер ресурсов. Дополнительные сведения см. в разделе [сценарий конфигурации по умолчанию](../virtual-machines/extensions/dsc-overview.md) в документации по расширению DSC.

### <a name="version-226"></a>Версия 2.26

- **Дата выпуска:** 9 июня 2017 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Windows Client 7, 8.1, 10, Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Усовершенствования телеметрии.

### <a name="version-225"></a>Версия 2.25

- **Дата выпуска:** 2 июня 2017 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Windows Client 7, 8.1, 10, Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Было добавлено несколько исправлений ошибок и ряд других незначительных улучшений.

### <a name="version-224"></a>Версия 2.24

- **Дата выпуска:** 13 апреля 2017 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Предоставляет UUID виртуальной машины и идентификатор агента DSC как метаданные расширения. Были добавлены другие незначительные улучшения.

### <a name="version-223"></a>Версия 2.23

- **Дата выпуска:** 15 марта 2017 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Добавлены исправления ошибок и другие улучшения.

### <a name="version-222"></a>Версия 2.22

- **Дата выпуска:** 8 февраля 2017 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Nano Server
- **Поддержка WMF:** WMF 5.1, WMF 5.0 RTM, WMF 4.0 Update, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,1](https://devblogs.microsoft.com/powershell/wmf-5-1-releasing-january-2017/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Теперь расширение DSC поддерживает WMF 5,1.
  - Были добавлены другие незначительные улучшения.

### <a name="version-221"></a>Версия 2.21

- **Дата выпуска:** 2 декабря 2016 г.
- **Поддержка ОС:** Windows Server 2016, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1), Nano Server
- **Поддержка WMF:** Предварительная версия WMF 5.1, WMF 5.0 RTM, обновление WMF 4.0, WMF 4.0
- **Среда**. Azure
- **Примечания.** В этой версии используется DSC в составе Windows Server 2016; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка). В случае Nano Server роль DSC устанавливается на виртуальной машине.
- **Новые функции:**
  - Теперь расширение DSC доступно на сервере Nano Server. Эта версия в основном содержит изменения кода для запуска расширения на Nano Server.
  - Были добавлены другие незначительные улучшения.

### <a name="version-220"></a>Версия 2.20

- **Дата выпуска:** 2 августа 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** Предварительная версия WMF 5.1, WMF 5.0 RTM, обновление WMF 4.0, WMF 4.0
- **Среда**. Azure
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - Поддержка предварительной версии WMF 5.1. При первой публикации эта версия являлась необязательным обновлением, и для установки предварительной версии WMF 5.1 приходилось указывать Wmfversion = "5.1PP" в шаблонах Resource Manager.
    Выбор Wmfversion = "latest" по-прежнему приводит к установке [WMF 5.0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/).
    Дополнительные сведения о предварительной версии WMF 5.1 см. в [этом блоге](https://devblogs.microsoft.com/powershell/announcing-windows-management-framework-wmf-5-1-preview/).
  - Были добавлены другие незначительные исправления и улучшения.

### <a name="version--219"></a>Версия 2.19

- **Дата выпуска:** 3 июня 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** WMF 5.0 RTM, обновление WMF 4.0, WMF 4.0
- **Среда**. Azure, Azure для Китая (21Vianet), Azure для государственных организаций
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - Теперь расширение DSC доступно в Azure для Китая ВИАНЕТ 21. Эта версия содержит исправления для запуска расширения в Azure Китая ВИАНЕТ 21.

### <a name="version-218"></a>Версия 2.18

- **Дата выпуска:** 3 июня 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** WMF 5.0 RTM, обновление WMF 4.0, WMF 4.0
- **Среда**. Azure
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - Неблокируемость телеметрии при возникновении ошибки во время скачивания исправления телеметрии (известная проблема Azure DNS) или во время установки.
  - Исправлена временная проблема, когда расширение прекращало обработку конфигурации после перезагрузки. Это привело к тому, что расширение DSC останется в состоянии перехода.
  - Были добавлены другие незначительные исправления и улучшения.

### <a name="version-217"></a>Версия 2.17

- **Дата выпуска:** 26 апреля 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** WMF 5.0 RTM, обновление WMF 4.0, WMF 4.0
- **Среда**. Azure
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - Поддержка обновления WMF 4.0. Дополнительные сведения об обновлениях WMF 4.0 см. в [этом блоге](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-4-0-update-now-available-for-windows-server-2012-windows-server-2008-r2-sp1-and-windows-7-sp1/).
  - Логика повторных попыток при ошибках, возникающих во время установки расширения DSC или при применении расширения DSC после установки. В рамках этого изменения расширение будет повторять установку в случае сбоя предыдущей установки или повторно активировать ранее неудачную конфигурацию DSC не более трех раз до перехода в состояние завершения (успех или ошибка) либо до поступления нового запроса. Если работа расширения завершается ошибкой из-за недопустимых параметров или введенных данных, повторная попытка не предпринимается. В этом случае расширение необходимо вызывать еще раз с помощью нового запроса и правильных параметров пользователя. 

  > [!NOTE]
   > Расширение DSC зависит от агента виртуальной машины Azure на наличие повторных попыток. Агент виртуальных машин Azure вызывает расширение с использованием последнего неудачного запроса, пока оно не перейдет в состояние успеха или ошибки.

### <a name="version-216"></a>Версия 2.16

- **Дата выпуска:** 21 апреля 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** WMF 5.0 RTM, WMF 4.0
- **Среда**. Azure
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - Улучшения в обработке ошибок и другие незначительные исправления ошибок.
  - Новое свойство в параметрах расширения DSC. `ForcePullAndApply` в Адванцедоптионс добавляется, чтобы включить расширение DSC для использования конфигурации DSC, когда режим обновления — Pull (в отличие от режима принудительной отправки по умолчанию). Дополнительные сведения о параметрах расширения DSC см. в [этом блоге](https://devblogs.microsoft.com/powershell/arm-dsc-extension-settings/).

### <a name="version-215"></a>Версия 2.15

- **Дата выпуска:** 14 марта 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** WMF 5.0 RTM, WMF 4.0
- **Среда**. Azure
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - В версию 2.14 расширения были внесены изменения в установку WMF RTM. При обновлении с версии расширения 2.13.2.0 до 2.14.0.0 можно заметить, что выполнение некоторых командлетов DSC или конфигурации завершается ошибкой — "Экземпляр с заданными значениями свойств не найден". Дополнительные сведения см. в [заметках о выпуске DSC](/powershell/scripting/wmf/known-issues/known-issues-dsc). Способы решения этих проблем были добавлены в версию 2.15. 
  - Если вы уже установили версию 2,14 и используете одну из указанных выше двух проблем, необходимо выполнить эти действия вручную. В сеансе PowerShell с повышенными привилегиями выполните следующие команды:
    - `Remove-Item -Path $env:SystemRoot\system32\Configuration\DSCEngineCache.mof`
    - `mofcomp $env:windir\system32\wbem\DscCoreConfProv.mof`

### <a name="version-214"></a>Версия 2.14

- **Дата выпуска:** 25 февраля 2016 г.
- **Поддержка ОС:** Windows Server 2016 Technical Preview, Windows Server 2012 R2, Windows Server 2012, Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- **Поддержка WMF:** WMF 5.0 RTM, WMF 4.0
- **Среда**. Azure
- **Примечания.** Эта версия использует DSC в составе Windows Server 2016 Technical Preview; для других ОС Windows устанавливается [Windows Management Framework 5,0 RTM](https://devblogs.microsoft.com/powershell/windows-management-framework-wmf-5-0-rtm-is-now-available-via-the-microsoft-update-catalog/) (для установки WMF требуется перезагрузка).
- **Новые функции:**
  - Использование WMF RTM.
  - Включает сбор данных для улучшения качества расширения DSC. Дополнительные сведения см. в этой [статье блога](https://devblogs.microsoft.com/powershell/azure-dsc-extension-data-collection-2/).
  - Обновленный формат параметров для расширения в шаблоне Resource Manager. Дополнительные сведения см. в этой [статье блога](https://devblogs.microsoft.com/powershell/arm-dsc-extension-settings/).
  - Исправления ошибок и другие усовершенствования.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о PowerShell DSC см. в [центре документации по PowerShell](/powershell/scripting/dsc/overview/overview).
- Изучите [шаблон Resource Manager для расширения DSC](../virtual-machines/extensions/dsc-template.md).
- Для других функций и ресурсов, которыми можно управлять с помощью PowerShell DSC, перейдите в [коллекцию PowerShell](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0).
- Сведения о передаче конфиденциальных параметров в конфигурации см. в статье о [безопасном управлении учетными данными с помощью обработчика расширения DSC](../virtual-machines/extensions/dsc-credentials.md).

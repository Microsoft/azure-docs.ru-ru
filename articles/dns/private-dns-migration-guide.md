---
title: Перенос устаревших частных зон Azure DNS на новую модель ресурсов
titleSuffix: Azure DNS
description: Это руководство предоставляет пошаговые инструкции по переносу устаревших частных зон DNS на последнюю модель ресурсов
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 06/18/2019
ms.author: rohink
ms.openlocfilehash: 3f0856f85e279f97934fff506a052c8fd214ff73
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105641232"
---
# <a name="migrating-legacy-azure-dns-private-zones-to-new-resource-model"></a>Перенос устаревших частных зон Azure DNS на новую модель ресурсов

Во время действия общедоступной предварительной версии частные зоны DNS создавались с помощью ресурса "dnszones" со свойством "zoneType", для которого установлено значение "Private". Такие зоны не будут поддерживаться после 31 декабря 2019 г. и должны быть перенесены в общедоступную модель ресурсов, которая использует тип ресурса "privateDnsZones" вместо "dnszones". Процесс переноса прост, и мы предоставили сценарий PowerShell для автоматизации этого процесса. Это руководство содержит пошаговые инструкции по переводу частных зон Azure DNS на новую модель ресурсов.

Чтобы найти ресурсы dnszones, требующие миграции, выполните приведенную ниже команду в Azure CLI.
```azurecli
az account set --subscription <SubscriptionId>
az network dns zone list --query "[?zoneType=='Private']"
```

## <a name="prerequisites"></a>Предварительные условия

Убедитесь, что установлена последняя версия Azure PowerShell. Дополнительные сведения по средству Azure PowerShell (Az) и его установке см. по адресу https://docs.microsoft.com/powershell/azure/new-azureps-module-az.

Установите модуль Az.PrivateDns для Azure PowerShell. Для этого откройте окно PowerShell с повышенными привилегиями (режим администратора) и введите следующую команду:

```powershell
Install-Module -Name Az.PrivateDns
```

>[!IMPORTANT]
>Процесс переноса полностью автоматизирован и не вызывает простоев. Но если вы используете частные зоны Azure DNS (предварительная версия) в критически важной рабочей среде, следует выполнить описанный ниже процесс переноса во время планового обслуживания. Не изменяйте конфигурацию или наборы записей частных зон DNS во время выполнения сценария миграции.

## <a name="installing-the-script"></a>Установка сценария

Откройте окно PowerShell с повышенными привилегиями (режим администратора) и выполните следующую команду:

```powershell
install-script PrivateDnsMigrationScript
```

Введите "A", когда будет предложено установить сценарий.

![Установка сценария](./media/private-dns-migration-guide/install-migration-script.png)

Вы можете также вручную получить последнюю версию сценария PowerShell по адресу https://www.powershellgallery.com/packages/PrivateDnsMigrationScript.

>[!IMPORTANT]
>Скрипт миграции не должен запускаться в Azure Cloud Shell, а должен выполняться на виртуальной машине или на локальном компьютере, подключенном к Интернету.

## <a name="running-the-script"></a>Запуск скрипта

Выполните следующую команду, чтобы запустить сценарий:

```powershell
PrivateDnsMigrationScript.ps1
```

![Запуск скрипта](./media/private-dns-migration-guide/running-migration-script.png)

### <a name="enter-the-subscription-id-and-sign-in-to-azure"></a>Ввод идентификатора подписки и вход в Azure

Вам будет предложено ввести идентификатор подписки, содержащей частные зоны DNS, которые требуется перенести. Вам будет предложено войти в учетную запись Azure. Выполните вход, чтобы сценарий получил доступ к ресурсам частных зон DNS в подписке.

![Вход в Azure](./media/private-dns-migration-guide/login-migration-script.png)

### <a name="select-the-dns-zones-you-want-to-migrate"></a>Выбор зон DNS, которые требуется перенести

Сценарий получает список всех частных зон DNS в подписке и предлагает подтвердить те, которые требуется перенести. Введите "A", чтобы перенести все частные зоны DNS. После выполнения этого шага сценарий создаст новые частные зоны DNS с помощью новой модели ресурсов и скопирует данные в новую зону DNS. Этот шаг никоим образом не изменит существующие частные зоны DNS.

![Выбор зон DNS](./media/private-dns-migration-guide/migratezone-migration-script.png)

### <a name="switching-dns-resolution-to-the-new-dns-zones"></a>Переключение разрешения DNS на новые зоны DNS

После копирования зон и записей в новую модель ресурсов сценарий предложит переключить разрешение DNS на новые зоны DNS. Этот шаг удаляет связь между устаревшими частными зонами DNS и вашими виртуальными сетями. Когда устаревшая зона теряет связь с виртуальными сетями, новые зоны DNS, созданные на предыдущем шаге, автоматически принимают разрешение DNS для этих виртуальных сетей.

Выберите "A", чтобы переключить разрешение DNS для всех виртуальных сетей.

![Переключение разрешения имен](./media/private-dns-migration-guide/switchresolution-migration-script.png)

### <a name="verify-the-dns-resolution"></a>Проверка разрешения DNS

Прежде чем продолжить, проверьте, что разрешение DNS в зонах DNS работает должным образом. Вы можете войти на виртуальные машины Azure и отправить запрос nslookup к перенесенным зонам, чтобы убедиться, что разрешение DNS работает.

![Проверка разрешения имен](./media/private-dns-migration-guide/verifyresolution-migration-script.png)

Если обнаружится, что запросы DNS не разрешаются, подождите несколько минут и повторите их. Если запросы DNS выполняются надлежащим образом, введите "Y", когда будет предложено удалить виртуальную сеть из частной зоны DNS.

![Подтверждение разрешения имен](./media/private-dns-migration-guide/confirmresolution-migration-script.png)

>[!IMPORTANT]
>Если по какой-то причине разрешение DNS в перенесенных зонах не работает должным образом, введите "N" на предыдущем шаге, и сценарий переключит разрешение DNS обратно на устаревшие зоны. Создайте запрос в службу поддержки, и мы поможем вам с переносом ваших зон DNS.

## <a name="cleanup"></a>Очистка

Этот шаг приведет к удалению устаревших зон DNS. Его следует выполнять только после того, как вы проверите, что разрешение DNS работает должным образом. Появится запрос на удаление каждой частной зоны DNS. Убедившись, что разрешение DNS для этих зон работает правильно, введите "Y" для каждого запроса.

![Очистка](./media/private-dns-migration-guide/cleanup-migration-script.png)

## <a name="update-your-automation"></a>Обновление службы автоматизации

Если вы применяете службу автоматизации, включая шаблоны, сценарии PowerShell или пользовательский код, разработанный с помощью пакета SDK, необходимо обновить ее, чтобы использовать новую модель ресурсов для частных зон DNS. Ниже приведены ссылки на документацию по CLI/PS/SDK для новых частных зон DNS.
* [REST API для частных зон Azure DNS](/rest/api/dns/privatedns/privatezones)
* [CLI для частных зон Azure DNS](/cli/azure/network/private-dns/link/vnet?view=azure-cli-latest)
* [PowerShell для частных зон Azure DNS](/powershell/module/az.privatedns/)
* [Пакет SDK для частных зон Azure DNS](/dotnet/api/overview/azure/privatedns/management?view=azure-dotnet-preview)

## <a name="need-further-help"></a>Дополнительная помощь

Если необходимо получить дополнительную помощь в процессе переноса или по какой-либо причине перечисленные выше шаги не помогли, создайте запрос в службу поддержки. Приложите файл расшифровку, созданный с помощью сценария PowerShell, к вашему запросу в службу поддержки.

## <a name="next-steps"></a>Дальнейшие шаги

* Узнайте, как создать частную зону в Azure DNS с помощью [Azure PowerShell](./private-dns-getstarted-powershell.md) или [интерфейса командной строки Azure](./private-dns-getstarted-cli.md).

* Ознакомьтесь с распространенными [сценариями для частных зон](./private-dns-scenarios.md), которые можно реализовать с использованием частных зон в Azure DNS.

* Изучите [часто задаваемые вопросы](./dns-faq-private.md) о частных зонах в Azure DNS, где описаны характерные реакции на события для некоторых типов операций.

* Дополнительные сведения о записях и зонах DNS см. в [обзоре зон и записей DNS](dns-zones-records.md).

* Дополнительные сведения о некоторых других ключевых [сетевых возможностях](../networking/networking-overview.md) Azure.

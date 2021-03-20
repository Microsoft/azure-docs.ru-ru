---
title: Ритмичность исправления операционной системы и среды выполнения
description: Узнайте, как служба приложений Azure обновляет операционную систему и среды выполнения, какие уровни среды выполнения и обновления имеют ваши приложения, а также как можно получить объявления об обновлениях.
ms.topic: article
ms.date: 02/02/2018
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 8b52223aea0f0bdfecf58906ac192e893da3b47d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96558493"
---
# <a name="os-and-runtime-patching-in-azure-app-service"></a>Установка исправлений для ОС и среды выполнения в Службе приложений Azure

В этой статье объясняется, как получить сведения о версии операционной системы или программного обеспечения в [службе приложений](overview.md). 

Служба приложений предоставляется в качестве платформы как услуги (PaaS). Это означает, что операционной системой и стеком приложений управляет Azure. Вы управляете только своим приложением и его данными. Больше средств управления операционной системой и стеком приложений доступно в [виртуальных машинах Azure](../virtual-machines/index.yml). С учетом вышесказанного, вам, как пользователю службы приложений, пригодится эта информация:

-   Как и когда применяются обновления ОС?
-   Как в службе приложений устанавливаются исправления для защиты от значительных уязвимостей (например, нулевого дня).
-   В каких версиях операционной системы и среды выполнения работают ваши приложения?

В целях безопасности некоторые детали сведений о безопасности не публикуются. Эта статья поможет вам устранить проблемы за счет повышения прозрачности процесса. Из нее вы узнаете, как получать актуальные объявления, связанные с безопасностью, и сведения об обновлениях среды выполнения.

## <a name="how-and-when-are-os-updates-applied"></a>Как и когда применяются обновления ОС?

Azure управляет установкой исправлений ОС на двух уровнях: физические серверы и гостевые виртуальные машины, на которых работают ресурсы службы приложений. Физические серверы и гостевые виртуальные машины обновляются раз в месяц, что соответствует графика, называемого [вторником обновлений](/security-updates/). Эти обновления применяются автоматически. Во время обновлений гарантируется высокий уровень доступности служб Azure согласно Соглашению об уровне обслуживания. 

Подробные сведения о применении обновлений см. в статье [Demystifying the magic behind App Service OS updates](https://azure.github.io/AppService/2018/01/18/Demystifying-the-magic-behind-App-Service-OS-updates.html) (Базовые сведения об обновлениях ОС с помощью службы приложений).

## <a name="how-does-azure-deal-with-significant-vulnerabilities"></a>Как Azure устраняет значительные уязвимости?

Когда серьезные уязвимости требуют немедленного применения исправлений, как например [уязвимость нулевого дня](https://wikipedia.org/wiki/Zero-day_(computing)), высокоприоритетные обновления обрабатываются для каждого отдельного случая.

Чтобы получать критически важные извещения о безопасности в Azure, просматривайте [блог, посвященный вопросам безопасности Azure](https://azure.microsoft.com/blog/topics/security/). 

## <a name="when-are-supported-language-runtimes-updated-added-or-deprecated"></a>Когда среды выполнения на поддерживаемых языках обновляются, добавляются или устаревают?

В экземпляры службы приложений периодически добавляются новые стабильные версии сред выполнения на поддерживаемых языках (основная, дополнительная и исправленная). Некоторые обновления перезаписывают существующую установку, пока другие устанавливаются параллельно с существующими версиями. Установка перезаписи означает, что приложение автоматически выполняется в обновленной среде выполнения. При параллельной установке необходимо вручную перенести приложение, чтобы воспользоваться преимуществами новой версии среды выполнения. Дополнительные сведения см. в одном из подразделов.

Об обновлениях и прекращении обслуживании сред выполнения сообщается здесь:

- https://azure.microsoft.com/updates/?product=app-service 
- https://github.com/Azure/app-service-announcements/issues

> [!NOTE] 
> Эти сведения относятся к языковым средам выполнения, которые встроены в приложение служб приложений. Например, среда выполнения, которую вы отправляете в службу приложений, остается неизменной, пока вы вручную не обновите ее.
>
>

### <a name="new-patch-updates"></a>Новые обновления исправлений

Обновления с исправлениями для .NET, PHP, Java SDK или Tomcat версии применяются автоматически, перезаписывая существующую установку с последней версией. Обновления исправлений Node.js устанавливаются параллельно с существующими версиями (как основная и дополнительная версии в следующем разделе). Новые версии исправлений Python можно установить вручную с помощью [расширений сайта](https://azure.microsoft.com/blog/azure-web-sites-extensions/), параллельно со встроенными установками Python.

### <a name="new-major-and-minor-versions"></a>Новые основная и вспомогательная версии

Когда добавляется новая основная или вспомогательная версия, она устанавливается параллельно с существующими. Можно вручную обновить приложение до новой версии. Если вы настроили версию среды выполнения в файле конфигурации (например, в `web.config` и `package.json`), необходимо выполнить обновление с помощью того же метода. Если вы использовали параметр службы приложений для настройки версии среды выполнения, вы можете изменить его на [портале Azure](https://portal.azure.com) или с помощью команды[Azure CLI](/cli/azure/get-started-with-azure-cli) в [Cloud Shell](../cloud-shell/overview.md), как показано в следующих примерах:

```azurecli-interactive
az webapp config set --net-framework-version v4.7 --resource-group <groupname> --name <appname>
az webapp config set --php-version 7.0 --resource-group <groupname> --name <appname>
az webapp config appsettings set --settings WEBSITE_NODE_DEFAULT_VERSION=8.9.3 --resource-group <groupname> --name <appname>
az webapp config set --python-version 3.4 --resource-group <groupname> --name <appname>
az webapp config set --java-version 1.8 --java-container Tomcat --java-container-version 9.0 --resource-group <groupname> --name <appname>
```

### <a name="deprecated-versions"></a>Устаревшие версии  

Когда версия устаревает, мы сообщаем дату ее удаления, чтобы вы могли спланировать обновление версии среды выполнения. 

## <a name="how-can-i-query-os-and-runtime-update-status-on-my-instances"></a>Как запросить состояние обновления среды выполнения и ОС в моих экземплярах?  

Если блокируется доступ к критически важной информации ОС (см. статью [Функциональные возможности операционной системы для службы приложений Azure](operating-system-functionality.md)), с помощью [консоли Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) к экземпляру службы приложения можно отправить запрос на получение данных о версии среды выполнения и операционной системы. 

В следующей таблице показано, как запросить версии Windows и языковой среды выполнения, в которых работают ваши приложения:

| Сведения | Где ее найти | 
|-|-|
| Версия Windows | См. `https://<appname>.scm.azurewebsites.net/Env.cshtml` (в разделе "Сведения о системе") |
| Версия .NET | Откройте `https://<appname>.scm.azurewebsites.net/DebugConsole` и выполните следующую команду в командной строке: <br>`reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full"` |
| Версия .NET Core | Откройте `https://<appname>.scm.azurewebsites.net/DebugConsole` и выполните следующую команду в командной строке: <br> `dotnet --version` |
| Версия PHP | Откройте `https://<appname>.scm.azurewebsites.net/DebugConsole` и выполните следующую команду в командной строке: <br> `php --version` |
| Версия Node.js по умолчанию | В [Cloud Shell](../cloud-shell/overview.md)выполните следующую команду: <br> `az webapp config appsettings list --resource-group <groupname> --name <appname> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION']"` |
| Версия Python | Откройте `https://<appname>.scm.azurewebsites.net/DebugConsole` и выполните следующую команду в командной строке: <br> `python --version` |  
| Версия Java | Откройте `https://<appname>.scm.azurewebsites.net/DebugConsole` и выполните следующую команду в командной строке: <br> `java -version` |  

> [!NOTE]  
> Доступ к расположению реестра `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\Packages`, где хранятся сведения об [исправлениях (номер в базе знаний)](/security-updates/SecurityBulletins/securitybulletins), заблокирован.
>
>

## <a name="more-resources"></a>Дополнительные ресурсы

[Центр управления безопасностью: безопасность](https://www.microsoft.com/en-us/trustcenter/security)  
[64-разрядная версия ASP.NET Core в службе приложений Azure](https://gist.github.com/glennc/e705cd85c9680d6a8f1bdb62099c7ac7)

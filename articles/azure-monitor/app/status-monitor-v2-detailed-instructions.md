---
title: Подробные инструкции по агенту Application Insights Azure | Документация Майкрософт
description: Подробные инструкции по началу работы с агентом Application Insights. Отслеживайте производительность веб-сайта без повторного развертывания веб-сайта. Работает с веб-приложениями ASP.NET, размещенными локально, в виртуальных машинах или в Azure.
ms.topic: conceptual
author: TimothyMothra
ms.author: tilee
ms.date: 04/23/2019
ms.openlocfilehash: 29922f088a51e4876e5e2ec8fe87c3bbce4482f3
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102521685"
---
# <a name="application-insights-agent-formerly-named-status-monitor-v2-detailed-instructions"></a>Агент Application Insights (прежнее название — монитор состояния v2): подробные инструкции

В этой статье описывается, как подключить коллекция PowerShell и скачать модуль Аппликатионмонитор.
Включены наиболее распространенные параметры, которые необходимо приступить к работе.
Мы также предоставили инструкции по скачиванию вручную на случай, если у вас нет доступа к Интернету.

## <a name="get-an-instrumentation-key"></a>Получение ключа инструментирования

Чтобы приступить к работе, необходим ключ инструментирования. Дополнительные сведения см. в статье [создание Application Insights ресурса](create-new-resource.md#copy-the-instrumentation-key).

## <a name="run-powershell-as-admin-with-an-elevated-execution-policy"></a>Запуск PowerShell от имени администратора с политикой выполнения с повышенными правами

### <a name="run-as-admin"></a>Запуск от имени администратора

Для внесения изменений на компьютер PowerShell требуются разрешения уровня администратора.
### <a name="execution-policy"></a>Политика выполнения
- Описание: по умолчанию запуск сценариев PowerShell отключен. Рекомендуется разрешить скрипты RemoteSigned только для текущей области.
- Справочник: [Общие сведения о политиках выполнения](/powershell/module/microsoft.powershell.core/about/about_execution_policies) и [Set-ExecutionPolicy](/powershell/module/microsoft.powershell.security/set-executionpolicy).
- Команда: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process` .
- Необязательный параметр:
    - `-Force`. Обход запроса подтверждения.

**Примеры ошибок**

```output
Install-Module : The 'Install-Module' command was found in the module 'PowerShellGet', but the module could not be
loaded. For more information, run 'Import-Module PowerShellGet'.

Import-Module : File C:\Program Files\WindowsPowerShell\Modules\PackageManagement\1.3.1\PackageManagement.psm1 cannot
be loaded because running scripts is disabled on this system. For more information, see about_Execution_Policies at
https:/go.microsoft.com/fwlink/?LinkID=135170.
```


## <a name="prerequisites-for-powershell"></a>Предварительные требования для PowerShell

Произведите аудит экземпляра PowerShell, выполнив `$PSVersionTable` команду.
Эта команда создает следующие выходные данные:

```output
Name                           Value
----                           -----
PSVersion                      5.1.17763.316
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.17763.316
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

Эти инструкции были написаны и протестированы на компьютере под управлением Windows 10 и перечисленных выше версиях.

## <a name="prerequisites-for-powershell-gallery"></a>Необходимые условия для коллекция PowerShell

Эти действия приводят к подготовке сервера к скачиванию модулей из коллекция PowerShell.

> [!NOTE] 
> Коллекция PowerShell поддерживается в Windows 10, Windows Server 2016 и PowerShell 6 +.
> Дополнительные сведения о более ранних версиях см. в разделе [Установка PowerShellGet](/powershell/scripting/gallery/installing-psget).


1. Запустите PowerShell от имени администратора с политикой выполнения с повышенными правами.
2. Установите поставщик пакетов NuGet.
    - Описание. Этот поставщик необходим для взаимодействия с репозиториями на основе NuGet, например коллекция PowerShell.
    - Ссылка: [Install-PackageProvider](/powershell/module/packagemanagement/install-packageprovider).
    - Команда: `Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201` .
    - Дополнительные параметры:
        - `-Proxy`. Указывает прокси-сервер для запроса.
        - `-Force`. Обход запроса подтверждения.
    
    Вы получите этот запрос, если NuGet не настроен:

    ```output
    NuGet provider is required to continue
    PowerShellGet requires NuGet provider version '2.8.5.201' or newer to interact with NuGet-based repositories. 
    The NuGet provider must be available in 'C:\Program Files\PackageManagement\ProviderAssemblies' or
    'C:\Users\t\AppData\Local\PackageManagement\ProviderAssemblies'. You can also install the NuGet provider by running
    'Install-PackageProvider -Name NuGet -MinimumVersion 2.8.5.201 -Force'. Do you want PowerShellGet to install and import
    the NuGet provider now?
    [Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
    ```    

3. Настройте коллекция PowerShell как доверенный репозиторий.
    - Описание: по умолчанию коллекция PowerShell является недоверенным репозиторием.
    - Ссылка: [Set-PSRepository](/powershell/module/powershellget/set-psrepository).
    - Команда: `Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted` .
    - Необязательный параметр:
        - `-Proxy`. Указывает прокси-сервер для запроса.

    Вы получите запрос, если коллекция PowerShell не является доверенным:

    ```output
    Untrusted repository
    You are installing the modules from an untrusted repository. 
    If you trust this repository, change its InstallationPolicy value 
    by running the Set-PSRepository cmdlet. Are you sure you want to 
    install the modules from 'PSGallery'?
    [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
    ```

    Вы можете подтвердить это изменение и выполнить аудит всех Псрепоситориес, выполнив `Get-PSRepository` команду.

4. Установите последнюю версию PowerShellGet.
    - Описание: Этот модуль содержит средства, используемые для получения других модулей из коллекция PowerShell. Версия 1.0.0.1 поставляется с Windows 10 и Windows Server. Требуется версия 1.6.0 или более поздняя. Чтобы определить, какая версия установлена, выполните `Get-Command -Module PowerShellGet` команду.
    - Ссылка: [Установка PowerShellGet](/powershell/scripting/gallery/installing-psget).
    - Команда: `Install-Module -Name PowerShellGet` .
    - Дополнительные параметры:
        - `-Proxy`. Указывает прокси-сервер для запроса.
        - `-Force`. Обходит предупреждение "уже установлено" и устанавливает последнюю версию.

    Эта ошибка возникает, если вы не используете самую новую версию PowerShellGet:

    ```output
    Install-Module : A parameter cannot be found that matches parameter name 'AllowPrerelease'.
    At line:1 char:20
    Install-Module abc -AllowPrerelease
                   ~~~~~~~~~~~~~~~~
    CategoryInfo          : InvalidArgument: (:) [Install-Module], ParameterBindingException
    FullyQualifiedErrorId : NamedParameterNotFound,Install-Module
    ```

5. Перезапустите PowerShell. Невозможно загрузить новую версию в текущем сеансе. Новые сеансы PowerShell будут загружать последнюю версию PowerShellGet.

## <a name="download-and-install-the-module-via-powershell-gallery"></a>Скачайте и установите модуль с помощью коллекция PowerShell

Эти действия загрузит модуль AZ. Аппликатионмонитор из коллекция PowerShell.

1. Убедитесь, что выполнены все необходимые условия для коллекция PowerShell.
2. Запустите PowerShell от имени администратора с политикой выполнения с повышенными правами.
3. Установите модуль AZ. Аппликатионмонитор.
    - Ссылка: [Install-Module](/powershell/module/powershellget/install-module).
    - Команда: `Install-Module -Name Az.ApplicationMonitor` .
    - Дополнительные параметры:
        - `-Proxy`. Указывает прокси-сервер для запроса.
        - `-AllowPrerelease`. Позволяет устанавливать пакеты Alpha и бета-версии.
        - `-AcceptLicense`. Обход приглашения "принять лицензию"
        - `-Force`. Обходит предупреждение "недоверенный репозиторий".

## <a name="download-and-install-the-module-manually-offline-option"></a>Загрузка и установка модуля вручную (автономный режим)

Если по какой бы то ни было причине не удается подключиться к модулю PowerShell, можно вручную скачать и установить модуль AZ. Аппликатионмонитор.

### <a name="manually-download-the-latest-nupkg-file"></a>Скачайте последнюю версию файла nupkg вручную

1. Перейдите к https://www.powershellgallery.com/packages/Az.ApplicationMonitor.
2. Выберите последнюю версию файла в таблице **журнала версий** .
3. В разделе **Параметры установки** выберите **Загрузка вручную**.

### <a name="option-1-install-into-a-powershell-modules-directory"></a>Вариант 1. Установка в каталог модулей PowerShell
Установите модуль PowerShell, загруженный вручную, в каталог PowerShell, чтобы он был обнаружен сеансами PowerShell.
Дополнительные сведения см. [в разделе Установка модуля PowerShell](/powershell/scripting/developer/module/installing-a-powershell-module).


#### <a name="unzip-nupkg-as-a-zip-file-by-using-expand-archive-v1010"></a>Распакуйте nupkg как ZIP-файл с помощью Expand-Archive (v 1.0.1.0)

- Описание: Базовая версия Microsoft. PowerShell. Archive (v 1.0.1.0) не может распаковать файлы nupkg. Переименуйте файл с расширением ZIP.
- Ссылка: [разверните-Archive](/powershell/module/microsoft.powershell.archive/expand-archive).
- Команда:

    ```console
    $pathToNupkg = "C:\az.applicationmonitor.0.3.0-alpha.nupkg"
    $pathToZip = ([io.path]::ChangeExtension($pathToNupkg, "zip"))
    $pathToNupkg | rename-item -newname $pathToZip
    $pathInstalledModule = "$Env:ProgramFiles\WindowsPowerShell\Modules\az.applicationmonitor"
    Expand-Archive -LiteralPath $pathToZip -DestinationPath $pathInstalledModule
    ```

#### <a name="unzip-nupkg-by-using-expand-archive-v1100"></a>Распакуйте nupkg с помощью Expand-Archive (v 1.1.0.0)

- Описание: Используйте текущую версию Expand-Archive для распаковки файлов nupkg без изменения расширения.
- Ссылка: [разверните узел Archive](/powershell/module/microsoft.powershell.archive/expand-archive) и [Microsoft. PowerShell. Archive](https://www.powershellgallery.com/packages/Microsoft.PowerShell.Archive/1.1.0.0).
- Команда:

    ```console
    $pathToNupkg = "C:\az.applicationmonitor.0.2.1-alpha.nupkg"
    $pathInstalledModule = "$Env:ProgramFiles\WindowsPowerShell\Modules\az.applicationmonitor"
    Expand-Archive -LiteralPath $pathToNupkg -DestinationPath $pathInstalledModule
    ```

### <a name="option-2-unzip-and-import-nupkg-manually"></a>Вариант 2. Распакуйте и импортируйте nupkg вручную
Установите модуль PowerShell, загруженный вручную, в каталог PowerShell, чтобы он был обнаружен сеансами PowerShell.
Дополнительные сведения см. [в разделе Установка модуля PowerShell](/powershell/scripting/developer/module/installing-a-powershell-module).

Если вы устанавливаете модуль в любой другой каталог, вручную Импортируйте модуль с помощью [Import-Module](/powershell/module/microsoft.powershell.core/import-module).

> [!IMPORTANT] 
> Библиотеки DLL будут устанавливаться через относительные пути.
> Сохраните содержимое пакета в требуемом каталоге среды выполнения и убедитесь, что разрешения на доступ разрешают чтение, но не запись.

1. Измените расширение на ZIP и извлеките содержимое пакета в нужный каталог установки.
2. Найдите путь к файлу Az.ApplicationMonitor.psd1.
3. Запустите PowerShell от имени администратора с политикой выполнения с повышенными правами.
4. Загрузите модуль с помощью `Import-Module Az.ApplicationMonitor.psd1` команды.
    

## <a name="route-traffic-through-a-proxy"></a>Маршрутизация трафика через прокси-сервер

При мониторинге компьютера в частной интрасети необходимо маршрутизировать HTTP-трафик через прокси-сервер.

Команды PowerShell для загрузки и установки AZ. Аппликатионмонитор из коллекция PowerShell поддерживают `-Proxy` параметр.
При написании сценариев установки ознакомьтесь с приведенными выше инструкциями.

Пакету SDK для Application Insights потребуется отправить данные телеметрии приложения в корпорацию Майкрософт. Рекомендуется настроить параметры прокси-сервера для приложения в файле web.config. Дополнительные сведения см. в разделе [Application Insights FAQ: passthrough прокси](../faq.md#proxy-passthrough).


## <a name="enable-monitoring"></a>Включение мониторинга

Используйте `Enable-ApplicationInsightsMonitoring` команду, чтобы включить мониторинг.

Подробное описание использования этого командлета см. в [справочнике по API](./status-monitor-v2-api-reference.md#enable-applicationinsightsmonitoring) .



## <a name="next-steps"></a>Дальнейшие действия

 Просмотр телеметрии:

- [Изучите метрики](../essentials/metrics-charts.md) для мониторинга производительности и использования.
- [Поиск событий и журналов](./diagnostic-search.md) для диагностики проблем.
- [Используйте аналитику](../logs/log-query-overview.md) для более сложных запросов.
- [Создание панелей мониторинга](./overview-dashboard.md).

 Добавление данных телеметрии:

- [Создайте веб-тесты](monitor-web-app-availability.md), чтобы убедиться, что ваш сайт продолжает работать.
- [Добавьте данные телеметрии веб-клиента](./javascript.md) , чтобы просмотреть исключения из кода веб-страницы и включить вызовы трассировки.
- [Добавьте в код пакет SDK для Application Insights](./asp-net.md) , чтобы можно было вставить вызовы трассировки и журнала.

Другие действия с агентом Application Insights:

- Используйте наше справочное по для [устранения неполадок](status-monitor-v2-troubleshoot.md) агента Application Insights.


---
title: Настройка пользовательского контейнера
description: Узнайте, как настроить пользовательский контейнер в службе приложений Azure. В этой статье показаны наиболее распространенные задачи настройки.
ms.topic: article
ms.date: 02/23/2021
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 8083c3c0c88d904ccb3ec75ae69a699867bd0f25
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101704877"
---
# <a name="configure-a-custom-container-for-azure-app-service"></a>Настройка контейнера клиента в Службе приложений Azure

В этой статье показано, как настроить пользовательский контейнер для работы в службе приложений Azure.

::: zone pivot="container-windows"

Это краткое описание содержит основные понятия и инструкции для работы с контейнерами приложений Windows в службе приложений. Если вы никогда не использовали службу приложений Azure, сначала ознакомьтесь с кратким [руководством](tutorial-custom-container.md) по [пользовательскому контейнеру](quickstart-custom-container.md) и руководству.

::: zone-end

::: zone pivot="container-linux"

Это краткое описание содержит основные понятия и инструкции по работе с контейнерами приложений Linux в службе приложений. Если вы никогда не использовали службу приложений Azure, сначала ознакомьтесь с кратким [руководством](tutorial-custom-container.md) по [пользовательскому контейнеру](quickstart-custom-container.md) и руководству. Существует также краткое [руководство](tutorial-multi-container-app.md)по [многоконтейнерному приложению](quickstart-multi-container.md) .

::: zone-end

::: zone pivot="container-windows"

## <a name="supported-parent-images"></a>Поддерживаемые родительские образы

Для пользовательского образа Windows необходимо выбрать правильный [родительский образ (базовый образ)](https://docs.docker.com/develop/develop-images/baseimages/) для нужной платформы:

- Чтобы развернуть платформа .NET Framework приложения, используйте родительский образ, основанный на выпуске Windows Server Core [Servicing Channel (LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) . 
- Для развертывания приложений .NET Core Используйте Родительский образ, основанный на выпуске Windows Server Nano [Channel Servicing (SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel) . 

Скачивание родительского образа во время запуска приложения занимает некоторое время. Но вы можете ускорить запуск, используя один из следующих родительских образов, уже кэшированных в службе приложений Azure:

- [MCR.Microsoft.com/Windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore): 2004
- [MCR.Microsoft.com/Windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore): ltsc2019
- [MCR.Microsoft.com/DotNet/Framework/ASPNET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/): 4,8-windowsservercore-2004
- [MCR.Microsoft.com/DotNet/Framework/ASPNET](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/): 4,8-windowsservercore-ltsc2019
- [MCR.Microsoft.com/DotNet/Core/Runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/): 3,1-сервер-2004
- [MCR.Microsoft.com/DotNet/Core/Runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/): 3,1-сервер-1909
- [MCR.Microsoft.com/DotNet/Core/Runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/): 3,1-сервер-1903
- [MCR.Microsoft.com/DotNet/Core/Runtime](https://hub.docker.com/_/microsoft-dotnet-core-runtime/): 3,1-сервер-1809
- [MCR.Microsoft.com/DotNet/Core/ASPNET](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/): 3,1-сервер-2004
- [MCR.Microsoft.com/DotNet/Core/ASPNET](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/): 3,1-сервер-1909
- [MCR.Microsoft.com/DotNet/Core/ASPNET](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/): 3,1-сервер-1903
- [MCR.Microsoft.com/DotNet/Core/ASPNET](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/): 3,1-сервер-1809

::: zone-end

## <a name="change-the-docker-image-of-a-custom-container"></a>Изменение образа DOCKER пользовательского контейнера

Чтобы изменить существующее приложение-контейнер из текущего образа DOCKER на новый образ, используйте следующую команду:

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <docker-hub-repo>/<image>
```

## <a name="use-an-image-from-a-private-registry"></a>Использование образа из частного реестра

Чтобы использовать образ из частного реестра, например реестра контейнеров Azure, выполните следующую команду:

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name> --docker-registry-server-url <private-repo-url> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

Для *\<username>* и *\<password>* укажите учетные данные для входа в закрытую учетную запись реестра.

## <a name="i-dont-see-the-updated-container"></a>Я не вижу обновленный контейнер

Если изменить параметры контейнера DOCKER так, чтобы они указывали на новый контейнер, это может занять несколько минут, прежде чем приложение будет обслуживать HTTP-запросы из нового контейнера. Во время извлечения и запуска нового контейнера служба приложений продолжит обслуживать запросы из старого контейнера. При запуске нового контейнера и готовности к получению запросов служба приложений начинает отправлять запросы на нее.

## <a name="how-container-images-are-stored"></a>Как хранятся образы контейнеров

При первом запуске пользовательского образа DOCKER в службе приложений служба приложений выполняет `docker pull` и извлекает все слои изображений. Эти слои хранятся на диске, например при использовании DOCKER в локальной среде. При каждом перезапуске приложения служба приложений выполняет `docker pull` , но только извлекают слои, которые были изменены. Если изменений не было, служба приложений использует существующие слои на локальном диске.

Если приложение изменяет экземпляры вычислений по какой бы то ни было причине, например, при масштабировании ценовых категорий, служба приложений должна снова извлечь все слои. То же самое справедливо и при масштабировании для добавления дополнительных экземпляров. Существуют также редкие случаи, когда экземпляры приложения могут измениться без операции масштабирования.

## <a name="configure-port-number"></a>Настройка номера порта

По умолчанию служба приложений предполагает, что ваш пользовательский контейнер прослушивает порт 80. Если контейнер прослушивает другой порт, задайте `WEBSITES_PORT` параметр приложения в приложении службы приложений. Его можно задать с помощью [Cloud Shell](https://shell.azure.com). В Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_PORT=8000
```

В PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITES_PORT"="8000"}
```

В настоящее время служба приложений позволяет контейнеру предоставлять только один порт для HTTP-запросов. 

## <a name="configure-environment-variables"></a>Настройка переменных среды

Пользовательский контейнер может использовать переменные среды, которые необходимо предоставить извне. Их можно передать с помощью [Cloud Shell](https://shell.azure.com). В Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings DB_HOST="myownserver.mysql.database.azure.com"
```

В PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"DB_HOST"="myownserver.mysql.database.azure.com"}
```

При запуске приложения параметры приложения службы приложений вставляются в процесс как переменные среды автоматически. Вы можете проверить переменные среды контейнера с помощью URL-адреса `https://<app-name>.scm.azurewebsites.net/Env)` .

::: zone pivot="container-windows"
Для контейнеров на основе IIS или платформа .NET Framework (4,0 или выше) они внедряются в `System.ConfigurationManager` качестве параметров приложения .NET и строки подключения автоматически службой приложений. Для любого другого языка или платформы они предоставляются как переменные среды для процесса с одним из следующих соответствующих префиксов:

- `APPSETTING_`
- `SQLCONTR_`
- `MYSQLCONTR_`
- `SQLAZURECOSTR_`
- `POSTGRESQLCONTR_`
- `CUSTOMCONNSTR_`

::: zone-end

::: zone pivot="container-linux"

Этот метод работает как для приложений с одним контейнером, так и для приложений с несколькими контейнерами, где переменные среды указаны в файле *DOCKER-Compose. yml* .

::: zone-end

## <a name="use-persistent-shared-storage"></a>Использовать постоянное общее хранилище

::: zone pivot="container-windows"

Вы можете использовать каталог *к:\хоме* в файловой системе приложения для сохранения файлов между перезапусками и их совместного использования в экземплярах. В `C:\home` приложении предоставляется разрешение на доступ к постоянному хранилищу в приложении-контейнере.

Если постоянное хранилище отключено, записи в `C:\home` каталог не сохраняются. [Журналы узлов DOCKER и журналы контейнеров](#access-diagnostic-logs) сохраняются в постоянном общем хранилище по умолчанию, которое не подключено к контейнеру. Если постоянное хранилище включено, все операции записи в `C:\home` Каталог сохраняются, и доступ к ним может осуществляться всеми экземплярами масштабируемого приложения, и журнал доступен по адресу `C:\home\LogFiles` .

::: zone-end

::: zone pivot="container-linux"

Вы можете использовать каталог */Home* в файловой системе приложения для сохранения файлов между перезапусками и их совместного использования в экземплярах. В `/home` приложении предоставляется разрешение на доступ к постоянному хранилищу в приложении-контейнере.

Если постоянное хранилище отключено, записи в `/home` каталог не сохраняются при перезапуске приложения или в нескольких экземплярах. Единственным исключением является `/home/LogFiles` каталог, который используется для хранения журналов DOCKER и контейнеров. Если постоянное хранилище включено, все операции записи в `/home` Каталог сохраняются и доступны для всех экземпляров масштабируемого приложения.

::: zone-end

По умолчанию постоянное хранилище отключено и параметр не отображается в параметрах приложения. Чтобы включить его, задайте `WEBSITES_ENABLE_APP_SERVICE_STORAGE` параметр приложения с помощью [Cloud Shell](https://shell.azure.com). В Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=true
```

В PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITES_ENABLE_APP_SERVICE_STORAGE"=true}
```

> [!NOTE]
> Вы также можете [настроить постоянное хранилище](configure-connect-to-azure-storage.md).

## <a name="detect-https-session"></a>Обнаружение сеанса HTTPS

Служба приложений завершает протокол TLS/SSL на внешних интерфейсах. Это означает, что запросы TLS/SSL никогда не поступают в приложение. Это не обязательно и не должно реализовывать поддержку TLS/SSL в приложении. 

Внешние интерфейсы находятся в центрах обработки данных Azure. Если вы используете TLS/SSL с приложением, трафик через Интернет всегда будет безопасно зашифрован.

::: zone pivot="container-windows"

## <a name="customize-aspnet-machine-key-injection"></a>Настройка введения ключа компьютера ASP.NET

 Во время запуска контейнера автоматически созданные ключи вставляются в контейнер в качестве ключей компьютера для подпрограмм шифрования ASP.NET. [Эти ключи можно найти в контейнере](#connect-to-the-container) , выполнив поиск по следующим переменным среды: `MACHINEKEY_Decryption` , `MACHINEKEY_DecryptionKey` , `MACHINEKEY_ValidationKey` , `MACHINEKEY_Validation` . 

Новые ключи при каждой перезагрузке могут сбрасывать проверку подлинности ASP.NET Forms и состояние просмотра, если приложение зависит от них. Чтобы предотвратить автоматическое повторное создание ключей, [Задайте их вручную в качестве параметров приложения службы приложений](#configure-environment-variables). 

## <a name="connect-to-the-container"></a>Подключение к контейнеру

Вы можете подключиться к контейнеру Windows напрямую для диагностических задач, перейдя по адресу `https://<app-name>.scm.azurewebsites.net/DebugConsole` . Это работает следующим образом:

- Консоль отладки позволяет выполнять Интерактивные команды, такие как запуск сеансов PowerShell, Проверка разделов реестра и Навигация по всей файловой системе контейнера.
- Он работает отдельно от графического браузера над ним, в котором отображаются только файлы в [общем хранилище](#use-persistent-shared-storage).
- В масштабируемом приложении консоль отладки подключена к одному из экземпляров контейнера. Вы можете выбрать другой экземпляр из раскрывающегося списка **экземпляр** в верхнем меню.
- Любые изменения, вносимые в контейнер из консоли, *не* сохраняются при перезапуске приложения (за исключением изменений в общем хранилище), так как они не являются частью образа DOCKER. Чтобы сохранить изменения, такие как параметры реестра и установка программного обеспечения, сделайте их частью Dockerfile.

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

Служба приложений регистрирует действия, выполняемые узлом DOCKER, а также действия в контейнере. Журналы узла DOCKER (журналы платформы) поставляются по умолчанию, но журналы приложений или веб-сервера в контейнере необходимо включить вручную. Дополнительные сведения см. в разделе [Включение ведения журнала приложений](troubleshoot-diagnostic-logs.md#enable-application-logging-linuxcontainer) и [Включение ведения журнала веб-сервера](troubleshoot-diagnostic-logs.md#enable-web-server-logging). 

Есть несколько способов доступа к журналам docker:

- [На портале Azure](#in-azure-portal)
- [Из консоли KUDU](#from-the-kudu-console)
- [С помощью API KUDU](#with-the-kudu-api)
- [Отправка журналов в Azure Monitor](troubleshoot-diagnostic-logs.md#send-logs-to-azure-monitor-preview)

### <a name="in-azure-portal"></a>На портале Azure

Журналы DOCKER отображаются на портале на странице **Параметры контейнера** приложения. Журналы усекаются, но вы можете скачать все журналы, нажав кнопку **скачать**. 

### <a name="from-the-kudu-console"></a>Из консоли KUDU

Перейдите к `https://<app-name>.scm.azurewebsites.net/DebugConsole` папке **журнала** и щелкните ее, чтобы просмотреть отдельные файлы журналов. Чтобы скачать весь каталог **файл_журнала** , щелкните значок **скачивания** слева от имени каталога. Доступ к этой папке также можно получить с помощью FTP-клиента.

В окне терминала консоли доступ к папке по умолчанию невозможен, `C:\home\LogFiles` так как постоянное общее хранилище не включено. Чтобы включить такое поведение в консольном терминале, [включите постоянное общее хранилище](#use-persistent-shared-storage).

При попытке скачать журнал DOCKER, который используется в данный момент с помощью FTP-клиента, может возникнуть ошибка, вызванная блокировкой файла.

### <a name="with-the-kudu-api"></a>С помощью API KUDU

Перейдите непосредственно к, `https://<app-name>.scm.azurewebsites.net/api/logs/docker` чтобы просмотреть метаданные для журналов DOCKER. В списке может присутствовать несколько файлов журналов, и `href` свойство позволяет скачать файл журнала напрямую. 

Чтобы скачать все журналы вместе в одном ZIP-файле, Access `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip` .

## <a name="customize-container-memory"></a>Настройка памяти контейнера

По умолчанию все контейнеры Windows, развернутые в службе приложений Azure, имеют ограничение в 1 ГБ ОЗУ. Это значение можно изменить, предоставив `WEBSITE_MEMORY_LIMIT_MB` параметр приложения через [Cloud Shell](https://shell.azure.com). В Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITE_MEMORY_LIMIT_MB=2000
```

В PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITE_MEMORY_LIMIT_MB"=2000}
```

Значение определяется в МБ и должно быть меньше и равно общему объему физической памяти узла. Например, в плане службы приложений с 8 ГБ ОЗУ совокупная сумма `WEBSITE_MEMORY_LIMIT_MB` всех приложений не должна превышать 8 ГБ. Сведения о том, сколько памяти доступно для каждой ценовой категории, можно найти в статье [цены на службу приложений](https://azure.microsoft.com/pricing/details/app-service/windows/)в разделе **план контейнера Premium (Windows)** .

## <a name="customize-the-number-of-compute-cores"></a>Настройка количества ядер вычислений

По умолчанию контейнер Windows выполняется со всеми доступными ядрами для выбранной ценовой категории. Например, может потребоваться уменьшить количество ядер, используемых в промежуточном слоте. Чтобы уменьшить количество ядер, используемых контейнером, задайте `WEBSITE_CPU_CORES_LIMIT` для параметра приложения предпочтительное число ядер. Его можно задать с помощью [Cloud Shell](https://shell.azure.com). В Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --slot staging --settings WEBSITE_CPU_CORES_LIMIT=1
```

В PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"WEBSITE_CPU_CORES_LIMIT"=1}
```

> [!NOTE]
> Обновление параметра приложения активирует автоматический перезапуск, что приводит к минимальному простою. Для рабочего приложения рассмотрите возможность переключения его в промежуточный слот, измените параметр приложения в промежуточном слоте, а затем переключите его обратно в рабочую среду.

Проверьте скорректированное число, перейдя в консоль KUDU ( `https://<app-name>.scm.azurewebsites.net` ) и введя следующие команды с помощью PowerShell. Каждая команда выводит число.

```PowerShell
Get-ComputerInfo | ft CsNumberOfLogicalProcessors # Total number of enabled logical processors. Disabled processors are excluded.
Get-ComputerInfo | ft CsNumberOfProcessors # Number of physical processors.
```

Процессоры могут быть многоядерными или обработчиками технологии Hyper. Сведения о том, сколько ядер доступно для каждой ценовой категории, можно найти **в** разделе [цены на службу приложений](https://azure.microsoft.com/pricing/details/app-service/windows/).

## <a name="customize-health-ping-behavior"></a>Настройка поведения проверки связи для работоспособности

Служба приложений считает, что контейнер должен быть успешно запущен, когда контейнер запускается и отвечает на запрос HTTP ping. Запрос проверки работоспособности содержит заголовок `User-Agent= "App Service Hyper-V Container Availability Check"` . Если контейнер запускается, но не отвечает на запрос проверки связи через определенное время, служба приложений регистрирует в журнале DOCKER событие, сообщающее, что контейнер не запущен. 

Если приложение занимает много ресурсов, контейнер может не отвечать на запросы проверки связи HTTP. Чтобы управлять действиями при сбое проверки связи HTTP, задайте `CONTAINER_AVAILABILITY_CHECK_MODE` параметр приложения. Его можно задать с помощью [Cloud Shell](https://shell.azure.com). В Bash:

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings CONTAINER_AVAILABILITY_CHECK_MODE="ReportOnly"
```

В PowerShell:

```azurepowershell-interactive
Set-AzWebApp -ResourceGroupName <group-name> -Name <app-name> -AppSettings @{"CONTAINER_AVAILABILITY_CHECK_MODE"="ReportOnly"}
```

Возможные значения приведены в следующей таблице.

| Значение | Описания |
| - | - |
| **Чинить** | Перезапуск контейнера после трех последовательных проверок доступности |
| **ReportOnly** | Значение по умолчанию. Не перезапускайте контейнер, но отчет в журналах DOCKER для контейнера после трех последовательных проверок доступности. |
| **Выключено** | Не проверять доступность. |

## <a name="support-for-group-managed-service-accounts"></a>Поддержка групповых учетных записей управляемой службы

Групповые управляемые учетные записи служб (Gmsa) в настоящее время не поддерживаются в контейнерах Windows в службе приложений.

::: zone-end

::: zone pivot="container-linux"

## <a name="enable-ssh"></a>Включение SSH

SSH обеспечивает безопасный обмен данными между клиентом и контейнером. Чтобы пользовательский контейнер поддерживал SSH, его необходимо добавить в образ DOCKER.

> [!TIP]
> Все встроенные контейнеры Linux в службе приложений добавили инструкции SSH в свои репозитории образов. Чтобы увидеть, как он включен, можно выполнить приведенные ниже инструкции с [ репозиториемNode.js 10,14](https://github.com/Azure-App-Service/node/blob/master/10.14) . Конфигурация в встроенном образе Node.js немного отличается, но аналогична.

- Добавьте [файл sshd_config](https://man.openbsd.org/sshd_config) в репозиторий, как показано в следующем примере.

    ```
    Port            2222
    ListenAddress       0.0.0.0
    LoginGraceTime      180
    X11Forwarding       yes
    Ciphers aes128-cbc,3des-cbc,aes256-cbc,aes128-ctr,aes192-ctr,aes256-ctr
    MACs hmac-sha1,hmac-sha1-96
    StrictModes         yes
    SyslogFacility      DAEMON
    PasswordAuthentication  yes
    PermitEmptyPasswords    no
    PermitRootLogin     yes
    Subsystem sftp internal-sftp
    ```

    > [!NOTE]
    > Этот файл настраивает OpenSSH и должен включать следующие элементы:
    > - `Port` необходимо задать значение 2222.
    > - `Ciphers` должен содержать по крайней мере один элемент в этом списке: `aes128-cbc,3des-cbc,aes256-cbc`.
    > - `MACs` должен содержать по крайней мере один элемент в этом списке: `hmac-sha1,hmac-sha1-96`.

- В Dockerfile добавьте следующие команды:

    ```Dockerfile
    # Install OpenSSH and set the password for root to "Docker!". In this example, "apk add" is the install instruction for an Alpine Linux-based image.
    RUN apk add openssh \
         && echo "root:Docker!" | chpasswd 

    # Copy the sshd_config file to the /etc/ssh/ directory
    COPY sshd_config /etc/ssh/

    # Open port 2222 for SSH access
    EXPOSE 80 2222
    ```

    Эта конфигурация не допускает внешние подключения к контейнеру. Порт 2222 контейнера доступен только в сети моста частной виртуальной сети и недоступен для злоумышленника в Интернете.

- В скрипте запуска для контейнера запустите SSH-сервер.

    ```bash
    /usr/sbin/sshd
    ```

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

## <a name="configure-multi-container-apps"></a>Настройка приложений с несколькими контейнерами

- [Использование постоянного хранилища в Docker Compose](#use-persistent-storage-in-docker-compose)
- [Ограничения предварительной версии](#preview-limitations)
- [Параметры Docker Compose](#docker-compose-options)

### <a name="use-persistent-storage-in-docker-compose"></a>Использование постоянного хранилища в Docker Compose

Для правильной работы приложений с несколькими контейнерами, таких как WordPress, требуется постоянное хранилище. Чтобы включить его, конфигурация Docker Compose должна указывать на место хранения *за пределами* контейнера. Расположения хранилищ внутри контейнера не сохраняют изменения после перезапуска приложения.

Включите постоянное хранилище, задав `WEBSITES_ENABLE_APP_SERVICE_STORAGE` параметр приложения с помощью команды [AZ webapp config appSettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) в [Cloud Shell](https://shell.azure.com).

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

В файле *DOCKER-Compose. yml* сопоставьте `volumes` параметр с `${WEBAPP_STORAGE_HOME}` . 

`WEBAPP_STORAGE_HOME` — это переменная среды в Службе приложений, которая сопоставляется с постоянным хранилищем для вашего приложения. Пример:

```yaml
wordpress:
  image: wordpress:latest
  volumes:
  - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
  - ${WEBAPP_STORAGE_HOME}/phpmyadmin:/var/www/phpmyadmin
  - ${WEBAPP_STORAGE_HOME}/LogFiles:/var/log
```

### <a name="preview-limitations"></a>Ограничения предварительной версии

В настоящее время поддерживается Предварительная версия нескольких контейнеров. Следующие функции платформы службы приложений не поддерживаются:

- проверка подлинности/авторизация;
- управляемые удостоверения.
- CORS

### <a name="docker-compose-options"></a>Параметры Docker Compose

В следующих списках показаны поддерживаемые и неподдерживаемые параметры конфигурации Docker Compose.

#### <a name="supported-options"></a>Поддерживаемые варианты

- .
- entrypoint;
- среда
- Изображение
- ports;
- restart
- services;
- volumes.

#### <a name="unsupported-options"></a>Неподдерживаемые параметры

- build (недопустимо);
- depends_on (игнорируется);
- networks (игнорируется);
- secrets (игнорируется);
- порты, отличные от 80 и 8080 (игнорируются).

> [!NOTE]
> Любые другие параметры, не вызываемые явно, игнорируются в общедоступной предварительной версии.

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Миграция пользовательского программного обеспечения в службу приложений Azure с помощью пользовательского контейнера](tutorial-custom-container.md)

::: zone pivot="container-linux"

> [!div class="nextstepaction"]
> [Руководство. по приложению WordPress с несколькими контейнерами](tutorial-multi-container-app.md)

::: zone-end

Или см. раздел Дополнительные ресурсы:

[Загрузка сертификата в контейнерах Windows или Linux](configure-ssl-certificate-in-code.md#load-certificate-in-linuxwindows-containers)

---
title: Включение Azure Monitor для гибридной среды
description: В этой статье описывается, как включить аналитику VM для гибридной облачной среды, содержащей одну или несколько виртуальных машин.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: c3c8495b7355ee5d9ee8c28b4e0097a0080964d4
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102046645"
---
# <a name="enable-vm-insights-for-a-hybrid-virtual-machine"></a>Включение аналитики VM для гибридной виртуальной машины
В этой статье описывается, как включить службу "аналитика виртуальных машин" для виртуальной машины за пределами Azure, включая локальные и другие облачные среды.

> [!IMPORTANT]
> Рекомендуемый способ включения гибридных виртуальных машин — сначала включить службу " [дуга" для серверов Azure](../../azure-arc/servers/overview.md) , чтобы виртуальные машины можно было включить для получения аналитики виртуальных машин с помощью процессов, аналогичных виртуальным машинам Azure. В этой статье описывается, как подключить гибридные виртуальные машины, если вы решили не использовать Azure ARC.

## <a name="prerequisites"></a>Предварительные требования

- [Создание и Настройка рабочей области log Analytics](./vminsights-configure-workspace.md).
- Ознакомьтесь с [поддерживаемыми операционными системами](./vminsights-enable-overview.md#supported-operating-systems) , чтобы убедиться, что операционная система, которую вы включаете, поддерживается. 


## <a name="overview"></a>Обзор
Для виртуальных машин за пределами Azure требуется один и тот же агент Log Analytics и агент зависимостей, которые используются для виртуальных машин Azure. Так как вы не можете использовать расширения виртуальной машины для установки агентов, необходимо вручную установить их в операционной системе на виртуальной машине или установить их с помощью другого метода. 

Дополнительные сведения о развертывании агента Log Analytics см. в статье [Подключение компьютеров Windows к Azure Monitor](../agents/agent-windows.md) или [Подключение компьютеров Linux к Azure Monitor](../agents/agent-linux.md) . Сведения об агенте зависимостей приведены в этой статье. 

## <a name="firewall-requirements"></a>Требования к брандмауэру
Требования к брандмауэру для агента Log Analytics приведены в [обзоре log Analytics Agent](../agents/log-analytics-agent.md#network-requirements). Агент зависимостей "Map Insights" не передает никаких данных и не требует каких бы то ни было изменений в брандмауэрах или портах. Данные карт всегда передаются агентом Log Analytics в службу Azure Monitor, либо напрямую, либо через [шлюз Operations Management Suite](../../azure-monitor/agents/gateway.md) , если политики безопасности ИТ не разрешают компьютерам сети подключаться к Интернету.


## <a name="dependency-agent"></a>Агент зависимостей

>[!NOTE]
>Следующие сведения, описанные в этом разделе, также применимы к [решению сопоставление служб](./service-map.md).  

Вы можете скачать агент зависимостей из следующих расположений:

| Файл | Операционная система | Version | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.10.7.12710 | CA29CC328F991D7301FD0360F4F56DF78275545BB8CDA853679899CA885E96F0  |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.10.7.12710 | 98380DBEB2E2A5848F2202BC22422C68B20B62090C1BFC1DECAB37ED5451ED8C |


## <a name="install-the-dependency-agent-on-windows"></a>Установка Dependency Agent на Windows

Dependency Agent можно вручную установить на компьютерах Windows с помощью `InstallDependencyAgent-Windows.exe`. Если запустить этот исполняемый файл без параметров, запустится мастер для установки агента в интерактивном режиме. Для установки или удаления агента требуются права *администратора* в гостевой ОС.

В приведенной ниже таблице выделены параметры, поддерживаемые при установке агента из командной строки.

| Параметр | Description |
|:--|:--|
| /? | Возвращает список параметров командной строки. |
| /S | Автоматическая установка без взаимодействия с пользователем. |

Например, чтобы запустить программу установки с `/?` параметром, введите **InstallDependencyAgent-Windows.exe/?**.

Файлы для Windows Dependency Agent по умолчанию устанавливаются в папку *C:\Program Files\Microsoft Dependency Agent*. Если не удается запустить агент зависимостей после завершения установки, просмотрите подробные сведения об ошибке в журналах. Каталогом журналов является *%Programfiles%\Microsoft Dependency Agent\logs*.

### <a name="powershell-script"></a>Сценарий PowerShell
Чтобы скачать и установить агент, используйте следующий пример скрипта PowerShell:

```powershell
Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S
```


## <a name="install-the-dependency-agent-on-linux"></a>Установка Dependency Agent в Linux

Dependency Agent устанавливается на серверах Linux с помощью файла *InstallDependencyAgent-Linux64.bin*. Это сценарий оболочки с самоизвлекающимся двоичным файлом. Вы можете запустить этот файл с помощью `sh` или добавить разрешения на выполнение в сам файл.

>[!NOTE]
> Для установки или настройки агента требуется доступ с правами привилегированного пользователя.
>

| Параметр | Описание |
|:--|:--|
| -help | Получает список параметров командной строки. |
| -S | Выполняет автоматическую установку без вывода сообщений для пользователя. |
| --check | Проверка разрешений и операционной системы без установки агента. |

Например, чтобы запустить программу установки с `-help` параметром, введите **InstallDependencyAgent-Linux64. bin-Help**. Установите агент зависимостей Linux в качестве привилегированного пользователя, выполнив команду `sh InstallDependencyAgent-Linux64.bin` .

Если Dependency Agent не запускается, просмотрите подробные сведения об ошибке в записях журналов. В агентах Linux каталог журнала — *расположении/var/opt/Microsoft/dependency-Agent/log*.

Файлы для Dependency Agent размещаются в следующих каталогах:

| Файлы | Расположение |
|:--|:--|
| Основные файлы | /opt/microsoft/dependency-agent |
| файлы журналов. | /var/opt/microsoft/dependency-agent/log |
| Файлы конфигурации | /etc/opt/microsoft/dependency-agent/config |
| Исполняемые файлы службы | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| Двоичные файлы хранилища | /var/opt/microsoft/dependency-agent/storage |

### <a name="shell-script"></a>Сценарий оболочки 
Используйте следующий пример скрипта оболочки для загрузки и установки агента:

```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
sudo sh InstallDependencyAgent-Linux64.bin -s
```

## <a name="desired-state-configuration"></a>Настройка требуемого состояния (DSC)

Чтобы развернуть Dependency Agent с помощью Desired State Configuration, можно использовать модуль xPSDesiredStateConfiguration со следующим примером кода.

```powershell
configuration VMInsights {

    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    $DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"

    Node localhost
    {
        # Download and install the Dependency agent
        xRemoteFile DAPackage
        {
            Uri = "https://aka.ms/dependencyagentwindows"
            DestinationPath = $DAPackageLocalPath
        }

        xPackage DA
        {
            Ensure="Present"
            Name = "Dependency Agent"
            Path = $DAPackageLocalPath
            Arguments = '/S'
            ProductId = ""
            InstalledCheckRegKey = "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"
            InstalledCheckRegValueName = "DisplayName"
            InstalledCheckRegValueData = "Dependency Agent"
            DependsOn = "[xRemoteFile]DAPackage"
        }
    }
}
```



## <a name="troubleshooting"></a>Устранение неполадок

### <a name="vm-doesnt-appear-on-the-map"></a>Виртуальная машина не отображается на карте

Если установка агента зависимостей прошла удачно, но компьютер не отображается на карте, определите проблему, выполнив следующие действия.

1. Успешно ли установлен Dependency Agent? Для этого проверьте, установлена и запущена ли служба.

    **Windows**: Найдите службу с именем "Microsoft dependency Agent".

    **Linux**: Найдите выполняющийся процесс «Microsoft-dependency-Agent».

2. Вы в ценовой категории " [бесплатный" log Analytics](../insights/solutions.md)? Бесплатный план позволяет создавать до пяти уникальных компьютеров. Все последующие компьютеры не будут отображаться на карте, даже если предыдущие пять больше не отправляют данные.

3. Отправляет ли компьютер журнал и данные о производительности в Azure Monitor журналы? Выполните следующий запрос к компьютеру:

    ```Kusto
    Usage | where Computer == "computer-name" | summarize sum(Quantity), any(QuantityUnit) by DataType
    ```

    Возвращает ли он один или несколько результатов? Это последние данные? Если это так, то агент Log Analytics работает правильно и взаимодействует со службой. Если это не так, проверьте агент на сервере. Дополнительные сведения см. в разделах [Устранение неполадок ориентирования Management Suite операций](../agents/agent-windows-troubleshoot.md) или [Устранение неполадок с агентом Azure Log Analytics для Linux](../agents/agent-linux-troubleshoot.md).

#### <a name="computer-appears-on-the-map-but-has-no-processes"></a>Компьютер отображается на карте, но не содержит процессов

Если сервер отображается на карте, но у него нет данных процесса или соединения, это означает, что агент зависимостей установлен и работает, но драйвер ядра не был загружен.

Проверьте файл C:\Program Files\Microsoft Dependency Agent\logs\wrapper.log (Windows) или /var/opt/microsoft/dependency-agent/log/service.log (Linux). В последних строках файла должно быть указано, почему не удалось загрузить ядро. Например, если вы обновили ядро, оно может не поддерживаться в Linux.


## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда наблюдение включено для виртуальных машин, эти сведения доступны для анализа с помощью VM Insights.

- Чтобы просмотреть зависимости обнаруженных приложений, см. статью [Просмотр схемы VM Insights](vminsights-maps.md).

- Чтобы определить узкие места и общее использование с производительностью виртуальной машины, см. статью [Просмотр производительности виртуальной машины Azure](vminsights-performance.md).
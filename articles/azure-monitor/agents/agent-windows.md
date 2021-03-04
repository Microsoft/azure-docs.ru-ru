---
title: Установка агента Log Analytics на компьютерах Windows
description: В этой статье объясняется, как с помощью агента Log Analytics для Windows подключить компьютеры, размещенные в облаках или в локальной среде, к Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/03/2020
ms.openlocfilehash: aec39b86f9651539028efce93ba6a88c3be75b0c
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102038316"
---
# <a name="install-log-analytics-agent-on-windows-computers"></a>Установка агента Log Analytics на компьютерах Windows
В этой статье содержатся сведения об установке агента Log Analytics на компьютерах Windows с помощью следующих методов.

* Установка вручную с помощью [мастера установки](#install-agent-using-setup-wizard) или [командной строки](#install-agent-using-command-line).
* [Настройка требуемого состояния службы автоматизации Azure (DSC)](#install-agent-using-dsc-in-azure-automation). 

>[!IMPORTANT]
> Методы установки, описанные в этой статье, обычно используются для виртуальных машин в локальной среде или в других облаках. Дополнительные параметры, которые можно использовать для виртуальных машин Azure, см. в статье [варианты установки](./log-analytics-agent.md#installation-options) .

> [!NOTE]
> Если необходимо настроить агент для передачи отчетов в несколько рабочих областей, это невозможно сделать во время начальной установки. Но вы можете обновить параметры с помощью панели управления или PowerShell, как описано в разделе [Добавление или удаление рабочей области](agent-manage.md#adding-or-removing-a-workspace).  

## <a name="supported-operating-systems"></a>Поддерживаемые операционные системы

Список версий Windows, поддерживаемых агентом Log Analytics, см. в разделе [Общие сведения об агентах Azure Monitor](agents-overview.md#supported-operating-systems) .

### <a name="sha-2-code-signing-support-requirement"></a>Требование о поддержке подписывания кода SHA-2 
Агент Windows начнет монопольное использование подписывания SHA-2 с 17 августа 2020 года. Это изменение повлияет на клиентов, использующих агент Log Analytics в устаревшей ОС в составе любой службы Azure (Azure Monitor, Служба автоматизации Azure, Управление обновлениями Azure, Отслеживание изменений Azure, Центр безопасности Azure, Azure Sentinel, Защитник Windows ATP). Это изменение не требует никаких действий клиента, если только агент не используется в устаревшей версии ОС (Windows 7, Windows Server 2008 R2 и Windows Server 2008). Клиенты, работающие в устаревшей версии ОС, должны выполнить такие действия на своих компьютерах до 17 августа 2020 года, или их агенты перестанут отправлять данные в свои рабочие области Log Analytics:

1. установить последний пакет обновления для своей ОС. Необходимые версии пакета обновления:
    - Windows 7 с пакетом обновления 1 (SP1)
    - Windows Server 2008 с пакетом обновления 2 (SP2)
    - Windows Server 2008 R2 с пакетом обновления 1 (SP1)

2. Установите обновления Windows для подписывания SHA-2 для вашей ОС, как описано в разделе [Требования для поддержки подписывания кода SHA-2 для Windows и WSUS (2019)](https://support.microsoft.com/help/4472027/2019-sha-2-code-signing-support-requirement-for-windows-and-wsus)
3. Обновите до последней версии агент Windows (версия 10.20.18029).
4. Рекомендуется настроить агент для [использования TLS 1.2](agent-windows.md#configure-agent-to-use-tls-12). 

## <a name="network-requirements"></a>Требования к сети
Требования к сети для агента Windows см. в разделе [Общие сведения об агенте log Analytics](./log-analytics-agent.md#network-requirements) .


   
## <a name="configure-agent-to-use-tls-12"></a>Настройка агента для использования TLS 1.2
Протокол [TLS 1,2](/windows-server/security/tls/tls-registry-settings#tls-12) обеспечивает безопасность передаваемых данных для обмена данными между агентом Windows и службой log Analytics. При установке в [операционной системе без включенного по умолчанию tls 1,2](../logs/data-security.md#sending-data-securely-using-tls-12)необходимо настроить TLS 1,2, выполнив приведенные ниже действия.

1. Найдите следующий подраздел реестра: **HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols**
2. Создайте подраздел в разделе **Protocols** для TLS 1.2 **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2**
3. Создайте подраздел **Client** в созданном ранее подразделе версии протокола TLS 1.2. Например, **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client**.
4. Создайте следующие значения DWORD в папке **HKLM\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client**:

    * **Enabled** [значение = 1];
    * **DisabledByDefault** [значение = 0].  

Настройте .NET Framework 4.6 или более поздней версии для поддержки функции безопасного шифрования, так как по умолчанию она отключена. [Устойчивое шифрование](/dotnet/framework/network-programming/tls#schusestrongcrypto) подразумевает использование более безопасных сетевых протоколов, таких как TLS 1.2, а также блокировку протоколов, которые не являются безопасными. 

1. Найдите следующий подраздел реестра: **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\\.NETFramework\v4.0.30319**.  
2. Создайте параметр DWORD **SchUseStrongCrypto** в этом подразделе со значением **1**.  
3. Найдите следующий подраздел реестра: **HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\\.NETFramework\v4.0.30319**.  
4. Создайте параметр DWORD **SchUseStrongCrypto** в этом подразделе со значением **1**. 
5. Перезагрузите систему, чтобы параметры вступили в силу. 

## <a name="install-agent-using-setup-wizard"></a>Установка агента с помощью мастера установки
Ниже приведены инструкции по установке и настройке агента Log Analytics в Azure и облаке Azure для государственных организаций с помощью мастера установки для агента на компьютере. Сведения о настройке агента для отправки отчетов в группу управления System Center Operations Manager см. в разделе [Развертывание агента Operations Manager с помощью мастера установки агента](/system-center/scom/manage-deploy-windows-agent-manually#to-deploy-the-operations-manager-agent-with-the-agent-setup-wizard).

1. В рабочей области Log Analytics на ранее открытой странице **Серверы Windows** выберите соответствующую версию агента для скачивания (**Download Windows Agent** (Скачать агент для Windows)) в зависимости от архитектуры процессора, на котором выполняется операционная система Windows.   
2. Запустите программу установки, чтобы установить агент на компьютере.
2. На странице **приветствия** нажмите кнопку **Далее**.
3. На странице **Условия лицензии** прочтите лицензию и нажмите кнопку **Принимаю**.
4. На странице **Папка назначения** измените или оставьте папку установки по умолчанию и нажмите кнопку **Далее**.
5. На странице **Параметры установки агента** выберите подключение агента к Azure Log Analytics и нажмите **Далее**.   
6. На странице **Azure Log Analytics** выполните следующее.
   1. Вставьте **идентификатор рабочей области** и **ключ рабочей области (первичный ключ)** , скопированные ранее.  Если компьютер должен передавать данные в рабочую область Log Analytics в облаке Azure для государственных организаций, выберите **Azure для государственных организаций США** из раскрывающегося списка **Облако Azure**.  
   2. Если компьютер должен обмениваться данными со службой Log Analytics через прокси-сервер, щелкните **Дополнительно** и укажите URL-адрес и номер порта прокси-сервера.  Если для доступа к прокси-серверу требуется аутентификация, введите имя пользователя и пароль для аутентификации на прокси-сервере, затем нажмите кнопку **Далее**.  
7. Нажмите кнопку **Далее** после завершения ввода необходимых параметров конфигурации.<br><br> ![Вставка идентификатора рабочей области и первичного ключа](media/agent-windows/log-analytics-mma-setup-laworkspace.png)<br><br>
8. На странице **Готовность к установке** просмотрите выбранные параметры и нажмите кнопку **Установить**.
9. На странице **Настройка успешно завершена** нажмите кнопку **Готово**.

После завершения установки на **панели управления** появится **Microsoft Monitoring Agent**. Чтобы убедиться, что он отправляет отчеты в Log Analytics, ознакомьтесь с разделом [Проверка подключения агента к Log Analytics](#verify-agent-connectivity-to-azure-monitor). 

## <a name="install-agent-using-command-line"></a>Установка агента с помощью командной строки
Скачанный файл агента — это автономный пакет установки.  В нем содержатся программа установки агента и вспомогательные файлы. Чтобы установить агент с помощью командной строки, содержимое этого пакета нужно извлечь. Ниже описано, как это сделать.    

>[!NOTE]
>Для обновления агента необходимо использовать API сценариев службы Log Analytics. Дополнительные сведения см. в статье [Обслуживание агента Log Analytics для Windows и Linux и управление им](agent-manage.md).

В следующей таблице описаны определенные параметры, поддерживаемые программой установки агента, а также при развертывании с помощью Automation DSC.

|Параметры MMA                   |Примечания         |
|---------------------------------------|--------------|
| NOAPM=1                               | Необязательный параметр. Установка агента без мониторинга производительности приложений .NET.|   
|ADD_OPINSIGHTS_WORKSPACE               | 1 — настройка агента для передачи отчетов в рабочую область.                |
|OPINSIGHTS_WORKSPACE_ID                | Идентификатор добавляемой рабочей области (GUID).                    |
|OPINSIGHTS_WORKSPACE_KEY               | Ключ рабочей области, используемый для первоначальной аутентификации в рабочей области. |
|OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE  | Укажите облачную среду, в которой находится рабочая область. <br> 0 — коммерческое облако Azure (по умолчанию). <br> 1 — Azure для государственных организаций. |
|OPINSIGHTS_PROXY_URL               | Универсальный код ресурса (URI) для прокси-сервера. |
|OPINSIGHTS_PROXY_USERNAME               | Имя пользователя для доступа к прокси-серверу после аутентификации. |
|OPINSIGHTS_PROXY_PASSWORD               | Пароль для доступа к прокси-серверу после аутентификации. |

1. Чтобы извлечь файлы установки агента из командной строки с повышенными привилегиями, запустите файл `MMASetup-<platform>.exe /c`. После этого вы сможете выбрать путь для извлечения файлов.  Кроме того, путь можно указать, передав аргументы `MMASetup-<platform>.exe /c /t:<Full Path>`.  
2. Чтобы автоматически установить агент и настроить его для отправки отчетов в рабочую область коммерческого облака Azure, из папки, в которую вы извлекли файлы установки, выполните следующую команду: 
   
     ```dos
    setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<your workspace ID>" OPINSIGHTS_WORKSPACE_KEY="<your workspace key>" AcceptEndUserLicenseAgreement=1
    ```

   Чтобы настроить агент для отправки отчетов в облако Azure для US Gov организаций, выполните следующую команду: 

     ```dos
    setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=1 OPINSIGHTS_WORKSPACE_ID="<your workspace ID>" OPINSIGHTS_WORKSPACE_KEY="<your workspace key>" AcceptEndUserLicenseAgreement=1
    ```
    >[!NOTE]
    >Строковые значения для параметров *OPINSIGHTS_WORKSPACE_ID* и *OPINSIGHTS_WORKSPACE_KEY* должны быть заключены в двойные кавычки, чтобы указать установщику Windows интерпретировать их как допустимые параметры для пакета. 

## <a name="install-agent-using-dsc-in-azure-automation"></a>Установка агента с помощью DSC в службе автоматизации Azure

Вы можете использовать следующий пример сценария для установки агента с помощью DSC службы автоматизации Azure.   Если у вас нет учетной записи автоматизации, просмотрите статью [Начало работы со службой автоматизации Azure](../../automation/index.yml), чтобы ознакомиться с необходимыми требованиями и шагами для ее создания перед использованием Automation DSC.  Если вы не знакомы с Automation DSC, просмотрите статью [Приступая к работе с DSC службы автоматизации Azure](../../automation/automation-dsc-getting-started.md).

В указанном ниже примере выполняется установка 64-разрядного агента, идентифицируемого значением `URI`. Вы также можете использовать 32-разрядную версию, заменив значение универсального кода ресурса (URI). URI для обеих версий:

- Агент 64-разрядной версии Windows — https://go.microsoft.com/fwlink/?LinkId=828603.
- Агент 32-разрядной версии Windows — https://go.microsoft.com/fwlink/?LinkId=828604.


>[!NOTE]
>Эти примеры процедуры и сценария не поддерживают обновление агента, развернутого на компьютере Windows.

32- и 64-разрядные версии пакета агента имеют разные коды продукта. Новые выпущенные версии также имеют уникальное значение.  Код продукта — это глобальный уникальный идентификатор, который представляет собой основной идентификатор приложения или продукта. Он представляется свойством **ProductCode** установщика Windows.  Значение `ProductId` в скрипте **MMAgent.ps1** должно соответствовать коду продукта в пакете установщика 32- или 64-разрядного агента.

Чтобы извлечь код продукта из пакета установщика агента напрямую, вы можете воспользоваться Orca.exe (компонент пакета разработки программного обеспечения для Windows), который можно найти на странице [компонентов Windows SDK для разработчиков установщика Windows](/windows/win32/msi/platform-sdk-components-for-windows-installer-developers). Кроме того, вы можете использовать PowerShell с [примером сценария](https://www.scconfigmgr.com/2014/08/22/how-to-get-msi-file-information-with-powershell/), написанного специалистом с рангом Microsoft Valuable Professional (MVP).  В обоих случаях сначала нужно извлечь файл **MOMagent.msi** из пакета установки MMASetup.  Эта процедура рассматривается выше в разделе [Установка агента с помощью командной строки](#install-agent-using-command-line).  

1. Импортируйте модуль DSC xPSDesiredStateConfiguration со страницы [https://www.powershellgallery.com/packages/xPSDesiredStateConfiguration](https://www.powershellgallery.com/packages/xPSDesiredStateConfiguration) в службу автоматизации Azure.  
2.    Создайте в службе автоматизации Azure ресурсы-контейнеры для переменных *OPSINSIGHTS_WS_ID* и *OPSINSIGHTS_WS_KEY*. В качестве значения параметра *OPSINSIGHTS_WS_ID* укажите идентификатор рабочей области Log Analytics, а для параметра *OPSINSIGHTS_WS_KEY* — первичный ключ рабочей области.
3.    Скопируйте скрипт и сохраните его как файл MMAgent.ps1.

```powershell
Configuration MMAgent
{
    $OIPackageLocalPath = "C:\Deploy\MMASetup-AMD64.exe"
    $OPSINSIGHTS_WS_ID = Get-AutomationVariable -Name "OPSINSIGHTS_WS_ID"
    $OPSINSIGHTS_WS_KEY = Get-AutomationVariable -Name "OPSINSIGHTS_WS_KEY"

    Import-DscResource -ModuleName xPSDesiredStateConfiguration
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node OMSnode {
        Service OIService
        {
            Name = "HealthService"
            State = "Running"
            DependsOn = "[Package]OI"
        }

        xRemoteFile OIPackage {
            Uri = "https://go.microsoft.com/fwlink/?LinkId=828603"
            DestinationPath = $OIPackageLocalPath
        }

        Package OI {
            Ensure = "Present"
            Path  = $OIPackageLocalPath
            Name = "Microsoft Monitoring Agent"
            ProductId = "8A7F2C51-4C7D-4BFD-9014-91D11F24AAE2"
            Arguments = '/C:"setup.exe /qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_ID=' + $OPSINSIGHTS_WS_ID + ' OPINSIGHTS_WORKSPACE_KEY=' + $OPSINSIGHTS_WS_KEY + ' AcceptEndUserLicenseAgreement=1"'
            DependsOn = "[xRemoteFile]OIPackage"
        }
    }
}

```

4. Обновите значение `ProductId` в скрипте с кодом продукта, извлеченном из последней версии установленного пакета агента, используя рекомендованные выше методы. 
5. [Переместите сценарий настройки MMAgent.ps1](../../automation/automation-dsc-getting-started.md#import-a-configuration-into-azure-automation) в учетную запись автоматизации. 
6. [Назначьте компьютер Windows или узел](../../automation/automation-dsc-getting-started.md#enable-an-azure-resource-manager-vm-for-management-with-state-configuration) конфигурации. В течение 15 минут узел проверит свою конфигурацию, а агент будет помещен в узел.

## <a name="verify-agent-connectivity-to-azure-monitor"></a>Проверьте подключение агента к Azure Monitor

По завершении установки агента вы можете проверить, успешно ли он подключен и отправляет ли он отчеты. Есть два способа.  

На **панели управления** компьютера найдите элемент **Microsoft Monitoring Agent**.  Выберите его, а затем на вкладке **Azure Log Analytics** должно появиться следующее сообщение от агента: **The Microsoft Monitoring Agent has successfully connected to the Microsoft Operations Management Suite service** (Microsoft Monitoring Agent успешно подключен к службе Microsoft Operations Management Suite).<br><br> ![Состояние подключения MMA к Log Analytics](media/agent-windows/log-analytics-mma-laworkspace-status.png)

Вы также можете выполнить простой запрос журнала на портале Azure.  

1. На портале Azure найдите и выберите область **Монитор**.
1. Выберите **Журналы** в меню.
1. На панели **Журналы** в поле запроса введите следующее:  

    ```
    Heartbeat 
    | where Category == "Direct Agent" 
    | where TimeGenerated > ago(30m)  
    ```

В возвращаемых результатах поиска должны отображаться записи пульса для компьютера, указывающие, что агент успешно подключен и отправляет отчеты в службу.

## <a name="cache-information"></a>Сведения о кэше

Данные из агента Log Analytics кэшируются на локальном компьютере в папке *C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State* перед отправкой на Azure Monitor. Агент пытается отправить каждые 20 секунд. В случае сбоя он будет ожидать экспоненциального увеличения продолжительности времени, пока не завершится успешно. Он ждет 30 секунд перед второй попыткой, 60 секунд до следующего, 120 секунд и так далее до 8,5 часов между повторными попытками до успешного подключения. Это время ожидания слегка изменено, чтобы не допустить одновременной попытки подключения всеми агентами. Самые старые данные удаляются при достижении максимального размера буфера.

Размер кэша по умолчанию составляет 50 МБ, но его можно настроить между минимум 5 МБ и максимумом 1,5 ГБ. Он хранится в разделе реестра *HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HealthService\Parameters\Persistence Cache Maximum*. Значение представляет число страниц с 8 КБ на страницу.


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о перенастройке, обновлении или удалении агента из виртуальной машины см. в статье [Обслуживание агента Log Analytics для Windows и Linux и управление им](agent-manage.md).

- Если во время установки агента или управления им возникают проблемы, см. статью об [устранении неполадок с агентом Windows](agent-windows-troubleshoot.md).
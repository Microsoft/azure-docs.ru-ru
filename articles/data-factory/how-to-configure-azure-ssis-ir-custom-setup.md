---
title: Настройка среды выполнения интеграции Azure Integration Services
description: В этой статье описано, как использовать интерфейс пользовательской установки для среды выполнения интеграции Azure-SSIS и установить дополнительные компоненты или изменить параметры
ms.service: data-factory
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.custom: seo-lt-2019
ms.date: 11/06/2020
ms.openlocfilehash: b1b3a34ac495936ed92e29353a41efb8c462768f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100387739"
---
# <a name="customize-the-setup-for-an-azure-ssis-integration-runtime"></a>Настройка среды выполнения интеграции Azure Integration Services

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Вы можете настроить службу Azure-SQL Server Integration Services (SSIS) Integration Runtime (IR) в фабрике данных Azure (ADF) с помощью настраиваемых настроек. Они позволяют добавлять собственные шаги во время подготовки или перенастройки Azure-SSIS IR. 

С помощью настраиваемых настроек можно изменить операционную конфигурацию по умолчанию или среду Azure-SSIS IR. Например, для запуска дополнительных служб Windows, сохранения учетных данных доступа для файловых ресурсов или использования надежного шифрования или более защищенного сетевого протокола (TLS 1,2). Также можно установить дополнительные компоненты, такие как сборки, драйверы или расширения, на каждом узле Azure-SSIS IR. Они могут быть настраиваемыми, открытыми или сторонними компонентами. Дополнительные сведения о встроенных и предварительно установленных компонентах см. в статье [Встроенные и предварительно установленные компоненты в Azure-SSIS IR](./built-in-preinstalled-components-ssis-integration-runtime.md).

Вы можете выполнять пользовательскую установку в Azure-SSIS IR одним из двух способов: 
* **Стандартная пользовательская установка со скриптами**: подготовьте скрипт и связанные файлы, а затем отправьте их в контейнер больших двоичных объектов в учетной записи хранения Azure. После этого для установки или перенастройки Azure-SSIS IR необходимо предоставить универсальный код ресурса (URI) SAS для контейнера. Затем каждый узел Azure-SSIS IR скачивает скрипт и связанные с ним файлы из контейнера и выполняет пользовательскую установку с более высоким уровнем разрешений. После завершения пользовательской установки каждый узел передает стандартный поток вывода выполнения и другие журналы в контейнер.
* **Быстрая пользовательская установка без скриптов**: запустите некоторые распространенные системные конфигурации и команды Windows или установите некоторые популярные или рекомендуемые дополнительные компоненты без использования скриптов.

Можно установить бесплатные (нелицензированные) и платные (лицензированные) компоненты со стандартными и Экспресс-пользовательскими настройками. Если вы являетесь независимым поставщиком программного обеспечения (ISV), см. статью [Разработка платных или лицензированных компонентов для Azure-SSIS IR](how-to-develop-azure-ssis-ir-licensed-components.md).

> [!IMPORTANT]
> Чтобы воспользоваться преимуществами будущих улучшений, мы рекомендуем использовать серию узлов версии 3 или более поздней для Azure-SSIS IR с помощью пользовательской установки.

## <a name="current-limitations"></a>Текущие ограничения

Следующие ограничения применяются только к стандартным пользовательским установкам:

- Если вы хотите использовать файл *gacutil.exe* в своем скрипте, чтобы установить сборки в глобальном кэше сборок, необходимо предоставить *gacutil.exe* как часть пользовательской установки. Кроме того, вы можете использовать копию, указанную в папке *Sample* в *общедоступном контейнере предварительной версии* , см. раздел **стандартные примеры выборочной установки** ниже.

- Если в скрипт нужно добавить ссылку на вложенную папку, не используйте *msiexec.exe*. Эта команда не поддерживает нотацию `.\` для ссылки на корневую папку. Используйте команду `msiexec /i "MySubfolder\MyInstallerx64.msi" ...` вместо `msiexec /i ".\MySubfolder\MyInstallerx64.msi" ...`.

- В Azure-SSIS IR в настоящее время не поддерживаются административные общие папки или скрытые сетевые ресурсы, которые автоматически создаются Windows.

- Драйвер ODBC для IBM iSeries Access не поддерживается в Azure-SSIS IR. Вы можете столкнуться с ошибками установки во время пользовательской установки. Если это случится, обратитесь за помощью в службу поддержки IBM.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы настроить Azure-SSIS IR, понадобятся:

- [Подписка Azure](https://azure.microsoft.com/)

- [Подготовка Azure SSIS IR.](./tutorial-deploy-ssis-packages-azure.md)

- [Учетная запись хранения Azure.](https://azure.microsoft.com/services/storage/) Не требуется для быстрой пользовательской установки. Чтобы выполнить стандартную пользовательскую установку, отправьте и сохраните свой скрипт пользовательской установки и связанные с ним файлы в контейнер больших двоичных объектов. Процесс пользовательской установки отправляет свои журналы выполнения в тот же контейнер больших двоичных объектов.

## <a name="instructions"></a>Instructions

Вы можете подготавливать или перенастраивать Azure-SSIS IR с помощью настраиваемых настроек в пользовательском интерфейсе ADF. Если вы хотите сделать то же самое с помощью PowerShell, скачайте и установите [Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="standard-custom-setup"></a>Стандартная Настраиваемая установка

Чтобы подготавливать или перенастроить Azure-SSIS IR со стандартными пользовательскими настройками в пользовательском интерфейсе ADF, выполните следующие действия.

1. Подготовьте свой собственный скрипт пользовательской установки и связанные с ним файлы (например, файлы BAT, CMD, EXE, DLL, MSI или PS1).

   * Необходимо иметь файл скрипта с именем *main.cmd*, который является точкой входа в пользовательской установке.  
   * Чтобы обеспечить возможность автоматического выполнения скрипта, сначала следует проверить его на локальном компьютере.  
   * Если необходимо, чтобы в контейнер отправлялись дополнительные журналы, генерируемые другими инструментами (например, *msiexec.exe*), укажите предопределенную переменную среды `CUSTOM_SETUP_SCRIPT_LOG_DIR` в качестве папки журнала в ваших скриптах (например, *msiexec /i xxx.msi /quiet /lv %CUSTOM_SETUP_SCRIPT_LOG_DIR%\install.log*).

1. Скачайте, установите и откройте [Обозреватель службы хранилища Azure](https://storageexplorer.com/).

   а. В узле **(Локальные и присоединенные)** щелкните правой кнопкой мыши **Учетные записи хранения**, а затем выберите **Подключиться к службе хранилища Azure**.

      ![Подключение к службе хранилища Azure](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image1.png)

   b. Выберите **Использовать имя и ключ учетной записи хранения**, а затем нажмите кнопку **Далее**.

      ![Использование имени и ключа учетной записи хранения](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image2.png)

   c. Введите имя и ключ учетной записи хранения Azure, нажмите кнопку **Далее**, а затем — **Подключить**.

      ![Предоставление имени и ключа учетной записи хранения](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image3.png)

   d. В разделе подключенной учетной записи хранения Azure щелкните правой кнопкой мыши **Контейнеры больших двоичных объектов**, выберите **Создать контейнер BLOB-объектов** и присвойте имя новому контейнеру.

      ![Создание контейнера BLOB-объектов](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image4.png)

   e. Выберите новый контейнер и передайте скрипт пользовательской установки и связанные с ним файлы. Убедитесь, что файл *main.cmd* отправлен в верхний уровень вашего контейнера, а не в любую другую папку. Контейнер должен содержать только необходимые файлы пользовательской установки, поэтому их загрузка в Azure-SSIS IR позже не займет много времени. Максимальная продолжительность пользовательской установки в настоящее время настроена на 45 минут, а затем истекает время ее ожидания. Сюда входит время скачивания всех файлов из контейнера и их установка в Azure-SSIS IR. Если программе установки требуется больше времени, создайте запрос в службу поддержки.

      ![Отправка файлов в контейнер больших двоичных объектов](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image5.png)

   f. Щелкните правой кнопкой мыши контейнер и выберите **Получить подписанный URL-адрес**.

      ![Получение подписанного URL-адреса для контейнера](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image6.png)

   g. Создайте URI SAS для своего контейнера с достаточно длительным сроком действия, а также разрешениями чтения, записи и вывода списка. URI SAS необходим для скачивания и запуска пользовательского скрипта установки и связанных с ним файлов. Это происходит каждый раз при повторном создании образа или перезапуске любого узла Azure-SSIS IR. Кроме того, необходимо разрешение на запись для отправки журналов выполнения установки.

      > [!IMPORTANT]
      > Убедитесь, что не истек срок действия URI SAS, а ресурсы пользовательской установки всегда доступны в течение всего жизненного цикла вашей Azure-SSIS IR — от создания до удаления, особенно в том случае, если вы регулярно останавливаете и запускаете вашу Azure-SSIS IR в течение этого периода.

      ![Создание подписанного URL-адреса для контейнера](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image7.png)

   h. Скопируйте и сохраните URI SAS контейнера.

      ![Копирование и сохранение подписанного URL-адреса](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image8.png)

1. Установите флажок **настроить Azure-SSIS Integration Runtime с дополнительными конфигурациями системы или установками компонентов** на странице **Дополнительные параметры** панели **настройки среды выполнения интеграции** . Затем введите URI SAS контейнера в текстовом поле **URI SAS контейнера пользовательской установки** .

   ![Расширенные параметры для пользовательских установок](./media/tutorial-create-azure-ssis-runtime-portal/advanced-settings-custom.png)

После завершения стандартной пользовательской установки и запуска Azure-SSIS IR можно найти все журналы настраиваемой установки в папке *Main. cmd. log* контейнера. Они включают стандартные выходные данные файла *Main. cmd* и другие журналы выполнения.

### <a name="express-custom-setup"></a>Быстрая Настраиваемая установка

Чтобы подготавливать или перенастроить Azure-SSIS IR с помощью Экспресс настраиваемых настроек в пользовательском интерфейсе ADF, выполните следующие действия.

1. Установите флажок **настроить Azure-SSIS Integration Runtime с дополнительными конфигурациями системы или установками компонентов** на странице **Дополнительные параметры** панели **настройки среды выполнения интеграции** . 

1. Выберите **создать** , чтобы открыть панель " **Добавить Экспресс-настройку** ", а затем выберите тип в раскрывающемся списке Экспресс-настраиваемый **тип установки** . В настоящее время мы предлагаем быстрые пользовательские настройки для выполнения команды cmdkey, добавления переменных среды, установки Azure PowerShell и установки лицензированных компонентов.

#### <a name="running-cmdkey-command"></a>Выполнение команды cmdkey

Если вы выбрали тип **команды Run cmdkey** для Экспресс настраиваемой установки, то можете выполнить команду Windows cmdkey на Azure-SSIS IR. Для этого введите имя целевого компьютера или имя домена, имя пользователя или учетной записи, а также пароль или ключ учетной записи в текстовые поля **/Add**, **/User** и **/Пасс** соответственно. Это позволит сохранить учетные данные доступа для серверов SQL Server, файловых ресурсов или файлов Azure на Azure-SSIS IR. Например, для доступа к службе файлов Azure можно ввести `YourAzureStorageAccountName.file.core.windows.net` , `azure\YourAzureStorageAccountName` и `YourAzureStorageAccountKey` для **/Add**, **/User** и **/Пасс** соответственно. Это похоже на выполнение команды Windows [cmdkey](/windows-server/administration/windows-commands/cmdkey) на локальном компьютере. Сейчас поддерживается только одна Экспресс-Настраиваемая установка для выполнения команды cmdkey. Чтобы выполнить несколько команд cmdkey, используйте стандартную пользовательскую установку.

#### <a name="adding-environment-variables"></a>Добавление переменных среды

Если выбрать тип **переменной среды** для настраиваемой установки Express, можно добавить переменную среды Windows в Azure-SSIS IR. Для этого введите имя и значение переменной среды в текстовые поля **имя переменной** и **значение переменной** , соответственно. Это позволит использовать переменную среды в пакетах, выполняемых на Azure-SSIS IR, например в компонентах или задачах скрипта. Это похоже на выполнение команды Windows [set](/windows-server/administration/windows-commands/set_1) на локальном компьютере.

#### <a name="installing-azure-powershell"></a>Установка Azure PowerShell

Если выбрать тип **установки Azure PowerShell** для экспресс – установки, можно установить модуль AZ PowerShell на Azure-SSIS IR. Для этого введите номер версии AZ Module (x. y. z), который вы хотите использовать, из [списка поддерживаемых](https://www.powershellgallery.com/packages/az). Это позволит выполнять командлеты и сценарии Azure PowerShell в пакетах для управления ресурсами Azure, например [Azure Analysis Services (AAS)](../analysis-services/analysis-services-powershell.md).

#### <a name="installing-licensed-components"></a>Установка лицензированных компонентов

Если вы выбрали тип **компонента установить лицензию** для экспресс-установки, то можете выбрать интегрированный компонент от наших партнеров ISV в раскрывающемся списке **Component Name (имя компонента** ):

   * При выборе компонента **фабрики задач SentryOne** можно установить набор компонентов [фабрики задач](https://www.sentryone.com/products/task-factory/high-performance-ssis-components) из SentryOne на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **2020.1.3**.

   * При выборе **OH22'S Хедда. Компонент ввода-вывода** можно установить [Хедда.](https://github.com/oh22is/HEDDA.IO/tree/master/SSIS-IR) Компонент качества или очистки данных ввода-вывода из oh22 на Azure-SSIS IR. Для этого необходимо приобрести свою службу заранее. Текущая интегрированная версия — **1.0.14**.

   * При выборе компонента **oh22's SQLPhonetics.NET** можно установить компонент качества данных [SQLPhonetics.NET](https://appsource.microsoft.com/product/web-apps/oh22.sqlphonetics-ssis) и соответствующий параметр из oh22 на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **1.0.45**.

   * Если вы выбрали компонент **Кингсвайсофт Integration Toolkit для служб** SSIS, можно установить набор соединителей [SSIS Integration Toolkit](https://www.kingswaysoft.com/products/ssis-integration-toolkit-for-microsoft-dynamics-365) для приложений CRM/ERP/Marketing/Collaboration, таких как Microsoft Dynamics, SharePoint, Project Server, Oracle/Salesforce Marketing Cloud и т. п. из кингсвайсофт на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **2020,1**.

   * При выборе компонента **пакета производительности SSIS кингсвайсофт** можно установить набор компонентов [пакета производительности SSIS](https://www.kingswaysoft.com/products/ssis-productivity-pack) из кингсвайсофт на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **20,1**.

   * Если выбрать компонент **Звлечь Сеобалд Software** , то можно установить [звлечь — это](https://theobald-software.com/en/xtract-is/) набор соединителей для систем SAP (ERP, s/4HANA, BW) из сеобалд Software на Azure-SSIS IR. Для этого перетащите & Отгрузите или отправьте файл лицензии продукта, который вы приобрели заранее, в поле ввода **файла лицензии** . Текущая интегрированная версия — **6.1.1.3**.

   * Если вы выбрали компонент **службы интеграции аекорсофт** , то можете установить набор соединителей [службы интеграции](https://www.aecorsoft.com/en/products/integrationservice) для систем SAP и Salesforce из аекорсофт на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **3.0.00**.

   * При выборе компонента **CDATA "Стандартный пакет служб SSIS** " можно установить набор [стандартных пакетов служб SSIS](https://www.cdata.com/kb/entries/ssis-adf-packages.rst#standard) для большинства популярных компонентов из раздела CDATA, например соединители Microsoft SharePoint, на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **19,7354**.

   * Если выбрать компонент **расширенного пакета служб SSIS CDATA** , можно установить [Расширенный набор пакетов служб SSIS](https://www.cdata.com/kb/entries/ssis-adf-packages.rst#extended) для всех компонентов из раздела CDATA, например Microsoft Dynamics 365 Business Central Connectors и другие компоненты в **стандартном пакете служб SSIS**, на Azure-SSIS IR. Для этого введите лицензионный ключ продукта, который вы приобрели заранее, в текстовом поле **лицензионный ключ** . Текущая интегрированная версия — **19,7354**. Из-за большого размера, чтобы избежать тайм-аута установки, убедитесь, что в Azure-SSIS IR имеется по крайней мере 4 ядра ЦП на узел.

Добавленные экспресс-установки будут отображаться на странице **Дополнительные параметры** . Чтобы удалить их, установите соответствующие флажки, а затем выберите **Удалить**.

### <a name="azure-powershell"></a>Azure PowerShell

Чтобы подготавливать или перенастроить Azure-SSIS IR с помощью настраиваемых настроек, используя Azure PowerShell, выполните следующие действия.

1. Если Azure-SSIS IR уже запущена или запущена, сначала прервите ее.

1. Затем можно добавить или удалить пользовательские настройки, выполнив `Set-AzDataFactoryV2IntegrationRuntime` командлет перед началом Azure-SSIS IR.

   ```powershell
   $ResourceGroupName = "[your Azure resource group name]"
   $DataFactoryName = "[your data factory name]"
   $AzureSSISName = "[your Azure-SSIS IR name]"
   # Custom setup info: Standard/express custom setups
   $SetupScriptContainerSasUri = "" # OPTIONAL to provide a SAS URI of blob container for standard custom setup where your script and its associated files are stored
   $ExpressCustomSetup = "[RunCmdkey|SetEnvironmentVariable|InstallAzurePowerShell|SentryOne.TaskFactory|oh22is.SQLPhonetics.NET|oh22is.HEDDA.IO|KingswaySoft.IntegrationToolkit|KingswaySoft.ProductivityPack|Theobald.XtractIS|AecorSoft.IntegrationService|CData.Standard|CData.Extended or leave it empty]" # OPTIONAL to configure an express custom setup without script

   # Add custom setup parameters if you use standard/express custom setups
   if(![string]::IsNullOrEmpty($SetupScriptContainerSasUri))
   {
       Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
           -DataFactoryName $DataFactoryName `
           -Name $AzureSSISName `
           -SetupScriptContainerSasUri $SetupScriptContainerSasUri
   }
   if(![string]::IsNullOrEmpty($ExpressCustomSetup))
   {
       if($ExpressCustomSetup -eq "RunCmdkey")
       {
           $addCmdkeyArgument = "YourFileShareServerName or YourAzureStorageAccountName.file.core.windows.net"
           $userCmdkeyArgument = "YourDomainName\YourUsername or azure\YourAzureStorageAccountName"
           $passCmdkeyArgument = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourPassword or YourAccessKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.CmdkeySetup($addCmdkeyArgument, $userCmdkeyArgument, $passCmdkeyArgument)
       }
       if($ExpressCustomSetup -eq "SetEnvironmentVariable")
       {
           $variableName = "YourVariableName"
           $variableValue = "YourVariableValue"
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.EnvironmentVariableSetup($variableName, $variableValue)
       }
       if($ExpressCustomSetup -eq "InstallAzurePowerShell")
       {
           $moduleVersion = "YourAzModuleVersion"
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.AzPowerShellSetup($moduleVersion)
       }
       if($ExpressCustomSetup -eq "SentryOne.TaskFactory")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "oh22is.SQLPhonetics.NET")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "oh22is.HEDDA.IO")
       {
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup)
       }
       if($ExpressCustomSetup -eq "KingswaySoft.IntegrationToolkit")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "KingswaySoft.ProductivityPack")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }    
       if($ExpressCustomSetup -eq "Theobald.XtractIS")
       {
           $jsonData = Get-Content -Raw -Path YourLicenseFile.json
           $jsonData = $jsonData -replace '\s',''
           $jsonData = $jsonData.replace('"','\"')
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString($jsonData)
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "AecorSoft.IntegrationService")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "CData.Standard")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }
       if($ExpressCustomSetup -eq "CData.Extended")
       {
           $licenseKey = New-Object Microsoft.Azure.Management.DataFactory.Models.SecureString("YourLicenseKey")
           $setup = New-Object Microsoft.Azure.Management.DataFactory.Models.ComponentSetup($ExpressCustomSetup, $licenseKey)
       }    
       # Create an array of one or more express custom setups
       $setups = New-Object System.Collections.ArrayList
       $setups.Add($setup)

       Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
           -DataFactoryName $DataFactoryName `
           -Name $AzureSSISName `
           -ExpressCustomSetup $setups
   }
   Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
       -DataFactoryName $DataFactoryName `
       -Name $AzureSSISName `
       -Force
   ```

### <a name="standard-custom-setup-samples"></a>Стандартные примеры выборочной установки

Для просмотра и повторного использования некоторых примеров стандартных пользовательских настроек выполните следующие действия.

1. Подключитесь к нашему общедоступному контейнеру предварительной версии с помощью Обозреватель службы хранилища Azure.

   а. В узле **(Local and Attached)** (Локальные и присоединенные) щелкните правой кнопкой мыши **Учетные записи хранения**, выберите пункт **Connect to Azure storage** (Подключиться к службе хранилища Azure), а затем — **Use a storage account name and key** (Использовать имя и ключ учетной записи хранения) и нажмите кнопку **Далее**.

      ![Подключение к хранилищу Azure с помощью подписанного URL-адреса](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image9.png)

   b. Выберите **использовать URI SAS** , а затем в текстовом поле **URI** введите следующий URI SAS:

      `https://ssisazurefileshare.blob.core.windows.net/publicpreview?sp=rl&st=2020-03-25T04:00:00Z&se=2025-03-25T04:00:00Z&sv=2019-02-02&sr=c&sig=WAD3DATezJjhBCO3ezrQ7TUZ8syEUxZZtGIhhP6Pt4I%3D`

      ![Предоставление подписанного URL-адреса для контейнера](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image10.png)

   c. Нажмите кнопку **Далее**, а затем — **Подключить**.

   d. В левой области выберите подключенный контейнер **publicpreview**, а затем дважды щелкните папку *CustomSetupScript*. Внутри этой папки находятся следующие элементы:

      * Папка *Sample*, которая содержит файл пользовательской установки для установки базовой задачи на каждом узле Azure-SSIS IR. Задача не делает ничего, только несколько секунд находится в спящем режиме. Папка также содержит папку *gacutil*, все содержимое которой (*gacutil.exe*, *gacutil.exe.config* и *1033\gacutlrc.dll*) можно скопировать в ваш контейнер.

      * Папка *UserScenarios*, содержащая несколько примеров конфигураций пользовательской установки из реальных ситуаций. Если вы хотите установить несколько образцов на Azure-SSIS IR, можно объединить свои файлы сценария пользовательской установки (*Main. cmd*) в одну и передать их со всеми связанными файлами в контейнер.

        ![Содержимое контейнера общедоступной предварительной версии](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image11.png)

   e. Дважды щелкните папку *UserScenarios*, чтобы найти следующие элементы:

      * Папка *.NET FRAMEWORK 3,5* , которая содержит пользовательский сценарий установки (*Main. cmd*) для установки более ранней версии платформа .NET Framework на каждом узле Azure-SSIS IR. Эта версия может потребоваться для некоторых пользовательских компонентов.

      * Папка *bcp* , которая содержит пользовательский сценарий установки (*Main. cmd*) для установки SQL Server служебных программ командной строки (*MsSqlCmdLnUtils.msi*) на каждом узле Azure-SSIS IR. Одной из этих служебных программ является программа копирования (*bcp*).

      * Папка *DNS-суффикса* , которая содержит настраиваемый сценарий установки (*Main. cmd*) для добавления собственного DNS-суффикса (например, *Test.com*) в любое неполное имя домена с одной меткой и преобразует его в полное доменное имя (FQDN), прежде чем использовать его в запросах DNS из Azure-SSIS IR.

      * Папка *Excel* , которая содержит пользовательский сценарий установки (*Main. cmd*) для установки некоторых сборок и библиотек C# на каждом узле Azure-SSIS IR. Их можно использовать в задачах скриптов для динамического чтения и записи файлов Excel. 
      
        Сначала скачайте [*ExcelDataReader.dll*](https://www.nuget.org/packages/ExcelDataReader/) и [*DocumentFormat.OpenXml.dll*](https://www.nuget.org/packages/DocumentFormat.OpenXml/), а затем отправьте их вместе с *main.cmd* в ваш контейнер. Кроме того, если вы просто хотите использовать стандартные соединители Excel (диспетчер соединений, источник и назначение), распространяемый пакет доступа, который их содержит, уже предустановлен на Azure-SSIS IR, поэтому пользовательская настройка не требуется.
      
      * Папка *MYSQL ODBC*, которая содержит скрипт пользовательской установки (*main.cmd*) для установки драйвера MySQL ODBC на каждом узле Azure-SSIS IR. Эта программа установки позволяет использовать соединители ODBC (диспетчер соединений, источник и назначение) для подключения к серверу MySQL. 
     
        Сначала [скачайте последние 64- и 32-разрядные версии установщиков драйвера ODBC для MySQL](https://dev.mysql.com/downloads/connector/odbc/) (например, *mysql-connector-odbc-8.0.13-winx64.msi* и *mysql-connector-odbc-8.0.13-win32.msi*), а затем отправьте их вместе с *main.cmd* в контейнер.

      * Папка *ORACLE ENTERPRISE*, которая содержит пользовательский скрипт установки (*main.cmd*) и файл конфигурации автоматической установки (*client.rsp*), позволяющие установить соединители Oracle и драйвер OCI на каждом узле Azure-SSIS IR Enterprise Edition. Эта программа установки позволяет использовать диспетчер соединений Oracle, источник и назначение для подключения к серверу Oracle. 
      
        Сначала скачайте Microsoft Connectors v 5.0 для Oracle (*AttunitySSISOraAdaptersSetup.msi* и *AttunitySSISOraAdaptersSetup64.msi*) из [центра загрузки Майкрософт](https://www.microsoft.com/en-us/download/details.aspx?id=55179) и последней версии клиента Oracle (например, *winx64_12102_client.zip*) из [Oracle](https://www.oracle.com/database/technologies/oracle19c-windows-downloads.html). Затем отправьте их вместе с *Main. cmd* и *Client. rsp* в контейнер. Если для подключения к Oracle используется TNS, необходимо также скачать *tnsnames. ORA*, изменить его и передать в контейнер. Таким образом, его можно скопировать в папку установки Oracle во время установки.

      * Папка *ORACLE STANDARD ADO.NET*, которая содержит скрипт пользовательской установки (*main.cmd*) для установки драйвера Oracle ODP.NET на каждом узле Azure-SSIS IR. Эта программа установки позволяет использовать диспетчер соединений ADO.NET, источник и назначение для подключения к серверу Oracle. 
      
        Сначала [скачайте последнюю версию драйвера Oracle ODP.NET](https://www.oracle.com/technetwork/database/windows/downloads/index-090165.html) (например, *ODP.NET_Managed_ODAC122cR1.zip*) из Oracle и затем передайте его вместе с *main.cmd* в свой контейнер.
       
      * *Стандартная папка ODBC Oracle* , которая содержит пользовательский сценарий установки (*Main. cmd*) для установки драйвера Oracle ODBC на каждом узле Azure-SSIS IR. Скрипт также настраивает имя источника данных (DSN). Эта программа установки позволяет использовать диспетчер соединений ODBC, источник и назначение или диспетчер соединений Power Query и источник с типом источника данных ODBC для подключения к серверу Oracle. 
      
        Сначала скачайте последнюю версию клиента Oracle Instant Client (пакет Basic или Basic Lite) и пакет ODBC, а затем отправьте их вместе с *main.cmd* в контейнер:
        * [Скачайте 64-разрядные пакеты](https://www.oracle.com/technetwork/topics/winx64soft-089540.html) (пакет Basic: *instantclient-basic-windows.x64-18.3.0.0.0dbru.zip*; пакет Basic Lite: *instantclient-basiclite-windows.x64-18.3.0.0.0dbru.zip*; пакет ODBC: *instantclient-odbc-windows.x64-18.3.0.0.0dbru.zip*) 
        * [Скачайте 32-разрядные пакеты](https://www.oracle.com/technetwork/topics/winsoft-085727.html) (пакет Basic: *instantclient-basic-nt-18.3.0.0.0dbru.zip*; пакет Basic Lite: *instantclient-basiclite-nt-18.3.0.0.0dbru.zip*; пакет ODBC: *instantclient-odbc-nt-18.3.0.0.0dbru.zip*)

      * Папка *ORACLE STANDARD OLEDB*, которая содержит скрипт пользовательской установки (*main.cmd*) для установки драйвера Oracle OLEDB на каждом узле Azure-SSIS IR. Эта программа установки позволяет использовать диспетчер соединений OLEDB, источник и назначение для подключения к серверу Oracle. 
     
        Сначала [скачайте последнюю версию драйвера Oracle OLEDB](https://www.oracle.com/partners/campaign/index-090165.html) (например, *ODAC122010Xcopy_x64.zip*) из Oracle, а затем отправьте его вместе с *main.cmd* в свой контейнер.

      * Папка *POSTGRESQL ODBC*, которая содержит скрипт пользовательской установки (*main.cmd*) для установки драйвера PostgreSQL ODBC на каждом узле Azure-SSIS IR. Эта программа установки позволяет использовать диспетчер соединений ODBC, источник и назначение для подключения к серверу PostgreSQL. 
     
        Сначала [скачайте последние 64- и 32-разрядные версии установщиков драйвера ODBC для PostgreSQL](https://www.postgresql.org/ftp/odbc/versions/msi/) (например, *psqlodbc_x64.msi* и *psqlodbc_x86.msi*), а затем отправьте их вместе с *main.cmd* в ваш контейнер.

      * Папка *SAP BW*, которая содержит скрипт пользовательской установки (*main.cmd*), чтобы установить сборку соединителя SAP .NET (*librfc32.dll*) на каждом узле среды выполнения интеграции Azure-SSIS IR Enterprise Edition. Эта программа установки позволяет использовать диспетчер соединений SAP BW, источник и назначение для подключения к серверу SAP BW. 
      
        Сначала передайте в контейнер 64- или 32-разрядную версию *librfc32.dll* из папки установки SAP вместе с *main.cmd* в ваш контейнер. Затем скрипт скопирует сборку SAP в папку *%windir%\SysWow64* или *%windir%\System32* во время установки.

      * Папка *STORAGE*, которая содержит скрипт пользовательской установки (*main.cmd*) для установки Azure PowerShell на каждом узле Azure-SSIS IR. Эта программа установки позволяет развертывать и запускать пакеты служб SSIS, которые запускают [Azure PowerShell командлетов и сценариев для управления хранилищем Azure](../storage/blobs/storage-quickstart-blobs-powershell.md). 
      
        Скопируйте *main.cmd*, пример *AzurePowerShell.msi* (или используйте последнюю версию) и *storage.ps1* в ваш контейнер. Используйте *PowerShell.dtsx* в качестве шаблона для пакетов. Шаблон пакета сочетает [задачу скачивания больших двоичных объектов Azure](/sql/integration-services/control-flow/azure-blob-download-task), которая скачивает изменяемый скрипт PowerShell (*storage.ps1*) и [задачу «Выполнение процесса](https://blogs.msdn.microsoft.com/ssis/2017/01/26/run-powershell-scripts-in-ssis/)», которая выполняет скрипт на каждом узле.

      * Папка *TERADATA*, которая содержит скрипт пользовательской установки (*main.cmd*), связанный файл (*install.cmd*) и пакеты установщика ( *.msi*). Эти файлы устанавливают соединители Teradata, API Teradata Parallel Transporter (TPT) и драйвер ODBC на каждый узел среды выполнения интеграции Azure-SSIS IR Enterprise Edition. Эта программа установки позволяет использовать диспетчер соединений Teradata, источник и назначение для подключения к серверу Teradata. 
      
        Сначала [скачайте ZIP-файл Teradata Tools and Utilities 15.x](http://partnerintelligence.teradata.com) (например, *TeradataToolsAndUtilitiesBase__windows_indep.15.10.22.00.zip*), а затем отправьте его вместе с ранее упомянутыми файлами *.cmd* и *.msi* в ваш контейнер.

      * Папка *tls 1,2* , которая содержит пользовательский сценарий установки (*Main. cmd*) для использования надежного шифрования и более защищенного сетевого протокола (TLS 1,2) на каждом узле Azure-SSIS IR. Скрипт также отключает более старые версии SSL/TLS.

      * Папка *ZULU OPENJDK*, которая содержит скрипт пользовательской установки (*main.cmd*) и файл PowerShell (*install_openjdk.ps1*) для установки Zulu OpenJDK на каждом узле Azure-SSIS IR. Эта программа установки позволяет использовать Azure Data Lake Store и гибкие соединители файлов для обработки файлов ORC и Parquet. Дополнительные сведения см. в статье [Пакет дополнительных компонентов Azure для служб Integration Services (SSIS)](/sql/integration-services/azure-feature-pack-for-integration-services-ssis#dependency-on-java). 
      
        Сначала [скачайте последнюю версию Zulu OpenJDK](https://www.azul.com/downloads/zulu/zulu-windows/) (например, *zulu8.33.0.1-jdk8.0.192-win_x64.zip*), а затем отправьте этот файл вместе с *main.cmd* и *install_openjdk.ps1* в ваш контейнер.

        ![Папки в папке сценариев пользователя](media/how-to-configure-azure-ssis-ir-custom-setup/custom-setup-image12.png)

   f. Чтобы повторно использовать эти стандартные образцы пользовательской установки, скопируйте содержимое выбранной папки в контейнер.

1. При подготовке или повторной настройке Azure-SSIS IR в пользовательском интерфейсе ADF установите флажок **настроить Azure-SSIS Integration Runtime с дополнительными конфигурациями системы или установками компонентов** на странице **Дополнительные параметры** панели **настройки среды выполнения интеграции** . Затем введите URI SAS контейнера в текстовом поле **URI SAS контейнера пользовательской установки** .
   
1. При подготовке или повторной настройке Azure-SSIS IR с помощью Azure PowerShell, если она уже запущена или запущена, запустите `Set-AzDataFactoryV2IntegrationRuntime` командлет с URI SAS контейнера в качестве значения `SetupScriptContainerSasUri` параметра, а затем запустите Azure-SSIS IR.

1. После завершения стандартной пользовательской установки и запуска Azure-SSIS IR можно найти все журналы настраиваемой установки в папке *Main. cmd. log* контейнера. Они включают стандартные выходные данные файла *Main. cmd* и другие журналы выполнения.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка выпуска Enterprise Azure-SSIS IR](how-to-configure-azure-ssis-ir-enterprise-edition.md)
- [Разработка платных или лицензированных компонентов для Azure-SSIS IR](how-to-develop-azure-ssis-ir-licensed-components.md)

---
title: Управление пакетами с помощью Azure-SSIS Integration Runtime хранилища пакетов
description: Узнайте, как управлять пакетами с помощью Azure-SSIS Integration Runtime хранилища пакетов.
ms.service: data-factory
ms.topic: conceptual
author: swinarko
ms.author: sawinark
ms.custom: seo-lt-2019
ms.date: 09/29/2020
ms.openlocfilehash: 66fdd20e66360be7cdaa6d918e18ca09c413da07
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100393842"
---
# <a name="manage-packages-with-azure-ssis-integration-runtime-package-store"></a>Управление пакетами с помощью Azure-SSIS Integration Runtime хранилища пакетов

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Чтобы претянуть & сдвиги локальных рабочих нагрузок SQL Server Integration Services (SSIS) в облако, вы можете подготавливать Azure-SSIS Integration Runtime (IR) в фабрике данных Azure (ADF). Дополнительные сведения см. в разделе [подготавливает Azure-SSIS IR](./tutorial-deploy-ssis-packages-azure.md). Среда выполнения интеграции Azure-SSIS поддерживает следующие возможности:

- выполнение пакетов, развернутых в каталоге SSIS (SSISDB), которые размещаются на сервере Базы данных SQL Azure или в Управляемом экземпляре (модель развертывания для проектов);
- выполнение пакетов, развернутых в файловой системе, службе "Файлы Azure" или базе данных SQL Server (MSDB), которые размещаются в Управляемом экземпляре SQL Azure (модель развертывания для пакетов).

При использовании модели развертывания пакетов можно выбрать, нужно ли подготавливать Azure-SSIS IR с хранилищами пакетов. Они обеспечивают уровень управления пакетами поверх файловой системы, файлов Azure или базы данных MSDB, размещенной в Azure SQL Управляемый экземпляр. Azure-SSIS IR хранилище пакетов позволяет импортировать, экспортировать, удалять и запускать пакеты, а также отслеживать и прекращать выполнение пакетов с помощью SQL Server Management Studio (SSMS), аналогичного [устаревшему хранилищу пакетов служб SSIS](/sql/integration-services/service/package-management-ssis-service). 

## <a name="connect-to-azure-ssis-ir"></a>Подключение к Azure-SSIS IR

После подготовки Azure-SSIS IR можно подключиться к ней, чтобы просмотреть ее хранилища пакетов в среде SSMS.

![Подключение к Azure-SSIS IR](media/azure-ssis-integration-runtime-package-store/ssms-package-store-connect.png)

В окне **обозревателя объектов** SSMS выберите **Azure-SSIS Integration Runtime** в раскрывающемся меню **Подключение** . Затем войдите в Azure и выберите подходящую подписку, ADF и Azure-SSIS IR, которые были подготовлены с помощью хранилищ пакетов. В Azure-SSIS IR будут отображаться узлы **Запуск пакетов** и **Хранимые пакеты** под. Разверните узел **Сохраненные пакеты** , чтобы просмотреть хранилища пакетов. Разверните хранилища пакетов, чтобы просмотреть папки и пакеты под ним. Возможно, вам будет предложено ввести учетные данные доступа для хранилищ пакетов, если SSMS не удается автоматически подключиться к ним. Например, если вы развернете хранилище пакетов поверх базы данных MSDB, вам может быть предложено сначала подключиться к Azure SQL Управляемый экземпляр.

![Подключение к Azure SQL Управляемый экземпляр](media/azure-ssis-integration-runtime-package-store/ssms-package-store-connect2.png)

## <a name="manage-folders-and-packages"></a>Управление папками и пакетами

После подключения к Azure-SSIS IR в среде SSMS можно щелкнуть правой кнопкой мыши любое хранилище пакетов, папки или пакеты, чтобы открыть меню и выбрать пункт **создать папку**, **импортировать пакет**, **экспортировать пакет**, **Удалить** или **Обновить**.

   ![Управление папками и пакетами](media/azure-ssis-integration-runtime-package-store/ssms-package-store-manage.png)

   *  Выберите **создать папку** , чтобы создать новую папку для импортированных пакетов.

   *  Выберите **импортировать пакет** , чтобы импортировать пакеты из **файловой системы**, **SQL Server** (msdb) или из устаревшего **хранилища пакетов служб SSIS** в хранилище пакетов.

      ![Импорт пакета](media/azure-ssis-integration-runtime-package-store/ssms-package-store-import.png)

      В зависимости от **расположения пакета** для импорта выберите соответствующий  / **тип проверки подлинности** сервера, при необходимости введите учетные данные для доступа, выберите **путь к пакету** и введите новое **имя пакета**. При импорте пакетов их уровень защиты нельзя изменить. Чтобы изменить его, используйте SQL Server Data Tools (SSDT) или `dtutil` служебную программу командной строки.

      > [!NOTE]
      > Импорт пакетов служб SSIS в Azure-SSIS IR хранилища пакетов можно выполнить только один за другим и просто скопировать их в базовые файлы MSDB/File System или Azure, сохранив их версию SQL Server или SSIS. 
      >
      > Поскольку Azure-SSIS IR в настоящее время основывается на **SQL Server 2017**, выполнение пакетов с более низкими версиями на нем приведет к их обновлению в пакетах служб SSIS 2017 во время выполнения. Исполнение пакетов более поздних версий не поддерживается.
      >
      > Кроме того, так как устаревшие хранилища пакетов служб SSIS привязаны к определенной версии SQL Server и доступны только в среде SSMS для этой версии, пакеты с более низкими версиями в устаревших хранилищах пакетов служб SSIS должны быть экспортированы в файловую систему сначала с помощью указанной версии SSMS, прежде чем их можно будет импортировать в Azure-SSIS IR хранилища пакетов с помощью SSMS 2019 или более поздних версий.
      >
      > Кроме того, чтобы импортировать несколько пакетов служб SSIS в хранилища пакетов Azure-SSIS IR при переключении своего уровня защиты, можно использовать программу командной строки [dtutil](/sql/integration-services/dtutil-utility) , описанную в разделе [развертывание нескольких пакетов с помощью dtutil](#deploying-multiple-packages-with-dtutil).

   *  Выберите **Экспорт пакета** для экспорта пакетов из хранилища пакетов в **файловую систему**, **SQL Server** (msdb) или устаревшее **хранилище пакетов служб SSIS**.

      ![Экспорт пакета](media/azure-ssis-integration-runtime-package-store/ssms-package-store-export.png)

      В зависимости от **расположения пакета** для экспорта выберите соответствующий  / **тип проверки подлинности** сервера, при необходимости введите учетные данные доступа и выберите путь к **пакету**. При экспорте пакетов, если они зашифрованы, введите пароли для их расшифровки, а затем измените уровень защиты, например, чтобы не хранить конфиденциальные данные или шифровать их или все данные с помощью ключа пользователя или пароля.

      > [!NOTE]
      > Экспорт пакетов служб SSIS из хранилищ пакетов Azure-SSIS IR может выполняться только один за другим. без переключения уровня защиты они просто копируются при сохранении версии SQL Server или SSIS, в противном случае они будут обновлены в пакетах SSIS 2019 или более поздней версии.
      >
      > Поскольку Azure-SSIS IR в настоящее время основывается на **SQL Server 2017**, выполнение пакетов с более низкими версиями на нем приведет к их обновлению в пакетах служб SSIS 2017 во время выполнения. Исполнение пакетов более поздних версий не поддерживается.
      >
      > Кроме того, для экспорта нескольких пакетов служб SSIS из хранилищ пакетов Azure-SSIS IR при переключении уровня защиты можно использовать программу командной строки [dtutil](/sql/integration-services/dtutil-utility) , которая описывается в разделе [развертывание нескольких пакетов с помощью dtutil](#deploying-multiple-packages-with-dtutil).

   *  Выберите **Удалить** , чтобы удалить существующие папки или пакеты из хранилища пакетов.

   *  Выберите **Обновить** , чтобы отобразить только что добавленные папки или пакеты в хранилище пакетов.

## <a name="execute-packages"></a>Выполнение пакетов

После подключения к Azure-SSIS IR в среде SSMS можно щелкнуть правой кнопкой мыши любой сохраненный пакет, чтобы открыть меню и выбрать пункт **выполнить пакет**.  Откроется диалоговое окно **служебная программа для запуска пакетов** , где можно настроить выполнение пакета на Azure-SSIS IR как выполнение действий пакета служб SSIS в конвейерах ADF.

![Служебная программа для запуска пакетов страниц 1 & 2](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute.png)

![Служебная программа для запуска пакетов стр. 3 & 4](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute2.png)

Страницы " **Общие**", " **конфигурации**", " **Параметры выполнения**" и " **ведение журнала** " диалогового окна **служебная программа для запуска пакетов** соответствуют вкладке "  **Параметры** " действия "выполнение пакета служб SSIS". На этих страницах можно ввести пароль шифрования для пакета и сведения для доступа к файлу конфигурации пакета. Можно также ввести учетные данные и свойства выполнения пакета, а также сведения о доступе к папке журнала.  Страница « **Задание значений** » диалогового окна **служебная программа для запуска пакетов** соответствует вкладке « **переопределения свойств** » действия «выполнение пакета служб SSIS», где можно ввести существующие свойства пакета для переопределения. Дополнительные сведения см. [в разделе Запуск пакетов служб SSIS как выполнение действий пакета служб SSIS в конвейерах ADF](./how-to-invoke-ssis-package-ssis-activity.md).

При нажатии кнопки **выполнить** будет автоматически создано и активирован новый конвейер ADF с действием выполнить пакет служб SSIS. Если конвейер ADF с такими же параметрами уже существует, он будет перезапущен, а новый конвейер создаваться не будет. Действие конвейера ADF и выполнения пакета служб SSIS будет иметь имя `Pipeline_SSMS_YourPackageName_HashString` и `Activity_SSMS_YourPackageName` соответственно.

![Кнопка служебная программа для запуска пакетов](media/azure-ssis-integration-runtime-package-store/ssms-package-store-execute3.png)

![Действие "Выполнение пакета SSIS"](media/azure-ssis-integration-runtime-package-store/ssis-activity-package-store.png)

## <a name="monitor-and-stop-running-packages"></a>Мониторинг и завершение выполнения пакетов

После подключения к Azure-SSIS IR в среде SSMS можно развернуть узел " **выполняющиеся пакеты** ", чтобы просмотреть текущие выполняемые пакеты.  Щелкните правой кнопкой мыши любую из них, чтобы открыть меню, и выберите команду " **Закрыть** " или " **Обновить**".

   ![Мониторинг и завершение выполнения пакетов](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor.png)

   *  Выберите **остановить** , чтобы отменить выполняющийся в данный момент конвейер ADF, который запускает пакет как действие выполнить пакет служб SSIS.

   *  Выберите **Обновить** , чтобы отобразить только что выполняемые пакеты из хранилищ пакетов.

## <a name="monitor-azure-ssis-ir-and-edit-package-stores"></a>Мониторинг Azure-SSIS IR и изменение хранилищ пакетов

После подключения к Azure-SSIS IR в среде SSMS можно щелкнуть ее правой кнопкой мыши, чтобы открыть меню и выбрать пункт **Переход на портал фабрики данных Azure** или **Обновить**.

   ![Переход на портал ADF](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor2.png)

   *  Выберите **Перейти на портал фабрики данных Azure** , чтобы открыть страницу **среды выполнения интеграции** центра мониторинга ADF, где можно отслеживать Azure-SSIS IR. На плитке **хранилища пакетов** можно увидеть количество хранилищ пакетов, прикрепленных к Azure-SSIS IR.  При выборе этого номера откроется окно, в котором можно изменить связанные службы ADF, в которых хранятся сведения о доступе для хранилищ пакетов.

      ![Изменение хранилищ пакетов](media/azure-ssis-integration-runtime-package-store/ssms-package-store-monitor3.png)

   *  Выберите **Обновить** , чтобы отобразить только что добавленные папки или пакеты в пакете, в котором хранятся и запускаются пакеты из хранилищ пакетов.

## <a name="deploying-multiple-packages-with-dtutil"></a>Развертывание нескольких пакетов с помощью dtutil

Чтобы претянуть & сдвинуть локальные рабочие нагрузки служб SSIS на службы SSIS в ADF-файле, сохранив устаревшую модель развертывания пакетов, необходимо развернуть пакеты из файловой системы, базы данных MSDB, размещенной в SQL Server, или из устаревших хранилищ пакетов служб SSIS в службе файлов Azure, в базе данных MSDB, размещенной Управляемый экземпляр Azure SQL, или в хранилище пакетов Azure-SSIS IR. В то же время следует переключить уровень защиты с шифрования на ключ пользователя в незашифрованное шифрование или с помощью пароля, если это еще не сделано.

Для развертывания нескольких пакетов в пакетах можно использовать программу командной строки [dtutil](/sql/integration-services/dtutil-utility) , поставляемую вместе с SQL Server или службами SSIS. Он привязан к определенной версии служб SSIS, поэтому, если вы используете ее для развертывания пакетов с более низкими версиями без переключения уровня защиты, он просто копирует их, сохраняя их версию служб SSIS. Если вы используете его для развертывания и переключения уровня защиты одновременно, он обновит их до версии служб SSIS.

 Поскольку Azure-SSIS IR в настоящее время основывается на **SQL Server 2017**, выполнение пакетов с более низкими версиями на нем приведет к их обновлению в пакетах служб SSIS 2017 во время выполнения. Исполнение пакетов более поздних версий не поддерживается.

Следовательно, чтобы избежать обновления во время выполнения, развертывание пакетов для запуска на Azure-SSIS IR в модели развертывания пакетов должно использовать dtutil 2017, поставляемую с установкой SQL Server/SSIS 2017. Для этой цели можно скачать и установить бесплатную [версию для разработчиков SQL Server или SSIS 2017](https://go.microsoft.com/fwlink/?linkid=853016) . После установки можно найти dtutil 2017 в этой папке: `YourLocalDrive:\Program Files\Microsoft SQL Server\140\DTS\Binn` .

### <a name="deploying-multiple-packages-from-file-system-on-premises-into-azure-files-with-dtutil"></a>Развертывание нескольких пакетов из файловой системы в локальной среде в службе файлов Azure с помощью dtutil

 Чтобы развернуть несколько пакетов из файловой системы в файлы Azure и переключить уровень защиты одновременно, выполните в командной строке следующие команды. Замените все строки, которые относятся к конкретному варианту.
  
```dos
REM Persist the access credentials for Azure Files on your local machine
cmdkey /ADD:YourStorageAccountName.file.core.windows.net /USER:azure\YourStorageAccountName /PASS:YourStorageAccountKey

REM Connect Azure Files to a drive on your local machine
net use Z: \\YourStorageAccountName.file.core.windows.net\YourFileShare /PERSISTENT:Yes

REM Go to a local folder where you store your packages
cd YourLocalDrive:\...\YourPackageFolder

REM Run dtutil in a loop to deploy your packages from the local folder into Azure Files while switching their protection level
for %f in (*.dtsx) do dtutil.exe /FILE %f /ENCRYPT FILE;Z:\%f;2;YourEncryptionPassword
```

Чтобы выполнить приведенные выше команды в пакетном файле, замените `%f` на `%%f` .

Чтобы развернуть несколько пакетов из устаревших хранилищ пакетов служб SSIS на основе файловой системы в службе файлов Azure и одновременно переключить уровень защиты, можно использовать те же команды, но заменить их `YourLocalDrive:\...\YourPackageFolder` локальной папкой, используемой устаревшими хранилищами пакетов служб SSIS: `YourLocalDrive:\Program Files\Microsoft SQL Server\YourSQLServerDefaultCompatibilityLevel\DTS\Packages\YourPackageFolder` . Например, если устаревшее хранилище пакетов служб SSIS привязано к SQL Server 2016, перейдите по адресу `YourLocalDrive:\Program Files\Microsoft SQL Server\130\DTS\Packages\YourPackageFolder` .  Для значения `YourSQLServerDefaultCompatibilityLevel` см. [список уровней совместимости SQL Server по умолчанию](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level#arguments).

Если вы настроили Azure-SSIS IR хранения пакетов на основе файлов Azure, развернутые пакеты будут отображаться в них при подключении к Azure-SSIS IR в среде SSMS 2019 или более поздней версии.

### <a name="deploying-multiple-packages-from-msdb-on-premises-into-msdb-in-azure-with-dtutil"></a>Развертывание нескольких пакетов из базы данных MSDB в локальной среде в базе данных MSDB в Azure с помощью dtutil

 Чтобы развернуть несколько пакетов из базы данных MSDB, размещенных SQL Server или устаревшие хранилища пакетов служб SSIS, расположенные в базе данных MSDB, размещенной Управляемый экземпляр SQL Azure, и одновременно переключить уровень защиты, можно подключиться к SQL Server в среде SSMS, щелкнуть правой кнопкой мыши `Databases->System Databases->msdb` узел в **ОБОЗРЕВАТЕЛЕ объектов** SSMS, чтобы открыть **новое окно запроса** , и выполнить следующий сценарий T-SQL. Замените все строки, относящиеся к вашему регистру:  
  
```sql
BEGIN
  SELECT 'dtutil /SQL '+f.foldername+'\'+NAME+' /ENCRYPT SQL;'+f.foldername+'\'+NAME+';2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourSQLAuthUsername /DestPassword YourSQLAuthPassword'
  FROM msdb.dbo.sysssispackages p
  inner join msdb.dbo.sysssispackagefolders f
  ON p.folderid = f.folderid
END
```

Чтобы использовать конечную точку частной или общедоступной конечной точки управляемый экземпляр Azure SQL, замените `YourSQLManagedInstanceEndpoint` на `YourSQLMIName.YourDNSPrefix.database.windows.net` / `YourSQLMIName.public.YourDNSPrefix.database.windows.net,3342` , соответственно.

Скрипт создаст командные строки dtutil для всех пакетов в базе данных MSDB, которые можно выбрать, скопировать & вставить и запустить из командной строки.

![Создать командные строки dtutil](media/azure-ssis-integration-runtime-package-store/sql-server-msdb-to-sql-mi-msdb.png)

```dos
dtutil /SQL YourFolder\YourPackage1 /ENCRYPT SQL;YourFolder\YourPackage1;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
dtutil /SQL YourFolder\YourPackage2 /ENCRYPT SQL;YourFolder\YourPackage2;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
dtutil /SQL YourFolder\YourPackage3 /ENCRYPT SQL;YourFolder\YourPackage3;2;YourEncryptionPassword /DestServer YourSQLManagedInstanceEndpoint /DestUser YourUserName /DestPassword YourPassword
```

Если вы настроили Azure-SSIS IR хранилища пакетов поверх базы данных MSDB, развернутые пакеты будут отображаться в них при подключении к Azure-SSIS IR в среде SSMS 2019 или более поздней версии.

### <a name="deploying-multiple-packages-from-msdb-on-premises-into-azure-files-with-dtutil"></a>Развертывание нескольких пакетов из базы данных MSDB в локальной среде в службе файлов Azure с помощью dtutil

 Чтобы развернуть несколько пакетов из базы данных MSDB, размещенных SQL Server или устаревших хранилищ пакетов служб SSIS, расположенных поверх базы данных MSDB, и одновременно переключить уровень защиты, можно подключиться к SQL Server в SSMS, щелкнуть правой кнопкой мыши `Databases->System Databases->msdb` узел в **ОБОЗРЕВАТЕЛЕ объектов** SSMS, чтобы открыть **новое окно запроса** , и выполнить следующий сценарий T-SQL. Замените все строки, относящиеся к вашему регистру:  
  
```sql
BEGIN
  SELECT 'dtutil /SQL '+f.foldername+'\'+NAME+' /ENCRYPT FILE;Z:\'+f.foldername+'\'+NAME+'.dtsx;2;YourEncryptionPassword' 
  FROM msdb.dbo.sysssispackages p
  inner join msdb.dbo.sysssispackagefolders f
  ON p.folderid = f.folderid
END
```

Скрипт создаст командные строки dtutil для всех пакетов в базе данных MSDB, которые можно выбрать, скопировать & вставить и запустить из командной строки.

```dos
REM Persist the access credentials for Azure Files on your local machine
cmdkey /ADD:YourStorageAccountName.file.core.windows.net /USER:azure\YourStorageAccountName /PASS:YourStorageAccountKey

REM Connect Azure Files to a drive on your local machine
net use Z: \\YourStorageAccountName.file.core.windows.net\YourFileShare /PERSISTENT:Yes

REM Multiselect, copy & paste, and run the T-SQL-generated dtutil command lines to deploy your packages from MSDB on premises into Azure Files while switching their protection level
dtutil /SQL YourFolder\YourPackage1 /ENCRYPT FILE;Z:\YourFolder\YourPackage1.dtsx;2;YourEncryptionPassword
dtutil /SQL YourFolder\YourPackage2 /ENCRYPT FILE;Z:\YourFolder\YourPackage2.dtsx;2;YourEncryptionPassword
dtutil /SQL YourFolder\YourPackage3 /ENCRYPT FILE;Z:\YourFolder\YourPackage3.dtsx;2;YourEncryptionPassword
```

Если вы настроили Azure-SSIS IR хранения пакетов на основе файлов Azure, развернутые пакеты будут отображаться в них при подключении к Azure-SSIS IR в среде SSMS 2019 или более поздней версии.

## <a name="next-steps"></a>Дальнейшие действия

Вы можете повторно выполнить или изменить автоматически созданные конвейеры ADF с помощью действия "выполнение пакета служб SSIS" или создать новые на портале ADF. Дополнительные сведения см. [в разделе Запуск пакетов служб SSIS как выполнение действий пакета служб SSIS в конвейерах ADF](./how-to-invoke-ssis-package-ssis-activity.md).
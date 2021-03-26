---
title: Краткое руководство. Создание экземпляра в гибридном режиме с помощью портала Azure
titleSuffix: Azure Database Migration Service
description: Использование портала Azure для создания экземпляра службы Azure Database Migration Service в гибридном режиме.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: quickstart
ms.date: 03/13/2020
ms.openlocfilehash: 39f893e9375970ff4bb6e3cfa2c93ceea48ec896
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92331810"
---
# <a name="quickstart-create-a-hybrid-mode-instance-with-azure-portal--azure-database-migration-service"></a>Краткое руководство. Создание экземпляра в гибридном режиме с помощью портала Azure и Azure Database Migration Service

Гибридный режим Azure Database Migration Service позволяет управлять миграцией баз данных с помощью рабочей роли миграции, размещенной локально, и экземпляра Azure Database Migration Service, работающего в облаке. Гибридный режим особенно удобен для сценариев, в которых подключение типа "сеть — сеть" между локальной сетью и Azure отсутствует или его пропускная способность ограничена.

>[!NOTE]
>В настоящее время Azure Database Migration Service, выполняемый в гибридном режиме, поддерживает миграцию SQL Server в:
>
>- Управляемый экземпляр SQL Azure с почти нулевым временем простоя (в сети).
>- Единую базу данных SQL Azure с некоторым временем простоя (вне сети).
>- MongoDb в Azure CosmosDB с почти нулевым временем простоя (в сети).
>- MongoDb в Azure CosmosDB с некоторым временем простоя (вне сети).

В этом кратком руководстве вы создадите экземпляр службы Azure Database Migration Service в гибридном режиме с помощью портала Azure. После этого вы скачаете, установите и настроите гибридную рабочую роль в локальной сети. На этапе предварительной версии гибридный режим Azure Database Migration Service можно использовать для миграции данных из локального экземпляра SQL Server в Базу данных SQL Azure.

> [!NOTE]
> Гибридный установщик Azure Database Migration Service работает на Microsoft Windows Server 2012 R2, Window Server 2016, Windows Server 2019 и Windows 10.

> [!IMPORTANT]
> Для гибридного установщика Azure Database Migration Service требуется .NET 4.7.2 или более поздней версии. Чтобы найти последние версии .NET, перейдите на страницу скачивания [.NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).

Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись, прежде чем начинать работу.

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure

Откройте веб-браузер, перейдите к [порталу Microsoft Azure](https://portal.azure.com/) и введите учетные данные для входа на портал.

Панель мониторинга службы является представлением по умолчанию.

## <a name="register-the-resource-provider"></a>Регистрация поставщика ресурсов

Прежде чем создать свой первый экземпляр Azure Database Migration Service, зарегистрируйте поставщик ресурсов Microsoft.DataMigration.

1. На портале Azure щелкните **Подписки**, выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Поиск поставщика ресурсов](media/quickstart-create-data-migration-service-hybrid-portal/dms-portal-search-resource-provider.png)

2. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/quickstart-create-data-migration-service-hybrid-portal/dms-portal-register-resource-provider.png)

## <a name="create-an-instance-of-the-service"></a>Создание экземпляра службы

1. Щелкните **+Создать ресурс**, чтобы создать экземпляр Azure Database Migration Service.

2. Выполните в Marketplace поиск по слову "migration" (миграция), выберите службу **Azure Database Migration Service**, а затем на экране **Azure Database Migration Service** нажмите кнопку **Создать**.

3. На экране **Создать Migration Service** сделайте следующее:

    - Выберите **имя службы**, которое хорошо запоминается и будет уникальным для идентификации экземпляра Azure Database Migration Service.
    - Выберите **подписку** Azure, в которой нужно создать экземпляр.
    - Выберите существующую **группу ресурсов** или создайте новую.
    - Выберите **расположение**, наиболее близкое к исходному или целевому серверу.
    - Для параметра **Режим службы** выберите **Гибридная служба (предварительная версия)** .

         ![Создание службы миграции: основы](media/quickstart-create-data-migration-service-hybrid-portal/dms-create-service-basics.png)

4. Выберите **Просмотреть и создать**.

5. На вкладке **Просмотр и создание** просмотрите условия, проверьте другие предоставленные сведения, а затем выберите **Создать**.

    ![Создание службы миграции: просмотр и создание](media/quickstart-create-data-migration-service-hybrid-portal/dms-create-service-review-and-create.png)

    Через несколько секунд экземпляр Azure Database Migration Service в гибридном режиме будет создан и готов к настройке. Экземпляр Azure Database Migration Service отобразится, как показано на рисунке ниже.

    ![Создание экземпляра Azure Database Migration Service в гибридном режиме](media/quickstart-create-data-migration-service-hybrid-portal/dms-instance-hybrid-mode.png)

6. После создания службы выберите **Свойства**, а затем скопируйте значение поля **Идентификатор ресурса**, которое будет использоваться для установки гибридной рабочей роли Azure Database Migration Service.

    ![Свойства экземпляра Azure Database Migration Service в гибридном режиме](media/quickstart-create-data-migration-service-hybrid-portal/dms-copy-resource-id.png)

## <a name="create-azure-app-registration-id"></a>Создание идентификатора регистрации приложения Azure

Необходимо создать идентификатор регистрации приложения Azure, который локальный гибридная рабочая роль сможет использовать для взаимодействия с Azure Database Migration Service в облаке.

1. Войдите на **портал Azure** и выберите **Azure Active Directory**, а затем выберите **Новая регистрация**.
2. Укажите имя приложения, а затем в разделе **Поддерживаемые типы учетных записей** выберите поддерживаемый тип учетных записей, чтобы указать, кто может использовать приложение.

    ![Azure Database Migration Service в гибридном режиме: регистрация приложения](media/quickstart-create-data-migration-service-hybrid-portal/dms-register-application.png)

3. Используйте значения по умолчанию для полей **URI перенаправления (необязательно)** , а затем выберите **Зарегистрировать**.

4. После завершения регистрации идентификатора приложения запишите **идентификатор приложения (клиента)** , который будет использоваться при установке гибридной рабочей роли.

5. На портале Azure перейдите к Azure Database Migration Service, выберите **Управление доступом (IAM)** , а затем выберите **Добавить назначение ролей**, чтобы назначить участнику доступ по идентификатору приложения.

    ![Azure Database Migration Service в гибридном режиме: назначение роли участника](media/quickstart-create-data-migration-service-hybrid-portal/dms-app-assign-contributor.png)

6. Выберите **Участник** в качестве роли, назначьте доступ для **пользователя Azure AD или субъекта-службы**, а затем выберите имя идентификатора приложения.

    ![Azure Database Migration Service в гибридном режиме: сведения о назначении роли участника](media/quickstart-create-data-migration-service-hybrid-portal/dms-add-role-assignment.png)

7. Выберите **Сохранить**, чтобы сохранить назначение роли для идентификатора приложения для ресурса Azure Database Migration Service.

## <a name="download-and-install-the-hybrid-worker"></a>Скачивание и установка гибридной рабочей роли

1. На портале Azure перейдите к своему экземпляру Azure Database Migration Service.

2. В разделе **Параметры** выберите **Гибридные**, а затем выберите **Скачать установщик**, чтобы скачать гибридную рабочую роль.

    ![Скачивание гибридной рабочей роли Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-installer-download.png)

3. Извлеките содержимое ZIP-файла на сервер, на котором будет размещена гибридная рабочая роль Azure Database Migration Service.

    > [!IMPORTANT]
    > Для гибридного установщика Azure Database Migration Service требуется .NET 4.7.2 или более поздней версии. Чтобы найти последние версии .NET, перейдите на страницу скачивания [.NET Framework](https://dotnet.microsoft.com/download/dotnet-framework).

4. В папке установки найдите и откройте файл **dmsSettings.json**, укажите в нем **ApplicationId** и **resourceId**, а затем сохраните этот файл.

    ![Параметры гибридной рабочей роли Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-settings.png)

5. Создайте сертификат, который Azure Database Migration Service сможет использовать для аутентификации связи с гибридной рабочей ролью, с помощью следующей команды.

    ```
    <drive>:\<folder>\Install>DMSWorkerBootstrap.exe -a GenerateCert
    ```

    Сертификат создается в папке установки.

    ![Сертификат гибридной рабочей роли Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-certificate.png)

6. На портале Azure перейдите к идентификатору приложения в разделе **Управление**, выберите **Сертификаты и секреты**, а затем выберите **Отправить сертификат**, чтобы выбрать созданный открытый сертификат.

    ![Передача сертификата гибридной рабочей роли Azure Database Migration Service](media/quickstart-create-data-migration-service-hybrid-portal/dms-app-upload-certificate.png)

7. Установите гибридную рабочую роль Azure Database Migration Service на локальном сервере, выполнив следующую команду.

    ```
    <drive>:\<folder>\Install>DMSWorkerBootstrap.exe -a Install -IAcceptDMSLicenseTerms -d
    ```

    > [!NOTE]
    > При выполнении команды "install" можно также использовать приведенные ниже параметры.
    >
    > - **-TelemetryOptOut** — останавливает отправку данных телеметрии рабочей ролью, но продолжает вести журнал, как минимум локально.  Установщик по-прежнему отправляет данные телеметрии.
    > - **-p {InstallLocation}** . Включает изменение пути установки, который по умолчанию имеет значение "C:\Program Files\DatabaseMigrationServiceHybrid".

8. Если установщик выполнится без ошибок, служба будет показывать состояние "В сети" для Azure Database Migration Service и все будет готово к миграции баз данных.

    ![Azure Database Migration Service в сети](media/quickstart-create-data-migration-service-hybrid-portal/dms-instance-hybrid-mode-online.png)

## <a name="uninstall-azure-database-migration-service-hybrid-mode"></a>Удаление гибридного режима Azure Database Migration Service

В настоящее время удаление гибридного режима Azure Database Migration Service поддерживается только посредством установщика гибридной рабочей роли Azure Database Migration Service на локальном сервере с помощью следующей команды.

```
<drive>:\<folder>\Install>DMSWorkerBootstrap.exe -a uninstall
```

> [!NOTE]
> При выполнении команды удаления можно также использовать параметр "-ReuseCert", который хранит сертификат AdApp, созданный рабочим процессом generateCert.  Это позволяет использовать уже созданный и отправленный сертификат.

## <a name="set-up-the-azure-database-migration-service-hybrid-worker-using-powershell"></a>Настройка гибридной рабочей роли Azure Database Migration Service с помощью PowerShell

Помимо установки гибридной рабочей роли Azure Database Migration Service с помощью портала Azure мы предоставляем [сценарий PowerShell](https://techcommunity.microsoft.com/gxcuf89792/attachments/gxcuf89792/MicrosoftDataMigration/119/1/DMS_Hybrid_Script.zip), который можно использовать для автоматизации шагов установки рабочих ролей после создания нового экземпляра Azure Database Migration Service в гибридном режиме. Сценарий:

1. Создает новый AdApp.
2. Скачивает установщик.
3. Запускает рабочий процесс generateCert.
4. Отправляет сертификат.
5. Добавляет AdApp в качестве участника в экземпляр Azure Database Migration Service.
6. Запускает рабочий процесс установки.

Этот сценарий предназначен для быстрого создания прототипов, когда у пользователя уже есть все необходимые разрешения в среде. Обратите внимание, что AdApp и Cert в рабочей среде могут иметь разные требования, поэтому сценарий может завершиться ошибкой.

> [!IMPORTANT]
> В этом сценарии предполагается, что у вас есть экземпляр Azure Database Migration Service в гибридном режиме и что используемая учетная запись Azure имеет права на создание AdApp в клиенте и изменение Azure RBAC в подписке.

Заполните параметры в верхней части скрипта, а затем выполните скрипт из экземпляра Администратора PowerShell.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Перенос SQL Server в Управляемый экземпляр SQL Azure по сети](tutorial-sql-server-managed-instance-online.md)
> [Перенос SQL Server в Базу данных SQL Azure в автономном режиме](tutorial-sql-server-to-azure-sql.md)

---
title: Миграция пакетов служб SSIS в SQL Управляемый экземпляр
titleSuffix: Azure Database Migration Service
description: Узнайте, как перенести пакеты и проекты SQL Server Integration Services (SSIS) в Управляемый экземпляр SQL Azure с помощью Azure Database Migration Service или Помощник по миграции данных.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: e3e2aa055baf3dfb4bee0629040fc7c140844637
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101094025"
---
# <a name="migrate-sql-server-integration-services-packages-to-an-azure-sql-managed-instance"></a>Перенос пакетов SQL Server Integration Services в Управляемый экземпляр SQL Azure
Если вы используете SQL Server Integration Services (SSIS) и хотите перенести проекты или пакеты служб SSIS из исходной SSISDB, размещенной SQL Server, в целевую SSISDB, размещенную Управляемый экземпляр Azure SQL, можно использовать Azure Database Migration Service.

Если используемая версия служб SSIS более ранняя, чем 2012, или вы используете типы хранилищ пакетов, отличные от SSISDB, перед переносом проектов или пакетов служб SSIS их необходимо преобразовать с помощью мастера преобразования Integration Services проектов, который также можно запустить из SSMS. Дополнительные сведения см. в статье [Преобразование проектов в модель развертывания проекта](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages?view=sql-server-2017#convert).

> [!NOTE]
> Azure Database Migration Service (DMS) сейчас не поддерживает базу данных SQL Azure в качестве целевого назначения миграции. Сведения о повторном развертывании проектов и пакетов служб SSIS в базе данных SQL Azure см. в статье [Повторное развертывание пакетов SQL Server Integration Services в базе данных SQL Azure](./how-to-migrate-ssis-packages.md).

Вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
>
> * Оценка проектов и пакетов исходной SSIS.
> * Перенос проектов и пакетов SSIS в Azure.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этих действий вам потребуется следующее:

* Чтобы создать виртуальная сеть Microsoft Azure для Azure Database Migration Service с помощью модели развертывания Azure Resource Manager, которая обеспечивает подключение типа "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Дополнительные сведения см. в статье [сетевые топологии для миграции SQL управляемый экземпляр с помощью Azure Database Migration Service]( https://aka.ms/dmsnetworkformi). Дополнительные сведения о создании виртуальной сети приведены в [документации по виртуальным сетям](../virtual-network/index.yml). В частности, уделите внимание кратким руководствам с пошаговыми инструкциями.
* Чтобы убедиться, что правила группы безопасности сети виртуальной сети не блокируют исходящий порт 443 Сервицетаг для ServiceBus, Storage и Азуремонитор. См. дополнительные сведения о [фильтрации трафика, предназначенного для виртуальной сети, с помощью групп безопасности сети](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Настройка [брандмауэра Windows для доступа к ядру базы данных источника](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access?view=sql-server-2017).
* Чтобы открыть брандмауэр Windows, чтобы разрешить Azure Database Migration Service доступ к исходному SQL Server, по умолчанию — TCP-порт 1433.
* Если вы запустили несколько именованных экземпляров SQL Server, использующих динамические порты, вы можете включить службу обозревателя SQL и разрешить доступ к UDP-порту 1434 через брандмауэры. Это позволит службе Azure Database Migration Service подключиться к именованному экземпляру на исходном сервере.
* Если перед исходными базами данных используется брандмауэр, его правила должны разрешать службе Azure Database Migration Service доступ к исходным базам данных для миграции и файлам SMB через порт 445.
* Управляемый экземпляр SQL для размещения SSISDB. Если необходимо создать его, следуйте указаниям в статье [создание управляемый экземпляр Azure SQL](../azure-sql/managed-instance/instance-create-quickstart.md).
* Чтобы гарантировать, что имена входа, используемые для подключения исходного SQL Server и целевого управляемого экземпляра, являются членами роли сервера sysadmin.
* Чтобы убедиться, что службы SSIS подготовлены в фабрике данных Azure (ADF), которая содержит Azure-SSIS Integration Runtime (IR) с целевой SSISDB, размещенной в Управляемый экземпляр SQL (как описано в статье [Создание среды выполнения интеграции Azure SSIS в фабрике данных Azure](../data-factory/create-azure-ssis-integration-runtime.md)).

## <a name="assess-source-ssis-projectspackages"></a>Оценка проектов и пакетов исходной SSIS

Хотя оценка исходного SSISDB еще не интегрирована в базу данных Помощник по миграции (DMA), проекты и пакеты служб SSIS будут оцениваться и проверяться при повторном развертывании в целевом SSISDB, размещенном на Управляемый экземпляр SQL Azure.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Регистрация поставщика ресурсов Microsoft.DataMigration

1. Войдите на портал Azure, щелкните **Все службы** и выберите **Подписки**.

    ![Отображение подписок на портале](media/how-to-migrate-ssis-packages-mi/portal-select-subscriptions.png)

2. Выберите подписку, в которой нужно создать экземпляр Azure Database Migration Service, а затем щелкните **Поставщики ресурсов**.

    ![Отображение поставщиков ресурсов](media/how-to-migrate-ssis-packages-mi/portal-select-resource-provider.png)

3. В поле поиска введите migration, а затем справа от **Microsoft.DataMigration** щелкните **Зарегистрировать**.

    ![Регистрация поставщика ресурсов](media/how-to-migrate-ssis-packages-mi/portal-register-resource-provider.png)

## <a name="create-an-azure-database-migration-service-instance"></a>Создание экземпляра Azure Database Migration Service

1. На портале Azure выберите + **Создать ресурс**, введите в поле поиска **Azure Database Migration Service**, а затем в раскрывающемся списке выберите **Azure Database Migration Service**.

     ![Azure Marketplace](media/how-to-migrate-ssis-packages-mi/portal-marketplace.png)

2. На экране **Azure Database Migration Service** выберите **Создать**.

    ![Создание экземпляра Azure Database Migration Service](media/how-to-migrate-ssis-packages-mi/dms-create1.png)

3. На экране **Создание службы миграции** укажите имя службы, подписку и новую или существующую группу ресурсов.

4. Выберите расположение, в котором хотите создать экземпляр DMS.

5. Выберите существующую виртуальную сеть или создайте новую.

    Виртуальная сеть предоставляет Azure Database Migration Service с доступом к исходным SQL Server и целевым Управляемый экземпляр Azure SQL.

    См. дополнительные сведения о [создании виртуальной сети с помощью портала Azure](../virtual-network/quick-create-portal.md).

    Дополнительные сведения см. в статье [сетевые топологии для миграции управляемый экземпляр Azure SQL с помощью Azure Database Migration Service](./resource-network-topologies.md).

6. Выберите ценовую категорию.

    Дополнительные сведения о ценовых категориях и затратах см. на [странице с описанием цен](https://aka.ms/dms-pricing).

    ![Создание службы DMS](media/how-to-migrate-ssis-packages-mi/dms-create-service2.png)

7. Выберите **Создать**, чтобы создать службу.

## <a name="create-a-migration-project"></a>Создание проекта миграции

После создания экземпляра службы найдите его на портале Azure, откройте и создайте проект миграции.

1. На портале Azure щелкните **Все службы**, выполните поиск по запросу "Azure Database Migration Service" и выберите **Azure Database Migration Services** (Службы Azure Database Migration Service).

    ![Поиск всех экземпляров Azure Database Migration Service](media/how-to-migrate-ssis-packages-mi/dms-search.png)

2. На экране **Служба миграции баз данных Azure** найдите имя созданного экземпляра и выберите его.

3. Выберите **+ Новый проект миграции**.

4. На экране **Новый проект миграции** укажите имя проекта, в текстовом поле **Тип исходного сервера** выберите **SQL Server**, в текстовом поле **тип целевого сервера** выберите **управляемый экземпляр SQL Azure**, а затем в списке **выберите тип действия** выберите **Миграция пакетов служб SSIS**.

   ![Создание проекта DMS](media/how-to-migrate-ssis-packages-mi/dms-create-project2.png)

5. Выберите **Создать**, чтобы создать проект.

## <a name="specify-source-details"></a>Указание сведений об источнике

1. На экране **Migration source detail** (Сведения об источнике миграции) задайте сведения о подключении для исходного SQL Server.

2. Если на сервере не установлен доверенный сертификат, установите флажок **Доверять сертификату сервера**.

    Если доверенный сертификат не установлен, SQL Server создаст самозаверяющий сертификат при запуске экземпляра. Этот сертификат используется с целью шифрования учетных данных для клиентских подключений.

    > [!CAUTION]
    > TLS-соединения, шифруемые с помощью самозаверяющего сертификата, не обеспечивают надежной защиты. Они уязвимы для атак "злоумышленник в середине". Не следует надеяться на защиту TLS с самозаверяющими сертификатами в рабочей среде или на серверах, подключенных к Интернету.

   ![Сведения об источнике](media/how-to-migrate-ssis-packages-mi/dms-source-details1.png)

3. Щелкните **Сохранить**.

## <a name="specify-target-details"></a>Указание сведений о цели

1. На экране **сведения о целевом объекте миграции** укажите сведения о подключении для целевого объекта.

     ![Сведения о целевом объекте](media/how-to-migrate-ssis-packages-mi/dms-target-details2.png)

2. Щелкните **Сохранить**.

## <a name="review-the-migration-summary"></a>Просмотр сводки по миграции

1. На экране **Сводка по миграции** в текстовом поле **Имя активности** задайте имя действия миграции.

2. Для **параметра перезапись проектов и сред служб SSIS** укажите, следует ли перезаписывать или игнорировать существующие проекты и среды служб SSIS.

    ![Сводка по проекту миграции](media/how-to-migrate-ssis-packages-mi/dms-project-summary2.png)

3. Проверьте и подтвердите сведения, связанные с проектом миграции.

## <a name="run-the-migration"></a>Выполнение миграции

* Выберите **Запустить миграцию**.

## <a name="next-steps"></a>Дальнейшие действия

* Просмотрите [руководство по миграции базы данных Майкрософт](https://datamigration.microsoft.com/).
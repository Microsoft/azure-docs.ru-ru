---
title: Жизненный цикл приложения в Service Fabric
description: Описывает разработку, развертывание, тестирование, обновление, обслуживание и удаление приложений Service Fabric.
ms.topic: conceptual
ms.date: 1/19/2018
ms.openlocfilehash: ae0c79cdaafc8fc016d463a01046f0a02121330a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98785741"
---
# <a name="service-fabric-application-lifecycle"></a>Жизненный цикл приложения Service Fabric
Как и в случае с другими платформами, приложение в Azure Service Fabric обычно проходит следующие фазы: проектирование, разработка, тестирование, развертывание, обновление, техническое обслуживание и удаление. Service Fabric предоставляет первоклассную поддержку полного жизненного цикла приложений в облаке: от разработки, развертывания, ежедневного управления и технического обслуживания до вывода приложения из эксплуатации. Модель службы использует несколько различных ролей для независимого участия в жизненном цикле приложения. В этой статье представлен обзор API и того, как они используются различными ролями на протяжении всех фаз жизненного цикла приложения в Service Fabric.

[!INCLUDE [links to azure cli and service fabric cli](../../includes/service-fabric-sfctl.md)]

## <a name="service-model-roles"></a>Роли моделей служб
Роли моделей служб:

* **Разработчик службы**: разрабатывает модульные и универсальные службы, которые могут быть переназначены и использованы в нескольких приложениях одного типа или разных типов. Например, служба очередей может использоваться для создания приложения для обработки обращений (служба поддержки) или приложения для электронной коммерции (корзины).
* **Разработчик приложения.** Создает приложения путем интеграции коллекции служб для обеспечения соответствия определенным конкретным требованиям или сценариям. Например, на веб-сайте электронной коммерции могут интегрироваться интерфейсные службы JSON без отслеживания состояния службы, аукцион с отслеживанием состояния службы и служба очереди с отслеживанием состояния службы для создания аукционного приложения.
* **Администратор приложения.** Принимает решения о конфигурации приложения (установка параметров шаблона конфигурации), развертывании (сопоставление с доступными ресурсами) и обеспечении качества обслуживания. Например, администратор приложения принимает решение о языковых настройках приложения, используемых в зависимости от региона (например, английский для США или японский для Японии). Другое развернутое приложение может иметь другие настройки.
* **Оператор.** Развертывает приложения с учетом конфигурации приложения и требований, указанных администратором приложения. Например, оператор выделяет и развертывает приложение и обеспечивает его работу в Azure. Операторы должны отслеживать работоспособность и производительность приложений и поддерживать соответствующую физическую инфраструктуру при необходимости.

## <a name="develop"></a>Разработка
1. *Разработчик службы* разрабатывает различные типы служб, используя модель программирования [Reliable Actors](service-fabric-reliable-actors-introduction.md) или [Reliable Services](service-fabric-reliable-services-introduction.md).
2. *Разработчик службы* декларативно описывает создаваемые типы служб в файле манифеста службы, состоящем из одного или нескольких пакетов кода, конфигурации и данных.
3. *Разработчик приложения* затем выполняет построение приложения с использованием различных типов служб.
4. *Разработчик приложения* декларативно описывает тип приложения в его манифесте, указывая ссылки на манифесты составляющих служб и соответствующим образом переопределяя и параметризируя различные параметры конфигурации и развертывания составляющих служб.

См. статьи [Приступая к работе с Reliable Actors](service-fabric-reliable-actors-get-started.md) и [Начало работы со службами Reliable Services в Service Fabric](service-fabric-reliable-services-quick-start.md), чтобы ознакомиться с примерами.

## <a name="deploy"></a>Развернуть
1. *Администратор приложения* изменяет приложение определенного типа для конкретного приложения, которое будет развернуто в кластере Service Fabric путем указания соответствующих параметров элемента **ApplicationType** в манифесте приложения.
2. *Оператор* загружает пакет приложения в хранилище образов кластера с помощью [метода **CopyApplicationPackage**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient) или [командлета **Copy-ServiceFabricApplicationPackage**](/powershell/module/servicefabric/copy-servicefabricapplicationpackage). Пакет приложения содержит манифест приложения и коллекцию пакетов служб. Структура служб выполняет развертывание приложений из пакета приложений, размещенного в хранилище образов, который может представлять собой магазин больших двоичных объектов Azure или системную службу Service Fabric.
3. *Оператор* затем выделяет тип приложения в целевом кластере из загруженного пакета приложения с помощью [метода **ProvisionApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Register-ServiceFabricApplicationType**](/powershell/module/servicefabric/register-servicefabricapplicationtype) или [операции REST **Provision an Application**](/rest/api/servicefabric/provision-an-application).
4. После подготовки приложения *оператор* запускает приложение с параметрами, предоставленными *администратором приложения* с помощью [метода **CreateApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **New-ServiceFabricApplication**](/powershell/module/servicefabric/new-servicefabricapplication) или [операции REST **Create Application**](/rest/api/servicefabric/create-an-application).
5. После того как приложение будет развернуто, *оператор* использует [метод **CreateServiceAsync**](/dotnet/api/system.fabric.fabricclient.servicemanagementclient), [командлет **New-ServiceFabricService**](/powershell/module/servicefabric/new-servicefabricservice) или [операцию REST **Create Service**](/rest/api/servicefabric/create-a-service) для создания экземпляров службы для приложения на основании доступных типов службы.
6. Теперь приложение работает в кластере Service Fabric.

См. статью [Развертывание и удаление приложений с помощью PowerShell](service-fabric-deploy-remove-applications.md), чтобы ознакомиться с примерами.

## <a name="test"></a>Проверить
1. После развертывания в локальном кластере разработки или в тестовом кластере *разработчик службы* запускает встроенный сценарий проверки переключения на резервный ресурс с помощью классов [**FailoverTestScenarioParameters**](/dotnet/api/system.fabric.testability.scenario.failovertestscenarioparameters) и [**FailoverTestScenario**](/dotnet/api/system.fabric.testability.scenario.failovertestscenario) или [командлета **Invoke-ServiceFabricFailoverTestScenario**](/powershell/module/servicefabric/invoke-servicefabricfailovertestscenario). Сценарий тестирования отказа пропускает указанную службу через важные преобразования и отказы, чтобы гарантировать, что она остается доступной и продолжает работать.
2. Затем *разработчик службы* запускает встроенный хаотический сценарий тестирования с помощью классов [**ChaosTestScenarioParameters**](/dotnet/api/system.fabric.testability.scenario.chaostestscenarioparameters) и [**ChaosTestScenario**](/dotnet/api/system.fabric.testability.scenario.chaostestscenario) или [командлета **Invoke-ServiceFabricChaosTestScenario**](/powershell/module/servicefabric/invoke-servicefabricchaostestscenario). Хаотический сценарий тестирования в случайном порядке вызывает множественные ошибки на уровне узла, пакета кода и реплики в кластере.
3. *Разработчик службы* [тестирует обмен данными между службами](service-fabric-testability-scenarios-service-communication.md) , создавая сценарии тестирования для перемещения первичных реплик в кластере.

Дополнительные сведения см. в статье [Общие сведения о службе анализа сбоев](service-fabric-testability-overview.md).

## <a name="upgrade"></a>Обновление
1. *Разработчик службы* обновляет составляющие службы экземпляра приложения или устраняет ошибки кода и предоставляет новую версию манифеста служб.
2. *Разработчик приложения* переопределяет и параметризует настройки конфигурации и развертывания и предоставляет новую версию манифеста приложения. Разработчик приложения затем включает новые версии манифестов служб в приложение и собирает новую версию приложения этого же типа в обновленном пакете приложения.
3. *Администратор приложения* включает новую версию типа приложения в целевое приложение, обновляя соответствующие параметры.
4. *Оператор* загружает обновленный пакет приложения в хранилище образов кластера с помощью [метода **CopyApplicationPackage**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient) или [командлета **Copy-ServiceFabricApplicationPackage**](/powershell/module/servicefabric/copy-servicefabricapplicationpackage). Пакет приложения содержит манифест приложения и коллекцию пакетов служб.
5. *Оператор* предоставляет новую версию приложения для целевого кластера с помощью [метода **ProvisionApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Register-ServiceFabricApplicationType**](/powershell/module/servicefabric/register-servicefabricapplicationtype) или [операции REST **Provision an Application**](/rest/api/servicefabric/provision-an-application).
6. *Оператор* обновляет целевое приложение до новой версии, используя [метод **UpgradeApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлет **Start-ServiceFabricApplicationUpgrade**](/powershell/module/servicefabric/start-servicefabricapplicationupgrade) или [операцию **REST Upgrade an Application**](/rest/api/servicefabric/upgrade-an-application).
7. *Оператор* проверяет ход обновления с помощью [метода **GetApplicationUpgradeProgressAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Get-ServiceFabricApplicationUpgrade**](/powershell/module/servicefabric/get-servicefabricapplicationupgrade) или [операции REST **Get Application Upgrade Progress**](/rest/api/servicefabric/get-the-progress-of-an-application-upgrade1).
8. При необходимости *оператор* изменяет и повторно применяет параметры текущего обновления приложения с помощью [метода **UpdateApplicationUpgradeAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Update-ServiceFabricApplicationUpgrade**](/powershell/module/servicefabric/update-servicefabricapplicationupgrade) или [операции **REST Update Application Upgrade**](/rest/api/servicefabric/update-an-application-upgrade).
9. При необходимости *оператор* применяет откат текущего обновления приложения с помощью [метода **RollbackApplicationUpgradeAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Start-ServiceFabricApplicationRollback**](/powershell/module/servicefabric/start-servicefabricapplicationrollback) или [операции **RESTRollback Application Upgrade**](/rest/api/servicefabric/rollback-an-application-upgrade).
10. Структура службы обновляет целевое приложение, запущенное в кластере приложения без потери доступности служб, которые входят в его состав.

См. [руководство по обновлению приложений Service Fabric с помощью Visual Studio](service-fabric-application-upgrade-tutorial.md), чтобы ознакомиться с примерами.

## <a name="maintain"></a>Техническое обслуживание
1. Для обновлений и исправлений операционной системы структура служб взаимодействует с инфраструктурой Azure таким образом, чтобы обеспечивать гарантированную доступность всех приложений в кластере.
2. Для обновлений и исправлений в платформе Service Fabric обновление самой Service Fabric выполняется без потери доступности любых приложений, запущенных в кластере.
3. *Администратор приложения* утверждает добавление или удаление узлов в кластере после выполнения анализа архивных данных об использовании мощностей и прогнозируемой потребности в мощностях в будущем.
4. *Оператор* добавляет и удаляет узлы, указанные *администратором приложения*.
5. При добавлении новых или удалении существующих узлов из кластера структура службы автоматически балансирует нагрузку запущенных приложений на всех узлах в кластере, чтобы достичь оптимальной производительности.

## <a name="remove"></a>Удалить
1. *Оператор* может удалить определенный экземпляр запущенной службы в кластере без удаления всего приложения с помощью [метода **DeleteServiceAsync**](/dotnet/api/system.fabric.fabricclient.servicemanagementclient), [командлета **Remove-ServiceFabricService**](/powershell/module/servicefabric/remove-servicefabricservice) или [операции REST **Delete Service**](/rest/api/servicefabric/delete-a-service).  
2. *Оператор* может также удалить экземпляр приложения и все его службы с помощью [метода **DeleteApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Remove-ServiceFabricApplication**](/powershell/module/servicefabric/remove-servicefabricapplication) или [операции **REST Delete Application**](/rest/api/servicefabric/delete-an-application).
3. После того как приложения и службы будут остановлены, *оператор* может отменить выделение мощностей для этого типа приложений с помощью [метода **UnprovisionApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), [командлета **Unregister-ServiceFabricApplicationType**](/powershell/module/servicefabric/unregister-servicefabricapplicationtype) или [операции **REST Unprovision an Application**](/rest/api/servicefabric/unprovision-an-application). При отмене подготовки мощностей для определенного типа приложения пакет приложения не удаляется из хранилища образов; необходимо удалить его вручную. Пакет приложения необходимо удалить вручную.
4. *Оператор* удаляет пакет приложений из ImageStore с помощью [метода **RemoveApplicationPackage**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient) или [командлета **Remove-ServiceFabricApplicationPackage**](/powershell/module/servicefabric/remove-servicefabricapplicationpackage).

См. статью [Развертывание и удаление приложений с помощью PowerShell](service-fabric-deploy-remove-applications.md), чтобы ознакомиться с примерами.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о разработке и тестировании приложений Service Fabric и служб, а также об управлении ими см. в следующих разделах.

* [Надежные субъекты](service-fabric-reliable-actors-introduction.md)
* [Надежные службы](service-fabric-reliable-services-introduction.md)
* [Развертывание приложения](service-fabric-deploy-remove-applications.md)
* [Обновление приложения](service-fabric-application-upgrade.md)
* [Обзор возможностей тестирования](service-fabric-testability-overview.md)

---
title: Общие сведения о Service Fabric
description: Service Fabric — это платформа распределенных систем для создания надежных и легко управляемых масштабируемых микрослужб.
ms.topic: overview
ms.date: 09/22/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: b6a14ba3f905fbef3fc3796b616cd594941a3d60
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97028637"
---
# <a name="overview-of-azure-service-fabric"></a>Общие сведения о Service Fabric

Azure Service Fabric является [платформой распределенных систем](#container-orchestration), которая дает возможность не только легко упаковывать и развертывать масштабируемые надежные микрослужбы и контейнеры, но и управлять ими. Service Fabric также позволяет устранить значительные трудности, возникающие при [разработке облачных собственных приложений и управлении ими](#application-lifecycle-management).

Ключевое отличие Service Fabric — активная ориентация на создание служб с отслеживанием состояния. Вы можете использовать [модель программирования](#stateless-and-stateful-microservices) Service Fabric или запускать контейнерные службы с отслеживанием состояния, написанные с помощью любого языка или кода. [Кластеры Service Fabric можно создавать](#any-os-any-cloud) не только в Azure, а и в других средах, включая локальные среды и другие общедоступные облака Windows Server и Linux.

![Платформа Service Fabric предоставляет возможности для управления жизненным циклом, доступность, оркестрацию, модели программирования, проверку работоспособности и мониторинг, средства для разработки и эксплуатации, а также автоматическое масштабирование, которые можно использовать в Azure, в локальной среде, в других облаках и на компьютере разработки.][Image1]

Сейчас на базе Service Fabric работают многие службы Майкрософт, в том числе база данных SQL Azure, Azure Cosmos DB, Cortana, Microsoft Power BI, Microsoft Intune, Центры событий Azure, Центр Интернета вещей Azure, Dynamics 365, Skype для бизнеса, а также многие основные службы Azure.

## <a name="container-orchestration"></a>Оркестрация контейнеров

Service Fabric — это [оркестратор контейнеров](service-fabric-cluster-resource-manager-introduction.md) от Майкрософт, который предназначен для развертывания микрослужб, а также управления ими в кластере компьютеров и позволяет извлечь пользу при выполнении служб Microsoft в самом крупном масштабе. Service Fabric развертывает приложения за считаные секунды с плотностью в сотни или тысячи приложений или контейнеров на компьютер. С помощью Service Fabric вы можете объединять два подхода, используя в одном приложении службы с процессами и контейнерами.

[Узнайте больше об основных понятиях, моделях программирования, жизненном цикле приложения, тестировании, кластерах и мониторинге работоспособности Service Fabric](service-fabric-content-roadmap.md).

## <a name="stateless-and-stateful-microservices"></a>Микрослужбы без отслеживания состояния и с отслеживанием состояния

Service Fabric предоставляет упрощенную среду выполнения, поддерживающую микрослужбы с отслеживанием и без отслеживания состояния. Ключевое отличие Service Fabric — надежная поддержка при создании служб с отслеживанием состояния на основе [встроенных моделей программирования](service-fabric-choose-framework.md) или контейнерных служб с отслеживанием состояния Service Fabric.

Узнайте больше о [сценариях приложений](service-fabric-application-scenarios.md), используя которые вы сможете воспользоваться преимуществами служб с отслеживанием состояния Service Fabric.

## <a name="application-lifecycle-management"></a>Управление жизненным циклом приложения

Service Fabric предоставляет поддержку полного жизненного цикла приложений и CI/CD для облачных приложений и контейнеров: от разработки, развертывания, ежедневного мониторинга, управления и технического обслуживания до вывода приложения из эксплуатации. В Service Fabric интегрированы инструменты CI/CD, такие как [Azure Pipelines](https://www.visualstudio.com/team-services/), [Jenkins](https://jenkins.io/index.html) и [Octopus Deploy](https://octopus.com/). Эту службу также можно использовать с любым другим популярным инструментом CI/CD.

Дополнительные сведения об управлении жизненным циклом приложения см. в статье [Жизненный цикл приложения Service Fabric](service-fabric-application-lifecycle.md). Сведения о развертывании существующих приложений в Service Fabric см. в статье [Развертывание гостевого исполняемого файла](service-fabric-deploy-existing-app.md).

## <a name="any-os-any-cloud"></a>Любая ОС, любое облако

Кластеры Service Fabric можно создавать во многих средах, включая среду [Azure или локальную среду](service-fabric-deploy-anywhere.md), [платформу Windows Server или Linux](service-fabric-linux-windows-differences.md). Можно даже создать кластеры в других общедоступных облаках. Среда разработки в пакете SDK для Service Fabric аналогична рабочей среде и не содержит никаких эмуляторов. Другими словами, все, что работает в локальном кластере разработки, будет успешно развертываться в вашем кластере в других средах.

Пакет SDK Service Fabric для .NET для [разработки приложений Windows](service-fabric-get-started.md) интегрируется с Visual Studio и PowerShell. Пакет SDK Service Fabric для Java для [разработки приложений Linux](service-fabric-get-started-linux.md) интегрирован с Eclipse. Yeoman используется для создания шаблонов для приложений Java, .NET Core и приложений-контейнеров.

## <a name="compliance"></a>Соответствие нормативным требованиям

Поставщик ресурсов Azure Service Fabric доступен во всех регионах Azure и соответствует требованиям всех сертификатов соответствия Azure, включая такие: SOC, ISO, PCI DSS, HIPAA и GDPR. Полный список см. в статье [Предложения для соответствия требованиям Майкрософт](https://www.microsoft.com/trustcenter/compliance/complianceofferings).

## <a name="next-steps"></a>Дальнейшие действия

Создайте и разверните первое приложение в Azure Service Fabric, выполнив действия, приведенные в кратком руководстве:

> [!div class="nextstepaction"]
> [Развертывание приложения надежных служб .NET в Service Fabric][sf-quickstart]

[Image1]: media/service-fabric-overview/Service-Fabric-Overview.png
[sf-quickstart]: ./service-fabric-quickstart-dotnet.md

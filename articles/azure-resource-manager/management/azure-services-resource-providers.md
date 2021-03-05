---
title: Поставщики ресурсов по службам Azure
description: Выводит список всех пространств имен поставщиков ресурсов для Azure Resource Manager и отображает службу Azure для этого пространства имен.
ms.topic: conceptual
ms.date: 12/01/2020
ms.openlocfilehash: cc9793bfc0ca6cc0afbede241534453209685d94
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102198937"
---
# <a name="resource-providers-for-azure-services"></a>Поставщики ресурсов для служб Azure

В этой статье показано, как пространства имен поставщиков ресурсов сопоставляются со службами Azure.

## <a name="match-resource-provider-to-service"></a>Сопоставление поставщика ресурсов со службой

Поставщики ресурсов, помеченные как **зарегистрированные** , регистрируются по умолчанию для вашей подписки. Дополнительные сведения см. в разделе [Регистрация](#registration).

| Пространство имен поставщика ресурсов | Служба Azure |
| --------------------------- | ------------- |
| Microsoft.AAD | [Доменные службы Azure Active Directory](../../active-directory-domain-services/index.yml) |
| Microsoft.Addons | core |
| Microsoft. Адхибридхеалссервице — [зарегистрировано](#registration) | [Azure Active Directory](../../active-directory/index.yml) |
| Microsoft.Advisor | [Помощник по Azure](../../advisor/index.yml) |
| Microsoft.AlertsManagement | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.AnalysisServices | [Службы Azure Analysis Services](../../analysis-services/index.yml) |
| Microsoft.ApiManagement | [Управление API](../../api-management/index.yml) |
| Microsoft.AppConfiguration | [Конфигурация приложений Azure](../../azure-app-configuration/index.yml) |
| Microsoft.AppPlatform | [Azure Spring Cloud](../../spring-cloud/spring-cloud-overview.md) |
| Microsoft.Attestation | Служба аттестации Azure |
| Microsoft. Authorization- [Registered](#registration) | [Azure Resource Manager](../index.yml) |
| Microsoft.Automation | [Служба автоматизации](../../automation/index.yml) |
| Microsoft. Аутономауссистемс | [Автономные системы](https://www.microsoft.com/ai/autonomous-systems) |
| Microsoft.AVS | [Решение Azure VMware](../../azure-vmware/index.yml) |
| Microsoft.AzureActiveDirectory | [Azure Active Directory B2C](../../active-directory-b2c/index.yml) |
| Microsoft. Азуреаркдата | Службы данных с поддержкой Azure Arc |
| Microsoft.AzureData | Реестр SQL Server |
| Microsoft.AzureStack | core |
| Microsoft.AzureStackHCI | [Azure Stack HCI](/azure-stack/hci/overview) |
| Microsoft.Batch | [Пакетная служба](../../batch/index.yml) |
| Microsoft. выставление счетов — [зарегистрировано](#registration) | [Управление затратами и выставление счетов](/azure/billing/) |
| Microsoft.BingMaps | [Карты Bing](/BingMaps/#pivot=main&panel=BingMapsAPI) |
| Microsoft.Blockchain | [Служба "Блокчейн Azure"](../../blockchain/workbench/index.yml) |
| Microsoft.BlockchainTokens | [Служба блокчейн-токенов Azure](https://azure.microsoft.com/services/blockchain-tokens/) |
| Microsoft.Blueprint | [Azure Blueprints](../../governance/blueprints/index.yml) |
| Microsoft.BotService | [Служба Azure Bot](/azure/bot-service/) |
| Microsoft.Cache | [Кэш Azure для Redis](../../azure-cache-for-redis/index.yml) |
| Microsoft.Capacity | core |
| Microsoft.Cdn | [Сеть доставки содержимого](../../cdn/index.yml) |
| Microsoft.CertificateRegistration | [Сертификаты службы приложений](../../app-service/configure-ssl-certificate.md#import-an-app-service-certificate) |
| Microsoft.ChangeAnalysis | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.ClassicCompute | Виртуальная машина классической модели развертывания |
| Microsoft.ClassicInfrastructureMigrate | Миграция классической модели развертывания |
| Microsoft.ClassicNetwork | Виртуальная сеть классической модели развертывания |
| Microsoft.ClassicStorage | Хранилище классической модели развертывания |
| Microsoft. Классиксубскриптион — [зарегистрировано](#registration) | Классическая модель развертывания |
| Microsoft.CognitiveServices | [Cognitive Services](../../cognitive-services/index.yml) |
| Microsoft. Commerce — [зарегистрировано](#registration) | core |
| Microsoft.Compute; | [Виртуальные машины](../../virtual-machines/index.yml)<br />[Наборы для масштабирования виртуальных машин](../../virtual-machine-scale-sets/index.yml) |
| Microsoft. потребление — [зарегистрировано](#registration) | [Управление затратами](/azure/cost-management/) |
| Microsoft.ContainerInstance | [Экземпляры контейнеров](../../container-instances/index.yml) |
| Microsoft.ContainerRegistry | [Реестр контейнеров](../../container-registry/index.yml) |
| Microsoft.ContainerService | [Служба Azure Kubernetes (AKS)](../../aks/index.yml) |
| Microsoft. Костманажемент — [зарегистрировано](#registration) | [Управление затратами](/azure/cost-management/) |
| Microsoft.CostManagementExports | [Управление затратами](/azure/cost-management/) |
| Microsoft.CustomerLockbox | [Защищенное хранилище для Microsoft Azure](../../security/fundamentals/customer-lockbox-overview.md) |
| Microsoft.CustomProviders | [Настраиваемые поставщики Azure](../custom-providers/overview.md) |
| Microsoft.DataBox | [Azure Data Box](../../databox/index.yml) |
| Microsoft.DataBoxEdge | [Azure Stack Edge](../../databox-online/azure-stack-edge-overview.md) |
| Microsoft.Databricks | [Azure Databricks](/azure/azure-databricks/) |
| Microsoft.DataCatalog | [Каталог данных](../../data-catalog/index.yml) |
| Microsoft.DataFactory | [Фабрика данных](../../data-factory/index.yml) |
| Microsoft.DataLakeAnalytics | [Аналитика озера данных](../../data-lake-analytics/index.yml) |
| Microsoft.DataLakeStore | [Azure Data Lake Storage 2-го поколения](../../storage/blobs/data-lake-storage-introduction.md) |
| Microsoft.DataMigration | [Azure Database Migration Service](../../dms/index.yml) |
| Microsoft.DataProtection | Защита данных |
| Microsoft.DataShare | [Azure Data Share](../../data-share/index.yml) |
| Microsoft.DBforMariaDB | [База данных Azure для MariaDB](../../mariadb/index.yml) |
| Microsoft.DBforMySQL | [База данных Azure для MySQL](../../mysql/index.yml) |
| Microsoft.DBforPostgreSQL | [База данных Azure для PostgreSQL](../../postgresql/index.yml) |
| Microsoft.DeploymentManager | [диспетчер развертывания Azure](../templates/deployment-manager-overview.md) |
| Microsoft.DesktopVirtualization | [Виртуальный рабочий стол Windows](../../virtual-desktop/index.yml) |
| Microsoft.Devices | [Центр Интернета вещей Azure](../../iot-hub/index.yml)<br />[Служба подготовки устройств для Центра Интернета вещей Azure](../../iot-dps/index.yml) |
| Microsoft.DevOps | [Azure DevOps](/azure/devops/) |
| Microsoft.DevSpaces | [Azure Dev Spaces](../../dev-spaces/index.yml); |
| Microsoft.DevTestLab | [Службы лабораторий Azure](../../lab-services/index.yml) |
| Microsoft.DigitalTwins | [Azure Digital Twins](../../digital-twins/overview.md) |
| Microsoft.DocumentDB | [Azure Cosmos DB](../../cosmos-db/index.yml) |
| Microsoft.DomainRegistration | [Служба приложений](../../app-service/index.yml) |
| Microsoft.DynamicsLcs | [Службы жизненного цикла](https://lcs.dynamics.com/Logon/Index ) |
| Microsoft.EnterpriseKnowledgeGraph | График базы знаний предприятия |
| Microsoft.EventGrid | [Сетка событий](../../event-grid/index.yml) |
| Microsoft.EventHub | [Центры событий](../../event-hubs/index.yml) |
| Microsoft. Features- [зарегистрировано](#registration) | [Azure Resource Manager](../index.yml) |
| Microsoft.GuestConfiguration | [Политика Azure](../../governance/policy/index.yml) |
| Microsoft.HanaOnAzure | [SAP HANA на крупных экземплярах Azure](../../virtual-machines/workloads/sap/hana-overview-architecture.md) |
| Microsoft.HardwareSecurityModules | [Выделенный модуль HSM Azure](../../dedicated-hsm/index.yml) |
| Microsoft.HDInsight | [HDInsight](../../hdinsight/index.yml) |
| Microsoft.HealthcareApis | [Azure API для FHIR](../../healthcare-apis/index.yml) |
| Microsoft.HybridCompute | [Azure Arc](../../azure-arc/index.yml) |
| Microsoft.HybridData | [StorSimple](../../storsimple/index.yml) |
| Microsoft. Хибриднетворк  | [Зоны частного периметра](../../networking/edge-zones-overview.md) |
| Microsoft.ImportExport | [Импорт и экспорт Microsoft Azure](../../import-export/storage-import-export-service.md) |
| microsoft.insights | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.IoTCentral | [Azure IoT Central](../../iot-central/index.yml) |
| Microsoft.IoTSpaces | [Azure Digital Twins](../../digital-twins/index.yml) |
| Microsoft.Intune | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.KeyVault | [хранилище ключей;](../../key-vault/index.yml) |
| Microsoft.Kubernetes | [Служба Azure Kubernetes (AKS)](../../aks/index.yml) |
| Microsoft.KubernetesConfiguration | [Служба Azure Kubernetes (AKS)](../../aks/index.yml) |
| Microsoft.Kusto | [Azure Data Explorer](/azure/data-explorer/) |
| Microsoft.LabServices | [Службы лабораторий Azure](../../lab-services/index.yml) |
| Microsoft.Logic | [Logic Apps](../../logic-apps/index.yml) |
| Microsoft.MachineLearning | [Студия машинного обучения](../../machine-learning/classic/index.yml) |
| Microsoft.MachineLearningServices | [Машинное обучение Azure](../../machine-learning/index.yml) |
| Microsoft.Maintenance | [Обслуживание Azure](../../virtual-machines/maintenance-control-cli.md) |
| Microsoft.ManagedIdentity | [Управляемые удостоверения для ресурсов Azure](../../active-directory/managed-identities-azure-resources/index.yml) |
| Microsoft.ManagedNetwork | Виртуальные сети, управляемые службами PaaS |
| Microsoft.ManagedServices | [Azure Lighthouse](../../lighthouse/index.yml) |
| Microsoft.Management | [Группы управления](../../governance/management-groups/index.yml) |
| Microsoft.Maps | [Azure Maps](../../azure-maps/index.yml) |
| Microsoft.Marketplace | core |
| Microsoft.MarketplaceApps | core |
| Microsoft. Маркетплацеордеринг — [зарегистрировано](#registration) | core |
| Microsoft.Media | [Службы мультимедиа](../../media-services/index.yml) |
| Microsoft.Microservices4Spring | [Azure Spring Cloud](../../spring-cloud/spring-cloud-overview.md) |
| Microsoft.Migrate | [Миграция Azure](../../migrate/migrate-services-overview.md) |
| Microsoft.MixedReality | [Пространственные привязки Azure.](../../spatial-anchors/index.yml) |
| Microsoft.NetApp | [Azure NetApp Files](../../azure-netapp-files/index.yml) |
| Microsoft.Network. | [Шлюз приложений](../../application-gateway/index.yml)<br />[Бастион Azure](../../bastion/index.yml)<br />[Защита от атак DDoS Azure](../../ddos-protection/ddos-protection-overview.md)<br />[Azure DNS](../../dns/index.yml)<br />[Azure ExpressRoute](../../expressroute/index.yml)<br />[Брандмауэр Azure](../../firewall/index.yml)<br />[Azure Front Door Service](../../frontdoor/index.yml)<br />[Приватный канал Azure](../../private-link/index.yml)<br />[Load Balancer](../../load-balancer/index.yml)<br />[Наблюдатель за сетями](../../network-watcher/index.yml)<br />[Диспетчер трафика](../../traffic-manager/index.yml)<br />[Виртуальная сеть](../../virtual-network/index.yml)<br />[Виртуальная глобальная сеть](../../virtual-wan/index.yml)<br />[VPN-шлюз](../../vpn-gateway/index.yml)<br /> |
| Microsoft. Notebooks | [Записные книжки Azure](https://notebooks.azure.com/help/introduction) |
| Microsoft.NotificationHubs | [Центры уведомлений](../../notification-hubs/index.yml) |
| Microsoft.ObjectStore | Хранилище объектов |
| Microsoft.OffAzure | [Миграция Azure](../../migrate/migrate-services-overview.md) |
| Microsoft.OperationalInsights | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.OperationsManagement | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.Peering | [Служба пиринга Azure](../../peering-service/index.yml) |
| Microsoft.PolicyInsights | [Политика Azure](../../governance/policy/index.yml) |
| Microsoft. Portal — [зарегистрировано](#registration) | [Портал Azure](../../azure-portal/index.yml) |
| Microsoft.PowerBI | [Power BI](/power-bi/power-bi-overview) |
| Microsoft.PowerBIDedicated | [Что такое Power BI Embedded в Azure?](/azure/power-bi-embedded/) |
| Microsoft. Поверплатформ | [Power Platform](/power-platform/) |
| Microsoft.ProjectBabylon | [Каталог данных Azure](../../data-catalog/overview.md) |
| Microsoft.Quantum | [Такт Azure](https://azure.microsoft.com/services/quantum/) |
| Microsoft.RecoveryServices | [Azure Site Recovery](../../site-recovery/index.yml) |
| Microsoft.RedHatOpenShift | [Azure Red Hat OpenShift](../../virtual-machines/linux/openshift-get-started.md) |
| Microsoft.Relay | [Azure Relay](../../azure-relay/relay-what-is-it.md) |
| Microsoft. Ресаурцеграф — [зарегистрировано](#registration) | [Azure Resource Graph](../../governance/resource-graph/index.yml) |
| Microsoft.ResourceHealth | [Работоспособность служб Azure](../../service-health/index.yml) |
| Microsoft. Resources — [зарегистрировано](#registration) | [Azure Resource Manager](../index.yml) |
| Microsoft.SaaS | core |
| Microsoft.Scheduler | [Планировщик](../../scheduler/index.yml) |
| Microsoft.Search | [Когнитивный поиск Azure](../../search/index.yml) |
| Microsoft.Security | [Центр безопасности](../../security-center/index.yml) |
| Microsoft.SecurityInsights | [Azure Sentinel](../../sentinel/index.yml) |
| Microsoft. Сериалконсоле — [зарегистрировано](#registration) | [Серийная консоль Azure для Windows](../../virtual-machines/troubleshooting/serial-console-windows.md) |
| Microsoft.ServiceBus | [Служебная шина](/azure/service-bus/) |
| Microsoft.ServiceFabric | [Service Fabric](../../service-fabric/index.yml) |
| Microsoft.ServiceFabricMesh | [Служба Microsoft Azure Service Fabric](../../service-fabric-mesh/index.yml) |
| Microsoft.Services | core |
| Microsoft.SignalRService | [Служба SignalR Azure](../../azure-signalr/index.yml) |
| Microsoft.SoftwarePlan | Лицензия |
| Microsoft.Solutions | [Управляемые приложения Azure](../managed-applications/index.yml) |
| Microsoft.Sql | [База данных SQL Azure](../../azure-sql/database/index.yml)<br /> [Управляемый экземпляр SQL Azure](../../azure-sql/managed-instance/index.yml) <br />[Azure Synapse Analytics](/azure/sql-data-warehouse/) |
| Microsoft.SqlVirtualMachine | [SQL Server на Виртуальных машинах Microsoft Azure](../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) |
| Microsoft.Storage; | [Память](../../storage/index.yml) |
| Microsoft.StorageCache | [Azure HPC Cache](../../hpc-cache/index.yml) |
| Microsoft.StorageSync | [Память](../../storage/index.yml) |
| Microsoft.StorSimple | [StorSimple](../../storsimple/index.yml) |
| Microsoft.StreamAnalytics | [Azure Stream Analytics](../../stream-analytics/index.yml) |
| Microsoft.Subscription | core |
| Microsoft. support — [зарегистрировано](#registration) | core |
| Microsoft.Synapse | [Azure Synapse Analytics](/azure/sql-data-warehouse/) |
| Microsoft.TimeSeriesInsights | [Аналитика временных рядов Azure](../../time-series-insights/index.yml) |
| Microsoft.Token | Токен |
| Microsoft.VirtualMachineImages | [Средство создания образов Azure](../../virtual-machines/image-builder-overview.md) |
| microsoft.visualstudio | [Azure DevOps](/azure/devops/) |
| Microsoft.VMware | [Решение Azure VMware](../../azure-vmware/index.yml) |
| Microsoft.VMwareCloudSimple | [Решение VMware в Azure от CloudSimple](../../vmware-cloudsimple/index.md) |
| Microsoft.VSOnline | [Azure DevOps](/azure/devops/) |
| Microsoft.Web | [Служба приложений](../../app-service/index.yml)<br />[Функции Azure](../../azure-functions/index.yml) |
| Microsoft.WindowsDefenderATP | [Advanced Threat Protection в Microsoft Defender](../../security-center/security-center-wdatp.md) |
| Microsoft.WindowsESU | Расширенные обновления для системы безопасности |
| Microsoft.WindowsIoT | [Основные службы Windows 10 IoT](/windows-hardware/manufacture/iot/iotcoreservicesoverview) |
| Microsoft.WorkloadMonitor | [Azure Monitor](../../azure-monitor/index.yml) |

## <a name="registration"></a>Регистрация

Поставщики ресурсов, помеченные как " **-зарегистрированные** ", регистрируются по умолчанию для вашей подписки. Чтобы использовать другие поставщики ресурсов, [их необходимо зарегистрировать](resource-providers-and-types.md). Однако многие поставщики ресурсов регистрируются при выполнении определенных действий. Например, при создании ресурса на портале портал автоматически регистрирует все необходимые незарегистрированные поставщики ресурсов. При развертывании ресурсов с помощью [шаблона Azure Resource Manager](../templates/overview.md)также регистрируются все необходимые поставщики ресурсов.

> [!IMPORTANT]
> Зарегистрируйте поставщик ресурсов только тогда, когда будете готовы использовать его. Этап регистрации позволяет поддерживать минимальные привилегии в рамках подписки. Пользователь-злоумышленник не может использовать незарегистрированные поставщики ресурсов.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о поставщиках ресурсов, включая регистрацию поставщика ресурсов, см. в статье [поставщики и типы ресурсов Azure](resource-providers-and-types.md).
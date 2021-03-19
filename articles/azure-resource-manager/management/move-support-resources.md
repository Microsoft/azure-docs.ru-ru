---
title: Поддержка операции перемещения типом ресурсов
description: Список типов ресурсов Azure, подлежащих перемещению в новую группу ресурсов или подписку.
ms.topic: conceptual
ms.date: 01/11/2021
ms.openlocfilehash: 3a15ddc62c19d74871de33003d46ec2fc5437f51
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104584692"
---
# <a name="move-operation-support-for-resources"></a>Поддержка операции перемещения для ресурсов

В этой статье указано, поддерживается ли для тех или иных типов ресурсов Azure операция перемещения. В нем также содержатся сведения о специальных условиях, которые следует учитывать при перемещении ресурса.

> [!IMPORTANT]
> В большинстве случаев дочерний ресурс нельзя перемещать независимо от родительского ресурса. Дочерние ресурсы имеют тип ресурса в формате `<resource-provider-namespace>/<parent-resource>/<child-resource>` . Например, `Microsoft.ServiceBus/namespaces/queues` является дочерним ресурсом `Microsoft.ServiceBus/namespaces` . При перемещении родительского ресурса его дочерний ресурс автоматически перемещается вместе с ним. Если в этой статье не отображается дочерний ресурс, можно предположить, что он перемещен с родительским ресурсом. Если родительский ресурс не поддерживает перемещение, дочерний ресурс нельзя переместить.

Переход к пространству имен поставщика ресурсов:
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.AppService](#microsoftappservice)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.BizTalkServices](#microsoftbiztalkservices)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.BlockchainTokens](#microsoftblockchaintokens)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Capacity](#microsoftcapacity)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.ClassicSubscription](#microsoftclassicsubscription)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.ContentModerator](#microsoftcontentmoderator)
> - [Microsoft.CortanaAnalytics](#microsoftcortanaanalytics)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerInsights](#microsoftcustomerinsights)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataConnect](#microsoftdataconnect)
> - [Microsoft.DataExchange](#microsoftdataexchange)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DataLake](#microsoftdatalake)
> - [Microsoft.DataLakeAnalytics](#microsoftdatalakeanalytics)
> - [Microsoft.DataLakeStore](#microsoftdatalakestore)
> - [Microsoft.DataMigration](#microsoftdatamigration)
> - [Microsoft.DataProtection](#microsoftdataprotection)
> - [Microsoft.DataShare](#microsoftdatashare)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.DeploymentManager](#microsoftdeploymentmanager)
> - [Microsoft.DesktopVirtualization](#microsoftdesktopvirtualization)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DevOps](#microsoftdevops)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DigitalTwins](#microsoftdigitaltwins)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.Genomics](#microsoftgenomics)
> - [Microsoft.GuestConfiguration](#microsoftguestconfiguration)
> - [Microsoft.HanaOnAzure](#microsofthanaonazure)
> - [Microsoft.HardwareSecurityModules](#microsofthardwaresecuritymodules)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.HealthcareApis](#microsofthealthcareapis)
> - [Microsoft.HybridCompute](#microsofthybridcompute)
> - [Microsoft.HybridData](#microsofthybriddata)
> - [Microsoft. Хибриднетворк](#microsofthybridnetwork)
> - [Microsoft.Hydra](#microsofthydra)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.LocationBasedServices](#microsoftlocationbasedservices)
> - [Microsoft.LocationServices](#microsoftlocationservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
> - [Microsoft.MachineLearningCompute](#microsoftmachinelearningcompute)
> - [Microsoft.MachineLearningExperimentation](#microsoftmachinelearningexperimentation)
> - [Microsoft.MachineLearningModelManagement](#microsoftmachinelearningmodelmanagement)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.Maintenance](#microsoftmaintenance)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.ManagedNetwork](#microsoftmanagednetwork)
> - [Microsoft.ManagedServices](#microsoftmanagedservices)
> - [Microsoft.Management](#microsoftmanagement)
> - [Microsoft.Maps](#microsoftmaps)
> - [Microsoft.Marketplace](#microsoftmarketplace)
> - [Microsoft.MarketplaceApps](#microsoftmarketplaceapps)
> - [Microsoft.MarketplaceOrdering](#microsoftmarketplaceordering)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Microservices4Spring](#microsoftmicroservices4spring)
> - [Microsoft.Migrate](#microsoftmigrate)
> - [Microsoft.MixedReality](#microsoftmixedreality)
> - [Microsoft.NetApp](#microsoftnetapp)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.ObjectStore](#microsoftobjectstore)
> - [Microsoft.OffAzure](#microsoftoffazure)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.Peering](#microsoftpeering)
> - [Microsoft.PolicyInsights](#microsoftpolicyinsights).
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.ProjectBabylon](#microsoftprojectbabylon)
> - [Microsoft.ProviderHub](#microsoftproviderhub)
> - [Microsoft.Quantum](#microsoftquantum)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.RedHatOpenShift](#microsoftredhatopenshift)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.ResourceHealth](#microsoftresourcehealth)
> - [Microsoft.Resources](#microsoftresources)
> - [Microsoft.SaaS](#microsoftsaas)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServerManagement](#microsoftservermanagement)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.Sql](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.StreamAnalyticsExplorer](#microsoftstreamanalyticsexplorer)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [microsoft.support](#microsoftsupport)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [microsoft.visualstudio](#microsoftvisualstudio)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft. Ворклоадбуилдер](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | domainservices | нет | нет |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | diagnosticsettings | нет | нет |
> | diagnosticsettingscategories | нет | Нет |
> | privatelinkforazuread | Да | Да |
> | tenants | Да | Да |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | supportproviders | нет | нет |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | aadsupportcases | нет | нет |
> | addsservices | нет | нет |
> | agents | нет | нет |
> | anonymousapiusers | нет | нет |
> | настройка | нет | нет |
> | журналы | нет | нет |
> | reports | нет | нет |
> | servicehealthmetrics | нет | нет |
> | services; | нет | нет |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | конфигурации | нет | нет |
> | generaterecommendations | нет | нет |
> | метаданные | нет | нет |
> | к просмотру фильмов | нет | нет |
> | suppressions | нет | нет |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | actionrules | Да | Да |
> | оповещения | нет | нет |
> | alertslist | нет | нет |
> | alertsmetadata | нет | нет |
> | alertssummary | нет | нет |
> | alertssummarylist | нет | нет |
> | smartdetectoralertrules | Да | Да |
> | smartgroups | нет | нет |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | servers | Да | Да |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> Невозможно переместить службу управления API, для которой настроено значение "SKU потребления".

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | reportfeedback | нет | Нет |
> | служба | Да | Да |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | configurationstores | Да | Да |
> | configurationstores / eventgridfilters | нет | нет |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | spring | Да | Да |

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!IMPORTANT]
> См. статью [Руководство по перемещению службы приложений](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | apiapps | нет | нет |
> | appidentities | нет | нет |
> | gateways | нет | Нет |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | attestationproviders | Да | Да |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | classicadministrators | нет | нет |
> | dataaliases | нет | нет |
> | denyassignments | нет | нет |
> | elevateaccess | нет | нет |
> | findorphanroleassignments | нет | нет |
> | locks | нет | нет |
> | разрешения | нет | нет |
> | policyassignments | нет | нет |
> | policydefinitions | нет | нет |
> | policysetdefinitions | нет | Нет |
> | привателинкассоЦиатионс | Нет | Нет |
> | ресаурцеманажементпривателинкс | Нет | нет |
> | roleassignments | нет | нет |
> | roleassignmentsusagemetrics | нет | нет |
> | roledefinitions | нет | нет |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!IMPORTANT]
> Модули Runbook должны находиться в той же группе ресурсов, что и учетная запись службы автоматизации.
>
> Дополнительные сведения см. [в статье Перемещение учетной записи службы автоматизации Azure в другую подписку](../../automation/how-to/move-account.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | automationaccounts | Да | Да |
> | automationaccounts / configurations | Да | Да |
> | automationaccounts / runbooks | Да | Да |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | privateclouds | Да | Да |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | b2cdirectories | Да | Да |
> | b2ctenants | нет | нет |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | datacontrollers | нет | нет |
> | hybriddatamanagers | нет | нет |
> | postgresinstances | нет | нет |
> | sqlinstances | нет | нет |
> | sqlmanagedinstances | нет | нет |
> | sqlserverinstances | нет | нет |
> | sqlserverregistrations | Да | Да |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | cloudmanifestfiles | нет | нет |
> | registrations | Да | Да |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | clusters | нет | нет |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | batchaccounts | Да | Да |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | billingaccounts | нет | нет |
> | billingperiods | нет | нет |
> | billingpermissions | нет | нет |
> | billingproperty | нет | нет |
> | billingroleassignments | нет | нет |
> | billingroledefinitions | нет | нет |
> | departments | нет | нет |
> | enrollmentaccounts | нет | нет |
> | invoices | нет | нет |
> | transfers | нет | нет |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | mapapis | нет | нет |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | biztalk | нет | нет |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | blockchainmembers | нет | нет |
> | cordamembers | нет | нет |
> | watchers | нет | нет |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | tokenservices | нет | нет |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | blueprintassignments | нет | нет |
> | blueprints | нет | нет |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | botservices | Да | Да |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> Если экземпляр кэша Azure для Redis настроен с помощью виртуальной сети, его нельзя переместить в другую подписку. См. [ограничения перемещения сети](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | redis | Да | Да |
> | redisenterprise | нет | Нет |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | appliedreservations | нет | нет |
> | calculateexchange | нет | нет |
> | calculateprice | нет | нет |
> | calculatepurchaseprice | нет | нет |
> | catalogs | нет | нет |
> | commercialreservationorders | нет | нет |
> | обмен валюты | нет | нет |
> | reservationorders | нет | нет |
> | reservations | нет | нет |
> | ресурсов | нет | нет |
> | validatereservationorder | нет | нет |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | cdnwebapplicationfirewallmanagedrulesets | нет | нет |
> | cdnwebapplicationfirewallpolicies | Да | Да |
> | edgenodes | нет | нет |
> | профили | Да | Да |
> | profiles / endpoints | Да | Да |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!IMPORTANT]
> См. статью [Руководство по перемещению службы приложений](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | certificateorders | Да | Да |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!IMPORTANT]
> См. [Руководство по перемещению классического развертывания](./move-limitations/classic-model-move-limitations.md). Ресурсы классической модели развертывания можно перемещать между подписками с помощью операции, характерной для этого сценария.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | capabilities | нет | нет |
> | domainnames | Да | нет |
> | quotas | нет | нет |
> | resourcetypes | нет | нет |
> | validatesubscriptionmoveavailability | нет | нет |
> | virtualmachines | Да | Да |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!IMPORTANT]
> См. [Руководство по перемещению классического развертывания](./move-limitations/classic-model-move-limitations.md). Ресурсы классической модели развертывания можно перемещать между подписками с помощью операции, характерной для этого сценария.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | classicinfrastructureresources | нет | Нет |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!IMPORTANT]
> См. [Руководство по перемещению классического развертывания](./move-limitations/classic-model-move-limitations.md). Ресурсы классической модели развертывания можно перемещать между подписками с помощью операции, характерной для этого сценария.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | capabilities | нет | нет |
> | expressroutecrossconnections | нет | нет |
> | expressroutecrossconnections / peerings | нет | нет |
> | gatewaysupporteddevices | нет | нет |
> | networksecuritygroups | нет | нет |
> | quotas | нет | нет |
> | reservedips | нет | нет |
> | virtualnetworks | нет | Нет |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> См. [Руководство по перемещению классического развертывания](./move-limitations/classic-model-move-limitations.md). Ресурсы классической модели развертывания можно перемещать между подписками с помощью операции, характерной для этого сценария.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | disks | нет | нет |
> | images | нет | нет |
> | osimages | нет | нет |
> | osplatformimages | нет | нет |
> | publicimages | нет | нет |
> | quotas | нет | нет |
> | storageaccounts | Да | нет |
> | vmimages | нет | Нет |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!IMPORTANT]
> См. [Руководство по перемещению классического развертывания](./move-limitations/classic-model-move-limitations.md). Ресурсы классической модели развертывания можно перемещать между подписками с помощью операции, характерной для этого сценария.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | Операции | нет | нет |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | Да | Да |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | ratecard | нет | нет |
> | usageaggregates | нет | нет |

## <a name="microsoftcompute"></a>Microsoft.Compute;

> [!IMPORTANT]
> См. [Руководство по перемещению виртуальных машин](./move-limitations/virtual-machines-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | availabilitysets | Да | Да |
> | diskaccesses | нет | нет |
> | diskencryptionsets | нет | нет |
> | disks | Да | Да |
> | galleries | нет | нет |
> | galleries / images | нет | нет |
> | galleries / images / versions | нет | нет |
> | hostgroups | нет | нет |
> | hostgroups / hosts | нет | нет |
> | images | Да | Да |
> | proximityplacementgroups | Да | Да |
> | restorepointcollections | нет | нет |
> | restorepointcollections / restorepoints | нет | нет |
> | sharedvmextensions | нет | нет |
> | sharedvmimages | нет | нет |
> | sharedvmimages / versions | нет | нет |
> | snapshots | Да | Да |
> | sshpublickeys | нет | нет |
> | virtualmachines | Да | Да |
> | virtualmachines / extensions | Да | Да |
> | virtualmachinescalesets | Да | Да |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | aggregatedcost | нет | нет |
> | balances | нет | нет |
> | budgets | нет | нет |
> | charges | нет | нет |
> | costtags | нет | нет |
> | credits | нет | нет |
> | события | нет | нет |
> | forecasts | нет | нет |
> | lots | нет | нет |
> | marketplaces | нет | нет |
> | pricesheets | нет | нет |
> | products | нет | нет |
> | reservationdetails | нет | нет |
> | reservationrecommendationdetails | нет | нет |
> | reservationrecommendations | нет | нет |
> | reservationsummaries | нет | нет |
> | reservationtransactions | нет | нет |
> | tags | Нет | нет |
> | tenants | нет | нет |
> | terms | нет | нет |
> | usagedetails | нет | нет |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | containergroups | нет | нет |
> | serviceassociationlinks | нет | нет |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | registries | Да | Да |
> | registries / agentpools | Да | Да |
> | registries / buildtasks | Да | Да |
> | registries / replications | Да | Да |
> | registries / tasks | Да | Да |
> | registries / webhooks | Да | Да |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | containerservices | нет | нет |
> | managedclusters | нет | нет |
> | openshiftmanagedclusters | нет | нет |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | веб-масштабированием; | нет | нет |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | оповещения | нет | нет |
> | billingaccounts | нет | нет |
> | budgets | нет | нет |
> | cloudconnectors | нет | нет |
> | соединители | Да | Да |
> | departments | нет | нет |
> | dimensions | нет | нет |
> | enrollmentaccounts | нет | нет |
> | exports | нет | нет |
> | externalbillingaccounts | нет | нет |
> | прогноз | нет | нет |
> | query | нет | нет |
> | регистрация | нет | нет |
> | reportconfigs | нет | нет |
> | reports | нет | нет |
> | Параметры | нет | нет |
> | showbackrules | нет | нет |
> | узел "Представления" | нет | нет |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | hubs | нет | нет |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | requests | нет | нет |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | associations | нет | нет |
> | resourceproviders | Да | Да |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | jobs | нет | нет |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | аваилаблескус | Нет | Нет |
> | databoxedgedevices | нет | нет |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | workspaces | нет | нет |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | catalogs | Да | Да |
> | datacatalogs | нет | нет |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | connectionmanagers | нет | нет |

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | Пакеты | нет | нет |
> | Планы | нет | нет |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | datafactories | Да | Да |
> | factories | Да | Да |

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | datalakeaccounts | нет | нет |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | Да | Да |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | Да | Да |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | services; | нет | нет |
> | services / projects | нет | нет |
> | slots | нет | нет |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | backupvaults | нет | нет |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | Да | Да |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | servers | Да | Да |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | флексиблесерверс | Нет | нет |
> | servers | Да | Да |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | флексиблесерверс | Нет | нет |
> | servergroups | нет | нет |
> | servers | Да | Да |
> | serversv2 | Да | Да |
> | singleservers | Да | Да |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | artifactsources | Да | Да |
> | rollouts | Да | Да |
> | servicetopologies | Да | Да |
> | servicetopologies / services | Да | Да |
> | servicetopologies / services / serviceunits | Да | Да |
> | steps | Да | Да |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | applicationgroups | Да | Да |
> | hostpools | Да | Да |
> | workspaces | Да | Да |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | elasticpools | нет | нет |
> | elasticpools / iothubtenants | нет | нет |
> | iothubs | Да | Да |
> | provisioningservices | Да | Да |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | pipelines | Да | Да |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | controllers | Да | Да |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | labcenters | нет | нет |
> | labs | Да | нет |
> | labs / environments | Да | Да |
> | labs / servicerunners | Да | Да |
> | labs / virtualmachines | Да | нет |
> | schedules | Да | Да |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | digitaltwinsinstances | нет | нет |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | databaseaccountnames | нет | нет |
> | databaseaccounts | Да | Да |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | domains | Да | Да |
> | generatessorequest | нет | нет |
> | topleveldomains | нет | нет |
> | validatedomainregistrationinformation | нет | нет |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | services; | Да | Да |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | domains | Да | Да |
> | eventsubscriptions | Нет — нельзя перемещать независимо, но автоматически перемещается вместе с подписанным ресурсом. | Нет — нельзя перемещать независимо, но автоматически перемещается вместе с подписанным ресурсом. |
> | extensiontopics | нет | нет |
> | partnernamespaces | Да | Да |
> | partnerregistrations | нет | нет |
> | partnertopics | Да | Да |
> | systemtopics | Да | Да |
> | topics | Да | Да |
> | topictypes | нет | нет |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | clusters | Да | Да |
> | пространства имен | Да | Да |
> | sku | нет | нет |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | experimentworkspaces | нет | нет |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | пространства имен | Да | Да |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | featureproviders | нет | нет |
> | features | нет | нет |
> | providers | нет | нет |
> | subscriptionfeatureregistrations | нет | нет |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | automanagedaccounts | нет | нет |
> | automanagedvmconfigurationprofiles | нет | нет |
> | guestconfigurationassignments | нет | нет |
> | software | нет | нет |
> | softwareupdateprofile | нет | нет |
> | softwareupdates | нет | нет |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | hanainstances | нет | нет |
> | sapmonitors | Нет | Нет |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | dedicatedhsms | нет | нет |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!IMPORTANT]
> Кластеры HDInsight можно переместить в новую подписку или группу ресурсов. Однако перемещение между подписками недоступно для сетевых ресурсов, связанных с кластером HDInsight (таких как виртуальная сеть, сетевой адаптер или подсистема балансировки нагрузки). Также невозможно переместить в новую группу ресурсов сетевой адаптер, который подключен к виртуальной машине кластера.
>
> При перемещении кластера HDInsight в новую подписку сначала необходимо переместить другие ресурсы (такие как учетная запись хранения). А затем переместить сам кластер HDInsight.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | clusters | Да | Да |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | services; | Да | Да |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | machines | Да | Да |
> | machines / extensions | Да | Да |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | datamanagers | Да | Да |

## <a name="microsofthybridnetwork"></a>Microsoft. Хибриднетворк

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | устройства | нет | нет |
> | vnfs | нет | нет |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | components | нет | нет |
> | networkscopes | нет | нет |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | jobs | Да | Да |

## <a name="microsoftinsights"></a>microsoft.insights

> [!IMPORTANT]
> Убедитесь, что переход на новую подписку не превышает [квоты для подписки](azure-subscription-service-limits.md#azure-monitor-limits).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | actiongroups | Да | Да |
> | activitylogalerts | нет | нет |
> | alertrules | Да | Да |
> | autoscalesettings | Да | Да |
> | baseline | нет | нет |
> | components | Да | Да |
> | datacollectionrules | нет | нет |
> | diagnosticsettings | нет | нет |
> | diagnosticsettingscategories | нет | нет |
> | eventcategories | нет | нет |
> | eventtypes | нет | нет |
> | extendeddiagnosticsettings | нет | нет |
> | guestdiagnosticsettings | нет | нет |
> | listmigrationdate | нет | нет |
> | logdefinitions | нет | нет |
> | logprofiles | нет | нет |
> | журналы | нет | нет |
> | metricalerts | нет | нет |
> | metricbaselines | нет | нет |
> | metricbatch | нет | нет |
> | metricdefinitions | нет | нет |
> | metricnamespaces | нет | нет |
> | Метрики | нет | нет |
> | migratealertrules | нет | нет |
> | migratetonewpricingmodel | нет | нет |
> | myworkbooks | нет | нет |
> | notificationgroups | нет | нет |
> | privatelinkscopes | нет | нет |
> | rollbacktolegacypricingmodel | нет | нет |
> | scheduledqueryrules | Да | Да |
> | Топология | нет | нет |
> | транзакции | нет | нет |
> | vminsightsonboardingstatuses | нет | нет |
> | webtests | Да | Да |
> | веб-тесты и жеттестресултфиле | Нет | Нет |
> | workbooks | Да | Да |
> | workbooktemplates | Да | Да |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | apptemplates | нет | нет |
> | iotapps | Да | Да |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | graph | Да | Да |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> Хранилища ключей, используемые для шифрования дисков, невозможно переместить в группы ресурсов в той же подписке или между подписками.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | deletedvaults | нет | нет |
> | hsmpools | нет | Нет |
> | манажедхсмс | Нет | нет |
> | vaults | Да | Да |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | connectedclusters | Да | Да |
> | registeredsubscriptions | нет | нет |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | sourcecontrolconfigurations | нет | нет |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | clusters | Да | Да |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | labaccounts | нет | нет |
> | users | нет | нет |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | hostingenvironments | нет | нет |
> | integrationaccounts | Да | Да |
> | integrationserviceenvironments | Да | нет |
> | integrationserviceenvironments / managedapis | Да | нет |
> | isolatedenvironments | нет | нет |
> | workflows | Да | Да |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | commitmentplans | нет | нет |
> | webservices | Да | нет |
> | workspaces | Да | Да |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | operationalizationclusters | нет | нет |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |
> | teamaccounts | нет | нет |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | workspaces | нет | нет |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | configurationassignments | нет | нет |
> | maintenanceconfigurations | Да | Да |
> | updates | нет | нет |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | удостоверения; | нет | нет |
> | userassignedidentities | нет | нет |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | managednetworks | нет | нет |
> | managednetworks / managednetworkgroups | нет | нет |
> | managednetworks / managednetworkpeeringpolicies | нет | Нет |
> | уведомление | нет | нет |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | marketplaceregistrationdefinitions | нет | нет |
> | registrationassignments | нет | нет |
> | registrationdefinitions | нет | нет |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | getentities | нет | нет |
> | managementgroups | нет | нет |
> | managementgroups / settings | нет | нет |
> | ресурсов | нет | нет |
> | starttenantbackfill | нет | нет |
> | tenantbackfillstatus | нет | нет |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | Да | Да |
> | accounts / privateatlases | Да | Да |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | offers | нет | нет |
> | offertypes | нет | нет |
> | privategalleryitems | нет | нет |
> | privatestoreclient | нет | нет |
> | privatestores | нет | нет |
> | products | нет | нет |
> | publishers | нет | нет |
> | регистрация | нет | нет |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | classicdevservices | нет | нет |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | agreements | нет | нет |
> | offertypes | нет | нет |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | mediaservices | Да | Да |
> | mediaservices / liveevents | Да | Да |
> | mediaservices / streamingendpoints | Да | Да |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | appclusters | нет | нет |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | assessmentprojects | Нет | нет |
> | migrateprojects | Нет | Нет |
> | movecollections | нет | нет |
> | projects | нет | нет |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | holographicsbroadcastaccounts | нет | нет |
> | objectunderstandingaccounts | нет | нет |
> | remoterenderingaccounts | Да | Да |
> | spatialanchorsaccounts | Да | Да |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | netappaccounts | нет | нет |
> | netappaccounts / capacitypools | нет | нет |
> | netappaccounts / capacitypools / volumes | нет | нет |

## <a name="microsoftnetwork"></a>Microsoft.Network.

> [!IMPORTANT]
> См. [руководство по перемещению сети](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | applicationgateways | нет | нет |
> | applicationgatewaywebapplicationfirewallpolicies | нет | нет |
> | applicationsecuritygroups | Да | Да |
> | azurefirewalls | нет | нет |
> | bastionhosts | нет | нет |
> | bgpservicecommunities | нет | нет |
> | connections | Да | Да |
> | ddoscustompolicies | Да | Да |
> | ddosprotectionplans | нет | нет |
> | dnszones | Да | Да |
> | expressroutecircuits | нет | нет |
> | expressroutegateways | нет | нет |
> | expressrouteserviceproviders | нет | нет |
> | firewallpolicies | Да | Да |
> | frontdoors | нет | Нет |
> | ипаллокатионс | Да | Да |
> | ipgroups | Да | Да |
> | loadbalancers | Да — Базовый SKU<br> Да — SKU "Стандартный" | Да — Базовый SKU<br>Нет — Стандартный SKU |
> | localnetworkgateways | Да | Да |
> | natgateways | Нет | Нет |
> | networkexperimentprofiles | нет | нет |
> | networkintentpolicies | Да | Да |
> | networkinterfaces | Да | Да |
> | networkprofiles | нет | нет |
> | networksecuritygroups | Да | Да |
> | networkwatchers | Нет | нет |
> | networkwatchers / connectionmonitors | Да | нет |
> | networkwatchers / flowlogs | Да | нет |
> | networkwatchers / pingmeshes | Да | нет |
> | p2svpngateways | нет | нет |
> | privatednszones | Да | Да |
> | privatednszones / virtualnetworklinks | Да | Да |
> | privatednszonesinternal | нет | нет |
> | privateendpointredirectmaps | нет | нет |
> | privateendpoints | Нет | Нет |
> | privatelinkservices | нет | нет |
> | publicipaddresses | Да — Базовый SKU<br>Да — SKU "Стандартный" | Да — Базовый SKU<br>Нет — Стандартный SKU |
> | publicipprefixes | Да | Да |
> | routefilters | нет | нет |
> | routetables | Да | Да |
> | securitypartnerproviders | Да | Да |
> | serviceendpointpolicies | Да | Да |
> | trafficmanagergeographichierarchies | нет | нет |
> | trafficmanagerprofiles | Да | Да |
> | trafficmanagerprofiles / heatmaps | нет | нет |
> | trafficmanagerusermetricskeys | нет | нет |
> | virtualhubs | нет | нет |
> | virtualnetworkgateways | Да | Да |
> | virtualnetworks | Да | Да |
> | virtualnetworktaps | нет | нет |
> | virtualrouters | Да | Да |
> | virtualwans | нет | нет |
> | vpngateways (Virtual WAN) | нет | нет |
> | vpnserverconfigurations | нет | нет |
> | vpnsites (Virtual WAN) | нет | Нет |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | пространства имен | Да | Да |
> | namespaces / notificationhubs | Да | Да |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | osnamespaces | Да | Да |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | hypervsites | Нет | Нет |
> | importsites | Нет | нет |
> | serversites | Нет | Нет |
> | vmwaresites | Нет | Нет |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!IMPORTANT]
> Убедитесь, что переход на новую подписку не превышает [квоты для подписки](azure-subscription-service-limits.md#azure-monitor-limits).
>
> Рабочие области, имеющие связанную учетную запись автоматизации, нельзя перемещать. Прежде чем начать операцию перемещения, не забудьте удалить связь с учетными записями службы автоматизации.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | clusters | нет | нет |
> | deletedworkspaces | нет | нет |
> | linktargets | нет | нет |
> | storageinsightconfigs | нет | нет |
> | workspaces | Да | Да |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | managementassociations | нет | нет |
> | managementconfigurations | Да | Да |
> | solutions | Да | Да |
> | узел "Представления" | Да | Да |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | legacypeerings | нет | нет |
> | peerasns | нет | нет |
> | peeringlocations | нет | нет |
> | peerings | Нет | Нет |
> | peeringservicecountries | нет | нет |
> | peeringservicelocations | нет | нет |
> | peeringserviceproviders | нет | нет |
> | peeringservices | нет | нет |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | policyevents | нет | нет |
> | policystates | нет | нет |
> | policytrackedresources | нет | нет |
> | remediations | нет | нет |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | consoles | нет | нет |
> | dashboards | Да | Да |
> | usersettings | нет | нет |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | workspacecollections | Да | Да |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | capacities | Да | Да |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | availableaccounts | нет | нет |
> | providerregistrations | нет | нет |
> | rollouts | нет | нет |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | workspaces | нет | нет |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!IMPORTANT]
> Ознакомьтесь с разделом [Руководство по перемещению служб восстановления](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | replicationeligibilityresults | нет | нет |
> | vaults | Да | Да |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | openshiftclusters | нет | нет |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | пространства имен | Да | Да |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | Запросы | Да | Да |
> | resourcechangedetails | нет | нет |
> | resourcechanges | нет | нет |
> | ресурсов | нет | нет |
> | resourceshistory | нет | нет |
> | subscriptionsstatus | нет | нет |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | childresources | нет | нет |
> | emergingissues | нет | нет |
> | события | нет | нет |
> | метаданные | нет | нет |
> | Уведомления | нет | нет |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | deployments | нет | нет |
> | deploymentscripts | нет | нет |
> | deploymentscripts / logs | нет | нет |
> | ссылки | нет | нет |
> | providers | нет | нет |
> | resourcegroups | нет | нет |
> | ресурсов | нет | нет |
> | subscriptions | нет | нет |
> | tags | Нет | нет |
> | templatespecs | нет | нет |
> | templatespecs / versions | нет | нет |
> | tenants | нет | нет |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | веб-масштабированием; | Да | нет |
> | saasresources | нет | нет |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!IMPORTANT]
> Вы не можете за одну операцию переместить ресурсы поиска, размещенные в разных регионах. Переместите их в отдельных операциях.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | resourcehealthmetadata | нет | нет |
> | searchservices | Да | Да |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | adaptivenetworkhardenings | нет | нет |
> | advancedthreatprotectionsettings | нет | нет |
> | оповещения | нет | нет |
> | allowedconnections | нет | нет |
> | applicationwhitelistings | нет | нет |
> | assessmentmetadata | нет | нет |
> | assessments | нет | нет |
> | autodismissalertsrules | нет | нет |
> | automations | Да | Да |
> | autoprovisioningsettings | нет | нет |
> | complianceresults | нет | нет |
> | compliances | нет | нет |
> | datacollectionagents | нет | нет |
> | devicesecuritygroups | нет | нет |
> | discoveredsecuritysolutions | нет | нет |
> | externalsecuritysolutions | нет | нет |
> | informationprotectionpolicies | нет | нет |
> | iotsecuritysolutions | Да | Да |
> | iotsecuritysolutions / analyticsmodels | нет | нет |
> | iotsecuritysolutions / analyticsmodels / aggregatedalerts | нет | нет |
> | iotsecuritysolutions / analyticsmodels / aggregatedrecommendations | нет | нет |
> | jitnetworkaccesspolicies | нет | нет |
> | политики | нет | нет |
> | pricings | нет | нет |
> | regulatorycompliancestandards | нет | нет |
> | regulatorycompliancestandards / regulatorycompliancecontrols | нет | нет |
> | regulatorycompliancestandards / regulatorycompliancecontrols / regulatorycomplianceassessments | нет | нет |
> | securitycontacts | нет | нет |
> | securitysolutions | нет | нет |
> | securitysolutionsreferencedata | нет | нет |
> | securitystatuses | нет | нет |
> | securitystatusessummaries | нет | нет |
> | servervulnerabilityassessments | нет | нет |
> | Параметры | нет | нет |
> | subassessments | нет | нет |
> | задачи | нет | нет |
> | topologies | нет | нет |
> | workspacesettings | нет | нет |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | aggregations | нет | нет |
> | alertrules | нет | нет |
> | alertruletemplates | нет | Нет |
> | аутоматионрулес | Нет | нет |
> | bookmarks | нет | нет |
> | cases | нет | нет |
> | dataconnectors | нет | нет |
> | Сущности | нет | нет |
> | entityqueries | нет | нет |
> | incidents | нет | нет |
> | officeconsents | нет | нет |
> | Параметры | нет | нет |
> | threatintelligence | нет | нет |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | consoleservices | нет | нет |

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | gateways | нет | нет |
> | Узлы | нет | нет |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | пространства имен | Да | Да |
> | premiummessagingregions | нет | нет |
> | sku | нет | нет |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | веб-масштабированием; | нет | нет |
> | clusters | Да | Да |
> | containergroups | нет | нет |
> | containergroupsets | нет | нет |
> | edgeclusters | нет | нет |
> | managedclusters | нет | нет |
> | networks | нет | нет |
> | secretstores | нет | нет |
> | volumes. | нет | нет |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | веб-масштабированием; | Да | Да |
> | containergroups | нет | нет |
> | gateways | Да | Да |
> | networks | Да | Да |
> | секретные коды | Да | Да |
> | volumes. | Да | Да |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | rollouts | нет | нет |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | signalr | Да | Да |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | hybridusebenefits | нет | нет |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | applicationdefinitions | нет | нет |
> | веб-масштабированием; | нет | нет |
> | jitrequests | нет | нет |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> База данных и сервер должны быть в одной группе ресурсов. При перемещении сервера SQL Server все его базы данных также перемещаются. Это поведение применяется к базам данных SQL Azure и Azure синапсе Analytics.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | instancepools | нет | нет |
> | Расположения | Да | Да |
> | managedinstances | нет | нет |
> | servers | Да | Да |
> | servers / databases | Да | Да |
> | servers / databases / backuplongtermretentionpolicies | Да | Да |
> | servers / elasticpools | Да | Да |
> | servers / jobaccounts | Да | Да |
> | servers / jobagents | Да | Да |
> | virtualclusters | Да | Да |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | sqlvirtualmachinegroups | Да | Да |
> | sqlvirtualmachines | Да | Да |

## <a name="microsoftstorage"></a>Microsoft.Storage;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | storageaccounts | Да | Да |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | caches | нет | нет |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | Да | Да |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | нет | нет |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | нет | нет |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | managers | нет | нет |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!IMPORTANT]
> Задания Stream Analytics в состоянии выполнения нельзя переместить.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | clusters | нет | нет |
> | streamingjobs | Да | Да |

## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | environments | нет | нет |
> | instances | нет | нет |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | subscriptions | нет | нет |

## <a name="microsoftsupport"></a>microsoft.support

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | services; | нет | нет |
> | supporttickets | нет | нет |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | workspaces | нет | Нет |
> | workspaces / bigdatapools | Нет | нет |
> | workspaces / sqlpools | Нет | нет |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | environments | Да | Да |
> | environments / eventsources | Да | Да |
> | environments / referencedatasets | Да | Да |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | stores | Да | Да |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | imagetemplates | нет | нет |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio

> [!IMPORTANT]
> Чтобы изменить подписку на Azure DevOps, см. статью [Изменить подписку Azure, используемую для выставления счетов](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | account | нет | нет |
> | account / extension | нет | нет |
> | account / project | нет | Нет |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | arczones | нет | нет |
> | resourcepools | нет | нет |
> | vcenters | нет | нет |
> | virtualmachines | нет | нет |
> | virtualmachinetemplates | нет | нет |
> | virtualnetworks | нет | нет |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | dedicatedcloudnodes | нет | нет |
> | dedicatedcloudservices | нет | нет |
> | virtualmachines | нет | нет |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | устройства | нет | нет |
> | vnfs | нет | нет |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | accounts | нет | нет |
> | Планы | нет | нет |
> | registeredsubscriptions | нет | нет |

## <a name="microsoftweb"></a>Microsoft.Web

> [!IMPORTANT]
> См. статью [Руководство по перемещению службы приложений](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | availablestacks | нет | нет |
> | billingmeters | нет | нет |
> | certificates | нет | Да |
> | connectiongateways | Да | Да |
> | connections | Да | Да |
> | customapis | Да | Да |
> | deletedsites | нет | нет |
> | deploymentlocations | нет | нет |
> | georegions | нет | нет |
> | hostingenvironments | нет | нет |
> | kubeenvironments | Да | Да |
> | publishingusers | нет | нет |
> | к просмотру фильмов | нет | нет |
> | resourcehealthmetadata | нет | нет |
> | runtimes | нет | нет |
> | serverfarms | Да | Да |
> | serverfarms / eventgridfilters | нет | нет |
> | sites | Да | Да |
> | sites / premieraddons | Да | Да |
> | sites / slots | Да | Да |
> | sourcecontrols | нет | нет |
> | staticsites | нет | Нет |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | multipleactivationkeys | нет | нет |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | deviceservices | нет | Нет |

## <a name="microsoftworkloadbuilder"></a>Microsoft. Ворклоадбуилдер

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | рабочие нагрузки | Нет | нет |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Тип ресурса | Группа ресурсов | Подписка |
> | ------------- | ----------- | ---------- |
> | components | нет | нет |
> | componentssummary | нет | нет |
> | monitorinstances | нет | нет |
> | monitorinstancessummary | нет | нет |
> | monitors | нет | нет |

## <a name="third-party-services"></a>Сторонние службы

Сейчас для сторонних служб не поддерживается операция перемещения.

## <a name="next-steps"></a>Дальнейшие действия

Команды для перемещения ресурсов см. в статье [Перемещение ресурсов в новую группу ресурсов или подписку](move-resource-group-and-subscription.md).

Чтобы получить данные, идентичные значению файла с разделителями-запятыми, необходимо скачать [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv).

---
title: Поддержка перемещения ресурсов Azure в разных регионах
description: Список типов ресурсов Azure, которые можно перемещать между регионами Azure.
author: rayne-wiselman
ms.service: azure-resource-manager
ms.topic: reference
ms.date: 08/25/2020
ms.author: raynew
ms.openlocfilehash: 18d4d84462d528b718d784ff6a16ecf990ed0d20
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100094021"
---
# <a name="support-for-moving-azure-resources-across-regions"></a>Поддержка перемещения ресурсов Azure в разных регионах

В этой статье мы подтверждаем, поддерживает ли тип ресурсов Azure перемещение в другой регион Azure. 

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
> - [Microsoft.Purview](#microsoftpurview)
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
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | domainservices | нет | 


## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | diagnosticsettings | нет |
> | diagnosticsettingscategories | нет |
> | privatelinkforazuread | нет |
> | tenants |  нет |

## <a name="microsoftaddons"></a>NNTP. Надстройки

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | supportproviders | нет |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | aadsupportcases | нет |
> | addsservices | нет | 
> | agents | нет | 
> | anonymousapiusers | нет |
> | настройка | нет | 
> | журналы | нет | 
> | reports | нет | 
> | servicehealthmetrics | нет | 
> | services; | нет | 

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | конфигурации | нет | 
> | generaterecommendations | нет |
> | метаданные | нет |
> | к просмотру фильмов | нет |
> | suppressions | нет | 

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | actionrules | нет | 
> | оповещения | нет | 
> | alertslist | нет | 
> | alertsmetadata | нет | 
> | alertssummary | нет | 
> | alertssummarylist | нет | 
> | smartdetectoralertrules | Нет | 
> | smartgroups | нет | 

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | servers | Нет |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | reportfeedback | нет |
> | служба |  Да (с использованием шаблона) <br/><br/> [Перемещение управления API между регионами](../../api-management/api-management-howto-migrate.md). | 

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | configurationstores | Нет | 
> | configurationstores / eventgridfilters | нет |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | spring | Нет | 

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | apiapps | Да (с использованием шаблона)<br/><br/> [Перемещение приложения службы приложений в другой регион](../../app-service/manage-move-across-regions.md) | 
> | appidentities | нет | 
> | gateways | нет | 

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | attestationproviders | нет | 

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | classicadministrators | нет | 
> | dataaliases | нет | 
> | denyassignments | нет | 
> | elevateaccess | нет | 
> | findorphanroleassignments | нет | 
> | locks | нет | 
> | разрешения | нет | 
> | policyassignments | нет | 
> | policydefinitions | нет | 
> | policysetdefinitions | нет | 
> | привателинкассоЦиатионс | Нет | 
> | ресаурцеманажементпривателинкс | нет | 
> | roleassignments | нет | 
> | roleassignmentsusagemetrics | нет | 
> | roledefinitions | нет | 

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | automationaccounts | Да (с использованием шаблона) <br/><br/> [Использование георепликации](../../automation/automation-managing-data.md#geo-replication-in-azure-automation) |  
> | automationaccounts / configurations | нет | 
> | automationaccounts / runbooks | Нет | 

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | Подписка |
> | ------------- | ----------- | 
> | privateclouds | нет | 


## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | b2cdirectories | Нет | 
> | b2ctenants | нет | 

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | datacontrollers | нет | 
> | hybriddatamanagers | нет | 
> | postgresinstances | нет | 
> | sqlinstances | нет | 
> | sqlmanagedinstances | нет |
> | sqlserverinstances | нет | 
> | sqlserverregistrations | нет |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | cloudmanifestfiles | нет |
> | registrations | нет | 

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | clusters | нет | 

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | batchaccounts |  Учетные записи пакетной службы нельзя переместить непосредственно из одного региона в другой, но можно использовать шаблон для экспорта шаблона, его изменения и развертывания шаблона в новом регионе. <br/><br/> Сведения о [перемещении учетной записи пакетной службы между регионами](../../batch/best-practices.md#moving-batch-accounts-across-regions) |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | billingaccounts | нет | 
> | billingperiods | нет | 
> | billingpermissions | нет | 
> | billingproperty | нет | 
> | billingroleassignments | нет | 
> | billingroledefinitions | нет | 
> | departments | нет | 
> | enrollmentaccounts | нет | 
> | invoices | нет | 
> | transfers | нет | 

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | mapapis | нет | 

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | biztalk | нет | 

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | blockchainmembers | нет <br/><br/> Сеть блокчейн не может иметь узлы в разных регионах. 
> | cordamembers | нет |
> | watchers | нет | 

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | tokenservices | нет |


## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | blueprintassignments | нет | 
> | blueprints | нет |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | botservices | нет | 

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | redis | нет | 
> | redisenterprise | нет | 

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | appliedreservations | нет | 
> | calculateexchange | нет | 
> | calculateprice | нет | 
> | calculatepurchaseprice | нет | 
> | catalogs | нет | 
> | commercialreservationorders | нет | 
> | обмен валюты | нет |
> | reservationorders | нет | 
> | reservations | нет | 
> | ресурсов | нет | 
> | validatereservationorder | нет | 

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | cdnwebapplicationfirewallpolicies | нет |
> | edgenodes | нет
> | профили | Нет | 
> | profiles / endpoints | нет | 

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | certificateorders | нет | 


## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | capabilities | нет | 
> | domainnames | Да | нет |
> | quotas | нет | 
> | resourcetypes | нет |
> | validatesubscriptionmoveavailability | нет | 
> | virtualmachines | нет 

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | classicinfrastructureresources | нет | 

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | capabilities | нет | 
> | expressroutecrossconnections | нет | 
> | expressroutecrossconnections / peerings | нет | 
> | gatewaysupporteddevices | нет | 
> | networksecuritygroups | нет |
> | quotas | нет |
> | reservedips | нет | 
> | virtualnetworks | нет | 

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | disks | нет | 
> | images | нет | 
> | osimages | нет | 
> | osplatformimages | нет | 
> | publicimages | нет | 
> | quotas | нет | 
> | storageaccounts | Да |  
> | vmimages | нет |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | Операции | нет | 

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 
> | Когнитивный поиск | Поддерживается с ручными действиями.<br/><br/> Дополнительные сведения о [перемещении службы когнитивный Поиск Azure в другой регион](../../search/search-howto-move-across-regions.md)

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | ratecard | нет | 
> | usageaggregates | нет | 

## <a name="microsoftcompute"></a>Microsoft.Compute;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | availabilitysets | Да <br/><br/> Используйте перемещение [ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) для перемещения групп доступности. | 
> | diskaccesses | нет |
> | diskencryptionsets | нет | 
> | disks | Да <br/><br/> Используйте перемещение [ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) для перемещения виртуальных машин Azure и связанных дисков. | 
> | galleries | нет | 
> | galleries / images | нет | 
> | galleries / images / versions | нет | 
> | hostgroups | нет | 
> | hostgroups / hosts | нет | 
> | images | нет | 
> | proximityplacementgroups | Нет | 
> | restorepointcollections | нет | 
> | sharedvmimages | нет | 
> | sharedvmimages / versions | нет | 
> | snapshots | Нет | 
> | sshpublickeys | нет |
> | virtualmachines | Да <br/><br/> Перемещение виртуальных машин Azure с помощью [перемещения ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) . | 
> | virtualmachines / extensions | нет | 
> | virtualmachinescalesets | Нет | 

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | aggregatedcost | нет | 
> | balances | нет | 
> | budgets | нет | 
> | charges | нет | 
> | costtags | нет | 
> | credits | нет | 
> | события | нет | 
> | forecasts | нет | 
> | lots | нет | 
> | marketplaces | нет | 
> | pricesheets | нет | 
> | products | нет | 
> | reservationdetails | нет | 
> | reservationrecommendationdetails | нет | 
> | reservationrecommendations | нет | 
> | reservationsummaries | нет | 
> | reservationtransactions | нет | 
> | tags | нет | 
> | tenants | нет | 
> | terms | нет | 
> | usagedetails | нет | 


## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | containergroups | нет | 
> | serviceassociationlinks | нет |


## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | registries | Нет |  
> | registries / agentpools | нет | 
> | registries / buildtasks | нет |  
> | registries / replications | нет | 
> | registries / tasks | нет |  
> | registries / webhooks | нет | 

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | containerservices | нет |
> | managedclusters | нет | 
> | openshiftmanagedclusters | нет | 

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | веб-масштабированием; | нет | 

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | оповещения | нет | 
> | billingaccounts | нет | 
> | budgets | нет | 
> | cloudconnectors | нет | 
> | соединители | нет | 
> | departments | нет | 
> | dimensions | нет | 
> | enrollmentaccounts | нет | 
> | exports | нет | 
> | externalbillingaccounts | нет | 
> | прогноз | нет | 
> | query | нет | 
> | регистрация | нет | 
> | reportconfigs | нет | 
> | reports | нет | 
> | Параметры | нет | 
> | showbackrules | нет | 
> | узел "Представления" | нет | 

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | hubs | нет |  

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | requests | нет | 

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | associations | нет |
> | resourceproviders | Нет | 

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | jobs | нет | 

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | аваилаблескус | Нет |
> | databoxedgedevices | нет | 

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | workspaces | нет | 

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | catalogs | нет | 
> | datacatalogs | нет | 

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | connectionmanagers | нет | 

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | Пакеты | нет | 
> | Планы | нет | 

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | datafactories | нет | 
> | factories | нет |  

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | datalakeaccounts | нет | 

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | services; | нет | 
> | services / projects | нет | 
> | slots | нет | 

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | ---------- |
> | backupvaults | нет | 

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | servers | Для перемещения существующего сервера можно использовать реплику чтения между регионами. [Подробнее](../../postgresql/howto-move-regions-portal.md).<br/><br/> Если служба подготовлена с геоизбыточным хранилищем резервных копий, можно использовать геовосстановление для восстановления в других регионах. [Подробнее](../../mariadb/concepts-business-continuity.md#recover-from-an-azure-regional-data-center-outage).

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | servers | Для перемещения существующего сервера можно использовать реплику чтения между регионами. [Подробнее](../../mysql/howto-move-regions-portal.md).

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | servergroups | нет | 
> | servers | Для перемещения существующего сервера можно использовать реплику чтения между регионами. Дополнительные [сведения](../../postgresql/howto-move-regions-portal.md).
> | serversv2 | Нет | 

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | artifactsources | нет | 
> | rollouts | нет |  
> | servicetopologies | Нет | 
> | servicetopologies / services | Нет |  
> | servicetopologies / services / serviceunits | Нет | 
> | steps | Нет | 


## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | applicationgroups | нет | 
> | workspaces | нет | 

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | elasticpools | Нет. Ресурс не предоставлен.
> | elasticpools / iothubtenants | Нет. Ресурс не предоставлен.
> | iothubs | Да. [Дополнительные сведения](../../iot-hub/iot-hub-how-to-clone.md)
> | provisioningservices | нет | 

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | controllers | Нет | 


## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | controllers | Нет | 
> | Кластер AKS | Нет<br/><br/> Дополнительные [сведения](../../dev-spaces/faq.md#can-i-migrate-my-aks-cluster-with-azure-dev-spaces-to-another-region) о переходе в другой регион.

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | labcenters | нет | 
> | labs | нет | 
> | labs / environments | Нет |  
> | labs / servicerunners | Нет | 
> | labs / virtualmachines | нет |  
> | schedules | Нет |  

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | digitaltwinsinstances | Да, путем повторного создания ресурсов в новом регионе. [Дополнительные сведения](../../digital-twins/how-to-move-regions.md) |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | databaseaccounts | нет | 
> | databaseaccounts | нет | 

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | domains | нет | 
> | generatessorequest | нет | 
> | topleveldomains | нет | 
> | validatedomainregistrationinformation | нет |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | services; | нет |  

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | domains | нет | 
> | eventsubscriptions | Нет |
> | extensiontopics | нет | 
> | partnernamespaces | нет | 
> | partnerregistrations | нет | 
> | partnertopics | нет | 
> | systemtopics | нет | 
> | topics | Нет | 
> | topictypes | нет | 

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | clusters | нет |  
> | пространства имен | Да (с шаблоном)<br/><br/> [Перемещение пространства имен концентратора событий в другой регион](../../event-hubs/move-across-regions.md) | 
> | sku | нет |  

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | experimentworkspaces | нет | 

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | пространства имен | Нет | 

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | featureproviders | нет | 
> | features | нет | 
> | providers | нет | 
> | subscriptionfeatureregistrations | нет | 

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | automanagedaccounts | нет | 
> | automanagedvmconfigurationprofiles | нет | 
> | guestconfigurationassignments | нет | 
> | software | нет | 
> | softwareupdateprofile | нет | 
> | softwareupdates | нет | 

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | hanainstances | нет | 
> | sapmonitors | Нет |  

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | dedicatedhsms | нет | 


## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | clusters | нет | 

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | services; | нет |  

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | machines | Нет | 
> | machines / extensions | нет |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | datamanagers |  Нет | 

## <a name="microsofthybridnetwork"></a>Microsoft. Хибриднетворк

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | устройства | нет | 
> | vnfs | нет | 

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | components | нет | 
> | networkscopes | нет | 

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | jobs |  нет | 

## <a name="microsoftinsights"></a>microsoft.insights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | Нет. [Подробнее](../../azure-monitor/faq.md#how-do-i-move-an-application-insights-resource-to-a-new-region).
> | actiongroups |  Нет | 
> | activitylogalerts | нет | 
> | alertrules |  нет | 
> | autoscalesettings |  Нет | 
> | baseline | нет |
> | components |  нет |  
> | datacollectionrules | нет | 
> | diagnosticsettings | нет | 
> | diagnosticsettingscategories | нет | 
> | eventcategories | нет | 
> | eventtypes | нет | 
> | extendeddiagnosticsettings | нет | |
> | guestdiagnosticsettings | нет | 
> | listmigrationdate | нет | 
> | logdefinitions | нет | 
> | logprofiles | нет | 
> | журналы | нет | нет |
> | metricalerts | нет | 
> | metricbaselines | нет | 
> | metricbatch | нет | 
> | metricdefinitions | нет | 
> | metricnamespaces | нет | 
> | Метрики | нет | 
> | migratealertrules | нет |
> | migratetonewpricingmodel | нет | 
> | myworkbooks | нет |
> | notificationgroups | нет | 
> | privatelinkscopes | нет |
> | rollbacktolegacypricingmodel | нет |
> | scheduledqueryrules |  Нет | 
> | Топология | нет |
> | транзакции | нет |
> | vminsightsonboardingstatuses | нет |
> | webtests |  Нет | 
> | веб-тесты и жеттестресултфиле | Нет |
> | workbooks |  Нет |  
> | workbooktemplates | Нет |


## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | apptemplates | нет | 
> | iotapps | Нет | 



## <a name="microsoftiothub"></a>Microsoft.IoTHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> |  iothub |  Да (концентратор клонирования) <br/><br/> [Клонирование центра Интернета вещей в другой регион](../../iot-hub/iot-hub-how-to-clone.md)

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона |
> | ------------- | ----------- | 
> | graph | нет | 

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | deletedvaults | нет |
> | hsmpools | нет | 
> | манажедхсмс | нет |
> | vaults |  Нет | 

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | connectedclusters | нет | 
> | registeredsubscriptions | нет | 

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | sourcecontrolconfigurations | нет | 

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | clusters |  нет |  

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | labaccounts | нет | 
> | users | нет | 

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | Нет, это глобальная служба.

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | hostingenvironments | нет | 
> | integrationaccounts |  Нет |  
> | integrationserviceenvironments | нет | 
> | integrationserviceenvironments / managedapis | нет |
> | isolatedenvironments | нет | 
> | workflows |  Нет |  

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | commitmentplans |  нет | 
> | webservices |  нет | 
> | workspaces |  нет | 

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | operationalizationclusters |  нет | 

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 
> | teamaccounts | нет | 


## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 


## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | workspaces | нет | 

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | configurationassignments | Да. [Дополнительные сведения](../../virtual-machines/move-region-maintenance-configuration.md) | 
> | maintenanceconfigurations | Да. [Дополнительные сведения](../../virtual-machines/move-region-maintenance-configuration-resources.md) |
> | updates | нет | 

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | удостоверения; | нет | 
> | userassignedidentities | нет | 

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | managednetworks | нет | 
> | managednetworks / managednetworkgroups | нет |
> | managednetworks / managednetworkpeeringpolicies | нет | 
> | уведомление | нет | 

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | marketplaceregistrationdefinitions | нет | 
> | registrationassignments | нет |
> | registrationdefinitions | нет | 

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | getentities | нет | 
> | managementgroups | нет | 
> | managementgroups / settings | нет | 
> | ресурсов | нет | 
> | starttenantbackfill | нет | 
> | tenantbackfillstatus | нет | 

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts |  Нет, Azure Maps является геопространственной службой. 
> | accounts / privateatlases | нет

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | offers | нет | 
> | offertypes | нет | 
> | privategalleryitems | нет | 
> | privatestoreclient | нет | 
> | privatestores | нет | 
> | products | нет | 
> | publishers | нет | 
> | регистрация | нет | 

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | classicdevservices | нет | 

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | agreements | нет | 
> | offertypes | нет | 

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | mediaservices |  нет | 
> | mediaservices / liveevents |  нет | 
> | mediaservices / streamingendpoints |  нет | 

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | appclusters | нет | 

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | assessmentprojects | нет | 
> | migrateprojects | Нет | 
> | movecollections | нет
> | projects | нет | 

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | ---------- |
> | holographicsbroadcastaccounts | нет | 
> | objectunderstandingaccounts | нет | 
> | remoterenderingaccounts | Нет | 
> | spatialanchorsaccounts | Нет | 

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | netappaccounts | нет | 
> | netappaccounts / capacitypools | нет | 
> | netappaccounts / capacitypools / volumes | нет | 
> | netappaccounts / capacitypools / volumes / mounttargets | нет | 
> | netappaccounts / capacitypools / volumes / snapshots | нет | 

## <a name="microsoftnetwork"></a>Microsoft.Network.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | applicationgateways | нет |
> | applicationgatewaywebapplicationfirewallpolicies | нет | 
> | applicationsecuritygroups |  нет |  
> | azurefirewalls |  нет |  
> | bastionhosts | нет | 
> | bgpservicecommunities | нет |
> | connections |  Нет | 
> | ddoscustompolicies |  Нет | 
> | ddosprotectionplans | нет | 
> | dnszones |  нет | 
> | expressroutecircuits | нет | 
> | expressroutegateways | нет | 
> | expressrouteserviceproviders | нет | 
> | firewallpolicies | нет |
> | frontdoors | нет | 
> | ипаллокатионс | нет |
> | ipgroups | Нет |
> | loadbalancers | Да <br/><br/> Используйте перемещение [ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) для перемещения внутренних и внешних подсистем балансировки нагрузки. |
> | localnetworkgateways |  нет | 
> | natgateways |  Нет | 
> | networkexperimentprofiles | нет |
> | networkintentpolicies |  Нет | 
> | networkinterfaces | Да <br/><br/> Перемещение сетевых карт с помощью перемещения [ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) . | 
> | networkprofiles | нет | 
> | networksecuritygroups | Да <br/><br/> Используйте перемещение [ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) для перемещения групп безопасности сети (нгсс). | 
> | networkwatchers |  нет |  
> | networkwatchers / connectionmonitors |  нет | 
> | networkwatchers / flowlogs |  нет | 
> | networkwatchers / pingmeshes |  нет | 
> | p2svpngateways | нет | 
> | privatednszones |  нет |  
> | privatednszones / virtualnetworklinks | Нет |> | privatednszonesinternal | нет |
> | privateendpointredirectmaps | нет |
> | privateendpoints | Нет | 
> | privatelinkservices | нет | 
> | publicipaddresses | Да<br/><br/> Используйте перемещение [ресурсов Azure](../../resource-mover/tutorial-move-region-virtual-machines.md) для перемещения общедоступных IP-адресов. |
> | publicipprefixes | Нет | 
> | routefilters | нет | 
> | routetables |  Нет | 
> | securitypartnerproviders | Нет |
> | serviceendpointpolicies |  Нет | 
> | trafficmanagergeographichierarchies | нет | 
> | trafficmanagerprofiles |  нет | 
> | trafficmanagerusermetricskeys | нет |
> | virtualhubs | нет | 
> | virtualnetworkgateways |  нет |  
> | virtualnetworks |  нет | 
> | virtualnetworktaps | нет | 
> | virtualwans | нет | 
> | vpngateways (Virtual WAN) | нет | 
> | vpnsites (Virtual WAN) | нет | 
> | vpnsites (Virtual WAN) | нет |


## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | пространства имен |  Нет | 
> | namespaces / notificationhubs |  нет |  

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | osnamespaces | Нет | 

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | hypervsites | Нет | 
> | importsites | нет | 
> | serversites | Нет | 
> | vmwaresites | Нет | 

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | clusters | нет | 
> | deletedworkspaces | нет | 
> | linktargets | нет | 
> | storageinsightconfigs | нет |
> | workspaces | нет |



## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | managementassociations | нет |
> | managementconfigurations |  нет | 
> | solutions | нет |
> | узел "Представления" |  нет | 

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | legacypeerings | нет | 
> | peerasns | нет | 
> | peeringlocations | нет | 
> | peerings | Нет | 
> | peeringservicecountries | нет | 
> | peeringservicelocations | нет | 
> | peeringserviceproviders | нет | 
> | peeringservices | нет | 

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | policyevents | нет | 
> | policystates | нет | 
> | policytrackedresources | нет | 
> | remediations | нет | 

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | consoles | нет |
> | dashboards | нет | 
> | usersettings | нет | 


## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | workspacecollections |  Нет | 

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | capacities |  Нет | 

## <a name="microsoftpurview"></a>Microsoft.Purview

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | accounts | нет | 

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | availableaccounts | нет | 
> | providerregistrations | нет | 
> | rollouts | нет | 

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | workspaces | нет | 

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | replicationeligibilityresults | нет |
> | vaults | Нет.<br/><br/> Перемещение хранилищ служб восстановления для Azure Backup в регионах Azure не поддерживается.<br/><br/> В хранилищах служб восстановления для Azure Site Recovery можно [Отключить и повторно создать хранилище](../../site-recovery/move-vaults-across-regions.md) в целевом регионе. | 

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | openshiftclusters | нет | 

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | пространства имен |  нет | 

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | Запросы |  Нет |  
> | resourcechangedetails | нет | 
> | resourcechanges | нет | 
> | ресурсов | нет | 
> | resourceshistory | нет | 
> | subscriptionsstatus | нет | 

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | childresources | нет | 
> | emergingissues | нет | 
> | события | нет | 
> | метаданные | нет | 
> | Уведомления | нет | 

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона |
> | ------------- | ----------- |
> | deploymentScripts |  Да<br/><br/>[Перемещение ресурсов Microsoft. Resources в новый регион](microsoft-resources-move-regions.md) |
> | темплатеспекс |  Да<br/><br/>[Перемещение ресурсов Microsoft. Resources в новый регион](microsoft-resources-move-regions.md) |  

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | веб-масштабированием; |  нет | 
> | saasresources | нет | 


## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | resourcehealthmetadata | нет |
> | searchservices |  Нет | 


## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | adaptivenetworkhardenings | нет | 
> | advancedthreatprotectionsettings | нет | 
> | оповещения | нет | 
> | allowedconnections | нет | 
> | applicationwhitelistings | нет | 
> | assessmentmetadata | нет | 
> | assessments | нет | 
> | autodismissalertsrules | нет | 
> | automations | Нет | 
> | autoprovisioningsettings | нет |
> | complianceresults | нет | 
> | compliances | нет | 
> | datacollectionagents | нет | 
> | devicesecuritygroups | нет | 
> | discoveredsecuritysolutions | нет | 
> | externalsecuritysolutions | нет | 
> | informationprotectionpolicies | нет | 
> | iotsecuritysolutions | Нет | 
> | iotsecuritysolutions / analyticsmodels | нет | 
> | iotsecuritysolutions / analyticsmodels / aggregatedalerts | нет | 
> | iotsecuritysolutions / analyticsmodels / aggregatedrecommendations | нет | 
> | jitnetworkaccesspolicies | нет | 
> | политики | нет | 
> | pricings | нет | 
> | regulatorycompliancestandards | нет | 
> | regulatorycompliancestandards / regulatorycompliancecontrols | нет | 
> | regulatorycompliancestandards / regulatorycompliancecontrols / regulatorycomplianceassessments | нет | 
> | securitycontacts | нет | 
> | securitysolutions | нет | 
> | securitysolutionsreferencedata | нет | 
> | securitystatuses | нет | 
> | securitystatusessummaries | нет | 
> | servervulnerabilityassessments | нет | 
> | Параметры | нет | 
> | subassessments | нет |
> | задачи | нет | 
> | topologies | нет | 
> | workspacesettings | нет | 

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | aggregations | нет | 
> | alertrules | нет | 
> | alertruletemplates | нет | 
> | аутоматионрулес | нет |
> | cases | нет | 
> | dataconnectors | нет | 
> | Сущности | нет | 
> | entityqueries | нет |
> | incidents | нет | 
> | officeconsents | нет | 
> | Параметры | нет | 
> | threatintelligence | нет | 

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | consoleservices | нет | 

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | gateways | нет | 
> | Узлы | нет | 

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | пространства имен |  нет | 
> | premiummessagingregions | нет | 
> | sku | нет | 

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | веб-масштабированием; | нет | 
> | clusters |  нет |  
> | containergroups | нет | 
> | containergroupsets | нет | 
> | edgeclusters | нет | 
> | managedclusters | нет |
> | networks | нет | 
> | secretstores | нет | 
> | volumes. | нет | 

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | веб-масштабированием; |  нет | 
> | containergroups | нет | 
> | gateways |  нет | 
> | networks |  нет | 
> | секретные коды |  нет | 
> | volumes. |  нет |  

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | rollouts | нет | 

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | signalr |  нет |  

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | hybridusebenefits | нет | 

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | appliancedefinitions | Нет | 
> | appliances | нет | 
> | jitrequests | нет | 

## <a name="microsoftsql"></a>Microsoft.Sql

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | instancepools | нет | 
> | Расположения | нет |
> | managedinstances | Да <br/><br/> [Дополнительные сведения](../../azure-sql/database/move-resources-across-regions.md) о перемещении управляемых экземпляров между регионами. | 
> | managedinstances / databases | Да | 
> | servers | Да | 
> | servers / databases | Да <br/><br/> [Узнайте больше](../../azure-sql/database/move-resources-across-regions.md) о перемещении баз данных между регионами.<br/><br/> [Дополнительные сведения](../../resource-mover/tutorial-move-region-sql.md) об использовании средства перемещения ресурсов Azure для перемещения баз данных SQL Azure.  | 
> | servers / elasticpools | Да <br/><br/> [Узнайте больше](../../azure-sql/database/move-resources-across-regions.md) о перемещении пулов эластичных БД по регионам.<br/><br/> [Узнайте больше](../../resource-mover/tutorial-move-region-sql.md) об использовании перемещения ресурсов Azure для перемещения эластичных ПУЛОВ Azure SQL.  | 
> | virtualclusters | Да | 

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | sqlvirtualmachinegroups |  нет |  
> | sqlvirtualmachines |  нет |  


## <a name="microsoftstorage"></a>Microsoft.Storage;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | storageaccounts | Да<br/><br/> [Перемещение учетной записи хранения Azure в другой регион](../../storage/common/storage-account-move.md) | 

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | caches | нет | 

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | storagesyncservices |  нет | 

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | storagesyncservices | нет | 

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | storagesyncservices | нет | 

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | managers | нет | 

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | clusters | нет |
> | streamingjobs |  Нет |  


## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | environments | нет | 
> | instances | нет | 

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | subscriptions | нет | 

## <a name="microsoftsupport"></a>microsoft.support

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | services; | нет | 
> | supporttickets | нет | 

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | workspaces | нет | 
> | workspaces / bigdatapools | нет | 
> | workspaces / sqlpools | нет | 


## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | environments |  нет | 
> | environments / eventsources |  нет |  
> | environments / referencedatasets |  нет | 

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | stores | нет | 

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | imagetemplates | нет | 

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | account |  нет | 
> | account / extension |  нет | 
> | account / project |  нет | 

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | arczones | нет | 
> | resourcepools | нет | 
> | vcenters | нет | 
> | virtualmachines | нет | 
> | virtualmachinetemplates | нет | 
> | virtualnetworks | нет | 

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | dedicatedcloudnodes | нет | 
> | dedicatedcloudservices | нет | 
> | virtualmachines | нет | 

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | устройства | нет | 
> | vnfs | нет | 

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | accounts | нет | 
> | Планы | нет | 
> | registeredsubscriptions | нет |


## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | availablestacks | нет | 
> | billingmeters | нет | 
> | certificates | нет | 
> | connectiongateways |  нет |  
> | connections |  Нет |  
> | customapis |  Нет | 
> | deletedsites | нет | 
> | deploymentlocations | нет | 
> | georegions | нет | 
> | hostingenvironments | нет | 
> | kubeenvironments | нет | 
> | publishingusers | нет |
> | к просмотру фильмов | нет | 
> | resourcehealthmetadata | нет | 
> | runtimes | нет | 
> | serverfarms | Нет |  
> | serverfarms / eventgridfilters | N
> | sites |  нет | 
> | sites / premieraddons |  Нет |  
> | sites / slots |  нет |  
> | sourcecontrols | нет |
> | staticsites | нет | 

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | multipleactivationkeys | нет |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- |
> | deviceservices | нет | 

## <a name="microsoftworkloadbuilder"></a>Microsoft. Ворклоадбуилдер

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | рабочие нагрузки | нет | 

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Тип ресурса | Перемещение региона | 
> | ------------- | ----------- | 
> | components | нет |
> | componentssummary | нет | 
> | monitorinstances | нет | 
> | monitorinstancessummary | нет | 
> | monitors | нет | 
## <a name="third-party-services"></a>Сторонние службы

Сейчас для сторонних служб не поддерживается операция перемещения.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные [сведения](../../resource-mover/overview.md) о службе перемещения ресурсов.


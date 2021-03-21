---
title: Удаление ресурсов в полном режиме
description: Здесь показано, как происходит удаление ресурсов в полном режиме в шаблонах Azure Resource Manager по типу ресурса.
ms.topic: conceptual
ms.date: 10/21/2020
ms.openlocfilehash: e0c67bfcda81ad128e0018c4ab37c4b0cbe680f0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96184031"
---
# <a name="deletion-of-azure-resources-for-complete-mode-deployments"></a>Удаление ресурсов Azure для развертываний в полном режиме

В этой статье описывается, как процесс удаления будет обрабатываться различными типами, когда он не происходит в шаблоне, развернутом в завершенном режиме.

Типы ресурсов, помеченные **Да** , удаляются, если тип не находится в шаблоне, развернутом в полном режиме.

Типы ресурсов, помеченные как " **нет** ", не удаляются автоматически при отсутствии в шаблоне. Однако они удаляются, если родительский ресурс удаляется. Чтобы получить полное описание поведения см. статью [Режимы развертывания в Azure Resource Manager](deployment-modes.md).

Если развернуть в [шаблоне несколько групп ресурсов](./deploy-to-resource-group.md), то ресурсы в группе ресурсов, указанной в операции развертывания, могут быть удалены. Ресурсы во вторичных группах ресурсов не удаляются.

Ресурсы перечисляются по пространству имен поставщика ресурсов. Чтобы сопоставить пространство имен поставщика ресурсов с именем службы Azure, см. статью [поставщики ресурсов для служб Azure](../management/azure-services-resource-providers.md).

> [!NOTE]
> Всегда используйте [операцию "что если](template-deploy-what-if.md) " перед развертыванием шаблона в полном режиме. Что если показывает, какие ресурсы будут созданы, удалены или изменены. Используйте действие что если, чтобы избежать непреднамеренного удаления ресурсов.
Переход к пространству имен поставщика ресурсов:
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft. Агфудплатформ](#microsoftagfoodplatform)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft. Управление](#microsoftautomanage)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.Azure.Geneva](#microsoftazuregeneva)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft. Бареметалинфраструктуре](#microsoftbaremetalinfrastructure)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.BlockchainTokens](#microsoftblockchaintokens)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Capacity](#microsoftcapacity)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ChangeAnalysis](#microsoftchangeanalysis)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft. Кодеспацес](#microsoftcodespaces)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft. Коннектедкаче](#microsoftconnectedcache)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft. D365CustomerInsights](#microsoftd365customerinsights)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataFactory](#microsoftdatafactory)
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
> - [Microsoft. Девицеупдате](#microsoftdeviceupdate)
> - [Microsoft.DevOps](#microsoftdevops)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DigitalTwins](#microsoftdigitaltwins)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.DynamicsLcs](#microsoftdynamicslcs)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.Gallery](#microsoftgallery)
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
> - [Microsoft.Intune](#microsoftintune)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
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
> - [Microsoft. Notebooks](#microsoftnotebooks)
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
> - [Microsoft. ScVmm](#microsoftscvmm)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.SecurityGraph](#microsoftsecuritygraph)
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft. Единственное,](#microsoftsingularity)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft. SQL](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft. Сторажерепликатион](#microsoftstoragereplication)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsDefenderATP](#microsoftwindowsdefenderatp)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft. Ворклоадбуилдер](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | DomainServices | Да |
> | DomainServices/ауконтаинер | нет |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | supportProviders | нет |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
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
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | адвисорскоре | Нет |
> | конфигурации | нет |
> | generateRecommendations | нет |
> | метаданные | нет |
> | к просмотру фильмов | нет |
> | suppressions | нет |

## <a name="microsoftagfoodplatform"></a>Microsoft. Агфудплатформ

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | фармбеатс | Да |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | actionRules | Да |
> | оповещения | нет |
> | alertsList | Нет |
> | алертсметадата | Нет |
> | alertsSummary | Нет |
> | alertsSummaryList | Нет |
> | smartDetectorAlertRules | Да |
> | smartGroups | нет |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | servers | Да |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | reportFeedback | Нет |
> | служба | Да |
> | validateServiceName | Нет |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | configurationStores | Да |
> | Конфигуратионсторес/Евентгридфилтерс | Нет |
> | Конфигуратионсторес/Кэйвалуес | нет |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Spring | Да |
> | Пружина/приложения | Нет |
> | Пружина/приложения/развертывания | Нет |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | attestationProviders | Да |
> | дефаултпровидерс | нет |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | акцессревиевсчедуледефинитионс | Нет |
> | акцессревиевсчедулесеттингс | Нет |
> | classicAdministrators | Нет |
> | Псевдонимы | Нет |
> | denyAssignments | Нет |
> | elevateAccess | Нет |
> | финдорфанролеассигнментс | нет |
> | locks | нет |
> | разрешения | нет |
> | policyAssignments | Нет |
> | policyDefinitions | Нет |
> | полициексемптионс | Нет |
> | policySetDefinitions | Нет |
> | привателинкассоЦиатионс | Нет |
> | providerOperations | Нет |
> | ресаурцеманажементпривателинкс | Да |
> | roleAssignments | Нет |
> | ролеассигнментсусажеметрикс | Нет |
> | roleDefinitions | Нет |

## <a name="microsoftautomanage"></a>Microsoft. Управление

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | конфигуратионпрофилеассигнментс | Нет |
> | конфигуратионпрофилепреференцес | Да |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | automationAccounts | Да |
> | automationAccounts и конфигурации | Да |
> | automationAccounts и задания | Нет |
> | automationAccounts/Приватиндпоинтконнектионпроксиес | Нет |
> | automationAccounts/Приватиндпоинтконнектионс | Нет |
> | automationAccounts/Привателинкресаурцес | Нет |
> | automationAccounts / runbooks | Да |
> | automationAccounts/Софтвареупдатеконфигуратионс | Нет |
> | automationAccounts / webhooks | Нет |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | приватеклаудс | Да |
> | Приватеклаудс и надстройки | Нет |
> | Приватеклаудс и авторизация | Нет |
> | Приватеклаудс и кластеры | Нет |
> | Приватеклаудс/Глобалреачконнектионс | Нет |
> | Приватеклаудс/Хкксентерприсеситес | Нет |
> | Приватеклаудс/Ворклоаднетворкс | Нет |
> | Приватеклаудс/Ворклоаднетворкс/Дхкпконфигуратионс | Нет |
> | Приватеклаудс/Ворклоаднетворкс/шлюзы | Нет |
> | Приватеклаудс/Ворклоаднетворкс/Портмиррорингпрофилес | Нет |
> | Приватеклаудс/Ворклоаднетворкс/сегменты | Нет |
> | Приватеклаудс/Ворклоаднетворкс/virtualMachines | Нет |
> | Приватеклаудс/Ворклоаднетворкс/Вмграупс | Нет |

## <a name="microsoftazuregeneva"></a>Microsoft.Azure.Geneva

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | environments | нет |
> | среды и учетные записи | Нет |
> | среды, учетные записи и пространства имен | Нет |
> | среды, учетные записи, пространства имен и конфигурации | нет |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | b2cDirectories | Да |
> | b2ctenants | нет |
> | гуестусажес | Да |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Контроллеры | Да |
> | постгресинстанцес | Да |
> | склманажединстанцес | Да |
> | склсерверинстанцес | Да |
> | склсерверрегистратионс | Да |
> | Склсерверрегистратионс и sqlServer | нет |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | клаудманифестфилес | Нет |
> | еджесубскриптионс | Да |
> | линкедсубскриптионс | Да |
> | registrations | Да |
> | регистрации и Кустомерсубскриптионс | нет |
> | registrations / products | нет |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | clusters | Да |

## <a name="microsoftbaremetalinfrastructure"></a>Microsoft. Бареметалинфраструктуре

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | бареметалинстанцес | Да |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | batchAccounts | Да |
> | batchAccounts / certificates | Нет |
> | batchAccounts / pools | нет |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | billingAccounts | Нет |
> | Биллингаккаунтс и соглашения | Нет |
> | Биллингаккаунтс/Биллингпермиссионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингпермиссионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингролеассигнментс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингроледефинитионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингсубскриптионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Креатебиллингролеассигнмент | Нет |
> | Биллингаккаунтс/Биллингпрофилес/клиенты | Нет |
> | Биллингаккаунтс/Биллингпрофилес/инструкции | Нет |
> | Биллингаккаунтс/Биллингпрофилес/счета | Нет |
> | Биллингаккаунтс/Биллингпрофилес/счета/прайс-лист | Нет |
> | Биллингаккаунтс/Биллингпрофилес/счета/транзакции | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингпермиссионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингролеассигнментс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингроледефинитионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингсубскриптионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Креатебиллингролеассигнмент | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Инитиатетрансфер | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Products | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Products/перемещение | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Products/Упдатеауторенев | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/транзакции | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/передача | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Патчоператионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Пайментмесодс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/политики | Нет |
> | Биллингаккаунтс/Биллингпрофилес/прайс-лист | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Прицешитдовнлоадоператионс | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Products | Нет |
> | Биллингаккаунтс/Биллингпрофилес/резервирования | Нет |
> | Биллингаккаунтс/Биллингпрофилес/транзакции | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Валидатедетачпайментмесоделигибилити | Нет |
> | Биллингаккаунтс/Биллингролеассигнментс | Нет |
> | Биллингаккаунтс/Биллингроледефинитионс | Нет |
> | Биллингаккаунтс/Биллингсубскриптионс | Нет |
> | Биллингаккаунтс/Биллингсубскриптионс/счета | Нет |
> | Биллингаккаунтс/Креатебиллингролеассигнмент | Нет |
> | Биллингаккаунтс/Креатеинвоицесектионоператионс | Нет |
> | Биллингаккаунтс и клиенты | Нет |
> | Биллингаккаунтс/Customers/Биллингпермиссионс | Нет |
> | Биллингаккаунтс/Customers/Биллингсубскриптионс | Нет |
> | Биллингаккаунтс/Customers/Инитиатетрансфер | Нет |
> | Биллингаккаунтс, клиенты и политики | Нет |
> | Биллингаккаунтс/Customers/Products | Нет |
> | Биллингаккаунтс/Customers/Transactions | Нет |
> | Биллингаккаунтс/Customers/Transfers | Нет |
> | Биллингаккаунтс и отделы | Нет |
> | Биллингаккаунтс/Departments/Биллингпермиссионс | Нет |
> | Биллингаккаунтс/Departments/Биллингролеассигнментс | Нет |
> | Биллингаккаунтс/Departments/Биллингроледефинитионс | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс/Биллингпермиссионс | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс/Биллингролеассигнментс | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс/Биллингроледефинитионс | Нет |
> | Биллингаккаунтс/счета | Нет |
> | Биллингаккаунтс/счета/транзакции | Нет |
> | Биллингаккаунтс/Инвоицесектионс | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Биллингсубскриптионмовеоператионс | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Биллингсубскриптионс | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Биллингсубскриптионс/перемещение | Нет |
> | Биллингаккаунтс/Инвоицесектионс/повышение прав | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Инитиатетрансфер | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Патчоператионс | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Продуктмовеоператионс | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Products | Нет |
> | Биллингаккаунтс/Инвоицесектионс/продукты/перемещение | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Products/Упдатеауторенев | Нет |
> | Биллингаккаунтс/Инвоицесектионс/транзакции | Нет |
> | Биллингаккаунтс/Инвоицесектионс/передача | Нет |
> | Биллингаккаунтс/Линеофкредит | Нет |
> | Биллингаккаунтс/Патчоператионс | Нет |
> | Биллингаккаунтс/Пайментмесодс | Нет |
> | Биллингаккаунтс и продукты | Нет |
> | Биллингаккаунтс/резервирования | Нет |
> | Биллингаккаунтс/транзакции | Нет |
> | billingPeriods | Нет |
> | биллингпермиссионс | Нет |
> | billingProperty | Нет |
> | биллингролеассигнментс | Нет |
> | биллингроледефинитионс | Нет |
> | креатебиллингролеассигнмент | нет |
> | departments | нет |
> | enrollmentAccounts | нет |
> | invoices | нет |
> | transfers | нет |
> | передачи/Акцепттрансфер | Нет |
> | передачи/Деклинетрансфер | Нет |
> | передачи/значение operationstatus | Нет |
> | передачи/Валидатетрансфер | Нет |
> | валидатеаддресс | нет |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | mapApis | Да |
> | updateCommunicationPreference | нет |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | blockchainMembers | Да |
> | кордамемберс | Да |
> | watchers | Да |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | токенсервицес | Да |
> | Токенсервицес/Блоккчаиннетворкс | Нет |
> | Токенсервицес и группы | Нет |
> | Токенсервицес/группы/учетные записи | Нет |
> | Токенсервицес/Токентемплатес | нет |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | blueprintAssignments | Нет |
> | Блуепринтассигнментс/Ассигнментоператионс | Нет |
> | Блуепринтассигнментс/операции | нет |
> | blueprints | нет |
> | blueprints / artifacts | нет |
> | blueprints / versions | нет |
> | blueprints / versions / artifacts | нет |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | botServices | Да |
> | botServices / channels | Нет |
> | Ботсервицес и подключения | Нет |
> | языки | Нет |
> | шаблоны | нет |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Redis | Да |
> | Redis/Евентгридфилтерс | Нет |
> | Redis/Приватиндпоинтконнектионпроксиес | Нет |
> | Redis/Приватиндпоинтконнектионпроксиес/проверка | Нет |
> | Redis/Приватиндпоинтконнектионс | Нет |
> | Redis/Привателинкресаурцес | Нет |
> | редисентерприсе | Да |
> | Редисентерприсе/Приватиндпоинтконнектионпроксиес | Нет |
> | Редисентерприсе/Приватиндпоинтконнектионпроксиес/проверка | Нет |
> | Редисентерприсе/Приватиндпоинтконнектионс | Нет |
> | Редисентерприсе/Привателинкресаурцес | Нет |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | appliedReservations | Нет |
> | аутокуотаинкреасе | Нет |
> | калкулатиксчанже | Нет |
> | calculatePrice | Нет |
> | калкулатепурчасеприце | нет |
> | catalogs | нет |
> | commercialReservationOrders | нет |
> | обмен валюты | нет |
> | овнресерватионс | Нет |
> | плацепурчасеордер | Нет |
> | reservationOrders | Нет |
> | Ресерватионордерс/Калкулатерефунд | Нет |
> | Ресерватионордерс/Merge | Нет |
> | Ресерватионордерс/резервирования | Нет |
> | Ресерватионордерс/резервирования/редакции | Нет |
> | Ресерватионордерс/Return | Нет |
> | Ресерватионордерс/Split | Нет |
> | Ресерватионордерс/swap | нет |
> | reservations | нет |
> | resourceProviders | нет |
> | ресурсов | нет |
> | validateReservationOrder | нет |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | кднвебаппликатионфиреваллманажедрулесетс | Нет |
> | кднвебаппликатионфиреваллполиЦиес | Да |
> | edgenodes | нет |
> | профили | Да |
> | profiles / endpoints | Да |
> | profiles / endpoints / customdomains | нет |
> | profiles / endpoints / origingroups | нет |
> | profiles / endpoints / origins | нет |
> | validateProbe | нет |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | certificateOrders | Да |
> | Certificateorder и сертификаты | Нет |
> | validateCertificateRegistrationInformation | Нет |

## <a name="microsoftchangeanalysis"></a>Microsoft.ChangeAnalysis

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | профиль | Нет |
> | ресаурцечанжес | нет |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | capabilities | нет |
> | domainNames | Да |
> | имя_домена/возможности | Нет |
> | имя_домена/Интерналлоадбаланцерс | Нет |
> | имя_домена/Сервицецертификатес | Нет |
> | имя_домена/слоты | Нет |
> | Доменные имя, слоты и роли | Нет |
> | имя_домена/слоты/роли/metricDefinitions | Нет |
> | имя_домена/слоты/роли/метрики | Нет |
> | moveSubscriptionResources | Нет |
> | operatingSystemFamilies | Нет |
> | operatingSystems | нет |
> | quotas | нет |
> | resourceTypes | Нет |
> | validateSubscriptionMoveAvailability | Нет |
> | virtualMachines | Да |
> | virtualMachines/diagnosticSettings | Нет |
> | virtualMachines/metricDefinitions | Нет |
> | virtualMachines/метрики | Нет |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | classicInfrastructureResources | Нет |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | capabilities | нет |
> | expressRouteCrossConnections | Нет |
> | Експрессраутекроссконнектионс и пиринг | Нет |
> | gatewaySupportedDevices | Нет |
> | networkSecurityGroups | Да |
> | quotas | нет |
> | reservedIps | Да |
> | virtualNetworks | Да |
> | virtualNetworks/Ремотевиртуалнетворкпирингпроксиес | Нет |
> | virtualNetworks / virtualNetworkPeerings | Нет |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | capabilities | нет |
> | disks | нет |
> | images | нет |
> | osImages | Нет |
> | osPlatformImages | Нет |
> | publicImages | нет |
> | quotas | нет |
> | storageAccounts | Да |
> | storageAccounts / blobServices | Нет |
> | storageAccounts / fileServices | Нет |
> | storageAccounts/metricDefinitions | Нет |
> | storageAccounts/метрики | Нет |
> | storageAccounts/Куеуесервицес | Нет |
> | storageAccounts и службы | Нет |
> | storageAccounts/Services/diagnosticSettings | Нет |
> | storageAccounts/Services/metricDefinitions | Нет |
> | storageAccounts/Services/метрики | Нет |
> | storageAccounts/Таблесервицес | Нет |
> | storageAccounts/Вмимажес | Нет |
> | vmImages | Нет |

## <a name="microsoftcodespaces"></a>Microsoft. Кодеспацес

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Планы | Да |
> | регистередсубскриптионс | нет |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | Accounts/Приватиндпоинтконнектионпроксиес | Нет |
> | Accounts/Приватиндпоинтконнектионс | Нет |
> | Accounts/Привателинкресаурцес | нет |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | RateCard | Нет |
> | UsageAggregates | нет |

## <a name="microsoftcompute"></a>Microsoft.Compute;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | availabilitySets | Да |
> | клаудсервицес | Да |
> | Клаудсервицес/networkInterfaces | Нет |
> | Клаудсервицес/publicIPAddresses | Нет |
> | Клаудсервицес/Ролеинстанцес | Нет |
> | Клаудсервицес/Ролеинстанцес/networkInterfaces | Нет |
> | Клаудсервицес и роли | Нет |
> | дискакцессес | Да |
> | diskEncryptionSets | Да |
> | disks | Да |
> | galleries | Да |
> | galleries / applications | Нет |
> | коллекции, приложения и версии | нет |
> | galleries / images | нет |
> | galleries / images / versions | нет |
> | хостграупс | Да |
> | Хостграупс и узлы | Да |
> | images | Да |
> | проксимитиплацементграупс | Да |
> | restorePointCollections | Да |
> | Ресторепоинтколлектионс/Ресторепоинтс | Нет |
> | шаредвмекстенсионс | Да |
> | Шаредвмекстенсионс и версии | Нет |
> | sharedVMImages | Да |
> | Шаредвмимажес и версии | нет |
> | snapshots | Да |
> | сшпубликкэйс | Да |
> | virtualMachines | Да |
> | virtualMachines и расширения | Да |
> | virtualMachines/metricDefinitions | Нет |
> | virtualMachines/Рункоммандс | Да |
> | virtualMachineScaleSets | Да |
> | virtualMachineScaleSets и расширения | Нет |
> | virtualMachineScaleSets/networkInterfaces | Нет |
> | virtualMachineScaleSets/publicIPAddresses | Нет |
> | virtualMachineScaleSets/virtualMachines | Нет |
> | virtualMachineScaleSets/virtualMachines/networkInterfaces | Нет |

## <a name="microsoftconnectedcache"></a>Microsoft. Коннектедкаче

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | каченодес | Да |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | AggregatedCost | Нет |
> | сведения о балансе. | Нет |
> | сведения о бюджете; | Нет |
> | Расходы | Нет |
> | CostTags | нет |
> | credits | нет |
> | события | нет |
> | Прогнозы | нет |
> | lots | нет |
> | Marketplace | Нет |
> | Pricesheets | нет |
> | products | нет |
> | ReservationDetails | Нет |
> | ресерватионрекоммендатиондетаилс | Нет |
> | ReservationRecommendations | Нет |
> | ReservationSummaries | Нет |
> | ReservationTransactions | Нет |
> | Теги | Нет |
> | tenants | нет |
> | Термины | Нет |
> | UsageDetails | нет |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | containerGroups | Да |
> | serviceAssociationLinks | нет |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | registries | Да |
> | реестры и Ажентпулс | Да |
> | registries / builds | нет |
> | registries / builds / cancel | нет |
> | реестры/сборки/Жетлоглинк | Нет |
> | registries / buildTasks | Да |
> | реестры, Буилдтаскс и шаги | Нет |
> | реестры и Евентгридфилтерс | Нет |
> | реестры и Експортпипелинес | Нет |
> | реестры и Женератекредентиалс | Нет |
> | реестры и Жетбуилдсаурцеуплоадурл | Нет |
> | реестры и учетные данные | Нет |
> | реестры и Импортимаже | Нет |
> | реестры и Импортпипелинес | Нет |
> | реестры и Пипелинерунс | Нет |
> | реестры и Приватиндпоинтконнектионпроксиес | Нет |
> | реестры/Приватиндпоинтконнектионпроксиес/проверка | Нет |
> | реестры и Приватиндпоинтконнектионс | Нет |
> | реестры и Привателинкресаурцес | Нет |
> | реестры и Куеуебуилд | Нет |
> | реестры и Реженератекредентиал | Нет |
> | реестры и Реженератекредентиалс | нет |
> | registries / replications | Да |
> | registries / runs | нет |
> | registries / runs / cancel | нет |
> | реестры и Счедулерун | Нет |
> | registries / scopeMaps | Нет |
> | реестры и Таскрунс | нет |
> | registries / tasks | Да |
> | registries / tokens | нет |
> | реестры и УпдатеполиЦиес | нет |
> | registries / webhooks | Да |
> | реестры/веб-перехватчики/Жеткаллбаккконфиг | нет |
> | registries / webhooks / ping | нет |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | containerServices | Да |
> | managedClusters | Да |
> | openShiftManagedClusters | Да |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | видны узлы | Нет |
> | BillingAccounts | Нет |
> | сведения о бюджете; | Нет |
> | клаудконнекторс | Нет |
> | Соединители | Да |
> | косталлокатионрулес | Нет |
> | Departments | Нет |
> | Измерения | Нет |
> | EnrollmentAccounts | Нет |
> | Экспортируемые элементы | Нет |
> | екстерналбиллингаккаунтс | Нет |
> | Екстерналбиллингаккаунтс и оповещения | Нет |
> | Екстерналбиллингаккаунтс и измерения | Нет |
> | Екстерналбиллингаккаунтс/прогноз | Нет |
> | Екстерналбиллингаккаунтс/запрос | Нет |
> | екстерналсубскриптионс | Нет |
> | Екстерналсубскриптионс и оповещения | Нет |
> | Екстерналсубскриптионс и измерения | Нет |
> | Екстерналсубскриптионс/прогноз | Нет |
> | Екстерналсубскриптионс/запрос | Нет |
> | Прогноз | Нет |
> | Аналитика | Нет |
> | Запрос | нет |
> | регистрация | нет |
> | Reportconfigs | Нет |
> | Отчеты | Нет |
> | Параметры | Нет |
> | шовбаккрулес | Нет |
> | Представления | нет |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | requests | нет |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | associations | нет |
> | resourceProviders | Да |

## <a name="microsoftd365customerinsights"></a>Microsoft. D365CustomerInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | instances | Да |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | jobs | Да |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | DataBoxEdgeDevices | Да |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | workspaces | Да |
> | рабочие области и Дбворкспацес | Нет |
> | рабочие области и Виртуалнетворкпирингс | нет |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | catalogs | Да |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | dataFactories | Да |
> | Коэффициенты и diagnosticSettings | Нет |
> | Коэффициенты и metricDefinitions | Нет |
> | dataFactorySchema | нет |
> | factories | Да |
> | factories / integrationRuntimes | нет |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | accounts / dataLakeStoreAccounts | Нет |
> | accounts / storageAccounts | Нет |
> | Accounts/storageAccounts/Containers | Нет |
> | Accounts/Трансфераналитиксунитс | нет |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | Accounts/Евентгридфилтерс | Нет |
> | accounts / firewallRules | нет |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | services; | Да |
> | services / projects | Да |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | баккупваултс | Да |
> | ресаурцеоператионгатекиперс | Да |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | accounts / shares | нет |
> | accounts / shares / datasets | нет |
> | accounts / shares / invitations | нет |
> | accounts / shares / providersharesubscriptions | нет |
> | Accounts/shares/Синчронизатионсеттингс | нет |
> | accounts / sharesubscriptions | нет |
> | Accounts/шаресубскриптионс/Консумерсаурцедатасетс | нет |
> | accounts / sharesubscriptions / datasetmappings | нет |
> | accounts / sharesubscriptions / triggers | нет |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | servers | Да |
> | servers / advisors | нет |
> | servers / keys | нет |
> | серверы и Приватиндпоинтконнектионпроксиес | Нет |
> | серверы и Приватиндпоинтконнектионс | Нет |
> | серверы и Привателинкресаурцес | Нет |
> | серверы и Куеритекстс | Нет |
> | серверы и Рековераблесерверс | Нет |
> | серверы и запуск | Нет |
> | серверы/прерывать | Нет |
> | серверы и Топкуеристатистикс | Нет |
> | servers / virtualNetworkRules | Нет |
> | серверы и Ваитстатистикс | нет |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | флексиблесерверс | Да |
> | servers | Да |
> | servers / advisors | нет |
> | servers / keys | нет |
> | серверы и Приватиндпоинтконнектионпроксиес | Нет |
> | серверы и Приватиндпоинтконнектионс | Нет |
> | серверы и Привателинкресаурцес | Нет |
> | серверы и Куеритекстс | Нет |
> | серверы и Рековераблесерверс | Нет |
> | серверы и запуск | Нет |
> | серверы/прерывать | Нет |
> | серверы и Топкуеристатистикс | Нет |
> | серверы и обновление | Нет |
> | servers / virtualNetworkRules | Нет |
> | серверы и Ваитстатистикс | нет |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | флексиблесерверс | Да |
> | серверграупс | Да |
> | servers | Да |
> | servers / advisors | нет |
> | servers / keys | нет |
> | серверы и Приватиндпоинтконнектионпроксиес | Нет |
> | серверы и Приватиндпоинтконнектионс | Нет |
> | серверы и Привателинкресаурцес | Нет |
> | серверы и Куеритекстс | Нет |
> | серверы и Рековераблесерверс | Нет |
> | серверы и Топкуеристатистикс | Нет |
> | servers / virtualNetworkRules | Нет |
> | серверы и Ваитстатистикс | нет |
> | serversv2 | Да |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | артифактсаурцес | Да |
> | rollouts | Да |
> | сервицетопологиес | Да |
> | Сервицетопологиес и службы | Да |
> | Сервицетопологиес/Services/Сервицеунитс | Да |
> | steps | Да |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | applicationgroups | Да |
> | applicationgroups / applications | нет |
> | applicationgroups / desktops | нет |
> | applicationgroups / startmenuitems | нет |
> | hostpools | Да |
> | хостпулс/мсикспаккажес | Нет |
> | hostpools / sessionhosts | нет |
> | hostpools / sessionhosts / usersessions | нет |
> | hostpools / usersessions | нет |
> | workspaces | Да |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | еластикпулс | Да |
> | Еластикпулс/Иосубтенантс | Да |
> | Еластикпулс/Иосубтенантс/securitySettings | Нет |
> | IotHubs | Да |
> | IotHubs/Евентгридфилтерс | Нет |
> | IotHubs/securitySettings | Нет |
> | ProvisioningServices | Да |
> | usages | нет |

## <a name="microsoftdeviceupdate"></a>Microsoft. Девицеупдате

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | учетные записи и экземпляры | Да |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | pipelines | Да |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | controllers | Да |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | labcenters | Да |
> | labs | Да |
> | labs / environments | Да |
> | Labs и Сервицеруннерс | Да |
> | Labs и virtualMachines | Да |
> | schedules | Да |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | дигиталтвинсинстанцес | Да |
> | Дигиталтвинсинстанцес/конечные точки | нет |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | databaseAccountNames | Нет |
> | databaseAccounts | Да |
> | ресторабледатабасеаккаунтс | нет |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | domains | Да |
> | домены/Домаиновнершипидентифиерс | Нет |
> | generateSsoRequest | Нет |
> | topLevelDomains | Нет |
> | validateDomainRegistrationInformation | Нет |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | lcsprojects | Нет |
> | лкспрожектс/клауддеплойментс | Нет |
> | лкспрожектс и соединители | нет |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | services; | Да |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | domains | Да |
> | domains / topics | нет |
> | eventSubscriptions | Нет |
> | extensionTopics | Нет |
> | партнернамеспацес | Да |
> | Партнернамеспацес/eventChannels | Нет |
> | партнеррегистратионс | Да |
> | партнертопикс | Да |
> | Партнертопикс/eventSubscriptions | Нет |
> | системтопикс | Да |
> | Системтопикс/eventSubscriptions | нет |
> | topics | Да |
> | topicTypes | нет |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | clusters | Да |
> | пространства имен | Да |
> | namespaces / authorizationrules | нет |
> | namespaces / disasterrecoveryconfigs | нет |
> | namespaces / eventhubs | нет |
> | namespaces / eventhubs / authorizationrules | нет |
> | namespaces / eventhubs / consumergroups | нет |
> | namespaces / networkrulesets | нет |
> | пространства имен/Приватиндпоинтконнектионс | нет |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | експериментворкспацес | Да |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | пространства имен | Да |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | феатурепровидерс | нет |
> | features | нет |
> | providers | нет |
> | субскриптионфеатуререгистратионс | Нет |

## <a name="microsoftgallery"></a>Microsoft.Gallery

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | enroll | Нет |
> | galleryitems | Нет |
> | generateartifactaccessuri | Нет |
> | myareas | Нет |
> | мяреас/области | Нет |
> | мяреас/области/области | Нет |
> | мяреас/области/области/галлеритемс | Нет |
> | мяреас/области/галлеритемс | Нет |
> | мяреас/галлеритемс | нет |
> | регистрация | нет |
> | ресурсов | нет |
> | retrieveresourcesbyid | нет |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | аутоманажедаккаунтс | Да |
> | аутоманажедвмконфигуратионпрофилес | Да |
> | конфигуратионпрофилеассигнментс | Нет |
> | guestConfigurationAssignments | нет |
> | software | нет |
> | софтвареупдатепрофиле | Нет |
> | софтвареупдатес | нет |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | hanaInstances | Да |
> | сапмониторс | Да |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | дедикатедхсмс | Да |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | clusters | Да |
> | clusters / applications | нет |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | services; | Да |
> | службы/иомтконнекторс | Нет |
> | службы/иомтконнекторс/подключения | Нет |
> | службы/иомтконнекторс/сопоставления | Нет |
> | службы/Приватиндпоинтконнектионпроксиес | Нет |
> | службы/Приватиндпоинтконнектионс | Нет |
> | службы/Привателинкресаурцес | нет |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | machines | Да |
> | Machines/Ассесспатчес | Нет |
> | machines / extensions | Да |
> | Machines/Инсталлпатчес | нет |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Диспетчеры | Да |

## <a name="microsofthybridnetwork"></a>Microsoft. Хибриднетворк

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Устройства | Да |
> | нетворкфунктионс | Да |
> | нетворкфунктионвендорс | Нет |
> | регистередсубскриптионс | Нет |
> | разработчиков | Нет |
> | поставщики и Вендорскус | Нет |
> | поставщики/Вендорскус/Превиевсубскриптионс | Нет |
> | виртуалнетворкфунктионс | Да |
> | виртуалнетворкфунктионвендорс | нет |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | components | Да |
> | нетворкскопес | Да |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | jobs | Да |

## <a name="microsoftintune"></a>Microsoft.Intune

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | diagnosticSettings | Нет |
> | diagnosticSettingsCategories | Нет |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | апптемплатес | Нет |
> | IoTApps | Да |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | График | Да |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | deletedVaults | Нет |
> | хсмпулс | Да |
> | манажедхсмс | Да |
> | vaults | Да |
> | хранилища и accessPolicies | Нет |
> | хранилища и Евентгридфилтерс | Нет |
> | хранилища и ключи | Нет |
> | хранилища/ключи/версии | нет |
> | vaults / secrets | нет |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | коннектедклустерс | Да |
> | регистередсубскриптионс | нет |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | extensions | Нет |
> | саурцеконтролконфигуратионс | нет |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | clusters | Да |
> | clusters / attacheddatabaseconfigurations | нет |
> | clusters / databases | нет |
> | clusters / databases / dataconnections | нет |
> | clusters / databases / eventhubconnections | нет |
> | clusters / databases / principalassignments | нет |
> | кластеры и подключения к соединениям | нет |
> | clusters / principalassignments | нет |
> | кластеры/шаредидентитиес | нет |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | labaccounts | Да |
> | users | нет |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | hostingEnvironments | Да |
> | integrationAccounts | Да |
> | integrationServiceEnvironments | Да |
> | integrationServiceEnvironments / managedApis | Да |
> | исолатеденвиронментс | Да |
> | workflows | Да |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | commitmentPlans | Да |
> | webServices | Да |
> | Рабочие области | Да |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | workspaces | Да |
> | рабочие области и Батчендпоинтс | Да |
> | рабочие области, Батчендпоинтси и развертывания | Да |
> | рабочие области и коды | Нет |
> | рабочие области, коды и версии | нет |
> | workspaces / computes | нет |
> | рабочие области и хранилища данных | Нет |
> | рабочие области и Евентгридфилтерс | Нет |
> | рабочие области и задания | Нет |
> | рабочие области и Лабелингжобс | Нет |
> | рабочие области и linkedServices | Нет |
> | рабочие области и модели | Нет |
> | рабочие области, модели и версии | Нет |
> | рабочие области и Онлининдпоинтс | Да |
> | рабочие области, Онлининдпоинтси и развертывания | Да |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | апплюпдатес | Нет |
> | конфигуратионассигнментс | Нет |
> | маинтенанцеконфигуратионс | Да |
> | публикмаинтенанцеконфигуратионс | Нет |
> | updates | нет |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Identities | Нет |
> | userAssignedIdentities | Да |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | манажеднетворкс | Да |
> | Манажеднетворкс/Манажеднетворкграупс | Да |
> | Манажеднетворкс/МанажеднетворкпирингполиЦиес | Да |
> | уведомление | Да |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | маркетплацерегистратиондефинитионс | Нет |
> | регистратионассигнментс | Нет |
> | регистратиондефинитионс | нет |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | getEntities | Нет |
> | managementGroups | Нет |
> | Манажементграупс/параметры | нет |
> | ресурсов | нет |
> | startTenantBackfill | Нет |
> | tenantBackfillStatus | нет |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | Accounts/Евентгридфилтерс | Нет |
> | Accounts/Приватеатласес | Да |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | макк | нет |
> | offers | нет |
> | offerTypes | Нет |
> | Оффертипес и издатели | Нет |
> | Оффертипес, издатели и предложения | Нет |
> | Оффертипес, издатели, предложения и планы | Нет |
> | Оффертипес/Publishers/Offers/Plans/Agreement | Нет |
> | Оффертипес/Publishers/предложения, планы и конфигурации | Нет |
> | Оффертипес/Publishers/Offers/Plans/Configurations/Импортимаже | нет |
> | privategalleryitems | нет |
> | приватестореклиент | Нет |
> | приватесторес | Нет |
> | Приватесторес/предложения | нет |
> | products | нет |
> | publishers | нет |
> | publishers / offers | нет |
> | publishers / offers / amendments | нет |
> | регистрация | нет |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | classicDevServices | Да |
> | updateCommunicationPreference | нет |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | agreements | нет |
> | offertypes | нет |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | mediaservices | Да |
> | mediaservices/Аккаунтфилтерс | нет |
> | mediaservices / assets | нет |
> | mediaservices/активы/Ассетфилтерс | Нет |
> | mediaservices/КонтенткэйполиЦиес | Нет |
> | mediaservices/Евентгридфилтерс | Нет |
> | mediaservices/Лививентоператионс | Нет |
> | mediaservices / liveEvents | Да |
> | mediaservices / liveEvents / liveOutputs | Нет |
> | mediaservices/Ливеаутпутоператионс | Нет |
> | mediaservices/Медиаграфс | Нет |
> | mediaservices/Приватиндпоинтконнектионоператионс | Нет |
> | mediaservices/Приватиндпоинтконнектионпроксиес | Нет |
> | mediaservices/Приватиндпоинтконнектионс | Нет |
> | mediaservices/Стреаминжендпоинтоператионс | Нет |
> | mediaservices / streamingEndpoints | Да |
> | mediaservices/Стреаминглокаторс | Нет |
> | mediaservices/СтреамингполиЦиес | нет |
> | mediaservices / transforms | нет |
> | mediaservices / transforms / jobs | нет |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | аппклустерс | Да |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | ассессментпрожектс | Да |
> | migrateprojects | Да |
> | мовеколлектионс | Да |
> | projects | Да |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | холографиксброадкастаккаунтс | Да |
> | обжектундерстандингаккаунтс | Да |
> | ремотерендерингаккаунтс | Да |
> | спатиаланчорсаккаунтс | Да |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | нетаппаккаунтс | Да |
> | Нетаппаккаунтс/Аккаунтбаккупс | Нет |
> | Нетаппаккаунтс/КапаЦитипулс | Да |
> | Нетаппаккаунтс/КапаЦитипулс/тома | Да |
> | Нетаппаккаунтс/КапаЦитипулс/тома/моментальные снимки | нет |
## <a name="microsoftnetwork"></a>Microsoft.Network.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | applicationGateways | Да |
> | аппликатионгатевайвебаппликатионфиреваллполиЦиес | Да |
> | applicationSecurityGroups | Да |
> | azureFirewallFqdnTags | Нет |
> | azureFirewalls | Да |
> | bastionHosts | Да |
> | bgpServiceCommunities | нет |
> | connections | Да |
> | ddosCustomPolicies | Да |
> | ddosProtectionPlans | Да |
> | dnsOperationStatuses | нет |
> | dnszones | Да |
> | днсзонес/A | Нет |
> | днсзонес/AAAA | нет |
> | dnszones / all | нет |
> | днсзонес/CAA | Нет |
> | днсзонес и CNAME | Нет |
> | днсзонес/MX | Нет |
> | днсзонес/NS | Нет |
> | днсзонес/PTR | нет |
> | dnszones / recordsets | нет |
> | днсзонес/SOA | Нет |
> | днсзонес/SRV | Нет |
> | днсзонес/TXT | Нет |
> | expressRouteCircuits | Да |
> | expressRouteCrossConnections | Да |
> | експрессраутегатевайс | Да |
> | експрессраутепортс | Да |
> | expressRouteServiceProviders | Нет |
> | firewallPolicies | Да |
> | frontdoors | Да |
> | фронтдурвебаппликатионфиреваллманажедрулесетс | Нет |
> | frontdoorWebApplicationFirewallPolicies | Да |
> | getDnsResourceReference | Нет |
> | internalNotify | Нет |
> | ипграупс | Да |
> | loadBalancers | Да |
> | localNetworkGateways | Да |
> | natGateways | Да |
> | networkIntentPolicies | Да |
> | networkInterfaces | Да |
> | networkProfiles | Да |
> | networkSecurityGroups | Да |
> | networkWatchers | Да |
> | Нетворкватчерс/Коннектионмониторс | Да |
> | Нетворкватчерс/панель | Да |
> | Нетворкватчерс/lenses | Да |
> | Нетворкватчерс/Пингмешес | Да |
> | p2sVpnGateways | Да |
> | приватеднсоператионстатусес | Нет |
> | privateDnsZones | Да |
> | Приватеднсзонес/A | Нет |
> | Приватеднсзонес/AAAA | Нет |
> | Приватеднсзонес/все | Нет |
> | Приватеднсзонес и CNAME | Нет |
> | Приватеднсзонес/MX | Нет |
> | Приватеднсзонес/PTR | Нет |
> | Приватеднсзонес/SOA | Нет |
> | Приватеднсзонес/SRV | Нет |
> | Приватеднсзонес/TXT | Нет |
> | privateDnsZones / virtualNetworkLinks | Да |
> | приватиндпоинтс | Да |
> | privateLinkServices | Да |
> | publicIPAddresses | Да |
> | publicIPPrefixes | Да |
> | routeFilters | Да |
> | routeTables | Да |
> | serviceEndpointPolicies | Да |
> | trafficManagerGeographicHierarchies | нет |
> | trafficmanagerprofiles | Да |
> | trafficmanagerprofiles/тепловые карты | Нет |
> | траффикманажерусерметрикскэйс | Нет |
> | virtualHubs | Да |
> | virtualNetworkGateways | Да |
> | virtualNetworks | Да |
> | virtualNetworks и подсети | Нет |
> | virtualNetworkTaps | Да |
> | virtualWans | Да |
> | vpnGateways | Да |
> | vpnSites | Да |
> | webApplicationFirewallPolicies | Да |

## <a name="microsoftnotebooks"></a>Microsoft. Notebooks

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | нотебукпроксиес | Нет |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | пространства имен | Да |
> | namespaces / notificationHubs | Да |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | оснамеспацес | Да |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | хипервситес | Да |
> | импортситес | Да |
> | мастерситес | Да |
> | серверситес | Да |
> | вмвареситес | Да |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | clusters | Да |
> | делетедворкспацес | Нет |
> | linkTargets | Нет |
> | storageInsightConfigs | нет |
> | workspaces | Да |
> | рабочие области и экспорты | Нет |
> | рабочие области и источники данных | Нет |
> | рабочие области и linkedServices | Нет |
> | рабочие области и Линкедсторажеаккаунтс | нет |
> | workspaces / metadata | нет |
> | workspaces / query | нет |
> | рабочие области и Скопедпривателинкпроксиес | Нет |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | managementassociations | нет |
> | managementconfigurations | Да |
> | solutions | Да |
> | узел "Представления" | Да |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | легаципирингс | Нет |
> | пираснс | нет |
> | peerings | Да |
> | пирингсервицекаунтриес | Нет |
> | пирингсервицепровидерс | Нет |
> | пирингсервицес | Да |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | аттестации | Нет |
> | policyEvents | Нет |
> | полициметадата | Нет |
> | policyStates | Нет |
> | policyTrackedResources | нет |
> | remediations | нет |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | consoles | нет |
> | dashboards | Да |
> | userSettings | нет |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | привателинксервицесфорповерби | Да |
> | tenants | Да |
> | Клиенты и рабочие области | Нет |
> | workspaceCollections | Да |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | capacities | Да |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | делетедаккаунтс | нет |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | providerRegistrations | Нет |
> | Провидеррегистратионс/Дефаултроллаутс | Нет |
> | Провидеррегистратионс/Ресаурцетиперегистратионс | нет |
> | rollouts | Да |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Рабочие области | Да |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | backupProtectedItems | нет |
> | vaults | Да |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | опеншифтклустерс | Да |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | пространства имен | Да |
> | namespaces / authorizationrules | нет |
> | namespaces / hybridconnections | нет |
> | namespaces / hybridconnections / authorizationrules | нет |
> | пространства имен/Приватиндпоинтконнектионс | нет |
> | namespaces / wcfrelays | нет |
> | namespaces / wcfrelays / authorizationrules | нет |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Запросы | Да |
> | ресаурцечанжедетаилс | Нет |
> | ресаурцечанжес | нет |
> | ресурсов | нет |
> | ресаурцешистори | Нет |
> | subscriptionsStatus | нет |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | availabilityStatuses | Нет |
> | childAvailabilityStatuses | Нет |
> | childResources | нет |
> | emergingissues | нет |
> | события | нет |
> | impactedResources | нет |
> | метаданные | нет |
> | Уведомления | нет |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | калкулатетемплатехаш | нет |
> | deployments | нет |
> | deployments / operations | нет |
> | deploymentScripts | Да |
> | deploymentScripts и журналы | нет |
> | ссылки | нет |
> | notifyResourceJobs | нет |
> | providers | нет |
> | resourceGroups | нет |
> | subscriptions | нет |
> | темплатеспекс | Да |
> | Темплатеспекс и версии | Да |
> | tenants | нет |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | веб-масштабированием; | Да |
> | saasresources | нет |

## <a name="microsoftscvmm"></a>Microsoft. ScVmm

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | облака | Да |
> | VirtualMachines | Да |
> | виртуалмачинетемплатес | Да |
> | VirtualNetworks | Да |
> | вммсерверс | Да |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | resourceHealthMetadata | Нет |
> | searchServices | Да |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | адаптивенетворкхарденингс | Нет |
> | advancedThreatProtectionSettings | нет |
> | оповещения | нет |
> | алертссуппрессионрулес | Нет |
> | allowedConnections | Нет |
> | applicationWhitelistings | Нет |
> | ассессментметадата | нет |
> | assessments | нет |
> | аутодисмиссалертсрулес | нет |
> | automations | Да |
> | AutoProvisioningSettings | Нет |
> | Compliances | нет |
> | соединители | Нет |
> | dataCollectionAgents | Нет |
> | девицесекуритиграупс | Нет |
> | discoveredSecuritySolutions | Нет |
> | externalSecuritySolutions | Нет |
> | InformationProtectionPolicies | Нет |
> | иотдефендерсеттингс | Нет |
> | иотсекуритисолутионс | Да |
> | Иотсекуритисолутионс/Аналитиксмоделс | Нет |
> | Иотсекуритисолутионс/Аналитиксмоделс/Аггрегатедалертс | Нет |
> | Иотсекуритисолутионс/Аналитиксмоделс/Аггрегатедрекоммендатионс | Нет |
> | Иотсекуритисолутионс/Иоталертс | Нет |
> | Иотсекуритисолутионс/Иоталерттипес | Нет |
> | Иотсекуритисолутионс/Иотрекоммендатионс | Нет |
> | Иотсекуритисолутионс/Иотрекоммендатионтипес | Нет |
> | иотсенсорс | Нет |
> | jitNetworkAccessPolicies | Нет |
> | житполиЦиес | нет |
> | политики | нет |
> | pricings | нет |
> | регулаторикомплианцестандардс | Нет |
> | Регулаторикомплианцестандардс/Регулаторикомплианцеконтролс | Нет |
> | Регулаторикомплианцестандардс/Регулаторикомплианцеконтролс/Регулаторикомплианцеассессментс | Нет |
> | секурескореконтролдефинитионс | Нет |
> | секурескореконтролс | Нет |
> | секурескорес | Нет |
> | Секурескорес/Секурескореконтролс | Нет |
> | securityContacts | Нет |
> | securitySolutions | Нет |
> | securitySolutionsReferenceData | Нет |
> | securityStatuses | Нет |
> | securityStatusesSummaries | Нет |
> | сервервулнерабилитяссессментс | нет |
> | Параметры | нет |
> | склвулнерабилитяссессментс | Нет |
> | подоценка | нет |
> | задачи | нет |
> | topologies | нет |
> | workspaceSettings | Нет |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | diagnosticSettings | Нет |
> | diagnosticSettingsCategories | нет |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | aggregations | нет |
> | alertRules | Нет |
> | алертрулетемплатес | Нет |
> | аутоматионрулес | нет |
> | bookmarks | нет |
> | cases | нет |
> | Подключения к компонентам | Нет |
> | датаконнекторсчеккрекуирементс | нет |
> | Сущности | нет |
> | ентитикуериес | нет |
> | incidents | нет |
> | оффицеконсентс | нет |
> | Параметры | нет |
> | среатинтеллиженце | Нет |
> | ватчлистс | нет |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | консолесервицес | нет |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | пространства имен | Да |
> | namespaces / authorizationrules | нет |
> | namespaces / disasterrecoveryconfigs | нет |
> | namespaces / eventgridfilters | нет |
> | namespaces / networkrulesets | нет |
> | пространства имен/Приватиндпоинтконнектионс | нет |
> | namespaces / queues | нет |
> | namespaces / queues / authorizationrules | нет |
> | namespaces / topics | нет |
> | namespaces / topics / authorizationrules | нет |
> | namespaces / topics / subscriptions | нет |
> | namespaces / topics / subscriptions / rules | нет |
> | premiumMessagingRegions | нет |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | веб-масштабированием; | Да |
> | clusters | Да |
> | clusters / applications | нет |
> | containerGroups | Да |
> | контаинерграупсетс | Да |
> | edgeclusters | Да |
> | еджеклустерс и приложения | нет |
> | managedclusters | Да |
> | манажедклустерс/NodeType | нет |
> | networks | Да |
> | secretstores | Да |
> | секретсторес и сертификаты | Нет |
> | секретсторес/секреты | нет |
> | volumes. | Да |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | веб-масштабированием; | Да |
> | containerGroups | Да |
> | gateways | Да |
> | networks | Да |
> | секретные коды | Да |
> | volumes. | Да |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | providerRegistrations | Нет |
> | Провидеррегистратионс/Ресаурцетиперегистратионс | нет |
> | rollouts | Да |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | SignalR | Да |
> | SignalR/Евентгридфилтерс | Нет |

## <a name="microsoftsingularity"></a>Microsoft. Единственное,

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | Accounts/АккаунткуотаполиЦиес | Нет |
> | Accounts/ГраупполиЦиес | Нет |
> | учетные записи и задания | Нет |
> | Accounts/Сторажеконтаинерс | нет |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | хибридусебенефитс | нет |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | applicationDefinitions | Да |
> | веб-масштабированием; | Да |
> | jitRequests | Да |

## <a name="microsoftsql"></a>Microsoft.SQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | managedInstances | Да |
> | Манажединстанцес и базы данных | Да |
> | Манажединстанцес/databases/БаккупшорттермретентионполиЦиес | Нет |
> | Манажединстанцес/базы данных, схемы, таблицы, столбцы и Сенситивитилабелс | Нет |
> | Манажединстанцес/databases/Вулнерабилитяссессментс | Нет |
> | Манажединстанцес/базы данных/Вулнерабилитяссессментс/правила и базовые планы | Нет |
> | Манажединстанцес/Енкриптионпротектор | Нет |
> | Манажединстанцес и ключи | Нет |
> | Манажединстанцес/Ресторабледроппеддатабасес/БаккупшорттермретентионполиЦиес | Нет |
> | Манажединстанцес/Вулнерабилитяссессментс | нет |
> | servers | Да |
> | servers / administrators | нет |
> | серверы и Коммуникатионлинкс | нет |
> | servers / databases | Да |
> | серверы и Енкриптионпротектор | Нет |
> | servers / firewallRules | нет |
> | servers / keys | нет |
> | серверы и Ресторабледроппеддатабасес | нет |
> | servers / serviceobjectives | нет |
> | серверы и Тдецертификатес | Нет |
> | виртуалклустерс | Нет |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | SqlVirtualMachineGroups | Да |
> | Склвиртуалмачинеграупс/Аваилабилитиграуплистенерс | Нет |
> | SqlVirtualMachines | Да |

## <a name="microsoftstorage"></a>Microsoft.Storage;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | делетедаккаунтс | Нет |
> | storageAccounts | Да |
> | storageAccounts / blobServices | Нет |
> | storageAccounts / fileServices | Нет |
> | storageAccounts/Куеуесервицес | Нет |
> | storageAccounts и службы | Нет |
> | storageAccounts/Services/metricDefinitions | Нет |
> | storageAccounts/Таблесервицес | нет |
> | usages | нет |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | caches | Да |
> | кэши/Сторажетаржетс | Нет |
> | усажемоделс | Нет |

## <a name="microsoftstoragereplication"></a>Microsoft. Сторажерепликатион

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | репликатионграупс | нет |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | storageSyncServices | Да |
> | Сторажесинксервицес/Регистередсерверс | Нет |
> | storageSyncServices / syncGroups | Нет |
> | Сторажесинксервицес/Синкграупс/Клаудендпоинтс | Нет |
> | Сторажесинксервицес/Синкграупс/Серверендпоинтс | Нет |
> | Сторажесинксервицес и рабочие процессы | нет |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | storageSyncServices | Да |
> | Сторажесинксервицес/Регистередсерверс | Нет |
> | storageSyncServices / syncGroups | Нет |
> | Сторажесинксервицес/Синкграупс/Клаудендпоинтс | Нет |
> | Сторажесинксервицес/Синкграупс/Серверендпоинтс | Нет |
> | Сторажесинксервицес и рабочие процессы | нет |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | storageSyncServices | Да |
> | Сторажесинксервицес/Регистередсерверс | Нет |
> | storageSyncServices / syncGroups | Нет |
> | Сторажесинксервицес/Синкграупс/Клаудендпоинтс | Нет |
> | Сторажесинксервицес/Синкграупс/Серверендпоинтс | Нет |
> | Сторажесинксервицес и рабочие процессы | нет |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | managers | Да |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | clusters | Да |
> | кластеры/Приватиндпоинтс | нет |
> | streamingjobs | Да |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | акцептчанжетенант | Нет |
> | псевдонимы | Нет |
> | cancel | нет |
> | чанжетенантрекуест | Нет |
> | чанжетенантстатус | Нет |
> | CreateSubscription | Нет |
> | enable | нет |
> | переименовать | нет |
> | SubscriptionDefinitions | Нет |
> | SubscriptionOperations | нет |
> | subscriptions | нет |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | привателинкхубс | Да |
> | workspaces | Да |
> | рабочие области и Бигдатапулс | Да |
> | рабочие области и Оператионстатусес | Нет |
> | рабочие области и Склдатабасес | Да |
> | рабочие области и Склпулс | Да |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | environments | Да |
> | environments / accessPolicies | нет |
> | environments / eventsources | Да |
> | environments / referenceDataSets | Да |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | stores | Да |
> | Магазины и accessPolicies | нет |
> | stores / services | нет |
> | stores / services / tokens | нет |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | имажетемплатес | Да |
> | Имажетемплатес/Рунаутпутс | Нет |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | аркзонес | Да |
> | ресаурцепулс | Да |
> | Серверами vCenter | Да |
> | VirtualMachines | Да |
> | виртуалмачинетемплатес | Да |
> | VirtualNetworks | Да |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | дедикатедклауднодес | Да |
> | дедикатедклаудсервицес | Да |
> | virtualMachines | Да |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | Устройства | Да |
> | регистередсубскриптионс | Нет |
> | разработчиков | Нет |
> | поставщики и номера SKU | Нет |
> | поставщики и внфс | Нет |
> | виртуалнетворкфунктионскус | нет |
> | vnfs | Да |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | accounts | Да |
> | Планы | Да |
> | регистередсубскриптионс | нет |

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | apiManagementAccounts | Нет |
> | Апиманажементаккаунтс/списков ACL для | Нет |
> | Апиманажементаккаунтс/API | Нет |
> | Апиманажементаккаунтс/API/списков ACL для | Нет |
> | Апиманажементаккаунтс/API/списков ACL для | Нет |
> | Апиманажементаккаунтс/API/подключения | Нет |
> | Апиманажементаккаунтс/API/Connections/списков ACL для | Нет |
> | Апиманажементаккаунтс/API/Локализеддефинитионс | Нет |
> | Апиманажементаккаунтс/списков ACL для | Нет |
> | Апиманажементаккаунтс и подключения | Нет |
> | billingMeters | нет |
> | certificates | Да |
> | connectionGateways | Да |
> | connections | Да |
> | customApis | Да |
> | deletedSites | Нет |
> | hostingEnvironments | Да |
> | hostingEnvironments/Евентгридфилтерс | Нет |
> | hostingEnvironments/Мултиролепулс | Нет |
> | hostingEnvironments/внешнего размещения | Нет |
> | кубинвиронментс | Да |
> | publishingUsers | нет |
> | к просмотру фильмов | нет |
> | resourceHealthMetadata | нет |
> | runtimes | нет |
> | serverFarms | Да |
> | serverFarms/Евентгридфилтерс | Нет |
> | serverFarms/Фирстпартяппс | Нет |
> | serverFarms/Фирстпартяппс/Кэйваултсеттингс | нет |
> | sites | Да |
> | сайты и конфигурация  | Нет |
> | Sites/Евентгридфилтерс | Нет |
> | Sites/Хостнамебиндингс | Нет |
> | Sites/файл networkconfig | нет |
> | sites / premieraddons | Да |
> | sites / slots | Да |
> | сайты/слоты/Евентгридфилтерс | Нет |
> | сайты/слоты/Хостнамебиндингс | Нет |
> | сайты/слоты/файл networkconfig | Нет |
> | sourceControls | Нет |
> | статикситес | Да |
> | validate | нет |
> | verifyHostingEnvironmentVnet | Нет |

## <a name="microsoftwindowsdefenderatp"></a>Microsoft.WindowsDefenderATP

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | diagnosticSettings | Нет |
> | diagnosticSettingsCategories | Нет |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | мултиплеактиватионкэйс | Да |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | DeviceServices | Да |

## <a name="microsoftworkloadbuilder"></a>Microsoft. Ворклоадбуилдер

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | рабочие нагрузки | Да |
> | рабочие нагрузки и экземпляры | Нет |
> | рабочие нагрузки и версии | Нет |
> | рабочие нагрузки, версии и артефакты | нет |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Тип ресурса | Удаление ресурсов в полном режиме |
> | ------------- | ----------- |
> | components | нет |
> | componentsSummary | Нет |
> | monitorInstances | Нет |
> | monitorInstancesSummary | нет |
> | monitors | нет |
> | notificationSettings | Нет |

## <a name="next-steps"></a>Дальнейшие действия

Чтобы получить данные, идентичные значению файла с разделителями-запятыми, необходимо скачать [complete-mode-deletion.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/complete-mode-deletion.csv).
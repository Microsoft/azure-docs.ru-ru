---
title: Поддержка тегов для ресурсов
description: В этой статье описываются типы ресурсов Azure, поддерживающие теги, а также приводятся сведения о всех службах Azure.
ms.topic: conceptual
ms.date: 10/21/2020
ms.openlocfilehash: b1f9e61b409c395a13cc6a18d21d17530f38ac7e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101722897"
---
# <a name="tag-support-for-azure-resources"></a>Поддержка тегов для ресурсов Azure
В этой статье приводятся сведения о поддержке типом ресурса функции [тегов](tag-resources.md). Столбец с подписью **поддерживает теги** . указывает, имеет ли тип ресурса свойство для тега. Столбец с меткой **тег в отчете о затратах** указывает, передает ли этот тип ресурса тег в отчет о затратах. Вы можете просматривать затраты по тегам в [анализе затрат на управление затратами](../../cost-management-billing/costs/group-filter.md) , а также в [счетах на счета и данные о ежедневном использовании в Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md).

Чтобы получить данные, идентичные значению файла с разделителями-запятыми, необходимо скачать [tag-support.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/tag-support.csv).

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
> - [Microsoft. Insights](#microsoftinsights)
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
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | DomainServices | Да | Да |
> | DomainServices/ауконтаинер | Нет | нет |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | supportProviders | Нет | нет |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
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
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | адвисорскоре | Нет | Нет |
> | конфигурации | нет | Нет |
> | generateRecommendations | Нет | нет |
> | метаданные | нет | нет |
> | к просмотру фильмов | нет | нет |
> | suppressions | нет | Нет |

## <a name="microsoftagfoodplatform"></a>Microsoft. Агфудплатформ

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | фармбеатс | Да | Да |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | actionRules | Да | Да |
> | оповещения | нет | Нет |
> | alertsList | Нет | Нет |
> | алертсметадата | Нет | Нет |
> | alertsSummary | Нет | Нет |
> | alertsSummaryList | Нет | Нет |
> | smartDetectorAlertRules | Да | Да |
> | smartGroups | Нет | нет |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | servers | Да | Да |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | reportFeedback | Нет | Нет |
> | служба | Да | Да |
> | validateServiceName | Нет | Нет |

> [!NOTE]
> Служба управления API Azure поддерживает создание не более 15 пар "имя-значение" для каждой службы.

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | configurationStores | Да | Нет |
> | Конфигуратионсторес/Евентгридфилтерс | Нет | Нет |
> | Конфигуратионсторес/Кэйвалуес | Нет | нет |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Spring | Да | Да |
> | Пружина/приложения | Нет | Нет |
> | Пружина/приложения/развертывания | Нет | Нет |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | attestationProviders | Да | Да |
> | дефаултпровидерс | Нет | нет |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | акцессревиевсчедуледефинитионс | Нет | Нет |
> | акцессревиевсчедулесеттингс | Нет | Нет |
> | classicAdministrators | Нет | Нет |
> | Псевдонимы | Нет | Нет |
> | denyAssignments | Нет | Нет |
> | elevateAccess | Нет | Нет |
> | финдорфанролеассигнментс | Нет | нет |
> | locks | нет | нет |
> | разрешения | нет | Нет |
> | policyAssignments | Нет | Нет |
> | policyDefinitions | Нет | Нет |
> | полициексемптионс | Нет | Нет |
> | policySetDefinitions | Нет | Нет |
> | привателинкассоЦиатионс | Нет | Нет |
> | providerOperations | Нет | Нет |
> | ресаурцеманажементпривателинкс | Да | Да |
> | roleAssignments | Нет | Нет |
> | ролеассигнментсусажеметрикс | Нет | Нет |
> | roleDefinitions | Нет | Нет |

## <a name="microsoftautomanage"></a>Microsoft. Управление

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | конфигуратионпрофилеассигнментс | Нет | Нет |
> | конфигуратионпрофилепреференцес | Да | Да |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | automationAccounts | Да | Да |
> | automationAccounts и конфигурации | Да | Да |
> | automationAccounts и задания | Нет | Нет |
> | automationAccounts/Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | automationAccounts/Приватиндпоинтконнектионс | Нет | Нет |
> | automationAccounts/Привателинкресаурцес | Нет | Нет |
> | automationAccounts / runbooks | Да | Да |
> | automationAccounts/Софтвареупдатеконфигуратионс | Нет | Нет |
> | automationAccounts / webhooks | Нет | Нет |

> [!NOTE]
> Служба автоматизации Azure поддерживает создание не более 15 пар "имя-значение" тега для каждого ресурса службы автоматизации.

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | приватеклаудс | Да | Да |
> | Приватеклаудс и надстройки | Нет | Нет |
> | Приватеклаудс и авторизация | Нет | Нет |
> | Приватеклаудс и кластеры | Нет | Нет |
> | Приватеклаудс/Глобалреачконнектионс | Нет | Нет |
> | Приватеклаудс/Хкксентерприсеситес | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс/Дхкпконфигуратионс | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс/шлюзы | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс/Портмиррорингпрофилес | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс/сегменты | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс/virtualMachines | Нет | Нет |
> | Приватеклаудс/Ворклоаднетворкс/Вмграупс | Нет | Нет |

## <a name="microsoftazuregeneva"></a>Microsoft.Azure.Geneva

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | environments | нет | Нет |
> | среды и учетные записи | Нет | Нет |
> | среды, учетные записи и пространства имен | Нет | Нет |
> | среды, учетные записи, пространства имен и конфигурации | Нет | нет |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | b2cDirectories | Да | Нет |
> | b2ctenants | нет | Нет |
> | гуестусажес | Да | Да |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Контроллеры | Да | Да |
> | постгресинстанцес | Да | Да |
> | склманажединстанцес | Да | Да |
> | склсерверинстанцес | Да | Да |
> | склсерверрегистратионс | Да | Да |
> | Склсерверрегистратионс и sqlServer | Нет | нет |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | клаудманифестфилес | Нет | Нет |
> | еджесубскриптионс | Да | Да |
> | линкедсубскриптионс | Да | Да |
> | registrations | Да | Да |
> | регистрации и Кустомерсубскриптионс | Нет | нет |
> | registrations / products | нет | нет |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | clusters | Да | Да |

## <a name="microsoftbaremetalinfrastructure"></a>Microsoft. Бареметалинфраструктуре

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | бареметалинстанцес | Да | Да |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | batchAccounts | Да | Да |
> | batchAccounts / certificates | Нет | Нет |
> | batchAccounts / pools | Нет | нет |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | billingAccounts | Нет | Нет |
> | Биллингаккаунтс и соглашения | Нет | Нет |
> | Биллингаккаунтс/Биллингпермиссионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингпермиссионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингролеассигнментс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингроледефинитионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Биллингсубскриптионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Креатебиллингролеассигнмент | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/клиенты | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/инструкции | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/счета | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/счета/прайс-лист | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/счета/транзакции | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингпермиссионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингролеассигнментс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингроледефинитионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Биллингсубскриптионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Креатебиллингролеассигнмент | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Инитиатетрансфер | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Products | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Products/перемещение | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/Products/Упдатеауторенев | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/транзакции | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Инвоицесектионс/передача | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Патчоператионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Пайментмесодс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/политики | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/прайс-лист | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Прицешитдовнлоадоператионс | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Products | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/резервирования | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/транзакции | Нет | Нет |
> | Биллингаккаунтс/Биллингпрофилес/Валидатедетачпайментмесоделигибилити | Нет | Нет |
> | Биллингаккаунтс/Биллингролеассигнментс | Нет | Нет |
> | Биллингаккаунтс/Биллингроледефинитионс | Нет | Нет |
> | Биллингаккаунтс/Биллингсубскриптионс | Нет | Нет |
> | Биллингаккаунтс/Биллингсубскриптионс/счета | Нет | Нет |
> | Биллингаккаунтс/Креатебиллингролеассигнмент | Нет | Нет |
> | Биллингаккаунтс/Креатеинвоицесектионоператионс | Нет | Нет |
> | Биллингаккаунтс и клиенты | Нет | Нет |
> | Биллингаккаунтс/Customers/Биллингпермиссионс | Нет | Нет |
> | Биллингаккаунтс/Customers/Биллингсубскриптионс | Нет | Нет |
> | Биллингаккаунтс/Customers/Инитиатетрансфер | Нет | Нет |
> | Биллингаккаунтс, клиенты и политики | Нет | Нет |
> | Биллингаккаунтс/Customers/Products | Нет | Нет |
> | Биллингаккаунтс/Customers/Transactions | Нет | Нет |
> | Биллингаккаунтс/Customers/Transfers | Нет | Нет |
> | Биллингаккаунтс и отделы | Нет | Нет |
> | Биллингаккаунтс/Departments/Биллингпермиссионс | Нет | Нет |
> | Биллингаккаунтс/Departments/Биллингролеассигнментс | Нет | Нет |
> | Биллингаккаунтс/Departments/Биллингроледефинитионс | Нет | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс | Нет | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс/Биллингпермиссионс | Нет | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс/Биллингролеассигнментс | Нет | Нет |
> | Биллингаккаунтс/Енроллментаккаунтс/Биллингроледефинитионс | Нет | Нет |
> | Биллингаккаунтс/счета | Нет | Нет |
> | Биллингаккаунтс/счета/транзакции | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Биллингсубскриптионмовеоператионс | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Биллингсубскриптионс | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Биллингсубскриптионс/перемещение | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/повышение прав | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Инитиатетрансфер | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Патчоператионс | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Продуктмовеоператионс | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Products | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/продукты/перемещение | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/Products/Упдатеауторенев | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/транзакции | Нет | Нет |
> | Биллингаккаунтс/Инвоицесектионс/передача | Нет | Нет |
> | Биллингаккаунтс/Линеофкредит | Нет | Нет |
> | Биллингаккаунтс/Патчоператионс | Нет | Нет |
> | Биллингаккаунтс/Пайментмесодс | Нет | Нет |
> | Биллингаккаунтс и продукты | Нет | Нет |
> | Биллингаккаунтс/резервирования | Нет | Нет |
> | Биллингаккаунтс/транзакции | Нет | Нет |
> | billingPeriods | Нет | Нет |
> | биллингпермиссионс | Нет | Нет |
> | billingProperty | Нет | Нет |
> | биллингролеассигнментс | Нет | Нет |
> | биллингроледефинитионс | Нет | Нет |
> | креатебиллингролеассигнмент | Нет | нет |
> | departments | нет | Нет |
> | enrollmentAccounts | Нет | нет |
> | invoices | нет | нет |
> | transfers | нет | Нет |
> | передачи/Акцепттрансфер | Нет | Нет |
> | передачи/Деклинетрансфер | Нет | Нет |
> | передачи/значение operationstatus | Нет | Нет |
> | передачи/Валидатетрансфер | Нет | Нет |
> | валидатеаддресс | Нет | нет |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | mapApis | Да | Да |
> | updateCommunicationPreference | Нет | нет |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | blockchainMembers | Да | Да |
> | кордамемберс | Да | Да |
> | watchers | Да | Да |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | токенсервицес | Да | Да |
> | Токенсервицес/Блоккчаиннетворкс | Нет | Нет |
> | Токенсервицес и группы | Нет | Нет |
> | Токенсервицес/группы/учетные записи | Нет | Нет |
> | Токенсервицес/Токентемплатес | Нет | нет |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | blueprintAssignments | Нет | Нет |
> | Блуепринтассигнментс/Ассигнментоператионс | Нет | Нет |
> | Блуепринтассигнментс/операции | Нет | нет |
> | blueprints | нет | нет |
> | blueprints / artifacts | нет | нет |
> | blueprints / versions | нет | нет |
> | blueprints / versions / artifacts | нет | нет |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | botServices | Да | Да |
> | botServices / channels | Нет | Нет |
> | Ботсервицес и подключения | Нет | Нет |
> | языки | Нет | Нет |
> | шаблоны | Нет | нет |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Redis | Да | Да |
> | Redis/Евентгридфилтерс | Нет | Нет |
> | Redis/Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | Redis/Приватиндпоинтконнектионпроксиес/проверка | Нет | Нет |
> | Redis/Приватиндпоинтконнектионс | Нет | Нет |
> | Redis/Привателинкресаурцес | Нет | Нет |
> | редисентерприсе | Да | Да |
> | Редисентерприсе/Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | Редисентерприсе/Приватиндпоинтконнектионпроксиес/проверка | Нет | Нет |
> | Редисентерприсе/Приватиндпоинтконнектионс | Нет | Нет |
> | Редисентерприсе/Привателинкресаурцес | Нет | Нет |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | appliedReservations | Нет | Нет |
> | аутокуотаинкреасе | Нет | Нет |
> | калкулатиксчанже | Нет | Нет |
> | calculatePrice | Нет | Нет |
> | калкулатепурчасеприце | Нет | нет |
> | catalogs | нет | Нет |
> | commercialReservationOrders | Нет | нет |
> | обмен валюты | нет | Нет |
> | овнресерватионс | Нет | Нет |
> | плацепурчасеордер | Нет | Нет |
> | reservationOrders | Нет | Нет |
> | Ресерватионордерс/Калкулатерефунд | Нет | Нет |
> | Ресерватионордерс/Merge | Нет | Нет |
> | Ресерватионордерс/резервирования | Нет | Нет |
> | Ресерватионордерс/резервирования/редакции | Нет | Нет |
> | Ресерватионордерс/Return | Нет | Нет |
> | Ресерватионордерс/Split | Нет | Нет |
> | Ресерватионордерс/swap | Нет | нет |
> | reservations | нет | Нет |
> | resourceProviders | Нет | нет |
> | ресурсов | нет | Нет |
> | validateReservationOrder | Нет | нет |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | кднвебаппликатионфиреваллманажедрулесетс | Нет | Нет |
> | кднвебаппликатионфиреваллполиЦиес | Да | Да |
> | edgenodes | нет | нет |
> | профили | Да | Да |
> | profiles / endpoints | Да | Да |
> | profiles / endpoints / customdomains | нет | нет |
> | profiles / endpoints / origingroups | нет | нет |
> | profiles / endpoints / origins | нет | Нет |
> | validateProbe | Нет | нет |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | certificateOrders | Да | Да |
> | Certificateorder и сертификаты | Нет | Нет |
> | validateCertificateRegistrationInformation | Нет | Нет |

## <a name="microsoftchangeanalysis"></a>Microsoft.ChangeAnalysis

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | профиль | Нет | Нет |
> | ресаурцечанжес | Нет | нет |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | capabilities | нет | Нет |
> | domainNames | Нет | Нет |
> | имя_домена/возможности | Нет | Нет |
> | имя_домена/Интерналлоадбаланцерс | Нет | Нет |
> | имя_домена/Сервицецертификатес | Нет | Нет |
> | имя_домена/слоты | Нет | Нет |
> | Доменные имя, слоты и роли | Нет | Нет |
> | имя_домена/слоты/роли/metricDefinitions | Нет | Нет |
> | имя_домена/слоты/роли/метрики | Нет | Нет |
> | moveSubscriptionResources | Нет | Нет |
> | operatingSystemFamilies | Нет | Нет |
> | operatingSystems | Нет | нет |
> | quotas | нет | Нет |
> | resourceTypes | Нет | Нет |
> | validateSubscriptionMoveAvailability | Нет | Нет |
> | virtualMachines | Нет | Нет |
> | virtualMachines/diagnosticSettings | Нет | Нет |
> | virtualMachines/metricDefinitions | Нет | Нет |
> | virtualMachines/метрики | Нет | Нет |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | classicInfrastructureResources | Нет | Нет |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | capabilities | нет | Нет |
> | expressRouteCrossConnections | Нет | Нет |
> | Експрессраутекроссконнектионс и пиринг | Нет | Нет |
> | gatewaySupportedDevices | Нет | Нет |
> | networkSecurityGroups | Нет | нет |
> | quotas | нет | Нет |
> | reservedIps | Нет | Нет |
> | virtualNetworks | Нет | Нет |
> | virtualNetworks/Ремотевиртуалнетворкпирингпроксиес | Нет | Нет |
> | virtualNetworks / virtualNetworkPeerings | Нет | Нет |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | capabilities | нет | нет |
> | disks | нет | нет |
> | images | нет | Нет |
> | osImages | Нет | Нет |
> | osPlatformImages | Нет | Нет |
> | publicImages | Нет | нет |
> | quotas | нет | Нет |
> | storageAccounts | Нет | Нет |
> | storageAccounts / blobServices | Нет | Нет |
> | storageAccounts / fileServices | Нет | Нет |
> | storageAccounts/metricDefinitions | Нет | Нет |
> | storageAccounts/метрики | Нет | Нет |
> | storageAccounts/Куеуесервицес | Нет | Нет |
> | storageAccounts и службы | Нет | Нет |
> | storageAccounts/Services/diagnosticSettings | Нет | Нет |
> | storageAccounts/Services/metricDefinitions | Нет | Нет |
> | storageAccounts/Services/метрики | Нет | Нет |
> | storageAccounts/Таблесервицес | Нет | Нет |
> | storageAccounts/Вмимажес | Нет | Нет |
> | vmImages | Нет | Нет |

## <a name="microsoftcodespaces"></a>Microsoft. Кодеспацес

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Планы | Да | Нет |
> | регистередсубскриптионс | Нет | нет |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | Accounts/Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | Accounts/Приватиндпоинтконнектионс | Нет | Нет |
> | Accounts/Привателинкресаурцес | Нет | нет |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | RateCard | Нет | Нет |
> | UsageAggregates | Нет | нет |

## <a name="microsoftcompute"></a>Microsoft.Compute;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | availabilitySets | Да | Да |
> | клаудсервицес | Да | Да |
> | Клаудсервицес/networkInterfaces | Нет | Нет |
> | Клаудсервицес/publicIPAddresses | Нет | Нет |
> | Клаудсервицес/Ролеинстанцес | Нет | Нет |
> | Клаудсервицес/Ролеинстанцес/networkInterfaces | Нет | Нет |
> | Клаудсервицес и роли | Нет | Нет |
> | дискакцессес | Да | Да |
> | diskEncryptionSets | Да | Да |
> | disks | Да | Да |
> | galleries | Да | Да |
> | galleries / applications | Нет | Нет |
> | коллекции, приложения и версии | Нет | нет |
> | galleries / images | нет | нет |
> | galleries / images / versions | нет | Нет |
> | хостграупс | Да | Да |
> | Хостграупс и узлы | Да | Да |
> | images | Да | Да |
> | проксимитиплацементграупс | Да | Да |
> | restorePointCollections | Да | Да |
> | Ресторепоинтколлектионс/Ресторепоинтс | Нет | Нет |
> | шаредвмекстенсионс | Да | Да |
> | Шаредвмекстенсионс и версии | Нет | Нет |
> | sharedVMImages | Да | Да |
> | Шаредвмимажес и версии | Нет | нет |
> | snapshots | Да | Да |
> | сшпубликкэйс | Да | Да |
> | virtualMachines | Да | Да |
> | virtualMachines и расширения | Да | Да |
> | virtualMachines/metricDefinitions | Нет | Нет |
> | virtualMachines/Рункоммандс | Да | Да |
> | virtualMachineScaleSets | Да | Да |
> | virtualMachineScaleSets и расширения | Нет | Нет |
> | virtualMachineScaleSets/networkInterfaces | Нет | Нет |
> | virtualMachineScaleSets/publicIPAddresses | Да | Нет |
> | virtualMachineScaleSets/virtualMachines | Нет | Нет |
> | virtualMachineScaleSets/virtualMachines/networkInterfaces | Нет | Нет |

> [!NOTE]
> Вы не можете добавить тег к виртуальной машине, которая была помечена как обобщенная. Вы помечаете виртуальную машину как обобщенную с помощью командлета [Set-AzVm-обобщенного](/powershell/module/Az.Compute/Set-AzVM) или [AZ VM generalize](/cli/azure/vm#az-vm-generalize).

## <a name="microsoftconnectedcache"></a>Microsoft. Коннектедкаче

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | каченодес | Да | Да |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | AggregatedCost | Нет | Нет |
> | сведения о балансе. | Нет | Нет |
> | сведения о бюджете; | Нет | Нет |
> | Расходы | Нет | Нет |
> | CostTags | Нет | нет |
> | credits | нет | нет |
> | события | нет | Нет |
> | Прогнозы | Нет | нет |
> | lots | нет | Нет |
> | Marketplace | Нет | Нет |
> | Pricesheets | Нет | нет |
> | products | нет | Нет |
> | ReservationDetails | Нет | Нет |
> | ресерватионрекоммендатиондетаилс | Нет | Нет |
> | ReservationRecommendations | Нет | Нет |
> | ReservationSummaries | Нет | Нет |
> | ReservationTransactions | Нет | Нет |
> | Теги | Нет | нет |
> | tenants | нет | Нет |
> | Термины | Нет | Нет |
> | UsageDetails | Нет | нет |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | containerGroups | Да | Да |
> | serviceAssociationLinks | Нет | нет |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | registries | Да | Да |
> | реестры и Ажентпулс | Да | Да |
> | registries / builds | нет | нет |
> | registries / builds / cancel | нет | Нет |
> | реестры/сборки/Жетлоглинк | Нет | Нет |
> | registries / buildTasks | Да | Да |
> | реестры, Буилдтаскс и шаги | Нет | Нет |
> | реестры и Евентгридфилтерс | Нет | Нет |
> | реестры и Експортпипелинес | Нет | Нет |
> | реестры и Женератекредентиалс | Нет | Нет |
> | реестры и Жетбуилдсаурцеуплоадурл | Нет | Нет |
> | реестры и учетные данные | Нет | Нет |
> | реестры и Импортимаже | Нет | Нет |
> | реестры и Импортпипелинес | Нет | Нет |
> | реестры и Пипелинерунс | Нет | Нет |
> | реестры и Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | реестры/Приватиндпоинтконнектионпроксиес/проверка | Нет | Нет |
> | реестры и Приватиндпоинтконнектионс | Нет | Нет |
> | реестры и Привателинкресаурцес | Нет | Нет |
> | реестры и Куеуебуилд | Нет | Нет |
> | реестры и Реженератекредентиал | Нет | Нет |
> | реестры и Реженератекредентиалс | Нет | нет |
> | registries / replications | Да | Да |
> | registries / runs | нет | нет |
> | registries / runs / cancel | нет | Нет |
> | реестры и Счедулерун | Нет | Нет |
> | registries / scopeMaps | Нет | Нет |
> | реестры и Таскрунс | Нет | нет |
> | registries / tasks | Да | Да |
> | registries / tokens | нет | Нет |
> | реестры и УпдатеполиЦиес | Нет | нет |
> | registries / webhooks | Да | Да |
> | реестры/веб-перехватчики/Жеткаллбаккконфиг | Нет | нет |
> | registries / webhooks / ping | нет | нет |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | containerServices | Да | Да |
> | managedClusters | Да | Да |
> | openShiftManagedClusters | Да | Да |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | видны узлы | Нет | Нет |
> | BillingAccounts | Нет | Нет |
> | сведения о бюджете; | Нет | Нет |
> | клаудконнекторс | Нет | Нет |
> | Соединители | Да | Да |
> | косталлокатионрулес | Нет | Нет |
> | Departments | Нет | Нет |
> | Измерения | Нет | Нет |
> | EnrollmentAccounts | Нет | Нет |
> | Экспортируемые элементы | Нет | Нет |
> | екстерналбиллингаккаунтс | Нет | Нет |
> | Екстерналбиллингаккаунтс и оповещения | Нет | Нет |
> | Екстерналбиллингаккаунтс и измерения | Нет | Нет |
> | Екстерналбиллингаккаунтс/прогноз | Нет | Нет |
> | Екстерналбиллингаккаунтс/запрос | Нет | Нет |
> | екстерналсубскриптионс | Нет | Нет |
> | Екстерналсубскриптионс и оповещения | Нет | Нет |
> | Екстерналсубскриптионс и измерения | Нет | Нет |
> | Екстерналсубскриптионс/прогноз | Нет | Нет |
> | Екстерналсубскриптионс/запрос | Нет | Нет |
> | Прогноз | Нет | Нет |
> | Аналитика | Нет | Нет |
> | Запрос | Нет | нет |
> | регистрация | нет | Нет |
> | Reportconfigs | Нет | Нет |
> | Отчеты | Нет | Нет |
> | Параметры | Нет | Нет |
> | шовбаккрулес | Нет | Нет |
> | Представления | Нет | нет |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | requests | нет | нет |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | associations | нет | Нет |
> | resourceProviders | Да | Да |

## <a name="microsoftd365customerinsights"></a>Microsoft. D365CustomerInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | instances | Да | Да |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | jobs | Да | Да |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | DataBoxEdgeDevices | Да | Да |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | workspaces | Да | Да |
> | рабочие области и Дбворкспацес | Нет | Нет |
> | рабочие области и Виртуалнетворкпирингс | Нет | нет |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | catalogs | Да | Да |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | dataFactories | Да | Да |
> | Коэффициенты и diagnosticSettings | Нет | Нет |
> | Коэффициенты и metricDefinitions | Нет | Нет |
> | dataFactorySchema | Нет | нет |
> | factories | Да | Да |
> | factories / integrationRuntimes | Нет | Нет |

> [!NOTE]
> Если в фабрике данных есть среды выполнения интеграции Azure SSIS, то их стоимость работы будет помечена тегами фабрики данных. Запуск среды выполнения интеграции Azure SSIS должен быть остановлен и перезапущен для применения новых тегов фабрики данных к их затратам на выполнение.

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | accounts / dataLakeStoreAccounts | Нет | Нет |
> | accounts / storageAccounts | Нет | Нет |
> | Accounts/storageAccounts/Containers | Нет | Нет |
> | Accounts/Трансфераналитиксунитс | Нет | нет |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | Accounts/Евентгридфилтерс | Нет | Нет |
> | accounts / firewallRules | Нет | нет |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | services; | нет | нет |
> | services / projects | нет | нет |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | баккупваултс | Да | Да |
> | ресаурцеоператионгатекиперс | Да | Да |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | accounts / shares | нет | нет |
> | accounts / shares / datasets | нет | нет |
> | accounts / shares / invitations | нет | нет |
> | accounts / shares / providersharesubscriptions | нет | Нет |
> | Accounts/shares/Синчронизатионсеттингс | Нет | нет |
> | accounts / sharesubscriptions | нет | Нет |
> | Accounts/шаресубскриптионс/Консумерсаурцедатасетс | Нет | нет |
> | accounts / sharesubscriptions / datasetmappings | нет | нет |
> | accounts / sharesubscriptions / triggers | нет | нет |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | servers | Да | Да |
> | servers / advisors | нет | нет |
> | servers / keys | нет | Нет |
> | серверы и Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | серверы и Приватиндпоинтконнектионс | Нет | Нет |
> | серверы и Привателинкресаурцес | Нет | Нет |
> | серверы и Куеритекстс | Нет | Нет |
> | серверы и Рековераблесерверс | Нет | Нет |
> | серверы и запуск | Нет | Нет |
> | серверы/прерывать | Нет | Нет |
> | серверы и Топкуеристатистикс | Нет | Нет |
> | servers / virtualNetworkRules | Нет | Нет |
> | серверы и Ваитстатистикс | Нет | нет |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | флексиблесерверс | Да | Да |
> | servers | Да | Да |
> | servers / advisors | нет | нет |
> | servers / keys | нет | Нет |
> | серверы и Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | серверы и Приватиндпоинтконнектионс | Нет | Нет |
> | серверы и Привателинкресаурцес | Нет | Нет |
> | серверы и Куеритекстс | Нет | Нет |
> | серверы и Рековераблесерверс | Нет | Нет |
> | серверы и запуск | Нет | Нет |
> | серверы/прерывать | Нет | Нет |
> | серверы и Топкуеристатистикс | Нет | Нет |
> | серверы и обновление | Нет | Нет |
> | servers / virtualNetworkRules | Нет | Нет |
> | серверы и Ваитстатистикс | Нет | нет |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | флексиблесерверс | Да | Да |
> | серверграупс | Да | Да |
> | servers | Да | Да |
> | servers / advisors | нет | нет |
> | servers / keys | нет | Нет |
> | серверы и Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | серверы и Приватиндпоинтконнектионс | Нет | Нет |
> | серверы и Привателинкресаурцес | Нет | Нет |
> | серверы и Куеритекстс | Нет | Нет |
> | серверы и Рековераблесерверс | Нет | Нет |
> | серверы и Топкуеристатистикс | Нет | Нет |
> | servers / virtualNetworkRules | Нет | Нет |
> | серверы и Ваитстатистикс | Нет | нет |
> | serversv2 | Да | Да |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | артифактсаурцес | Да | Да |
> | rollouts | Да | Да |
> | сервицетопологиес | Да | Да |
> | Сервицетопологиес и службы | Да | Да |
> | Сервицетопологиес/Services/Сервицеунитс | Да | Да |
> | steps | Да | Да |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | applicationgroups | Да | Да |
> | applicationgroups / applications | нет | нет |
> | applicationgroups / desktops | нет | нет |
> | applicationgroups / startmenuitems | нет | нет |
> | hostpools | Да | Да |
> | хостпулс/мсикспаккажес | Нет | Нет |
> | hostpools / sessionhosts | нет | нет |
> | hostpools / sessionhosts / usersessions | нет | нет |
> | hostpools / usersessions | нет | нет |
> | workspaces | Да | Да |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | еластикпулс | Да | Да |
> | Еластикпулс/Иосубтенантс | Да | Да |
> | Еластикпулс/Иосубтенантс/securitySettings | Нет | Нет |
> | IotHubs | Да | Да |
> | IotHubs/Евентгридфилтерс | Нет | Нет |
> | IotHubs/securitySettings | Нет | Нет |
> | ProvisioningServices | Да | Да |
> | usages | нет | Нет |

## <a name="microsoftdeviceupdate"></a>Microsoft. Девицеупдате

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | учетные записи и экземпляры | Да | Да |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | pipelines | Да | Да |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | controllers | Да | Да |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | labcenters | Да | Да |
> | labs | Да | Да |
> | labs / environments | Да | Да |
> | Labs и Сервицеруннерс | Да | Да |
> | Labs и virtualMachines | Да | Да |
> | schedules | Да | Да |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | дигиталтвинсинстанцес | Да | Да |
> | Дигиталтвинсинстанцес/конечные точки | Нет | нет |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | databaseAccountNames | Нет | Нет |
> | databaseAccounts | Да | Да |
> | ресторабледатабасеаккаунтс | Нет | нет |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | domains | Да | Да |
> | домены/Домаиновнершипидентифиерс | Нет | Нет |
> | generateSsoRequest | Нет | Нет |
> | topLevelDomains | Нет | Нет |
> | validateDomainRegistrationInformation | Нет | Нет |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | lcsprojects | Нет | Нет |
> | лкспрожектс/клауддеплойментс | Нет | Нет |
> | лкспрожектс и соединители | Нет | нет |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | services; | Да | Да |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | domains | Да | Да |
> | domains / topics | нет | Нет |
> | eventSubscriptions | Нет | Нет |
> | extensionTopics | Нет | Нет |
> | партнернамеспацес | Да | Да |
> | Партнернамеспацес/eventChannels | Нет | Нет |
> | партнеррегистратионс | Да | Да |
> | партнертопикс | Да | Да |
> | Партнертопикс/eventSubscriptions | Нет | Нет |
> | системтопикс | Да | Да |
> | Системтопикс/eventSubscriptions | Нет | нет |
> | topics | Да | Да |
> | topicTypes | Нет | нет |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | clusters | Да | Да |
> | пространства имен | Да | Да |
> | namespaces / authorizationrules | нет | нет |
> | namespaces / disasterrecoveryconfigs | нет | нет |
> | namespaces / eventhubs | нет | нет |
> | namespaces / eventhubs / authorizationrules | нет | нет |
> | namespaces / eventhubs / consumergroups | нет | нет |
> | namespaces / networkrulesets | нет | Нет |
> | пространства имен/Приватиндпоинтконнектионс | Нет | нет |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | експериментворкспацес | Да | Да |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | пространства имен | Да | Да |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | феатурепровидерс | Нет | нет |
> | features | нет | нет |
> | providers | нет | Нет |
> | субскриптионфеатуререгистратионс | Нет | Нет |

## <a name="microsoftgallery"></a>Microsoft.Gallery

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | enroll | Нет | Нет |
> | galleryitems | Нет | Нет |
> | generateartifactaccessuri | Нет | Нет |
> | myareas | Нет | Нет |
> | мяреас/области | Нет | Нет |
> | мяреас/области/области | Нет | Нет |
> | мяреас/области/области/галлеритемс | Нет | Нет |
> | мяреас/области/галлеритемс | Нет | Нет |
> | мяреас/галлеритемс | Нет | нет |
> | регистрация | нет | нет |
> | ресурсов | нет | Нет |
> | retrieveresourcesbyid | Нет | нет |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | аутоманажедаккаунтс | Да | Да |
> | аутоманажедвмконфигуратионпрофилес | Да | Да |
> | конфигуратионпрофилеассигнментс | Нет | Нет |
> | guestConfigurationAssignments | Нет | нет |
> | software | нет | Нет |
> | софтвареупдатепрофиле | Нет | Нет |
> | софтвареупдатес | Нет | нет |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | hanaInstances | Да | Да |
> | сапмониторс | Да | Да |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | дедикатедхсмс | Да | Да |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | clusters | Да | Да |
> | clusters / applications | нет | Нет |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | services; | Да | Да |
> | службы/иомтконнекторс | Нет | Нет |
> | службы/иомтконнекторс/подключения | Нет | Нет |
> | службы/иомтконнекторс/сопоставления | Нет | Нет |
> | службы/Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | службы/Приватиндпоинтконнектионс | Нет | Нет |
> | службы/Привателинкресаурцес | Нет | нет |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | machines | Да | Да |
> | Machines/Ассесспатчес | Нет | Нет |
> | machines / extensions | Да | Да |
> | Machines/Инсталлпатчес | Нет | нет |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Диспетчеры | Да | Да |

## <a name="microsofthybridnetwork"></a>Microsoft. Хибриднетворк

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Устройства | Да | Да |
> | нетворкфунктионс | Да | Да |
> | нетворкфунктионвендорс | Нет | Нет |
> | регистередсубскриптионс | Нет | Нет |
> | разработчиков | Нет | Нет |
> | поставщики и Вендорскус | Нет | Нет |
> | поставщики/Вендорскус/Превиевсубскриптионс | Нет | Нет |
> | виртуалнетворкфунктионс | Да | Да |
> | виртуалнетворкфунктионвендорс | Нет | нет |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | components | Да | Да |
> | нетворкскопес | Да | Да |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | jobs | Да | Да |

## <a name="microsoftinsights"></a>Microsoft.Insights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | actionGroups | Да | Да |
> | activityLogAlerts | Да | Да |
> | alertrules | Да | Да |
> | autoscalesettings | Да | Да |
> | components | Да | Да |
> | Components/Линкедсторажеаккаунтс | Нет | Нет |
> | Components/Проактиведетектионконфигс | Нет | Нет |
> | diagnosticSettings | Нет | Нет |
> | гуестдиагностиксеттингс | Да | Да |
> | гуестдиагностиксеттингсассоЦиатион | Да | Да |
> | logprofiles | Да | Да |
> | metricAlerts | Да | Да |
> | привателинкскопес | Да | Да |
> | Привателинкскопес/Приватиндпоинтконнектионс | Нет | Нет |
> | Привателинкскопес/Скопедресаурцес | Нет | Нет |
> | куерипаккс | Да | Да |
> | Куерипаккс/запросы | Нет | Нет |
> | счедуледкуерирулес | Да | Да |
> | webtests | Да | Да |
> | workbooks | Да | Да |
> | workbooktemplates | Да | Да |

## <a name="microsoftintune"></a>Microsoft.Intune

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | Нет | Нет |
> | diagnosticSettingsCategories | Нет | Нет |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | апптемплатес | Нет | Нет |
> | IoTApps | Да | Да |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | График | Да | Да |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | deletedVaults | Нет | Нет |
> | хсмпулс | Да | Да |
> | манажедхсмс | Да | Да |
> | vaults | Да | Да |
> | хранилища и accessPolicies | Нет | Нет |
> | хранилища и Евентгридфилтерс | Нет | Нет |
> | хранилища и ключи | Нет | Нет |
> | хранилища/ключи/версии | Нет | нет |
> | vaults / secrets | нет | Нет |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | коннектедклустерс | Да | Да |
> | регистередсубскриптионс | Нет | нет |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | extensions | Нет | Нет |
> | саурцеконтролконфигуратионс | Нет | нет |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | clusters | Да | Да |
> | clusters / attacheddatabaseconfigurations | нет | нет |
> | clusters / databases | нет | нет |
> | clusters / databases / dataconnections | нет | нет |
> | clusters / databases / eventhubconnections | нет | нет |
> | clusters / databases / principalassignments | нет | Нет |
> | кластеры и подключения к соединениям | Нет | нет |
> | clusters / principalassignments | нет | Нет |
> | кластеры/шаредидентитиес | Нет | нет |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | labaccounts | Да | нет |
> | users | нет | нет |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | hostingEnvironments | Да | Да |
> | integrationAccounts | Да | Да |
> | integrationServiceEnvironments | Да | Да |
> | integrationServiceEnvironments / managedApis | Нет | Нет |
> | исолатеденвиронментс | Да | Да |
> | workflows | Да | Да |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | commitmentPlans | Да | Да |
> | webServices | Да | Да |
> | Рабочие области | Да | Да |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | workspaces | Да | Да |
> | рабочие области и Батчендпоинтс | Да | Да |
> | рабочие области, Батчендпоинтси и развертывания | Да | Да |
> | рабочие области и коды | Нет | Нет |
> | рабочие области, коды и версии | Нет | нет |
> | workspaces / computes | нет | Нет |
> | рабочие области и хранилища данных | Нет | Нет |
> | рабочие области и Евентгридфилтерс | Нет | Нет |
> | рабочие области и задания | Нет | Нет |
> | рабочие области и Лабелингжобс | Нет | Нет |
> | рабочие области и linkedServices | Нет | Нет |
> | рабочие области и модели | Нет | Нет |
> | рабочие области, модели и версии | Нет | Нет |
> | рабочие области и Онлининдпоинтс | Да | Да |
> | рабочие области, Онлининдпоинтси и развертывания | Да | Да |
 
> [!NOTE]
> Теги рабочей области не распространяются на кластеры вычислений и экземпляры вычислений. 

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | апплюпдатес | Нет | Нет |
> | конфигуратионассигнментс | Нет | Нет |
> | маинтенанцеконфигуратионс | Да | Да |
> | публикмаинтенанцеконфигуратионс | Нет | Нет |
> | updates | нет | нет |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Identities | Нет | Нет |
> | userAssignedIdentities | Да | Да |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | манажеднетворкс | Да | Да |
> | Манажеднетворкс/Манажеднетворкграупс | Да | Да |
> | Манажеднетворкс/МанажеднетворкпирингполиЦиес | Да | Да |
> | уведомление | Да | Да |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | маркетплацерегистратиондефинитионс | Нет | Нет |
> | регистратионассигнментс | Нет | Нет |
> | регистратиондефинитионс | Нет | нет |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | getEntities | Нет | Нет |
> | managementGroups | Нет | Нет |
> | Манажементграупс/параметры | Нет | нет |
> | ресурсов | нет | Нет |
> | startTenantBackfill | Нет | Нет |
> | tenantBackfillStatus | Нет | нет |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | Accounts/Евентгридфилтерс | Нет | Нет |
> | Accounts/Приватеатласес | Да | Да |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | макк | Нет | нет |
> | offers | нет | Нет |
> | offerTypes | Нет | Нет |
> | Оффертипес и издатели | Нет | Нет |
> | Оффертипес, издатели и предложения | Нет | Нет |
> | Оффертипес, издатели, предложения и планы | Нет | Нет |
> | Оффертипес/Publishers/Offers/Plans/Agreement | Нет | Нет |
> | Оффертипес/Publishers/предложения, планы и конфигурации | Нет | Нет |
> | Оффертипес/Publishers/Offers/Plans/Configurations/Импортимаже | Нет | нет |
> | privategalleryitems | нет | Нет |
> | приватестореклиент | Нет | Нет |
> | приватесторес | Нет | Нет |
> | Приватесторес/предложения | Нет | нет |
> | products | нет | нет |
> | publishers | нет | нет |
> | publishers / offers | нет | нет |
> | publishers / offers / amendments | нет | нет |
> | регистрация | нет | нет |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | classicDevServices | Да | Да |
> | updateCommunicationPreference | Нет | нет |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | agreements | нет | нет |
> | offertypes | нет | нет |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | mediaservices | Да | Да |
> | mediaservices/Аккаунтфилтерс | Нет | нет |
> | mediaservices / assets | нет | Нет |
> | mediaservices/активы/Ассетфилтерс | Нет | Нет |
> | mediaservices/КонтенткэйполиЦиес | Нет | Нет |
> | mediaservices/Евентгридфилтерс | Нет | Нет |
> | mediaservices/Лививентоператионс | Нет | Нет |
> | mediaservices / liveEvents | Да | Да |
> | mediaservices / liveEvents / liveOutputs | Нет | Нет |
> | mediaservices/Ливеаутпутоператионс | Нет | Нет |
> | mediaservices/Медиаграфс | Нет | Нет |
> | mediaservices/Приватиндпоинтконнектионоператионс | Нет | Нет |
> | mediaservices/Приватиндпоинтконнектионпроксиес | Нет | Нет |
> | mediaservices/Приватиндпоинтконнектионс | Нет | Нет |
> | mediaservices/Стреаминжендпоинтоператионс | Нет | Нет |
> | mediaservices / streamingEndpoints | Да | Да |
> | mediaservices/Стреаминглокаторс | Нет | Нет |
> | mediaservices/СтреамингполиЦиес | Нет | нет |
> | mediaservices / transforms | нет | нет |
> | mediaservices / transforms / jobs | нет | нет |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | аппклустерс | Да | Да |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | ассессментпрожектс | Да | Да |
> | migrateprojects | Да | Да |
> | мовеколлектионс | Да | Да |
> | projects | Да | Да |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | холографиксброадкастаккаунтс | Да | Да |
> | обжектундерстандингаккаунтс | Да | Да |
> | ремотерендерингаккаунтс | Да | Да |
> | спатиаланчорсаккаунтс | Да | Да |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | нетаппаккаунтс | Да | Нет |
> | Нетаппаккаунтс/Аккаунтбаккупс | Нет | Нет |
> | Нетаппаккаунтс/КапаЦитипулс | Да | Нет |
> | Нетаппаккаунтс/КапаЦитипулс/тома | Да | Нет |
> | Нетаппаккаунтс/КапаЦитипулс/тома/моментальные снимки | Нет | нет |

## <a name="microsoftnetwork"></a>Microsoft.Network.

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | applicationGateways | Да | Да |
> | аппликатионгатевайвебаппликатионфиреваллполиЦиес | Да | Да |
> | applicationSecurityGroups | Да | Да |
> | azureFirewallFqdnTags | Нет | Нет |
> | azureFirewalls | Да | Нет |
> | bastionHosts | Да | Нет |
> | bgpServiceCommunities | Нет | нет |
> | connections | Да | Да |
> | ddosCustomPolicies | Да | Да |
> | ddosProtectionPlans | Да | Да |
> | dnsOperationStatuses | Нет | нет |
> | dnszones | Да | Да |
> | днсзонес/A | Нет | Нет |
> | днсзонес/AAAA | Нет | нет |
> | dnszones / all | нет | Нет |
> | днсзонес/CAA | Нет | Нет |
> | днсзонес и CNAME | Нет | Нет |
> | днсзонес/MX | Нет | Нет |
> | днсзонес/NS | Нет | Нет |
> | днсзонес/PTR | Нет | нет |
> | dnszones / recordsets | нет | Нет |
> | днсзонес/SOA | Нет | Нет |
> | днсзонес/SRV | Нет | Нет |
> | днсзонес/TXT | Нет | Нет |
> | expressRouteCircuits | Да | Да |
> | expressRouteCrossConnections | Да | Да |
> | експрессраутегатевайс | Да | Да |
> | експрессраутепортс | Да | Да |
> | expressRouteServiceProviders | Нет | Нет |
> | firewallPolicies | Да | Да |
> | frontdoors | Да, но ограничено (см. [Примечание ниже](#frontdoor)) | Да |
> | фронтдурвебаппликатионфиреваллманажедрулесетс | Да, но ограничено (см. [Примечание ниже](#frontdoor)) | Нет |
> | frontdoorWebApplicationFirewallPolicies | Да, но ограничено (см. [Примечание ниже](#frontdoor)) | Да |
> | getDnsResourceReference | Нет | Нет |
> | internalNotify | Нет | Нет |
> | ипграупс | Да | Да |
> | loadBalancers | Да | Да |
> | localNetworkGateways | Да | Да |
> | natGateways | Да | Да |
> | networkIntentPolicies | Да | Да |
> | networkInterfaces | Да | Да |
> | networkProfiles | Да | Да |
> | networkSecurityGroups | Да | Да |
> | networkWatchers | Да | Да |
> | Нетворкватчерс/Коннектионмониторс | Да | Нет |
> | Нетворкватчерс/панель | Нет | Нет |
> | Нетворкватчерс/lenses | Да | Нет |
> | Нетворкватчерс/Пингмешес | Да | Нет |
> | p2sVpnGateways | Да | Да |
> | приватеднсоператионстатусес | Нет | Нет |
> | privateDnsZones | Да | Да |
> | Приватеднсзонес/A | Нет | Нет |
> | Приватеднсзонес/AAAA | Нет | Нет |
> | Приватеднсзонес/все | Нет | Нет |
> | Приватеднсзонес и CNAME | Нет | Нет |
> | Приватеднсзонес/MX | Нет | Нет |
> | Приватеднсзонес/PTR | Нет | Нет |
> | Приватеднсзонес/SOA | Нет | Нет |
> | Приватеднсзонес/SRV | Нет | Нет |
> | Приватеднсзонес/TXT | Нет | Нет |
> | privateDnsZones / virtualNetworkLinks | Да | Да |
> | приватиндпоинтс | Да | Да |
> | privateLinkServices | Да | Да |
> | publicIPAddresses | Да | Да |
> | publicIPPrefixes | Да | Да |
> | routeFilters | Да | Да |
> | routeTables | Да | Да |
> | serviceEndpointPolicies | Да | Да |
> | trafficManagerGeographicHierarchies | Нет | нет |
> | trafficmanagerprofiles | Да | Да |
> | trafficmanagerprofiles/heatMaps | Нет | Нет |
> | траффикманажерусерметрикскэйс | Нет | Нет |
> | virtualHubs | Да | Да |
> | virtualNetworkGateways | Да | Да |
> | virtualNetworks | Да | Да |
> | virtualNetworks и подсети | Нет | Нет |
> | virtualNetworkTaps | Да | Да |
> | virtualWans | Да | Нет |
> | vpnGateways | Да | Да |
> | vpnSites | Да | Да |
> | webApplicationFirewallPolicies | Да | Да |

<a id="frontdoor"></a>

> [!NOTE]
> Для службы "Передняя дверца Azure" можно применять теги при создании ресурса, но обновление или добавление тегов в настоящее время не поддерживается.


## <a name="microsoftnotebooks"></a>Microsoft. Notebooks

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | нотебукпроксиес | Нет | Нет |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | пространства имен | Да | Нет |
> | namespaces / notificationHubs | Да | нет |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | оснамеспацес | Да | Да |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | хипервситес | Да | Да |
> | импортситес | Да | Да |
> | мастерситес | Да | Да |
> | серверситес | Да | Да |
> | вмвареситес | Да | Да |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | clusters | Да | Да |
> | делетедворкспацес | Нет | Нет |
> | linkTargets | Нет | Нет |
> | storageInsightConfigs | Нет | нет |
> | workspaces | Да | Да |
> | рабочие области и экспорты | Нет | Нет |
> | рабочие области и источники данных | Нет | Нет |
> | рабочие области и linkedServices | Нет | Нет |
> | рабочие области и Линкедсторажеаккаунтс | Нет | нет |
> | workspaces / metadata | нет | нет |
> | workspaces / query | нет | Нет |
> | рабочие области и Скопедпривателинкпроксиес | Нет | Нет |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | managementassociations | нет | нет |
> | managementconfigurations | Да | Да |
> | solutions | Да | Да |
> | узел "Представления" | Да | Да |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | легаципирингс | Нет | Нет |
> | пираснс | Нет | нет |
> | peerings | Да | Да |
> | пирингсервицекаунтриес | Нет | Нет |
> | пирингсервицепровидерс | Нет | Нет |
> | пирингсервицес | Да | Да |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | аттестации | Нет | Нет |
> | policyEvents | Нет | Нет |
> | полициметадата | Нет | Нет |
> | policyStates | Нет | Нет |
> | policyTrackedResources | Нет | нет |
> | remediations | нет | нет |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | consoles | нет | нет |
> | dashboards | Да | Да |
> | userSettings | Нет | нет |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | привателинксервицесфорповерби | Да | Да |
> | tenants | Да | Да |
> | Клиенты и рабочие области | Нет | Нет |
> | workspaceCollections | Да | Да |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | capacities | Да | Да |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | делетедаккаунтс | Нет | нет |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | providerRegistrations | Нет | Нет |
> | Провидеррегистратионс/Дефаултроллаутс | Нет | Нет |
> | Провидеррегистратионс/Ресаурцетиперегистратионс | Нет | нет |
> | rollouts | Да | Да |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Рабочие области | Да | Да |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | backupProtectedItems | Нет | нет |
> | vaults | Да | Да |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | опеншифтклустерс | Да | Да |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | пространства имен | Да | Да |
> | namespaces / authorizationrules | нет | нет |
> | namespaces / hybridconnections | нет | нет |
> | namespaces / hybridconnections / authorizationrules | нет | Нет |
> | пространства имен/Приватиндпоинтконнектионс | Нет | нет |
> | namespaces / wcfrelays | нет | нет |
> | namespaces / wcfrelays / authorizationrules | нет | нет |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Запросы | Да | Да |
> | ресаурцечанжедетаилс | Нет | Нет |
> | ресаурцечанжес | Нет | нет |
> | ресурсов | нет | Нет |
> | ресаурцешистори | Нет | Нет |
> | subscriptionsStatus | Нет | нет |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | availabilityStatuses | Нет | Нет |
> | childAvailabilityStatuses | Нет | Нет |
> | childResources | Нет | нет |
> | emergingissues | нет | нет |
> | события | нет | Нет |
> | impactedResources | Нет | нет |
> | метаданные | нет | нет |
> | Уведомления | нет | нет |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | калкулатетемплатехаш | Нет | нет |
> | deployments | Да | нет |
> | deployments / operations | нет | Нет |
> | deploymentScripts | Да | Да |
> | deploymentScripts и журналы | Нет | нет |
> | ссылки | нет | Нет |
> | notifyResourceJobs | Нет | нет |
> | providers | нет | Нет |
> | resourceGroups | Да | нет |
> | subscriptions | Да | Нет |
> | темплатеспекс | Да | Да |
> | Темплатеспекс и версии | Да | Да |
> | tenants | нет | нет |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | веб-масштабированием; | Да | Да |
> | saasresources | нет | Нет |

## <a name="microsoftscvmm"></a>Microsoft. ScVmm

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | облака | Да | Да |
> | VirtualMachines | Да | Да |
> | виртуалмачинетемплатес | Да | Да |
> | VirtualNetworks | Да | Да |
> | вммсерверс | Да | Да |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | resourceHealthMetadata | Нет | Нет |
> | searchServices | Да | Да |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | адаптивенетворкхарденингс | Нет | Нет |
> | advancedThreatProtectionSettings | Нет | нет |
> | оповещения | нет | Нет |
> | алертссуппрессионрулес | Нет | Нет |
> | allowedConnections | Нет | Нет |
> | applicationWhitelistings | Нет | Нет |
> | ассессментметадата | Нет | нет |
> | assessments | нет | Нет |
> | аутодисмиссалертсрулес | Нет | нет |
> | automations | Да | Да |
> | AutoProvisioningSettings | Нет | Нет |
> | Compliances | Нет | нет |
> | соединители | Нет | Нет |
> | dataCollectionAgents | Нет | Нет |
> | девицесекуритиграупс | Нет | Нет |
> | discoveredSecuritySolutions | Нет | Нет |
> | externalSecuritySolutions | Нет | Нет |
> | InformationProtectionPolicies | Нет | Нет |
> | иотдефендерсеттингс | Нет | Нет |
> | иотсекуритисолутионс | Да | Да |
> | Иотсекуритисолутионс/Аналитиксмоделс | Нет | Нет |
> | Иотсекуритисолутионс/Аналитиксмоделс/Аггрегатедалертс | Нет | Нет |
> | Иотсекуритисолутионс/Аналитиксмоделс/Аггрегатедрекоммендатионс | Нет | Нет |
> | Иотсекуритисолутионс/Иоталертс | Нет | Нет |
> | Иотсекуритисолутионс/Иоталерттипес | Нет | Нет |
> | Иотсекуритисолутионс/Иотрекоммендатионс | Нет | Нет |
> | Иотсекуритисолутионс/Иотрекоммендатионтипес | Нет | Нет |
> | иотсенсорс | Нет | Нет |
> | jitNetworkAccessPolicies | Нет | Нет |
> | житполиЦиес | Нет | нет |
> | политики | нет | нет |
> | pricings | нет | Нет |
> | регулаторикомплианцестандардс | Нет | Нет |
> | Регулаторикомплианцестандардс/Регулаторикомплианцеконтролс | Нет | Нет |
> | Регулаторикомплианцестандардс/Регулаторикомплианцеконтролс/Регулаторикомплианцеассессментс | Нет | Нет |
> | секурескореконтролдефинитионс | Нет | Нет |
> | секурескореконтролс | Нет | Нет |
> | секурескорес | Нет | Нет |
> | Секурескорес/Секурескореконтролс | Нет | Нет |
> | securityContacts | Нет | Нет |
> | securitySolutions | Нет | Нет |
> | securitySolutionsReferenceData | Нет | Нет |
> | securityStatuses | Нет | Нет |
> | securityStatusesSummaries | Нет | Нет |
> | сервервулнерабилитяссессментс | Нет | нет |
> | Параметры | нет | Нет |
> | склвулнерабилитяссессментс | Нет | Нет |
> | подоценка | Нет | нет |
> | задачи | нет | нет |
> | topologies | нет | Нет |
> | workspaceSettings | Нет | Нет |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | Нет | Нет |
> | diagnosticSettingsCategories | Нет | нет |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | aggregations | нет | Нет |
> | alertRules | Нет | Нет |
> | алертрулетемплатес | Нет | Нет |
> | аутоматионрулес | Нет | нет |
> | bookmarks | нет | нет |
> | cases | нет | Нет |
> | Подключения к компонентам | Нет | Нет |
> | датаконнекторсчеккрекуирементс | Нет | нет |
> | Сущности | нет | Нет |
> | ентитикуериес | Нет | нет |
> | incidents | нет | Нет |
> | оффицеконсентс | Нет | нет |
> | Параметры | нет | Нет |
> | среатинтеллиженце | Нет | Нет |
> | ватчлистс | Нет | нет |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | консолесервицес | Нет | нет |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | пространства имен | Да | Да |
> | namespaces / authorizationrules | нет | нет |
> | namespaces / disasterrecoveryconfigs | нет | нет |
> | namespaces / eventgridfilters | нет | нет |
> | namespaces / networkrulesets | нет | Нет |
> | пространства имен/Приватиндпоинтконнектионс | Нет | нет |
> | namespaces / queues | нет | нет |
> | namespaces / queues / authorizationrules | нет | нет |
> | namespaces / topics | нет | нет |
> | namespaces / topics / authorizationrules | нет | нет |
> | namespaces / topics / subscriptions | нет | нет |
> | namespaces / topics / subscriptions / rules | нет | Нет |
> | premiumMessagingRegions | Нет | нет |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | веб-масштабированием; | Да | Да |
> | clusters | Да | Да |
> | clusters / applications | нет | Нет |
> | containerGroups | Да | Да |
> | контаинерграупсетс | Да | Да |
> | edgeclusters | Да | Да |
> | еджеклустерс и приложения | Нет | нет |
> | managedclusters | Да | Да |
> | манажедклустерс/NodeType | Нет | нет |
> | networks | Да | Да |
> | secretstores | Да | Да |
> | секретсторес и сертификаты | Нет | Нет |
> | секретсторес/секреты | Нет | нет |
> | volumes. | Да | Да |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | веб-масштабированием; | Да | Да |
> | containerGroups | Да | Да |
> | gateways | Да | Да |
> | networks | Да | Да |
> | секретные коды | Да | Да |
> | volumes. | Да | Да |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | providerRegistrations | Нет | Нет |
> | Провидеррегистратионс/Ресаурцетиперегистратионс | Нет | нет |
> | rollouts | Да | Да |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | SignalR | Да | Да |
> | SignalR/Евентгридфилтерс | Нет | Нет |

## <a name="microsoftsingularity"></a>Microsoft. Единственное,

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | Да |
> | Accounts/АккаунткуотаполиЦиес | Нет | Нет |
> | Accounts/ГраупполиЦиес | Нет | Нет |
> | учетные записи и задания | Нет | Нет |
> | Accounts/Сторажеконтаинерс | Нет | нет |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | хибридусебенефитс | Нет | нет |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | applicationDefinitions | Да | Да |
> | веб-масштабированием; | Да | Да |
> | jitRequests | Да | Да |


## <a name="microsoftsql"></a>Microsoft.SQL

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | managedInstances | Да | Да |
> | Манажединстанцес и базы данных | Нет | Нет |
> | Манажединстанцес/databases/БаккупшорттермретентионполиЦиес | Нет | Нет |
> | Манажединстанцес/базы данных, схемы, таблицы, столбцы и Сенситивитилабелс | Нет | Нет |
> | Манажединстанцес/databases/Вулнерабилитяссессментс | Нет | Нет |
> | Манажединстанцес/базы данных/Вулнерабилитяссессментс/правила и базовые планы | Нет | Нет |
> | Манажединстанцес/Енкриптионпротектор | Нет | Нет |
> | Манажединстанцес и ключи | Нет | Нет |
> | Манажединстанцес/Ресторабледроппеддатабасес/БаккупшорттермретентионполиЦиес | Нет | Нет |
> | Манажединстанцес/Вулнерабилитяссессментс | Нет | Нет |
> | Лонгтермретентионманажединстанце/Лонгтермретентиондатабасе/режим longtermretentionbackup | Нет | нет |
> | servers | Да | Да |
> | servers / administrators | нет | Нет |
> | серверы и Коммуникатионлинкс | Нет | нет |
> | servers / databases | Да (см. [Примечание ниже](#sqlnote)) | Да |
> | серверы и Енкриптионпротектор | Нет | Нет |
> | servers / firewallRules | Нет | нет |
> | servers / keys | нет | Нет |
> | серверы и Ресторабледроппеддатабасес | Нет | нет |
> | servers / serviceobjectives | нет | Нет |
> | серверы и Тдецертификатес | Нет | Нет |
> | Лонгтермретентионсервер/Лонгтермретентиондатабасе/режим longtermretentionbackup | Нет | Нет |
> | виртуалклустерс | Нет | Нет |

<a id="sqlnote"></a>

> [!NOTE]
> База данных master не поддерживает теги, а другие базы данных, включая базы данных Azure синапсе Analytics, поддерживают теги. Базы данных Azure синапсе Analytics должны находиться в активном состоянии (не приостановлено).

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | SqlVirtualMachineGroups | Да | Да |
> | Склвиртуалмачинеграупс/Аваилабилитиграуплистенерс | Нет | Нет |
> | SqlVirtualMachines | Да | Да |

## <a name="microsoftstorage"></a>Microsoft.Storage;

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | делетедаккаунтс | Нет | Нет |
> | storageAccounts | Да | Да |
> | storageAccounts / blobServices | Нет | Нет |
> | storageAccounts / fileServices | Нет | Нет |
> | storageAccounts/Куеуесервицес | Нет | Нет |
> | storageAccounts и службы | Нет | Нет |
> | storageAccounts/Services/metricDefinitions | Нет | Нет |
> | storageAccounts/Таблесервицес | Нет | нет |
> | usages | нет | нет |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | caches | Да | Да |
> | кэши/Сторажетаржетс | Нет | Нет |
> | усажемоделс | Нет | Нет |

## <a name="microsoftstoragereplication"></a>Microsoft. Сторажерепликатион

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | репликатионграупс | Нет | нет |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Да | Да |
> | Сторажесинксервицес/Регистередсерверс | Нет | Нет |
> | storageSyncServices / syncGroups | Нет | Нет |
> | Сторажесинксервицес/Синкграупс/Клаудендпоинтс | Нет | Нет |
> | Сторажесинксервицес/Синкграупс/Серверендпоинтс | Нет | Нет |
> | Сторажесинксервицес и рабочие процессы | Нет | нет |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Да | Да |
> | Сторажесинксервицес/Регистередсерверс | Нет | Нет |
> | storageSyncServices / syncGroups | Нет | Нет |
> | Сторажесинксервицес/Синкграупс/Клаудендпоинтс | Нет | Нет |
> | Сторажесинксервицес/Синкграупс/Серверендпоинтс | Нет | Нет |
> | Сторажесинксервицес и рабочие процессы | Нет | нет |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Да | Да |
> | Сторажесинксервицес/Регистередсерверс | Нет | Нет |
> | storageSyncServices / syncGroups | Нет | Нет |
> | Сторажесинксервицес/Синкграупс/Клаудендпоинтс | Нет | Нет |
> | Сторажесинксервицес/Синкграупс/Серверендпоинтс | Нет | Нет |
> | Сторажесинксервицес и рабочие процессы | Нет | нет |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | managers | Да | Да |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | clusters | Да | Да |
> | кластеры/Приватиндпоинтс | Нет | нет |
> | streamingjobs | Да (см. примечание ниже) | Да |

> [!NOTE]
> Невозможно добавить тег при выполнении streamingjobs. Остановите ресурс, чтобы добавить тег.

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | акцептчанжетенант | Нет | Нет |
> | псевдонимы | Нет | Нет |
> | cancel | нет | Нет |
> | чанжетенантрекуест | Нет | Нет |
> | чанжетенантстатус | Нет | Нет |
> | CreateSubscription | Нет | Нет |
> | enable | нет | Нет |
> | переименовать | нет | Нет |
> | SubscriptionDefinitions | Нет | Нет |
> | SubscriptionOperations | Нет | нет |
> | subscriptions | нет | нет |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | привателинкхубс | Да | Да |
> | workspaces | Да | Да |
> | рабочие области и Бигдатапулс | Да | Да |
> | рабочие области и Оператионстатусес | Нет | Нет |
> | рабочие области и Склдатабасес | Да | Да |
> | рабочие области и Склпулс | Да | Да |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | environments | Да | Нет |
> | environments / accessPolicies | Нет | нет |
> | environments / eventsources | Да | Нет |
> | environments / referenceDataSets | Да | нет |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | stores | Да | Да |
> | Магазины и accessPolicies | Нет | нет |
> | stores / services | нет | нет |
> | stores / services / tokens | нет | нет |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | имажетемплатес | Да | Да |
> | Имажетемплатес/Рунаутпутс | Нет | Нет |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | аркзонес | Да | Да |
> | ресаурцепулс | Да | Да |
> | Серверами vCenter | Да | Да |
> | VirtualMachines | Да | Да |
> | виртуалмачинетемплатес | Да | Да |
> | VirtualNetworks | Да | Да |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | дедикатедклауднодес | Да | Да |
> | дедикатедклаудсервицес | Да | Да |
> | virtualMachines | Да | Да |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | Устройства | Да | Да |
> | регистередсубскриптионс | Нет | Нет |
> | разработчиков | Нет | Нет |
> | поставщики и номера SKU | Нет | Нет |
> | поставщики и внфс | Нет | Нет |
> | виртуалнетворкфунктионскус | Нет | нет |
> | vnfs | Да | Да |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | accounts | Да | нет |
> | Планы | Да | Нет |
> | регистередсубскриптионс | Нет | нет |

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | apiManagementAccounts | Нет | Нет |
> | Апиманажементаккаунтс/списков ACL для | Нет | Нет |
> | Апиманажементаккаунтс/API | Нет | Нет |
> | Апиманажементаккаунтс/API/списков ACL для | Нет | Нет |
> | Апиманажементаккаунтс/API/списков ACL для | Нет | Нет |
> | Апиманажементаккаунтс/API/подключения | Нет | Нет |
> | Апиманажементаккаунтс/API/Connections/списков ACL для | Нет | Нет |
> | Апиманажементаккаунтс/API/Локализеддефинитионс | Нет | Нет |
> | Апиманажементаккаунтс/списков ACL для | Нет | Нет |
> | Апиманажементаккаунтс и подключения | Нет | Нет |
> | billingMeters | Нет | нет |
> | certificates | Да | Да |
> | connectionGateways | Да | Да |
> | connections | Да | Да |
> | customApis | Да | Да |
> | deletedSites | Нет | Нет |
> | hostingEnvironments | Да | Да |
> | hostingEnvironments/Евентгридфилтерс | Нет | Нет |
> | hostingEnvironments/Мултиролепулс | Нет | Нет |
> | hostingEnvironments/внешнего размещения | Нет | Нет |
> | кубинвиронментс | Да | Да |
> | publishingUsers | Нет | нет |
> | к просмотру фильмов | нет | Нет |
> | resourceHealthMetadata | Нет | нет |
> | runtimes | нет | Нет |
> | serverFarms | Да | Да |
> | serverFarms/Евентгридфилтерс | Нет | Нет |
> | serverFarms/Фирстпартяппс | Нет | Нет |
> | serverFarms/Фирстпартяппс/Кэйваултсеттингс | Нет | нет |
> | sites | Да | Да |
> | сайты и конфигурация  | Нет | Нет |
> | Sites/Евентгридфилтерс | Нет | Нет |
> | Sites/Хостнамебиндингс | Нет | Нет |
> | Sites/файл networkconfig | Нет | нет |
> | sites / premieraddons | Да | Да |
> | sites / slots | Да | Да |
> | сайты/слоты/Евентгридфилтерс | Нет | Нет |
> | сайты/слоты/Хостнамебиндингс | Нет | Нет |
> | сайты/слоты/файл networkconfig | Нет | Нет |
> | sourceControls | Нет | Нет |
> | статикситес | Да | Да |
> | validate | нет | Нет |
> | verifyHostingEnvironmentVnet | Нет | Нет |

## <a name="microsoftwindowsdefenderatp"></a>Microsoft.WindowsDefenderATP

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | Нет | Нет |
> | diagnosticSettingsCategories | Нет | Нет |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | мултиплеактиватионкэйс | Да | Да |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | DeviceServices | Да | Да |

## <a name="microsoftworkloadbuilder"></a>Microsoft. Ворклоадбуилдер

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | рабочие нагрузки | Да | Да |
> | рабочие нагрузки и экземпляры | Нет | Нет |
> | рабочие нагрузки и версии | Нет | Нет |
> | рабочие нагрузки, версии и артефакты | Нет | нет |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Тип ресурса | Поддержка тегов | Тег в отчете о затратах |
> | ------------- | ----------- | ----------- |
> | components | нет | Нет |
> | componentsSummary | Нет | Нет |
> | monitorInstances | Нет | Нет |
> | monitorInstancesSummary | Нет | нет |
> | monitors | нет | Нет |
> | notificationSettings | Нет | Нет |

## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как применить теги к ресурсам, см. в статье, посвященной [использованию тегов для организации ресурсов Azure](tag-resources.md).

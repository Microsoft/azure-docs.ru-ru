---
title: Общие сведения о тегах FQDN для Брандмауэра Azure
description: Тег FQDN — это группа полных доменных имен (FQDN), связанных с известными службами Майкрософт.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: article
ms.date: 06/30/2020
ms.author: victorh
ms.openlocfilehash: c0bd2f6a021baae80dafcc3d544e1062d8e022e8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94653340"
---
# <a name="fqdn-tags-overview"></a>Общие сведения о тегах FQDN

Тег FQDN — это группа полных доменных имен (FQDN), связанных с известными службами Майкрософт. Теги FQDN можно использовать в правилах приложений, чтобы разрешить нужный исходящий сетевой трафик через брандмауэр.

Например, чтобы вручную разрешить сетевой трафик для Центра обновления Windows через брандмауэр, нужно создать несколько правил приложения согласно документации Майкрософт. А с помощью тегов FQDN можно создать правило приложения и включить тег **Центра обновлений Windows**. После этого сетевой трафик сможет проходить через брандмауэр к конечным точкам Центра обновления Windows.

Вы не можете создать собственные теги FQDN или указать, какие имена FQDN должны входить в тег. Майкрософт управляет полными доменными именами, включенными в теги FQDN, и обновляет тег, если эти имена меняются. 

<!--- screenshot of application rule with a FQDN tag.-->

В следующей таблице показаны текущие теги FQDN, которые можно использовать. Корпорация Майкрософт поддерживает эти теги и периодически будет добавлять новые.

## <a name="current-fqdn-tags"></a>Текущие теги полного доменного имени

|Тег FQDN  |Описание  |
|---------|---------|
|Центра обновления Windows;     |Разрешает исходящий доступ к Центру обновления Майкрософт, как описано в разделе [Настройка брандмауэра для обновлений программного обеспечения](/mem/configmgr/sum/get-started/install-a-software-update-point).|
|диагностика Windows.|Разрешает исходящий доступ ко всем [конечным точкам диагностики Windows](/windows/privacy/configure-windows-diagnostic-data-in-your-organization#endpoints).|
|Служба Microsoft Active Protection Service (MAPS)|Разрешает исходящий доступ к [MAPS](https://cloudblogs.microsoft.com/enterprisemobility/2016/05/31/important-changes-to-microsoft-active-protection-service-maps-endpoint/).|
|Среда службы приложений (ASE).|Разрешает исходящий доступ к трафику платформы ASE. Этот тег не распространяется на конечные точки пользовательского хранилища и SQL, созданные ASE. Их следует включить через [конечные точки службы](../virtual-network/tutorial-restrict-network-access-to-resources.md) или добавить вручную.<br><br>Дополнительные сведения об интеграции брандмауэра Azure с ASE см. в разделе [блокировка среда службы приложений](../app-service/environment/firewall-integration.md#configuring-azure-firewall-with-your-ase).|
|Azure Backup|Разрешает исходящий доступ к службам Azure Backup.|
|Azure HDInsight|Разрешает исходящий доступ для трафика платформы HDInsight. Этот тег не охватывает специфические для клиента хранилища или трафик SQL из HDInsight. Включите их с помощью [конечных точек службы](../virtual-network/tutorial-restrict-network-access-to-resources.md) или добавьте их вручную.|
|Виндовсвиртуалдесктоп (ВВД)|Разрешает исходящий трафик платформы виртуальных рабочих столов Windows. Этот тег не охватывает специфические для развертывания хранилища и конечные точки служебной шины, созданные ВВД. Кроме того, требуются правила сети DNS и службы KMS. Дополнительные сведения об интеграции брандмауэра Azure с ВВД см. в статье [Использование брандмауэра Azure для защиты развертывания виртуальных рабочих столов в Windows](protect-windows-virtual-desktop.md).|
|Служба Azure Kubernetes (AKS)|Разрешает исходящий доступ к AKS. Дополнительные сведения см. [в статье Использование брандмауэра Azure для защиты развертываний Azure Kubernetes Service (AKS)](protect-azure-kubernetes-service.md).|

> [!NOTE]
> При выборе тега FQDN в правиле приложения поле protocol:port должно иметь значение **https**.

## <a name="next-steps"></a>Дальнейшие действия

См. дополнительные сведения о том, как [развернуть и настроить Брандмауэр Azure с помощью портала Azure](tutorial-firewall-deploy-portal.md).
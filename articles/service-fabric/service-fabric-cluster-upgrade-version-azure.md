---
title: Обновление версии Service Fabric Azure кластера
description: Обновление кода и (или) конфигурации Service Fabric, под управлением которой работает кластер Service Fabric, в том числе изменение режима обновления кластера, обновление сертификатов, добавление портов приложений, применение исправлений для ОС и т. д. Чего можно ожидать при выполнении обновлений?
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: 01fe916f0ee78c8481ac6b17b8f7409b47c852ee
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90564293"
---
# <a name="upgrade-the-service-fabric-version-of-a-cluster"></a>Изменение версии Service Fabric в кластере

Для любой современной системы разработка с учетом возможности модернизации является неотъемлемой составляющей успеха продукта. Кластер Service Fabric Azure представляет собой ресурс, который принадлежит вам, но частично управляется корпорацией Майкрософт. В этой статье описано, как обновить версию Service Fabric в автономном кластере или кластере Azure.

Вы можете настроить для кластера автоматическое обновление Service Fabric по мере выпуска новых версий корпорацией Майкрософт или выбрать поддерживаемую версию, в которой должен работать ваш кластер.

Для этого нужно настроить параметр upgradeMode в конфигурации кластера с помощью портала или Resource Manager. Это можно сделать как при создания кластера, так и во время его работы. 

> [!NOTE]
> Обязательно следите за тем, чтобы кластер работал под управлением поддерживаемой версии Service Fabric. Когда мы объявляем о выпуске новой версии Service Fabric, для предыдущей версии определяется срок завершения жизненного цикла. Этот срок составляет по меньшей мере 60 дней. Новые выпуски объявляются [в блоге команды разработчиков Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric). после чего вы можете использовать их. 
> 
> 

За 14 дней до истечения жизненного цикла выпуска, под управлением которого работает ваш кластер, будет создано событие работоспособности, которое переводит кластер в состояние предупреждения. Состояние предупреждения изменится, когда кластер будет обновлен до поддерживаемой версии.

## <a name="set-the-upgrade-mode-in-the-azure-portal"></a>Настройка режима обновления на портале Azure
При создании кластера вы можете выбрать режим обновления: автоматический или вручную.

![На снимке экрана показана панель Создание кластера Service Fabric с выбранным вариантом конфигурация кластера и областью конфигурации кластера.][Create_Manualmode]

Эти варианты режима обновления можно также указать для работающего кластера, используя интерфейс управления. 

### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-portal"></a>Обновление кластера, для которого установлен режим обновления вручную, с помощью портала.
Чтобы обновить кластер до новой версии, нужно просто выбрать доступную версию в раскрывающемся списке и сохранить изменения. Обновление Service Fabric запустится автоматически. В ходе обновления соблюдаются все политики работоспособности кластера (политики, определяющие состояние работоспособности для узлов и всех приложений, выполняющихся в кластере).

Если требования политик работоспособности кластера не реализуются, выполняется откат обновления. Прокрутите этот документ вниз, чтобы узнать о том, как настроить пользовательские политики работоспособности. 

Устранив проблемы, которые привели к откату, запустите обновление снова, выполнив те же действия.

![На снимке экрана показано окно Service Fabric кластеры с открытой панелью обновления структуры, а также выделены параметры обновления, включая автоматические и ручные.][Manage_Automaticmode]

## <a name="set-the-upgrade-mode-using-a-resource-manager-template"></a>Настройка режима обновления с помощью шаблона Resource Manager
Добавьте параметр upgradeMode к определению ресурса Microsoft.ServiceFabric/clusters, а для параметра clusterCodeVersion установите значение одной из поддерживаемых версий Service Fabric, как показано ниже, а затем разверните шаблон. Для параметра upgradeMode поддерживаются значения Manual и Automatic.

![На снимке экрана показан шаблон, который является обычным текстом с отступом для отражения структуры, а clusterCodeVersion и upgradeMode выделены.][ARMUpgradeMode]

### <a name="upgrading-to-a-new-version-on-a-cluster-that-is-set-to-manual-mode-via-a-resource-manager-template"></a>Обновление кластера, для которого установлен режим обновления вручную, с помощью шаблона Resource Manager
Чтобы обновить до новой версии кластер, для которого установлен режим обновления вручную, замените значение параметра clusterCodeVersion значением поддерживаемой версии и разверните шаблон. Развертывание шаблона автоматически запускает обновление Service Fabric. В ходе обновления соблюдаются все политики работоспособности кластера (политики, определяющие состояние работоспособности для узлов и всех приложений, выполняющихся в кластере).

Если требования политик работоспособности кластера не реализуются, выполняется откат обновления.  

Устранив проблемы, которые привели к откату, запустите обновление снова, выполнив те же действия.

## <a name="set-custom-health-polices-for-upgrades"></a>Настройка пользовательских политик работоспособности для обновлений
Вы можете указать пользовательские политики работоспособности для обновления Service Fabric. Если вы настроили автоматическое обновление структуры кластера, то эти политики будут применены к [этапу 1 автоматических обновлений структуры](service-fabric-cluster-upgrade.md#fabric-upgrade-behavior-during-automatic-upgrades).
Если для кластера указано обновление вручную, эти политики применяются при каждом выборе новой версии, когда запускается обновление Service Fabric в кластере. Если политики не переопределить, будут использоваться значения по умолчанию.

Определить пользовательские политики работоспособности или просмотреть текущие параметры можно в колонке "Обновление Service Fabric", выбрав расширенные параметры обновления. Как это сделать, показано ниже. 

![Управление пользовательскими политиками работоспособности][HealthPolices]

## <a name="list-all-available-versions-for-all-environments-for-a-given-subscription"></a>Получение списка всех доступных версий для всех сред в указанной подписке
Запустите следующую команду, и вы увидите результат, аналогичный приведенному ниже.

Параметр supportExpiryUtc сообщает, когда истекает или истек срок действия текущей версии. Для последнего выпуска нет допустимой даты, то есть параметр имеет значение 9999-12-31T23:59:59.9999999. Это означает, что дата окончания срока действия еще не определена.

```REST
GET https://<endpoint>/subscriptions/{{subscriptionId}}/providers/Microsoft.ServiceFabric/locations/{{location}}/clusterVersions?api-version=2016-09-01

Example: https://management.azure.com/subscriptions/1857f442-3bce-4b96-ad95-627f76437a67/providers/Microsoft.ServiceFabric/locations/eastus/clusterVersions?api-version=2016-09-01

Output:
{
                  "value": [
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/5.0.1427.9490",
                      "name": "5.0.1427.9490",
                      "type": "Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "5.0.1427.9490",
                        "supportExpiryUtc": "2016-11-26T23:59:59.9999999",
                        "environment": "Windows"
                      }
                    },
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.0.1427.9490",
                      "name": "5.1.1427.9490",
                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "5.1.1427.9490",
                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
                        "environment": "Windows"
                      }
                    },
                    {
                      "id": "subscriptions/35349203-a0b3-405e-8a23-9f1450984307/providers/Microsoft.ServiceFabric/environments/Windows/clusterVersions/4.4.1427.9490",
                      "name": "4.4.1427.9490",
                      "type": " Microsoft.ServiceFabric/environments/clusterVersions",
                      "properties": {
                        "codeVersion": "4.4.1427.9490",
                        "supportExpiryUtc": "9999-12-31T23:59:59.9999999",
                        "environment": "Linux"
                      }
                    }
                  ]
                }
```

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте, как настроить некоторые [Параметры структуры кластера Service Fabric](service-fabric-cluster-fabric-settings.md) .
* Узнайте, как выполнять [масштабирование кластера](service-fabric-cluster-scale-in-out.md)
* Дополнительные сведения об [обновлении приложений](service-fabric-application-upgrade.md)

<!--Image references-->
[CertificateUpgrade]: ./media/service-fabric-cluster-upgrade/CertificateUpgrade2.png
[AddingProbes]: ./media/service-fabric-cluster-upgrade/addingProbes2.PNG
[AddingLBRules]: ./media/service-fabric-cluster-upgrade/addingLBRules.png
[HealthPolices]: ./media/service-fabric-cluster-upgrade/Manage_AutomodeWadvSettings.PNG
[ARMUpgradeMode]: ./media/service-fabric-cluster-upgrade/ARMUpgradeMode.PNG
[Create_Manualmode]: ./media/service-fabric-cluster-upgrade/Create_Manualmode.PNG
[Manage_Automaticmode]: ./media/service-fabric-cluster-upgrade/Manage_Automaticmode.PNG

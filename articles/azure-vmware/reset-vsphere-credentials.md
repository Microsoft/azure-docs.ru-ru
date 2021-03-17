---
title: Сброс учетных данных vSphere для решения VMware для Azure
description: Узнайте, как сбросить учетные данные vSphere для частного облака решения Azure VMware и убедиться, что у соединителя ХККС есть последние учетные данные vSphere.
ms.topic: how-to
ms.date: 03/16/2021
ms.openlocfilehash: e5a15caa98a46b0ae75b68ee7b568dabdbf1956c
ms.sourcegitcommit: 27cd3e515fee7821807c03e64ce8ac2dd2dd82d2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103603272"
---
# <a name="reset-vsphere-credentials-for-azure-vmware-solution"></a>Сброс учетных данных vSphere для решения VMware для Azure

В этой статье мы рассмотрим шаги сброса учетных данных vSphere для частного облака решения Azure VMware. Это позволит убедиться, что у соединителя ХККС есть последние учетные данные vSphere.

## <a name="reset-your-vsphere-credentials"></a>Сброс учетных данных vSphere

 Сначала выполним Сброс учетных данных vSphere. Срок действия учетных данных администратора vCenter CloudAdmin и НСКС-T не истекает; Однако можно выполнить следующие действия, чтобы создать новые пароли для этих учетных записей.

> [!NOTE]
> Если вы используете учетные данные CloudAdmin для подключенных служб, таких как ХККС, vCenter Orchestrator, vCloud Director или Вреализе, подключения будут перестанут работать после обновления пароля.  Эти службы должны быть остановлены перед инициацией смены пароля.  Если этого не сделать, это может привести к временным блокировкам учетных записей администратора vCenter CloudAdmin и НСКС-T, так как эти службы будут постоянно вызываться с использованием старых учетных данных.  Дополнительные сведения о настройке отдельных учетных записей для подключенных служб см. в разделе [Основные понятия доступа и удостоверений](https://docs.microsoft.com/azure/azure-vmware/concepts-identity).

1. На портале решений Azure VMware откройте командную строку.

2. Выполните следующую команду, чтобы обновить пароль vCenter CloudAdmin.  Вам потребуется заменить {SubscriptionID}, {ResourceGroup} и {Приватеклауднаме} фактическими значениями частного облака, к которому принадлежит учетная запись CloudAdmin.

```
az resource invoke-action --action rotateVcenterPassword --ids "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.AVS/privateClouds/{PrivateCloudName}" --api-version "2020-07-17-preview"
```
          
3. Выполните следующую команду, чтобы обновить пароль администратора НСКС-T. Вам потребуется заменить {SubscriptionID}, {ResourceGroup} и {Приватеклауднаме} фактическими значениями частного облака, к которому принадлежит учетная запись администратора НСКС-T.

```
az resource invoke-action --action rotateNSXTPassword --ids "/subscriptions/{SubscriptionID}/resourceGroups/{ResourceGroup}/providers/Microsoft.AVS/privateClouds/{PrivateCloudName}" --api-version "2020-07-17-preview"
```

## <a name="ensure-the-hcx-connector-has-your-latest-vsphere-credentials"></a>Убедитесь, что у соединителя ХККС есть последние учетные данные vSphere

После сброса учетных данных выполните следующие действия, чтобы убедиться, что у соединителя ХККС есть обновленные учетные данные.

1. После изменения пароля перейдите к локальному веб-интерфейсу соединителя ХККС с помощью HTTPS://{IP-адреса устройства ХККС Connector}: 443. Обязательно используйте порт 443. Войдите в систему, используя новые учетные данные.

2. На панели мониторинга ХККС VMware выберите **Связывание сайтов**.
    
    :::image type="content" source="media/reset-vsphere-credentials/hcx-site-pairing.png" alt-text="Снимок экрана панели мониторинга VMware ХККС с выделенным связыванием сайтов.":::
 
3. Выберите правильное подключение к AVS (если имеется более одного) и выберите **изменить подключение**.
 
4. Укажите новые учетные данные vSphere и выберите **изменить**, чтобы сохранить учетные данные. Сохранение должно отобразиться успешно.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы рассмотрели Сброс учетных данных vSphere для решения VMware для Azure, вы можете узнать о следующих возможностях:

- [Настройка сетевых компонентов НСКС в решении VMware для Azure](configure-nsx-network-components-azure-portal.md).
- [Управление жизненным циклом виртуальных машин Azure VMware](lifecycle-management-of-azure-vmware-solution-vms.md).
- [Развертывание аварийного восстановления виртуальных машин с помощью решения VMware для Azure](disaster-recovery-for-virtual-machines.md).

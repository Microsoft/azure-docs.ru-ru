---
title: Преимущества классической миграции развертываний в доменных службах Azure AD | Документация Майкрософт
description: Дополнительные сведения о преимуществах миграции классического развертывания Azure Active Directory доменных служб в модель развертывания диспетчер ресурсов
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: justinha
ms.openlocfilehash: 8cc5f5ebf389d35df02474d0561dc7827cde4d0b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96620092"
---
# <a name="benefits-of-migration-from-the-classic-to-resource-manager-deployment-model-in-azure-active-directory-domain-services"></a>Преимущества перехода с классической модели на модель развертывания диспетчер ресурсов в доменных службах Azure Active Directory

Azure Active Directory доменных служб (Azure AD DS) позволяет перенести существующий управляемый домен, использующий классическую модель развертывания, в модель развертывания диспетчер ресурсов. Управляемые домены Azure AD DS, использующие модель развертывания диспетчер ресурсов, предоставляют дополнительные функции, такие как детально детализированная политика паролей, журналы аудита и защита от блокировки учетных записей.

В этой статье описываются преимущества миграции. Чтобы приступить к работе, см. статью [перенос доменных служб Azure AD из классической модели виртуальной сети в Диспетчер ресурсов][howto-migrate].

> [!NOTE]
> В 2017 доменные службы Azure AD стали доступны для размещения в Azure Resource Manager сети. С тех пор мы смогли создать более безопасную службу с помощью современных возможностей Azure Resource Manager. Поскольку Azure Resource Manager развертывания полностью заменяют классические развертывания, развертывание классической виртуальной сети Azure AD DS будет прекращено 1 марта 2023 г.
>
> Дополнительные сведения см. в [официальном](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/) объявлении об устаревании.

## <a name="migration-benefits"></a>Преимущества миграции

Процесс миграции принимает существующий управляемый домен, использующий классическую модель развертывания, и переходит к использованию модели развертывания диспетчер ресурсов. При переносе управляемого домена из классической модели в модель развертывания диспетчер ресурсов можно избежать необходимости повторного присоединить компьютеры к управляемому домену или удалить управляемый домен и создать новый с нуля. Виртуальные машины по прежнему будут присоединены к управляемому домену в конце процесса миграции.

После миграции Azure AD DS предоставляет множество функций, доступных только для доменов, использующих модель развертывания диспетчер ресурсов, включая следующие:

* [Поддержка детальной политики паролей][password-policy].
* Более быстрые скорости синхронизации между Azure AD и доменными службами Azure AD.
* Два новых [атрибута, которые синхронизируются из Azure AD][attributes]  -  *Manager* и *EmployeeID*.
* Доступ к контроллерам домена с повышенным питанием при [обновлении SKU][skus].
* Защита от блокировки учетной записи Active Directory.
* [Уведомления по электронной почте об оповещениях в управляемом домене][email-alerts].
* [Используйте рабочие книги Azure и Azure Monitor для просмотра журналов аудита и действий входа][workbooks].
* В области Поддерживаемые регионы [зоны доступности Azure][availability-zones].
* Интеграция с другими продуктами Azure, такими как служба [файлов Azure][azure-files], служба " [HD Insights][hd-insights]" и [виртуальный рабочий стол Windows][wvd].
* Поддержка имеет доступ к большему объему телеметрии и может помочь в более эффективном устранении неполадок.
* Шифрование при хранении с использованием [управляемых дисков Azure][managed-disks] для данных на управляемых контроллерах домена.

Управляемые домены, использующие модель развертывания диспетчер ресурсов, помогут вам оставаться в курсе последних новых функций. Новые функции недоступны для управляемых доменов, использующих классическую модель развертывания.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к работе, см. статью [перенос доменных служб Azure AD из классической модели виртуальной сети в Диспетчер ресурсов][howto-migrate].

<!-- LINKS - INTERNAL -->
[password-policy]: password-policy.md
[skus]: change-sku.md
[email-alerts]: notifications.md
[workbooks]: use-azure-monitor-workbooks.md
[azure-files]: ../storage/files/storage-files-identity-auth-active-directory-domain-service-enable.md
[hd-insights]: ../hdinsight/domain-joined/apache-domain-joined-configure-using-azure-adds.md
[wvd]: ../virtual-desktop/overview.md
[availability-zones]: ../availability-zones/az-overview.md
[howto-migrate]: migrate-from-classic-vnet.md
[attributes]: synchronization.md#attribute-synchronization-and-mapping-to-azure-ad-ds
[managed-disks]: ../virtual-machines/managed-disks-overview.md

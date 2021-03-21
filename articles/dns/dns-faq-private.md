---
title: 'Частная зона DNS: часто задаваемые вопросы'
description: В этой статье приведены часто задаваемые вопросы об Azure Частная зона DNS
services: dns
author: rohinkoul
ms.service: dns
ms.topic: article
ms.date: 01/15/2021
ms.author: rohink
ms.openlocfilehash: 3272f4fea4169fd5e91b1ba2b97ad72dfbb49080
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98247392"
---
# <a name="azure-private-dns-faq"></a>Частная зона DNS: часто задаваемые вопросы

Ниже приведены часто задаваемые вопросы о частных DNS-службах Azure.

## <a name="does-azure-dns-support-private-domains"></a>Поддерживает ли Azure DNS частные домены?

Частные домены поддерживаются с помощью функции Azure Частная зона DNS Zones. Зоны Частная зона DNS разрешимы только в указанных виртуальных сетях. Дополнительные сведения см. в [обзоре](private-dns-overview.md).

Сведения о других внутренних параметрах DNS в Azure см. в статье [Разрешение имен ресурсов в виртуальных сетях Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md).

## <a name="will-azure-private-dns-zones-work-across-azure-regions"></a>Будут ли зоны Azure Частная зона DNS работать в разных регионах Azure?

Да. Частные зоны поддерживаются для разрешения DNS между виртуальными сетями в разных регионах Azure. Частные зоны работают даже без явной настройки пиринга виртуальных сетей. Все виртуальные сети должны быть связаны с частной зоной DNS.

## <a name="is-connectivity-to-the-internet-from-virtual-networks-required-for-private-zones"></a>Требуется ли подключение к Интернету из виртуальных сетей, необходимых для частных зон?

Нет. Частные зоны работают вместе с виртуальными сетями. Они используются для управления доменами виртуальных машин или других ресурсов в виртуальных сетях и между ними. Для разрешения имен не нужно подключение к Интернету.

## <a name="can-the-same-private-zone-be-used-for-several-virtual-networks-for-resolution"></a>Можно ли использовать одну частную зону для разрешения имен в нескольких виртуальных сетях?

Да. Вы можете связать частную зону DNS с тысячами виртуальных сетей. Дополнительные сведения см. в разделе [ограничения Azure DNS](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-dns-limits)

## <a name="can-a-virtual-network-that-belongs-to-a-different-subscription-be-linked-to-a-private-zone"></a>Можно ли связать с частной зоной виртуальную сеть, принадлежащую другой подписке?

Да. Вы должны обладать разрешением на операцию записи для виртуальных сетей и частной зоны DNS. Разрешение на запись может быть предоставлено нескольким ролям Azure. Например, роль Azure "участник классической сети" имеет разрешения на запись в виртуальные сети и роль участника "зоны Частная зона DNS" имеет разрешения на запись в частные зоны DNS. Дополнительные сведения о ролях Azure см. в статье [Управление доступом на основе ролей в Azure (Azure RBAC)](../role-based-access-control/overview.md).

## <a name="will-the-automatically-registered-virtual-machine-dns-records-in-a-private-zone-be-automatically-deleted-when-you-delete-the-virtual-machine"></a>Будут ли автоматически зарегистрированные записи DNS виртуальных машин в частной зоне автоматически удалены при удалении виртуальной машины?

Да. При удалении виртуальной машины в связанной виртуальной сети с включенной автоматической регистрацией зарегистрированные записи автоматически удаляются.

## <a name="ive-reconfigured-the-os-in-my-virtual-machine-to-have-a-new-host-name-or-static-ip-address-why-dont-i-see-that-change-reflected-in-the-private-zone"></a>Я перестроил ОС в моей виртуальной машине для нового имени узла или статического IP-адреса. Почему я не вижу, что это изменение отражено в частной зоне?

Записи частной зоны заполняются службой DHCP Azure. сообщения регистрации клиента игнорируются. Если вы отключили поддержку DHCP-клиентов на виртуальной машине, настроив статический IP-адрес, изменения имени узла или статического IP-адреса в виртуальной машине не отражаются в зоне.

## <a name="i-have-configured-a-preferred-dns-suffix-in-my-windows-virtual-machine-why-are-my-records-still-registered-in-the-zone-linked-to-the-virtual-network"></a>В виртуальной машине Windows настроен предпочитаемый DNS-суффикс. Почему мои записи все еще зарегистрированы в зоне, связанной с виртуальной сетью?

Служба DHCP Azure при регистрации частной зоны DNS игнорирует все DNS-суффиксы. Например, если виртуальная машина настроена в `contoso.com` качестве основного DNS-суффикса, но виртуальная сеть связана с `fabrikam.com` частной зоной DNS, регистрация виртуальной машины отображается в `fabrikam.com` частной зоне DNS.

## <a name="can-an-automatically-registered-virtual-machine-record-in-a-private-zone-from-a-linked-virtual-network-be-deleted-manually"></a>Можно ли удалить автоматически зарегистрированную запись виртуальной машины в частной зоне из связанной виртуальной сети вручную?

Да. Однако вы можете перезаписать их записями DNS, созданными в зоне вручную. Чтобы узнать об этом больше, ознакомьтесь со следующими вопросом и ответом.

## <a name="what-happens-when-i-try-to-manually-create-a-new-dns-record-into-a-private-zone-that-has-the-same-hostname-as-an-automatically-registered-existing-virtual-machine-in-a-linked-virtual-network"></a>Что происходит при попытке вручную создать новую запись DNS в частной зоне с тем же именем узла, что и автоматически зарегистрированная Существующая виртуальная машина в связанной виртуальной сети?

Вы пытаетесь вручную создать новую запись DNS в частной зоне с тем же именем узла, что и у существующей, автоматически зарегистрированной виртуальной машины в связанной виртуальной сети. При этом новая запись DNS перезаписывает автоматически зарегистрированную виртуальную машину. Повторное удаление этой созданной записи DNS из зоны происходит успешно. Автоматическая регистрация выполняется еще раз, пока существует виртуальная машина и к ней присоединен частный IP-адрес. Запись DNS автоматически не будет создана повторно в зоне.

## <a name="what-happens-when-we-unlink-a-linked-virtual-network-from-a-private-zone-will-the-automatically-registered-virtual-machine-records-from-the-virtual-network-be-removed-from-the-zone-too"></a>Что происходит при разрыве связи между связанной виртуальной сетью и частной зоной? Будут ли также удалены из зоны автоматически зарегистрированные записи виртуальных машин из виртуальной сети?

Да. Чтобы удалить связь связанной виртуальной сети из частной зоны, обновите зону DNS, удалив связанную с ней связь виртуальной сети. В этом процессе записи виртуальных машин, которые были автоматически зарегистрированы, удаляются из зоны.

## <a name="what-happens-when-we-delete-a-linked-virtual-network-thats-linked-to-a-private-zone-do-we-have-to-manually-update-the-private-zone-to-unlink-the-virtual-network-as-a-linked-virtual-network-from-the-zone"></a>Что происходит при удалении связанной виртуальной сети, связанной с частной зоной? Нужно ли вручную обновить частную зону, чтобы удалить связь между виртуальной сетью и связанной виртуальной сетью из зоны?

Нет. Если удалить связанную виртуальную сеть без удаления связи с частной зоной, операция удаления будет выполнена, а ссылки на зону DNS будут автоматически удалены.

## <a name="will-dns-resolution-by-using-the-default-fqdn-internalcloudappnet-still-work-even-when-a-private-zone-for-example-privatecontosocom-is-linked-to-a-virtual-network"></a>Будет ли решение DNS использовать полное доменное имя по умолчанию (internal.cloudapp.net) по-прежнему работать, даже если частная зона (например, private.contoso.com) связана с виртуальной сетью?

Да. Частные зоны не заменяют стандартную зону internal.cloudapp.net, предоставляемую Azure по умолчанию. При использовании зоны internal.cloudapp.net, предоставляемой Azure, или собственной частной зоны используйте полное доменное имя зоны, выбранной для разрешения имен.

## <a name="will-the-dns-suffix-on-virtual-machines-within-a-linked-virtual-network-be-changed-to-that-of-the-private-zone"></a>Изменится ли DNS-суффикс виртуальных машин в связанной виртуальной сети с учетом этой частной зоны?

Нет. В настоящее время DNS-суффикс виртуальных машин в связанной виртуальной сети сохраняет формат суффикса Azure по умолчанию ("*.internal.cloudapp.net"). Однако можно вручную изменить этот DNS-суффикс на виртуальных машинах, указав частную зону.
Инструкции по изменению этого суффикса см. в статье [Использование динамического DNS для регистрации имен узлов на собственном DNS-сервере](../virtual-network/virtual-networks-name-resolution-ddns.md#windows-clients) .

## <a name="what-are-the-usage-limits-for-azure-dns-private-zones"></a>Каковы ограничения использования для Azure DNS частных зон?

Дополнительные сведения об ограничениях использования для Azure DNS частных зон см. в разделе [ограничения Azure DNS](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-dns-limits) .

## <a name="why-dont-my-existing-private-dns-zones-show-up-in-new-portal-experience"></a>Почему мои существующие частные зоны DNS не отображаются в новом интерфейсе портала?

Если существующая частная зона DNS была создана с помощью предварительной версии API, необходимо перенести эти зоны в новую модель ресурсов. Частная зона DNS зоны, созданные с помощью API предварительной версии, не будут отображаться в новом интерфейсе портала. Инструкции по переходу на новую модель ресурсов см. ниже.

## <a name="how-do-i-migrate-my-existing-private-dns-zones-to-the-new-model"></a>Разделы справки перенести существующие частные зоны DNS в новую модель?

Настоятельно рекомендуется как можно скорее перейти на новую модель ресурсов. Поддержка устаревшей модели ресурсов будет поддерживаться, однако дополнительные функции не будут разрабатываться поверх этой модели. В будущем мы планируем использовать ее в пользу новой модели ресурсов. Инструкции по переносу существующих частных зон DNS в новую модель ресурсов см. в[руководстве по миграции для Azure DNS частных зон](private-dns-migration-guide.md).

### <a name="does-azure-dns-private-zones-store-any-customer-content"></a>Хранит ли содержимое клиента в частных зонах Azure DNS?

Нет, Azure DNS частные зоны не хранят содержимое клиента.

## <a name="next-steps"></a>Дальнейшие действия

- [Дополнительные сведения об Azure Частная зона DNS](private-dns-overview.md)

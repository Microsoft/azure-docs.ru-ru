---
author: dlepow
ms.service: api-management
ms.topic: include
ms.date: 01/26/2021
ms.author: danlep
ms.openlocfilehash: a9dbedd8516f3a3a592c7fd4f4f5563011d6c6db
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99491023"
---
#### <a name="requirements-for-key-vault-firewall"></a>Требования к Key Vault брандмауэру

Если [Key Vault брандмауэр](../articles/key-vault/general/network-security.md) включен в хранилище ключей, ниже приведены дополнительные требования.

* Для доступа к хранилищу ключей необходимо использовать управляемое **системное** удостоверение экземпляра службы управления API.
* В Key Vault брандмауэре установите флажок **разрешить доверенным службам Майкрософт обходить этот брандмауэр** .

#### <a name="virtual-network-requirements"></a>Требования к виртуальной сети

Если экземпляр управления API развернут в виртуальной сети, также настройте следующие параметры сети.

* Включите Azure Key Vault [конечной точки службы](../articles/key-vault/general/overview-vnet-service-endpoints.md) в подсети управления API.
* Настройте правило группы безопасности сети (NSG), разрешающее исходящий трафик для [тегов службы](../articles/virtual-network/service-tags-overview.md)AzureKeyVault и AzureActiveDirectory. 

Дополнительные сведения см. в разделе сведения о конфигурации сети в статье [Подключение к виртуальной сети](../articles/api-management/api-management-using-with-vnet.md#-common-network-configuration-issues).
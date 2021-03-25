---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/24/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 0eb83417350d5dca85802269b698aee30a310f6f
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105104380"
---
|Имя<br /><sub>(портал Azure)</sub> |Описание |Действие |Версия<br /><sub>(GitHub)</sub> |
|---|---|---|---|
|[Управляемые экземпляры SQL должны использовать ключи, управляемые клиентом, для шифрования неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F048248b0-55cd-46da-b1ff-39efd52db260) |Реализация прозрачного шифрования данных (TDE) с вашим собственным ключом обеспечивает повышенную прозрачность и контроль над предохранителем TDE, а также дополнительную защиту за счет использования внешней службы с поддержкой HSM и содействие разделению обязанностей. Эта рекомендация относится к организациям, имеющим связанное требование по соответствию. |AuditIfNotExists, Disabled |[1.0.2](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlManagedInstance_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |
|[Серверы SQL должны использовать управляемые клиентом ключи для шифрования неактивных данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0d134df8-db83-46fb-ad72-fe0c9428c8dd) |Реализация прозрачного шифрования данных (TDE) с вашим ключом обеспечивает повышенную прозрачность и контроль над предохранителем TDE, а также дополнительную защиту за счет использования внешней службы с поддержкой HSM и содействие разделению обязанностей. Эта рекомендация относится к организациям, имеющим связанное требование по соответствию. |AuditIfNotExists, Disabled |[2.0.1](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlServer_EnsureServerTDEisEncryptedWithYourOwnKey_Audit.json) |
|[В базах данных SQL должно применяться прозрачное шифрование данных](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F17k78e20-9358-41c9-923c-fb736d382a12) |Для защиты неактивных данных и обеспечения соответствия требованиям должно быть включено прозрачное шифрование данных. |AuditIfNotExists, Disabled |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/SQL/SqlDBEncryption_Audit.json) |

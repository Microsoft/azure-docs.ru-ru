---
title: Управление данными пользователя в центре безопасности Azure | Документация Майкрософт
description: Узнайте, как управлять пользовательскими данными в центре безопасности Azure. Управление данными пользователя включает в себя возможность доступа, удаления и экспорта данных.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 411d7bae-c9d4-4e83-be63-9f2f2312b075
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/23/2018
ms.author: memildin
ms.openlocfilehash: 5b5c78ffec736f29a481aa95426ff663199613b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100595648"
---
# <a name="manage-user-data-in-azure-security-center"></a>Управление данными пользователя в центре безопасности Azure
В этой статье приводятся сведения об управлении данными пользователя в центре безопасности Azure. Управление данными пользователя включает в себя возможность доступа, удаления и экспорта данных.

[!INCLUDE [gdpr-intro-sentence.md](../../includes/gdpr-intro-sentence.md)]

Пользователь центра безопасности Azure, которому назначена роль читателя, владельца, участника или администратора учетной записи, может получить доступ к данным клиента в рамках этого центра. Дополнительные сведения о роли администратора учетной записи см. в статьях [встроенные роли для управления доступом на основе ролей в Azure](../role-based-access-control/built-in-roles.md) , чтобы узнать больше о ролях читателей, владельца и участника. См. раздел [Администраторы подписки Azure](../cost-management-billing/manage/add-change-subscription-administrator.md).

## <a name="searching-for-and-identifying-personal-data"></a>Поиск и идентификация персональных данных
Пользователи Центра безопасности Azure могут просматривать свои персональные данные через портал Azure. В Центре безопасности Azure хранятся только сведения о контактных лицах по вопросам безопасности, такие как адреса электронной почты и номера телефонов. Дополнительные сведения см. [в статье указание контактных сведений по безопасности в центре безопасности Azure](security-center-provide-security-contact-details.md).

В портал Azure пользователь может просматривать разрешенные IP-конфигурации с помощью функции JIT-доступа к виртуальной машине центра безопасности. Дополнительные сведения см. [в статье Управление доступом к виртуальным машинам с помощью JIT](security-center-just-in-time.md).

На портале Azure пользователь может просмотреть оповещения безопасности, предоставляемые Центром безопасности Azure, включая сведения об IP-адресах и злоумышленниках. Дополнительные сведения см. [в статье Управление оповещениями безопасности в центре безопасности Azure и реагирование на них](security-center-managing-and-responding-alerts.md).

## <a name="classifying-personal-data"></a>Классификация персональных данных
Вам не нужно классифицировать персональные данные, найденные в средстве связи с безопасностью центра безопасности. Это адрес электронной почты (или несколько адресов) и номер телефона. [Контактные данные](security-center-provide-security-contact-details.md) проверяются Центром безопасности Azure.

Вам не нужно классифицировать IP-адреса и номера портов, сохраненные с помощью функции [JIT](security-center-just-in-time.md) центра безопасности.

Только пользователь, которому назначена роль администратора, может классифицировать персональные данные, перейдя в окно [просмотра оповещений](security-center-managing-and-responding-alerts.md) в Центре безопасности Azure.

## <a name="securing-and-controlling-access-to-personal-data"></a>Защита доступа к персональным данным и управлением им
Пользователь Центра безопасности Azure, которому назначена роль читателя, владельца, участника или администратора учетной записи, может получить доступ к [данным контактного лица по вопросам безопасности](security-center-provide-security-contact-details.md).

Пользователь центра безопасности, которому назначена роль читателя, владельца, участника или администратора учетной записи, может получить доступ к своим [JIT](security-center-just-in-time.md) -политикам.

Пользователи Центра безопасности Azure, которым назначены роли читателя, владельца, участника или администратора учетной записи, могут просматривать [оповещения](security-center-managing-and-responding-alerts.md).

## <a name="updating-personal-data"></a>Обновление персональных данных
Пользователь Центра безопасности Azure, которому назначена роль читателя, владельца, участника или администратора учетной записи, может обновить [данные контактного лица по вопросам безопасности](security-center-provide-security-contact-details.md) на портале Azure.

Пользователь центра безопасности, которому назначена роль владельца, участника или администратора учетной записи, может обновить свои [JIT-политики](security-center-just-in-time.md).

Администратор учетной записи не может изменять инциденты предупреждений. [Инцидент с оповещением](security-center-managing-and-responding-alerts.md) относится к данным безопасности и предназначен только для чтения.

## <a name="deleting-personal-data"></a>Удаление персональных данных
Пользователь Центра безопасности Azure, которому назначена роль читателя, владельца, участника или администратора учетной записи, может удалить [данные контактного лица по вопросам безопасности](security-center-provide-security-contact-details.md) через портал Azure.

Пользователь центра безопасности, которому назначена роль владельца, участника или администратора учетной записи, может удалить [JIT-политики](security-center-just-in-time.md) с помощью портал Azure.

Пользователь центра безопасности не может удалять инциденты предупреждений. По соображениям безопасности [инцидент предупреждения](security-center-managing-and-responding-alerts.md) рассматривается как данные только для чтения.

## <a name="exporting-personal-data"></a>Экспорт персональных данных
Пользователь Центра безопасности Azure, которому назначена роль читателя, владельца, участника или администратора учетной записи, может экспортировать [данные контактного лица по вопросам безопасности](security-center-provide-security-contact-details.md) следующим образом:

- Копирование из портал Azure
- вызвав Azure REST API и используя метод GET HTTP:
  ```HTTP
  GET https://<endpoint>/subscriptions/{subscriptionId}/providers/Microsoft.Security/securityContacts?api-version={api-version}
  ```

Пользователь центра безопасности, которому назначена роль администратора учетной записи, может экспортировать [политики JIT](security-center-just-in-time.md) , содержащие IP-адреса, следующим образом.

- Копирование из портал Azure
- вызвав Azure REST API и используя метод GET HTTP:
  ```HTTP
  GET https://<endpoint>/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Security/locations/{location}/jitNetworkAccessPolicies/default?api-version={api-version}
  ```

Администратор учетной записи может экспортировать сведения об оповещении следующим образом:

- Копирование из портал Azure
- вызвав Azure REST API и используя метод GET HTTP:
  ```HTTP
  GET https://<endpoint>/subscriptions/{subscriptionId}/providers/microsoft.Security/alerts?api-version={api-version}
  ```

Дополнительные сведения см. в разделе [Получение оповещений системы безопасности (получение коллекции)](/previous-versions/azure/reference/mt704050(v=azure.100)).

## <a name="restricting-the-use-of-personal-data-for-profiling-or-marketing-without-consent"></a>Ограничение использования персональных данных для профилирования или маркетинга без согласия
Пользователь Центра безопасности Azure может отказаться, удалив [данные контактного лица по вопросам безопасности](security-center-provide-security-contact-details.md).

[Своевременные данные](security-center-just-in-time.md) считаются неидентифицируемыми и соотносятся за период в 30 дней.

[Данные об оповещении](security-center-managing-and-responding-alerts.md) считаются данными безопасности и хранятся в течение двух лет.

## <a name="auditing-and-reporting"></a>Аудит и создание отчетов
В [журналах действий Azure](../azure-monitor/essentials/platform-logs-overview.md)хранятся журналы аудита контактного лица по вопросам безопасности, своевременного и обновления оповещений.
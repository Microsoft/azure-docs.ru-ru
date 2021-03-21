---
title: Настройка ключей, управляемых клиентом, в Azure Sentinel | Документация Майкрософт
description: Узнайте, как настроить ключи, управляемые клиентом (CMK), в Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/12/2020
ms.author: yelevin
ms.openlocfilehash: a1c2754fcae5768c6b87d6280fc882acd46d9a0a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100585332"
---
# <a name="set-up-azure-sentinel-customer-managed-key"></a>Настройка ключа, управляемого клиентом, в Azure Sentinel

В этой статье содержатся общие сведения и инструкции по настройке ключа, управляемого клиентом (CMK), для Azure Sentinel. С помощью ключа CMK можно шифровать все данные, сохраненные или отправленные в Azure Sentinel, во всех соответствующих ресурсах хранилища с помощью ключа Azure Key Vault, созданного вами или принадлежащего вам.

> [!NOTE]
> - Функция Azure Sentinel CMK предоставляется только **новым клиентам**.
>
> - Доступ к этой возможности контролируется с помощью регистрации компонентов Azure. Вы можете запросить доступ, обратившись по адресу azuresentinelCMK@microsoft.com . Ожидающие запросы будут утверждены в соответствии с доступной емкостью.
>
> - Возможность CMK доступна только для клиентов, отправляющих 1 ТБ данных или более в день. Вы получите сведения о дополнительных ценах, когда отправите в корпорацию Майкрософт запрос на подготовку CMK для своей подписки Azure. Узнайте больше о [ценах на Log Analytics](../azure-monitor/logs/manage-cost-storage.md#log-analytics-dedicated-clusters).

## <a name="how-cmk-works"></a>Как работает CMK 

Решение Sentinel Azure использует несколько ресурсов хранилища для сбора журналов и предоставления функций, включая Log Analytics и пр. При настройке CMK Azure Sentinel потребуется также настроить параметры CMK для связанных ресурсов хранилища. Данные, сохраненные в ресурсах хранилища, отличных от Log Analytics, также будут шифроваться.

Узнайте больше о [CMK](../azure-monitor/logs/customer-managed-keys.md#customer-managed-key-overview).

> [!NOTE]
> Если включить CMK в Azure Sentinel, то любая общедоступная предварительная версия функции, которая не поддерживает CMK, не будет включена.

## <a name="enable-cmk"></a>Включение CMK 

Чтобы подготовить CMK, выполните следующие действия. 

1.  Создайте Azure Key Vault и ключ хранилища.

2.  Включите CMK для рабочей области Log Analytics.

3.  Зарегистрируйтесь для использования Cosmos DB.

4.  Добавьте политику доступа в свой экземпляр Azure Key Vault.

5.  Включите CMK в Azure Sentinel.

6.  Включите Azure Sentinel.

### <a name="step-1-create-an-azure-key-vault-and-storing-key"></a>Шаг 1. Создание Azure Key Vault и ключа хранилища

1.  [Создайте ресурс Azure Key Vault](/azure-stack/user/azure-stack-key-vault-manage-portal), затем создайте или импортируйте ключ, который будет использоваться для шифрования данных.
    > [!NOTE]
    >  Для Azure Key Vault нужно настроить возможность восстановления, чтобы обезопасить ключи и доступ.

1.  [Включите параметры восстановления.](../key-vault/general/key-vault-recovery.md)

    -   Убедитесь, что включено [обратимое удаление](../key-vault/general/soft-delete-overview.md).

    -   Включите [защиту от очистки](../key-vault/general/soft-delete-overview.md#purge-protection), чтобы защитить секрет и данные хранилища от принудительного удаления даже после обратимого удаления.

### <a name="step-2-enable-cmk-on-your-log-analytics-workspace"></a>Шаг 2. Включение CMK для рабочей области Log Analytics

Чтобы создать рабочую область CMK, которая будет использоваться в качестве рабочей области Azure Sentinel, следуйте инструкциям по [настройке ключей, управляемых клиентом, в Azure Monitor](../azure-monitor/logs/customer-managed-keys.md).

### <a name="step-3-register-for-cosmos-db"></a>Шаг 3. Регистрация для использования Cosmos DB

Azure Sentinel использует Cosmos DB в качестве дополнительного ресурса хранилища. Обязательно выполните регистрацию в Cosmos DB.

Следуйте инструкциям для Cosmos DB, чтобы [зарегистрировать поставщик ресурсов Azure Cosmos DB](../cosmos-db/how-to-setup-cmk.md#register-resource-provider) для своей подписки Azure

### <a name="step-4-add-an-access-policy-to-your-azure-key-vault-instance"></a>Шаг 4. Добавление политики доступа к экземпляру Azure Key Vault

Не забудьте настроить доступ из Cosmos DB к экземпляру Azure Key Vault. Следуйте инструкциям для Cosmos DB, чтобы [добавить политику доступа к экземпляру Azure Key Vault](../cosmos-db/how-to-setup-cmk.md#add-an-access-policy-to-your-azure-key-vault-instance) с помощью субъекта Azure Cosmos DB.

### <a name="step-5-enable-cmk-in-azure-sentinel"></a>Шаг 5. Включение CMK в Azure Sentinel

Возможность CMK Azure Sentinel предоставляется только новым клиентам, получившим доступ непосредственно от группы разработчиков продуктов Azure. С помощью контактов обратитесь в корпорацию Майкрософт, чтобы группа разработчиков Azure Sentinel утвердила ваш запрос, после чего включите CMK в своем решении.

После получения утверждения вам будет предложено предоставить приведенные ниже сведения, чтобы включить функцию CMK.

-  Идентификатор рабочей области, в которой необходимо включить CMK.

-  URL-адрес Key Vault: Скопируйте "идентификатор ключа" ключа до последней косой черты:  
    

    ![Идентификатор ключа](./media/customer-managed-keys/key-identifier.png)

    Группа Azure Sentinel включит возможность CMK Sentinel для вашей рабочей области.

-  Полученное от группы разработчиков продуктов Sentinel Azure подтверждение того, что вы можете использовать эту возможность. Это подтверждение необходимо получить, прежде чем продолжить.

### <a name="step-6-enable-azure-sentinel"></a>Шаг 6. Включение Azure Sentinel


Перейдите на портал Azure и включите Azure Sentinel для рабочей области, в которой вы настроили CMK. Дополнительные сведения см. в статье [Краткое руководство. Подключение к Azure Sentinel](quickstart-onboard.md).

## <a name="key-encryption-key-revocation-or-deletion"></a>Отзыв или удаление ключа шифрования ключа


В случае, если пользователь отзывает ключ шифрования ключа, удаляя его или отменяя доступ Sentinel Azure, в течение одного часа Azure Sentinel учтет эти изменения и перестанет обращаться к данным, считая их недоступными. На этом этапе все выполняемые операции, которые используют ресурсы постоянного хранилища, такие как прием данных, изменение постоянной конфигурации и создание инцидентов, будут предотвращаться. Ранее сохраненные данные не будут удалены, но останутся недоступными. Недоступные данные управляются политикой хранения данных и удаляются в соответствии с ней.

Единственная операция, которая возможна после отзыва или удаления ключа шифрования, — это удаление учетной записи.

Если после отзыва доступ будет восстановлен, то Azure Sentinel восстановит доступ к данным в течение часа.

Чтобы лучше понять, как это работает в Azure Monitor, ознакомьтесь с разделом [Отзыв CMK (KEK)](../azure-monitor/logs/customer-managed-keys.md#key-revocation).

## <a name="key-encryption-key-rotation"></a>Смена ключей шифрования ключа


Azure Sentinel и Log Analytics поддерживают смену ключей. Когда пользователь выполняет смену ключа в Key Vault, Azure Sentinel получает новый ключ в течение часа.

Смена ключа выполняется путем создания его новой версии в Key Vault.

![Смена ключей](./media/customer-managed-keys/key-rotation.png)

Предыдущую версию ключа можно отключить через 24 часа или после того, как в журналах аудита Azure Key Vault перестанут отображаться действия, использующие предыдущую версию.

Если вы используете один и тот же ключ в Azure Sentinel и Log Analytics, то после смены ключа необходимо явным образом обновить ресурс кластера в Log Analytics с помощью новой версии ключа Azure Key Vault. Дополнительные сведения см. в разделе о [смене ключей CMK в Azure Monitor](../azure-monitor/logs/customer-managed-keys.md#key-rotation).

## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как настроить ключ, управляемый клиентом, в Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
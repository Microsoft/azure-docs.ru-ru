---
title: Ведение журнала Azure Key Vault | Документация Майкрософт
description: Сведения о том, как отслеживать доступ к хранилищам ключей, включив функцию ведения журнала Azure Key Vault, которая сохраняет информацию в указанной учетной записи хранения Azure.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 12/18/2020
ms.author: mbaldwin
ms.openlocfilehash: 99313cf1248ef5e90d7cd60d528a58c7925298d0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102499532"
---
# <a name="azure-key-vault-logging"></a>Ведение журнала Azure Key Vault

Создав одно или несколько хранилищ ключей вы, вероятно, захотите отслеживать, кто, как и когда осуществлял доступ к этим хранилищам. Это можно сделать с помощью функции ведения журнала Azure Key Vault, которая сохраняет информацию в указанной учетной записи хранения Azure. Пошаговые инструкции по настройке этой функции см. в статье [Включение ведения журнала Key Vault](howto-logging.md).

Регистрируемые в журналах сведения становятся доступны не позднее чем через 10 минут после выполнения операции с хранилищем ключей. В большинстве случаев это будет еще быстрее.  Способ управления журналами в своей учетной записи хранения вы выбираете сами.

* В учетной записи хранения используйте стандартные методы контроля доступа, предоставляемые Azure, для защиты журналов путем ограничения доступа к ним.
* Удаляйте журналы, которые больше не нужно хранить в учетной записи хранения.

Общие сведения о Key Vault см. в статье [Что такое хранилище ключей Azure?](overview.md) Сведения о регионах доступности Key Vault можно узнать на [странице цен](https://azure.microsoft.com/pricing/details/key-vault/). Сведения об использовании Azure Monitor для Key Vault см. [здесь](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="interpret-your-key-vault-logs"></a>Интерпретация журналов хранилища ключей

При включении ведения журнала для указанной вами учетной записи хранения автоматически создается контейнер с именем **insights-logs-auditevent**. Эту же учетную запись можно использовать для сбора журналов нескольких хранилищ ключей.

Отдельные BLOB-объекты хранятся как текст в формате JSON. Давайте рассмотрим пример записи журнала. 

```json
    {
        "records":
        [
            {
                "time": "2016-01-05T01:32:01.2691226Z",
                "resourceId": "/SUBSCRIPTIONS/361DA5D4-A47A-4C79-AFDD-XXXXXXXXXXXX/RESOURCEGROUPS/CONTOSOGROUP/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/CONTOSOKEYVAULT",
                "operationName": "VaultGet",
                "operationVersion": "2015-06-01",
                "category": "AuditEvent",
                "resultType": "Success",
                "resultSignature": "OK",
                "resultDescription": "",
                "durationMs": "78",
                "callerIpAddress": "104.40.82.76",
                "correlationId": "",
                "identity": {"claim":{"http://schemas.microsoft.com/identity/claims/objectidentifier":"d9da5048-2737-4770-bd64-XXXXXXXXXXXX","http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn":"live.com#username@outlook.com","appid":"1950a258-227b-4e31-a9cf-XXXXXXXXXXXX"}},
                "properties": {"clientInfo":"azure-resource-manager/2.0","requestUri":"https://control-prod-wus.vaultcore.azure.net/subscriptions/361da5d4-a47a-4c79-afdd-XXXXXXXXXXXX/resourcegroups/contosoresourcegroup/providers/Microsoft.KeyVault/vaults/contosokeyvault?api-version=2015-06-01","id":"https://contosokeyvault.vault.azure.net/","httpStatusCode":200}
            }
        ]
    }
```

В следующей таблице перечислены имена и описания полей.

| Имя поля | Описание |
| --- | --- |
| **time** |Дата и время (в формате UTC). |
| **resourceId** |Идентификатор ресурса Azure Resource Manager. Для журналов хранилища ключей это всегда идентификатор ресурса хранилища ключей. |
| **operationName** |Имя операции, как описано в следующей таблице. |
| **operationVersion** |Запрошенная клиентом версия REST API. |
| **category** |Тип результата. Для журналов Key Vault единственным доступным значением является `AuditEvent`. |
| **resultType** |Результат запроса к REST API. |
| **resultSignature** |Состояние HTTP. |
| **resultDescription** |Дополнительное описание результата, если доступно. |
| **durationMs** |Время обслуживания запроса REST API в миллисекундах. Сюда не входит задержка сети, поэтому время, зарегистрированное на стороне клиента, может не соответствовать этому значению. |
| **callerIpAddress** |IP-адрес клиента, выполнившего запрос. |
| **correlationId** |Необязательный GUID, который клиент может передавать для сопоставления журналов на стороне клиента с журналами на стороне службы (хранилища ключей). |
| **identity** |Удостоверение из маркера, предоставляемое в запросе к REST API. Обычно это "пользователь", "субъект-служба" или комбинация "пользователь + идентификатор приложения", как например при запросе из командлета Azure PowerShell. |
| **properties** |Эта информация зависит от типа операции (**operationName**). В большинстве случаев это поле содержит сведения о клиенте (передаваемая клиентом строка useragent), точный URI запроса REST API и код состояния HTTP. Кроме того, когда результат запроса содержит объект (например, **KeyCreate** или **VaultGet**), это поле содержит еще и URI ключа (в значении `id`), URI хранилища или URI секрета. |

Значения поля **operationName** отображаются в формате *ObjectVerb*. Пример:

* Все операции с хранилищем ключей отображаются в формате `Vault<action>`, например `VaultGet` или `VaultCreate`.
* Все операции с ключами отображаются в формате `Key<action>`, например `KeySign` или `KeyList`.
* Все операции с секретами отображаются в формате `Secret<action>`, например `SecretGet` или `SecretListVersions`.

В следующей таблице перечислены значения **operationName** и соответствующие команды REST API.

### <a name="operation-names-table"></a>Таблица имен операций

# <a name="vault"></a>[Хранилище](#tab/Vault)

| operationName | Команда REST API |
| --- | --- |
| **Аутентификация** |Аутентификация через конечную точку Azure Active Directory |
| **VaultGet** |[Получение сведений о хранилище ключей](/rest/api/keyvault/vaults) |
| **VaultPut** |[Создание или обновление хранилища ключей](/rest/api/keyvault/vaults) |
| **VaultDelete** |[Удаление хранилища ключей](/rest/api/keyvault/vaults) |
| **VaultPatch** |[Update a key vault (Создание или обновление хранилища ключей)](/rest/api/keyvault/vaults) |
| **VaultList** |[Список всех хранилищ ключей в группе ресурсов](/rest/api/keyvault/vaults) |
| **VaultPurge** |[Очистка удаленного хранилища](/rest/api/keyvault/vaults/purgedeleted) |
| **VaultRecover** |Восстановление удаленного хранилища|
| **VaultGetDeleted** |[Получение данных об удаленном хранилище](/rest/api/keyvault/vaults/getdeleted) |
| **VaultListDeleted** |[Вывод списка удаленных хранилищ](/rest/api/keyvault/vaults/listdeleted) |
| **VaultAccessPolicyChangedEventGridNotification** | Опубликованное событие изменения политики доступа к хранилищу |

# <a name="keys"></a>[Ключи](#tab/Keys)

| operationName | Команда REST API |
| --- | --- |
| **KeyCreate** |[Создание ключа](/rest/api/keyvault/createkey) |
| **KeyGet** |[Получение сведений о ключе](/rest/api/keyvault/getkey) |
| **KeyImport** |[Импорт ключа в хранилище](/rest/api/keyvault/vaults) |
| **KeyDelete** |[Удаление ключа](/rest/api/keyvault/deletekey) |
| **KeySign** |[Вход с помощью ключа](/rest/api/keyvault/sign) |
| **KeyVerify** |[Проверка с помощью ключа](/rest/api/keyvault/vaults) |
| **KeyWrap** |[Перенос ключа](/rest/api/keyvault/wrapkey) |
| **KeyUnwrap** |[Развертывание ключа](/rest/api/keyvault/unwrapkey) |
| **KeyEncrypt** |[Шифрование с помощью ключа](/rest/api/keyvault/encrypt) |
| **KeyDecrypt** |[Расшифровка с помощью ключа](/rest/api/keyvault/decrypt) |
| **KeyUpdate** |[Обновление ключа](/rest/api/keyvault/updatekey) |
| **KeyList** |[Перечисление ключей в хранилище](/rest/api/keyvault/getkeys) |
| **KeyListVersions** |[Перечисление версий ключа](/rest/api/keyvault/getkeyversions) |
| **KeyPurge** |[Очистка ключа](/rest/api/keyvault/purgedeletedkey) |
| **KeyBackup** |[Создание резервной копии ключа](/rest/api/keyvault/backupkey) |
| **KeyRestore** |[Восстановление ключа](/rest/api/keyvault/restorekey) |
| **KeyRecover** |[Восстановление ключа](/rest/api/keyvault/recoverdeletedkey) |
| **KeyGetDeleted** |[Получение данных об удаленном ключе](/rest/api/keyvault/getdeletedkey) |
| **KeyListDeleted** |[Вывод списка удаленных ключей в хранилище](/rest/api/keyvault/getdeletedkeys) |
| **KeyNearExpiryEventGridNotification** |Опубликованное событие приближения истечения срока действия ключа |
| **KeyExpiredEventGridNotification** |Опубликованное событие истечения срока действия ключа |

# <a name="secrets"></a>[Секреты](#tab/Secrets)

| operationName | Команда REST API |
| --- | --- |
| **SecretSet** |[Создание секрета](/rest/api/keyvault/updatecertificate) |
| **SecretGet** |[Получение секрета](/rest/api/keyvault/getsecret) |
| **SecretUpdate** |[Обновление секрета](/rest/api/keyvault/updatesecret) |
| **SecretDelete** |[Удаление секрета](/rest/api/keyvault/deletesecret) |
| **SecretList** |[Список секретов в хранилище](/rest/api/keyvault/getsecrets) |
| **SecretListVersions** |[Список версий секрета](/rest/api/keyvault/getsecretversions) |
| **SecretPurge** |[Очистка секрета](/rest/api/keyvault/purgedeletedsecret) |
| **SecretBackup** |[Создание резервной копии для секрета](/rest/api/keyvault/backupsecret) |
| **SecretRestore** |[Восстановление секрета](/rest/api/keyvault/restoresecret) |
| **SecretRecover** |[Отмена удаления секрета](/rest/api/keyvault/recoverdeletedsecret) |
| **SecretGetDeleted** |[Получение данных об удаленном секрете](/rest/api/keyvault/getdeletedsecret) |
| **SecretListDeleted** |[Вывод списка удаленных секретов в хранилище](/rest/api/keyvault/getdeletedsecrets) |
| **SecretNearExpiryEventGridNotification** |Опубликованное событие приближения истечения срока действия секрета |
| **SecretExpiredEventGridNotification** |Опубликованное событие истечения срока действия секрета |

# <a name="certificates"></a>[Сертификаты](#tab/Cerificates)

| operationName | Команда REST API |
| --- | --- |
| **CertificateGet** |[Получение информации о сертификате](/rest/api/keyvault/getcertificate) |
| **CertificateCreate** |[Создание сертификата](/rest/api/keyvault/createcertificate) |
| **CertificateImport** |[Импорт сертификата в хранилище](/rest/api/keyvault/importcertificate) |
| **CertificateUpdate** |[Обновление сертификата](/rest/api/keyvault/updatecertificate) |
| **CertificateList** |[Вывод списка сертификатов в хранилище](/rest/api/keyvault/getcertificates) |
| **CertificateListVersions** |[Вывод списка версий сертификата](/rest/api/keyvault/getcertificateversions) |
| **CertificateDelete** |[Удаление сертификата](/rest/api/keyvault/deletecertificate) |
| **CertificatePurge** |[Очистка сертификата](/rest/api/keyvault/purgedeletedcertificate) |
| **CertificateBackup** |[Создание резервной копии сертификата](/rest/api/keyvault/backupcertificate) |
| **CertificateRestore** |[Восстановление сертификата](/rest/api/keyvault/restorecertificate) |
| **CertificateRecover** |[Отмена удаления сертификата](/rest/api/keyvault/recoverdeletedcertificate) |
| **CertificateGetDeleted** |[Получение данных об удаленном сертификате](/rest/api/keyvault/getdeletedcertificate) |
| **CertificateListDeleted** |[Вывод списка удаленных сертификатов в хранилище](/rest/api/keyvault/getdeletedcertificates) |
| **CertificatePolicyGet** |[Получение данных о политике сертификата](/rest/api/keyvault/getcertificatepolicy) |
| **CertificatePolicyUpdate** |[Обновление политики сертификата](/rest/api/keyvault/updatecertificatepolicy) |
| **CertificatePolicySet** |[Создание политики сертификата](/rest/api/keyvault/createcertificate) |
| **CertificateContactsGet** |[Получение контактных данных для управления сертификатами](/rest/api/keyvault/getcertificatecontacts) |
| **CertificateContactsSet** |[Указание контактных данных для управления сертификатами](/rest/api/keyvault/setcertificatecontacts) |
| **CertificateContactsDelete** |[Удаление контактных данных для управления сертификатами](/rest/api/keyvault/deletecertificatecontacts) |
| **CertificateIssuerGet** |[Получение данных об издателе сертификата](/rest/api/keyvault/getcertificateissuer) |
| **CertificateIssuerSet** |[Указание данных об издателе сертификата](/rest/api/keyvault/setcertificateissuer) |
| **CertificateIssuerUpdate** |[Обновление данных об издателе сертификата](/rest/api/keyvault/updatecertificateissuer) |
| **CertificateIssuerDelete** |[Удаление издателя сертификата](/rest/api/keyvault/deletecertificateissuer) |
| **CertificateIssuersList** |[Вывод списка издателей сертификатов](/rest/api/keyvault/getcertificateissuers) |
| **CertificateEnroll** |Регистрация сертификата |
| **CertificateRenew** |Обновление сертификата |
| **CertificatePendingGet** |Получение данных об ожидающем сертификате |
| **CertificatePendingMerge** |Ожидание слияния сертификатов |
| **CertificatePendingUpdate** |Ожидание обновления сертификата |
| **CertificatePendingDelete** |Удаление ожидающего сертификата |
| **CertificateNearExpiryEventGridNotification** |Опубликованное событие приближения истечения срока действия сертификата |
| **CertificateExpiredEventGridNotification** |Опубликованное событие истечения срока действия сертификата |

---

## <a name="use-azure-monitor-logs"></a>Использование журналов Azure Monitor

Решение Key Vault в журналах Azure Monitor позволяет просматривать журналы `AuditEvent` для Key Vault. В журналах Azure Monitor запросы по журналам используются для анализа данных и получения необходимых сведений. 

Дополнительные сведения, включая инструкции по настройке, см. в разделе [Мониторинг службы Key Vault с помощью Azure Monitor для Key Vault (предварительная версия)](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Включение ведения журнала Key Vault](howto-logging.md)
- [Azure Monitor](../../azure-monitor/index.yml)
- Руководство по использованию Azure Key Vault в веб-приложении .NET см. в [этой статье](tutorial-net-create-vault-azure-web-app.md).
- Справочные материалы по программированию см. в статье [Руководство разработчика хранилища ключей Azure](developers-guide.md).
- Полный список командлетов Azure PowerShell 1.0 для хранилища ключей Azure см. в статье [Azure Key Vault cmdlets](/powershell/module/az.keyvault/#key_vault) (Командлеты хранилища ключей Azure).
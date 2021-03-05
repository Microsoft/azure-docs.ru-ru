---
title: Конечные точки служб для виртуальной сети для Azure Key Vault
description: Узнайте, как конечные точки службы виртуальной сети для Azure Key Vault позволяют ограничить доступ к указанной виртуальной сети, включая сценарии использования.
services: key-vault
author: amitbapat
ms.author: ambapat
manager: rkarlin
ms.date: 01/02/2019
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.openlocfilehash: 4f615ed44594379c79f179a467888652c597e286
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102217182"
---
# <a name="virtual-network-service-endpoints-for-azure-key-vault"></a>Конечные точки служб для виртуальной сети для Azure Key Vault

Конечные точки служб для виртуальной сети для Azure Key Vault позволяют ограничить доступ к определенной виртуальной сети. Также эти конечные точки позволяют ограничить доступ определенным набором диапазонов адресов IPv4 (протокол IP версии 4). Для любого пользователя при попытке подключения к хранилищу ключей из любых других расположений доступ будет отклонен.

Из этого ограничения есть одно важное исключение. Если пользователь разрешил доступ доверенным службам Майкрософт, брандмауэр будет пропускать подключения из этих служб. К таким службам относятся, к примеру, Office 365 Exchange Online, Office 365 SharePoint Online, служба вычислений Azure, Azure Resource Manager и Azure Backup. Эти пользователи по-прежнему должны предоставить допустимый маркер Azure Active Directory и иметь разрешения (настроенные в политиках доступа) для выполнения запрошенной операции. Дополнительные сведения см. в статье [Конечные точки служб для виртуальной сети](../../virtual-network/virtual-network-service-endpoints-overview.md).

## <a name="usage-scenarios"></a>Сценарии использования

[Брандмауэры и виртуальные сети Key Vault](network-security.md) можно настроить так, чтобы по умолчанию запрещать трафик из всех сетей (в том числе из Интернета). Вы можете разрешить доступ для трафика из конкретных виртуальных сетей Azure или диапазонов общедоступных IP-адресов, что позволит определить границу защищенной сети для приложений.

> [!NOTE]
> Правила брандмауэра и виртуальной сети Key Vault применяются только к [плоскости данных](secure-your-key-vault.md#data-plane-access-control) в Key Vault. Правила брандмауэра и виртуальной сети не влияют на операции плоскости управления Key Vault (создание, удаление, изменение хранилища ключей, настройка политик доступа, настройка правил брандмауэров и виртуальной сети).

Ниже приведены некоторые примеры использования конечных точек службы.

* Если Key Vault используется для хранения ключей шифрования, секретов приложения и сертификатов, доступ к которым из общедоступного Интернета вы хотите запретить.
* Если вы хотите заблокировать доступ к хранилищу ключей, чтобы к нему могли подключаться только одно приложение или небольшой набор назначенных узлов.
* Если у вас есть приложение, работающее в виртуальной сети Azure, для которой заблокирован любой исходящий и входящий трафик, но приложению требуется подключение к хранилищу ключей для получения секретов, сертификатов или криптографических ключей.

## <a name="trusted-services"></a>Доверенные службы

Ниже приведен список **доверенных служб**, которые получают доступ к хранилищу ключей, если включен соответствующий параметр.

|Доверенная служба|Поддерживаемые сценарии использования|
| --- | --- |
|Служба развертывания виртуальных машин Azure|[Развертывание сертификатов на виртуальных машинах из хранилища ключей, управляемого пользователем](/archive/blogs/kv/updated-deploy-certificates-to-vms-from-customer-managed-key-vault)|
|Служба развертывания шаблонов Azure Resource Manager|[Передача защищенных значений в процессе развертывания](../../azure-resource-manager/templates/key-vault-parameter.md).|
|Служба шифрования томов для шифрования дисков Azure|Разрешение доступа к ключу BitLocker (виртуальная машина Windows), парольной фразе DM (виртуальная машина Linux) и ключу шифрования ключей во время развертывания виртуальной машины. Это позволяет включить [шифрование дисков Azure](../../security/fundamentals/encryption-overview.md).|
|Azure Backup|Разрешение резервного копирования и восстановления соответствующих ключей и секретов во время резервного копирования виртуальной машины Azure с помощью службы [Azure Backup](../../backup/backup-overview.md).|
|Exchange Online и SharePoint Online|Разрешение доступа к ключу клиента для функции шифрования службы хранилища Azure с использованием [ключа пользователя](/microsoft-365/compliance/customer-key-overview).|
|Azure Information Protection|Разрешение доступа к ключу клиента для [Azure Information Protection.](/azure/information-protection/what-is-information-protection)|
|Служба приложений Azure|[Развертывание сертификата веб-приложения Azure через Key Vault](https://azure.github.io/AppService/2016/05/24/Deploying-Azure-Web-App-Certificate-through-Key-Vault.html)|
|База данных SQL Azure|[Прозрачное шифрование данных с поддержкой создание собственных ключей для базы данных SQL Azure и Azure синапсе Analytics](../../azure-sql/database/transparent-data-encryption-byok-overview.md?view=sql-server-2017&preserve-view=true&viewFallbackFrom=azuresqldb-current).|
|Хранилище Azure|[Шифрование службы хранилища использование ключей, управляемых клиентом, в Azure Key Vault](../../storage/common/customer-managed-keys-configure-key-vault.md).|
|Хранилище озера данных Azure|[Шифрование данных в Azure Data Lake Storage](../../data-lake-store/data-lake-store-encryption.md) с помощью управляемого пользователем ключа.|
|Azure Synapse Analytics|[Шифрование данных с помощью управляемых клиентом ключей в Azure Key Vault](../../synapse-analytics/security/workspaces-encryption.md)|
|Azure Databricks|[Быстрая и простая служба аналитики на основе Apache Spark для совместной работы](/azure/databricks/scenarios/what-is-azure-databricks).|
|Служба "Управление API Azure"|[Развертывание сертификатов для пользовательского домена из Key Vault с помощью MSI](../../api-management/api-management-howto-use-managed-service-identity.md#use-ssl-tls-certificate-from-azure-key-vault)|
|Фабрика данных Azure|[Выбор учетных данных хранилища данных в Key Vault из фабрики данных](https://go.microsoft.com/fwlink/?linkid=2109491)|
|Центры событий Azure|[Разрешить доступ к хранилищу ключей для сценария с управляемыми клиентом ключами](../../event-hubs/configure-customer-managed-key.md)|
|Служебная шина Azure|[Разрешить доступ к хранилищу ключей для сценария с управляемыми клиентом ключами](../../service-bus-messaging/configure-customer-managed-key.md)|
|Импорт и экспорт Microsoft Azure| [Использование управляемых клиентом ключей в Azure Key Vault для службы импорта и экспорта](../../import-export/storage-import-export-encryption-key-portal.md)
|Реестр контейнеров Azure|[Шифрование реестра с помощью управляемых клиентом ключей](../../container-registry/container-registry-customer-managed-keys.md)

> [!NOTE]
> Настройте политики доступа в Key Vault так, чтобы разрешить доступ к Key Vault соответствующим службам.

## <a name="next-steps"></a>Дальнейшие действия

- Пошаговые инструкции см. в разделе [Настройка брандмауэров Azure Key Vault и виртуальных сетей](network-security.md) .
- см. [Azure Key Vault Обзор безопасности](security-overview.md) .

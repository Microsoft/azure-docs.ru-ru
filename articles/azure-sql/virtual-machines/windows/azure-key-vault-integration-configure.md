---
title: Интеграция Key Vault с SQL Server на виртуальных машинах Windows в Azure (Resource Manager) | Документация Майкрософт
description: Узнайте, как автоматизировать настройку шифрования SQL Server для использования с хранилищем ключей Azure. В этом разделе описывается, как использовать интеграцию Azure Key Vault с виртуальными машинами SQL, созданными с помощью Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: cd66dfb1-0e9b-4fb0-a471-9deaf4ab4ab8
ms.service: virtual-machines-sql
ms.subservice: security
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/30/2018
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: a6955b7fc4948faaea6db426545f8cc3d1eece35
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97359903"
---
# <a name="configure-azure-key-vault-integration-for-sql-server-on-azure-vms-resource-manager"></a>Настройка интеграции Azure Key Vault для SQL Server на виртуальных машинах Azure (Resource Manager)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Существует несколько функций шифрования SQL Server, например [прозрачное шифрование данных (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption), [шифрование на уровне столбцов (CLE)](/sql/t-sql/functions/cryptographic-functions-transact-sql) и [шифрование резервной копии](/sql/relational-databases/backup-restore/backup-encryption). Эти формы шифрования требуют хранить используемые для шифрования ключи и управлять ими. Служба Azure Key Vault предназначена для обеспечения лучшей защиты этих ключей и управления ими в надежном расположении с высоким уровнем доступности. [Соединитель SQL Server](https://www.microsoft.com/download/details.aspx?id=45344) позволяет SQL Server использовать эти ключи из хранилища ключей Azure.

Если вы используете SQL Server в локальной среде, можете выполнить шаги для получения [доступа к Azure Key Vault из локального экземпляра SQL Server](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server). Если SQL Server расположен на виртуальных машинах Azure, можно сэкономить время, воспользовавшись функцией *интеграции Azure Key Vault*.

Если эта функция включена, она автоматически устанавливает соединитель SQL Server, настраивает поставщик расширенного управления ключами для доступа к хранилищу ключей Azure и создает учетные данные, которые позволяют получить доступ к вашему хранилищу. Если вы изучили шаги в ранее упомянутой документации для локального компьютера, то увидели, что эта функция автоматизирует шаги 2 и 3. Единственное, что останется сделать вручную, — создать хранилище ключей и ключи. Далее вся настройка виртуальной машины SQL Server выполняется автоматически. Когда настройка этой функции завершится, можно выполнить инструкции Transact-SQL (T-SQL), чтобы начать шифрование или резервное копирование баз данных как обычно.

[!INCLUDE [Prepare for Key Vault integration](../../../../includes/virtual-machines-sql-server-akv-prepare.md)]

  >[!NOTE]
  > Поставщик расширенного управления ключами (EKM) версии 1.0.4.0 устанавливается на виртуальной машине SQL Server с помощью расширения [IaaS (инфраструктура как услуга) для SQL](./sql-server-iaas-agent-extension-automate-management.md). При обновлении расширения IaaS для SQL версия поставщика не обновляется. При необходимости вручную обновите версию поставщика EKM (например, во время миграции на Управляемый экземпляр Базы данных SQL).


## <a name="enabling-and-configuring-key-vault-integration"></a>Включение и настройка интеграции Key Vault
Интеграцию Key Vault можно включить во время подготовки или настроить ее для существующих виртуальных машин.

### <a name="new-vms"></a>Новые виртуальные машины
При подготовке новой виртуальной машины SQL с помощью Resource Manager на портале Azure предоставляется возможность включить интеграцию Azure Key Vault. Функции Azure Key Vault доступны только для выпусков SQL Server Enterprise, Developer и Evaluation.

![Интеграция SQL с хранилищем ключей Azure](./media/azure-key-vault-integration-configure/azure-sql-arm-akv.png)

Пошаговые инструкции по подготовке см. в статье [Подготовка виртуальной машины SQL на портале Azure](create-sql-vm-portal.md).

### <a name="existing-vms"></a>Существующие виртуальные машины

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Для существующих виртуальных машин SQL перейдите к ресурсу [Виртуальные машины SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) и в разделе **Настройки** выберите **Безопасность**. Выберите **Включить**, чтобы включить интеграцию Azure Key Vault. 

![Интеграция SQL с Key Vault для существующих виртуальных машин](./media/azure-key-vault-integration-configure/azure-sql-rm-akv-existing-vms.png)

По завершении в нижней части страницы **Безопасность** нажмите кнопку **Применить**, чтобы сохранить изменения.

> [!NOTE]
> Имя учетных данных, которое мы создали, позже будет сопоставлено с именем для входа в SQL. С помощью этих данных для входа в SQL можно получить доступ к хранилищу ключей. 


> [!NOTE]
> Можно также настроить интеграцию Key Vault с помощью шаблона. Чтобы получить дополнительные сведения, ознакомьтесь с [шаблоном быстрого запуска Azure для интеграции с хранилищем ключей Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-sql-existing-keyvault-update).


[!INCLUDE [Key Vault integration next steps](../../../../includes/virtual-machines-sql-server-akv-next-steps.md)]
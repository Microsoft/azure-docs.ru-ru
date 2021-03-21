---
title: Изменение модели лицензирования для виртуальной машины SQL в Azure
description: Узнайте, как изменить модель лицензирования для виртуальной машины SQL Server в Azure с оплаты по мере использования на использование собственной лицензии, применяя Преимущество гибридного использования Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.subservice: management
ms.workload: iaas-sql-server
ms.date: 11/13/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 5813331d5eafd953d776dd19d9cc885ff71b8be0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100361559"
---
# <a name="change-the-license-model-for-a-sql-virtual-machine-in-azure"></a>Изменение модели лицензирования для виртуальной машины SQL в Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]


В этой статье описывается, как изменить модель лицензии для SQL Server виртуальной машины в Azure с помощью [расширения агента IaaS SQL](./sql-server-iaas-agent-extension-automate-management.md).

## <a name="overview"></a>Обзор

Существует три модели лицензирования для виртуальной машины Azure, на которой размещена SQL Server: оплата по мере использования, Преимущество гибридного использования Azure (АХБ), высокая доступность и аварийное восстановление (HA/DR). Вы можете изменить модель лицензирования для виртуальной машины SQL Server с помощью портала Azure, Azure CLI или PowerShell. 

- Модель **с оплатой по мере использования** означает, что стоимость лицензии SQL Server включается в стоимость каждой секунды выполнения виртуальной машины Azure.
- [Преимущество гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit/) позволяет применять собственную лицензию SQL Server для виртуальной машины с SQL Server. 
- Тип лицензии **Ha/Dr** используется для [бесплатной реплики высокой доступности или аварийного восстановления](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) в Azure. 

Преимущество гибридного использования Azure позволяет применять на виртуальных машинах Azure лицензии SQL Server с поддержкой Software Assurance ("соответствующие лицензии"). В рамках Преимущества гибридного использования Azure клиентам не нужно платить за лицензию SQL Server на виртуальной машине. При этом клиенты должны платить за использование базовых облачных вычислительных ресурсов (базовый тариф), хранилищ и резервных копий. Кроме того, клиент оплачивает входящий и исходящий трафик, связанный с использованием служб (если это применимо).

Согласно [условиям продукта](https://www.microsoft.com/licensing/terms/productoffering/MicrosoftAzureServices/EAEAS)Майкрософт: "клиенты должны указать, что они используют базу данных SQL azure (Управляемый экземпляр, Эластичный пул и отдельная база данных), фабрику данных azure, SQL Server Integration Services или SQL Server виртуальные машины в Преимущество гибридного использования Azure для SQL Server при настройке рабочих нагрузок в Azure".

Обеспечить соблюдение требований и сообщить об использовании Преимущества гибридного использования Azure для SQL Server на виртуальной машине Azure можно тремя способами.

- Подготовьте виртуальную машину на основе образа SQL Server с использованием собственной лицензии из Azure Marketplace. Этот вариант сейчас доступен только клиентам с Соглашением Enterprise.
- Подготовьте виртуальную машину на основе образа SQL Server с оплатой по мере использования из Azure Marketplace, а затем активируйте Преимущество гибридного использования Azure.
- Самостоятельная установка SQL Server на виртуальной машине Azure, ручная [Регистрация в расширении агента IaaS SQL](sql-agent-extension-manually-register-single-vm.md)и активация преимущество гибридного использования Azure.

Тип лицензии для SQL Server можно настроить при подготовке виртуальной машины или в любое время позже. Переключение между моделями лицензирования выполняется без простоя, перезапуска виртуальной машины или службы SQL Server и дополнительных затрат, вступая в силу немедленно. По сути, активация Преимущества гибридного использования Azure *сокращает* затраты.

## <a name="prerequisites"></a>Предварительные требования

Для изменения модели лицензирования для виртуальной машины SQL Server требуется следующее: 

- [Подписка Azure](https://azure.microsoft.com/free/).
- [SQL Server виртуальную машину](./create-sql-vm-portal.md) , зарегистрированную с [расширением агента IaaS SQL](./sql-server-iaas-agent-extension-automate-management.md).
- [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) как обязательное условие для использования [Преимущества гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-benefit/). 


## <a name="change-license-model"></a>Изменение модели лицензирования

# <a name="azure-portal"></a>[Портал Azure](#tab/azure-portal)

[!INCLUDE [windows-virtual-machines-sql-use-new-management-blade](../../../../includes/windows-virtual-machines-sql-new-resource.md)]

Модель лицензирования можно изменить непосредственно на портале, сделав следующее: 

1. Откройте [портал Azure](https://portal.azure.com) и найдите [ресурс виртуальных машин SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource) для нужной виртуальной машины SQL Server. 
1. Выберите **Конфигурация** в разделе **Параметры**. 
1. Выберите вариант **Преимущество гибридного использования Azure** и установите флажок, чтобы подтвердить наличие лицензии SQL Server с поддержкой Software Assurance. 
1. Щелкните **Применить** в нижней части страницы **Конфигурация**. 

![Преимущество гибридного использования Azure на портале](./media/licensing-model-azure-hybrid-benefit-ahb-change/ahb-in-portal.png)


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Вы можете изменить модель лицензирования с помощью Azure CLI.  

Укажите следующие значения для **типа лицензии**:
- `AHUB` для Преимущество гибридного использования Azure
- `PAYG` для оплаты по мере использования
- `DR` Активация бесплатной реплики высокой доступности или аварийного восстановления


```azurecli-interactive
# example: az sql vm update -n AHBTest -g AHBTest --license-type AHUB

az sql vm update -n <VMName> -g <ResourceGroupName> --license-type <license-type>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Вы можете изменить модель лицензирования с помощью PowerShell.

Укажите следующие значения для **типа лицензии**:
- `AHUB` для Преимущество гибридного использования Azure
- `PAYG` для оплаты по мере использования
- `DR` Активация бесплатной реплики высокой доступности или аварийного восстановления


```powershell-interactive
Update-AzSqlVM -ResourceGroupName <resource_group_name> -Name <VM_name> -LicenseType <license-type>
```

---

## <a name="remarks"></a>Remarks

- Клиенты, участвующие в программе "Поставщик облачных речений Azure" (CSP), могут использовать Преимущество гибридного использования Azure, предварительно развернув виртуальную машину с оплатой по мере использования, а затем переключив ее на использование собственной лицензии, если есть активная поддержка Software Assurance.
- При удалении ресурса виртуальной машины SQL вы вернетесь к жестко запрограммированному параметру лицензии образа. 
- Возможность изменения модели лицензии является функцией расширения агента IaaS SQL. Развертывание образа Azure Marketplace с помощью портал Azure автоматически регистрирует SQL Server виртуальную машину с расширением. Но тем клиентам, которые самостоятельно устанавливают SQL Server, нужно будет вручную [зарегистрировать виртуальные машины SQL Server](sql-agent-extension-manually-register-single-vm.md). 
- Для добавления виртуальной машины SQL Server в группу доступности необходимо повторно создать эту виртуальную машину. Это означает, что все добавленные в группу доступности виртуальные машины снова получат тип лицензии с оплатой по мере использования, который настроен по умолчанию. После этого нужно будет снова включить Преимущество гибридного использования Azure. 


## <a name="limitations"></a>Ограничения

Изменение модели лицензирования связано со следующими ограничениями:
   - Эта возможность доступна только клиентам, участвующим в программе [Software Assurance](https://www.microsoft.com/en-us/licensing/licensing-programs/software-assurance-overview).
   - Поддерживаются только выпуски SQL Server Standard и Enterprise. Изменять тип лицензии для выпусков Express, Web и Developer нельзя. 
   - Поддерживаются только виртуальные машины, развернутые с помощью модели развертывания с Azure Resource Manager. Виртуальные машины, развернутые с помощью классической модели развертывания, не поддерживаются. 
   - Доступно только для общедоступных облаков и Azure для государственных организаций. 

> [!Note]
> Для Преимущество гибридного использования Azure доступны только SQL Server на основе ядер с лицензированием Software Assurance или подпиской на подписку. Если вы используете лицензирование Server + CAL для SQL Server и у вас есть программа Software Assurance, можно использовать собственную лицензию на образ виртуальной машины Azure SQL Server, чтобы использовать перемещение лицензий для этих серверов, но нельзя использовать другие функции Преимущество гибридного использования Azure. 

## <a name="known-errors"></a>Известные ошибки

Ознакомьтесь с типичными ошибками и их разрешениями. 

**Ресурс "Microsoft. Склвиртуалмачине/Склвиртуалмачинес/ \<resource-group> " в группе ресурсов " \<resource-group> " не найден.**

Эта ошибка возникает при попытке изменить модель лицензии на SQL Server виртуальной машине, которая не зарегистрирована в расширении агента IaaS SQL Server:

`The Resource 'Microsoft.SqlVirtualMachine/SqlVirtualMachines/\<resource-group>' under resource group '\<resource-group>' was not found. The property 'sqlServerLicenseType' cannot be found on this object. Verify that the property exists and can be set.`

Вам потребуется зарегистрировать подписку в поставщике ресурсов, а затем [зарегистрировать SQL Server виртуальную машину с расширением агента IaaS SQL](sql-agent-extension-manually-register-single-vm.md). 



## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях: 

* [Обзор SQL Server на виртуальных машинах Windows](sql-server-on-azure-vm-iaas-what-is-overview.md).
* [Вопросы и ответы об SQL Server на виртуальных машинах Windows](frequently-asked-questions-faq.md)
* [Руководство по ценам для виртуальных машин SQL Server в Azure](pricing-guidance.md)
* [Заметки о выпуске SQL Server на виртуальных машинах Windows](../../database/doc-changes-updates-release-notes.md)
* [Общие сведения о расширении агента IaaS SQL](./sql-server-iaas-agent-extension-automate-management.md)

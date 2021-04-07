---
author: stevestein
ms.service: sql-database
ms.subservice: operations
ms.topic: include
ms.date: 08/23/2019
ms.author: sstein
ms.openlocfilehash: 3cc8e836d39a38f950e15d31aeb3e3acdca45acc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "84031755"
---
## <a name="create-and-manage-azure-sql-resources-with-the-azure-portal"></a>Создание ресурсов SQL Azure и управление ими с помощью портала Azure

Портал Azure предоставляет отдельную страницу, на которой вы можете управлять [всеми ресурсами SQL Azure](https://go.microsoft.com/fwlink/?linkid=2100641), а также виртуальными машинами SQL.

Чтобы получить доступ к странице **SQL Azure** в меню на портале Azure, выберите **SQL Azure** или найдите и выберите **SQL Azure** на любой странице.

> [!NOTE]
> **Azure SQL** — это быстрый и простой способ получения доступа ко всем ресурсам SQL на портале Azure, включая отдельную базу данных и базу данных в пуле в Базе данных SQL Azure, а также логические экземпляры SQL Server, на которых размещены эти ресурсы, Управляемые экземпляры SQL и виртуальные машины SQL.  [SQL Azure](../azure-sql-iaas-vs-paas-what-is-overview.md) — это не служба или ресурс, а семейство служб, связанных с SQL. 

Чтобы управлять существующими ресурсами, выберите нужный элемент в списке. Чтобы создать ресурсы SQL Azure, выберите **+ Добавить**. 

![Страница портала SQL Azure](./media/sql-database-create-manage-portal/add-azure-sql-resources.png)

После выбора параметра **+ Добавить**, просмотрите дополнительные сведения о различных параметрах, щелкнув **Показать сведения** для любой плитки.

![сведения о плитке баз данных](./media/sql-database-create-manage-portal/single-sql-database-deployment-options.png)

Подробная информация доступна в следующих статьях:

- [Создание отдельной базы данных](../database/single-database-create-quickstart.md)
- [Создание эластичного пула](../database/elastic-pool-overview.md#creating-a-new-sql-database-elastic-pool-using-the-azure-portal)
- [Создание управляемого экземпляра](../managed-instance/instance-create-quickstart.md)
- [Создание виртуальной машины SQL](../virtual-machines/windows/sql-vm-create-portal-quickstart.md)
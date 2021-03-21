---
title: Отмена операций управления
titleSuffix: Azure SQL Managed Instance
description: Узнайте, как отменить операции управления Управляемый экземпляр SQL Azure.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, bonova, MashaMSFT
ms.date: 09/03/2020
ms.openlocfilehash: 342491178d55dacbdc68e6c9042623d381dff898
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96861550"
---
# <a name="canceling-azure-sql-managed-instance-management-operations"></a>Отмена операций управления Управляемый экземпляр SQL Azure
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Управляемый экземпляр Azure SQL предоставляет возможность отмены некоторых [операций управления](management-operations-overview.md), таких как развертывание нового управляемого экземпляра или обновление свойств экземпляра. 

## <a name="overview"></a>Обзор

 Все операции управления можно классифицировать следующим образом:

- Развертывание экземпляра (создание нового экземпляра).
- Обновление экземпляра (изменение свойств экземпляра, например виртуальных ядер или зарезервированное хранилище).
- Удаление экземпляра.

Вы можете [отслеживать ход выполнения и состояние операций управления](management-operations-monitor.md) и при необходимости отменять некоторые из них. 

В следующей таблице перечислены операции управления, которые можно отменить, а также типичная Общая длительность:

Category  |Операция  |Отменяемого  |Предполагаемая длительность отмены  |
|---------|---------|---------|---------|
|Deployment (Развертывание) |Создание экземпляра |Да |90 % операций завершается в течение 5 минут. |
|Update |Увеличение/уменьшение масштаба хранилища экземпляров (общего назначения) |Нет |  |
|Update |Увеличение/уменьшение масштаба хранилища экземпляров (критически важный для бизнеса) |Да |90 % операций завершается в течение 5 минут. |
|Update |Вертикальное изменение масштаба вычислительных ресурсов экземпляра (количества виртуальных ядер, уровень "Общего назначения") |Да |90 % операций завершается в течение 5 минут. |
|Update |Вертикальное изменение масштаба вычислительных ресурсов экземпляра (количества виртуальных ядер, уровень "Критически важный для бизнеса") |Да |90 % операций завершается в течение 5 минут. |
|Update |Изменение уровня служб экземпляра (с "Общего назначения" на "Критически важный для бизнеса" и наоборот) |Да |90 % операций завершается в течение 5 минут. |
|Удалить |Удаление экземпляра |Нет |  |
|Удалить |Удаление виртуального кластера (выполняется в качестве операции, инициированной пользователем) |Нет |  |

## <a name="cancel-management-operation"></a>Отменить операцию управления

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы отменить операции управления с помощью портал Azure, выполните следующие действия.

1. Перейдите к [портал Azure](https://portal.azure.com)
1. Перейдите в колонку **обзор** управляемый экземпляр SQL. 
1. Щелкните поле **уведомления** рядом с текущей операцией, чтобы открыть страницу **текущей операции** . 

   :::image type="content" source="media/management-operations-cancel/open-ongoing-operation.png" alt-text="Выберите поле текущая операция, чтобы открыть страницу текущей операции.":::

1. Выберите **отменить операцию** в нижней части страницы. 

   :::image type="content" source="media/management-operations-cancel/cancel-operation.png" alt-text="Нажмите кнопку Отмена, чтобы отменить операцию.":::

1. Подтвердите, что вы хотите отменить операцию. 


Если запрос на отмену выполнен успешно, операция управления отменяется и происходит сбой. Вы получите уведомление о том, что отмена будет выполнена успешно или неудачно.

![Результат операции отмены](./media/management-operations-cancel/canceling-operation-result.png)


Если запрос Cancel завершается ошибкой или кнопка Отмена неактивна, это означает, что операция управления перешла в недоступное состояние и вскоре завершится.  Операция управления продолжит выполнение до ее завершения.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Если у вас еще не установлен Azure PowerShell, см. раздел [Установка модуля Azure PowerShell](/powershell/azure/install-az-ps).

Чтобы отменить операцию управления, необходимо указать имя операции управления. Поэтому сначала используйте команду Get, чтобы получить список операций, а затем отмените определенную операцию.

```powershell-interactive
$managedInstance = "<Your-instance-name>"
$resourceGroup = "<Your-resource-group-name>"

$managementOperations = Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup

foreach ($mo in $managementOperations ) {
    if($mo.State -eq "InProgress" -and $mo.IsCancellable){
        $cancelRequest = Stop-AzSqlInstanceOperation -ResourceGroupName $resourceGroup -ManagedInstanceName $managedInstance -Name $mo.Name
        Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup -Name $mo.Name
    }
}
```

Подробные сведения о командах см. в разделе [Get-азсклинстанцеоператион](/powershell/module/az.sql/get-azsqlinstanceoperation) и [останавливает-азсклинстанцеоператион](/powershell/module/az.sql/stop-azsqlinstanceoperation).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Если вы еще не установили Azure CLI, обратитесь к разделу [Установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы отменить операцию управления, необходимо указать имя операции управления. Поэтому сначала используйте команду Get, чтобы получить список операций, а затем отмените определенную операцию.

```azurecli-interactive
az sql mi op list -g yourResourceGroupName --mi yourInstanceName |
   --query "[?state=='InProgress' && isCancellable].{Name: name}" -o tsv |
while read -r operationName; do

az sql mi op cancel -g yourResourceGroupName --mi yourInstanceName -n $operationName
done
```

Подробное описание команд см. в разделе [AZ SQL MI Op](/cli/azure/sql/mi/op).

---

## <a name="canceled-deployment-request"></a>Отмененный запрос на развертывание

При использовании API версии 2020-02-02, как только запрос на создание экземпляра будет принят, экземпляр начнет существовать как ресурс, независимо от хода выполнения процесса развертывания (состояние управляемого экземпляра — **Подготовка**). Если отменить запрос на развертывание экземпляра (создание нового экземпляра), управляемый экземпляр перейдет из состояния **подготовки** в **фаиледтокреате**.

Экземпляры, которые не удалось создать, по-прежнему находятся в виде ресурса и: 

- Не начисляются
- Не подсчитайте ограничения ресурсов (подсеть или квота Виртуальное ядро).


> [!NOTE]
> Чтобы максимально ограничить шум в списке ресурсов или управляемых экземплярах, удалите экземпляры, которые не удалось развернуть или экземпляры с отмененными развертываниями. 


## <a name="next-steps"></a>Дальнейшие действия

- Сведения о создании первого управляемого экземпляра см. в [этом руководстве по началу работы](instance-create-quickstart.md).
- Сведения о функциях и список сравнения см. в статье [Сравнение функций: база данных SQL Azure и Управляемый экземпляр Azure SQL](../database/features-comparison.md).
- Дополнительные сведения о конфигурации виртуальной сети см. в разделе [Архитектура подключения к Управляемому экземпляру SQL Azure](connectivity-architecture-overview.md).
- Сведения о создании управляемого экземпляра и восстановлении базы данных из файла резервной копии см. в разделе [Краткое руководство. Создание управляемого экземпляра Управляемого экземпляра SQL](instance-create-quickstart.md).
- Сведения о миграции с помощью Azure Database Migration Service см. в разделе [Руководство. Миграция из SQL Server в Управляемый экземпляр базы данных SQL Azure в автономном режиме с помощью DMS](../../dms/tutorial-sql-server-to-managed-instance.md).

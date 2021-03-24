---
title: Удаление журнала развертывания
description: Описывает, как Azure Resource Manager автоматически удаляет развертывания из журнала развертывания. Развертывания удаляются, если размер журнала близок к превышению лимита в 800.
ms.topic: conceptual
ms.date: 03/23/2021
ms.openlocfilehash: fc4f7f33cdd7ccce3158aa95bd002f12c8c44c00
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104951969"
---
# <a name="automatic-deletions-from-deployment-history"></a>Автоматическое удаление из журнала развертывания

Каждый раз при развертывании шаблона сведения о развертывании записываются в журнал развертывания. Каждая группа ресурсов ограничена до 800 развертываний в журнале развертывания.

Azure Resource Manager автоматически удаляет развертывания из журнала, так как приближается предельное значение. Автоматическое удаление — это изменение предыдущего поведения. Ранее было необходимо вручную удалить развертывания из журнала развертывания, чтобы избежать возникновения ошибки. Это изменение было реализовано 6 августа 2020.

**Автоматические удаления поддерживаются для развертываний групп ресурсов. В настоящее время развертывания в журнале для [подписки](deploy-to-subscription.md), [группы управления](deploy-to-management-group.md)и развертывания [клиентов](deploy-to-tenant.md) не удаляются автоматически.**

> [!NOTE]
> Удаление развертывания из журнала не влияет ни на один из развернутых ресурсов.

## <a name="when-deployments-are-deleted"></a>При удалении развертываний

Развертывания удаляются из журнала при превышении 775 развертываний. Azure Resource Manager удаляет развертывания до тех пор, пока журнал не будет отключен до 750. Самые старые развертывания всегда удаляются первыми.

:::image type="content" border="false" source="./media/deployment-history-deletions/deployment-history.svg" alt-text="Удаления из журнала развертывания":::

> [!NOTE]
> Начальный номер (775) и конечный номер (750) могут быть изменены.
>
> Если у вашей группы ресурсов уже есть ограничение в 800, следующее развертывание завершится ошибкой. Процесс автоматического удаления запускается немедленно. Вы можете повторить попытку развертывания после короткого ожидания.

Помимо развертываний, при выполнении [операции "что-If](template-deploy-what-if.md) " или проверке развертывания также запускаются удаления.

При присвоении развертыванию имени, совпадающего с именем в журнале, вы сбрасываете его место в журнале. Развертывание перемещается в последнее место в журнале. При [откате к развертыванию](rollback-on-error.md) после возникновения ошибки также выполняется сброс развертывания.

## <a name="remove-locks-that-block-deletions"></a>Удалить блокировки, которые блокируют удаление

При наличии [CanNotDelete блокировки](../management/lock-resources.md) для группы ресурсов невозможно удалить развертывания для этой группы ресурсов. Необходимо снять блокировку, чтобы воспользоваться преимуществами автоматического удаления в журнале развертывания.

Чтобы удалить блокировку с помощью PowerShell, выполните следующие команды:

```azurepowershell-interactive
$lockId = (Get-AzResourceLock -ResourceGroupName lockedRG).LockId
Remove-AzResourceLock -LockId $lockId
```

Чтобы использовать Azure CLI для удаления блокировки, выполните следующие команды:

```azurecli-interactive
lockid=$(az lock show --resource-group lockedRG --name deleteLock --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="required-permissions"></a>Необходимые разрешения

Удаление запрашивается по идентификатору пользователя, который развернул шаблон. Чтобы удалить развертывания, пользователь должен иметь доступ к действию **Microsoft. Resources/deployments/Delete** . Если у пользователя нет необходимых разрешений, развертывания не удаляются из журнала.

Если у текущего пользователя нет необходимых разрешений, то при следующем развертывании будет предпринята попытка автоматического удаления.

## <a name="opt-out-of-automatic-deletions"></a>Отказаться от автоматических удалений

Вы можете отказаться от автоматического удаления из журнала. **Используйте этот параметр, только если требуется самостоятельно управлять журналом развертывания.** Ограничение в 800 развертываний в журнале по-прежнему применяется. Если вы превысили 800 развертываний, вы получите сообщение об ошибке, и развертывание завершится ошибкой.

Чтобы отключить автоматическое удаление, зарегистрируйте `Microsoft.Resources/DisableDeploymentGrooming` флаг компонента. При регистрации флага функции вы отказываетесь от автоматического удаления для всей подписки Azure. Вы не можете отказаться от конкретной группы ресурсов. Чтобы повторно включить автоматическое удаление, отмените регистрацию флага компонента.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для PowerShell используйте командлет [Register-азпровидерфеатуре](/powershell/module/az.resources/Register-AzProviderFeature).

```azurepowershell-interactive
Register-AzProviderFeature -ProviderNamespace Microsoft.Resources -FeatureName DisableDeploymentGrooming
```

Чтобы просмотреть текущее состояние подписки, используйте:

```azurepowershell-interactive
Get-AzProviderFeature -ProviderNamespace Microsoft.Resources -FeatureName DisableDeploymentGrooming
```

Чтобы повторно включить автоматическое удаление, используйте Azure REST API или Azure CLI.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Для Azure CLI используйте команду [AZ Feature Register](/cli/azure/feature#az-feature-register).

```azurecli-interactive
az feature register --namespace Microsoft.Resources --name DisableDeploymentGrooming
```

Чтобы просмотреть текущее состояние подписки, используйте:

```azurecli-interactive
az feature show --namespace Microsoft.Resources --name DisableDeploymentGrooming
```

Чтобы повторно включить автоматическое удаление, используйте команду [AZ Feature Отмена регистрации](/cli/azure/feature#az-feature-unregister).

```azurecli-interactive
az feature unregister --namespace Microsoft.Resources --name DisableDeploymentGrooming
```

# <a name="rest"></a>[REST](#tab/rest)

Для REST API используйте [функции-Register](/rest/api/resources/features/register).

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Resources/features/DisableDeploymentGrooming/register?api-version=2015-12-01
```

Чтобы просмотреть текущее состояние подписки, используйте:

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Resources/features/DisableDeploymentGrooming/register?api-version=2015-12-01
```

Чтобы повторно включить автоматическое удаление, используйте [функции — Отмена регистрации](/rest/api/resources/features/unregister)

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Features/providers/Microsoft.Resources/features/DisableDeploymentGrooming/unregister?api-version=2015-12-01
```

---

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о просмотре журнала развертывания см. в статье [Просмотр журнала развертывания с помощью Azure Resource Manager](deployment-history.md).

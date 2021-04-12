---
title: Устранение неполадок с учетной записью службы автоматизации Azure
description: В этой статье рассказывается, как устранять неполадки с учетной записью Azure.
services: automation
ms.subservice: ''
ms.date: 03/24/2020
ms.topic: troubleshooting
ms.openlocfilehash: 06c15136e9d2fabdf50031c8b4be455cf2f7bbca
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98896585"
---
# <a name="troubleshoot-azure-automation-account-issues"></a>Устранение неполадок с учетной записью службы автоматизации Azure

В этой статье рассматриваются решения проблем, которые могут возникнуть при использовании учетной записи службы автоматизации Azure. Общие сведения об учетных записях службы автоматизации см. в статье [Общие сведения о проверке подлинности учетной записи службы автоматизации Azure](../automation-security-overview.md).

## <a name="scenario-unable-to-register-automation-resource-provider-for-subscriptions"></a><a name="rp-register"></a>Сценарий. Не удается оформить подписки для поставщика ресурсов службы автоматизации

### <a name="issue"></a>Проблема

При работе с функциями управления, например Управлением обновлениями, в учетной записи службы автоматизации возникает следующая ошибка:

```error
Error details: Unable to register Automation Resource Provider for subscriptions:
```

### <a name="cause"></a>Причина

Поставщик ресурсов службы автоматизации не зарегистрирован в подписке.

### <a name="resolution"></a>Решение

Чтобы зарегистрировать поставщик ресурсов службы автоматизации, выполните указанные ниже действия на портале Azure.

1. В браузере перейдите на [портал Azure](https://portal.azure.com).

2. Перейдите на страницу **Подписки** и выберите подписку.   

3. В разделе **Параметры** выберите **Поставщики ресурсов**.

4. В списке поставщиков ресурсов убедитесь в том, что поставщик ресурсов **Microsoft.Automation** зарегистрирован.

5. Если поставщика нет в списке, зарегистрируйте его, как описано в разделе [Устранение ошибок регистрации поставщика ресурсов](../../azure-resource-manager/templates/error-register-resource-provider.md).

## <a name="next-steps"></a>Дальнейшие действия

Если эта статья не помогла вам решить проблему, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Обратитесь за помощью в Твиттер — [@AzureSupport](https://twitter.com/azuresupport). Это официальная учетная запись Microsoft Azure для доступа к ресурсам сообщества Azure (ответы на вопросы, поддержка и консультации специалистов).
* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.

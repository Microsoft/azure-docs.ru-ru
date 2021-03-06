---
title: Оплата счета за Microsoft Azure
description: Практическое руководство по оплате счета на портале Azure. Для оплаты счетов на портале, нужно быть владельцем профиля выставления счетов, участником или менеджером счетов.
keywords: выставление счетов, дата истечения, баланс, оплата сейчас,
author: banders
ms.reviewer: judupont
tags: billing, past due, pay now, bill, invoice, pay
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: how-to
ms.date: 01/13/2021
ms.author: banders
ms.openlocfilehash: 8117f3ca70f51f2d9b11c479803ac33b49f416e7
ms.sourcegitcommit: fc23b4c625f0b26d14a5a6433e8b7b6fb42d868b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2021
ms.locfileid: "98540045"
---
# <a name="how-to-pay-your-bill-for-microsoft-azure"></a>Как оплатить счет за Microsoft Azure

Эта статья предназначена для клиентов с Клиентским соглашением Майкрософт (MCA).

[Проверка доступа к Клиентскому соглашению Майкрософт](#check-access-to-a-microsoft-customer-agreement).

Существует два способа оплаты счета за использование Azure. Вы можете выполнить оплату используя метод оплаты по умолчанию своего профиля выставления счетов или выполнить однократную оплату, которая называется **Платить сейчас**.

Если вы зарегистрировались в Azure через представителя Майкрософт, то в качестве метода оплаты по умолчанию всегда будет использоваться *чек или банковский перевод*.

Если у вас есть кредиты Azure, они автоматически применяются к вашему счету за каждый расчетный период.

## <a name="pay-by-default-payment-method"></a>Оплата методом по умолчанию

Метод оплаты по умолчанию для профиля выставления счетов можно выбрать из оплаты кредитной или дебетовой картой, либо банковским переводом.

### <a name="credit-or-debit-card"></a>Кредитная или дебетовая карта

Если в качестве способа оплаты по умолчанию для профиля выставления счетов используется кредитная или дебетовая карта, оплата автоматически взимается за каждый расчетный период.

Если автоматическая оплата с помощью кредитной или дебетовой карты по какой-либо причине отклоняется, вы можете выполнить одноразовый платеж с кредитной или дебетовой картой на портале Azure используя метод **Платить сейчас**.

Если вы хотите узнать, как изменить метод оплаты по умолчанию на банковский платеж, см. статью [Как оплачивать счета](../manage/pay-by-invoice.md).

### <a name="check-or-wire-transfer"></a>Банковский платеж

Если в профиле выставления счетов для метода оплаты по умолчанию указан банковский платеж, следуйте инструкциям по оплате, указанным в PDF-файле счета.

Кроме того, если счет меньше пороговой суммы для вашей валюты, вы можете выполнить одноразовую оплату на портале Azure с помощью кредитной или дебетовой карты, используя метод **Платить сейчас**. Если сумма счета превышает пороговое значение, счет нельзя оплатить с помощью кредитной или дебетовой карты. Пороговое значение для валюты можно найти на портале Azure после выбора метода **Платить сейчас**.

## <a name="pay-now-in-the-azure-portal"></a>Оплатите сейчас на портале Azure

Чтобы оплатить счета на портале Azure, необходимо быть администратором учетной записи выставления счетов или иметь соответствующие [разрешения MCA](../manage/understand-mca-roles.md). Администратор учетной записи выставления счетов является пользователем, который изначально регистрировался для учетной записи MCA.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выполните поиск по **Управление затратами + выставление счетов**.
1. В меню слева в разделе **Выставление счетов** выберите **Счета**.
1. Если какие-либо из ваших счетов подлежат оплате или просрочены, вы увидите для этого счета синюю ссылку **Платить сейчас**. Щелкните **Платить сейчас**.
1. В окне "Платить сейчас" выберите команду **Выбрать метод оплаты**, чтобы выбрать существующую кредитную карту или добавить новую.
1. После выбора метода оплаты щелкните **Платить сейчас**.

Состояние счета изменится на *оплачен* в течение 24 часов.

## <a name="check-access-to-a-microsoft-customer-agreement"></a>Проверка доступа к Клиентскому соглашению Майкрософт
[!INCLUDE [billing-check-mca](../../../includes/billing-check-mca.md)]

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы получить право на оплату с помощью чека или банковского перевода, ознакомьтесь со статьей [Оплата подписок Azure по счету](../manage/pay-by-invoice.md).
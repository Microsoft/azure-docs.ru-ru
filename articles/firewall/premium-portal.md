---
title: Предварительная версия Azure Firewall Premium Preview на портал Azure
description: Дополнительные сведения о предварительной версии Azure Firewall Premium см. в портал Azure.
author: vhorne
ms.service: firewall
services: firewall
ms.topic: conceptual
ms.date: 02/16/2021
ms.author: victorh
ms.openlocfilehash: 3d56fc73faeb0d48ba7e5b21449c61d6213afa40
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100549835"
---
# <a name="azure-firewall-premium-preview-in-the-azure-portal"></a>Предварительная версия Azure Firewall Premium Preview на портал Azure

> [!IMPORTANT]
> Брандмауэр Azure уровня "Премиум" сейчас находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

 Брандмауэр Azure уровня "Премиум" (предварительная версия) — это брандмауэр следующего поколения с возможностями, требуемыми для работы в строго конфиденциальных и регулируемых средах. Он включает следующие компоненты.

- **Проверка TLS** — расшифровывает исходящий трафик, обрабатывает данные, затем шифрует данные и отправляет их в место назначения.
- **Поставщиков удостоверений** — система обнаружения и предотвращения вторжения в сеть (поставщиков удостоверений) позволяет отслеживать сетевые действия для вредоносных действий, записывать в журнал сведения об этом действии, сообщать о нем и при необходимости пытаться заблокировать его.
- **Фильтрация URL-адресов** . расширяет возможности фильтрации полного доменного имени в брандмауэре Azure, чтобы учесть весь URL-адрес. Например, `www.contoso.com/a/c` вместо `www.contoso.com` .
- **Веб-категории** . Администраторы могут разрешать или запрещать доступ пользователей к категориям веб-сайтов, таким как веб-сайты азартных игр, веб-сайты социальных сетей и др.

Дополнительные сведения см. в статье [Azure Firewall Premium Features](premium-features.md).

## <a name="deploy-the-firewall"></a>Развертывание брандмауэра

Развертывание брандмауэра Azure Premium Preview аналогично развертыванию стандартного брандмауэра Azure:

:::image type="content" source="media/premium-portal/premium-portal.png" alt-text="развертывание портала":::

Для **уровня брандмауэра** выберите **Premium (Предварительная версия)** и для **политики брандмауэра** выберите существующую политику Premium или создайте новую.

## <a name="configure-the-premium-policy"></a>Настройка политики Premium

Настройка политики брандмауэра уровня "Премиум" аналогична настройке стандартной политики брандмауэра. С помощью политики Premium можно настроить функции Premium:

:::image type="content" source="media/premium-portal/premium-policy.png" alt-text="Развертывание политики Premium":::

### <a name="rule-configuration"></a>Конфигурация правила

При настройке правил приложения в политике Premium можно настроить дополнительные функции Premium:

:::image type="content" source="media/premium-portal/premium-application-rule.png" alt-text="Правило Premium":::

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о функциях предварительной версии Azure Firewall Premium см. в статье [развертывание и настройка брандмауэра Azure Premium Preview](premium-deploy.md).
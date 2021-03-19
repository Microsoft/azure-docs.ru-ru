---
title: Настройка программы Bot Protection для брандмауэра веб-приложения Azure (WAF)
description: Узнайте, как настроить программу Bot Protection для брандмауэра веб-приложения (WAF) в шлюзе приложений Azure.
services: web-application-firewall
ms.topic: article
author: vhorne
ms.service: web-application-firewall
ms.date: 10/31/2019
ms.author: victorh
ms.openlocfilehash: 89c863e85d9eab27a47bc1bf7b98cd1c8d89e900
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "73516868"
---
# <a name="configure-bot-protection-for-web-application-firewall-on-azure-application-gateway-preview"></a>Настройка защиты от ботов для брандмауэра веб-приложения Azure в Шлюзе приложений Azure (предварительная версия)

В этой статье показано, как настроить правило защиты Bot в брандмауэре веб-приложения Azure (WAF) для шлюза приложений с помощью портал Azure. 

Вы можете включить управляемый набор правил защиты от ботов, чтобы WAF блокировал и (или) сохранял в журнал запросы от известных вредоносных IP-адресов. Эти IP-адреса передаются из канала Microsoft Threat Intelligence. Intelligent Security Graph поддерживает Microsoft Threat Intelligence и используется во множестве служб, включая Центр безопасности Azure.

> [!NOTE]
> Сейчас набор правил для защиты от ботов предоставляется в режиме общедоступной предварительной версии и поддерживается соглашением об уровне обслуживания для предварительной версии. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Подробные сведения см. на странице  [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) .

## <a name="prerequisites"></a>Предварительные требования

Создайте базовую политику WAF для шлюза приложений, следуя инструкциям, описанным в разделе [Создание политик брандмауэра веб-приложения для шлюза](create-waf-policy-ag.md)приложений.

## <a name="enable-bot-protection-rule-set"></a>Включить набор правил для защиты Bot

1. На странице **Basic** Policy (базовая политика), созданной ранее, в разделе **Параметры** выберите **правила**.  

2. На странице сведения в разделе  **Управление правилами** в раскрывающемся   меню установите флажок для правила защиты Bot и нажмите кнопку **сохранить**.

> [!div class="mx-imgBorder"]
> ![Защита от ботов](../media/bot-protection/bot-protection.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о настраиваемых правилах см. [в статье пользовательские правила для брандмауэра веб-приложения версии 2 в шлюзе приложений Azure](custom-waf-rules-overview.md).
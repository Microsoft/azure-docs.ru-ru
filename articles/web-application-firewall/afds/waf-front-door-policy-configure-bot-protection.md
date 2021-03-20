---
title: Настройка программы Bot Protection для брандмауэра веб-приложения с помощью передней дверцы Azure (Предварительная версия)
description: Узнайте, как настроить правило защиты Bot в брандмауэре веб-приложения Azure (WAF) для использования в качестве передней дверцы с помощью портал Azure.
author: vhorne
ms.service: web-application-firewall
ms.topic: article
services: web-application-firewall
ms.date: 08/21/2019
ms.author: victorh
ms.openlocfilehash: 2357c51f47bcb9bd8bbc6c408cb6d8edbab4d10e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91267012"
---
# <a name="configure-bot-protection-for-web-application-firewall-preview"></a>Настройка программы Bot Protection для брандмауэра веб-приложения (Предварительная версия)
В этой статье показано, как настроить правило защиты Bot в брандмауэре веб-приложения Azure (WAF) для использования в качестве передней дверцы с помощью портал Azure. Правило защиты Bot можно также настроить с помощью интерфейса командной строки, Azure PowerShell или Azure Resource Manager шаблона.

> [!IMPORTANT]
> Сейчас набор правил защиты от ботов предоставляется в общедоступной предварительной версии и поддерживается соглашением об уровне обслуживания для предварительной версии. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.  См. [дополнительные условия использования для предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные условия

Создайте базовую политику WAF для Front двери, следуя инструкциям, описанным в статье [Создание политики WAF для передней дверцы Azure с помощью портал Azure](waf-front-door-create-portal.md).

## <a name="enable-bot-protection-rule-set"></a>Включить набор правил для защиты Bot

На странице " **управляемые правила** " при создании политики брандмауэра веб-приложения сначала найдите раздел " **набор управляемых правил** ", установите флажок перед правилом **Microsoft_BotManager_1 0** в раскрывающемся меню, а затем выберите **Просмотреть и создать**.

   ![Правило защиты Bot](.././media/waf-front-door-configure-bot-protection/botmanager112019.png)

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [ОТСЛЕЖИВАТЬ WAF](waf-front-door-monitor.md).

---
title: Подключение защита идентификации Azure AD данных к Azure Sentinel
description: Узнайте, как выполнять потоковую передачу журналов и оповещений из защита идентификации Azure AD в Azure Sentinel для просмотра панелей мониторинга, создания пользовательских оповещений и улучшения расследования.
author: yelevin
manager: rkarlin
ms.assetid: 91c870e5-2669-437f-9896-ee6c7fe1d51d
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: how-to
ms.date: 06/24/2020
ms.author: yelevin
ms.openlocfilehash: f420e80f10072c440f5401c042d6370a061b1a7f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98632247"
---
# <a name="connect-data-from-azure-active-directory-azure-ad-identity-protection"></a>Подключение данных из защиты идентификации Azure Active Directory (Azure AD)

Вы можете выполнять потоковую передачу журналов из [Защита идентификации Azure AD](../active-directory/identity-protection/overview-identity-protection.md) в метку Azure для потоковой передачи оповещений в Azure Sentinel для просмотра панелей мониторинга, создания пользовательских оповещений и улучшения расследования. Защита идентификации Azure Active Directory предоставляет объединенное представление для пользователей с рисками, обнаружения рисков и уязвимостей, с возможностью немедленного устранения рисков и установки политик для автоматического устранения будущих событий. Служба построена на основе опыта корпорации Майкрософт по защите идентификаторов потребителей и значительной точности от сигнала с более чем 13 000 000 000 журнала в день. 

## <a name="prerequisites"></a>Предварительные условия

- Необходимо иметь [подписку Azure AD Premium P2](https://azure.microsoft.com/pricing/details/active-directory/).
- У вас должен быть пользователь с правами глобального администратора или администратора безопасности.

## <a name="connect-to-azure-ad-identity-protection"></a>Подключение к защита идентификации Azure AD

Если у вас есть подписка Azure AD Premium P2, включается защита идентификации Azure AD. Если какие бы то ни было [политики включены](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md) и создает предупреждения, данные предупреждений можно легко передать в Azure Sentinel.

1. В поле Sentinel Azure выберите **соединители данных** , а затем щелкните плитку **Защита идентификации Azure AD** .

1. Нажмите кнопку **подключить** , чтобы начать потоковую передачу событий Защита идентификации Azure AD события в Azure Sentinel.

1. Чтобы использовать соответствующую схему в Log Analytics для защита идентификации Azure AD предупреждений, выполните поиск по запросу **секуритялерт**.

Если вы хотите протестировать соединитель, вы можете [имитировать обнаружение](../active-directory/identity-protection/howto-identity-protection-simulate-risk.md) , чтобы создать образцы оповещений, которые будут передаваться в поток Azure Sentinel.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить защита идентификации Azure AD к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).

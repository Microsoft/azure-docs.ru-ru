---
title: Увеличение лимита сети
description: Увеличение лимита сети
author: anavinahar
ms.author: anavin
ms.date: 01/23/2020
ms.topic: how-to
ms.assetid: ce37c848-ddd9-46ab-978e-6a1445728a3b
ms.openlocfilehash: e945dec2ce8514c3e4f1edecdecd13c5c43f7c75
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96745457"
---
# <a name="networking-limit-increase"></a>Увеличение лимита сети

Используйте [портал Azure](https://portal.azure.com) , чтобы увеличить квоту сети.

Чтобы просмотреть текущие сведения об использовании и квоте сети в портал Azure, откройте подписку, а затем выберите **использование и квоты**. Можно также использовать следующие параметры для просмотра использования и ограничений сети.

* [Интерфейс командной строки использования](/cli/azure/network#az-network-list-usages)
* [PowerShell](/powershell/module/azurerm.network/get-azurermnetworkusage)
* [API использования сети](/rest/api/virtualnetwork/virtualnetworks/listusage)

Вы можете запросить увеличение с помощью команды **Справка и поддержка** или в области **использование и квоты** на портале.

> [!Note]
> Чтобы изменить размер по умолчанию для префиксов общедоступного **IP-адреса**, в раскрывающемся списке выберите значение **Минимальная длина префикса объединенной IP-адреса** .

## <a name="request-networking-quota-increase-at-subscription-level-using-help--support"></a>Запрос увеличения квоты сети на уровне подписки с помощью справки и поддержки

Следуйте приведенным ниже инструкциям, чтобы создать запрос в службу поддержки, используя **справку и поддержку** в портал Azure.

1. Войдите в [портал Azure](https://portal.azure.com), а затем выберите **Справка и поддержка** в меню портал Azure или выполните поиск и выберите Справка и **Поддержка**.

    ![Справка и поддержка](./media/networking-quota-request/help-plus-support.png)

1. Выберите **Новый запрос в службу поддержки**.

    ![Новый запрос на поддержку](./media/networking-quota-request/new-support-request.png)

1. В качестве **типа проблемы** выберите **пределы службы и подписки (квоты)**.

    ![Раскрывающийся список "ограничения подписки из типа проблемы"](./media/networking-quota-request/select-quota-issue-type.png)

1. Выберите подписку, которая требует увеличенную квоту.

    ![Выберите Новости подписки](./media/networking-quota-request/select-subscription-support-request.png)

1. В поле **тип квоты** выберите **сеть**. Нажмите кнопку **Далее: решения**.

    ![Выберите тип квоты](./media/networking-quota-request/select-quota-type-network.png)

1. В окне **сведения о проблеме** выберите **Укажите сведения** и введите дополнительные сведения, которые помогут обработать запрос.

    ![Укажите сведения](./media/networking-quota-request/provide-details-link.png)

1. На панели **сведения о квоте** выберите модель развертывания, расположение и ресурсы для включения в запрос.

    ![Сведения о квоте DM](./media/networking-quota-request/quota-details-network.png)

1. Введите желаемые ограничения для подписки. Чтобы удалить линию, отмените выбор ресурса в меню **ресурсы** или выберите значок отклонить "x". После ввода квоты для каждого ресурса нажмите кнопку **сохранить и продолжайте** , чтобы продолжить создание запроса на поддержку.

    ![Новые ограничения](./media/networking-quota-request/network-new-limits.png)

## <a name="request-networking-quota-increase-at-subscription-level-using-usages--quotas"></a>Запрос увеличения квоты сети на уровне подписки с использованием использования + квоты

Выполните эти инструкции, чтобы создать запрос в службу поддержки с помощью функции " **использование и квота** " в портал Azure.

1. В https://portal.azure.com найдите и выберите **подписки**.

    ![Подписки](./media/networking-quota-request/search-for-suscriptions.png)

1. Выберите подписку, которая требует увеличенную квоту.

    ![Выбор подписки](./media/networking-quota-request/select-subscription-change-quota.png)

1. Выбор **использования + квоты**

    ![Использование и квоты](./media/networking-quota-request/select-usage-plus-quotas.png)

1. В правом верхнем углу выберите **запросить увеличение**.

    ![Запросить увеличение](./media/networking-quota-request/request-increase-from-subscription.png)

1. Выполните шаги, начиная с шага 3 в разделе [запрос на увеличение квоты сети на уровне подписки](#request-networking-quota-increase-at-subscription-level-using-help--support).

## <a name="about-networking-limits"></a>Ограничения сети

Дополнительные сведения об ограничениях сети см. в [разделе "сети](../../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits) " страницы "ограничения" или на странице "ограничения сети".

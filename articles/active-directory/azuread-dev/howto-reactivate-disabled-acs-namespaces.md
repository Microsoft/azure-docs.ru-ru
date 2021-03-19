---
title: Повторная активация отключенных пространств имен службы контроля доступа Azure (ACS)
description: Найдите и включите пространства имен службы контроля доступа Azure (ACS) и запросите расширение, чтобы они оставались включенными до 4 февраля 2019 г.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: how-to
ms.date: 01/21/2019
ms.author: ryanwi
ms.reviewer: jlu
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 7bb572e9e22519491290e54417f5ca350d6c0b5c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85383723"
---
# <a name="how-to-reactivate-disabled-access-control-service-namespaces"></a>Практическое руководство. Повторная активация отключенных пространств имен службы контроля доступа

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

В ноябре 2017 года мы объявили, что служба контроля доступа Microsoft Azure (ACS), относящаяся к Azure Active Directory (Azure AD), прекратит функционирование 7 ноября 2018 года.

С этого момента мы отправили сообщения электронной почты администраторам подписки ACS о прекращении использования ACS 12 месяцев, 9 месяцах, 6 месяцах, 3 месяцах, 1 месяце, 2 недели, 1 неделю и 1 день до даты выхода 7 ноября 2018.

3 октября 2018 года мы объявили (по электронной почте и в [записи блога](https://azure.microsoft.com/blog/one-month-retirement-notice-access-control-service/)) о предложении расширения для клиентов, которые не могут завершить миграцию до 7 ноября 2018 года. В объявлении также содержались указания по запросам расширения.

## <a name="why-your-namespace-is-disabled"></a>Причины отключения пространства имен

Если вы еще не дали согласие на расширение, пространства имен ACS будут отключены начиная с 7 ноября 2018 года. Возможно, вы уже запросили расширение за 4 февраля 2019 года. В противном случае вы не сможете включить пространство имен с помощью PowerShell.

> [!NOTE]
> Чтобы запустить команды PowerShell и запросить расширение, необходимо иметь права администратора служб или соадминистратора подписки.

## <a name="find-and-enable-your-acs-namespaces"></a>Поиск и включение пространств имен ACS

Чтобы просмотреть список всех пространств имен ACS и повторно активировать те, которые были отключены, можно использовать ACS PowerShell.

1. Загрузка и установка ACS PowerShell
    1. Перейдите в коллекцию PowerShell и загрузите [Acs.Namespaces](https://www.powershellgallery.com/packages/Acs.Namespaces/1.0.2).
    1. Установите модуль:

        ```powershell
        Install-Module -Name Acs.Namespaces
        ```

    1. Получите список всех возможных команд:

        ```powershell
        Get-Command -Module Acs.Namespaces
        ```

        Чтобы получить справку по конкретной команде, выполните следующую команду:

        ```powershell
        Get-Help [Command-Name] -Full
        ```
    
        где `[Command-Name]` — это имя команды ACS.
1. Подключитесь к ACS с помощью командлета **Connect-AcsAccount**. 

    Прежде чем вы сможете выполнить команду, может потребоваться изменить политику выполнения путем выполнения команды **Set-ExecutionPolicy**.
1. Получите список доступных подписок Azure с помощью командлета **Get-AcsSubscription**.
1. Получите список пространств имен ACS с помощью командлета **Get-AcsNamespace**.
1. Удостоверьтесь, что пространства имен отключены. Для этого убедитесь, что для `State` задано значение `Disabled`.

    [![Убедитесь, что пространства имен отключены.](./media/howto-reactivate-disabled-acs-namespaces/confirm-disabled-namespace.png)](./media/howto-reactivate-disabled-acs-namespaces/confirm-disabled-namespace.png#lightbox)

    Кроме того, можно использовать `nslookup {your-namespace}.accesscontrol.windows.net`, чтобы убедиться, что домен по-прежнему активен.

1. Включите пространства имен ACS с помощью командлета **Enable-AcsNamespace**.

    После включения пространств имен можно запросить расширение, чтобы их снова не отключили до 4 февраля 2019 года. После этой даты все запросы к ACS завершатся ошибкой.

## <a name="request-an-extension"></a>Запрос расширения

Начиная с 21 января 2019 года, мы получаем запросы расширения.

К 4 февраля 2019 года мы начнем отключать пространства имен для пользователей, запросивших расширения. Вы все еще сможете повторно включить пространства имен с помощью PowerShell, но они снова отключатся спустя 48 часов.

После 4 марта 2019 года пользователи больше не смогут повторно включать пространства имен с помощью PowerShell.

Дополнительные расширения больше не будут автоматически утверждаться. Если вам нужно дополнительное время для миграции, свяжитесь с [поддержкой Azure](https://portal.azure.com/#create/Microsoft.Support), чтобы получить более детальную временную шкалу миграции.

### <a name="to-request-an-extension"></a>Запрос расширения

1. Войдите в портал Azure и создайте [новый запрос в службу поддержки](https://portal.azure.com/#create/Microsoft.Support).
1. Заполните новую форму запроса на поддержку, как показано в следующем примере.

    | Поле запроса на поддержку | Значение |
    |-----------------------|--------------------|
    | **Тип проблемы** | `Technical` |
    | **Подписка** | Задается вашей подписке |
    | **Служба** | `All services` |
    | **Ресурс** | `General question/Resource not available` |
    | **Тип проблемы** | `ACS to SAS Migration` |
    | **Тема** | Описание проблемы |

   ![Показывает пример нового запроса на техническую поддержку](./media/howto-reactivate-disabled-acs-namespaces/new-technical-support-request.png)

<!--

1. Navigate to your ACS namespace's management portal by going to `https://{your-namespace}.accesscontrol.windows.net`.
1. Select the **Read Terms** button to read the [updated Terms of Use](https://azure.microsoft.com/support/legal/access-control/), which will direct you to a page with the updated Terms of Use.

    [![Select the Read Terms button](./media/howto-reactivate-disabled-acs-namespaces/read-terms-button-expanded.png)](./media/howto-reactivate-disabled-acs-namespaces/read-terms-button-expanded.png#lightbox)

1. Select **Request Extension** on the banner at the top of the page. The button will only be enabled after you read the [updated Terms of Use](https://azure.microsoft.com/support/legal/access-control/).

    [![Select the Request Extension button](./media/howto-reactivate-disabled-acs-namespaces/request-extension-button-expanded.png)](./media/howto-reactivate-disabled-acs-namespaces/request-extension-button-expanded.png#lightbox)

1. After the extension request is registered, the page will refresh with a new banner at the top of the page.

    [![Updated page with refreshed banner](./media/howto-reactivate-disabled-acs-namespaces/updated-banner-expanded.png)](./media/howto-reactivate-disabled-acs-namespaces/updated-banner-expanded.png#lightbox)
-->

## <a name="help-and-support"></a>Справка и поддержка

- Если вы столкнетесь с какими-либо проблемами после выполнения инструкций из этого пошагового руководства, обратитесь в [службу поддержки Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).
- Свяжитесь с нами по адресу acsfeedback@microsoft.com, если у вас есть вопросы или предложения, связанные с прекращением поддержки ACS.

## <a name="next-steps"></a>Дальнейшие действия

- Просмотрите сведения о прекращении поддержки ACS в статье [Руководство. Перенос из службы контроля доступа Azure](active-directory-acs-migration.md).

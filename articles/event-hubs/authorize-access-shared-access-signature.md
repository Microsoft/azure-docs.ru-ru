---
title: Авторизация доступа с помощью подписанного URL-доступа в концентраторах событий Azure
description: В этой статье содержатся сведения о предоставлении доступа к ресурсам концентраторов событий Azure с помощью подписанных URL-адресов (SAS).
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 6a2d7385f82864e8d378055333377fb9c3f73c19
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85323118"
---
# <a name="authorizing-access-to-event-hubs-resources-using-shared-access-signatures"></a>Авторизация доступа к ресурсам концентраторов событий с помощью подписанных URL
Подписанный URL-адрес (SAS) предоставляет способ предоставления ограниченного доступа к ресурсам в пространстве имен концентраторов событий. SAS защищает доступ к ресурсам концентраторов событий на основе правил авторизации. Эти правила настраиваются либо в пространстве имен, либо в сущности (в концентраторе событий или разделе). В этой статье приводится обзор модели SAS и рассматриваются рекомендации по использованию SAS.

## <a name="what-are-shared-access-signatures"></a>Что такое подписанный URL?
Подписанный URL-адрес (SAS) предоставляет делегированный доступ к ресурсам концентраторов событий на основе правил авторизации. Право авторизации имеет имя, связано с определенными правами и содержит пару криптографических ключей. Имя и ключ правила используются клиентами концентраторов событий или в собственном коде для создания маркеров SAS. Затем клиент может передать маркер в концентраторы событий, чтобы подтвердить авторизацию для запрошенной операции.

SAS — это механизм авторизации на основе утверждений, использующий простые токены. При использовании SAS ключи никогда не передаются по сети. Ключи криптографически подписывают сведения, которые впоследствии будут проверены службой. SAS можно использовать так же, как и имя пользователя и пароль, где клиент немедленно получает имя и соответствующий ключ правила авторизации. SAS можно использовать аналогично Федеративной модели безопасности, где клиент получает маркер доступа с ограниченным временем и подписанный токен из службы маркеров безопасности, не прибегая к ключу подписывания.

> [!NOTE]
> Концентраторы событий Azure поддерживают авторизацию для ресурсов концентраторов событий с помощью Azure Active Directory (Azure AD). Авторизация пользователей или приложений с помощью токена OAuth 2,0, возвращенного Azure AD, обеспечивает более высокую безопасность и простоту использования подписей общего доступа (SAS). В Azure AD нет необходимости хранить маркеры в коде и угрожать потенциальным уязвимостям безопасности.
>
> Корпорация Майкрософт рекомендует использовать Azure AD вместе с приложениями концентраторов событий Azure, когда это возможно. Дополнительные сведения см. [в статье авторизация доступа к ресурсам концентраторов событий Azure с помощью Azure Active Directory](authorize-access-azure-active-directory.md).

> [!IMPORTANT]
> Токены SAS являются критически важными для защиты ресурсов. При обеспечении гранулярности SAS предоставляет клиентам доступ к ресурсам концентраторов событий. Они не должны предоставлять общий доступ. При совместном использовании, если это необходимо для устранения неполадок, рекомендуется использовать уменьшенную версию любого файла журнала или удалить маркеры SAS (если они есть) из файлов журнала и убедиться, что снимки экрана не содержат сведения SAS.

## <a name="shared-access-authorization-policies"></a>Политики авторизации общего доступа
Каждое пространство имен концентраторов событий и каждая сущность концентраторов событий (экземпляр концентратора событий или раздел Kafka) имеют политику авторизации общего доступа, состоящие из правил. Политика на уровне пространства имен применяется ко всем сущностям в пространстве имен независимо от настройки их политики.
Для каждого правила политики авторизации необходимо выбрать три элемента информации: имя, область и права. Имя является уникальным именем в этой области. Область — это URI рассматриваемого ресурса. Для пространства имен концентраторов событий областью является полное доменное имя (FQDN), например `https://<yournamespace>.servicebus.windows.net/` .

Права, предоставляемые правилом политики, могут быть комбинацией:
- **Send** — дает право на отправку сообщений в сущность.
- **Listen** — дает право на прослушивание или получение сущности.
- **Управление** — предоставляет право на управление топологией пространства имен, включая создание и удаление сущностей.

> [!NOTE]
> Право " **Управление** " включает права **отправки** и **прослушивания** .

Пространство имен или политика сущности могут содержать до 12 правил авторизации общего доступа, предоставляя пространство для трех наборов правил, каждое из которых охватывает основные права, и сочетание отправки и прослушивания. Это ограничение подчеркивает, что хранилище политик SAS не предназначено для пользователя или хранилища учетных записей служб. Если приложению требуется предоставить доступ к ресурсам концентраторов событий на основе удостоверений пользователя или службы, он должен реализовать службу маркеров безопасности, которая выдает маркеры SAS после проверки подлинности и доступа.

Правилу авторизации назначается **первичный ключ** и **вторичный ключ**. Эти ключи являются криптографически надежными ключами. Не потеряйте их или утечек. Они всегда доступны в портал Azure. Вы можете использовать любой из созданных ключей и создавать их повторно в любое время. Если вы повторно создадите или измените ключ в политике, все ранее выданные маркеры на его основе моментально станут недействительными. Тем не менее текущие подключения, созданные на основе этих маркеров, продолжат работать до истечения срока их действия.

При создании пространства имен концентраторов событий правило политики с именем **RootManageSharedAccessKey** автоматически создается для пространства имен. Эта политика имеет разрешения на **Управление** для всего пространства имен. Рекомендуется рассматривать это правило как административную корневую учетную запись и не использовать ее в приложении. Вы можете создать дополнительные правила политики на вкладке **Настройка** для пространства имен на портале с помощью PowerShell или Azure CLI.

## <a name="best-practices-when-using-sas"></a>Рекомендации по использованию SAS
При использовании подписей общего доступа в приложениях необходимо иметь в виду две потенциальные проблемы:

- При утечке SAS его может использовать любой пользователь, который ее получает, что может повредить ресурсы концентраторов событий.
- Если срок действия SAS, предоставленного клиентскому приложению, истекает, и приложению не удается получить новый SAS из службы, это может препятствовать работе приложения.

Следующие рекомендации по использованию подписанных URL-адресов помогут нейтрализовать эти риски:

- **Клиенты автоматически возобновляют SAS при необходимости**: клиенты должны обновлять SAS хорошо до истечения срока действия, чтобы допустить время повторных попыток, если служба, ПРЕДоставляющая SAS, недоступна. Если ваша SAS предназначена для небольшого числа немедленных операций, которые должны быть выполнены в течение срока действия, это может оказаться ненужным, так как обновление SAS не ожидается. Однако если имеется клиент, регулярно выполняющий запросы через подпись, то возможность истечения срока действия следует принять во внимание. Основное внимание заключается в том, чтобы сбалансировать необходимость кратковременного использования SAS (как было сказано выше) с целью убедиться, что клиент запрашивает продление на раннем этапе (чтобы избежать сбоев из-за истечения срока действия SAS до успешного продления).
- Будьте **внимательны с временем начала SAS**: Если задать время начала для SAS в **настоящее** время, то из-за неравномерного распределения (разницы в текущем времени в зависимости от разных компьютеров) сбои могут наблюдаться периодически в течение первых нескольких минут. В общем, устанавливайте время начала действия на 15 минут или более в прошлом времени. Или вообще не устанавливайте его, что сделает его действительным сразу во всех случаях. То же самое относится и к времени окончания срока действия. Помните, что в любом направлении при любом запросе может возникнуть до 15 минут наклона часов. 
- Не **относиться к ресурсу, к которому будет осуществляться доступ**: рекомендуется предоставить пользователю минимально необходимые привилегии. Если пользователю требуется только доступ для чтения к отдельной сущности, предоставьте доступ на чтение к этой сущности, а не доступ на чтение, запись и удаление для всех сущностей. Кроме того, это позволяет уменьшить ущерб при компрометации SAS, так как SAS имеет меньше энергии на руках злоумышленника.
- **Не всегда используйте SAS**. иногда риски, связанные с определенной операцией с концентраторами событий, перевешивают преимущества SAS. Для таких операций Создайте службу среднего уровня, которая записывает данные в концентраторы событий после проверки бизнес-правил, проверки подлинности и аудита.
- **Всегда использовать HTTPS**: всегда используйте HTTPS для создания или распространения SAS. Если ПОДПИСАНный URL-адрес передается по протоколу HTTP и перехвачен, злоумышленник, выполняющий присоединение "злоумышленник в середине", может читать SAS, а затем использовать его так же, как и тот, что и может привести к повреждению конфиденциальных данных или разрешив повреждение данных злоумышленником.

## <a name="conclusion"></a>Заключение
Подписанные URL можно использовать для предоставления клиентам ограниченных разрешений на доступ к ресурсам концентраторов событий. Они являются жизненно важной частью модели безопасности для любого приложения, использующего концентраторы событий Azure. Если следовать рекомендациям, приведенным в этой статье, вы можете использовать SAS для предоставления большей гибкости доступа к ресурсам без ущерба для безопасности приложения.

## <a name="next-steps"></a>Дальнейшие действия
См. следующие статьи: 

- [Проверка подлинности запросов к концентраторам событий Azure из приложения с помощью Azure Active Directory](authenticate-application.md)
- [Проверка подлинности управляемого удостоверения с Azure Active Directory для доступа к ресурсам концентраторов событий](authenticate-managed-identity.md)
- [Проверка подлинности запросов к концентраторам событий Azure с помощью подписанных URL](authenticate-shared-access-signature.md)
- [Авторизация доступа к ресурсам концентраторов событий с помощью Azure Active Directory](authorize-access-azure-active-directory.md)



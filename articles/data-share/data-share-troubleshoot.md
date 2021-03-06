---
title: Устранение неполадок в Azure Data Share
description: Узнайте, как устранять неполадки с приглашениями и ошибками при создании или получении общих папок данных в общем ресурсе данных Azure.
services: data-share
author: jifems
ms.author: jife
ms.service: data-share
ms.topic: troubleshooting
ms.date: 12/16/2020
ms.openlocfilehash: 3aa1c0b8579bd37d2bb51cbde70997131c696813
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97964513"
---
# <a name="troubleshoot-common-problems-in-azure-data-share"></a>Устранение распространенных неполадок в общей папке данных Azure 

В этой статье объясняется, как устранять распространенные проблемы в общем ресурсе данных Azure. 

## <a name="azure-data-share-invitations"></a>Приглашения Azure Data Share 

В некоторых случаях, когда новые пользователи выбирают команду **принять приглашение** в приглашении по электронной почте, они могут увидеть пустой список приглашений. 

:::image type="content" source="media/no-invites.png" alt-text="Снимок экрана, показывающий пустой список приглашений.":::

Эта проблема может быть вызвана одной из следующих причин:

* **Служба общего ресурса данных Azure не зарегистрирована в качестве поставщика ресурсов для любой подписки Azure в клиенте Azure.** Эта проблема возникает, если у клиента Azure нет общего ресурса данных. 

    Когда вы создаете ресурс Azure Data Share, он автоматически регистрирует поставщика ресурсов в подписке Azure. Службу общих ресурсов данных можно зарегистрировать вручную, выполнив следующие действия. Для выполнения этих действий требуется [роль участника](../role-based-access-control/built-in-roles.md#contributor) для подписки Azure. 

    1. На портале Azure перейдите к разделу **Подписки**.
    1. Выберите подписку, которую вы хотите использовать для создания общего ресурса данных Azure.
    1. Выберите **поставщики ресурсов**.
    1. Найдите **файл Microsoft.**
    1. Выберите **Зарегистрировать**.

* **Приглашение отправляется в псевдоним электронной почты вместо адреса электронной почты Azure для входа.** Если вы уже зарегистрировали службу общего доступа к данным Azure или создали ресурс общего ресурса в клиенте Azure, но вы по-прежнему не видите это приглашение, псевдоним электронной почты может быть указан в качестве получателя. Обратитесь к поставщику данных и убедитесь, что приглашение будет отправлено на ваш адрес электронной почты для входа в Azure, а не на ваш псевдоним электронной почты.

* **Приглашение уже принято.** Ссылка в сообщении электронной почты переводит вас на страницу **приглашения для обмена данными** в портал Azure. На этой странице перечислены только ожидающие приглашения. Принятые приглашения не отображаются на странице. Чтобы просмотреть полученные общие ресурсы и настроить целевой параметр кластера Azure обозреватель данных, перейдите к ресурсу общего доступа к данным, который использовался для принятия приглашения.

## <a name="creating-and-receiving-shares"></a>Создание и получение общих ресурсов

При создании новой общей папки, добавления наборов данных или наборов данных карт могут появляться следующие ошибки:

* Не удалось добавить наборы данных.
* Не удалось отобразить наборы данных.
* Не удалось предоставить доступ к общему ресурсу данных x к y.
* У вас нет необходимых разрешений для x.
* Не удалось добавить разрешения на запись для общей учетной записи Azure Data Share в один или несколько выбранных ресурсов.

Если у вас недостаточно разрешений для хранилища данных Azure, может появиться одна из этих ошибок. Дополнительные сведения см. в разделе [Роли и требования](concepts-roles-permissions.md). 

Необходимо разрешение на запись, чтобы предоставить общий доступ или получить данные из хранилища данных Azure. Это разрешение обычно является частью роли участника. 

Если вы впервые собираете данные или получаете данные из хранилища данных Azure, вам также потребуется разрешение *Microsoft. Authorization, роли/запись* . Это разрешение обычно является частью роли владельца. Даже если вы создали ресурс хранилища данных Azure, вы не обязательно являетесь владельцем ресурса. 

При наличии соответствующих разрешений Служба общих ресурсов Azure автоматически разрешает управляемому удостоверению общего ресурса для доступа к хранилищу данных. Это может занять несколько минут. Если у вас возникли проблемы из-за этой задержки, повторите попытку через несколько минут.

Для совместного использования на основе SQL требуются дополнительные разрешения. Дополнительные сведения о предварительных требованиях см. [в разделе Общий доступ из источников SQL](how-to-share-from-sql.md).

## <a name="snapshots"></a>Моментальные снимки
Создание моментального снимка может завершиться ошибкой по различным причинам. Откройте подробное сообщение об ошибке, выбрав время начала создания моментального снимка, а затем состояние каждого набора данных. 

Обычно моментальные снимки не удается выполнить по следующим причинам:

* Общая папка данных не имеет разрешения на чтение из исходного хранилища данных или на запись в целевое хранилище данных. Дополнительные сведения см. в разделе [Роли и требования](concepts-roles-permissions.md). Если вы создаете моментальный снимок в первый раз, то для получения доступа к хранилищу данных Azure может потребоваться несколько минут. Через несколько минут повторите попытку.
* Подключение общего доступа к данным к исходному хранилищу данных или целевому хранилищу данных блокируется брандмауэром.
* Удален общий набор данных, исходное хранилище данных или целевое хранилище данных.

Для учетных записей хранения моментальный снимок может завершиться ошибкой, так как файл обновляется в источнике во время создания моментального снимка. В результате в целевом объекте может появиться 0-байтовый файл. После обновления в источнике моментальные снимки должны быть выполнены.

Для источников SQL может произойти сбой моментального снимка по следующим причинам:

* Исходный скрипт SQL или целевой скрипт SQL, предоставляющий разрешение на доступ к данным, не запущен. Для базы данных SQL Azure или Azure синапсе Analytics (ранее — хранилище данных SQL Azure) сценарий выполняется с использованием проверки подлинности SQL, а не Azure Active Directory проверки подлинности.  
* Исходное хранилище данных или целевое хранилище данных SQL приостановлены.
* Процесс создания моментального снимка или целевое хранилище данных не поддерживает типы данных SQL. Дополнительные сведения см. [в разделе Общий доступ из источников SQL](how-to-share-from-sql.md#supported-data-types).
* Исходное хранилище данных или целевое хранилище данных SQL заблокированы другими процессами. Общий ресурс данных Azure не блокирует эти хранилища данных. Но существующие блокировки для этих хранилищ данных могут привести к сбою создания моментального снимка.
* На целевую таблицу SQL ссылается ограничение внешнего ключа. Если во время создания моментального снимка целевая таблица имеет то же имя, что и таблица в источнике данных, то общая папка данных Azure удаляет таблицу и создает новую таблицу. Если на целевую таблицу SQL ссылается ограничение FOREIGN KEY, таблица не может быть удалена.
* Создается целевой CSV-файл, но данные не могут быть прочитаны в Excel. Эта проблема может возникать, если исходная таблица SQL содержит данные, содержащие символы, отличные от английского. В Excel откройте вкладку **Получение данных** и выберите CSV-файл. Выберите источник файла **65001: Юникод (UTF-8)**, а затем загрузите данные.

## <a name="updated-snapshot-schedules"></a>Обновленные расписания моментальных снимков
После того как поставщик данных обновит расписание моментальных снимков для отправленного общего ресурса, потребитель данных должен отключить предыдущее расписание моментальных снимков. Затем включите Обновленное расписание моментальных снимков для полученного общего ресурса. 

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как начать совместное использование данных, перейдите к учебнику по работе с [данными общего доступа](share-your-data.md) . 

Чтобы узнать, как получать данные, перейдите к руководству по [принятию и получению данных](subscribe-to-data-share.md) .
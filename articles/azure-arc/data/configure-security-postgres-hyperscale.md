---
title: Настройка безопасности для группы серверов Гипермасштабирования PostgreSQL с поддержкой Azure Arc
description: Настройка безопасности для группы серверов Гипермасштабирования PostgreSQL с поддержкой Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: d6e27fddceb69efbb2c1697c09ee9b61d7f38ee4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101687980"
---
# <a name="configure-security-for-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Настройка безопасности для группы серверов Гипермасштабирования PostgreSQL с поддержкой Azure Arc

В этом документе описаны различные аспекты, связанные с безопасностью группы серверов.
- Шифрование при хранении
- Управление пользователями
   - Общие перспективы
   - Изменение пароля администратора _postgres_
- Аудит

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="encryption-at-rest"></a>Шифрование при хранении
Вы можете реализовать шифрование неактивных данных, зашифровать диски, на которых хранятся базы данных, или с помощью функций базы данных, чтобы зашифровать вставляемые или обновляемые данные.

### <a name="hardware-linux-host-volume-encryption"></a>Оборудование: шифрование тома узла Linux
Реализуйте шифрование системных данных для защиты любых данных, которые находятся на дисках, используемых программой установки служб данных, включенной в службу "Дуга Azure". Дополнительные сведения об этом разделе:
- [Шифрование неактивных данных](https://wiki.archlinux.org/index.php/Data-at-rest_encryption) в Linux в целом 
- Шифрование дисков с помощью `cryptsetup` команды LUKS Encrypt (Linux) ( https://www.cyberciti.biz/security/howto-linux-hard-disk-encryption-with-luks-cryptsetup-command/) в частности, поскольку службы данных, поддерживающие службу Arc Azure, работают в предоставляемой вами физической инфраструктуре, вы оплачиваете безопасность инфраструктуры.

### <a name="software-use-the-postgresql-pgcrypto-extension-in-your-server-group"></a>Программное обеспечение: используйте `pgcrypto` расширение PostgreSQL в группе серверов.
Помимо шифрования дисков, используемых для размещения настройки дуги Azure, можно настроить PostgreSQL группу серверов в службе "Дуга Azure", чтобы предоставить механизмы, которые приложения могут использовать для шифрования данных в ваших базах данных. `pgcrypto`Расширение является частью `contrib` расширений postgres и доступно в вашей службе "Дуга Azure" PostgreSQL "Масштабируемая группа серверов". Сведения о расширении можно найти `pgcrypto` [здесь](https://www.postgresql.org/docs/current/pgcrypto.html).
В сводке, используя следующие команды, вы включаете расширение, создаете его и используете.


#### <a name="create-the-pgcrypto-extension"></a>Создание `pgcrypto` расширения
Подключитесь к группе серверов с помощью выбранного клиентского средства и выполните стандартный запрос PostgreSQL:
```console
CREATE EXTENSION pgcrypto;
```

> Сведения о подключении см. [здесь](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md) .

#### <a name="verify-the-list-the-extensions-ready-to-use-in-your-server-group"></a>Проверьте список расширений, готовых к использованию в группе серверов.
Чтобы убедиться, что `pgcrypto` расширение готово к использованию, перечислите расширения, доступные в группе серверов.
Подключитесь к группе серверов с помощью выбранного клиентского средства и выполните стандартный запрос PostgreSQL:
```console
select * from pg_extension;
```
`pgcrypto`Если вы включили и создали его с помощью команд, указанных выше.

#### <a name="use-the-pgcrypto-extension"></a>Использование `pgcrypto` расширения
Теперь можно настроить код приложений таким образом, чтобы они использовали любую из функций, предлагаемых `pgcrypto` :
- Общие функции хэширования
- Функции хэширования паролей
- Функции шифрования PGP
- Необработанные функции шифрования
- Функции случайного выбора данных

Например, для создания хэш-значений. Выполните приведенную ниже команду.

```console
Select crypt('Les sanglots longs des violons de l_automne', gen_salt('md5'));
```

Возвращает следующий хэш:

```console
              crypt
------------------------------------
 $1$/9ACBYOV$z52PAGjQ5WTU9xvEECBNv/   
```

Или, например:

```console
select hmac('Les sanglots longs des violons de l_automne', 'md5', 'sha256');
```

Возвращает следующий хэш:

```console
                                hmac
--------------------------------------------------------------------
 \xd4e4790b69d2cc8dbce3385ee63272bc7760f1603640bb211a7b864e695570c5
```

Например, для хранения зашифрованных данных, таких как пароль:

Предположим, что мои приложения хранят секреты в следующей таблице:

```console
create table mysecrets(USERid int, USERname char(255), USERpassword char(512));
```

И я шифруя пароль при создании пользователя:

```console
insert into mysecrets values (1, 'Me', crypt('MySecretPasswrod', gen_salt('md5')));
```

Теперь я вижу, что пароль зашифрован:

```console
select * from mysecrets;
```

Выходные данные:

- USERid: 1
- USERname: Me
- USERpassword: $1 $ Uc7jzZOp $ NTfcGo7F10zGOkXOwjHy31

Когда я подключаюсь к моему приложению и передаем пароль, он будет искать в `mysecrets` таблице и будет возвращать имя пользователя, если существует совпадение между паролем, предоставленным приложению, и паролями, хранящимися в таблице. Пример:

- Я передаем неправильный пароль:
   ```console
   select USERname from mysecrets where (USERpassword = crypt('WrongPassword', USERpassword));
   ```

   Выходные данные 

   ```returns
    USERname
   ---------
   (0 rows)
   ```
- Я передаем правильный пароль:

   ```console
   select USERname from mysecrets where (USERpassword = crypt('MySecretPasswrod', USERpassword));
   ``` 

   Выходные данные:

   ```output
    USERname
   ---------
   Me
   (1 row)
   ```

В этом небольшом примере показано, как можно зашифровать неактивные данные (хранить зашифрованные данные) в PostgreSQL с помощью расширения postgres, `pgcrypto` а приложения могут использовать функции, предложенные `pgcrypto` для управления этими зашифрованными данными.

## <a name="user-management"></a>Управление пользователями
### <a name="general-perspectives"></a>Общие перспективы
Вы можете использовать стандартный postgres способ создания пользователей или ролей. Однако при этом эти артефакты будут доступны только для роли координатора. На этапе предварительной версии эти пользователи и роли не смогут получить доступ к данным, которые распространяются за пределы узла координатора и рабочих узлов группы серверов. Причина в том, что в предварительной версии определение пользователя не реплицируется на рабочие узлы.

### <a name="change-the-password-of-the-_postgres_-administrative-user"></a>Изменение пароля администратора _postgres_
В PostgreSQL для службы "Дуга Azure" включена стандартная postgres административный пользователь _postgres_ , для которого задается пароль при создании группы серверов.
Общий формат команды для изменения пароля:
```console
azdata arc postgres server edit --name <server group name> --admin-password
```

Где `--admin-password` — это логическое значение, связанное с присутствием значения в переменной среды **сеанса** AZDATA_PASSWORD.
Если переменная среды **сеанса** AZDATA_PASSWORD существует и имеет значение, выполнение приведенной выше команды установит для пароля пользователя postgres значение этой переменной среды.

Если переменная среды **сеанса** AZDATA_PASSWORD существует, но не имеет значения или переменная среды AZDATA_PASSWORD **сеанса** не существует, то при выполнении указанной выше команды пользователю будет предложено ввести пароль в интерактивном режиме.

#### <a name="change-the-password-of-the-postgres-administrative-user-in-an-interactive-way"></a>Изменение пароля пользователя postgres с помощью интерактивного способа

1. Удалите AZDATA_PASSWORD переменную среды **сеанса** или удалите ее значение.
2. Выполните приведенную ниже команду.
   ```console
   azdata arc postgres server edit --name <server group name> --admin-password
   ```
   Например:
   ```console
   azdata arc postgres server edit -n postgres01 --admin-password
   ```
   Вам будет предложено ввести пароль и подтвердить его:
   ```console
   Postgres Server password:
   Confirm Postgres Server password:
   ```
   При обновлении пароля выходные данные команды показывают:
   ```console
   Updating password
   Updating postgres01 in namespace `arc`
   postgres01 is Ready
   ```
   
#### <a name="change-the-password-of-the-postgres-administrative-user-using-the-azdata_password-session-environment-variable"></a>Измените пароль администратора postgres с помощью переменной среды **сеанса** AZDATA_PASSWORD.
1. Задайте для переменной среды **сеанса** AZDATA_PASSWORD значение, которое вы хотите использовать для пароля.
2. Выполните команду:
   ```console
   azdata arc postgres server edit --name <server group name> --admin-password
   ```
   Например:
   ```console
   azdata arc postgres server edit -n postgres01 --admin-password
   ```
   
   При обновлении пароля выходные данные команды показывают:
   ```console
   Updating password
   Updating postgres01 in namespace `arc`
   postgres01 is Ready
   ```

> [!NOTE]
> Чтобы проверить, существует ли переменная среды сеанса AZDATA_PASSWORD и какое значение она имеет, выполните:
> - На клиенте Linux:
> ```console
> printenv AZDATA_PASSWORD
> ```
>
> - В клиенте Windows с помощью PowerShell:
> ```console
> echo $env:AZDATA_PASSWORD
> ```

## <a name="audit"></a>Аудит

Для сценариев аудита Настройте группу серверов для использования `pgaudit` расширений postgres. Дополнительные сведения `pgaudit` см. в разделе [ `pgAudit` проект GitHub](https://github.com/pgaudit/pgaudit/blob/master/README.md). Чтобы включить `pgaudit` расширение в группе серверов, [Используйте расширения PostgreSQL](using-extensions-in-postgresql-hyperscale-server-group.md).


## <a name="next-steps"></a>Дальнейшие действия
- См. [ `pgcrypto` расширение](https://www.postgresql.org/docs/current/pgcrypto.html)
- См. раздел [Использование расширений PostgreSQL](using-extensions-in-postgresql-hyperscale-server-group.md) .


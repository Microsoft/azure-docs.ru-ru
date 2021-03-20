---
title: Использование Azure Active Directory. База данных Azure для PostgreSQL. Отдельный сервер
description: Узнайте, как настроить аутентификацию Azure Active Directory (AAD) с помощью Базы данных Azure для PostgreSQL. Отдельный сервер
author: lfittl
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 07/23/2020
ms.openlocfilehash: 178c339f6f47569160a9a748794678c610f35734
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87171636"
---
# <a name="use-azure-active-directory-for-authentication-with-postgresql"></a>Использование Azure Active Directory для проверки подлинности с помощью PostgreSQL

Эта статья поможет вам настроить доступ Azure Active Directory к Базе данных Azure для PostgreSQL и выполнить подключение с помощью маркера Azure AD.

## <a name="setting-the-azure-ad-admin-user"></a>Настройка администратора Azure AD

Только администраторы Azure AD могут создавать или включать пользователей для аутентификации на основе Azure AD. Не рекомендуется использовать аккаунт администратора Azure AD для выполнения обычных операций с базами данных, поскольку он обладает повышенными правами (например, CREATEDB).

Чтобы настроить администратора Azure AD (вы можете взять пользователя или группу), выполните следующие действия.

1. На портале Azure выберите экземпляр Базы данных Azure для PostgreSQL, который необходимо включить для Azure AD.
2. В разделе "Параметры" выберите "Администратор Active Directory",

![установите параметр "Администратор Azure AD".][2]

3. Выберите действительного пользователя Azure AD в арендаторе клиента, который будет администратором Azure AD.

> [!IMPORTANT]
> При настройке администратора в Базу данных Azure для сервера PostgreSQL добавляется новый пользователь со всеми разрешениями администратора. У администратора Azure AD в Базе данных Azure для PostgreSQL будет роль `azure_ad_admin`.

Для каждого сервера PostgreSQL можно создать только одного администратора Azure AD. Выбор другого администратора приведет к перезаписи имеющегося администратора Azure AD, настроенного для сервера. Вместо отдельного пользователя вы можете указать группу Azure AD, чтобы у вас было несколько администраторов. Обратите внимание, что для выполнения действий, связанных с администрированием, вам нужно будет войти с помощью имени группы.

## <a name="connecting-to-azure-database-for-postgresql-using-azure-ad"></a>Подключение к Базе данных Azure для PostgreSQL с помощью Azure AD

На следующей общей схеме представлен рабочий процесс использования аутентификации Azure AD с Базой данных Azure для PostgreSQL:

![поток аутентификации.][1]

Мы разработали интеграцию Azure AD для работы с распространенными инструментами PostgreSQL (например, psql), которые не поддерживают Azure AD и поддерживают указание имени пользователя и пароля только при подключении к PostgreSQL. Мы передаем маркер проверки подлинности Azure AD в качестве пароля, как показано на приведенном выше рисунке.

В настоящее время мы протестировали следующие клиенты:

- командную строку psql (для передачи маркера проверки подлинности используйте переменную PGPASSWORD, см. ниже);
- Azure Data Studio (использующее расширение PostgreSQL);
- другие клиенты на основе libpq (например, обычные исполняющие среды и ORM).

> [!NOTE]
> Обратите внимание, что использование маркера проверки подлинности Azure AD с pgAdmin в данный момент не поддерживается из-за жестко запрограммированного ограничения в 256 символов на длину пароля (длина маркера превышает это значение).

Ниже приведены шаги, которые пользователь или приложение должны будут выполнить для аутентификации в Azure AD.

### <a name="prerequisites"></a>Предварительные требования

Вы можете следовать указаниям в Azure Cloud Shell, виртуальной машине Azure или на локальном компьютере. Убедитесь, что у вас [установлен Azure CLI](/cli/azure/install-azure-cli).

### <a name="step-1-authenticate-with-azure-ad"></a>Шаг 1. Аутентификация с помощью Azure AD

Начните с проверки подлинности в Azure AD с помощью средства Azure CLI. Этот шаг не требуется в Azure Cloud Shell.

```
az login
```

Команда запустит окно браузера на странице аутентификации Azure AD. Для этого нужно указать ИД пользователя Azure AD и пароль.

### <a name="step-2-retrieve-azure-ad-access-token"></a>Шаг 2. Получение маркера доступа Azure AD

Вызовите инструмент Azure CLI, чтобы получить маркер доступа для пользователя Azure AD, прошедшего аутентификацию, из шага 1, необходимый для доступа к Базе данных Azure для PostgreSQL.

Пример (для общедоступного облака)

```azurecli-interactive
az account get-access-token --resource https://ossrdbms-aad.database.windows.net
```

Приведенное выше значение ресурса должно быть указано точно так, как показано. Для других облаков значение ресурса можно найти с помощью следующего.

```azurecli-interactive
az cloud show
```

Для Azure CLI 2.0.71 и более поздних версий команду можно указать в следующей более удобной версии для всех облаков.

```azurecli-interactive
az account get-access-token --resource-type oss-rdbms
```

После успешной аутентификации Azure AD вернет маркер доступа.

```json
{
  "accessToken": "TOKEN",
  "expiresOn": "...",
  "subscription": "...",
  "tenant": "...",
  "tokenType": "Bearer"
}
```

Маркер представляет собой строку в кодировке Base 64, которая кодирует все сведения о пользователе, прошедшем аутентификацию, и которая предназначена для службы "База данных Azure для PostgreSQL".

> [!NOTE]
> Срок действия маркера доступа составляет от 5 до 60 минут. Мы рекомендуем получить маркер доступа непосредственно перед началом входа в Базу данных Azure для PostgreSQL.

### <a name="step-3-use-token-as-password-for-logging-in-with-postgresql"></a>Шаг 3. Использование маркера в качестве пароля для входа с помощью PostgreSQL

При подключении в качестве пароля пользователя PostgreSQL необходимо использовать маркер доступа.

При использовании клиента командной строки `psql` маркер доступа необходимо передавать с помощью переменной среды `PGPASSWORD`, поскольку длина маркера доступа превышает длину пароля, которую `psql` может принять напрямую:

Пример (Windows):

```cmd
set PGPASSWORD=<copy/pasted TOKEN value from step 2>
```

```PowerShell
$env:PGPASSWORD='<copy/pasted TOKEN value from step 2>'
```

Пример (Linux/macOS):

```shell
export PGPASSWORD=<copy/pasted TOKEN value from step 2>
```

Теперь вы можете инициировать подключение к Базе данных Azure для PostgreSQL как обычно:

```shell
psql "host=mydb.postgres... user=user@tenant.onmicrosoft.com@mydb dbname=postgres sslmode=require"
```

Важные моменты при подключении:

* `user@tenant.onmicrosoft.com` — имя пользователя или группы Azure AD, которую вы пытаетесь подключить.
* Всегда добавлять имя сервера после имени пользователя или группы Azure AD (например, `@mydb` )
* Убедитесь, что используется точный способ написания имени пользователя или группы Azure AD.
* Имена пользователей и групп Azure AD чувствительны к регистру
* При подключении в качестве группы используйте только имя группы (например,). `GroupName@mydb`
* Если имя содержит пробелы, используйте `\` перед каждым пробелом, чтобы экранировать его.

Теперь вы прошли аутентификацию на сервере PostgreSQL с использованием аутентификации Azure AD.

## <a name="creating-azure-ad-users-in-azure-database-for-postgresql"></a>Создание пользователей Azure AD в Базе данных Azure для PostgreSQL

Чтобы добавить пользователя Azure AD в Базу данных Azure для PostgreSQL, после подключения выполните приведенные ниже действия (сведения о подключении см. в разделе ниже).

1. Сначала убедитесь, что пользователь Azure AD `<user>@yourtenant.onmicrosoft.com` является действительным пользователем в клиенте Azure AD.
2. Войдите в свой экземпляр Базы данных Azure для PostgreSQL в качестве администратора Azure AD.
3. Создайте роль `<user>@yourtenant.onmicrosoft.com` в Базе данных Azure для PostgreSQL.
4. Сделайте `<user>@yourtenant.onmicrosoft.com` членом роли azure_ad_user. Эту роль можно назначать только пользователям Azure AD.

**Пример**.

```sql
CREATE ROLE "user1@yourtenant.onmicrosoft.com" WITH LOGIN IN ROLE azure_ad_user;
```

> [!NOTE]
> Аутентификация пользователя через Azure AD не дает пользователю никаких разрешений для доступа к объектам в Базе данных Azure для PostgreSQL. Вы должны предоставить пользователю необходимые разрешения вручную.

## <a name="creating-azure-ad-groups-in-azure-database-for-postgresql"></a>Создание групп Azure AD в Базе данных Azure для PostgreSQL

Чтобы разрешить группе Azure AD доступ к базе данных, используйте тот же механизм, что и для пользователей, но вместо имени пользователя укажите имя группы:

**Пример**.

```sql
CREATE ROLE "Prod DB Readonly" WITH LOGIN IN ROLE azure_ad_user;
```

При входе в систему участники группы будут использовать свои персональные маркеры доступа, но будут подписаны с именем группы, указанным в качестве имени пользователя.

## <a name="token-validation"></a>Проверка маркеров

Аутентификация Azure AD в Базе данных Azure для PostgreSQL гарантирует, что пользователь существует на сервере PostgreSQL. Она выполняет проверку маркера путем проверки его содержимого. Для проверки маркера выполняются приведенные ниже шаги.

- Маркер подписан с помощью Azure AD и не был подделан.
- Azure AD выдал маркер для клиента, связанного с сервером.
- Срок действия маркера не истек.
- Маркер предназначен для ресурса Базы данных Azure для PostgreSQL (а не другого ресурса Azure).

## <a name="migrating-existing-postgresql-users-to-azure-ad-based-authentication"></a>Перенос существующих пользователей PostgreSQL для использования аутентификации на основе Azure AD

Вы можете включить аутентификацию с помощью Azure AD для существующих пользователей. Здесь необходимо рассмотреть два случая:

### <a name="case-1-postgresql-username-matches-the-azure-ad-user-principal-name"></a>Вариант 1. Имя пользователя PostgreSQL совпадает с именем субъекта-пользователя Azure AD

Маловероятно, что имена существующих пользователей совпадут с именами пользователей Azure AD. Если это все же произошло, вы можете предоставить этим пользователям роль `azure_ad_user`, чтобы разрешить для них аутентификацию с помощью Azure AD:

```sql
GRANT azure_ad_user TO "existinguser@yourtenant.onmicrosoft.com";
```

Теперь они смогут войти в систему с помощью учетных данных Azure AD вместо ранее настроенного пароля пользователя PostgreSQL.

### <a name="case-2-postgresql-username-is-different-than-the-azure-ad-user-principal-name"></a>Вариант 2. Имя пользователя PostgreSQL отличается от имени субъекта-пользователя Azure AD

Если пользователь PostgreSQL не зарегистрирован в Azure AD или у него другое имя, для аутентификации в качестве этого пользователя PostgreSQL можно использовать группы Azure AD. Существующих пользователей Базы данных Azure для PostgreSQL можно перенести в Azure AD, создав группу Azure AD с именем, соответствующим имени пользователя PostgreSQL, а затем предоставить этому пользователю PostgreSQL роль azure_ad_user:

```sql
GRANT azure_ad_user TO "DBReadUser";
```

Это предполагает создание группы DBReadUser в Azure AD. Пользователи, являющиеся членами этой группы, теперь смогут входить в базу данных как указанный пользователь.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомление с общими концепциями [проверки подлинности Azure Active Directory с помощью базы данных Azure для PostgreSQL (отдельный сервер)](concepts-aad-authentication.md)

<!--Image references-->

[1]: ./media/concepts-aad-authentication/authentication-flow.png
[2]: ./media/concepts-aad-authentication/set-aad-admin.png

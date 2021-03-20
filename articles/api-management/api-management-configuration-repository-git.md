---
title: Настройка службы управления API в Azure с помощью Git | Документация Майкрософт
description: Узнайте, как сохранить и настроить конфигурацию службы управления API с помощью Git.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: mattfarm
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/12/2019
ms.author: apimpm
ms.openlocfilehash: 18cc42c3447de733447c27db52a9a6d664539464
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89400367"
---
# <a name="how-to-save-and-configure-your-api-management-service-configuration-using-git"></a>Сохранение и настройка конфигурации службы управления API с помощью Git

Каждый экземпляр службы управления API поддерживает базу данных конфигурации, содержащую сведения о конфигурации и метаданных для экземпляра службы. Внесение изменений в экземпляр службы выполняется путем изменения параметра на портале Azure с помощью командлета PowerShell или вызова REST API. Помимо этих методов для управления конфигурацией экземпляра службы можно использовать Git для реализации приведенных далее сценариев управления службой.

* Управление версиями конфигурации — скачивание и хранение различных версий конфигурации службы
* Массовые изменения конфигурации — внесение изменений в несколько частей конфигурации службы в локальном репозитории и интеграция изменений на сервер с помощью одной операции.
* Знакомая цепочка инструментов и рабочий процесс Git — использование знакомых средств и рабочих процессов Git.

На схеме ниже приведен обзор различных способов настройки экземпляра службы управления API.

![Настройка с помощью Git][api-management-git-configure]

При внесении изменений в службу с использованием портала Azure, а также командлетов PowerShell или REST API управление базой данных конфигурации службы осуществляется с помощью конечной точки `https://{name}.management.azure-api.net`, как показано в правой части схемы. В левой части схемы демонстрируется вариант управления конфигурацией службы с помощью Git и репозитория Git для вашей службы, расположенного в `https://{name}.scm.azure-api.net`.

Ниже приведены действия с общими сведениями по управлению экземпляром службы управления API с помощью Git.

1. Доступ к конфигурации Git в службе
2. Сохранение базы данных конфигурации службы в репозиторий Git.
3. Клонирование репозитория Git на локальный компьютер.
4. Извлечение последнего репозитория на локальном компьютере, фиксация и отправка изменений обратно в репозиторий.
5. Развертывание изменений из репозитория в базу данных конфигурации службы.

В этой статье описывается включение и использование Git для управления конфигурацией и содержатся справочные сведения о файлах и папках в репозитории Git.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="access-git-configuration-in-your-service"></a>Доступ к конфигурации Git в службе

Чтобы просмотреть и настроить параметры конфигурации Git, можно щелкнуть меню **развертывание и инфраструктура** и перейти на вкладку **репозиторий** .

![Включение доступа к GIT][api-management-enable-git]

> [!IMPORTANT]
> Все секреты, которые не определены как именованные значения, будут сохранены в репозитории и останутся в его журнале до отключения и повторного включения доступа к Git. Именованные значения обеспечивают безопасное управление постоянными строковыми значениями, включая секреты, во всех конфигурациях API и политиках, поэтому их не нужно хранить непосредственно в правилах политики. Дополнительные сведения см. в статье [Использование именованных значений в политиках управления API Azure](api-management-howto-properties.md).
>
>

См. дополнительные сведения о [включении и отключении доступа к Git с помощью REST API](/rest/api/apimanagement/2019-12-01/tenantaccess?EnableGit).

## <a name="to-save-the-service-configuration-to-the-git-repository"></a>Сохранение конфигураций службы в репозитории Git

Первым действием перед клонированием репозитория является сохранение текущего состояния конфигурации службы в репозитории. Щелкните **Сохранить в репозитории**.

Внесите необходимые изменения на экране подтверждения и нажмите кнопку **сохранить** , чтобы сохранить его.

Через некоторое время конфигурация будет сохранена и отобразится состояние конфигурации репозитория, включая дату и время последнего изменения конфигурации и последней синхронизации конфигурации службы и репозитория.

Сохраненную в репозитории конфигурацию можно клонировать.

Дополнительные сведения о выполнении этой операции с помощью REST API см. в статье о [фиксации конфигурации с помощью REST API](/rest/api/apimanagement/2019-12-01/tenantaccess?CommitSnapshot).

## <a name="to-clone-the-repository-to-your-local-machine"></a>Клонирование репозитория на локальный компьютер

Чтобы клонировать репозиторий, требуется URL-адрес репозитория, имя пользователя и пароль. Чтобы получить имя пользователя и другие учетные данные, щелкните **Учетные данные для доступа** в верхней части страницы.

Чтобы создать пароль, проверьте, указаны ли в строке **Срок действия** нужные значения даты и времени срока действия, а затем нажмите кнопку **Создать**.

> [!IMPORTANT]
> Запомните этот пароль. После закрытия этой страницы пароль больше не будет отображаться.
>

В следующих примерах применяется средство Git Bash из [Git для Windows](https://www.git-scm.com/downloads) , но можно использовать любое знакомое вам средство Git.

Откройте средство Git в нужной папке и выполните следующую команду, чтобы клонировать репозиторий Git на локальный компьютер с помощью команды, предоставленной портал Azure.

```
git clone https://{name}.scm.azure-api.net/
```

При появлении запроса введите имя пользователя и пароль.

При появлении ошибок попробуйте изменить команду `git clone` так, чтобы она включала в себя имя пользователя и пароль, как показано в следующем примере.

```
git clone https://username:password@{name}.scm.azure-api.net/
```

Если ошибка возникнет и в этом случае, попробуйте использовать кодирование URL части пароля, содержащегося в команде. Чтобы быстро решить эту задачу, откройте Visual Studio и выполните следующую команду в **окне интерпретации**. Чтобы открыть **окно интерпретации**, откройте любое решение или проект в Visual Studio (или создайте пустое консольное приложение), а затем выберите **Окна** и **Интерпретация** в меню **Отладка**.

```
?System.Net.WebUtility.UrlEncode("password from the Azure portal")
```

Зашифрованный пароль вместе с именем пользователя и расположением репозитория можно использовать для создания команды git.

```
git clone https://username:url encoded password@{name}.scm.azure-api.net/
```

После клонирования репозиторий доступен для просмотра и работы в локальной файловой системе. Дополнительные сведения см. в разделе [Справочные сведения по структуре файлов и папок локального репозитория Git](#file-and-folder-structure-reference-of-local-git-repository).

## <a name="to-update-your-local-repository-with-the-most-current-service-instance-configuration"></a>Обновление локального репозитория самой последней конфигурацией экземпляра службы

Перед обновлением локального репозитория с помощью последних изменений необходимо сохранить изменения, вносимые в экземпляр службы управления API, на портале Azure или с помощью REST API. Для этого щелкните **сохранить в репозитории** на вкладке **репозиторий** в портал Azure, а затем выполните следующую команду в локальном репозитории.

```
git pull
```

Перед выполнением `git pull` убедитесь, что выбрана папка локального хранилища. Если вы только что завершили команду `git clone` , необходимо изменить каталог на репозиторий, выполнив команду, аналогичную приведенной ниже.

```
cd {name}.scm.azure-api.net/
```

## <a name="to-push-changes-from-your-local-repo-to-the-server-repo"></a>Отправка изменений из локального репозитория в репозиторий сервера
Чтобы отправить изменения из локального репозитория в репозиторий сервера, необходимо зафиксировать изменения и после этого отправить их в репозиторий сервера. Чтобы зафиксировать изменения, откройте командное средство Git, перейдите в каталог локального репозитория и выполните следующие команды.

```
git add --all
git commit -m "Description of your changes"
```

Для отправки всех фиксаций на сервер выполните следующую команду.

```
git push
```

## <a name="to-deploy-any-service-configuration-changes-to-the-api-management-service-instance"></a>Развертывание изменений конфигурации службы в экземпляре службы управления API

После фиксации и отправки локальных изменений в репозиторий сервера их можно развернуть в экземпляре службы управления API.

Дополнительные сведения о выполнении этой операции с помощью REST API см. в статье о [развертывании изменений Git в базе данных конфигурации с помощью REST API](/rest/api/apimanagement/2019-12-01/tenantconfiguration).

## <a name="file-and-folder-structure-reference-of-local-git-repository"></a>Справочные сведения по структуре файлов и папок локального репозитория Git

Файлы и папки в локальном репозитории Git содержат сведения о конфигурации экземпляра службы.

| Элемент | Описание |
| --- | --- |
| Корневая папка api-management |Содержит конфигурацию верхнего уровня для экземпляра службы |
| Папка apis |Содержит конфигурацию для API в экземпляре службы |
| Папка groups |Содержит конфигурацию для групп в экземпляре службы |
| Папка policies |Содержит политики в экземпляре службы |
| Папка portalStyles |Содержит конфигурацию для настроек портала разработчика в экземпляре службы |
| Папка products |Содержит конфигурацию для продуктов в экземпляре службы |
| Папка templates |Содержит конфигурацию для шаблонов электронной почты в экземпляре службы |

В каждой папке может находиться один или несколько файлов и в некоторых случаях одна или несколько папок, например папка для каждого API, продукта или группы. Файлы в каждой папке специфичны для типа сущности, описанного в имени папки.

| Тип файла | Назначение |
| --- | --- |
| json |Сведения о конфигурации соответствующей сущности |
| html |Описания сущности, часто отображаемые на портале разработчика |
| Xml |Правила политики |
| css |Таблицы стилей для настройки портала разработчика |

Эти файлы можно создавать, удалять, изменять и контролировать в локальной файловой системе. А изменения можно снова развернуть в экземпляре службы управления API.

> [!NOTE]
> Указанные далее сущности отсутствуют в репозитории Git и не могут быть настроены с помощью Git.
>
> * [Пользователи](/rest/api/apimanagement/2019-12-01/user)
> * [Подписки](/rest/api/apimanagement/2019-12-01/subscription)
> * Именованные значения
> * Сущности портала разработчика, отличные от стилей
>

### <a name="root-api-management-folder"></a>Корневая папка api-management
Корневая папка `api-management` содержит файл `configuration.json` со сведениями верхнего уровня об экземпляре службы в следующем формате.

```json
{
  "settings": {
    "RegistrationEnabled": "True",
    "UserRegistrationTerms": null,
    "UserRegistrationTermsEnabled": "False",
    "UserRegistrationTermsConsentRequired": "False",
    "DelegationEnabled": "False",
    "DelegationUrl": "",
    "DelegatedSubscriptionEnabled": "False",
    "DelegationValidationKey": "",
    "RequireUserSigninEnabled": "false"
  },
  "$ref-policy": "api-management/policies/global.xml"
}
```

Первые четыре параметра ( `RegistrationEnabled` ,, `UserRegistrationTerms` `UserRegistrationTermsEnabled` и `UserRegistrationTermsConsentRequired` ) соответствуют следующим параметрам на вкладке **удостоверения** в разделе **портал разработчика** .

| Параметр удостоверения | Соответствует параметру |
| --- | --- |
| RegistrationEnabled |Присутствие поставщика удостоверений с **именем пользователя и паролем**. |
| UserRegistrationTerms |**Условия использования при регистрации пользователя** |
| UserRegistrationTermsEnabled |**Показывать условия использования на странице регистрации** |
| UserRegistrationTermsConsentRequired |**Требовать согласия** |
| RequireUserSigninEnabled |**Перенаправлять анонимных пользователей на страницу входа** |

Следующие четыре параметра ( `DelegationEnabled` ,, `DelegationUrl` `DelegatedSubscriptionEnabled` и `DelegationValidationKey` ) соответствуют следующим параметрам на вкладке **Делегирование** в разделе **портал разработчика** .

| Параметр делегирования | Соответствует параметру |
| --- | --- |
| DelegationEnabled |Флажок **Делегировать вход и регистрацию** |
| DelegationUrl |**URL-адрес конечной точки делегирования** |
| DelegatedSubscriptionEnabled |**Делегировать подписку на продукт** |
| DelegationValidationKey |**Делегировать ключ проверки** |

Последний параметр `$ref-policy`соответствует глобальному файлу правил политики для экземпляра службы.

### <a name="apis-folder"></a>Папка apis
Папка `apis` содержит папку для каждого API в экземпляре службы, в котором находятся указанные далее элементы.

* `apis\<api name>\configuration.json` — это конфигурация для API, которая содержит сведения о серверном URL-адресе службы и операциях. Это те же сведения, которые возвращаются при вызове операции [Получить определенный API](/rest/api/apimanagement/2019-12-01/apis/get) с `export=true` в формате `application/json`.
* `apis\<api name>\api.description.html` — это описание API, которое соответствует свойству `description`[сущности API](/java/api/com.microsoft.azure.storage.table.entityproperty).
* `apis\<api name>\operations\` — эта папка содержит файлы `<operation name>.description.html`, соответствующие операциям в API. Каждый файл содержит описание одной операции в API, которая сопоставляется со свойством `description`[сущности operation](/rest/api/visualstudio/operations/list#operationproperties) в REST API.

### <a name="groups-folder"></a>Папка groups
Папка `groups` содержит папку для каждой группы, определенной в экземпляре службы.

* `groups\<group name>\configuration.json` — это конфигурация для группы. Это те же сведения, которые возвращаются при вызове операции [Получить определенную группу](/rest/api/apimanagement/2019-12-01/group/get) .
* `groups\<group name>\description.html` — это описание группы, которое соответствует свойству `description`[сущности group](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity).

### <a name="policies-folder"></a>Папка policies
Папка `policies` содержит правила политики для экземпляра службы.

* `policies\global.xml` содержит политики, определенные в глобальной области для экземпляра службы.
* `policies\apis\<api name>\` — если в области API определены какие-либо политики, они содержатся в этой папке.
* Папка `policies\apis\<api name>\<operation name>\` — если в области операции определены какие-либо политики, они содержатся в этой папке в файлах `<operation name>.xml`, соответствующих правилам политики для каждой операции.
* `policies\products\` — если в области продукта определены какие-либо политики, они содержатся в этой папке в файлах `<product name>.xml`, соответствующих правилам политики для каждого продукта.

### <a name="portalstyles-folder"></a>Папка portalStyles
Папка `portalStyles` содержит конфигурацию и таблицы стилей для настроек портала разработчика для экземпляра службы.

* `portalStyles\configuration.json` — содержит имена таблиц стилей, используемых на портале разработчика.
* `portalStyles\<style name>.css` — каждый файл `<style name>.css` содержит стили для портала разработчика (`Preview.css` и `Production.css` по умолчанию).

### <a name="products-folder"></a>Папка products
Папка `products` содержит папку для каждого продукта, определенного в экземпляре службы.

* `products\<product name>\configuration.json` — это конфигурация для продукта. Это те же сведения, которые возвращаются при вызове операции [Получить определенный продукт](/rest/api/apimanagement/2019-12-01/product/get) .
* `products\<product name>\product.description.html` — это описание продукта, которое соответствует свойству `description`[сущности product](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-product-entity) в REST API.

### <a name="templates"></a>шаблоны
Папка `templates` содержит конфигурацию для [шаблонов электронной почты](api-management-howto-configure-notifications.md) экземпляра службы.

* `<template name>\configuration.json` — это конфигурация для шаблона электронной почты.
* `<template name>\body.html` — это текст сообщения электронной почты.

## <a name="next-steps"></a>Дальнейшие действия
Сведения о других способах управления экземпляром службы см. в приведенных ниже статьях.

* Управление экземпляром службы с помощью следующих командлетов PowerShell
  * [Справочник по командлетам PowerShell для развертывания службы](/powershell/module/wds)
  * [Справочник по командлетам PowerShell для управления службами](/powershell/azure/servicemanagement/overview)
* Управление экземпляром службы с помощью REST API
  * [Справочник по REST API для управления API](/rest/api/apimanagement/)


[api-management-enable-git]: ./media/api-management-configuration-repository-git/api-management-enable-git.png
[api-management-git-enabled]: ./media/api-management-configuration-repository-git/api-management-git-enabled.png
[api-management-save-configuration]: ./media/api-management-configuration-repository-git/api-management-save-configuration.png
[api-management-save-configuration-confirm]: ./media/api-management-configuration-repository-git/api-management-save-configuration-confirm.png
[api-management-configuration-status]: ./media/api-management-configuration-repository-git/api-management-configuration-status.png
[api-management-configuration-git-clone]: ./media/api-management-configuration-repository-git/api-management-configuration-git-clone.png
[api-management-generate-password]: ./media/api-management-configuration-repository-git/api-management-generate-password.png
[api-management-password]: ./media/api-management-configuration-repository-git/api-management-password.png
[api-management-git-configure]: ./media/api-management-configuration-repository-git/api-management-git-configure.png
[api-management-configuration-deploy]: ./media/api-management-configuration-repository-git/api-management-configuration-deploy.png
[api-management-identity-settings]: ./media/api-management-configuration-repository-git/api-management-identity-settings.png
[api-management-delegation-settings]: ./media/api-management-configuration-repository-git/api-management-delegation-settings.png
[api-management-git-icon-enable]: ./media/api-management-configuration-repository-git/api-management-git-icon-enable.png

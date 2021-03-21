---
title: Azure Red Hat OpenShift с OpenShift 4. Настройка проверки подлинности Azure Active Directory с помощью портал Azure и веб-консоли OpenShift
description: Узнайте, как настроить проверку подлинности Azure Active Directory для кластера Azure Red Hat OpenShift с OpenShift 4 с помощью веб-консоли портал Azure и OpenShift.
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 03/12/2020
author: sabbour
ms.author: asabbour
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: eda45b1a1a011a646915cf45e45218ae168a2af6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102213085"
---
# <a name="configure-azure-active-directory-authentication-for-an-azure-red-hat-openshift-4-cluster-portal"></a>Настройка проверки подлинности Azure Active Directory для кластера Azure Red Hat OpenShift 4 (портал)

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Перед началом

Создайте **URL-адрес обратного вызова OAuth** кластера и запишите его. Не забудьте заменить **АТО-RG** именем группы ресурсов, а **АТО-Cluster —** именем кластера.

> [!NOTE]
> `AAD`Раздел в URL-адресе обратного вызова OAuth должен соответствовать имени поставщика удостоверений OAuth, которое будет настроено позже.

```azurecli-interactive
domain=$(az aro show -g aro-rg -n aro-cluster --query clusterProfile.domain -o tsv)
location=$(az aro show -g aro-rg -n aro-cluster --query location -o tsv)
echo "OAuth callback URL: https://oauth-openshift.apps.$domain.$location.aroapp.io/oauth2callback/AAD"
```

## <a name="create-an-azure-active-directory-application-for-authentication"></a>Создание приложения Azure Active Directory для проверки подлинности

Войдите в портал Azure и перейдите в [колонку регистрация приложений](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade), а затем щелкните **Новая регистрация** , чтобы создать новое приложение.

Укажите имя приложения, например **АТО-azuread-auth**, и заполните **URI перенаправления** , используя значение URL-адреса обратного вызова OAuth, полученное ранее.

![Регистрация нового приложения](media/aro4-ad-registerapp.png)

Перейдите в раздел **сертификаты & секреты** и щелкните **новый секрет клиента** и введите сведения. Запишите значение ключа, так как оно будет использоваться на более позднем этапе. Вы не сможете получить его снова.

![Создание секрета](media/aro4-ad-clientsecret.png)

Перейдите к **обзору** и запишите идентификатор **приложения (клиента)** и **идентификатор каталога (клиента)**. Они понадобятся вам на более позднем этапе.

![Получение идентификаторов приложения (клиента) и каталога (клиента)](media/aro4-ad-ids.png)

## <a name="configure-optional-claims"></a>Настройка необязательных утверждений

Разработчики приложений могут использовать [необязательные утверждения](../active-directory/develop/active-directory-optional-claims.md) в своих приложениях Azure AD, чтобы указать, какие утверждения они хотят в токенах, отправляемых в приложение.

Необязательные утверждения можно использовать, чтобы:

* выбрать дополнительные утверждения для включения в токены для приложения;
* изменить поведение определенных утверждений в токенах, возвращаемых Azure AD;
* добавлять пользовательские утверждения для приложения и обращаться к ним.

Мы настроим OpenShift на использование `email` утверждения и вернемся к `upn` установке предпочтительного имени пользователя, добавив в `upn` маркер идентификации, возвращенный Azure Active Directory.

Перейдите в раздел **Конфигурация маркера (Предварительная версия)** и щелкните **добавить необязательное утверждение**. Выберите **идентификатор** , а затем проверьте **адрес электронной почты** и утверждения **имени участника-пользователя** .

![Снимок экрана, показывающий сообщения электронной почты и утверждения имени участника-пользователя, которые были добавлены.](media/aro4-ad-tokens.png)

## <a name="assign-users-and-groups-to-the-cluster-optional"></a>Назначение пользователей и групп кластеру (необязательно)

Приложения, зарегистрированные в клиенте Azure Active Directory (Azure AD), по умолчанию являются доступными для всех пользователей клиента, которые успешно прошли проверку подлинности. Azure AD позволяет администраторам и разработчикам клиента ограничить доступ к приложению определенными пользователями или группами безопасности в клиенте.

Следуйте инструкциям в документации по Azure Active Directory, чтобы [назначить пользователей и группы для приложения](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md#app-registration).

## <a name="configure-openshift-openid-authentication"></a>Настройка проверки подлинности OpenShift OpenID Connect

Получите `kubeadmin` учетные данные. Выполните следующую команду, чтобы найти пароль для пользователя `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name aro-cluster \
  --resource-group aro-rg
```

В следующем примере вывода показано, что пароль будет находиться в `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

URL-адрес консоли кластера можно найти, выполнив следующую команду, которая будет выглядеть вот так: `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`

```azurecli-interactive
 az aro show \
    --name aro-cluster \
    --resource-group aro-rg \
    --query "consoleProfile.url" -o tsv
```

Перейдите по URL-адресу консоли в браузере и войдите с помощью учетных данных `kubeadmin`.

Перейдите в меню **Администрирование**, щелкните **Параметры кластера**, а затем выберите вкладку **Глобальная конфигурация** . Прокрутите экран, чтобы выбрать **OAuth**.

Прокрутите вниз, чтобы выбрать **Добавить** в разделе **поставщики удостоверений** , и выберите **OpenID Connect Connect**.
![Выберите OpenID Connect Connect в раскрывающемся списке поставщиков удостоверений.](media/aro4-oauth-idpdrop.png)

Заполните имя именем **AAD**, **идентификатором клиента** в качестве **идентификатора приложения** и **секретом клиента**. **URL-адрес издателя** имеет такой формат: `https://login.microsoftonline.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` . Замените заполнитель ИДЕНТИФИКАТОРом клиента, полученным ранее.

![Сведения о заполнении OAuth](media/aro4-oauth-idp-1.png)

Прокрутите вниз до раздела **claims** и обновите **предпочтительное имя пользователя** , чтобы использовать значение из утверждения **имени участника-пользователя** .

![Сведения о заполнении утверждений](media/aro4-oauth-idp-2.png)

## <a name="verify-login-through-azure-active-directory"></a>Проверка имени входа с помощью Azure Active Directory

Если теперь выйти из веб-консоли OpenShift и повторить попытку входа, отобразится новый параметр для входа с помощью **AAD**. Возможно, потребуется подождать несколько минут.

![Экран входа с параметром Azure Active Directory](media/aro4-login-2.png)

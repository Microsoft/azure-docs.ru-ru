---
title: Настройка встроенного реестра контейнеров для Azure Red Hat OpenShift 4
description: Настройка встроенного реестра контейнеров для Azure Red Hat OpenShift 4
author: jiangma
ms.author: jiangma
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 10/15/2020
ms.openlocfilehash: dda050b8d824f0ff0ac1c84d2f008387de55aedf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100636389"
---
# <a name="configure-built-in-container-registry-for-azure-red-hat-openshift-4"></a>Настройка встроенного реестра контейнеров для Azure Red Hat OpenShift 4

Azure Red Hat OpenShift предоставляет интегрированный реестр образов контейнеров, именуемый [реестром OpenShift контейнеров (OCR)](https://docs.openshift.com/container-platform/4.6/registry/architecture-component-imageregistry.html) , который добавляет возможность автоматически подготавливать новые репозитории образов по запросу. Это дает пользователям встроенное расположение для сборок приложений, чтобы отправлять полученные образы.

В этой статье вы настроите встроенный реестр образов контейнера для кластера Azure Red Hat OpenShift (АТО) 4. Вы узнаете, как:

> [!div class="checklist"]
> * Настройка Azure AD
> * Настройка подключения OpenID Connect
> * Доступ к встроенному реестру образов контейнеров

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас уже есть кластер АТО. Если вам нужен кластер АТО, см. Руководство по АТО, в котором описывается [Создание кластера Azure Red Hat OpenShift 4](./tutorial-create-cluster.md). Не забудьте создать кластер с `--pull-secret` аргументом для `az aro create` .  Это необходимо для настройки проверки подлинности Azure Active Directory и встроенного реестра контейнеров.

После создания кластера подключитесь к кластеру, выполнив действия, описанные в подразделе [Подключение к кластеру Azure Red Hat OpenShift 4](./tutorial-connect-cluster.md).
   * Обязательно выполните действия, описанные в разделе "Установка OpenShift CLI", так как мы будем использовать эту `oc` команду Далее в этой статье.
   * Запишите URL-адрес консоли кластера, который выглядит так `https://console-openshift-console.apps.<random>.<region>.aroapp.io/` :. Значения для `<random>` и `<region>` будут использоваться далее в этой статье.
   * Запишите `kubeadmin` учетные данные. Они также будут использоваться далее в этой статье.

### <a name="configure-azure-active-directory-authentication"></a>Настройка аутентификации Azure Active Directory 

Azure Active Directory (Azure AD) реализует OpenID Connect Connect (OIDC). OIDC позволяет использовать Azure AD для входа в кластер АТО. Выполните действия, описанные в разделе [Настройка проверки подлинности Azure Active Directory](configure-azure-ad-cli.md) для настройки кластера.

## <a name="access-the-built-in-container-image-registry"></a>Доступ к встроенному реестру образов контейнеров

Теперь, когда методы проверки подлинности настроены в кластере АТО, давайте разберем доступ к встроенному реестру.

#### <a name="define-the-azure-ad-user-to-be-an-administrator"></a>Определите пользователя Azure AD в качестве администратора.

1. Войдите в веб-консоль OpenShift из браузера, используя учетные данные пользователя Azure AD. Мы воспользуемся проверкой подлинности OpenShift OpenID Connect в Azure Active Directory, чтобы использовать OpenID Connect для определения администратора.

   1. Используйте InPrivate, режиме инкогнито или другую эквивалентную функцию окна браузера для входа в консоль. После включения OIDC окно будет выглядеть иначе.
   
   :::image type="content" source="media/built-in-container-registry/oidc-enabled-login-window.png" alt-text="Окно входа с включенным OpenID Connect подключением.":::
   1. Выбор **OpenID Connect**

   > [!NOTE]
   > Запишите имя пользователя и пароль, которые вы используете для входа. Это имя пользователя и пароль будут работать как администратор для других действий в этой и других статьях.
2. Выполните вход с помощью интерфейса командной строки OpenShift, выполнив следующие действия.  Для обсуждения этот процесс называется `oc login` .
   1. В правой верхней части веб-консоли разверните контекстное меню пользователя, выполнившего вход, и выберите **команду Копировать имя входа**.
   2. При необходимости войдите в новое окно вкладки с тем же пользователем.
   3. Выберите **Показать маркер**.
   4. Скопируйте указанное ниже значение **Login с этим маркером** в буфер обмена и запустите его в оболочке, как показано ниже.

       ```bash
       oc login --token=XOdASlzeT7BHT0JZW6Fd4dl5EwHpeBlN27TAdWHseob --server=https://api.aqlm62xm.rnfghf.aroapp.io:6443
       Logged into "https://api.aqlm62xm.rnfghf.aroapp.io:6443" as "kube:admin" using the token provided.

       You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'

       Using project "default".
       ```

3. Запустите `oc whoami` в консоли и запишите выходные данные в виде **\<aad-user>** .  Это значение будет использоваться позже в этой статье.
4. Выйдите из веб-консоли OpenShift. Нажмите кнопку в правом верхнем углу окна браузера с меткой **\<aad-user>** и выберите **выход**.


#### <a name="grant-the-azure-ad-user-the-necessary-roles-for-registry-interaction"></a>Предоставьте пользователю Azure AD необходимые роли для взаимодействия с реестром.

1. Войдите в веб-консоль OpenShift из браузера, используя `kubeadmin` учетные данные.
1. Войдите в OpenShift CLI с маркером для, выполнив `kubeadmin` действия, описанные `oc login` выше, но выполните их после входа в веб-консоль с помощью `kubeadmin` .
1. Выполните следующие команды, чтобы включить доступ к встроенному реестру для **AAD-User**.

   ```bash
   # Switch to project "openshift-image-registry"
   oc project openshift-image-registry
   
   # Output should look similar to the following.
   # Now using project "openshift-image-registry" on server "https://api.x8xl3f4y.eastus.aroapp.io:6443".
   ```

   ```bash
   # Expose the registry using "DefaultRoute"
   oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge

   # Output should look similar to the following.
   # config.imageregistry.operator.openshift.io/cluster patched
   ```

   ```bash
   # Add roles to "aad-user" for pulling and pushing images
   # Note: replace "<aad-user>" with the one you wrote down before
   oc policy add-role-to-user registry-viewer <aad-user>

   # Output should look similar to the following.
   # clusterrole.rbac.authorization.k8s.io/registry-viewer added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

   ```bash
   oc policy add-role-to-user registry-editor <aad-user>
   # Output should look similar to the following.
   # clusterrole.rbac.authorization.k8s.io/registry-editor added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

#### <a name="obtain-the-container-registry-url"></a>Получение URL-адреса реестра контейнеров

Используйте `oc get route` команду, как показано ниже, чтобы получить URL-адрес реестра контейнеров.

```bash
# Note: the value of "Container Registry URL" in the output is the fully qualified registry name.
HOST=$(oc get route default-route --template='{{ .spec.host }}')
echo "Container Registry URL: $HOST"
```

   > [!NOTE]
   > Обратите внимание на выходные данные консоли **URL-адреса реестра контейнеров**. Он будет использоваться в качестве полного имени реестра для данного руководства и последующих.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили встроенный реестр образов контейнеров, вы можете начать с развертывания приложения на OpenShift. Для приложений Java ознакомьтесь [с развернутым приложением Java с помощью Open Liberty/WebSphere Liberty в кластере Azure Red Hat OpenShift 4](howto-deploy-java-liberty-app.md).
---
title: Настройка доступа к Azure Active Directoryу Azure Блокчейн Service
description: Как настроить службу Блокчейн Azure с доступом к Azure Active Directory
ms.date: 11/22/2019
ms.topic: how-to
ms.reviewer: janders
ms.openlocfilehash: 337d01abc51d310d06aeea3427b770132be4824c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85208779"
---
# <a name="how-to-configure-azure-active-directory-access-for-azure-blockchain-service"></a>Настройка Azure Active Directoryного доступа для службы Azure Блокчейн

Из этой статьи вы узнаете, как предоставить доступ и подключиться к узлам службы Блокчейн Azure с помощью Azure Active Directory (Azure AD) пользователей, групп или идентификаторов приложений.

Azure AD обеспечивает облачное управление удостоверениями и позволяет использовать единое удостоверение во всем корпоративном приложении и доступ к приложениям в Azure. Служба Блокчейн Azure интегрирована с Azure AD и предоставляет такие преимущества, как Федерация идентификации, единый вход и многофакторная идентификация.

## <a name="prerequisites"></a>Предварительные требования

* [Создание элемента блокчейн с помощью портал Azure](create-member.md)

## <a name="grant-access"></a>Предоставление доступа

Доступ можно предоставить как на уровне члена, так и на уровне узла. Предоставление прав доступа на уровне членов, в свою очередь, предоставляет доступ ко всем узлам в члене.

### <a name="grant-member-level-access"></a>Предоставление доступа на уровне членов

Значение, чтобы предоставить разрешение на доступ на уровне членов.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Перейдите к **элементу управления доступом (IAM) > добавить > добавить назначение ролей**.
1. Выберите роль **доступ к узлу-члену блокчейн (Предварительная версия)** и добавьте объект идентификатора Azure AD, к которому вы хотите предоставить доступ. Объект идентификатора Azure AD может быть следующим:

    | Объект Azure AD | Пример |
    |-----------------|---------|
    | Пользователь Azure AD   | `kim@contoso.onmicrosoft.com` |
    | Группа Azure AD  | `sales@contoso.onmicrosoft.com` |
    | Идентификатор приложения  | `13925ab1-4161-4534-8d18-812f5ca1ab1e` |

    ![Добавление назначения роли](./media/configure-aad/add-role-assignment.png)

1. Щелкните **Сохранить**.

### <a name="grant-node-level-access"></a>Предоставление доступа на уровне узла

Вы можете предоставить доступ на уровне узла, перейдя к разделу Безопасность узла и щелкнув имя узла, доступ к которому требуется предоставить.

Выберите роль доступ к узлу-члену Блокчейн (Предварительная версия) и добавьте объект идентификатора Azure AD, к которому вы хотите предоставить доступ.

Дополнительные сведения см. в статье [Настройка узлов транзакций службы Azure блокчейн](configure-transaction-nodes.md#azure-active-directory-access-control).

## <a name="connect-using-azure-blockchain-connector"></a>Подключение с помощью соединителя Azure Блокчейн

Скачайте или клонировать [соединитель Azure блокчейн с сайта GitHub](https://github.com/Microsoft/azure-blockchain-connector/).

```bash
git clone https://github.com/Microsoft/azure-blockchain-connector.git
```

Чтобы создать соединитель из исходного кода, следуйте инструкциям в разделе Краткое руководство в **файле readme** .

### <a name="connect-using-an-azure-ad-user-account"></a>Подключение с использованием учетной записи пользователя Azure AD

1. Выполните следующую команду для проверки подлинности с помощью учетной записи пользователя Azure AD. Замените \<myAADDirectory\> доменом Azure AD. Например, `yourdomain.onmicrosoft.com`.

    ```
    connector.exe -remote <myMemberName>.blockchain.azure.com:3200 -method aadauthcode -tenant-id <myAADDirectory> 
    ```

1. Azure AD запрашивает учетные данные.
1. Выполните вход, используя имя пользователя и пароль.
1. После успешной проверки подлинности локальный прокси-сервер подключается к блокчейн узлу. Теперь вы можете подключить клиент жес к локальной конечной точке.

    ```bash
    geth attach http://127.0.0.1:3100
    ```

### <a name="connect-using-an-application-id"></a>Подключение с помощью идентификатора приложения

Многие приложения проходят проверку подлинности в Azure AD, используя идентификатор приложения, а не учетную запись пользователя Azure AD.

Чтобы подключиться к узлу с помощью идентификатора приложения, замените **аадаускоде** на **аадклиент**.

```
connector.exe -remote <myBlockchainEndpoint>  -method aadclient -client-id <myClientID> -client-secret "<myClientSecret>" -tenant-id <myAADDirectory>
```

| Параметр | Описание |
|-----------|-------------|
| Идентификатор клиента | Домен Azure AD, например `yourdomain.onmicrosoft.com`
| client-id | Идентификатор клиента зарегистрированного приложения в Azure AD
| client-secret | Секрет клиента зарегистрированного приложения в Azure AD

Дополнительные сведения о регистрации приложения в Azure AD см. в статье [как использовать портал для создания приложения Azure AD и субъекта-службы, которые могут получать доступ к ресурсам.](../../active-directory/develop/howto-create-service-principal-portal.md)

### <a name="connect-a-mobile-device-or-text-browser"></a>Подключение мобильного устройства или текстового браузера

Для мобильного устройства или текстового браузера, где всплывающее окно проверки подлинности Azure AD невозможно, Azure AD создает одноразовый секретный код. Вы можете скопировать секретный код и продолжить проверку подлинности Azure AD в другой среде.

Чтобы создать секретный код, замените **аадаускоде** на **ааддевице**. Замените \<myAADDirectory\> доменом Azure AD. Например, `yourdomain.onmicrosoft.com`.

```
connector.exe -remote <myBlockchainEndpoint>  -method aaddevice -tenant-id <myAADDirectory>
```

## <a name="next-steps"></a>Дальнейшие шаги

Дополнительные сведения о безопасности данных в службе Azure Блокчейн см. в статье [Безопасность службы Блокчейн Azure](data-security.md).
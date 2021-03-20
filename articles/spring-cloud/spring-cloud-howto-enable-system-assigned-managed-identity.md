---
title: Включение управляемого удостоверения, назначенного системой, для приложения Azure Spring Cloud
description: Включение управляемого удостоверения, назначенного системой для приложения.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 05/13/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 2fbdd46b872c4c70999355d457045f4aac8aa34c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92742794"
---
# <a name="how-to-enable-system-assigned-managed-identity-for-azure-spring-cloud-application"></a>Как включить управляемое удостоверение, назначенное системой, для приложения Azure Spring Cloud

**Эта статья применима к:** ✔️ Java ✔️ C#

Управляемые удостоверения для ресурсов Azure предоставляют автоматически управляемое удостоверение в Azure Active Directory к ресурсу Azure, например к облачному приложению Azure весны. Это удостоверение можно использовать для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, не храня какие-либо учетные данные в коде.

В этой статье показано, как включить и отключить назначенные системой управляемые удостоверения для облачного приложения Azure весны с помощью портал Azure и CLI (доступно в версии 0.2.4).

## <a name="prerequisites"></a>Предварительные условия
Если вы не знакомы с управляемыми удостоверениями для ресурсов Azure, см. [раздел Обзор](../active-directory/managed-identities-azure-resources/overview.md).
Вам потребуется развернутый экземпляр облака Azure весны. Следуйте инструкциям [краткого руководства, чтобы выполнить развертывание с помощью Azure CLI](spring-cloud-quickstart.md).

## <a name="add-a-system-assigned-identity"></a>Добавление назначаемого системой удостоверения
Создание приложения с удостоверением, назначенным системой, требует установки дополнительного свойства приложения.

### <a name="using-azure-portal"></a>Использование портала Azure
Чтобы настроить управляемое удостоверение в [портал Azure](https://portal.azure.com/), сначала создайте приложение, а затем включите эту функцию.

1. Создайте приложение на портале обычным образом. Перейдите к нему на портале.
2. Прокрутите вниз до группы **Параметры** в левой области навигации.
3. Выберите **Удостоверение**.
4. На вкладке **Назначено системой** для параметра **Состояние** установите значение *Вкл*. Выберите команду **Сохранить**.

 ![Управляемое удостоверение на портале](./media/spring-cloud-managed-identity/identity-1.png)

### <a name="using-azure-cli"></a>Использование Azure CLI
Управляемое системой удостоверение можно включить во время создания приложения или существующего приложения.

**Включение назначенного системой управляемого удостоверения во время создания приложения**

В следующем примере создается приложение с именем *APP_NAME* и управляемым удостоверением, назначенным системой, согласно запросу `--assign-identity` параметра.

```azurecli
az spring-cloud app create -n app_name -s service_name -g resource_group_name --assign-identity
```

**Включение управляемого удостоверения, назначенного системой, в существующем приложении** Используйте `az spring-cloud app identity assign` команду, чтобы включить назначенное системой удостоверение для существующего приложения.

```azurecli
az spring-cloud app identity assign -n app_name -s service_name -g resource_group_name
```

## <a name="obtain-tokens-for-azure-resources"></a>Получение маркеров для ресурсов Azure
Приложение может использовать управляемое удостоверение для получения маркеров для доступа к другим ресурсам, защищенным Azure Active Directory, например Azure Key Vault. Эти маркеры представляют приложение, обращающееся к ресурсу, а не к конкретному пользователю приложения.

Возможно, потребуется [настроить целевой ресурс, чтобы разрешить доступ из приложения](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md). Например, если вы запрашиваете маркер для доступа к Key Vault, убедитесь, что вы добавили политику доступа, включающую в себя удостоверение приложения. В противном случае вызовы Key Vault будут отклонены, даже если они включают маркеры. Чтобы узнать больше о том, какие ресурсы поддерживают маркеры Azure Active Directory, см. сведения в разделе о [службах Azure, поддерживающих аутентификацию Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

Azure Веснное облако использует одну и ту же конечную точку для получения маркера с помощью виртуальной машины Azure. Для получения маркера рекомендуется использовать пакет SDK для Java или пружинный запуск.  См. раздел [Использование маркера виртуальной машины](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) для различных примеров кода и сценариев, а также рекомендации по важным темам, таким как обработка истечения срока действия маркера и ошибок HTTP.

Рекомендуется. для получения маркеров используйте Java SDK или пружинные стартовые загрузчики.  Ознакомьтесь с примерами на [следующих шагах](#next-steps).

## <a name="disable-system-assigned-identity-from-an-app"></a>Отключение назначенного системой удостоверения из приложения
Удаление назначенного системой удостоверения также приведет к его удалению из Azure AD. При удалении ресурса приложения автоматически удаляются назначенные системой удостоверения из Azure AD.

### <a name="using-azure-portal"></a>Использование портала Azure
Чтобы удалить назначенное системой управляемое удостоверение из приложения, которое больше не требуется, сделайте следующее:

1. Войдите в [портал Azure](https://portal.azure.com/) с помощью учетной записи, связанной с подпиской Azure, которая содержит экземпляр облака Azure весны.
1. Перейдите к нужной виртуальной машине и выберите **Удостоверение**.
1. В разделе состояние **назначения системы** / выберите **выкл** ., а затем нажмите кнопку **сохранить**.

 ![Управляемое удостоверение](./media/spring-cloud-managed-identity/remove-identity.png)

### <a name="using-azure-cli"></a>Использование Azure CLI
Чтобы удалить назначенное системой управляемое удостоверение из приложения, которое больше не требуется, используйте следующую команду:
```azurecli
az spring-cloud app identity remove -n app_name -s service_name -g resource_group_name
```

## <a name="next-steps"></a>Дальнейшие действия

* [Доступ к Azure Key Vault с управляемыми удостоверениями в пружинной заначальной загрузке](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/spring/azure-spring-boot-starter-keyvault-secrets/README.md#use-msi--managed-identities)
* [Дополнительные сведения об управляемых удостоверениях для ресурсов Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Использование управляемых удостоверений с пакетом SDK для Java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples)

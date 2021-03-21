---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 01/27/2021
ms.author: mimart
ms.openlocfilehash: 73216b1b089444c1dc92bbe73ed07895de3711b2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98951544"
---
## <a name="configure-the-relying-party-policy"></a>Настройка политики проверяющей стороны

Политика проверяющей стороны, например [SignUpSignIn.xml](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/master/SocialAndLocalAccounts/SignUpOrSignin.xml), указывает путь взаимодействия пользователя, который Azure AD B2C будет выполняться. Найдите элемент **дефаултусержаурнэй** внутри [проверяющей стороны](../articles/active-directory-b2c/relyingparty.md). Обновите  **ReferenceId** в соответствии с идентификатором пути взаимодействия пользователя, в который добавлен поставщик удостоверений. 

В следующем примере для `CustomSignUpOrSignIn` пути взаимодействия пользователя параметр **ReferenceId** имеет значение `CustomSignUpOrSignIn` :

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="CustomSignUpSignIn" />
  ...
</RelyingParty>
```

### <a name="upload-the-custom-policy"></a>Передача настраиваемой политики

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. В разделе **Политики** выберите **Identity Experience Framework**.
1. Выберите **Отправить пользовательскую политику**, а затем отправьте два измененных файла политики в следующем порядке: политика расширения, например `TrustFrameworkExtensions.xml` , политика проверяющей стороны, например `SignUpSignIn.xml` .




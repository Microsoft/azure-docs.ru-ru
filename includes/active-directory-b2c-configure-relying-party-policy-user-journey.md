---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 01/18/2021
ms.author: mimart
ms.openlocfilehash: f94076f06fb13bae2a26e8ab6003d7574a2dacfd
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/20/2021
ms.locfileid: "98674250"
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

## <a name="test-your-custom-policy"></a>Тестирование пользовательской политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin`
1. Для **приложения** выберите ранее зарегистрированное веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.


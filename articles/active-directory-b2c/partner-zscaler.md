---
title: Руководство по настройке Azure Active Directory B2C с помощью Zscaler
titleSuffix: Azure AD B2C
description: Узнайте, как интегрировать проверку подлинности Azure AD B2C с Zscaler.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/09/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: dbeb3cd4fccf80f434e6c7ac08c658632f64b135
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102096859"
---
# <a name="tutorial-configure-zscaler-private-access-with-azure-active-directory-b2c"></a>Руководство. Настройка частного доступа Zscaler с помощью Azure Active Directory B2C

В этом руководстве вы узнаете, как интегрировать проверку подлинности Azure Active Directory B2C (Azure AD B2C) с [Zscaler Private Access (ZPA)](https://www.zscaler.com/products/zscaler-private-access). ZPA обеспечивает безопасный доступ к частным приложениям и ресурсам на основе политик без затрат, проблем или угроз безопасности виртуальной частной сети (VPN). Предложение безопасного гибридного доступа Zscaler позволяет получать нулевую атаку для приложений, ориентированных на потребителей, при объединении с Azure AD B2C.

## <a name="prerequisites"></a>Предварительные условия

Прежде чем начать, вам потребуется:

- Подписка Azure. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).  
- [Azure AD B2C клиент](./tutorial-create-tenant.md) , связанный с подпиской Azure.  
- [Подписка ZPA](https://azuremarketplace.microsoft.com/marketplace/apps/aad.zscalerprivateaccess?tab=Overview).

## <a name="scenario-description"></a>Описание сценария

Интеграция ZPA включает следующие компоненты:

- **Azure AD B2C**: поставщик удостоверений (IDP), отвечающий за проверку учетных данных пользователя. Он также отвечает за регистрацию нового пользователя.  
- **ZPA**. служба, отвечающая за защиту веб-приложения путем принудительного [доступа с нулевым доверием](https://www.microsoft.com/security/blog/2018/12/17/zero-trust-part-1-identity-and-access-management/#:~:text=Azure%20Active%20Directory%20%28Azure%20AD%29%20provides%20the%20strong%2C,to%20express%20their%20access%20requirements%20in%20simple%20terms.).  
- **Веб-приложение**: содержит службу, к которой пользователь пытается получить доступ.

На следующей схеме показано, как ZPA интегрируется с Azure AD B2C.

![Схема архитектуры Zscaler, в которой показано, как ZPA интегрируется с Azure AD B2C.](media/partner-zscaler/zscaler-architecture-diagram.png)

Последовательность описана в следующей таблице.

|Шаг | Описание |
| :-----:| :-----------|
| 1 | Пользователь попадает на пользовательский портал ZPA или приложение ZPA для доступа к браузеру.
| 2 | ZPA требует сведения о контексте пользователя, прежде чем он сможет решить, следует ли разрешить пользователю доступ к веб-приложению. Для проверки подлинности пользователя ZPA выполняет перенаправление SAML на страницу входа Azure AD B2C.  
| 3 | Пользователь поступает на страницу входа в Azure AB B2C. Новые пользователи подписываются для создания учетной записи, а существующие пользователи входят с помощью существующих учетных данных. Azure AD B2C проверяет удостоверение пользователя.
| 4 | После успешной проверки подлинности Azure AD B2C перенаправляет пользователя обратно в ZPA вместе с утверждением SAML. ZPA проверяет SAML assertion и задает контекст пользователя.
| 5 | ZPA оценивает политики доступа для пользователя. Если пользователю разрешен доступ к веб-приложению, подключение может проходить через.

## <a name="onboard-to-zpa"></a>Подключение к ZPA

В этом руководстве предполагается, что у вас уже есть Рабочая установка ZPA. Если вы приступите к работе с ZPA, см. [Пошаговое руководство по настройке ZPA](https://help.zscaler.com/zpa/step-step-configuration-guide-zpa).

## <a name="integrate-zpa-with-azure-ad-b2c"></a>Интеграция ZPA с Azure AD B2C

### <a name="step-1-configure-azure-ad-b2c-as-an-idp-on-zpa"></a>Шаг 1. Настройка Azure AD B2C в качестве IdP в ZPA

Чтобы настроить Azure AD B2C в качестве [IDP в ZPA](https://help.zscaler.com/zpa/configuring-idp-single-sign), выполните следующие действия.

1. Войдите на [портал администрирования ZPA](https://admin.private.zscaler.com).

1. Перейдите в раздел **Administration**  >  **IDP Configuration**(администрирование конфигурации).

1. Выберите **Add IDP Configuration (Добавить конфигурацию конфигурации**).

   Откроется панель **Добавление конфигурации IDP** .

   ![Снимок экрана: вкладка "сведения о IdP" на панели "Добавление конфигурации IdP".](media/partner-zscaler/add-idp-configuration.png)

1. Откройте вкладку **сведения о IDP** и выполните следующие действия.

   а. В поле **имя** введите **Azure AD B2C**.  
   b. В разделе **единый вход** выберите **пользователь**.  
   c. В раскрывающемся списке **домены** выберите домены проверки подлинности, которые необходимо связать с этим IDP.

1. Выберите **Далее**.

1. Перейдите на вкладку **метаданные SP** и выполните следующие действия.

   а. В разделе **URL-адрес поставщика услуг** скопируйте или запишите значение для последующего использования.  
   b. В разделе **идентификатор сущности поставщика услуг** скопируйте или запишите значение для последующего использования.

   ![Снимок экрана вкладки "метаданные SP" на панели "Добавление конфигурации IdP".](media/partner-zscaler/sp-metadata.png)

1. Выберите **приостановить**.

После того как вы настроили Azure AD B2C, оставшаяся часть конфигурации IdP возобновляется.

### <a name="step-2-configure-custom-policies-in-azure-ad-b2c"></a>Шаг 2. Настройка пользовательских политик в Azure AD B2C

>[!Note]
>Этот шаг необходим только в том случае, если вы еще не настроили пользовательские политики. Если у вас уже есть одна или несколько настраиваемых политик, этот шаг можно пропустить.

Сведения о настройке пользовательских политик в клиенте Azure AD B2C см. [в разделе Приступая к работе с пользовательскими политиками в Azure Active Directory B2C](./custom-policy-get-started.md).

### <a name="step-3-register-zpa-as-a-saml-application-in-azure-ad-b2c"></a>Шаг 3. Регистрация ZPA в качестве приложения SAML в Azure AD B2C

Сведения о настройке приложения SAML в Azure AD B2C см. в разделе [Регистрация приложения SAML в Azure AD B2C](./saml-service-provider.md). 

На шаге ["Отправка политики"](./saml-service-provider.md#upload-your-policy)скопируйте или запишите URL-адрес метаданных SAML IDP, используемый Azure AD B2C. Он понадобится вам позднее.

Следуйте инструкциям на шаге ["Настройка приложения в Azure AD B2C"](./saml-service-provider.md#configure-your-application-in-azure-ad-b2c). На шаге 4,2 Обновите свойства манифеста приложения следующим образом:

- Для **identifierUris**: используйте идентификатор сущности поставщика услуг, который вы скопировали или отметили ранее в шаге 1.6. b.  
- Для **самлметадатаурл**: пропустите это свойство, так как ZPA не размещает URL-адрес метаданных SAML.  
- Для **реплюрлсвистипе**: используйте URL-адрес поставщика услуг, который вы скопировали или отмечали ранее в шаге 1.6. a.  
- Для **логаутурл**: пропустите это свойство, так как ZPA не поддерживает URL-адрес выхода.

Остальные шаги не относятся к этому учебнику.

### <a name="step-4-extract-the-idp-saml-metadata-from-azure-ad-b2c"></a>Шаг 4. Извлечение метаданных IdP SAML из Azure AD B2C

Затем необходимо получить URL-адрес метаданных SAML в следующем формате:

```https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/Samlp/metadata```

Обратите внимание, что `<tenant-name>` — это имя клиента Azure AD B2C, а `<policy-name>` — это имя настраиваемой политики SAML, созданной на предыдущем шаге.

Например, URL-адрес может иметь вид `https://safemarch.b2clogin.com/safemarch.onmicrosoft.com/B2C_1A_signup_signin_saml//Samlp/metadata`.

Откройте веб-браузер и перейдите по URL-адресу метаданных SAML. Щелкните правой кнопкой мыши в любом месте страницы, выберите **Сохранить как** и сохраните файл на компьютере для использования на следующем шаге.

### <a name="step-5-complete-the-idp-configuration-on-zpa"></a>Шаг 5. Завершение настройки IdP на ZPA

Завершите [настройку IDP на портале администрирования ZPA](https://help.zscaler.com/zpa/configuring-idp-single-sign) , который был частично настроен ранее в шаге 1: Настройка Azure AD B2C как IDP на ZPA ".

1. На [портале администрирования ZPA](https://admin.private.zscaler.com)перейдите в раздел **Администрирование**  >  **IDP конфигурация**.

1. Выберите IdP, настроенный в шаге 1, и нажмите кнопку **возобновить**.

1. На панели **Добавить конфигурацию IDP** выберите вкладку **создать IDP** и выполните следующие действия.

   а. В разделе **файл метаданных IDP** передайте файл метаданных, сохраненный ранее на шаге 4: извлечение метаданных SAML IdP из Azure AD B2C.  
   b. Убедитесь, что для конфигурации IdP **включено** **состояние** .  
   В. Нажмите кнопку **Сохранить**.

   ![Снимок экрана: вкладка "Создание IdP" в области "Добавление конфигурации IdP".](media/partner-zscaler/create-idp.png)

## <a name="test-the-solution"></a>Тестирование решения

Перейдите на пользовательский портал ZPA или приложение для доступа к браузеру и протестируйте процесс регистрации или входа. Проверка должна привести к успешной проверке подлинности SAML.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- [Приступая к работе с пользовательскими политиками в Azure AD B2C](./custom-policy-get-started.md)
- [Регистрация приложения SAML в Azure AD B2C](./saml-service-provider.md)
- [Пошаговое руководств по настройке ZPA](https://help.zscaler.com/zpa/step-step-configuration-guide-zpa)
- [Настройка IdP для единого входа](https://help.zscaler.com/zpa/configuring-idp-single-sign)
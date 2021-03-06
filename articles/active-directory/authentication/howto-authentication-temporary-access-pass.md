---
title: Настройка временного прохода доступа в Azure AD для регистрации методов проверки подлинности, не защищенных паролем
description: Узнайте, как настроить и разрешить пользователям регистрировать методы проверки подлинности с незащищенным паролем с помощью временного прохода доступа.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 03/18/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: inbarckms
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0805ac84318a4fee98c30127ac80c0dac2b96309
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105558267"
---
# <a name="configure-temporary-access-pass-in-azure-ad-to-register-passwordless-authentication-methods-preview"></a>Настройка временного доступа в Azure AD для регистрации методов проверки подлинности, не прошедших проверку пароля (Предварительная версия)

Методы проверки подлинности без пароля, такие как вход в систему с помощью FIDO2 и безпарольного входа в приложение Microsoft Authenticator, позволяют пользователям безопасно выполнять вход без пароля. Пользователи могут выполнять загрузку методов, не имеющих паролей, одним из двух способов:

- Использование существующих методов многофакторной идентификации Azure AD 
- Использование временного прохода доступа (TAP) 

Временный проход доступа — это секретный код, выданный администратором, который удовлетворяет требованиям строгой проверки подлинности и может использоваться для подключения других методов проверки подлинности, включая безпарольные. Временный проход также упрощает восстановление, когда пользователь теряет или забыл свой надежный фактор проверки подлинности, такой как FIDO2ный ключ безопасности или приложение Microsoft Authenticator, но для регистрации новых методов проверки подлинности необходимо войти в систему.


В этой статье показано, как включить и использовать временный проход доступа в Azure AD с помощью портал Azure. Эти действия также можно выполнить с помощью интерфейсов API. 

>[!NOTE]
>Временный проход доступа в настоящее время находится в общедоступной предварительной версии. Некоторые функции могут не поддерживаться или иметь ограниченные возможности. 

## <a name="enable-the-temporary-access-pass-policy"></a>Включение политики временного прохода доступа

Временная политика для прохода доступа определяет такие параметры, как время существования проходов, созданных в клиенте, или пользователей и групп, которые могут использовать временный проход доступа для входа. Прежде чем любой пользователь сможет войти с временным проходом, необходимо включить политику метода проверки подлинности и выбрать, какие пользователи и группы могут выполнять вход с помощью временного прохода доступа.
Несмотря на то, что можно создать временный проход доступа для любого пользователя, с ним могут входить только те, которые входят в эту политику.

Владельцы роли администратора политики для глобального администратора и метода проверки подлинности могут обновить политику метода проверки подлинности во временном доступе.
Настройка политики метода проверки подлинности для временного доступа:

1. Войдите в портал Azure в качестве глобального администратора и щелкните **Azure Active Directory**  >    >  **методы проверки подлинности** безопасности  >  **временный доступ**.
1. Нажмите кнопку **Да** , чтобы включить политику, выбрать пользователей, к которым применена политика, и все **Общие** параметры.

   ![Снимок экрана: Включение политики метода проверки подлинности для временного доступа](./media/how-to-authentication-temporary-access-pass/policy.png)

   Значение по умолчанию и диапазон допустимых значений описаны в следующей таблице.


   | Параметр | Значения по умолчанию | Допустимые значения | Комментарии |
   |---|---|---|---|
   | Минимальное время существования | 1 час | 10 – 43200 минут (30 дней) | Минимальное число минут, в течение которых действует временный проход доступа. |
   | Максимальный срок жизни | 24 часа | 10 – 43200 минут (30 дней) | Максимальное число минут, в течение которых действует временный проход доступа. |
   | Время существования по умолчанию | 1 час | 10 – 43200 минут (30 дней) | Значения по умолчанию могут быть переопределены отдельными проходами в пределах минимального и максимального времени существования, настроенного политикой. |
   | Однократное использование | False | True/false | Если для политики задано значение false, передача в клиент может быть использована либо один раз, либо несколько раз в течение срока действия (максимальное время существования). При принудительном использовании в политике временных поправок доступа все этапы, созданные в клиенте, будут создаваться как одноразовые. |
   | Длина | 8 | 8-48 символов | Определяет длину секретного кода. |

## <a name="create-a-temporary-access-pass-in-the-azure-ad-portal"></a>Создание временного прохода доступа на портале Azure AD

После включения политики можно создать временный проход доступа для пользователя в Azure AD. Эти роли могут выполнять следующие действия, связанные с временным проходом доступа.

- Глобальный администратор может создавать, удалять и просматривать временный проход доступа для любого пользователя (за исключением самих себя)
- Администраторы привилегированной проверки подлинности могут создавать, удалять и просматривать временный проход доступа для администраторов и участников (за исключением самих себя).
- Администраторы проверки подлинности могут создавать, удалять и просматривать временный проход доступа к членам (за исключением самих себя)
- Глобальный администратор может просматривать сведения о ходе временного прохода доступа для пользователя (без считывания самого кода).

Чтобы создать временный проход доступа, сделайте следующее:

1. Войдите на портал в качестве глобального администратора, привилегированного администратора проверки подлинности или администратора проверки подлинности. 
1. Щелкните **Azure Active Directory**, перейдите к пользователям, выберите пользователя, например *Крис Green*, а затем выберите **методы проверки подлинности**.
1. При необходимости выберите вариант, чтобы **использовать новые методы проверки подлинности пользователя**.
1. Выберите параметр, чтобы **добавить методы проверки подлинности**.
1. Ниже **выберите метод**, щелкните **временный проход доступа (Предварительная версия)**.
1. Определите пользовательское время или длительность активации и нажмите кнопку **Добавить**.

   ![Снимок экрана создания временного прохода доступа](./media/how-to-authentication-temporary-access-pass/create.png)

1. После добавления отображаются сведения о временном проходе доступа. Запишите фактическое значение временного прохода доступа. Это значение предоставляется пользователю. Это значение нельзя просмотреть после нажатия кнопки **ОК**.
   
   ![Снимок экрана сведений о временном проходе доступа](./media/how-to-authentication-temporary-access-pass/details.png)

## <a name="use-a-temporary-access-pass"></a>Использование временного прохода доступа

Чаще всего для временного прохода доступа пользователь регистрирует сведения о проверке подлинности во время первого входа без необходимости выполнять дополнительные запросы безопасности. Методы проверки подлинности регистрируются в [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) . Пользователи также могут обновлять существующие методы проверки подлинности здесь.

1. Откройте веб-браузер в [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) .
1. Введите имя участника-пользователя учетной записи, для которой вы создали временный проход доступа, например *tapuser@contoso.com* .
1. Если пользователь включен в политику временного доступа, он увидит экран, чтобы ввести временный проход доступа.
1. Введите временный этап доступа, который отображался в портал Azure.

   ![Снимок экрана, посвященный вводу временного прохода доступа](./media/how-to-authentication-temporary-access-pass/enter.png)

>[!NOTE]
>Для федеративных доменов предпочтительным является временный проход доступа по Федерации. Пользователь с временным проходом доступа завершит проверку подлинности в Azure AD и не будет перенаправлен на федеративный поставщик удостоверений (IdP).

Теперь пользователь вошел в систему и может обновить или зарегистрировать метод, например ключ безопасности FIDO2. Пользователям, которые обновляют методы проверки подлинности из-за потери учетных данных или устройства, следует убедиться, что они удаляют старые методы проверки подлинности.

Пользователи также могут использовать их временный проход для регистрации для входа в систему без пароля непосредственно из приложения для проверки подлинности. Дополнительные сведения см. в разделе [Добавление рабочей или учебной учетной записи в приложение Microsoft Authenticator](../user-help/user-help-auth-app-add-work-school-account.md).

![Снимок экрана: как ввести временный проход доступа с помощью рабочей или учебной учетной записи](./media/how-to-authentication-temporary-access-pass/enter-work-school.png)

## <a name="delete-a-temporary-access-pass"></a>Удаление временного прохода доступа

Невозможно использовать временный проход доступа с истекшим сроком действия. В **методах проверки подлинности** для пользователя в столбце **сведения** отображается время истечения временного доступа. Вы можете удалить временный проход доступа с истекшим сроком, выполнив следующие действия.

1. На портале Azure AD перейдите к **пользователям**, выберите пользователя, например *Коснитесь пользователя*, а затем выберите **методы проверки подлинности**.
1. В правой части метода проверки подлинности **временного доступа (Предварительная версия)** , показанного в списке, выберите **Удалить**.

## <a name="replace-a-temporary-access-pass"></a>Замена временного прохода доступа 

- Пользователь может иметь только один временный проход доступа. Секретный код может использоваться во время начала и окончания временного прохода доступа.
- Если пользователю требуется новый временный проход доступа:
  - Если существующий временный этап доступа допустим, администратору необходимо удалить существующий временный проход доступа и создать новый проход для пользователя. Удаление допустимого временного прохода доступа приведет к отмене сеансов пользователя. 
  - Если срок действия существующего временного доступа истек, новый временный проход доступа переопределит существующий временный проход доступа и не будет отзывать сеансы пользователя.

Дополнительные сведения о стандарте NIST для адаптации и восстановления см. в разделе [Специальная публикация NIST 800-63A](https://pages.nist.gov/800-63-3/sp800-63a.html#sec4).

## <a name="limitations"></a>Ограничения

Учитывайте следующие ограничения.

- При использовании одновременного временного прохода доступа для регистрации метода без пароля, например FIDO2 или телефона, пользователь должен выполнить регистрацию в течение 10 минут после входа с помощью однократного временного прохода доступа. Это ограничение не распространяется на временный проход доступа, который можно использовать более одного раза.
- Гостевые пользователи не могут войти с временным проходом доступа.
- Пользователям в области действия для политики регистрации самостоятельного сброса пароля (SSPR) будет необходимо зарегистрировать один из методов SSPR после того, как они вошли с временным проходом доступа. Если пользователь будет использовать только ключ FIDO2, исключите его из политики SSPR или отключите политику регистрации SSPR. 
- Временный проход доступа не может использоваться с расширением сервера политики сети (NPS) и адаптером службы федерации Active Directory (AD FS) (AD FS).
- Если на клиенте включен простой единый вход, пользователям предлагается ввести пароль. Вместо этого для входа в систему с временным проходом доступа будет доступен параметр **использовать временный проход доступа** .

  ![Снимок экрана: вместо этого используйте временный проход доступа](./media/how-to-authentication-temporary-access-pass/alternative.png)

## <a name="troubleshooting"></a>Устранение неполадок    

- Если временный проход доступа не предлагается пользователю во время входа, проверьте следующее:
  - Пользователь находится в области применения политики метода проверки подлинности для временного доступа.
  - У пользователя есть допустимый временный проход, и если он используется однократно, он еще не использовался.
- Если **временный вход в систему был заблокирован из-за того** , что во время входа отображается политика учетных данных пользователя с временным прохождением прав доступа, проверьте следующее:
  - Пользователь имеет временный проход по временному доступу, а политика метода проверки подлинности требует однократного временного прохода доступа.
  - Одноразовый временный проход доступа уже использовался.

## <a name="next-steps"></a>Дальнейшие действия

- [Планирование развертывания проверки подлинности с незащищенным паролем в Azure Active Directory](howto-authentication-passwordless-deployment.md)


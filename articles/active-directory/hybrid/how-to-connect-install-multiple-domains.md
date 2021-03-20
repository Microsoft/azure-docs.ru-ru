---
title: Поддержка нескольких доменов в Azure AD Connect
description: В этом документе описывается настройка и настройка нескольких доменов верхнего уровня с помощью Microsoft 365 и Azure AD.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 5595fb2f-2131-4304-8a31-c52559128ea4
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/31/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53a0da5b5db21c9a543d39d1b252b0b4c64e2a56
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91306367"
---
# <a name="multiple-domain-support-for-federating-with-azure-ad"></a>Поддержка нескольких доменов для федерации с Azure AD
В следующей документации приводятся рекомендации по использованию нескольких доменов верхнего уровня и поддоменов при интеграции с Microsoft 365 или доменами Azure AD.

## <a name="multiple-top-level-domain-support"></a>Поддержка нескольких доменов верхнего уровня
Для создания федерации нескольких доменов верхнего уровня с Azure AD необходимо выполнить некоторые дополнительные настройки, которые не требуются при создании федерации с одним доменом верхнего уровня.

При добавлении домена в федерацию с Azure AD необходимо установить несколько свойств для этого домена в Azure.  Важным свойством является IssuerUri.  Это свойство, содержащее универсальный код ресурса (URI), который используется Azure AD для идентификации домена, с которым связан маркер.  URI не должен разрешаться, но обязан быть действительным.  По умолчанию Azure AD устанавливает для этого универсального кода ресурса (URI) значение идентификатора службы федерации в локальной конфигурации службы AD FS.

> [!NOTE]
> Идентификатор службы федерации — это URI, который уникально идентифицирует службу федерации.  Служба федерации — это экземпляр AD FS, который работает как служба токенов безопасности.
>
>

Для просмотра IssuerUri можно воспользоваться командой PowerShell `Get-MsolDomainFederationSettings -DomainName <your domain>`.

![Снимок экрана, показывающий результаты после ввода команды "Get-MsolDomainFederationSettings" в PowerShell.](./media/how-to-connect-install-multiple-domains/MsolDomainFederationSettings.png)

Проблема возникает при добавлении нескольких доменов верхнего уровня.  Предположим, что у вас настроена федерация между Azure AD и локальной средой.  В этом документе используется домен bmcontoso.com.  Теперь добавляется второй домен верхнего уровня, bmfabrikam.com.

![Снимок экрана, показывающий несколько доменов верхнего уровня](./media/how-to-connect-install-multiple-domains/domains.png)

Пр попытке преобразовать домен bmfabrikam.com в федеративный домен происходит ошибка.  Причина в том, что в Azure AD существует ограничение, при котором свойство IssuerUri не может иметь одно и то же значение для нескольких доменов.  

![Снимок экрана, на котором показана ошибка Федерации в PowerShell.](./media/how-to-connect-install-multiple-domains/error.png)

### <a name="supportmultipledomain-parameter"></a>Параметр SupportMultipleDomain
Чтобы обойти это ограничение, необходимо добавить другой IssuerUri. Это можно сделать с помощью параметра `-SupportMultipleDomain`.  Этот параметр используется со следующими командлетами:

* `New-MsolFederatedDomain`
* `Convert-MsolDomaintoFederated`
* `Update-MsolFederatedDomain`

Этот параметр заставляет Azure AD настроить IssuerUri таким образом, чтобы он был основан на имени домена.  Значение IssuerUri будет уникальным в различных каталогах в Azure AD.  Использование параметра позволяет успешно выполнить команду PowerShell.

![Снимок экрана, на котором показано успешное выполнение команды PowerShell.](./media/how-to-connect-install-multiple-domains/convert.png)

Просмотрев параметры домена bmfabrikam.com, вы увидите следующее:

![Снимок экрана, на котором показаны параметры для домена "bmfabrikam.com".](./media/how-to-connect-install-multiple-domains/settings.png)

`-SupportMultipleDomain` не изменяет другие конечные точки, в которых адрес службы федерации по-прежнему задан как adfs.bmcontoso.com.

Параметр `-SupportMultipleDomain` также гарантирует, что система AD FS содержит правильное значение издателя для маркеров, выпущенных для Azure AD. Для задания этого значения извлекается доменная часть UPN пользователей и устанавливается в свойстве IssuerURI домена, т. е. https://{суффикс UPN}/adfs/services/trust.

Поэтому во время проверки подлинности в Azure AD или Microsoft 365 элемент IssuerUri в маркере пользователя используется для нахождение домена в Azure AD. Если совпадение не найдено, аутентификация завершится ошибкой.

Например, если UPN пользователя имеет значение bsimon@bmcontoso.com, элементу IssuerUri в маркере, который выпускается AD FS, будет присвоено значение `http://bmcontoso.com/adfs/services/trust`. Этот элемент будет соответствовать конфигурации Azure AD, и аутентификация пройдет успешно.

Ниже приведено пользовательское правило утверждения, которое реализует эту логику.

```
c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)", "http://${domain}/adfs/services/trust/"));
```


> [!IMPORTANT]
> Чтобы использовать параметр -SupportMultipleDomain при попытке добавить новые или преобразовать существующие домены, необходимо заранее настроить федеративное отношение доверия для этих доменов.
>
>

## <a name="how-to-update-the-trust-between-ad-fs-and-azure-ad"></a>Обновление отношения доверия между службами федерации Active Directory и Azure AD
Если вы не настроили федеративное отношение доверия между AD FS и своим экземпляром Azure AD, может потребоваться создать это отношение доверия снова.  Причина этого в том, что при изначальной настройке без параметра `-SupportMultipleDomain` в IssuerUri записывается значение по умолчанию.  На снимке экрана ниже вы увидите, что для IssuerUri задано значение `https://adfs.bmcontoso.com/adfs/services/trust`.

Если добавить новый домен на портале Azure AD, а затем попытаться преобразовать его с помощью `Convert-MsolDomaintoFederated -DomainName <your domain>`, то произойдет следующая ошибка.

![Снимок экрана, на котором показана ошибка Федерации в PowerShell после попытки преобразования нового домена с помощью команды "Convert-MsolDomaintoFederated".](./media/how-to-connect-install-multiple-domains/trust1.png)

При попытке добавить `-SupportMultipleDomain` произойдет следующая ошибка.

![Снимок экрана, на котором показана ошибка Федерации после добавления параметра "-SupportMultipleDomain".](./media/how-to-connect-install-multiple-domains/trust2.png)

Попытка просто запустить `Update-MsolFederatedDomain -DomainName <your domain> -SupportMultipleDomain` в исходном домене также приведет к ошибке.

![Ошибка федерации](./media/how-to-connect-install-multiple-domains/trust3.png)

Выполните указанные ниже действия для добавления дополнительного домена верхнего уровня.  Если домен уже был добавлен и параметр `-SupportMultipleDomain` не использовался, начните с удаления и обновления исходного домена.  Если вы не еще добавили домен верхнего уровня, можете начать с добавления домена с помощью PowerShell или Azure AD Connect.

Выполните следующие действия для удаления доверия Microsoft Online и обновления исходного домена.

1. На сервере федерации AD FS откройте **Управление AD FS**
2. Разверните **Отношения доверия** и **Отношения доверия проверяющей стороны** слева.
3. Удалите запись **Платформа идентификации Microsoft Office 365** справа.
   ![Удалить Microsoft Online](./media/how-to-connect-install-multiple-domains/trust4.png)
4. На компьютере, на котором установлен [модуль Azure Active Directory для Windows PowerShell](/previous-versions/azure/jj151815(v=azure.100)), выполните следующую команду: `$cred=Get-Credential`.  
5. Введите имя пользователя и пароль глобального администратора домена Azure AD, для которого настраивается федерация.
6. В PowerShell введите `Connect-MsolService -Credential $cred`.
7. В PowerShell введите `Update-MSOLFederatedDomain -DomainName <Federated Domain Name> -SupportMultipleDomain`.  Это изменение предназначено для исходного домена.  С приведенными выше доменами мы получим следующее: `Update-MsolFederatedDomain -DomainName bmcontoso.com -SupportMultipleDomain`.

Выполните следующие действия для добавления нового домена верхнего уровня с помощью PowerShell.

1. На компьютере, на котором установлен [модуль Azure Active Directory для Windows PowerShell](/previous-versions/azure/jj151815(v=azure.100)), выполните следующую команду: `$cred=Get-Credential`.  
2. Введите имя пользователя и пароль глобального администратора домена Azure AD, для которого настраивается федерация.
3. В PowerShell введите `Connect-MsolService -Credential $cred`.
4. В PowerShell введите `New-MsolFederatedDomain –SupportMultipleDomain –DomainName`.

Выполните следующие действия для добавления нового домена верхнего уровня с помощью Azure AD Connect.

1. Запустите Azure AD Connect на рабочем столе или в меню "Пуск".
2. Щелкните снимок экрана "добавить дополнительный домен Azure AD" ![ , на котором отображается страница "дополнительные задачи" с выбранным параметром "добавить дополнительный домен Azure AD".](./media/how-to-connect-install-multiple-domains/add1.png)
3. Введите учетные данные Azure AD и Active Directory.
4. Выберите второй домен, который необходимо настроить для федерации.
   ![Добавить дополнительный домен Azure AD](./media/how-to-connect-install-multiple-domains/add2.png)
5. Нажмите "Установить".

### <a name="verify-the-new-top-level-domain"></a>Проверка нового домена верхнего уровня
С помощью команды PowerShell `Get-MsolDomainFederationSettings -DomainName <your domain>`можно просмотреть обновленный IssuerUri.  На приведенном ниже снимке экрана показано, что настройки федерации для исходного домена `http://bmcontoso.com/adfs/services/trust` были обновлены.

![Снимок экрана, на котором показаны параметры Федерации, обновленные в исходном домене.](./media/how-to-connect-install-multiple-domains/MsolDomainFederationSettings.png)

И для IssuerUri нового домена задано значение `https://bmfabrikam.com/adfs/services/trust`.

![Get-MsolDomainFederationSettings](./media/how-to-connect-install-multiple-domains/settings2.png)

## <a name="support-for-subdomains"></a>Поддержка поддоменов
При добавлении поддомена он унаследует параметры родительского домена из-за особенностей обработки доменов Azure AD.  Поэтому значение IssuerUri должно совпадать со значением этого параметра у родительских элементов.

Давайте предположим, что у меня был домен bmcontoso.com, а затем я добавил поддомен corp.bmcontoso.com.  IssuerUri для пользователя из corp.bmcontoso.com должен быть **`http://bmcontoso.com/adfs/services/trust`** .  Однако стандартное правило, реализованное выше для Azure AD, создаст маркер с издателем как **`http://corp.bmcontoso.com/adfs/services/trust`** . , который не будет соответствовать необходимому значению для домена, и аутентификация завершится неудачно.

### <a name="how-to-enable-support-for-subdomains"></a>Как включить поддержку для поддоменов
Чтобы обойти это поведение, необходимо обновить отношение доверия проверяющей стороны AD FS для Microsoft Online.  Для этого необходимо настроить пользовательское правило утверждения так, чтобы оно удаляло поддомены из суффикса UPN пользователя при создании настраиваемого значения элемента Issuer.

Это можно сделать с помощью следующего утверждения:

```    
c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^.*@([^.]+\.)*?(?<domain>([^.]+\.?){2})$", "http://${domain}/adfs/services/trust/"));
```

[!NOTE]
Последнее число в регулярном выражении задает количество родительских доменов в корневом домене. Так как здесь используется bmcontoso.com, необходимы два родительских домена. Если бы нужно было сохранить три родительских домена (т. е. corp.bmcontoso.com), использовалось бы число три. Впоследствии может быть задан диапазон. Совпадение всегда будет выполняться для соответствия максимальному количеству доменов. "{2,3}" означает, что два домена будут сопоставлены с тремя (т. е. bmfabrikam.com и corp.bmcontoso.com).

Используйте следующие инструкции для добавления пользовательского утверждения для поддержки поддоменов.

1. Откройте оснастку управления AD FS.
2. Щелкните правой кнопкой мыши отношение доверия Microsoft Online RP и выберите "Изменить правила утверждений".
3. Выберите третье правило утверждения и замените ![Изменить правила утверждений](./media/how-to-connect-install-multiple-domains/sub1.png).
4. Замените текущее утверждение:

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)","http://${domain}/adfs/services/trust/"));
   ```
    на

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/UPN"] => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, "^.*@([^.]+\.)*?(?<domain>([^.]+\.?){2})$", "http://${domain}/adfs/services/trust/"));
   ```

    ![Заменить утверждение](./media/how-to-connect-install-multiple-domains/sub2.png)

5. Нажмите кнопку "ОК".  Нажмите кнопку "Применить".  Нажмите кнопку "ОК".  Откройте оснастку управления AD FS.

## <a name="next-steps"></a>Дальнейшие действия
После установки Azure AD Connect можно [проверить установку и назначить лицензии](how-to-connect-post-installation.md).

Дополнительные сведения о функциях, которые были включены в процессе установки, см. в следующих статьях: [Azure AD Connect: автоматическое обновление](how-to-connect-install-automatic-upgrade.md), [Синхронизация Azure AD Connect: предотвращение случайного удаления](how-to-connect-sync-feature-prevent-accidental-deletes.md) и [Использование Azure AD Connect Health для синхронизации](how-to-connect-health-sync.md).

Дополнительные сведения см. в статье [Синхронизация Azure AD Connect: планировщик](how-to-connect-sync-feature-scheduler.md).

Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
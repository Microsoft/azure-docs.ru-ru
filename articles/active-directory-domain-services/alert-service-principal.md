---
title: Разрешение оповещений от субъекта-службы в доменных службах Azure AD | Документация Майкрософт
description: Сведения о том, как устранять неполадки по оповещениям конфигурации субъекта-службы в доменных службах Azure AD
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: f168870c-b43a-4dd6-a13f-5cfadc5edf2c
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 15dba02d3db45d663b8ba0aa7eb93bbc3f388532
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101645880"
---
# <a name="known-issues-service-principal-alerts-in-azure-active-directory-domain-services"></a>Известные проблемы: Оповещения субъекта-службы в службах Azure Active Directory

[Субъекты-службы](../active-directory/develop/app-objects-and-service-principals.md) — это приложения, которые Платформа Azure использует для управления, обновления и обслуживания управляемого домена Azure Active Directory доменных служб (Azure AD DS). Если субъект-служба удалена, затронута функциональность в управляемом домене.

Эта статья поможет вам устранить неполадки и ошибки конфигурации субъекта-службы, которые приводят к появлению оповещений.

## <a name="alert-aadds102-service-principal-not-found"></a>Оповещение AADDS102: не удалось найти субъект-службу

### <a name="alert-message"></a>Текст оповещения

*Субъект-служба, необходимая для правильной работы доменных служб Azure AD, удалена из каталога Azure AD. Эта конфигурация влияет на возможность корпорации Майкрософт наблюдать за управляемым доменом, управлять им, обновлять и синхронизировать его.*

При удалении обязательного субъекта-службы платформа Azure не сможет выполнять автоматизированные задачи управления. Управляемый домен может неправильно применять обновления или создавать резервные копии.

### <a name="check-for-missing-service-principals"></a>Проверка отсутствующих субъектов-служб

Чтобы проверить, какой субъект-служба отсутствует и должен быть создан повторно, выполните следующие действия.

1. На портале Azure в области навигации слева выберите **Azure Active Directory**.
1. Выберите **Корпоративные приложения**. Выберите *Все приложения* в раскрывающемся меню **Тип приложения** и щелкните **Применить**.
1. Найдите каждый из следующих идентификаторов приложений. Если существующее приложение не найдено, выполните инструкции из раздела *Решение*, чтобы создать субъект-службу или повторно зарегистрировать пространство имен.

    | Идентификатор приложения | Решение |
    | :--- | :--- |
    | 2565bd9d-da50-47d4-8b85-4c97f669dc36 | [Восстановление отсутствующего субъекта-службы](#recreate-a-missing-service-principal) |
    | 443155a6-77f3-45e3-882b-22b3a8d431fb | [Повторная регистрация пространства имен Microsoft.AAD](#re-register-the-microsoft-aad-namespace) |
    | abba844e-bc0e-44b0-947a-dc74e5d09022 | [Повторная регистрация пространства имен Microsoft.AAD](#re-register-the-microsoft-aad-namespace) |
    | d87dcbc6-a371-462e-88e3-28ad15ec4e64 | [Повторная регистрация пространства имен Microsoft.AAD](#re-register-the-microsoft-aad-namespace) |

### <a name="recreate-a-missing-service-principal"></a>Восстановление отсутствующего субъекта-службы

Если в каталоге Azure AD отсутствует приложение с идентификатором *2565bd9d-da50-47d4-8b85-4c97f669dc36*, выполните следующие действия в Azure AD PowerShell. Дополнительные сведения см. в статье [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).

1. При необходимости установите модуль Azure AD PowerShell и импортируйте его следующим образом:

    ```powershell
    Install-Module AzureAD
    Import-Module AzureAD
    ```

1. Теперь восстановите субъект-службу с помощью командлета [New-AzureADServicePrincipal][New-AzureAdServicePrincipal]:

    ```powershell
    New-AzureAdServicePrincipal -AppId "2565bd9d-da50-47d4-8b85-4c97f669dc36"
    ```

Работоспособность управляемого домена автоматически обновляется в течение двух часов и удаляет оповещение.

### <a name="re-register-the-microsoft-aad-namespace"></a>Повторная регистрация пространства имен Microsoft.AAD

Если в каталоге Azure AD отсутствует приложение с идентификатором *443155a6-77F3-45e3-882b-22b3a8d431fb*, *abba844e-bc0e-44b0-947a-dc74e5d09022* или *d87dcbc6-a371-462e-88e3-28ad15ec4e64*, выполните следующие действия, чтобы повторно зарегистрировать поставщик ресурсов *Microsoft.AAD*.

1. На портале Azure найдите и выберите **Подписки**.
1. Выберите подписку, связанную с управляемым доменом.
1. На панели навигации слева выберите **Поставщики ресурсов**.
1. Найдите здесь *Microsoft.AAD* и щелкните **Повторная регистрация**.

Работоспособность управляемого домена автоматически обновляется в течение двух часов и удаляет оповещение.

## <a name="alert-aadds105-password-synchronization-application-is-out-of-date"></a>Оповещение AADDS105: приложение синхронизации паролей устарело

### <a name="alert-message"></a>Текст оповещения

*Субъект-служба с ИДЕНТИФИКАТОРом приложения "d87dcbc6-a371-462e-88e3-28ad15ec4e64" был удален, а затем создан повторно. Отдых оставляет за неправильные разрешения на ресурсы доменных служб Azure AD, необходимые для обслуживания управляемого домена. Может повлиять на синхронизацию паролей в управляемом домене.*

Azure AD DS автоматически синхронизирует учетные записи и учетные данные пользователей из Azure AD. Если возникнут проблемы с приложением Azure AD, которое выполняет этот процесс, синхронизация учетных данных между Azure AD DS и Azure AD не будет выполнена.

### <a name="resolution"></a>Решение

Чтобы повторно создать приложение Azure AD для синхронизации учетных данных, выполните следующие действия с помощью Azure AD PowerShell. Дополнительные сведения см. в статье об [установке модуля Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).

1. При необходимости установите модуль Azure AD PowerShell и импортируйте его следующим образом:

    ```powershell
    Install-Module AzureAD
    Import-Module AzureAD
    ```

2. Теперь удалите старое приложение и объект с помощью следующих командлетов PowerShell:

    ```powershell
    $app = Get-AzureADApplication -Filter "IdentifierUris eq 'https://sync.aaddc.activedirectory.windowsazure.com'"
    Remove-AzureADApplication -ObjectId $app.ObjectId
    $spObject = Get-AzureADServicePrincipal -Filter "DisplayName eq 'Azure AD Domain Services Sync'"
    Remove-AzureADServicePrincipal -ObjectId $spObject
    ```

После удаления обоих приложений платформа Azure автоматически восстановит их и попытается возобновить синхронизацию паролей. Работоспособность управляемого домена автоматически обновляется в течение двух часов и удаляет оповещение.

## <a name="next-steps"></a>Дальнейшие действия

Если проблемы сохраняются, [откройте запрос на поддержку в Azure][azure-support] для получения дополнительной помощи в устранении неполадок.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md

<!-- EXTERNAL LINKS -->
[New-AzureAdServicePrincipal]: /powershell/module/AzureAD/New-AzureADServicePrincipal

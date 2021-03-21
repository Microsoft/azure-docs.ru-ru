---
title: Руководство по устранению неполадок Аттестации Azure
description: Пошаговое решение проблем с типичными замеченными проблемами
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 07/20/2020
ms.author: mbaldwin
ms.openlocfilehash: 3ae3e12c11f194b3efcc149382dc952bd74d38b5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97704322"
---
# <a name="microsoft-azure-attestation-troubleshooting-guide"></a>Руководство по устранению неполадок аттестации Microsoft Azure

Обработка ошибок в аттестации Azure реализуется после [рекомендаций корпорации майкрософт REST API](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#7102-error-condition-responses). Ответ об ошибке, возвращенный API аттестации Azure, содержит код состояния HTTP и пары "имя-значение" с именами "Code" и "Message". Значение Code является понятным для человека и является индикатором типа ошибки. Значение «Message» часто помогает пользователю и предоставляет подробные сведения об ошибке.

Если проблема не устранена в этой статье, можно также отправить запрос на поддержку Azure на [странице поддержки Azure](https://azure.microsoft.com/support/options/).

Ниже приведены некоторые примеры ошибок, возвращаемых аттестацией Azure.

## <a name="1-http401--unauthorized-exception"></a>1. HTTP – 401: неавторизованное исключение

### <a name="http-status-code"></a>Код состояния HTTP
401

**Код ошибки** Запрещен

**Примеры сценариев**
  - Сбой аттестации, если пользователю не назначена роль читателя аттестации
  - Не удалось управлять политиками аттестации, так как пользователь не назначен с помощью соответствующих ролей
  - Не удалось управлять подписыванием политик аттестации, так как пользователь не назначен с помощью соответствующих ролей

Пользователь с ролью читателя, пытающийся изменить политику аттестации в PowerShell 

  ```powershell
  Set-AzAttestationPolicy : Operation returned HTTP Status Code 401
At line:1 char:1
+ Set-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy
  ```

**Действия по устранению неполадок**

Чтобы просмотреть политики аттестации и подписи политик, пользователю Azure AD требуется разрешение "действия":
- Microsoft.Attestation/attestationProviders/attestation/read

  Это разрешение может быть назначено пользователю AD с помощью такой роли, как "владелец" (разрешения с подстановочными знаками) или "читатель" (разрешения с подстановочными знаками) или "читатель аттестации" (только определенные разрешения для аттестации Azure).

Чтобы добавить или удалить подписывающих политик или настроить политики, пользователю Azure AD требуются следующие разрешения для действий:
- Microsoft.Attestation/attestationProviders/attestation/write
- Microsoft.Attestation/attestationProviders/attestation/delete

  Эти разрешения могут назначаться пользователю AD с помощью такой роли, как "владелец" (разрешения с подстановочными знаками), "участник" (разрешения с подстановочными знаками) или "участник аттестации" (только определенные разрешения для аттестации Azure).

Клиенты могут применять для аттестации поставщик по умолчанию или создавать собственные поставщики с настраиваемыми политиками. Чтобы отправить запросы аттестации настраиваемым поставщикам аттестации, для пользователя требуется "владелец" (разрешения с подстановочными знаками) или "читатель" (разрешения с подстановочными знаками) или "читатель аттестации". Поставщики по умолчанию доступны для любого пользователя Azure AD.

Чтобы проверить роли в PowerShell, выполните следующую команду:

а. Запустите PowerShell и войдите в Azure с помощью командлета Connect-Азаккаунт.

b. Проверка параметров назначения ролей Azure


  ```powershell
  $c = Get-AzContext
  Get-AzRoleAssignment -ResourceGroupName $attestationResourceGroup -ResourceName $attestationProvider -ResourceType Microsoft.Attestation/attestationProviders -SignInName $c.Account.Id
  ```

  Отобразятся примерно следующие сведения:

  ```
  RoleAssignmentId   :/subscriptions/subscriptionId/providers/Microsoft.Authorization/roleAssignments/roleAssignmentId
  
  Scope              : /subscriptions/subscriptionId
  
  DisplayName        : displayName
  
  SignInName         : signInName
  
  RoleDefinitionName : Reader
  
  RoleDefinitionId   : roleDefinitionId
  
  ObjectId           : objectid
  
  ObjectType         : User
  
  CanDelegate        : False
 
  ```

c. Если в списке нет нужного назначения роли, следуйте инструкциям в этой [статье](../role-based-access-control/role-assignments-powershell.md) .

## <a name="2-http--400-errors"></a>2. HTTP — ошибки 400

### <a name="http-status-code"></a>Код состояния HTTP
400

Существует несколько причин, по которым запрос может вернуть 400. Ниже приведены некоторые примеры ошибок, возвращаемых API аттестации Azure.

### <a name="21-attestation-failure-due-to-policy-evaluation-errors"></a>2.1. Сбой аттестации из-за ошибок оценки политики

Политика аттестации включает правила авторизации и правила выдачи. Свидетельство анклава оценивается на основе правил авторизации. Правила выдачи определяют утверждения, включаемые в токен аттестации. Если утверждения в анклава свидетельства не соответствуют правилам авторизации, то вызовы аттестации будут возвращать ошибку оценки политики. 

**Код ошибки** полициевалуатионеррор

**Примеры сценариев** Когда утверждения в анклава quote не соответствуют правилам авторизации политики аттестации

```
Native operation failed with 65518: G:\Az\security\Attestation\src\AttestationServices\Instance\NativePolicyWrapper\NativePolicyEngine.cpp(168)\(null)!00007FF801762308: (caller: 00007FF80143DCC8) Exception(0) 83FFFFEE Policy Evaluation Error has occurred Msg:[Policy Engine Exception: A Deny claim was issued, authorization failed.]

G:\Az\security\Attestation\src\AttestationServices\Instance\Enclave\api.cpp(840)\(null)!00007FF801739FF3: (caller: 00007FF801232801) LogHr(0) 83FFFFEE Policy Evaluation Error has occurred Msg:[Unhandled Enclave Exception: "Policy Evaluation Error has occurred"]

```

**Действия по устранению неполадок** Пользователи могут оценить анклава свидетельство для политики аттестации SGX, прежде чем настраивать их.

Отправьте запрос в API аттестации, указав текст политики в параметре "Драфтполицифораттестатион". API Аттестсгксенклаве будет использовать этот документ политики во время вызова аттестации, и это можно использовать для проверки политик аттестации перед их использованием. Токен аттестации, созданный при наличии этого поля, будет незащищенным.

См. [примеры политики аттестации](./policy-examples.md)

### <a name="22-attestation-failure-due-to-invalid-input"></a>2.2. Сбой аттестации из-за недопустимых входных данных

**Код ошибки** инвалидпараметер

**Примеры сценариев** Сбой аттестации SGX из-за недопустимых входных данных. Ниже приведены некоторые примеры сообщений об ошибках.
- Указанная кавычка недопустима из-за ошибки в сопутствующих предложениях 
- Указанная цитата недопустима, так как устройство, на котором была создана квота, не соответствует требованиям к базовому плану Azure.
- Указана недопустимая кавычка, так как Ткбинфо или КЕИД, предоставленные службой кэша ПРЕДПОЛАГАЮ, недопустимы

**Действия по устранению неполадок**

Аттестация Microsoft Azure поддерживает аттестацию квот SGX, созданных пакетом SDK Intel и Open анклава SDK.

Ознакомьтесь с [примерами кода](/samples/browse/?expanded=azure&terms=attestation) для выполнения аттестации с помощью Open анклава SDK/пакета SDK Intel

### <a name="23-invalid-certificate-chain-error-while-uploading-policypolicy-signer"></a>2.3. Недопустимая ошибка цепочки сертификатов при отправке подписи политики или политики

**Код ошибки** инвалидпараметер

**Примеры сценариев** Настройка подписанной политики или добавление или удаление подписавшего политики, подписанную недействительной цепочкой сертификатов (например, если в расширении базовых ограничений корневого сертификата не задан тип субъекта = CA)

```
Native operation failed with 65529: C:\source\src\AttestationServices\Instance\SgxPal\sgxcert.cpp(1074)\(null)!00007FFA285CDAED: (caller: 00007FFA285C36E8) Exception(0) 83FFFFF9 The requested item is not found    Msg:[Unable to find issuer certificate CN=attestationsigningcert]
C:\source\src\AttestationServices\Instance\Enclave\api.cpp(618)\(null)!00007FFA286DCBF8: (caller: 00007FFA285860D3) LogHr(0) 83FFFFF9 The requested item is not found    Msg:[Unhandled Enclave Exception: "The requested item is not found"]
At line:1 char:1
+ Set-AzAttestationPolicy -Name "testpolicy1" -ResourceGroupName "BugBa ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy

```

**Действия по устранению неполадок** Корневой сертификат необходимо пометить как выдаваемый центром сертификации (базовыми ограничениями X. 509), иначе он не будет считаться действительным сертификатом. 

Убедитесь, что расширение основных ограничений корневого сертификата указывает на то, что субъект имеет тип = CA.

В противном случае цепочка сертификатов считается недопустимой.

См. примеры [политик](./policy-examples.md) и [подписания политики](./policy-signer-examples.md) 

### <a name="24-adddelete-policy-signer-failure"></a>2.4. Ошибка при добавлении или удалении политики

**Код ошибки** InvalidOperation

**Примеры сценариев**

Когда пользователь отправляет ЖВС без утверждения "МАА-Полицицертификате"

```
Add-AzAttestationPolicySigner : Operation returned HTTP Status Code 400
Code: InvalidOperation
Message: Native operation failed with 74: ..\Enclave\enclave.cpp(2213)\(null)!: (caller: ) Exception(0) 83FF004A Bad
message    Msg:[Could not find "maa-policyCertificate" claim in policy token]
..\Enclave\api.cpp(496)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message    Msg:[Unhandled Enclave Exception: "Bad
message"]
At line:1 char:1
+ Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGro ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Add-AzAttestationPolicySigner], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.AddAzureAttestationPolicySigner

```

Когда пользователь не отправляет сертификат в формате ЖВС

```
Add-AzAttestationPolicySigner : Operation returned HTTP Status Code 400
Code: InvalidOperation
Message: Native operation failed with 74: ..\JsonWebToken\jsonwebtoken.cpp(375)\(null)!: (caller: ) LogHr(0) 83FF004A
Bad message    Msg:[RETURN_IF_TRUE('(firstPeriod == std::string::npos)') failed with 0x4a: Malformed JWT: Could not
find first period in the token.]
..\Enclave\enclave.cpp(2106)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message
Msg:[THROW_IF_ERROR('DecomposeJsonWebSignature(&policyJws, encodedJoseHeader, encodedJwsBody, jwsSignature)') failed
with 0x4a: 'Bad message']
..\Enclave\enclave.cpp(2106)\(null)!: (caller: ) Exception(0) 83FF004A Bad message
..\Enclave\api.cpp(496)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message    Msg:[Unhandled Enclave Exception: "Bad
message"]
At line:1 char:1
+ Add-AzAttestationPolicySigner -Name $attestationProvider -ResourceGro ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Add-AzAttestationPolicySigner], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.AddAzureAttestationPolicySigner
```

**Действия по устранению неполадок** Чтобы добавить или удалить новый сертификат подписи политики, используйте RFC7519 JSON Web Token (JWT) с утверждением "x-MS-Полицицертификате". Значением утверждения является веб-ключ JSON RFC7517, который содержит добавляемый сертификат. JWT должен быть подписан закрытым ключом любого из допустимых сертификатов подписавшего политику, связанных с поставщиком. См. [примеры подписывания политики](./policy-signer-examples.md).

### <a name="25-attestation-policy-configuration-failure"></a>2.5. Сбой конфигурации политики аттестации

**Код ошибки** полиципарсинжеррор

**Примеры сценариев** Политика указана с неверным синтаксисом (например, отсутствует точка с запятой)/Валид JWT)

```
Native operation failed with 65526: ..\NativePolicyWrapper\NativePolicyEngine.cpp(31)\(null)!: (caller: ) Exception(0) 83FFFFF6 Invalid policy was specified    Msg:[Policy Parser Exception Thrown: Offending
symbol: '['
Line: '2', Column: '1'
Failure message: 'mismatched input '[' expecting ';''
Failing rule: 'policy > versionInfo']
..\Enclave\api.cpp(618)\(null)!: (caller: ) LogHr(0) 83FFFFF6 Invalid policy was specified    Msg:[Unhandled Enclave Exception: "Invalid policy was specified"]
At line:1 char:1
+ set-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy
```

**Код ошибки** InvalidOperation

**Примеры сценариев** Указано недопустимое содержимое (например, "Передача политики/неподписанной политики при необходимости подписи политики")

```
Native operation failed with 74: ..\Shared\base64url.h(226)\(null)!: (caller: ) Exception(0) 83FF004A Bad message    Msg:[Unknown base64 character: 41 (')')]
..\Enclave\api.cpp(618)\(null)!: (caller: ) LogHr(0) 83FF004A Bad message    Msg:[Unhandled Enclave Exception: "Bad message"]
At line:1 char:1
+ set-AzAttestationPolicy -Name $attestationProvider -ResourceGroupName ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Set-AzAttestationPolicy], RestException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Attestation.SetAzureAttestationPolicy
```

**Действия по устранению неполадок** Убедитесь, что политика в текстовом формате имеет кодировку UTF-8.

Если требуется подписывание политики, политика аттестации должна быть настроена только в формате RFC7519 JSON Web Token (JWT). Если подписывание политики не требуется, политику можно настроить в формате текста или JWT.

Чтобы настроить политику в формате JWT, используйте JWT с утверждением с именем "Аттестатионполици". Значение утверждения — Base64URL кодированная версия текста политики. Если поставщик аттестации настроен с сертификатами подписавшего политики, JWT должен быть подписан закрытым ключом любого из допустимых сертификатов подписавшего политику, связанных с поставщиком. 

Чтобы настроить политику в текстовом формате, укажите текст политики напрямую.

В PowerShell укажите Полициформат как JWT, чтобы настроить политику в формате JWT. Формат политики по умолчанию — Text.

См. [примеры политики](./policy-examples.md) аттестации и [Создание политики аттестации](./author-sign-policy.md) . 

## <a name="3-azattestation-installation-issues-in-powershell"></a>3. az. Аттестация — проблемы с установкой в PowerShell

Не удалось установить модули az или AZ. аттестации в PowerShell

### <a name="error"></a>Ошибка

ВНИМАНИЕ! не удалось разрешить источник пакета " https://www.powershellgallery.com/api/v2 паккажеманажемент\инсталл-паккаже: не найдено совпадений для указанных условий поиска и имени модуля

### <a name="troubleshooting-steps"></a>Действия по устранению неполадок

Коллекция PowerShell имеет устаревшие протоколы TLS версии 1,0 и 1,1. 

Используйте TLS версии 1.2 или более поздней. 

Чтобы продолжить взаимодействие с коллекцией PowerShell Gallery, выполните следующие команды перед командами Install-Module.

**[NET. ServicePointManager]:: экземпляр SecurityProtocol = [NET. Секуритипротоколтипе]:: Tls12**

## <a name="4-policy-accessconfiguration-issues-in-powershell"></a>4. проблемы с доступом к политике и конфигурацией в PowerShell

Пользователь назначен с соответствующими ролями. Но при управлении политиками аттестации с помощью PowerShell возникают проблемы с авторизацией.

### <a name="error"></a>Ошибка
Клиент с идентификатором объекта идентификатор объекта не &lt; &gt;  имеет авторизации для выполнения действия Microsoft. Authorization/roleassignments/overwrite Scope "подписками/ &lt; subscriptionId &gt; resourcegroups/secure_enclave_poc/провидерс/Микрософт.аусоризатион/ролеассигнментс/ &lt; Role assignmentId", &gt; либо Недопустимая область. Если доступ был недавно предоставлен, обновите учетные данные.

### <a name="troubleshooting-steps"></a>Действия по устранению неполадок

Ниже приведена минимальная версия AZ Modules, необходимая для поддержки операций аттестации. 

 **Az 4.5.0;** 
 
 **Az.Accounts 1.9.2;**
 
 **Az.Attestation 0.1.8.** 

Выполните приведенную ниже команду, чтобы проверить версии всех установленных модулей Az. 

```powershell
Get-InstalledModule 
```

Если версии не совпадают с минимальным требованием, выполните Update-Module команды.

Например,-Update-Module-Name AZ. Аттестация

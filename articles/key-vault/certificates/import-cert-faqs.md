---
title: Часто задаваемые вопросы о Azure Key Vault импорте сертификатов
description: Получите ответы на часто задаваемые вопросы об импорте сертификатов Azure Key Vault.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.date: 07/20/2020
ms.author: sebansal
ms.openlocfilehash: 78ec0af0acbb74436af16abba75d64de061d5268
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102202171"
---
# <a name="importing-azure-key-vault-certificates-faq"></a>Импорт сертификатов Azure Key Vault вопросы и ответы

В этой статье содержатся ответы на часто задаваемые вопросы об импорте сертификатов Azure Key Vault.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="how-can-i-import-a-certificate-in-azure-key-vault"></a>Как импортировать сертификат в Azure Key Vault?

Для операции импорта сертификата Azure Key Vault принимает два формата файлов сертификатов: PEM и PFX. Хотя есть PEM-файлы, содержащие только общую часть, Key Vault требует и принимает только PEM-файл или PFX с закрытым ключом. Дополнительные сведения см. [в разделе Импорт сертификата в Key Vault](./tutorial-import-certificate.md#import-a-certificate-to-key-vault).

### <a name="after-i-import-a-password-protected-certificate-to-key-vault-and-then-download-it-why-cant-i-see-the-password-thats-associated-with-it"></a>Почему после импорта защищенного паролем сертификата для Key Vault, а затем его загрузки, почему не отображается связанный с ним пароль?
    
После импорта и защиты сертификата в Key Vault связанный с ним пароль не сохраняется. Пароль требуется только один раз во время операции импорта. Это предназначено для разработки, но вы всегда можете получить сертификат в качестве секрета и преобразовать его из Base64 в PFX, добавив пароль с помощью [Azure PowerShell](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx).

### <a name="how-can-i-resolve-a-bad-parameter-error-what-are-the-supported-certificate-formats-for-importing-to-key-vault"></a>Как можно устранить ошибку "неверный параметр"? Какие форматы сертификатов поддерживаются для импорта в Key Vault?

При импорте сертификата необходимо убедиться, что ключ включен в файл. Если закрытый ключ хранится отдельно в другом формате, необходимо объединить ключ с сертификатом. Некоторые центры сертификации (ЦС) предоставляют сертификаты в других форматах. Поэтому перед импортом сертификата убедитесь, что он находится в формате PEM или PFX и что в ключе используется шифрование Ривест – Шамир – Адельман (RSA) или алгоритм шифрования на основе эллиптических кривых (ECC). 

Дополнительные сведения см. в разделе [требования к сертификату](./certificate-scenarios.md#formats-of-import-we-support) и [требования к ключу сертификата](../keys/about-keys.md).

###  <a name="can-i-import-a-certificate-by-using-an-arm-template"></a>Можно ли импортировать сертификат с помощью шаблона ARM?

Нет, операции с сертификатами невозможно выполнить с помощью шаблона Azure Resource Manager (ARM). Рекомендуемым решением будет использование методов импорта сертификатов в API Azure, Azure CLI или PowerShell. Если у вас уже есть сертификат, его можно импортировать в качестве секрета.

### <a name="when-i-import-a-certificate-via-the-azure-portal-i-get-a-something-went-wrong-error-how-can-i-investigate-further"></a>При импорте сертификата с помощью портал Azure появляется сообщение об ошибке "что-то пошло не так". Как можно исследовать дальнейшие исследования?
    
Чтобы просмотреть более описательную ошибку, импортируйте файл сертификата с помощью [Azure CLI](/cli/azure/keyvault/certificate#az-keyvault-certificate-import) или [PowerShell](/powershell/module/azurerm.keyvault/import-azurekeyvaultcertificate?view=azurermps-6.13.0).

### <a name="how-can-i-resolve-error-type-access-denied-or-user-is-unauthorized-to-import-certificate"></a>Как разрешить "тип ошибки: отказано в доступе или пользователь не авторизован для импорта сертификата"?
    
Для операции импорта необходимо предоставить пользователю разрешения на импорт сертификата в политиках доступа. Для этого перейдите в хранилище ключей, выберите **политики доступа**  >  **Добавить политику доступа**  >  **выбрать субъект разрешений сертификата**  >  , найдите пользователя, а затем добавьте адрес электронной почты пользователя. 

Дополнительные сведения о политиках доступа, связанных с сертификатами, см. в статье [о сертификатах Azure Key Vault](./about-certificates.md#certificate-access-control).


### <a name="how-can-i-resolve-error-type-conflict-when-creating-a-certificate"></a>Как можно разрешить «тип ошибки: конфликт при создании сертификата»?
    
Имя каждого сертификата должно быть уникальным. Сертификат с таким именем может находиться в состоянии обратимого удаления. Кроме того, в соответствии с [композицией сертификата](./about-certificates.md#composition-of-a-certificate)при создании нового сертификата он создает секрет с тем же именем, поэтому при наличии другого ключа или секрета в хранилище ключей с тем же именем, которое вы пытаетесь указать для вашего сертификата, создание сертификата завершится сбоем, и вам нужно будет либо удалить этот ключ или секрет, либо использовать другое имя для сертификата. 

Дополнительные сведения см. в разделе [Получение удаленной операции с сертификатом](/rest/api/keyvault/getdeletedcertificate/getdeletedcertificate).

### <a name="why-am-i-getting-error-type-char-length-is-too-long"></a>Почему я получаю сообщение "тип ошибки: длина знака слишком велика"?
Эта ошибка может быть вызвана одной из двух причин:    
* Длина имени субъекта сертификата ограничена 200 символами.
* Длина пароля сертификата ограничена 200 символами.


### <a name="error-the-specified-pem-x509-certificate-content-is-in-an-unexpected-format-please-check-if-certificate-is-in-valid-pem-format"></a>Ошибка "указанное содержимое сертификата PEM X. 509 имеет непредвиденный формат. Убедитесь, что сертификат имеет допустимый формат PEM. "
Убедитесь, что содержимое в PEM-файле использует разделители строк в стиле UNIX. `(\n)`

### <a name="can-i-import-an-expired-certificate-to-azure-key-vault"></a>Можно ли импортировать сертификат с истекшим сроком действия для Azure Key Vault?
    
Нет, сертификаты PFX с истекшим сроком действия не могут быть импортированы в Key Vault.

### <a name="how-can-i-convert-my-certificate-to-the-proper-format"></a>Как можно преобразовать сертификат в правильный формат?

Вы можете попросить ЦС предоставить сертификат в нужном формате. Кроме того, существуют сторонние средства, которые могут помочь преобразовать сертификат в правильный формат.

### <a name="can-i-import-certificates-from-non-partner-cas"></a>Можно ли импортировать сертификаты из ЦС, не являющегося партнером?
Да, можно импортировать сертификаты из любого центра сертификации, но ваше хранилище ключей не сможет обновлять их автоматически. Вы можете настроить напоминания об истечении срока действия сертификата.

### <a name="if-i-import-a-certificate-from-a-partner-ca-will-the-autorenewal-feature-still-work"></a>Если импортировать сертификат из ЦС партнера, будет ли продолжать работать функция автоматического продления?
Да. После отправки сертификата не забудьте указать Автоповорот в политике выдачи сертификата. Параметры вступят в силу, пока не будет выпущен следующий цикл или версия сертификата.

### <a name="why-cant-i-see-the-app-service-certificate-that-i-imported-to-key-vault"></a>Почему не отображается сертификат службы приложений, импортированный в Key Vault? 
Если сертификат успешно импортирован, вы сможете подтвердить его, перейдя на панель **секреты** .


## <a name="next-steps"></a>Дальнейшие действия

- [Сертификаты Azure Key Vault](./about-certificates.md)

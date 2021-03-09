---
title: Сертификаты Брандмауэра Azure уровня "Премиум" в предварительной версии
description: Чтобы правильно настроить проверку TLS в предварительной версии Azure Firewall Premium, необходимо настроить и установить промежуточные сертификаты ЦС.
author: vhorne
ms.service: firewall
services: firewall
ms.topic: conceptual
ms.date: 03/09/2021
ms.author: victorh
ms.openlocfilehash: 621bf6138e4336c63ca137a6a8c54f77a4a99d61
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102520291"
---
# <a name="azure-firewall-premium-preview-certificates"></a>Сертификаты Брандмауэра Azure уровня "Премиум" в предварительной версии 

> [!IMPORTANT]
> Брандмауэр Azure уровня "Премиум" сейчас находится в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

 Чтобы правильно настроить проверку TLS брандмауэра Azure Premium Preview, необходимо предоставить действительный сертификат промежуточного ЦС и поместить его в хранилище ключей Azure.

## <a name="certificates-used-by-azure-firewall-premium-preview"></a>Сертификаты, используемые в предварительной версии Azure Firewall Premium

Существует три типа сертификатов, используемых в типичном развертывании:

- **Промежуточный сертификат ЦС (сертификат ЦС)**

   Центр сертификации (CA) — это организация, которая является доверенной для подписания цифровых сертификатов. Центр сертификации проверяет удостоверение и Закон подлинности компании или отдельного запроса сертификата. Если проверка прошла успешно, ЦС выдает подписанный сертификат. Когда сервер предоставляет сертификат клиенту (например, веб-браузеру) во время подтверждения SSL/TLS, клиент пытается проверить подпись по списку *известных хороших* подписывающих. В веб-браузерах обычно используются списки центров сертификации, которые неявно доверяются на обнаружение узлов. Если в списке нет центра, как и для некоторых сайтов, которые подписывают свои собственные сертификаты, браузер предупреждает пользователя о том, что сертификат не подписан доверенным центром сертификации, и запрашивает у пользователя возможность продолжить обмен данными с непроверенным сайтом.

- **Сертификат сервера (сертификат веб-сайта)**

   Сертификат, связанный с конкретным доменным именем. Если у веб-сайта есть действительный сертификат, то это означает, что центр сертификации выполнил действия по проверке факта принадлежности веб-адреса к этой Организации. При вводе URL-адреса или ссылки на защищенный веб-сайт браузер проверяет сертификат на наличие следующих характеристик:
   - Адрес веб-сайта соответствует адресу сертификата.
   - Сертификат подписывается центром сертификации, который браузер распознает как *доверенный* центр.
   
   Иногда пользователи могут подключаться к серверу с недоверенным сертификатом. Брандмауэр Azure удалит подключение, как если бы сервер прервал подключение.

- **Сертификат корневого ЦС (корневой сертификат)**

   Центр сертификации может выдавать несколько сертификатов в виде древовидной структуры. Корневой сертификат — это самый верхний сертификат дерева.

Предварительная версия Azure Firewall Premium позволяет перехватывать исходящий трафик HTTP/S и автоматически создавать сертификат сервера для `www.website.com` . Этот сертификат создается с помощью предоставленного промежуточного сертификата ЦС. Чтобы эта процедура работала, браузер и клиентские приложения для конечных пользователей должны доверять сертификату корневого ЦС организации или сертификату промежуточного ЦС. 

:::image type="content" source="media/premium-certificates/certificate-process.png" alt-text="Процесс сертификата":::

## <a name="intermediate-ca-certificate-requirements"></a>Требования к сертификатам промежуточного ЦС

Убедитесь, что сертификат ЦС соответствует следующим требованиям.

- При развертывании в качестве Key Vault секрета необходимо использовать PFX-файл без пароля (PKCS12) с сертификатом и закрытым ключом.

- Он должен быть одним сертификатом и не должен включать всю цепочку сертификатов.  

- Она должна быть действительна в течение одного года вперед.  

- Это должен быть закрытый ключ RSA с минимальным размером в 4096 байт.  

- Оно должно иметь `KeyUsage` расширение, помеченное как критическое, с `KeyCertSign` флагом (RFC 5280; использование ключа 4.2.1.3).

- Оно должно иметь `BasicContraints` расширение, помеченное как критическое (RFC 5280; базовые ограничения 4.2.1.9).  

- `CA`Флаг должен иметь значение true.

- Длина пути должна быть больше или равна единице.

## <a name="azure-key-vault"></a>Azure Key Vault

Управляемое платформой хранилище секретов [Azure Key Vault](../key-vault/general/overview.md) можно применять для защиты секретов, ключей и сертификатов TLS/SSL. Брандмауэр Azure Premium поддерживает интеграцию с Key Vault для сертификатов сервера, подключенных к политике брандмауэра.
 
Чтобы настроить хранилище ключей, сделайте следующее:

- Необходимо импортировать существующий сертификат со своей парой ключей в хранилище ключей. 
- Кроме того, можно использовать секрет хранилища ключей, который хранится в виде PFX-файла в формате "без пароля", с основанием 64.  PFX-файл — это цифровой сертификат, содержащий закрытый ключ и открытый ключ.
- Рекомендуется использовать импорт сертификатов ЦС, так как он позволяет настроить оповещение на основе даты истечения срока действия сертификата.
- После импорта сертификата или секрета необходимо определить политики доступа в хранилище ключей, чтобы предоставить удостоверению доступ для получения доступа к сертификату или секрету.
- Указанный сертификат ЦС должен быть доверенным для рабочей нагрузки Azure. Убедитесь, что они развернуты правильно.

Вы можете создать или повторно использовать существующее управляемое удостоверение, назначенное пользователем, которое используется брандмауэром Azure для получения сертификатов от Key Vault от вашего имени. Дополнительные сведения см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md) 

## <a name="configure-a-certificate-in-your-policy"></a>Настройка сертификата в политике

Чтобы настроить сертификат ЦС в политике Premium брандмауэра, выберите свою политику и щелкните **Проверка TLS (Предварительная версия)**. Выберите **включено** на странице **Проверка TLS** . Затем выберите сертификат ЦС в Azure Key Vault, как показано на следующем рисунке.

:::image type="content" source="media/premium-certificates/tls-inspection.png" alt-text="Обзорная схема брандмауэра Azure Premium":::
 
> [!IMPORTANT]
> Чтобы просмотреть и настроить сертификат из портал Azure, необходимо добавить учетную запись пользователя Azure в политику доступа к Key Vault. Предоставьте учетной записи пользователя и **получите** **список** в разделе " **секретные разрешения**".
   :::image type="content" source="media/premium-certificates/secret-permissions.png" alt-text="Политика доступа Azure Key Vault":::


## <a name="create-your-own-self-signed-ca-certificate"></a>Создание собственного самозаверяющего сертификата ЦС

Чтобы помочь вам протестировать и проверить проверку подлинности TLS, можно использовать следующие сценарии для создания собственного самозаверяющего корневого ЦС и промежуточного ЦС.

> [!IMPORTANT]
> Для рабочей среды следует использовать корпоративную PKI для создания промежуточного сертификата ЦС. Корпоративная PKI использует существующую инфраструктуру и выполняет распределение корневого ЦС на всех конечных компьютерах. Дополнительные сведения см. в статье [развертывание и настройка сертификатов ЦС предприятия для предварительной версии брандмауэра Azure](premium-deploy-certificates-enterprise-ca.md).

Существует две версии этого сценария:
- Скрипт bash `cert.sh` 
- сценарий PowerShell `cert.ps1` 

 Кроме того, оба скрипта используют `openssl.cnf` файл конфигурации. Чтобы использовать скрипты, скопируйте содержимое `openssl.cnf` , и `cert.sh` или `cert.ps1` на локальный компьютер.

Скрипты создают следующие файлы:
- rootCA. CRT/rootCA. Открытый сертификат корневого ЦС и закрытый ключ.
- Интерка. CRT/Интерка. ключ — открытый сертификат ЦС и закрытый ключ.
- Интерка. pfx — промежуточный ЦС PKCS12 пакет, который будет использоваться брандмауэром.

> [!IMPORTANT]
> rootCA. Key должен храниться в безопасном автономном расположении. Скрипты создают сертификат со сроком действия 1024 дней.

После создания сертификатов разверните их в следующих расположениях:
- rootCA. CRT — развертывание на компьютерах конечных точек (только открытый сертификат).
- Интерка. pfx — импортируйте сертификат в Key Vault и назначьте его политике брандмауэра.

### <a name="opensslcnf"></a>**OpenSSL. cnf**
```
[ req ]
default_bits        = 4096
distinguished_name  = req_distinguished_name
string_mask         = utf8only
default_md          = sha512

[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
stateOrProvinceName             = State or Province Name
localityName                    = Locality Name
0.organizationName              = Organization Name
organizationalUnitName          = Organizational Unit Name
commonName                      = Common Name
emailAddress                    = Email Address

[ rootCA_ext ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ interCA_ext ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:1
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

[ server_ext ]
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:false
keyUsage = critical, digitalSignature
extendedKeyUsage = serverAuth
```

###  <a name="bash-script---certsh"></a>Bash-скрипт — cert.sh 
```bash
#!/bin/bash

# Create root CA
openssl req -x509 -new -nodes -newkey rsa:4096 -keyout rootCA.key -sha256 -days 1024 -out rootCA.crt -subj "/C=US/ST=US/O=Self Signed/CN=Self Signed Root CA" -config openssl.cnf -extensions rootCA_ext

# Create intermediate CA request
openssl req -new -nodes -newkey rsa:4096 -keyout interCA.key -sha256 -out interCA.csr -subj "/C=US/ST=US/O=Self Signed/CN=Self Signed Intermediate CA"

# Sign on the intermediate CA
openssl x509 -req -in interCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out interCA.crt -days 1024 -sha256 -extfile openssl.cnf -extensions interCA_ext

# Export the intermediate CA into PFX
openssl pkcs12 -export -out interCA.pfx -inkey interCA.key -in interCA.crt -password "pass:"

echo ""
echo "================"
echo "Successfully generated root and intermediate CA certificates"
echo "   - rootCA.crt/rootCA.key - Root CA public certificate and private key"
echo "   - interCA.crt/interCA.key - Intermediate CA public certificate and private key"
echo "   - interCA.pfx - Intermediate CA pkcs12 package which could be uploaded to Key Vault"
echo "================"
```

### <a name="powershell---certps1"></a>PowerShell — cert.ps1
```powershell
# Create root CA
openssl req -x509 -new -nodes -newkey rsa:4096 -keyout rootCA.key -sha256 -days 3650 -out rootCA.crt -subj '/C=US/ST=US/O=Self Signed/CN=Self Signed Root CA' -config openssl.cnf -extensions rootCA_ext

# Create intermediate CA request
openssl req -new -nodes -newkey rsa:4096 -keyout interCA.key -sha256 -out interCA.csr -subj '/C=US/ST=US/O=Self Signed/CN=Self Signed Intermediate CA'

# Sign on the intermediate CA
openssl x509 -req -in interCA.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out interCA.crt -days 3650 -sha256 -extfile openssl.cnf -extensions interCA_ext

# Export the intermediate CA into PFX
openssl pkcs12 -export -out interCA.pfx -inkey interCA.key -in interCA.crt -password 'pass:'

Write-Host ""
Write-Host "================"
Write-Host "Successfully generated root and intermediate CA certificates"
Write-Host "   - rootCA.crt/rootCA.key - Root CA public certificate and private key"
Write-Host "   - interCA.crt/interCA.key - Intermediate CA public certificate and private key"
Write-Host "   - interCA.pfx - Intermediate CA pkcs12 package which could be uploaded to Key Vault"
Write-Host "================"

```

## <a name="troubleshooting"></a>Диагностика

Если сертификат ЦС действителен, но вы не можете получить доступ к полным доменным именам или URL-адресам при проверке TLS, проверьте следующие пункты:

- Убедитесь, что сертификат веб-сервера действителен.  

- Убедитесь, что сертификат корневого ЦС установлен в клиентской операционной системе.  

- Убедитесь, что браузер или HTTPS-клиент содержит допустимый корневой сертификат. Firefox и другие браузеры могут иметь особые политики сертификации.  

- Убедитесь, что тип назначения URL в правиле приложения охватывает правильный путь и все другие гиперссылки, внедренные на HTML-страницу назначения. Можно использовать подстановочные знаки для простого покрытия всего требуемого URL-пути.  


## <a name="next-steps"></a>Дальнейшие действия

- [Дополнительные сведения о расширенных возможностях брандмауэра Azure](premium-features.md)

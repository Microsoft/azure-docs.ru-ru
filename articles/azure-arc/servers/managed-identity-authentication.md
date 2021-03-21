---
title: Аутентификация в ресурсах Azure с помощью серверов с поддержкой Arc
description: В этой статье описывается поддержка службы метаданных экземпляров Azure для серверов с поддержкой Arc, а также способы проверки подлинности в ресурсах Azure и локальных с использованием секрета.
ms.topic: conceptual
ms.date: 12/09/2020
ms.openlocfilehash: 49b70928ae972da8e0a0d14d711e4b6f246cca6a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96939195"
---
# <a name="authenticate-against-azure-resources-with-arc-enabled-servers"></a>Аутентификация в ресурсах Azure с помощью серверов с поддержкой Arc

Приложения или процессы, работающие непосредственно на серверах с поддержкой Arc Azure, могут использовать управляемые удостоверения для доступа к другим ресурсам Azure, поддерживающим проверку подлинности на основе Azure Active Directory. Приложение может получить [маркер доступа](../../active-directory/develop/developer-glossary.md#access-token) , представляющий удостоверение, которое назначается системой для серверов с поддержкой Arc, и использовать его в качестве токена носителя для проверки подлинности в другой службе.

Подробное описание управляемых удостоверений, а также различие между назначенными системой и назначенными пользователем удостоверениями см. в документации по [управляемым удостоверениям](../../active-directory/managed-identities-azure-resources/overview.md) .

В этой статье показано, как сервер может использовать управляемое системой удостоверение для доступа к Azure [Key Vault](../../key-vault/general/overview.md). При использовании в качестве начальной загрузки хранилище Key Vault позволяет клиентскому приложению использовать секрет для обращения к ресурсам, которые не защищены с помощью Azure Active Directory (AD). Например, сертификаты TLS/SSL, используемые веб-серверами IIS, могут храниться в Azure Key Vault и безопасно развертывать сертификаты на серверах Windows или Linux за пределами Azure.

## <a name="security-overview"></a>Общие сведения о безопасности

При подключении сервера к серверам с поддержкой ARC в Azure для настройки с помощью управляемого удостоверения выполняются несколько действий, аналогично тому, что выполняется для виртуальной машины Azure.

- Azure Resource Manager получает запрос на включение управляемого удостоверения, назначенного системой на сервере с поддержкой ARC.

- Azure Resource Manager создает субъект-службу в Azure AD для удостоверения сервера. В клиенте Azure AD, который является доверенным для этой подписки, создается субъект-служба.

- Azure Resource Manager настраивает удостоверение на сервере, обновляя конечную точку удостоверения службы метаданных экземпляра Azure (IMDS) для [Windows](../../virtual-machines/windows/instance-metadata-service.md) или [Linux](../../virtual-machines/linux/instance-metadata-service.md) с помощью идентификатора клиента и сертификата субъекта-службы. Конечная точка является конечной точкой RESTFUL, доступной только с сервера, используя известный, не поддерживающий маршрутизацию IP-адрес. Эта служба предоставляет подмножество метаданных для сервера с поддержкой ARC для помощи в управлении и настройке.

Среда сервера с управляемым удостоверением будет настроена со следующими переменными на сервере с поддержкой Arc Windows:

- **IMDS_ENDPOINT**: IP-адрес КОНЕЧНОЙ точки IMDS `http://localhost:40342` для серверов с поддержкой ARC.

- **IDENTITY_ENDPOINT**: конечная точка localhost, соответствующая управляемому удостоверению службы `http://localhost:40342/metadata/identity/oauth2/token` .

Код, выполняемый на сервере, может запросить маркер из конечной точки службы метаданных экземпляра Azure, доступного только с сервера.

Переменная среды системы **IDENTITY_ENDPOINT** используется для обнаружения конечной точки удостоверения приложениями. Приложения должны попытаться получить значения **IDENTITY_ENDPOINT** и **IMDS_ENDPOINT** и использовать их. Приложения с любым уровнем доступа могут выполнять запросы к конечным точкам. Ответы на метаданные обрабатываются как обычные и передаются в любой процесс на компьютере. Однако при выполнении запроса, который предоставит маркер, клиент должен предоставить секрет для подтверждения того, что он может получить доступ только к данным, доступным для пользователей с высоким уровнем привилегий.

## <a name="prerequisites"></a>Предварительные требования

- Основные сведения об управляемых удостоверениях.
- Сервер, подключенный и зарегистрированный на серверах с поддержкой ARC.
- Вы являетесь членом [группы владельцев](../../role-based-access-control/built-in-roles.md#owner)* * в подписке или группе ресурсов для выполнения необходимых действий по созданию ресурсов и управлению ролями.
- Azure Key Vault для хранения и получения учетных данных. и назначьте удостоверению Arc Azure доступ к KeyVault.

    - Если вы не создали Key Vault, см. раздел [Create Key Vault](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad.md#create-a-key-vault-).
    - Сведения о настройке доступа с помощью управляемого удостоверения, используемого сервером, см. в разделе [предоставление доступа для Linux](../../active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-nonaad.md#grant-access) или [предоставление доступа для Windows](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad.md#grant-access). В шаге 5 будет введено имя сервера с поддержкой ARC. Чтобы завершить эту работу с помощью PowerShell, см. раздел [Назначение политики доступа с помощью PowerShell](../../key-vault/general/assign-access-policy-powershell.md).

## <a name="acquiring-an-access-token-using-rest-api"></a>Получение маркера доступа с помощью REST API

Метод получения и использования управляемого системой удостоверения для аутентификации в ресурсах Azure аналогичен тому, как он выполняется с помощью виртуальной машины Azure.

Для дуги с поддержкой Windows Server с помощью PowerShell вы вызываете веб-запрос для получения маркера с локального узла в определенном порте. Укажите запрос, используя IP-адрес или переменную среды **IDENTITY_ENDPOINT**.

```powershell
$apiVersion = "2020-06-01"
$resource = "https://management.azure.com/"
$endpoint = "{0}?resource={1}&api-version={2}" -f $env:IDENTITY_ENDPOINT,$resource,$apiVersion
$secretFile = ""
try
{
    Invoke-WebRequest -Method GET -Uri $endpoint -Headers @{Metadata='True'} -UseBasicParsing
}
catch
{
    $wwwAuthHeader = $_.Exception.Response.Headers["WWW-Authenticate"]
    if ($wwwAuthHeader -match "Basic realm=.+")
    {
        $secretFile = ($wwwAuthHeader -split "Basic realm=")[1]
    }
}
Write-Host "Secret file path: " $secretFile`n
$secret = cat -Raw $secretFile
$response = Invoke-WebRequest -Method GET -Uri $endpoint -Headers @{Metadata='True'; Authorization="Basic $secret"} -UseBasicParsing
if ($response)
{
    $token = (ConvertFrom-Json -InputObject $response.Content).access_token
    Write-Host "Access token: " $token
}
```

Следующий ответ — это пример, который возвращает:

:::image type="content" source="media/managed-identity-authentication/powershell-token-output-example.png" alt-text="Успешное получение маркера доступа с помощью PowerShell.":::

Для сервера Linux с поддержкой ARC с помощью bash вы вызываете веб-запрос для получения токена с локального узла в определенном порте. Укажите следующий запрос, используя IP-адрес или переменную среды **IDENTITY_ENDPOINT**. Для выполнения этого шага необходим клиент SSH.

```bash
ChallengeTokenPath=$(curl -s -D - -H Metadata:true "http://127.0.0.1:40342/metadata/identity/oauth2/token?api-version=2019-11-01&resource=https%3A%2F%2Fmanagement.azure.com" | grep Www-Authenticate | cut -d "=" -f 2 | tr -d "[:cntrl:]")
ChallengeToken=$(cat $ChallengeTokenPath)
if [ $? -ne 0 ]; then
    echo "Could not retrieve challenge token, double check that this command is run with root privileges."
else
    curl -s -H Metadata:true -H "Authorization: Basic $ChallengeToken" "http://127.0.0.1:40342/metadata/identity/oauth2/token?api-version=2019-11-01&resource=https%3A%2F%2Fmanagement.azure.com"
fi
```

Следующий ответ — это пример, который возвращает:

:::image type="content" source="media/managed-identity-authentication/bash-token-output-example.png" alt-text="Успешное получение маркера доступа с помощью bash.":::

Ответ включает маркер доступа, необходимый для доступа к любому ресурсу в Azure. Чтобы завершить настройку проверки подлинности в Azure Key Vault, см. статью [доступ к Key Vault с помощью Windows](../../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad.md#access-data) или [Access Key Vault с Linux](../../active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-nonaad.md#access-data).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о Azure Key Vault см. в разделе [Общие сведения о Key Vault](../../key-vault/general/overview.md).

- Узнайте, как назначить управляемому удостоверению доступ к ресурсу [с помощью PowerShell](../../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md) или с помощью [Azure CLI](../../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md).
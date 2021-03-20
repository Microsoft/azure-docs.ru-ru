---
title: Конечные точки безопасности в службе подготовки устройств Интернета вещей | Документация Майкрософт
description: Основные понятия. Управление доступом к службе подготовки устройств Интернета вещей (DPS) для серверных приложений. Эта статья включает сведения о маркерах безопасности.
author: wesmc7777
manager: philmea
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 04/09/2019
ms.author: wesmc
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 024dbf6518748a4048873de4eb54a53f9d9a6362
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94954330"
---
# <a name="control-access-to-azure-iot-hub-device-provisioning-service"></a>Управление доступом к службе подготовки устройств Центра Интернета вещей Azure

В этой статье описаны возможности защиты службы "Подготовка устройств Интернета вещей". Служба подготовки использует *разрешения* для предоставления доступа к каждой из конечных точек. Разрешения ограничивают доступ к экземпляру службы на основе функций.

В этой статье рассматриваются следующие вопросы:

* Различные разрешения, которые можно предоставить серверному приложению для доступа к службе подготовки.
* Процесс аутентификации и маркеры, используемые для проверки разрешений.

### <a name="when-to-use"></a>Назначение

Для доступа к любой конечной точке службы подготовки необходимы соответствующие разрешения. Например, серверное приложение должно содержать токен с учетными данными системы безопасности, а также все сообщения, отправленные в службу.

## <a name="access-control-and-permissions"></a>Контроль доступа и разрешений

Предоставить [разрешения](#device-provisioning-service-permissions) можно следующими способами:

* **Политики авторизации общего доступа**. Политики общего доступа могут предоставлять любое сочетание [разрешений](#device-provisioning-service-permissions). Политики можно задавать на [портале Azure][lnk-management-portal] или программными средствами, используя [интерфейсы REST API службы подготовки устройств][lnk-resource-provider-apis]. По умолчанию для только что созданной службы подготовки задана следующая политика по умолчанию:

* **provisioningserviceowner.** Политика со всеми разрешениями.

> [!NOTE]
> Дополнительные сведения см. в статье о [разрешениях](#device-provisioning-service-permissions).

## <a name="authentication"></a>Аутентификация

Служба подготовки устройств Центра Интернета вещей Azure предоставляет доступ к конечным точкам, проверяя токен на соответствие политикам общего доступа. Учетные данные безопасности, например симметричные ключи, никогда не отправляются по сети.

> [!NOTE]
> Безопасность для поставщика ресурсов службы подготовки устройств обеспечивается с помощью подписки Azure (это касается всех поставщиков в [Azure Resource Manager][lnk-azure-resource-manager]).

Дополнительные сведения о способах создания и использования маркеров безопасности см. в следующем разделе.

Поддерживается только протокол HTTP. Он реализует аутентификацию за счет включения допустимого токена в заголовок запроса **авторизации**.

#### <a name="example"></a>Пример
```csharp
SharedAccessSignature sr = 
   mydps.azure-devices-provisioning.net&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501&skn=provisioningserviceowner`\
```

> [!NOTE]
> [Пакеты SDK для службы подготовки устройств Интернета вещей Azure][lnk-sdks] автоматически создают токены при подключении к службе.

## <a name="security-tokens"></a>Маркеры безопасности Центра Интернета вещей

Служба подготовки устройств использует маркеры безопасности для аутентификации служб, чтобы избежать отправки ключей по сети. Кроме того, маркеры безопасности ограничены по времени и области действия. [Пакеты SDK для службы подготовки устройств Интернета вещей Azure][lnk-sdks] автоматически создают токены без специальной настройки. В некоторых случаях требуется напрямую создать и использовать маркеры безопасности. Эти сценарии включают непосредственное использование поверхности HTTP.

### <a name="security-token-structure"></a>Структура маркера безопасности

Маркеры безопасности используются для предоставления службам ограниченного по времени доступа к определенным функциям в службе подготовки устройств Интернета вещей. Чтобы получить авторизацию для подключения к службе подготовки, службы должны отправить маркеры безопасности, подписанные с помощью общего ключа доступа или симметричного ключа.

Маркер, подписанный с помощью ключа общего доступа, предоставляет доступа ко всем функциям, связанным с разрешениями политики общего доступа. 

Маркер безопасности имеет следующий формат:

`SharedAccessSignature sig={signature}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

Это ожидаемые значения:

| Значение | Описание |
| --- | --- |
| {signature} |Строка подписи HMAC-SHA256 формата `{URL-encoded-resourceURI} + "\n" + expiry`. **Важно!** Ключ шифруется в кодировке base64 и используется для вычислений HMAC-SHA256.|
| {expiry} |Строки в формате UTF8, отображающие количество секунд с начала эры 00:00:00 (в формате UTC) 1 января 1970 г. |
| {URL-encoded-resourceURI} | Строчное URL-кодирование строчного URL ресурса. Начинающийся с имени узла службы подготовки устройств Интернета вещей (без протокола) префикс URI (по сегменту) для конечных точек, доступ к которым можно получить с помощью этого маркера. Например, `mydps.azure-devices-provisioning.net`. |
| {policyName} |Имя политики общего доступа, к которой относится этот маркер. |

**Обратите внимание**, что префикс универсального кода ресурса (URI) вычисляется по сегменту, а не по символу. Например, `/a/b` — это префикс для `/a/b/c`, а не для `/a/bc`.

В следующем фрагменте кода Node.js показан функция **generateSasToken**, которая вычисляет маркер, используя входные данные`resourceUri, signingKey, policyName, expiresInMins`. В следующих разделах показано, как инициализировать различные входные данные для различных сценариев использования маркеров.

```javascript
var generateSasToken = function(resourceUri, signingKey, policyName, expiresInMins) {
    resourceUri = encodeURIComponent(resourceUri);

    // Set expiration in seconds
    var expires = (Date.now() / 1000) + expiresInMins * 60;
    expires = Math.ceil(expires);
    var toSign = resourceUri + '\n' + expires;

    // Use crypto
    var hmac = crypto.createHmac('sha256', new Buffer(signingKey, 'base64'));
    hmac.update(toSign);
    var base64UriEncoded = encodeURIComponent(hmac.digest('base64'));

    // Construct authorization string
    var token = "SharedAccessSignature sr=" + resourceUri + "&sig="
    + base64UriEncoded + "&se=" + expires + "&skn="+ policyName;
    return token;
};
```

Для сравнения приведен эквивалентный код Python для создания маркера безопасности.

```python
from base64 import b64encode, b64decode
from hashlib import sha256
from time import time
from urllib import quote_plus, urlencode
from hmac import HMAC

def generate_sas_token(uri, key, policy_name, expiry=3600):
    ttl = time() + expiry
    sign_key = "%s\n%d" % ((quote_plus(uri)), int(ttl))
    print sign_key
    signature = b64encode(HMAC(b64decode(key), sign_key, sha256).digest())

    rawtoken = {
        'sr' :  uri,
        'sig': signature,
        'se' : str(int(ttl)),
        'skn' : policy_name
    }

    return 'SharedAccessSignature ' + urlencode(rawtoken)
```

> [!NOTE]
> Так как срок действия токена проверяется на компьютерах службы подготовки устройств Интернета вещей, нужно обеспечить минимальное смещение на часах компьютера, где создается маркер.

### <a name="use-security-tokens-from-service-components"></a>Использование маркеров безопасности из компонентов службы

Компоненты службы могут создавать маркеры безопасности только с помощью политик общего доступа, предоставляющих соответствующие разрешения, как описано ранее.

Ниже приведены функции службы, предоставляемые в конечных точках.

| Конечная точка | Функциональность |
| --- | --- |
| `{your-service}.azure-devices-provisioning.net/enrollments` |Предоставляет операции регистрации устройства в службе подготовки устройств. |
| `{your-service}.azure-devices-provisioning.net/enrollmentGroups` |Предоставляет операции для управления группами регистрации устройств. |
| `{your-service}.azure-devices-provisioning.net/registrations/{id}` |Предоставляет операции для получения данных о состоянии регистрации устройств и управления ими. |


Например, служба, использующая предварительно созданную политику общего доступа с именем **enrollmentread**, создаст токен со следующими параметрами:

* универсальный код ресурса (URI): `{mydps}.azure-devices-provisioning.net`;
* ключ подписывания: один из ключей политики `enrollmentread` ;
* имя политики: `enrollmentread`;
* окончания срока действия time.backn.

![Создание политики общего доступа для экземпляра службы подготовки устройств на портале][img-add-shared-access-policy]

```javascript
var endpoint ="mydps.azure-devices-provisioning.net";
var policyName = 'enrollmentread'; 
var policyKey = '...';

var token = generateSasToken(endpoint, policyKey, policyName, 60);
```

Результат, предоставляющий доступ для чтения всех записей регистрации, будет иметь следующий вид:

`SharedAccessSignature sr=mydps.azure-devices-provisioning.net&sig=JdyscqTpXdEJs49elIUCcohw2DlFDR3zfH5KqGJo4r4%3D&se=1456973447&skn=enrollmentread`

## <a name="reference-topics"></a>Справочные материалы

В указанных ниже материалах предоставлены дополнительные сведения об управлении доступом к службе подготовки устройств Интернета вещей.

### <a name="device-provisioning-service-permissions"></a>Разрешения службы подготовки устройств

В следующей таблице указаны разрешения, с помощью которых можно управлять доступом к службе подготовки устройств Интернета вещей.

| Разрешение | Примечания |
| --- | --- |
| **ServiceConfig** |Предоставляет доступ для изменения конфигурации службы. <br/>Это разрешение используется серверными облачными службами. |
| **EnrollmentRead** |Предоставляет доступ на чтение к регистрации устройств и группам регистрации. <br/>Это разрешение используется серверными облачными службами. |
| **EnrollmentWrite** |Предоставляет доступ на запись к регистрации устройств и группам регистрации. <br/>Это разрешение используется серверными облачными службами. |
| **RegistrationStatusRead** |Предоставляет доступ на чтение к состоянию регистрации устройства. <br/>Это разрешение используется серверными облачными службами. |
| **RegistrationStatusWrite**  |Предоставляет удаленный доступ к состоянию регистрации устройства. <br/>Это разрешение используется серверными облачными службами. |

<!-- links and images -->

[img-add-shared-access-policy]: ./media/how-to-control-access/how-to-add-shared-access-policy.PNG
[lnk-sdks]: ../iot-hub/iot-hub-devguide-sdks.md
[lnk-management-portal]: https://portal.azure.com
[lnk-azure-resource-manager]: ../azure-resource-manager/management/overview.md
[lnk-resource-provider-apis]: /rest/api/iot-dps/
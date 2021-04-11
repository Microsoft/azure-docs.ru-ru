---
title: Учебник. Подписание и создание запросов к API SMS для службы ACS с помощью Postman
titleSuffix: An Azure Communication Services tutorial
description: Узнайте, как подписывать и делать запросы к ACS с помощью Postman для отправки SMS-сообщения.
author: ProbablePrime
services: azure-communication-services
ms.author: rifox
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 5805734a9253962d672a4236a5650e9de8b37f0a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105044300"
---
# <a name="tutorial-sign-and-make-requests-with-postman"></a>Учебник. Подписание и создание запросов с помощью Postman
В этом руководстве мы будем настраивать и использовать Postman для выполнения запросов к Службам коммуникации Azure (ACS) с помощью HTTP. По завершении работы с этим руководством вы отправите SMS-сообщение с помощью ACS и Postman и сможете использовать Postman для изучения других API-интерфейсов в ACS.

В этом учебнике мы выполним следующее:
> [!div class="checklist"]
> * Скачивание Postman.
> * Настройка Postman для подписи HTTP-запросов.
> * Запрос к API SMS для службы ACS для отправки сообщения.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). Бесплатная учетная запись предусматривает предоставление 200 долл. США в качестве денег на счете в Azure, что позволяет испытать в работе любое сочетание служб.
- Активный ресурс Служб коммуникации и строка подключения. [Практическое руководство по созданию ресурса Служб коммуникации.](../quickstarts/create-communication-resource.md)
- Номер телефона ACS, с которого можно отправить SMS-сообщения, см. в [кратком руководстве по получению номера телефона с помощью портала Azure](../quickstarts/telephony-sms/get-phone-number.md).

## <a name="downloading-and-installing-postman"></a>Скачивание и установка Postman

Postman — это классическое приложение, способное выполнять запросы API к любому API HTTP. Он обычно используется для тестирования и изучения API-интерфейсов. Мы скачаем последнюю [версию для настольных систем с веб-сайта Postman](https://www.postman.com/downloads/). Существуют версии Postman для Windows, Mac и Linux, поэтому скачайте версию, соответствующую вашей операционной системе. После скачивания откройте приложение. Появится окно с начальным экраном, в котором будет предложено выполнить вход или создать учетную запись Postman. Создание учетной записи является необязательным. Его можно пропустить, щелкнув ссылку Skip and go to app (Пропустить и перейти к приложению). При создании учетной записи параметры запроса API будут сохранены в Postman, что позволит вам применять запросы на других компьютерах.

:::image type="content" source="media/postman/create-account-or-skip.png" alt-text="Возможность создания учетной записи или пропуска и перехода к приложению на начальном экране Postman.":::

После создания учетной записи или пропуска этого шага отобразится главное окно Postman.

## <a name="creating-and-configuring-a-postman-collection"></a>Создание и настройка коллекции Postman

После этого запросы могут упорядочиваться различными способами. В рамках изучения этого учебника мы создадим коллекцию Postman. Для этого нажмите кнопку Collections (Коллекции) в левой части приложения:

:::image type="content" source="media/postman/collections-tab.png" alt-text="Главный экран Postman с выделенной вкладкой Collections (Коллекции).":::

Затем щелкните Create new Collection (Создать коллекцию), чтобы начать процесс создания коллекции. В центральной области Postman откроется новая вкладка. Присвойте коллекции любое имя. Здесь коллекция называется "ACS":

:::image type="content" source="media/postman/acs-collection.png" alt-text="После этого откроется коллекция ACS с выделенным именем коллекции.":::

После создания коллекции и присвоения ей имени можно приступать к ее настройке.

### <a name="adding-collection-variables"></a>Добавление переменных коллекции

Для обеспечения проверки подлинности и упрощения запросов мы будем указывать две переменные коллекции в только что созданной коллекции ACS. Эти переменные доступны для всех запросов в коллекции ACS. Чтобы приступить к созданию переменных, посетите вкладку Variables (Переменные) в разделе Collections (Коллекции).

:::image type="content" source="media/postman/variable-stab.png" alt-text="Postman со вкладкой Variables (Переменные) в разделе Collections (Коллекции).":::

На вкладке Collections (Коллекции) создайте две переменные:
- key — эта переменная должна быть одним из ключей со страницы ключа Службы коммуникации Azure на портале Azure. Например, `oW...A==`.
- endpoint — эта переменная должна быть конечной точкой Службы коммуникации Azure на странице ключей. **Убедитесь, что вы удалили замыкающую косую черту**. Например, `https://contoso.communication.azure.com`.

Введите эти значения в столбец Initial Value (Начальное значение) на экране Variables (Переменные). После входа нажмите кнопку Persist All (Сохранить все) над таблицей справа. При правильной настройке экран Postman должен выглядеть примерно так:

:::image type="content" source="media/postman/acs-variables-set.png" alt-text="Postman с правильно настроенными переменными коллекции ACS.":::

Дополнительные сведения о переменных см. [в документации по Postman](https://learning.postman.com/docs/sending-requests/variables).

### <a name="creating-a-pre-request-script"></a>Создание скрипта, выполняемого перед запросом

Следующим шагом является создание скрипта, выполняемого перед запросом, в Postman. Скрипт, выполняемый перед запросом, — это скрипт, который выполняется перед каждым запросом в Postman и может изменять параметры запроса от вашего имени. Мы будем использовать его для подписывания HTTP-запросов, чтобы они могли быть разрешены службами ACS. Дополнительные сведения о требованиях к подписыванию см. [в нашем справочном материале по проверке подлинности](/rest/api/communication/authentication).

Мы создадим этот скрипт в коллекции так, чтобы он выполнялся по любому запросу в коллекции. Для этого на вкладке Collections (Коллекции) щелкните вложенную вкладку Pre-request Script (Скрипт перед запросом).

:::image type="content" source="media/postman/start-pre-request-script.png" alt-text="Postman с выбранной вложенной вкладкой Pre-request Script (Скрипт перед запросом) коллекции ACS. ":::

На этой вложенной вкладке можно создать скрипт, выполняемый перед запросом, введя его в текстовом поле ниже. Возможно, будет проще написать это в редакторе кода, таком как [Visual Studio Code](https://code.visualstudio.com/), прежде чем вставлять его по завершению. В этом учебнике мы будем рассматривать все части скрипта. Вы можете перейти к завершающему этапу обучения, если хотите просто скопировать его в Postman и начать работу. Давайте создадим скрипт.

### <a name="writing-the-pre-request-script"></a>Написание скрипта, выполняемого перед запросом

Сначала создадим строку в формате UTC и добавим ее в заголовок HTTP "Дата". Мы также сохраним эту строку в переменной, чтобы использовать ее позже при подписывании:

```JavaScript
// Set the Date header to our Date as a UTC String.
const dateStr = new Date().toUTCString();
pm.request.headers.upsert({key:'Date', value: dateStr});
```

Далее мы создадим хэш текста запроса с помощью шифрования SHA 256 и поместим его в заголовок `x-ms-content-sha256`. Postman содержит [стандартные библиотеки](https://learning.postman.com/docs/writing-scripts/script-references/postman-sandbox-api-reference/#using-external-libraries) для глобального хэширования и подписывания, поэтому нам не нужно их устанавливать:

```JavaScript
// Hash the request body using SHA256 and encode it as Base64
const hashedBodyStr = CryptoJS.SHA256(pm.request.body.raw).toString(CryptoJS.enc.Base64)
// And add that to the header x-ms-content-sha256
pm.request.headers.upsert({
    key:'x-ms-content-sha256',
    value: hashedBodyStr
});
```

Теперь мы используем ранее указанную переменную конечной точки, чтобы обозначить значение заголовка узла HTTP. Это необходимо сделать, так как заголовок узла можно задать только после обработки этого скрипта:

```JavaScript
// Get our previously specified endpoint variable
const endpoint = pm.variables.get('endpoint')
// Remove the https, prefix to create a suitable "Host" value
const hostStr = endpoint.replace('https://','');
```

После этого можно создать строку, которая будет подписываться для HTTP-запроса. Для этого потребуются несколько ранее созданных значений:

```JavaScript
// This gets the part of our URL that is after the endpoint, for example in https://contoso.communication.azure.com/sms, it will get '/sms'
const url = pm.request.url.toString().replace('{{endpoint}}','');

// Construct our string which we'll sign, using various previously created values.
const stringToSign = pm.request.method + '\n' + url + '\n' + dateStr + ';' + hostStr + ';' + hashedBodyStr;
```

Наконец, необходимо подписать эту строку с помощью нашего ключа ACS, а затем добавить ее в наш запрос в заголовке `Authorization`:

```JavaScript
// Decode our access key from previously created variables, into bytes from base64.
const key = CryptoJS.enc.Base64.parse(pm.variables.get('key'));
// Sign our previously calculated string with HMAC 256 and our key. Convert it to Base64.
const signature = CryptoJS.HmacSHA256(stringToSign, key).toString(CryptoJS.enc.Base64);

// Add our final signature in Base64 to the authorization header of the request.
pm.request.headers.upsert({
    key:'Authorization',
    value: "HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=" + signature
});
```

### <a name="the-final-pre-request-script"></a>Окончательный скрипт, выполняемый перед запросом

Готовый скрипт, выполняемый перед запросом, должен выглядеть примерно так:

```JavaScript
// Set the Date header to our Date as a UTC String.
const dateStr = new Date().toUTCString();
pm.request.headers.upsert({key:'Date', value: dateStr});

// Hash the request body using SHA256 and encode it as Base64
const hashedBodyStr = CryptoJS.SHA256(pm.request.body.raw).toString(CryptoJS.enc.Base64)
// And add that to the header x-ms-content-sha256
pm.request.headers.upsert({
    key:'x-ms-content-sha256',
    value: hashedBodyStr
});

// Get our previously specified endpoint variable
const endpoint = pm.variables.get('endpoint')
// Remove the https, prefix to create a suitable "Host" value
const hostStr = endpoint.replace('https://','');

// This gets the part of our URL that is after the endpoint, for example in https://contoso.communication.azure.com/sms, it will get '/sms'
const url = pm.request.url.toString().replace('{{endpoint}}','');

// Construct our string which we'll sign, using various previously created values.
const stringToSign = pm.request.method + '\n' + url + '\n' + dateStr + ';' + hostStr + ';' + hashedBodyStr;

// Decode our access key from previously created variables, into bytes from base64.
const key = CryptoJS.enc.Base64.parse(pm.variables.get('key'));
// Sign our previously calculated string with HMAC 256 and our key. Convert it to Base64.
const signature = CryptoJS.HmacSHA256(stringToSign, key).toString(CryptoJS.enc.Base64);

// Add our final signature in Base64 to the authorization header of the request.
pm.request.headers.upsert({
    key:'Authorization',
    value: "HMAC-SHA256 SignedHeaders=date;host;x-ms-content-sha256&Signature=" + signature
});
```

Введите или вставьте этот окончательный скрипт в текстовую область на вкладке Pre-request Script (Скрипт перед запросом):

:::image type="content" source="media/postman/finish-pre-request.png" alt-text="Postman с указанным скриптом, выполняемым перед запросом, коллекции ACS.":::

Затем нажмите клавиши CTRL+S или кнопку Save (Сохранить), чтобы сохранить скрипт в коллекции. 

:::image type="content" source="media/postman/save-pre-request-script.png" alt-text="Сохранение скрипта, выполняемого перед запросом, в Postman.":::

## <a name="creating-a-request-in-postman"></a>Создание запроса в Postman

Теперь, когда все настроено, создадим запрос ACS в Postman. Чтобы начать работу, щелкните значок плюса (+) рядом с коллекцией ACS:

:::image type="content" source="media/postman/create-request.png" alt-text="Кнопка со знаком &quot;плюс&quot;.":::

Будет создана новая вкладка для нашего запроса в Postman. После создания его следует настроить. Мы выполним запрос к API отправки SMS, поэтому обязательно ознакомьтесь с [документацией по этому API](/rest/api/communication/sms/send). Давайте настроим запрос Postman.

Настройте тип запроса `POST` и введите `{{endpoint}}/sms?api-version=2021-03-07` в поле URL-адреса запроса. Этот URL-адрес использует ранее созданную переменную `endpoint` для автоматической отправки ее в ресурс ACS.

:::image type="content" source="media/postman/post-request-and-url.png" alt-text="Запрос Postman с типом POST и указанным URL-адресом.":::

Теперь выберите вкладку Body (Текст) запроса и переведите переключатель в положение RAW. Справа отображается раскрывающийся список Text (Текст). Измените его на JSON:

:::image type="content" source="media/postman/postman-json.png" alt-text="Установка для текста запроса формата RAW и JSON":::

Будет настроен запрос на отправку и получение сведений в формате JSON.

В текстовой области ниже необходимо ввести текст запроса, который должен иметь следующий формат:

```JSON
{
    "from":"<Your ACS Telephone Number>",
    "message":"<The message you'd like to send>",
    "smsRecipients": [
        {
            "to":"<The number you'd like to send the message to>"
        }
    ]
}
```

Чтобы указать значение from (от), необходимо [получить номер телефона](../quickstarts/telephony-sms/get-phone-number.md) на портале ACS, как упоминалось ранее. Введите его без пробелов и укажите код страны. Например, `+15555551234`. В поле message (сообщение) можно ввести любой текст, но лучше указать `Hello from ACS`. Значение to (кому) должно быть телефоном, к которому у вас есть доступ, чтобы вы могли получать SMS-сообщения. Лучше использовать собственный номер мобильного телефона.

После входа необходимо сохранить этот запрос в созданной ранее коллекции ACS. Это обеспечит получение ранее созданных переменных и скрипта, выполняемого перед запросом. Для этого нажмите кнопку Save (Сохранить) в правом верхнем углу области запроса.

:::image type="content" source="media/postman/postman-save.png" alt-text="Кнопка Save (Сохранить) для запроса Postman.":::

Откроется диалоговое окно с запросом о том, что необходимо вызвать и где сохранить данные. Вы можете назвать его как угодно, но убедитесь, что вы выбрали коллекцию ACS в нижней части диалогового окна:

:::image type="content" source="media/postman/postman-save-to-acs.png" alt-text="Диалоговое окно отправки запроса на сохранение с выбранной коллекцией ACS.":::

## <a name="sending-a-request"></a>Отправка запроса

Теперь, когда все настроено, вы сможете отправить запрос и получить SMS на телефон. Для этого убедитесь, что созданный запрос выбран и нажмите кнопку Send (Отправить) справа:

:::image type="content" source="media/postman/postman-send.png" alt-text="Запрос Postman с выделенной кнопкой Send (Отправить).":::

Если все прошло успешно, отобразится ответ от службы ACS, который должен иметь код состояния 202:

:::image type="content" source="media/postman/postman-202.png" alt-text="Успешно отправленный запрос Postman с кодом состояния 202.":::

Мобильный телефон, которому принадлежит номер, указанный в значении to (кому), должен также получать SMS. Теперь Postman настроен после установки, и он может взаимодействовать со службами ACS и отправлять SMS.


## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Знакомство с API ACS](/rest/api/communication/)
> [Дополнительные сведения о проверке подлинности](/rest/api/communication/authentication)
> [Дополнительные сведения о Postman](https://learning.postman.com/)

Кроме того, вам может понадобиться следующее:

- [Добавление чата в приложение](../quickstarts/chat/get-started.md)
- [Создание маркеров доступа пользователей](../quickstarts/access-tokens.md)
- [Сведения об архитектуре клиент-сервер](../concepts/client-and-server-architecture.md)
- [Сведения о проверке подлинности](../concepts/authentication.md)
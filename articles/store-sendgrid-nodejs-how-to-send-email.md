---
title: Как использовать службу электронной почты SendGrid (Node.js) | Документация Майкрософт
description: Узнайте, как отправить электронное сообщение с помощью службы электронной почты SendGrid в Azure. Примеры написаны с использованием API Node.js.
services: ''
documentationcenter: nodejs
author: erikre
manager: wpickett
editor: ''
ms.assetid: cac444b4-26b0-45ea-9c3d-eca28d57dacb
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 01/05/2016
ms.author: erikre
ms.openlocfilehash: f2d653441598a47986913d525057672eed24b435
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96015436"
---
# <a name="how-to-send-email-using-sendgrid-from-nodejs"></a>Как отправлять электронную почту с помощью SendGrid из Node.js

В этом руководстве показано, как выполнять типовые задачи программирования для службы электронной почты SendGrid в Azure. Примеры написаны с использованием интерфейса API Node.js. Описываемые здесь сценарии включают в себя **создание электронного сообщения**, **отправку электронного сообщения**, **добавление вложений**, **использование фильтров** и **обновление свойств**. Дополнительные сведения о SendGrid и отправке электронной почты см. в разделе [Дальнейшие действия](#next-steps).

## <a name="what-is-the-sendgrid-email-service"></a>Что такое служба электронной почты SendGrid?

SendGrid — это [облачная служба электронной почты], которая обеспечивает надежную [доставку транзакционных писем], предоставляет возможности масштабирования и аналитики в реальном времени, а также предоставляет гибкие интерфейсы API для пользовательской интеграции. Ниже перечислены наиболее распространенные сценарии использования SendGrid.

* Автоматическая отправка уведомлений о получении клиентам
* Администрирование списков рассылки для ежемесячной отправки клиентам электронных листовок и специальных предложений
* Сбор показателей в режиме реального времени по таким параметрам, как заблокированная электронная почта и реагирование клиентов
* Создание отчетов для определения тенденций
* Пересылка запросов клиентов
* Уведомления от приложения по электронной почте

Дополнительные сведения см. на веб-сайте [https://sendgrid.com](https://sendgrid.com).

## <a name="create-a-sendgrid-account"></a>Создание учетной записи SendGrid

[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="reference-the-sendgrid-nodejs-module"></a>Ссылка на модуль Node.js SendGrid

Модуль SendGrid для Node.js можно установить через диспетчер пакетов узлов (npm) с помощью следующей команды:

```bash
npm install sendgrid
```

После установки вы можете запросить наличие этого модуля в приложении, используя следующий код:

```javascript
var sendgrid = require('sendgrid')(sendgrid_username, sendgrid_password);
```

Модуль SendGrid экспортирует функции **SendGrid** и **Email**.
**SendGrid** отвечает за отправку почты через SMTP или веб-API, а **Email** инкапсулирует электронное сообщение.

## <a name="how-to-create-an-email"></a>Практическое руководство. Создание сообщения эл. почты

Для создания электронного сообщения с помощью модуля SendGrid сначала создается электронное сообщение с помощью функции Email, а затем оно отправляется с помощью функции SendGrid. В следующем примере показано создание нового сообщения с помощью функции Email:

```javascript
var email = new sendgrid.Email({
    to: 'john@contoso.com',
    from: 'anna@contoso.com',
    subject: 'test mail',
    text: 'This is a sample email message.'
});
```

Вы также можете создать HTML-сообщение для клиентов, которые поддерживают этот формат, задав свойство html. Пример:

```javascript
html: This is a sample <b>HTML<b> email message.
```

Задав свойства text и html, можно обеспечить корректный переход к текстовому содержимому для клиентов, которые не поддерживают формат HTML.

Дополнительную информацию обо всех свойствах, поддерживаемых функцией Email, см. в разделе [sendgrid-nodejs][sendgrid-nodejs].

## <a name="how-to-send-an-email"></a>Практическое руководство. Отправка сообщения эл. почты

После создания электронного сообщения с помощью функции Email его можно отправить с помощью веб-интерфейса API, предоставленного SendGrid. 

### <a name="web-api"></a>Веб-интерфейс API

```javascript
sendgrid.send(email, function(err, json){
    if(err) { return console.error(err); }
    console.log(json);
});
```

> [!NOTE]
> Хотя в приведенных выше примерах показана передача для объекта электронной почты и функции обратного вызова, вы можете также напрямую вызвать функции отправки, задав свойства электронной почты. Пример:  
> 
> ```javascript
> sendgrid.send({
> to: 'john@contoso.com',
> from: 'anna@contoso.com',
> subject: 'test mail',
> text: 'This is a sample email message.'
> });
> ```
>

## <a name="how-to-add-an-attachment"></a>Практическое руководство. Добавление вложения
В сообщение можно добавить вложения, указав имена файлов и пути к ним в свойстве **files**. В следующем примере показана отправка вложения.

```javascript
sendgrid.send({
    to: 'john@contoso.com',
    from: 'anna@contoso.com',
    subject: 'test mail',
    text: 'This is a sample email message.',
    files: [
        {
            filename:     '',           // required only if file.content is used.
            contentType:  '',           // optional
            cid:          '',           // optional, used to specify cid for inline content
            path:         '',           //
            url:          '',           // == One of these three options is required
            content:      ('' | Buffer) //
        }
    ],
});
```

> [!NOTE]
> При использовании свойства **files** файл должен быть доступен через [fs.readFile](https://nodejs.org/docs/v0.6.7/api/fs.html#fs.readFile). Если вы хотите вложить файл, находящийся в службе хранилища Azure, например контейнер больших двоичных объектов, необходимо сначала скопировать этот файл в локальное хранилище или на диск Azure, прежде чем его можно будет отправить как вложение с помощью свойства **files**.
> 
> 

## <a name="how-to-use-filters-to-enable-footers-and-tracking"></a>Практическое руководство. Использование фильтров для включения нижних колонтитулов и отслеживания

SendGrid поддерживает дополнительные функциональные возможности электронной почты благодаря использованию фильтров. Это параметры, которые могут быть добавлены в сообщение электронной почты для включения определенных функций, например включения отслеживания щелчков, аналитики Google, отслеживания подписок и т. д. Полный список фильтров см. в разделе [Filter Settings][Filter Settings] (Параметры фильтрации).

Фильтры могут применяться к сообщению с использованием свойства **filters**.
Каждый фильтр определяется хэшем, содержащим параметры, которые связаны с данным конкретным фильтром.
В следующих примерах показаны фильтры нижних колонтитулов и отслеживания щелчков:

### <a name="footer"></a>Нижний колонтитул

```javascript
var email = new sendgrid.Email({
    to: 'john@contoso.com',
    from: 'anna@contoso.com',
    subject: 'test mail',
    text: 'This is a sample email message.'
});

email.setFilters({
    'footer': {
        'settings': {
            'enable': 1,
            'text/plain': 'This is a text footer.'
        }
    }
});

sendgrid.send(email);
```

### <a name="click-tracking"></a>Отслеживание щелчков

```javascript
var email = new sendgrid.Email({
    to: 'john@contoso.com',
    from: 'anna@contoso.com',
    subject: 'test mail',
    text: 'This is a sample email message.'
});

email.setFilters({
    'clicktrack': {
        'settings': {
            'enable': 1
        }
    }
});

sendgrid.send(email);
```

## <a name="how-to-update-email-properties"></a>Практическое руководство. Обновление свойств электронной почты

Некоторые свойства электронной почты можно перезаписать с помощью оператора **setProperty** или добавить с помощью оператора **addProperty**. Например, вы можете добавить дополнительных получателей с помощью

```javascript
email.addTo('jeff@contoso.com');
```

или задать фильтр с помощью

```javascript
email.addFilter('footer', 'enable', 1);
email.addFilter('footer', 'text/html', '<strong>boo</strong>');
```

Дополнительные сведения можно найти в разделе [sendgrid-nodejs][sendgrid-nodejs].

## <a name="how-to-use-additional-sendgrid-services"></a>Практическое руководство. Использование дополнительных служб SendGrid

SendGrid поддерживает различные веб-интерфейсы API, с помощью которых вы можете использовать дополнительные функции SendGrid, доступные в приложении Azure. Дополнительные сведения см. в [документации по интерфейсу API SendGrid][SendGrid API documentation].

## <a name="next-steps"></a>Next Steps

Вы получили основные сведения о службе доставки электронной почты SendGrid. Дополнительные сведения вы найдете по следующим ссылкам.

* Репозиторий модуля SendGrid для Node.js: [sendgrid-nodejs][sendgrid-nodejs].
* Документация по API SendGrid: <https://sendgrid.com/docs>
* Специальное предложение SendGrid для клиентов Azure: [http://sendgrid.com/azure.html](https://sendgrid.com/windowsazure.html)

[special offer]: https://sendgrid.com/windowsazure.html
[sendgrid-nodejs]: https://github.com/sendgrid/sendgrid-nodejs
[Filter Settings]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html
[SendGrid API documentation]: https://sendgrid.com/docs
[облачная служба электронной почты]: https://sendgrid.com/email-solutions
[доставку электронной почты]: https://sendgrid.com/transactional-email

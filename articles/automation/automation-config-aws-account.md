---
title: Аутентификация модулей runbook службы автоматизации Azure с помощью Amazon Web Services
description: В этой статье рассказывается, как выполнять проверку подлинности модулей Runbook с помощью Amazon Web Services.
keywords: проверка подлинности aws, настройка aws
services: automation
ms.subservice: process-automation
ms.date: 04/23/2020
ms.topic: conceptual
ms.openlocfilehash: acb056fb959fda320a14059e2b36a0f5e3fb0b37
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83837197"
---
# <a name="authenticate-runbooks-with-amazon-web-services"></a>Проверка подлинности модулей runbook с помощью Amazon Web Services

Автоматизацию стандартных задач с использованием ресурсов в Amazon Web Services (AWS) можно выполнить с помощью модулей Runbook службы автоматизации в Azure. Многие задачи можно автоматизировать в AWS с помощью модулей Runbook службы автоматизации так же, как и с помощью ресурсов в Azure. Для проверки подлинности необходима подписка Azure.

## <a name="obtain-aws-subscription-and-credentials"></a>Получение подписки и учетных данных AWS

Чтобы пройти проверку подлинности в AWS, необходимо получить подписку AWS и указать набор учетных данных AWS для проверки подлинности модулей Runbook, запущенных в службе автоматизации Azure. Требуются определенные учетные данные: ключ доступа AWS и секретный ключ. См. статью [Использование учетных данных AWS](https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html).

## <a name="configure-automation-account"></a>Настройка учетной записи службы автоматизации

Для проверки подлинности с помощью AWS можно использовать существующую учетную запись службы автоматизации. Кроме того, можно выделить учетную запись для модулей Runbook, предназначенных для ресурсов AWS. В этом случае [создайте новую учетную запись службы автоматизации](automation-create-standalone-account.md).  

## <a name="store-aws-credentials"></a>Сохранение учетных данных AWS

Учетные данные AWS необходимо хранить как ресурсы в службе автоматизации Azure. Инструкции по созданию ключа доступа и секретного ключа см. в разделе [Управление ключами доступа для учетной записи AWS](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html). Когда ключи будут доступны, скопируйте идентификатор ключа доступа и идентификатор секретного ключа в надежное место. Можно скачать файл ключа, чтобы сохранить его в безопасном месте.

## <a name="create-credential-asset"></a>Создание ресурса учетных данных

После создания и копирования ключей безопасности AWS необходимо создать ресурс учетных данных с учетной записью службы автоматизации. Ресурс позволяет безопасно хранить ключи AWS и ссылаться на них в модулях Runbook. См. раздел [Создание нового ресурса учетных данных на портале Azure](shared-resources/credentials.md#create-a-new-credential-asset-with-the-azure-portal). Введите следующие сведения AWS в указанные поля.
    
* В поле **Имя** - **AWScred** или другое значение, соответствующее вашим стандартам именования.
* **Имя пользователя** — ваш идентификатор доступа.
* **Пароль** — имя секретного ключа. 

## <a name="next-steps"></a>Дальнейшие действия

* Сведения о создании модулей Runbook для автоматизации задач в AWS см. в статье [Развертывание виртуальных машин Amazon Web Services с помощью модуля Runbook](automation-scenario-aws-deployment.md).
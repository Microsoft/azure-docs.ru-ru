---
title: Добавление пользовательского сертификата ЦС — служба управления API Azure | Документация Майкрософт
description: Узнайте, как добавить пользовательский сертификат ЦС в службе управления API Azure. Вы также можете просмотреть инструкции по удалению сертификата.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/20/2018
ms.author: apimpm
ms.openlocfilehash: 124bc053aa2c6e59e205bb6f33a9a96190799499
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93102043"
---
# <a name="how-to-add-a-custom-ca-certificate-in-azure-api-management"></a>Добавление пользовательского сертификата ЦС в службе управления API Azure

Служба управления API Azure позволяет выполнять установку сертификатов ЦС на компьютере внутри доверенных корневых и промежуточных хранилищ сертификатов. Эту функциональность следует использовать, если вашим службам требуется пользовательский сертификат ЦС.

В этом руководстве описывается, как на портале Azure управлять сертификатами в экземпляре службы управления API Azure.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="upload-a-ca-certificate"></a><a name="step1"> </a>Передача сертификата ЦС

![Добавление сертификатов ЦС](media/api-management-howto-ca-certificates/00.png)

Чтобы передать новый сертификат ЦС, выполните действия ниже. Если экземпляр службы управления API еще не создан, выполните инструкции из руководства [Создание экземпляра службы управления API Azure](get-started-create-service-instance.md).

1. На портале Azure перейдите к экземпляру службы управления API Azure.

2. В меню выберите **Сертификаты ЦС**.

3. Щелкните **+ Добавить**.  

    ![Снимок экрана, на котором показана кнопка "+ Добавить" для добавления сертификата ЦС.](media/api-management-howto-ca-certificates/01.png)  

4. Перейдите к сертификату и выберите хранилище сертификатов. Нужен только открытый ключ, поэтому пароль не требуется.

    ![Снимок экрана, на котором показано, как найти сертификат.](media/api-management-howto-ca-certificates/02.png)  

5. Выберите команду **Сохранить**. Это может занять несколько минут.

    ![Снимок экрана, показывающий, как сохранить сертификат.](media/api-management-howto-ca-certificates/03.png)  

> [!NOTE]
> Вы можете отправить сертификат ЦС с помощью команды PowerShell `New-AzApiManagementSystemCertificate`.

## <a name="delete-a-client-certificate"></a><a name="step1a"> </a>Удаление сертификата клиента

Чтобы удалить сертификат, щелкните контекстное меню **...** и выберите **Удалить** рядом с именем сертификата.

![Удаление сертификатов ЦС](media/api-management-howto-ca-certificates/04.png)  

[Upload a CA certificate]: #step1
[Delete a CA certificate]: #step1a

---
title: Подключение веб-клиента Windows Virtual Desktop (классическая модель) к Azure
description: Как подключиться к виртуальному рабочему столу Windows (классическая модель) с помощью веб-клиента.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: c8a55e35ba89caefc362024fc871ac5eb800447b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88008429"
---
# <a name="connect-to-windows-virtual-desktop-classic-with-the-web-client"></a>Подключение к виртуальному рабочему столу Windows (классическая модель) с помощью веб-клиента

>[!IMPORTANT]
>Это содержимое применимо к Виртуальному рабочему столу Windows (классическому), который не поддерживает объекты Azure Resource Manager для Виртуального рабочего стола Windows. Сведения об обеспечении управления объектами Azure Resource Manager для Виртуального рабочего стола Windows см. в [этой статье](../connect-web.md).

Веб-клиент позволяет получить доступ к ресурсам Виртуального рабочего стола Windows из веб-браузера без необходимости устанавливать программное обеспечение.

>[!NOTE]
>В настоящее время веб-клиент не поддерживает мобильные ОС.

## <a name="supported-operating-systems-and-browsers"></a>Поддерживаемые операционные системы и браузеры

Несмотря на то, что подойдет любой браузер, поддерживающий HTML5, корпорация Майкрософт официально поддерживает следующие операционные системы и браузеры.

| Браузер           | Поддерживаемая ОС                     | Примечания               |
|-------------------|----------------------------------|---------------------|
| Microsoft Edge    | Windows                          |                     |
| Internet Explorer | Windows                          |                     |
| Apple Safari;      | macOS                            |                     |
| Mozilla Firefox   | Windows, macOS, Linux            | Версии 55 или более поздней |
| Google Chrome     | Windows, macOS, Linux, ОС Chrome |                     |

## <a name="access-remote-resources-feed"></a>Доступ к веб-каналу удаленных ресурсов

В браузере перейдите к веб-клиенту Виртуального рабочего стола Windows по адресу <https://rdweb.wvd.microsoft.com/webclient> и выполните вход с помощью учетной записи пользователя.

>[!NOTE]
>Если вы используете виртуальный рабочий стол Windows с Azure Resource Managerной интеграцией, подключитесь к ресурсам по адресу <https://rdweb.wvd.microsoft.com/arm/webclient> .

>[!NOTE]
>Если вы уже вошли, используя другую учетную запись Azure Active Directory, отличную от той, которую вы хотите использовать для Виртуального рабочего стола Windows, необходимо выйти из системы или использовать вкладку браузера в режиме конфиденциального просмотра.

После входа в систему отобразится список ресурсов. Чтобы запустить ресурсы, выберите их, как при работе с обычным приложением, на вкладке **Все ресурсы**.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании веб-клиента см. в статье [Начало работы с веб-клиентом](/windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client).

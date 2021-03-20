---
title: Подключение к службе приложений Azure с базой данных Azure для MySQL
description: Инструкции по правильному подключению существующей службы приложений Azure к базе данных Azure для MySQL
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: 6355afe6ce5decbed029db4536b1b1b19f5a876c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94541510"
---
# <a name="connect-an-existing-azure-app-service-to-azure-database-for-mysql-server"></a>Подключение существующей службы приложений Azure к базе данных Azure для сервера MySQL
В этом разделе объясняется, как подключить имеющуюся службу приложений Azure к базе данных Azure для сервера MySQL.

## <a name="before-you-begin"></a>Перед началом
Войдите на [портал Azure](https://portal.azure.com). Создайте базу данных Azure для сервера MySQL. Дополнительные сведения см. в статье [Создание базы данных Azure для сервера MySQL с помощью портала Azure](quickstart-create-mysql-server-database-using-azure-portal.md) или [Создание сервера базы данных Azure для MySQL с помощью Azure CLI](quickstart-create-mysql-server-database-using-azure-cli.md).

В настоящее время существует два решения предоставления доступа из службы приложений Azure к базе данных Azure для MySQL. Оба решения включают настройку правил брандмауэра уровня сервера.

## <a name="solution-1---allow-azure-services"></a>Решение 1: разрешение доступа к службам Azure
База данных Azure для MySQL обеспечивает безопасность доступа, используя брандмауэр для защиты данных. При настройке подключения из службы приложений Azure к базе данных Azure для сервера MySQL имейте в виду, что исходящие IP-адреса службы приложений являются динамическими по своей природе. Если выбрать параметр "Разрешить доступ к службам Azure", служба приложений сможет подключаться к серверу MySQL.

1. В колонке сервера MySQL в разделе "Параметры" щелкните **Безопасность подключения**, чтобы открыть колонку "Безопасность подключения" базы данных Azure для MySQL.

   :::image type="content" source="./media/howto-connect-webapp/1-connection-security.png" alt-text="Портал Azure: щелчок пункта &quot;Безопасность подключения&quot;":::

2. Выберите значение **ВКЛ.** для параметра **Разрешить доступ к службам Azure**, а затем нажмите кнопку **Сохранить**.
   :::image type="content" source="./media/howto-connect-webapp/allow-azure.png" alt-text="Портал Azure: параметр &quot;Разрешить доступ к службам Azure&quot;":::

## <a name="solution-2---create-a-firewall-rule-to-explicitly-allow-outbound-ips"></a>Решение 2. Создание правила брандмауэра, явно разрешающего исходящие IP-адреса
Вы можете явно добавить все исходящие IP-адреса службы приложений Azure.

1. В колонке "Свойства службы приложений" обратите внимание на поле **ИСХОДЯЩИЙ IP-АДРЕС**.

   :::image type="content" source="./media/howto-connect-webapp/2_1-outbound-ip-address.png" alt-text="Портал Azure — просмотр исходящих IP-адресов":::

2. В колонке "Безопасность подключения" для MySQL добавьте исходящие IP-адреса по одному.

   :::image type="content" source="./media/howto-connect-webapp/2_2-add-explicit-ips.png" alt-text="Портал Azure — добавление явных IP-адресов":::

3. Не забывайте **сохранять** правила брандмауэра.

Хотя служба приложений Azure пытается поддерживать постоянные IP-адреса, бывают случаи, когда IP-адреса могут измениться. Например, это происходит, когда приложение выполняет перезапуск или возникает операция масштабирования, а также при добавлении новых компьютеров в региональные центры обработки данных Azure для повышения емкости. При изменении IP-адресов может наблюдаться простой приложения, если оно больше не может подключиться к серверу MySQL. Учтите это при выборе одного из предыдущих решений.

## <a name="ssl-configuration"></a>Настройка SSL
Протокол SSL по умолчанию включен для базы данных Azure для MySQL. Если приложение не использует SSL для подключения к базе данных, необходимо отключить SSL для сервера MySQL. Дополнительные сведения о настройке SSL см. в статье [Настройка SSL-подключений в приложении для безопасного подключения к базе данных Azure для MySQL](howto-configure-ssl.md).

### <a name="django-pymysql"></a>Django (Пимискл)
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'quickstartdb',
        'USER': 'myadmin@mydemoserver',
        'PASSWORD': 'yourpassword',
        'HOST': 'mydemoserver.mysql.database.azure.com',
        'PORT': '3306',
        'OPTIONS': {
            'ssl': {'ssl-ca': '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'}
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о строках подключения см. в [соответствующей статье](howto-connection-string.md).

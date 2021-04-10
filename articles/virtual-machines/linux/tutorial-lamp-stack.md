---
title: Учебник. Развертывание LAMP на виртуальной машине Linux в Azure
description: В этом руководство описано, как установить стек LAMP на виртуальной машине Linux в Azure.
services: virtual-machines
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.collection: linux
ms.assetid: 6c12603a-e391-4d3e-acce-442dd7ebb2fe
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 01/30/2019
ms.author: cynthn
ms.openlocfilehash: 3813931f47c110abcfb595065c1415ca9ed84c9d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102564719"
---
# <a name="tutorial-install-a-lamp-web-server-on-a-linux-virtual-machine-in-azure"></a>Руководство по Установка веб-сервера LAMP на виртуальной машине Linux в Azure

Эта статья содержит указания по развертыванию веб-сервера Apache, MySQL и PHP (стека LAMP) на виртуальной машине Ubuntu в Azure. Если необходимо оценить работу сервера LAMP в действии, вы можете установить и настроить сайт WordPress. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * создание виртуальной машины Ubuntu (L в названии стека LAMP);
> * Открытие порта 80 для веб-трафика
> * Установка Apache, PHP и MySQL
> * Проверка установки и настройки
> * установка WordPress на сервере LAMP.

Эта настройка выполняется для быстрых тестов или подтверждения концепции. Дополнительные сведения о стеке LAMP, включая рекомендации для рабочей среды, см. в [документации Ubuntu](https://help.ubuntu.com/community/ApacheMySQLPHP).

При работе с этим руководством используется интерфейс командной строки (CLI) в [Azure Cloud Shell](../../cloud-shell/overview.md), который всегда обновлен до последней версии. Чтобы открыть Cloud Shell, выберите **Попробовать** в верхнем углу любого блока кода.

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.0.30 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0]( /cli/azure/install-azure-cli).

[!INCLUDE [virtual-machines-linux-tutorial-stack-intro.md](../../../includes/virtual-machines-linux-tutorial-stack-intro.md)]

## <a name="install-apache-mysql-and-php"></a>Установка Apache, PHP и MySQL

Чтобы обновить источники пакетов Ubuntu и установить Apache, PHP и MySQL, выполните указанную ниже команду. Обратите внимание на знак курсора (^) в конце команды, который является частью имени пакета `lamp-server^`. 


```bash
sudo apt update && sudo apt install lamp-server^
```

Отобразится запрос на установку пакетов и других зависимостей. Это позволит установить минимальный набор расширений PHP, необходимый для использования PHP с MySQL.  

## <a name="verify-installation-and-configuration"></a>Проверка установки и настройки


### <a name="verify-apache"></a>Проверка Apache

Проверьте версию Apache, выполнив следующую команду:
```bash
apache2 -v
```

Установив Apache и открыв порт 80 для виртуальной машины, вы можете получить доступ к веб-серверу через Интернет. Чтобы просмотреть страницу по умолчанию Apache2 Ubuntu, откройте веб-браузер и введите общедоступный IP-адрес виртуальной машины. Укажите общедоступный IP-адрес, используемый для подключения по протоколу SSH к виртуальной машине.

![Страница Apache по умолчанию][3]


### <a name="verify-and-secure-mysql"></a>Проверка и обеспечение защиты MySQL

Узнайте версию MySQL, выполнив указанную ниже команду. Обратите внимание, что параметр `V` указан с заглавной буквы.

```bash
mysql -V
```

Чтобы обеспечить безопасную установку MySQL и указать пароль привилегированного пользователя, выполните скрипт `mysql_secure_installation`. 

```bash
sudo mysql_secure_installation
```

При необходимости можно настроить подключаемый модуль проверки пароля (рекомендуется). Затем укажите пароль привилегированного пользователя MySQL и настройте остальные параметры безопасности для вашей среды. Мы рекомендуем ответить "Да" на все вопросы.

Чтобы опробовать функции MySQL (создать базу данных MySQL, добавить пользователей или изменить параметры конфигурации), войдите в MySQL. Этот шаг не требуется для изучения данного руководства.

```bash
sudo mysql -u root -p
```

По окончании выйдите из командной строки MySQL, введя `\q`.

### <a name="verify-php"></a>Проверка PHP

Узнайте версию PHP, выполнив следующую команду:

```bash
php -v
```

Для дальнейшего тестирования создайте страницу кратких сведений о PHP для просмотра в браузере. Следующая команда создает страницу сведений о PHP:

```bash
sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
```

Теперь можно просмотреть созданную страницу сведений о PHP. Откройте веб-браузер и перейдите по адресу `http://yourPublicIPAddress/info.php`. Замените общедоступный IP-адрес своей виртуальной машины. Она должна выглядеть, как показано ниже.

![Страница сведений о PHP][2]

[!INCLUDE [virtual-machines-linux-tutorial-wordpress.md](../../../includes/virtual-machines-linux-tutorial-wordpress.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как развернуть сервер LAMP в Azure. Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины Ubuntu.
> * Открытие порта 80 для веб-трафика
> * Установка Apache, PHP и MySQL
> * Проверка установки и настройки
> * установка WordPress на сервере LAMP.

Перейдите к следующему руководству, чтобы узнать, как защитить веб-серверы с помощью TLS/SSL-сертификатов.

> [!div class="nextstepaction"]
> [Защита веб-сервера с помощью TLS](tutorial-secure-web-server.md)

[2]: ./media/tutorial-lamp-stack/phpsuccesspage.png
[3]: ./media/tutorial-lamp-stack/apachesuccesspage.png

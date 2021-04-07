---
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: 76a080d40721fa78ad703f77f6dbe7a3363ab77e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "68857477"
---
## <a name="install-wordpress"></a>Установка WordPress

Чтобы проверить работу стека, установите пример приложения. Например, выполните приведенные ниже шаги, чтобы установить платформу [WordPress](https://wordpress.org/) с открытым кодом для создания веб-сайтов и блогов. Для других рабочих нагрузок потребуется установка платформ [Drupal](http://www.drupal.org) и [Moodle](https://moodle.org/). 

Эта конфигурация WordPress предназначена только для подтверждения концепции. Чтобы установить последнюю версию WordPress в рабочей среде с рекомендуемыми параметрами безопасности, ознакомьтесь с [документацией по WordPress](https://codex.wordpress.org/Main_Page). 



### <a name="install-the-wordpress-package"></a>Установка пакета WordPress

Выполните следующую команду:

```bash
sudo apt install wordpress
```

### <a name="configure-wordpress"></a>Настройка WordPress

Настройте WordPress, чтобы использовать MySQL и PHP.

В рабочей папке создайте текстовый файл `wordpress.sql`, чтобы настроить базу данных MySQL для WordPress. 

```bash
sudo sensible-editor wordpress.sql
```

Добавьте следующие команды, заменив пароль базы данных на *свой_пароль*. Другие значения оставьте без изменений. Если вы ранее настроили политику безопасности MySQL для проверки надежности паролей, убедитесь, что пароль отвечает этим требованиям надежности. Сохраните файл.

```sql
CREATE DATABASE wordpress;
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
ON wordpress.*
TO wordpress@localhost
IDENTIFIED BY 'yourPassword';
```

Выполните следующую команду для создания базы данных.

```bash
cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf
```

Так как файл `wordpress.sql` содержит учетные данные базы данных, удалите его после использования.

```bash
sudo rm wordpress.sql
```

Чтобы настроить PHP, откройте текстовый редактор на свой выбор и создайте файл `/etc/wordpress/config-localhost.php`, выполнив следующую команду.

```bash
sudo sensible-editor /etc/wordpress/config-localhost.php
```
Скопируйте в этот файл следующие строки, заменив пароль базы данных WordPress на *свой_пароль*. Другие значения оставьте без изменений. Затем сохраните файл.

```php
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'yourPassword');
define('DB_HOST', 'localhost');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```


Переместите файлы установки WordPress в корневой каталог документов на веб-сервере.

```bash
sudo ln -s /usr/share/wordpress /var/www/html/wordpress

sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
```

Теперь можно завершить настройку WordPress и выполнить публикацию на платформе. Откройте веб-браузер и перейдите по адресу `http://yourPublicIPAddress/wordpress`. Замените общедоступный IP-адрес своей виртуальной машины. Она должна выглядеть, как показано ниже.

![Страница установки WordPress](./media/virtual-machines-linux-tutorial-wordpress/wordpressstartpage.png)

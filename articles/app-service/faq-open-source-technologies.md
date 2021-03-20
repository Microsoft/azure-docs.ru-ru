---
title: Вопросы и ответы по технологиям с открытым кодом
description: Получите ответы на часто задаваемые вопросы о технологиях с открытым кодом в службе приложений Azure.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 10/31/2018
ms.author: genli
ms.custom: seodec18, devx-track-python
ms.openlocfilehash: 36dfbf0fda060a8f273fee64098d6234b575088c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97831845"
---
# <a name="open-source-technologies-faqs-for-web-apps-in-azure"></a>Часто задаваемые вопросы о технологиях с открытым кодом в веб-приложениях Azure

В этой статье содержатся ответы на часто задаваемые вопросы о проблемах, связанных с технологиями с открытым кодом, в [веб-приложениях службы приложений Azure](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="how-do-i-turn-on-php-logging-to-troubleshoot-php-issues"></a>Как включить ведение журнала PHP, чтобы устранить неполадки с приложением PHP?

Чтобы включить ведение журнала PHP, сделайте следующее:

1. Войдите на **веб-сайт KUDU** ( `https://*yourwebsitename*.scm.azurewebsites.net` ).
2. В верхнем меню выберите **консоль отладки**  >  **cmd**.
3. Выберите папку **Site**.
4. Выберите папку **wwwroot**.
5. Щелкните **+** значок, а затем выберите **создать файл**.
6. Задайте файлу имя **.user.ini**.
7. Щелкните значок карандаша рядом с файлом **.user.ini**.
8. Добавьте в файл следующий код: `log_errors=on`.
9. Щелкните **Сохранить**.
10. Щелкните значок карандаша рядом с файлом **wp-config.php**.
11. Вместо текста в файле добавьте следующий код:
    ```php
    //Enable WP_DEBUG modedefine('WP_DEBUG', true);//Enable debug logging to /wp-content/debug.logdefine('WP_DEBUG_LOG', true);
    //Suppress errors and warnings to screendefine('WP_DEBUG_DISPLAY', false);//Suppress PHP errors to screenini_set('display_errors', 0);
    ```
12. На портале Azure в меню веб-приложения перезапустите свое веб-приложение.

Дополнительные сведения см. в записи блога [Enable WordPress error logs](/archive/blogs/azureossds/logging-php-errors-in-wordpress-2) (Включение журналов ошибок WordPress).

## <a name="how-do-i-log-python-application-errors-in-apps-that-are-hosted-in-app-service"></a>Как включить ведение журнала ошибок в приложениях Python, размещенных в службе приложений?
[!INCLUDE [web-sites-python-troubleshooting-wsgi-error-log](../../includes/web-sites-python-troubleshooting-wsgi-error-log.md)]

## <a name="how-do-i-change-the-version-of-the-nodejs-application-that-is-hosted-in-app-service"></a>Как изменить версию приложения Node.js, размещенного в службе приложений?

Версию приложения Node.js можно изменить одним из следующих методов:

* На портале Azure в меню **Параметры приложения**.
  1. Откройте портал Azure и перейдите к своему веб-приложению.
  2. В колонке **Параметры** щелкните **Параметры приложения**.
  3. В разделе **Параметры приложения** добавьте WEBSITE_NODE_DEFAULT_VERSION в качестве ключа и укажите необходимую версию Node.js в качестве значения.
  4. Перейдите в **консоль KUDU** ( `https://*yourwebsitename*.scm.azurewebsites.net` ).
  5. Чтобы проверить версию Node.js, введите следующую команду:  
     ```
     node -v
     ```
* Измените файл iisnode.yml. Если изменить версию Node.js в файле iisnode.yml, вы зададите только среду выполнения, используемую IISNode. Командная строка Kudu и остальные компоненты по-прежнему будут использовать версию Node.js, заданную в разделе **Параметры приложения** на портале Azure.

  Чтобы вручную задать файл iisnode.yml, создайте файл iisnode.yml в корневой папке приложения. Добавьте в этот файл следующую строку:
  ```yml
  nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
  ```
   
* Задайте файл iisnode.yml с помощью файла package.json во время развертывания системы управления версиями.
  Процесс развертывания системы управления версиями Azure состоит из следующих этапов:
  1. Перемещение содержимого в веб-приложение Azure.
  2. Создание скрипта развертывания по умолчанию, если его нет (файл deploy.cmd) в корневой папке веб-приложения.
  3. Выполнение скрипта развертывания, который создает файл iisnode.yml, если вы указали версию Node.js в файле package.json в строке `"engines": {"node": "5.9.1","npm": "3.7.3"}`.
  4. Файл iisnode.yml содержит следующую строку кода:
      ```yml
      nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
      ```

## <a name="i-see-the-message-error-establishing-a-database-connection-in-my-wordpress-app-thats-hosted-in-app-service-how-do-i-troubleshoot-this"></a>В приложении WordPress, размещенном в службе приложений, отображается сообщение Error establishing a database connection (Ошибка при установке подключения к базе данных). Как устранить эту проблему?

При появлении этой ошибки в приложении Azure WordPress необходимо включить файлы php_errors.log и debug.log. Для этого выполните действия, описанные в записи блога [Enable WordPress error logs](/archive/blogs/azureossds/logging-php-errors-in-wordpress-2) (Включение журналов ошибок WordPress).

После включения журналов воспроизведите ошибку и проверьте журналы:
```
[09-Oct-2015 00:03:13 UTC] PHP Warning: mysqli_real_connect(): (HY000/1226): User ‘abcdefghijk79' has exceeded the ‘max_user_connections’ resource (current value: 4) in D:\home\site\wwwroot\wp-includes\wp-db.php on line 1454
```

Если в файле debug.log или php_errors.log отображается приведенное выше сообщение об ошибке, это означает, что приложение превысило количество доступных подключений. При использовании базы данных ClearDB проверьте доступное количество подключений для вашего [плана обслуживания](https://www.cleardb.com/pricing.view).

## <a name="how-do-i-debug-a-nodejs-app-thats-hosted-in-app-service"></a>Как отлаживать приложение Node.js, размещенное в службе приложений?

1.  Перейдите в **консоль KUDU** ( `https://*yourwebsitename*.scm.azurewebsites.net/DebugConsole` ).
2.  Перейдите к папке журналов приложений (D:\home\LogFiles\Application).
3.  Проверьте содержимое файла logging_errors.txt.

## <a name="how-do-i-install-native-python-modules-in-an-app-service-web-app-or-api-app"></a>Как установить собственные модули Python в веб-приложении службы приложений или приложении API?

Некоторые пакеты невозможно установить с помощью pip в Azure. Эти пакеты могут быть недоступны в индексе пакетов Python, или требуется компилятор (компилятор недоступен на компьютере, на котором выполняется веб-приложение в службе приложений). Сведения об установке собственных модулей в веб-приложениях службы приложений или приложениях API см. в [этой записи блога](/archive/blogs/azureossds/install-native-python-modules-on-azure-web-apps-api-apps).

## <a name="how-do-i-deploy-a-django-app-to-app-service-by-using-git-and-the-new-version-of-python"></a>Как развернуть приложение Django в службу приложений с помощью Git и новой версии Python?

Сведения об установке Django см. в [этой записи блога](/archive/blogs/azureossds/deploying-django-app-to-azure-app-services-using-git-and-new-version-of-python).

## <a name="where-are-the-tomcat-log-files-located"></a>Где находятся файлы журналов Tomcat?

Для развертываний Microsoft Azure Marketplace и пользовательских развертываний:

* Расположение папки: D:\home\site\wwwroot\bin\apache-tomcat-8.0.33\logs.
* Необходимые файлы:
    * catalina.*ГГГГ-ММ-ДД*.log
    * host-manager.*ГГГГ-ММ-ДД*.log
    * localhost.*ГГГГ-ММ-ДД*.log
    * manager.*ГГГГ-ММ-ДД*.log
    * site_access_log.*ГГГГ-ММ-ДД*.log


Для развертываний из меню **Параметры приложения** портала:

* Расположение папки: D:\home\LogFiles.
* Необходимые файлы:
    * catalina.*ГГГГ-ММ-ДД*.log
    * host-manager.*ГГГГ-ММ-ДД*.log
    * localhost.*ГГГГ-ММ-ДД*.log
    * manager.*ГГГГ-ММ-ДД*.log
    * site_access_log.*ГГГГ-ММ-ДД*.log

## <a name="how-do-i-troubleshoot-jdbc-driver-connection-errors"></a>Как устранить ошибки подключения драйвера JDBC Driver?

В журналах Tomcat может появиться следующее сообщение:

```
The web application[ROOT] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak,the JDBC Driver has been forcibly unregistered
```

Чтобы устранить эту ошибку, сделайте следующее:

1. Удалите файл sqljdbc*.jar из папки app/lib.
2. При использовании пользовательского веб-сервера Tomcat или веб-сервера Tomcat из Microsoft Azure Marketplace скопируйте этот JAR-файл в папку lib.
3. Если вы включаете Java из портал Azure (выберите **Java 1,8**  >  **Tomcat Server**), скопируйте JAR-файл sqljdbc. * в папку, которая находится параллельно с приложением. Затем добавьте в файл web.config следующий параметр classpath:

    ```xml
    <httpPlatform>
    <environmentVariables>
    <environmentVariablename ="JAVA_OPTS" value=" -Djava.net.preferIPv4Stack=true
    -Xms128M -classpath %CLASSPATH%;[Path to the sqljdbc*.jarfile]" />
    </environmentVariables>
    </httpPlatform>
    ```

## <a name="why-do-i-see-errors-when-i-attempt-to-copy-live-log-files"></a>Почему возникают ошибки при попытке скопировать динамические файлы журналов?

При попытке скопировать динамические файлы журналов приложения Java (например, Tomcat) может отображаться следующая ошибка FTP:

```
Error transferring file [filename] Copying files from remote side failed.
    
The process cannot access the file because it is being used by another process.
```

Сообщение об ошибке может отличаться в зависимости от FTP-клиента.

Все приложения Java имеют эту проблему. Только приложения Kudu поддерживают загрузку этого файла во время выполнения приложения.

Чтобы предоставить к этим файлам доступ по протоколу FTP, остановите приложение.

Кроме того, вы можете написать веб-задание, которое будет копировать эти файлы в другой каталог по установленному расписанию. Пример проекта см. [здесь](https://github.com/kamilsykora/CopyLogsJob).

## <a name="where-do-i-find-the-log-files-for-jetty"></a>Где найти файлы журналов для Jetty?

Файл журнала для развертываний Microsoft Azure Marketplace и пользовательских развертываний расположен в папке D:\home\site\wwwroot\bin\jetty-distribution-9.1.2.v20140210\logs. Обратите внимание, что расположение папки зависит от используемой версии Jetty. Например, приведенный выше путь предназначен для Jetty версии 9.1.2. Найдите файл jetty_ *ГГГГ_ММ_ДД*.stderrout.log.

Файл журнала для развертываний из меню "Параметры приложения" портала находится в папке D:\home\LogFiles. Найдите jetty_ *YYYY_MM_DD*. stderr. log

## <a name="can-i-send-email-from-my-azure-web-app"></a>Можно ли отправлять электронные сообщения из веб-приложения Azure?

Служба приложений не имеет встроенной функции отправки электронных сообщений. Альтернативные варианты отправки электронных сообщений из приложения см. в [этом обсуждении Stack Overflow](https://stackoverflow.com/questions/17666161/sending-email-from-azure).

## <a name="why-does-my-wordpress-site-redirect-to-another-url"></a>Почему сайт WordPress перенаправляет на другой URL-адрес?

Если вы недавно перенесли свои ресурсы в Azure, WordPress может перенаправлять на старый URL-адрес домена. Это вызвано конфигурацией базы данных MySQL.

WordPress Buddy+ — это расширение сайта Azure, с помощью которого можно обновить URL-адрес перенаправления непосредственно в базе данных. Дополнительные сведения об использовании WordPress Buddy+ см. в записи блога [WordPress tools and MySQL migration with WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows) (Средства WordPress и перенос базы данных MySQL с помощью WordPress Buddy+).

Кроме того, если вы предпочитаете вручную обновить URL-адрес перенаправления с помощью SQL-запросов или PHPMyAdmin, см. запись блога [WordPress: Redirecting to wrong URL](/archive/blogs/azureossds/wordpress-redirecting-to-wrong-url) (WordPress: перенаправление на неправильный URL-адрес).

## <a name="how-do-i-change-my-wordpress-sign-in-password"></a>Как изменить пароль для входа в WordPress?

Если вы забыли пароль для входа в WordPress, его можно обновить с помощью WordPress Buddy+. Чтобы сбросить пароль, установите расширение сайта Azure WordPress Buddy+, а затем выполните действия, описанные в записи блога [WordPress tools and MySQL migration with WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows) (Средства WordPress и перенос базы данных MySQL с помощью WordPress Buddy+).

## <a name="i-cant-sign-in-to-wordpress-how-do-i-resolve-this"></a>Не удается войти в WordPress. Как решить эту проблему?

Если вы не можете войти в WordPress после установки подключаемого модуля, возможно, вы установили не тот модуль. WordPress Buddy+ — это расширение сайта Azure, с помощью которого можно отключить подключаемые модули в WordPress. Дополнительные сведения см. в записи блога [WordPress tools and MySQL migration with WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows) (Средства WordPress и перенос базы данных MySQL с помощью WordPress Buddy+).

## <a name="how-do-i-migrate-my-wordpress-database"></a>Как перенести базу данных WordPress?

Базу данных MySQL, подключенную к веб-сайту WordPress, можно перенести несколькими способами:

* с помощью [командной строки или PHPMyAdmin](/archive/blogs/azureossds/migrating-data-between-mysql-databases-using-kudu-console-azure-app-service) (для разработчиков);
* с помощью [WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows) (остальные пользователи).

## <a name="how-do-i-help-make-wordpress-more-secure"></a>Как повысить уровень защиты WordPress?

Рекомендации по обеспечению безопасности WordPress см. в [этой записи блога](/archive/blogs/azureossds/best-practices-for-wordpress-security-on-azure).

## <a name="i-am-trying-to-use-phpmyadmin-and-i-see-the-message-access-denied-how-do-i-resolve-this"></a>При использовании PHPMyAdmin отображается ошибка "Доступ запрещен". Как решить эту проблему?

Эта проблема может возникать, если компонент MySQL приложения еще не запущен в этом экземпляре службы приложений. Чтобы устранить эту проблему, попробуйте получить доступ к веб-сайту. При этом запустятся необходимые процессы, в том числе процессы MySQL в приложении. Чтобы проверить, выполняется ли MySQL в приложении, найдите процесс mysqld.exe в списке процессов в обозревателе процессов.

Если MySQL выполняется, попробуйте запустить PHPMyAdmin.

## <a name="i-get-an-http-403-error-when-i-try-to-import-or-export-my-mysql-in-app-database-by-using-phpmyadmin-how-do-i-resolve-this"></a>При попытке импорта или экспорта базы данных MySQL в приложении с помощью PHPMyadmin произошла ошибка HTTP 403. Как решить эту проблему?

Это известная ошибка в старой версии браузера Chrome. Чтобы устранить эту проблему, обновите браузер Chrome. Кроме того, попробуйте использовать другой браузер, например Internet Explorer или Microsoft Edge, где эта проблема не возникает.
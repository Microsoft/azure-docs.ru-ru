---
title: Настройка SSL — база данных Azure для MySQL
description: Инструкции по настройке базы данных Azure для MySQL и связанных приложений для правильного использования SSL-соединений.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 07/08/2020
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 1c3e5a44e01f3fa43b82644103066f5a03684ad2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94591586"
---
# <a name="configure-ssl-connectivity-in-your-application-to-securely-connect-to-azure-database-for-mysql"></a>Настройка SSL-подключений в приложении для безопасного подключения к базе данных Azure для MySQL

База данных Azure для MySQL поддерживает подключение сервера базы данных Azure для MySQL к клиентским приложениям с помощью протокола SSL (Secure Sockets Layer). Применение SSL-соединений между сервером базы данных и клиентскими приложениями обеспечивает защиту от атак "злоумышленник в середине" за счет шифрования потока данных между сервером и приложением.

## <a name="step-1-obtain-ssl-certificate"></a>Шаг 1. Получение SSL-сертификата

Скачайте сертификат, необходимый для обмена данными по протоколу SSL с сервером базы данных Azure для MySQL [https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) , и сохраните файл сертификата на локальном диске (например, в этом руководстве используется используем c:\ssl).
**Для браузеров Microsoft Internet Explorer и Microsoft Edge:** по завершении скачивания переименуйте сертификат в BaltimoreCyberTrustRoot.crt.pem.

>[!NOTE]
> Основываясь на отзывах клиентов, мы расширили устаревший корневой сертификат для нашего существующего корневого ЦС Baltimore до 15 февраля 2021 (02/15/2021).

> [!IMPORTANT] 
> Срок действия корневого сертификата SSL истекает 15 февраля 2021 (02/15/2021). Обновите приложение, чтобы использовать [новый сертификат](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem). Дополнительные сведения см. в разделе [запланированные обновления сертификатов](concepts-certificate-rotation.md) .

См. следующие ссылки на сертификаты для серверов в облаках независимых: [Azure для государственных организаций](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem), [Azure для Китая](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem)и Azure для [Германии](https://www.d-trust.net/cgi-bin/D-TRUST_Root_Class_3_CA_2_2009.crt).

## <a name="step-2-bind-ssl"></a>Шаг 2. Привязка SSL

Сведения о строках подключения для конкретных языков программирования см. в приведенном ниже [образце кода](howto-configure-ssl.md#sample-code) .

### <a name="connecting-to-server-using-mysql-workbench-over-ssl"></a>Подключение к серверу с помощью MySQL Workbench по протоколу SSL

Настройте MySQL Workbench, чтобы безопасно подключаться по протоколу SSL.

1. В диалоговом окне Setup New Connection (Настройка нового подключения) откройте вкладку **SSL**.

1. Обновите поле " **использовать SSL** " для "обязательно".

1. Введите расположение файла **BaltimoreCyberTrustRoot.crt.pem** в поле **SSL CA File:** (Файл центра сертификации SSL-сертификата).

   :::image type="content" source="./media/howto-configure-ssl/mysql-workbench-ssl.png" alt-text="Сохранить конфигурацию SSL":::

Для существующих подключений можно привязать SSL, щелкнув правой кнопкой мыши значок подключения и выбрав пункт изменить. Откройте вкладку **SSL** и привяжите файл сертификата.

### <a name="connecting-to-server-using-the-mysql-cli-over-ssl"></a>Подключение к серверу с помощью интерфейса командной строки MySQL по протоколу SSL

Кроме того, можно привязать SSL-сертификат при помощи интерфейса командной строки MySQL, выполнив следующие команды.

```bash
mysql.exe -h mydemoserver.mysql.database.azure.com -u Username@mydemoserver -p --ssl-mode=REQUIRED --ssl-ca=c:\ssl\BaltimoreCyberTrustRoot.crt.pem
```

> [!NOTE]
> При использовании интерфейса командной строки MySQL для Windows может появиться ошибка `SSL connection error: Certificate signature check failed`. В этом случае замените параметр `--ssl-mode=REQUIRED --ssl-ca={filepath}` на `--ssl`.

## <a name="step-3--enforcing-ssl-connections-in-azure"></a>Шаг 3. Применение SSL-соединений в Azure

### <a name="using-the-azure-portal"></a>Использование портала Azure

С помощью портала Azure перейдите к серверу базы данных Azure для MySQL и щелкните **Безопасность подключения**. Воспользуйтесь выключателем для включения или отключения параметра **Enforce SSL connection** (Применять SSL-соединение), а затем щелкните **Сохранить**. Для повышения безопасности корпорация Майкрософт рекомендует всегда включать параметр **Enforce SSL connection** (Применять SSL-соединение).

:::image type="content" source="./media/howto-configure-ssl/enable-ssl.png" alt-text="Снимок экрана портал Azure для принудительного применения SSL-подключений в базе данных Azure для MySQL":::

### <a name="using-azure-cli"></a>Использование Azure CLI

Вы также можете включить или отключить параметр **ssl-enforcement** с помощью Azure CLI, используя значения Enabled и Disabled соответственно.

```azurecli-interactive
az mysql server update --resource-group myresource --name mydemoserver --ssl-enforcement Enabled
```

## <a name="step-4-verify-the-ssl-connection"></a>Шаг 4. Проверка SSL-соединения

Выполните команду mysql **status**, чтобы проверить наличие подключения к серверу MySQL с помощью протокола SSL.

```dos
mysql> status
```

Убедитесь, что соединение зашифровано, просмотрев выходные данные, в которых должно отображаться следующее: **SSL: используемый шифр — AES256 SHA**.

## <a name="sample-code"></a>Пример кода

Чтобы установить безопасное подключение приложения к Базе данных Azure для MySQL по протоколу SSL, изучите приведенные ниже примеры кода.

См. список [совместимых драйверов](concepts-compatibility.md) , поддерживаемых службой "база данных Azure для MySQL".

### <a name="php"></a>PHP

```php
$conn = mysqli_init();
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/BaltimoreCyberTrustRoot.crt.pem", NULL, NULL);
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin@mydemoserver', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```

### <a name="php-using-pdo"></a>PHP (с использованием PDO)

```phppdo
$options = array(
    PDO::MYSQL_ATTR_SSL_CA => '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'
);
$db = new PDO('mysql:host=mydemoserver.mysql.database.azure.com;port=3306;dbname=databasename', 'username@mydemoserver', 'yourpassword', $options);
```

### <a name="python-mysqlconnector-python"></a>Python (MySQLConnector Python)

```python
try:
    conn = mysql.connector.connect(user='myadmin@mydemoserver',
                                   password='yourpassword',
                                   database='quickstartdb',
                                   host='mydemoserver.mysql.database.azure.com',
                                   ssl_ca='/var/www/html/BaltimoreCyberTrustRoot.crt.pem')
except mysql.connector.Error as err:
    print(err)
```

### <a name="python-pymysql"></a>Python (PyMySQL)

```python
conn = pymysql.connect(user='myadmin@mydemoserver',
                       password='yourpassword',
                       database='quickstartdb',
                       host='mydemoserver.mysql.database.azure.com',
                       ssl={'ca': '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'})
```

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
            'ssl': {'ca': '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'}
        }
    }
}
```

### <a name="ruby"></a>Ruby

```ruby
client = Mysql2::Client.new(
        :host     => 'mydemoserver.mysql.database.azure.com',
        :username => 'myadmin@mydemoserver',
        :password => 'yourpassword',
        :database => 'quickstartdb',
        :sslca => '/var/www/html/BaltimoreCyberTrustRoot.crt.pem'
    )
```

### <a name="golang"></a>Golang

```go
rootCertPool := x509.NewCertPool()
pem, _ := ioutil.ReadFile("/var/www/html/BaltimoreCyberTrustRoot.crt.pem")
if ok := rootCertPool.AppendCertsFromPEM(pem); !ok {
    log.Fatal("Failed to append PEM.")
}
mysql.RegisterTLSConfig("custom", &tls.Config{RootCAs: rootCertPool})
var connectionString string
connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true&tls=custom",'myadmin@mydemoserver' , 'yourpassword', 'mydemoserver.mysql.database.azure.com', 'quickstartdb')
db, _ := sql.Open("mysql", connectionString)
```

### <a name="java-mysql-connector-for-java"></a>Java (соединитель MySQL для Java)

```java
# generate truststore and keystore in code
String importCert = " -import "+
    " -alias mysqlServerCACert "+
    " -file " + ssl_ca +
    " -keystore truststore "+
    " -trustcacerts " +
    " -storepass password -noprompt ";
String genKey = " -genkey -keyalg rsa " +
    " -alias mysqlClientCertificate -keystore keystore " +
    " -storepass password123 -keypass password " +
    " -dname CN=MS ";
sun.security.tools.keytool.Main.main(importCert.trim().split("\\s+"));
sun.security.tools.keytool.Main.main(genKey.trim().split("\\s+"));

# use the generated keystore and truststore
System.setProperty("javax.net.ssl.keyStore","path_to_keystore_file");
System.setProperty("javax.net.ssl.keyStorePassword","password");
System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file");
System.setProperty("javax.net.ssl.trustStorePassword","password");

url = String.format("jdbc:mysql://%s/%s?serverTimezone=UTC&useSSL=true", 'mydemoserver.mysql.database.azure.com', 'quickstartdb');
properties.setProperty("user", 'myadmin@mydemoserver');
properties.setProperty("password", 'yourpassword');
conn = DriverManager.getConnection(url, properties);
```

### <a name="java-mariadb-connector-for-java"></a>Java (соединитель MariaDB для Java)

```java
# generate truststore and keystore in code
String importCert = " -import "+
    " -alias mysqlServerCACert "+
    " -file " + ssl_ca +
    " -keystore truststore "+
    " -trustcacerts " +
    " -storepass password -noprompt ";
String genKey = " -genkey -keyalg rsa " +
    " -alias mysqlClientCertificate -keystore keystore " +
    " -storepass password123 -keypass password " +
    " -dname CN=MS ";
sun.security.tools.keytool.Main.main(importCert.trim().split("\\s+"));
sun.security.tools.keytool.Main.main(genKey.trim().split("\\s+"));

# use the generated keystore and truststore
System.setProperty("javax.net.ssl.keyStore","path_to_keystore_file");
System.setProperty("javax.net.ssl.keyStorePassword","password");
System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file");
System.setProperty("javax.net.ssl.trustStorePassword","password");

url = String.format("jdbc:mariadb://%s/%s?useSSL=true&trustServerCertificate=true", 'mydemoserver.mysql.database.azure.com', 'quickstartdb');
properties.setProperty("user", 'myadmin@mydemoserver');
properties.setProperty("password", 'yourpassword');
conn = DriverManager.getConnection(url, properties);
```

### <a name="net-mysqlconnector"></a>.NET (MySqlConnector)

```csharp
var builder = new MySqlConnectionStringBuilder
{
    Server = "mydemoserver.mysql.database.azure.com",
    UserID = "myadmin@mydemoserver",
    Password = "yourpassword",
    Database = "quickstartdb",
    SslMode = MySqlSslMode.VerifyCA,
    SslCa = "BaltimoreCyberTrustRoot.crt.pem",
};
using (var connection = new MySqlConnection(builder.ConnectionString))
{
    connection.Open();
}
```

## <a name="next-steps"></a>Дальнейшие действия

Сведения о вариантах подключения приложений см. в статье [Библиотеки подключений для базы данных Azure для MySQL](concepts-connection-libraries.md).

---
title: Смена сертификатов для базы данных Azure для MySQL
description: Узнайте о предстоящих изменениях изменений корневого сертификата, которые повлияют на базу данных Azure для MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 01/18/2021
ms.openlocfilehash: 23fa3e93565066ce4b897bffe63164486efc179e
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102449889"
---
# <a name="understanding-the-changes-in-the-root-ca-change-for-azure-database-for-mysql-single-server"></a>Основные сведения об изменениях в корневом ЦС для одного сервера базы данных Azure для MySQL

Один сервер базы данных Azure для MySQL успешно выполнил изменение корневого сертификата **15 февраля 2021 (02/15/2021)** в рамках стандартных рекомендаций по обслуживанию и обеспечению безопасности. В этой статье приводятся дополнительные сведения об изменениях, затронутых ресурсах и действиях, которые необходимо выполнить, чтобы обеспечить поддержку подключения приложения к серверу базы данных.

> [!NOTE]
> Эта статья относится только к [базе данных Azure для MySQL — только для одного сервера](single-server-overview.md) . Для службы " [база данных Azure для MySQL — гибкий сервер](flexible-server/overview.md)" сертификат, необходимый для взаимодействия по протоколу SSL, — это [DigiCert глобальный корневой ЦС](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) .
> 
> Эта статья содержит ссылки на термин « _Ведомый_» термин, который корпорация Майкрософт больше не использует. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.
>

## <a name="why-root-certificate-update-is-required"></a>Зачем требуется обновление корневого сертификата?

Пользователи базы данных Azure для MySQL могут использовать только предопределенный сертификат для подключения к серверу MySQL, расположенному [здесь](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem). Однако в [браузере центра сертификации (ЦС) на форуме](https://cabforum.org/)   Недавно опубликованные отчеты о нескольких сертификатах, выданных поставщиками ЦС, не соответствуют требованиям.

Согласно требованиям отрасли поставщики ЦС начали отменять сертификаты ЦС для несоответствующих центров сертификации, требовать, чтобы серверы использовали сертификаты, выпущенные соответствующими центрами сертификации, и подписали сертификаты ЦС из соответствующих центров сертификации. Так как база данных Azure для MySQL использовала один из этих сертификатов, которые не соответствуют требованиям, нам нужно было поворачивать сертификат до совместимой версии, чтобы максимально защитить потенциальные угрозы для серверов MySQL.

Новый сертификат размещается и действует начиная с 15 февраля 2021 (02/15/2021). 

## <a name="what-change-was-performed-on-february-15-2021-02152021"></a>Какие изменения выполнялись 15 февраля 2021 (02/15/2021)?

15 февраля 2021 г. [корневой сертификат BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) был заменен соответствующей **версией** того же [корневого сертификата BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) , чтобы гарантировать, что существующим клиентам не нужно изменять что-либо, что не повлияет на подключения к серверу. Во время этого изменения [корневой сертификат BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) **не был заменен** на [DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) , и это изменение откладывается, чтобы предоставить клиентам больше времени на внесение изменений.

## <a name="do-i-need-to-make-any-changes-on-my-client-to-maintain-connectivity"></a>Нужно ли вносить изменения на клиенте для поддержания подключения?

На стороне клиента не требуется никаких изменений. Если следовать приведенной выше рекомендации, вы все равно сможете продолжать подключение, пока **сертификат BaltimoreCyberTrustRoot не будет удален** из Объединенного сертификата ЦС. **Не рекомендуется удалять BaltimoreCyberTrustRoot из Объединенного сертификата ЦС, пока не будет продолжено дополнительное уведомление для поддержания подключения.**

### <a name="previous-recommendation"></a>Предыдущая рекомендация

Чтобы избежать прерывания доступности приложения из-за неожиданного отзыва сертификатов или обновления отозванного сертификата, выполните следующие действия. Идея состоит в создании нового *PEM* -файла, который объединяет текущий сертификат и новый, и при проверке SSL-сертификата будет использоваться одно из допустимых значений. Ознакомьтесь с приведенными ниже шагами.

* Скачайте корневой центр сертификации BaltimoreCyberTrustRoot & DigiCertGlobalRootG2 по следующим ссылкам:

  * [https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem)
  * [https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem)

* Создайте объединенное хранилище сертификатов ЦС с сертификатами **BaltimoreCyberTrustRoot** и **DigiCertGlobalRootG2** .

  * Для пользователей Java (соединителя MySQL/J) выполните:

    ```console
    keytool -importcert -alias MySQLServerCACert -file D:\BaltimoreCyberTrustRoot.crt.pem -keystore truststore -storepass password -noprompt
    ```

    ```console
    keytool -importcert -alias MySQLServerCACert2 -file D:\DigiCertGlobalRootG2.crt.pem -keystore truststore -storepass password -noprompt
    ```

    Затем замените исходный файл хранилища ключей новым созданным файлом.

    * System. setProperty ("жавакс. NET. SSL. trustStore", "path_to_truststore_file");
    * System. setProperty ("жавакс. NET. SSL. trustStorePassword", "пароль");

  * Для пользователей .NET (соединители MySQL или NET, MySQLConnector) убедитесь, что **BaltimoreCyberTrustRoot** и **DigiCertGlobalRootG2** существуют в хранилище сертификатов Windows — доверенные корневые центры сертификации. Если какие либо сертификаты не существуют, импортируйте недостающий сертификат.

    :::image type="content" source="media/overview/netconnecter-cert.png" alt-text="Схема сертификата .NET базы данных Azure для MySQL":::

  * Для пользователей .NET в Linux, использующих SSL_CERT_DIR, убедитесь, что **BaltimoreCyberTrustRoot** и **DigiCertGlobalRootG2** существуют в каталоге, указанном SSL_CERT_DIR. Если какие либо сертификаты не существуют, создайте недостающий файл сертификата.

  * Для других пользователей (клиент MySQL, MySQL Workbench/C/C++/го/Писон/Руби/ФП/нодежс/Перл/Свифт) можно объединить два файла сертификатов ЦС в следующий формат:

      ```
      -----BEGIN CERTIFICATE-----
      (Root CA1: BaltimoreCyberTrustRoot.crt.pem)
      -----END CERTIFICATE-----
      -----BEGIN CERTIFICATE-----
      (Root CA2: DigiCertGlobalRootG2.crt.pem)
      -----END CERTIFICATE-----
      ```

* Замените исходный файл PEM корневого центра сертификации на объединенный файл корневого ЦС и перезапустите приложение или клиент.
* В будущем после развертывания нового сертификата на стороне сервера можно изменить PEM-файл CA на DigiCertGlobalRootG2. CRT. PEM.

> [!NOTE]
> Не удаляйте и не изменяйте **сертификат Baltimore** , пока не будет внесено изменение сертификата. После внесения изменений будет отправлено сообщение, после чего его можно безопасно удалить с помощью сертификата Baltimore. 

## <a name="why-was-baltimorecybertrustroot-certificate-not-replaced-to-digicertglobalrootg2-during-this-change-on-february-15-2021"></a>Почему BaltimoreCyberTrustRoot сертификат не был заменен на DigiCertGlobalRootG2 в течение этого изменения 15 февраля 2021?

Мы оценили готовность клиентов к этому изменению, и многие клиенты искали дополнительное время для управления этим изменением. В интересах предоставления клиентам большего времени на подготовку, мы решили отложить изменение сертификата на DigiCertGlobalRootG2 по крайней мере за год, предоставляя достаточно времени для клиентов и конечных пользователей. 

Наши рекомендации для пользователей — используйте описанные выше шаги, чтобы создать объединенный сертификат и подключиться к серверу, но не удаляйте сертификат BaltimoreCyberTrustRoot, пока не будет отправлено сообщение для удаления подключения. 

## <a name="what-if-we-removed-the-baltimorecybertrustroot-certificate"></a>Что делать, если мы удалили сертификат BaltimoreCyberTrustRoot?

При подключении к серверу базы данных Azure для MySQL вы начнете работу с ошибками подключения. Для поддержания подключения необходимо снова [настроить SSL](howto-configure-ssl.md) с сертификатом [BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) .


## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="1-if-im-not-using-ssltls-do-i-still-need-to-update-the-root-ca"></a>1. Если я не использую SSL/TLS, мне все равно нужно обновить корневой ЦС?

  Если вы не используете SSL/TLS, никаких действий не требуется.

### <a name="2-if-im-using-ssltls-do-i-need-to-restart-my-database-server-to-update-the-root-ca"></a>2. Если я использую SSL/TLS, нужно ли перезапустить сервер базы данных, чтобы обновить корневой ЦС?

Нет, не нужно перезагружать сервер базы данных, чтобы приступить к использованию нового сертификата. Этот корневой сертификат является изменением на стороне клиента, и для входящих клиентских подключений необходимо использовать новый сертификат, чтобы обеспечить возможность подключения к серверу базы данных.

### <a name="3-how-do-i-know-if-im-using-ssltls-with-root-certificate-verification"></a>3. Разделы справки узнать, используется ли SSL/TLS с проверкой корневого сертификата?

Вы можете определить, проверяют ли ваши подключения корневой сертификат, просмотрев строку подключения.

- Если строка подключения включает `sslmode=verify-ca` или `sslmode=verify-identity` , необходимо обновить сертификат.
- Если строка подключения включает `sslmode=disable` , `sslmode=allow` , `sslmode=prefer` или `sslmode=require` , не требуется обновлять сертификаты.
- Если в строке подключения не указано sslmode, обновлять сертификаты не требуется.

Если вы используете клиент, который абстрагирует строку подключения, изучите документацию клиента, чтобы понять, будет ли он проверять сертификаты.

### <a name="4-what-is-the-impact-if-using-app-service-with-azure-database-for-mysql"></a>4. Каковы последствия использования службы приложений с базой данных Azure для MySQL?

Для служб приложений Azure, подключающихся к базе данных Azure для MySQL, существует два возможных сценария и в зависимости от того, как используется SSL с приложением.

* Этот новый сертификат добавлен в службу приложений на уровне платформы. Если вы используете SSL-сертификаты, включенные в приложение в платформе службы приложений, никаких действий не требуется. Это наиболее распространенный сценарий. 
* Если вы явно включаете путь к файлу SSL-сертификата в коде, необходимо скачать новый сертификат и создать объединенный сертификат, как упоминалось выше, и использовать файл сертификата. Хорошим примером этого сценария является использование пользовательских контейнеров в службе приложений в качестве общих в [документации по службе приложений](../app-service/tutorial-multi-container-app.md#configure-database-variables-in-wordpress). Это редкий сценарий, но мы видели, что некоторые пользователи используют это.

### <a name="5-what-is-the-impact-if-using-azure-kubernetes-services-aks-with-azure-database-for-mysql"></a>5. Каковы последствия использования Azure Kubernetes Services (AKS) с базой данных Azure для MySQL?

Если вы пытаетесь подключиться к базе данных Azure для MySQL с помощью Azure Kubernetes Services (AKS), она похожа на доступ из выделенной среды размещения клиентов. Инструкции см. [здесь](../aks/ingress-own-tls.md).

### <a name="6-what-is-the-impact-if-using-azure-data-factory-to-connect-to-azure-database-for-mysql"></a>6. Каково влияние использования фабрики данных Azure для подключения к базе данных Azure для MySQL?

Для соединителя, использующего Azure Integration Runtime, соединитель использует сертификаты из хранилища сертификатов Windows в среде, размещенной в Azure. Эти сертификаты уже совместимы с вновь применяемыми сертификатами и поэтому никаких действий не требуется.

Для соединителя, использующего локально размещенный Integration Runtime, в котором явно указан путь к файлу SSL-сертификата в строке подключения, необходимо скачать [новый сертификат](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) и обновить строку подключения, чтобы она использовалась.

### <a name="7-do-i-need-to-plan-a-database-server-maintenance-downtime-for-this-change"></a>7. нужно ли мне запланировать время простоя обслуживания сервера базы данных для этого изменения?

Нет. Так как изменение здесь происходит только на стороне клиента для подключения к серверу базы данных, время простоя обслуживания для этого изменения не требуется для сервера базы данных.

### <a name="8-if-i-create-a-new-server-after-february-15-2021-02152021-will-i-be-impacted"></a>8. Если я создам новый сервер после 15 февраля 2021 (02/15/2021), будет ли это затронуто?

Для серверов, созданных после 15 февраля 2021 (02/15/2021), вы по-прежнему будете использовать [BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) для подключения приложений с помощью SSL.

### <a name="9-how-often-does-microsoft-update-their-certificates-or-what-is-the-expiry-policy"></a>9. как часто Корпорация Майкрософт обновляет свои сертификаты или какова политика срока действия?

Эти сертификаты, используемые базой данных Azure для MySQL, предоставляются доверенными центрами сертификации (CA). Поэтому поддержка этих сертификатов привязывается к поддержке этих сертификатов центром сертификации. Срок действия сертификата [BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) истекает через 2025, поэтому перед истечением срока действия корпорация Майкрософт должна выполнить изменение сертификата. Кроме того, если в этих стандартных сертификатах возникают непредвиденные ошибки, то для обеспечения безопасности и совместимости службы Корпорация Майкрософт должна будет выполнить смену сертификатов в наиболее ранней степени, как в случае с 15 февраля 2021.

### <a name="10-if-im-using-read-replicas-do-i-need-to-perform-this-update-only-on-source-server-or-the-read-replicas"></a>10. Если я использую для чтения реплики, нужно ли выполнять это обновление только на исходном сервере или репликах чтения?

Так как это обновление является изменением на стороне клиента, если клиент использовался для считывания данных с сервера реплики, необходимо также применить изменения для этих клиентов.

### <a name="11-if-im-using-data-in-replication-do-i-need-to-perform-any-action"></a>11. Если я использую репликацию данных, нужно ли выполнять какие-либо действия?

Если вы используете [репликацию данных](concepts-data-in-replication.md) для подключения к базе данных Azure для MySQL, необходимо учитывать два фактора:

* Если репликация данных выполняется из виртуальной машины (локальной или виртуальной машины Azure) в базу данных Azure для MySQL, необходимо проверить, используется ли SSL для создания реплики. Выполните команду **ОТОБРАЗИТЬ состояние Slave** и проверьте следующий параметр.  

    ```azurecli-interactive
    Master_SSL_Allowed            : Yes
    Master_SSL_CA_File            : ~\azure_mysqlservice.pem
    Master_SSL_CA_Path            :
    Master_SSL_Cert               : ~\azure_mysqlclient_cert.pem
    Master_SSL_Cipher             :
    Master_SSL_Key                : ~\azure_mysqlclient_key.pem
    ```

    Если вы видите, что сертификат предоставляется для CA_file, SSL_Cert и SSL_Key, необходимо обновить файл, добавив [новый сертификат](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) и создав объединенный файл сертификата.

* Если репликация данных выполняется между двумя базами данных Azure для MySQL, необходимо сбросить реплику, выполнив команду **CALL MySQL.az_replication_change_master** и указав новый двойной корневой сертификат в качестве последнего параметра [master_ssl_ca](howto-data-in-replication.md#link-source-and-replica-servers-to-start-data-in-replication)

### <a name="12-do-we-have-server-side-query-to-verify-if-ssl-is-being-used"></a>12. у нас есть запрос на стороне сервера, чтобы проверить, используется ли протокол SSL?

Чтобы проверить, используется ли SSL-соединение для подключения к серверу, обратитесь к [проверке SSL](howto-configure-ssl.md#step-4-verify-the-ssl-connection).

### <a name="13-is-there-an-action-needed-if-i-already-have-the-digicertglobalrootg2-in-my-certificate-file"></a>13. требуется ли действие, если в файле сертификата уже есть DigiCertGlobalRootG2?

Нет. Если файл сертификата уже содержит **DigiCertGlobalRootG2**, никаких действий не требуется.

### <a name="14-what-if-i-have-further-questions"></a>14. что делать, если у меня возникнут дополнительные вопросы?

Если у вас есть вопросы, получите ответы от экспертов сообщества в [Microsoft Q&а](mailto:AzureDatabaseforMySQL@service.microsoft.com). Если у вас есть план поддержки и вам нужна техническая помощь, [свяжитесь с нами](mailto:AzureDatabaseforMySQL@service.microsoft.com).

---
title: Смена сертификатов для одного сервера базы данных Azure для PostgreSQL
description: Узнайте о предстоящих изменениях изменений корневого сертификата, которые повлияют на работу базы данных Azure для PostgreSQL на одном сервере.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 6dc687879eb646b4abd081b40bce292d20ff3186
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102123995"
---
# <a name="understanding-the-changes-in-the-root-ca-change-for-azure-database-for-postgresql-single-server"></a>Основные сведения об изменениях в корневом ЦС для одного сервера базы данных Azure для PostgreSQL

Служба "база данных Azure для PostgreSQL" успешно завершила изменение корневого сертификата **15 февраля 2021 (02/15/2021)** в рамках стандартных рекомендаций по обслуживанию и обеспечению безопасности. В этой статье приводятся дополнительные сведения об изменениях, затронутых ресурсах и действиях, которые необходимо выполнить, чтобы обеспечить поддержку подключения приложения к серверу базы данных.

## <a name="why-root-certificate-update-is-required"></a>Зачем требуется обновление корневого сертификата?

Пользователи базы данных Azure для PostgreSQL могут использовать только предопределенный сертификат для подключения к их серверу PostgreSQL, расположенному [здесь](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem). Однако в [браузере центра сертификации (ЦС) на форуме](https://cabforum.org/)   Недавно опубликованные отчеты о нескольких сертификатах, выданных поставщиками ЦС, не соответствуют требованиям.

Согласно требованиям отрасли поставщики ЦС начали отменять сертификаты ЦС для несоответствующих центров сертификации, требовать, чтобы серверы использовали сертификаты, выпущенные соответствующими центрами сертификации, и подписали сертификаты ЦС из соответствующих центров сертификации. Так как база данных Azure для MySQL использовала один из этих сертификатов, которые не соответствуют требованиям, нам нужно было поворачивать сертификат до совместимой версии, чтобы максимально защитить потенциальные угрозы для серверов MySQL.

Новый сертификат размещается и действует начиная с 15 февраля 2021 (02/15/2021). 

## <a name="what-change-was-performed-on-february-15-2021-02152021"></a>Какие изменения выполнялись 15 февраля 2021 (02/15/2021)?

15 февраля 2021 г. [корневой сертификат BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) был заменен соответствующей **версией** того же [корневого сертификата BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) , чтобы гарантировать, что существующим клиентам не нужно изменять что-либо, что не повлияет на подключения к серверу. Во время этого изменения [корневой сертификат BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) **не был заменен** на [DigiCertGlobalRootG2](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) , и это изменение откладывается, чтобы предоставить клиентам больше времени на внесение изменений.

## <a name="do-i-need-to-make-any-changes-on-my-client-to-maintain-connectivity"></a>Нужно ли вносить изменения на клиенте для поддержания подключения?

На стороне клиента не требуется никаких изменений. Если следовать приведенной выше рекомендации, вы все равно сможете продолжать подключение, пока **сертификат BaltimoreCyberTrustRoot не будет удален** из Объединенного сертификата ЦС. **Не рекомендуется удалять BaltimoreCyberTrustRoot из Объединенного сертификата ЦС, пока не будет продолжено дополнительное уведомление для поддержания подключения.**

### <a name="previous-recommendation"></a>Предыдущая рекомендация

*   Скачайте корневой центр сертификации BaltimoreCyberTrustRoot & DigiCertGlobalRootG2 по ссылкам ниже:
    *   https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem
    *   https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem

*   Создайте объединенное хранилище сертификатов ЦС с сертификатами **BaltimoreCyberTrustRoot** и **DigiCertGlobalRootG2** .
    *   Для пользователей Java (PostgreSQL JDBC), использующих Дефаултжавасслфактори, выполните:

          ```console
          keytool -importcert -alias PostgreSQLServerCACert  -file D:\BaltimoreCyberTrustRoot.crt.pem  -keystore truststore -storepass password -noprompt
          ```

          ```console
          keytool -importcert -alias PostgreSQLServerCACert2  -file D:\DigiCertGlobalRootG2.crt.pem -keystore truststore -storepass password  -noprompt
          ```

          Затем замените исходный файл хранилища ключей новым созданным файлом.
        *   System. setProperty ("жавакс. NET. SSL. trustStore", "path_to_truststore_file"); 
        *   System. setProperty ("жавакс. NET. SSL. trustStorePassword", "пароль");
        
    *   Для пользователей .NET (Npgsql) в Windows убедитесь в том, что **Baltimore CyberTrust root** и **DigiCert глобального корневого каталога G2** существуют в хранилище сертификатов Windows, доверенных корневых центрах сертификации. Если какие либо сертификаты не существуют, импортируйте недостающий сертификат.

        ![Сертификат .NET для базы данных Azure для PostgreSQL](media/overview/netconnecter-cert.png)

    *   Для пользователей .NET (Npgsql) в Linux, использующих SSL_CERT_DIR, убедитесь, что **BaltimoreCyberTrustRoot** и **DigiCertGlobalRootG2** находятся в каталоге, указанном SSL_CERT_DIR. Если какие либо сертификаты не существуют, создайте недостающий файл сертификата.

    *   Для других пользователей PostgreSQL клиента можно выполнить слияние двух файлов сертификатов ЦС, таких как приведенный ниже формат.

        </br>-----НАЧАТЬ-----СЕРТИФИКАТА
 </br>(Корневой CA1: BaltimoreCyberTrustRoot. CRT. PEM)
 </br>----------КОНЕЧНОГО СЕРТИФИКАТА
 </br>-----НАЧАТЬ-----СЕРТИФИКАТА
 </br>(Корневой CA2: DigiCertGlobalRootG2. CRT. PEM)
 </br>----------КОНЕЧНОГО СЕРТИФИКАТА

*   Замените исходный файл PEM корневого центра сертификации на объединенный файл корневого ЦС и перезапустите приложение или клиент.
*    В будущем после развертывания нового сертификата на стороне сервера можно изменить PEM-файл CA на DigiCertGlobalRootG2. CRT. PEM.

> [!NOTE]
> Не удаляйте и не изменяйте **сертификат Baltimore** , пока не будет внесено изменение сертификата. После внесения изменений будет отправлено сообщение, после чего его можно безопасно удалить с помощью сертификата Baltimore. 

## <a name="why-was-baltimorecybertrustroot-certificate-not-replaced-to-digicertglobalrootg2-during-this-change-on-february-15-2021"></a>Почему BaltimoreCyberTrustRoot сертификат не был заменен на DigiCertGlobalRootG2 в течение этого изменения 15 февраля 2021?

Мы оценили готовность клиентов к этому изменению, и многие клиенты искали дополнительное время для управления этим изменением. В интересах предоставления клиентам большего времени на подготовку, мы решили отложить изменение сертификата на DigiCertGlobalRootG2 по крайней мере за год, предоставляя достаточно времени для клиентов и конечных пользователей. 

Наши рекомендации для пользователей — используйте описанные выше шаги, чтобы создать объединенный сертификат и подключиться к серверу, но не удаляйте сертификат BaltimoreCyberTrustRoot, пока не будет отправлено сообщение для удаления подключения. 

## <a name="what-if-we-removed-the-baltimorecybertrustroot-certificate"></a>Что делать, если мы удалили сертификат BaltimoreCyberTrustRoot?

При подключении к серверу базы данных Azure для PostgreSQL будут возникнет ошибки подключения. Для поддержания подключения необходимо снова настроить SSL с сертификатом [BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) .


## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

###    <a name="1-if-i-am-not-using-ssltls-do-i-still-need-to-update-the-root-ca"></a>1. Если я не использую SSL/TLS, мне все равно нужно обновить корневой ЦС?
Если вы не используете SSL/TLS, никаких действий не требуется. 

### <a name="2-if-i-am-using-ssltls-do-i-need-to-restart-my-database-server-to-update-the-root-ca"></a>2. Если я использую SSL/TLS, нужно ли перезапустить сервер базы данных, чтобы обновить корневой ЦС?
Нет, не нужно перезагружать сервер базы данных, чтобы приступить к использованию нового сертификата. Это изменение на стороне клиента, и для входящих клиентских подключений необходимо использовать новый сертификат, чтобы обеспечить возможность подключения к серверу базы данных.

### <a name="3-how-do-i-know-if-im-using-ssltls-with-root-certificate-verification"></a>3. Разделы справки узнать, используется ли SSL/TLS с проверкой корневого сертификата?

Вы можете определить, проверяют ли ваши подключения корневой сертификат, просмотрев строку подключения.
-    Если строка подключения включает `sslmode=verify-ca` или `sslmode=verify-full` , необходимо обновить сертификат.
-    Если строка подключения включает `sslmode=disable` , `sslmode=allow` , `sslmode=prefer` или `sslmode=require` , не требуется обновлять сертификаты. 
-    Если в строке подключения не указано sslmode, обновлять сертификаты не требуется.

Если вы используете клиент, который абстрагирует строку подключения, изучите документацию клиента, чтобы понять, проверяется ли сертификат. Чтобы понять PostgreSQL sslmode, ознакомьтесь с [описанием режима SSL](https://www.postgresql.org/docs/11/libpq-ssl.html#ssl-mode-descriptions) в документации PostgreSQL.

### <a name="4-what-is-the-impact-if-using-app-service-with-azure-database-for-postgresql"></a>4. Каковы последствия использования службы приложений с базой данных Azure для PostgreSQL?
Для служб приложений Azure, подключающихся к базе данных Azure для PostgreSQL, можно использовать два возможных сценария, которые зависят от того, как вы используете SSL с приложением.
*   Этот новый сертификат добавлен в службу приложений на уровне платформы. Если в приложении используются SSL-сертификаты, включенные в платформу службы приложений, никаких действий не требуется.
*   Если вы явно включаете путь к файлу SSL-сертификата в коде, необходимо скачать новый сертификат и обновить код для использования нового сертификата. Хорошим примером этого сценария является использование пользовательских контейнеров в службе приложений в качестве общих в [документации по службе приложений](../app-service/tutorial-multi-container-app.md#configure-database-variables-in-wordpress) .

### <a name="5-what-is-the-impact-if-using-azure-kubernetes-services-aks-with-azure-database-for-postgresql"></a>5. Каковы последствия использования Azure Kubernetes Services (AKS) с базой данных Azure для PostgreSQL?
Если вы пытаетесь подключиться к базе данных Azure для PostgreSQL с помощью служб Kubernetes Azure (AKS), она аналогична доступу из выделенной среды размещения клиентов. Инструкции см. [здесь](../aks/ingress-own-tls.md).

### <a name="6-what-is-the-impact-if-using-azure-data-factory-to-connect-to-azure-database-for-postgresql"></a>6. Каковы последствия использования фабрики данных Azure для подключения к базе данных Azure для PostgreSQL?
Для соединителя, использующего Azure Integration Runtime, соединитель использует сертификаты из хранилища сертификатов Windows в среде, размещенной в Azure. Эти сертификаты уже совместимы с вновь применяемыми сертификатами и поэтому никаких действий не требуется.

Для соединителя, использующего локально размещенный Integration Runtime, в который явно включен путь к файлу SSL-сертификата в строке подключения, необходимо скачать [новый сертификат](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) и обновить строку подключения, чтобы она использовалась.

### <a name="7-do-i-need-to-plan-a-database-server-maintenance-downtime-for-this-change"></a>7. нужно ли мне запланировать время простоя обслуживания сервера базы данных для этого изменения?
Нет. Так как изменение здесь происходит только на стороне клиента для подключения к серверу базы данных, время простоя обслуживания сервера базы данных для этого изменения не требуется.

### <a name="8-if-i-create-a-new-server-after-february-15-2021-02152021-will-i-be-impacted"></a>8. Если я создам новый сервер после 15 февраля 2021 (02/15/2021), будет ли это затронуто?
Для серверов, созданных после 15 февраля 2021 (02/15/2021), вы по-прежнему будете использовать [BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) для подключения приложений с помощью SSL.

### <a name="9-how-often-does-microsoft-update-their-certificates-or-what-is-the-expiry-policy"></a>9. как часто Корпорация Майкрософт обновляет свои сертификаты или какова политика срока действия?
Эти сертификаты, используемые базой данных Azure для PostgreSQL, предоставляются доверенными центрами сертификации (CA). Поэтому поддержка этих сертификатов привязывается к поддержке этих сертификатов центром сертификации. Срок действия сертификата [BaltimoreCyberTrustRoot](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) истекает через 2025, поэтому перед истечением срока действия корпорация Майкрософт должна выполнить изменение сертификата. Кроме того, если в этих стандартных сертификатах возникают непредвиденные ошибки, то для обеспечения безопасности и совместимости службы Корпорация Майкрософт должна будет выполнить смену сертификатов в наиболее ранней степени, как в случае с 15 февраля 2021.

### <a name="10-if-i-am-using-read-replicas-do-i-need-to-perform-this-update-only-on-the-primary-server-or-the-read-replicas"></a>10. Если я использую для чтения реплики, нужно ли выполнять это обновление только на основном сервере или репликах чтения?
Так как это обновление является изменением на стороне клиента, если клиент использовался для считывания данных с сервера реплики, необходимо также применить изменения для этих клиентов. 

### <a name="11-do-we-have-server-side-query-to-verify-if-ssl-is-being-used"></a>11. у нас есть запрос на стороне сервера, чтобы проверить, используется ли протокол SSL?
Чтобы проверить, используется ли SSL-соединение для подключения к серверу, обратитесь к [проверке SSL](concepts-ssl-connection-security.md#applications-that-require-certificate-verification-for-tls-connectivity).

### <a name="12-is-there-an-action-needed-if-i-already-have-the-digicertglobalrootg2-in-my-certificate-file"></a>12. требуется ли действие, если у меня уже есть DigiCertGlobalRootG2 в файле сертификата?
Нет. Если файл сертификата уже содержит **DigiCertGlobalRootG2**, никаких действий не требуется.

### <a name="13-what-if-you-are-using-docker-image-of-pgbouncer-sidecar-provided-by-microsoft"></a>13. что если вы используете образ DOCKER из Пгбаунцер расширения, предоставляемый корпорацией Майкрософт?
Новый образ DOCKER, который поддерживает как [**Baltimore**](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) , так и [**DigiCert**](https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem) , опубликован ниже [(последний](https://hub.docker.com/_/microsoft-azure-oss-db-tools-pgbouncer-sidecar) тег). Вы можете извлечь этот новый образ, чтобы избежать перерывов в подключении, начиная с 15 февраля 2021. 

### <a name="14-what-if-i-have-further-questions"></a>14. что делать, если у меня возникнут дополнительные вопросы?
Если у вас есть вопросы, получите ответы от экспертов сообщества в [Microsoft Q&а](mailto:AzureDatabaseforPostgreSQL@service.microsoft.com). Если у вас есть план поддержки и вам нужна техническая помощь,  [свяжитесь с нами](mailto:AzureDatabaseforPostgreSQL@service.microsoft.com)

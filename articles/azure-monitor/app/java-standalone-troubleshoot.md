---
title: Устранение неполадок Azure Monitor Application Insights для Java
description: Узнайте, как устранять неполадки агента Java для Azure Monitor Application Insights
ms.topic: conceptual
ms.date: 11/30/2020
ms.custom: devx-track-java
ms.openlocfilehash: f971466f25c2b7a4bd28e5b7eec6268f1b2e8b3d
ms.sourcegitcommit: 94c3c1be6bc17403adbb2bab6bbaf4a717a66009
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103225580"
---
# <a name="troubleshooting-guide-azure-monitor-application-insights-for-java"></a>Руководство по устранению неполадок: Azure Monitor Application Insights для Java

В этой статье рассматриваются некоторые распространенные проблемы, которые могут возникнуть при инструментировании приложения Java с помощью агента Java для Application Insights. Мы также рассмотрим действия по устранению этих проблем. Application Insights является компонентом службы платформы Azure Monitor.

## <a name="check-the-self-diagnostic-log-file"></a>Проверка файла журнала самодиагностики

По умолчанию агент Java 3,0 для Application Insights создает файл журнала с именем `applicationinsights.log` в том же каталоге, в котором находится `applicationinsights-agent-3.0.2.jar` файл.

Этот файл журнала является первым местом для поиска подсказок о любых проблемах, которые могут возникнуть.

## <a name="jvm-fails-to-start"></a>Не удается запустить ВИРТУАЛЬНОЙ машины Java

Если ВИРТУАЛЬНОЙ машины Java не удается запустить с ошибкой "Ошибка открытия ZIP-файла или манифеста JAR", попробуйте повторно Скачать JAR-файл агента, так как он мог быть поврежден при переносе файлов.

## <a name="upgrade-from-the-application-insights-java-2x-sdk"></a>Обновление с помощью пакета SDK для Java 2. x Application Insights

Если вы уже используете пакет SDK для Java 2. x Application Insights в приложении, вы можете использовать его. Агент Java 3,0 определит его. Дополнительные сведения см. [в статье обновление с помощью пакета SDK для Java 2. x](./java-standalone-upgrade-from-2x.md).

## <a name="upgrade-from-application-insights-java-30-preview"></a>Обновление с предварительной версии Application Insights Java 3,0

Если вы выполняете обновление с помощью предварительной версии агента Java 3,0, внимательно просмотрите все [Параметры конфигурации](./java-standalone-config.md) . Структура JSON полностью изменена в выпуске 3,0 общей доступности (общедоступной версии).

Это следующие изменения:

-  Имя файла конфигурации изменено с `ApplicationInsights.json` на `applicationinsights.json` .
-  `instrumentationSettings`Узел больше не существует. Все содержимое в `instrumentationSettings` перемещается на корневой уровень. 
-  Узлы конфигурации, такие как `sampling` ,, `jmxMetrics` и, `instrumentation` `heartbeat` перемещаются из `preview` в корневой уровень.

## <a name="some-logging-is-not-auto-collected"></a>Некоторые журналы не собираются

Ведение журнала захватывается только в том случае, если оно сначала соответствует уровню, настроенному для платформы ведения журнала, и, во-вторых, также соответствует уровню, настроенному для Application Insights.

Например, если ваша платформа ведения журналов настроена для ведения журнала `WARN` (и выше) из пакета `com.example` , а Application Insights настроена для записи `INFO` (и более поздних версий), Application Insights будет записывать только `WARN` (и выше) из пакета `com.example` .

Лучший способ узнать, соответствует ли определенная инструкция ведения журнала заданному пороговому значению платформ ведения журналов, — убедиться, что она отображается в нормальном журнале приложений (например, файл или консоль).

Также обратите внимание, что если в средство ведения журнала передается объект исключения, то сообщение журнала (и сведения об объекте исключения) будет отображаться в портал Azure в `exceptions` таблице, а не в `traces` таблице.

Дополнительные сведения см. в разделе [Конфигурация автоматического сбора журналов](./java-standalone-config.md#auto-collected-logging) .

## <a name="import-ssl-certificates"></a>Импорт SSL-сертификатов

Этот раздел поможет устранить неполадки и, возможно, исправить исключения, связанные с SSL-сертификатами при использовании агента Java.

Ниже приведены два разных пути для решения этой проблемы.
* При использовании хранилища ключей Java по умолчанию
* При использовании пользовательского хранилища ключей Java

Если вы не уверены, какой путь следует отслеживать, проверьте, есть ли у вас аргумент ВИРТУАЛЬНОЙ машины Java `-Djavax.net.ssl.trustStore=...` .
Если у вас _нет_ такого аргумента виртуальной машины Java, вы, вероятно, используете хранилище ключей Java по умолчанию.
Если у _вас есть такой_ аргумент виртуальной машины Java, вы, вероятно, используете настраиваемое хранилище ключей, а аргумент виртуальной машины Java указывает на пользовательский хранилище ключей.

### <a name="if-using-the-default-java-keystore"></a>При использовании хранилища ключей Java по умолчанию:

Как правило, хранилище ключей Java по умолчанию уже содержит все корневые сертификаты ЦС. Однако могут возникнуть некоторые исключения, например сертификат конечной точки приема может быть подписан другим корневым сертификатом. Чтобы устранить эту проблему, мы рекомендуем выполнить следующие три шага:

1.  Убедитесь, что корневой сертификат, использованный для подписывания конечной точки Application Insights, уже существует в хранилище ключей по умолчанию. Сертификаты доверенного ЦС по умолчанию хранятся в `$JAVA_HOME/jre/lib/security/cacerts` . Чтобы получить список сертификатов в хранилище ключей Java, используйте следующую команду:
    > `keytool -list -v -keystore $PATH_TO_KEYSTORE_FILE`
 
    Выходные данные можно перенаправить во временный файл, как в этом случае (будет легко выполнить поиск в дальнейшем)
    > `keytool -list -v -keystore $JAVA_HOME/jre/lib/security/cacerts > temp.txt`

2. Получив список сертификатов, выполните следующие [действия](#steps-to-download-ssl-certificate) , чтобы скачать корневой сертификат, который использовался для подписания конечной точки Application Insights.

    После загрузки сертификата создайте хэш SHA-1 для сертификата с помощью следующей команды:
    > `keytool -printcert -v -file "your_downloaded_root_certificate.cer"`
 
    Скопируйте значение SHA-1 и проверьте, есть ли это значение в ранее сохраненном файле "temp.txt".  Если вы не можете найти значение SHA-1 во временном файле, это означает, что скачанный корневой сертификат отсутствует в хранилище ключей Java по умолчанию.


3. Импортируйте корневой сертификат в хранилище ключей Java по умолчанию с помощью следующей команды:
    >   `keytool -import -file "the cert file" -alias "some meaningful name" -keystore "path to cacerts file"`
 
    В этом случае он будет
 
    > `keytool -import -file "your downloaded root cert file" -alias "some meaningful name" $JAVA_HOME/jre/lib/security/cacerts`


### <a name="if-using-a-custom-java-keystore"></a>При использовании пользовательского хранилища ключей Java:

Если вы используете настраиваемый хранилище ключей Java, может потребоваться импортировать в него корневые сертификаты SSL Application Insights конечных точек.
Для решения этой проблемы рекомендуется выполнить следующие два действия.
1. Выполните следующие [действия](#steps-to-download-ssl-certificate) , чтобы скачать корневой сертификат из конечной точки Application Insights.
2. Используйте следующую команду, чтобы импортировать корневой SSL-сертификат в настраиваемый хранилище ключей Java:
    > `keytool -importcert -alias your_ssl_certificate -file "your downloaded SSL certificate name.cer" -keystore "Your KeyStore name" -storepass "Your keystore password" -noprompt`

### <a name="steps-to-download-ssl-certificate"></a>Действия по скачиванию SSL-сертификата

1.  Откройте свой любимый браузер и перейдите по `IngestionEndpoint` URL-адресу, указанному в строке подключения, используемой для инструментирования приложения.

    :::image type="content" source="media/java-ipa/troubleshooting/ingestion-endpoint-snippet.png" alt-text="Снимок экрана, на котором показана строка подключения Application Insights." lightbox="media/java-ipa/troubleshooting/ingestion-endpoint-snippet.png":::

2.  Выберите значок **Просмотр сведений о сайте** (блокировка) в браузере, а затем выберите параметр **сертификат** .

    :::image type="content" source="media/java-ipa/troubleshooting/certificate-icon-capture.png" alt-text="Снимок экрана с параметром сертификата в сведениях о сайте." lightbox="media/java-ipa/troubleshooting/certificate-icon-capture.png":::

3.  Вместо загрузки конечного сертификата необходимо скачать корневой сертификат, как показано ниже. Позже необходимо щелкнуть "путь к сертификату" — > выбрать корневой сертификат, > щелкните "Просмотр сертификата". Откроется меню нового сертификата, и вы сможете скачать сертификат из меню создать.

    :::image type="content" source="media/java-ipa/troubleshooting/root-certificate-selection.png" alt-text="Снимок экрана выбора корневого сертификата." lightbox="media/java-ipa/troubleshooting/root-certificate-selection.png":::

4.  Перейдите на вкладку **сведения** и выберите **Копировать в файл**.
5.  Нажмите кнопку **Далее** , выберите **base-64 Encoded X. 509 (. CER)** , а затем снова нажмите кнопку **Далее** .

    :::image type="content" source="media/java-ipa/troubleshooting/certificate-export-wizard.png" alt-text="Снимок экрана мастера экспорта сертификатов с выбранным форматом." lightbox="media/java-ipa/troubleshooting/certificate-export-wizard.png":::

6.  Укажите файл, в котором вы хотите сохранить SSL-сертификат. Затем нажмите кнопку **Далее**  >  **Готово**. Вы увидите сообщение "Экспорт выполнен успешно".

> [!WARNING]
> Чтобы получить новый сертификат до истечения срока действия текущего сертификата, необходимо повторить эти действия. Сведения об истечении срока действия можно найти на вкладке **сведения** в диалоговом окне **сертификат** .
>
> :::image type="content" source="media/java-ipa/troubleshooting/certificate-details.png" alt-text="Снимок экрана, показывающий сведения о SSL-сертификате." lightbox="media/java-ipa/troubleshooting/certificate-details.png":::

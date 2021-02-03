---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 02/20/2020
ms.author: trbye
ms.openlocfilehash: 39cead8306ece6638f85813efc318dd41a6faf8c
ms.sourcegitcommit: d1e56036f3ecb79bfbdb2d6a84e6932ee6a0830e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/29/2021
ms.locfileid: "99213809"
---
В этом кратком руководстве вы узнаете, как использовать пакет SDK для речевых устройств в Linux для создания устройства с поддержкой речевых функций или его использования в качестве устройства для [транскрибирования бесед](../conversation-transcription.md). Сейчас поддерживается только [Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/).

Приложение создается с использованием пакета SDK для службы "Речь" и интегрированной среды разработки Eclipse Java (версия 4) в 64-разрядной версии ОС Linux (Ubuntu 16.04, Ubuntu 18.04, Debian 9, RHEL 7/8, CentOS 7/8). Оно работает в 64-разрядной среде выполнения Java 8 (JRE).

Для этого руководства требуется [учетная запись Azure Cognitive Services](../overview.md#try-the-speech-service-for-free) и ресурс службы "Речь". 

Исходный код [примера приложения](https://aka.ms/sdsdk-download-JRE) входит в состав пакета SDK для речевых устройств. Он также [доступен на веб-сайте GitHub](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством вам понадобится:

* Операционная система: 64-разрядная версия Linux (Ubuntu 16.04, Ubuntu 18.04, Debian 9, RHEL 7/8, CentOS 7/8).
* [Azure Kinect DK](https://azure.microsoft.com/services/kinect-dk/)
* [Eclipse Java IDE](https://www.eclipse.org/downloads/)
* Только [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html) или [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html).
* Ключ подписки Azure для службы "Речь". [Получить бесплатно](../overview.md#try-the-speech-service-for-free).
* Скачайте последнюю версию [пакета SDK для речевых устройств](https://aka.ms/sdsdk-download-JRE) для Java и извлеките ZIP-файл в рабочую папку.
   > [!NOTE]
   > В этом кратком руководстве предполагается, что приложение извлечено в папку /home/wcaltest/JRE-Sample-Release.

Перед запуском Eclipse убедитесь, что установлены следующие зависимости.

* В Ubuntu:

  ```sh
  sudo apt-get update
  sudo apt-get install libssl1.0.0 libasound2
  ```

* Для Debian 9:

  ```sh
  sudo apt-get update
  sudo apt-get install libssl1.0.2 libasound2
  ```

* Для RHEL/CentOS.
  
  ```sh
  sudo yum update
  sudo yum install alsa-lib openssl
  ```

  > [!NOTE]
  > - Если вы используете RHEL/CentOS 7, выполните инструкции по [настройке пакета SDK службы "Речь" в RHEL/CentOS 7](~/articles/cognitive-services/speech-service/how-to-configure-rhel-centos-7.md).
> - Если вы используете RHEL/CentOS 8, выполните инструкции по [настройке OpenSSL для Linux](~/articles/cognitive-services/speech-service/how-to-configure-openssl-linux.md).

Сейчас транскрибирование бесед доступно для американского английского и китайского языков в регионах "центральная часть США" и "Восточная Азия". Для использования транскрибирования бесед вам понадобится речевой ключ в одном из упомянутых регионов.

Если вы планируете использовать намерения, вам понадобится подписка на [службу "Распознавания речи" (LUIS)](../../luis/luis-how-to-azure-subscription.md). Дополнительные сведения о службе распознавания речи и распознавании намерений см. в статье [Распознавание намерений в речи с помощью пакета SDK службы распознавания речи для C#](../how-to-recognize-intents-from-speech-csharp.md). [Модель примера LUIS](https://aka.ms/sdsdk-luis) доступна для этого приложения.

## <a name="create-and-configure-the-project"></a>Создание и настройка проекта

1. Запустите Eclipse.

1. В **средстве запуска интегрированной среды разработки Eclipse** в поле **Workspace** (Рабочая область) введите имя каталога новой рабочей области. Затем выберите **Запустить**.

   ![Снимок экрана, на котором показано средство запуска Eclipse.](../media/speech-devices-sdk/eclipse-launcher-linux.png)

1. После этого отобразится главное окно интегрированной среды разработки Eclipse. Если отобразится экран приветствия, закройте его.

1. В строке меню Eclipse создайте новый проект, выбрав **File** > **New** > **Java Project** (Файл > Создать > Проект Java). Если последнего пункта нет, выберите **Project** (Проект), а затем — **Java Project** (Проект Java).

1. После этого запустится мастер **создания проектов Java**. **Перейдите** к расположению примера проекта. Нажмите кнопку **Готово**.

   ![Снимок экрана с изображением мастера создания проекта Java](../media/speech-devices-sdk/eclipse-new-java-project-linux.png)

1. В **обозревателе пакетов** щелкните проект правой кнопкой мыши. В контекстном меню выберите **Настроить** > **Convert to Maven Project** (Преобразовать в проект Maven). Нажмите кнопку **Готово**.

   ![Снимок экрана обозревателя пакетов](../media/speech-devices-sdk/eclipse-convert-to-maven.png)

1. Откройте файл pom.xml и измените его.

    В конце файла перед закрывающим тегом `</project>` создайте элементы `repositories` и `dependencies`, как показано здесь, и убедитесь, что `version` соответствует текущей версии:
    ```xml    
    <repositories>
         <repository>
             <id>maven-cognitiveservices-speech</id>
             <name>Microsoft Cognitive Services Speech Maven Repository</name>
             <url>https://csspeechstorage.blob.core.windows.net/maven/</url>
         </repository>
    </repositories>
 
    <dependencies>
        <dependency>
             <groupId>com.microsoft.cognitiveservices.speech</groupId>
             <artifactId>client-sdk</artifactId>
             <version>1.15.0</version>
        </dependency>
    </dependencies>
   ```

1. В **обозревателе пакетов** щелкните проект правой кнопкой мыши. Выберите **Properties** (Свойства), а затем — **Run/Debug Settings** (Параметры отладки/запуска) > **New…** (Создать…) > **Java Application** (Приложение Java). 

1. Появится окно **Edit Configuration** (Изменение конфигурации). В поле **Name** (Имя) введите **Main** и используйте кнопку **Search** (Поиск) в разделе **Main Class** (Класс Main), чтобы найти и выбрать **com.microsoft.cognitiveservices.speech.samples.FunctionsList**.

   ![Снимок экрана: изменение конфигурации запуска](../media/speech-devices-sdk/eclipse-edit-launch-configuration-linux.png)

1. Скопируйте двоичные файлы аудио для целевой архитектуры из **Linux-arm** или **Linux-x64** в расположение проекта Java, например в **/home/wcaltest/JRE-Sample-Release**.

1. В окне **Edit Configuration** (Изменение конфигурации) выберите страницу **Environment** (Среда) и **New** (Новая). Появится окно переменной **New Environment Variable** (Новая переменная среды). В поле **Name** (Имя) введите **LD_LIBRARY_PATH** и в поле **Value** (Значение) введите папку, содержащую файлы *.so, например **/home/wcaltest/JRE-Sample-Release**.

1. Скопируйте `kws.table` и `participants.properties` в папку проекта **target/classes**.


## <a name="configure-the-sample-application"></a>Настройка примера приложения

1. Добавьте свой ключ подписки на речевые службы в исходный код. Если вы хотите попробовать распознать намерения, также добавьте свой ключ подписки [службы "Распознавание речи"](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/) и идентификатор приложения.

   Для распознавания речи и LUIS ваши данные записываются в файл `FunctionsList.java`:

   ```java
    // Subscription
    private static String SpeechSubscriptionKey = "<enter your subscription info here>";
    private static String SpeechRegion = "westus"; // You can change this if your speech region is different.
    private static String LuisSubscriptionKey = "<enter your subscription info here>";
    private static String LuisRegion = "westus2"; // you can change this, if you want to test the intent, and your LUIS region is different.
    private static String LuisAppId = "<enter your LUIS AppId>";
   ```

    Если вы пользуетесь транскрибированием бесед, информация о вашем речевом ключе и регионе также должна быть в файле `Cts.java`:

   ```java
    private static final String CTSKey = "<Conversation Transcription Service Key>";
    private static final String CTSRegion="<Conversation Transcription Service Region>";// Region may be "centralus" or "eastasia"
    ```

1. Ключевое слово по умолчанию — "Компьютер". Можно также попробовать одно из других предоставленных ключевых слов — "Машина" или "Помощник". Файлы ресурсов для этих альтернативных ключевых слов находятся в пакете SDK для речевых устройств в папке keyword. Например, `/home/wcaltest/JRE-Sample-Release/keyword/Computer` содержит файлы, используемые для ключевого слова "Компьютер".

   > [!TIP]
   > Кроме того, вы можете [создать пользовательское ключевое слово](../custom-keyword-basics.md).

    Для использования нового ключевого слова обновите следующую строку в файле `FunctionsList.java` и скопируйте ключевое слово в приложение. Например, чтобы использовать ключевое слово "Машина" из файла `machine.zip` пакета ключевых слов, сделайте следующее.

   * Скопируйте файл `kws.table` из ZIP-пакета слов в папку проекта **target/classes**.

   * Укажите в файле `FunctionsList.java` имя ключевого слова:

     ```java
     private static final String Keyword = "Machine";
     ```

## <a name="run-the-sample-application-from-eclipse"></a>Запуск примера приложения из Eclipse

1. В строке меню Eclipse выберите **Run** (Запуск) > **Run** (Запуск). 

1. Запускается пример приложения пакета SDK для речевых устройств, отображающий следующие параметры:

   ![Снимок экрана, на котором показан пример приложения из пакета SDK для речевых устройств и параметры.](../media/speech-devices-sdk/java-sample-app-linux.png)

1. Попробуйте новую демоверсию **транскрибирования бесед**. Запустите транскрибирование, выбрав **Session** (Сеанс) > **Start** (Запустить). По умолчанию все являются гостями. Тем не менее, если у вас есть голосовые подписи участников, вы можете добавить их в файл `participants.properties` в папке проекта **target/classes**. Сведения о том, как создать голосовую подпись, см. в статье о [транскрибировании бесед (пакет SDK)](../how-to-use-conversation-transcription.md).

   ![Снимок экрана, на котором показана демоверсия приложения транскрибирования бесед.](../media/speech-devices-sdk/cts-sample-app-linux.png)

## <a name="create-and-run-standalone-the-application"></a>Создание и запуск автономного приложения

1. В **обозревателе пакетов** щелкните проект правой кнопкой мыши. Выберите **Export** (Экспорт). 
1. Появится окно **Export** (Экспорт). Разверните узел **Java** и выберите **Runnable JAR file** (Готовый к запуску JAR-файл), а затем нажмите кнопку **Next** (Далее).

   ![Снимок экрана, на котором показаны параметры экспорта.](../media/speech-devices-sdk/eclipse-export-linux.png) 

1. Появится окно **Runnable JAR File Export** (Экспорт готовых к запуску JAR-файлов). Выберите **назначение экспорта** для приложения, а затем нажмите кнопку **Finish** (Готово).
 
   ![Снимок экрана, на котором показано окно для экспорта готовых к запуску JAR-файлов.](../media/speech-devices-sdk/eclipse-export-jar-linux.png)

1. Поместите `kws.table` и `participants.properties` в папку назначения, выбранную выше, так как эти файлы необходимы для приложения.

1. Задайте для параметра LD_LIBRARY_LIB папку, содержащую файлы *.so.

     ```bash
     export LD_LIBRARY_PATH=/home/wcaltest/JRE-Sample-Release
     ```

1. Чтобы запустить автономное приложение, выполните следующую команду:

     ```bash
     java -jar SpeechDemo.jar
     ```
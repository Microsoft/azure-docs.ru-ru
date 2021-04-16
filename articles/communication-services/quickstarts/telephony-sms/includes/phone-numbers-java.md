---
ms.openlocfilehash: 4891a602e9fb5ebbdaadc980c1960b95816c6653
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113310"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [комплект SDK для Java (JDK)](/java/azure/jdk/?preserve-view=true&view=azure-java-stable) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org/download.cgi). Развернутый ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-java-application"></a>Создание нового приложения Java

Откройте терминал или командное окно. Перейдите в каталог, в котором нужно создать приложение Java. Выполните приведенную ниже команду, чтобы создать проект Java из шаблона maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Вы заметите, что задача generate создала каталог с тем же именем, что и у `artifactId`. В этом каталоге каталог src/main/java содержит исходный код проекта, `pom.xml` содержит источник теста, а файл `src/test/java directory` является объектной моделью проекта (POM).

### <a name="install-the-package"></a>Установка пакета

Откройте файл **pom.xml** в текстовом редакторе. Добавьте приведенные ниже элементы зависимости в группу зависимостей.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-phonenumbers</artifactId>
    <version>1.0.0-beta.7</version>
</dependency>

<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.2.3</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Перейдите в каталог */src/main/java/com/communication/quickstart*.
1. Откройте файл *App.java* в редакторе.
1. Замените оператор `System.out.println("Hello world!");`.
1. Добавьте директивы `import`.

Используйте следующий код:

```java
import com.azure.communication.phonenumbers.*;
import com.azure.communication.phonenumbers.models.*;
import java.io.*;
import com.azure.core.util.Context;
import com.azure.core.util.polling.PollResponse;
import com.azure.identity.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - Phone Numbers Quickstart");
        // Quickstart code goes here
    }
}
```

## <a name="authenticate-the-phone-numbers-client"></a>Проверка подлинности клиента номеров телефона

PhoneNumberClientBuilder включен, чтобы использовать проверку подлинности Azure Active Directory
<!-- embedme ./src/samples/java/com/azure/communication/phonenumbers/ReadmeSamples.java#L52-L62 -->
```java
// You can find your endpoint and access key from your resource in the Azure portal
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";

PhoneNumbersClient phoneNumberClient = new PhoneNumbersClientBuilder()
    .endpoint(endpoint)
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();
```

Кроме того, для проверки подлинности также можно использовать конечную точку и ключ доступа из ресурса коммуникации.
<!-- embedme ./src/samples/java/com/azure/communication/phonenumbers/ReadmeSamples.java#L30-L41 -->
```java
// You can find your connection string from your resource in the Azure portal
String connectionString = "https://<RESOURCE_NAME>.communication.azure.com/;accesskey=<ACCESS_KEY>";

PhoneNumbersClient phoneNumberClient = new PhoneNumbersClientBuilder()
    .connectionString(connectionString)
    .buildClient();
```

## <a name="manage-phone-numbers"></a>Управление номерами телефонов

### <a name="search-for-available-phone-numbers"></a>Поиск доступных номеров телефонов

Чтобы приобрести номера телефонов, сначала необходимо найти доступные номера. Для поиска номеров телефонов укажите код города, тип назначения, [возможности номеров телефона](../../../concepts/telephony-sms/plan-solution.md#phone-number-capabilities-in-azure-communication-services), [тип номера телефона](../../../concepts/telephony-sms/plan-solution.md#phone-number-types-in-azure-communication-services) и количество. Обратите внимание, что для типа бесплатного телефонного номера указывать код города необязательно.

```java
 PhoneNumberCapabilities capabilities = new PhoneNumberCapabilities()
    .setCalling(PhoneNumberCapabilityType.INBOUND)
    .setSms(PhoneNumberCapabilityType.INBOUND_OUTBOUND);
PhoneNumberSearchOptions searchOptions = new PhoneNumberSearchOptions().setAreaCode("833").setQuantity(1);

PhoneNumberSearchResult searchResult = phoneNumberClient
    .beginSearchAvailablePhoneNumbers("US", PhoneNumberType.TOLL_FREE, PhoneNumberAssignmentType.APPLICATION, capabilities,  searchOptions, Context.NONE)
    .getFinalResult();

System.out.println("Searched phone numbers: " + searchResult.getPhoneNumbers());
System.out.println("Search expires by: " + searchResult.getSearchExpiresBy());
System.out.println("Phone number costs:" + searchResult.getCost().getAmount());
```

### <a name="purchase-phone-numbers"></a>Покупка номеров телефонов

Результатом поиска номеров телефонов является PhoneNumberSearchResult. Он содержит `searchId`, который можно передать в API для покупки номеров, чтобы купить найденные номера. Обратите внимание, что при вызове API для покупки номеров телефона на учетную запись Azure будет начисляться платеж.

```java
PollResponse<PhoneNumberOperation> purchaseResponse = phoneNumberClient.beginPurchasePhoneNumbers(searchResult.getSearchId(), Context.NONE).waitForCompletion();
System.out.println("Purchase phone numbers operation is: " + purchaseResponse.getStatus());
```

### <a name="get-phone-numbers"></a>Получение номеров телефона

После покупки номера его можно получить из клиента.
```java
PurchasedPhoneNumber phoneNumber = phoneNumberClient.getPurchasedPhoneNumber("+14255550123");
System.out.println("Phone Number Country Code: " + phoneNumber.getCountryCode());
```

Вы также можете получить все приобретенные номера телефонов.
``` java
PagedIterable<PurchasedPhoneNumber> phoneNumbers = phoneNumberClient.listPurchasedPhoneNumbers(Context.NONE);
PurchasedPhoneNumber phoneNumber = phoneNumbers.iterator().next();
System.out.println("Phone Number Country Code: " + phoneNumber.getCountryCode());
```

### <a name="update-phone-number-capabilities"></a>Обновление возможностей номера телефона

Возможности приобретенного номера можно обновить.
```java
PhoneNumberCapabilities capabilities = new PhoneNumberCapabilities();
capabilities
    .setCalling(PhoneNumberCapabilityType.INBOUND)
    .setSms(PhoneNumberCapabilityType.INBOUND_OUTBOUND);
PurchasedPhoneNumber phoneNumber = phoneNumberClient.beginUpdatePhoneNumberCapabilities("+14255550123", capabilities, Context.NONE).getFinalResult();

System.out.println("Phone Number Calling capabilities: " + phoneNumber.getCapabilities().getCalling()); //Phone Number Calling capabilities: inbound
System.out.println("Phone Number SMS capabilities: " + phoneNumber.getCapabilities().getSms()); //Phone Number SMS capabilities: inbound+outbound
```

### <a name="release-phone-number"></a>Освобождение номера телефона

Приобретенный номер телефона можно освободить.
```java
PollResponse<PhoneNumberOperation> releaseResponse =
    phoneNumberClient.beginReleasePhoneNumber("+14255550123", Context.NONE).waitForCompletion();
System.out.println("Release phone number operation is: " + releaseResponse.getStatus());
```

## <a name="run-the-code"></a>Выполнение кода

Перейдите в каталог, содержащий файл *pom.xml*, и скомпилируйте проект с помощью следующей команды `mvn`.

```console
mvn compile
```

Затем выполните сборку пакета.

```console
mvn package
```

Выполните следующую команду `mvn` для запуска приложения.

```console
mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false
```

В выходных данных приложения описывают каждое выполненное действие.
<!---cSpell:disable --->
```console
Azure Communication Services - Phone Numbers Quickstart

Searched phone numbers: [+18001234567]

Purchase phone numbers operation is: SUCCESSFULLY_COMPLETED

Phone Number Country Code: US

Phone Number Calling capabilities: inbound

Phone Number SMS capabilities: inbound

Release phone number operation is: SUCCESSFULLY_COMPLETED

```
<!---cSpell:enable --->
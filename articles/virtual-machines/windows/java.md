---
title: Создание виртуальной машины Azure и управление ею с помощью Java
description: Развертывание виртуальной машины и всех ее вспомогательных ресурсов с помощью Java и Azure Resource Manager.
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure
ms.topic: how-to
ms.date: 07/17/2017
ms.custom: devx-track-java
ms.author: cynthn
ms.openlocfilehash: ea9d5d92d45db50470276929b6e7b4200bb427d6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102557528"
---
# <a name="create-and-manage-windows-vms-in-azure-using-java"></a>Создание виртуальных машин Windows в Azure и управление ими с помощью Java

[Виртуальной машине Azure](overview.md) требуется несколько вспомогательных ресурсов Azure. В этой статье описывается создание, управление и удаление ресурсов виртуальной машины с помощью Java. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание проекта Maven
> * Добавление зависимостей
> * Создание учетных данных
> * Создание ресурсов
> * Выполнение задач управления.
> * Удаление ресурсов
> * Выполнение приложения

На выполнение этих действий требуется примерно 20 минут.

## <a name="create-a-maven-project"></a>Создание проекта Maven

1. Установите [Java](/azure/developer/java/fundamentals/java-jdk-long-term-support), если это еще не сделано.
2. Установите [Maven](https://maven.apache.org/download.cgi).
3. Создайте папку и проект:
    
    ```
    mkdir java-azure-test
    cd java-azure-test
    
    mvn archetype:generate -DgroupId=com.fabrikam -DartifactId=testAzureApp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

## <a name="add-dependencies"></a>Добавление зависимостей

1. В папке `testAzureApp` откройте файл `pom.xml` и добавьте конфигурацию сборки к &lt;проекту&gt;, чтобы включить создание приложения:

    ```xml
    <build>
      <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.fabrikam.testAzureApp.App</mainClass>
            </configuration>
        </plugin>
      </plugins>
    </build>
    ```

2. Добавьте зависимости, необходимые для доступа к пакету Azure SDK для Java.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-compute</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-resources</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-network</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.squareup.okio</groupId>
      <artifactId>okio</artifactId>
      <version>1.13.0</version>
    </dependency>
    <dependency>
      <groupId>com.nimbusds</groupId>
      <artifactId>nimbus-jose-jwt</artifactId>
      <version>3.6</version>
    </dependency>
    <dependency>
      <groupId>net.minidev</groupId>
      <artifactId>json-smart</artifactId>
      <version>1.0.6.3</version>
    </dependency>
    <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
      <version>1.4.5</version>
    </dependency>
    ```

3. Сохраните файл.

## <a name="create-credentials"></a>Создание учетных данных

Прежде чем выполнить этот шаг, убедитесь в наличии доступа к [субъекту-службе Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md). Кроме того, необходимо записать идентификатор приложения, ключ проверки подлинности и идентификатор клиента, которые понадобятся позже.

### <a name="create-the-authorization-file"></a>Создание файла авторизации

1. Создайте файл `azureauth.properties` и добавьте в него следующие свойства:

    ```
    subscription=<subscription-id>
    client=<application-id>
    key=<authentication-key>
    tenant=<tenant-id>
    managementURI=https://management.core.windows.net/
    baseURL=https://management.azure.com/
    authURL=https://login.windows.net/
    graphURL=https://graph.microsoft.com/
    ```

    Замените **&lt;subscription-id&gt;** своим идентификатором подписки, **&lt;application-id&gt;** — идентификатором приложения Active Directory, **&lt;authentication-key&gt;** — ключом приложения, а **&lt;tenant-id&gt;** — идентификатором клиента.

2. Сохраните файл.
3. Задайте в оболочке переменную среды AZURE_AUTH_LOCATION и укажите полный путь к файлу аутентификации.

### <a name="create-the-management-client"></a>Создание клиента управления

1. В папке `src\main\java\com\fabrikam` откройте файл `App.java` и убедитесь, что этот оператор statement находится в верхней части:

    ```java
    package com.fabrikam.testAzureApp;
    ```

2. Под оператором statement добавьте следующие операторы import:
   
    ```java
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.compute.AvailabilitySet;
    import com.microsoft.azure.management.compute.AvailabilitySetSkuTypes;
    import com.microsoft.azure.management.compute.CachingTypes;
    import com.microsoft.azure.management.compute.InstanceViewStatus;
    import com.microsoft.azure.management.compute.DiskInstanceView;
    import com.microsoft.azure.management.compute.VirtualMachine;
    import com.microsoft.azure.management.compute.VirtualMachineSizeTypes;
    import com.microsoft.azure.management.network.PublicIPAddress;
    import com.microsoft.azure.management.network.Network;
    import com.microsoft.azure.management.network.NetworkInterface;
    import com.microsoft.azure.management.resources.ResourceGroup;
    import com.microsoft.azure.management.resources.fluentcore.arm.Region;
    import com.microsoft.azure.management.resources.fluentcore.model.Creatable;
    import com.microsoft.rest.LogLevel;
    import java.io.File;
    import java.util.Scanner;
    ```

2. Чтобы создать учетные данные Active Directory, необходимые для выполнения запросов, добавьте следующий код в метод Main класса App:
   
    ```java
    try {
        final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));
        Azure azure = Azure.configure()
            .withLogLevel(LogLevel.BASIC)
            .authenticate(credFile)
            .withDefaultSubscription();
    } catch (Exception e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
    }

    ```

## <a name="create-resources"></a>Создание ресурсов

### <a name="create-the-resource-group"></a>Создание группы ресурсов

Все ресурсы должны содержаться в [группе ресурсов](../../azure-resource-manager/management/overview.md).

Чтобы определить значения для приложения и создать группу ресурсов, добавьте этот код в блок Try метода Main:

```java
System.out.println("Creating resource group...");
ResourceGroup resourceGroup = azure.resourceGroups()
    .define("myResourceGroup")
    .withRegion(Region.US_EAST)
    .create();
```

### <a name="create-the-availability-set"></a>Создание группы доступности

[Группы доступности](tutorial-availability-sets.md) упрощают обслуживание виртуальных машин, используемых приложением.

Чтобы создать группу доступности, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Creating availability set...");
AvailabilitySet availabilitySet = azure.availabilitySets()
    .define("myAvailabilitySet")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withSku(AvailabilitySetSkuTypes.MANAGED)
    .create();
```
### <a name="create-the-public-ip-address"></a>Создание общедоступного IP-адреса

[Общедоступный IP-адрес](../../virtual-network/public-ip-addresses.md) необходим для взаимодействия с виртуальной машиной.

Чтобы создать общедоступный IP-адрес виртуальной машины, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Creating public IP address...");
PublicIPAddress publicIPAddress = azure.publicIPAddresses()
    .define("myPublicIP")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withDynamicIP()
    .create();
```

### <a name="create-the-virtual-network"></a>Создание виртуальной сети

Виртуальная машина должна быть в подсети [виртуальной сети](../../virtual-network/virtual-networks-overview.md).

Чтобы создать подсеть и виртуальную сеть, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Creating virtual network...");
Network network = azure.networks()
    .define("myVN")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withAddressSpace("10.0.0.0/16")
    .withSubnet("mySubnet","10.0.0.0/24")
    .create();
```

### <a name="create-the-network-interface"></a>Создание сетевого интерфейса

Для обмена данными в виртуальной сети виртуальной машине нужен сетевой интерфейс.

Чтобы создать сетевой интерфейс, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Creating network interface...");
NetworkInterface networkInterface = azure.networkInterfaces()
    .define("myNIC")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetwork(network)
    .withSubnet("mySubnet")
    .withPrimaryPrivateIPAddressDynamic()
    .withExistingPrimaryPublicIPAddress(publicIPAddress)
    .create();
```

### <a name="create-the-virtual-machine"></a>Создание виртуальной машины

Теперь, когда вы создали все вспомогательные ресурсы, можно создать виртуальную машину.

Чтобы создать виртуальную машину, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Creating virtual machine...");
VirtualMachine virtualMachine = azure.virtualMachines()
    .define("myVM")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetworkInterface(networkInterface)
    .withLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2012-R2-Datacenter")
    .withAdminUsername("azureuser")
    .withAdminPassword("Azure12345678")
    .withComputerName("myVM")
    .withExistingAvailabilitySet(availabilitySet)
    .withSize("Standard_DS1")
    .create();
Scanner input = new Scanner(System.in);
System.out.println("Press enter to get information about the VM...");
input.nextLine();
```

> [!NOTE]
> В этом учебнике создается виртуальная машина под управлением одной из версий операционной системы Windows Server. Дополнительные сведения о выборе других образов см. в статье [Просмотр и выбор образов виртуальных машин Windows в Azure с помощью оболочки PowerShell или интерфейса командной строки](../linux/cli-ps-findimage.md).
> 
>

Если вы хотите использовать имеющийся диск вместо образа Marketplace, используйте следующий код: 

```java
ManagedDisk managedDisk = azure.disks.define("myosdisk")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withWindowsFromVhd("https://mystorage.blob.core.windows.net/vhds/myosdisk.vhd")
    .withSizeInGB(128)
    .withSku(DiskSkuTypes.PremiumLRS)
    .create();

azure.virtualMachines.define("myVM")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetworkInterface(networkInterface)
    .withSpecializedOSDisk(managedDisk, OperatingSystemTypes.Windows)
    .withExistingAvailabilitySet(availabilitySet)
    .withSize(VirtualMachineSizeTypes.StandardDS1)
    .create();
```

## <a name="perform-management-tasks"></a>Выполнение задач управления.

В течение жизненного цикла виртуальной машины можно выполнять задачи управления, такие как запуск, остановка или удаление виртуальной машины. Кроме того, можно создавать код для автоматизации повторяющихся или сложных задач.

Если необходимо выполнить любые действия с виртуальной машиной, необходимо получить ее экземпляр. Добавьте следующий код в блок Try метода Main:

```java
VirtualMachine vm = azure.virtualMachines().getByResourceGroup("myResourceGroup", "myVM");
```

### <a name="get-information-about-the-vm"></a>Получение информации о виртуальной машине

Чтобы получить сведения о виртуальной машине, добавьте следующий код в блок Try метода Main:

```java
System.out.println("hardwareProfile");
System.out.println("    vmSize: " + vm.size());
System.out.println("storageProfile");
System.out.println("  imageReference");
System.out.println("    publisher: " + vm.storageProfile().imageReference().publisher());
System.out.println("    offer: " + vm.storageProfile().imageReference().offer());
System.out.println("    sku: " + vm.storageProfile().imageReference().sku());
System.out.println("    version: " + vm.storageProfile().imageReference().version());
System.out.println("  osDisk");
System.out.println("    osType: " + vm.storageProfile().osDisk().osType());
System.out.println("    name: " + vm.storageProfile().osDisk().name());
System.out.println("    createOption: " + vm.storageProfile().osDisk().createOption());
System.out.println("    caching: " + vm.storageProfile().osDisk().caching());
System.out.println("osProfile");
System.out.println("    computerName: " + vm.osProfile().computerName());
System.out.println("    adminUserName: " + vm.osProfile().adminUsername());
System.out.println("    provisionVMAgent: " + vm.osProfile().windowsConfiguration().provisionVMAgent());
System.out.println("    enableAutomaticUpdates: " + vm.osProfile().windowsConfiguration().enableAutomaticUpdates());
System.out.println("networkProfile");
System.out.println("    networkInterface: " + vm.primaryNetworkInterfaceId());
System.out.println("vmAgent");
System.out.println("  vmAgentVersion: " + vm.instanceView().vmAgent().vmAgentVersion());
System.out.println("    statuses");
for(InstanceViewStatus status : vm.instanceView().vmAgent().statuses()) {
    System.out.println("    code: " + status.code());
    System.out.println("    displayStatus: " + status.displayStatus());
    System.out.println("    message: " + status.message());
    System.out.println("    time: " + status.time());
}
System.out.println("disks");
for(DiskInstanceView disk : vm.instanceView().disks()) {
    System.out.println("  name: " + disk.name());
    System.out.println("  statuses");
    for(InstanceViewStatus status : disk.statuses()) {
        System.out.println("    code: " + status.code());
        System.out.println("    displayStatus: " + status.displayStatus());
        System.out.println("    time: " + status.time());
    }
}
System.out.println("VM general status");
System.out.println("  provisioningStatus: " + vm.provisioningState());
System.out.println("  id: " + vm.id());
System.out.println("  name: " + vm.name());
System.out.println("  type: " + vm.type());
System.out.println("VM instance status");
for(InstanceViewStatus status : vm.instanceView().statuses()) {
    System.out.println("  code: " + status.code());
    System.out.println("  displayStatus: " + status.displayStatus());
}
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="stop-the-vm"></a>Остановка виртуальной машины

Вы можете остановить виртуальную машину, сохранив все ее настройки (при этом за нее будет продолжать взиматься плата) или остановить ее и отменить ее распределение. При этом освобождаются все ресурсы, связанные с ней, а также прекращается выставление счетов за эту виртуальную машину.

Чтобы остановить виртуальную машину без ее освобождения, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Stopping vm...");
vm.powerOff();
System.out.println("Press enter to continue...");
input.nextLine();
```

Если вы хотите отменить распределение виртуальной машины, измените вызов PowerOff на следующий код:

```java
vm.deallocate();
```

### <a name="start-the-vm"></a>Запуск виртуальной машины

Чтобы запустить виртуальную машину, добавьте следующий код в блок Try метода Main:

```java
System.out.println("Starting vm...");
vm.start();
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="resize-the-vm"></a>Изменение размера виртуальной машины

При выборе размера виртуальной машины необходимо учесть многие аспекты развертывания. Дополнительные сведения см. в статье [Размеры виртуальных машин Windows в Azure](../sizes.md).  

Чтобы изменить размер виртуальной машины, добавьте следующий код в блок Try метода Main :

```java
System.out.println("Resizing vm...");
vm.update()
    .withSize(VirtualMachineSizeTypes.STANDARD_DS2)
    .apply();
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="add-a-data-disk-to-the-vm"></a>Добавление диска данных в виртуальную машину

Чтобы добавить диск данных в виртуальную машину (при этом будет добавлен диск данных размером 2 ГБ, имеющий логический номер устройства (LUN) 0 и тип кэширования ReadWrite), добавьте следующий код в блок Try метода Main:

```java
System.out.println("Adding data disk...");
vm.update()
    .withNewDataDisk(2, 0, CachingTypes.READ_WRITE)
    .apply();
System.out.println("Press enter to delete resources...");
input.nextLine();
```

## <a name="delete-resources"></a>Удаление ресурсов

Так как за использование ресурсов Azure взимается плата, рекомендуется всегда удалять ресурсы, которые больше не нужны. Если вы хотите удалить виртуальные машины и все вспомогательные ресурсы, достаточно удалить группу ресурсов.

1. Чтобы удалить группу ресурсов, добавьте следующий код в блок Try метода Main:
   
    ```java
    System.out.println("Deleting resources...");
    azure.resourceGroups().deleteByName("myResourceGroup");
    ```

2. Сохраните файл App.java.

## <a name="run-the-application"></a>Выполнение приложения

На полное выполнение этого консольного приложения потребуется примерно 5 минут.

1. Чтобы запустить приложение, используйте следующую команду Maven:

    ```
    mvn compile exec:java
    ```

2. Прежде чем нажать клавишу **ВВОД** и начать удаление ресурсов, потратьте несколько минут и проверьте на портале Azure, созданы ли эти ресурсы. Щелкните состояние развертывания, чтобы просмотреть сведения о развертывании.


## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения об использовании библиотек Azure для Java см. в [этой статье](/java/azure/java-sdk-azure-overview).
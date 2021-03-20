---
title: Управление сертификатами в кластере Azure Service Fabric
description: В этой статье рассказывается о том, как добавить новые, а также сменить или удалить имеющиеся сертификаты в кластере Service Fabric.
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: 6dd4440d76bed9d110c13baab9f4e67b3a5c64c0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94660922"
---
# <a name="add-or-remove-certificates-for-a-service-fabric-cluster-in-azure"></a>Добавление и удаление сертификатов для кластера Service Fabric в Azure
Рекомендуется ознакомиться с тем, как Service Fabric использует сертификаты X.509, и просмотреть раздел [Сценарии защиты кластера Service Fabric](service-fabric-cluster-security.md). Необходимо понять, что такое сертификат кластера и для чего он используется, прежде чем продолжить.

По умолчанию режим загрузки сертификата пакета SDK для Azure Service Fabrics заключается в развертывании и использовании определенного сертификата с датой срока истечения далеко в будущем, независимо от их первичного или вторичного определения конфигурации. Возвращение к классическому режиму работы — это не рекомендованное расширенное действие, которое потребует установить для параметра "UseSecondaryIfNever" значение "false" в конфигурации `Fabric.Code`.

Service Fabric позволяет указать в дополнение к сертификатам клиента два сертификата кластера, основной и дополнительный. Сделать это можно при настройке сертификата безопасности во время создания кластера. Чтобы узнать больше об их настройке во время создания, ознакомьтесь с процедурой [создания кластера Azure с помощью портала](service-fabric-cluster-creation-via-portal.md) или [создания кластера Azure с помощью Azure Resource Manager](service-fabric-cluster-creation-via-arm.md). Если во время создания указан только один сертификат кластера, то он используется в качестве основного сертификата. После создания кластера можно добавить новый сертификат в качестве дополнительного.

> [!NOTE]
> Для работы с безопасным кластером всегда требуется хотя бы один действительный (не отозванный и не просроченный) развернутый сертификат кластера (основной или дополнительный). В противном случае кластер не будет функционировать. 90 дней до истечения срока действия всех действительных сертификатов система создает трассировку предупреждения и событие работоспособности предупреждения на узле. В настоящее время это только уведомления, Service Fabric отправляется в отношении истечения срока действия сертификата.
> 
> 


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="add-a-secondary-cluster-certificate-using-the-portal"></a>Добавление дополнительного сертификата кластера с помощью портала
Невозможно добавить дополнительный сертификат кластера с помощью портал Azure; Используйте Azure PowerShell. Данная процедура рассмотрена далее в этом документе.

## <a name="remove-a-cluster-certificate-using-the-portal"></a>Удаление сертификата кластера с помощью портала
Для защиты кластера всегда требуется хотя бы один действительный (не отозванный и не просроченный) сертификат. Поэтому будет использоваться сертификат с самым продолжительным сроком действия, а его удаление заставит кластер остановить работу. Убедитесь, что вы удалили только сертификат, срок действия которого истек, или неиспользуемый сертификат, срок действия которого истекает.

Чтобы удалить неиспользуемый сертификат безопасности кластера, перейдите в раздел "Безопасность" и выберите "Удалить" в контекстном меню неиспользуемого сертификата.

Если вы намерены удалить сертификат, который помечен как основной, потребуется развернуть вторичный сертификат с датой истечения срока действия позднее, чем основной сертификат, включение которого автоматически сменит режим работы. Удалите основной сертификат после завершения автоматической смены режима работы.

## <a name="add-a-secondary-certificate-using-azure-resource-manager"></a>Добавление дополнительного сертификата с помощью Azure Resource Manager

В этих инструкциях предполагается, что у вас есть опыт работы с Resource Manager, вы развернули хотя бы один кластер Service Fabric с помощью шаблона Resource Manager и у вас есть шаблон, который использовался для настройки этого кластера. Предполагается также, что вы уверенно используете JSON.

> [!NOTE]
> Если вам нужен пример шаблона и параметров, которые можно использовать в качестве образца или отправной точки, скачайте его из этого репозитория [git-repo](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Cert-Rollover-Sample). 
> 
> 

### <a name="edit-your-resource-manager-template"></a>Изменение шаблона Resource Manager

Для удобства выполнения инструкций пример 5-VM-1-NodeTypes-Secure_Step2.JSON содержит все изменения, которые мы будем вносить. Этот пример доступен в репозитории [git-repo](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Cert-Rollover-Sample).

**Обязательные шаги**

1. Откройте шаблон Resource Manager, который использовался для развертывания кластера. (Если вы скачали пример из указанного выше репозитория, воспользуйтесь файлом 5-VM-1-NodeTypes-Secure_Step1.JSON для развертывания защищенного кластера, а затем откройте этот шаблон.)

2. Добавьте в раздел параметров шаблона **два новых параметра** строкового типа (secCertificateThumbprint и secCertificateUrlValue). Можно скопировать следующий фрагмент кода и добавить его в шаблон. В зависимости от источника шаблона, эти параметры уже могут быть определены. В таком случае перейдите к следующему шагу. 
 
    ```json
       "secCertificateThumbprint": {
          "type": "string",
          "metadata": {
            "description": "Certificate Thumbprint"
          }
        },
        "secCertificateUrlValue": {
          "type": "string",
          "metadata": {
            "description": "Refers to the location URL in your key vault where the certificate was uploaded, it is should be in the format of https://<name of the vault>.vault.azure.net:443/secrets/<exact location>"
          }
        },
    
    ```

3. Внесите изменения в ресурс **Microsoft.ServiceFabric/clusters**. Для этого найдите определение ресурса Microsoft.ServiceFabric/clusters в шаблоне. В разделе свойств этого определения вы увидите тег Certificate в формате JSON, который должен выглядеть как приведенный ниже фрагмент JSON.
   
    ```JSON
          "properties": {
            "certificate": {
              "thumbprint": "[parameters('certificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
         }
    ``` 

    Добавьте новый тег thumbprintSecondary и присвойте ему значение [parameters('secCertificateThumbprint')].  

    Теперь определение ресурса должно выглядеть следующим образом (в зависимости от источника шаблона оно может отличаться от приведенного ниже фрагмента). 

    ```JSON
          "properties": {
            "certificate": {
              "thumbprint": "[parameters('certificateThumbprint')]",
              "thumbprintSecondary": "[parameters('secCertificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
         }
    ``` 

    Если необходимо **сменить сертификат**, укажите новый сертификат в качестве основного, а текущий основной сертификат назначьте на роль дополнительного. В результате текущий основной сертификат заменится новым сертификатом за один шаг развертывания.
    
    ```JSON
          "properties": {
            "certificate": {
              "thumbprint": "[parameters('secCertificateThumbprint')]",
              "thumbprintSecondary": "[parameters('certificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
         }
    ``` 

4. Внесите изменения во **все** определения ресурсов **Microsoft.Compute/virtualMachineScaleSets**. Для этого найдите определение ресурса Microsoft.Compute/virtualMachineScaleSets. Прокрутите до подраздела "publisher": "Microsoft.Azure.ServiceFabric" в разделе "virtualMachineProfile".

    В списке параметров издателя Service Fabric должно отображаться примерно следующее.
    
    ![Json_Pub_Setting1][Json_Pub_Setting1]
    
    Добавьте к ним новый сертификат.
    
    ```json
                   "certificateSecondary": {
                        "thumbprint": "[parameters('secCertificateThumbprint')]",
                        "x509StoreName": "[parameters('certificateStoreValue')]"
                        }
                      },
    
    ```

    Свойства должны выглядеть следующим образом:
    
    ![Json_Pub_Setting2][Json_Pub_Setting2]
    
    Если необходимо **сменить сертификат**, укажите новый сертификат в качестве основного, а текущий основной сертификат назначьте на роль дополнительного. В результате текущий сертификат заменяется новым сертификатом за один шаг развертывания.     

    ```json
                   "certificate": {
                       "thumbprint": "[parameters('secCertificateThumbprint')]",
                       "x509StoreName": "[parameters('certificateStoreValue')]"
                         },
                   "certificateSecondary": {
                        "thumbprint": "[parameters('certificateThumbprint')]",
                        "x509StoreName": "[parameters('certificateStoreValue')]"
                        }
                      },
    ```

    Свойства должны выглядеть следующим образом:    
    ![Json_Pub_Setting3][Json_Pub_Setting3]

5. Внесите изменения во **все** определения ресурсов **Microsoft.Compute/virtualMachineScaleSets**. Для этого найдите определение ресурса Microsoft.Compute/virtualMachineScaleSets. Перейдите к подразделу "vaultCertificates": в разделе "OSProfile". Должно отобразиться примерно следующее.

    ![Json_Pub_Setting4][Json_Pub_Setting4]
    
    Добавьте к нему secCertificateUrlValue. Воспользуйтесь следующим фрагментом кода:
    
    ```json
                      {
                        "certificateStore": "[parameters('certificateStoreValue')]",
                        "certificateUrl": "[parameters('secCertificateUrlValue')]"
                      }
    
    ```
    Теперь окончательная версия кода JSON должна выглядеть следующим образом:
    ![Json_Pub_Setting5][Json_Pub_Setting5]


> [!NOTE]
> Обязательно повторите шаги 4 и 5 для всех определений ресурса Nodetypes/Microsoft.Compute/virtualMachineScaleSets в шаблоне. Если пропустить хотя бы одно из них, сертификат не будет установлен в этот масштабируемый набор виртуальных машин, что может привести к непредсказуемым результатам в кластере, в том числе выходу кластера из строя (если у вас не будет действительного сертификата для обеспечения безопасности кластера). Поэтому проверьте все еще раз, прежде чем продолжать.
> 
> 

### <a name="edit-your-template-file-to-reflect-the-new-parameters-you-added-above"></a>Изменение файла шаблона с учетом новых параметров, добавленных выше
Если далее вы будете использовать пример из репозитория [git-repo](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Cert-Rollover-Sample), можете начать вносить изменения в пример 5-VM-1-NodeTypes-Secure_Step2.JSON. 

Измените параметр File в шаблоне Resource Manager. Добавьте два новых параметра secCertificateThumbprint и secCertificateUrlValue. 

```JSON
    "secCertificateThumbprint": {
      "value": "thumbprint value"
    },
    "secCertificateUrlValue": {
      "value": "Refers to the location URL in your key vault where the certificate was uploaded, it is should be in the format of https://<name of the vault>.vault.azure.net:443/secrets/<exact location>"
     },

```

### <a name="deploy-the-template-to-azure"></a>Развертывание шаблона в Azure

- Теперь все готово к развертыванию шаблона в Azure. Откройте командную строку Azure PowerShell версии не ниже 1.
- Войдите со своей учетной записью Azure и выберите подписку Azure. Это важный шаг для пользователей, имеющих доступ к нескольким подпискам Azure.

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionId <Subscription ID> 

```

Проверьте шаблон перед его развертыванием. Используйте группу ресурсов, в которой развернут кластер.

```powershell
Test-AzResourceGroupDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>

```

Разверните шаблон в группу ресурсов. Используйте группу ресурсов, в которой развернут кластер. Выполните команду New-AzResourceGroupDeployment. Не нужно указывать режим, так как по умолчанию используется **добавочный** режим.

> [!NOTE]
> Если задан полный режим, то вы можете случайно удалить ресурсы, которые находятся не в шаблоне. Поэтому не используйте этот режим данном сценарии.
> 
> 

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>
```

Вот пример того же PowerShell.

```powershell
$ResourceGroup2 = "chackosecure5"
$TemplateFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure_Step2.json"
$TemplateParmFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure.parameters_Step2.json"

New-AzResourceGroupDeployment -ResourceGroupName $ResourceGroup2 -TemplateParameterFile $TemplateParmFile -TemplateUri $TemplateFile -clusterName $ResourceGroup2

```

После завершения развертывания подключитесь к кластеру с помощью нового сертификата и выполните несколько запросов. Если получится это сделать. Затем можно удалить старый сертификат. 

Если используется самозаверяющий сертификат, то не забудьте импортировать его в локальное хранилище сертификатов TrustedPeople.

```powershell
######## Set up the certs on your local box
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)

```
Для краткой справки ознакомьтесь с командой для подключения к безопасному кластеру: 

```powershell
$ClusterName= "chackosecure5.westus.cloudapp.azure.com:19000"
$CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7SD1D630F8F3" 

Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $CertThumbprint  `
    -FindType FindByThumbprint `
    -FindValue $CertThumbprint `
    -StoreLocation CurrentUser `
    -StoreName My
```
Для краткой справки ознакомьтесь с командой для получения данных о работоспособности кластера:

```powershell
Get-ServiceFabricClusterHealth 
```

## <a name="deploying-client-certificates-to-the-cluster"></a>Разверните сертификаты клиента в кластере.

Для развертывания сертификатов из хранилища ключей в узлах можно выполнить те же действия, которые описаны на шаге 5 выше. Необходимо только определить и использовать другие параметры.


## <a name="adding-or-removing-client-certificates"></a>Добавление или удаление сертификатов клиента

В дополнение к сертификатам кластера можно добавить сертификаты клиента, которые позволят выполнять операции управления в кластере Service Fabric.

Вы можете добавлять сертификаты клиента двух типов: для администратора или только для чтения. Затем их можно использовать для управления доступом к операциям, выполняемым администратором, и операциям, связанным с запросами в кластере. По умолчанию сертификаты кластера добавляются в список разрешенных сертификатов администратора.

Можно указать любое количество клиентских сертификатов. Каждое Добавление или удаление приводит к обновлению конфигурации в кластере Service Fabric.


### <a name="adding-client-certificates---admin-or-read-only-via-portal"></a>Добавление сертификатов клиента для администратора или только для чтения с помощью портала

1. Перейдите к разделу "Безопасность" и выберите "+ Проверка подлинности" в его верхней части.
2. В разделе "Добавление проверки подлинности" выберите "Тип проверки подлинности" — Read-only client (Клиент с доступом только для чтения) или Admin client (Клиент с правами администратора).
3. Теперь выберите метод авторизации. указывает способ поиска сертификата в Service Fabric (по имени субъекта или по отпечатку). Как правило, по соображениям безопасности не рекомендуется использовать метод авторизации по имени субъекта. 

![Добавление сертификата клиента][Add_Client_Cert]

### <a name="deletion-of-client-certificates---admin-or-read-only-using-the-portal"></a>Удаление сертификатов клиентов для администратора или только для чтения с помощью портала

Чтобы дополнительный сертификат больше не использовался для обеспечения безопасности кластера, перейдите к разделу "Безопасность" и в контекстном меню соответствующего сертификата выберите параметр "Удалить".

## <a name="adding-application-certificates-to-a-virtual-machine-scale-set"></a>Добавление сертификатов приложений в масштабируемый набор виртуальных машин

Сведения о развертывании сертификата, используемого для приложений в кластере, см. в [этом образце сценария PowerShell](scripts/service-fabric-powershell-add-application-certificate.md).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения об управлении кластерами доступны в следующих статьях.

* [Обновление кластера Service Fabric](service-fabric-cluster-upgrade.md)
* [Настройка доступа на основе ролей для клиентов](service-fabric-cluster-security-roles.md)

<!--Image references-->
[Add_Client_Cert]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_13.PNG
[Json_Pub_Setting1]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_14.PNG
[Json_Pub_Setting2]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_15.PNG
[Json_Pub_Setting3]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_16.PNG
[Json_Pub_Setting4]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_17.PNG
[Json_Pub_Setting5]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_18.PNG



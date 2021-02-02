---
title: Развертывание облачной службы Azure (расширенная поддержка) — шаблоны
description: Развертывание облачной службы Azure (расширенная поддержка) с использованием шаблонов ARM
ms.topic: tutorial
ms.service: cloud-services-extended-support
author: gachandw
ms.author: gachandw
ms.reviewer: mimckitt
ms.date: 10/13/2020
ms.custom: ''
ms.openlocfilehash: f86b2a50040704aac2827c463a362a04f78ba34f
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/27/2021
ms.locfileid: "98881827"
---
# <a name="deploy-a-cloud-service-extended-support-using-arm-templates"></a>Развертывание облачной службы (расширенная поддержка) с использованием шаблонов ARM

В этом учебнике объясняется, как создать развертывание облачной службы (расширенная поддержка) с помощью [шаблонов ARM](https://docs.microsoft.com/azure/azure-resource-manager/templates/overview). 

> [!IMPORTANT]
> Облачные службы (расширенная поддержка) сейчас предоставляются в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="before-you-begin"></a>Подготовка к работе
1. Ознакомьтесь с [предварительными требованиями развертывания](deploy-prerequisite.md) для Облачных служб (расширенная поддержка) и создайте связанные ресурсы. 

2. Создайте новую группу ресурсов с помощью [портала Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal) или [PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-powershell). Этот шаг можно пропустить, если вы используете существующую группу ресурсов. 
 
3. Создайте учетную запись хранения с помощью [портала Azure](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-portal) или [PowerShell](https://docs.microsoft.com/azure/storage/common/storage-account-create?tabs=azure-powershell). Этот шаг можно пропустить, если вы используете существующую учетную запись хранения. 

4. Передайте CSDEF-файлы определения службы и CSCFG-файлы конфигурации службы в учетную запись хранения с помощью [портала Azure](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal#upload-a-block-blob), [AzCopy](https://docs.microsoft.com/azure/storage/common/storage-use-azcopy-blobs-upload?toc=/azure/storage/blobs/toc.json) или [PowerShell](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-powershell#upload-blobs-to-the-container). Получите подписанные URL-адреса обоих файлов, которые вы добавите в шаблон ARM далее в этом руководстве. 

5. (Необязательно) Создайте хранилище ключей и передайте в него сертификаты. 
    -  Сертификаты можно присоединять к облачным службам, чтобы обеспечить безопасное взаимодействие со службой. Для использования сертификатов необходимо, чтобы их отпечатки были указаны в CSCFG-файле конфигурации службы и переданы в хранилище ключей. Хранилище ключей можно создать с помощью [портала Azure](https://docs.microsoft.com/azure/key-vault/general/quick-create-portal) или [PowerShell](https://docs.microsoft.com/azure/key-vault/general/quick-create-powershell). 
    - Связанное с облачной службой хранилище ключей должно находиться в том же регионе и подписке, что и эта служба.   
    - Для связанного хранилища ключей необходимо включить разрешения, позволяющие ресурсу Облачных служб (расширенная поддержка) получить сертификат из хранилища ключей. Дополнительные сведения см. в статье [о сертификатах и хранилище ключей](certificates-and-key-vault.md).
    - Ссылка на хранилище ключей должна быть включена в раздел OsProfile шаблона ARM, к которому мы перейдем ниже.

## <a name="deploy-a-cloud-service-extended-support"></a>Развертывание облачной службы (расширенная поддержка) 
1. Создайте виртуальную сеть. Имя виртуальной сети должно соответствовать ссылкам в CSCFG-файле конфигурации службы. Если используется существующая виртуальная сеть, не включайте этот раздел в шаблон ARM.

    ```json
    "resources": [ 
        { 
          "apiVersion": "2019-08-01", 
          "type": "Microsoft.Network/virtualNetworks", 
          "name": "[parameters('vnetName')]", 
          "location": "[parameters('location')]", 
          "properties": { 
            "addressSpace": { 
              "addressPrefixes": [ 
                "10.0.0.0/16" 
              ] 
            }, 
            "subnets": [ 
              { 
                "name": "WebTier", 
                "properties": { 
                  "addressPrefix": "10.0.0.0/24" 
                } 
              } 
            ] 
          } 
        } 
    ] 
    ```
    
     Если вы создаете новую виртуальную сеть, добавьте следующий код в раздел `dependsOn`, чтобы платформа обязательно создала виртуальную сеть перед созданием облачной службы. 

    ```json
    "dependsOn": [ 
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]" 
     ] 
    ```
 
2. Создайте общедоступный IP-адрес и (необязательно) задайте свойство метки DNS для общедоступного IP-адреса. Если вы используете статический IP-адрес, он должен быть указан как зарезервированный IP-адрес в CSCFG-файле конфигурации службы. Если используется существующий IP-адрес, пропустите этот шаг и добавьте сведения об IP-адресе непосредственно в параметры конфигурации балансировщика нагрузки в шаблоне ARM.
 
    ```json
    "resources": [ 
        { 
          "apiVersion": "2019-08-01", 
          "type": "Microsoft.Network/publicIPAddresses", 
          "name": "[parameters('publicIPName')]", 
          "location": "[parameters('location')]", 
          "properties": { 
            "publicIPAllocationMethod": "Dynamic", 
            "idleTimeoutInMinutes": 10, 
            "publicIPAddressVersion": "IPv4", 
            "dnsSettings": { 
              "domainNameLabel": "[variables('dnsName')]" 
            } 
          }, 
          "sku": { 
            "name": "Basic" 
          } 
        } 
    ] 
    ```
     
     Если вы создаете новый IP-адрес, добавьте следующий код в раздел `dependsOn`, чтобы платформа обязательно создала IP-адрес перед созданием облачной службы. 
    
    ```json
    "dependsOn": [ 
            "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]" 
          ] 
    ```
 
3. Создайте объект сетевого профиля и свяжите общедоступный IP-адрес с внешним интерфейсом подсистемы балансировки нагрузки. Подсистему балансировки нагрузки платформа создает автоматически.  

    ```json
    "networkProfile": { 
        "loadBalancerConfigurations": [ 
          { 
            "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/loadBalancers/', variables('lbName'))]", 
            "name": "[variables('lbName')]", 
            "properties": { 
              "frontendIPConfigurations": [ 
                { 
                  "name": "[variables('lbFEName')]", 
                  "properties": { 
                    "publicIPAddress": { 
                      "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]" 
                    } 
                  } 
                } 
              ] 
            } 
          } 
        ] 
      } 
    ```
 

4. Добавьте ссылку на хранилище ключей в раздел  `OsProfile` шаблона ARM. Хранилище ключей используется для хранения сертификатов, связанных с Облачными службами (расширенная поддержка). Добавьте сертификаты в хранилище ключей, а затем добавьте ссылки на отпечатки сертификатов в CSCFG-файл конфигурации службы. Также для хранилища ключей необходимо включить соответствующие разрешения, позволяющие ресурсу Облачных служб (расширенная поддержка) получить из хранилища ключей сертификат, хранимый в виде секрета. Дополнительные сведения см. в статье [Использование сертификатов с Облачными службами (расширенная поддержка)](certificates-and-key-vault.md).
     
    ```json
    "osProfile": { 
          "secrets": [ 
            { 
              "sourceVault": { 
                "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.KeyVault/vaults/{keyvault-name}" 
              }, 
              "vaultCertificates": [ 
                { 
                  "certificateUrl": "https://{keyvault-name}.vault.azure.net:443/secrets/ContosoCertificate/{secret-id}" 
                } 
              ] 
            } 
          ] 
        } 
    ```
  
    > [!NOTE]
    > SourceVault — это идентификатор ресурса ARM для хранилища ключей. Это значение можно получить, найдя идентификатор ресурса в разделе свойств хранилища ключей. 
    > - Чтобы получить certificateUrl, перейдите в хранилище ключей к сертификату с меткой **Secret Identifier** (Секретный идентификатор).  
   >  - certificateUrl имеет формат https://{конечная_точка_хранилища_ключей}/secrets/{имя_секрета}/{ИД_секрета}

5. Создайте профиль роли. Убедитесь, что количество ролей, имена ролей, число и размеры экземпляров в каждой роли одинаковы в CSCFG-файле конфигурации службы, CSDEF-файле определения службы и в разделе профиля роли в шаблоне ARM. 
    
    ```json
    "roleProfile": { 
          "roles": { 
          "value": [ 
            { 
              "name": "WebRole1", 
              "sku": { 
                "name": "Standard_D1_v2", 
                "capacity": "1" 
              } 
            }, 
            { 
              "name": "WorkerRole1", 
              "sku": { 
                "name": "Standard_D1_v2", 
                "capacity": "1" 
              } 
            } 
        }
    }   
    ```

6. (Необязательно) Создайте профиль расширения, чтобы добавить расширения в облачную службу. В этом примере мы добавляем расширения удаленного рабочего стола и Диагностики Windows Azure. 
    
    ```json
        "extensionProfile": {
              "extensions": [
                {
                  "name": "RDPExtension",
                  "properties": {
                    "autoUpgradeMinorVersion": true,
                    "publisher": "Microsoft.Windows.Azure.Extensions",
                    "type": "RDP",
                    "typeHandlerVersion": "1.2.1",
                    "settings": "<PublicConfig>\r\n <UserName>[Insert Username]</UserName>\r\n <Expiration>1/21/2022 12:00:00 AM</Expiration>\r\n</PublicConfig>",
                    "protectedSettings": "<PrivateConfig>\r\n <Password>[Insert Password]</Password>\r\n</PrivateConfig>"
                  }
                },
                {
                  "name": "Microsoft.Insights.VMDiagnosticsSettings_WebRole1",
                  "properties": {
                    "autoUpgradeMinorVersion": true,
                    "publisher": "Microsoft.Azure.Diagnostics",
                    "type": "PaaSDiagnostics",
                    "typeHandlerVersion": "1.5",
                    "settings": "[parameters('wadPublicConfig_WebRole1')]",
                    "protectedSettings": "[parameters('wadPrivateConfig_WebRole1')]",
                    "rolesAppliedTo": [
                      "WebRole1"
              ]
            }
          }
        ]
      }

  
    ```    

7. Проверьте весь шаблон. 

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "cloudServiceName": {
          "type": "string",
          "metadata": {
            "description": "Name of the cloud service"
          }
        },
        "location": {
          "type": "string",
          "metadata": {
            "description": "Location of the cloud service"
          }
        },
        "deploymentLabel": {
          "type": "string",
          "metadata": {
            "description": "Label of the deployment"
          }
        },
        "packageSasUri": {
          "type": "securestring",
          "metadata": {
            "description": "SAS Uri of the CSPKG file to deploy"
          }
        },
        "configurationSasUri": {
          "type": "securestring",
          "metadata": {
            "description": "SAS Uri of the service configuration (.cscfg)"
          }
        },
        "roles": {
          "type": "array",
          "metadata": {
            "description": "Roles created in the cloud service application"
          }
        },
        "wadPublicConfig_WebRole1": {
          "type": "string",
          "metadata": {
             "description": "Public configuration of Windows Azure Diagnostics extension"
          }
         },
        "wadPrivateConfig_WebRole1": {
          "type": "securestring",
          "metadata": {
            "description": "Private configuration of Windows Azure Diagnostics extension"
         }
        },
        "vnetName": {
          "type": "string",
          "defaultValue": "[concat(parameters('cloudServiceName'), 'VNet')]",
          "metadata": {
            "description": "Name of vitual network"
          }
        },
        "publicIPName": {
          "type": "string",
          "defaultValue": "contosocsIP",
          "metadata": {
            "description": "Name of public IP address"
          }
        },
        "upgradeMode": {
          "type": "string",
          "defaultValue": "Auto",
          "metadata": {
            "UpgradeMode": "UpgradeMode of the CloudService"
          }
        }
      },
      "variables": {
        "cloudServiceName": "[parameters('cloudServiceName')]",
        "subscriptionID": "[subscription().subscriptionId]",
        "dnsName": "[variables('cloudServiceName')]",
        "lbName": "[concat(variables('cloudServiceName'), 'LB')]",
        "lbFEName": "[concat(variables('cloudServiceName'), 'LBFE')]",
        "resourcePrefix": "[concat('/subscriptions/', variables('subscriptionID'), '/resourceGroups/', resourceGroup().name, '/providers/')]"
      },
      "resources": [
        {
          "apiVersion": "2019-08-01",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "[parameters('vnetName')]",
          "location": "[parameters('location')]",
          "properties": {
            "addressSpace": {
              "addressPrefixes": [
                "10.0.0.0/16"
              ]
            },
            "subnets": [
              {
                "name": "WebTier",
                "properties": {
                  "addressPrefix": "10.0.0.0/24"
                }
              }
            ]
          }
        },
        {
          "apiVersion": "2019-08-01",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "[parameters('publicIPName')]",
          "location": "[parameters('location')]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "idleTimeoutInMinutes": 10,
            "publicIPAddressVersion": "IPv4",
            "dnsSettings": {
              "domainNameLabel": "[variables('dnsName')]"
            }
          },
          "sku": {
            "name": "Basic"
          }
        },
        {
          "apiVersion": "2020-10-01-preview",
          "type": "Microsoft.Compute/cloudServices",
          "name": "[variables('cloudServiceName')]",
          "location": "[parameters('location')]",
          "tags": {
            "DeploymentLabel": "[parameters('deploymentLabel')]",
            "DeployFromVisualStudio": "true"
          },
          "dependsOn": [
            "[concat('Microsoft.Network/virtualNetworks/', parameters('vnetName'))]",
            "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]"
          ],
          "properties": {
            "packageUrl": "[parameters('packageSasUri')]",
            "configurationUrl": "[parameters('configurationSasUri')]",
            "upgradeMode": "[parameters('upgradeMode')]",
            "roleProfile": {
              "roles": [
                {
                  "name": "WebRole1",
                  "sku": {
                    "name": "Standard_D1_v2",
                    "capacity": "1"
                  }
                },
                {
                  "name": "WorkerRole1",
                  "sku": {
                    "name": "Standard_D1_v2",
                    "capacity": "1"
                  }
                }
              ]
            },
            "networkProfile": {
              "loadBalancerConfigurations": [
                {
                  "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/loadBalancers/', variables('lbName'))]",
                  "name": "[variables('lbName')]",
                  "properties": {
                    "frontendIPConfigurations": [
                      {
                        "name": "[variables('lbFEName')]",
                        "properties": {
                          "publicIPAddress": {
                            "id": "[concat(variables('resourcePrefix'), 'Microsoft.Network/publicIPAddresses/', parameters('publicIPName'))]"
                          }
                        }
                      }
                    ]
                  }
                }
              ]
            },
            "osProfile": {
              "secrets": [
                {
                  "sourceVault": {
                    "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.KeyVault/vaults/{keyvault-name}"
                  },
                  "vaultCertificates": [
                    {
                      "certificateUrl": "https://{keyvault-name}.vault.azure.net:443/secrets/ContosoCertificate/{secret-id}"
                    }
                  ]
                }
              ]
            },
        "extensionProfile": {
              "extensions": [
                {
                  "name": "RDPExtension",
                  "properties": {
                    "autoUpgradeMinorVersion": true,
                    "publisher": "Microsoft.Windows.Azure.Extensions",
                    "type": "RDP",
                    "typeHandlerVersion": "1.2.1",
                    "settings": "<PublicConfig>\r\n <UserName>[Insert Username]</UserName>\r\n <Expiration>1/21/2022 12:00:00 AM</Expiration>\r\n</PublicConfig>",
                    "protectedSettings": "<PrivateConfig>\r\n <Password>[Insert Password]</Password>\r\n</PrivateConfig>"
                  }
                },
                {
                  "name": "Microsoft.Insights.VMDiagnosticsSettings_WebRole1",
                  "properties": {
                    "autoUpgradeMinorVersion": true,
                    "publisher": "Microsoft.Azure.Diagnostics",
                    "type": "PaaSDiagnostics",
                    "typeHandlerVersion": "1.5",
                    "settings": "[parameters('wadPublicConfig_WebRole1')]",
                    "protectedSettings": "[parameters('wadPrivateConfig_WebRole1')]",
                    "rolesAppliedTo": [
                      "WebRole1"
                  ]
                }
              }
            ]
          }
        }
      }
    }
    ```
 
8. Разверните этот шаблон и создайте развертывание облачной службы (расширенная поддержка). 

    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName “ContosOrg -TemplateFile "file path to your template file”  
    ```
 
## <a name="next-steps"></a>Дальнейшие действия 
- Просмотрите [часто задаваемые вопросы](faq.md) об Облачных службах (расширенная поддержка)
- Разверните облачную службу (расширенная поддержка) с помощью [портала Azure](deploy-portal.md), [PowerShell](deploy-powershell.md), [шаблона](deploy-template.md) или [Visual Studio](deploy-visual-studio.md).
- Перейдите в [репозиторий примеров для Облачных служб (расширенная поддержка) ](https://github.com/Azure-Samples/cloud-services-extended-support)
---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 09/03/2020
ms.author: msmbaldwin
ms.openlocfilehash: 59359d37fe347c8568c7944f75accdbc04cddb93
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105967133"
---
1. Для создания группы ресурсов используйте команду `az group create`:

    ```azurecli
    az group create --name KeyVault-PythonQS-rg --location eastus
    ```

    При необходимости вы можете изменить расположение eastus на ближайшее к вам.

1. Для создания хранилища ключей используйте `az keyvault create`:

    ```azurecli
    az keyvault create --name <your-unique-keyvault-name> --resource-group KeyVault-PythonQS-rg
    ```

    Замените `<your-unique-keyvault-name>` именем, уникальным в пределах Azure. Обычно используется личное имя или название организации, а также числа и идентификаторы. 

1. Создайте переменную среды, которая предоставляет имя Key Vault коду:

    ```bash
    export KEY_VAULT_NAME=<your-unique-keyvault-name>
    ```
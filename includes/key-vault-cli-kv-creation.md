---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: a87fbbb276fd8813492cc293bc08b958ee3d6b7b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99070288"
---
Чтобы создать хранилище ключей в группе ресурсов из предыдущего шага, используйте команду [az keyvault create](/cli/azure/keyvault#az_keyvault_create) в Azure CLI. Необходимо будет указать следующие сведения:

- Имя хранилища ключей: строка длиной от 3 до 24 знаков, которая может содержать только цифры (0–9), буквы (a-z, A-Z) и дефисы (-).

  > [!Important]
  > Каждое хранилище ключей должно иметь уникальное имя. В следующих примерах замените <your-unique-keyvault-name> именем своего хранилища ключей.

- Имя группы ресурсов: **myResourceGroup**.
- Расположение: **EastUS**.

```azurecli
az keyvault create --name "<your-unique-keyvault-name>" --resource-group "myResourceGroup" --location "EastUS"
```

В выходных данных команды будут показаны свойства созданного хранилища ключей. Запишите значения двух указанных ниже свойств.

- **Имя хранилища.** Имя, которое вы указали в параметре --name выше.
- **URI хранилища**. В нашем примере это https://&lt;your-unique-keyvault-name&gt;.vault.vault.azure.net. Необходимо, чтобы приложения, использующие ваше хранилище через REST API, использовали этот URI.

На этом этапе любые операции в этом хранилище ключей может выполнять только учетная запись Azure.

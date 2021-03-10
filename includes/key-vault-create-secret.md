---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: d1a67a131a94bc017eaf2199546ef08f0291cd54
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102244622"
---
Мы создадим секрет с именем **mySecret** и значением **Success!** . Секретом может быть пароль, строка подключения SQL или другие сведения, которые должны быть защищены и при этом доступны приложению. 

Чтобы добавить секрет в созданное хранилище ключей, используйте команду Azure CLI [az keyvault secret set](/cli/azure/keyvault/secret#az-keyvault-secret-set):

```azurecli
az keyvault secret set --vault-name "<your-unique-keyvault-name>" --name "mySecret" --value "Success!"
```
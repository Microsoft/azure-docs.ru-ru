---
author: msmbaldwin
ms.service: key-vault
ms.topic: include
ms.date: 07/20/2020
ms.author: msmbaldwin
ms.openlocfilehash: d19c656946817b06cd620d8a48073bed8299af7d
ms.sourcegitcommit: 3b5cb7fb84a427aee5b15fb96b89ec213a6536c2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107502268"
---
Чтобы создать хранилище ключей в группе ресурсов из предыдущего шага, используйте командлет [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) в Azure PowerShell. Необходимо будет указать следующие сведения:

- Имя хранилища ключей: строка длиной от 3 до 24 знаков, которая может содержать только цифры (0–9), буквы (a-z, A-Z) и дефисы (-).

  > [!Important]
  > Каждое хранилище ключей должно иметь уникальное имя. В следующих примерах замените <your-unique-keyvault-name> именем своего хранилища ключей.

- Имя группы ресурсов: **myResourceGroup**.
- Расположение: **EastUS**.

```azurepowershell-interactive
New-AzKeyVault -Name "<your-unique-keyvault-name>" -ResourceGroupName "myResourceGroup" -Location "East US"
```

В выходных данных командлета будут показаны свойства созданного Key Vault. Запишите значения двух указанных ниже свойств.

- **Имя хранилища.** Имя, которое вы указали в параметре --name выше.
- **URI хранилища**. В нашем примере это https://&lt;your-unique-keyvault-name&gt;.vault.vault.azure.net. Необходимо, чтобы приложения, использующие ваше хранилище через REST API, использовали этот URI.

На этом этапе любые операции в этом хранилище ключей может выполнять только учетная запись Azure.

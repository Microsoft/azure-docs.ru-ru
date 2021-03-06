---
title: Создание ключей SSH в портал Azure
description: Узнайте, как создавать и хранить ключи SSH в портал Azure подключения к виртуальным машинам Linux.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: article
ms.date: 08/25/2020
ms.author: cynthn
ms.openlocfilehash: abc9a2ae130d987c90ce87ffaecbf2bb44b06010
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88929450"
---
# <a name="generate-and-store-ssh-keys-in-the-azure-portal"></a>Создание и хранение ключей SSH в портал Azure

Если вы часто используете портал для развертывания виртуальных машин Linux, можно упростить использование ключей SSH, создав их непосредственно на портале или загружая их с компьютера.

Вы можете создать ключи SSH при первом создании виртуальной машины и повторно использовать их для других виртуальных машин. Кроме того, вы можете создавать ключи SSH отдельно, чтобы иметь набор ключей, хранящихся в Azure, в соответствии с потребностями Организации. 

Если у вас есть ключи и вы хотите упростить их использование на портале, вы можете отправить их и сохранить в Azure для повторного использования.

Более подробные сведения о создании и использовании ключей SSH с виртуальными машинами Linux см. [в статье Использование ключей SSH для подключения к виртуальным машинам Linux](./linux/ssh-from-windows.md).

## <a name="generate-new-keys"></a>Создание новых ключей

1. Откройте [портал Azure](https://portal.azure.com).

1. В верхней части страницы введите *SSH* для поиска. В разделе **Marketplace** выберите **ключи SSH**.

1. На странице **ключ SSH** выберите **создать**.

   :::image type="content" source="./media/ssh-keys/portal-sshkey.png" alt-text="Создание новой группы ресурсов и создание пары ключей SSH":::

1. В **группе ресурсов** выберите **создать** , чтобы создать новую группу ресурсов для хранения ключей. Введите имя группы ресурсов и нажмите кнопку **ОК**.

1. В **области регион** выберите регион для хранения ключей. Вы можете использовать ключи в любом регионе, это только область, где они будут храниться.

1. Введите имя ключа в качестве **имени пары ключей**.

1. В разделе " **источник открытого ключа SSH**" выберите **создать источник открытого ключа**. 

1. Когда все будет готово, нажмите кнопку **Просмотр и создание**.

1. После прохождения проверки щелкните **Создать**.

1. Затем вы получите всплывающее окно, выберите **скачать закрытый ключ и создать ресурс**. При этом ключ SSH будет скачан как PEM-файл.

   :::image type="content" source="./media/ssh-keys/download-key.png" alt-text="Скачивание закрытого ключа в виде PEM-файла":::

1. После скачивания PEM-файла его можно переместить в другое место на компьютере, где его легко указать из клиента SSH.


## <a name="connect-to-the-vm"></a>Подключение к виртуальной машине

На локальном компьютере откройте командную строку PowerShell и введите следующую команду:

```powershell
ssh -i <path to the .pem file> username@<ipaddress of the VM>
```

Например, введите: `ssh -i /Downloads/mySSHKey.pem azureuser@123.45.67.890`


## <a name="upload-an-ssh-key"></a>Отправка ключа SSH

Вы также можете отправить открытый ключ SSH для хранения в Azure. Сведения о том, как создать пару ключей SSH, см. в статье [Использование ключей SSH для подключения к виртуальным машинам Linux](./linux/ssh-from-windows.md).

1. Откройте [портал Azure](https://portal.azure.com).

1. В верхней части страницы введите *SSH* для поиска. В разделе **Marketplace* выберите **ключи SSH**.

1. На странице **ключ SSH** выберите **создать**.

   :::image type="content" source="./media/ssh-keys/upload.png" alt-text="Отправка открытого ключа SSH для сохранения в Azure":::

1. В **группе ресурсов** выберите **создать** , чтобы создать новую группу ресурсов для хранения ключей. Введите имя группы ресурсов и нажмите кнопку **ОК**.

1. В **области регион** выберите регион для хранения ключей. Вы можете использовать ключи в любом регионе, это только область, где они будут храниться.

1. Введите имя ключа в качестве **имени пары ключей**.

1. В разделе " **источник открытого ключа SSH**" выберите **Отправить существующий открытый ключ**. 

1. Вставьте полное содержимое открытого ключа в **раздел upload** , а затем выберите проверить и **создать**.

1. После завершения проверки щелкните **Создать**. 

После отправки ключа его можно использовать при создании виртуальной машины.

## <a name="list-keys"></a>получение списка ключей;

Ключи SSH, созданные на портале, хранятся в виде ресурсов, поэтому можно отфильтровать представление ресурсов, чтобы увидеть их все.

1. На портале выберите **все ресурсы**.
1. В фильтрах выберите **тип**, снимите флажок **выбрать все** , чтобы очистить список.
1. Введите **SSH** в фильтре и выберите **ключ SSH**.

   :::image type="content" source="./media/ssh-keys/filter.png" alt-text="Снимок экрана, посвященный фильтрации списка для просмотра всех ключей SSH.":::

## <a name="get-the-public-key"></a>Получение открытого ключа

Если вам нужен открытый ключ, вы можете легко скопировать его со страницы портала для этого ключа. Просто перечислите ключи (используя процесс в последнем разделе), а затем выберите ключ из списка. Откроется страница ключа, и можно щелкнуть значок **Копировать в буфер обмена** рядом с ключом, чтобы скопировать его.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании ключей SSH с виртуальными машинами Azure см. в статье [Использование ключей SSH для подключения к виртуальным машинам Linux](./linux/ssh-from-windows.md).

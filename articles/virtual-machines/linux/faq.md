---
title: Часто задаваемые вопросы о виртуальных машинах Linux в Azure
description: В этой статье содержатся ответы на некоторые распространенные вопросы о виртуальных машинах Linux, созданных с помощью модели Resource Manager.
author: cynthn
ms.service: virtual-machines
ms.collection: linux
ms.workload: infrastructure-services
ms.topic: conceptual
ms.date: 05/08/2019
ms.author: cynthn
ms.openlocfilehash: 22be45403a7863328c5f6f2c883886296b734914
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102549096"
---
# <a name="frequently-asked-question-about-linux-virtual-machines"></a>Часто задаваемые вопросы по виртуальным машинам Linux
В этой статье содержатся ответы на некоторые распространенные вопросы о виртуальных машинах Linux, созданных в Azure посредством модели развертывания с помощью Resource Manager. Версия этой статьи для Windows — [Часто задаваемые вопросы по виртуальным машинам Windows](../windows/faq.md).

## <a name="what-can-i-run-on-an-azure-vm"></a>Что можно запускать на виртуальной машине Azure?
Все подписчики могут запускать на виртуальной машине Azure серверное программное обеспечение. Дополнительные сведения см. в статье [Linux в Azure — рекомендованные дистрибутивы](endorsed-distros.md).

## <a name="how-much-storage-can-i-use-with-a-virtual-machine"></a>Какой объем памяти можно использовать с виртуальной машиной?
Каждый диск данных может иметь до 32 767 гиб. Количество дисков данных, которое можно использовать, зависит от размера виртуальной машины. Дополнительные сведения см. в статье [размеры виртуальных машин](../sizes.md).

Управляемые диски Azure — это рекомендуемые предложения хранилища дисков, используемые с виртуальными машинами Azure и предназначенные для постоянного хранения данных. Можно использовать несколько управляемых дисков для каждой виртуальной машины. Есть два класса управляемых дисков с разными возможностями хранения: управляемые диски "Премиум" и "Стандартный". Сведения о ценах см. на [странице с расценками на управляемые диски](https://azure.microsoft.com/pricing/details/managed-disks).

Учетная запись хранения Azure также предоставляет хранилище для диска операционной системы и любых дисков данных. Каждый из этих дисков представляет собой VHD-файл, хранящийся как страничный BLOB-объект. Информацию о ценах см. в статье [Информация о ценах на хранилища](https://azure.microsoft.com/pricing/details/storage/).

## <a name="how-can-i-access-my-virtual-machine"></a>Как получить доступ к своей виртуальной машине?
Установите удаленное подключение для входа в виртуальную машину с помощью Secure Shell (SSH). Ознакомьтесь с инструкциями по подключению [из Windows](ssh-from-windows.md) или [Linux и Mac](mac-create-ssh-keys.md). По умолчанию SSH поддерживает не более 10 параллельных подключений. Число доступных параллельных подключений можно увеличить, изменив файл конфигурации.

Если возникают проблемы, ознакомьтесь со статьей об [устранении неполадок с подключением Secure Shell (SSH)](../troubleshooting/troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).

## <a name="can-i-use-the-temporary-disk-devsdb1-to-store-data"></a>Можно ли использовать временный диск (/dev/sdb1) для хранения данных?
Не используйте временный диск (/dev/sdb1) для хранения данных. Он обеспечивает лишь временное хранение. Вы рискуете потерять данные, которые невозможно будет восстановить.

## <a name="can-i-copy-or-clone-an-existing-azure-vm"></a>Можно ли копировать или клонировать существующую виртуальную машину Azure?
Да. Указания доступны в статье [Создание копии виртуальной машины Linux, работающей в Azure](copy-vm.md).

## <a name="why-am-i-not-seeing-canada-central-and-canada-east-regions-through-azure-resource-manager"></a>Почему в Azure Resource Manager не отображаются регионы центральной и восточной Канады?
При создании виртуальных машин в существующих подписках Azure эти два новых региона не регистрируются автоматически. Регистрация осуществляется автоматически при развертывании виртуальной машины на портале Azure в любом другом регионе с помощью Azure Resource Manager. После развертывания виртуальной машины в любом другом регионе Azure новые регионы станут доступными для последующих виртуальных машин.

## <a name="can-i-add-a-nic-to-my-vm-after-its-created"></a>Создав виртуальную машину, могу я добавить к ней сетевой адаптер?
Да, теперь это возможно. Сначала виртуальную машину нужно остановить и отменить ее распределение. Затем можно добавить или удалить сетевую карту (за исключением последней сетевой карты на виртуальной машине). 

## <a name="are-there-any-computer-name-requirements"></a>Есть ли какие-либо требования к имени компьютера?
Да. Длина имени компьютера не должна превышать 64 знака. Дополнительные сведения об именовании ресурсов см. в [правилах и ограничениях соглашений об именовании](/azure/architecture/best-practices/resource-naming).

## <a name="are-there-any-resource-group-name-requirements"></a>Есть ли какие-либо требования к имени группы ресурсов?
Да. Длина имени группы ресурсов не должна превышать 90 знаков. Дополнительные сведения о группах ресурсов см. в [правилах и ограничениях соглашений об именовании](/azure/architecture/best-practices/resource-naming).

## <a name="what-are-the-username-requirements-when-creating-a-vm"></a>Какие требования к имени пользователя при создании виртуальной машины?

Имя пользователя должно содержать от 1 до 32 знаков.

Не допускаются следующие имена пользователей:

- `1`
- `123`
- `a`
- `actuser`
- `adm`
- `admin`
- `admin1`
- `admin2`
- `administrator`
- `aspnet`
- `backup`
- `console`
- `david`
- `guest`
- `john`
- `owner`
- `root`
- `server`
- `sql`
- `support_388945a0`
- `support`
- `sys`
- `test`
- `test1`
- `test2`
- `test3`
- `user`
- `user1`
- `user2`
- `user3`
- `user4`
- `user5`
- `video`


## <a name="what-are-the-password-requirements-when-creating-a-vm"></a>Какие требования к паролю при создании виртуальной машины?

В зависимости от используемого средства существуют различные требования к длине пароля.
 - Портал — от 12-72 символов
 - PowerShell — от 8-123 символов
 - CLI — от 12-123 символов
 - Шаблоны Azure Resource Manager (ARM) — 12-72 символов и управляющих символов не допускаются
 

Пароли также должны соответствовать трем из следующих 4 требований к сложности:

* используются строчные знаки;
* используются знаки верхнего регистра;
* используется хотя бы одна цифра;
* используется специальный знак (регулярное выражение [\W_]).

Не допускаются следующие пароли:

<table>
    <tr>
        <td style="text-align:center">abc@123</td>
        <td style="text-align:center">P@$$w0rd</td>
        <td style="text-align:center">P@ssw0rd</td>
        <td style="text-align:center">P@ssword123</td>
        <td style="text-align:center">Pa$$word</td>
    </tr>
    <tr>
        <td style="text-align:center">pass@word1</td>
        <td style="text-align:center">Password!</td>
        <td style="text-align:center">Password1</td>
        <td style="text-align:center">Password22</td>
        <td style="text-align:center">iloveyou!</td>
    </tr>
</table>

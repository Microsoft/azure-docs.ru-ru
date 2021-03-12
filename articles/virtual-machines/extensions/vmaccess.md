---
title: Сброс доступа к виртуальной машине Linux в Azure
description: В этой статье показано, как управлять пользователями с правами администратора и сбрасывать параметры доступа на виртуальных машинах Linux, используя расширение VMAccess и Azure CLI.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 05/10/2018
ms.openlocfilehash: e0e63762fc151d93cb34885c04dd4d03285c29f1
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102557052"
---
# <a name="manage-administrative-users-ssh-and-check-or-repair-disks-on-linux-vms-using-the-vmaccess-extension-with-the-azure-cli"></a>Управление пользователями с правами администратора, SSH и проверка или восстановление дисков на виртуальных машинах Linux с помощью расширения VMAccess и Azure CLI
## <a name="overview"></a>Обзор
На диске в виртуальной машине Linux имеются ошибки. Вы каким-то образом сбросили пароль пользователя root для виртуальной машины Linux или случайно удалили закрытый ключ SSH. Если бы такое случилось раньше, вам пришлось бы ехать в центр данных и открывать KVM-консоль для доступа к серверу. Расширение Azure VMAccess можно представить как KVM-коммутатор, который позволяет открывать консоль для сброса разрешений на доступ к Linux или обслуживания дисков.

В этой статье показано, как использовать расширение Azure VMAccess для проверки или восстановления диска, сброса прав доступа пользователей, управления учетными записями пользователей с правами администратора или обновления конфигурации SSH в Linux при использовании виртуальных машин Azure Resource Manager. Если необходимо управлять классическими виртуальными машинами, следуйте инструкциям, приведенным в [документации по классическим виртуальным машинам](/previous-versions/azure/virtual-machines/linux/classic/reset-access-classic). 
 
> [!NOTE]
> Если вы с помощью VMAccess сбрасываете пароль виртуальной машины после установки расширения входа в AAD, это расширение необходимо повторно запустить, чтобы повторно активировать его на виртуальной машине.

## <a name="prerequisites"></a>Предварительные требования
### <a name="operating-system"></a>Операционная система

Расширение VMAccess можно выполнять с использованием следующих дистрибутивов Linux.

| Distribution | Версия |
|---|---|
| Ubuntu | 16.04 LTS, 14.04 LTS и 12.04 LTS |
| Debian | Debian 7.9+, 8.2+ |
| Red Hat | RHEL 6.7 или более поздней версии, 7.1 или более поздней версии |
| Oracle Linux | 6.4+, 7.0+ |
| Suse | 11 и 12 |
| openSUSE | openSUSE LEAP 42.2+ |
| CentOS | CentOS 6.3+, 7.0+ |
| CoreOS | 494.4.0 + |

## <a name="ways-to-use-the-vmaccess-extension"></a>Использование расширения VMAccess
Расширение VMAccess для виртуальных машин Linux можно использовать двумя способами:

* с помощью командной строки Azure и необходимых параметров;
* [с помощью необработанных файлов JSON, которые расширение VMAccess обрабатывает](#use-json-files-and-the-vmaccess-extension) для выполнения операций.

В следующих примерах используются команды [az vm user](/cli/azure/vm/user). Чтобы выполнить эти действия, нужно установить последнюю версию [Azure CLI](/cli/azure/install-az-cli2) и войти в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index).

## <a name="update-ssh-key"></a>Обновление ключа SSH
В следующем примере обновляется ключ SSH для пользователя `azureuser` на виртуальной машине `myVM`:

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

> **Примечание.** При использовании команды `az vm user update` в файл `~/.ssh/authorized_keys` на виртуальной машине добавляется новый текст открытого ключа для администратора. Существующие ключи SSH не заменяются и не удаляются. Предыдущий набор ключей не будет удален во время развертывания или последующих обновлений при помощи расширения VMAccess.

## <a name="reset-password"></a>Сброс пароля
В следующем примере сбрасывается пароль для пользователя `azureuser` на виртуальной машине `myVM`.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --password myNewPassword
```

## <a name="restart-ssh"></a>Перезапуск SSH
В следующем примере показан перезапуск управляющей программы SSH и сброс настроек SSH до значений по умолчанию на виртуальной машине `myVM`.

```azurecli-interactive
az vm user reset-ssh \
  --resource-group myResourceGroup \
  --name myVM
```

## <a name="create-an-administrativesudo-user"></a>Создание пользователя с правами администратора (с правами sudo)
В следующем примере создается пользователь с именем `myNewUser` и правами **sudo**. Учетная запись использует ключ SSH для проверки подлинности на виртуальной машине с именем `myVM`. Этот метод предназначен для восстановления доступа к виртуальной машине в случае, если пользователь забудет или потеряет текущие учетные данные. Количество учетных записей с правами **sudo** рекомендуется ограничить.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="delete-a-user"></a>Удаление пользователя
В следующем примере удаляется пользователь `myNewUser` на виртуальной машине `myVM`.

```azurecli-interactive
az vm user delete \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser
```

## <a name="use-json-files-and-the-vmaccess-extension"></a>Использование JSON-файлов и расширения VMAccess
В следующих примерах используется необработанные JSON-файлы. Используйте команду [az vm extension set](/cli/azure/vm/extension) для последующего вызова своих JSON-файлов. Эти JSON-файлы также можно вызывать из шаблонов Azure. 

### <a name="reset-user-access"></a>Сброс доступа пользователей
Если вы потеряли доступ к учетной записи root на виртуальной машине Linux, можете выполнить скрипт VMAccess, чтобы обновить ключ SSH или пароль пользователя.

Чтобы обновить открытый ключ SSH, создайте файл `update_ssh_key.json` и добавьте параметры в указанном ниже формате. Для параметров `username` и `ssh_key` задайте свои значения.

```json
{
  "username":"azureuser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNGxxxxxx2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7xxxxxxwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5wxxtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== azureuser@myVM"
}
```

Выполните сценарий VMAccess с помощью следующей команды:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings update_ssh_key.json
```

Чтобы сбросить пароль пользователя, создайте файл `reset_user_password.json` и добавьте параметры в следующем формате. Для параметров `username` и `password` задайте свои значения.

```json
{
  "username":"azureuser",
  "password":"myNewPassword" 
}
```

Выполните сценарий VMAccess с помощью следующей команды:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_user_password.json
```

### <a name="restart-ssh"></a>Перезапуск SSH
Чтобы перезапустить управляющую программу SSH и сбросить настройки SSH до значений по умолчанию, создайте файл `reset_sshd.json`. Добавьте следующее содержимое:

```json
{
  "reset_ssh": true
}
```

Выполните сценарий VMAccess с помощью следующей команды:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_sshd.json
```

### <a name="manage-administrative-users"></a>Управление пользователями с правами администратора

Чтобы создать пользователя с правами **sudo**, использующего ключ SSH для проверки подлинности, создайте файл `create_new_user.json` и добавьте параметры в следующем формате. Для параметров `username` и `ssh_key` задайте свои значения. Этот метод предназначен для восстановления доступа к виртуальной машине в случае, если пользователь забудет или потеряет текущие учетные данные. Количество учетных записей с правами **sudo** рекомендуется ограничить.

```json
{
  "username":"myNewUser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
  "password":"myNewUserPassword"
}
```

Выполните сценарий VMAccess с помощью следующей команды:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings create_new_user.json
```

Чтобы удалить пользователя, создайте файл `delete_user.json` и добавьте в него следующее содержимое. Для параметра `remove_user` задайте свое значение.

```json
{
  "remove_user":"myNewUser"
}
```

Выполните сценарий VMAccess с помощью следующей команды:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings delete_user.json
```

### <a name="check-or-repair-the-disk"></a>Проверка или исправление диска
С помощью VMAccess можно также проверить и восстановить диск, добавленный к виртуальной машине Linux.

Чтобы проверить, а затем восстановить диск, создайте файл `disk_check_repair.json` и добавьте параметры в следующем формате. Для параметра `repair_disk` введите имя своего диска.

```json
{
  "check_disk": "true",
  "repair_disk": "true, mydiskname"
}
```

Выполните сценарий VMAccess с помощью следующей команды:

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings disk_check_repair.json
```
## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Данные о состоянии развертывания расширения можно получить на портале Azure, а также использовав Azure CLI. Чтобы просмотреть состояние развертывания расширений для определенной виртуальной машины, выполните следующую команду в Azure CLI.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните "Получить поддержку". Дополнительные сведения об использовании службы поддержки Azure см. в статье [Часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

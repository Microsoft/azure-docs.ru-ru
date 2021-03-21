---
title: Отключение или удаление агента подготовки
description: Узнайте, как отключить или удалить агент подготовки в виртуальных машинах и образах Linux.
author: danielsollondon
ms.service: virtual-machines
ms.collection: linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 7c797957c292b9859ca41951b15f58c3d0be40b2
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102561078"
---
# <a name="disable-or-remove-the-linux-agent-from-vms-and-images"></a>Отключение или удаление агента Linux с виртуальных машин и образов

Перед удалением агента Linux необходимо ознакомиться с тем, какие виртуальные машины не удастся выполнить после удаления агента Linux.

[Расширения](../extensions/overview.md) виртуальных машин Azure — это небольшие приложения, которые обеспечивают настройку и задачи автоматизации, выполняемые после развертывания на виртуальных машинах Azure. расширения устанавливаются и управляются плоскостью управления Azure. [Агент Linux для Azure](../extensions/agent-linux.md) обрабатывает команды расширения платформы и обеспечивает правильное состояние расширения внутри виртуальной машины.

На платформе Azure доступно множество расширений, в число которых входят приложения для конфигурации, мониторинга, безопасности виртуальных машин, а также служебные приложения. Существует большой вариант использования первых и сторонних расширений, примеры основных сценариев использования расширений для:
* Поддержка служб Azure первого производителя, таких как Azure Backup, мониторинг, шифрование дисков, безопасность, репликация сайта и другие.
* Сброс SSH/пароль
* Конфигурация виртуальной машины — выполнение пользовательских сценариев, установка Chef, агенты Puppet и т. д.
* Сторонние продукты, такие как антивирусные продукты, средства уязвимостей виртуальных машин, средства мониторинга виртуальных машин и приложений.
* Расширения могут идти в комплекте с новым развертыванием виртуальной машины. Например, они могут входить в состав более крупного развертывания и использоваться для настройки приложений при подготовке виртуальной машины или выполняться в поддерживаемых операционных системах после развертывания.

## <a name="disabling-extension-processing"></a>Отключение обработки расширения

Существует несколько способов отключить обработку расширения в зависимости от потребностей, но прежде чем продолжить, **необходимо** удалить все расширения, развернутые на виртуальной машине, например с помощью Azure CLI можно [вывести список](/cli/azure/vm/extension#az-vm-extension-list) и [Удалить](/cli/azure/vm/extension#az-vm-extension-delete):

```azurecli
az vm extension delete -g MyResourceGroup --vm-name MyVm -n extension_name
```
> [!Note]
> 
> Если этого не сделать, платформа попытается отправить конфигурацию расширения и время ожидания после 40min.

### <a name="disable-at-the-control-plane"></a>Отключить на плоскости управления
Если вы не уверены, требуются ли расширения в будущем, можно оставить агент Linux установленным на виртуальной машине, а затем отключить возможность обработки расширения на платформе. Этот параметр доступен в `Microsoft.Compute` версии API `2018-06-01` или выше и не зависит от установленной версии агента Linux.

```azurecli
az vm update -g <resourceGroup> -n <vmName> --set osProfile.allowExtensionOperations=false
```
Вы можете легко повторно включить эту обработку расширения с платформы, выполнив указанную выше команду, но присвоить ей значение true.

## <a name="remove-the-linux-agent-from-a-running-vm"></a>Удаление агента Linux с работающей виртуальной машины

Убедитесь, что вы **удалили** все существующие расширения с виртуальной машины, как описано выше.

### <a name="step-1-remove-the-azure-linux-agent"></a>Шаг 1. Удаление агента Linux для Azure

Если вы просто удаляете агент Linux, а не связанные с ним артефакты конфигурации, можно переустановить позднее. Чтобы удалить агент Linux для Azure, выполните одно из следующих действий в качестве привилегированного пользователя:

#### <a name="for-ubuntu-1804"></a>Для Ubuntu >= 18,04
```bash
apt -y remove walinuxagent
```

#### <a name="for-redhat--77"></a>Для RedHat >= 7,7
```bash
yum -y remove WALinuxAgent
```

#### <a name="for-suse"></a>Для SUSE
```bash
zypper --non-interactive remove python-azure-agent
```

### <a name="step-2-optional-remove-the-azure-linux-agent-artifacts"></a>Шаг 2. Удаление артефактов агента Linux для Azure (необязательно)
> [!IMPORTANT] 
>
> Вы можете удалить все связанные артефакты агента Linux, но это значит, что вы не сможете переустановить его позже. Поэтому настоятельно рекомендуется сначала отключить агент Linux, удалив агент Linux с использованием только указанного выше. 

Если вы уверены, что не будете повторно переустанавливать агент Linux, можно выполнить следующие действия:

#### <a name="for-ubuntu-1804"></a>Для Ubuntu >= 18,04
```bash
apt -y purge walinuxagent
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

#### <a name="for-redhat--77"></a>Для RedHat >= 7,7
```bash
yum -y remove WALinuxAgent
rm -f /etc/waagent.conf.rpmsave
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

#### <a name="for-suse"></a>Для SUSE
```bash
zypper --non-interactive remove python-azure-agent
rm -f /etc/waagent.conf.rpmsave
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

## <a name="preparing-an-image-without-the-linux-agent"></a>Подготовка образа без агента Linux
Если у вас есть образ, который уже содержит Cloud-init, и вы хотите удалить агент Linux, но по-прежнему подготавливать его с помощью Cloud-init, выполните действия, описанные в шаге 2 (и при необходимости шаг 3) в качестве корневого каталога, чтобы удалить агент Linux для Azure, после чего будет удалена конфигурация Cloud-init и кэшированные данные, а затем подготовьте ВИРТУАЛЬ

```bash
cloud-init clean --logs --seed 
```

## <a name="deprovision-and-create-an-image"></a>Отменить наполнение и создание образа
Агент Linux может очистить некоторые из существующих метаданных образа с шагом "waagent-unготовить + User", однако после удаления необходимо выполнить следующие действия, а также удалить из него все другие конфиденциальные данные.

- Удалить все существующие ключи узла SSH

   ```bash
   rm /etc/ssh/ssh_host_*key*
   ```
- Удаление учетной записи администратора

   ```bash
   touch /var/run/utmp
   userdel -f -r <admin_user_account>
   ```
- Удаление корневого пароля

   ```bash
   passwd -d root
   ```

После завершения приведенного выше можно создать пользовательский образ с помощью Azure CLI.


**Создание обычного управляемого образа**
```azurecli
az vm deallocate -g <resource_group> -n <vm_name>
az vm generalize -g <resource_group> -n <vm_name>
az image create -g <resource_group> -n <image_name> --source <vm_name>
```

**Создание версии образа в общей коллекции образов**

```azurecli
az sig image-version create \
    -g $sigResourceGroup 
    --gallery-name $sigName 
    --gallery-image-definition $imageDefName 
    --gallery-image-version 1.0.0 
    --managed-image /subscriptions/00000000-0000-0000-0000-00000000xxxx/resourceGroups/imageGroups/providers/images/MyManagedImage
```
### <a name="creating-a-vm-from-an-image-that-does-not-contain-a-linux-agent"></a>Создание виртуальной машины из образа, который не содержит агент Linux
При создании виртуальной машины из образа без агента Linux необходимо убедиться, что конфигурация развертывания виртуальной машины указывает, что расширения не поддерживаются на этой виртуальной машине.

> [!NOTE] 
> 
> Если этого не сделать, платформа попытается отправить конфигурацию расширения и время ожидания после 40min.

Чтобы развернуть виртуальную машину с отключенными расширениями, можно использовать Azure CLI с параметром [--Enable-Agent](/cli/azure/vm#az-vm-create).

```azurecli
az vm create \
    --resource-group $resourceGroup \
    --name $prodVmName \
    --image RedHat:RHEL:8.1-ci:latest \
    --admin-username azadmin \
    --ssh-key-value "$sshPubkeyPath" \
    --enable-agent false
```

Кроме того, это можно сделать с помощью шаблонов Azure Resource Manager (ARM), установив `"provisionVMAgent": false,` .

```json
"osProfile": {
    "computerName": "[parameters('virtualMachineName')]",
    "adminUsername": "[parameters('adminUsername')]",
    "linuxConfiguration": {
        "disablePasswordAuthentication": "true",
        "provisionVMAgent": false,
        "ssh": {
            "publicKeys": [
                {
                    "path": "[concat('/home/', parameters('adminUsername'), '/.ssh/authorized_keys')]",
                    "keyData": "[parameters('adminPublicKey')]"
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Подготовка Linux](provisioning.md).

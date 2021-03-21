---
title: Устранение неполадок Azure бастиона | Документация Майкрософт
description: Из этой статьи вы узнаете, как устранять неполадки Azure бастиона.
services: bastion
author: charwen
ms.service: bastion
ms.topic: troubleshooting
ms.date: 10/16/2019
ms.author: charwen
ms.openlocfilehash: 23b7a66afcc91cf1cf4a5dd9f720aad24ad40071
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98133996"
---
# <a name="troubleshoot-azure-bastion"></a>Устранение неполадок в службе "Бастион Azure"

В этой статье показано, как устранять неполадки Azure бастиона.

## <a name="unable-to-create-an-nsg-on-azurebastionsubnet"></a><a name="nsg"></a>Не удалось создать NSG на Азуребастионсубнет

**Вопрос.** При попытке создать NSG в подсети Azure бастиона возникает следующая ошибка: *"Группа безопасности сети <NSG name> не имеет необходимых правил для Azure бастиона Subnet азуребастионсубнет"*.

Ответ **.** Если вы создали и примените NSG к *азуребастионсубнет*, убедитесь, что вы добавили необходимые правила в NSG. Список необходимых правил см. в разделе [Работа с NSG Access и Azure бастиона](./bastion-nsg.md). Если не добавить эти правила, создание или обновление NSG завершится ошибкой.

Пример правил NSG доступен для справки в [шаблоне](https://github.com/Azure/azure-quickstart-templates/tree/master/101-azure-bastion-nsg)быстрого запуска.
Дополнительные сведения см. в [статье Руководство по NSG для Azure бастиона](bastion-nsg.md).

## <a name="unable-to-use-my-ssh-key-with-azure-bastion"></a><a name="sshkey"></a>Не удается использовать ключ SSH с Azure бастиона

**Вопрос.** При попытке просмотра файла ключа SSH возникает следующая ошибка: *"закрытый ключ SSH должен начинаться с-----BEGIN RSA закрытый ключ-----и заканчивается-----End RSA закрытый ключ-----"*.

Ответ **.** Azure бастиона поддерживает только ключи RSA SSH в этот момент времени. Убедитесь, что вы просматриваете файл ключа, который является закрытым ключом RSA для SSH, с открытым ключом, подготовленным на целевой виртуальной машине. 

В качестве примера можно использовать следующую команду, чтобы создать новый ключ RSA SSH.

**SSH-Keygen-t RSA-b 4096-C " email@domain.com "**

Выходные данные:

```
ashishj@dreamcatcher:~$ ssh-keygen -t rsa -b 4096 -C "email@domain.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ashishj/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ashishj/.ssh/id_rsa.
Your public key has been saved in /home/ashishj/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:c+SBciKXnwceaNQ8Ms8C4h46BsNosYx+9d+AUxdazuE email@domain.com
The key's randomart image is:
+---[RSA 4096]----+
|      .o         |
| .. ..oo+. +     |
|=.o...B==.O o    |
|==o  =.*oO E     |
|++ .. ..S =      |
|oo..   + =       |
|...     o o      |
|         . .     |
|                 |
+----[SHA256]-----+
```

## <a name="unable-to-sign-in-to-my-windows-domain-joined-virtual-machine"></a><a name="domain"></a>Не удается войти в виртуальную машину, присоединенную к домену Windows

**Вопрос.** Мне не удается подключиться к виртуальной машине Windows, присоединенной к домену.

Ответ **.** Azure бастиона поддерживает вход в виртуальную машину, присоединенную к домену, для входа в домен на основе пароля пользователя. При указании учетных данных домена в портал Azure используйте формат имени участника-пользователя ( username@domain ) вместо формата *домен \ имя_пользователя* для входа. Эта поддержка поддерживается для присоединения к домену или гибридных виртуальных машин (присоединенных к домену, а также к присоединенным к Azure AD). Он не поддерживается для виртуальных машин, присоединенных к Azure AD.

## <a name="file-transfer-issues"></a><a name="filetransfer"></a>Проблемы при переносе файлов

**Вопрос.** Поддерживается ли обмен файлами с Azure бастиона?

Ответ **.** В настоящее время перенос файлов не поддерживается. Мы работаем над добавлением поддержки.

## <a name="black-screen-in-the-azure-portal"></a><a name="blackscreen"></a>Черный экран в портал Azure

**Вопрос.** Когда я пытаюсь подключиться с помощью Azure бастиона, я не могу Connnect на целевую виртуальную машину и получил черный экран в портал Azure.

Ответ **.** Это происходит при возникновении проблемы с сетевым подключением между веб-браузером и Azure бастиона (ваш клиентский брандмауэр Интернета может блокировать трафик WebSocket или аналогичный), а также между бастиона Azure и целевой виртуальной машиной. В большинстве случаев NSG применяется либо к Азуребастионсубнет, либо к целевой подсети виртуальной машины, которая блокирует трафик RDP/SSH в виртуальной сети. Разрешите трафик WebSocket на клиентском брандмауэре Интернета и проверьте группы безопасности сети в подсети целевой виртуальной машины.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [часто задаваемые вопросы о бастиона](bastion-faq.md).

---
title: Подключение файлового ресурса Azure к виртуальной машине, подключенной к AD DS
description: Узнайте, как подключить общую папку к локальным компьютерам, подключенным к домен Active Directory службам.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: 1dd2768c7541312b68e2a44a35fc4260d67c5655
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104587004"
---
# <a name="part-four-mount-a-file-share-from-a-domain-joined-vm"></a>Часть 4. Подключение общей папки из виртуальной машины, присоединенной к домену

Перед началом работы с этой статьей убедитесь, что вы завершили предыдущую статью, [настроили разрешения на уровне каталога и файлов по протоколу SMB](storage-files-identity-ad-ds-configure-permissions.md).

Процесс, описанный в этой статье, проверяет правильность настройки общей папки и разрешений на доступ, а также доступ к файловому ресурсу Azure с виртуальной машины, присоединенной к домену. Назначение роли Azure на уровне общего ресурса может занять некоторое время. 

Войдите на клиент, используя учетные данные, для которых предоставлены разрешения, как показано на следующем рисунке.

![Снимок экрана с окном входа в Azure AD для выполнения аутентификации пользователя](media/storage-files-aad-permissions-and-mounting/azure-active-directory-authentication-dialog.png)

## <a name="mounting-prerequisites"></a>Необходимые компоненты для монтажа

Прежде чем можно будет подключить файловый ресурс, убедитесь, что выполнены следующие предварительные требования:

- Если вы подключаете общую папку от клиента, который ранее подключил общую папку с помощью ключа учетной записи хранения, убедитесь, что вы отключили общую папку, удалили постоянные учетные данные ключа учетной записи хранения и используете AD DS учетные данные для проверки подлинности. Инструкции по очистке подключенного общего ресурса с помощью ключа учетной записи хранения см. на [странице часто задаваемых вопросов](./storage-files-faq.md#ad-ds--azure-ad-ds-authentication).
- Ваш клиент должен иметь строку с подробными сведениями о AD DS. Если компьютер или виртуальная машина находится за пределами сети, управляемой AD DS, необходимо включить VPN для доступа к AD DS для проверки подлинности.

Замените значения заполнителей собственными значениями, а затем используйте следующую команду, чтобы подключить файловый ресурс Azure. Всегда необходимо подключать, используя приведенный ниже путь. Использование CNAME для подключения файлов не поддерживается для проверки подлинности на основе удостоверений (AD DS или Azure AD DS).

```PSH
# Always mount your share using.file.core.windows.net, even if you setup a private endpoint for your share.
$connectTestResult = Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded)
{
  net use <desired-drive letter>: \\<storage-account-name>.file.core.windows.net\<fileshare-name>
} 
else 
{
  Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN, Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."
}

```

Если у вас возникли проблемы при подключении с учетными данными AD DS, обратитесь к статье [не удалось подключить службу файлов Azure с учетными данными AD](storage-troubleshoot-windows-file-connection-problems.md#unable-to-mount-azure-files-with-ad-credentials) .

После успешного подключения общей папки вы успешно включили и настроили локальную проверку подлинности AD DS для файловых ресурсов Azure.

## <a name="next-steps"></a>Дальнейшие действия

Если удостоверение, созданное в AD DS для представления учетной записи хранения, находится в домене или подразделении, обеспечивающем смену паролей, перейдите к следующей статье, чтобы получить инструкции по обновлению пароля:

[Обновите пароль для удостоверения учетной записи хранения в AD DS](storage-files-identity-ad-ds-update-password.md)
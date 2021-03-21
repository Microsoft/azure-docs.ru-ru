---
title: Безопасность транспортного уровня в Azure Backup
description: Узнайте, как разрешить Azure Backup использовать протокол шифрования TLS для обеспечения безопасности данных при передаче по сети.
ms.topic: conceptual
ms.date: 11/01/2020
ms.openlocfilehash: ba9c9d91f562f54695a0739908c8a409d14d5852
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96327123"
---
# <a name="transport-layer-security-in-azure-backup"></a>Безопасность транспортного уровня в Azure Backup

TLS — это протокол шифрования, который обеспечивает безопасность данных при передаче по сети. Azure Backup использует безопасность транспортного уровня для защиты конфиденциальности передаваемых данных резервных копий. В этой статье описаны действия по включению протокола TLS 1,2, обеспечивающего улучшенную безопасность по сравнению с предыдущими версиями.

## <a name="earlier-versions-of-windows"></a>Предшествующие версии Windows

Если на компьютере установлены более ранние версии Windows, необходимо установить соответствующие обновления, описанные в статьях базы знаний, и внести изменения в реестр.

|Операционная система  |статья базы знаний |
|---------|---------|
|Windows Server 2008 с пакетом обновления 2 (SP2)   |   <https://support.microsoft.com/help/4019276>      |
|Windows Server 2008 R2, Windows 7, Windows Server 2012   | <https://support.microsoft.com/help/3140245>         |

>[!NOTE]
>При обновлении будут установлены необходимые компоненты протокола. После установки необходимо внести изменения в раздел реестра, упомянутые в статьях базы знаний выше, чтобы правильно включить необходимые протоколы.

## <a name="verify-windows-registry"></a>Проверка реестра Windows

### <a name="configuring-schannel-protocols"></a>Настройка протоколов SChannel

Следующие разделы реестра гарантируют, что протокол TLS 1,2 включен на уровне компонентов SChannel:

```reg
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
    "Enabled"=dword:00000001

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
    "DisabledByDefault"=dword:00000000
```

>[!NOTE]
>Значения, показанные по умолчанию в Windows Server 2012 R2 и более поздних версиях. Для этих версий Windows, если разделы реестра отсутствуют, их не нужно создавать.

### <a name="configuring-net-framework"></a>Настройка платформа .NET Framework

Следующие разделы реестра настраивают платформа .NET Framework для поддержки надежного шифрования. Дополнительные сведения о [настройке платформа .NET Framework](/dotnet/framework/network-programming/tls#configuring-schannel-protocols-in-the-windows-registry)можно узнать здесь.

```reg
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto" = dword:00000001

[HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
    "SystemDefaultTlsVersions"=dword:00000001
    "SchUseStrongCrypto" = dword:00000001
```

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="why-enable-tls-12"></a>Зачем включать TLS 1,2?

TLS 1,2 более безопасен по сравнению с предыдущими протоколами шифрования, такими как SSL 2,0, SSL 3,0, TLS 1,0 и TLS 1,1. Azure Backup Services уже полностью поддерживают TLS 1,2.

### <a name="what-determines-the-encryption-protocol-used"></a>Что определяет используемый протокол шифрования?

Самая высокая версия протокола, поддерживаемая клиентом и сервером, согласовывается для установления зашифрованного диалога. Дополнительные сведения о протоколе подтверждения TLS см. в статье [Установка безопасного сеанса с помощью протокола TLS](/windows/win32/secauthn/tls-handshake-protocol#establishing-a-secure-session-by-using-tls).

### <a name="what-is-the-impact-of-not-enabling-tls-12"></a>Каково влияние включения TLS 1,2?

Для повышения безопасности при атаках на более раннюю версию протокола Azure Backup начинает отключать версии TLS более ранних версий, чем 1,2, в поэтапном режиме. Это является частью долгосрочного сдвига между службами, чтобы запретить устаревшие соединения по протоколу и набору шифров. Azure Backup службы и компоненты полностью поддерживают TLS 1,2. Однако версии Windows, в которых отсутствуют необходимые обновления или определенные настраиваемые конфигурации, по-прежнему могут препятствовать предложению протоколов TLS 1,2. Это может привести к сбоям, включая, но не ограничиваясь одним или несколькими из следующих:

- Операции резервного копирования и восстановления могут завершиться ошибкой.
- Сбои подключений к компонентам резервного копирования с ошибкой 10054 (существующее подключение было принудительно закрыто удаленным узлом).
- Службы, связанные с Azure Backup, не будут останавливаться или запускаться как обычно.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Протокол безопасности транспортного уровня](/windows/win32/secauthn/transport-layer-security-protocol)
- [Обеспечение поддержки TLS 1,2 в развернутых операционных системах](/security/engineering/solving-tls1-problem#ensuring-support-for-tls-12-across-deployed-operating-systems)
- [Рекомендации по использованию протокола TLS с .NET Framework](/dotnet/framework/network-programming/tls)
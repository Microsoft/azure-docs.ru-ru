---
title: Настройка сервера обработки для аварийного восстановления физических серверов в Azure с помощью Azure Site Recovery | Документация Майкрософт
description: В этой статье описано, как настроить сервер в локальной конфигурации для аварийного восстановления локальных физических серверов в Azure.
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/03/2019
ms.author: ramamill
ms.openlocfilehash: 327e995a8fe2f66903548fba054804768d2538ab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96001573"
---
# <a name="set-up-the-configuration-server-for-disaster-recovery-of-physical-servers-to-azure"></a>Настройка сервера конфигурации для аварийного восстановления физических серверов в Azure

В этой статье приведены сведения о настройке локальной среды для запуска репликации физических серверов под управлением Windows или Linux в Azure.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что у вас уже имеется:
- Хранилище служб восстановления в [портал Azure](https://portal.azure.com "Портал Azure").
- Физический компьютер для установки сервера конфигурации.
- Если вы отключили TLS 1,0 на компьютере, на котором устанавливается сервер конфигурации, убедитесь, что TLs 1,2 включен и что на компьютере установлен платформа .NET Framework версии 4,6 или более поздней (с включенной стойким шифрованием). [Подробнее](https://support.microsoft.com/help/4033999/how-to-resolve-azure-site-recovery-agent-issues-after-disabling-tls-1).

### <a name="configuration-server-minimum-requirements"></a>Минимальные требования к серверу конфигурации
В следующей таблице перечислены минимальные требования к оборудованию, программному обеспечению и сети сервера конфигурации.
[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

> [!NOTE]
> Сервер конфигурации не поддерживает прокси-серверы на основе HTTPS.

## <a name="choose-your-protection-goals"></a>Выбор целевых объектов для защиты

1. На портале Azure откройте колонку **Хранилища служб восстановления** и выберите свое хранилище.
2. В меню **ресурсов** хранилища выберите **Начало работы**  >  **Site Recovery**  >  **Шаг 1. Подготовка инфраструктуры**  >  **цель защиты**.

    ![Снимок экрана, на котором показано, где выбрать цель защиты.](./media/physical-azure-set-up-source/choose-goals.png)
3. На странице **Цель защиты** выберите **To Azure** (В Azure), а затем — **Без виртуализации или иное**. Нажмите кнопку **ОК**.

    ![Выбор цели](./media/physical-azure-set-up-source/physical-protection-goal.png)

## <a name="set-up-the-source-environment"></a>Настройка исходной среды

1. В окне **Подготовка источника**, если у вас нет сервера конфигурации, щелкните **+ сервер конфигурации** , чтобы добавить его.

   ![Снимок экрана, на котором показано, как выбрать сервер конфигурации.](./media/physical-azure-set-up-source/plus-config-srv.png)
2. В колонке **Добавление сервера** в поле **Тип сервера** должно быть указано **Сервер конфигурации**.
4. Скачайте файл единой установки Site Recovery.
5. Скачайте ключ регистрации хранилища. При запуске программы единой установки вам потребуется ключ регистрации. Ключ действителен в течение пяти дней после создания.

    ![Настройка источника](./media/physical-azure-set-up-source/set-source2.png)
6. На компьютере, используемом в качестве сервера конфигурации, запустите **программу единой установки Azure Site Recovery**, чтобы установить сервер конфигурации, сервер обработки и главный целевой сервер.

#### <a name="run-azure-site-recovery-unified-setup"></a>Выполнение единой установки Azure Site Recovery

> [!TIP]
> Если системное время на компьютере отличается от местного более чем на 5 минут, то регистрация сервера конфигурации завершится сбоем. Синхронизируйте системные часы с [сервером времени](/windows-server/networking/windows-time-service/windows-time-service-top) перед началом установки.

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

> [!NOTE]
> Сервер конфигурации можно установить с помощью командной строки. [Подробнее](physical-manage-configuration-server.md#install-from-the-command-line).


## <a name="common-issues"></a>Распространенные проблемы

[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]


## <a name="next-steps"></a>Дальнейшие действия

Следующий этап заключается в [настройке целевой среды](physical-azure-set-up-target.md) в Azure.

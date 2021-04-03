---
title: Обновление устройства StorSimple | Документация Майкрософт
description: Описаны способы использования функции обновления StorSimple для установки обычных обновлений и исправлений и в режиме обслуживания.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 786059f5-2a38-4105-941d-0860ce4ac515
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/23/2018
ms.author: alkohli
ms.openlocfilehash: faf06775b78c5a7c90cea000ac0a1eb768107ef4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94956846"
---
# <a name="update-your-storsimple-8000-series-device"></a>Обновление устройства StorSimple серии 8000
> [!NOTE]
> Классический портал StorSimple устарел. Диспетчеры устройств StorSimple автоматически перейдут на новый портал Azure в соответствии с графиком устаревания. Вы получите сообщение электронной почты и уведомление с портала, касающиеся этого перехода. Этот документ скоро также перестанет использоваться. Сведения, связанные с переходом, см. в [ответах на вопросы о перемещении на портал Azure](./index.yml).

## <a name="overview"></a>Обзор
Функции обновления StorSimple позволяют легко поддерживать устройство StorSimple в актуальном состоянии. В зависимости от типа обновлений можно применить обновления к устройству с помощью классического портала Azure или интерфейса Windows PowerShell. В этом учебнике описываются типы обновлений и способы установки каждого из них.

Можно применить два вида обновлений устройств: 

* Обычные обновления.
* Обновления режима обслуживания.

Обычные обновления можно установить с помощью классического портала Azure или Windows PowerShell. Однако для установки обновлений режима обслуживания необходимо использовать именно Windows PowerShell. 

Каждый тип обновления будет описан отдельно ниже.

### <a name="regular-updates"></a>Обычные обновления
Обычные обновления не нарушают рабочие процессы и их можно установить, когда устройство находится в обычном режиме. Эти обновления применяются через веб-сайт Центра обновления Майкрософт для каждого контроллера устройства. 

> [!IMPORTANT]
> Во время обновления может произойти отработка отказа контроллера. Однако это не повлияет на доступность системы или операции.
> 
> 

* Дополнительные сведения об установке обычных обновлений с помощью классического портала Azure см. в статье [Установка обычных обновлений через классический портал Azure](#install-regular-updates-via-the-azure-classic-portal).
* Обычные обновления можно также установить через Windows PowerShell для StorSimple. Дополнительные сведения см. в статье [Установка обычных обновлений через Windows PowerShell для StorSimple](#install-regular-updates-via-windows-powershell-for-storsimple).

### <a name="maintenance-mode-updates"></a>Обновления режима обслуживания.
Обновления режима обслуживания прерывают работу. К ним относятся, например, обновления встроенного ПО дисков. Для этих обновлений необходимо перевести устройство в режим обслуживания. Дополнительные сведения см. в разделе [Шаг 2. Вход в режим обслуживания](#step2). Нельзя использовать классический портал Azure для установки обновлений режима обслуживания. Вместо этого необходимо использовать Windows PowerShell для StorSimple. 

Дополнительные сведения о том, как установить обновления режима обслуживания, см. в статье [Установка обновлений режима обслуживания через Windows PowerShell для StorSimple](#install-maintenance-mode-updates-via-windows-powershell-for-storsimple).

> [!IMPORTANT]
> Обновления режима обслуживания необходимо применять отдельно для каждого контроллера. 
> 
> 

## <a name="install-regular-updates-via-the-azure-classic-portal"></a>Установка обычных обновлений через классический портал Azure
Обновления для устройства StorSimple можно применять с помощью классического портала Azure.

[!INCLUDE [storsimple-install-updates-manually](../../includes/storsimple-install-updates-manually.md)]

## <a name="install-regular-updates-via-windows-powershell-for-storsimple"></a>Установка обычных обновлений через Windows PowerShell для StorSimple
Кроме того, можно использовать Windows PowerShell для StorSimple для применения обычных обновлений.

> [!IMPORTANT]
> Вы можете устанавливать обычные обновления с помощью Windows PowerShell для StorSimple, но мы настоятельно рекомендуем устанавливать их через классический портал Azure. Начиная с обновления 1 перед установкой обновлений с портала будут выполняться предварительные проверки. Эти предварительные проверки устраняют сбои и обеспечивают более стабильную работу. 
> 
> 

[!INCLUDE [storsimple-install-regular-updates-powershell](../../includes/storsimple-install-regular-updates-powershell.md)]

## <a name="install-maintenance-mode-updates-via-windows-powershell-for-storsimple"></a>Установка обновлений режима обслуживания через Windows PowerShell для StorSimple
Windows PowerShell для StorSimple используется, чтобы применять обновления режима обслуживания к устройству StorSimple. В этом режиме приостанавливаются все запросы ввода-вывода. Также останавливаются некоторые другие службы, например энергонезависимое ОЗУ (NVRAM) или служба кластеров. Оба контроллера перезагружаются при входе и выходе из этого режима. При выходе из этого режима все службы будут возобновлена и должны быть исправны. Это может занять несколько минут.

Если необходимо применить обновления режима обслуживания, на классическом портале Azure вы получите оповещение о наличии обновлений, которые должны быть установлены. Это оповещение будет содержать инструкции по использованию Windows PowerShell для StorSimple для установки обновлений. После обновления устройства выполните ту же процедуру для переключения устройства в обычный режим. Пошаговые инструкции см. в разделе [Шаг 4. Выход из режима обслуживания](#step4).

> [!IMPORTANT]
> * Перед переключением в режим обслуживания убедитесь, что оба контроллера устройства работоспособны, проверив **состояние оборудования** на странице **Обслуживание** классического портала Azure. Если контроллер неработоспособен, обратитесь в службу поддержки Майкрософт для получения дальнейших указаний. Для получения дополнительных сведений обратитесь в службу поддержки Майкрософт. 
> * В режиме обслуживания необходимо сначала установить обновление на одном контроллере, а затем на другом.
> 
> 

### <a name="step-1-connect-to-the-serial-console"></a>Шаг 1. Подключение к последовательной консоли <a name="step1"></a>
Сначала используйте приложение, например PuTTY, для доступа к последовательной консоли. В следующей процедуре объясняется, как использовать PuTTY для подключения к последовательной консоли.

[!INCLUDE [storsimple-use-putty](../../includes/storsimple-use-putty.md)]

### <a name="step-2-enter-maintenance-mode"></a>Шаг 2. Вход в режим обслуживания <a name="step2"></a>
После подключения к консоли проверьте наличие обновлений для установки и перейдите в режим обслуживания, чтобы установить их.

[!INCLUDE [storsimple-enter-maintenance-mode](../../includes/storsimple-enter-maintenance-mode.md)]

### <a name="step-3-install-your-updates"></a>Шаг 3. Установка обновлений <a name="step3"></a>
Затем установите обновления.

[!INCLUDE [storsimple-install-maintenance-mode-updates](../../includes/storsimple-install-maintenance-mode-updates.md)]

### <a name="step-4-exit-maintenance-mode"></a>Шаг 4. Выход из режима обслуживания <a name="step4"></a>
В конце выйдите из режима обслуживания.

[!INCLUDE [storsimple-exit-maintenance-mode](../../includes/storsimple-exit-maintenance-mode.md)]

## <a name="install-hotfixes-via-windows-powershell-for-storsimple"></a>Установка исправлений через Windows PowerShell или StorSimple
В отличие от обновлений для Microsoft Azure StorSimple, исправления устанавливаются из общей папки. Как и в случае обновлений, существует два вида исправлений: 

* Обычные исправления. 
* Исправления режима обслуживания.  

В следующих процедурах описывается способ использования Windows PowerShell для StorSimple для установки обычных исправлений и исправлений режима обслуживания.

[!INCLUDE [storsimple-install-regular-hotfixes](../../includes/storsimple-install-regular-hotfixes.md)]

[!INCLUDE [storsimple-install-maintenance-mode-hotfixes](../../includes/storsimple-install-maintenance-mode-hotfixes.md)]

## <a name="what-happens-to-updates-if-you-perform-a-factory-reset-of-the-device"></a>Что происходит с обновлениями при выполнении сброса параметров к значениям по умолчанию?
Если для устройства выполнить сброс параметров к значениям по умолчанию, все обновления будут потеряны. После того как устройство, для которого выполнен сброс параметров, будет зарегистрировано и настроено, необходимо будет вручную установить обновления с помощью классического портала Azure или Windows PowerShell для StorSimple. Дополнительные сведения о сбросе параметров см. в разделе [Восстановление на устройстве заводских настроек](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).

## <a name="next-steps"></a>Дальнейшие действия
* Узнайте больше об [использовании Windows PowerShell для администрирования устройства StorSimple](./storsimple-8000-windows-powershell-administration.md).
* Узнайте больше об [использовании службы StorSimple Manager для администрирования устройства StorSimple](./storsimple-8000-manager-service-administration.md).
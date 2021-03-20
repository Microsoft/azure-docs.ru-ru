---
title: Настройка инструментов Azure Stream Analytics для Visual Studio
description: В этой статье описаны требования к установке и настройке средств Azure Stream Analytics для Visual Studio.
author: su-jie
ms.author: sujie
ms.service: stream-analytics
ms.topic: how-to
ms.date: 05/22/2018
ms.openlocfilehash: 0077ac8465e8f785e772b384f26e0edc6874a1a0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98018706"
---
# <a name="install-azure-stream-analytics-tools-for-visual-studio"></a>Установка инструментов Azure Stream Analytics для Visual Studio

Visual Studio 2019 и Visual Studio 2017 поддерживают Azure Data Lake и Stream Analytics средства. В этой статье показано, как устанавливать и удалять инструменты.

Дополнительные сведения об использовании этих средств см. в разделе Краткое руководство. [создание Azure Stream Analytics задания с помощью Visual Studio](stream-analytics-quick-create-vs.md).

## <a name="install"></a>Установка

Visual Studio Enterprise (Ultimate/Premium), Professional и Community Editions поддерживают эти средства. Express Edition и Visual Studio для Mac не поддерживают их.

Рекомендуем использовать Visual Studio 2019.

### <a name="install-for-visual-studio-2019-and-2017"></a>Установка для Visual Studio 2019 и 2017<a name="recommended-visual-studio-2019-and-2017"></a>

Средства Azure Data Lake и Stream Analytics являются частью рабочих нагрузок **разработки** и **хранения и обработки данных** в Azure. Включите одну из этих двух рабочих нагрузок во время установки. Если Visual Studio уже установлена, выберите **инструменты**  >  **получить инструменты и компоненты** , чтобы добавить рабочие нагрузки.

Скачайте [Visual studio 2019 (Предварительная версия 2 или более поздней версии) или Visual studio 2017 (15,3 или более поздней версии)](https://www.visualstudio.com/) и следуйте инструкциям по установке.

Выберите рабочую нагрузку **Хранение и обработка данных** , как показано ниже.

![Выбранная рабочая нагрузка "Хранение и обработка данных"](./media/stream-analytics-tools-for-visual-studio-install/stream-analytics-tools-for-vs-2019-install-01.png)

Выберите рабочую нагрузку **разработки Azure** , как показано ниже.

![Выбранная рабочая нагрузка "Разработка для Azure"](./media/stream-analytics-tools-for-visual-studio-install/stream-analytics-tools-for-vs-2019-install-02.png)

После добавления рабочей нагрузки обновите средства. Эта процедура относится к Visual Studio 2019:

1. Выберите **расширения**  >  **Управление расширениями**.

1. В окне **Управление расширениями** выберите **обновления** и выберите **Azure Data Lake и Stream Analytics Инструменты**.

1. Выберите **Обновить** , чтобы установить последнее расширение.

![Расширения и обновления Visual Studio](./media/stream-analytics-tools-for-visual-studio-install/stream-analytics-tools-vs2019-extensions-updates.png)

### <a name="install-for-visual-studio-2015-and-2013"></a>Установка для Visual Studio 2015 и 2013<a name="visual-studio-2015-2013"></a>

Visual Studio Enterprise (Ultimate/Premium), Professional и Community Editions поддерживают эти средства. Выпуск Express не поддерживает их.

* Установите Visual Studio 2015 или Visual Studio 2013 с обновлением 4.
* Установите пакет SDK Microsoft Azure для .NET версии 2.7.1 или более поздней версии с помощью [установщика веб-платформы](https://www.microsoft.com/web/downloads/platform.aspx).
* Установите [средства Microsoft Azure Data Lake и Stream Analytics для Visual Studio](https://www.microsoft.com/en-us/download/details.aspx?id=49504).

## <a name="update"></a>Обновляют<a name="visual-studio-2019-and-2017"></a><a name="visual-studio-2015-and-2013"></a>

Для Visual Studio 2019 и Visual Studio 2017 новое напоминание о версии отображается как уведомление Visual Studio.

Для Visual Studio 2015 и Visual Studio 2013 средства автоматически проверяют наличие новых версий. Следуйте инструкциям по установке последней версии.

## <a name="uninstall"></a>Удаление

Средства Azure Data Lake и Stream Analytics можно удалить. Для Visual Studio 2019 или Visual Studio 2017 выберите **инструменты**  >  **получить средства и компоненты**. В этом случае отмените выбор **средств Azure Data Lake и Stream Analytics**. Он отображается либо в рабочей нагрузке **Хранение и обработка данных** , либо в рабочей нагрузке **разработки Azure** .

Чтобы удалить Visual Studio 2015 или Visual Studio 2013, выберите **Панель управления**  >  **программы и компоненты**. Удалите **Microsoft Azure Data Lake и средства Stream Analytics для Visual Studio**.

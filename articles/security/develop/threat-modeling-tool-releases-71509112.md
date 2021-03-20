---
title: Microsoft Threat Modeling Tool выпуск 9/12/2018
titleSuffix: Azure
description: Ознакомьтесь с заметками о выпуске Microsoft Threat Modeling Tool, выпущенной на 9/12/2018. Примечания включают изменения функций и исправления ошибок.
author: jegeib
ms.author: jegeib
ms.service: security
ms.subservice: security-develop
ms.topic: article
ms.date: 01/15/2019
ms.openlocfilehash: 7b0afdd90f2a1413c7f8364fc4518c4d1116454a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92913591"
---
# <a name="threat-modeling-tool-ga-release-71509112---9122018"></a>Threat Modeling Tool, общедоступный выпуск 7.1.50911.2 от 12.09.2018

Мы рады сообщить о том, что Microsoft Threat Modeling Tool теперь доступно для скачивания в качестве поддерживаемого общедоступного выпуска. Этот выпуск содержит важные обновления конфиденциальности и безопасности, а также исправления ошибок, обновления функций и улучшения стабильности. Пользователям предварительной версии 2017 года будет предложено обновить ее до последнего выпуска с помощью технологии ClickOnce после открытия клиента. Для новых пользователей этого средства можно [скачать клиент](https://aka.ms/threatmodelingtool).

В этом выпуске мы прекращаем поддержку предварительной версии 2017 года и рекомендуем всем пользователям обновить ее до общедоступного выпуска. 15 октября 2018 года или позже мы установим минимальную требуемую версию ClickOnce для Threat Modeling Tool, и все клиенты предварительной версии должны будут выполнить обновление.

Средство Microsoft Threat Modeling Tool 2016, доступное в [Центре загрузки Майкрософт](https://www.microsoft.com/en-us/download/details.aspx?id=49168), поддерживается до 1 октября 2019 года только для важных исправлений безопасности.

## <a name="feature-changes"></a>Изменения функций

### <a name="azure-stencil-updates"></a>Обновления набора элементов Azure

В этот выпуск были добавлены дополнительные наборы элементов Azure и связанные с ними угрозы и способы устранения рисков. Значительные изменения были внесены в основные области служб приложений Azure, предложений базы данных Azure и службы хранилища Azure.

![Обновления набора элементов Azure](./media/threat-modeling-tool-releases-71509112/tmt_azure_stencil_update-300x70.png)

### <a name="onedrive-integration-feature-removed"></a>Функция интеграции OneDrive удалена

Функции предварительной версии, позволяющие сохранить в OneDrive, открыть из OneDrive и поделиться ссылкой, были удалены. Пользователям OneDrive рекомендуется использовать клиент Майкрософт [OneDrive для Windows](https://onedrive.live.com/about/en-us/download/) для доступа к файлам, хранящимся в OneDrive, через стандартные диалоговые окна сохранения и открытия файлов.

## <a name="notable-fixed-bugs-reported-by-customers"></a>Важные исправленные ошибки, о которых сообщили клиенты

### <a name="in-tmt-preview-the-tool-crashes-when-using-the-standard-template"></a>В предварительной версии TMT происходит аварийное завершение средства при использовании стандартного шаблона

- Когда универсальный набор элементов (например, "Универсальный поток данных") добавляется на поверхность конструктора и создает угрозы, средство может аварийно завершить работу. Теперь эта проблема устранена.

### <a name="in-tmt-preview-when-i-save-a-report-or-copy-the-threats-the-risk-levels-are-incorrect"></a>В предварительной версии TMT указываются неверные уровни риска при сохранении отчета или копировании угроз

- Если пользователь изменяет уровень риска конкретных угроз, а затем сохраняет отчет или копирует риски, уровень риска может измениться на "Высокий". Теперь эта проблема устранена.

## <a name="known-issues-and-faq"></a>Известные проблемы и часто задаваемые вопросы

### <a name="users-of-high-resolution-screens-may-experience-small-text-in-the-threat-properties"></a>Пользователи экранов с высоким разрешением могут видеть мелкий текст в свойствах угрозы

#### <a name="issue"></a>Проблема

В представлении "Анализ" средства, если пользователь использует экран с высоким разрешением, который по умолчанию настроен на увеличение масштаба для удобства чтения в Windows, раздел Possible Mitigation(s) (Возможные способы устранения рисков) может отображаться мелким текстом.

![Известная проблема с экранами с высоким разрешением](./media/threat-modeling-tool-releases-71509112/tmt_screen_resolution-300x153.png)

#### <a name="workaround"></a>Обходной путь

Пользователь может щелкнуть текст устранения рисков и использовать стандартное управление масштабированием Windows (CRTL+прокрутить колесико мыши вверх), чтобы увеличить масштаб этого раздела.

### <a name="files-in-the-recently-opened-models-section-of-the-main-window-may-fail-to-open"></a>Не удается открыть файлы в разделе Recently Opened Models (Недавно открытые модели) главного окна

#### <a name="issue"></a>Проблема

Функция, позволяющая открыть из OneDrive, в предварительной версии была удалена. Пользователи с недавно открытыми моделями, которые были сохранены в OneDrive, получат следующую ошибку.

![На снимках экрана показана ошибка: ошибка при открытии ссылки на объект файла, которая не задана экземпляром объекта.](./media/threat-modeling-tool-releases-71509112/tmt_save_error-300x131.png)

#### <a name="workaround"></a>Обходной путь

Пользователям OneDrive рекомендуется использовать клиент Майкрософт [OneDrive для Windows](https://onedrive.live.com/about/en-us/download/) для доступа к файлам, хранящимся в OneDrive, через стандартное диалоговое окно Open a model (Открытие модели).

![Снимок экрана: OneDrive, выбранный в диалоговом окне «Открытие модели».](./media/threat-modeling-tool-releases-71509112/tmt_save_onedrive-300x149.png)

### <a name="my-organization-uses-the-2016-version-of-the-tool-can-i-use-the-azure-stencil-set"></a>Моя организация использует версию средства 2016 года. Могу ли я использовать набор элементов Azure?

Да, можете. [Набор элементов Azure доступен на сайте GitHub](https://github.com/Microsoft/threat-modeling-templates/), и его можно загрузить в версии 2016 средства. Чтобы создать модель с набором элементов Azure, используйте диалоговое окно Template For New Models (Шаблон для новых моделей) на экране главного меню. TMT 2016 не может отобразить ссылки, найденные в полях Possible Mitigations (Возможные способы устранения) набора элементов Azure, поэтому вы можете увидеть ссылки, отображаемые в виде тегов HTML.

![Обновления набора элементов Azure в клиенте 2016](./media/threat-modeling-tool-releases-71509112/tmt_azure_stencils-300x212.png)

## <a name="system-requirements"></a>Требования к системе

- Поддерживаемые операционные системы
  - Microsoft Windows 10
- Требуемая версия .NET
  - 3.5.2 .NET
- Дополнительные требования
  - Для получения обновлений средства и шаблонов требуется подключение к Интернету.

## <a name="documentation-and-feedback"></a>Документация и отзывы

- Документация по Threat Modeling Tool находится на сайте [docs.microsoft.com](threat-modeling-tool.md) и содержит информацию [об использовании средства](threat-modeling-tool-getting-started.md).

## <a name="next-steps"></a>Дальнейшие действия

Скачайте последнюю версию [Microsoft Threat Modeling Tool](https://aka.ms/threatmodelingtool).

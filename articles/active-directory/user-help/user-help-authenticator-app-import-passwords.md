---
title: Импорт паролей в приложение Microsoft Authenticator — Azure AD
description: Узнайте, как импортировать пароли в приложение Майкрософт для проверки подлинности из популярных диспетчеров паролей.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 01/28/2021
ms.author: curtand
ms.reviewer: olhaun
ms.openlocfilehash: ecc6580148dfba92077336a26ff9160fbe88eb2c
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99806161"
---
# <a name="import-passwords-into-the-microsoft-authenticator-app"></a>Импорт паролей в приложение Microsoft Authenticator

Microsoft Authenticator поддерживает импорт паролей из Google Chrome, Firefox, LastPass, Bitwarden и RoboForm. Если Майкрософт пока не поддерживает ваш диспетчер паролей, вы можете [вручную ввести учетные данные для входа в CSV-шаблон](https://go.microsoft.com/fwlink/?linkid=2134938). Чтобы импортировать существующие пароли и управлять ими в приложении Authenticator, просто экспортируйте их из диспетчера паролей в файл данных с разделителями-запятыми (CSV). Затем импортируйте экспортированный CSV-файл в Authenticator в расширении браузера Chrome или непосредственно в самом приложении Authenticator для Android или iOS.

## <a name="import-from-google-chrome-or-android-smart-lock"></a>Импорт паролей из Google Chrome или Smart Lock для Android

Вы можете импортировать в приложение Authenticator пароли из Google Chrome или Smart Lock для Android на своем смартфоне или компьютере. Доступные варианты:

- [Импорт из браузера Chrome для устройств с Android и iOS](#import-from-chrome-on-android-and-ios)
- [Импорт из браузера Chrome для компьютеров](#import-from-chrome-desktop-browser)

### <a name="import-from-chrome-on-android-and-ios"></a>Импорт из браузера Chrome для устройств с Android и iOS

Пользователи Google Chrome для устройств с Android и iOS могут импортировать пароли с помощью телефона. Для этого нужно выполнить несколько простых шагов.

1. Установите приложение Authenticator на телефоне и откройте вкладку **Пароли**.

1. Войдите в Google Chrome на своем устройстве.

1. Коснитесь ![значок меню Google Chrome в виде трех точек](./media/user-help-authenticator-app-import-passwords/ellipsis-chrome.png) в верхнем правом углу на телефонах с Android или нижнем правом углу на устройствах с iOS. Затем выберите **Настройки**.

   Платформа | Расположение
   ---------- | --------
   Android | ![Расположение меню настроек Google Chrome](./media/user-help-authenticator-app-import-passwords/android-settings-menu.png)
   iOS | ![Значок меню настроек Google Chrome](./media/user-help-authenticator-app-import-passwords/apple-settings-menu.png)

1. В разделе **Настройки** выберите **Пароли**.

   Платформа | Расположение
   ---------- | --------
   Android | ![Расположение пункта "Пароли" в Chrome на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-passwords-location.png)
   iOS | ![Расположение пункта "Пароли" в Chrome на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-passwords-location.png)

1. Коснитесь ![значок меню Google Chrome в виде трех точек](./media/user-help-authenticator-app-import-passwords/ellipsis-chrome.png) в верхнем правом углу на телефонах с Android или нижнем правом углу на устройствах с iOS. Затем выберите **Экспорт паролей**.

   Платформа | Расположение
   ---------- | --------
   Android | ![Расположение пункта "Экспорт паролей" в Chrome на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-export-passwords-location.png)
   iOS | ![Расположение пункта "Экспорт паролей" в Chrome на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-export-passwords-location.png)

   Введите PIN-код либо используйте отпечаток пальца или функцию распознавания лиц. Подтвердите свою личность и еще раз нажмите **Экспорт паролей**, чтобы запустить процедуру.

1. После экспорта паролей Chrome предложит выбрать приложение, в которое нужно их импортировать. Выберите **Authenticator**, чтобы начать импорт паролей. Когда этот процесс завершится, вы получите уведомление.

   Платформа | Расположение
   ---------- | --------
   Android | ![Расположение команды импорта паролей из Chrome на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-chrome-import.png)
   iOS | ![Расположение команды импорта паролей из Chrome на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-chrome-import.png)

### <a name="import-from-chrome-desktop-browser"></a>Импорт из браузера Chrome для компьютеров

Сначала в браузере Chrome нужно установить [расширение "Автозаполнение Microsoft"](https://chrome.google.com/webstore/detail/microsoft-autofill/fiedbfgcleddlbcmgdigjgdfcggjcion) и войти в него.

1. Откройте [Диспетчер паролей Google](https://passwords.google.com) в любом браузере. Войдите в учетную запись Google, если вы еще этого не сделали.

1. Щелкните значок шестеренки, ![Значок шестеренки диспетчера паролей на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-password-manager-gear.png) чтобы открыть страницу настроек паролей.

1. Выберите **Экспортировать**, а затем на следующей странице еще раз нажмите **Экспортировать**, чтобы начать экспорт паролей. При появлении запроса на подтверждение личности введите пароль Google. Когда этот процесс завершится, вы получите уведомление.

   ![Расположение команды для экспорта паролей в браузере Chrome на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-chrome-export-passwords-location.png)

1. В Chrome откройте расширение "Автозаполнение Microsoft" и выберите **Параметры**.

   ![Расположение пункта "Параметры" в расширении "Автозаполнение Microsoft" в браузере Chrome на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-chrome-autofill-settings.png)

1. Выберите **Импорт данных**. Откроется диалоговое окно. Затем нажмите кнопку **Выберите файл**, чтобы указать расположение CSV-файла и импортировать его.

   ![Расположение пункта "Импорт данных" и кнопки поиска CSV-файла в браузере Chrome на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-chrome-import-csv.png)

## <a name="import-from-firefox"></a>Импорт из браузера Firefox

Firefox позволяет экспортировать пароли только из браузера для компьютеров. Поэтому перед импортом паролей из Firefox убедитесь, что у вас есть доступ к классической версии этого браузера.

1. Войдите в последнюю версию Firefox на компьютере и выберите меню ![значок меню Firefox в виде трех линий](./media/user-help-authenticator-app-import-passwords/desktop-firefox-ellipsis-icon.png) в верхнем правом углу экрана.

1. Выберите настройка **Логины и пароли**.

   ![Расположение пункта "Логины и пароли" в браузера Firefox на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-firefox-passwords-location.png)

1. На странице Firefox Lockwise щелкните ![значок меню Firefox в виде трех точек](./media/user-help-authenticator-app-import-passwords/desktop-firefox-ellipsis-icon.png) и выберите пункт **Экспорт логинов**, а затем подтвердите действие, нажав кнопку **Экспортировать**. Вам нужно будет подтвердить личность с помощью ввода PIN-кода или пароля устройства либо сканирования отпечатка пальца. После успешной идентификации браузер Firefox экспортирует пароли в выбранное расположение в формате CSV.

   ![Расположение кнопки экспорта паролей в браузера Firefox на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-firefox-export-passwords-location.png)

1. Вы можете импортировать пароли в Authenticator в браузере для компьютера либо на телефоне с iOS или Android. Чтобы импортировать данные в приложение Authenticator на телефоне, выполните указанные ниже действия.

      1. Перенесите экспортированный CSV-файл на телефон c Android или iOS. Используйте предпочтительный для вас надежный способ передачи. Затем скачайте этот файл. Предоставьте доступ к CSV-файлу для приложения Authenticator, чтобы начать импорт.

         Платформа | Расположение
         ---------- | --------
         Android | ![Расположение команды импорта паролей из Chrome на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-chrome-import.png)
         iOS | ![Расположение команды импорта паролей из Chrome на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-chrome-import.png)

      1. После импорта паролей в приложение Authenticator удалите CSV-файл с компьютера или мобильного телефона.

## <a name="import-from-lastpass"></a>Импорт из LastPass

LastPass позволяет экспортировать пароли только из браузера для компьютеров. Поэтому перед импортом паролей убедитесь, что у вас есть доступ к классической версии браузера.

1. Войдите в систему [на веб-сайте LastPass](https://lastpass.com) и выберите **Advanced Options** (Дополнительные параметры), а затем нажмите **Export** (Экспорт).

   ![Расположение команды экспорта паролей в LastPass на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-lastpass-export-passwords-location.png)

1. Введите свой основной пароль, чтобы подтвердить личность. После этого вы увидите на веб-странице экспортированные пароли.

1. Скопируйте содержимое ключа этой веб-страницы.

1. Откройте Блокнот или другой текстовый редактор и вставьте скопированные данные.

1. Выберите **Файл** &gt; **Сохранить**, чтобы сохранить этот файл Блокнота. Укажите имя файла, в конце добавьте расширение .csv (например, LastPass.csv) и сохраните его в безопасном расположении на компьютере.

   ![Сохранение CSV-файла с данными из LastPass на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-lastpass-save-import-file.png)

1. Вы можете импортировать пароли в Authenticator в браузере для компьютера либо на телефоне с iOS или Android. Чтобы импортировать данные в приложение Authenticator на телефоне, выполните указанные ниже действия.

      1. Перенесите экспортированный CSV-файл на смартфон. Используйте предпочтительный для вас надежный способ передачи. Затем скачайте этот файл. Предоставьте доступ к CSV-файлу для приложения Authenticator, чтобы начать импорт.

         Платформа | Расположение
         ---------- | --------
         Android | ![Расположение команды импорта паролей из LastPass на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-chrome-import.png)
         iOS | ![Расположение команды импорта паролей из LastPass на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-chrome-import.png)

      1. После импорта паролей в приложение Authenticator удалите CSV-файл с компьютера или мобильного телефона.

## <a name="import-from-bitwarden"></a>Импорт из Bitwarden

Bitwarden позволяет экспортировать пароли только из браузера для компьютеров. Поэтому перед импортом паролей убедитесь, что у вас есть доступ к классической версии браузера.

1. Войдите в систему на сайте https://vault.bitwarden.com/ и выберите **Инструменты** &gt; **Экспортировать хранилище**. Выберите CSV для формата файла, введите свой основной пароль, а затем нажмите кнопку **Экспортировать хранилище**, чтобы начать экспорт.

   ![Расположение пункта "Экспортировать хранилище" в Bitwarden](./media/user-help-authenticator-app-import-passwords/desktop-bitwarden-export-command-location.png)

1. Вы можете импортировать пароли в Authenticator в браузере для компьютера либо на телефоне с iOS или Android. Чтобы импортировать данные в приложение Authenticator на телефоне, выполните указанные ниже действия.

      1. Перенесите экспортированный CSV-файл на смартфон. Используйте предпочтительный для вас надежный способ передачи. Затем скачайте этот файл. Предоставьте доступ к CSV-файлу для приложения Authenticator, чтобы начать импорт.

         Платформа | Расположение
         ---------- | --------
         Android | ![Расположение команды импорта паролей из Bitwarden на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-chrome-import.png)
         iOS | ![Расположение команды импорта паролей из Bitwarden на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-chrome-import.png)

      1. После импорта паролей в приложение Authenticator удалите CSV-файл с компьютера или мобильного телефона.

## <a name="import-from-roboform"></a>Импорт из RoboForm

Вы можете экспортировать пароли только из классического приложения RoboForm. Поэтому перед импортом паролей убедитесь, что у вас есть доступ к приложению RoboForm для компьютера.

1. Запустите клиент RoboForm на компьютере и войдите в свою учетную запись.

1. В меню **RoboForm** выберите пункт **Параметры**.

   ![Меню параметров RoboForm на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-roboform-options.png)

1. Выберите **Учетная запись и Данные** &gt; **Экспорт**.

   ![Расположение команды экспорта в RoboForm на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-roboform-accounts-data.png)

1. Выберите безопасное расположение для сохранения экспортированного файла. Для параметра **Данные** выберите пункт **Логины**, в качестве формата файла укажите CSV, а затем нажмите кнопку **Экспорт**.

   ![Диалоговое окно экспорта в RoboForm на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-roboform-export-dialog.png)

1. Подтвердите действие, и CSV-файл экспортируется в указанное расположение.

   ![Диалоговое окно подтверждения экспорта в RoboForm на компьютере](./media/user-help-authenticator-app-import-passwords/desktop-roboform-confirmation.png)

1. Вы можете импортировать пароли в Authenticator в браузере для компьютера либо на телефоне с iOS или Android. Чтобы импортировать данные в приложение Authenticator на телефоне, выполните указанные ниже действия.

      1. Перенесите экспортированный CSV-файл на смартфон. Используйте предпочтительный для вас надежный способ передачи. Затем скачайте этот файл. Предоставьте доступ к CSV-файлу для приложения Authenticator, чтобы начать импорт.

         Платформа | Расположение
         ---------- | --------
         Android | ![Расположение команды импорта паролей из RoboForm на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-chrome-import.png)
         iOS | ![Расположение команды импорта паролей из RoboForm на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-chrome-import.png)

      1. После импорта паролей в приложение Authenticator удалите CSV-файл с компьютера или мобильного телефона.

## <a name="import-by-creating-a-csv"></a>Создание CSV-файла для импорта

Если в этой статье нет инструкций по импорту данных из вашего диспетчера паролей, вы можете создать CSV-файл и использовать его для импорта паролей в приложение Authenticator. Майкрософт рекомендует выполнять эти действия на компьютере, чтобы вам было удобнее работать с форматированием.

1. На компьютере [скачайте и откройте наш шаблон для импорта данных](https://go.microsoft.com/fwlink/?linkid=2134938). Если вы используете iPhone, браузер Safari и службу "Связка ключей" от Apple, переходите к шагу 4.

1. Экспортируйте пароли из своего диспетчера паролей в незашифрованный CSV-файл.

1. Скопируйте соответствующие столбцы из экспортированного CSV-файла в шаблон CSV и сохраните его.

1. Если у вас нет экспортированного CSV-файла, вы можете скопировать каждый логин из диспетчера паролей в шаблон CSV. Не удаляйте и не изменяйте строку заголовка. Когда закончите, проверьте целостность данных перед переходом к следующему шагу.

1. Вы можете импортировать пароли в Authenticator в браузере для компьютера либо на телефоне с iOS или Android. Чтобы импортировать данные в приложение Authenticator на телефоне, выполните указанные ниже действия.

      1. Перенесите экспортированный CSV-файл на смартфон. Используйте предпочтительный для вас надежный способ передачи. Затем скачайте этот файл. Предоставьте доступ к CSV-файлу для приложения Authenticator, чтобы начать импорт.

         Платформа | Расположение
         ---------- | --------
         Android | ![Расположение команды импорта паролей из CSV-файла на устройстве с Android](./media/user-help-authenticator-app-import-passwords/android-chrome-import.png)
         iOS | ![Расположение команды импорта паролей из CSV-файла на устройстве с iOS](./media/user-help-authenticator-app-import-passwords/apple-chrome-import.png)

      1. После импорта паролей в приложение Authenticator удалите CSV-файл с компьютера или мобильного телефона.

## <a name="troubleshooting-steps"></a>Действия по устранению неполадок

Самая распространенная причина ошибок при импорте — это неправильное форматирование CSV-файла. Чтобы устранить эту проблему, выполните указанные ниже действия.

- Ознакомьтесь с этой статьей и выясните, поддерживается ли импорт данных из вашего диспетчера паролей. Если импорт поддерживается, попробуйте еще раз выполнить процедуру по инструкции для соответствующего поставщика.

- Если же на данный момент импорт из вашего диспетчера паролей не поддерживается, еще раз [создайте CSV-файл вручную](#import-by-creating-a-csv).

- Проверьте целостность данных в CSV-файле, следуя рекомендациям ниже.

  - Первая строка должна содержать заголовок с тремя столбцами: **url**, **username** и **password**.

  - Каждая строка должна содержать значение в столбцах **url** и **password**.

- Вы можете повторно создать CSV-файл, вставив данные в [CSV-шаблон](https://go.microsoft.com/fwlink/?linkid=2134938).

- Если эти рекомендации не помогают, сообщите о проблеме с помощью ссылки **Отправка отзыва** в приложении Authenticator.

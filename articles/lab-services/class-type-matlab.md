---
title: Настройка лаборатории для обучения MATLAB с помощью служб лабораторий Azure | Документация Майкрософт
description: Узнайте, как настроить лабораторию для изучения MATLAB с помощью служб лаборатории Azure.
author: emaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 137959f51b08dceee150962f77110ee2ac1dc193
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85445004"
---
# <a name="setup-a-lab-to-teach-matlab"></a>Настройка лаборатории для обучения MATLAB

[MATLAB](https://www.mathworks.com/products/matlab.html), который означает Лаборатория матрицы, представляет собой платформу программирования от [MathWorks](https://www.mathworks.com/).  Она сочетает вычислительную мощность и визуализацию, делая ее популярным средством в сфере математических, инженерных, физических и химия.

Если вы используете лицензию на [уровне университета](https://www.mathworks.com/academia/tah-support-program/administrators.html), см. инструкции по [скачиванию файлов установки MATLAB](https://www.mathworks.com/matlabcentral/answers/259632-how-can-i-get-matlab-installation-files-for-use-on-an-offline-machine) для загрузки файлов установщика MATLAB на компьютере шаблона.  

В этой статье мы покажем, как настроить класс, который использует клиентское программное обеспечение MATLAB с сервером лицензирования.

## <a name="license-server"></a>Сервер лицензирования

Прежде чем изменять шаблон компьютера для лаборатории, необходимо настроить сервер для запуска программного обеспечения [диспетчера лицензий сети](https://www.mathworks.com/help/install/administer-network-licenses.html) .  Эти инструкции применимы только для учреждений, которые выбирают вариант лицензирования сети для MATLAB, что позволяет пользователям совместно использовать пул лицензионных ключей.  Также потребуется сохранить файл лицензии и ключ установки файла для последующего использования.  Подробные инструкции по загрузке файла лицензии см. на первом шаге в статье [Установка диспетчера сетевых лицензий с подключением к Интернету](https://www.mathworks.com/help/install/ug/install-network-license-manager-with-internet-connection.html) .

Подробные инструкции по установке сервера лицензирования доступны в [статье Установка диспетчера лицензий сети с подключением к Интернету](https://www.mathworks.com/help/install/ug/install-network-license-manager-with-internet-connection.html).  Сведения о включении заимствования см. в статье о [лицензии на Позаимствование](https://www.mathworks.com/help/install/license/borrow-licenses.html) .

Предполагая, что сервер лицензирования расположен в локальной сети или в частной сети в Azure, не забудьте сделать [виртуальную сеть одноранговой](how-to-connect-peer-virtual-network.md) для вашей [учетной записи лаборатории](tutorial-setup-lab-account.md).  Перед созданием лаборатории необходимо выполнить пиринг сети, чтобы виртуальные машины лаборатории могли получить доступ к серверу лицензирования.

## <a name="lab-configuration"></a>Настройка лаборатории

Для настройки этой лаборатории вам прежде всего потребуется подписка Azure.  Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу. После получения подписки Azure можно либо создать новую учетную запись лаборатории в службах лаборатории Azure, либо использовать существующую учетную запись.  Чтобы создать новую учетную запись лаборатории, см. [руководство по настройке учетной записи лаборатории](tutorial-setup-lab-account.md).

Чтобы создать лабораторию, следуйте инструкциям [из руководства по настройке лаборатории для занятий](tutorial-setup-classroom-lab.md).  Примените следующие параметры:

| размер виртуальной машины; | Образ — |
| -------------------- | ----- |
| Средний | Windows 10 |

MATLAB поддерживается в других операционных системах.  Дополнительные сведения см. в разделе [требования к системе для MATLAB](https://www.mathworks.com/support/requirements/matlab-system-requirements.html) .

> [!WARNING]
> Не забудьте сделать [виртуальную сеть виртуальной](https://www.mathworks.com/support/requirements/matlab-system-requirements.html) сетью для учетной записи лаборатории для сервера лицензирования перед созданием лаборатории.

## <a name="template-machine"></a>Компьютер шаблона

После создания шаблона компьютера запустите его и подключитесь к нему, чтобы выполнить следующие основные задачи.

1. Скачайте файлы установки для клиентского программного обеспечения MATLAB.
2. Установите MATLAB с помощью ключа установки файлов.

Установка MATLAB будет состоять из нескольких частей.  В первой части будут скачаны файлы для программы MATLAB и другие продукты, которые нужно установить.  Использование ключа установки файлов требует предварительной загрузки всех установочных файлов для установленных продуктов.  Вторая часть будет устанавливать программное обеспечение MATLAB на виртуальную машину шаблона и активировать программное обеспечение.  Если шаблон виртуальной машины настроен для активации с использованием сервера лицензирования, виртуальные машины учащихся будут делать то же самое.

### <a name="download-installation-files"></a>Скачать файлы установки

Чтобы скачать файлы установки, а также получить файл лицензии и ключ установки файла, необходимо быть администратором лицензий.  Ниже приведены действия по скачиванию файлов установки.

1. Войдите в свою учетную запись для [https://www.mathworks.com](https://www.mathworks.com) .
2. Выберите **Моя учетная запись**.
3. В разделе **мое программное обеспечение** на странице Учетная запись щелкните лицензию, подключенную к программе установки диспетчера лицензий сети для лабораторной работы.
4. На странице сведения о лицензии щелкните **загрузить продукты**.
5. Дождитесь самостоятельного извлечения установщика.
6. Запустите установщик.  
7. На странице **Вход на учетную запись MathWorks** введите учетную запись MathWorks.
8. На странице **лицензионное соглашение MathWorks** примите условия использования и нажмите кнопку **Далее** .
9. Щелкните раскрывающийся список **Дополнительные параметры** и выберите **загрузку без установки**.
10. В **папке выбор назначения** нажмите кнопку **Далее**.
11. Выберите **Windows** в качестве платформы компьютера, на котором вы будете устанавливать MATLAB.
12. На странице **Выбор продукта** убедитесь, что программа MATLAB выбрана вместе с другими продуктами MathWorks, которые вы хотите установить.
13. На странице **Подтверждение выбора и загрузки** нажмите кнопку **начать скачивание**.  
14. Дождитесь загрузки выбранных продуктов.  Нажмите кнопку **Готово**.

Вы также можете скачать ISO-образ с сайта MathWorks.

1. Войдите в свою учетную запись для [https://www.mathworks.com](https://www.mathworks.com) .
2. Перейдите на страницу [https://www.mathworks.com/downloads](https://www.mathworks.com/downloads).
3. Выберите выпуск программы MATLAB, который вы хотите установить.
4. Щелкните ссылку "получить {Version}. ISO-образ" под связанными ссылками, где {Version} — это нечто вроде R2020a.
5. Щелкните ссылку **Загрузка выпуска** синего цвета для Windows.

### <a name="run-installer"></a>Запустить установщик

После загрузки файлов второй шаг — запуск установщика. Опять же, для выполнения этого шага необходимо быть администратором лицензий.  Только администраторы лицензий могут установить MATLAB с помощью ключа установки файлов.

1. Проверьте скачанный файл лицензии и убедитесь, что в строке сервера указан правильный список серверов лицензирования.  Сведения о том, как следует отформатировать файл лицензии, см. в статьях [обновление сетевой лицензии](https://www.mathworks.com/help/install/ug/network-license-files.html), [позаимствование лицензий](https://www.mathworks.com/help/install/license/borrow-licenses.html)и поиск статей с [идентификаторами узлов](https://www.mathworks.com/matlabcentral/answers/101892-what-is-a-host-id-how-do-i-find-my-host-id-in-order-to-activate-my-license) .
2. Запустите установщик MATLAB.
3. На странице **Вход на учетную запись MathWorks** введите учетную запись MathWorks.
4. На странице **лицензионное соглашение MathWorks** примите условия использования и нажмите кнопку **Далее** .
5. Щелкните раскрывающийся список **Дополнительные параметры** и выберите **у меня ключ установки файла**.
6. На странице **Установка с помощью ключа установки файлов** введите ключ установки файла для сервера лицензирования.   Щелкните **Далее**.
7. На странице **Выбор файла лицензии** перейдите к файлу лицензии, сохраненному при скачивании файлов установки ранее.
8. На странице **Выбор конечной папки** нажмите кнопку **Далее**.
9. На странице **Выбор продуктов** нажмите кнопку **Далее**.
10. На странице **Выбор параметров** нажмите кнопку **Далее**.
11. На странице **Подтверждение выбора и установки** нажмите кнопку **начать установку**.
12. На странице **Установка завершена** убедитесь, что установлен флажок **активировать MATLAB** .  Нажмите кнопку **Готово**.

## <a name="cost-estimate"></a>Оценка стоимости

Давайте рассмотрим возможную оценку затрат для этого класса.  Эта оценка не включает затраты на запуск сервера лицензирования.  Мы будем использовать класс из 25 учащихся.  Существует 20 часов запланированного времени класса.  Кроме того, каждый учащийся получает 10 часов квоты на домашние задания или назначения за пределами запланированного времени класса.  Выбран размер виртуальной машины Medium, то есть 55 лабораторных единиц.

Ниже приведен пример возможной оценки затрат для этого класса:

25 учащихся \* (20 запланированных часов + 10 квот) \* 55 единицы лабораторий \*  0,01 долл. США в час = 412,50 долл. США

>[!IMPORTANT]
> Оценка стоимости предназначена только для примера.  Текущие сведения о ценах см. на странице [цен на службы лаборатории Azure](https://azure.microsoft.com/pricing/details/lab-services/).  

## <a name="next-steps"></a>Дальнейшие действия

Дальнейшие действия являются общими для настройки любой лаборатории.

- [Создание, управление и публикация шаблона](how-to-create-manage-template.md)
- [Добавление пользователей](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [Настройка квоты](how-to-configure-student-usage.md#set-quotas-for-users)
- [Настройка расписания](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
- [Ссылки для регистрации электронной почты учащихся](how-to-configure-student-usage.md#send-invitations-to-users)

---
title: Подключение к вычислительному экземпляру в Visual Studio Code (Предварительная версия)
titleSuffix: Azure Machine Learning
description: Узнайте, как подключиться к вычислительному экземпляру Машинное обучение Azure в Visual Studio Code для выполнения рабочих нагрузок интерактивного Jupyter Notebook и удаленной разработки.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: luquinta
author: luisquintanilla
ms.date: 11/16/2020
ms.openlocfilehash: 3f3794ed4748eda44e863ff4627dedc0a34691af
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98070105"
---
# <a name="connect-to-an-azure-machine-learning-compute-instance-in-visual-studio-code-preview"></a>Подключение к вычислительному экземпляру Машинное обучение Azure в Visual Studio Code (Предварительная версия)

В этой статье вы узнаете, как подключиться к вычислительному экземпляру Машинное обучение Azure с помощью Visual Studio Code.

[Вычислительный экземпляр машинное обучение Azure](concept-compute-instance.md) — это полностью управляемая облачная Рабочая станция для специалистов по обработке и анализу данных, которая предоставляет ИТ-администраторам возможности управления и готовности предприятия.

Существует два способа подключения к вычислительному экземпляру из Visual Studio Code.

* Удаленный Jupyter Notebook Server. Этот параметр позволяет задать вычислительный экземпляр в качестве удаленного Jupyter Notebook сервера.
* [Visual Studio Code Удаленная разработка](https://code.visualstudio.com/docs/remote/remote-overview). Visual Studio Code Удаленная разработка позволяет использовать контейнер, удаленный компьютер или подсистему Windows для Linux (WSL) в качестве полнофункциональной среды разработки.

## <a name="configure-compute-instance-as-remote-notebook-server"></a>Настройка экземпляра COMPUTE в качестве сервера удаленной записной книжки

Чтобы настроить вычислительный экземпляр в качестве удаленного Jupyter Notebook сервера, потребуется выполнить несколько предварительных условий.

* Расширение Машинное обучение Azure Visual Studio Code. Дополнительные сведения см. в разделе с [руководством по установке расширения Машинное обучение Azure Visual Studio Code](tutorial-setup-vscode-extension.md).
* Рабочая область Машинное обучение Azure. [Используйте расширение Машинное обучение Azure Visual Studio Code, чтобы создать рабочую область](how-to-manage-resources-vscode.md#create-a-workspace) , если она еще не создана.

Для подключения к вычислительному экземпляру:

1. Откройте Jupyter Notebook в Visual Studio Code.
1. При загрузке интегрированной записной книжки выберите **Jupyter Server**.

    > [!div class="mx-imgBorder"]
    > ![Раскрывающийся список Машинное обучение Azure запуска сервера удаленного Jupyter Notebook](media/how-to-set-up-vs-code-remote/launch-server-selection-dropdown.png)

    Кроме того, можно использовать палитру команд:

    1. Откройте палитру команд. Для этого на панели меню выберите **Вид > Палитра команд**.
    1. Введите в текстовое поле `Azure ML: Connect to Compute instance Jupyter server` .

1. Выберите `Azure ML Compute Instances` из списка параметров сервера Jupyter.
1. Выберите подписку в списке подписок. Если ранее вы настроили рабочую область по умолчанию Машинное обучение Azure, этот шаг пропускается.
1. Щелкните рабочую область.
1. Выберите вычислительный экземпляр из списка. Если у вас ее нет, выберите **создать новый экземпляр Azure вычислительная среда ML** и следуйте инструкциям на экране, чтобы создать его.
1. Чтобы изменения вступили в силу, необходимо перезагрузить Visual Studio Code.
1. Откройте Jupyter Notebook и выполните ячейку.

> [!IMPORTANT]
> Чтобы установить соединение, **необходимо** запустить ячейку.

На этом этапе можно продолжить выполнение ячеек в Jupyter Notebook.

> [!TIP]
> Вы также можете работать с файлами скриптов Python (. Копировать), содержащими ячейки кода Jupyter. Дополнительные сведения см. в [интерактивной документации по Visual Studio Code Python](https://code.visualstudio.com/docs/python/jupyter-support-py).

## <a name="configure-compute-instance-remote-development"></a>Настройка удаленной разработки вычислительных экземпляров

Для полнофункциональной удаленной разработки потребуется несколько предварительных условий:

* [Visual Studio Code удаленное расширение SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh).
* Вычислительный экземпляр с поддержкой SSH. Дополнительные сведения см. [в разделе "создание вычислительного экземпляра"](how-to-create-manage-compute-instance.md).

> [!NOTE]
> На платформах Windows необходимо [установить клиент SSH, совместимый с OpenSSH](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client) , если он еще не существует. Выводимое значение не поддерживается в Windows, так как команда SSH должна находиться в пути.

### <a name="get-the-ip-and-ssh-port-for-your-compute-instance"></a>Получение IP-адреса и порта SSH для вычислительного экземпляра

1. Перейдите в Машинное обучение Azure Studio по адресу https://ml.azure.com/ .
2. Выберите [рабочую область](concept-workspace.md).
1. Перейдите на вкладку « **экземпляры вычислений** ».
1. В столбце **URI приложения** щелкните ссылку **SSH** для вычислительного экземпляра, который требуется использовать в качестве удаленного вычислений. 
1. В диалоговом окне запишите IP-адрес и порт SSH. 
1. Сохраните закрытый ключ в каталоге ~/.СШ/на локальном компьютере. Например, откройте редактор для нового файла и вставьте ключ в: 

   **Linux**:

   ```sh
   vi ~/.ssh/id_azmlcitest_rsa  
   ```

   **Windows**:

   ```cmd
   notepad C:\Users\<username>\.ssh\id_azmlcitest_rsa
   ```

   Закрытый ключ будет выглядеть примерно так:

   ```text
   -----BEGIN RSA PRIVATE KEY-----

   MIIEpAIBAAKCAQEAr99EPm0P4CaTPT2KtBt+kpN3rmsNNE5dS0vmGWxIXq4vAWXD
   ..... 
   ewMtLnDgXWYJo0IyQ91ynOdxbFoVOuuGNdDoBykUZPQfeHDONy2Raw==

   -----END RSA PRIVATE KEY-----
   ```

1. Измените разрешения для файла, чтобы убедиться, что файл доступен только для чтения.  

   ```sh
   chmod 600 ~/.ssh/id_azmlcitest_rsa
   ```

### <a name="add-instance-as-a-host"></a>Добавление экземпляра в качестве узла

Откройте файл `~/.ssh/config` (Linux) или `C:\Users<username>.ssh\config` (Windows) в редакторе и добавьте новую запись, аналогичную приведенной ниже:

```
Host azmlci1 

    HostName 13.69.56.51 

    Port 50000 

    User azureuser 

    IdentityFile ~/.ssh/id_azmlcitest_rsa
```

Ниже приведены некоторые сведения о полях.

|Поле|Описание|
|----|---------|
|Узел|Использование любой краткой формы для вычислительного экземпляра |
|HostName|Это IP-адрес вычислительного экземпляра |
|Порт|Это порт, показанный в диалоговом окне SSH выше |
|Пользователь|Это должно быть `azureuser` |
|идентитифиле|Должен указывать на файл, в который был сохранен закрытый ключ |

Теперь вы можете подключиться к вычислительному экземпляру по протоколу SSH, используя сокращенную версию, которую вы использовали ранее `ssh azmlci1` .

### <a name="connect-vs-code-to-the-instance"></a>Подключение VS Code к экземпляру

1. Щелкните значок Remote-SSH на панели действий Visual Studio Code, чтобы отобразить конфигурации SSH.

1. Щелкните правой кнопкой мыши только что созданную конфигурацию узла SSH.

1. Выберите **Подключение к узлу в текущем окне**. 

С этого момента вы полностью работаете с вычислительным экземпляром, и теперь можете редактировать, отлаживать, использовать Git, использовать расширения и т. д., как и в случае с локальными Visual Studio Code.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили Visual Studio Code удаленно, вы можете использовать вычислительный экземпляр в качестве удаленного вычислений от Visual Studio Code для [интерактивной отладки кода](how-to-debug-visual-studio-code.md).

[Руководство. Обучение модели Машинного обучения](tutorial-1st-experiment-sdk-train.md) — демонстрация использования вычислительного экземпляра со встроенной записной книжкой.

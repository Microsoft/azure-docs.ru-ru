---
title: PySpark интерактивной среды с помощью средств Azure HDInsight
description: Сведения о создании и отправке запросов и скриптов с помощью средств Azure HDInsight для Visual Studio Code.
keywords: VScode,средства Azure HDInsight,Hive,Python,PySpark,Spark,HDInsight,Hadoop,LLAP,Interactive Hive,Interactive Query
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-python
ms.date: 04/23/2020
ms.openlocfilehash: a6247116cdf579691e48883231f57712da36c4ad
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104864573"
---
# <a name="set-up-the-pyspark-interactive-environment-for-visual-studio-code"></a>Настройка интерактивной среды PySpark для Visual Studio Code

Ниже описано, как настроить интерактивную среду PySpark в VSCode. Этот шаг предназначен только для пользователей, не являющихся пользователями Windows.

Мы используем команду **python или pip** для сборки виртуального окружения в основной папке. Если вы хотите использовать другую версию, вам нужно вручную изменить версию по умолчанию команды **python или pip**. Дополнительные сведения см. в статье о команде [update-alternatives](https://linux.die.net/man/8/update-alternatives).

1. Установите [Python](https://www.python.org/downloads/) и [PIP](https://pip.pypa.io/en/stable/installing/).

   * Установите Python из [https://www.python.org/downloads/](https://www.python.org/downloads/) . 
   * Установите PIP с [https://pip.pypa.io/en/stable/installing](https://pip.pypa.io/en/stable/installing/) (если он не установлен из установки Python).
   * При необходимости убедитесь, что Python и PIP установлены успешно с помощью команд `python --version` и `pip --version` соответственно. 

     > [!NOTE]
     > Рекомендуется вручную установить Python вместо использования версии macOS по умолчанию.

2. Установите **virtualenv**, выполнив команду, приведенную ниже.

   ```bash
   pip install virtualenv
   ```

## <a name="other-packages"></a>другие пакеты.

В Linux, если вы используете приведенное ниже сообщение об ошибке, установите необходимые пакеты, выполнив следующие две команды.

   :::image type="content" source="./media/set-up-pyspark-interactive-environment/install-libkrb5-package.png" alt-text="Установка пакета libkrb5 для Python" border="true":::

```bash
sudo apt-get install libkrb5-dev
```

```bash
sudo apt-get install python-dev
```

Перезапустите VSCode, а затем вернитесь в редактор VSCode и выполните команду **Spark: PySPark Interactive** .

## <a name="next-steps"></a>Дальнейшие действия

### <a name="demo"></a>Демонстрация

* HDInsight для VS Code: [видео](https://go.microsoft.com/fwlink/?linkid=858706)

### <a name="tools-and-extensions"></a>Инструменты и расширения

* [Использование средств Azure HDInsight для Visual Studio Code](hdinsight-for-vscode.md)
* [Создание приложений Apache Spark для кластера HDInsight с помощью Azure Toolkit for IntelliJ](spark/apache-spark-intellij-tool-plugin.md)
* [Установка записной книжки Jupyter на компьютере и ее подключение к кластеру Apache Spark в Azure HDInsight (предварительная версия)](spark/apache-spark-jupyter-notebook-install-locally.md)

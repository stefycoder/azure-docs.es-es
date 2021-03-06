---
title: "MapReduce y conexión de SSH con Hadoop en HDInsight | Microsoft Docs"
description: "Obtenga más información sobre cómo usar SSH para ejecutar trabajos de MapReduce mediante Hadoop en HDInsight."
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 844678ba-1e1f-4fda-b9ef-34df4035d547
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 05/03/2017
ms.author: larryfr
ms.translationtype: Human Translation
ms.sourcegitcommit: 8f987d079b8658d591994ce678f4a09239270181
ms.openlocfilehash: d13e6782115bf4f229ed232dc6e3c21ff81c86eb
ms.contentlocale: es-es
ms.lasthandoff: 05/18/2017


---
# <a name="use-mapreduce-with-hadoop-on-hdinsight-with-ssh"></a>Uso de MapReduce con Hadoop en HDInsight con SSH

[!INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

Aprenda a enviar trabajos de MapReduce desde una conexión Secure Shell (SSH) a HDInsight.

> [!NOTE]
> Si ya está familiarizado con el uso de servidores de Hadoop basados en Linux, pero no conoce HDInsight, consulte [Información sobre el uso de HDInsight en Linux](hdinsight-hadoop-linux-information.md).

## <a id="prereq"></a>Requisitos previos

* Un clúster de HDInsight basado en Linux (Hadoop en HDInsight)

  > [!IMPORTANT]
  > Linux es el único sistema operativo que se usa en la versión 3.4 de HDInsight, o en las superiores. Consulte la información sobre la [retirada de HDInsight en Windows](hdinsight-component-versioning.md#hdi-version-33-nearing-retirement-date).

* Un cliente SSH. Para más información, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="ssh"></a>Conexión con SSH

Conéctese al clúster con SSH. Por ejemplo, el siguiente comando conecta con un clúster llamado **myhdinsight**:

    ssh admin@myhdinsight-ssh.azurehdinsight.net

**Si usa una clave de certificado para la autenticación SSH**, puede que deba especificar la ubicación de la clave privada en su sistema cliente, por ejemplo:

    ssh -i ~/mykey.key admin@myhdinsight-ssh.azurehdinsight.net

**Si usa una contraseña para la autenticación SSH**, deberá proporcionar la contraseña cuando se le solicite.

Para obtener más información sobre cómo utilizar SSH con HDInsight, consulte [Uso de SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="hadoop"></a>Uso de comandos Hadoop

1. Después de conectarse al clúster de HDInsight, use el siguiente comando para iniciar un trabajo de MapReduce:

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput
    ```

    Este comando inicia la clase `wordcount`, que está contenido en el archivo `hadoop-mapreduce-examples.jar`. Emplea como entrada el documento `/example/data/gutenberg/davinci.txt` y la salida se almacena en `/example/data/WordCountOutput`.

    > [!NOTE]
    > Para obtener más información sobre este trabajo de MapReduce y los datos de ejemplo, vea [Uso de MapReduce en Hadoop en HDInsight](hdinsight-use-mapreduce.md).

2. El trabajo emite detalles a medida que se procesa y devuelve información similar al siguiente texto cuando finaliza el trabajo:

        File Input Format Counters
        Bytes Read=1395666
        File Output Format Counters
        Bytes Written=337623

3. Una vez completado el trabajo, use el siguiente comando para enumerar los archivos de salida:

    ```bash
    hdfs dfs -ls /example/data/WordCountOutput
    ```

    Este comando muestra dos archivos, `_SUCCESS` y `part-r-00000`. El archivo `part-r-00000` contiene la salida de este trabajo.

    > [!NOTE]
    > Algunos trabajos de MapReduce pueden dividir los resultados entre varios archivos **part-r-####** . Si es así, utilice el sufijo #### para indicar el orden de los archivos.

4. Para ver la salida, use el comando siguiente:

    ```bash
    hdfs dfs -cat /example/data/WordCountOutput/part-r-00000
    ```

    Este comando muestra una lista de las palabras contenidas en el archivo **wasb://example/data/gutenberg/davinci.txt**, junto con el número de veces que aparecía cada palabra. El texto siguiente es un ejemplo de los datos contenidos en el archivo:

        wreathed        3
        wreathing       1
        wreaths         1
        wrecked         3
        wrenching       1
        wretched        6
        wriggling       1

## <a id="summary"></a>Resumen

Como se puede ver, los comando Hadoop proporcionan una manera fácil de ejecutar trabajos de MapReduce en un clúster de HDInsight y, a continuación, ver la salida del trabajo.

## <a id="nextsteps"></a>Pasos siguientes

Para obtener información general sobre los trabajos de MapReduce en HDInsight:

* [Uso de MapReduce en Hadoop de HDInsight](hdinsight-use-mapreduce.md)

Para obtener información sobre otras maneras de trabajar con Hadoop en HDInsight:

* [Uso de Hive con Hadoop en HDInsight](hdinsight-use-hive.md)
* [Uso de Pig con Hadoop en HDInsight](hdinsight-use-pig.md)


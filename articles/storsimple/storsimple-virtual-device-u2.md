---
title: "Actualización 2 del dispositivo virtual de StorSimple | Microsoft Docs"
description: "Obtenga información acerca de cómo crear, implementar y administrar un dispositivo virtual StorSimple en una red virtual de Microsoft Azure. (Se aplica a la actualización 2 de StorSimple)."
services: storsimple
documentationcenter: 
author: alkohli
manager: timlt
editor: 
ms.assetid: f37752a5-cd0c-479b-bef2-ac2c724bcc37
ms.service: storsimple
ms.devlang: NA
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 04/07/2017
ms.author: alkohli
translationtype: Human Translation
ms.sourcegitcommit: 538f282b28e5f43f43bf6ef28af20a4d8daea369
ms.openlocfilehash: d9ba200dba2b1460a9d3c9ba07e2bca54c30b53a
ms.lasthandoff: 04/07/2017


---
# <a name="deploy-and-manage-a-storsimple-virtual-device-in-azure"></a>Implementar y administrar un dispositivo virtual StorSimple en Azure
## <a name="overview"></a>Información general
El dispositivo virtual de StorSimple de la serie 8000 es una funcionalidad adicional que se incluye con la solución de Microsoft Azure StorSimple. El dispositivo virtual de StorSimple se ejecuta en una máquina virtual en una red virtual de Microsoft Azure y se puede usar para realizar copias de seguridad y clonar los datos de los hosts. Este tutorial describe cómo implementar y administrar un dispositivo virtual en Azure y es aplicable a todos los dispositivos virtuales que ejecutan la versión del software Update 2 u otra inferior.

#### <a name="virtual-device-model-comparison"></a>Comparación del modelo de dispositivo virtual
El dispositivo virtual StorSimple está disponible en dos modelos: estándar 8010 (anteriormente conocido como 1100) y premium 8020 (introducido en la actualización 2). A continuación se incluye una tabla comparativa de los dos modelos.

| Modelo de dispositivo | 8010<sup>1</sup> | 8020 |
| --- | --- | --- |
| **Capacidad máxima** |30 TB |64 TB |
| **MV de Azure** |Standard_A3 (4 núcleos, 7 GB de memoria) |Standard_DS3 (4 núcleos, 14 GB de memoria) |
| **Compatibilidad de versión** |Ejecuta las versiones previas a la actualización 2 o posterior |Ejecuta las versiones de la actualización 2 o posterior |
| **Disponibilidad en regiones** |Todas las regiones de Azure |Todas las regiones de Azure que admiten Premium Storage y máquinas virtuales de Azure DS3<br></br> Use [esta lista](https://azure.microsoft.com/en-us/regions/services) para ver si las dos opciones *Virtual Machines > Serie DS* y *Storage > Almacenamiento en disco* están disponibles en su región. |
| **Tipo de almacenamiento** |Usa el almacenamiento estándar de Azure para discos locales <br></br> Infórmese de cómo [crear una cuenta de almacenamiento estándar](../storage/storage-create-storage-account.md) |Usa el almacenamiento premium de Azure para discos locales<sup>2</sup> <br></br>Infórmese de cómo [crear una cuenta de Premium Storage ](../storage/storage-premium-storage.md) |
| **Guía de la carga de trabajo** |Recuperación a nivel de elemento de archivos de copias de seguridad |Escenarios de desarrollo y pruebas en la nube, baja latencia, mayores cargas de trabajo de rendimiento  <br></br>Dispositivo secundario para recuperación ante desastres |

<sup>1</sup> *Anteriormente conocido como 1100*.

<sup>2</sup> *8010 y 8020 usan el almacenamiento estándar de Azure para el nivel de nube. La diferencia solo existe en el nivel local del dispositivo*.

Este artículo describe paso a paso el proceso de implementación de un dispositivo virtual de StorSimple en Azure. Después de leer este artículo, habrá aprendido lo siguiente:

* En qué difiere el dispositivo virtual del dispositivo físico.
* Procedimiento a seguir para crear y configurar el dispositivo virtual.
* Realizar la conexión con la máquina virtual.
* Cómo trabajar con el dispositivo virtual.

Este tutorial se aplica a todos los dispositivos virtuales StorSimple que ejecuten la actualización 2 y posterior.

## <a name="how-the-virtual-device-differs-from-the-physical-device"></a>¿Cómo difiere el dispositivo virtual del dispositivo físico
El dispositivo virtual de StorSimple es una versión solo de software de StorSimple que se ejecuta en un único nodo en una máquina virtual de Microsoft Azure. El dispositivo virtual admite escenarios de recuperación ante desastres en los que el dispositivo físico no está disponible, y es adecuado para su uso en la recuperación a nivel de elemento de copias de seguridad, en la recuperación ante desastres local y en escenarios de prueba y desarrollo en la nube.

#### <a name="differences-from-the-physical-device"></a>Diferencias del dispositivo físico
La tabla siguiente muestra algunas diferencias clave entre el dispositivo virtual de StorSimple y el dispositivo físico StorSimple:

|  | Dispositivo físico | Dispositivo virtual |
| --- | --- | --- |
| **Ubicación** |Se encuentra en el centro de datos. |Se ejecuta en Azure. |
| **Interfaces de red** |Tiene seis interfaces de red: de DATA 0 a DATA 5. |Solo tiene una interfaz de red: DATA 0. |
| **Registro** |Registrado durante el paso de configuración. |Registrado a través de una tarea independiente. |
| **Clave de cifrado de datos de servicio** |Vuelve a generar en el dispositivo físico y, a continuación, actualiza el dispositivo virtual con la clave nueva. |No se puede volver a generar desde el dispositivo virtual. |

## <a name="prerequisites-for-the-virtual-device"></a>Requisitos previos para el dispositivo virtual
Las siguientes secciones explican los requisitos previos de configuración para el dispositivo virtual de StorSimple. Antes de implementar un dispositivo virtual, consulte el artículo [Protección de datos y seguridad de StorSimple](storsimple-security.md#storsimple-virtual-device-security).

#### <a name="azure-requirements"></a>Requisitos de Azure
Antes de aprovisionar el dispositivo virtual, deberá realizar los siguientes preparativos en el entorno de Azure:

* Para el dispositivo virtual, [configure una red virtual en Azure](../virtual-network/virtual-networks-create-vnet-classic-portal.md). Si usa Almacenamiento premium, tiene que crear una red virtual en una región de Azure que admita dicho almacenamiento. Las regiones Premium Storage son las que corresponden a la fila de *Almacenamiento en disco* de la lista de [servicios de Azure por región](https://azure.microsoft.com/en-us/regions/services).
* Puede usar el servidor DNS predeterminado proporcionado por Azure en lugar de especificar su propio nombre del servidor DNS. Si el nombre del servidor DNS no es válido o si el servidor DNS no es capaz de resolver las direcciones IP correctamente, no se podrá crear el dispositivo virtual.
* Punto a sitio y sitio a sitio son opcionales, pero no obligatorios. Si lo desea, puede configurar estas opciones para escenarios más avanzados.
* Puede crear [Máquinas virtuales de Azure](../virtual-machines/virtual-machines-linux-about.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (servidores host) en la red virtual que pueden usar los volúmenes expuestos por el dispositivo virtual. Estos servidores deben cumplir los siguientes requisitos:                             

  * Estar en máquinas virtuales de Windows o Linux con el software iSCSI Initiator instalado.
  * Ejecutarse en la misma red virtual como el dispositivo virtual.
  * Ser capaz de conectarse al destino iSCSI del dispositivo virtual a través de la dirección IP interna del dispositivo virtual.
* Asegúrese de que haya habilitado la compatibilidad para el tráfico iSCSI y en la nube en la misma red virtual.

#### <a name="storsimple-requirements"></a>Requisitos de StorSimple
Realice las siguientes actualizaciones en su servicio StorSimple de Azure antes de crear un dispositivo virtual:

* Agregue [registros de control de acceso](storsimple-manage-acrs.md) para las máquinas virtuales que vayan a ser servidores de host para el dispositivo virtual.
* Utilice una [cuenta de almacenamiento](storsimple-manage-storage-accounts.md#add-a-storage-account) en la misma región que el dispositivo virtual. Las cuentas de almacenamiento en regiones diferentes pueden causar un bajo rendimiento. Puede usar una cuenta de almacenamiento estándar o premium con el dispositivo virtual. Obtenga más información sobre cómo crear una [cuenta de Standard Storage](../storage/storage-create-storage-account.md) o una [cuenta de Premium Storage](../storage/storage-premium-storage.md).
* Para crear el dispositivo virtual use una cuenta diferente de la que usa para los datos. Utilizar la misma cuenta de almacenamiento puede causar un bajo rendimiento.

Asegúrese de que tiene la siguiente información antes de empezar:

* Cuenta del Portal de Azure clásico con credenciales de acceso.
* Una copia de la clave de cifrado de datos del servicio del dispositivo físico.

## <a name="create-and-configure-the-virtual-device"></a>Creación y configuración del dispositivo virtual
Antes de realizar estos procedimientos, asegúrese de que se han cumplido los [requisitos previos para el dispositivo virtual](#prerequisites-for-the-virtual-device).

Después de crear una red virtual, configurar un servicio del Administrador de StorSimple y registrar el dispositivo físico de StorSimple con el servicio, siga los pasos a continuación para crear un dispositivo virtual de StorSimple.

### <a name="step-1-create-a-virtual-device"></a>Paso 1: Creación de un dispositivo virtual
Realice los pasos siguientes para crear el dispositivo virtual de StorSimple.

[!INCLUDE [Create a virtual device](../../includes/storsimple-create-virtual-device-u2.md)]

Si se produce un error en este paso durante la creación del dispositivo virtual, quizás no tenga conexión a Internet. Para más información, vea cómo [solucionar problemas de conexión a Internet](#troubleshoot-internet-connectivity-errors) al crear un dispositivo virtual.

### <a name="step-2-configure-and-register-the-virtual-device"></a>Paso 2: Configuración y registro del dispositivo virtual
Antes de comenzar este procedimiento, asegúrese de que tiene una copia de la clave de cifrado de datos del servicio. La clave de cifrado de datos del servicio se creó cuando se configuró el primer dispositivo de StorSimple y se le pidió que lo guardara en una ubicación segura. Si no tiene una copia de la clave de cifrado de datos del servicio, debe ponerse en contacto con Microsoft Support para obtener ayuda.

Realice los pasos siguientes para configurar y registrar el dispositivo virtual de StorSimple.

[!INCLUDE [Configure and register a virtual device](../../includes/storsimple-configure-register-virtual-device.md)]

### <a name="step-3-optional-modify-the-device-configuration-settings"></a>Paso 3: (Opcional) Modificación de la configuración del dispositivo
En la siguiente sección se describen las opciones de configuración de dispositivo necesarias para el dispositivo virtual de StorSimple si desea usar CHAP, Administrador de instantáneas StorSimple, o si quiere cambiar la contraseña del administrador de dispositivos.

#### <a name="configure-the-chap-initiator"></a>Configuración del iniciador CHAP
Este parámetro contiene las credenciales que espera el dispositivo virtual (destino) de los iniciadores (servidores) que intentan obtener acceso a los volúmenes. Los iniciadores proporcionarán un nombre de usuario y una contraseña CHAP para identificarse en el dispositivo durante la autenticación. Para obtener los pasos detallados necesarios, vaya a [Configurar CHAP para el dispositivo StorSimple](storsimple-configure-chap.md#unidirectional-or-one-way-authentication).

#### <a name="configure-the-chap-target"></a>Configuración del destino CHAP
Este parámetro contiene las credenciales que el dispositivo virtual utiliza cuando un iniciador habilitado para CHAP solicita la autenticación mutua o bidireccional. El dispositivo virtual utilizará un nombre de usuario de CHAP inverso y una contraseña de CHAP inversa para identificarse con el iniciador durante este proceso de autenticación. Tenga en cuenta que los valores de destino de CHAP son valores globales. Cuando se aplican, todos los volúmenes conectados al dispositivo virtual de almacenamiento utilizan la autenticación CHAP. Para obtener los pasos detallados necesarios, vaya a [Configurar CHAP para el dispositivo StorSimple](storsimple-configure-chap.md#bidirectional-or-mutual-authentication).

#### <a name="configure-the-storsimple-snapshot-manager-password"></a>Configuración de la contraseña de StorSimple Snapshot Manager
El software StorSimple Snapshot Manager reside en el host de Windows y permite a los administradores administrar copias de seguridad del dispositivo StorSimple en forma de instantáneas locales y en la nube.

> [!NOTE]
> Para el dispositivo virtual, el host de Windows es una máquina virtual de Azure.
>
>

Al configurar un dispositivo en StorSimple Snapshot Manager, se le pedirá que proporcione la dirección IP del dispositivo StorSimple y la contraseña para autenticar el dispositivo de almacenamiento. Para obtener los pasos detallados necesarios, vaya a [Cambio de la contraseña de StorSimple Snapshot Manager](storsimple-change-passwords.md#change-the-storsimple-snapshot-manager-password)

#### <a name="change-the-device-administrator-password"></a>Cambio de la contraseña del administrador de dispositivos
Cuando se utiliza la interfaz de Windows PowerShell para tener acceso al dispositivo virtual, se le pedirá que escriba una contraseña de administrador de dispositivos. Por seguridad de sus datos, es necesario cambiar esta contraseña para poder usar el dispositivo virtual. Para obtener los pasos detallados necesarios, vaya a [Cambio de la contraseña del administrador de dispositivos](storsimple-change-passwords.md#change-the-device-administrator-password).

## <a name="connect-remotely-to-the-virtual-device"></a>Conexión remota con el dispositivo virtual.
El acceso remoto a su dispositivo virtual mediante la interfaz de Windows PowerShell no está habilitado de forma predeterminada. Deberá habilitar primero la administración remota en el dispositivo virtual y, a continuación, habilitarlo en el cliente que se utilizará para acceder al dispositivo virtual.

A continuación se detalla los dos pasos del procedimiento para conectarse de forma remota.

### <a name="step-1-configure-remote-management"></a>Paso 1: Configuración de la administración remota
Realice los pasos siguientes para configurar la administración remota para el dispositivo virtual de StorSimple.

[!INCLUDE [Configure remote management via HTTP for virtual device](../../includes/storsimple-configure-remote-management-http-device.md)]

### <a name="step-2-remotely-access-the-virtual-device"></a>Paso 2: Obtención de acceso remoto al dispositivo virtual
Una vez habilitada la administración remota en la página de configuración del dispositivo StorSimple, puede usar la comunicación remota de Windows PowerShell para conectarse al dispositivo virtual desde otra máquina virtual dentro de la misma red virtual; por ejemplo, puede conectarse desde la máquina virtual del host que ha configurado y utilizado para conectarse a iSCSI. En la mayoría de las implementaciones, ya habrá abierto un punto de conexión público para tener acceso a su máquina virtual host que se puede utilizar para tener acceso al dispositivo virtual.

> [!WARNING]
> **Para mayor seguridad, se recomienda utilizar HTTPS al conectarse a los extremos y, a continuación, eliminar los extremos después de haber completado la sesión remota de PowerShell.**
>
>

Debe seguir los procedimientos que aparecen en [Conectarse de forma remota al dispositivo StorSimple](storsimple-remote-connect.md) para establecer la comunicación remota para el dispositivo virtual.

## <a name="connect-directly-to-the-virtual-device"></a>Conexión directa con el dispositivo virtual.
También puede realizar una conexión directa con el dispositivo virtual. Si desea conectarse directamente al dispositivo virtual desde otro equipo fuera de la red virtual o fuera del entorno de Microsoft Azure, tendrá que crear puntos de conexión adicionales, como se describe en el procedimiento siguiente.

Realice los pasos siguientes para crear un punto de conexión público en el dispositivo virtual.

[!INCLUDE [Create public endpoints on a virtual device](../../includes/storsimple-create-public-endpoints-virtual-device.md)]

Se recomienda conectarse desde otra máquina virtual dentro de la misma red virtual, ya que esta práctica minimiza el número de puntos de conexión públicos de la red virtual. Cuando utiliza este método, simplemente conéctese a la máquina virtual mediante una sesión de escritorio remoto y, a continuación, configure esa máquina virtual para su uso como haría con cualquier otro cliente de Windows en una red local. No es necesario anexar el número de puerto público porque ya se conocerá el puerto.

## <a name="work-with-the-storsimple-virtual-device"></a>Trabajo con el dispositivo virtual de StorSimple
Ahora que ha creado y configurado el dispositivo virtual de StorSimple, está listo para comenzar a trabajar con él. Puede trabajar con contenedores de volúmenes, volúmenes y directivas de copia de seguridad en un dispositivo virtual como lo haría en un dispositivo físico de StorSimple; la única diferencia es que necesita asegurarse de que selecciona el dispositivo virtual en la lista de dispositivos. Consulte [Utilizar el servicio de Administrador de StorSimple para administrar su dispositivo StorSimple](storsimple-manager-service-administration.md) para conocer los procedimientos paso a paso de las diversas tareas de administración para el dispositivo virtual.

Las secciones siguientes describen algunas de las diferencias que surgen al trabajar con el dispositivo virtual.

### <a name="maintain-a-storsimple-virtual-device"></a>Mantenimiento de un dispositivo virtual de StorSimple
Dado que es un dispositivo solo de software, el mantenimiento del dispositivo virtual es mínimo en comparación con el mantenimiento del dispositivo físico. Tiene las siguientes opciones:

* **Actualizaciones de software** : puede ver la fecha en que se actualizó por última vez el software, junto con cualquier actualización de los mensajes de estado. Puede utilizar el botón **Buscar actualizaciones** en la parte inferior de la página para realizar una búsqueda manual si desea comprobar si hay nuevas actualizaciones. Si hay actualizaciones disponibles, haga clic en **Instalar actualizaciones** para instalar. Puesto que solo hay una única interfaz en el dispositivo virtual, significa que habrá una pequeña interrupción del servicio cuando se apliquen las actualizaciones. El dispositivo virtual se apagará automáticamente y se reiniciará (si es necesario) para aplicar las actualizaciones que se han publicado. Para obtener un procedimiento paso a paso, vaya a [Actualización del dispositivo de la serie StorSimple 8000](storsimple-update-device.md#install-regular-updates-via-the-azure-classic-portal).
* **Paquete de soporte** : puede crear y cargar un paquete de soporte para ayudar al Soporte de Microsoft a solucionar problemas con el dispositivo virtual. Para obtener un procedimiento paso a paso, vaya a [Crear y administrar paquetes de soporte técnico de StorSimple](storsimple-create-manage-support-package.md).

### <a name="storage-accounts-for-a-virtual-device"></a>Cuentas de almacenamiento para un dispositivo virtual
Las cuentas de almacenamiento se crean para su uso por el servicio StorSimple Manager, el dispositivo virtual y el dispositivo físico. Al crear las cuentas de almacenamiento, se recomienda usar un identificador de región en el nombre descriptivo para ayudar a garantizar que la región sea coherente en todos los componentes del sistema. Para un dispositivo virtual, es importante que todos los componentes estén en la misma región para evitar problemas de rendimiento.

Para conocer el procedimiento paso a paso, vaya a [Agregar una cuenta de almacenamiento](storsimple-manage-storage-accounts.md#add-a-storage-account).

### <a name="deactivate-a-storsimple-virtual-device"></a>Desactivación de un dispositivo virtual de StorSimple
Desactivar un dispositivo virtual elimina la máquina virtual y los recursos que se crean cuando se aprovisionaron los servicios. Después de desactivar el dispositivo virtual, no se puede restaurar a su estado anterior. Antes de desactivar el dispositivo virtual, asegúrese de detener o eliminar clientes y hosts que dependen de él.

Desactivar un dispositivo virtual genera las siguientes acciones:

* Se quita el dispositivo virtual.
* Se quitan los discos del sistema operativo y de los datos creados para el dispositivo virtual.
* Se mantienen el servicio hospedado y la red virtual creados durante el aprovisionamiento. Si no los utiliza, debe eliminarlos manualmente.
* Se conservan las instantáneas de nube creadas para el dispositivo virtual.

Para obtener un procedimiento paso a paso, consulte [Desactivación y eliminación de un dispositivo de StorSimple](storsimple-deactivate-and-delete-device.md).

Tan pronto como se muestre el dispositivo como desactivado en la página de servicio del Administrador de StorSimple, puede eliminar el dispositivo virtual de la lista en la página **Dispositivos** .

### <a name="start-stop-and-restart-a-virtual-device"></a>Inicio, detención y reinicio de un dispositivo virtual
A diferencia del dispositivo físico de StorSimple, en un dispositivo virtual de StorSimple no hay botón de encendido o apagado. Sin embargo, puede haber ocasiones en las que necesite detener y reiniciar el dispositivo virtual. Por ejemplo, algunas actualizaciones pueden requerir que se reinicie la máquina virtual para finalizar el proceso de actualización. La manera más fácil para iniciar, detener y reiniciar un dispositivo virtual es utilizar la consola de administración de máquinas virtuales.

Cuando observa la consola de administración, el estado del dispositivo virtual es **En ejecución** porque se inicia de forma predeterminada después de crearlo. Puede iniciar, detener y reiniciar una máquina virtual en cualquier momento.

[!INCLUDE [Stop and restart virtual device](../../includes/storsimple-stop-restart-virtual-device.md)]

### <a name="reset-to-factory-defaults"></a>Restablecimiento de los valores predeterminados de fábrica
Si decide que desea empezar de nuevo con su dispositivo virtual, simplemente desactívelo y elimínelo y, a continuación, cree uno nuevo. Al igual que cuando se restablece el dispositivo físico, el dispositivo virtual nuevo no tendrá las actualizaciones instaladas; por lo tanto, asegúrese de comprobar si hay actualizaciones antes de usarlo.

## <a name="fail-over-to-the-virtual-device"></a>Conmutación por error en el dispositivo virtual
La recuperación ante desastres (DR) es uno de los escenarios clave para los que se diseñó el dispositivo virtual de StorSimple. En este escenario, es posible que el dispositivo físico de StorSimple o todo el centro de datos no esté disponible. Afortunadamente, puede usar un dispositivo virtual para restaurar las operaciones en una ubicación alternativa. Durante la recuperación ante desastres, los contenedores de volúmenes del dispositivo de origen cambia la propiedad y se transfieren al dispositivo virtual. Los requisitos previos para la recuperación ante desastres son que haya creado y configurado el dispositivo virtual, todos los volúmenes en el contenedor de volúmenes se hayan desconectado y el contenedor de volúmenes tenga asociada una instantánea en la nube.

> [!NOTE]
> * Al utilizar un dispositivo virtual como el dispositivo secundario para recuperación ante desastres, tenga en cuenta que el 8010 tiene 30 TB de almacenamiento estándar y 8020 tiene 64 TB de almacenamiento premium. La mayor capacidad del 8020 puede resultar más adecuada para un escenario de recuperación ante desastres.
> * No se puede efectuar la conmutación por error ni clonar desde un dispositivo que ejecute la actualización 2 a un dispositivo que ejecute el software anterior a la actualización 1. Sin embargo puede conmutar por error un dispositivo que ejecuta la actualización 2 a un dispositivo que ejecuta la actualización 1 (1.1 o 1.2)
>
>

Para obtener un procedimiento paso a paso, vaya a [Conmutación por error y recuperación ante desastres para el dispositivo StorSimple](storsimple-device-failover-disaster-recovery.md#fail-over-to-a-storsimple-virtual-device).

## <a name="shut-down-or-delete-the-virtual-device"></a>Cerrar o eliminar el dispositivo virtual
Si ha configurado y usado previamente un dispositivo virtual de StorSimple, pero ahora desea detener la acumulación de cargos para su uso por el proceso, puede apagar el dispositivo virtual. Apagar el dispositivo virtual no elimina su sistema operativo ni los discos de datos del almacenamiento. Detiene el cargo acumulado en su suscripción, pero seguirán los cargos de almacenamiento del sistema operativo y los discos de datos.

Si elimina o apaga el dispositivo virtual, aparecerá como **Desconectado** en la página de dispositivos del servicio StorSimple Manager. Puede desactivar o eliminar el dispositivo si desea eliminar las copias de seguridad creadas por el dispositivo virtual. Para más información, consulte [Desactivación y eliminación de un dispositivo de StorSimple](storsimple-deactivate-and-delete-device.md).

[!INCLUDE [Shut down a virtual device](../../includes/storsimple-shutdown-virtual-device.md)]

[!INCLUDE [Delete a virtual device](../../includes/storsimple-delete-virtual-device.md)]

## <a name="troubleshoot-internet-connectivity-errors"></a>Solucionar errores de conexión a Internet
Si no hay ninguna conexión a Internet, se producirá un error durante la creación de un dispositivo virtual. Para solucionar problemas debidos a la conexión a Internet, realice los pasos siguientes en el Portal de Azure clásico:

1. Cree una máquina virtual de Windows Server 2012 en Azure. Esta máquina virtual debe usar la misma cuenta de almacenamiento, red virtual y subred que usa el dispositivo virtual. Si ya tiene un host de Windows Server en Azure con la misma cuenta de almacenamiento, red virtual y subred, también puede usarlo para solucionar los problemas de conexión a Internet.
2. Inicie sesión de forma remota en la máquina virtual creada en el paso anterior.
3. Abra una ventana de comandos en la máquina virtual (Win + R y, después, escriba `cmd`).
4. En el símbolo del sistema, ejecute el siguiente comando.

    `nslookup windows.net`
5. Si `nslookup` produce un error, quiere decir que un error de conexión a Internet impide que el dispositivo virtual se registre en el servicio StorSimple Manager.
6. Realice los cambios necesarios en la red virtual para garantizar que el dispositivo virtual pueda acceder a sitios de Azure como "windows.net".

## <a name="next-steps"></a>Pasos siguientes
* Aprenda a [usar el servicio StorSimple Manager para administrar un dispositivo virtual](storsimple-manager-service-administration.md).
* Sepa cómo [restaurar un volumen de StorSimple de un conjunto de copias de seguridad](storsimple-restore-from-backup-set.md).


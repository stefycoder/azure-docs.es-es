---
title: "Azure Virtual Networks y máquinas virtuales de Linux | Microsoft Docs"
description: "Tutorial: Administración de Azure Virtual Networks y máquinas virtuales de Linux con la CLI de Azure"
services: virtual-machines-linux
documentationcenter: virtual-machines
author: neilpeterson
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/10/2017
ms.author: nepeters
ms.translationtype: Human Translation
ms.sourcegitcommit: 44eac1ae8676912bc0eb461e7e38569432ad3393
ms.openlocfilehash: e843e444d2fe32f578c5a887b606db982920a9e0
ms.contentlocale: es-es
ms.lasthandoff: 05/17/2017

---

# <a name="manage-azure-virtual-networks-and-linux-virtual-machines-with-the-azure-cli"></a>Administrar Azure Virtual Networks y máquinas virtuales de Linux con la CLI de Azure

Las máquinas virtuales de Azure utilizan las redes de Azure para la comunicación de red interna y externa. Este tutorial le guía a través de la implementación de dos máquinas virtuales y la configuración de redes de Azure para estas máquinas virtuales. Se da por supuesto que en los ejemplos de este tutorial las máquinas virtuales hospedan una aplicación web con un back-end de base de datos, sin embargo, no se implementa ninguna aplicación en el tutorial. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Implementar una red virtual
> * Crear una subred dentro de una red virtual
> * Conectar máquinas virtuales a una subred
> * Administrar direcciones IP públicas de máquinas virtuales
> * Proteger el tráfico entrante de Internet
> * Protección de máquinas virtuales para el tráfico de máquinas virtuales

Para realizar este tutorial es necesaria la versión 2.0.4 o superior de la CLI de Azure. Para buscar la versión de la CLI, ejecute `az --version`. Si necesita actualizarla, consulte [Instalación de la CLI de Azure 2.0]( /cli/azure/install-azure-cli). También puede usar [Cloud Shell](/azure/cloud-shell/quickstart) desde el explorador.

## <a name="vm-networking-overview"></a>Introducción a las redes de máquinas virtuales

Las redes virtuales de Azure habilitan las conexiones de red seguras entre máquinas virtuales, Internet y otros servicios de Azure SQL Database. Las redes virtuales se dividen en segmentos lógicos llamados subredes. Las subredes se utilizan para controlar el flujo de red y como límite de seguridad. La implementación de una máquina virtual incluye, por lo general, una interfaz de red virtual, que está conectada a una subred.

## <a name="deploy-virtual-network"></a>Implementación de una red virtual

Para este tutorial, se crea una única red virtual con dos subredes: una subred de front-end para hospedar una aplicación web y una subred de back-end para hospedar un servidor de bases de datos.

Antes de poder crear una red virtual, cree un grupo de recursos con [az group create](/cli/azure/group#create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myRGNetwork* en la ubicación eastus.

```azurecli
az group create --name myRGNetwork --location eastus
```

### <a name="create-virtual-network"></a>Creación de una red virtual

Cree una red virtual con el comando [az network vnet create](/cli/azure/network/vnet#create). En este ejemplo, la red se denomina *mvVnet* y se le asigna un prefijo de dirección de *10.0.0.0/16*. También se crea una subred con el nombre de *mySubnetFrontEnd* y un prefijo de *10.0.1.0/24*. Más adelante en este tutorial, se va a conectar una máquina virtual de front-end a esta subred. 

```azurecli
az network vnet create \
  --resource-group myRGNetwork \
  --name myVnet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnetFrontEnd \
  --subnet-prefix 10.0.1.0/24
```

### <a name="create-subnet"></a>Creación de una subred

Para agregar una subred nueva a la red virtual, se utiliza el comando [az network vnet subnet create](/cli/azure/network/vnet/subnet#create). En este ejemplo, la subred se denomina *mySubnetBackEnd* y se le asigna un prefijo de dirección de *10.0.2.0/24*. Esta subred se usa con todos los servicios back-end.

```azurecli
az network vnet subnet create \
  --resource-group myRGNetwork \
  --vnet-name myVnet \
  --name mySubnetBackEnd \
  --address-prefix 10.0.2.0/24
```

Entonces, se ha creado una red y se ha segmentado en dos subredes, una para servicios front-end y otra para servicios back-end. En la siguiente sección, se crean máquinas virtuales y se conectan a estas subredes.

## <a name="understand-public-ip-address"></a>Información sobre la dirección IP pública

Una dirección IP pública permite que los recursos de Azure estén accesibles en Internet. En esta sección del tutorial, se crea una máquina virtual para demostrar cómo se trabaja con direcciones IP públicas.

### <a name="allocation-method"></a>Método de asignación

Una dirección IP pública puede asignarse ya sea de forma dinámica o estática. De forma predeterminada, la dirección IP pública se asigna dinámicamente. Las direcciones IP dinámicas se liberan al desasignar una máquina virtual. Este comportamiento hace que la dirección IP cambie durante cualquier operación que incluya una desasignación de máquina virtual.

El método de asignación se puede establecer en estático, lo que garantiza que la dirección IP siga asignada a una máquina virtual, incluso durante un estado de desasignación. Cuando se usa una dirección IP asignada estáticamente, no se puede especificar la propia dirección IP, sino que se asigna desde un grupo de direcciones disponibles.

### <a name="dynamic-allocation"></a>Asignación dinámica

Al crear una máquina virtual con el comando [az vm create](/cli/azure/vm#create), el método predeterminado para la asignación de una dirección IP pública es dinámico. En el ejemplo siguiente, se crea una máquina virtual con una dirección IP dinámica. 

```azurecli
az vm create \
  --resource-group myRGNetwork \
  --name myFrontEndVM \
  --vnet-name myVnet \
  --subnet mySubnetFrontEnd \
  --nsg myNSGFrontEnd \
  --public-ip-address myFrontEndIP \
  --image UbuntuLTS \
  --generate-ssh-keys
```

### <a name="static-allocation"></a>Asignación estática

Al crear una máquina virtual mediante el comando [az vm create](/cli/azure/vm#create), incluya el argumento `--public-ip-address-allocation static` para asignar una dirección IP pública estática. Esta operación no se muestra en este tutorial, sin embargo, en la sección siguiente se cambia una dirección IP asignada dinámicamente a una dirección asignada estáticamente. 

### <a name="change-allocation-method"></a>Cambio del método de asignación

Se puede cambiar el método de asignación de dirección IP mediante el comando [az network public-ip update](/cli/azure/network/public-ip#update). En este ejemplo, se cambia el método de asignación de dirección IP de la máquina virtual de front-end a estático.

En primer lugar, desasigne la máquina virtual.

```azurecli
az vm deallocate --resource-group myRGNetwork --name myFrontEndVM
```

Use el comando [az network public-ip update](/azure/network/public-ip#update) para actualizar el método de asignación. En este caso, `--allocaion-metod` se establece en *estático*.

```azurecli
az network public-ip update --resource-group myRGNetwork --name myFrontEndIP --allocation-method static
```

Inicie la máquina virtual.

```azurecli
az vm start --resource-group myRGNetwork --name myFrontEndVM --no-wait
```

### <a name="no-public-ip-address"></a>Sin dirección IP pública

A menudo, no es necesario que una máquina virtual sea accesible a través de Internet. Para crear una máquina virtual sin una dirección IP pública, use el argumento `--public-ip-address ""` con un conjunto vacío de comillas dobles. Esta configuración se muestra más adelante en este tutorial.

## <a name="secure-network-traffic"></a>Protegen el tráfico de red.

Un grupo de seguridad de red (NSG) contiene una lista de reglas de seguridad que permiten o deniegan el tráfico de red a recursos conectados a redes virtuales de Azure (VNet). Los NSG se pueden asociar a subredes o a interfaces de red individuales. Cuando un NSG está asociado a una interfaz de red, se aplica solo a la máquina virtual asociada. Cuando un grupo de seguridad de red está asociado a una subred, las reglas se aplican a todos los recursos conectados a la subred. 

### <a name="network-security-group-rules"></a>Reglas del grupo de seguridad de red

Las reglas de NSG definen puertos de redes a través de los que se permite o deniega el tráfico. Las reglas pueden incluir intervalos de direcciones IP de origen y destino para que el tráfico se controle entre sistemas o subredes específicos. Las reglas de NSG también incluyen una prioridad (entre 1 y 4096). Las reglas se evalúan por orden de prioridad. Una regla con una prioridad de 100 se evalúa antes que una regla con prioridad de 200.

Todos los grupos de seguridad de red contienen un conjunto de reglas predeterminadas. No se pueden eliminar las reglas predeterminadas, pero dado que tienen asignada la mínima prioridad, pueden reemplazarse por las reglas que cree.

- **Red virtual:** el tráfico que se origina y termina en una red virtual se permite en las direcciones tanto de entrada como de salida.
- **Internet:** se permite el tráfico saliente pero se bloquea el entrante.
- **Equilibrador de carga:** se permite que el equilibrador de carga de Azure sondee el estado de las máquinas virtuales y las instancias de rol. Si no va a usar un conjunto con equilibrio de carga, puede invalidar esta regla.

### <a name="create-network-security-groups"></a>Creación de grupos de seguridad de red

Un grupo de seguridad de red puede crearse a la vez como máquina virtual mediante el comando [az vm create](/cli/azure/vm#create). De este modo, el NSG está asociado a la interfaz de red de máquinas virtuales y se crea automáticamente una regla de NSG para permitir el tráfico en el puerto *22* desde cualquier destino. Anteriormente en este tutorial, se ha creado automáticamente el NSG de front-end con la máquina virtual de front-end. También se crea automáticamente una regla de NSG para el puerto 22. 

En algunos casos, puede resultar útil crear previamente un NSG, por ejemplo, cuando no se deben crear reglas predeterminadas de SSH o cuando el NSG debe asociarse a una subred. 

Utilice el comando [az network nsg create](/cli/azure/network/nsg#create) para crear un grupo de seguridad de red.

```azurecli
az network nsg create --resource-group myRGNetwork --name myNSGBackEnd
```

En lugar de asociar el NSG a una interfaz de red, está asociado a una subred. En esta configuración, todas las máquinas virtuales que están conectadas a la subred heredan las reglas de NSG.

Actualice la subred existente denominada *mySubnetBackEnd* con el nuevo NSG.

```azurecli
az network vnet subnet update \
  --resource-group myRGNetwork \
  --vnet-name myVnet \
  --name mySubnetBackEnd \
  --network-security-group myNSGBackEnd
```

Ahora cree una máquina virtual que esté asociada a *mySubnetBackEnd*. Tenga en cuenta que el argumento `--nsg` tiene un valor de comillas dobles vacías. No es obligatorio crear un NSG con la máquina virtual. La máquina virtual está conectada a la subred de back-end que está protegida con el NSG de back-end creado previamente. Este NSG se aplica a la máquina virtual. Además, tenga en cuenta aquí que el argumento `--public-ip-address` tiene un valor de comillas dobles vacías. Esta configuración crea una máquina virtual sin una dirección IP pública. 

```azurecli
az vm create \
  --resource-group myRGNetwork \
  --name myBackEndVM \
  --vnet-name myVnet \
  --subnet mySubnetBackEnd \
  --public-ip-address "" \
  --nsg "" \
  --image UbuntuLTS \
  --generate-ssh-keys
```

### <a name="secure-incoming-traffic"></a>Protección del tráfico entrante

Al crear la máquina virtual de front-end, se ha creado una regla de NSG para permitir el tráfico entrante en el puerto 22. Esta regla permite las conexiones SSH con la máquina virtual. En este ejemplo, también se debería permitir el tráfico en el puerto *80*. Esta configuración permite tener acceso a una aplicación web en la máquina virtual.

Utilice el comando [az network nsg rule create](/cli/azure/network/nsg/rule#create) para crear una regla para el puerto *80*.

```azurecli
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myNSGFrontEnd \
  --name http \
  --access allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 200 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range 80
```

La máquina virtual de front-end solo está accesible ahora en el puerto *22* y el puerto *80*. El resto del tráfico entrante se bloquea en el grupo de seguridad de red. Puede ser útil visualizar las configuraciones de reglas de NSG. Se puede volver a la configuración de las reglas de NSG con el comando [az network rule list](/cli/azure/network/nsg/rule#list). 

```azurecli
az network nsg rule list --resource-group myRGNetwork --nsg-name myNSGFrontEnd --output table
```

Salida:

```azurecli
Access    DestinationAddressPrefix      DestinationPortRange  Direction    Name                 Priority  Protocol    ProvisioningState    ResourceGroup    SourceAddressPrefix    SourcePortRange
--------  --------------------------  ----------------------  -----------  -----------------  ----------  ----------  -------------------  ---------------  ---------------------  -----------------
Allow     *                                               22  Inbound      default-allow-ssh        1000  Tcp         Succeeded            myRGNetwork      *                      *
Allow     *                                               80  Inbound      http                      200  Tcp         Succeeded            myRGNetwork      *                      *
```

### <a name="secure-vm-to-vm-traffic"></a>Protección de máquinas virtuales para el tráfico de máquinas virtuales

También pueden aplicar reglas del grupo de seguridad de red entre máquinas virtuales. En este ejemplo, la máquina virtual de front-end necesita comunicarse con la máquina virtual de back-end en el puerto *22* y el *3306*. Esta configuración permite las conexiones SSH desde la máquina virtual de front-end y también permite que una aplicación de la máquina virtual de front-end se comunique con una base de datos MySQL de back-end. Se debe bloquear el resto del tráfico entre las máquinas virtuales de front-end y back-end.

Utilice el comando [az network nsg rule create](/cli/azure/network/nsg/rule#create) para crear una regla para el puerto 22. Tenga en cuenta que el argumento `--source-address-prefix` especifica un valor de *10.0.1.0/24*. Esta configuración garantiza que solo se permite el tráfico de la subred de front-end a través de NSG.

```azurecli
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myNSGBackEnd \
  --name SSH \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 100 \
  --source-address-prefix 10.0.1.0/24 \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "22"
```

Ahora, agregue una regla para el tráfico de MySQL en el puerto 3306.

```azurecli
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myNSGBackEnd \
  --name MySQL \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 200 \
  --source-address-prefix 10.0.1.0/24 \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "3306"
```

Por último, como los NSG tienen una regla predeterminada que permite todo el tráfico entre las máquinas virtuales de la misma red virtual, se puede crear una regla para que los NSG de back-end bloqueen todo el tráfico. Observe que a `--priority` se le asigna un valor de *300*, que es menor que las reglas de NSG y MySQL. Esta configuración garantiza que se sigue permitiendo el tráfico SSH y MySQL a través de NSG.

```azurecli
az network nsg rule create \
  --resource-group myRGNetwork \
  --nsg-name myNSGBackEnd \
  --name denyAll \
  --access Deny \
  --protocol Tcp \
  --direction Inbound \
  --priority 300 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "*" \
  --destination-port-range "*"
```

La máquina virtual de back-end solo está accesible ahora en el puerto *22* y el puerto *3306* desde la subred de front-end. El resto del tráfico entrante se bloquea en el grupo de seguridad de red. Puede ser útil visualizar las configuraciones de reglas de NSG. Se puede volver a la configuración de las reglas de NSG con el comando [az network rule list](/cli/azure/network/nsg/rule#list). 

```azurecli
az network nsg rule list --resource-group myRGNetwork --nsg-name myNSGBackEnd --output table
```

Salida:

```azurecli
Access    DestinationAddressPrefix    DestinationPortRange    Direction    Name       Priority  Protocol    ProvisioningState    ResourceGroup    SourceAddressPrefix    SourcePortRange
--------  --------------------------  ----------------------  -----------  -------  ----------  ----------  -------------------  ---------------  ---------------------  -----------------
Allow     *                           22                      Inbound      SSH             100  Tcp         Succeeded            myRGNetwork      10.0.1.0/24            *
Allow     *                           3306                    Inbound      MySQL           200  Tcp         Succeeded            myRGNetwork      10.0.1.0/24            *
Deny      *                           *                       Inbound      denyAll         300  Tcp         Succeeded            myRGNetwork      *                      *
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado y protegido redes de Azure cuando están relacionadas con máquinas virtuales. Ha aprendido a:

> [!div class="checklist"]
> * Implementar una red virtual
> * Crear una subred dentro de una red virtual
> * Conectar máquinas virtuales a una subred
> * Administrar direcciones IP públicas de máquinas virtuales
> * Proteger el tráfico entrante de Internet
> * Protección de máquinas virtuales para el tráfico de máquinas virtuales

Prosiga en el siguiente tutorial para aprender a proteger los datos en máquinas virtuales mediante la copia de seguridad de Azure. 

> [!div class="nextstepaction"]
> [Copia de seguridad de máquinas virtuales Linux en Azure](./tutorial-backup-vms.md)

---
title: "Tutorial: Configuración de Workday para la sincronización de entrada | Microsoft Docs"
description: Aprenda a usar Workday como origen de datos de identidad para Azure Active Directory.
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 1a2c375a-1bb1-4a61-8115-5a69972c6ad6
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/24/2017
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: c18c92a1ff7d98b61c09138c492aa78dc99ec1cf
ms.lasthandoff: 04/03/2017


---
# <a name="tutorial-configure-workday-for-inbound-synchronization"></a>Tutorial: Configuración de Workday para la sincronización de entrada
El objetivo de este tutorial es mostrar los pasos que se deben realizar en Workday y Azure AD para importar contactos de Workday a Azure AD. 

En la situación descrita en este tutorial se supone que ya cuenta con los elementos siguientes:

* Una suscripción válida a Azure AD Premium
* Un inquilino en Workday

La situación descrita en este tutorial consta de los siguientes bloques de creación:

1. Habilitación de la integración de aplicaciones en Workday 
2. Creación de un usuario del sistema de integración 
3. Creación de un grupo de seguridad 
4. Asignación del usuario del sistema de integración al grupo de seguridad 
5. Configuración de las opciones del grupo de seguridad 
6. Activación de cambios en directiva de seguridad 
7. Configuración de la importación de usuarios en Azure AD 

## <a name="enable-the-application-integration-for-workday"></a>Habilitación de la integración de aplicaciones para Workday
El objetivo de esta sección es describir cómo se habilita la integración de aplicaciones para Workday.

**Para habilitar la integración de aplicaciones para Workday:**

1. En el panel de navegación izquierdo del Portal de Azure clásico, haga clic en **Active Directory**.
   
   ![Active Directory](./media/active-directory-saas-workday-inbound-tutorial/IC700993.png "Active Directory")
2. En la lista **Directory**, seleccione el directorio cuya integración desee habilitar.
3. Para abrir la vista de aplicaciones, haga clic en **Applications**, en el menú superior de la vista de directorios.
   
   ![Aplicaciones](./media/active-directory-saas-workday-inbound-tutorial/IC700994.png "Aplicaciones")
4. Haga clic en **Agregar** en la parte inferior de la página.
   
   ![Agregar aplicaciones](./media/active-directory-saas-workday-inbound-tutorial/IC749321.png "Agregar aplicaciones")
5. En el cuadro de búsqueda, escriba **Workday**.
   
   ![Agregar una aplicación de la galería](./media/active-directory-saas-workday-inbound-tutorial/IC701021.png "Agregar una aplicación de la galería")
6. En el panel de resultados, seleccione Workday y luego haga clic en Completar para agregar la aplicación.
   
   ![Galería de aplicaciones](./media/active-directory-saas-workday-inbound-tutorial/IC701022.png "Galería de aplicaciones")

## <a name="create-an-integration-system-user"></a>Creación de un usuario del sistema de integración

**Para crear un usuario del sistema de integración:**

1. En **Workday Workbench** (Área de trabajo de Workbench), escriba create user (crear usuario) en el cuadro de búsqueda y haga clic en **Create Integration System User** (Crear usuario del sistema de integración). 
   
    ![Creación de usuarios](./media/active-directory-saas-workday-inbound-tutorial/IC750979.png "Creación de usuarios")
2. Para completar la tarea **Create Integration System User** (Crear usuario del sistema de integración), especifique el nombre de usuario y la contraseña del nuevo usuario del sistema de integración.  
 * Deje la opción **Require New Password at Next Sign In** (Solicitar una nueva contraseña en el siguiente inicio de sesión) sin activar, ya que este usuario iniciará sesión mediante programación. 
 * Deje **Session Timeout Minutes** (Minutos de tiempo de espera de la sesión) en 0 (su valor predeterminado), con el fin de evitar que las sesiones del usuario agoten el tiempo de espera de manera prematura. 
   
    ![Crear usuario de sistema de integración](./media/active-directory-saas-workday-inbound-tutorial/IC750980.png "Crear usuario de sistema de integración")

## <a name="create-a-security-group"></a>Creación de un grupo de seguridad
Para el escenario que se describe en este tutorial, es preciso crear un grupo de seguridad del sistema de integración sin restricciones y asignarle el usuario.

**Para crear un grupo de seguridad:**

1. Escriba create security group (crear grupo de seguridad) en el cuadro de búsqueda y luego haga clic en **Create Security Group**(Crear grupo de seguridad). 
   
    ![Crear grupo de seguridad](./media/active-directory-saas-workday-inbound-tutorial/IC750981.png "Crear grupo de seguridad")
2. Complete la tarea **Create Security Group** (Crear grupo de seguridad).  
3. En la lista desplegable **Type of Tenanted Security Group** (Tipo de grupo de seguridad con inquilinos), seleccione Integration System Security Group—Unconstrained (Grupo de seguridad del sistema de integración: sin restringir).
4. Crear un grupo de seguridad al que se agregarán explícitamente los miembros. 
   
    ![Crear grupo de seguridad](./media/active-directory-saas-workday-inbound-tutorial/IC750982.png "Crear grupo de seguridad")

## <a name="assign-the-integration-system-user-to-the-security-group"></a>Asignación del usuario del sistema de integración al grupo de seguridad

**Para asignar al usuario del sistema de integración:**

1. Escriba edit security group (editar grupo de seguridad) en el cuadro de búsqueda y luego haga clic en **Edit Security Group**(Editar grupo de seguridad). 
   
    ![Editar grupo de seguridad](./media/active-directory-saas-workday-inbound-tutorial/IC750983.png "Editar grupo de seguridad")
2. Busque y seleccione el nuevo grupo de seguridad de integración por nombre. 
   
    ![Editar grupo de seguridad](./media/active-directory-saas-workday-inbound-tutorial/IC750984.png "Editar grupo de seguridad")
3. Agregue el nuevo usuario del sistema de integración al nuevo grupo de seguridad. 
   
    ![Grupo de seguridad del sistema](./media/active-directory-saas-workday-inbound-tutorial/IC750985.png "Grupo de seguridad del sistema")  

## <a name="configure-security-group-options"></a>Configuración de las opciones del grupo de seguridad
En este paso, se conceden al nuevo grupo de seguridad permisos para las operaciones **Get** y **Put** en los objetos protegidos por las siguientes directivas de seguridad del dominio:

* External Account Provisioning
* Worker Data: Public Worker Reports
* Worker Data: All Positions
* Worker Data: Current Staffing Information
* Worker Data: Business Title on Worker Profile

**Para configurar las opciones del grupo de seguridad:**

1. Escriba directivas de seguridad de dominio en el cuadro de búsqueda y, a continuación, haga clic en el vínculo **Domain Security Policies for Functional Area** (Directivas de seguridad de dominio para área funcional).  
   
    ![Directivas de seguridad de dominio](./media/active-directory-saas-workday-inbound-tutorial/IC750986.png "Directivas de seguridad de dominio")  
2. Busque el sistema y seleccione el área funcional **Sistema** .  Haga clic en **Aceptar**.  
   
    ![Directivas de seguridad de dominio](./media/active-directory-saas-workday-inbound-tutorial/IC750987.png "Directivas de seguridad de dominio")  
3. En la lista de directivas de seguridad del área funcional System (Sistema), expanda **Security Administration** (Administración de seguridad) y seleccione la directiva de seguridad de dominio, **External Account Provisioning** (Aprovisionamiento de cuentas externas).  
   
    ![Directivas de seguridad de dominio](./media/active-directory-saas-workday-inbound-tutorial/IC750988.png "Directivas de seguridad de dominio")  
4. Haga clic en **Edit Permissions** (Editar permisos) y, en la página de diálogo **Edit Permissions**, agregue el nuevo grupo de seguridad a la lista de grupos de seguridad con permisos de integración de **Get** y **Put**. 
   
    ![Editar permiso](./media/active-directory-saas-workday-inbound-tutorial/IC750989.png "Editar permiso")  
5. Repita el paso 1 anterior para volver a la pantalla para seleccionar las áreas funcionales y, esta vez, busque personal, seleccione el área funcional **Staffing** (Personal) y haga clic en el botón **OK** (Aceptar).
   
    ![Directivas de seguridad de dominio](./media/active-directory-saas-workday-inbound-tutorial/IC750990.png "Directivas de seguridad de dominio")  
6. En la lista de directivas de seguridad del área funcional Staffing (Personal), expanda **Worker Data: Staffing** y repita el paso 4 anterior en cada una de las restantes directivas de seguridad:

   * Worker Data: Public Worker Reports
   * Worker Data: All Positions
   * Worker Data: Current Staffing Information
   * Worker Data: Business Title on Worker Profile

    ![Directivas de seguridad de dominio](./media/active-directory-saas-workday-inbound-tutorial/IC750991.png "Directivas de seguridad de dominio")  
    
## <a name="activate-security-policy-changes"></a>Activación de cambios en directivas de seguridad

**Para activar cambios en directiva de seguridad:**

1. Escriba activate en el cuadro de búsqueda y, a continuación, haga clic en el vínculo **Activate Pending Security Policy Changes** (Activar cambios en la directiva de seguridad pendientes). 
   
    ![Activar](./media/active-directory-saas-workday-inbound-tutorial/IC750992.png "Activar") 
2. Inicie la tarea Activate Pending Security Policy Changes (Activar cambios en la directiva de seguridad pendientes), para ello tiene que escribir un comentario para fines de auditoría y luego hacer clic en el botón **Aceptar**. 
   
    ![Activar seguridad pendiente](./media/active-directory-saas-workday-inbound-tutorial/IC750993.png "Activar seguridad pendiente")   
3. Complete la tarea que aparece en la pantalla siguiente; para ello, active la casilla etiquetada como **Confirm** (Confirmar) y luego haga clic en **OK** (Aceptar). 
   
    ![Activar seguridad pendiente](./media/active-directory-saas-workday-inbound-tutorial/IC750994.png "Activar seguridad pendiente")  

## <a name="configure-user-import-in-azure-ad"></a>Configuración de la importación de usuarios en Azure AD
El objetivo de esta sección es describir cómo configurar Azure AD para importar contactos desde Workday.

**Para configurar la importación de usuarios:**

1. En la página de integración de aplicaciones de **Workday**, haga clic en **Configurar importación de usuarios** para abrir el cuadro de diálogo **Configurar aprovisionamiento**.
2. En la página **Configuración y credenciales de administrador**, realice los pasos siguientes y haga clic en **Siguiente**: 
   
    ![Configuración y credenciales de administrador](./media/active-directory-saas-workday-inbound-tutorial/IC750995.png "Configuración y credenciales de administrador")    
  1. En el cuadro de texto **Workday admin user name** (Nombre de usuario administrador de Workday), escriba el nombre del usuario que ha creado en la sección Creación de un usuario del sistema de integración.
  2. En el cuadro de texto **Workday admin password** (Contraseña de administrador de Workday), escriba la contraseña del usuario que ha creado en la sección Creación de un usuario del sistema de integración.
  3. En el cuadro de texto **Dirección URL de inquilino de Workday**, escriba la dirección URL o el inquilino de Workday.
3. En la página **Probar conexión**, haga clic en **Iniciar prueba** para confirmar la conectividad y en **Siguiente**. 
   
    ![Probar conexión](./media/active-directory-saas-workday-inbound-tutorial/IC750996.png "Probar conexión")  
4. En la página **Opciones de aprovisionamiento**, haga clic en **Siguiente**. 
   
    ![Opciones de aprovisionamiento](./media/active-directory-saas-workday-inbound-tutorial/IC750997.png "Opciones de aprovisionamiento")
5. En el cuadro de diálogo **Iniciar aprovisionamiento**, haga clic en **Completar**. 
   
    ![Iniciar aprovisionamiento](./media/active-directory-saas-workday-inbound-tutorial/IC750998.png "Iniciar aprovisionamiento")

Ahora puede ir a la sección **Usuarios** y comprobar si se importó el usuario de Workday.

## <a name="additional-resources"></a>Recursos adicionales
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS con Azure Active Directory](active-directory-saas-tutorial-list.md)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](active-directory-appssoaccess-whatis.md)



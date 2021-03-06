---
title: "Análisis de uso de aplicaciones web con Azure Application Insights | Microsoft Docs"
description: "Entender a los usuarios y lo qué hacen con su aplicación web."
services: application-insights
documentationcenter: 
author: botatoes
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: article
ms.date: 05/03/2017
ms.author: cfreeman
ms.translationtype: Human Translation
ms.sourcegitcommit: 71fea4a41b2e3a60f2f610609a14372e678b7ec4
ms.openlocfilehash: fe513adfe40b675b1154e24388316eb7ee09a37d
ms.contentlocale: es-es
ms.lasthandoff: 05/10/2017

---

# <a name="usage-analysis-for-web-applications-with-application-insights"></a>Análisis de uso de aplicaciones web con Application Insights

¿Qué características de su aplicación web son más populares? ¿Los usuarios logran sus objetivos con la aplicación? ¿Salen de ella en momentos concretos y vuelven más tarde?  [Azure Application Insights](app-insights-overview.md) lo ayudará a obtener información eficaz sobre cómo los usuarios usan su aplicación web. Cada vez que actualice la aplicación, puede evaluar también si funciona bien para los usuarios. Con este conocimiento, puede tomar decisiones basadas en datos sobre los ciclos de desarrollo siguientes.

## <a name="send-telemetry-from-your-app"></a>Envío de telemetría desde la aplicación

La mejor experiencia se obtiene mediante la instalación de Application Insights en el código de servidor de aplicaciones y en las páginas web. Los componentes de cliente y servidor de la aplicación devuelven telemetría al portal de Azure para su análisis.

1. **Código de servidor:** instale el módulo adecuado para [ASP.NET](app-insights-asp-net.md), [Azure](app-insights-azure.md), [Java](app-insights-java-get-started.md), [Node.js](app-insights-nodejs.md), u [otra aplicación ](app-insights-platforms.md).

    * *¿No desea instalar código del servidor? Simplemente [cree un recurso de Azure Application Insights](app-insights-create-new-resource.md).*

2. **Código de página web:** abra el [portal de Azure](https://portal.azure.com), abra el recurso de Application Insights para su aplicación y luego abra **Introducción > Supervisar y diagnosticar la aplicación del lado cliente**. 

    ![Copie el script en el encabezado de la página web maestra.](./media/app-insights-usage-overview/02-monitor-web-page.png)


3. **Obtener telemetría:** ejecute su proyecto en modo de depuración durante unos minutos y luego busque resultados en la hoja de información general en Application Insights.

    Publique su aplicación para supervisar el rendimiento de su aplicación y descubra lo que hacen sus usuarios con ella.


## <a name="explore-usage-demographics-and-statistics"></a>Exploración de estadísticas y datos demográficos de uso
Descubra cuándo los usuarios utilizan la aplicación, en qué páginas que están más interesados, en qué ubicación se encuentran dichos usuarios, y los sistemas operativos y exploradores que emplean. 

Los informes Usuarios y sesiones filtran los datos por páginas o eventos personalizados, y los segmentan por propiedades tales como la ubicación, el entorno y la página. También puede agregar sus propios filtros.

![Usuarios](./media/app-insights-usage-overview/users.png)  

La información de la derecha señala patrones de interés en el conjunto de datos.  

* El informe **Usuarios** indica el número de usuarios únicos que tienen acceso a las páginas dentro de los periodos seleccionados. (Los usuarios se cuentan mediante el uso de cookies. Si alguien accede a su sitio con distintos exploradores o equipos cliente, o borra sus cookies, se contabilizarán de más de una vez.)
* El informe **Sesiones** indica el número de sesiones de usuario que acceden al sitio. Una sesión es un periodo de actividad por parte de un usuario, que finaliza con un periodo de inactividad de más de media hora.

[Más información sobre las herramientas Usuarios, Sesiones y Eventos](app-insights-usage-segmentation.md)  

## <a name="page-views"></a>Vistas de página

En la hoja Uso, haga clic en el icono de vistas de página para obtener un desglose de las páginas más populares:

![En la hoja de información general, haga clic en el gráfico de vistas de páginas.](./media/app-insights-usage-overview/05-games.png)

El ejemplo anterior es de un sitio web de juegos. A partir de los gráficos, podemos ver al instante:

* El uso no ha mejorado durante la semana anterior. ¿Quizás debemos pensar en la optimización de motor de búsqueda?
* Tennis es la página de juegos más popular. Vamos a centrarnos en mejorar aún más esta página.
* Por término medio, los usuarios visitan la página Tennis una tres veces por semana. (Hay unas tres veces más sesiones que usuarios.)
* La mayoría de usuarios visitan el sitio durante la semana laboral en EE. UU. y en el horario laborable. Quizás deberíamos proporcionar un botón "ocultar rápidamente" en la página web.
* Las [anotaciones](app-insights-annotations.md) en el gráfico muestran cuándo se implementaron nuevas versiones del sitio web. Ninguna de las implementaciones ha tenido un efecto notable sobre el uso.

¿Qué ocurre si desea investigar el tráfico de su sitio con más detalle, por ejemplo, dividiendo por propiedad personalizada que el sitio envía en su telemetría de vista de página?

1. Abra la herramienta **Eventos** del menú de recursos de Application Insights. Esta herramienta permite analizar el número de eventos personalizados y vistas de página que se enviaron desde su aplicación, en función de una variedad de opciones de filtrado, cohorte y segmentación.
2. En la lista desplegable "que usaron", seleccione "Cualquier página de vista".
3. En la lista desplegable de "Dividir por", seleccione una propiedad por la que va a dividir la telemetría de vista de página.

## <a name="retention---how-many-users-come-back"></a>Retención : ¿cuántos usuarios regresan?

Retención lo ayudará a comprender la frecuencia con la que los usuarios vuelven a usar su aplicación, en función de las cohortes de usuarios que realizan alguna acción empresarial durante un intervalo de tiempo determinado. 

- Qué características específicas provocan que los usuarios vuelvan más veces que otras 
- Formular hipótesis basadas en datos de usuarios reales 
- Determinar si la retención es un problema del producto 

![Retención](./media/app-insights-usage-overview/retention.png) 

Los controles de retención de la parte superior permiten definir eventos específicos y el intervalo de tiempo para calcular la retención. El gráfico situado en la parte central proporciona una representación visual del porcentaje total de retención por el intervalo de tiempo especificado. El gráfico de la parte inferior representa la retención individual en un periodo determinado. Este nivel de detalle permite entender lo que hacen los usuarios y qué podría afectar al regreso de los usuarios con una granularidad más detallada.  

[Más información de la herramienta Retención](app-insights-usage-retention.md)

## <a name="custom-business-events"></a>Eventos de negocio personalizados

Para obtener una idea clara de lo que los usuarios hacen con la aplicación web, es útil insertar líneas de código para registrar eventos personalizados. Estos eventos pueden realizar un seguimiento desde acciones del usuario detalladas como hacer clic en botones específicos hasta eventos de negocio más importantes como realizar una compra o ganar una partida. 

Aunque, en algunos casos, las vistas de página pueden representar eventos útiles, en general, no es así. Un usuario puede abrir una página de un producto sin necesidad de adquirirlo. 

Con los eventos específicos del negocio, puede realizar un gráfico del progreso de los usuarios en su sitio. Puede averiguar sus preferencias para diferentes opciones y en qué partes salen o tienen dificultades. Con este conocimiento, puedan tomar decisiones fundamentadas en lo que respecta a las prioridades del trabajo pendiente en materia de desarrollo.

Los eventos se pueden registrar en la página web:

```JavaScript

    appInsights.trackEvent("ExpandDetailTab", {DetailTab: tabName});
```

O bien, en el lado del servidor de la aplicación web:

```C#
    var tc = new Microsoft.ApplicationInsights.TelemetryClient();
    tc.TrackEvent("CreatedAccount", new Dictionary<string,string> {"AccountType":account.Type}, null);
    ...
    tc.TrackEvent("AddedItemToCart", new Dictionary<string,string> {"Item":item.Name}, null);
    ...
    tc.TrackEvent("CompletedPurchase");
```

Puede adjuntar los valores de propiedad a estos eventos, para que pueda filtrar o dividir los eventos al examinarlos en el portal. Además, se adjunta un conjunto estándar de propiedades a cada evento, como el identificador de usuario anónimo, lo que permite realizar un seguimiento de la secuencia de actividades de un usuario individual.

Obtenga más información sobre los [eventos personalizados](app-insights-api-custom-events-metrics.md#trackevent) y las [propiedades](app-insights-api-custom-events-metrics.md#properties).

### <a name="slice-and-dice-events"></a>Eventos de segmentación y desglose

En las herramientas Usuarios, Sesiones y Eventos, puede segmentar y desglosar los eventos personalizados por usuario, nombre del evento y propiedades.

![Usuarios](./media/app-insights-usage-overview/users.png)  
  
## <a name="design-the-telemetry-with-the-app"></a>Diseño de la telemetría con la aplicación

Al diseñar cada característica de la aplicación, tenga en cuenta cómo va a medir su éxito con los usuarios. Decida qué eventos empresariales necesita registrar y codifique las llamadas de seguimiento de esos eventos en la aplicación desde el principio.

## <a name="a--b-testing"></a>Prueba A | B
Si no conoce qué variante de una característica tendrá más éxito, publique ambas para que estén accesibles a los diferentes usuarios. Mida el éxito de cada una y, a continuación, cambie a una versión unificada.

Para realizar esta técnica, adjunte valores de propiedad distintas a toda la telemetría que se envía con cada versión de la aplicación. Puede hacerlo al definir las propiedades en el TelemetryContext activo. Estas propiedades predeterminadas se agregan a cada mensaje de telemetría que envía la aplicación: no solo los mensajes personalizados, sino también la telemetría estándar.

En el portal de Application Insights, podrá filtrar y dividir los datos en los valores de propiedad, con el fin de comparar las distintas versiones.

Para ello, [configure un inicializador de telemetría](app-insights-api-filtering-sampling.md##add-properties-itelemetryinitializer):

```C#


    // Telemetry initializer class
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize (ITelemetry telemetry)
        {
            telemetry.Properties["AppVersion"] = "v2.1";
        }
    }
```

En el inicializador de la aplicación web, como Global.asax.cs:

```C#

    protected void Application_Start()
    {
        // ...
        TelemetryConfiguration.Active.TelemetryInitializers
        .Add(new MyTelemetryInitializer());
    }
```

Todos los nuevos clientes de telemetría agregan automáticamente el valor de propiedad especificado. La telemetría individual puede invalidar los valores predeterminados.

## <a name="next-steps"></a>Pasos siguientes

* [Usuarios y sesiones](app-insights-usage-segmentation.md)
* [Retención](app-insights-usage-retention.md)
* [Eventos personalizados de codificación](app-insights-api-custom-events-metrics.md)



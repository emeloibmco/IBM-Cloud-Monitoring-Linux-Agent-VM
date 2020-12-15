# IBM-Cloud-Monitoring-Linux-Agent-VM


Monitoring with sysdig es una herramienta de vigilancia que brinda IBM y permite tener supervisión sobre diferentes servicios de IBM CLOUD. En esta guía se explicará el paso a paso para el aprovisionamiento de Monitoring With Sysdig en una maquina Ubuntu dentro de un VMWare ESXi.

### Antes de empezar :computer:

1.	Debe contar con una cuenta de IBM Cloud que disponga con los permisos necesarios para el aprovisionamiento de Monitoring With Sysdig. Para más información se recomienda visitar el siguien link:
https://cloud.ibm.com/docs/Monitoring-with-Sysdig?topic=Sysdig-iam.
2.	Debe tener corriendo VMWare ESXi ya sea dentro de BareMetal, Maquina virtual o en un Gateway Appliance.
3.  Debe tener corriendo una maquina ubuntu dentro del VMWare.

### Paso 1: Suministrar una instancia de IBM Cloud Monitoring with Sysdig

En esta guía de aprendizaje de iniciación se ofrecen instrucciones para suministrar una instancia en IBM Cloud Monitoring with Sysdig en la región EE. UU. sur. Para obtener más información sobre las regiones soportadas, consulte el apartado [Regiones.](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-endpoints&locale=es)

Para suministrar una instancia de IBM Cloud Monitoring with Sysdig mediante la interfaz de usuario de IBM Cloud, siga los pasos siguientes:

1.	[Inicie una sesión en su cuenta de IBM Cloud  .](https://cloud.ibm.com/login)
Cuando inicia una sesión con su ID de usuario y su contraseña, se abre la interfaz de usuario de IBM Cloud.
2.	Pulse **“Catálogo”**. Se abrirá la lista de servicios disponibles en IBM Cloud.
3.	Para filtrar la lista de servicios que se visualiza, seleccione la categoría **“Herramientas de desarrollador”.**
4.	Pulse el mosaico **“IBM Cloud Monitoring with Sysdig”**. Se abre el panel de control *_Observabilidad_*.
5.	Seleccione **“Crear instancia”**.
6.	Especifique un nombre para la instancia de servicio.
7.	Seleccione el grupo de recursos **“default”**.
Puede suministrar la instancia en cualquier grupo de recursos en el que tenga permisos para crear recursos.
De forma predeterminada, está establecido el grupo de recursos **predeterminado (default)**.
8.	Seleccione el plan de servicio **“Prueba”**.
De forma predeterminada, se selecciona el plan de *_Prueba_*.
Para obtener más información acerca de los otros planes de servicio, consulte [Planes de servicio.](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-pricing_plans&locale=es#pricing_plans)
9.	Pulse **"Crear"**.
Después de suministrar una instancia, se abre el panel de control *Observabilidad* y se muestran detalles correspondientes a las instancias de **Supervisión**.

### Paso 2: Configurar el clúster de Kubernetes para que envíe métricas a la instancia

<p style="text-align: justify;">Para configurar el clúster de Kubernetes para que envíe métricas a su instancia de IBM Cloud Monitoring with Sysdig, debe instalar un pod de agente de Sysdig en cada nodo del clúster. El agente de Sysdig se instala mediante un DaemonSet que garantiza que se ejecute una instancia del agente en cada nodo trabajador. El agente de Sysdig recopila métricas del pod en el que está instalado y los reenvía a la instancia.Para configurar el clúster de Kubernetes para que reenvíe métricas a la instancia de IBM Cloud Monitoring with Sysdig, siga los pasos siguientes pasos.</p>

**Obtenga la clave de acceso de Sysdig.**
1.	Obtenga la clave de acceso de Sysdig. 
En IBMCLOUD, haga click sobre el “menú de hamburguesa” y seleccione *“observabilidad”*.

<p align="center">
  <img width="auto" height="auto" src="https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_HostUbuntu/Imagenes/Imagne001.PNG">
</p>

2.	Una vez se encuentre dentro de observabilidad, selecciones *“Monitoring”*, deberá aparecer la instancia que previamente ha sido creada. 

3.	Haga click en los tres puntos y seleccione *“Obtain Key”*.
![Reference](https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_HostUbuntu/Imagenes/Imagen002.PNG)

4.	Copie la clave que aparece en pantalla.

5.	Obtenga la URL donde se encuentran los puntos finales de Sysdig, como esta guía está enfocada en el aprovisionamiento de Monitoring With Sysdig en la región de EE.UU sur, la URL dónde se encuentra el punto final de Sysdig es la siguiente: 

`*ingest.us-south.monitoring.cloud.ibm.com*`.

Para más información sobre los puntos finales disponibles por regiones, visite el siguiente enlace: [Puntos finales del recopilador de Sysdig](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-endpoints&locale=es#endpoints_ingestion)

6.	En IBMCLOUD navegue a **“lista de recursos”** luego a **“Clústeres”**, y seleccione el clúster que desea configurar para monitorearlo.

7.	Una vez se encuentre dentro de la visión general de la página del clúster, oprima en el botón **“Actions”** y seleccione **“terminal web”**.

<p align="center">
  <img width="auto" height="auto" src="https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_Cl%C3%BAsterKubernetes/Imagenes/Captura1.PNG">
</p>


8.	Se deberá desplegar en la parte inferior de la pantalla el terminal web del clúster.
Consejo: Para mayor comodidad, haga click sobre el cuadrado que aparece en la esquina superior derecha de la terminal, esto le permitirá utilizarlo en otra pestaña del navegador.

9.	Dentro del terminal web ejecute el siguiente comando:

`curl -sL https://raw.githubusercontent.com/draios/sysdig-cloud-scripts/master/agent_deploy/IBMCloud-Kubernetes-Service/install-agent-k8s.sh | bash -s -- -a SYSDIG_ACCESS_KEY -c COLLECTOR_ENDPOINT -t TAG_DATA -ac 'sysdig_capture_enabled: false'`

-	**SYSDIG_ACCESS_KEY** es la clave de ingestión de la instancia que ha recuperado anteriormente. (instrucción 4)

-	**COLLECTOR_ENDPOINT** es el URL de ingestión de la región en la que está disponible la instancia de supervisión que ha recuperado anteriormente. (instrucción 5)

-	**TAG_DATA** son etiquetas separadas por comas con el formato NOMBRE_ETIQUETA_VALOR:ETIQUETA. Puede asociar una o varias etiquetas al agente de Sysdig. Por ejemplo: role:serviceX,location:us-south. Más adelante podrá utilizar estas etiquetas para identificar las métricas del entorno en el que se ejecuta el agente.

-	Establezca **sysdig_capture_enabled** en false para inhabilitar la característica de captura de Sysdig. De forma predeterminada, está establecido en true. Para obtener más información, consulte [Cómo trabajar con capturas](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-captures&locale=es#captures).

10.	Verifique que el agente de Sysdig se ha creado correctamente y compruebe su estado. Ejecute el siguiente comando:

` kubectl get pods -n ibm-observe`

El despliegue se realiza correctamente cuando se ven una o varios pods de `sysdig-agent`. El número de pods de `sysdig-agent` es igual al número de nodos trabajadores del clúster. Todos los pods deben estar en un estado `Running`.

### Paso 3. Iniciar la interfaz de usuario web de Sysdig

Para iniciar la interfaz de usuario web de Sysdig a través de la consola IBM Cloud, siga los pasos siguientes.

Solo puede tener una sesión de interfaz de usuario web abierta por navegador.

1.	[Inicie una sesión en su cuenta de IBM Cloud  .](https://cloud.ibm.com/login)
Cuando inicia una sesión con su ID de usuario y su contraseña, se abre el panel de control de IBM Cloud.

2.	En el menú  , seleccione **Observabilidad**.

3.	Seleccione **Monitoring**. Se muestra la lista de instancias que están disponibles en IBM Cloud.

4.	Localice su instancia y pulse **View Sysdig**.

-	Primera vez: como ya ha instalado el agente de Sysdig, puede saltarse los pasos del asistente de instalación, iniciación y realización de la incorporación.
-	Siguiente veces: se abre la vista Explorar.

Si el agente de Sysdig no se ha instalado correctamente, si apunta a un punto final de ingestión incorrecto o si la clave de acceso es incorrecta, la página que se abre le indica qué debe hacer a continuación.

Por ejemplo, si el agente de Sysdig no se ha instalado correctamente, no puede saltarse el asistente de instalación. Es posible que aparezca un mensaje parecido al siguiente:

_Waiting for the first node to connect... Go ahead and follow the instructions below._

Puede intentar las acciones siguientes:

-	Verifique que está utilizando el [punto final](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-endpoints&locale=es#endpoints_ingestion) `ingest`y no el punto final de Sysdig.

-	Verifique que la [clave de acceso](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-access_key&locale=es) sea correcta.

-	Siga las instrucciones y repita los pasos de esta guía de aprendizaje.


### Paso 4: Supervisar el clúster

Puede supervisar el clúster en la vista *EXPLORE* que está disponible a través de la interfaz de usuario web de Sysdig. Esta vista es la página de inicio predeterminada y el punto de partida para resolver problemas y supervisar la infraestructura y los recursos del clúster.

En la sección Host y contenedores, puede ver la tabla Explorar, que es una lista de los nodos trabajadores de su clúster que están reenviando métricas a la instancia de *Monitoring*. Cada entrada de nodo trabajador representa un grupo de objetos relacionados de la infraestructura para ese nodo trabajador.

Pulse **Host y contenedores**  para cambiar los orígenes de datos. A continuación, seleccione un nodo trabajador. Los datos que se muestran corresponden al nodo trabajador que ha seleccionado. Si pulsa **Volver a la tabla Explorar**, se muestra la tabla Explorar.

**Personalización de la tabla Explorar**

Puede personalizar la tabla Explorar.

-	Cada columna muestra una métrica diferente.

-	Puede configurar cada métrica individualmente.

-	Puede cambiar el orden de las columnas.

Tenga en cuenta que, cuando se hacen cambios en el orden de las columnas existentes, el cambio se aplica a las distintas agrupaciones mientras dure la sesión. Si añade o elimina una columna, el cambio es permanente.

-	También puede configurar colores para resaltar valores y facilitar su lectura.

Por ejemplo, para configurar códigos de colores para una columna, siga los pasos siguientes:

1.	Seleccione una columna. Mueva el puntero del ratón sobre el título de la columna. A continuación, seleccione el icono de lápiz.

2.	Conmute la barra para habilitar la codificación por colores.

3.	Defina valores para los distintos umbrales.

**Personalización de paneles de control**

Para ver más detalles acerca de un nodo trabajador concreto, pulse en la entrada de la infraestructura y se abrirá en la tabla el panel de control _Visión general por host_. Puede explorar diferentes paneles de control y métricas pulsando el icono 

<p align="center">
  <img width="auto" height="auto" src="https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_Cl%C3%BAsterKubernetes/Imagenes/Captura2.PNG">
</p>


Tenga en cuenta que solo puede seleccionar métricas y paneles de control que sean relevantes para el nodo trabajador seleccionado.
Para volver a la tabla Explorar completa, pulse el botón **X (volver a la tabla Explorar)**.

### Pasos siguientes
Cree un panel de control personalizado. Para obtener más información, consulte [Cómo trabajar con paneles de control.](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-dashboards&locale=es#dashboards)
También puede aprender sobre las alertas. Para obtener más información, consulte [Cómo trabajar con alertas](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-monitoring&locale=es#monitoring_alerts).


# IBM-Cloud-Observability :cloud:

## APROVISIONAMIENTO DE MONITORING WITH SYSDIG PARA UN HOST UBUNTU 

Monitoring with sysdig es una herramienta de vigilancia que brinda IBM y permite tener supervisión sobre diferentes servicios de IBM CLOUD. En esta guía se explicará el paso a paso para el aprovisionamiento de Monitoring With Sysdig en un Host ubuntu.

### Antes de empezar :computer:
1.	Debe contar con una cuenta de IBM Cloud que disponga con los permisos necesarios para el aprovisionamiento de Monitoring With Sysdig. Para más información se recomienda visitar el siguien link:
https://cloud.ibm.com/docs/Monitoring-with-Sysdig?topic=Sysdig-iam.
2.	Instale la CLI de IBM Cloud y el plugin de la CLI de Kubernetes. Para obtener más información, consulte [Instalación de la CLI de IBM Cloud.](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started&locale=es)
3.	Cree un servidor ubuntu.
4.	Asegúrese de que a su ID de usuario se le asignen las siguientes políticas de IBM® Cloud Identity and Access Management:
Para obtener más información sobre los roles de IAM de IBM Cloud™ Kubernetes Service, consulte [Permisos de acceso de usuario.](https://cloud.ibm.com/docs/containers?topic=containers-access_reference&locale=es#access_reference)

### Paso 1: Suministrar una instancia de IBM Cloud Monitoring with Sysdig
En esta guía de aprendizaje de iniciación se ofrecen instrucciones para suministrar una instancia en IBM Cloud Monitoring with Sysdig en la región EE. UU. sur. Para obtener más información sobre las regiones soportadas, consulte el apartado [Regiones](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-endpoints&locale=es).
Para suministrar una instancia de IBM Cloud Monitoring with Sysdig mediante la interfaz de usuario de IBM Cloud, siga los pasos siguientes:
1.	[Inicie una sesión en su cuenta de IBM Cloud](https://cloud.ibm.com/login)  .
Cuando inicia una sesión con su ID de usuario y su contraseña, se abre la interfaz de usuario de IBM Cloud.
2.	Pulse “Catálogo”. Se abrirá la lista de servicios disponibles en IBM Cloud.
3.	Para filtrar la lista de servicios que se visualiza, seleccione la categoría “Herramientas de desarrollador”.
4.	Pulse el mosaico “IBM Cloud Monitoring with Sysdig”. Se abre el panel de control Observabilidad.
5.	Seleccione “Crear instancia”.
6.	Especifique un nombre para la instancia de servicio.
7.	Seleccione el grupo de recursos “predeterminado”.
Puede suministrar la instancia en cualquier grupo de recursos en el que tenga permisos para crear recursos.
De forma predeterminada, está establecido el grupo de recursos predeterminado (default).
8.	Seleccione el plan de servicio “Prueba”.
De forma predeterminada, se selecciona el plan de Prueba.
Para obtener más información acerca de los otros planes de servicio, consulte [Planes de servicio.](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-pricing_plans&locale=es#pricing_plans)
9.	Pulse Crear.
Después de suministrar una instancia, se abre el panel de control Observabilidad y se muestran detalles correspondientes a las instancias de Supervisión.
### Paso 2: Configurar el servidor Ubuntu para que envíe métricas a la instancia
Para configurar el servidor Ubuntu para que envíe métricas a la instancia de IBM Cloud Monitoring with Sysdig, debe instalar un agente de Sysdig.
Siga los pasos siguientes desde la línea de comando
Obtenga la clave de acceso de Sysdig.

1.	Obtenga la clave de acceso de Sysdig. 
En IBMCLOUD, haga click sobre el “menú de hamburguesa” y seleccione “observabilidad”.

<p align="center">
  <img width="auto" height="auto" src="https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_HostUbuntu/Imagenes/Imagne001.PNG">
</p>

2.	Una vez se encuentre dentro de observabilidad, selecciones “Monitoring”, deberá aparecer la instancia que previamente ha sido creada. 
3.	Haga click en los tres puntos y seleccione “Obtain Key”.

![Reference](https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_HostUbuntu/Imagenes/Imagen002.PNG)

4.	Copie la clave que aparece en pantalla.

5.	Obtenga la URL donde se encuentran los puntos finales de Sysdig, como esta guía está enfocada en el aprovisionamiento de Monitoring With Sysdig en la región de EE.UU sur, la URL dónde se encuentra el punto final de Sysdig es la siguiente: ingest.us-south.monitoring.cloud.ibm.com.
Para más información sobre los puntos finales disponibles por regiones, visite el siguiente enlace: [Puntos finales del recopilador de Sysdig](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-endpoints&locale=es#endpoints_ingestion)

6. Desde la CLI utilizando SSH, despliegue el agente Sysdig en su servidor; para ello habra PowerShell y utilice el siguiente comando para ingresar a IBMCLOUD.

    ` Ibmcloud login -sso  ` 

7.	Se le redirigirá a una página web, copie el código proporcionado y péguelo en el PowerShell.
8.	A continuación, seleccione la cuenta en dónde se encuentre desplegado el servidor de Ubuntu. El proceso hasta este paso deberá verse como se muestra en la siguiente imagen:

<p align="center">
  <img width="auto" height="auto" src="https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_HostUbuntu/Imagenes/Imagen003.PNG">
</p>

9.	Conéctese a su servidor Ubuntu utilizando el siguiente comando:
   `ssh -i  <dirección donde se encuentra la llave publica> root@<Ip flotante del servidor>`
10.	Una vez este conectado a su servidor ejecute el siguiente comando:
   `curl -s https://s3.amazonaws.com/download.draios.com/stable/install-agent | sudo bash -s -- --access_key SYSDIG_ACCES_KEY --collector COLLECTOR_ENDPOINT --collector_port 6443 --secure true --tags example_tag:example_value `
   
Donde:

-	**SYSDIG_ACCESS_KEY** es la clave de ingestión de la instancia que ha recuperado anteriormente. (instrucción 4)

-	**COLLECTOR_ENDPOINT** es el URL de ingestión de la región en la que está disponible la instancia de supervisión que ha recuperado anteriormente. (instrucción 5)

-	**TAG_DATA** son etiquetas separadas por comas con el formato NOMBRE_ETIQUETA_VALOR:ETIQUETA. Puede asociar una o varias etiquetas al agente de Sysdig. Por ejemplo: role:serviceX,location:us-south. Más adelante podrá utilizar estas etiquetas para identificar las métricas del entorno en el que se ejecuta el agente.

-	Establezca **sysdig_capture_enabled** en false para inhabilitar la característica de captura de Sysdig. De forma predeterminada, está establecido en true. Para obtener más información, consulte [Cómo trabajar con capturas](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-captures&locale=es#captures).

Una vez ejecutado el comando, comenzará el proceso de configuración en el servidor. Deberá verse como se muestra en la siguiente imagen:

<p align="center">
  <img width="auto" height="auto" src="https://github.com/javierjimenezm/IBM-Cloud-Observability/blob/master/Monitoring_with_Sysdig_HostUbuntu/Imagenes/Imagen004.PNG">
</p>

### Paso 3. Iniciar la interfaz de usuario web de Sysdig

Para iniciar la interfaz de usuario web de Sysdig a través de la consola IBM Cloud, siga los pasos siguientes.
Solo puede tener una sesión de interfaz de usuario web abierta por navegador.
1.	[Inicie una sesión en su cuenta de IBM Cloud](https://cloud.ibm.com/login)  .
Cuando inicia una sesión con su ID de usuario y su contraseña, se abre el panel de control de IBM Cloud.
2.	En el menú  , seleccione Observabilidad.
3.	Seleccione Supervisión. Se muestra la lista de instancias que están disponibles en IBM Cloud.
4.	Localice su instancia y pulse Ver Sysdig.

-	Primera vez: como ya ha instalado el agente de Sysdig, puede saltarse los pasos del asistente de instalación, iniciación y realización de la incorporación.

-	Siguiente veces: se abre la vista Explorar.
Si el agente de Sysdig no se ha instalado correctamente, si apunta a un punto final de ingestión incorrecto o si la clave de acceso es incorrecta, la página que se abre le indica qué debe hacer a continuación.
Por ejemplo, si el agente de Sysdig no se ha instalado correctamente, no puede saltarse el asistente de instalación. Es posible que aparezca un mensaje parecido al siguiente:
Waiting for the first node to connect... Go ahead and follow the instructions below.

Puede intentar las acciones siguientes:

-	Verifique que está utilizando el [punto final](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-endpoints&locale=es#endpoints_ingestion) `ingest`y no el punto final de Sysdig.

-	Verifique que la [clave de acceso](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-access_key&locale=es) sea correcta.

-	Siga las instrucciones y repita los pasos de esta guía de aprendizaje.

### Paso 4. Supervisar el servidor Ubuntu
Puede supervisar el servidor Ubuntu en la vista EXPLORAR que está disponible a través de la interfaz de usuario web. Esta vista constituye el punto de partida para solucionar problemas y supervisar la infraestructura. Es la página de inicio predeterminada de la interfaz de usuario web para los usuarios.
En la sección Host y contenedores, encontrará la entrada correspondiente al servidor Ubuntu. Pulse Host y contenedores  para cambiar los orígenes de datos. A continuación, seleccione el servidor Ubuntu. Los datos que se muestran corresponden al servidor Ubuntu que seleccione.
Por ejemplo, para configurar códigos de colores para una columna, siga los pasos siguientes:
1.	Seleccione una columna. Mueva el puntero del ratón sobre el título de la columna. A continuación, seleccione el icono de lápiz.
2.	Conmute la barra para habilitar la codificación por colores.
3.	Defina valores para los distintos umbrales.
### Pasos siguientes
-Cree un panel de control personalizado. Para obtener más información, consulte [Cómo trabajar con paneles de control](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-dashboards&locale=es#dashboards)

-También puede aprender sobre las alertas. Para obtener más información, consulte [Cómo trabajar con alertas](https://cloud.ibm.com/docs/services/Monitoring-with-Sysdig?topic=Sysdig-monitoring&locale=es#monitoring_alerts).

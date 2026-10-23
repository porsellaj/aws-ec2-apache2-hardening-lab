# Apache2 on AWS EC2: Web Server Deployment and Basic Hardening

Laboratorio académico desarrollado para la asignatura **Introducción a la Ciberseguridad** de la Licenciatura en Ciberseguridad de la Universidad Tecnológica de Panamá.

El proyecto consiste en desplegar un servidor web Apache2 en una instancia Ubuntu de AWS EC2, publicar una plantilla HTML y aplicar configuraciones básicas de seguridad mediante UFW, ajustes de Apache y Fail2Ban.

## 1. Objetivos

- Desplegar y administrar un servidor Ubuntu en AWS EC2.
- Establecer acceso remoto mediante SSH utilizando autenticación con llave privada.
- Instalar y verificar el funcionamiento de Apache2.
- Publicar contenido HTML accesible desde un navegador.
- Configurar reglas de firewall con UFW.
- Reducir la información del servidor expuesta en las respuestas HTTP.
- Deshabilitar el módulo `autoindex` de Apache.
- Instalar, habilitar y verificar el servicio Fail2Ban.

## 2. Tecnologías y herramientas

| Tecnología | Uso |
|---|---|
| AWS EC2 | Infraestructura de servidor virtual en la nube. |
| Ubuntu Server 26.04 LTS | Sistema operativo de la instancia. |
| SSH | Administración remota mediante llave privada. |
| Apache2 | Servidor web HTTP. |
| HTML5 UP — Lens | Plantilla web publicada en el servidor. |
| UFW | Configuración del firewall local. |
| Fail2Ban | Herramienta de protección mediante bloqueo temporal de direcciones IP según las reglas configuradas. |
| APT | Instalación y actualización de paquetes. |
| systemd | Administración del estado de los servicios. |

## 3. Requisitos previos

- Una cuenta de AWS.
- Una instancia EC2 con Ubuntu Server.
- Una llave privada SSH correspondiente a la instancia.
- Acceso administrativo al servidor.
- Reglas de entrada en el Security Group que permitan las conexiones SSH y HTTP necesarias.
- AWS CloudShell o una terminal desde la que pueda establecerse la conexión SSH.

La IP pública, el nombre de la llave y las reglas de red pueden variar según la instancia utilizada.

**Nota de seguridad:** en este laboratorio se utilizó una dirección IP pública específica. En esta documentación se reemplaza por `TU_IP_PUBLICA` para evitar publicar la dirección original.

## 4. Preparación de la llave SSH

En AWS CloudShell se comprueba el archivo descargado, se renombra y se restringen sus permisos.

```bash
ls
mv labuser\ \(1\).pem lab-ubuntu26.pem
ls
chmod 400 lab-ubuntu26.pem
ls -l lab-ubuntu26.pem
```

El comando `chmod 400` permite que el propietario lea la llave privada sin conceder permisos de escritura ni acceso a otros usuarios.

La llave privada debe mantenerse protegida y no debe publicarse en GitHub.

## 5. Conexión SSH a la instancia

Desde CloudShell o una terminal que tenga acceso a la llave:

```bash
ssh -i lab-ubuntu26.pem ubuntu@TU_IP_PUBLICA
```

Sustituir `TU_IP_PUBLICA` por la dirección pública de la instancia EC2.

En la primera conexión SSH puede aparecer una advertencia indicando que la identidad del servidor no se conoce todavía. Antes de aceptar la huella, se debe verificar que corresponda a la instancia esperada.

Una vez establecida la conexión, la terminal permite administrar el sistema remoto.

## 6. Actualización del sistema e instalación de Apache2

### 6.1. Elevar los privilegios

```bash
sudo su
```

Este comando inicia una shell con privilegios de administrador para realizar las tareas de configuración del servidor.

### 6.2. Actualizar el índice de paquetes

```bash
apt update
```

Actualiza la información local de los paquetes disponibles en los repositorios configurados.

### 6.3. Instalar Apache2

```bash
apt install apache2 -y
```

Instala Apache2 utilizando APT. La opción `-y` confirma automáticamente las preguntas de instalación que admiten esa respuesta.

### 6.4. Verificar el estado del servicio

```bash
systemctl status apache2
```

La salida del laboratorio muestra el servicio en estado `active (running)`, lo que indica que Apache2 está ejecutándose.

### 6.5. Comprobar la respuesta HTTP local

```bash
curl -I http://localhost
```

Este comando realiza una solicitud HTTP de tipo HEAD y muestra las cabeceras de la respuesta.

En la evidencia del laboratorio, el servidor respondió con `HTTP/1.1 200 OK`, confirmando que Apache2 atendía solicitudes HTTP localmente.

La respuesta inicial también mostraba información sobre la versión de Apache, lo que motivó los ajustes de hardening posteriores.

## 7. Publicación de la plantilla web

Para personalizar la página inicial, se utilizó la plantilla Lens de HTML5 UP.

### 7.1. Descargar la plantilla

```bash
curl -L https://html5up.net/lens/download -o plantilla.zip
```

La opción `-L` permite seguir redirecciones HTTP. La opción `-o` guarda el contenido descargado con el nombre `plantilla.zip`.

### 7.2. Verificar el archivo descargado

```bash
file plantilla.zip
```

Este comando identifica el tipo de archivo descargado.

### 7.3. Retirar la página predeterminada

```bash
rm -rf /var/www/html/index.html
```

Se elimina el archivo `index.html` predeterminado de Apache para dar paso al contenido de la plantilla.

Antes de ejecutar un comando de eliminación, comprueba siempre la ruta indicada. En este caso, el comando se refiere al archivo especificado, no a todo el directorio.

### 7.4. Instalar las herramientas de compresión

```bash
apt install zip unzip -y
```

Instala las herramientas necesarias para trabajar con archivos ZIP.

### 7.5. Copiar y descomprimir la plantilla

```bash
cp plantilla.zip /var/www/html/
cd /var/www/html/
ls /var/www/html
unzip plantilla.zip
```

La plantilla se copia al directorio utilizado por Apache y se descomprime en esa ubicación.

Los archivos HTML, CSS, JavaScript e imágenes extraídos permiten que Apache sirva el sitio web.

### 7.6. Verificar la publicación

Abrir en el navegador la dirección pública de la instancia:

```text
http://TU_IP_PUBLICA
```

En el laboratorio se comprobó visualmente la carga de la plantilla Lens.

La práctica documenta la publicación del sitio mediante HTTP. No se documenta una configuración HTTPS o un certificado TLS para este servidor.

## 8. Configuración del firewall con UFW

UFW permite definir reglas de entrada y salida mediante una interfaz simplificada para la administración del firewall.

### 8.1. Instalar UFW

```bash
apt install ufw -y
```

### 8.2. Permitir las conexiones SSH

```bash
ufw allow openSSH
```

Esta regla permite las conexiones SSH que coincidan con el perfil configurado por UFW.

### 8.3. Permitir el tráfico HTTP

```bash
ufw allow 80/tcp
```

Permite las conexiones TCP entrantes al puerto 80, utilizado por Apache para servir contenido HTTP.

### 8.4. Activar el firewall

```bash
ufw enable
```

Durante la activación se muestra una advertencia de que el cambio puede interrumpir conexiones SSH existentes. En la evidencia del laboratorio se confirmó la operación.

### 8.5. Verificar las reglas activas

```bash
ufw status verbose
```

La salida documentada muestra UFW en estado `active`, con una política predeterminada que deniega el tráfico entrante y permite el tráfico saliente, además de las reglas que autorizan SSH y HTTP.

**Consideración de seguridad:** la captura muestra que las reglas de SSH y HTTP permiten tráfico desde cualquier origen. En un despliegue real conviene restringir SSH a la IP o red administrativa autorizada mediante las reglas apropiadas de AWS y del firewall. Esa restricción no se presenta como una configuración implementada en este laboratorio.

## 9. Hardening de Apache2

### 9.1. Reducir la información expuesta por el servidor

El informe del laboratorio indica que se realizaron ajustes destinados a reducir la exposición de información sobre la versión de Apache y los metadatos incluidos en las respuestas HTTP.

Después de aplicar esos cambios, se ejecutó:

```bash
systemctl restart apache2
```

Este comando reinicia Apache2 para que la configuración se vuelva a cargar.

**Limitación documental:** el informe no muestra las directivas exactas ni el archivo de configuración editado para reducir la información expuesta. Por ello, no se inventan aquí instrucciones de configuración adicionales. Para reproducir esta parte con precisión, es necesario documentar el archivo y las directivas que se modificaron durante la práctica.

### 9.2. Deshabilitar el módulo `autoindex`

```bash
a2dismod autoindex
```

Este comando deshabilita el módulo `autoindex` de Apache2, responsable de generar listados automáticos de directorios cuando no existe un archivo índice y la configuración permite ese comportamiento.

Durante la ejecución, Apache solicita una confirmación explícita porque deshabilitar un módulo puede afectar al funcionamiento del servidor. En la evidencia del laboratorio se aceptó la operación y se confirmó que el módulo quedó deshabilitado.

### 9.3. Recargar la configuración

```bash
systemctl reload apache2
```

Este comando solicita a Apache2 que recargue su configuración.

Deshabilitar `autoindex` reduce la posibilidad de mostrar listados automáticos de directorios, pero no constituye por sí solo un control completo de acceso al contenido del sitio.

## 10. Instalación y verificación de Fail2Ban

Fail2Ban analiza registros mediante filtros y aplica acciones configuradas cuando se cumplen determinadas condiciones, como múltiples intentos fallidos de autenticación.

### 10.1. Instalar Fail2Ban

```bash
apt install fail2ban -y
```

### 10.2. Habilitar el servicio al iniciar el sistema

```bash
systemctl enable fail2ban
```

Este comando configura el servicio para que pueda iniciarse automáticamente durante el arranque del sistema.

### 10.3. Iniciar el servicio

En la captura original aparece el siguiente comando:

```bash
sytemctl start fail2ban
```

La ejecución produjo un error porque `sytemctl` está escrito incorrectamente. La terminal devolvió un mensaje de comando no encontrado.

El comando correcto para iniciar el servicio es:

```bash
systemctl start fail2ban
```

Esta corrección se distingue de la secuencia originalmente registrada.

### 10.4. Comprobar el estado del servicio

```bash
systemctl status fail2ban
```

La salida documentada muestra `fail2ban.service` en estado `active (running)`.

Esto permite confirmar que el servicio está en ejecución. No demuestra por sí solo que se haya producido un bloqueo real ni que una dirección IP haya sido bloqueada durante la práctica.

El comportamiento efectivo depende de los filtros, las reglas y las acciones configuradas. El informe no documenta una prueba específica de bloqueo.

## 11. Resultados del laboratorio

El informe documenta los siguientes resultados:

| Comprobación | Resultado documentado |
|---|---|
| Conexión remota | Se estableció una sesión SSH usando una llave privada. |
| Instalación de Apache2 | El servicio apareció como `active (running)`. |
| Respuesta HTTP local | `curl -I http://localhost` devolvió `200 OK`. |
| Publicación web | La plantilla Lens se mostró en el navegador a través de la IP pública. |
| Firewall UFW | Se activó con reglas para SSH y HTTP. |
| Módulo `autoindex` | Se deshabilitó mediante `a2dismod`. |
| Fail2Ban | El servicio apareció como `active (running)`. |

Estos resultados corresponden a las evidencias incorporadas al informe académico. No implican que se haya repetido el procedimiento desde una instancia nueva utilizando este README.

## 12. Seguridad y limitaciones

- La llave privada SSH no debe publicarse ni incluirse en el repositorio.
- La IP pública utilizada durante el laboratorio se sustituye en la documentación por `TU_IP_PUBLICA`.
- SSH debe limitarse a los orígenes administrativos autorizados en un entorno real.
- El laboratorio publica contenido mediante HTTP. No se documenta una configuración de HTTPS.
- La desactivación de `autoindex` no sustituye otros controles de autorización y protección de archivos.
- La instalación y ejecución de Fail2Ban no demuestran por sí solas la eficacia de una política concreta de bloqueo.
- Los cambios destinados a reducir la información de versión de Apache requieren que se documenten las directivas exactas para permitir una reproducción completa.
- Las versiones de los paquetes instalados dependen de los repositorios disponibles en el momento de la instalación.
- La práctica constituye un ejercicio académico de despliegue y hardening básico, no una auditoría completa de seguridad ni una configuración certificada para producción.

## 13. Aprendizajes

Este laboratorio permitió practicar:

- Administración remota de una instancia Linux mediante SSH.
- Protección de una llave privada mediante permisos del sistema de archivos.
- Instalación y verificación de servicios con APT y systemd.
- Despliegue de un servidor web Apache2.
- Publicación de contenido HTML en el directorio web.
- Configuración inicial de un firewall local con UFW.
- Reducción de exposición de información y desactivación de un módulo de Apache.
- Instalación y verificación de Fail2Ban.
- Comprensión de la importancia de documentar los resultados y las limitaciones de las medidas de seguridad implementadas.

## 14. Referencias

- Amazon Web Services. *Amazon EC2 User Guide for Linux Instances*.  
  https://docs.aws.amazon.com/ec2/

- Apache Software Foundation. *Apache HTTP Server Documentation*.  
  https://httpd.apache.org/docs/

- Canonical. *Ubuntu Server Documentation*.  
  https://ubuntu.com/server/docs

- Fail2Ban. *Fail2Ban Documentation*.  
  https://www.fail2ban.org/wiki/index.php/Main_Page

- National Institute of Standards and Technology. *Security and Privacy Controls for Information Systems and Organizations (SP 800-53 Rev. 5)*.  
  https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final

- Open Web Application Security Project. *OWASP Top 10*.  
  https://owasp.org/www-project-top-ten/

- Ubuntu Community Help Wiki. *Uncomplicated Firewall (UFW)*.  
  https://help.ubuntu.com/community/UFW

- HTML5 UP. *Lens — Responsive HTML5 and CSS3 Template*.  
  https://html5up.net/lens

- HTML5 UP. *Creative Commons Attribution 3.0 License*.  
  https://html5up.net/license

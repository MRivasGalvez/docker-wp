<div align="center">
<H1> 🌐 Creación de sitio web con Wordpress usando Docker y GCP </H1> 
</div>

---

 Este pequeño proyecto te ayudará a desplegar un sitio web de manera automatizada y transportable usando Docker Compose utilizando Google Cloud.

 ## ⚙️ Requisitos

Antes de comenzar asegúrate de tener:

- Una cuenta de [Google Cloud Platform](https://cloud.google.com/) con créditos activos (Puedes usar la prueba gratuita de USD$300).
- Conocimiento básico de Linux y terminal (Aunque iré paso a paso explicando los comandos).
- Comprado un dominio web para poder conectar la DNS (Yo usé [Namecheap](https://www.namecheap.com) pero puedes usar el que mas te acomode)

---

## 🚀 Pasos para desplegar

### 1. Crear instancia VM en GCP

- Ir a **Compute Engine** > **VM Instances**.

<img src="https://i.imgur.com/0ZNie93.png" />

- Crear una nueva instancia con los siguientes parámetros recomendados:
  - OS: Debian 12 o Ubuntu 24.04
  - Tipo de máquina: `e2-micro` o `e2-medium` (dependerá cuanta carga tendrá la web)
  - Permitir HTTP y HTTPS
  - Disco de al menos 10 GB
  - Región: Dependerá de tu [latencia](https://gcping.com) al datacenter y los costos estimativos mensuales que más te acomoden.

### 2. 📦Instalar Docker y Docker Compose.

Una vez iniciada la máquina virtual haz click en el botón SSH para abrir la consola del sistema.

``````bash
# Conectarse por SSH a la instancia
sudo apt update && sudo apt upgrade -y  # Este comando actualizará la lista de paquetes y los paquetes instalados en nuestro sistema.

# Instalar Docker
sudo apt install -y docker.io # Comando para instalar Docker

sudo systemctl enable docker # Este comando configura que Docker se inicie automáticamente al arrancar la VM

sudo usermod -aG docker $USER # Para ejecutar comandos de Docker sin sudo, debes reemplazar "USER" por el tuyo.

# Instalar Docker Compose
sudo apt install -y docker-compose

```````

### 🛠️3. Configurar dominio comprado (DNS) en GCP.

Buscar "Servicios de Red" 

<img src="https://i.imgur.com/H354thI.png"/>

Seleccionamos "Cloud DNS"

<img src="https://i.imgur.com/4hTTdGB.png"/>

Y creamos una nueva zona con las siguientes características:

Tipo de zona:
  - Pública

Nombre de la Zona:
  - Puede ser cualquier nombre.

Nombre del DNS:
  - El nombre de tu pagina web (sin www).

DNSSEC:
  - Desactivado

**Click en Crear**

Se crearán 2 conjuntos de registros, en los cuales tomaremos los datos del registro del tipo NS.

<img src= https://i.imgur.com/B5yl4Ut.png >

Y los llevaremos a nuestro proveedor de dominios.

En Namecheap nos vamos a Domain List y en la sección de Nameservers seleccionamos "Custom DNS" y luego en "+ ADD NAMESERVER" 

<img src=https://i.imgur.com/uFnkwur.png >

Ingresamos los datos del registro y damos click en el check verde ✅

[🌐¡Revisa aquí si tu DNS ya está disponible para el mundo!](https://dnschecker.org)

Si todo está bien, deberías ver muchos ✅✅✅✅

### 🚀4. Desplegar el contenedor.

En la consola de nuestra máquina virtual crearemos un repositorio llamado wordpress-docker (o el que tu quieras).

```bash

mkdir wordpress-docker && cd wordpress-docker
```

Copia el contenido del docker-compose.yml de este repositorio y modificalo según tus datos:

````yml
21.   DEFAULT_EMAIL: #PON TU CORREO AQUÍ

37.   VIRTUAL_HOST: #Tu dominio web aquí
38.   LETSENCRYPT_HOST: #Tu dominio web aquí
39.   LETSENCRYPT_EMAIL: #PON TU CORREO AQUÍ
````

**⚠️Importante: la estructura del docker-compose.yml debe ser tal cual está publicado, ya que una sangría mal puesta arruina la jerarquía del archivo y docker no desplegará el documento ⚠️**

Una vez editado nuestro documento, vamos a nuestra consola de la máquina virtual nuevamente y crearemos el archivo **docker-compose.yml** en nuestro repositorio creado con el siguiente comando:

````bash

vim docker-compose.yml
````

Se abrirá el editor de texto Vim y pegaremos nuestro contenido del archivo **docker-compose.yml**

Pulsamos la tecla ESC y escribimos: **:wq!**

Explicación:
- **:** Este símbolo indica que vas a ingresar un comando de línea.
- **q**: Representa el comando quit (salir).
- **!**: Este símbolo fuerza la salida, incluso si hay cambios sin guardar. 

Para comprobar el contenido del archivo que se haya guardado correctamente ejecutamos el comando:

````bash
cat docker-compose.yml
````

Con esto veremos el contenido del archivo.

Desplegaremos nuestro contenedor con el comando:

````bash
sudo docker-compose up -d
````
Si todo está bien podemos comprobar que los contenedores de Docker están corriendo:

````bash
docker ps
````

### 🧱5. Configurar Reglas de Firewall.

En Google Cloud iremos a Red de VPC y luego en Firewall

<img src= https://i.imgur.com/HpOJctV.png >

Crearemos una regla de Firewall 

<img src= https://i.imgur.com/0t4wmee.png >

#### Llena los campos así:

| Campo                      | Valor                                                      |
| -------------------------- | ---------------------------------------------------------- |
| **Nombre**                 | `allow-http-https` (puedes elegir otro si prefieres)       |
| **Red**                    | `default` (o la red donde está tu VM)                      |
| **Prioridad**              | `1000` (por defecto está bien)                             |
| **Dirección del tráfico**  | `Entrada`                                                  |
| **Acción en coincidencia** | `Permitir`                                                 |
| **Objetivos**              | `Todas las instancias en la red` (más simple para empezar) |
| **Filtros de origen**      | `Rangos de IP de origen`                                   |
| **Rangos de IP de origen** | `0.0.0.0/0` (esto permite tráfico desde cualquier IP)      |
| **Protocolos y puertos**   | Marca “Protocolos y puertos especificados” y pon: tcp:80,443    

### 🗿6. Configurar IP Estática.

Ve a Red de VPC -> Direcciones IP

Verás tu instancia así: 

| Nombre | Dirección IP | Tipo        | En uso por  |
| ------ | ------------ | ----------- | ----------- |
| vm-1   | 34.xx.xx.xx  | *Ephemeral* | Instancia X |

**Cambia de "Ephemeral" a "Static"**
Haz clic en "Reservar dirección estática" junto a tu VM.

Completa los campos así:

**Nombre:** ip-estatica-wordpress (o lo que quieras)
**Red:** default
**Región:** la misma región de tu VM
**Tipo:** IPv4
**Uso:** asociada a tu instancia actual
Haz clic en **"Reservar"**.

### ¡Listo! ya deberías poder ingresar "tudominio/wp-admin" en el navegador y entrar a la configuración de WordPress

<img src=https://i.imgur.com/pRT7Srb.png>

**⚠️Importante: Recuerda que Google te da USD$300 dólares para 2 meses aproximadamente, una vez concluida la prueba se te cobrará a tu tarjeta asociada a la cuenta, por lo que tendrás que evaluar si sigues con GCP o migrarás a otro servicio de tu preferencia, el objetivo de este contenedor es poder transportarlo fácilmente a través de cualquier plataforma Cloud.**
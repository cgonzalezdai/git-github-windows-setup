# Git y GitHub en Windows con evolución hacia WSL

Este repositorio documenta una configuración real de Git y GitHub que comenzó en Windows y posteriormente se amplió para trabajar también con proyectos dentro de WSL.

Si estás valorando una configuración parecida, aquí puedes encontrar respuesta a cuatro preguntas concretas:

* **¿Qué configuración de Git se utiliza en Windows?**
* **¿Qué cambia cuando un proyecto pasa a trabajarse dentro de WSL?**
* **¿Qué conviene mantener coherente entre Git Windows y Git Linux y qué no hace falta duplicar?**
* **¿Cómo comprobar que cada capa funciona antes de añadir la siguiente?**

No pretende ser una configuración universal. Es una referencia práctica basada en un entorno que se ha ido ampliando únicamente cuando apareció una necesidad concreta.

## Arquitectura

La separación actual es:

```text
WINDOWS
├── Visual Studio Code
├── PowerShell
├── Git for Windows
├── Git Credential Manager
└── herramientas Windows

WSL
└── Linux
    ├── Bash
    ├── Git Linux
    ├── runtimes de proyecto
    ├── repositorios Linux
    └── herramientas CLI
```

Visual Studio Code se ejecuta gráficamente en Windows.

Cuando se abre un proyecto dentro de WSL:

```text
VS Code Server
terminal
Git
runtime
filesystem
```

pertenecen al entorno Linux.

No se instala una segunda copia gráfica de VS Code dentro de WSL.

## Git en Windows

Configuración global relevante:

```text
init.defaultBranch=main
pull.ff=only
core.autocrlf=input
credential.helper=manager
http.sslbackend=schannel
```

La configuración efectiva puede revisarse con:

```powershell
git config --list --show-origin
```

La procedencia es importante porque permite distinguir entre configuración global, configuración local del repositorio y otros valores que puedan estar actuando.

### Rama inicial

```powershell
git config --global init.defaultBranch main
```

### Pull solo mediante fast-forward

```powershell
git config --global pull.ff only
```

Si las ramas local y remota han divergido, Git se detiene en lugar de decidir automáticamente entre merge y rebase.

### Finales de línea

```powershell
git config --global core.autocrlf input
```

Esta configuración mantiene LF en el repositorio y evita convertir automáticamente a CRLF durante el checkout.

Los repositorios con necesidades específicas deberían definirlas mediante `.gitattributes`.

Un aviso similar a:

```text
CRLF will be replaced by LF
```

no demuestra por sí mismo que se esté produciendo una conversión masiva de archivos. Antes de modificar la configuración conviene comprobar el `diff` real.

## Identidad de Git

Git necesita una identidad para crear commits:

```bash
git config --global user.name "Nombre Apellidos"
git config --global user.email "<email>"
```

Para repositorios públicos puede utilizarse la dirección `noreply` proporcionada por GitHub.

No debe inventarse. Debe utilizarse exactamente la dirección asignada a la cuenta.

La configuración puede verificarse con:

```bash
git config --global --get user.name
git config --global --get user.email
```

y en un commit real mediante:

```bash
git log -1 --format=fuller
```

## Git en WSL

WSL utiliza una instalación independiente de Git:

```text
Git for Windows
≠
Git Linux
```

Cada una mantiene su propia configuración.

No se copia automáticamente toda la configuración de Windows a Linux.

Sí se mantienen coherentes las decisiones que deben representar el mismo flujo de trabajo:

```text
user.name
user.email
init.defaultBranch
pull.ff
core.autocrlf
```

Otras opciones son específicas de cada plataforma.

Por ejemplo:

```text
http.sslbackend=schannel
```

pertenece a Git for Windows y no debe trasladarse mecánicamente a Git Linux.

La configuración Linux puede comprobarse igual:

```bash
git config --list --show-origin
```

## Reutilizar Git Credential Manager desde WSL

Git Linux puede reutilizar Git Credential Manager instalado en Windows sin necesidad de mantener un segundo gestor independiente.

La configuración utilizada es equivalente a:

```bash
git config --global credential.helper \
"/mnt/c/Program\\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

Conceptualmente:

```text
Git Linux
    ↓
Git Credential Manager de Windows
    ↓
GitHub
```

Git Windows y Git Linux continúan siendo instalaciones independientes. Lo que se reutiliza es la gestión de credenciales.

## Dónde guardar los repositorios

Para proyectos cuyo entorno natural es Linux se utiliza el filesystem Linux:

```text
/home/<usuario>/...
```

Esto permite mantener en el mismo entorno:

```text
repositorio
Git
terminal
runtime
herramientas
```

WSL permite trabajar también sobre archivos de Windows mediante `/mnt/c`, pero no se utiliza esa ubicación por defecto para un proyecto Linux únicamente por comodidad.

La ubicación se decide según el entorno real de trabajo del proyecto.

## Validación por capas

La configuración no se dio por válida simplemente porque los comandos fueran aceptados.

Se comprobó progresivamente.

### 1. Git local

Crear un repositorio temporal:

```bash
git init
git status
```

Comprobar:

```text
rama inicial
identidad
commit
configuración efectiva
```

### 2. Repositorio remoto

Después de validar Git local se comprobó por separado la comunicación con el repositorio remoto.

De esta forma, un posible problema de autenticación o transporte no se confunde con un problema de la configuración local de Git.

### 3. Git dentro de WSL

Una vez incorporado Linux se repitieron las pruebas utilizando:

```text
Git Linux
filesystem Linux
terminal Linux
```

### 4. Escritura remota sin cambio real

Para comprobar desde WSL que la autenticación y los permisos de escritura funcionaban se utilizó:

```bash
git push --dry-run origin HEAD:refs/heads/wsl-auth-test
```

La prueba permite validar la operación sin crear realmente la rama remota.

Después se comprobó que la rama no existía.

## Configuración comprobada

### Windows

```text
Git for Windows                  OK
VS Code                          OK
rama main                        OK
identidad Git                    OK
pull.ff=only                     OK
core.autocrlf=input              OK
Git Credential Manager           OK
acceso al remoto                 OK
escritura                        OK
```

### WSL

```text
Git Linux                        OK
filesystem Linux                 OK
VS Code Server                   OK
identidad Git                    OK
pull.ff=only                     OK
core.autocrlf=input              OK
GCM de Windows desde WSL         OK
acceso al remoto                 OK
push --dry-run                   OK
```

## Herramientas no añadidas por anticipado

Durante la construcción de este entorno no se instalaron algunas capas mientras no existiese un proyecto que las necesitase.

Por ejemplo:

```text
GitHub CLI
SSH específico para GitHub
Docker
Dev Containers
CUDA en WSL
otros runtimes o servicios
```

Que no formen parte de esta configuración no significa que no sean útiles.

Simplemente no son requisitos de esta arquitectura.

## Referencia de la instalación inicial en Windows

El repositorio nació documentando con bastante detalle la instalación inicial de Git for Windows.

Esa información sigue siendo útil como referencia, aunque ya no es la parte principal del documento.

<details>
<summary>Opciones utilizadas durante la instalación inicial</summary>

### Visual Studio Code

Instalación:

```text
Visual Studio Code Stable
User Installer
x64
```

Opciones relevantes:

```text
Abrir con Code para directorios       ON
Registrar Code como editor            ON
Agregar VS Code al PATH               ON
```

Comprobación:

```powershell
code --version
```

### Git for Windows

Instalación x64 en la ruta estándar.

Componentes utilizados:

```text
Open Git Bash here                    ON
Git LFS                               ON
Associate .git* files                 ON
Git GUI                               OFF
Scalar                                OFF
```

Git accesible desde:

```text
PowerShell
VS Code
otras herramientas
```

SSH incluido:

```text
Use bundled OpenSSH
```

Backend HTTPS:

```text
Use the native Windows Secure Channel library
```

Editor Git:

```text
Visual Studio Code
```

Terminal de Git Bash:

```text
MinTTY
```

Opciones adicionales:

```text
Enable file system caching            ON
Enable symbolic links                 OFF
```

Después de instalar:

```powershell
git --version
git config --list --show-origin
```

</details>

## Criterio utilizado

La configuración se construyó siguiendo este orden:

```text
entender
→ comprobar
→ decidir
→ instalar
→ validar
→ documentar
```

El repositorio no pretende mostrar todas las herramientas que podrían formar parte de un entorno Windows + Linux.

Documenta únicamente las decisiones que llegaron a utilizarse y las comprobaciones realizadas para saber que funcionaban.

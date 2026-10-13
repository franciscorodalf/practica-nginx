# Práctica NGINX: documentación con Zensical y blog con Jekyll

Despliegue con **NGINX** (en un contenedor Docker) de dos sitios web estáticos servidos por el mismo servidor:

- **Fase 1**: la documentación del proyecto [monium/gitanddocs](https://github.com/monium/gitanddocs), generada con Zensical.
- **Fase 2**: un blog personal generado con Jekyll.

**Repositorio remoto:** https://github.com/franciscorodalf/practica-nginx

## Arquitectura prevista

Un único contenedor NGINX publica el puerto 8080 del host. Según la cabecera `Host` de cada petición, NGINX elige un bloque `server` u otro y sirve los archivos de una carpeta distinta.

## Estado de la práctica

| Paso | Estado |
| --- | --- |
| 0. Comprobar herramientas | Hecho |
| 1. Repositorio, identidad y remoto | Hecho |
| Fase 1. Documentación con Zensical | En curso (copia y `site_url` hechos) |
| Fase 2. Blog con Jekyll | Pendiente |

---

## Paso 0. Comprobar las herramientas

**Qué:** comprobar que están instaladas las herramientas necesarias: Docker y Docker Compose para ejecutar NGINX en un contenedor, Git para el control de versiones y `uv` para generar la documentación de Zensical.

**Cómo:**

```bash
docker --version
docker compose version
git --version
uv --version
```

**Resultado:**

```text
Docker version 29.7.2, build a7dcaa6
Docker Compose version v5.4.0
git version 2.54.0 (Apple Git-157)
uv 0.11.26 (396ef7ce4 2026-06-30 aarch64-apple-darwin)
```

Docker descarga la imagen, crea el contenedor y muestra el mensaje, así que el cliente y el daemon funcionan. La arquitectura es Mac con Apple Silicon.

### Problema: el daemon de Docker no estaba en marcha

La primera vez, `docker run` falló:

```text
failed to connect to the docker API at unix:///Users/fran/.docker/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /Users/fran/.docker/run/docker.sock: connect: no such file or directory
```

**Causa:** Esto pasa porque aunque escribiendo en la terminal `docker --version` funciona aunque Docker Desktop esté cerrado, porque solo consulta el cliente. En cambio, `docker run` necesita el daemon, que se comunica por el socket `docker.sock`, y ese socket solo existe con Docker Desktop abierto.

**Solución:** abrir Docker Desktop, esperar a que arranque y repetir el comando.

## Paso 1. Repositorio, identidad de Git y remoto

### 1.1 Crear el repositorio

**Qué:** crear el proyecto en su propia carpeta, con su propio repositorio Git y la rama principal `main`.

**Cómo:**

```bash
cd ~/Desktop/practica-nginx
git init -b main
```

**Resultado:**

```text
Initialized empty Git repository in /Users/fran/Desktop/practica-nginx/.git/
```

### 1.2 Configurar la identidad

**Qué:** configurar el nombre y el correo del autor de los commits. Se hace **en local** (sin `--global`) porque solo quiero que afecte a este repositorio y tambien deben coincidir con los datos del aula virtual.

**Cómo:**

```bash
git config user.name "Francisco Yariel Rodriguez Alfonso"
git config user.email "francuban1278@icloud.com"
git config user.name && git config user.email
```

**Resultado:**

```text
Francisco Yariel Rodriguez Alfonso
francuban1278@icloud.com
```

**Problema:** en el primer intento se coló un carácter `´` al final del nombre (`Francisco Yariel Rodriguez Alfonso´`). Se detectó al mostrar la configuración y se corrigió volviendo a ejecutar `git config user.name` con el nombre correcto. Si no se hubiera corregido, la autoría de los commits no coincidiría con el aula virtual.

### 1.3 README y .gitignore

**Qué:** crear un `README.md` inicial y un `.gitignore` que excluya los sitios generados, porque se regeneran al compilar y no deben versionarse.

Contenido de `.gitignore`:

```text
.DS_Store
docs-zensical/site/
docs-zensical/.venv/
blog/_site/
blog/.jekyll-cache/
blog/.sass-cache/
```

**Cómo:**

```bash
git add .
git commit -m "Inicializar practica-nginx" -m "Se crea el repositorio, un README inicial y un .gitignore."
```

**Resultado:**

```text
[main (root-commit) f907c8b] Inicializar practica-nginx
 2 files changed, 7 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 README.md
```

### 1.4 Conectar con GitHub y subir la rama

**Qué:** enlazar el repositorio local con el remoto de GitHub (creado vacío, sin README ni licencia) y subir `main`.

**Cómo (primer intento, por SSH):**

```bash
git remote add origin git@github.com:franciscorodalf/practica-nginx.git
git push -u origin main
```

**Resultado:**

```text
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

#### Problema: GitHub rechaza la conexión SSH

**Diagnóstico:**

```bash
ssh -T git@github.com
```

```text
git@github.com: Permission denied (publickey).
```

**Causa:** no hay ninguna clave SSH de este equipo registrada en la cuenta de GitHub, así que GitHub no puede autenticar la conexión.

**Solución:** cambiar el remoto a HTTPS y autenticar Git con la CLI de GitHub (`gh`):

```bash
git remote set-url origin https://github.com/franciscorodalf/practica-nginx.git
gh auth login
```

Opciones elegidas: `GitHub.com` → `HTTPS` → autenticar Git con las credenciales de GitHub → `Login with a web browser`.

```text
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as franciscorodalf
```

**Segundo intento:**

```bash
git push -u origin main
```

```text
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 406 bytes | 406.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/franciscorodalf/practica-nginx.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

La rama `main` está publicada en el remoto y configurada para seguir a `origin/main`.

### Comprobaciones de este paso

| Qué se comprueba | Comando | Resultado esperado |
| --- | --- | --- |
| Identidad local de Git | `git config user.name && git config user.email` | Nombre y correo del aula virtual |
| Autoría de los commits | `git log --format='%an <%ae>' \| sort -u` | Una sola línea con esa identidad |
| Remoto configurado | `git remote -v` | URL HTTPS de `franciscorodalf/practica-nginx` |
| Rama sincronizada | `git status` | `Your branch is up to date with 'origin/main'` |

---

## Fase 1. Documentación con Zensical servida por NGINX

### 1.1 Rama de trabajo

**Qué:** trabajar la fase 1 en una rama propia, `fase-1`, y fusionarla en `main` cuando esté comprobada.

**Cómo:**

```bash
git branch fase-1
git checkout fase-1
git branch
```

**Resultado:**

```text
* fase-1
  main
```

### 1.2 Copiar el proyecto de documentación

**Qué:** incorporar al repositorio el proyecto del profesor [monium/gitanddocs](https://github.com/monium/gitanddocs) en la carpeta `docs-zensical/`.

**Cómo (primer intento):**

```bash
git clone --depth 1 https://github.com/monium/gitanddocs.git docs-zensical
git add docs-zensical
git commit -m "Clonar el proyecto de documentación gitanddocs" -m "Se clona el repositorio de informacion de zensical"
```

**Resultado:**

```text
warning: adding embedded git repository: docs-zensical
hint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
...
[fase-1 a170229] Clonar el proyecto de documentación gitanddocs
 1 file changed, 1 insertion(+)
 create mode 160000 docs-zensical
```

#### Problema: Git registró la carpeta como repositorio anidado

**Síntoma:** al intentar añadir un archivo de la carpeta, Git lo rechazaba:

```bash
git add docs-zensical/zensical.toml
```

```text
fatal: Pathspec 'docs-zensical/zensical.toml' is in submodule 'docs-zensical'
```

Además, `git status` solo mostraba `modified: docs-zensical (modified content)`, y ni `git add .` ni `git add docs-zensical` añadían nada.

**Causa:** `git clone` deja dentro de `docs-zensical/` su propia carpeta `.git`. Al hacer `git add`, el repositorio exterior no guarda los archivos, sino solo un puntero a un commit del repositorio interior. Eso es lo que indica el modo `160000` del commit. Si se entregara así, la carpeta aparecería vacía en el repositorio remoto y en el ZIP.

**Diagnóstico:** el cambio pendiente pertenecía al repositorio interior:

```bash
git -C docs-zensical status --short
```

```text
 M zensical.toml
```

**Solución:** guardar aparte el `zensical.toml` modificado, quitar el puntero del índice, borrar el `.git` interior y rehacer el commit, que todavía no estaba subido:

```bash
cp docs-zensical/zensical.toml /tmp/zensical.toml
git -C docs-zensical checkout -- zensical.toml
git rm --cached docs-zensical
rm -rf docs-zensical/.git
git add docs-zensical
git status
```

```text
Changes to be committed:
        deleted:    docs-zensical
        new file:   docs-zensical/.github/ISSUE_TEMPLATE/bug_report.md
        ...
        new file:   docs-zensical/docs/index.md
        new file:   docs-zensical/pyproject.toml
        new file:   docs-zensical/uv.lock
        new file:   docs-zensical/zensical.toml
```

```bash
git commit --amend -m "Se clona el proyecto de documentación gitanddocs"
```

```text
[fase-1 b26c5b5] Se clona el proyecto de documentación gitanddocs
 20 files changed, 1686 insertions(+)
 create mode 100644 docs-zensical/.github/ISSUE_TEMPLATE/bug_report.md
 ...
 create mode 100644 docs-zensical/zensical.toml
```

Ahora el commit contiene 20 archivos reales (modo `100644`) en lugar del puntero `160000`.

### 1.3 Ajustar `site_url`

**Qué:** cambiar la URL pública del sitio en `docs-zensical/zensical.toml`. Zensical la usa para generar las URL canónicas y el sitemap, y el valor original apuntaba a `example.com`.

**Cómo:** se recupera el archivo modificado que se había guardado aparte y se comprueba el cambio:

```bash
cp /tmp/zensical.toml docs-zensical/zensical.toml
git diff
```

**Resultado:**

```diff
 [project]
-site_url = "https://www.example.com/"
+site_url = "http://docs.localhost:8080/"
 site_name = "Documentation"
```

```bash
git add docs-zensical/zensical.toml
git commit -m "Ajustar site_url a la URL servida por nginx"
```

```text
[fase-1 8423053] Ajustar site_url a la URL servida por nginx
 1 file changed, 1 insertion(+), 1 deletion(-)
```

Comprobación de los dos commits:

```bash
git log --stat -2
```

```text
commit 8423053 (HEAD -> fase-1)
    Ajustar site_url a la URL servida por nginx
 docs-zensical/zensical.toml | 2 +-

commit b26c5b5
    Se clona el proyecto de documentación gitanddocs
 20 files changed, 1686 insertions(+)
```

### 1.4 Generar el sitio estático

**Qué:** compilar la documentación de Markdown a HTML, CSS y JavaScript estáticos, que es lo que NGINX va a servir.

**Cómo:**

```bash
cd docs-zensical
uv run zensical build --clean
ls site | head
cd ..
```

**Resultado:**

```text
Using CPython 3.14.3 interpreter at: /opt/homebrew/opt/python@3.14/bin/python3.14
Creating virtual environment at: .venv
Installed 11 packages in 311ms
Build started
Warning: anchor does not exist
    ╭─[ index.md:22:25 ]
 22 │ 4. [Instalar Zensical](#paso-3-añadir-la-dependencia-de-zensical)
...
4 issues found
Build finished in 0.46s
```

```text
404.html
assets
get-site
index.html
markdown
search.json
sitemap.xml
tema-visual
trabajando-con-git
```

`uv run` lee `pyproject.toml` y `.python-version`, crea el entorno virtual `.venv` e instala Zensical. El sitio queda en `docs-zensical/site/`, con `index.html` y `404.html`. Las carpetas `site/` y `.venv/` están en `.gitignore` porque se regeneran.

### 1.5 Configuración de NGINX

**Qué:** definir un bloque `server` que sirva la documentación.

Archivo `nginx/conf.d/docs.conf`:

```nginx
server {
    listen 80 default_server;
    server_name docs.localhost localhost;

    root  /usr/share/nginx/docs;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
}
```

| Directiva | Función |
| --- | --- |
| `listen 80 default_server` | Escucha en el puerto 80 del contenedor. Si llega un host que no coincide con ningún `server_name`, lo atiende este bloque |
| `server_name` | Nombres de host que atiende, según la cabecera `Host` |
| `root` | Carpeta base: la URI se añade a esta ruta |
| `index` | Archivo que se sirve cuando se pide un directorio |
| `try_files $uri $uri/ =404` | Busca el archivo, después el directorio con su `index` y, si no existe ninguno, responde 404 |
| `error_page 404 /404.html` | Usa la página 404 generada por Zensical |

```bash
git add nginx/conf.d/docs.conf
git commit -m "Configurar el server block de la documentación" -m "Añade docs.conf con un server que responde a docs.localhost y localhost"
```

```text
[fase-1 b9a43b9] Configurar el server block de la documentación
 1 file changed, 13 insertions(+)
 create mode 100644 nginx/conf.d/docs.conf
```

### 1.6 Levantar NGINX con Docker Compose

**Qué:** ejecutar NGINX en un contenedor con la imagen oficial, publicar el puerto 8080 del Mac y montar la configuración y el sitio generado.

Archivo `compose.yaml`:

```yaml
services:
  web:
    image: nginx:stable-alpine
    container_name: practica-nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./docs-zensical/site:/usr/share/nginx/docs:ro
    restart: unless-stopped
```

- `"8080:80"`: el puerto 8080 del host se redirige al 80 del contenedor. Por eso en el navegador se usa `:8080`, mientras que `listen` indica `80`.
- El volumen de `conf.d` sustituye la carpeta de la imagen. Así se elimina el `default.conf` de serie y solo quedan los bloques `server` propios.
- `:ro` monta en solo lectura, porque NGINX no necesita escribir en esas carpetas.

**Cómo:**

```bash
docker compose up -d
docker compose ps
docker compose exec web nginx -t
```

**Resultado:**

```text
 ✔ Image nginx:stable-alpine      Pulled
 ✔ Network practica-nginx_default Created
 ✔ Container practica-nginx       Started
```

```text
NAME             IMAGE                 COMMAND                  SERVICE   CREATED         STATUS         PORTS
practica-nginx   nginx:stable-alpine   "/docker-entrypoint.…"   web       6 seconds ago   Up 5 seconds   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

```bash
git add compose.yaml
git commit -m "Levantar nginx en contenedor con Docker Compose" -m "Define el servicio web con la imagen oficial nginx:stable-alpine, publica el puerto 8080 del host en el 80 del contenedor y monta en solo lectura la configuración y el sitio generado."
```

```text
[fase-1 eb4407d] Levantar nginx en contenedor con Docker Compose
 1 file changed, 10 insertions(+)
 create mode 100644 compose.yaml
```

### 1.7 Comprobaciones

**Qué:** verificar que NGINX sirve la documentación por los dos nombres de host, que devuelve 404 para recursos inexistentes y que registra las peticiones.

**Cómo:**

```bash
curl -I http://localhost:8080/
curl -I http://docs.localhost:8080/
curl -I http://localhost:8080/no-existe
curl -s http://docs.localhost:8080/ | grep -i "<title>"
docker compose logs web | tail -n 5
```

**Resultado:**

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
Content-Type: text/html
Content-Length: 37507
ETag: "6ace90e2-9283"

HTTP/1.1 200 OK
Server: nginx/1.30.5
Content-Type: text/html
Content-Length: 37507
ETag: "6ace90e2-9283"

HTTP/1.1 404 Not Found
Server: nginx/1.30.5
Content-Type: text/html
Content-Length: 14342
```

```text
        <title>Get started - Documentation</title>
```

```text
practica-nginx  | 192.168.65.1 - - [13/Oct/2026:20:17:22 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.7.1" "-"
practica-nginx  | 192.168.65.1 - - [13/Oct/2026:20:17:22 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/8.7.1" "-"
practica-nginx  | 192.168.65.1 - - [13/Oct/2026:20:17:22 +0000] "HEAD /no-existe HTTP/1.1" 404 0 "-" "curl/8.7.1" "-"
practica-nginx  | 192.168.65.1 - - [13/Oct/2026:20:17:22 +0000] "GET / HTTP/1.1" 200 37507 "-" "curl/8.7.1" "-"
```

**Interpretación:**

- `localhost` y `docs.localhost` devuelven el mismo `ETag`, es decir, el mismo archivo: los dos nombres están en el `server_name` del mismo bloque.
- La respuesta 404 ocupa 14342 bytes. Es la página `404.html` de Zensical, servida por `error_page`; la página 404 propia de NGINX ocupa unos 150 bytes.
- La IP `192.168.65.1` de los logs es la puerta de enlace de la máquina virtual de Docker Desktop, por la que pasan las peticiones del Mac al contenedor.
### 8.1.1. Docker

**1. ¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.**
Una imagen es una plantilla estática y de solo lectura con el software empaquetado (en G2 descargamos los binarios de Oracle). Un contenedor es esa imagen instanciada y en ejecución (en G4 levantamos el proceso vivo `oralab-26ai` con memoria y estado propio).

**2. En el Ejercicio G5 el archivo nota.txt desapareció y en el G6 no. Explica por qué.**
En G5 se escribió en la capa de almacenamiento temporal del contenedor, la cual se destruye al eliminarlo. En G6 se montó un volumen Docker persistente (`oralab-26ai-data`), que reside fuera del ciclo de vida del contenedor.

**3. ¿Qué diferencia hay entre docker ps y docker ps -a, y qué significa STATUS = Exited (0)?**
`docker ps` lista únicamente los contenedores en ejecución; `docker ps -a` muestra todos (activos y detenidos). `Exited (0)` indica que el contenedor finalizó su proceso limpiamente, con código de salida sin errores.

**4. En -p 8181:8181, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con -p 80:8080 en el ejercicio de nginx?**
La sintaxis es `puerto_host:puerto_contenedor`. El primer valor es tu equipo y el segundo el contenedor. Con `-p 80:8080` la conexión fallaría, porque Nginx escucha por defecto en el puerto 80 dentro del contenedor y no en el 8080.

**5. ¿Por qué un contenedor de Oracle se queda en marcha y el de hello-world termina solo?**
Un contenedor vive mientras su proceso principal (PID 1) esté activo. `hello-world` solo ejecuta un comando puntual que imprime texto y finaliza; Oracle corre demonios en segundo plano (instancia y listener) que quedan esperando conexiones continuamente.

**6. ¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos :latest?**
La etiqueta `:latest` es un puntero mutable que puede cambiar con nuevas versiones. El *digest* es el hash SHA-256 criptográfico e inmutable del contenido exacto, lo que garantiza trazabilidad y reproducibilidad idéntica del entorno.

**7. ¿Qué comando borraría realmente los datos de Oracle? ¿Por qué docker rm oralab-26ai no lo hace?**
El comando es `docker volume rm oralab-26ai-data`. `docker rm oralab-26ai` solo elimina el contenedor, pero los datos residen de forma desacoplada dentro del volumen de Docker.

---

### 8.1.2. Git, organización y evidencia

**8. ¿Por qué este laboratorio se hace dentro del repositorio oracle-database-lab, con Issue, branch y Pull Request, en vez de en una carpeta aparte?**
Para seguir el estándar profesional (GitHub Flow): aislar cambios en ramas sin tocar código funcional, vincular el progreso al Issue correspondiente y validar las evidencias antes de fusionar en la rama principal.

**9. ¿Qué diferencia hay entre source 00-config.sh y bash 00-config.sh? ¿Por qué usamos source?**
`bash` ejecuta el script en una subshell y las variables se pierden al finalizar. `source` lo ejecuta en la sesión actual, dejando las variables de entorno (`$PORT_DB`, `$ORDS_PORT`, etc.) cargadas y listas para usar.

**10. Explica cada parte del nombre 20260915T091230Z_02-docker.script.log.**
* `20260915`: Fecha (YYYYMMDD).
* `T`: Separador estándar ISO 8601.
* `091230`: Hora, minuto y segundo (HHMMSS).
* `Z`: Zona horaria UTC (*Zulu*).
* `_02-docker`: Identificador de la tarea/tema.
* `.script.log`: Formato de salida de la utilidad `script`.

**11. ¿Para qué sirve .gitattributes y qué error evita?**
Fuerza el uso de finales de línea Unix (`eol=lf`). Evita que Git en Windows convierta los saltos a `CRLF`, lo que causaría fallos de ejecución en bash/Linux del tipo `\r: command not found`.

**12. ¿Por qué en este Pull Request elegimos Create a merge commit en lugar de Squash and merge?**
Porque `Squash` colapsa todo el historial en un único commit genérico. `Create a merge commit` mantiene intacto cada commit individual, permitiendo verificar y evaluar cronológicamente cada paso del laboratorio.

---

### 8.1.3. Seguridad

**13. Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.**
1. `.gitignore` bloqueando archivos con secretos.
2. Plantilla `.env.example` pública con variables sin valores reales.
3. Archivo local `config/.env` con las claves reales.
4. Inyección en memoria o stdin sin persistir en scripts.
Si omites la primera capa (`.gitignore`), el archivo con las contraseñas reales se incluirá en el commit y terminará publicado en el repositorio remoto.

**14. ¿Por qué no escribimos la contraseña directamente en el comando docker run, aunque el script no se suba a Git?**
Porque quedaría guardada en texto plano en el historial de comandos (`~/.bash_history`), visible para cualquier usuario del sistema mediante `ps aux`, y almacenada en los metadatos de inspección del contenedor (`docker inspect`).

**15. Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?**
No basta, ya que queda registrada permanentemente en el historial de Git. Lo primero es **cambiar (rotar) la contraseña en la base de datos de inmediato** y después purgar el historial del repositorio con herramientas como `git filter-repo`.

---

### 8.1.4. Oracle y herramientas

**16. ¿Por qué no usamos SPOOL ni @archivo.sql con sqlplus dentro del contenedor, y qué hicimos en su lugar?**
Porque requeriría copiar archivos hacia adentro y afuera del contenedor. En su lugar, canalizamos la ejecución desde el host vía entrada estándar (`docker exec -i ... sqlplus`) y capturamos la salida localmente con `tee`.

**17. ¿Qué hace WHENEVER SQLERROR EXIT SQL.SQLCODE al inicio de V000 y V001, y qué pasaría sin esa línea?**
Ordena a SQL*Plus detener la ejecución y retornar el código de error al sistema operativo en caso de fallo. Sin ella, SQL*Plus continuaría ejecutando sentencias posteriores a ciegas, dejando el esquema en un estado roto o inconsistente.

**18. ¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?**
Es un script incremental y versionado para alterar la estructura de la base de datos. No se editan una vez aplicadas para evitar discrepancias (*schema drift*) entre entornos; cualquier modificación requiere crear un nuevo archivo de migración (`V002`).

**19. ¿Por qué en SQL Developer se usa el servicio FREEPDB1 y no FREE ni un SID?**
`FREE` es el contenedor raíz (CDB), reservado para administración general; los objetos y esquemas de negocio residen en la base de datos conectable (`FREEPDB1`). El SID es un mecanismo obsoleto frente al estándar de nombres de servicio.

**20. ¿Qué aporta SQLcl frente a SQL*Plus, y por qué un DBA debe dominar ambas?**
SQLcl añade autocompletado, soporte directo de exportación JSON/CSV e integración con Liquibase. Un DBA debe conocer ambas porque SQL*Plus viene instalado por defecto en cualquier servidor para soporte de contingencias, mientras que SQLcl agiliza la operativa diaria.

---

### 8.1.5. Entorno de trabajo

**21. ¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.**
Git Bash es un emulador sobre Windows, mientras que WSL 2 es un entorno Linux real. En Ubuntu se eliminan:
1. La alteración y corrupción automática de rutas POSIX en Windows (que rompe montajes de Docker con `-v`).
2. La falta de un gestor de paquetes (`apt`) y de utilidades completas de sistema (`tmux`, `script`, `ss`).

**22. ¿Por qué clonamos el repositorio en ~/oracle-database-lab y no trabajamos sobre la carpeta de Windows (/mnt/c/...)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?**
* **Ubicación (`~` vs `/mnt/c/`):** Trabajar en `/mnt/c/` penaliza drásticamente el rendimiento por el puente de traducción del sistema de archivos y da problemas con permisos POSIX; en `~` se usa `ext4` de forma nativa a máxima velocidad.
* **Bash frente a Zsh:** Para garantizar portabilidad. Bash es el estándar por defecto en entornos corporativos y evita problemas de sintaxis con la gestión de comodines (globbing) y arrays propios de Zsh.

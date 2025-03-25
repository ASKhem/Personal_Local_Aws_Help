## Instalación de Docker

Instalación oficial de Docker Engine en WSL Ubuntu (Alternativa a Docker Desktop por el tema de Administrador, ya que puede dar problemas despues al hacer docker compose up -d con el Makefile):

Actualizar el sistema e instalar prerrequisitos:
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
```

Añadir la clave GPG oficial de Docker:
```bash
  sudo install -m 0755 -d /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

**🛠️ Configurar el repositorio:**
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**🐳 Actualizar el índice de paquetes e instalar Docker:**
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**🚀 Iniciar el servicio de Docker:**
```bash
sudo service docker start
```

**⚙️ Configurar Docker para iniciar automáticamente (crear archivo de configuración):**
```bash
sudo tee /etc/wsl.conf << EOF
[boot]
command="service docker start"
EOF
```

**👤 Añadir tu usuario al grupo docker para usar Docker sin sudo:**
```bash
sudo usermod -aG docker $USER
```

**✅ Verificar la instalación:**
```bash
docker --version
docker run hello-world
```
Después de ejecutar estos comandos, necesitarás:

*   🚪 Cerrar tu terminal actual
*   🔄 Reiniciar WSL (desde PowerShell/CMD de Windows: `wsl --shutdown`)
*   💻 Abrir una nueva terminal WSL

¿Quieres proceder con la instalación? Si es así, necesitarás cambiar a modo Act para ejecutar estos comandos paso a paso.

---

⚠️ **Error de Permisos en Docker** ⚠️

```bash
askhem@NTTD:~$ docker run hello-world
docker: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied

Run 'docker run --help' for more information
```

Este error indica que el usuario no tiene los permisos necesarios para conectar con el daemon de Docker. Hay dos formas de resolver esto:

**✅ Solución permanente (recomendada):**
```bash
# Añadir usuario al grupo docker
sudo usermod -aG docker $USER

# Verificar que el usuario está en el grupo
groups

# Reiniciar el servicio de Docker
sudo service docker restart
```
Después de ejecutar estos comandos, necesitarás:

*   🚪 Cerrar completamente la sesión de terminal
*   🔄 Reiniciar WSL desde PowerShell/CMD de Windows con: `wsl --shutdown`
*   💻 Abrir una nueva terminal WSL

**🧪 Probar de nuevo:**
```bash
docker run hello-world
```
Esta solución permanente te permitirá usar Docker sin necesidad de `sudo`. ¿Quieres proceder con la solución? Si es así, necesitarás cambiar a modo Act para ejecutar estos comandos.

---

**Editar el Makefile**
Cambiar las lineas del docker compose por estas

```bash
docker compose up -d
docker compose down
```

## Configuración del proyecto

**Instalación de Node.js 18:**
```bash
# Añadir repositorio
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
# Instalar Node.js
sudo apt-get install -y nodejs
```

☁️ **Instalación de herramientas CDK:**
```bash
sudo npm install -g aws-cdk-local aws-cdk
```

🐍 **Configuración del entorno Python:**
```bash
# Instalar venv
sudo apt install python3-venv python3-full

# Crear y activar entorno virtual
python3 -m venv .venv
source .venv/bin/activate

# Instalar dependencias en el entorno virtual (está en la carpeta cdk)
cd cdk
pip3 install -r requirements.txt
cd ..
```

🔑 **Configurar credenciales AWS locales:**
```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
```

🐳 ⛰️ **Comandos de Docker/LocalStack:**
```bash
# Iniciar LocalStack
make start-localstack
# o
docker compose up -d

# Detener LocalStack
make localstack-stop
# o
docker compose down
```

☁️ **Comandos CDK:**
```bash
make bootstrap    # Inicializa el entorno CDK
make deploy      # Despliega la Lambda y el bucket S3
```

⚠️ **Solución al error de contexto CDK:**

Si encuentras este error:
```
A reserved context key ('context.aws:') key was found in /home/askhem/Proyects/personal_local_aws/personal_local_aws/cdk/cdk.json, it might cause surprising behavior and should be removed.
```

Este error ocurre porque el archivo `cdk/cdk.json` contiene una clave de contexto con el prefijo reservado `aws:`. Para solucionarlo:

1. Abrir el archivo `cdk/cdk.json`
2. Modificar la sección de contexto eliminando el prefijo `aws:cdk:`:

```json
{
  "app": "python3 app.py",
  "context": {
    "enableDiffNoFail": "true"  // Antes era "aws:cdk:enableDiffNoFail"
  },
  "requireApproval": "never"
}
```

Este cambio elimina el warning y mantiene la misma funcionalidad.

🧪 **Comandos de prueba:**
```bash
make invoke         # Ejecuta la Lambda
make list-bucket   # Lista contenido del bucket
make get-object    # Obtiene el archivo creado
```

❗ **Problema principal:** El problema principal que encontramos fue el timeout en la Lambda al intentar invocarla. Esto necesitará revisión con el profesor ya que podría estar relacionado con la configuración de red entre Docker/WSL2.

⏱️ **Solución al error de timeout en Lambda:**

Si encuentras este error al invocar la Lambda:
```
"errorMessage":"Task timed out after 10.00 seconds"
```

El problema se debe a dos factores:

1. El endpoint de LocalStack no es accesible desde la Lambda
2. El timeout configurado es muy corto para un entorno local

Para solucionarlo, modificar el archivo `cdk/lambda_s3_local/lambda_s3_local_stack.py`:

1. Cambiar el endpoint para usar la IP del contenedor de LocalStack:
```python
environment={
    "BUCKET_NAME": bucket.bucket_name,
    "LOCALSTACK_ENDPOINT": "http://172.19.0.2:4566"  # IP del contenedor LocalStack
}
```

> **Nota**: La Lambda se ejecuta dentro de un contenedor y no puede acceder a localhost del host. Para obtener la IP correcta del contenedor LocalStack, usar el comando:
> ```bash
> docker inspect localstack | grep IPAddress
> ```
>
> **Explicación detallada de las IPs:**
> 1. `localhost:4566` no funciona porque:
>    - "localhost" dentro de un contenedor se refiere al propio contenedor, no al host
>    - La Lambda se ejecuta en su propio contenedor, por lo que "localhost" apunta al contenedor de la Lambda, no a LocalStack
> 
> 2. `172.17.0.1:4566` (IP del puente Docker) no funciona porque:
>    - Esta es la IP del puente de red predeterminado de Docker
>    - LocalStack se ejecuta en una red personalizada `personal_local_aws_default`
>    - Esta red usa un rango de IPs diferente (172.19.0.x)
>
> 3. `172.19.0.2:4566` funciona porque:
>    - Es la IP real del contenedor LocalStack en su red
>    - Los contenedores en la misma red Docker pueden comunicarse usando sus IPs internas
>    - Esta IP la obtenemos con el comando `docker inspect localstack`

2. Aumentar el timeout de la Lambda:
```python
lambda_fn = _lambda.Function(
    # ... otros parámetros ...
    timeout=Duration.seconds(30),  # Aumentado de 10 a 30 segundos
)
```

Después de hacer estos cambios, ejecutar:
```bash
make deploy    # Redespliega la Lambda con los cambios
make invoke    # Prueba la Lambda nuevamente
```


### Algunos comanods útiles para gestionar entornos virtuales:

```bash
# Desactivar entorno virtual actual
deactivate

# Eliminar carpeta del entorno virtual
rm -rf .venv/

# Listar entornos virtuales instalados
ls ~/.virtualenvs/

# Desinstalar todos los paquetes pip
pip freeze | xargs pip uninstall -y

# Crear nuevo entorno virtual
python3 -m venv .venv

# Activar el nuevo entorno
source .venv/bin/activate

# Instalar dependencias específicas
pip install -r requirements.txt

# Verificar el entorno
which python

# Ver paquetes instalados
pip list

# Ver versión de Python
python --version
```

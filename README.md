## ¡Hola, bienvenido a mi proyecto Training-DevOps-SelfHosted! 

Hey, ¿qué tal? Soy Ismael, y este es mi repositorio `training-devops-selfhosted`, consiste en una automatización del taller https://github.com/SirNeo/training-devops. Aquí te cuento cómo funciona todo, qué vas a encontrar y cómo puedes ponerlo a rodar en tu máquina. 

### ¿De qué va este proyecto? 
Mira, este repositorio es como una caja de herramientas para automatizar el ciclo de vida de dos aplicaciones: un frontend en **Angular** y un backend en **Spring Boot**. Lo que hacemos aquí es construirlas, empaquetarlas en imágenes Docker, subirlas a Docker Hub y luego desplegarlas en un clúster de Kubernetes que corre en **Rancher Desktop**. Todo esto lo automatizamos con un pipeline en **GitHub Actions**, y lo ejecutamos en un runner autohospedado que tengo configurado en mi máquina con WSL2.

### ¿Qué hay dentro del repositorio? 
Te explico cómo está organizado todo para que no te pierdas:

- **`training-angular/helloworld/`**: Aquí vive el frontend, una app sencilla en Angular. Tiene su `Dockerfile` para crear la imagen y un `package.json` con todo lo que necesita para funcionar.
  
- **`training-spring-boot/helloworld/`**: Este es el backend, una app básica en Spring Boot. La compilamos con Maven (usamos un `pom.xml` para las dependencias) y también tiene su `Dockerfile` para empaquetarla en una imagen.

- **`k8s/`**: Aquí están los archivos de Kubernetes (`angular-deployment.yaml` y `springboot-deployment.yaml`). Estos archivos le dicen a Kubernetes cómo desplegar las apps, qué imágenes usar, cuántas réplicas queremos, los puertos, etc.

- **`.github/workflows/ci.yml`**: Este es el cerebro de la automatización. Es el pipeline de GitHub Actions que hace todo el trabajo: compila, empaqueta, sube las imágenes a Docker Hub y las despliega en Kubernetes. Lo configuré para que use mi clúster de Rancher Desktop.

- **`README.md`**: ¡El archivo que estás leyendo ahora mismo!  Lo he escrito para que tengas todo claro y puedas seguir los pasos sin liarte.

### ¿Qué necesitas para empezar? 
Antes de ponerte manos a la obra, asegúrate de tener estas cosas listas en tu máquina:

- **Git**: Para clonar el repo y trabajar con el código.
- **Docker/Podman**: Lo usamos para crear y manejar las imágenes de las apps.
- **Maven**: Necesario para compilar el backend en Spring Boot (asegúrate de tener una versión reciente, como la 3.x).
- **Rancher Desktop**: Esto nos da un clúster de Kubernetes local donde desplegamos las apps. Asegúrate de que esté corriendo y que tengas el archivo `.kube/config` (en Windows suele estar en `C:\Users\TuUsuario\.kube\config`).
- **Un runner autohospedado de GitHub Actions**: Yo lo tengo configurado en WSL2, en `/mnt/c/Users/jose.i.marin.ghalem/actions-runner`. Si no sabes cómo montarlo, te cuento más abajo.
- **Cuenta en Docker Hub**: Necesitas un usuario y un token para subir las imágenes. Esto lo configuramos como secrets en GitHub.

### Paso a paso para echarlo a andar 
Te lo pongo fácil, sigue estos pasos y en un rato tendrás todo funcionando:

1. **Clona el repositorio**:
   Abre tu terminal (en WSL2 o donde quieras) y clona el repo:
   ```bash
   git clone https://github.com/ismaelGTI/training-devops-selfhosted.git
   cd training-devops-selfhosted
   ```

2. **Configura tu runner autohospedado**:
   Si no tienes un runner, ve a tu repositorio en GitHub, entra en "Settings" > "Actions" > "Runners" y sigue las instrucciones para añadir uno. Yo lo puse en `/mnt/c/Users/jose.i.marin.ghalem/actions-runner`. Una vez configurado, entra en ese directorio y ejecuta:
   ```bash
   ./run.sh
   ```
   Esto lo deja listo para que el pipeline lo use.

3. **Copia el `.kube/config` a WSL2**:
   Como estamos usando Rancher Desktop, necesitamos un archivo de configuración para que Kubernetes sepa dónde desplegar. Haz esto desde WSL2:
   ```bash
   mkdir -p ~/.kube
   cp /mnt/c/Users/TuUsuario/.kube/config ~/.kube/config
   ```
   Luego, comprueba que todo esté bien:
   ```bash
   kubectl config current-context
   kubectl cluster-info
   ```
   Si ves que el contexto es `rancher-desktop` y te sale info del clúster, ¡estás listo!

4. **Añade tus credenciales de Docker Hub**:
   En GitHub, ve a "Settings" > "Secrets and variables" > "Actions". Añade estas dos variables:
   - `DOCKERHUB_USERNAME`: Tu usuario de Docker Hub.
   - `DOCKERHUB_TOKEN`: Tu token de acceso (lo puedes generar en Docker Hub).

5. **Lanza el pipeline**:
   Ahora solo tienes que hacer un `git push` para que el pipeline se dispare automáticamente. También, si prefieres, ve a la pestaña "Actions" en GitHub, selecciona el workflow `CI - Build and Push Images` y dale a "Run workflow". ¡El pipeline hace el resto!

### ¿Qué hace el pipeline? 
El pipeline es como un asistente que hace todo el trabajo pesado por ti. Aquí te explico qué hace en cada paso:

- **Checkout**: Clona el código del repositorio para que el runner pueda trabajar con él.
- **Setup JDK**: Configura Java 17, necesario para compilar el backend en Spring Boot.
- **Debug directory structure**: Inspeccionamos todos los archivos en las carpetas importante para comprobar el funcionamiento correcto de la clonación y las rutas.
- **Configure kubeconfig**: Copia el archivo `.kube/config` de Rancher Desktop para que Kubernetes sepa dónde desplegar.
- **Build and push images**: Construye las imágenes Docker para Angular (`ismaelmrn/training-angular:0.0.1`) y Spring Boot (`ismaelmrn/training-spring-boot:0.0.1`), y las sube a Docker Hub.
- **Deploy to Kubernetes**: Usa los archivos YAML en `k8s/` para desplegar las apps en el clúster de Rancher Desktop.

### ¿Cómo sé si funcionó? 
Cuando el pipeline termine (puedes ver los logs en la pestaña "Actions" de GitHub), haz esto para confirmar que todo está OK:

- Abre Rancher Desktop en Windows y ve a la sección de Kubernetes. Deberías ver los deployments `angular` y `springboot` corriendo.

### Consejos y trucos 
- **Personaliza los despliegues**: Si quieres cambiar algo (como los puertos o el número de réplicas), edita los archivos `angular-deployment.yaml` y `springboot-deployment.yaml` en la carpeta `k8s/`.
- **Para producción**: Este setup es genial para aprender, pero si quieres llevarlo a producción, te recomiendo usar un clúster remoto como AWS EKS o Google GKE. Solo tendrías que cambiar el `.kube/config`.
- **Problemas**: Si algo falla, revisa los logs del pipeline en GitHub. Si no das con la solución, mándame los logs y lo miramos juntos.

---

Si tienes dudas o quieres charlar sobre DevOps, ya sabes dónde encontrarme. ¡A darle caña! 

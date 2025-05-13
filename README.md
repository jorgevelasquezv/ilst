# IBM Licensing Operator Deployment with GitOps

Este proyecto utiliza GitOps para gestionar la implementación del operador de IBM Licensing en un clúster de Kubernetes. Se basa en ArgoCD y Helm para automatizar y simplificar la configuración y despliegue de recursos personalizados (CRs) necesarios para el operador.

---

## Tabla de Contenidos

- [Descripción](#descripción)
- [Requisitos Previos](#requisitos-previos)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Instalación](#instalación)
- [Configuración](#configuración)
- [Uso](#uso)
- [Personalización](#personalización)
- [Solución de Problemas](#solución-de-problemas)
- [Contribuciones](#contribuciones)
- [Licencia](#licencia)

---

## Descripción

El **IBM Licensing Operator** es un componente clave para gestionar licencias en entornos de Kubernetes. Este proyecto utiliza un enfoque GitOps para implementar y gestionar el operador y sus recursos personalizados (CRs) mediante ArgoCD y Helm.

### Características principales:
- Implementación automatizada del operador de IBM Licensing.
- Gestión de recursos personalizados (CRs) como `IBMLicensing`.
- Configuración dinámica mediante plantillas Helm.
- Soporte para secretos de `imagePull` y configuración de Ingress.
- Integración con ArgoCD para sincronización continua.

---

## Requisitos Previos

Antes de comenzar, asegúrate de cumplir con los siguientes requisitos:

1. **Clúster de Kubernetes**:
   - Versión mínima: 1.21 o superior.
   - Acceso al servidor API del clúster.

2. **Herramientas instaladas**:
   - [kubectl](https://kubernetes.io/docs/tasks/tools/): CLI para interactuar con Kubernetes.
   - [Helm](https://helm.sh/): Gestor de paquetes para Kubernetes.
   - [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/): CLI para gestionar ArgoCD.

3. **Acceso a imágenes de contenedor**:
   - Credenciales para acceder a los registros de contenedores necesarios.

4. **Configuración de ArgoCD**:
   - ArgoCD debe estar instalado y configurado en el clúster.

---

## Estructura del Proyecto

La estructura del proyecto es la siguiente:

```bash
├── gitops/
│   ├── argocd/
│   │   ├── base-dev/
│   │   │   ├── helm/
│   │   │   │   ├── templates/
│   │   │   │   │   ├── cr.yaml
│   │   │   │   ├── values.yaml
│   │   │   ├── kustomization.yaml
│   ├── overlays/
│       ├── dev/
│       ├── prod/
├── config/
│   ├── crd/
│   │   ├── bases/
│   │   │   ├── operator.ibm.com_ibmlicensingdefinitions.yaml
│   ├── manager/
│       ├── manager.yaml
```

### Archivos clave:

- **`cr.yaml`**: Plantilla Helm para definir recursos personalizados (CRs).
- **`values.yaml`**: Archivo de configuración para valores predeterminados de Helm.
- **`operator.ibm.com_ibmlicensingdefinitions.yaml`**: Definición de la CRD para `IBMLicensing`.
- **`manager.yaml`**: Configuración del operador.

---

## Instalación

Sigue estos pasos para instalar el operador de IBM Licensing:

1. **Configura ArgoCD**:
   Apunta ArgoCD al repositorio Git para sincronizar los recursos:
   ```bash
   argocd app create ibm-licensing-operator \
     --repo <URL_DEL_REPOSITORIO> \
     --path gitops/argocd/base-dev \
     --dest-namespace ibm-licensing \
     --dest-server https://kubernetes.default.svc
   ```

2. **Sincroniza la aplicación**:
   ```bash
   argocd app sync ibm-licensing-operator
   ```

---

## Configuración

### Valores de Helm
El archivo `values.yaml` contiene configuraciones clave. Algunos valores importantes son:

```yaml
global:
  imagePullSecret: "my-global-secret"

spec:
  imagePullSecrets:
    - name: "my-local-secret"
```

- **`global.imagePullSecret`**: Define un secreto global para acceder a imágenes de contenedor.
- **`spec.imagePullSecrets`**: Lista de secretos adicionales para acceder a imágenes.

### Personalización de CR
Puedes personalizar el recurso `IBMLicensing` modificando el archivo `cr.yaml`.

---

## Uso

1. **Verifica el estado del operador**:
   ```bash
   kubectl get pods -n ibm-licensing
   ```

2. **Verifica el estado del CR**:
   ```bash
   kubectl get IBMLicensing -n ibm-licensing
   ```

---

## Personalización

### Agregar secretos de `imagePull`
Si necesitas agregar secretos adicionales para acceder a imágenes de contenedor, configúralos en `values.yaml`:

```yaml
spec:
  imagePullSecrets:
    - name: "my-secret"
```

---

## Solución de Problemas

### Error de conexión al servidor API
Si ves un error como:
```
dial tcp <IP>:443: i/o timeout
```
Asegúrate de que:
- Estás conectado a la red correcta (VPN, etc.).
- El contexto de `kubectl` está configurado correctamente:
  ```bash
  kubectl config get-contexts
  kubectl config use-context <nombre-del-contexto>
  ```

### Validación fallida al aplicar un CR
Si obtienes un error de validación, intenta desactivar la validación:
```bash
kubectl apply -f <archivo.yaml> --validate=false
```

---

## Licencia

Este proyecto está licenciado bajo la [Licencia Apache 2.0](LICENSE).
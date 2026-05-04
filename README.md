# Taller: Despliegue de Microservicios con Kubernetes

Este taller consiste en migrar la aplicación de microservicios previamente desplegada con Vagrant + Docker a Kubernetes usando Minikube.

---

## Arquitectura

La aplicación se compone de 7 microservicios desplegados como Pods en un cluster de Kubernetes:

| Servicio | Imagen | Puerto | Tipo de Service |
|---|---|---|---|
| frontend | lauravrv/frontend-app:v1 | 80 | ClusterIP |
| auth-api | lauravrv/auth-app:v1 | 8000 | ClusterIP |
| users-api | lauravrv/users-app:v1 | 8083 | ClusterIP |
| todos-api | lauravrv/todos-app:v1 | 8082 | ClusterIP |
| log-processor | lauravrv/processor-app:v1 | — | Sin Service |
| redis | redis:7 | 6379 | ClusterIP |
| zipkin | openzipkin/zipkin | 9411 | ClusterIP |

---


## Estructura del repositorio

```
k8s-microservices/
├── README.md
├── redis-deployment.yaml
├── zipkin-deployment.yaml
├── users-deployment.yaml
├── auth-deployment.yaml
├── todos-deployment.yaml
├── processor-deployment.yaml
├── frontend-deployment.yaml
└── ingress.yaml
```

---

## Requisitos

- Minikube instalado y corriendo
- kubectl instalado
- Acceso a internet para pull de imágenes desde Docker Hub

---

## Despliegue

### 1. Iniciar Minikube

```bash
minikube start
```

### 2. Habilitar el Ingress

```bash
minikube addons enable ingress
```

### 3. Aplicar todos los manifiestos

```bash
kubectl apply -f .
```

### 4. Verificar que todos los pods estén corriendo

```bash
kubectl get pods
```


### 5. Obtener la IP del Ingress

```bash
kubectl get ingress
```

---

## Verificación

### Login

```bash
curl -s -X POST http://<INGRESS-IP>/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}'
```

Respuesta esperada:

```json
{"accessToken":"eyJhbGci..."}
```

### Crear un TODO

```bash
TOKEN=$(curl -s -X POST http://<INGRESS-IP>/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin"}' | sed -E 's/.*"accessToken":"([^"]+)".*/\1/')

curl -s -X POST http://<INGRESS-IP>/todos \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"content":"probando kubernetes"}'
```

Respuesta esperada:

```json
{"content":"probando kubernetes","id":1}
```

### Verificar Log Processor (eventos de Redis)

```bash
kubectl logs deployment/processor
```

### Verificar Zipkin

```bash
kubectl port-forward deployment/zipkin 9411:9411
curl http://localhost:9411/api/v2/services
```

---

## Notas

- El servicio `todos` tiene `replicas: 2` para replicar el balanceo Round Robin que tenía HAProxy en la entrega anterior.
- El Ingress reemplaza completamente al HAProxy, enrutando `/login` hacia auth-api, `/todos` hacia todos-api y `/` hacia el frontend.
- Redis y Zipkin usan imágenes oficiales directamente desde Docker Hub sin Dockerfile propio.
- La comunicación entre servicios usa DNS interno de Kubernetes en vez de IPs fijas.



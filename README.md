# Laboratorio de Kubernetes
A continuación, veremos algunos conceptos fundamentales para comenzar a desplegar nuestras aplicaciones sobre Kubernetes, utilizando nuestras imágenes Docker, generadas en clases anteriores.
## Arquitectura Kubernetes

Kubernetes, está compuesto por los siguientes elementos:
![Arquitectura Kubernetes - Microbit](https://microbit.com/wp-content/uploads/2019/08/2.png)

Kubernetes se divide en 2 partes: el master que es quien realiza el procesamiento y da órdenes y por otro lado existen los nodos. El master se comunica con los nodos (máquinas virtuales o físicas donde corren contenedores) para indicarles qué contenedores se van a albergar o qué especificaciones tendrán.

Dentro de la arquitectura de k8s, tenemos los siguientes elementos:
 - Api-server: el cual permite que nos podamos comunicar con k8s a través del cli o por Json.  
 - Kube-schedule: se comunica con el api,  recibe la petición enviada y el schedule revisa donde o en qué nodo colocarlo. En caso no encuentre un nodo disponible, tendrá al contenedor en pendiente, hasta tener el nodo donde dejarlo.
 - Kube-controller: 
	 - Node-controller: controlar la vida de los nodos
	 - Replication-controller: controlar las réplicas de los contenedores
	 - Endpoint-controller: controlar los puntos de acceso a los servicios
	 - Service Account-controller: para temas de autenticación
 - Etcd: es una base de datos key-value donde el cluster almacena el estado, datos, configuraciones del cluster. 
 - Kubelet: servicio corriendo en cada nodo, el cual recibe órdenes del master y enviar información al master.
 - Kube-proxy: encargado de redes en contenedores dentro de k8s.
 -  Container-runtime: herramienta de contenedores, la cual debe tener cada nodo instalado.

## Principales objetos
  
### Pods
Un pod de Kubernetes es un conjunto de uno o varios contenedores y constituye la unidad más pequeña de las aplicaciones de Kubernetes. Puede estar compuesto por un solo contenedor, en un caso de uso común, o por varios con conexión directa, en un caso de uso avanzado. Los contenedores se agrupan en pods para aumentar la eficiencia del uso compartido de los recursos, como se describe a continuación.

Dentro de Kubernetes, los contenedores que se encuentran en el mismo pod comparten los recursos informáticos. Los recursos se agrupan en Kubernetes y forman clústeres, los cuales pueden ofrecer un sistema con más potencia y una distribución inteligente para ejecutar las aplicaciones.

Creación a través de un archivo **pod.yaml**:
```
apiVersion: v1
kind: Pod
metadata:
	name: pod-test
labels:
	app: front
	env: dev
spec:
	containers:
	- name: contenedor1
	  image: nginx:alpine
```
Comando para ejecutar el archivo:
> kubectl apply -f pod.yaml
   
### Replica-set
Es un objeto independiente al pod, pero en un nivel más alto. El Replica-set puede crear pods y sus respectivas réplicas, si en caso un pod creado, muere; el replica-set crea uno nuevo, cumpliendo con la orden que se le dió en el template, en resumen se encarga de mantener los pods que se le fueron solicitados.
El replica-set es el owner de los pods creados. También se usan labels.

Creación a través de un archivo **replica.yaml**:
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: fronted
	labels:
		app: rs-test
spec:
	replicas: 5
	selector:
		matchLabels:
			app: frontend
	template:
		metadata:
			labels:
				app: frontend
		spec:
			containers:
			- name: cont1
			image: nginx:alpine
```

Comando para ejecutar el archivo:
> kubectl apply -f replica.yaml
   

### Deployments
 Es un dueño de un replica-set, está a un nivel más alto y nos ayuda en la actualización de los pods dentro de un réplica-set. Cuando necesita actualizar la versión del replica-set, crea un nuevo replica-set, teniendo en cuenta dos parámetros: el % de indisponibilidad y el % de escalado mientras se dan las actualizaciones.
 Creación a través de un archivo **deployment.yaml**:
```
apiVersion: apps/v1
kind: Deployment
metadata:
	name: dep-test
	annotations:
		kubernetes.io/change-cause: "acutalizacion"
	labels:
		app: frontend
spec:
	replicas: 3
	selector:
		matchLabels:
			app: frontend
	template:
		metadata:
			labels:
				app: frontend
		spec:
			containers:
			- name: nginx
			  image: nginx:alpline
			  ports:
			  - containerPort: 80
```

Comando para ejecutar el archivo:
> kubectl apply -f deployment.yaml

Ver status del deployment
>kubectl rollout status deployment "nombre-deployment"

Ver historial de versiones del despliegue
>kubectl rollout history deployment "nombre-deployment"

Ver una revisión en específico
>kubectl rollout history deployment "nombre-deployment" --revision="# version"

Regresar a una versión
>kubectl rollout undo deployment "nombre-deployment" --to-revision="# version"

### Servicios
Objeto aislado, que observa los pods con cierto label. El servicio dispone una IP, para que cualquier usuario pueda hacer consultas, de tal forma que si un pod muere, el servicio apunta a otro pod disponible. En pocas palabras actúa como balanceador.
Creación a través de un archivo **service.yaml**:
```
apiVersion: apps/v1
kind: Deployment
metadata:
	name: dep-test
	annotations:
		kubernetes.io/change-cause: "actualizacion"
	labels:
		app: frontend
spec:
	replicas: 3
	selector:
		matchLabels:
			app: frontend
	template:
		metadata:
			labels:
				app: frontend
		spec:
			containers:
			- name: nginx
			  image: nginx:alpline
			  ports:
			  - containerPort: 80
	---
apiVersion: v1
kind: Service
metadata:
	name: service-test
spec:
	type: NodePort
	selector:
		app: backend
	ports:
		- protocol: TCP
		port: 8080
		targetPort: 80
```

Comando para ejecutar el archivo:
> kubectl apply -f service.yaml

### Endpoints
Se crea automáticamente cuando se crean los servicios. Cuando un servicio encuentra pods que cumplen con el label que tiene, guarda la ip de los pods, en el endpoint, para saber a quiénes debe consultar.


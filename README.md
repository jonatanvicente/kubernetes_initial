

## k8s INITIAL

### Creación de recursos con descriptores yaml (folder pods)

- <b>pod.yaml</b>
	- Creación simple de un pod (u otro recurso) con <b>kubectl apply -f pod.yaml</b>
	- <b>Borramos</b> con <b>kubectl delete -f pod.yaml</b>
- <b>pod2.yaml</b>
	- Descriptor con dos pods en el mismo yml.
- <b>containers.yaml</b>
	- <b>Creación de un pod con dos contenedores</b>. Demo de qué ocurre cuando colisionamos puertos. Instalación de comando en Alpine.
		- <b>OJO</b> a: no podremos acceder desde un container al otro, pero <b>sí</b> comparten la red (no podrán compartir el mismo puerto).
		- Los containers se ven como localhost
	- Probamos <b>en Docker</b> imagen de python (que tiene server http de ejecución simple)
		- Lo hacemos con <b>docker run --net host -ti python:3.6-alpine sh</b>. Esto ejecuta lo siguiente:
			- Utiliza la imagen indicada (la baja si no está en registry local)
			- La ejecuta en red del host
			- Abre terminal interactivo (ti)...
			- ... con la ejecución del comando sh (shell)
	- Ejecutamos en shell del container python ejecutándose <b>python -m http.server 8081</b>
	- Comprobamos correcta ejecución en localhost:8081
	- Añadimos al yml los command para que cree un file con echo diferente
	- <b>Si ponemos el mismo puerto en ambos containers, da error. kubectl describe doscont</b> no arroja explicación del error
	- <b>kubectl logs doscont -c cont1</b> no da error.
	- <b>kubectl logs doscont -c cont2</b> >> Address in use (si pusimos el mismo puerto)
	- <b>Entramos en container con kubectl exec -ti doscont -c cont1 -- sh</b>
		- El SO Alpine no trae curl. Lo instalamos con <b>apk add -U curl</b> para hacer las pruebas
	- Si, para rehacer los containers (corrigiendo puerto), ejecutamos de nuevo <b>kubectl apply</b> da error: <b>un pod no puede actualizarse a sí mismo</b> (hacen falta objetos de más alto nivel.)	
	- Una vez la definición es correcta, <b>desde <i>dentro</i> de cada container hay acceso a localhost:8082 y 8083</b> con <b>curl localhost:[puerto]</b>. Comparten host, cada container se ve a sí mismo y al otro.
	- <b>Desde fuera, podemos hacer port-forward a los puertos del pod y ver la salida de cada container del pod.</b>
- <b>labels.yaml</b>. Metadata para distinguir pods.
	- Distinguimos con <b>kubectl get pods -l app=backend</b>, p.ej.
	- Se añade el parámetro -l con la label y el valor
	- Siempre deberemos añadir por lo menos el de app para distinguir qué tiene cada pod.


### ReplicaSets (folder replicaSet)

- <b>replicaset.yaml</b>
	- Crea 5 pods de 2 containers cada uno (con réplicas). OJO a:
		- Utilización de apiVersion y labels
		- Se crea con <b>kubectl apply -f [nombreFile].yaml</b>
		- Listamos con kubectl get replicaset o <b>kubectl get rs</b>
	- <b>Si eliminamos a mano un pod con kubectl delete pod [nombrnieCompletoPod] --> ReplicaSet lo vuelve a crear</b>
	- <b>Corregimos las propiedades de replicaset.yaml y fijamos a 2 el numero de replicas.</b>	
	 	- Al ejecutar de nuevo <b>kubectl apply -f replicaset.yaml</b>, el replicaSet <b>elimina</b> los redundantes para dejar en funcionamiento los deseados (2).



### Deployments (folder deployment)

- <b>dep.yaml</b>
	- Creación de Deployment. <b>kubectl apply -f dep.yaml</b>
	- Deployment visible con <b>kubectl get deployment</b>
		- Labels visibles con <b>kubectl get deployment --show-labels</b>
- <b>dep2.yaml</b>
	- Actualización de Deployment sobre dep.yaml
- <b>IMPORTANTE:</b>	 	
	- Podemos ejecutar la actualización con <b>kubectl apply -f dep2.yaml</b>
	- Vemos ejecución de cambios en tiempo real con <b>kubectl rollout status deployment [nombreDeploy]</b>
	- La actualización se efectúa en base a los parámetros fijados (o default, si no) <b>MaxUnavailable</b> y <b>MaxSearch</b>
- <b>dep3.yaml</b>
	- Nuevo cambio para ver histórico de cambios en Deployments. Podemos verificar con <b>kubectl rollout history deployment [nombreDeploy]</b>
	- Volvemos a colocar versión de imagen de dep.yaml.
	- Ejecutamos con <b>kubectl apply -f dep3.yaml</b>
- <b>dep4.yaml</b>
	- Inclusión de parámetro revisionHistoryLimit a 1 para modificación del máximo de registros de histórico (por default es 10).
	- Eliminamos Deployment con <b>kubectl apply -f dep4.yaml</b>


##### Change Cause (dep5.yaml)

- Elemento que aparece al mostrar el history en Deployments con <b>kubectl rollout history deployment [deploymentName]</b>
- Contiene metadata para añadir CHANGE-CAUSE (kubernetes.io/change-cause)	


#### Rollback de un despliegue (dep6.yaml)

- File incorrecto (que apunta a una imagen que no existe) para forzar un rollback.
- Ejecutamos <b>kubectl apply -f dep6.yaml </b> (forzado error)
- <b>kubectl get pods<b> muestra el error
- <b>kubectl rollout history deployment deployment-test</b> nos indica el historial (para hacer el rollback)
	

### Services (folder service)

- Ver file svc.yaml
- Podemos añadir en el file distintos tipos de resources que se crearán de manera secuencial
- Mapeo de puertos y secuencia de entrada (file svc.yaml):
	- IP del service: Kubernetes garantizará la IP del service (que no cambia)
	- El port del service --> spec > ports > port
	- Ports de los pods adonde redirige el Service --> spec > ports > targetPort
- File svc2.yaml: Inclusión de tipo de cluster en el Service. Delete service anterior y aplicación del nuevo:
	- kubectl delete -f svc.yaml
	- kubectl apply -f svc2.yaml
	


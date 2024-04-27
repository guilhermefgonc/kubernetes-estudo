# Estudo sobre Kubernetes
Anotações realizadas sobre Kubernetes durante meu estudo com o curso de Orquestração de Containers com Kubernetes

## **Introdução à Kubernetes**

- K8s é uma ferramenta usada para simplificar, automatizar e escalar o gerenciamento de containers

### **Arquitetura do K8s**

**Node**

- É uma máquina onde o K8s está instalado
- No Node criamos containers com nossas aplicações

**Cluster**

- É um conjunto de nodes agrupados
- Dessa forma, mesmo que um Node falhe, a aplicação continua sendo acessada de outros Nodes

**Master**

- Quando temos um Cluster, é necessário um Master (Node Master) para gerenciar, monitorar e realizar funções dentro do Cluster

![k8s_1.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_1.png)

### **Componentes do Kubernetes**

- **API Server**: Através desse componente conseguimos fazer o uso do gerenciamento de usuários, dispositivos e interface de linha de comando. Funciona como front-end para o K8s
- **etcd**: Usado para armazenar dados no formato chave/valor para configuração e gerenciamento dos Clusters
- **Scheduler**: Responsável por distribuir o trabalho para os containers através de múltiplos Nodes.
- **Controller**: Responsável por tomar as decisões quando um Node falha, pode criar novos Nodes para substituir os que apresentam problemas
- **Container Runtime**: Faz interface com o software usado para criar os containers, por exemplo (Docker Engine)
- **kubelet**: Responsável por checar se os containers estão sendo executados no Node conforme o esperado

### **Master vs Worker Nodes**

![k8s_2.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_2.png)

---

## **Ambiente**

- Existem várias distribuições do K8s, nesse curso foi utilizada o Minikube
- O Minikube agrupa todos os componentes do K8s em um Node, sendo mais simples e leve

---

## **Conceitos Importantes do Kubernetes**

### **POD**

- POD é a menor unidade de computação publicável em um computador que podemos criar e gerenciar
- Quando criamos uma aplicação em um container, o K8s não faz acesso direto ao container, esses containers são encapsulados em um POD
- Geralmente, a relação POD/Container é 1:1
- Para escalar up, é criado um novo POD que irá instanciar um container idêntico ao já existente

![k8s_3.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_3.png)


```txt
kubectl run [pod_name] --image [image] # Cria um POD no cluster (minikube start) através da imagem
kubectl get pods # Acessa as informações dos PODs
```

### **Arquivos de Definição do K8s**

- São escritos em YAML
- Possuem sempre 4 top level: apiVersion, kind, metadata, spec

```txt
apiVersion: v1 # Definição da versão do objeto que estamos criando
kind: Pod # Define o tipo de objeto que estamos criando (Pod, Service, ReplicaSet, Deployment)
metadata: # dados que pertecem ao objeto criado
	name: nome-do-pod
	labels:
		app: nome-da-app
spec: # campo referenete as especificações do objeto criado
	containers:
	- name: nome-container
		image: imagem
```

- Com o arquivo definido, podemos criar o objeto com o comando:

```txt
kubectl create -f [nome-do-arquivo-yaml]
```

### **Replication Controller e ReplicaSets**

**Replication Controller**

- Uma das ferramentas do Controller Manager é o Replication Controller
- Replication Controller nos ajuda a executar múltiplas instâncias de um POD no nosso cluster. Ele faz isso replicando (duplicando) o POD existente, para que tenhamos uma cópia exata
- Caso um POD falhe, uma réplica é criada automaticamente
- O Replication Controller se certifica que o número especificado de PODs esteja sendo executado
    
![k8s_4.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_4.png)    

**Balanceamento de Carga e Escalamento de Aplicação**

- O Balanceamento de carga é a distribuição igualitária do tráfego da aplicação entre os recurso para não sobrecarregar um serviço
- O Replication Controller pode criar novas réplicas em outros Nodes do Cluster para o Balanceamento de Carga

![k8s_5.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_5.png)


**ReplicaSets**

- ReplicaSets está substituindo o Replication Controller
- Possuem o mesmo propósito, mas não são a mesma coisa. ReplicaSets possuem mais funções

**YAML**

Arquivo de Definição para as ReplicationController

```txt
apiVersion: v1
kind: ReplicationController
metadata:
	name: aplicacao-rc
	labels:
		app: applicacao
		type: frontend

spec:
	template: # modelo a ser replicado (todo conteúdo do POD, excluindo as duas primeiras linhas)
		metadata:
			name: aplicacao-pod


			label:
				app: aplicacao
				type: frontend
			spec:
				containers:
				- name: nginx-container
					image: nginx
	replicas: 2 # número de replicas que devem estar sendo executadas simultaneamente
```

- O comando de criação do Replica Controller é o mesmo para os PODs

```txt
kubectl create -f [nome-arquivo-yaml]
kubectl get replicationcontroller
```

- Ao executar o comando, o RC irá criar os PODs especificados na quantidade de réplicas que devem existir

Arquivo de Definição para ReplicaSets

```txt
apiVersion: apps/v1 # deve ser especificado como /apps
kind: ReplicaSets # nome do objeto
metadata:
	name: aplicacao-rc
	labels:
		app: applicacao
		type: frontend

spec:
	template: # modelo a ser replicado (todo conteúdo do POD, excluindo as duas primeiras linhas)
		metadata:
			name: aplicacao-pod
			label:
				app: aplicacao
				type: frontend
			spec:
				containers:
				- name: nginx-container
					image: nginx
	replicas: 2 # número de replicas que devem estar sendo executadas simultaneamente

	# indica entre os PODs existentes, qual deve ser usado para ser replicado no cluster
	selectior:
		matchLabels:
			type: frontend
```

**Labels e Selectors**

- O RC/RS estão monitorando os PODs existentes no cluster para verificar se estão sendo executados corretamente, é os Labels são necessários para identificar quais objetos precisam ser monitorados.

**Escalando a Aplicação**

```txt
kubectl scale --replicas=[numero] -f [arquivo-rs-yaml]
```

### **Deployments**

- Para atualizar uma versão e disponibilizar a nova versão, é interessante atualizar POD por POD. Isso é conhecido como rolling release
- Se um dos PODs apresentou um problema, e você deseja reverter a atualização, execute um rollback
- Para fazer uma atualização mais complexa, não apenas publicando uma nova versão, mas também atualizando a versão dos containers ou do K8s é interessante fazer uma “manutenção programada”, onde se pausa o ambiente, atualiza e depois reinicia
- O Deployment encapsula tudo que vimos antes

![k8s_6.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_6.png)


Definição do Deployment

```txt
apiVersion: apps/v1 # deve ser especificado como /apps
kind: Deployment # nome do objeto
metadata:
	name: frontend-dp
	labels:
		app: applicacao
		type: frontend

spec:
	template: # modelo a ser replicado (todo conteúdo do POD, excluindo as duas primeiras linhas)
		metadata:
			name: frontend-pod
			label:
				app: frontend-app
				type: frontend
			spec:
				containers:
				- name: nginx-container
					image: nginx

	# indica entre os PODs existentes, qual deve ser usado para ser replicado no cluster
	selector:
		matchLabels:
			type: frontend
	replicas: 2 # número de replicas que devem estar sendo executadas simultaneamente
```

**Atualizando e Desfazendo Deployments**

- Quando fazemos o deploy (publicação) com o Deployments, é disparado um recurso chamado “rollout”
- A cada execução do rollout, é criada uma revisão da publicação
- Esse processo de revisão, ajuda o K8s a manter o rastro das publicações da aplicação, fazendo com que seja fácil de realizar um rollback em caso de problemas

![k8s_7.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_7.png)


```txt
kubectl rollout status deployment/[nome-dp] #checar o status do rollout
kubectl rollout history deployment/[nome-dp] # consultar o histórico do rollout
kubectl apply -f deployment/[nome-dp] # aplica a atualização
```

**Estratégias de Deployment**

- Recreate: Todas as instâncias são desligadas, e depois a nova versão é upada
- Rolling Update: Atualiza uma instância de cada vez

---

## **Funcionamento de Redes no Kubernetes**

- O Cluster nasce com 2 endereços IP, um do Cluster e um da apiServer (usado para acessar o Cluster via SSH)
- Diferentemente do Docker, em Kubernetes, quem recebe o IP é o POD e não o container
- Também é criada uma rede que irá endereçar os PODs na medida em que são criados
- KubeDNS faz toda a tradução de nomes para endereços IP

![k8s_8.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_8.png)


### **Namespaces**

- É um “espaço” para separar os recursos em áreas separadas
- Podemos então agrupar PODs que fazem parte de uma mesma área em um Namespace

![k8s_9.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_9.png)


- Ao criar um objeto no K8s, se não for especificado o Namespace, ele será colocado no default do K8s

```txt
kubectl create namespace [name] --savec-congig
kubectl get ns
kubectl create -f pods/nginx.yaml --save-config --namespace=[namespace]
```

---

# Serviços com Kubernetes

- Services em K8s permite a comunicação entre os diversos componentes da aplicação com o mundo externo à ela.
- Permite a conexão fácil entre uma aplicação e outra, e com os usuários externos


- Os Services permitem a criação de microserviços para uma aplicação
- Os Services habilitam uma porta de acesso para acessar o serviço disponível

![k8s_10.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_10.png)


![k8s_11.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_11.png)


### **NodePort**

Tipos de Service

![k8s_12.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_12.png)


- NodePort abre uma comunicação entre o POD e agentes externos, como navegadores web, se o serviço for disponibilizado para web
- ClusterIP é usado quando a comunicação entre os serviços é apenas interna (privada) ao Cluster, sem acesso à agentes externos
- LoadBalancer habilita a distribuição de carga entre os diverso PODs que estão realizando o serviço.

```txt
apiVersion: v1
kind: Service 
metadata:
	name: frontend-svc
spec:
	selector:
		type: frontend
	ports:
		- name: http
			targetPort: 80 # Porta do servidor web
			port: 80 # porta que faz comunicação com o POD
	type: (NodePort, ClusterIP ou LoadBalancer)

```

---

# Microserviços

- Microserviços é uma arquitetura na qual uma aplicação é dividida em pequenos serviços independentes
- Cada microserviço se comunica, geralmente, com protocolos HTTP/REST, criando uma grande rede de APIs

**Vantagens**

- Escalabilidade horizontal (é quando replicamos a mesma máquina X vezes)
- Facilidade de desenvolvimento e manutenção
- Tolerância a falhas
- Melhor utilização dos recursos

**Kubernetes e Microserviços**

- K8s é ideal para os microserviços pois é totalmente em APIs

**Aplicação de Microserviços criada**

![k8s_13.png](https://github.com/guilhermefgonc/kubernetes-estudo/blob/main/imagens/k8s_13.png)

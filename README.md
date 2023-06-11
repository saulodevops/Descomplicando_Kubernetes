# Descomplicando_Kubernetes
Repositório utilizado para realização do curso Descomplicando Kubernetes e do Mutirão Kubernetes


## Conteúdo atualizado e prático

>O que é o Kubernetes?
Antes de falar o que é, é importante saber o que é o container: isolamento de recursos.

>Como assim isolamento de recursos?
Imagine que você tem a sua máquina -> ela possui recursos computacionais (cpu, memoria, disco). Quando estamos falando de container, é como se pegasse um pedacinho desses recursos e isolasse, isso é o container.

### Importante! O kernel, responsável por falar com o hardware. Quando você tem um container, ele utiliza o kernel do host. Com Docker, os containers ficaram populares.

### Leia o livro do descomplicando o kubernetes para entender melhor.

>Aonde entra o kubernetes nessa história?
Ele não é um concorrente do Docker!

>O que é um orquestrador de containers?
Quando você precisa ter réplicas do isolamento de recursos, de forma que o seu serviço funcione bem, ou seja, não dependa somente de um container apenas, você pode orquestrar, gerenciar, esses containers. 

Quando você ouve falar sobre nós (PODs) de clusters, você pode entender como instâncias.

O orquestrador vai colocar ordem na casa, ou seja, ele pode dizer se você tem um volume, um disco, para os seus PODs.

Kubernetes é o orquestrador mais popular do mundo. Além de ser um orquestrador de containers, foi iniciado pela Google. Em 2014 o Google tornou o kubernetes público, criou uma fundação para cuidar do kubernetes, o Google doou para a Linux Foundation.

Quando você utiliza o kubernetes, você utiliza um software aberto, não tem dono, é da comunidade, open source.

O kubernetes é o responsável por colocar o DNS nos PODs, configurar regras de comunicação entre serviços.

Cada Node tem o seu papel no cluster, quando você fala sobre o cluster inteiro, parte dos nós são para gerenciamento do cluster, para saber se a saúde do cluster está ok, ver quantidade de réplicas, se está tudo belezinha: esses nós são chamados de control plane, como se fosse o cérebro do cluster. É lá que está rodando a API do kubernetes, onde está rodando o APCD, o banco chave valor que guarda o estado do cluster e inúmeros outros controladores para saber saúde do cluster.

Por default, você não executa containers no control plane, você executa nos workers.

Quando você vai mexer com kubernetes, você pode ter mil possibilidades de cluster.

Exemplos:

1. Kubernetes gerenciado (EKS, AKS, GKE): o control plane, o cérebro, a Cloud gerencia (resiliência, itcd). Você só se preocupa com os workers. Instalação pode ser feita via interface, terraform ou via linha de comando do cloud provider.

2. Kubernetes vanilla (kubernetes original): para que você consiga instalar ele em cluster, você vai utilizar a ferramenta kubeadm para criar o cluster. Via repositório do kubernetes

3. Distribuições do kubernetes como o OKD (RedHat), kubernetes com algumas modificações.

4. Kubernetes para desenvolvimento local (teste), os mais conhecidos são minikube (em cima de VMs) e kind (em cima do Docker) -> Você consegue criar um cluster que simula 3 nós, fazer vários testes.

Comando para instalação do kind (precisa ter docker instalado):

```
sudo curl -Lo /usr/local/bin/kind https://github.com/kubernetes-sigs/kind/releases/latest/download/kind-linux-amd64
sudo chmod +x /usr/local/bin/kind
```

Laboratório dia 01:

Plataforma: instruqt

Repositório: https://github.com/badtuxx/CertifiedContainersExpert/tree/main/DescomplicandoKubernetes/day-1#kind

Existe um comando muito importante: kubectl (kubernetes control - responsável por administrar o cluster): É com ele que você vai fazer a interface com o cluster.

Se eu quero criar um POD ou fazer um deployment, preciso utilizar o comando kubectl. Existem duas maneiras para criar coisas com o kubernetes, mas ambas passam pelo kubectl.


Para instalar no linux:

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version --client
```

curl (utilizado para fazer download de conteúdo, ao acessar um determinado endereço)

chmod +x (permissão de execução)

mv (mover parao usr/local/bin, que é onde ficam os binários do sistema)

Quando o cluster estiver iniciado, quem vai se comunicar com a API do kubernetes é o kubectl (quero criar uma coisa nova, quero remover, etc.)

É legal utilizar um alias (k - alias mais utilizado) e o autocomplete do kubectl:

```
source <(kubectl completion bash) # configura o autocomplete na sua sessão atual (antes, certifique-se de ter instalado o pacote bash-completion).

echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanentemente ao seu shell.

alias k=kubectl

complete -F __start_kubectl k
```

Vamos executar o cluster com o Kind (kubernetes in Docker)

Caso queira saber mais detalhes sobre o kind, acesse a página e veja como funciona.

>Como criar um cluster?

kind create cluster --name giropops

Para instalar o Docker: curl -fsSL https://get.docker.com | bash

Para que o usuário tenha permissão para executar comandos docker, ele precisa fazer parte do grupo docker, para isso, utilize o comando:

sudo usermod -aG docker NOMEDOUSUARIO

Com o cluster ativo, você pode ver o cluster com o comando 

kind get clusters

k get nodes (mostra os nós)

k get pods -A (mostra os PODs)

POD é a menor unidade do kubernetes (normalmente 01 container, mas está cada vez mais comum ter mais de um container em um POD)

o Kubernetes não trata containers, trata PODs. Todos os containers dentro de um POD compartilha as mesmas características de isolamento (mesmo endereço, por exemplo)

Namespace é uma forma de agrupar recursos no kubernetes. kube-system é o namespace onde se encontram os serviços do control plane, da gestão do kubernetes (etcd, API server, controllers, shceduler)

Para deletar um cluster: 

kind delete clusters giropops

Feito isso, podemos criar um arquivo 

vim meu-primeiro-cluster.yaml

conteúdo:

kind: Cluster
apiVersion: kind.x-k8s.io/v1aplha4
nodes:
    - role: control-plane
    - role: worker
    - role: worker

Para salvar, aperte esc e digite :wq + enter

Desta forma, criamos um cluster com 3 nós (1 control plane e 2 workers)

agora vamos utilizar o comando kind create cluster, porém, nomeá-lo multinode

kind create cluster --name giropops-multinode --config meu-primeiro-cluster.yaml

Se rodarmos agora: k get nodes, veremos agora 03 nós

Agora posso colocar coisas para rodar no cluster da maneira que eu bem entender.

k get namespaces retorna os namespaces ativos

k get pods -A -o wide traz muitas informações sobre os PODs, onde ele está rodando, etc.

Posso rodar um POD do nginx

k run nginx --image nginx cria um POD do nginx

k describe pods nginx retorna detalhes sobre o POD do nginx

vou executar um comando no meu pod do nginx para startar um shell dentro do container:

k exec -ti nginx -- bash

-ti: interação via terminal

Desta forma eu caio no terminal do nginx. Posso por exemplo executar um cd /proc seguido de um ls para ver os processos em execução no container do webserver. Posso também executar o comando cat 1/cmdline para ver de que forma o nginx está em execução (master process)

Para acessar o diretório onde se encontra o arquivo index.html no nginx, utilizamos o comando: cd /usr/share/nginx/html/

Ao utilizar o comando echo GIROPOPS > index.html, substituimos o conteúdo do intex.html por GIROPOPS 

Agora, se eu executar curl localhost, terei como resposta GIROPOPS, pois o nginx está rodando no localhost dentro do container.

Para sair do container basta digitar exit + enter

Tenho a possibilidade de expor o serviço, ou seja, crio um service para que outras pessoas consigam acessar meu cluster.

Agora faça o desafio do dia 01 do treinamento (duas horas de conteúdo):

https://www.linuxtips.io/course/descomplicando-o-kubernetes


# kubeflow-pipelines-hello-world
Este repositório documenta um pipeline hello world para teste do Kubeflow. Executando-se o script `main.py`, gera-se uma especificação de pipeline `pipeline.yaml`, que pode ser enviada para um backend de Kubeflow e executada nele em uma run.

O pipeline de exemplo é composto por um único componente `say_hello(name)`, que, munido de uma string `name`, imprime a string `Hello, <name>!`, conforme a implementação abaixo.

```python
@dsl.component
def say_hello(name: str) -> str:
    hello_text = f'Hello, {name}!'
    print(hello_text)
    return hello_text
```

## Intalação minikube
Instale o minikube

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

Inicie o minikube, especificando quantidades de memória e CPUs suficientes para processar o pipeline.

```bash
minikube start --memory=4096 --cpus=4
```

Veifique se ele esta rodando mesmo com o seguinte comando

```bash
minikube status
```

Ele deve retornar algo como:

```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

## Acessar dashboard minikube 
Nós queremos acessar a dashboard para termos um controle melhor sobre nossos pods

Verificar se o dashboard esta ativo

```bash
minikube addons list
```

Ele retorna uma lista com as funcionalidades do minikube. Tente achar algo relacionado ao **Dashboard**, provavelmente ele não esta habilitado

rode este comando para habilitar
```bash
minikube addons enable dashboard 
```

agora rode este para acessar
```bash
minikube dashboard 
```
Vai gerar um link. Entre neste link, ele vai abrir um dashboard na web que você pode controlar seus clusters e namespaces e verificar a saúde dos seus pods.

## Criar namespace kubeflow
Para criar o namespace precisamos abrir outro terminal e entrar na documentação do KubeFlow. [Kubeflow Documentation](https://www.kubeflow.org/docs/components/pipelines/operator-guides/installation/)

Vamos primeiro copiar e colar está linha no terminal
```bash
export PIPELINE_VERSION=2.14.3
```

depois rodamos estes comandos
```bash
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/cluster-scoped-resources?ref=$PIPELINE_VERSION"
kubectl wait --for condition=established --timeout=60s crd/applications.app.k8s.io
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/dev?ref=$PIPELINE_VERSION"
```

Agora vamos no dashboard criado anteriormente e verificar se apareceu um namespace chamado `kubeflow` </br>
[imagem]

Espere os pods ficarem TODOS verdes.

## Acessar UI Kubeflow Pipelines
Para acessar a UI digite o seguinte comando

```bash
kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80
```

Acesse o frontend pelo seu navegador, pelo link [http://localhost:8080](http://localhost:8080)

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 13-56-52.png'></img>
</p>


## Compilação do Pipeline
Clone esse repositório

```bash
git clone https://github.com/henriqueedu2001/kubeflow-pipelines-hello-world
```

Entre do diretório do projeto
```bash
cd kubeflow-pipelines-hello-world
```

Crie um ambiente virtual isolado no python.
```bash
python3 -m venv env
source env/bin/activate
```

Para compilar o pipeline, execute o script `main.py`.

```bash
python3 main.py
```

Ao final desse processo, terás como resultado o arquivo `pipeline.yaml`, que é a especificação do pipeline compilado, pronto para ser executado por um backend com Kubeflow.

## Execução do pipeline

Faça o upload do pipeline via botão no canto superior direito. Você será direcionado para uma página em que deves fazer upload do pipeline, especificar metadados e determinar parâmetros relativos a ele.

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 14-01-42.png'></img>
</p>

Após criar o pipeline, serás redirecionado para a página específica desse pipeline, em que se pode interagir com o grafo da run. Para executá-lo de fato utilize o botão `create run`.

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 15-18-39.png'></img>
</p>

Após isso, estarás na página de criação de runs. Nomeie sua run.

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 15-18-51.png'></img>
</p>

Ao final da página, insira a string de input da função `say_hello(name)`. Para este exemplo, uso meu nome "Henrique S. Souza"

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 15-19-03.png'></img>
</p>

Finalmente, a run é criada e executada. Podes visualizar a execução do componente `say_hello()`, tanto sua entrada ("Henrique S. Souza") quanto sua saída ("Hello, Henrique S. Souza!").

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 15-23-56.png'></img>
</p>

O pipeline e as runs estarão visíveis nas páginas anteriores.

## Acesso remoto
Acesse o servidor remoto, criando um túnel via SSH.

```bash
ssh -p2238 -L 8080:localhost:8080 username@ip
```

Dentro do servidor, faça o port forwarding.

```bash
kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80
```

Agora, você poderá acessar a UI pelo link [http://localhost:8080](http://localhost:8080)

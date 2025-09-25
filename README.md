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

Em seguida, instale as dependências; no caso, o Kubeflow Pipelines.

```bash
pip install kfp
```

Para compilar o pipeline, execute o script `main.py`.

```bash
python3 main.py
```

Ao final desse processo, terás como resultado o arquivo `pipeline.yaml`, que é a especificação do pipeline compilado, pronto para ser executado por um backend com Kubeflow.

## Execução do pipeline
Você deve ter o minikube e o kubectl instalados em sua máquina local. Para instalar o minikube, baixe os binários com curl.

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
```

Em seguida, instale-o.

```bash
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

Podes verificar se a instalação foi bem sucedida com o comando a seguir.

```bash
minikube version
```

Em seguida, baixe os binários do kubectl.

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

E instale-os.

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Podes também verificar se a instalação do kubectl foi bem sucedida com o comando a seguir.

```bash
kubectl version
```

Inicie o minikube, especificando quantidades de memória e CPUs suficientes para processar o pipeline.

```bash
minikube start --memory=4096 --cpus=4
```

Caso não exista, crie um namespace `kubeflow` com kubectl

```bash
kubectl create namespace kubeflow
```

Instale o Kubeflow Pipelines do kubeflow via kubectl

```bash
kubectl apply -k "github.com/kubeflow/pipelines/manifests/kustomize/env/platform-agnostic?ref=2.0.0"
```

Com isso, o minkube iniciará um cluster com diversos pods do Kubeflow Pipelines. Você deve aguardar até todos os pods alcançarem o estado `running`. Para visualizar os status dos pods, use o comando abaixo.

```bash
kubectl get pods -n kubeflow
```

O comando mostrará todos os pods do namespace `kubeflow`, conforme a figura abaixo.

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 13-38-49.png'></img>
</p>

Após todos os pods estarem rodando, você deve ter o backend completo do Kubeflow Pipelines rodando localmente eu sua máquina.

Como estamos apenas rodando o Kubeflow Pipelines, não o Kubeflow completo, precisamos instalar o Argo Workflows. Crie um namespace próprio para o Argo.

```bash
kubectl create namespace argo
```

Em seguida, instale o Argo no minikube.

```bash
kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/stable/manifests/install.yaml
```

Verifique se os pods estão rodando.

```bash
kubectl get pods -n argo
```

Agora, faça forwarding de porta, para tornar o frontend da aplicação acessível ao seu navegador.

```bash
kubectl port-forward -n kubeflow svc/ml-pipeline-ui 8080:80
```

Acesse o frontend pelo seu navegador, pelo link [http://localhost:8080](http://localhost:8080)

<p style='text-align: center;'>
    <img src='docs/Screenshot from 2025-09-12 13-56-52.png'></img>
</p>

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
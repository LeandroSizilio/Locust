# Atividade Oficina - Testes Requisitos Qualitativos
## Tutorial de Testes de Carga com Locust 

Este tutorial apresenta um guia passo a passo para instalar e utilizar o **Locust**, uma ferramenta open-source para testes de carga e performance escrita em Python. Com o Locust, você pode simular o comportamento de milhares de usuários em sua aplicação para identificar gargalos e garantir a estabilidade do sistema.

**[Tutorial de Testes de Carga com Locust](https://youtu.be/Qmk_f8riB5E)**

## 📋 Pré-requisitos

Antes de começar, garanta que você tenha os seguintes itens instalados:

- **Python 3.6+**: O Locust é uma ferramenta baseada em Python.
- **PIP**: O gerenciador de pacotes do Python (geralmente instalado com o Python).
- **Um editor de código**: VS Code, PyCharm, Sublime Text, etc.
- **Um terminal**: Prompt de Comando, PowerShell, ou qualquer terminal do seu sistema operacional.

Para verificar sua versão do Python, execute no terminal:

```bash
python --version
```

## 🚀 Instalação

Para manter o projeto organizado e evitar conflitos de dependência, vamos utilizar um ambiente virtual (`venv`).

**1. Crie e acesse a pasta do projeto:**

mkdir teste
cd teste

**2. Crie o ambiente virtual:**

```bash
python -m venv venv
```

**3. Ative o ambiente virtual:**

  - **Windows (PowerShell/CMD):**
    ```bash
    .\venv\Scripts\activate
    ```
  - **macOS / Linux:**
    ```bash
    source venv/bin/activate
    ```
    > Após a ativação, o nome `(venv)` aparecerá no início da linha do seu terminal.

**4. Instale o Locust:**
Com o ambiente ativo, instale o Locust com um único comando:

```bash
pip install locust
```

## ✍️ Criando o Primeiro Teste

O Locust define o comportamento dos usuários em um arquivo Python, geralmente chamado `locustfile.py`.

Crie um arquivo chamado `locustfile.py` e adicione o seguinte código igual ao usado no video tutorial:

```python
import random
from locust import HttpUser, task, between

class BlogUser(HttpUser):

    #Exemplo com usuário virtual que navega em um blog.
    
    # Tempo de espera entre chamadas a um endpoint.
    wait_time = between(1, 3)
    
    # O host alvo dos testes
    host = "https://jsonplaceholder.typicode.com"

    def on_start(self):
        
        #Este método é executado quando cada usuário virtual que é iniciado.
        print("Iniciando um novo usuário virtual para o blog...")

    @task(3)  # Esta tarefa tem peso 3, sendo 3x mais provável de ser executada.
    def listar_todos_posts(self):
        
        #Simula um usuário visualizando a página principal do blog, que lista todos os posts.
        self.client.get(
            "/posts",
            name="/posts (lista todos)"  # Nome que aparece no relatório.
        )
        print("Visualizando todos os posts...")

    @task(1)  # Esta tarefa tem peso 1.
    def ver_post_especifico(self):

        #Simula um usuário clicando em um post específico para ler.
        
        # Escolhe um ID de post aleatório entre 1 e 100 para simular que o usuário está acessando diferentes posts.
        post_id = random.randint(1, 100)
        
        self.client.get(
            f"/posts/{post_id}",
            name="/posts/[id]"  # Agrupa posts diferentes (como /posts/1, /posts/5, etc.) em uma única entrada.
        )
        print(f"Visualizando post {post_id}...")
```

#### Componentes Principais:

  - **`HttpUser`**: Classe que representa um usuário que fará requisições HTTP.
  - **`wait_time`**: Simula o tempo que um usuário "pensa" entre as ações. Essencial para testes realistas.
  - **`@task`**: Decorador que marca uma função como uma tarefa a ser executada pelo usuário virtual que se repete a cada execução.

## ⚡ Executando o Teste

**1. Inicie o Locust:**
No terminal (com o `venv` ativo), execute:

```bash
locust -f locustfile.py
```
> o nome do arquivo `locustfile.py` pode ser diferente, basta substituir o nome no comando acima.

**2. Acesse a Interface Web:**
Abra seu navegador e acesse o endereço: `http://localhost:8089`

**3. Configure a Simulação:**
Na interface web, preencha os seguintes campos:

  - **Number of users**: O número total de usuários simultâneos a simular (ex: `10`).
  - **Spawn rate**: Quantos usuários por segundo serão iniciados até atingir o total (ex: `2`).
  - **Host**: O endereço do site a ser testado (ex: `https://jsonplaceholder.typicode.com`).

Clique em **"Start swarming"** para iniciar o teste. Você poderá acompanhar as estatísticas, gráficos e falhas em tempo real pelas abas da interface.

## 📈 Exemplo utilizado no nosso PDS

No nosso PDS, o teste é um cadastro de usuário profissional e o seu login.

Agora o meu `locustfile.py` é o seguinte:

```python
from locust import HttpUser, task, between
from faker import Faker
import random
import re

# Inicializa o Faker para gerar dados em português do Brasil
fake = Faker('pt_BR')

class CadastroLoginUser(HttpUser):
    
    wait_time = between(1, 3)
    host = "http://127.0.0.1:8000"  #  API


    @task
    def fluxo_cadastro_e_login(self):
        """
        Uma única tarefa que engloba o cadastro de um novo profissional
        e o login subsequente para validar o processo completo.
        """
        # --- 1. Preparar dados para um novo profissional ---
        # Gera dados únicos para cada execução da tarefa para evitar conflitos
        username = fake.user_name() + str(random.randint(1000, 9999))
        password = fake.password(length=12)
        
        cpf_formatado = fake.cpf()
        cpf_limpo = re.sub(r'\D', '', cpf_formatado) # Remove pontos e traços do CPF
        
        telefone_formatado = fake.phone_number()
        telefone_limpo = re.sub(r'\D', '', telefone_formatado) # Remove caracteres não numéricos do telefone
        
        # Gera um número de conselho e garante que não ultrapasse o limite do campo
        conselho_gerado = str(random.randint(100000, 999999))
        conselho_limitado = conselho_gerado[:20]

        cadastro_data = {
            "nome_completo": fake.name(),
            "conselho": conselho_limitado,
            "cpf": cpf_limpo,
            "contato": telefone_limpo,
            "user": {
                "username": username,
                "email": fake.email(),
                "perfil": "pro",
                "password": password
            }
        }
        
        # --- 2. Executar o POST de Cadastro ---
        with self.client.post(
            "/api/profissional/",
            name="/api/profissional/ [cadastro]",
            json=cadastro_data,
            catch_response=True
        ) as response:
            if response.status_code == 201:
                response.success()
                print(f"Cadastro do profissional {username} com sucesso!")
            else:
                response.failure(f"Falha ao cadastrar. Status: {response.status_code}, Resposta: {response.text}")
                print(f"Falha ao cadastrar o profissional {username}")
                return # Se o cadastro falhar, interrompe a tarefa

        # --- 3. Executar o POST de Login ---
        with self.client.post(
            "/api/login/",
            name="/api/login/",
            json={"username": username, "password": password},
            catch_response=True
        ) as login_response:
            if login_response.status_code == 200 and "token" in login_response.json():
                login_response.success()
                print(f"Login do profissional {username} com sucesso!")
            else:
                login_response.failure(f"Falha ao logar. Status: {login_response.status_code}, Resposta: {login_response.text}")
                print(f"Falha ao logar o profissional {username}")
```
> No codigo acima é utilizado o `Faker` para gerar dados aleatórios em português do Brasil. O `Faker` é uma biblioteca Python que gera dados aleatórios de maneira consistente e útil. Você pode instalar o `Faker` com o comando `pip install faker`.

#### Novos Conceitos:

- **Fluxo Completo em uma Única Tarefa:** Em vez de usar on_start para o setup, este modelo testa a jornada inteira (cadastro + login) como uma única tarefa (@task). Cada "usuário" virtual repetirá este fluxo completo, o que é excelente para estressar os endpoints de registro e login.

- **Geração de Dados Dinâmicos por Tarefa:** Os dados (username, CPF, etc.) são gerados com o Faker a cada execução da tarefa. Isso é crucial para garantir que cada tentativa de cadastro seja única, evitando erros de duplicação no banco de dados.

- **Requisições em Cadeia:** O script demonstra como encadear requisições de forma lógica. A requisição de login só é executada se a requisição de cadastro for bem-sucedida (status 201). O return dentro do else interrompe a execução da tarefa se o passo inicial falhar.

## 📚 Próximos Passos

Este tutorial cobriu o básico para começar. O Locust é uma ferramenta poderosa e flexível. Para cenários mais avançados, como manipulação de tokens de autenticação, fluxos de usuário complexos e testes distribuídos, a **[documentação oficial do Locust](https://docs.locust.io/)** é o melhor recurso.

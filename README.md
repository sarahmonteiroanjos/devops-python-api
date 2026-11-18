# API DevOps de Tarefas (`devops-python-api`)

API REST desenvolvida em Python com FastAPI para gerenciamento didático de tarefas, estruturada com esteira completa de Integração Contínua (CI) e Entrega Contínua (CD) utilizando GitHub Actions, Docker, Amazon ECR e Amazon EC2.

---

## 1. Descrição e Objetivo
O projeto tem como objetivo automatizar o ciclo de vida de entrega de uma API Python em nuvem. Toda alteração submetida via Pull Request passa por validação de qualidade de código, testes automatizados e teste de contêiner antes de ser integrada à branch `main`. Após o merge na `main`, a pipeline realiza automaticamente o build da imagem Docker, publica no Amazon ECR com a tag do commit e atualiza o contêiner em produção na instância Amazon EC2 sem intervenção manual.

---

## 2. Tecnologias Utilizadas
- **Linguagem e Framework:** Python 3.11, FastAPI, Uvicorn
- **Qualidade e Testes:** Pytest, Ruff (linter), Coverage
- **Containerização:** Docker
- **CI/CD:** GitHub Actions
- **Infraestrutura Cloud (AWS):** Amazon Elastic Container Registry (ECR), Amazon Elastic Compute Cloud (EC2) e IAM Roles

---

## 3. Execução Local

1. Clone o repositório e acesse a pasta do projeto:
   ```bash
   git clone [https://github.com/sarahmonteiroanjos/devops-python-api.git](https://github.com/sarahmonteiroanjos/devops-python-api.git)
   cd devops-python-api
Crie e ative um ambiente virtual Python:

Bash
python -m venv .venv
# Linux/macOS:
source .venv/bin/activate
# Windows (PowerShell):
.venv\Scripts\Activate.ps1
Instale as dependências:

Bash
pip install -r requirements.txt
Execute os testes automatizados:

Bash
pytest -v
Inicie a aplicação localmente:

Bash
uvicorn app.main:app --host 0.0.0.0 --port 8080 --reload
A API estará disponível em http://localhost:8080/health (e a documentação interativa em http://localhost:8080/docs).

4. Construção e Execução com Docker
Construa a imagem Docker localmente:

Bash
docker build -t devops-python-api:local .
Execute o contêiner mapeando a porta 8080:

Bash
docker run -d --name devops-api -p 8080:8080 devops-python-api:local
Verifique se o contêiner está ativo e responda ao endpoint de saúde:

Bash
docker ps
curl http://localhost:8080/health
5. Pipeline de Integração Contínua (CI)
A etapa de CI é definida no workflow .github/workflows/ci-cd.yml e é acionada automaticamente na abertura ou atualização de Pull Requests para a branch main:

Qualidade e Testes: Configura o ambiente Python, instala as dependências, executa a análise estática com ruff check . e roda a suíte de testes automatizados com pytest.

Validação de Contêiner: Constrói a imagem Docker, sobe um contêiner temporário no runner do GitHub Actions e valida a resposta HTTP do endpoint /health.

Caso qualquer etapa falhe, o merge e a etapa de entrega contínua são bloqueados.

6. Pipeline de Entrega Contínua (CD)
Após um merge aprovado na branch main, o workflow executa automaticamente os jobs de entrega contínua:

Revalida os testes automatizados e o build do contêiner.

Autentica-se na AWS via GitHub Secrets e realiza login no Amazon ECR.

Constrói a imagem Docker marcando-a com o hash curto do commit ($GITHUB_SHA) e latest, enviando-a ao repositório privado no ECR.

Conecta-se de forma segura via SSH à instância Amazon EC2.

Na EC2, autentica o Docker no ECR utilizando a IAM Role anexada à instância, baixa a nova imagem, interrompe e remove o contêiner anterior e inicia o novo contêiner na porta 8080.

7. Arquitetura de Implantação (GitHub + ECR + EC2)
Plaintext
[Desenvolvedor]
      │  (1. Branch -> Pull Request -> Merge na main)
      ▼
[GitHub / GitHub Actions]
      ├──► (2. CI: Ruff + Pytest + Teste Docker)
      ├──► (3. Build e Push da Imagem com tag do commit) ──► [Amazon ECR]
      └──► (4. Deploy via SSH) ────────────────────────────► [Amazon EC2]
                                                                 │ (5. IAM Role puxa imagem do ECR)
                                                                 ▼
                                                         [Contêiner Docker :8080]
8. Endereço de Acesso à API Publicada
Health Check: http://44.223.30.62:8080/health

Documentação Swagger UI: http://44.223.30.62:8080/docs

9. Evidências das Pipelines e Atualização Automática
As execuções completas de CI, publicação no ECR e deploy automatizado na EC2 estão registradas na aba Actions do repositório:

Pull Requests: Histórico de validação de branches e merges na aba Pull Requests (https://github.com/sarahmonteiroanjos/devops-python-api/pulls).

Atualização Automática (v1.0.0 → v1.1.0): Após o primeiro deploy com resposta {"status":"ok","version":"1.0.0"}, foi realizada a atualização de versão via branch/Pull Request e merge na main, disparando automaticamente a pipeline que atualizou a aplicação em produção para {"status":"ok","version":"1.1.0"}.

10. Como Verificar a Versão Implantada
Para confirmar a versão atualmente em execução na instância EC2, acesse pelo navegador ou execute via terminal:

Bash
curl [http://44.223.30.62:8080/health](http://44.223.30.62:8080/health)
Resposta esperada:

JSON
{"status":"ok","version":"1.1.0"}

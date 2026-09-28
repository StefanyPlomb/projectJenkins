# projectJenkins

Projeto de estudo de CI/CD: uma aplicação web em Django que é construída, testada e publicada automaticamente por um Jenkins sempre que o código muda.

## Como funciona

```
git push ──► GitHub ──► Jenkins ──► Docker Hub
                           │
                           └──► container web (porta 8000)
```

A cada commit na branch `main`, o Jenkins:

1. Baixa o código.
2. Constrói a imagem Docker.
3. Roda `manage.py check` e `manage.py test`.
4. Publica a imagem no Docker Hub (`stefanyplombon/projectjenkins`).
5. Recria o container `web` com a imagem nova.

O Jenkins verifica o repositório a cada ~2 minutos (`pollSCM`).

## Estrutura

```
.
├── config/                  # Configuração do Django
├── templates/hello.html     # Página inicial
├── manage.py
├── requirements.txt
├── Dockerfile               # Imagem da aplicação
├── docker-compose.yml       # Container web
├── Jenkinsfile              # Pipeline
└── jenkins/
    ├── Dockerfile           # Jenkins + Docker CLI
    └── docker-compose.yml   # Container do Jenkins
```

## Requisitos

- Docker e Docker Compose
- Conta no GitHub e no Docker Hub

## Como rodar

### Aplicação

```bash
docker build -t stefanyplombon/projectjenkins:latest .
docker compose up -d web
```

Acesse http://localhost:8000.

### Jenkins

```bash
cd jenkins
docker compose up -d --build
```

Acesse http://localhost:8080. A senha inicial está em:

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## Configuração do Jenkins

1. Instale os plugins sugeridos e crie o usuário admin.
2. Em **Manage Jenkins → Credentials**, crie duas credenciais do tipo *Username with password*:

   | ID          | Usuário           | Senha                          |
   | ----------- | ----------------- | ------------------------------ |
   | `github`    | `StefanyPlomb`    | Token do GitHub (`repo`)       |
   | `dockerhub` | `stefanyplombon`  | Token do Docker Hub (Read & Write) |

3. Crie um job do tipo **Pipeline** com:
   - **Definition:** Pipeline script from SCM
   - **SCM:** Git
   - **Repository URL:** `https://github.com/StefanyPlomb/projectJenkins.git`
   - **Credentials:** `github`
   - **Branch:** `*/main`
   - **Script Path:** `Jenkinsfile`
4. Clique em **Build Now** uma vez. Depois disso os builds disparam sozinhos a cada push.

## Pipeline

| Etapa    | O que faz                                                        |
| -------- | ---------------------------------------------------------------- |
| Checkout | Baixa o código do GitHub                                         |
| Build    | Constrói a imagem com as tags `<build>` e `latest`               |
| Check    | Executa `python manage.py check` dentro da imagem                |
| Test     | Executa `python manage.py test` dentro da imagem                 |
| Push     | Faz login no Docker Hub e envia as duas tags                     |
| Deploy   | Recria o container `web` com a imagem `latest`                   |

## Observações

- O Jenkins roda como `root` no container para acessar o `docker.sock` do host. Aceitável para estudo local, não para produção.
- A `SECRET_KEY` em `config/settings.py` é a chave de desenvolvimento do Django. Não use em produção.
- O `Jenkinsfile` e os compose não expõem nenhum segredo: as credenciais ficam somente no Jenkins.

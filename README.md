# 🔐 DevSecOps Juice Shop Pipeline

<p align="center">
  <img src="https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white"/>
  <img src="https://img.shields.io/badge/OWASP-000000?style=for-the-badge&logo=owasp&logoColor=white"/>
  <img src="https://img.shields.io/badge/ZAP-EE0000?style=for-the-badge&logo=owasp&logoColor=white"/>
  <img src="https://img.shields.io/badge/DefectDojo-1A1A2E?style=for-the-badge"/>
</p>

## 📌 Sobre o projeto

Este repositório demonstra um **pipeline DevSecOps completo**, integrando ferramentas de segurança (SAST, Secret Scanning, SCA e DAST) em um pipeline CI/CD orquestrado pelo **Jenkins**.

Como alvo de testes, utiliza o **[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)**, uma aplicação web intencionalmente vulnerável mantida pela OWASP para fins de aprendizado e prática em segurança de aplicações.

O objetivo é simular, de ponta a ponta, como um time de segurança integra verificações automatizadas ao ciclo de desenvolvimento — desde a análise do código-fonte até o ataque à aplicação em execução, centralizando todos os achados no **DefectDojo**.

---

## 🏗️ Arquitetura

O pipeline segue duas frentes de análise, que se complementam:

### 1. Análise sobre o código-fonte (estático)
Ferramentas que **não precisam** da aplicação rodando — analisam os arquivos do repositório diretamente, montados como volume dentro dos containers.

- **TruffleHog** → Secret Scanning (busca de credenciais, tokens e chaves expostas no código/histórico do git)
- **SonarQube** → SAST (análise estática de vulnerabilidades no código)
- **Trivy (modo `fs`)** → SCA (vulnerabilidades em dependências de terceiros, analisando o filesystem do projeto)

### 2. Análise sobre a aplicação em execução (dinâmico)
Ferramentas que atuam sobre a aplicação **de pé** ou sobre a imagem já construída.

- **Trivy (modo `image`)** → Escaneia a imagem Docker do Juice Shop em busca de vulnerabilidades conhecidas (CVEs) em pacotes do SO e camadas da imagem
- **OWASP ZAP** → DAST (varredura ativa via HTTP contra o Juice Shop rodando em container)

### Integração com o DefectDojo

Cada ferramenta gera seu próprio relatório em **JSON**, que é importado individualmente para o DefectDojo via **API** — cada uma com seu *scan type* específico. A exceção é o **SonarQube**, que se integra ao DefectDojo diretamente **via API** (sem gerar um arquivo JSON intermediário).

| Ferramenta   | Saída            | Integração com DefectDojo |
|--------------|------------------|-----------------------------|
| TruffleHog   | `.json`          | Import via API               |
| Trivy (fs)   | `.json`          | Import via API               |
| Trivy (image)| `.json`          | Import via API               |
| OWASP ZAP    | `.json`          | Import via API               |
| SonarQube    | —                | Integração direta via API    |

### Fluxo do pipeline

```
1. Checkout do código-fonte do Juice Shop (workspace do Jenkins)
        │
2. Secret Scanning (TruffleHog) → gera JSON
        │
3. SAST (SonarQube) → resultado integrado ao DefectDojo via API
        │
4. SCA (Trivy fs) → gera JSON, escaneia dependências do projeto
        │
5. Build da imagem Docker do Juice Shop
        │
6. Scan de imagem (Trivy image) → gera JSON, escaneia a imagem construída
        │
7. Sobe o Juice Shop via Docker (docker-compose up)
        │
8. DAST (OWASP ZAP) → ataca o Juice Shop rodando, gera JSON
        │
9. Todos os JSONs são importados ao DefectDojo via API
        │
10. DefectDojo centraliza, deduplica e apresenta as vulnerabilidades encontradas
```

### Diagrama de componentes

```
┌─────────────────────────────────────────────────────────┐
│                        Jenkins                           │
│         (orquestra todas as etapas do pipeline)          │
└──────────────┬──────────────────────────┬────────────────┘
               │                          │
   ┌───────────▼───────────┐   ┌──────────▼───────────────┐
   │   Código-fonte         │   │  Juice Shop (container /  │
   │   (volume montado)     │   │   imagem Docker)          │
   │                        │   │                            │
   │  • TruffleHog          │   │  • Trivy (image)           │
   │  • SonarQube            │   │  • OWASP ZAP (DAST)        │
   │  • Trivy (fs)           │   └────────────┬───────────────┘
   └────────────┬───────────┘                │
                │         JSON via API        │
                └───────────────┬─────────────┘
                        ┌───────▼────────┐
                        │   DefectDojo    │
                        │ (relatórios     │
                        │  centralizados) │
                        └─────────────────┘
```

Todos os componentes (Juice Shop, Jenkins, SonarQube, ZAP, DefectDojo) rodam em containers Docker, comunicando-se por uma rede interna compartilhada — garantindo reprodutibilidade total do ambiente.

---

## 🛠️ Ferramentas utilizadas

| Categoria             | Ferramenta                | Função                                              |
|------------------------|----------------------------|-------------------------------------------------------|
| Orquestração           | Jenkins                   | Automatiza e executa o pipeline CI/CD                  |
| Secret Scanning        | TruffleHog                | Detecta credenciais e segredos expostos no código      |
| SAST                   | SonarQube                  | Analisa vulnerabilidades no código-fonte (integração via API) |
| SCA                    | Trivy (modo `fs`)          | Verifica vulnerabilidades em dependências do projeto   |
| Scan de imagem         | Trivy (modo `image`)       | Verifica CVEs na imagem Docker do Juice Shop           |
| DAST                   | OWASP ZAP                  | Ataca a aplicação em execução                          |
| Gestão de vulns.       | DefectDojo                 | Centraliza, deduplica e reporta os achados (import via API) |
| Containerização        | Docker / Docker Compose    | Isola e orquestra todos os serviços                    |
| Aplicação alvo         | OWASP Juice Shop           | Aplicação intencionalmente vulnerável                  |

---

## 🚀 Como rodar o projeto

> 🚧 Seção em construção — será atualizada conforme o pipeline for implementado.

```bash
# Clonar o repositório
git clone https://github.com/hick12/devsecops-juice-shop-pipeline.git
cd devsecops-juice-shop-pipeline

# Subir o ambiente completo
docker-compose up -d
```

---

## 📊 Vulnerabilidades encontradas

> 🚧 Seção em construção — os relatórios e prints do DefectDojo serão adicionados aqui conforme as varreduras forem executadas.

---

## 📁 Estrutura do repositório

```
.
├── jenkins/            # Configurações e Jenkinsfile do pipeline
├── docker-compose.yml  # Orquestração dos containers
├── reports/            # Relatórios gerados pelas ferramentas
├── README.md
├── .gitignore
└── LICENSE
```

---

## 🎯 Objetivo do projeto

Este projeto faz parte do meu processo de estudo e construção de portfólio na área de **DevSecOps**, aplicando na prática conceitos de integração de segurança em pipelines de CI/CD (Shift-Left Security), com foco nas seguintes competências:

- Automação de pipelines de segurança
- Integração de ferramentas SAST, DAST e SCA
- Gestão e centralização de vulnerabilidades
- Containerização e orquestração com Docker

---

## 🌎 Contato

- 💼 [LinkedIn](https://www.linkedin.com/in/henrique-palorca-734a6b239/)
- ✉️ henriquepalorca@gmail.com
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
- **Semgrep / SonarQube** → SAST (análise estática de vulnerabilidades no código)
- **OWASP Dependency-Check** → SCA (vulnerabilidades em dependências de terceiros)

### 2. Análise sobre a aplicação em execução (dinâmico)
Ferramenta que ataca a aplicação **de pé**, via HTTP, como um usuário real faria.

- **OWASP ZAP** → DAST (varredura ativa contra o Juice Shop rodando em container)

### Fluxo do pipeline

```
1. Checkout do código-fonte do Juice Shop (workspace do Jenkins)
        │
2. Secret Scanning (TruffleHog) → sobre os arquivos do repositório
        │
3. SAST (Semgrep/SonarQube) → sobre os arquivos do repositório
        │
4. SCA (Dependency-Check) → sobre as dependências do projeto
        │
5. Sobe o Juice Shop via Docker (docker-compose up)
        │
6. DAST (OWASP ZAP) → ataca o Juice Shop rodando
        │
7. Todos os relatórios são enviados ao DefectDojo
        │
8. DefectDojo centraliza, deduplica e apresenta as vulnerabilidades encontradas
```

### Diagrama de componentes

```
┌─────────────────────────────────────────────────────────┐
│                        Jenkins                           │
│         (orquestra todas as etapas do pipeline)          │
└──────────────┬──────────────────────────┬────────────────┘
               │                          │
   ┌───────────▼───────────┐   ┌──────────▼───────────┐
   │   Código-fonte         │   │  Juice Shop (container)│
   │   (volume montado)     │   │   rodando na porta 3000│
   │                        │   │                        │
   │  • TruffleHog          │   │  • OWASP ZAP (DAST)    │
   │  • Semgrep/SonarQube   │   └────────────────────────┘
   │  • Dependency-Check    │
   └────────────┬───────────┘
                │
        ┌───────▼────────┐
        │   DefectDojo    │
        │ (relatórios     │
        │  centralizados) │
        └─────────────────┘
```

Todos os componentes (Juice Shop, Jenkins, ZAP, DefectDojo) rodam em containers Docker, comunicando-se por uma rede interna compartilhada — garantindo reprodutibilidade total do ambiente.

---

## 🛠️ Ferramentas utilizadas

| Categoria         | Ferramenta                | Função                                          |
|--------------------|----------------------------|--------------------------------------------------|
| Orquestração       | Jenkins                   | Automatiza e executa o pipeline CI/CD             |
| Secret Scanning    | TruffleHog                | Detecta credenciais e segredos expostos no código |
| SAST               | Semgrep / SonarQube        | Analisa vulnerabilidades no código-fonte          |
| SCA                | OWASP Dependency-Check     | Verifica vulnerabilidades em dependências         |
| DAST               | OWASP ZAP                  | Ataca a aplicação em execução                     |
| Gestão de vulns.   | DefectDojo                 | Centraliza, deduplica e reporta os achados        |
| Containerização    | Docker / Docker Compose    | Isola e orquestra todos os serviços               |
| Aplicação alvo     | OWASP Juice Shop           | Aplicação intencionalmente vulnerável             |

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

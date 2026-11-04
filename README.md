# TI Centronictec — Sistema Web Institucional com Atendimento Digital Integrado

## Sumário

- [Identificação Acadêmica](#identificação-acadêmica)
- [Descrição](#descrição)
- [Objetivos](#objetivos)
- [Arquitetura da Solução](#arquitetura-da-solução)
  - [Diagrama da Arquitetura](#diagrama-da-arquitetura)
  - [Pontos Arquiteturais Chave](#pontos-arquiteturais-chave)
- [Desafios Arquiteturais: Por que não usamos a Vercel?](#desafios-arquiteturais-por-que-não-usamos-a-vercel)
- [Resiliência e Recuperação de Desastres](#resiliência-e-recuperação-de-desastres)
- [Stack Tecnológica](#stack-tecnológica)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Funcionalidades](#funcionalidades)
- [Modelagem do Banco de Dados](#modelagem-do-banco-de-dados)
- [Pré-requisitos](#pré-requisitos)
- [Instalação](#instalação-desenvolvimento)
- [Execução em Produção](#execução-em-produção)
- [Estrutura do Projeto](#estrutura-do-projeto)
- [Fluxo de Atendimento Digital](#fluxo-de-atendimento-digital)
- [API — Rotas Principais](#api--rotas-principais)
- [Critérios de Aceitação](#critérios-de-aceitação)
- [Equipe do Projeto](#equipe-do-projeto)
- [Melhorias Futuras](#melhorias-futuras)
- [Licença](#licença)
- [Status do Projeto](#status-do-projeto)

---

## Identificação Acadêmica

- **Instituição de Ensino:** SENAC — CEP Jó Rufino 2025 / 2026
- **Curso:** Técnico em Desenvolvimento de Sistemas
- **Disciplina:** Aplicativo Node.js
- **Orientador:** Profº Hudson Neves
- **Aluno:** Vanderli Marques dos Reis
- **Local & Ambiente:** Taguatinga Norte — Oracle Cloud Infrastructure (OCI)
- **Domínio:** [https://ti.centronictec.com.br](https://ti.centronictec.com.br)

---

## Descrição

O **TI Centronictec** é um sistema web institucional destinado a apresentar serviços de assistência e suporte em tecnologia, equipamentos, sistemas operacionais e infraestrutura de redes.

A aplicação oferece navegação por áreas de serviço, conteúdo institucional, apresentação visual responsiva e um canal de atendimento digital integrado à plataforma **RoboCV**.

O escopo documentado abrange a interface web, a navegação entre páginas de serviços, a apresentação do conteúdo, o gerenciamento de tema visual, o backend de integração de chat e a comunicação com serviços externos. A plataforma RoboCV é tratada como sistema externo integrado; sua implementação interna não faz parte do escopo desta aplicação.

---

## Objetivos

- **Objetivo geral:** Entregar uma solução web institucional com atendimento digital integrado, escalável e segura, para apresentação de serviços de TI e captação de leads.
- **Problema que o sistema resolve:** Centralizar a apresentação institucional e o primeiro contato com o cliente em uma única interface web, encaminhando o atendimento digital para a plataforma RoboCV de forma autenticada e rastreável.
- **Público-alvo:** Visitantes do site institucional, potenciais clientes que desejam solicitar avaliação ou orçamento e operadores que dão continuidade aos atendimentos na plataforma RoboCV.

---

## Arquitetura da Solução

A aplicação é dividida em duas grandes camadas, com separação clara de responsabilidades:

- **Camada de apresentação (Frontend):** Aplicação construída com React Native e React Native Web, utilizando Expo e Expo Router, responsiva e com suporte a temas claro e escuro.
- **Camada de aplicação (Backend):** Servidor Node.js com Express, conectado ao MongoDB via Mongoose, responsável por expor a API de atendimento digital, validar dados, gerenciar sessões autenticadas e integrar-se à API RoboCV.

### Diagrama da Arquitetura

```mermaid
flowchart TB
    USER["Usuários / Navegadores"]

    subgraph DOMAIN["1. Registro e DNS"]
        REG["Registro.br<br/>Domínio: centronictec.com.br"]
        DNS["Cloudflare DNS<br/>Nameservers e registros DNS"]
        REG -->|"Delegação DNS"| DNS
    end

    subgraph CLOUDFLARE["2. Cloudflare Zero Trust"]
        CF["Cloudflare Edge<br/>HTTPS / Proxy / Proteção"]
        H1["ti.centronictec.com.br"]

        H1 --> CF
    end

    subgraph OCI["3. Oracle Cloud Infrastructure — OCI"]
        UB["OCI Compute<br/>Ubuntu Linux"]

        T["cloudflared<br/>Cloudflare Tunnel"]

        subgraph SERVICES["Serviços locais"]
            APP["API TI Centronictec<br/>Express :3001"]
        end

        UB --> T
        T --> APP
    end

    subgraph EXTERNAL["4. Serviços externos"]
        MDB["MongoDB Atlas<br/>Dados da aplicação"]
        ROBOCV["API RoboCV<br/>Plataforma de atendimento"]
    end

    USER -->|"HTTPS"| CF
    DNS -.->|"Resolução DNS"| CF
    CF <-->|"Túnel criptografado"| T

    APP -->|"Mongoose / MONGODB_URI"| MDB
    APP -->|"HTTP + Bearer Token<br/>ROBOCV_API_URL"| ROBOCV

    classDef dns fill:#e8f0fe,stroke:#4285f4,color:#172554
    classDef cloud fill:#f3e8ff,stroke:#9333ea,color:#581c87
    classDef app fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef data fill:#fef3c7,stroke:#d97706,color:#78350f

    class REG,DNS,USER dns
    class CF,H1,UB,T cloud
    class APP app
    class MDB,ROBOCV data

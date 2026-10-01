# FileServer Permission Auditor

> Auditoria de permissões de pastas em um FileServer Windows Server, cruzando as informações com os membros dos grupos de segurança do Active Directory, gerando um dashboard HTML interativo para facilitar a análise e tomada de decisão.

**Stack:** PowerShell · Active Directory · HTML/CSS/JS

![PowerShell](https://img.shields.io/badge/PowerShell-5.1+-blue?style=flat-square&logo=powershell)
![Active Directory](https://img.shields.io/badge/Active%20Directory-RSAT-0078D4?style=flat-square&logo=microsoft)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Version](https://img.shields.io/badge/Version-1.0-orange?style=flat-square)

---

## Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Funcionalidades](#-funcionalidades)
- [Pré-requisitos](#-pré-requisitos)
- [Instalação](#-instalação)
- [Configuração](#️-configuração)
- [Como Usar](#-como-usar)
- [Dashboard](#-dashboard)
- [Estrutura do Projeto](#-estrutura-do-projeto)
- [Exemplos de Saída](#-exemplos-de-saída)
- [Personalização](#-personalização)
- [Boas Práticas](#-boas-práticas)
- [Roadmap](#-roadmap)
- [Contribuindo](#-contribuindo)
- [Licença](#-licença)

---

## Sobre o Projeto

O **FileServer Permission Auditor** é uma ferramenta PowerShell que automatiza a auditoria de permissões em FileServers, resolvendo um problema comum em ambientes corporativos: **a dificuldade em visualizar quem tem acesso a quais pastas**.

### Problema Resolvido

- ❌ Dificuldade em visualizar quem tem acesso a quais pastas
- ❌ Necessidade de cruzar informações entre permissões NTFS e grupos AD
- ❌ Falta de um relatório visual e amigável para auditoria
- ❌ Dificuldade em identificar permissões desnecessárias ou excessivas
- ❌ Planilhas CSV extensas e de difícil leitura

### Solução

- ✅ Dashboard HTML interativo e visualmente agradável
- ✅ Cruzamento automático de permissões NTFS com grupos AD
- ✅ Busca e filtros em tempo real
- ✅ Filtragem automática de contas de sistema e administradores
- ✅ Relatório portátil (arquivo único HTML)

---

## Funcionalidades

### 1. Auditoria de Pastas

- Varre todas as pastas do diretório especificado
- Coleta as permissões NTFS (ACLs) de cada pasta
- Identifica se a permissão é herdada ou direta
- Filtra automaticamente contas de sistema e administradores

### 2. Coleta de Grupos AD

- Busca grupos dentro de uma OU específica
- Lista todos os membros de cada grupo
- Identifica o tipo de objeto (usuário, grupo, etc.)
- Filtra contas indesejadas automaticamente

### 3. Geração de Dashboard HTML

- Relatório visual e interativo
- Estatísticas resumidas (cards)
- Busca em tempo real
- Filtros por tipo e por pasta
- Cores diferenciadas por tipo de permissão
- Design responsivo (desktop, tablet, mobile)

---

## Pré-requisitos

| Requisito | Versão | Como Instalar |
|-----------|--------|---------------|
| **PowerShell** | 5.1+ | Nativo do Windows |
| **Módulo ActiveDirectory** | RSAT-AD-PowerShell | `Install-WindowsFeature RSAT-AD-PowerShell` |
| **Permissões no FileServer** | Leitura | Solicitar ao administrador |
| **Permissões no AD** | Leitura | Solicitar ao administrador |

⚠️ O relatório contém informações sensíveis (nomes de usuários, grupos e permissões)
⚠️ Não armazene em locais públicos ou sem proteção
⚠️ Considere criptografar o arquivo antes de enviar por e-mail

## Documentação adicional

Para instruções completas de configuração, consulte:
**[Guia de Configuração Detalhado](docs/TROUBLESHOOTING.md)**

Para resolução de erros, consulte:
**[Guia de Configuração Detalhado](docs/CONFIGURATION.md)**

Exemplo da saida em HTML, consulte:
**[Guia de Configuração Detalhado](docs/Dashboard_Permissoes.html)**

---

## 🖼️ Pré-visualização do Dashboard

Veja como fica o dashboard gerado pelo script com dados de exemplo:

<a href="https://htmlpreview.github.io/?https://github.com/MatheusCDB/FileServer-Permission-Auditor/blob/main/docs/Dashboard_Permissoes.html" target="_blank">
  <img src="docs/screenshots/dashboard-preview.png" alt="Preview do Dashboard" width="100%">
</a>

<div align="center">

### 🔗 [**Clique aqui para ver o Dashboard interativo ao vivo**](https://htmlpreview.github.io/?https://github.com/MatheusCDB/FileServer-Permission-Auditor/blob/main/docs/Dashboard_Permissoes.html)

*Explore busca, filtros e todas as funcionalidades diretamente no navegador*

---

## Possíveis evoluções

- Interface gráfica (WPF)
- Envio automático por e-mail
- Integração com SIEM
- API REST
- Alertas automáticos para extensões novas / suspeitas

## ⚙️ Configuração

### 📦 Pré-requisitos

| Requisito | Versão | Como Instalar |
|-----------|--------|---------------|
| **PowerShell** | 5.1+ | Nativo do Windows |
| **Módulo ActiveDirectory** | RSAT-AD-PowerShell | Veja abaixo |
| **Permissões no FileServer** | Leitura | Solicitar ao administrador |
| **Permissões no AD** | Leitura | Solicitar ao administrador |

#### Instalando o Módulo ActiveDirectory

```powershell
# Em Windows Server
Install-WindowsFeature RSAT-AD-PowerShell

# Em Windows 10/11
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```

Para verificar se o módulo está instalado:

```powershell
Get-Module -ListAvailable -Name ActiveDirectory
```

---

### Variáveis Editáveis

Antes de executar, configure as variáveis no início do script:

```powershell

# Caminho do FileServer a ser auditado
$FolderPathToAudit = "\\fileserver\Arquivos"

# Pasta onde o relatório será salvo
$OutputFolder = $env:USERPROFILE + "\Documents"

# Nome do arquivo de saída
$OutputFileName = "Dashboard_Permissoes.html"

# OU do Active Directory para busca dos grupos
$ADSearchBase = "OU=Matriz,OU=Fileserver,OU=Servicos,OU=Empresa,DC=Empresa,DC=local"

# Filtro para grupos (ex: "FileServer_*" para grupos específicos)
$ADGroupFilter = "*"

# Contas a serem excluídas do relatório
$ExcludedAccounts = @(
    "PROPRIETÁRIO CRIADOR",
    "CREATOR OWNER",
    "AUTORIDADE NT\SISTEMA",
    "NT AUTHORITY\SYSTEM",
    "BUILTIN\Administradores",
    "BUILTIN\ADMINISTRATORS",
    "ADMINS. DO DOMÍNIO",
    "DOMAIN ADMINS",
    "DOMINIO DA EMPRESA\ADMINS. DO DOMÍNIO",
    "DOMINIO DA EMPRESA\ADMINISTRADOR",
    "NT AUTHORITY\Authenticated Users",
    "USUÁRIOS AUTENTICADOS",
    "Everyone",
    "Todos"
)
```

#### Referência das Variáveis

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `$FolderPathToAudit` | Caminho UNC ou local do FileServer | `\\fileserver\Arquivos` |
| `$OutputFolder` | Pasta de destino do relatório | `C:\Relatorios` |
| `$OutputFileName` | Nome do arquivo HTML gerado | `Dashboard.html` |
| `$ADSearchBase` | DN da OU para busca de grupos | `OU=Fileserver,DC=empresa,DC=local` |
| `$ADGroupFilter` | Filtro para grupos específicos | `FileServer_*` |
| `$ExcludedAccounts` | Array de contas a excluir | `@("SYSTEM", "Administradores")` |

---

### Filtros Avançados

#### Filtrar grupos específicos

Você pode limitar a busca do AD a apenas grupos que sigam um padrão de nomenclatura:

```powershell
# Apenas grupos que começam com "FileServer_"
$ADGroupFilter = "FileServer_*"

# Apenas grupos com "Financeiro" no nome
$ADGroupFilter = "*Financeiro*"

# Apenas grupos que terminam com "_RW" (Read/Write)
$ADGroupFilter = "*_RW"
```

#### Adicionar mais contas para excluir

Para excluir contas adicionais do relatório, adicione-as ao array:

```powershell
$ExcludedAccounts += @(
    "NOVO_USUARIO",
    "OUTRO_GRUPO",
    "DOMINIO\CONTA_ESPECIFICA"
)
```

>  **Dica:** O filtro usa comparação `-like`, então `*palavra*` funciona para correspondência parcial.

---

### Fluxo de Execução

```
┌─────────────────────────────────────────┐
│  1. Coleta permissões das pastas        │
│     (Varre pastas + Lê ACLs)            │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  2. Coleta membros dos grupos AD        │
│     (Busca na OU + Lista membros)       │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  3. Cruza as informações                │
│     (Aplica filtros + Exclusões)        │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  4. Gera o dashboard HTML               │
│     (Aplica estilos + Interatividade)   │
└──────────────────┬──────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│  5. Abre automaticamente no navegador   │
│     (Start-Process)                     │
└─────────────────────────────────────────┘
```

---

### Execução Rápida

```powershell
# 1. Abra o PowerShell como Administrador

# 2. Navegue até a pasta do script
cd C:\Scripts\FileServer-Permission-Auditor

# 3. Execute o script
.\Invoke-FileServerAudit.ps1
```

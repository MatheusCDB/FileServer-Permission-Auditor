# Solução de Problemas

> Guia completo para resolver problemas comuns do **FileServer Permission Auditor**.

---

## 📋 Índice

- [Erros Comuns](#-erros-comuns)
- [Problemas de Permissão](#-problemas-de-permissão)
- [Problemas com Active Directory](#-problemas-com-active-directory)
- [Problemas de Performance](#-problemas-de-performance)
- [Problemas com o HTML](#-problemas-com-o-html)
- [Logs e Diagnóstico](#-logs-e-diagnóstico)
- [FAQ](#-faq)
- [Suporte](#-suporte)

---

## Erros Comuns

### Erro: "Módulo ActiveDirectory não encontrado"

**Sintoma:**
```
Get-ADGroup : O termo 'Get-ADGroup' não é reconhecido como nome de cmdlet,
função, arquivo de script ou programa operável.
```

**Causa:** O módulo RSAT-AD-PowerShell não está instalado.

**Solução:**

```powershell
# Windows Server
Install-WindowsFeature RSAT-AD-PowerShell

# Windows 10/11
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
```

**Verificar instalação:**
```powershell
Get-Module -ListAvailable -Name ActiveDirectory
```

---

### Erro: "Não é possível carregar o arquivo porque a execução de scripts foi desabilitada"

**Sintoma:**
```
Não é possível carregar o arquivo ... porque a execução de scripts foi desabilitada neste sistema.
```

**Causa:** A política de execução do PowerShell está restritiva.

**Solução:**

```powershell
# Verificar política atual
Get-ExecutionPolicy

# Permitir execução de scripts locais (recomendado)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Ou executar apenas este script
powershell.exe -ExecutionPolicy Bypass -File .\Invoke-FileServerAudit.ps1
```

>  **Atenção:** Não use `Unrestricted` a menos que entenda os riscos.

---

### Erro: "O acesso ao caminho 'C:\...' foi negado"

**Sintoma:**
```
Out-File : O acesso ao caminho 'C:\Users\...\Desktop' foi negado.
```

**Causa:** Falta de permissão de escrita na pasta de destino.

**Solução:**

**Opção 1:** Execute o PowerShell como Administrador

**Opção 2:** Altere a pasta de saída para um local com permissão:
```powershell
$OutputFolder = "$env:USERPROFILE\Documents"
# ou
$OutputFolder = "C:\Temp"
# ou
$OutputFolder = "C:\Relatorios"
```

**Opção 3:** Verifique as permissões da pasta:
```powershell
icacls "C:\Users\Administrator\Desktop"
```

---

###  Erro: "Diretório X não encontrado"

**Sintoma:**
```
AVISO: Diretório F:\ não encontrado!
✅ Permissões coletadas: 0
```

**Causa:** O caminho do FileServer está incorreto ou inacessível.

**Solução:**

```powershell
# Testar se o caminho existe
Test-Path "F:\"

# Testar acesso via UNC
Test-Path "\\fileserver\Arquivos"

# Listar conteúdo para confirmar
Get-ChildItem "\\fileserver\Arquivos" -Directory
```

**Verificações:**
- ✅ A letra da unidade está correta?
- ✅ O servidor está acessível na rede?
- ✅ Você tem permissão de leitura?
- ✅ O firewall está bloqueando o acesso SMB?

---

###  Erro: "Não há mais arquivos e pastas para processar"

**Sintoma:**
```
Get-ChildItem : Não há mais arquivos e pastas para processar.
```

**Causa:** Você está executando o script de uma unidade de rede que foi desconectada.

**Solução:**

```powershell
# Antes de executar, mapeie a unidade novamente
net use Z: \\fileserver\Arquivos /persistent:yes

# Ou use caminho UNC diretamente
$FolderPathToAudit = "\\fileserver\Arquivos"
```

---

### Erro: "Erro ao acessar permissões de: X"

**Sintoma:**
```
AVISO: Erro ao acessar permissões de: \\fileserver\PastaRestrita
```

**Causa:** Falta de permissão de leitura das ACLs em pastas específicas.

**Solução:**

**Opção 1:** Execute como Administrador de Domínio

**Opção 2:** Verifique se a conta tem `SeSecurityPrivilege`:
```powershell
whoami /priv | findstr SeSecurityPrivilege
```

**Opção 3:** Solicite acesso ao administrador do FileServer

>  **Nota:** Pastas com erros são automaticamente puladas e aparecem como aviso, não impedem a execução.

---

##  Problemas de Permissão

### Não consigo ler as permissões de algumas pastas

**Sintomas:**
- Pastas ausentes no relatório
- Avisos de "Erro ao acessar permissões"

**Soluções:**

1. **Execute como Administrador de Domínio**
   ```powershell
   # Verificar se está executando como admin
   ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
   ```

2. **Verifique se tem o privilégio necessário**
   ```powershell
   whoami /priv
   # Procure por: SeBackupPrivilege, SeSecurityPrivilege
   ```

3. **Solicite ao administrador do FileServer** permissão de leitura nas ACLs

---

### O relatório mostra grupos que não existem mais

**Causa:** Grupos foram deletados do AD mas as permissões NTFS ainda os referenciam.

**Solução:**

1. Identifique os grupos órfãos no dashboard
2. Verifique cada um no AD:
   ```powershell
   Get-ADGroup -Identity "NOME_DO_GRUPO" -ErrorAction SilentlyContinue
   ```
3. Documente para limpeza futura

---

## 🏢 Problemas com Active Directory

###  Erro: "Nenhum grupo encontrado na OU"

**Sintoma:**
```
AVISO: Nenhum grupo encontrado na OU: OU=Matriz,OU=Fileserver,...
✅ Membros coletados: 0
```

**Causas e Soluções:**

1. **Caminho da OU está incorreto**
   ```powershell
   # Verificar se a OU existe
   Get-ADOrganizationalUnit -Identity "OU=Matriz,OU=Fileserver,OU=Servicos,OU=Servitech,DC=servitech,DC=local"
   ```

2. **Ordem dos componentes está errada**
   ```
   ❌ ERRADO: OU=Servicos,OU=Matriz,OU=Fileserver,...
   ✅ CERTO:  OU=Matriz,OU=Fileserver,OU=Servicos,...
   ```
   >  **Regra:** A ordem é **do mais específico para o mais genérico**

3. **Não há grupos dentro desta OU**
   ```powershell
   # Listar grupos na OU
   Get-ADGroup -Filter * -SearchBase "OU=Matriz,OU=Fileserver,OU=Servicos,OU=Servitech,DC=servitech,DC=local"
   ```

---

### ❌ Erro: "Não é possível contatar o domínio"

**Sintoma:**
```
Get-ADGroup : Não é possível contatar o domínio...
```

**Soluções:**

1. **Verifique a conectividade com o DC:**
   ```powershell
   Test-NetConnection -ComputerName dc01.servitech.local -Port 389
   ```

2. **Verifique se o computador está no domínio:**
   ```powershell
   (Get-WmiObject Win32_ComputerSystem).Domain
   ```

3. **Force a descoberta do DC:**
   ```powershell
   $env:USERDNSDOMAIN
   nltest /dsgetdc:servitech.local
   ```

---

### ❌ Erro: "Membros não estão sendo coletados (1291 membros mas nenhum aparece)"

**Sintoma:** O script diz que coletou membros, mas o dashboard está vazio.

**Causa:** Todos os membros foram filtrados pelas exclusões.

**Solução:**

1. **Verifique a lista de exclusão:**
   ```powershell
   $ExcludedAccounts
   ```

2. **Reduza as exclusões temporariamente** para debug:
   ```powershell
   $ExcludedAccounts = @()  # Lista vazia para testar
   ```

3. **Verifique se o `-like` não está capturando demais:**
   ```powershell
   # Cuidado com padrões muito amplos
   "DOMINIO\Usuario" -like "*DOMINIO*"  # Retorna $true - pode ser problema
   ```

---

##  Problemas de Performance

### O script está muito lento

**Causas possíveis:**
- Muitas pastas para auditar
- Latência de rede alta
- Muitos grupos no AD

**Soluções:**

**1. Auditar subpastas específicas:**
```powershell
# Ao invés de auditar F:\
$FolderPathToAudit = "F:\Departamentos"
```

**2. Usar filtro de grupos:**
```powershell
# Ao invés de buscar TODOS os grupos
$ADGroupFilter = "FileServer_*"  # Apenas grupos relevantes
```

**3. Executar em horário de baixo uso:**
```powershell
# Agendar para 3h da manhã
$Trigger = New-ScheduledTaskTrigger -Daily -At 3am
```

**4. Auditar apenas um nível de profundidade:**
>  O script já audita apenas o primeiro nível. Se quiser recursivo, precisa modificar.

---

### O script travou e não terminou

**Sintomas:**
- Barra de progresso parada
- Console não responde

**Soluções:**

1. **Aguarde** — pastas de rede podem ser lentas
2. **Verifique a conectividade:**
   ```powershell
   Test-NetConnection -ComputerName fileserver
   ```
3. **Mate o processo e reinicie:**
   ```powershell
   Get-Process powershell | Stop-Process -Force
   ```
4. **Reduza o escopo** e execute em partes

---

### A barra de progresso mostra percentuais estranhos

**Sintoma:** Progresso pula de 10% para 90%, ou fica "preso".

**Causa:** Comportamento normal do `Write-Progress` com `Get-ADGroupMember`.

**Solução:** Nenhuma ação necessária. É apenas visual.

---

##  Problemas com o HTML

### O arquivo HTML não abre

**Sintomas:**
- Duplo clique não faz nada
- Navegador mostra erro

**Soluções:**

1. **Verifique se o arquivo foi criado:**
   ```powershell
   Test-Path $OutputPath
   Get-Item $OutputPath | Select-Object Length, LastWriteTime
   ```

2. **Abra manualmente:**
   ```powershell
   Start-Process $OutputPath
   # Ou especifique o navegador
   Start-Process "chrome.exe" -ArgumentList $OutputPath
   ```

3. **Verifique o tamanho do arquivo:**
   ```powershell
   # Se for 0 KB, houve erro na geração
   (Get-Item $OutputPath).Length / 1KB
   ```

---

### Acentuação está errada no dashboard

**Sintoma:** "Permissões" aparece como "PermissÃµes" ou "Permiss�es".

**Causa:** Encoding incorreto ao salvar o arquivo.

**Solução:**

O script já usa `-Encoding UTF8`, mas se o problema persistir:

```powershell
# Verificar o encoding do arquivo
Get-Content $OutputPath -Encoding Byte -TotalCount 3
# Deve retornar: 239 187 191 (BOM UTF-8)

# Regravar com encoding correto
$HtmlContent | Out-File -FilePath $OutputPath -Encoding UTF8 -Force
```

**No HTML, verifique se tem:**
```html
<meta charset="UTF-8">
```

---

### Os filtros não funcionam

**Sintomas:**
- Digitar na busca não filtra nada
- Dropdowns não têm efeito

**Causas:**
- JavaScript desabilitado no navegador
- Bloqueio de scripts locais
- Navegador muito antigo

**Soluções:**

1. **Habilite JavaScript:**
   - Chrome: Configurações → Privacidade → JavaScript
   - Edge: Configurações → Cookies e permissões → JavaScript

2. **Use navegador moderno:**
   - Chrome 90+
   - Edge 90+
   - Firefox 88+

3. **Teste com outro navegador** para isolar o problema

---

### O dashboard está lento para filtrar

**Causa:** Muitos registros (> 5.000 linhas).

**Soluções:**

1. **Reduza o escopo da auditoria**
2. **Use filtros antes de buscar** (reduz o conjunto)
3. **Considere dividir em relatórios menores**

---

### O dashboard não abre em outro computador

**Sintomas:**
- Funciona no seu PC mas não no do cliente
- Página em branco

**Causas:**
- Arquivo corrompido na transferência
- Encoding alterado pelo e-mail
- Navegador incompatível

**Soluções:**

1. **Compacte antes de enviar:**
   ```powershell
   Compress-Archive -Path $OutputPath -DestinationPath "Dashboard.zip"
   ```

2. **Envie por compartilhamento de rede** ao invés de e-mail

3. **Verifique se o arquivo chegou íntegro:**
   ```powershell
   # Compare o hash MD5
   Get-FileHash $OutputPath -Algorithm MD5
   ```

---

##  Logs e Diagnóstico

### Como habilitar logs detalhados

Adicione no início do script:

```powershell
# Habilita log de transcrição
Start-Transcript -Path "C:\Temp\AuditLog.txt" -Append

# ... resto do script ...

# No final
Stop-Transcript
```

### Como debugar problemas específicos

**Testar apenas a coleta de pastas:**
```powershell
$FolderPermissions = Get-FolderPermissions -FolderPath "F:\"
$FolderPermissions | Format-Table -AutoSize
```

**Testar apenas a coleta do AD:**
```powershell
$GroupMembers = Get-ADGroupMembers -SearchBase "OU=Matriz,OU=..." -GroupFilter "*"
$GroupMembers | Format-Table -AutoSize
```

**Testar o filtro de exclusão:**
```powershell
Should-Exclude -AccountName "NT AUTHORITY\SYSTEM"     # Deve retornar $true
Should-Exclude -AccountName "DOMINIO\Usuario.Comum"   # Deve retornar $false
```

### Coletar informações para suporte

Se for abrir uma issue, inclua:

```powershell
# Versão do PowerShell
$PSVersionTable.PSVersion

# Versão do módulo AD
Get-Module -ListAvailable ActiveDirectory | Select-Object Name, Version

# Sistema operacional
Get-ComputerInfo | Select-Object OsName, OsVersion, CsDomain

# Domínio
$env:USERDNSDOMAIN

# Permissões de admin
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

---

##  FAQ

### O script funciona em PowerShell 7?

**R:** Parcialmente. O módulo `ActiveDirectory` é compatível com PowerShell 7 desde a versão 7.0. Recomendamos **PowerShell 5.1** para máxima compatibilidade.

### Posso auditar múltiplos FileServers de uma vez?

**R:** Não nativamente. Execute o script uma vez para cada servidor, alterando `$FolderPathToAudit`.

### O script altera as permissões?

**R:** **Não!** O script é 100% **read-only**. Apenas lê e reporta as permissões.

### Posso agendar execução automática?

**R:** Sim! Use o Task Scheduler do Windows:

```powershell
$Action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File C:\Scripts\Invoke-FileServerAudit.ps1"

$Trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday -At 3am

Register-ScheduledTask -Action $Action -Trigger $Trigger `
    -TaskName "FileServer-Audit" -Description "Auditoria semanal de permissões"
```

### Como auditar pastas recursivamente (subpastas)?

**R:** O script audita apenas o **primeiro nível**. Para subpastas, modifique a função `Get-FolderPermissions`:

```powershell
# Adicione -Recurse no Get-ChildItem
$Folders = Get-ChildItem -Directory -Path $FolderPath -Force -Recurse
```

>  **Cuidado:** Isso pode gerar relatórios gigantes e demorar muito.

### Posso exportar para Excel?

**R:** Não diretamente. Mas você pode:
1. Salvar o HTML como PDF (`Ctrl+P`)
2. Modificar o script para exportar CSV adicional
3. Copiar dados das tabelas e colar no Excel

### O dashboard mostra permissões efetivas ou apenas NTFS?

**R:** Apenas **NTFS**. Não considera:
- Permissões de compartilhamento
- Grupos aninhados (mostra o grupo direto, não os membros)
- Permissões efetivas (combinação de Allow/Deny)

### Como lidar com grupos aninhados?

**R:** O script mostra a **membership direta**. Para expandir grupos aninhados recursivamente:

```powershell
Get-ADGroupMember -Identity "GRUPO" -Recursive
```

### O relatório inclui permissões de compartilhamento (Share)?

**R:** Não. O script audita apenas **NTFS**. Para auditar shares:

```powershell
Get-SmbShare -CimSession "fileserver"
Get-SmbShareAccess -Name "ShareName" -CimSession "fileserver"
```

### Como ignorar pastas específicas?

**R:** Modifique a função `Get-FolderPermissions`:

```powershell
$Folders = Get-ChildItem -Directory -Path $FolderPath -Force |
    Where-Object { $_.Name -notin @("PastaA", "PastaB", "Temp") }
```

### O script funciona em domínios com trust?

**R:** Parcialmente. Você pode precisar especificar o `-Server` nos cmdlets AD:

```powershell
Get-ADGroup -Filter * -Server "dc01.servitech.local"
```
Se este guia ajudou, considere dar uma ⭐ no projeto!

</div>

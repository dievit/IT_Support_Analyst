Você tem razão! A formatação precisa ser ajustada para o GitHub. Aqui está a versão corrigida com markdown adequado:
# 📘 Guia Completo de Manutenção, Atualização e Troubleshooting Windows

## Comandos Essenciais para SysAdmins e Analistas de Suporte

---

## 📑 Índice

- [1. Gerenciamento de Reinicialização Remota](#1-gerenciamento-de-reinicialização-remota)
- [2. Gerenciamento com Winget](#2-gerenciamento-com-winget)
- [3. Comandos de Saúde e Integridade do Sistema](#3-comandos-de-saúde-e-integridade-do-sistema)
- [4. Execução em Lote de Instaladores](#4-execução-em-lote-de-instaladores)
- [5. Consulta e Gerenciamento de Updates](#5-consulta-e-gerenciamento-de-updates)
- [6. Instalação Manual de Pacotes MSU](#6-instalação-manual-de-pacotes-msu)
- [7. Solução para Erro 0x800f0838](#7-solução-para-erro-0x800f0838-windows-11-24h2)
- [8. Ferramentas e Links Úteis](#8-ferramentas-e-links-úteis)
- [9. Instalação e Configuração do Winget](#9-instalação-e-configuração-do-winget)
- [10. Correção: "winget não reconhecido"](#10-correção-winget-não-reconhecido)
- [11. PsExec - Execução Remota Avançada](#11-psexec---execução-remota-avançada)
- [12. Comandos Avançados de Troubleshooting](#12-comandos-avançados-de-troubleshooting)
- [13. Gerenciamento de Usuários e Permissões](#13-gerenciamento-de-usuários-e-permissões)
- [14. Relatórios e Inventário](#14-relatórios-e-inventário)
- [15. Scripts de Automação Úteis](#15-scripts-de-automação-úteis)

---

# 1. Gerenciamento de Reinicialização Remota

## CMD/Batch

### Reinicialização imediata forçada
```
shutdown /r /m \\NOME_OU_IP /t 0 /f
```

### Reinicialização agendada (300 segundos = 5 minutos)
```
shutdown /r /m \\NOME_OU_IP /t 300 /c "Manutenção programada"
```

### Desligamento remoto
```
shutdown /s /m \\NOME_OU_IP /t 0 /f
```

### Cancelar reinicialização agendada
```
shutdown /a /m \\NOME_OU_IP
```

### Reiniciar múltiplas máquinas
```
for /f %i in (lista_pcs.txt) do shutdown /r /m \\%i /t 60 /f
```

## PowerShell
### Reinicialização simples
```
Restart-Computer -ComputerName NOME_DA_MAQUINA -Force
```
### Reiniciar múltiplas máquinas
```
$computers = Get-Content "C:\lista_pcs.txt"
Restart-Computer -ComputerName $computers -Force -WsmanAuthentication Default
```

### Reiniciar com credenciais alternativas
```
$cred = Get-Credential
Restart-Computer -ComputerName SERVIDOR01 -Credential $cred -Force
```

### Verificar uptime antes de reiniciar
```
Get-CimInstance -ClassName Win32_OperatingSystem -ComputerName NOME_PC | 
Select-Object CSName, LastBootUpTime, @{Name="Uptime";Expression={(Get-Date) - $_.LastBootUpTime}}
```

⬆️ Voltar ao topo

# 2. Gerenciamento com Winget

## Comandos Básicos

### Listar atualizações disponíveis
```
winget upgrade
```

### Atualizar pacote específico
```
winget upgrade --id Google.Chrome
```
### Atualizar todos os pacotes
```
winget upgrade --all
```
### Atualizar silenciosamente (sem interação)
```
winget upgrade --all --silent --accept-package-agreements --accept-source-agreements
```

### Buscar aplicativo
```
winget search "nome do app"
```
### Instalar aplicativo
```
winget install --id Microsoft.PowerToys --silent
```
### Desinstalar aplicativo
```
winget uninstall --id Adobe.Acrobat.Reader.64-bit
```
### Listar aplicativos instalados
```
winget list
```
### Exportar lista de aplicativos instalados
```
winget export -o C:\backup\apps.json
```
### Importar e instalar lista de aplicativos
```
winget import -i C:\backup\apps.json
```
## Atualização Remota com PsExec
### Atualização remota básica
```
psexec \\NOME_PC -s -i cmd.exe /c "winget upgrade --all --silent --accept-package-agreements --accept-source-agreements"
```
### Atualização em múltiplas máquinas
```
psexec @lista_pcs.txt -s -i cmd.exe /c "winget upgrade --all --silent --accept-package-agreements --accept-source-agreements"
```
### Com credenciais específicas
```
psexec \\NOME_PC -u DOMINIO\usuario -p senha -s cmd.exe /c "winget upgrade --all --silent"
```
### Verificar versão do winget remotamente
```
psexec \\NOME_PC -s cmd.exe /c "winget --version"
```
## Script PowerShell para Atualização em Massa
### Script para atualizar múltiplos computadores
```
$computers = Get-Content "C:\lista_pcs.txt"
$results = @()

foreach ($computer in $computers) {
    Write-Host "Atualizando $computer..." -ForegroundColor Cyan
    
    $result = Invoke-Command -ComputerName $computer -ScriptBlock {
        try {
            $output = winget upgrade --all --silent --accept-package-agreements --accept-source-agreements 2>&1
            return @{Status="Sucesso"; Output=$output}
        } catch {
            return @{Status="Erro"; Output=$_.Exception.Message}
        }
    } -ErrorAction SilentlyContinue
    
    $results += [PSCustomObject]@{
        Computer = $computer
        Status = $result.Status
        Details = $result.Output
    }
}

$results | Export-Csv "C:\logs\winget_update_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv" -NoTypeInformation
```

⬆️ Voltar ao topo

# 3. Comandos de Saúde e Integridade do Sistema

## DISM (Deployment Image Servicing and Management)
### Verificar integridade da imagem
```
DISM /Online /Cleanup-Image /CheckHealth
```

### Escanear integridade (mais detalhado)
```
DISM /Online /Cleanup-Image /ScanHealth
```
### Restaurar integridade da imagem
```
DISM /Online /Cleanup-Image /RestoreHealth
```
### Restaurar usando fonte alternativa (ISO montado)
```
DISM /Online /Cleanup-Image /RestoreHealth /Source:D:\sources\install.wim
```
### Limpar componentes antigos (liberar espaço)
```
DISM /Online /Cleanup-Image /StartComponentCleanup
```

### Limpar componentes com remoção de backups (irreversível)
```
DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase
```
### Analisar tamanho do component store
```
DISM /Online /Cleanup-Image /AnalyzeComponentStore
```
### Exportar log detalhado
```
DISM /Online /Cleanup-Image /RestoreHealth /LogPath:C:\logs\dism.log /LogLevel:4
```
## SFC (System File Checker)
### Verificar e reparar arquivos do sistema
```
sfc /scannow
```
### Verificar sem reparar
```
sfc /verifyonly
```
### Escanear arquivo específico
```
sfc /scanfile=C:\Windows\System32\kernel32.dll
```
### Verificar arquivos offline (Windows em outro disco)
```
sfc /scannow /offbootdir=D:\ /offwindir=D:\Windows
```
### Visualizar log do SFC
```
findstr /c:"[SR]" %windir%\Logs\CBS\CBS.log > C:\logs\sfcdetails.txt
```

## Sequência Recomendada para Problemas Graves
### 1. Executar DISM primeiro
```
DISM /Online /Cleanup-Image /RestoreHealth
```
### 2. Aguardar conclusão e executar SFC
```
sfc /scannow
```
### 3. Se persistir, executar em modo seguro
### Reiniciar em modo seguro: msconfig > Inicialização > Inicialização segura
### Depois executar novamente:
```
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```
### 4. Verificar integridade do disco
```
chkdsk C: /f /r /x
```
⬆️ Voltar ao topo

# 4. Execução em Lote de Instaladores

## Instaladores EXE
### Executar todos os EXE na pasta atual
```
for %f in (*.exe) do start /wait "%f" /S
```

### Com log individual
```
for %f in (*.exe) do (
    echo Instalando %f...
    start /wait "%f" /S /LOG="C:\logs\%~nf.log"
)
```

### Executar apenas se não estiver instalado (verificar registro)
```
for %f in (*.exe) do (
    reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall" /s | find /i "%~nf" || start /wait "%f" /S
)
```
## Instaladores MSI
### Instalar todos os MSI silenciosamente
```
for %f in (*.msi) do msiexec /i "%f" /qn /norestart
```
### Com log detalhado
```
for %f in (*.msi) do msiexec /i "%f" /qn /norestart /l*v "C:\logs\%~nf.log"
```
### Reparar instalações MSI
```
for %f in (*.msi) do msiexec /fa "%f" /qn
```
### Desinstalar MSI
```
for %f in (*.msi) do msiexec /x "%f" /qn /norestart
```

## Instaladores MSU (Windows Updates)
### Instalar todos os MSU
```
for %f in (*.msu) do wusa "%f" /quiet /norestart
```
### Com log
```
for %f in (*.msu) do wusa "%f" /quiet /norestart /log:"C:\logs\%~nf.log"
```
### Forçar instalação
```
for %f in (*.msu) do dism /online /add-package /packagepath:"%f" /quiet /norestart
```

## Script PowerShell Avançado para Instalação em Massa
### Script completo com verificação e log
```
$installerPath = "C:\Instaladores"
$logPath = "C:\logs\instalacao_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

function Write-Log {
    param($Message)
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "$timestamp - $Message" | Tee-Object -FilePath $logPath -Append
}
```

### Processar EXE
```
Get-ChildItem "$installerPath\*.exe" | ForEach-Object {
    Write-Log "Iniciando instalação: $($_.Name)"
    $process = Start-Process -FilePath $_.FullName -ArgumentList "/S" -Wait -PassThru
    if ($process.ExitCode -eq 0) {
        Write-Log "Sucesso: $($_.Name)"
    } else {
        Write-Log "Erro: $($_.Name) - Exit Code: $($process.ExitCode)"
    }
}
```

### Processar MSI
```
Get-ChildItem "$installerPath\*.msi" | ForEach-Object {
    Write-Log "Iniciando instalação: $($_.Name)"
    $arguments = "/i `"$($_.FullName)`" /qn /norestart /l*v `"$logPath`""
    $process = Start-Process "msiexec.exe" -ArgumentList $arguments -Wait -PassThru
    if ($process.ExitCode -eq 0) {
        Write-Log "Sucesso: $($_.Name)"
    } else {
        Write-Log "Erro: $($_.Name) - Exit Code: $($process.ExitCode)"
    }
}
```

### Processar MSU
```
Get-ChildItem "$installerPath\*.msu" | ForEach-Object {
    Write-Log "Iniciando instalação: $($_.Name)"
    $process = Start-Process "wusa.exe" -ArgumentList "`"$($_.FullName)`" /quiet /norestart" -Wait -PassThru
    if ($process.ExitCode -eq 0) {
        Write-Log "Sucesso: $($_.Name)"
    } else {
        Write-Log "Erro: $($_.Name) - Exit Code: $($process.ExitCode)"
    }
}

Write-Log "Processo de instalação concluído"
```
⬆️ Voltar ao topo

# 5. Consulta e Gerenciamento de Updates
## WMIC (Windows Management Instrumentation Command)
### Listar todas as atualizações instaladas
```
wmic qfe list brief /format:table
```
### Listar com mais detalhes
```
wmic qfe list full
```
### Buscar atualização específica
```
wmic qfe where "HotFixID='KB5000001'" list full
```
### Exportar para CSV
```
wmic qfe list brief /format:csv > C:\logs\updates.csv
```
### Listar atualizações por data
```
wmic qfe where "InstalledOn>'12/01/2025'" list brief
```
### Reinstalar WMIC no Windows 11 (se removido)
```
DISM /Online /Add-Capability /CapabilityName:WMIC~~~~
```

## PowerShell - Gerenciamento Avançado de Updates
### Listar todas as atualizações instaladas
```
Get-HotFix | Sort-Object InstalledOn -Descending | Format-Table -AutoSize
```
### Buscar atualização específica
```
Get-HotFix -Id KB5000001
```
### Listar atualizações dos últimos 30 dias
```
Get-HotFix | Where-Object {$_.InstalledOn -gt (Get-Date).AddDays(-30)}
```
### Exportar para CSV
```
Get-HotFix | Export-Csv "C:\logs\updates_$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation
```

### Verificar atualizações em computadores remotos
```
$computers = Get-Content "C:\lista_pcs.txt"
$results = @()

foreach ($computer in $computers) {
    try {
        $updates = Get-HotFix -ComputerName $computer | 
                   Select-Object @{N='Computer';E={$computer}}, Description, HotFixID, InstalledOn
        $results += $updates
    } catch {
        Write-Warning "Erro ao acessar $computer : $_"
    }
}

$results | Export-Csv "C:\logs\updates_remotos.csv" -NoTypeInformation
```

### Verificar última atualização instalada
```
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 1
```
### Contar atualizações por tipo
```
Get-HotFix | Group-Object Description | Select-Object Name, Count
```
## Módulo PSWindowsUpdate (Avançado)
### Instalar módulo (executar como administrador)
```
Install-Module PSWindowsUpdate -Force
```
### Importar módulo
```
Import-Module PSWindowsUpdate
```
### Listar atualizações disponíveis
```
Get-WindowsUpdate
```
### Listar apenas atualizações críticas
```
Get-WindowsUpdate -Category "Critical Updates"
```
### Baixar atualizações sem instalar
```
Get-WindowsUpdate -Download
```
### Instalar todas as atualizações
```
Install-WindowsUpdate -AcceptAll -AutoReboot
```
### Instalar sem reiniciar
```
Install-WindowsUpdate -AcceptAll -IgnoreReboot
```
### Instalar atualizações específicas
```
Get-WindowsUpdate -KBArticleID KB5000001 -Install
```
### Ocultar atualização
```
Hide-WindowsUpdate -KBArticleID KB5000001
```
### Exibir atualizações ocultas
```
Get-WindowsUpdate -IsHidden
```
### Remover atualização
```
Remove-WindowsUpdate -KBArticleID KB5000001
```
### Histórico de atualizações
```
Get-WUHistory | Select-Object Date, Title, Result | Format-Table -AutoSize
```
### Agendar instalação de atualizações
```
Install-WindowsUpdate -AcceptAll -ScheduleJob (Get-Date).AddHours(2)
```
### Atualização remota
```
Invoke-WUJob -ComputerName NOME_PC -Script {Install-WindowsUpdate -AcceptAll -AutoReboot} -Confirm:$false
```

⬆️ Voltar ao topo

# 6. Instalação Manual de Pacotes MSU
## Métodos de Instalação
### Método 1: WUSA (Windows Update Standalone Installer)
```
wusa.exe C:\caminho\update.msu /quiet /norestart
```
### Com log
```
wusa.exe C:\caminho\update.msu /quiet /norestart /log:C:\logs\update.log
```
### Método 2: DISM (mais confiável)
```
dism /online /add-package /packagepath:"C:\caminho\update.msu" /quiet /norestart
```
### Método 3: PowerShell
```
Add-WindowsPackage -Online -PackagePath "C:\caminho\update.msu" -NoRestart
```
### Desinstalar MSU
```
wusa.exe /uninstall /kb:5000001 /quiet /norestart
```
### Verificar se MSU está instalado
```
dism /online /get-packages | findstr "KB5000001"
```
### Script para Instalação em Lote de MSU
```
$msuPath = "C:\Updates"
$logPath = "C:\logs\msu_install_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

Get-ChildItem "$msuPath\*.msu" | ForEach-Object {
    $kbNumber = $_.Name -replace '.*?(KB\d+).*', '$1'
    
    Write-Host "Verificando $kbNumber..." -ForegroundColor Cyan
    
    # Verificar se já está instalado
    $installed = Get-HotFix | Where-Object {$_.HotFixID -eq $kbNumber}
    
    if ($installed) {
        Write-Host "$kbNumber já está instalado" -ForegroundColor Green
        "$kbNumber - Já instalado" | Out-File $logPath -Append
    } else {
        Write-Host "Instalando $kbNumber..." -ForegroundColor Yellow
        try {
            Add-WindowsPackage -Online -PackagePath $_.FullName -NoRestart -ErrorAction Stop
            Write-Host "$kbNumber instalado com sucesso" -ForegroundColor Green
            "$kbNumber - Instalado com sucesso" | Out-File $logPath -Append
        } catch {
            Write-Host "Erro ao instalar $kbNumber : $_" -ForegroundColor Red
            "$kbNumber - Erro: $_" | Out-File $logPath -Append
        }
    }
}

Write-Host "`nLog salvo em: $logPath" -ForegroundColor Cyan
```
⬆️ Voltar ao topo

# 7. Solução para Erro 0x800f0838 (Windows 11 24H2)
## Contexto do Problema
O erro 0x800f0838 ocorre frequentemente no Windows 11 24H2 quando há dependências não satisfeitas entre atualizações. Algumas KBs exigem que atualizações anteriores estejam instaladas primeiro.
Método 1: Instalação Manual com Dependências

### 1. Criar pasta para os pacotes
```
New-Item -Path "C:\Packages" -ItemType Directory -Force
```
## 2. Baixar as atualizações necessárias:
##    - Atualização base mencionada no KB (ex: KB5043080)
##    - Atualização desejada (ex: KB5050009)

### 3. Navegar até a pasta
```
Set-Location C:\Packages
```
### 4. Instalar a atualização base primeiro
```
Add-WindowsPackage -Online -PackagePath "C:\Packages\windows11.0-kb5043080-x64.msu" -NoRestart
```
### 5. Instalar a atualização desejada
```
Add-WindowsPackage -Online -PackagePath "C:\Packages\windows11.0-kb5050009-x64.msu" -NoRestart
```
### 6. Verificar instalação
```
Get-HotFix -Id KB5050009
```
## Método 2: Script Automatizado para Resolver Dependências
### Script para resolver dependências automaticamente
```
param(
    [string]$UpdatePath = "C:\Packages"
)

function Install-UpdateWithDependencies {
    param([string]$PackagePath)
    
    $kbNumber = [regex]::Match($PackagePath, 'KB\d+').Value
    
    Write-Host "Tentando instalar $kbNumber..." -ForegroundColor Cyan
    
    try {
        # Tentar instalação direta
        Add-WindowsPackage -Online -PackagePath $PackagePath -NoRestart -ErrorAction Stop
        Write-Host "$kbNumber instalado com sucesso!" -ForegroundColor Green
        return $true
    } catch {
        $errorCode = $_.Exception.HResult
        
        if ($errorCode -eq 0x800f0838) {
            Write-Host "Erro 0x800f0838 detectado. Verificando dependências..." -ForegroundColor Yellow
            
            # Extrair informações sobre dependências
            $cabPath = $PackagePath -replace '\.msu$', '.cab'
            
            # Tentar com DISM
            Write-Host "Tentando instalação com DISM..." -ForegroundColor Yellow
            $dismResult = dism /online /add-package /packagepath:"$PackagePath" /quiet /norestart
            
            if ($LASTEXITCODE -eq 0) {
                Write-Host "$kbNumber instalado via DISM!" -ForegroundColor Green
                return $true
            } else {
                Write-Host "Falha na instalação. Código de erro: $LASTEXITCODE" -ForegroundColor Red
                Write-Host "Verifique se há atualizações pendentes ou instale manualmente as dependências." -ForegroundColor Yellow
                return $false
            }
        } else {
            Write-Host "Erro desconhecido: $_" -ForegroundColor Red
            return $false
        }
    }
}
```
### Processar todos os MSU na pasta
```
$updates = Get-ChildItem "$UpdatePath\*.msu" | Sort-Object Name

foreach ($update in $updates) {
    Install-UpdateWithDependencies -PackagePath $update.FullName
    Start-Sleep -Seconds 5
}

Write-Host "`nProcesso concluído. Reinicie o computador se necessário." -ForegroundColor Cyan
```
## Troubleshooting Adicional
### Limpar cache do Windows Update
```
net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver

ren C:\Windows\SoftwareDistribution SoftwareDistribution.old
ren C:\Windows\System32\catroot2 catroot2.old

net start wuauserv
net start cryptSvc
net start bits
net start msiserver
```

### Resetar componentes do Windows Update
```
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```

### Verificar logs de erro
```
Get-WindowsUpdateLog
notepad C:\Windows\WindowsUpdate.log
```
⬆️ Voltar ao topo

# 8. Ferramentas e Links Úteis
## CISBOX (Lear)
```
URL: https://cisbox.lear.com/?lang=pt-br&plant=1105
Descrição: Portal interno para gestão de TI
```

## Microsoft Update Catalog

```
URL: https://www.catalog.update.microsoft.com/
Descrição: Repositório oficial de atualizações Microsoft
```

## Sysinternals Suite
```
URL: https://docs.microsoft.com/sysinternals/
Ferramentas essenciais:
```

## PsExec: Execução remota
## PsInfo: Informações do sistema
## Autoruns: Gerenciar inicialização
## Process Explorer: Monitor avançado de processos
## TCPView: Monitor de conexões de rede



## Comandos para Download de Ferramentas
### Baixar Sysinternals Suite
```
$url = "https://download.sysinternals.com/files/SysinternalsSuite.zip"
$output = "C:\Tools\SysinternalsSuite.zip"
Invoke-WebRequest -Uri $url -OutFile $output
Expand-Archive -Path $output -DestinationPath "C:\Tools\Sysinternals"
```
## Adicionar ao PATH
```
$env:Path += ";C:\Tools\Sysinternals"
[Environment]::SetEnvironmentVariable("Path", $env:Path, [System.EnvironmentVariableTarget]::Machine)
```
⬆️ Voltar ao topo

# 9. Instalação e Configuração do Winget
## Opção 1: Via Microsoft Store
### Abrir Microsoft Store na página do App Installer
```
Start-Process "ms-windows-store://pdp/?ProductId=9NBLGGH4NNS1"
```
### Verificar instalação
```
winget --version
```
### Atualizar App Installer
```
Get-AppxPackage -Name Microsoft.DesktopAppInstaller | ForEach-Object {
    Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"
}
```
## Opção 2: Instalação Offline (GitHub)
### Baixar última versão do GitHub
```
$repo = "microsoft/winget-cli"
$releases = "https://api.github.com/repos/$repo/releases/latest"
$downloadUrl = (Invoke-RestMethod -Uri $releases).assets | 
               Where-Object {$_.name -like "*.msixbundle"} | 
               Select-Object -ExpandProperty browser_download_url

$output = "$env:TEMP\Microsoft.DesktopAppInstaller.msixbundle"
Invoke-WebRequest -Uri $downloadUrl -OutFile $output
```
### Instalar
```
Add-AppxPackage -Path $output
```
### Verificar
```
winget --version
```
## Opção 3: Script de Instalação Automatizada
### Script completo para instalação do Winget
```
function Install-Winget {
    Write-Host "Verificando instalação do Winget..." -ForegroundColor Cyan
    
    # Verificar se já está instalado
    try {
        $version = winget --version
        Write-Host "Winget já está instalado: $version" -ForegroundColor Green
        return
    } catch {
        Write-Host "Winget não encontrado. Iniciando instalação..." -ForegroundColor Yellow
    }
    
    # Baixar dependências
    Write-Host "Baixando dependências..." -ForegroundColor Cyan
    
    # VCLibs
    $vcLibsUrl = "https://aka.ms/Microsoft.VCLibs.x64.14.00.Desktop.appx"
    $vcLibsPath = "$env:TEMP\Microsoft.VCLibs.x64.14.00.Desktop.appx"
    Invoke-WebRequest -Uri $vcLibsUrl -OutFile $vcLibsPath
    Add-AppxPackage -Path $vcLibsPath
    
    # UI.Xaml
    $uiXamlUrl = "https://github.com/microsoft/microsoft-ui-xaml/releases/download/v2.8.6/Microsoft.UI.Xaml.2.8.x64.appx"
    $uiXamlPath = "$env:TEMP\Microsoft.UI.Xaml.2.8.x64.appx"
    Invoke-WebRequest -Uri $uiXamlUrl -OutFile $uiXamlPath
    Add-AppxPackage -Path $uiXamlPath
    
    # Winget
    Write-Host "Baixando Winget..." -ForegroundColor Cyan
    $repo = "microsoft/winget-cli"
    $releases = "https://api.github.com/repos/$repo/releases/latest"
    $downloadUrl = (Invoke-RestMethod -Uri $releases).assets | 
                   Where-Object {$_.name -like "*.msixbundle"} | 
                   Select-Object -ExpandProperty browser_download_url
    
    $wingetPath = "$env:TEMP\Microsoft.DesktopAppInstaller.msixbundle"
    Invoke-WebRequest -Uri $downloadUrl -OutFile $wingetPath
    Add-AppxPackage -Path $wingetPath
    
    # Verificar instalação
    Start-Sleep -Seconds 5
    $version = winget --version
    Write-Host "Winget instalado com sucesso: $version" -ForegroundColor Green
    
    # Limpar arquivos temporários
    Remove-Item $vcLibsPath, $uiXamlPath, $wingetPath -Force
}
```
## Executar instalação
### Install-Winget

Configuração Inicial do Winget

### Aceitar termos de uso
```
winget list
```
### Configurar fontes
```
winget source list
winget source update
winget source reset --force
```
### Configurações avançadas (arquivo settings.json)
```
$settingsPath = "$env:LOCALAPPDATA\Packages\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\LocalState\settings.json"

$settings = @{
    "visual" = @{
        "progressBar" = "rainbow"
    }
    "experimentalFeatures" = @{
        "experimentalCmd" = $true
        "experimentalArg" = $true
    }
    "installBehavior" = @{
        "preferences" = @{
            "scope" = "machine"
        }
    }
} | ConvertTo-Json -Depth 10

$settings | Out-File $settingsPath -Encoding utf8

Write-Host "Configurações aplicadas com sucesso!" -ForegroundColor Green
```
⬆️ Voltar ao topo

# 10. Correção: "winget não reconhecido"
## Diagnóstico
### Verificar se o App Installer está instalado
```
Get-AppxPackage -Name Microsoft.DesktopAppInstaller
```
### Verificar PATH
```
$env:Path -split ';' | Where-Object {$_ -like "*WindowsApps*"}
```
### Verificar executável
```
Get-Command winget -ErrorAction SilentlyContinue
```
## Solução 1: Reparar App Installer
### Método 1: Re-registrar o pacote
```
Get-AppxPackage -Name Microsoft.DesktopAppInstaller | ForEach-Object {
    Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"
}
```
### Método 2: Reinstalar
```
Get-AppxPackage -Name Microsoft.DesktopAppInstaller | Remove-AppxPackage
Add-AppxPackage -RegisterByFamilyName -MainPackage Microsoft.DesktopAppInstaller_8wekyb3d8bbwe
```
### Método 3: Reset completo
```
Get-AppxPackage -Name Microsoft.DesktopAppInstaller | Reset-AppxPackage
```
## Solução 2: Adicionar ao PATH Manualmente
### Localizar winget.exe
```
$wingetPath = Get-ChildItem -Path "$env:LOCALAPPDATA\Microsoft\WindowsApps" -Filter "winget.exe" -Recurse -ErrorAction SilentlyContinue | Select-Object -First 1

if ($wingetPath) {
    $wingetDir = $wingetPath.DirectoryName
    
    # Adicionar ao PATH da sessão atual
    $env:Path += ";$wingetDir"
    
    # Adicionar permanentemente
    $currentPath = [Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::User)
    if ($currentPath -notlike "*$wingetDir*") {
        [Environment]::SetEnvironmentVariable("Path", "$currentPath;$wingetDir", [System.EnvironmentVariableTarget]::User)
        Write-Host "PATH atualizado. Reinicie o terminal." -ForegroundColor Green
    }
} else {
    Write-Host "winget.exe não encontrado. Reinstale o App Installer." -ForegroundColor Red
}
```
## Solução 3: Reinstalação Completa
### Script de reinstalação completa
```
function Repair-Winget {
    Write-Host "Iniciando reparo do Winget..." -ForegroundColor Cyan
    
    # Remover instalação atual
    Write-Host "Removendo instalação atual..." -ForegroundColor Yellow
    Get-AppxPackage -Name Microsoft.DesktopAppInstaller | Remove-AppxPackage -ErrorAction SilentlyContinue
    
    # Limpar cache
    $cachePath = "$env:LOCALAPPDATA\Packages\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe\LocalCache"
    if (Test-Path $cachePath) {
        Remove-Item $cachePath -Recurse -Force
    }
    
    # Reinstalar
    Write-Host "Reinstalando Winget..." -ForegroundColor Cyan
    
    # Baixar última versão
    $repo = "microsoft/winget-cli"
    $releases = "https://api.github.com/repos/$repo/releases/latest"
    $downloadUrl = (Invoke-RestMethod -Uri $releases).assets | 
                   Where-Object {$_.name -like "*.msixbundle"} | 
                   Select-Object -ExpandProperty browser_download_url
    
    $output = "$env:TEMP\Microsoft.DesktopAppInstaller.msixbundle"
    Invoke-WebRequest -Uri $downloadUrl -OutFile $output
    Add-AppxPackage -Path $output
    
    # Verificar
    Start-Sleep -Seconds 5
    try {
        $version = winget --version
        Write-Host "Winget reparado com sucesso: $version" -ForegroundColor Green
    } catch {
        Write-Host "Erro ao reparar Winget. Tente instalação manual." -ForegroundColor Red
    }
    
    # Limpar
    Remove-Item $output -Force
}

# Executar reparo
Repair-Winget
```
⬆️ Voltar ao topo

# 11. PsExec - Execução Remota Avançada
## Instalação do PsExec
### Baixar Sysinternals Suite
```
$url = "https://download.sysinternals.com/files/PSTools.zip"
$output = "C:\Tools\PSTools.zip"
New-Item -Path "C:\Tools" -ItemType Directory -Force
Invoke-WebRequest -Uri $url -OutFile $output
Expand-Archive -Path $output -DestinationPath "C:\Tools\PSTools" -Force

# Adicionar ao PATH
$env:Path += ";C:\Tools\PSTools"
[Environment]::SetEnvironmentVariable("Path", $env:Path, [System.EnvironmentVariableTarget]::Machine)

# Aceitar EULA automaticamente
psexec -accepteula
```
## Comandos Básicos do PsExec
### Abrir CMD remoto como SYSTEM
```
psexec \\COMPUTADOR -s cmd
```
### Abrir CMD remoto com privilégios elevados
```
psexec \\COMPUTADOR -h -s cmd
```
### Executar comando específico
```
psexec \\COMPUTADOR -s cmd /c "comando"
```
### Com credenciais específicas
```
psexec \\COMPUTADOR -u DOMINIO\usuario -p senha -s cmd
```
### Executar em múltiplos computadores
```
psexec \\COMP1,COMP2,COMP3 -s cmd /c "comando"
```
### Executar usando arquivo de lista
```
psexec @C:\lista_pcs.txt -s cmd /c "comando"
```
### Copiar arquivo antes de executar
```
psexec \\COMPUTADOR -c script.bat
```
### Executar interativamente
```
psexec \\COMPUTADOR -i -s cmd
```
### Executar com timeout
```
psexec \\COMPUTADOR -n 30 cmd /c "comando"
```
## Exemplos Práticos
## Instalação Remota de MSU

### Copiar MSU para máquina remota
```
copy "C:\Updates\KB5000001.msu" "\\COMPUTADOR\C$\Temp\"
```
### Executar instalação remotamente
```
psexec \\COMPUTADOR -s cmd /c "wusa.exe C:\Temp\KB5000001.msu /quiet /norestart"
```
### Verificar instalação
```
psexec \\COMPUTADOR -s cmd /c "wmic qfe where HotFixID='KB5000001' list brief"
```
## Acesso ao PowerShell Remoto via PsExec
### Abrir CMD remoto como SYSTEM
```
psexec \\COMPUTADOR -h -s cmd
```
### Dentro do CMD remoto, iniciar PowerShell
```
powershell
```
### Configurar ExecutionPolicy temporariamente
```
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
```
### Navegar e executar script
```
cd C:\Temp
.\SeuScript.ps1 -Remediate -FixSoftwareCenter -Diagnose
```

## Script PowerShell para Execução Remota em Massa
### Script para executar comandos em múltiplos computadores
```
$computers = Get-Content "C:\lista_pcs.txt"
$command = "winget upgrade --all --silent"
$results = @()

foreach ($computer in $computers) {
    Write-Host "Executando em $computer..." -ForegroundColor Cyan
    
    try {
        # Executar comando
        $output = psexec \\$computer -s cmd /c $command 2>&1
        
        $results += [PSCustomObject]@{
            Computer = $computer
            Status = "Sucesso"
            Output = $output -join "`n"
        }
        
        Write-Host "Concluído: $computer" -ForegroundColor Green
    } catch {
        $results += [PSCustomObject]@{
            Computer = $computer
            Status = "Erro"
            Output = $_.Exception.Message
        }
        
        Write-Host "Erro em $computer : $_" -ForegroundColor Red
    }
}
```
### Exportar resultados
```
$re```sults | Export-Csv "C:\logs\psexec_results_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv" -NoTypeInformation
```
## Exibir resumo
```
$successCount = ($results | Where-Object {$_.Status -eq "Sucesso"}).Count
$errorCount = ($results | Where-Object {$_.Status -eq "Erro"}).Count

Write-Host "`n=== RESUMO ===" -ForegroundColor Cyan
Write-Host "Sucesso: $successCount" -ForegroundColor Green
Write-Host "Erros: $errorCount" -ForegroundColor Red
```

## Troubleshooting PsExec
```
Erro: "Access is denied"
```
### Verificar compartilhamento administrativo
```
net use \\COMPUTADOR\C$ /user:DOMINIO\usuario senha
```
### Habilitar compartilhamento administrativo (executar no computador remoto)
```
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```
### Verificar firewall
```
netsh advfirewall firewall set rule group="File and Printer Sharing" new enable=Yes
```

## Erro: "The network path was not found"
### Verificar conectividade
```
ping COMPUTADOR
```
### Verificar serviço Server
```
sc \\COMPUTADOR query lanmanserver
```
### Iniciar serviço se necessário
```
sc \\COMPUTADOR start lanmanserver
```
⬆️ Voltar ao topo

# 12. Comandos Avançados de Troubleshooting

## Windows Update - Forçar Detecção e Instalação
### Parar serviços do Windows Update
```
net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver
```
### Renomear pastas de cache
```
ren C:\Windows\SoftwareDistribution SoftwareDistribution.old
ren C:\Windows\System32\catroot2 catroot2.old
```
## Reiniciar serviços
```
net start wuauserv
net start cryptSvc
net start bits
net start msiserver
```
### Forçar detecção
```
wuauclt /detectnow
wuauclt /updatenow
```
### Alternativa moderna (Windows 10/11)
```
usoclient StartScan
usoclient StartDownload
usoclient StartInstall
```
## Limpeza e Otimização do Sistema
### Limpeza de disco via linha de comando
```
cleanmgr /sagerun:1
```
## Limpar arquivos temporários
```
del /q /f /s %TEMP%\*
del /q /f /s C:\Windows\Temp\*
```
## Limpar cache do Windows Update
```
del /q /f /s C:\Windows\SoftwareDistribution\Download\*
```
### Limpar cache de instaladores
```
del /q /f /s C:\Windows\Installer\$PatchCache$\*
```
### Limpar logs antigos
```
forfiles /p C:\Windows\Logs /s /m *.log /d -30 /c "cmd /c del @path"
```
### Limpar pontos de restauração antigos (manter apenas o mais recente)
```
vssadmin delete shadows /for=C: /oldest /quiet
```
## PowerShell - Limpeza Avançada
### Script de limpeza completo
```
function Clear-SystemCache {
    Write-Host "Iniciando limpeza do sistema..." -ForegroundColor Cyan
    
    # Limpar arquivos temporários
    Write-Host "Limpando arquivos temporários..." -ForegroundColor Yellow
    Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
    Remove-Item "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue
    
    # Limpar cache do Windows Update
    Write-Host "Limpando cache do Windows Update..." -ForegroundColor Yellow
    Stop-Service wuauserv -Force
    Remove-Item "C:\Windows\SoftwareDistribution\Download\*" -Recurse -Force -ErrorAction SilentlyContinue
    Start-Service wuauserv
    
    # Limpar cache de navegadores
    Write-Host "Limpando cache de navegadores..." -ForegroundColor Yellow
    
    # Chrome
    $chromePath = "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cache"
    if (Test-Path $chromePath) {
        Remove-Item "$chromePath\*" -Recurse -Force -ErrorAction SilentlyContinue
    }
    
    # Edge
    $edgePath = "$env:LOCALAPPDATA\Microsoft\Edge\User Data\Default\Cache"
    if (Test-Path $edgePath) {
        Remove-Item "$edgePath\*" -Recurse -Force -ErrorAction SilentlyContinue
    }
    
    # Firefox
    $firefoxPath = "$env:APPDATA\Mozilla\Firefox\Profiles"
    if (Test-Path $firefoxPath) {
        Get-ChildItem $firefoxPath -Directory | ForEach-Object {
            Remove-Item "$($_.FullName)\cache2\*" -Recurse -Force -ErrorAction SilentlyContinue
        }
    }
    
    # Limpar lixeira
    Write-Host "Esvaziando lixeira..." -ForegroundColor Yellow
    Clear-RecycleBin -Force -ErrorAction SilentlyContinue
    
    # Executar limpeza de disco
    Write-Host "Executando limpeza de disco..." -ForegroundColor Yellow
    Start-Process cleanmgr -ArgumentList "/sagerun:1" -Wait
    
    # Limpar componentes do Windows
    Write-Host "Limpando componentes do Windows..." -ForegroundColor Yellow
    DISM /Online /Cleanup-Image /StartComponentCleanup /ResetBase
    
    Write-Host "Limpeza concluída!" -ForegroundColor Green
    
    # Calcular espaço liberado
    $disk = Get-PSDrive C
    Write-Host "Espaço livre em C: $([math]::Round($disk.Free/1GB, 2)) GB" -ForegroundColor Cyan
}
```
### Executar limpeza
```
Clear-SystemCache
```
## Diagnóstico de Rede
### Resetar configurações de rede
```
netsh winsock reset
netsh int ip reset
ipconfig /release
ipconfig /renew
ipconfig /flushdns
```
### Verificar conectividade
```
ping 8.8.8.8
ping google.com
tracert google.com
```
### Verificar portas abertas
```
netstat -ano | findstr LISTENING
```
### Testar DNS
```
nslookup google.com
nslookup google.com 8.8.8.8
```
### Verificar tabela de roteamento
```
route print
```
### Adicionar rota estática
```
route add 192.168.1.0 mask 255.255.255.0 192.168.0.1
```
### Verificar adaptadores de rede
```
ipconfig /all
```
## Gerenciamento de Serviços
### Listar todos os serviços
```
sc query type= service state= all
```
### Verificar status de serviço específico
```
sc query wuauserv
```
### Iniciar serviço
```
sc start wuauserv
```
### Parar serviço
```
sc stop wuauserv
```
### Configurar inicialização automática
```
sc config wuauserv start= auto
```
### Configurar inicialização manual
```
sc config wuauserv start= demand
```
### Desabilitar serviço
```
sc config wuauserv start= disabled
```
### Reiniciar serviço
```
net stop wuauserv && net start wuauserv
```
## PowerShell - Gerenciamento de Serviços
### Listar todos os serviços
```
Get-Service | Sort-Object Status, DisplayName | Format-Table -AutoSize
```
### Serviços em execução
```
Get-Service | Where-Object {$_.Status -eq "Running"}
```
### Serviços parados
```
Get-Service | Where-Object {$_.Status -eq "Stopped"}
```
### Buscar serviço específico
```
Get-Service | Where-Object {$_.DisplayName -like "*Windows Update*"}
```
### Iniciar serviço
```
Start-Service -Name wuauserv
```
### Parar serviço
```
Stop-Service -Name wuauserv -Force
```
### Reiniciar serviço
```
Restart-Service -Name wuauserv
```
### Configurar inicialização
```
Set-Service -Name wuauserv -StartupType Automatic
```
### Verificar dependências
```
Get-Service -Name wuauserv -DependentServices
Get-Service -Name wuauserv -RequiredServices
```
### Exportar lista de serviços
```
Get-Service | Export-Csv "C:\logs\services_$(Get-Date -Format 'yyyyMMdd').csv" -NoTypeInformation
```
### Script para reiniciar serviços críticos
```
$services = @("wuauserv", "bits", "cryptsvc", "msiserver")

foreach ($service in $services) {
    Write-Host "Reiniciando $service..." -ForegroundColor Cyan
    try {
        Restart-Service -Name $service -Force -ErrorAction Stop
        Write-Host "$service reiniciado com sucesso" -ForegroundColor Green
    } catch {
        Write-Host "Erro ao reiniciar $service : $_" -ForegroundColor Red
    }
}
```
⬆️ Voltar ao topo

# 13. Gerenciamento de Usuários e Permissões
## Comandos Básicos de Usuários
### Listar usuários locais
```
net user
```
### Informações de usuário específico
```
net user nome_usuario
```
### Criar novo usuário
```
net user novo_usuario senha123 /add
```
### Adicionar usuário ao grupo Administradores
```
net localgroup Administradores novo_usuario /add
```
### Remover usuário
```
net user nome_usuario /delete
```
### Alterar senha
```
net user nome_usuario nova_senha
```
### Desabilitar usuário
```
net user nome_usuario /active:no
```
### Habilitar usuário
```
net user nome_usuario /active:yes
```
### Listar grupos locais
```
net localgroup
```
### Membros de um grupo
```
net localgroup Administradores
```
## PowerShell - Gerenciamento Avançado
### Listar usuários locais
```
Get-LocalUser
```
### Criar novo usuário
```
$password = ConvertTo-SecureString "Senha123!" -AsPlainText -Force
New-LocalUser -Name "NovoUsuario" -Password $password -FullName "Nome Completo" -Description "Descrição"
```
## Adicionar ao grupo Administradores
```
Add-LocalGroupMember -Group "Administradores" -Member "NovoUsuario"
```
## Remover usuário
```
Remove-LocalUser -Name "NovoUsuario"
```
## Alterar senha
```
$password = ConvertTo-SecureString "NovaSenha123!" -AsPlainText -Force
Set-LocalUser -Name "Usuario" -Password $password
```
### Desabilitar usuário
```
Disable-LocalUser -Name "Usuario"
```
## Habilitar usuário
```
Enable-LocalUser -Name "Usuario"
```
## Verificar último logon
```
Get-LocalUser | Select-Object Name, Enabled, LastLogon | Format-Table -AutoSize
```
## Listar grupos e membros
```
Get-LocalGroup | ForEach-Object {
    Write-Host "`nGrupo: $($_.Name)" -ForegroundColor Cyan
    Get-LocalGroupMember -Group $_.Name | Select-Object Name, ObjectClass | Format-Table -AutoSize
}
```
⬆️ Voltar ao topo

# 14. Relatórios e Inventário
## Script de Inventário Completo
### Script para gerar inventário detalhado do sistema
```
function Get-SystemInventory {
    param(
        [string]$OutputPath = "C:\Inventario"
    )
    
    # Criar pasta de saída
    New-Item -Path $OutputPath -ItemType Directory -Force | Out-Null
    $timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
    $computerName = $env:COMPUTERNAME
    
    Write-Host "Gerando inventário para $computerName..." -ForegroundColor Cyan
    
    # Informações do sistema
    Write-Host "Coletando informações do sistema..." -ForegroundColor Yellow
    $os = Get-CimInstance Win32_OperatingSystem
    $cs = Get-CimInstance Win32_ComputerSystem
    $bios = Get-CimInstance Win32_BIOS
    
    $systemInfo = [PSCustomObject]@{
        ComputerName = $computerName
        Manufacturer = $cs.Manufacturer
        Model = $cs.Model
        SerialNumber = $bios.SerialNumber
        OS = $os.Caption
        OSVersion = $os.Version
        OSBuild = $os.BuildNumber
        InstallDate = $os.InstallDate
        LastBootTime = $os.LastBootUpTime
        TotalMemoryGB = [math]::Round($cs.TotalPhysicalMemory / 1GB, 2)
        Domain = $cs.Domain
        Workgroup = $cs.Workgroup
    }
    
    $systemInfo | Export-Csv "$OutputPath\${computerName}_SystemInfo_$timestamp.csv" -NoTypeInformation
    
    # Hardware
    Write-Host "Coletando informações de hardware..." -ForegroundColor Yellow
    
    # CPU
    $cpu = Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, NumberOfLogicalProcessors, MaxClockSpeed
    $cpu | Export-Csv "$OutputPath\${computerName}_CPU_$timestamp.csv" -NoTypeInformation
    
    # Memória
    $memory = Get-CimInstance Win32_PhysicalMemory | Select-Object Manufacturer, PartNumber, SerialNumber, @{N='CapacityGB';E={$_.Capacity/1GB}}, Speed
    $memory | Export-Csv "$OutputPath\${computerName}_Memory_$timestamp.csv" -NoTypeInformation
    
    # Discos
    $disks = Get-CimInstance Win32_DiskDrive | Select-Object Model, SerialNumber, @{N='SizeGB';E={[math]::Round($_.Size/1GB,2)}}, InterfaceType
    $disks | Export-Csv "$OutputPath\${computerName}_Disks_$timestamp.csv" -NoTypeInformation
    
    # Partições
    $partitions = Get-PSDrive -PSProvider FileSystem | Where-Object {$_.Used -gt 0} | 
        Select-Object Name, @{N='UsedGB';E={[math]::Round($_.Used/1GB,2)}}, @{N='FreeGB';E={[math]::Round($_.Free/1GB,2)}}, @{N='TotalGB';E={[math]::Round(($_.Used+$_.Free)/1GB,2)}}
    $partitions | Export-Csv "$OutputPath\${computerName}_Partitions_$timestamp.csv" -NoTypeInformation
    
    # Rede
    Write-Host "Coletando informações de rede..." -ForegroundColor Yellow
    $network = Get-NetAdapter | Where-Object {$_.Status -eq "Up"} | 
        Select-Object Name, InterfaceDescription, MacAddress, LinkSpeed, Status
    $network | Export-Csv "$OutputPath\${computerName}_Network_$timestamp.csv" -NoTypeInformation
    
    # IP
    $ipconfig = Get-NetIPAddress | Where-Object {$_.AddressFamily -eq "IPv4" -and $_.IPAddress -notlike "127.*"} | 
        Select-Object InterfaceAlias, IPAddress, PrefixLength
    $ipconfig | Export-Csv "$OutputPath\${computerName}_IPConfig_$timestamp.csv" -NoTypeInformation
    
    # Software instalado
    Write-Host "Coletando lista de software..." -ForegroundColor Yellow
    $software = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | 
        Where-Object {$_.DisplayName} | 
        Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | 
        Sort-Object DisplayName
    $software | Export-Csv "$OutputPath\${computerName}_Software_$timestamp.csv" -NoTypeInformation
    
    # Atualizações instaladas
    Write-Host "Coletando atualizações instaladas..." -ForegroundColor Yellow
    $updates = Get-HotFix | Select-Object HotFixID, Description, InstalledOn | Sort-Object InstalledOn -Descending
    $updates | Export-Csv "$OutputPath\${computerName}_Updates_$timestamp.csv" -NoTypeInformation
    
    # Usuários locais
    Write-Host "Coletando informações de usuários..." -ForegroundColor Yellow
    $users = Get-LocalUser | Select-Object Name, Enabled, LastLogon, PasswordLastSet
    $users | Export-Csv "$OutputPath\${computerName}_Users_$timestamp.csv" -NoTypeInformation
    
    # Serviços
    Write-Host "Coletando informações de serviços..." -ForegroundColor Yellow
    $services = Get-Service | Select-Object Name, DisplayName, Status, StartType | Sort-Object DisplayName
    $services | Export-Csv "$OutputPath\${computerName}_Services_$timestamp.csv" -NoTypeInformation
    
    # Processos em execução
    Write-Host "Coletando processos em execução..." -ForegroundColor Yellow
    $processes = Get-Process | Select-Object Name, Id, @{N='MemoryMB';E={[math]::Round($_.WS/1MB,2)}}, CPU, Path | Sort-Object MemoryMB -Descending
    $processes | Export-Csv "$OutputPath\${computerName}_Processes_$timestamp.csv" -NoTypeInformation
    
    Write-Host "`nInventário concluído!" -ForegroundColor Green
    Write-Host "Arquivos salvos em: $OutputPath" -ForegroundColor Cyan
}

# Executar inventário
Get-SystemInventory
```
⬆️ Voltar ao topo

# 15. Scripts de Automação Úteis
## Script de Manutenção Completa
### Script de manutenção automática do Windows
```
function Start-SystemMaintenance {
    param(
        [switch]$FullMaintenance,
        [switch]$QuickMaintenance,
        [string]$LogPath = "C:\Logs\Manutencao"
    )
    
    # Criar pasta de logs
    New-Item -Path $LogPath -ItemType Directory -Force | Out-Null
    $logFile = "$LogPath\Manutencao_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
    
    function Write-Log {
        param($Message, $Color = "White")
        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        $logMessage = "$timestamp - $Message"
        Write-Host $Message -ForegroundColor $Color
        $logMessage | Out-File $logFile -Append
    }
    
    Write-Log "=== INICIANDO MANUTENÇÃO DO SISTEMA ===" "Cyan"
    Write-Log "Computador: $env:COMPUTERNAME"
    Write-Log "Usuário: $env:USERNAME"
    
    # 1. Verificar espaço em disco
    Write-Log "`n[1/10] Verificando espaço em disco..." "Yellow"
    $disk = Get-PSDrive C
    $freeSpaceGB = [math]::Round($disk.Free / 1GB, 2)
    Write-Log "Espaço livre em C: $freeSpaceGB GB" "Cyan"
    
    if ($freeSpaceGB -lt 10) {
        Write-Log "AVISO: Espaço em disco baixo!" "Red"
    }
    
    # 2. Limpar arquivos temporários
    Write-Log "`n[2/10] Limpando arquivos temporários..." "Yellow"
    try {
        Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
        Remove-Item "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue
        Write-Log "Arquivos temporários removidos" "Green"
    } catch {
        Write-Log "Erro ao limpar temporários: $_" "Red"
    }
    
    # 3. Limpar cache do Windows Update
    Write-Log "`n[3/10] Limpando cache do Windows Update..." "Yellow"
    try {
        Stop-Service wuauserv -Force
        Remove-Item "C:\Windows\SoftwareDistribution\Download\*" -Recurse -Force -ErrorAction SilentlyContinue
        Start-Service wuauserv
        Write-Log "Cache do Windows Update limpo" "Green"
    } catch {
        Write-Log "Erro ao limpar cache: $_" "Red"
    }
    
    # 4. Executar limpeza de disco
    Write-Log "`n[4/10] Executando limpeza de disco..." "Yellow"
    try {
        Start-Process cleanmgr -ArgumentList "/sagerun:1" -Wait -NoNewWindow
        Write-Log "Limpeza de disco concluída" "Green"
    } catch {
        Write-Log "Erro na limpeza de disco: $_" "Red"
    }
    
    if ($FullMaintenance) {
        # 5. DISM
        Write-Log "`n[5/10] Executando DISM..." "Yellow"
        try {
            $dismResult = DISM /Online /Cleanup-Image /RestoreHealth
            Write-Log "DISM concluído" "Green"
        } catch {
            Write-Log "Erro no DISM: $_" "Red"
        }
        
        # 6. SFC
        Write-Log "`n[6/10] Executando SFC..." "Yellow"
        try {
            $sfcResult = sfc /scannow
            Write-Log "SFC concluído" "Green"
        } catch {
            Write-Log "Erro no SFC: $_" "Red"
        }
    }
    
    # 7. Atualizar Windows Defender
    Write-Log "`n[7/10] Atualizando Windows Defender..." "Yellow"
    try {
        Update-MpSignature
        Write-Log "Windows Defender atualizado" "Green"
    } catch {
        Write-Log "Erro ao atualizar Defender: $_" "Red"
    }
    
    # 8. Gerar relatório final
    Write-Log "`n[8/10] Gerando relatório final..." "Yellow"
    
    $disk = Get-PSDrive C
    $freeSpaceAfter = [math]::Round($disk.Free / 1GB, 2)
    $spaceRecovered = $freeSpaceAfter - $freeSpaceGB
    
    Write-Log "`n=== RELATÓRIO FINAL ===" "Cyan"
    Write-Log "Espaço livre antes: $freeSpaceGB GB" "Cyan"
    Write-Log "Espaço livre depois: $freeSpaceAfter GB" "Cyan"
    Write-Log "Espaço recuperado: $spaceRecovered GB" "Green"
    Write-Log "`nLog salvo em: $logFile" "Cyan"
    Write-Log "`n=== MANUTENÇÃO CONCLUÍDA ===" "Green"
    
    # Abrir log
    notepad $logFile
}

# Executar manutenção rápida
Start-SystemMaintenance -QuickMaintenance

# Executar manutenção completa
# Start-SystemMaintenance -FullMaintenance
```
⬆️ Voltar ao topo

📚 Referências e Recursos Adicionais
Documentação Oficial Microsoft

Windows Commands Reference
PowerShell Documentation
Windows Update Troubleshooting
DISM Command-Line Options

Ferramentas Recomendadas

Sysinternals Suite: https://docs.microsoft.com/sysinternals/
Windows Update Catalog: https://www.catalog.update.microsoft.com/
Winget: https://github.com/microsoft/winget-cli
PSWindowsUpdate: https://www.powershellgallery.com/packages/PSWindowsUpdate

Comunidades e Fóruns

Microsoft Tech Community
Reddit: r/sysadmin, r/PowerShell
Stack Overflow
Spiceworks Community


Última atualização: Maio de 2026
Versão: 2.0
Compilado para uso profissional em suporte técnico

Este guia é um documento vivo e deve ser atualizado conforme novas técnicas e ferramentas são descobertas. Contribuições e sugestões são bem-vindas!

Agora o arquivo está formatado corretamente para o GitHub! As principais melhorias incluem:

1. ✅ Índice clicável no topo
2. ✅ Links de "Voltar ao topo" em cada seção
3. ✅ Blocos de código com sintaxe destacada (```batch, ```powershell)
4. ✅ Emojis para melhor visualização
5. ✅ Separadores visuais entre seções
6. ✅ Formatação consistente de títulos e subtítulos
7. ✅ Comentários dentro dos blocos de código


# Backup do Registro
function Backup-Registry {
    param (
        [string]$path,
        [string]$backupFile
    )
    reg export $path $backupFile /y
    Write-Output "Backup do registro salvo em: $backupFile"
}

# Função para definir uma chave do registro
function Set-RegistryKey {
    param (
        [string]$path,
        [string]$name,
        [string]$value,
        [Microsoft.Win32.RegistryValueKind]$type
    )
    if (-not (Test-Path $path)) {
        New-Item -Path $path -Force | Out-Null
    }
    Set-ItemProperty -Path $path -Name $name -Value $value -PropertyType $type
    Write-Output "Chave do registro atualizada: $path\$name = $value"
}

# Desabilitar a animação de janelas
Backup-Registry -path "HKCU\Control Panel\Desktop" -backupFile "$env:USERPROFILE\Desktop\Desktop_Backup.reg"
Set-RegistryKey -path "HKCU\Control Panel\Desktop\WindowMetrics" -name "MinAnimate" -value "0" -type String

# Ajustar o tempo de espera para encerrar aplicativos
Set-RegistryKey -path "HKCU\Control Panel\Desktop" -name "WaitToKillAppTimeout" -value "2000" -type String
Set-RegistryKey -path "HKCU\Control Panel\Desktop" -name "HungAppTimeout" -value "1000" -type String

# Desabilitar a execução automática de programas de inicialização
# Nota: Remover programas específicos pode depender de quais programas você deseja desabilitar. Aqui é mostrado como remover um exemplo:
$startupPath = "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
Backup-Registry -path $startupPath -backupFile "$env:USERPROFILE\Desktop\Run_Backup.reg"
Remove-ItemProperty -Path $startupPath -Name "OneDrive" -ErrorAction SilentlyContinue

# Melhorar o tempo de inicialização
Backup-Registry -path "HKLM\SYSTEM\CurrentControlSet\Control" -backupFile "$env:USERPROFILE\Desktop\Control_Backup.reg"
Set-RegistryKey -path "HKLM\SYSTEM\CurrentControlSet\Control" -name "WaitToKillServiceTimeout" -value "2000" -type String

# Desabilitar o Prefetch e o Superfetch
$prefetchPath = "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PrefetchParameters"
Backup-Registry -path $prefetchPath -backupFile "$env:USERPROFILE\Desktop\Prefetch_Backup.reg"
Set-RegistryKey -path $prefetchPath -name "EnablePrefetcher" -value "0" -type DWord
Set-RegistryKey -path $prefetchPath -name "EnableSuperfetch" -value "0" -type DWord

Write-Output "Otimizações concluídas. É recomendado reiniciar o computador para aplicar todas as alterações."

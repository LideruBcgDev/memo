# Windowsインストール自動化完全ガイド

## 目次
1. [自動化手法の概要](#自動化手法の概要)
2. [無人インストール（Unattended Installation）](#無人インストール)
3. [Windows Deployment Services (WDS)](#windows-deployment-services)
4. [Microsoft Deployment Toolkit (MDT)](#microsoft-deployment-toolkit)
5. [Windows Autopilot](#windows-autopilot)
6. [PowerShellによる自動化](#powershellによる自動化)
7. [Sysprepとイメージ展開](#sysprepとイメージ展開)
8. [企業環境での実装戦略](#企業環境での実装戦略)

## 自動化手法の概要

### 主要な自動化方式の比較

**無人インストール（Unattended）**
- 難易度: 初級
- 適用規模: 小規模（1-10台）
- 特徴: answer fileを使用したインストール自動化

**WDS + MDT**
- 難易度: 中級
- 適用規模: 中〜大規模（10-1000台）
- 特徴: ネットワーク経由でのイメージ配布

**Windows Autopilot**
- 難易度: 中級
- 適用規模: 中〜大規模（クラウド統合）
- 特徴: ゼロタッチ展開、Azure AD統合

**カスタムイメージ展開**
- 難易度: 上級
- 適用規模: 大規模（1000台以上）
- 特徴: 完全にカスタマイズされた環境

## 無人インストール

### 1. Windows System Image Manager (SIM) の準備

Windows Assessment and Deployment Kit (ADK) をインストールし、SIMを起動します。

### 2. Answer File (unattend.xml) の作成

基本的なunattend.xmlの構造：

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <!-- Windows PE Pass -->
    <settings pass="windowsPE">
        <component name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <SetupUILanguage>
                <UILanguage>ja-JP</UILanguage>
            </SetupUILanguage>
            <InputLocale>ja-JP</InputLocale>
            <SystemLocale>ja-JP</SystemLocale>
            <UILanguage>ja-JP</UILanguage>
            <UserLocale>ja-JP</UserLocale>
        </component>
        <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <DiskConfiguration>
                <Disk wcm:action="add">
                    <DiskID>0</DiskID>
                    <WillWipeDisk>true</WillWipeDisk>
                    <CreatePartitions>
                        <CreatePartition wcm:action="add">
                            <Order>1</Order>
                            <Type>Primary</Type>
                            <Extend>true</Extend>
                        </CreatePartition>
                    </CreatePartitions>
                    <ModifyPartitions>
                        <ModifyPartition wcm:action="add">
                            <Order>1</Order>
                            <PartitionID>1</PartitionID>
                            <Letter>C</Letter>
                            <Label>Windows</Label>
                            <Format>NTFS</Format>
                            <Active>true</Active>
                        </ModifyPartition>
                    </ModifyPartitions>
                </Disk>
            </DiskConfiguration>
            <ImageInstall>
                <OSImage>
                    <InstallFrom>
                        <MetaData wcm:action="add">
                            <Key>/IMAGE/INDEX</Key>
                            <Value>1</Value>
                        </MetaData>
                    </InstallFrom>
                    <InstallTo>
                        <DiskID>0</DiskID>
                        <PartitionID>1</PartitionID>
                    </InstallTo>
                </OSImage>
            </ImageInstall>
            <UserData>
                <ProductKey>
                    <Key>XXXXX-XXXXX-XXXXX-XXXXX-XXXXX</Key>
                </ProductKey>
                <AcceptEula>true</AcceptEula>
            </UserData>
        </component>
    </settings>

    <!-- Specialize Pass -->
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <ComputerName>WIN-CLIENT</ComputerName>
            <RegisteredOrganization>Your Organization</RegisteredOrganization>
            <RegisteredOwner>IT Department</RegisteredOwner>
            <TimeZone>Tokyo Standard Time</TimeZone>
        </component>
    </settings>

    <!-- OOBE System Pass -->
    <settings pass="oobeSystem">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <OOBE>
                <HideEULAPage>true</HideEULAPage>
                <HideLocalAccountScreen>true</HideLocalAccountScreen>
                <HideOEMRegistrationScreen>true</HideOEMRegistrationScreen>
                <HideOnlineAccountScreens>true</HideOnlineAccountScreens>
                <HideWirelessSetupInOOBE>true</HideWirelessSetupInOOBE>
                <NetworkLocation>Work</NetworkLocation>
                <ProtectYourPC>1</ProtectYourPC>
            </OOBE>
            <UserAccounts>
                <LocalAccounts>
                    <LocalAccount wcm:action="add">
                        <Password>
                            <Value>UGFzc3dvcmQ=</Value>
                            <PlainText>false</PlainText>
                        </Password>
                        <Group>Administrators</Group>
                        <DisplayName>Administrator</DisplayName>
                        <Name>Admin</Name>
                        <Description>Local Administrator Account</Description>
                    </LocalAccount>
                </LocalAccounts>
            </UserAccounts>
            <AutoLogon>
                <Password>
                    <Value>UGFzc3dvcmQ=</Value>
                    <PlainText>false</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>1</LogonCount>
                <Username>Admin</Username>
            </AutoLogon>
        </component>
    </settings>
</unattend>
```

### 3. 実装手順

1. **インストールメディアの準備**
   ```cmd
   # USBドライブをフォーマット
   diskpart
   list disk
   select disk X
   clean
   create partition primary
   active
   format fs=fat32 quick
   assign
   exit
   ```

2. **ファイルのコピー**
   ```cmd
   # Windows ISOの内容をUSBにコピー
   xcopy D:\*.* E:\ /s /e /h
   
   # unattend.xmlを配置
   copy unattend.xml E:\
   ```

## Windows Deployment Services

### WDSサーバーの構築

1. **サーバー役割の追加**
   ```powershell
   Install-WindowsFeature -Name WDS -IncludeManagementTools
   ```

2. **WDSの初期設定**
   ```cmd
   wdsutil /initialize-server /remInst:C:\RemoteInstall
   wdsutil /start-server
   ```

3. **ブートイメージの追加**
   ```cmd
   wdsutil /add-image /imagefile:C:\boot.wim /imagetype:boot
   ```

4. **インストールイメージの追加**
   ```cmd
   wdsutil /add-image /imagefile:C:\install.wim /imagetype:install
   ```

### PXEブート設定

DHCPサーバーでのオプション設定：
- オプション 66: WDSサーバーのIPアドレス
- オプション 67: boot\x64\pxeboot.n12

## Microsoft Deployment Toolkit

### 1. MDTのセットアップ

```powershell
# ADKとMDTのインストール後
Import-Module "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"

# 展開共有の作成
New-PSDrive -Name "DS001" -PSProvider "MDTProvider" -Root "C:\DeploymentShare"
```

### 2. タスクシーケンスの作成

1. **Operating Systemsの追加**
   - Windows ISOをインポート
   - 複数エディションの選択

2. **Applicationsの追加**
   ```powershell
   # サイレントインストール用のアプリケーション追加例
   $AppName = "Google Chrome"
   $SourcePath = "C:\Software\Chrome"
   $CommandLine = "ChromeSetup.exe /silent /install"
   ```

3. **Driversの追加**
   - ハードウェア固有のドライバを追加
   - Out-of-boxドライバの管理

### 3. カスタマイゼーション

**CustomSettings.ini の設定例**
```ini
[Settings]
Priority=Default
Properties=MyCustomProperty

[Default]
OSInstall=Y
SkipCapture=YES
SkipAdminPassword=NO
SkipProductKey=YES
SkipComputerBackup=YES
SkipBitLocker=YES
SkipSummary=YES
SkipUserData=YES
SkipPackageDisplay=YES
SkipLocaleSelection=YES
SkipTimeZone=YES

TimeZoneName=Tokyo Standard Time
KeyboardLocale=ja-JP
UserLocale=ja-JP
UILanguage=ja-JP

JoinWorkgroup=WORKGROUP
HideShell=YES
ApplyGPOPack=NO

# アプリケーションの自動インストール
Applications001={GUID-of-Application}
Applications002={GUID-of-Application-2}

# 管理者アカウント設定
AdminPassword=P@ssw0rd
```

## Windows Autopilot

### 1. 前提条件

- Azure AD Premium またはMicrosoft 365 Business Premium
- Microsoft Intune ライセンス
- Windows 10/11 Pro以上

### 2. デバイス登録プロセス

**PowerShellスクリプトでの一括登録**
```powershell
# Get-WindowsAutoPilotInfo.ps1スクリプトを使用
Install-Script -Name Get-WindowsAutoPilotInfo

# ハードウェアハッシュの取得
Get-WindowsAutoPilotInfo.ps1 -OutputFile AutoPilotHWID.csv

# Azure ADへのアップロード
Import-AutopilotDevice -csvFile AutoPilotHWID.csv
```

### 3. 展開プロファイルの設定

```json
{
  "displayName": "Corporate Deployment Profile",
  "description": "Standard corporate device deployment",
  "deviceNameTemplate": "CORP-%SERIAL%",
  "deviceType": "windowsPc",
  "enableWhiteGlove": false,
  "outOfBoxExperienceSettings": {
    "hidePrivacySettings": true,
    "hideEULA": true,
    "userType": "standard",
    "deviceUsageType": "singleUser",
    "skipKeyboardSelectionPage": true,
    "hideEscapeLink": true
  }
}
```

## PowerShellによる自動化

### 1. 初期設定自動化スクリプト

```powershell
# Windows初期設定自動化スクリプト
param(
    [string]$ComputerName = "WIN-CLIENT",
    [string]$Domain = "contoso.com",
    [string]$OUPath = "OU=Computers,DC=contoso,DC=com"
)

# Windows Updateの設定
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" -Name "AUOptions" -Value 4

# ファイアウォールの設定
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True

# リモートデスクトップの有効化
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

# タイムゾーンの設定
Set-TimeZone -Id "Tokyo Standard Time"

# 地域設定
Set-WinSystemLocale -SystemLocale ja-JP
Set-WinUserLanguageList -LanguageList ja-JP -Force
Set-WinHomeLocation -GeoId 122

# ドメイン参加
if ($Domain) {
    $Credential = Get-Credential
    Add-Computer -DomainName $Domain -Credential $Credential -OUPath $OUPath -Restart
}

# ソフトウェアの一括インストール
$Applications = @(
    @{Name="Google Chrome"; Path="C:\Software\Chrome\ChromeSetup.exe"; Args="/silent /install"},
    @{Name="Adobe Reader"; Path="C:\Software\Adobe\AcroRdrDC.exe"; Args="/sAll /rs /msi EULA_ACCEPT=YES"}
)

foreach ($App in $Applications) {
    Write-Host "Installing $($App.Name)..."
    Start-Process -FilePath $App.Path -ArgumentList $App.Args -Wait
}
```

### 2. ソフトウェア展開スクリプト

```powershell
# ソフトウェア一括展開関数
function Install-Software {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Name,
        [Parameter(Mandatory=$true)]
        [string]$InstallerPath,
        [string]$Arguments = "",
        [string]$LogPath = "C:\Logs\Install-$Name.log"
    )
    
    try {
        Write-Host "Installing $Name..." -ForegroundColor Green
        
        # インストーラーの実行
        $Process = Start-Process -FilePath $InstallerPath -ArgumentList $Arguments -PassThru -Wait
        
        if ($Process.ExitCode -eq 0) {
            Write-Host "$Name installed successfully" -ForegroundColor Green
            Add-Content -Path $LogPath -Value "$(Get-Date): $Name installed successfully"
        } else {
            Write-Warning "$Name installation failed with exit code $($Process.ExitCode)"
            Add-Content -Path $LogPath -Value "$(Get-Date): $Name installation failed with exit code $($Process.ExitCode)"
        }
    }
    catch {
        Write-Error "Error installing $Name : $($_.Exception.Message)"
        Add-Content -Path $LogPath -Value "$(Get-Date): Error installing $Name : $($_.Exception.Message)"
    }
}

# 使用例
Install-Software -Name "7-Zip" -InstallerPath "C:\Software\7z1900-x64.msi" -Arguments "/quiet"
Install-Software -Name "Notepad++" -InstallerPath "C:\Software\npp.installer.exe" -Arguments "/S"
```

## Sysprepとイメージ展開

### 1. マスターイメージの準備

```cmd
# Sysprepの実行
C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown

# または unattend.xmlを使用
C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown /unattend:C:\unattend.xml
```

### 2. イメージキャプチャ

```cmd
# DISMを使用したイメージキャプチャ
dism /capture-image /imagefile:C:\master.wim /capturedir:D:\ /name:"Master Image" /compress:maximum

# 差分イメージの作成
dism /capture-image /imagefile:C:\update.wim /capturedir:D:\ /name:"Updated Image" /delta:C:\master.wim
```

### 3. イメージ展開

```cmd
# パーティションの準備
diskpart
select disk 0
clean
create partition primary
active
format fs=ntfs quick
assign letter=C
exit

# イメージの適用
dism /apply-image /imagefile:C:\master.wim /index:1 /applydir:C:\

# ブート構成の復元
bcdboot C:\Windows /s C:
```

## 企業環境での実装戦略

### 1. 段階的展開アプローチ

**フェーズ1: パイロット展開**
- 小規模グループ（10-20台）でのテスト
- フィードバック収集と改善

**フェーズ2: 部門別展開**
- 部門ごとの要件に応じたカスタマイズ
- 段階的なロールアウト

**フェーズ3: 全社展開**
- 標準化されたプロセスでの大規模展開
- 継続的な監視と改善

### 2. 品質管理プロセス

**テスト環境での検証**
```powershell
# 自動テストスクリプトの例
function Test-StandardInstallation {
    $TestResults = @()
    
    # OS バージョン確認
    $OSVersion = (Get-WmiObject -Class Win32_OperatingSystem).Version
    $TestResults += @{Test="OS Version"; Result=$OSVersion; Status="Pass"}
    
    # ドメイン参加確認
    $Domain = (Get-WmiObject -Class Win32_ComputerSystem).Domain
    $TestResults += @{Test="Domain Join"; Result=$Domain; Status=if($Domain -ne "WORKGROUP"){"Pass"}else{"Fail"}}
    
    # 必要なソフトウェアの確認
    $RequiredSoftware = @("Google Chrome", "Adobe Acrobat Reader", "Microsoft Office")
    foreach ($Software in $RequiredSoftware) {
        $Installed = Get-WmiObject -Class Win32_Product | Where-Object {$_.Name -like "*$Software*"}
        $TestResults += @{Test="Software: $Software"; Result=$Installed.Name; Status=if($Installed){"Pass"}else{"Fail"}}
    }
    
    return $TestResults
}
```

### 3. 監視とレポート

**展開状況の追跡**
```powershell
# 展開ログの集約
function Get-DeploymentStatus {
    param([string[]]$ComputerNames)
    
    $Results = @()
    foreach ($Computer in $ComputerNames) {
        try {
            $LastBoot = (Get-WmiObject -Class Win32_OperatingSystem -ComputerName $Computer).LastBootUpTime
            $LastBootTime = [Management.ManagementDateTimeConverter]::ToDateTime($LastBoot)
            
            $Results += [PSCustomObject]@{
                ComputerName = $Computer
                Status = "Online"
                LastBoot = $LastBootTime
                DaysSinceInstall = (Get-Date) - $LastBootTime | Select-Object -ExpandProperty Days
            }
        }
        catch {
            $Results += [PSCustomObject]@{
                ComputerName = $Computer
                Status = "Offline"
                LastBoot = $null
                DaysSinceInstall = $null
            }
        }
    }
    
    return $Results
}
```

### 4. トラブルシューティング

**一般的な問題と解決策**

1. **ドライバーの問題**
   - ハードウェア固有のドライバーパックを準備
   - Windows Updateでの自動取得設定

2. **ライセンス認証の問題**
   - KMS/MAKライセンスの適切な設定
   - ボリュームライセンスキーの管理

3. **ネットワーク接続の問題**
   - DHCP予約の設定
   - VLANとセキュリティポリシーの確認

### 5. セキュリティ考慮事項

**セキュアな展開のための設定**
```powershell
# セキュリティ強化スクリプト
function Set-SecurityHardening {
    # UAC設定
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA" -Value 1
    
    # Windows Defenderの有効化
    Set-MpPreference -DisableRealtimeMonitoring $false
    
    # 自動更新の有効化
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" -Name "AUOptions" -Value 4
    
    # ゲストアカウントの無効化
    Net User Guest /Active:No
    
    # 不要なサービスの停止
    $ServicesToDisable = @("Fax", "Remote Registry", "Routing and Remote Access")
    foreach ($Service in $ServicesToDisable) {
        Set-Service -Name $Service -StartupType Disabled -ErrorAction SilentlyContinue
    }
}
```

## まとめ

Windowsインストールの自動化は、規模と要件に応じて適切な手法を選択することが重要です。小規模な環境では無人インストールで十分ですが、企業環境では MDT + WDS や Windows Autopilot の組み合わせが効果的です。

成功のポイント：
- 段階的な展開アプローチの採用
- 十分なテストと検証プロセス
- 継続的な監視と改善
- セキュリティ要件の組み込み
- ドキュメント化と知識共有

適切な自動化により、展開時間の短縮、設定の標準化、運用コストの削減を実現できます。
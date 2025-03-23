# Second-Me Windows移植 設計ドキュメント

## 1. アーキテクチャ概要

オリジナルのSecond-Meプロジェクトは、macOS環境向けに設計されたAIセルフプロトタイプです。
このドキュメントでは、Windows環境への移植に必要な変更点と設計方針を詳細に記述します。

### 1.1 現行アーキテクチャ（macOS）

```
Second-Me/
├── .env                    # 環境変数設定
├── Makefile                # 各種コマンドのエイリアス
├── environment.yml         # Conda環境定義
├── pyproject.toml          # Poetry設定
├── scripts/                # 各種シェルスクリプト
│   ├── conda_utils.sh      # Conda関連ユーティリティ
│   ├── setup.sh            # 環境セットアップスクリプト
│   ├── start.sh            # サービス起動スクリプト
│   ├── start_local.sh      # ローカルサービス起動
│   ├── stop.sh             # サービス停止スクリプト
│   ├── restart.sh          # サービス再起動スクリプト
│   └── status.sh           # サービス状態確認
├── llama.cpp/              # llama.cpp ライブラリ
├── lpm_frontend/           # フロントエンドプロジェクト
└── dependencies/           # 外部依存ライブラリ
```

### 1.2 Windows向けアーキテクチャ

```
Second-Me-Windows/
├── .env                    # 環境変数設定（Windows用に修正）
├── make.ps1                # Makefileの代替
├── environment.yml         # Conda環境定義（変更なし）
├── pyproject.toml          # Poetry設定（変更なし）
├── scripts/                # PowerShellスクリプト
│   ├── conda_utils.ps1     # Conda関連ユーティリティ
│   ├── setup.ps1           # 環境セットアップスクリプト
│   ├── start.ps1           # サービス起動スクリプト
│   ├── start_local.ps1     # ローカルサービス起動
│   ├── stop.ps1            # サービス停止スクリプト
│   ├── restart.ps1         # サービス再起動スクリプト
│   └── status.ps1          # サービス状態確認
├── llama.cpp/              # llama.cpp ライブラリ（Windows向けビルド）
├── lpm_frontend/           # フロントエンドプロジェクト（変更なし）
└── dependencies/           # 外部依存ライブラリ
```

## 2. コンポーネント詳細設計

### 2.1 PowerShellスクリプト

#### 2.1.1 conda_utils.ps1

```powershell
# Conda環境管理ユーティリティ関数

# Condaをインストールディレクトリから見つけて初期化を試みる
function Find-And-Initialize-Conda {
    param (
        [string]$CondaRoot = $null
    )
    
    # Condaパスを探索
    if (-not $CondaRoot) {
        $potentialPaths = @(
            "$env:USERPROFILE\Miniconda3",
            "$env:USERPROFILE\Anaconda3",
            "$env:ProgramData\Miniconda3",
            "$env:ProgramData\Anaconda3"
        )
        
        foreach ($path in $potentialPaths) {
            if (Test-Path "$path\Scripts\activate.ps1") {
                $CondaRoot = $path
                break
            }
        }
    }
    
    if (-not $CondaRoot) {
        Write-Error "Conda installation not found"
        return $false
    }
    
    # Condaスクリプトのパス
    $condaActivatePath = "$CondaRoot\Scripts\activate.ps1"
    
    # スクリプトが見つかったか確認
    if (-not (Test-Path $condaActivatePath)) {
        Write-Error "Conda activation script not found at: $condaActivatePath"
        return $false
    }
    
    # Conda環境を初期化
    try {
        & $condaActivatePath
        return $true
    } catch {
        Write-Error "Failed to initialize Conda: $_"
        return $false
    }
}

# 指定されたConda環境をアクティベート
function Activate-CondaEnvironment {
    param (
        [string]$EnvName
    )
    
    try {
        conda activate $EnvName
        return $true
    } catch {
        Write-Error "Failed to activate Conda environment '$EnvName': $_"
        return $false
    }
}

# Conda環境名をENVファイルから取得
function Get-CondaEnvironmentName {
    if (Test-Path ".env") {
        $envFile = Get-Content ".env"
        $envName = $envFile | Where-Object { $_ -match "^CONDA_DEFAULT_ENV=" } | ForEach-Object { $_ -replace "^CONDA_DEFAULT_ENV=", "" }
        
        if ($envName) {
            return $envName
        }
    }
    
    # デフォルト値
    return "second-me"
}
```

#### 2.1.2 setup.ps1

```powershell
# Windows用セットアップスクリプト

$ScriptVersion = "1.0.0"
$ErrorActionPreference = "Stop"
$TotalStages = 6
$CurrentStage = 0

# ロギングの基本設定
$LogPath = ".\logs"
if (-not (Test-Path $LogPath)) {
    New-Item -Path $LogPath -ItemType Directory -Force | Out-Null
}

# ロギング関数
function Log-Info {
    param([string]$Message)
    Write-Host "[INFO] $Message" -ForegroundColor Cyan
}

function Log-Success {
    param([string]$Message)
    Write-Host "[SUCCESS] $Message" -ForegroundColor Green
}

function Log-Warning {
    param([string]$Message)
    Write-Host "[WARNING] $Message" -ForegroundColor Yellow
}

function Log-Error {
    param([string]$Message)
    Write-Host "[ERROR] $Message" -ForegroundColor Red
}

function Log-Section {
    param([string]$Title)
    Write-Host "`n======== $Title ========" -ForegroundColor Magenta
}

# 現在の時刻を取得
function Get-Timestamp {
    return Get-Date -Format "yyyy-MM-dd HH:mm:ss"
}

# 環境確認
function Check-SystemRequirements {
    Log-Section "CHECKING SYSTEM REQUIREMENTS"

    # Windows検出
    $OS = [System.Environment]::OSVersion.Platform
    if ($OS -ne "Win32NT") {
        Log-Error "This script only supports Windows"
        return $false
    }
    
    $WindowsVersion = [System.Environment]::OSVersion.Version
    Log-Info "Detected Windows version: $WindowsVersion"
    
    # Windows 10以上を要求
    if ($WindowsVersion.Major -lt 10) {
        Log-Error "This script requires Windows 10 or later"
        return $false
    }
    
    # PowerShellバージョン確認
    $PSVersion = $PSVersionTable.PSVersion
    Log-Info "PowerShell version: $PSVersion"
    
    if ($PSVersion.Major -lt 5) {
        Log-Error "PowerShell 5.0 or later is required"
        return $false
    }
    
    # Chocolateyチェック
    if (-not (Get-Command choco -ErrorAction SilentlyContinue)) {
        Log-Warning "Chocolatey is not installed, attempting to install it automatically..."
        
        try {
            # 管理者権限チェック
            $isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
            if (-not $isAdmin) {
                Log-Error "Administrator privileges are required to install Chocolatey."
                Log-Error "Please run this script as an administrator."
                return $false
            }
            
            # Chocolateyのインストール
            Set-ExecutionPolicy Bypass -Scope Process -Force
            [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
            Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
            
            # インストールの確認
            if (Get-Command choco -ErrorAction SilentlyContinue) {
                Log-Success "Chocolatey installed successfully"
            } else {
                Log-Error "Failed to install Chocolatey"
                return $false
            }
        } catch {
            Log-Error "Error installing Chocolatey: $_"
            return $false
        }
    } else {
        Log-Info "Chocolatey is already installed"
    }
    
    # Condaの確認とインストール
    if (-not (Get-Command conda -ErrorAction SilentlyContinue)) {
        Log-Warning "Conda is not installed, attempting to install Miniconda..."
        
        try {
            choco install miniconda3 -y
            
            # PATHを更新
            $env:Path = [System.Environment]::GetEnvironmentVariable("Path", "Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path", "User")
            
            # インストールの確認
            if (Get-Command conda -ErrorAction SilentlyContinue) {
                Log-Success "Miniconda installed successfully"
            } else {
                Log-Error "Failed to install Miniconda"
                return $false
            }
        } catch {
            Log-Error "Error installing Miniconda: $_"
            return $false
        }
    } else {
        Log-Info "Conda is already installed"
    }
    
    return $true
}

# その他スクリプトの内容は略
```

### 2.2 llama.cpp Windows向けビルド

Windows環境でllama.cppをビルドするためには、CMakeとVisual Studio（またはMSBuild）を使用します。

```powershell
# llama.cpp_build.ps1

function Build-LlamaCpp {
    Log-Section "BUILDING LLAMA.CPP FOR WINDOWS"
    
    $LlamaZipPath = "dependencies\llama.cpp.zip"
    
    # llama.cppディレクトリが存在するかチェック
    if (-not (Test-Path "llama.cpp")) {
        Log-Info "Setting up llama.cpp..."
        
        if (Test-Path $LlamaZipPath) {
            Log-Info "Using local llama.cpp archive..."
            Expand-Archive -Path $LlamaZipPath -DestinationPath "." -Force
        } else {
            Log-Error "Local llama.cpp archive not found at: $LlamaZipPath"
            Log-Error "Please ensure the llama.cpp.zip file exists in the dependencies directory"
            return $false
        }
    } else {
        Log-Info "Found existing llama.cpp directory"
    }
    
    # 既存のビルドをチェック
    if (Test-Path "llama.cpp\build\bin\Release\llama-server.exe") {
        Log-Info "Found existing llama-server build"
        # 実行ファイルをテスト
        try {
            $version = & "llama.cpp\build\bin\Release\llama-server.exe" --version 2>&1
            if ($version -match "version:") {
                Log-Success "Existing llama-server build is working properly ($version), skipping compilation"
                return $true
            } else {
                Log-Warning "Existing build seems broken or incompatible, will recompile..."
            }
        } catch {
            Log-Warning "Failed to run existing build, will recompile..."
        }
    }
    
    # 必要なツールのチェック
    if (-not (Get-Command cmake -ErrorAction SilentlyContinue)) {
        Log-Warning "CMake not found, installing..."
        choco install cmake -y
        $env:Path = [System.Environment]::GetEnvironmentVariable("Path", "Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path", "User")
        
        if (-not (Get-Command cmake -ErrorAction SilentlyContinue)) {
            Log-Error "Failed to install CMake"
            return $false
        }
    }
    
    # Visual Studioのチェック
    $vswhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
    if (-not (Test-Path $vswhere)) {
        Log-Error "Visual Studio not found. Please install Visual Studio 2019 or newer with C++ development tools."
        return $false
    }
    
    $vsInstallation = & $vswhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
    if (-not $vsInstallation) {
        Log-Error "Visual Studio with C++ development tools not found."
        return $false
    }
    
    # ビルドディレクトリを作成・クリーンアップ
    Push-Location "llama.cpp"
    if (Test-Path "build") {
        Log-Info "Cleaning previous build..."
        Remove-Item -Path "build" -Recurse -Force
    }
    
    New-Item -Path "build" -ItemType Directory -Force | Out-Null
    Push-Location "build"
    
    # CMakeとビルドを実行
    Log-Info "Configuring CMake..."
    $cmakeConfigResult = (cmake -G "Visual Studio 17 2022" -A x64 ..)
    if ($LASTEXITCODE -ne 0) {
        Log-Error "CMake configuration failed with exit code $LASTEXITCODE"
        Pop-Location
        Pop-Location
        return $false
    }
    
    Log-Info "Building project..."
    $buildResult = (cmake --build . --config Release)
    if ($LASTEXITCODE -ne 0) {
        Log-Error "Build failed with exit code $LASTEXITCODE"
        Pop-Location
        Pop-Location
        return $false
    }
    
    # ビルド結果の確認
    if (-not (Test-Path "bin\Release\llama-server.exe")) {
        Log-Error "Build failed: llama-server.exe not found at expected location"
        Pop-Location
        Pop-Location
        return $false
    }
    
    Log-Success "Successfully built llama-server.exe"
    Pop-Location
    Pop-Location
    
    return $true
}
```

### 2.3 プロセス管理

Windowsでのプロセス管理はmacOSとは大きく異なります。バックグラウンドジョブ、プロセストラッキング、サービス停止などを適切に実装するためのスクリプトを準備します。

```powershell
# process_manager.ps1

# 指定されたポートが使用可能かをチェック
function Test-PortAvailable {
    param (
        [int]$Port
    )
    
    $tcpConnection = Get-NetTCPConnection -LocalPort $Port -ErrorAction SilentlyContinue
    
    return ($null -eq $tcpConnection)
}

# バックグラウンドでプロセスを起動し、PIDを記録
function Start-BackgroundProcess {
    param (
        [string]$Command,
        [string]$Arguments,
        [string]$WorkingDirectory = $null,
        [string]$PidFile
    )
    
    $processStartInfo = New-Object System.Diagnostics.ProcessStartInfo
    $processStartInfo.FileName = $Command
    $processStartInfo.Arguments = $Arguments
    $processStartInfo.UseShellExecute = $false
    $processStartInfo.CreateNoWindow = $true
    $processStartInfo.RedirectStandardOutput = $true
    $processStartInfo.RedirectStandardError = $true
    
    if ($WorkingDirectory) {
        $processStartInfo.WorkingDirectory = $WorkingDirectory
    }
    
    $process = New-Object System.Diagnostics.Process
    $process.StartInfo = $processStartInfo
    
    # ログファイルパスを作成
    $logDirectory = "logs"
    if (-not (Test-Path $logDirectory)) {
        New-Item -Path $logDirectory -ItemType Directory -Force | Out-Null
    }
    
    $stdoutLogFile = Join-Path $logDirectory "$($PidFile)_stdout.log"
    $stderrLogFile = Join-Path $logDirectory "$($PidFile)_stderr.log"
    
    # プロセスイベントハンドラを設定
    $stdoutScriptBlock = {
        param($sender, $e)
        if (-not [string]::IsNullOrEmpty($e.Data)) {
            $e.Data | Out-File -Append -FilePath $stdoutLogFile
        }
    }
    
    $stderrScriptBlock = {
        param($sender, $e)
        if (-not [string]::IsNullOrEmpty($e.Data)) {
            $e.Data | Out-File -Append -FilePath $stderrLogFile
        }
    }
    
    $process.OutputDataReceived += $stdoutScriptBlock
    $process.ErrorDataReceived += $stderrScriptBlock
    
    # プロセスを開始
    $process.Start() | Out-Null
    $process.BeginOutputReadLine()
    $process.BeginErrorReadLine()
    
    # PIDをファイルに記録
    $process.Id | Set-Content -Path (Join-Path "run" $PidFile)
    
    return $process.Id
}

# PIDファイルに記録されたプロセスを停止
function Stop-BackgroundProcess {
    param (
        [string]$PidFile
    )
    
    $pidFilePath = Join-Path "run" $PidFile
    
    if (-not (Test-Path $pidFilePath)) {
        Log-Warning "PID file not found: $pidFilePath"
        return $false
    }
    
    $pid = Get-Content $pidFilePath
    
    try {
        $process = Get-Process -Id $pid -ErrorAction SilentlyContinue
        
        if ($null -eq $process) {
            Log-Warning "Process with PID $pid not found"
            Remove-Item $pidFilePath -Force
            return $true
        }
        
        # プロセスを停止
        $process.Kill()
        $process.WaitForExit(5000)
        
        # PIDファイルを削除
        Remove-Item $pidFilePath -Force
        
        Log-Success "Process with PID $pid stopped successfully"
        return $true
    } catch {
        Log-Error "Failed to stop process with PID $pid: $_"
        return $false
    }
}

# 指定されたURLのヘルスチェック
function Test-BackendHealth {
    param (
        [string]$Url,
        [int]$MaxAttempts = 30,
        [int]$DelaySeconds = 1
    )
    
    for ($attempt = 1; $attempt -le $MaxAttempts; $attempt++) {
        try {
            $response = Invoke-WebRequest -Uri $Url -UseBasicParsing -TimeoutSec 1 -ErrorAction SilentlyContinue
            if ($response.StatusCode -eq 200) {
                return $true
            }
        } catch {
            # エラーは無視して次の試行へ
        }
        
        Start-Sleep -Seconds $DelaySeconds
    }
    
    return $false
}
```

## 3. 移行計画

### 3.1 開発フェーズ

1. **Phase 1: 基本機能実装**
   - PowerShellスクリプトの作成（conda_utils.ps1, process_manager.ps1）
   - make.ps1の作成（Makefileの代替）

2. **Phase 2: llama.cpp Windows対応**
   - Windows向けビルドスクリプトの作成
   - ビルド検証

3. **Phase 3: スクリプト移植**
   - setup.ps1, start.ps1, stop.ps1の作成
   - パス処理、環境変数処理の対応

4. **Phase 4: テストと統合**
   - 各スクリプトの個別テスト
   - エンドツーエンドテスト
   - エラーケースの対応

### 3.2 テスト計画

1. **単体テスト**
   - 各PowerShellスクリプト関数のテスト
   - エラー処理の検証

2. **環境テスト**
   - クリーンな環境でのセットアップテスト
   - 既存環境での競合テスト

3. **機能テスト**
   - llama.cppのビルドと実行テスト
   - フロントエンド・バックエンドの起動・通信テスト

4. **エンドツーエンドテスト**
   - 完全なプロセス（セットアップ→起動→実行→停止）のテスト

## 4. 技術的な留意点

### 4.1 パス処理の違い

macOSとWindowsではパス区切り文字が異なります（`/` vs `\`）。このため、パス処理には適切な関数を使用する必要があります。

```powershell
# 例: Windowsでのパス連結
$path = Join-Path $directory $file
```

### 4.2 プロセス管理

Windowsでは、Unixのようなfork/execモデルではなく、異なるプロセス管理メカニズムを使用します。バックグラウンドジョブとプロセストラッキングには専用の関数を使用します。

### 4.3 権限処理

Windowsのユーザーアカウント制御（UAC）により、特定の操作には管理者権限が必要です。権限エスカレーションに関する処理を組み込む必要があります。

### 4.4 デバッグ方法

デバッグには、PowerShellのデバッグコマンドや詳細なロギングシステムを活用します。また、Windowsイベントログも重要な診断情報源となります。

## 5. リスク管理

### 5.1 識別されたリスク

1. **Windows環境の多様性**: Windowsのバージョン、インストール状況、権限設定の多様性に起因する問題
2. **C++ビルド環境の複雑さ**: Visual StudioやMSBuildの設定に関連するビルド問題
3. **長いパス名の制限**: Windowsのパス長制限（260文字）に関連する問題
4. **サードパーティライブラリの依存関係**: Windowsでの互換性がない可能性

### 5.2 リスク軽減策

1. **明確な最小要件**: Windowsバージョン、必須コンポーネントを明確に指定
2. **エラー処理**: 包括的なエラー処理とリカバリメカニズムの実装
3. **ロングパスサポート**: Windowsのロングパスサポートを有効化する方法の提供
4. **代替ライブラリ**: 互換性問題がある場合の代替ライブラリの特定と実装

## 6. 次のステップ

1. ローカル開発環境でのWindowsポートの初期実装
2. PowerShellスクリプトのコア機能実装とテスト
3. llama.cppのWindows向けビルドスクリプトの作成
4. 統合テストとバグ修正
5. ドキュメントの整備
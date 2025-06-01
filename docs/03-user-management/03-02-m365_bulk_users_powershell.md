# 3.2 ユーザーの一括管理 (PowerShell版)

---

## 📖 学習目標

CSVファイルとPowerShellを活用した効率的な大量ユーザー管理手法を習得し、教育機関の新年度対応や企業の大規模システム移行に対応できるスキルを身につけます。

### 🎯 この節で習得すること

- **CSVテンプレート**の作成・活用方法をマスターし、効率的なデータ準備ができる
- **Microsoft 365管理センター**での一括操作を理解し、中規模の展開に対応できる
- **PowerShell（Microsoft Graph）**による自動化技術を習得し、大規模展開を実現できる
- **エラーハンドリング**と検証方法を身につけ、確実な運用ができる
- **運用フロー**を設計し、組織に適した展開戦略を立案できる

### 💼 実務での活用場面

- **新年度対応**: 教育機関での新入生・新任教職員の一括アカウント作成（数百〜数千名規模）
- **システム移行**: 既存システムからMicrosoft 365への大規模ユーザー移行
- **組織再編**: 合併・統合時の大量アカウント統合・変更処理
- **定期メンテナンス**: 学期末・年度末のアカウント状態一括更新

---

## ⚡ 前提条件

### 必要な権限・環境

- [ ] **Microsoft 365テナント**（本番環境または十分な容量のある試用版）
- [ ] **グローバル管理者**または**ユーザー管理者**権限
- [ ] **PowerShell 5.1以上**（Windows PowerShell または PowerShell 7+）
- [ ] **Microsoft Graph PowerShell SDK**（v2.0以上推奨）
- [ ] **十分なライセンス在庫**（一括作成予定数以上）

### 事前準備

```markdown
📋 大規模展開前チェックリスト
- [ ] 対象ユーザー数の確定（100名〜1000名〜10000名）
- [ ] 利用可能ライセンス数の確認と不足分の調達
- [ ] 組織の命名規則・パスワードポリシーの確認
- [ ] ネットワーク帯域・処理時間の見積もり
- [ ] バックアップ・ロールバック計画の策定
- [ ] テスト環境での事前検証完了
- [ ] 関係者への作業通知・調整完了
```

> 💡 **重要**: 1000名以上の大規模展開では、段階的な実行（100-500名ずつ）を強く推奨します。一度に全件処理するとタイムアウトや部分的な失敗のリスクが高まります。

---

## 📊 一括ユーザー管理の手法比較

### 手法別特徴と適用場面

| 手法 | 対象規模 | 所要時間 | 複雑性 | 推奨場面 |
|------|----------|----------|---------|----------|
| **管理センター（CSV）** | 〜200名 | 30分〜1時間 | ★☆☆ | 中小規模、GUI操作重視 |
| **PowerShell（基本）** | 〜1000名 | 1〜3時間 | ★★☆ | 中規模、自動化開始 |
| **PowerShell（並列）** | 1000名〜 | 30分〜2時間 | ★★★ | 大規模、高速処理必要 |
| **Graph API（REST）** | 制限なし | 処理性能最高 | ★★★★ | 超大規模、システム統合 |

### 処理パフォーマンス目安

```markdown
⏱️ 処理時間の目安（参考値）
- 100名: 管理センター 15分 / PowerShell 5分
- 500名: 管理センター 45分 / PowerShell 15分
- 1000名: PowerShell 30分 / 並列処理 10分
- 5000名: 並列PowerShell 45分 / Graph API 20分

※ ネットワーク環境・テナント負荷・データの複雑性により変動
```

---

## 📊 テナント内ユーザーの一括取得・分析

一括管理を始める前に、現在のテナント状況を正確に把握することが重要です。既存ユーザーの情報取得、分析、CSVエクスポートを効率的に行う方法を解説します。

### 基本的なユーザー一括取得

#### 全ユーザーの基本情報取得

**🎯 スクリプトの目的**
テナント内の全ユーザーアカウントを取得し、基本的なプロパティ（表示名、UPN、部署、ライセンス状況）を一覧表示します。現状把握、監査、移行計画の基礎データとして活用できます。

**📝 Usage**
```powershell
# 基本的な全ユーザー取得
Get-AllTenantUsers

# CSV形式で出力
Get-AllTenantUsers -OutputPath "C:\reports\all_users.csv"

# 特定のプロパティのみ取得
Get-AllTenantUsers -Properties @("DisplayName", "UserPrincipalName", "Department", "JobTitle") -OutputPath "C:\reports\users_basic.csv"
```

**⚙️ パラメータ**
- `-OutputPath`: CSV出力先パス（指定時はファイル出力）
- `-Properties`: 取得するプロパティの配列
- `-IncludeInactive`: 無効化されたユーザーも含める

```powershell
# 全ユーザーの基本情報一括取得
function Get-AllTenantUsers {
    param(
        [string]$OutputPath,
        [string[]]$Properties = @("DisplayName", "UserPrincipalName", "Department", "JobTitle", "AccountEnabled", "CreatedDateTime"),
        [switch]$IncludeInactive
    )
    
    Write-Host "テナント内ユーザーの取得を開始します..." -ForegroundColor Cyan
    
    # フィルター条件の設定
    $filter = if ($IncludeInactive) { $null } else { "accountEnabled eq true" }
    
    try {
        # 大規模テナント対応（ページング処理）
        $allUsers = Get-MgUser -All -Filter $filter | Select-Object $Properties
        
        Write-Host "取得完了: $($allUsers.Count)名のユーザー" -ForegroundColor Green
        
        # 結果の表示
        $allUsers | Format-Table -AutoSize
        
        # CSV出力（指定時）
        if ($OutputPath) {
            $allUsers | Export-Csv $OutputPath -Encoding UTF8 -NoTypeInformation
            Write-Host "CSVファイルを出力しました: $OutputPath" -ForegroundColor Green
        }
        
        return $allUsers
        
    } catch {
        Write-Error "ユーザー取得中にエラーが発生しました: $($_.Exception.Message)"
    }
}
```

### 条件別ユーザー検索・取得

#### 部署・役職別ユーザー取得

**🎯 スクリプトの目的**
特定の条件（部署、役職、ライセンス種別、作成日など）に基づいてユーザーを絞り込み取得します。組織変更時の対象ユーザー特定、ライセンス最適化の分析、セキュリティ監査などに活用できます。

**📝 Usage**
```powershell
# 特定部署のユーザー取得
Get-FilteredUsers -Department "情報学部" -OutputPath "C:\reports\info_dept_users.csv"

# 複数条件での検索
Get-FilteredUsers -Department "工学部" -JobTitle "教授" -OnlyActive

# 最近作成されたユーザー（過去30日）
Get-FilteredUsers -CreatedAfter (Get-Date).AddDays(-30) -OutputPath "C:\reports\recent_users.csv"

# ライセンス未割り当てユーザー
Get-FilteredUsers -NoLicense -OutputPath "C:\reports\unlicensed_users.csv"
```

**⚙️ パラメータ**
- `-Department`: 対象部署名
- `-JobTitle`: 対象役職名
- `-CreatedAfter`: 指定日以降に作成されたユーザー
- `-OnlyActive`: アクティブなユーザーのみ
- `-NoLicense`: ライセンス未割り当てユーザーのみ
- `-OutputPath`: 結果をCSVファイルに出力

```powershell
function Get-FilteredUsers {
    param(
        [string]$Department,
        [string]$JobTitle,
        [datetime]$CreatedAfter,
        [switch]$OnlyActive,
        [switch]$NoLicense,
        [string]$OutputPath
    )
    
    Write-Host "条件に基づくユーザー検索を開始します..." -ForegroundColor Cyan
    
    # フィルター条件の構築
    $filters = @()
    
    if ($Department) { $filters += "department eq '$Department'" }
    if ($JobTitle) { $filters += "jobTitle eq '$JobTitle'" }
    if ($OnlyActive) { $filters += "accountEnabled eq true" }
    if ($CreatedAfter) { 
        $dateFilter = $CreatedAfter.ToString("yyyy-MM-ddTHH:mm:ssZ")
        $filters += "createdDateTime ge $dateFilter" 
    }
    
    $filter = if ($filters.Count -gt 0) { $filters -join " and " } else { $null }
    
    try {
        # ユーザー取得
        $users = if ($filter) {
            Get-MgUser -All -Filter $filter
        } else {
            Get-MgUser -All
        }
        
        # ライセンス情報の追加取得
        $userDetails = foreach ($user in $users) {
            $licenses = Get-MgUserLicenseDetail -UserId $user.Id -ErrorAction SilentlyContinue
            $licenseNames = if ($licenses) { 
                ($licenses | ForEach-Object { $_.SkuPartNumber }) -join "; " 
            } else { 
                "ライセンスなし" 
            }
            
            # ライセンス未割り当てフィルター
            if ($NoLicense -and $licenses) { continue }
            
            [PSCustomObject]@{
                DisplayName = $user.DisplayName
                UserPrincipalName = $user.UserPrincipalName
                Department = $user.Department
                JobTitle = $user.JobTitle
                OfficeLocation = $user.OfficeLocation
                BusinessPhones = ($user.BusinessPhones -join "; ")
                AccountEnabled = $user.AccountEnabled
                CreatedDateTime = $user.CreatedDateTime
                LastSignInDateTime = $user.SignInActivity.LastSignInDateTime
                Licenses = $licenseNames
            }
        }
        
        Write-Host "検索完了: $($userDetails.Count)名のユーザーが条件に合致" -ForegroundColor Green
        
        # 結果表示
        $userDetails | Format-Table DisplayName, UserPrincipalName, Department, JobTitle, AccountEnabled, Licenses -AutoSize
        
        # CSV出力
        if ($OutputPath) {
            $userDetails | Export-Csv $OutputPath -Encoding UTF8 -NoTypeInformation
            Write-Host "検索結果をCSVファイルに出力しました: $OutputPath" -ForegroundColor Green
        }
        
        return $userDetails
        
    } catch {
        Write-Error "ユーザー検索中にエラーが発生しました: $($_.Exception.Message)"
    }
}
```

### ライセンス使用状況の詳細分析

#### ユーザー別ライセンス情報の一括取得

**🎯 スクリプトの目的**
全ユーザーのライセンス割り当て状況を詳細に分析し、コスト最適化や使用状況監査のためのデータを提供します。未使用ライセンスの特定、重複割り当ての検出、利用率の低いユーザーの識別が可能です。

**📝 Usage**
```powershell
# 全ユーザーのライセンス状況分析
Get-UserLicenseReport -OutputPath "C:\reports\license_analysis.csv"

# 特定ライセンスの使用者のみ
Get-UserLicenseReport -LicenseFilter "A1" -OutputPath "C:\reports\a1_users.csv"

# 未使用・重複ライセンスの検出
Get-UserLicenseReport -AnalyzeUsage -DetectDuplicates -OutputPath "C:\reports\license_optimization.csv"

# サインイン状況付きの詳細分析
Get-UserLicenseReport -IncludeSignInActivity -DaysInactive 90 -OutputPath "C:\reports\inactive_licensed_users.csv"
```

**⚙️ パラメータ**
- `-LicenseFilter`: 特定ライセンスSKUでフィルター
- `-AnalyzeUsage`: 使用状況の詳細分析を実行
- `-DetectDuplicates`: 重複ライセンス割り当ての検出
- `-IncludeSignInActivity`: サインイン履歴を含める
- `-DaysInactive`: 指定日数以上サインインがないユーザーを特定
- `-OutputPath`: レポートのCSV出力先

```powershell
function Get-UserLicenseReport {
    param(
        [string]$LicenseFilter,
        [switch]$AnalyzeUsage,
        [switch]$DetectDuplicates,
        [switch]$IncludeSignInActivity,
        [int]$DaysInactive = 90,
        [string]$OutputPath
    )
    
    Write-Host "ライセンス使用状況レポートの生成を開始します..." -ForegroundColor Cyan
    
    try {
        # 利用可能ライセンス情報の取得
        $availableLicenses = Get-MgSubscribedSku | Select-Object SkuPartNumber, SkuId, ConsumedUnits, @{
            Name="AvailableUnits"
            Expression={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}
        }
        
        Write-Host "ライセンス情報取得完了: $($availableLicenses.Count)種類" -ForegroundColor Green
        
        # 全ユーザーのライセンス情報取得
        $users = Get-MgUser -All
        $licenseReport = @()
        
        $counter = 0
        foreach ($user in $users) {
            $counter++
            if ($counter % 100 -eq 0) {
                Write-Progress -Activity "ライセンス情報分析中" -Status "$counter/$($users.Count)" -PercentComplete (($counter / $users.Count) * 100)
            }
            
            # ユーザーのライセンス詳細取得
            $userLicenses = Get-MgUserLicenseDetail -UserId $user.Id -ErrorAction SilentlyContinue
            
            # サインイン情報取得（オプション）
            $lastSignIn = $null
            $isInactive = $false
            if ($IncludeSignInActivity) {
                $signInActivity = $user.SignInActivity
                $lastSignIn = $signInActivity.LastSignInDateTime
                if ($lastSignIn) {
                    $daysSinceSignIn = (Get-Date) - [datetime]$lastSignIn
                    $isInactive = $daysSinceSignIn.TotalDays -gt $DaysInactive
                }
            }
            
            if ($userLicenses) {
                foreach ($license in $userLicenses) {
                    # ライセンスフィルター適用
                    if ($LicenseFilter -and $license.SkuPartNumber -notlike "*$LicenseFilter*") {
                        continue
                    }
                    
                    $licenseInfo = [PSCustomObject]@{
                        DisplayName = $user.DisplayName
                        UserPrincipalName = $user.UserPrincipalName
                        Department = $user.Department
                        JobTitle = $user.JobTitle
                        AccountEnabled = $user.AccountEnabled
                        LicenseSku = $license.SkuPartNumber
                        LicenseSkuId = $license.SkuId
                        AssignedDateTime = $user.CreatedDateTime
                        LastSignInDateTime = $lastSignIn
                        IsInactiveUser = $isInactive
                        DaysSinceLastSignIn = if ($lastSignIn) { [math]::Round((Get-Date - [datetime]$lastSignIn).TotalDays) } else { $null }
                    }
                    
                    $licenseReport += $licenseInfo
                }
            } else {
                # ライセンス未割り当てユーザー
                $licenseInfo = [PSCustomObject]@{
                    DisplayName = $user.DisplayName
                    UserPrincipalName = $user.UserPrincipalName
                    Department = $user.Department
                    JobTitle = $user.JobTitle
                    AccountEnabled = $user.AccountEnabled
                    LicenseSku = "ライセンス未割り当て"
                    LicenseSkuId = $null
                    AssignedDateTime = $user.CreatedDateTime
                    LastSignInDateTime = $lastSignIn
                    IsInactiveUser = $isInactive
                    DaysSinceLastSignIn = if ($lastSignIn) { [math]::Round((Get-Date - [datetime]$lastSignIn).TotalDays) } else { $null }
                }
                
                $licenseReport += $licenseInfo
            }
        }
        
        Write-Progress -Activity "ライセンス情報分析中" -Completed
        
        # 分析結果の表示
        Write-Host "`n=== ライセンス使用状況サマリー ===" -ForegroundColor Magenta
        
        $licenseSummary = $licenseReport | Where-Object {$_.LicenseSku -ne "ライセンス未割り当て"} | 
                         Group-Object LicenseSku | 
                         Select-Object Name, Count
        
        $licenseSummary | Format-Table -AutoSize
        
        # 重複ライセンス検出（オプション）
        if ($DetectDuplicates) {
            $duplicateUsers = $licenseReport | Group-Object UserPrincipalName | Where-Object {$_.Count -gt 1}
            if ($duplicateUsers) {
                Write-Host "`n⚠️ 複数ライセンス保有ユーザー: $($duplicateUsers.Count)名" -ForegroundColor Yellow
                $duplicateUsers | ForEach-Object {
                    Write-Host "  $($_.Name): $($_.Count)個のライセンス" -ForegroundColor Yellow
                }
            }
        }
        
        # 非アクティブユーザー分析（オプション）
        if ($IncludeSignInActivity) {
            $inactiveCount = ($licenseReport | Where-Object {$_.IsInactiveUser -eq $true}).Count
            Write-Host "`n📊 $DaysInactive 日以上未サインインのライセンス保有ユーザー: $inactiveCount 名" -ForegroundColor Yellow
        }
        
        # CSV出力
        if ($OutputPath) {
            $licenseReport | Export-Csv $OutputPath -Encoding UTF8 -NoTypeInformation
            Write-Host "`nライセンスレポートをCSVファイルに出力しました: $OutputPath" -ForegroundColor Green
        }
        
        return $licenseReport
        
    } catch {
        Write-Error "ライセンス分析中にエラーが発生しました: $($_.Exception.Message)"
    }
}
```

### 大規模テナント対応の最適化

#### 効率的なデータ取得とパフォーマンス最適化

**🎯 スクリプトの目的**
数万人規模の大規模テナントでも効率的にユーザーデータを取得できるよう、ページング処理、並列取得、メモリ効率化を実装します。処理時間の短縮とシステム負荷の軽減を実現し、大規模環境での運用をサポートします。

**📝 Usage**
```powershell
# 大規模テナント用の最適化取得
Get-LargeScaleTenantUsers -PageSize 1000 -ParallelJobs 5 -OutputPath "C:\reports\large_tenant_users.csv"

# メモリ効率重視の処理
Get-LargeScaleTenantUsers -MemoryOptimized -StreamToFile "C:\reports\streaming_users.csv"

# 特定の期間で分割取得
Get-LargeScaleTenantUsers -SplitByCreationDate -StartDate "2023-01-01" -EndDate "2023-12-31"
```

**⚙️ パラメータ**
- `-PageSize`: 一度に取得するユーザー数（デフォルト1000）
- `-ParallelJobs`: 並列ジョブ数（デフォルト3）
- `-MemoryOptimized`: メモリ効率化モード
- `-StreamToFile`: ストリーミング出力先ファイル
- `-SplitByCreationDate`: 作成日期間での分割取得
- `-StartDate` / `-EndDate`: 取得対象期間

```powershell
function Get-LargeScaleTenantUsers {
    param(
        [int]$PageSize = 1000,
        [int]$ParallelJobs = 3,
        [switch]$MemoryOptimized,
        [string]$StreamToFile,
        [switch]$SplitByCreationDate,
        [datetime]$StartDate,
        [datetime]$EndDate,
        [string]$OutputPath
    )
    
    Write-Host "大規模テナント用ユーザー取得を開始します..." -ForegroundColor Cyan
    Write-Host "設定: PageSize=$PageSize, ParallelJobs=$ParallelJobs" -ForegroundColor Yellow
    
    try {
        if ($SplitByCreationDate -and $StartDate -and $EndDate) {
            # 期間分割での取得
            $filter = "createdDateTime ge $($StartDate.ToString('yyyy-MM-ddTHH:mm:ssZ')) and createdDateTime le $($EndDate.ToString('yyyy-MM-ddTHH:mm:ssZ'))"
            Write-Host "期間フィルター適用: $($StartDate.ToString('yyyy-MM-dd')) ～ $($EndDate.ToString('yyyy-MM-dd'))" -ForegroundColor Yellow
        } else {
            $filter = $null
        }
        
        if ($MemoryOptimized -and $StreamToFile) {
            # ストリーミング処理（メモリ効率重視）
            Write-Host "ストリーミングモードでファイル出力中..." -ForegroundColor Cyan
            
            # CSVヘッダーの書き込み
            "DisplayName,UserPrincipalName,Department,JobTitle,AccountEnabled,CreatedDateTime,LastSignInDateTime" | 
                Out-File $StreamToFile -Encoding UTF8
            
            # ページング処理でストリーミング出力
            $skip = 0
            $totalProcessed = 0
            
            do {
                $batch = Get-MgUser -Top $PageSize -Skip $skip -Filter $filter
                
                foreach ($user in $batch) {
                    $lastSignIn = $user.SignInActivity.LastSignInDateTime
                    $csvLine = "`"$($user.DisplayName)`",`"$($user.UserPrincipalName)`",`"$($user.Department)`",`"$($user.JobTitle)`",$($user.AccountEnabled),`"$($user.CreatedDateTime)`",`"$lastSignIn`""
                    $csvLine | Add-Content $StreamToFile -Encoding UTF8
                }
                
                $totalProcessed += $batch.Count
                $skip += $PageSize
                
                Write-Progress -Activity "ストリーミング出力中" -Status "処理済み: $totalProcessed 名" -PercentComplete -1
                
            } while ($batch.Count -eq $PageSize)
            
            Write-Progress -Activity "ストリーミング出力中" -Completed
            Write-Host "ストリーミング出力完了: $totalProcessed 名のユーザー → $StreamToFile" -ForegroundColor Green
            
        } else {
            # 通常の並列処理
            $allUsers = @()
            
            # 総ユーザー数の概算
            $userCount = (Get-MgUser -Top 1 -Filter $filter -CountVariable totalCount).Count
            Write-Host "概算ユーザー数: $totalCount 名" -ForegroundColor Yellow
            
            # 並列処理でのページング取得
            $batches = 0..[math]::Ceiling($totalCount / $PageSize) - 1
            
            $allUsers = $batches | ForEach-Object -ThrottleLimit $ParallelJobs -Parallel {
                $skip = $_ * $using:PageSize
                $pageSize = $using:PageSize
                $filter = $using:filter
                
                try {
                    Get-MgUser -Top $pageSize -Skip $skip -Filter $filter | 
                        Select-Object DisplayName, UserPrincipalName, Department, JobTitle, 
                                    AccountEnabled, CreatedDateTime, 
                                    @{Name="LastSignInDateTime"; Expression={$_.SignInActivity.LastSignInDateTime}}
                } catch {
                    Write-Warning "Batch $_ failed: $($_.Exception.Message)"
                    @()
                }
            }
            
            Write-Host "並列取得完了: $($allUsers.Count) 名のユーザー" -ForegroundColor Green
            
            # 結果表示（サンプル）
            $allUsers | Select-Object -First 10 | Format-Table -AutoSize
            
            if ($OutputPath) {
                $allUsers | Export-Csv $OutputPath -Encoding UTF8 -NoTypeInformation
                Write-Host "結果をCSVファイルに出力しました: $OutputPath" -ForegroundColor Green
            }
            
            return $allUsers
        }
        
    } catch {
        Write-Error "大規模取得中にエラーが発生しました: $($_.Exception.Message)"
    }
}
```

---

## 📄 CSVテンプレートの設計と準備

### 基本CSVテンプレートの構造

#### 最小構成（必須項目のみ）

```csv
DisplayName,UserPrincipalName,FirstName,LastName,Password,Department,JobTitle,LicenseSku
田中 太郎,t.tanaka@school.edu.jp,太郎,田中,TempPass123!,情報科学部,学生,Microsoft 365 A1 for students
佐藤 花子,h.sato@school.edu.jp,花子,佐藤,TempPass456!,数学科,准教授,Microsoft 365 A3 for faculty
山田 一郎,i.yamada@school.edu.jp,一郎,山田,TempPass789!,事務局,課長,Microsoft 365 A3 for faculty
```

#### 完全版（推奨項目含む）

```csv
DisplayName,UserPrincipalName,FirstName,LastName,Password,Department,JobTitle,OfficeLocation,PhoneNumber,Manager,LicenseSku,SecurityGroups,UsageLocation,PreferredLanguage,ForcePasswordChange
田中 太郎,t.tanaka@school.edu.jp,太郎,田中,TempPass123!,情報科学部,学生,1号館201,03-1234-5678,,Microsoft 365 A1 for students,Students;IT-Users,JP,ja-JP,true
佐藤 花子,h.sato@school.edu.jp,花子,佐藤,TempPass456!,数学科,准教授,2号館301,03-1234-5679,yamada@school.edu.jp,Microsoft 365 A3 for faculty,Faculty;Math-Dept,JP,ja-JP,true
山田 一郎,i.yamada@school.edu.jp,一郎,山田,TempPass789!,事務局,課長,本館101,03-1234-5680,,Microsoft 365 A3 for faculty,Staff;Admins,JP,ja-JP,true
```

### CSVテンプレート詳細仕様

#### 必須フィールド

| フィールド名 | 説明 | 形式・制約 | 例 |
|-------------|------|------------|-----|
| **DisplayName** | 表示名 | 全角64文字以内 | `田中 太郎` |
| **UserPrincipalName** | ユーザープリンシパル名 | メール形式、組織内一意 | `t.tanaka@school.edu.jp` |
| **FirstName** | 名 | 全角32文字以内 | `太郎` |
| **LastName** | 姓 | 全角32文字以内 | `田中` |
| **Password** | 初期パスワード | 8文字以上、複雑性要件満たす | `TempPass123!` |
| **LicenseSku** | ライセンス | SKU名または表示名 | `Microsoft 365 A1 for students` |

#### 推奨フィールド

| フィールド名 | 説明 | 形式・制約 | 例 |
|-------------|------|------------|-----|
| **Department** | 部署 | 全角64文字以内 | `情報科学部` |
| **JobTitle** | 役職 | 全角64文字以内 | `准教授` |
| **Manager** | 上司 | UPN形式 | `manager@school.edu.jp` |
| **SecurityGroups** | セキュリティグループ | セミコロン区切り | `Students;IT-Users` |
| **UsageLocation** | 利用地域 | ISO 3166-1 alpha-2 | `JP` |
| **ForcePasswordChange** | 初回PW変更強制 | true/false | `true` |

### CSVデータの検証・クリーニング

#### Excel/PowerShellでの事前検証

**🎯 スクリプトの目的**
CSVファイル内のユーザーデータが Microsoft 365 の要件を満たしているかを事前にチェックし、一括処理での失敗を防ぎます。必須フィールドの存在確認、UPN重複チェック、パスワード複雑性検証、メールアドレス形式確認を自動的に実行します。

**📝 Usage**
```powershell
# 基本的な使用方法
$csvPath = "C:\path\to\your\users.csv"
.\Validate-UserCSV.ps1 -CsvPath $csvPath

# 詳細レポートを出力する場合
.\Validate-UserCSV.ps1 -CsvPath $csvPath -Detailed -OutputReport "C:\temp\validation_report.txt"
```

**⚙️ パラメータ**
- `$csvPath`: 検証対象のCSVファイルパス
- `-Detailed`: 詳細な検証結果を表示
- `-OutputReport`: 検証結果をファイルに出力

```powershell
# CSVファイルの基本検証スクリプト
$csvPath = "C:\temp\users.csv"
$users = Import-Csv $csvPath -Encoding UTF8

# 1. 必須項目の存在確認
$requiredFields = @('DisplayName', 'UserPrincipalName', 'FirstName', 'LastName', 'Password', 'LicenseSku')
$csvHeaders = ($users | Get-Member -MemberType NoteProperty).Name

foreach ($field in $requiredFields) {
    if ($field -notin $csvHeaders) {
        Write-Error "必須フィールド '$field' が見つかりません"
    }
}

# 2. UPN重複チェック
$duplicateUPNs = $users | Group-Object UserPrincipalName | Where-Object {$_.Count -gt 1}
if ($duplicateUPNs) {
    Write-Warning "重複するUPNが見つかりました:"
    $duplicateUPNs.Name
}

# 3. パスワード複雑性チェック
$weakPasswords = $users | Where-Object {$_.Password.Length -lt 8 -or $_.Password -notmatch '[A-Z]' -or $_.Password -notmatch '[a-z]' -or $_.Password -notmatch '\d'}
if ($weakPasswords) {
    Write-Warning "パスワード要件を満たしないユーザー: $($weakPasswords.Count)名"
}

# 4. メールアドレス形式チェック
$invalidEmails = $users | Where-Object {$_.UserPrincipalName -notmatch '^[^@]+@[^@]+\.[^@]+$'}
if ($invalidEmails) {
    Write-Warning "無効なメールアドレス形式: $($invalidEmails.Count)名"
}

Write-Host "検証完了: 総ユーザー数 $($users.Count)名" -ForegroundColor Green
```

#### データクリーニングの自動化

**🎯 スクリプトの目的**
CSVデータの品質を向上させるため、氏名の表記統一、UPNの小文字化、部署名の正規化、ライセンスSKUの標準化などを自動的に実行します。これにより、一括処理時のエラーを大幅に削減できます。

**📝 Usage**
```powershell
# 基本的なデータクリーニング
$users = Import-Csv "C:\temp\users.csv" -Encoding UTF8
$cleanedUsers = Clean-UserData -Users $users
$cleanedUsers | Export-Csv "C:\temp\users_cleaned.csv" -Encoding UTF8 -NoTypeInformation

# 部署マッピングファイルを指定する場合
$cleanedUsers = Clean-UserData -Users $users -DepartmentMappingFile "C:\temp\dept_mapping.csv"
```

**⚙️ パラメータ**
- `-Users`: クリーニング対象のユーザーデータ配列
- `-DepartmentMappingFile`: 部署名正規化用のマッピングファイル（オプション）

```powershell
# CSVデータのクリーニング・標準化
function Clean-UserData {
    param($Users)
    
    foreach ($user in $Users) {
        # 1. 氏名の正規化（全角スペースを半角に統一）
        $user.DisplayName = $user.DisplayName -replace '　', ' '
        
        # 2. UPNの小文字化
        $user.UserPrincipalName = $user.UserPrincipalName.ToLower()
        
        # 3. 部署名の統一（マスターデータとの照合）
        $user.Department = Normalize-Department $user.Department
        
        # 4. ライセンスSKUの正規化
        $user.LicenseSku = Convert-ToLicenseSku $user.LicenseSku
    }
    
    return $Users
}

# 部署名正規化の例
function Normalize-Department {
    param($DeptName)
    
    $deptMappings = @{
        "情報科学" = "情報科学部"
        "数学" = "数学科"
        "事務" = "事務局"
        # 他の部署マッピング...
    }
    
    if ($deptMappings.ContainsKey($DeptName)) {
        return $deptMappings[$DeptName]
    }
    return $DeptName
}
```

---

## 🖥️ Microsoft 365管理センターでの一括操作

### 方法1: 管理センターのCSVインポート機能

#### Step 1: CSVファイルの準備

1. **Excelで作成**する場合の注意点：
   ```markdown
   ⚠️ Excel使用時の注意事項
   - 文字コード: UTF-8 with BOM で保存
   - 改行コード: Windows (CRLF) を使用
   - 特殊文字: カンマ・引用符をデータに含まない
   - 空白行: ファイル末尾に空白行を作らない
   ```

2. **推奨保存形式**:
   - ファイル名: `users_YYYYMMDD.csv`（例: `users_20250601.csv`）
   - エンコーディング: `UTF-8`
   - 最大行数: 2000行（推奨上限）

#### Step 2: 管理センターでのインポート実行

1. **Microsoft 365管理センター** ([https://admin.microsoft.com](https://admin.microsoft.com)) にアクセス
2. **「ユーザー」** → **「アクティブなユーザー」** をクリック
3. **「ユーザーの追加」** → **「複数のユーザーを追加」** を選択

![Microsoft 365管理センター - 一括ユーザー追加](../../assets/images/screenshots/admin-center/bulk-user-import.png)

4. **CSVファイルのアップロード**:
   ```markdown
   📤 アップロード手順
   1. 「CSVファイルをアップロード」をクリック
   2. 準備したCSVファイルを選択
   3. 「次へ」をクリックしてプレビュー確認
   4. フィールドマッピングの確認・調整
   ```

5. **フィールドマッピングの確認**:

   | CSVヘッダー | Microsoft 365フィールド | 必須 |
   |-------------|------------------------|------|
   | DisplayName | 表示名 | ✅ |
   | UserPrincipalName | ユーザー名 | ✅ |
   | FirstName | 名 | ✅ |
   | LastName | 姓 | ✅ |
   | Department | 部署 | ⭕ |
   | JobTitle | 役職 | ⭕ |

6. **ライセンス一括割り当て**:
   - ✅ **同一ライセンスを全員に割り当て**: 教育機関の学生など
   - ✅ **CSVの値に基づいて個別割り当て**: 複数職種混在時

#### Step 3: 実行結果の確認

```markdown
📊 処理結果レポートの確認項目
- ✅ 成功数: 正常に作成されたユーザー数
- ⚠️ 警告数: 作成されたが一部設定に問題があるユーザー数
- ❌ 失敗数: 作成に失敗したユーザー数
- 📁 詳細ログ: エラー詳細とUPNの一覧をダウンロード
```

### 方法2: PowerShellによる段階的処理

より柔軟で確実な処理のため、PowerShellを使った段階的アプローチを推奨します。

#### Step 1: 環境準備とモジュールインストール

**🎯 スクリプトの目的**
Microsoft Graph PowerShell SDK をインストールし、Microsoft 365 テナントに必要な権限で接続します。ユーザー管理、グループ管理、ディレクトリ操作に必要なスコープを取得し、後続の一括処理を実行できる環境を整備します。

**📝 Usage**
```powershell
# 初回セットアップ（管理者権限で実行）
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Install-Module Microsoft.Graph -Scope CurrentUser -Force

# 日常的な接続
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All", "Directory.ReadWrite.All"

# 接続状態の確認
Get-MgContext | Select-Object Account, Scopes, Environment
```

**⚙️ 重要な注意事項**
- 初回実行時は管理者権限が必要
- スコープは必要最小限に限定することを推奨
- 本番環境接続前にテスト環境での動作確認を実施

```powershell
# Microsoft Graph PowerShell モジュールのインストール
Install-Module Microsoft.Graph -Scope CurrentUser -Force

# 必要なスコープで接続
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All", "Directory.ReadWrite.All"

# 接続確認
Get-MgContext | Select-Object -Property Scopes, Account, Environment
```

#### Step 2: CSVファイルの読み込みと前処理

**🎯 スクリプトの目的**
CSVファイルからユーザーデータを安全に読み込み、利用可能なライセンス数と必要数を比較してリソース不足を事前に検出します。処理開始前にライセンス不足による失敗を防ぎ、確実な一括処理を実現します。

**📝 Usage**
```powershell
# 基本的な読み込みと検証
$csvPath = "C:\temp\users.csv"
.\Check-ResourcesBeforeBulkOperation.ps1 -CsvPath $csvPath

# ライセンス不足時の自動調達通知付き
.\Check-ResourcesBeforeBulkOperation.ps1 -CsvPath $csvPath -NotifyOnShortage -EmailTo "admin@company.com"
```

**⚙️ パラメータ**
- `$csvPath`: 処理対象のCSVファイルパス
- `-NotifyOnShortage`: ライセンス不足時の通知有効化
- `-EmailTo`: 通知先メールアドレス

```powershell
# CSVファイルの読み込み（UTF-8対応）
$csvPath = "C:\temp\users.csv"
$users = Import-Csv $csvPath -Encoding UTF8

Write-Host "読み込み完了: $($users.Count)名のユーザーデータ" -ForegroundColor Green

# 利用可能ライセンスの確認
$availableLicenses = Get-MgSubscribedSku | Select-Object SkuPartNumber, ConsumedUnits, @{Name="AvailableUnits";Expression={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}}
$availableLicenses | Format-Table

# ライセンス不足の確認
$requiredLicenses = $users | Group-Object LicenseSku | Select-Object Name, Count
foreach ($license in $requiredLicenses) {
    $available = ($availableLicenses | Where-Object {$_.SkuPartNumber -like "*$($license.Name)*"}).AvailableUnits
    if ($available -lt $license.Count) {
        Write-Warning "ライセンス不足: $($license.Name) - 必要:$($license.Count), 利用可能:$available"
    }
}
```

#### Step 3: 段階的ユーザー作成（推奨手法）

**🎯 スクリプトの目的**
大量のユーザーを安全に作成するため、指定されたバッチサイズに分割して段階的に処理します。各ユーザーの作成、ライセンス割り当て、グループ追加を自動化し、エラー発生時も処理を継続して最終的な結果レポートを提供します。タイムアウトや一時的な障害に対する耐性を持ち、大規模展開での信頼性を確保します。

**📝 Usage**
```powershell
# 基本的な一括ユーザー作成（50名ずつ処理）
$users = Import-Csv "C:\temp\users.csv" -Encoding UTF8
.\Invoke-BulkUserCreation.ps1 -Users $users -BatchSize 50 -DelayBetweenBatches 30

# 大規模処理用（100名ずつ、短い間隔）
.\Invoke-BulkUserCreation.ps1 -Users $users -BatchSize 100 -DelayBetweenBatches 15 -LogPath "C:\logs\bulk_creation.log"

# テスト実行（小規模で動作確認）
.\Invoke-BulkUserCreation.ps1 -Users $users[0..9] -BatchSize 5 -DelayBetweenBatches 10 -TestMode
```

**⚙️ パラメータ**
- `-Users`: 作成対象のユーザーデータ配列
- `-BatchSize`: 一度に処理するユーザー数（推奨: 25-100）
- `-DelayBetweenBatches`: バッチ間の待機時間（秒）
- `-LogPath`: 詳細ログの出力先パス
- `-TestMode`: テスト実行モード（実際の作成は行わない）

**💡 推奨設定**
- 小規模（〜100名）: BatchSize=25, Delay=15秒
- 中規模（100-500名）: BatchSize=50, Delay=30秒  
- 大規模（500名〜）: BatchSize=75, Delay=45秒

```powershell
# 段階的処理の設定
$batchSize = 50  # 一度に処理するユーザー数
$delayBetweenBatches = 30  # バッチ間の待機時間（秒）

# 処理結果記録用
$results = @{
    Success = @()
    Failed = @()
    Warnings = @()
}

# バッチ処理の実行
for ($i = 0; $i -lt $users.Count; $i += $batchSize) {
    $batch = $users[$i..[Math]::Min($i + $batchSize - 1, $users.Count - 1)]
    
    Write-Host "バッチ $([Math]::Floor($i / $batchSize) + 1) 処理開始: $($batch.Count)名" -ForegroundColor Cyan
    
    foreach ($user in $batch) {
        try {
            # ユーザー作成パラメータの構築
            $userParams = @{
                DisplayName = $user.DisplayName
                UserPrincipalName = $user.UserPrincipalName
                GivenName = $user.FirstName
                Surname = $user.LastName
                Department = $user.Department
                JobTitle = $user.JobTitle
                UsageLocation = if ($user.UsageLocation) { $user.UsageLocation } else { "JP" }
                AccountEnabled = $true
                PasswordProfile = @{
                    Password = $user.Password
                    ForceChangePasswordNextSignIn = [bool]$user.ForcePasswordChange
                }
            }
            
            # ユーザーの作成
            $newUser = New-MgUser @userParams
            
            # ライセンス割り当て
            if ($user.LicenseSku) {
                $licenseSku = Get-MgSubscribedSku | Where-Object {$_.SkuPartNumber -like "*$($user.LicenseSku)*" -or $_.SkuId -eq $user.LicenseSku}
                if ($licenseSku) {
                    Set-MgUserLicense -UserId $newUser.Id -AddLicenses @{SkuId = $licenseSku.SkuId} -RemoveLicenses @()
                }
            }
            
            # セキュリティグループへの追加
            if ($user.SecurityGroups) {
                $groups = $user.SecurityGroups -split ';'
                foreach ($groupName in $groups) {
                    $group = Get-MgGroup -Filter "displayName eq '$groupName'"
                    if ($group) {
                        New-MgGroupMember -GroupId $group.Id -DirectoryObjectId $newUser.Id
                    }
                }
            }
            
            $results.Success += $user.UserPrincipalName
            Write-Host "✅ 成功: $($user.UserPrincipalName)" -ForegroundColor Green
            
        } catch {
            $results.Failed += @{
                UserPrincipalName = $user.UserPrincipalName
                Error = $_.Exception.Message
            }
            Write-Host "❌ 失敗: $($user.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
        }
    }
    
    # バッチ間の待機
    if ($i + $batchSize -lt $users.Count) {
        Write-Host "待機中... ($delayBetweenBatches秒)" -ForegroundColor Yellow
        Start-Sleep -Seconds $delayBetweenBatches
    }
}

# 処理結果サマリー
Write-Host "`n=== 処理結果サマリー ===" -ForegroundColor Magenta
Write-Host "成功: $($results.Success.Count)名" -ForegroundColor Green
Write-Host "失敗: $($results.Failed.Count)名" -ForegroundColor Red

# 失敗したユーザーの詳細出力
if ($results.Failed.Count -gt 0) {
    Write-Host "`n失敗したユーザー:" -ForegroundColor Red
    $results.Failed | ForEach-Object {
        Write-Host "  - $($_.UserPrincipalName): $($_.Error)" -ForegroundColor Red
    }
}
```

---

## 🔄 ユーザー情報の一括編集・更新

### 既存ユーザーのプロパティ一括更新

教育機関では学期ごと、企業では組織変更時に大量のユーザー情報更新が発生します。

#### 更新用CSVテンプレート

```csv
UserPrincipalName,DisplayName,Department,JobTitle,Manager,OfficeLocation,PhoneNumber
t.tanaka@school.edu.jp,田中 太郎,情報工学科,大学院生,prof.sato@school.edu.jp,3号館401,03-1234-5678
h.sato@school.edu.jp,佐藤 花子,数学科,教授,,2号館301,03-1234-5679
i.yamada@school.edu.jp,山田 一郎,事務局,係長,manager@school.edu.jp,本館101,03-1234-5680
```

#### PowerShellによる一括更新

**🎯 スクリプトの目的**
組織変更や人事異動に対応するため、既存ユーザーの情報（部署、役職、マネージャー、連絡先など）を CSVデータに基づいて一括更新します。空白フィールドは更新対象外とし、必要な項目のみを効率的に変更できます。

**📝 Usage**
```powershell
# 基本的な一括更新
$updates = Import-Csv "C:\temp\user_updates.csv" -Encoding UTF8
.\Update-BulkUserProperties.ps1 -Updates $updates

# ドライラン実行（実際の更新は行わず、変更内容のみ表示）
.\Update-BulkUserProperties.ps1 -Updates $updates -DryRun

# 特定の属性のみ更新
.\Update-BulkUserProperties.ps1 -Updates $updates -UpdateFields @("Department", "JobTitle") -LogPath "C:\logs\updates.log"
```

**⚙️ パラメータ**
- `-Updates`: 更新データを含むCSV配列
- `-DryRun`: 実際の更新を行わず、変更内容を表示
- `-UpdateFields`: 更新対象フィールドを限定
- `-LogPath`: 更新結果ログの出力先

**📋 対応するCSVフィールド**
- `UserPrincipalName`: 更新対象ユーザー（必須）
- `DisplayName`: 表示名
- `Department`: 部署名  
- `JobTitle`: 役職
- `Manager`: 上司（UPN形式）
- `OfficeLocation`: オフィス所在地
- `PhoneNumber`: 電話番号

```powershell
# 更新データの読み込み
$updatesPath = "C:\temp\user_updates.csv"
$updates = Import-Csv $updatesPath -Encoding UTF8

Write-Host "更新対象: $($updates.Count)名のユーザー" -ForegroundColor Green

foreach ($update in $updates) {
    try {
        # 既存ユーザーの確認
        $existingUser = Get-MgUser -Filter "userPrincipalName eq '$($update.UserPrincipalName)'"
        
        if (-not $existingUser) {
            Write-Warning "ユーザーが見つかりません: $($update.UserPrincipalName)"
            continue
        }
        
        # 更新パラメータの構築（空白フィールドは更新しない）
        $updateParams = @{}
        
        if (![string]::IsNullOrWhiteSpace($update.DisplayName)) { $updateParams.DisplayName = $update.DisplayName }
        if (![string]::IsNullOrWhiteSpace($update.Department)) { $updateParams.Department = $update.Department }
        if (![string]::IsNullOrWhiteSpace($update.JobTitle)) { $updateParams.JobTitle = $update.JobTitle }
        if (![string]::IsNullOrWhiteSpace($update.OfficeLocation)) { $updateParams.OfficeLocation = $update.OfficeLocation }
        if (![string]::IsNullOrWhiteSpace($update.PhoneNumber)) { $updateParams.BusinessPhones = @($update.PhoneNumber) }
        
        # マネージャーの設定（別途処理）
        if (![string]::IsNullOrWhiteSpace($update.Manager)) {
            $manager = Get-MgUser -Filter "userPrincipalName eq '$($update.Manager)'"
            if ($manager) {
                $updateParams.Manager = @{ "@odata.id" = "https://graph.microsoft.com/v1.0/users/$($manager.Id)" }
            }
        }
        
        # ユーザー情報の更新
        if ($updateParams.Count -gt 0) {
            Update-MgUser -UserId $existingUser.Id @updateParams
            Write-Host "✅ 更新完了: $($update.UserPrincipalName)" -ForegroundColor Green
        } else {
            Write-Host "⚠️ 更新項目なし: $($update.UserPrincipalName)" -ForegroundColor Yellow
        }
        
    } catch {
        Write-Host "❌ 更新失敗: $($update.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

### ライセンス割り当ての一括変更

学生の卒業時や教職員の異動時にライセンスの一括変更が必要になります。

**🎯 スクリプトの目的**
ユーザーの役割変更（学生→卒業生、一般社員→管理職など）に応じて、ライセンスの削除・追加を一括実行します。コスト最適化とアクセス権限の適正化を同時に実現し、手動作業による漏れやミスを防止します。

**📝 Usage**
```powershell
# 基本的なライセンス一括変更
$licenseChanges = Import-Csv "C:\temp\license_changes.csv" -Encoding UTF8
.\Update-BulkUserLicenses.ps1 -LicenseChanges $licenseChanges

# 卒業生処理（ライセンス削除のみ）
.\Update-BulkUserLicenses.ps1 -LicenseChanges $licenseChanges -RemoveOnly -Reason "Graduation"

# 段階的実行（エラー時の影響を最小化）
.\Update-BulkUserLicenses.ps1 -LicenseChanges $licenseChanges -BatchSize 25 -DelayBetweenBatches 15
```

**⚙️ パラメータ**
- `-LicenseChanges`: ライセンス変更データのCSV配列
- `-RemoveOnly`: ライセンス削除のみ実行
- `-BatchSize`: 一度に処理するユーザー数
- `-Reason`: 変更理由（ログ記録用）

**📋 CSV形式例**
```csv
UserPrincipalName,RemoveLicense,AddLicense,ChangeReason
graduated.student@school.edu.jp,Microsoft 365 A1 for students,,Graduation
new.faculty@school.edu.jp,,Microsoft 365 A3 for faculty,Promotion
transferred.staff@school.edu.jp,Microsoft 365 A3 for faculty,Microsoft 365 A1 for students,Department Transfer
```

```powershell
# ライセンス変更用CSVの例
# UserPrincipalName,RemoveLicense,AddLicense
# graduated.student@school.edu.jp,Microsoft 365 A1 for students,
# new.faculty@school.edu.jp,,Microsoft 365 A3 for faculty

$licenseChanges = Import-Csv "C:\temp\license_changes.csv" -Encoding UTF8

foreach ($change in $licenseChanges) {
    try {
        $user = Get-MgUser -Filter "userPrincipalName eq '$($change.UserPrincipalName)'"
        
        $addLicenses = @()
        $removeLicenses = @()
        
        # 削除するライセンス
        if (![string]::IsNullOrWhiteSpace($change.RemoveLicense)) {
            $removeSku = Get-MgSubscribedSku | Where-Object {$_.SkuPartNumber -like "*$($change.RemoveLicense)*"}
            if ($removeSku) {
                $removeLicenses += $removeSku.SkuId
            }
        }
        
        # 追加するライセンス
        if (![string]::IsNullOrWhiteSpace($change.AddLicense)) {
            $addSku = Get-MgSubscribedSku | Where-Object {$_.SkuPartNumber -like "*$($change.AddLicense)*"}
            if ($addSku) {
                $addLicenses += @{SkuId = $addSku.SkuId}
            }
        }
        
        # ライセンス変更の実行
        Set-MgUserLicense -UserId $user.Id -AddLicenses $addLicenses -RemoveLicenses $removeLicenses
        
        Write-Host "✅ ライセンス変更完了: $($change.UserPrincipalName)" -ForegroundColor Green
        
    } catch {
        Write-Host "❌ ライセンス変更失敗: $($change.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

---

## 🗑️ ユーザーアカウントの一括削除・無効化

### 安全な一括削除手順

卒業生や退職者のアカウント処理では、データ保護と段階的な処理が重要です。

#### Step 1: 削除前の安全確認

**🎯 スクリプトの目的**
ユーザーアカウントの削除前に、重要な管理者権限を持つユーザーや、データバックアップが未完了のユーザーを特定し、誤削除を防止します。削除対象の妥当性を検証し、安全な削除実行を支援します。

**📝 Usage**
```powershell
# 基本的な削除前チェック
$deletionList = Import-Csv "C:\temp\users_to_delete.csv" -Encoding UTF8
.\Confirm-SafeDeletion.ps1 -DeletionList $deletionList

# 詳細レポート付きチェック
.\Confirm-SafeDeletion.ps1 -DeletionList $deletionList -DetailedReport -OutputPath "C:\reports\deletion_safety_check.xlsx"

# 管理者権限チェックのみ
.\Confirm-SafeDeletion.ps1 -DeletionList $deletionList -AdminCheckOnly
```

**⚙️ パラメータ**
- `-DeletionList`: 削除対象ユーザーリストのCSV配列
- `-DetailedReport`: 詳細なレポートを生成
- `-OutputPath`: レポート出力先パス
- `-AdminCheckOnly`: 管理者権限チェックのみ実行

**📋 必要なCSVフィールド**
- `UserPrincipalName`: 削除対象ユーザー（必須）
- `DeletionReason`: 削除理由
- `DataBackupCompleted`: データバックアップ完了フラグ（Yes/No）

```powershell
# 削除対象ユーザーリストの準備
$deletionList = Import-Csv "C:\temp\users_to_delete.csv" -Encoding UTF8
# CSV形式: UserPrincipalName,DeletionReason,DataBackupCompleted

Write-Host "削除対象確認: $($deletionList.Count)名" -ForegroundColor Yellow

# 削除前の安全チェック
$safetyChecks = @()

foreach ($user in $deletionList) {
    $userObj = Get-MgUser -Filter "userPrincipalName eq '$($user.UserPrincipalName)'" -ErrorAction SilentlyContinue
    
    if (-not $userObj) {
        Write-Warning "ユーザーが存在しません: $($user.UserPrincipalName)"
        continue
    }
    
    # 管理者権限チェック
    $adminRoles = Get-MgUserMemberOf -UserId $userObj.Id | Where-Object {$_.AdditionalProperties.displayName -like "*admin*"}
    
    $checkResult = @{
        UserPrincipalName = $user.UserPrincipalName
        UserExists = $true
        HasAdminRoles = $adminRoles.Count -gt 0
        AdminRoles = ($adminRoles.AdditionalProperties.displayName -join ", ")
        LastSignIn = $userObj.SignInActivity.LastSignInDateTime
        DataBackupCompleted = $user.DataBackupCompleted -eq "Yes"
        DeletionReason = $user.DeletionReason
    }
    
    $safetyChecks += $checkResult
}

# 安全チェック結果の表示
$safetyChecks | Format-Table -AutoSize

# 管理者権限を持つユーザーの警告
$adminUsers = $safetyChecks | Where-Object {$_.HasAdminRoles}
if ($adminUsers) {
    Write-Warning "以下のユーザーは管理者権限を持っています:"
    $adminUsers | ForEach-Object {
        Write-Host "  - $($_.UserPrincipalName): $($_.AdminRoles)" -ForegroundColor Red
    }
}
```

#### Step 2: 段階的な削除実行

**🎯 スクリプトの目的**
ユーザーアカウントの削除を段階的に実行し、データ保護とコスト最適化を両立します。まずアカウント無効化とライセンス削除でコストを削減し、その後論理削除（30日間復旧可能）を実行する安全な2段階プロセスを提供します。

**📝 Usage**
```powershell
# 段階1のみ実行（アカウント無効化）
$deletionList = Import-Csv "C:\temp\users_to_delete.csv" -Encoding UTF8
.\Invoke-StagedUserDeletion.ps1 -DeletionList $deletionList -Stage 1

# 段階2まで実行（論理削除含む）
.\Invoke-StagedUserDeletion.ps1 -DeletionList $deletionList -Stage 2 -RequireConfirmation

# 完全自動実行（テスト環境用）
.\Invoke-StagedUserDeletion.ps1 -DeletionList $deletionList -Stage 2 -Force -LogPath "C:\logs\deletion.log"
```

**⚙️ パラメータ**
- `-DeletionList`: 削除対象ユーザーリスト
- `-Stage`: 実行段階（1=無効化のみ、2=論理削除まで）
- `-RequireConfirmation`: 各段階で確認プロンプトを表示
- `-Force`: 確認なしで実行（注意）
- `-LogPath`: 詳細ログの出力先

**⚠️ 削除段階の説明**
1. **段階1（即座に実行可能）**: アカウント無効化 + ライセンス削除
2. **段階2（30日間復旧可能）**: 論理削除実行
3. **完全削除（復旧不可能）**: 別途手動実行を推奨

```powershell
# 段階1: アカウント無効化（即座に実行可能）
Write-Host "=== 段階1: アカウント無効化 ===" -ForegroundColor Cyan

foreach ($user in $deletionList) {
    try {
        $userObj = Get-MgUser -Filter "userPrincipalName eq '$($user.UserPrincipalName)'" -ErrorAction SilentlyContinue
        
        if ($userObj) {
            # アカウント無効化
            Update-MgUser -UserId $userObj.Id -AccountEnabled $false
            
            # ライセンス削除（コスト削減）
            $userLicenses = Get-MgUserLicenseDetail -UserId $userObj.Id
            if ($userLicenses) {
                $removeSkuIds = $userLicenses.SkuId
                Set-MgUserLicense -UserId $userObj.Id -AddLicenses @() -RemoveLicenses $removeSkuIds
            }
            
            Write-Host "✅ 無効化完了: $($user.UserPrincipalName)" -ForegroundColor Green
        }
    } catch {
        Write-Host "❌ 無効化失敗: $($user.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}

# 段階2: 論理削除（30日間は復旧可能）
Write-Host "`n=== 段階2: 論理削除 ===" -ForegroundColor Cyan
Write-Host "注意: 削除されたユーザーは30日間は復旧可能です" -ForegroundColor Yellow

$confirmation = Read-Host "論理削除を実行しますか？ (yes/no)"
if ($confirmation -eq "yes") {
    foreach ($user in $deletionList) {
        try {
            $userObj = Get-MgUser -Filter "userPrincipalName eq '$($user.UserPrincipalName)'" -ErrorAction SilentlyContinue
            
            if ($userObj) {
                Remove-MgUser -UserId $userObj.Id
                Write-Host "✅ 削除完了: $($user.UserPrincipalName)" -ForegroundColor Green
            }
        } catch {
            Write-Host "❌ 削除失敗: $($user.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
        }
    }
}
```

### 削除済みユーザーの管理

**🎯 スクリプトの目的**
論理削除されたユーザーアカウントの管理を行います。削除済みユーザーの一覧表示、誤削除時の復旧処理、保管期間経過後の完全削除を安全に実行できます。30日間の復旧可能期間を有効活用し、データ保護と運用効率を両立します。

**📝 Usage**
```powershell
# 削除済みユーザーの一覧表示
.\Manage-DeletedUsers.ps1 -Action List

# 特定ユーザーの復旧
.\Manage-DeletedUsers.ps1 -Action Restore -UserPrincipalName "t.tanaka@school.edu.jp"

# 複数ユーザーの一括復旧
$restoreList = @("user1@domain.com", "user2@domain.com")
.\Manage-DeletedUsers.ps1 -Action BulkRestore -UserList $restoreList

# 期限切れユーザーの完全削除（注意：復旧不可能）
.\Manage-DeletedUsers.ps1 -Action PermanentDelete -OlderThanDays 25 -RequireConfirmation
```

**⚙️ パラメータ**
- `-Action`: 実行アクション（List/Restore/BulkRestore/PermanentDelete）
- `-UserPrincipalName`: 対象ユーザーのUPN（復旧時）
- `-UserList`: 一括処理対象のUPN配列
- `-OlderThanDays`: 指定日数より古いユーザーを対象
- `-RequireConfirmation`: 確認プロンプトの表示

**⚠️ 重要な注意事項**
- 論理削除から30日経過後は自動的に完全削除
- 完全削除は取り消し不可能な操作
- 復旧時は元のライセンス・グループ設定の再適用が必要

```powershell
# 削除済みユーザーの一覧表示
$deletedUsers = Get-MgDirectoryDeletedItem -DirectoryObjectId User

Write-Host "削除済みユーザー: $($deletedUsers.Count)名" -ForegroundColor Yellow
$deletedUsers | Select-Object DisplayName, UserPrincipalName, DeletedDateTime | Format-Table

# 特定ユーザーの復旧
$restoreUPN = "t.tanaka@school.edu.jp"
$deletedUser = $deletedUsers | Where-Object {$_.AdditionalProperties.userPrincipalName -eq $restoreUPN}

if ($deletedUser) {
    Restore-MgDirectoryDeletedItem -DirectoryObjectId $deletedUser.Id
    Write-Host "✅ 復旧完了: $restoreUPN" -ForegroundColor Green
}

# 完全削除（復旧不可能）
# ⚠️ 注意: この操作は取り消せません
$permanentDeleteUPN = "graduated.student@school.edu.jp"
$deletedUser = $deletedUsers | Where-Object {$_.AdditionalProperties.userPrincipalName -eq $permanentDeleteUPN}

if ($deletedUser) {
    Remove-MgDirectoryDeletedItem -DirectoryObjectId $deletedUser.Id
    Write-Host "⚠️ 完全削除完了: $permanentDeleteUPN" -ForegroundColor Red
}
```

---

## 🧪 実習演習: 新年度シミュレーション

### 演習1: 新入生100名の一括登録（30分）

実際の教育機関の新年度対応を想定した演習です。

#### 前提条件

```markdown
🎓 架空の大学「サンプル大学」新年度設定
- 新入生: 100名（情報学部50名、工学部50名）
- 学籍番号: 2025001-2025100
- メールアドレス: s[学籍番号]@sample-univ.ac.jp
- ライセンス: Microsoft 365 A1 for students
- パスワード: Student2025! + 学籍番号下3桁
```

#### CSVデータ生成スクリプト

**🎯 スクリプトの目的**
演習用の新入生データを自動生成し、実際の新年度対応シミュレーションを可能にします。学籍番号ベースの命名規則、部署別の配分、セキュリティグループの自動割り当てなど、教育機関での実運用に近い設定を再現できます。

**📝 Usage**
```powershell
# 基本的なデータ生成（100名の新入生）
.\Generate-StudentData.ps1 -StudentCount 100 -StartingId 2025001 -OutputPath "C:\temp\new_students_2025.csv"

# 複数学部対応
.\Generate-StudentData.ps1 -StudentCount 200 -Departments @("情報学部", "工学部", "経済学部") -StartingId 2025001

# カスタムドメイン指定
.\Generate-StudentData.ps1 -StudentCount 50 -Domain "university.ac.jp" -StudentPrefix "s" -OutputPath "C:\temp\students.csv"
```

**⚙️ パラメータ**
- `-StudentCount`: 生成する学生数
- `-StartingId`: 開始学籍番号
- `-Departments`: 配属学部のリスト
- `-Domain`: メールドメイン
- `-StudentPrefix`: 学籍番号の接頭辞
- `-OutputPath`: 出力CSVファイルパス

**💡 生成されるデータ項目**
- 学籍番号ベースのUPN（例：s2025001@university.ac.jp）
- 部署別の均等配分
- 統一されたライセンス設定
- セキュリティグループの自動割り当て
- 初期パスワード（学籍番号ベース）

```powershell
# 新入生データの自動生成
$students = @()
$departments = @("情報学部", "工学部")

for ($i = 1; $i -le 100; $i++) {
    $studentId = "2025{0:D3}" -f $i
    $dept = if ($i -le 50) { $departments[0] } else { $departments[1] }
    $lastName = "学生$i"
    $firstName = "太郎"
    
    $student = [PSCustomObject]@{
        DisplayName = "$lastName $firstName"
        UserPrincipalName = "s$studentId@sample-univ.ac.jp"
        FirstName = $firstName
        LastName = $lastName
        Password = "Student2025!$($studentId.Substring(4))"
        Department = $dept
        JobTitle = "学部生"
        LicenseSku = "Microsoft 365 A1 for students"
        SecurityGroups = "Students;$dept"
        UsageLocation = "JP"
        ForcePasswordChange = "true"
    }
    
    $students += $student
}

# CSVファイルとして出力
$students | Export-Csv "C:\temp\new_students_2025.csv" -Encoding UTF8 -NoTypeInformation

Write-Host "新入生データ生成完了: 100名" -ForegroundColor Green
```

#### 演習手順

1. **データ準備**: 上記スクリプトで学生データを生成
2. **事前検証**: CSVファイルの妥当性チェック
3. **段階的実行**: 25名ずつ4回に分けて実行
4. **結果確認**: 作成状況とエラー対応

### 演習2: 卒業生アカウントの一括処理（20分）

#### 処理要件

```markdown
🎓 卒業生アカウント処理（2024年度卒業生）
- 対象: 2021001-2021080（80名の卒業生）
- 処理内容: 
  1. ライセンス削除（コスト削減）
  2. アカウント無効化
  3. 卒業生グループへ移動
  4. 6ヶ月後に完全削除予定
```

#### 処理スクリプト

**🎯 スクリプトの目的**
卒業生のアカウント処理を自動化し、ライセンスコストの削減とセキュリティの確保を両立します。ライセンス削除による即座のコスト削減、アカウント無効化による不正アクセス防止、卒業生グループへの移動による将来的な管理を一括実行します。

**📝 Usage**
```powershell
# 指定年度の卒業生処理
.\Process-Graduates.ps1 -GraduationYear 2021 -StudentIdRange "2021001-2021080"

# 特定学部の卒業生のみ処理
.\Process-Graduates.ps1 -Department "情報学部" -GraduationYear 2021 -DryRun

# カスタムリストによる処理
$graduateList = Import-Csv "C:\temp\graduates_2025.csv"
.\Process-Graduates.ps1 -CustomList $graduateList -MoveToGroup "卒業生2025"
```

**⚙️ パラメータ**
- `-GraduationYear`: 卒業年度
- `-StudentIdRange`: 学籍番号範囲（例：2021001-2021080）
- `-Department`: 対象学部（指定時はその学部のみ）
- `-CustomList`: カスタム卒業生リスト
- `-MoveToGroup`: 移動先グループ名
- `-DryRun`: 実際の処理を行わず結果を表示

**🔄 処理内容**
1. ライセンス削除（Microsoft 365 A1 → なし）
2. アカウント無効化（即座にサインイン不可）
3. 卒業生グループへの移動
4. 処理結果レポートの生成

```powershell
# 卒業生処理の実行
$graduatingStudents = @()

for ($i = 1; $i -le 80; $i++) {
    $studentId = "2021{0:D3}" -f $i
    $graduatingStudents += "s$studentId@sample-univ.ac.jp"
}

foreach ($upn in $graduatingStudents) {
    try {
        $user = Get-MgUser -Filter "userPrincipalName eq '$upn'" -ErrorAction SilentlyContinue
        
        if ($user) {
            # ライセンス削除
            $licenses = Get-MgUserLicenseDetail -UserId $user.Id
            if ($licenses) {
                Set-MgUserLicense -UserId $user.Id -AddLicenses @() -RemoveLicenses $licenses.SkuId
            }
            
            # アカウント無効化
            Update-MgUser -UserId $user.Id -AccountEnabled $false
            
            # 卒業生グループに追加
            $graduatesGroup = Get-MgGroup -Filter "displayName eq '卒業生'"
            if ($graduatesGroup) {
                New-MgGroupMember -GroupId $graduatesGroup.Id -DirectoryObjectId $user.Id -ErrorAction SilentlyContinue
            }
            
            Write-Host "✅ 卒業処理完了: $upn" -ForegroundColor Green
        }
    } catch {
        Write-Host "❌ 処理失敗: $upn - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

### 演習3: 組織変更対応（25分）

#### 変更内容

```markdown
🏢 組織改編対応（2025年4月）
- 情報学部 → 情報工学部 に名称変更
- 対象教職員: 情報学部所属の全員（約30名）
- 変更項目: 部署名、メールアドレスの署名、セキュリティグループ
```

# 組織変更の一括処理

**🎯 スクリプトの目的**
組織改編（部署名変更、統合、分割など）に対応し、関連するユーザー情報とグループメンバーシップを一括更新します。人事システムとの整合性を保ちながら、アクセス権限の継続性を確保し、運用中断を最小限に抑えます。

**📝 Usage**
```powershell
# 基本的な部署名変更
.\Update-OrganizationStructure.ps1 -OldDeptName "情報学部" -NewDeptName "情報工学部"

# 複数部署の一括変更
$changes = @(
    @{Old="情報学部"; New="情報工学部"},
    @{Old="機械学部"; New="機械システム学部"}
)
.\Update-OrganizationStructure.ps1 -DepartmentChanges $changes

# 詳細ログ付き実行
.\Update-OrganizationStructure.ps1 -OldDeptName "情報学部" -NewDeptName "情報工学部" -LogPath "C:\logs\org_change.log" -Verbose
```

**⚙️ パラメータ**
- `-OldDeptName`: 変更前の部署名
- `-NewDeptName`: 変更後の部署名  
- `-DepartmentChanges`: 複数変更の場合の変更リスト
- `-LogPath`: 詳細ログの出力先
- `-Verbose`: 詳細な進捗表示

**🔄 処理内容**
1. 対象ユーザーの検索（部署フィールドベース）
2. ユーザー属性の一括更新（Department フィールド）
3. 旧グループからの削除
4. 新グループへの追加
5. 変更結果レポートの生成

```powershell
$oldDept = "情報学部"
$newDept = "情報工学部"

# 対象ユーザーの検索
$targetUsers = Get-MgUser -Filter "department eq '$oldDept'" -All

Write-Host "対象ユーザー: $($targetUsers.Count)名" -ForegroundColor Cyan

foreach ($user in $targetUsers) {
    try {
        # 部署名の更新
        Update-MgUser -UserId $user.Id -Department $newDept
        
        # 古いグループから削除
        $oldGroup = Get-MgGroup -Filter "displayName eq '$oldDept'"
        if ($oldGroup) {
            Remove-MgGroupMember -GroupId $oldGroup.Id -DirectoryObjectId $user.Id -ErrorAction SilentlyContinue
        }
        
        # 新しいグループに追加
        $newGroup = Get-MgGroup -Filter "displayName eq '$newDept'"
        if ($newGroup) {
            New-MgGroupMember -GroupId $newGroup.Id -DirectoryObjectId $user.Id -ErrorAction SilentlyContinue
        }
        
        Write-Host "✅ 組織変更完了: $($user.UserPrincipalName)" -ForegroundColor Green
        
    } catch {
        Write-Host "❌ 変更失敗: $($user.UserPrincipalName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

---

## 🛠️ トラブルシューティング

### よくある問題と解決方法

#### 問題1: CSVインポート時の文字化け

**症状**: 日本語ユーザー名が文字化けする

**原因**: 文字コードの問題（Shift-JIS, UTF-8 BOM無しなど）

**解決方法**:

**🎯 スクリプトの目的**
ネットワーク遅延や一時的なサービス障害によるタイムアウトエラーを回避し、大規模な一括処理の確実性を向上させます。タイムアウト値の調整と自動再試行ロジックにより、処理の中断を最小限に抑えます。

**📝 Usage**
```powershell
# タイムアウト設定の調整
Set-GraphTimeout -TimeoutSeconds 300

# 再試行ロジック付きでコマンド実行
Invoke-WithRetry -ScriptBlock { New-MgUser @userParams } -MaxRetries 3 -DelaySeconds 5

# 大量処理での使用例
$users | ForEach-Object {
    Invoke-WithRetry -ScriptBlock { New-MgUser @(ConvertTo-UserParams $_) } -MaxRetries 5
}
```

**⚙️ パラメータ**
- `-TimeoutSeconds`: タイムアウト時間（秒）
- `-MaxRetries`: 最大再試行回数
- `-DelaySeconds`: 再試行間隔（秒）
- `-ScriptBlock`: 実行対象のスクリプトブロック

**💡 推奨設定**
- 小規模（〜100名）: Timeout=180秒, Retry=3回
- 中規模（100-500名）: Timeout=300秒, Retry=5回
- 大規模（500名〜）: Timeout=600秒, Retry=5回

```powershell
# PowerShellでの文字コード変換
$csvContent = Get-Content "C:\temp\users.csv" -Encoding Default
$csvContent | Out-File "C:\temp\users_utf8.csv" -Encoding UTF8

# Excelでの正しい保存方法
# 1. 「名前を付けて保存」を選択
# 2. ファイル形式：「CSV UTF-8 (コンマ区切り) (*.csv)」を選択
# 3. 保存
```

#### 問題2: PowerShell実行時のタイムアウト

**症状**: 大量処理中に "Operation timed out" エラー

**解決方法**:
```powershell
# タイムアウト値の調整
$PSDefaultParameterValues = @{
    '*-Mg*:TimeoutSec' = 300  # 5分に延長
}

# 再試行ロジックの実装
function Invoke-WithRetry {
    param(
        [scriptblock]$ScriptBlock,
        [int]$MaxRetries = 3,
        [int]$DelaySeconds = 5
    )
    
    for ($i = 1; $i -le $MaxRetries; $i++) {
        try {
            return & $ScriptBlock
        } catch {
            if ($i -eq $MaxRetries) {
                throw
            }
            Write-Warning "試行 $i 失敗、$DelaySeconds 秒後に再試行: $($_.Exception.Message)"
            Start-Sleep -Seconds $DelaySeconds
        }
    }
}

# 使用例
Invoke-WithRetry {
    New-MgUser @userParams
}
```

#### 問題3: ライセンス割り当てエラー

**症状**: "License assignment failed" エラー

**診断・解決スクリプト**:

**🎯 スクリプトの目的**
ライセンス割り当て失敗の原因を自動診断し、解決策を提示します。ユーザー存在確認、ライセンス在庫チェック、利用地域設定、既存ライセンス競合の確認を包括的に実行し、手動での原因調査時間を大幅に短縮します。

**📝 Usage**
```powershell
# 単一ユーザーの診断
Test-LicenseAssignment -UserPrincipalName "t.tanaka@school.edu.jp" -LicenseSku "A1"

# 複数ユーザーの一括診断
$users = @("user1@domain.com", "user2@domain.com")
$users | ForEach-Object { Test-LicenseAssignment -UserPrincipalName $_ -LicenseSku "A3" }

# 詳細レポート付き診断
Test-LicenseAssignment -UserPrincipalName "t.tanaka@school.edu.jp" -LicenseSku "A1" -DetailedReport -OutputPath "C:\reports\license_diagnosis.txt"
```

**⚙️ パラメータ**
- `-UserPrincipalName`: 診断対象ユーザーのUPN
- `-LicenseSku`: 割り当て予定のライセンスSKU
- `-DetailedReport`: 詳細な診断レポートを生成
- `-OutputPath`: レポート出力先パス

**🔍 診断項目**
1. ユーザーアカウントの存在確認
2. ライセンスSKUの有効性確認
3. 利用可能ライセンス数の確認
4. ユーザーの利用地域設定確認
5. 既存ライセンスとの競合確認

```powershell
function Test-LicenseAssignment {
    param($UserPrincipalName, $LicenseSku)
    
    # ユーザー確認
    $user = Get-MgUser -Filter "userPrincipalName eq '$UserPrincipalName'"
    if (-not $user) {
        Write-Error "ユーザーが見つかりません: $UserPrincipalName"
        return
    }
    
    # ライセンス在庫確認
    $sku = Get-MgSubscribedSku | Where-Object {$_.SkuPartNumber -like "*$LicenseSku*"}
    if (-not $sku) {
        Write-Error "ライセンスSKUが見つかりません: $LicenseSku"
        return
    }
    
    $available = $sku.PrepaidUnits.Enabled - $sku.ConsumedUnits
    if ($available -le 0) {
        Write-Error "利用可能ライセンスがありません: $LicenseSku (在庫: $available)"
        return
    }
    
    # 利用地域設定確認
    if (-not $user.UsageLocation) {
        Write-Warning "利用地域が設定されていません。日本(JP)に設定します。"
        Update-MgUser -UserId $user.Id -UsageLocation "JP"
    }
    
    # 既存ライセンス確認
    $existingLicenses = Get-MgUserLicenseDetail -UserId $user.Id
    $conflictLicense = $existingLicenses | Where-Object {$_.SkuId -eq $sku.SkuId}
    if ($conflictLicense) {
        Write-Warning "既に同じライセンスが割り当てられています: $LicenseSku"
        return
    }
    
    Write-Host "✅ ライセンス割り当て可能: $UserPrincipalName → $LicenseSku" -ForegroundColor Green
}

# 使用例
Test-LicenseAssignment -UserPrincipalName "t.tanaka@school.edu.jp" -LicenseSku "A1"
```

#### 問題4: 一括処理の中途停止

**症状**: 処理中にエラーで停止し、どこまで完了したか不明

**対策: 進捗追跡システム**:

**🎯 スクリプトの目的**
大規模な一括処理において、進捗状況の可視化、エラー発生時の詳細ログ記録、中断時の再開ポイント特定を可能にします。処理の透明性を確保し、失敗時の迅速な対応と部分的な再実行を支援します。

**📝 Usage**
```powershell
# 基本的な進捗管理付き処理
$users = Import-Csv "C:\temp\users.csv"
Invoke-BulkUserCreation -Users $users -LogPath "C:\logs\creation.log"

# 中断時の再開処理
$failedUsers = Import-Csv "C:\temp\retry_users.csv"
Invoke-BulkUserCreation -Users $failedUsers -LogPath "C:\logs\retry.log" -ResumeMode

# リアルタイム監視付き実行
Invoke-BulkUserCreation -Users $users -EnableProgressDashboard -RefreshInterval 10
```

**⚙️ パラメータ**
- `-Users`: 処理対象ユーザーの配列
- `-LogPath`: 詳細ログファイルのパス
- `-ResumeMode`: 中断からの再開モード
- `-EnableProgressDashboard`: リアルタイム進捗表示
- `-RefreshInterval`: 進捗更新間隔（秒）

**📊 提供される情報**
- 処理進捗（完了率、残り時間予測）
- 成功・失敗ユーザー数のリアルタイム集計
- エラー詳細とユーザー特定情報
- 失敗分の自動CSV再生成
- 中間レポートの定期出力

**🔄 中断・再開機能**
- 処理状況の自動保存
- 失敗分の自動抽出・再試行用CSV生成
- 重複処理の防止機能

```powershell
# 進捗管理付き一括処理
function Invoke-BulkUserCreation {
    param($Users, $LogPath = "C:\temp\bulk_creation_log.txt")
    
    # ログファイル初期化
    "開始時刻: $(Get-Date)" | Out-File $LogPath
    
    $completed = @()
    $failed = @()
    
    for ($i = 0; $i -lt $Users.Count; $i++) {
        $user = $Users[$i]
        $progress = [math]::Round(($i / $Users.Count) * 100, 1)
        
        Write-Progress -Activity "ユーザー作成中" -Status "$($i+1)/$($Users.Count) - $($user.UserPrincipalName)" -PercentComplete $progress
        
        try {
            # ユーザー作成処理
            $newUser = New-MgUser @(Convert-ToUserParams $user)
            
            $completed += $user.UserPrincipalName
            "成功: $($user.UserPrincipalName) - $(Get-Date)" | Add-Content $LogPath
            
        } catch {
            $failed += @{
                User = $user.UserPrincipalName
                Error = $_.Exception.Message
                Time = Get-Date
            }
            "失敗: $($user.UserPrincipalName) - $($_.Exception.Message) - $(Get-Date)" | Add-Content $LogPath
        }
        
        # 定期的な中間レポート
        if (($i + 1) % 50 -eq 0) {
            Write-Host "中間報告: $($i + 1)/$($Users.Count) 完了" -ForegroundColor Cyan
        }
    }
    
    Write-Progress -Activity "ユーザー作成中" -Completed
    
    # 最終レポート
    Write-Host "`n=== 最終結果 ===" -ForegroundColor Magenta
    Write-Host "成功: $($completed.Count)名" -ForegroundColor Green
    Write-Host "失敗: $($failed.Count)名" -ForegroundColor Red
    
    if ($failed.Count -gt 0) {
        Write-Host "`n失敗したユーザー:" -ForegroundColor Red
        $failed | ForEach-Object {
            Write-Host "  $($_.User): $($_.Error)" -ForegroundColor Red
        }
        
        # 失敗分のCSV再作成
        $retryUsers = $Users | Where-Object {$_.UserPrincipalName -in ($failed.User)}
        $retryUsers | Export-Csv "C:\temp\retry_users.csv" -Encoding UTF8 -NoTypeInformation
        Write-Host "`n再試行用CSVを作成しました: C:\temp\retry_users.csv" -ForegroundColor Yellow
    }
    
    "完了時刻: $(Get-Date)" | Add-Content $LogPath
    return @{
        Completed = $completed
        Failed = $failed
        LogPath = $LogPath
    }
}
```

---

## 📊 運用最適化とパフォーマンス

### 大規模処理の最適化手法

#### 並列処理による高速化

**🎯 スクリプトの目的**
PowerShell 7以上の並列処理機能を活用し、大規模ユーザー作成の処理時間を大幅に短縮します。複数のユーザー作成処理を同時実行することで、従来の逐次処理と比較して50-70%の時間短縮を実現できます。

**📝 Usage**
```powershell
# 基本的な並列処理（10並列）
$users = Import-Csv "C:\temp\users.csv"
Invoke-ParallelUserCreation -Users $users -ThrottleLimit 10

# 高速処理（20並列、大規模環境向け）
Invoke-ParallelUserCreation -Users $users -ThrottleLimit 20 -OutputPath "C:\logs\parallel_results.log"

# 安全重視（5並列、安定性重視）
Invoke-ParallelUserCreation -Users $users -ThrottleLimit 5 -ConservativeMode
```

**⚙️ パラメータ**
- `-Users`: 処理対象ユーザーの配列
- `-ThrottleLimit`: 並列実行数（推奨：5-20）
- `-OutputPath`: 結果ログの出力先
- `-ConservativeMode`: 安定性重視モード（エラー時の自動停止）

**💡 並列数の推奨設定**
- 小規模（〜200名）: ThrottleLimit=5（安定性重視）
- 中規模（200-1000名）: ThrottleLimit=10（バランス型）
- 大規模（1000名〜）: ThrottleLimit=15-20（速度重視）

**⚠️ 注意事項**
- PowerShell 7以上が必要
- 並列数を上げすぎると API レート制限に抵触する可能性
- 本番環境では事前にテスト実行を推奨

```powershell
# 並列処理による高速化（PowerShell 7+）
function Invoke-ParallelUserCreation {
    param(
        $Users,
        [int]$ThrottleLimit = 10  # 同時実行数
    )
    
    $Users | ForEach-Object -ThrottleLimit $ThrottleLimit -Parallel {
        # Microsoft Graph モジュールを並列ジョブ内でインポート
        Import-Module Microsoft.Graph.Users -Force
        
        try {
            $userParams = @{
                DisplayName = $_.DisplayName
                UserPrincipalName = $_.UserPrincipalName
                GivenName = $_.FirstName
                Surname = $_.LastName
                UsageLocation = "JP"
                PasswordProfile = @{
                    Password = $_.Password
                    ForceChangePasswordNextSignIn = $true
                }
                AccountEnabled = $true
            }
            
            $newUser = New-MgUser @userParams
            Write-Output "成功: $($_.UserPrincipalName)"
            
        } catch {
            Write-Output "失敗: $($_.UserPrincipalName) - $($_.Exception.Message)"
        }
    }
}
```

#### バッチサイズの動的調整

**🎯 スクリプトの目的**
ネットワーク環境やテナントの負荷状況に応じて、最適なバッチサイズを自動算出します。小規模テストでの処理時間測定により、目標処理時間内で最大のスループットを実現するバッチサイズを動的に決定し、環境に依存しない効率的な処理を可能にします。

**📝 Usage**
```powershell
# 基本的な最適化（目標10分/バッチ）
$allUsers = Import-Csv "C:\temp\users.csv"
$optimalSize = Get-OptimalBatchSize -TestUsers $allUsers[0..9] -TargetMinutes 10

# カスタム目標時間での最適化
$optimalSize = Get-OptimalBatchSize -TestUsers $allUsers[0..19] -TargetMinutes 5 -MaxBatchSize 150

# 詳細レポート付き最適化
$optimalSize = Get-OptimalBatchSize -TestUsers $allUsers[0..9] -Detailed -OutputReport "C:\reports\optimization.txt"
```

**⚙️ パラメータ**
- `-TestUsers`: 性能測定用のサンプルユーザー（5-20名推奨）
- `-TargetMinutes`: 目標処理時間（分）
- `-MaxBatchSize`: 安全上限値（デフォルト200）
- `-Detailed`: 詳細な測定結果を表示
- `-OutputReport`: 最適化レポートの出力先

**📊 算出される情報**
- ユーザー1名あたりの平均処理時間
- 推奨バッチサイズ
- 予想総処理時間
- ネットワーク性能指標

**💡 活用シーン**
- 初回大規模展開時の設定決定
- ネットワーク環境変更後の再調整
- 異なるテナント間での設定移行

```powershell
# ネットワーク状況に応じたバッチサイズ自動調整
function Get-OptimalBatchSize {
    $testStartTime = Get-Date
    
    # 小規模テスト（5ユーザー）で処理時間測定
    $testUsers = $allUsers[0..4]
    
    foreach ($user in $testUsers) {
        # テスト実行
        $null = Test-UserCreation $user
    }
    
    $testDuration = (Get-Date) - $testStartTime
    $avgTimePerUser = $testDuration.TotalSeconds / 5
    
    # 目標処理時間（10分）から最適バッチサイズを算出
    $targetMinutes = 10
    $optimalBatchSize = [math]::Floor(($targetMinutes * 60) / $avgTimePerUser)
    
    # 安全な範囲に制限
    $optimalBatchSize = [math]::Max(10, [math]::Min($optimalBatchSize, 200))
    
    Write-Host "推奨バッチサイズ: $optimalBatchSize 名/バッチ" -ForegroundColor Green
    return $optimalBatchSize
}
```

### 運用監視とアラート

**🎯 スクリプトの目的**
Microsoft 365 テナントの一括処理状況をリアルタイムで監視し、ユーザー数、ライセンス使用状況、最近のエラーを一元的に表示します。大規模な一括処理中の状況把握と、問題の早期発見・対応を支援するダッシュボード機能を提供します。

**📝 Usage**
```powershell
# 基本的な監視ダッシュボード起動
Show-BulkOperationDashboard

# カスタム更新間隔での監視
Show-BulkOperationDashboard -RefreshInterval 60  # 60秒間隔

# 特定の条件でアラート付き監視
Show-BulkOperationDashboard -AlertOnErrors -LicenseThreshold 90  # ライセンス使用率90%でアラート
```

**⚙️ パラメータ**
- `-RefreshInterval`: 画面更新間隔（秒、デフォルト30秒）
- `-AlertOnErrors`: エラー発生時のアラート有効化
- `-LicenseThreshold`: ライセンス使用率アラート閾値（%）

**📊 監視項目**
- **テナント概要**: 総ユーザー数、アクティブユーザー数、過去24時間の新規作成数
- **ライセンス状況**: SKU別の使用率、利用可能数、使用率パーセンテージ
- **エラー監視**: 過去1時間のエラー発生状況、失敗理由の詳細
- **処理状況**: 大規模処理の進行状況（実行中の場合）

**🚨 アラート機能**
- ライセンス使用率が閾値を超過した場合の警告
- 連続エラー発生時の自動通知
- 大規模処理の異常終了検出

**💡 運用での活用**
- 大規模一括処理中のリアルタイム監視
- 日常運用での健全性チェック
- 問題の早期発見と対応判断

```powershell
# 一括処理の監視ダッシュボード
function Show-BulkOperationDashboard {
    while ($true) {
        Clear-Host
        Write-Host "=== Microsoft 365 一括処理監視ダッシュボード ===" -ForegroundColor Cyan
        Write-Host "更新時刻: $(Get-Date)" -ForegroundColor Yellow
        
        # テナント全体の状況
        $totalUsers = (Get-MgUser -All | Measure-Object).Count
        $activeUsers = (Get-MgUser -Filter "accountEnabled eq true" -All | Measure-Object).Count
        $recentlyCreated = (Get-MgUser -Filter "createdDateTime ge $(Get-Date).AddDays(-1)" -All | Measure-Object).Count
        
        Write-Host "`n📊 テナント概要:" -ForegroundColor Green
        Write-Host "  総ユーザー数: $totalUsers 名"
        Write-Host "  アクティブユーザー: $activeUsers 名"
        Write-Host "  過去24時間の新規作成: $recentlyCreated 名"
        
        # ライセンス使用状況
        Write-Host "`n📋 ライセンス状況:" -ForegroundColor Green
        $licenses = Get-MgSubscribedSku | Select-Object SkuPartNumber, ConsumedUnits, @{Name="AvailableUnits";Expression={$_.PrepaidUnits.Enabled - $_.ConsumedUnits}}
        $licenses | ForEach-Object {
            $usagePercent = [math]::Round(($_.ConsumedUnits / $_.PrepaidUnits.Enabled) * 100, 1)
            Write-Host "  $($_.SkuPartNumber): $($_.ConsumedUnits)/$($_.PrepaidUnits.Enabled) 使用 ($usagePercent%)"
        }
        
        # 最近のエラー
        Write-Host "`n⚠️ 最近のエラー (過去1時間):" -ForegroundColor Red
        $recentErrors = Get-MgAuditLogDirectoryAudit -Filter "activityDateTime ge $((Get-Date).AddHours(-1))" | 
                       Where-Object {$_.Result -eq "failure"}
        
        if ($recentErrors) {
            $recentErrors | Select-Object -First 5 | ForEach-Object {
                Write-Host "  $($_.ActivityDateTime): $($_.ActivityDisplayName) - $($_.FailureReason)" -ForegroundColor Red
            }
        } else {
            Write-Host "  エラーなし" -ForegroundColor Green
        }
        
        Write-Host "`n[Ctrl+C で終了]" -ForegroundColor Yellow
        Start-Sleep -Seconds 30
    }
}
```

---

## ✅ 学習成果の確認

### 習得スキルチェック

この節を完了したら、以下の項目について実際に操作できることを確認してください：

- [ ] **CSVテンプレート設計**: 組織要件に合わせたフィールド設定
- [ ] **データ前処理**: 検証・クリーニング・形式統一
- [ ] **管理センター操作**: 中規模（〜200名）の一括処理
- [ ] **PowerShell自動化**: 大規模（1000名+）の効率的な処理
- [ ] **エラーハンドリング**: 失敗時の診断・復旧・再実行
- [ ] **進捗管理**: 長時間処理の監視と中間レポート
- [ ] **段階的削除**: 安全なアカウント無効化・削除手順

### 実践課題: 総合演習

実際のプロジェクトを想定した包括的な課題に取り組んでください：

```markdown
🎯 総合課題: 大学統合プロジェクト

## 背景
A大学とB大学が統合し、新しいC大学として統一のMicrosoft 365環境を構築します。

## 要件
- A大学: 既存ユーザー 800名（学生600名、教職員200名）
- B大学: 新規移行ユーザー 600名（学生450名、教職員150名）
- 統合後の命名規則変更が必要
- 段階的な移行実施（2週間で完了）

## 実施内容
1. データマッピング設計（既存→新規）
2. CSVテンプレート作成（変換ルール含む）
3. テスト環境での動作確認（50名規模）
4. 本番環境での段階的実行計画策定
5. 完了後の検証・レポート作成

## 成果物
- 移行手順書（PowerShellスクリプト含む）
- データ変換用CSVテンプレート
- エラー処理・復旧手順書
- 完了検証チェックリスト
```

---

## 🚀 次のステップ

この節を完了したら、以下の学習に進むことを推奨します：

### 推奨学習パス

1. **[3.3 ユーザーグループの管理](03-groups.md)**
   - 大量ユーザーの効率的な権限管理
   - 動的グループによる自動化

2. **[3.4 ユーザーロールと権限](04-roles-permissions.md)**
   - 適切な権限分離とセキュリティ
   - カスタムロールの設計

<!-- 3. **[第5章: セキュリティとコンプライアンス](../05-security-compliance/)** -->
- 大規模環境でのセキュリティポリシー
- コンプライアンス要件への対応

### 発展学習

- **Microsoft Graph API**: RESTベースの直接操作
- **Azure Logic Apps**: ワークフロー駆動の自動化
- **Power Automate**: ノーコード/ローコード自動化
- **Azure DevOps**: CI/CDパイプラインでの展開自動化

---

## 📚 参考リソース

### Microsoft公式ドキュメント

- [Microsoft Graph PowerShell SDK](https://docs.microsoft.com/en-us/powershell/microsoftgraph/)
- [Microsoft 365でのユーザーの一括追加](https://docs.microsoft.com/ja-jp/microsoft-365/enterprise/add-several-users-at-the-same-time)
- [Microsoft Graph API ユーザーリソース](https://docs.microsoft.com/en-us/graph/api/resources/user)

### 実用ツール・テンプレート

<!-- - [CSVテンプレート集](../../appendices/templates/) -->
- 教育機関向け・企業向け・複数職種対応版

<!-- - [PowerShell一括処理スクリプト](../../appendices/powershell-commands.md#一括ユーザー管理) -->
- 作成・更新・削除の各種スクリプト
- エラーハンドリング・進捗表示付き

<!-- - [運用チェックリスト](../../appendices/checklists/bulk-operation-checklist.md) -->
- 事前準備・実行・事後確認の各段階

### コミュニティ・サポート

- [Microsoft Tech Community - Microsoft 365](https://techcommunity.microsoft.com/t5/microsoft-365/ct-p/microsoft365)
- [PowerShell Gallery - Microsoft Graph](https://www.powershellgallery.com/packages/Microsoft.Graph)
- [GitHub - Microsoft Graph PowerShell SDK](https://github.com/microsoftgraph/msgraph-sdk-powershell)

---

*📅 最終更新: 2025年6月2日*  
*✍️ 更新者: Microsoft 365 ハンドブック編集チーム*  
*📖 対象バージョン: Microsoft 365 (2025年6月版) 
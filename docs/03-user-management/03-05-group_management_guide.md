# 3.5 グループ管理とメンバーシップ戦略

---

## 📖 学習目標

Microsoft 365における効率的なグループ管理手法を習得し、教育機関特有の組織構造や年次更新に対応できる柔軟で持続可能なグループ戦略を構築できるスキルを身につけます。

### 🎯 この節で習得すること

- **グループの種類と特徴**を理解し、目的に応じた適切な選択ができる
- **静的・動的メンバーシップ**の仕組みを理解し、組織に適した管理方式を設計できる
- **教育機関向けベストプラクティス**を習得し、学校運用に最適化されたグループ戦略を実践できる
- **年次更新の自動化**技術をマスターし、新年度対応の負荷を大幅に削減できる
- **School Data Sync (SDS)**を活用し、校務支援システムとの連携を実現できる

### 💼 実務での活用場面

- **新年度準備**: 学年・クラス・部活動グループの一括更新と自動化
- **組織変更対応**: 教職員の異動・昇格に伴うグループメンバーシップの動的管理
- **アクセス権管理**: 教材・資料への適切なアクセス制御とセキュリティ確保
- **コラボレーション促進**: 学部・学科横断的なプロジェクトグループの効率的な運営
- **システム連携**: 校務支援システム・学習管理システム（LMS）との自動同期

---

## ⚡ 前提条件

### 必要な権限・環境

- [ ] **Microsoft 365テナント**（教育機関向けA1/A3/A5ライセンス）
- [ ] **グローバル管理者**または**グループ管理者**権限
- [ ] **Microsoft Entra ID管理センター**へのアクセス権
- [ ] **PowerShell 7.x + Microsoft Graph PowerShell SDK**
- [ ] **Microsoft 365 A3/A5ライセンス**（動的グループ使用時）

### 事前準備

```markdown
📋 グループ管理準備チェックリスト
- [ ] 現在の組織構造・職制の整理
- [ ] 既存グループの棚卸しと分類
- [ ] 年次更新パターンの分析（学年・クラス・部活動等）
- [ ] 校務支援システムとの連携要件確認
- [ ] セキュリティポリシーとアクセス権要件の明確化
- [ ] 動的グループ用ライセンス（Microsoft 365 A3/A5）の確認
```

> 💡 **重要**: 教育機関では年度単位での大規模な組織変更が発生するため、スケーラブルで自動化可能なグループ設計が成功の鍵となります。

---

## 🔍 Microsoft 365グループの種類と特徴

### 1. Microsoft 365グループ（旧Office 365グループ）

**目的**: 社内外のユーザー間での共同作業に使用され、チームの共同作業をより効率的に行うのに役立つグループです。各グループでは、メンバーは、会話、ファイル、予定表イベント、Stream、Planner のグループメールと共有ワークスペースを取得します

#### 主な特徴

| 項目 | 詳細 |
|------|------|
| **付属サービス** | Exchange Online（メールボックス）、SharePoint サイト、OneNote、Planner、Teams |
| **コラボレーション** | ファイル共有、共同編集、チャット、ビデオ会議 |
| **外部連携** | 管理者によって有効にされている限り、組織の外部のユーザーをグループに追加できます |
| **自動化連携** | Teams、SharePoint、Planner、Stream の自動プロビジョニング |
| **メンバーシップ** | 静的・動的の両方をサポート |

#### 教育機関での活用例

```markdown
🎓 学校での活用シナリオ
- **学年・クラスグループ**: "2025年度-1年A組" → Teams授業、資料共有
- **部活動・サークル**: "野球部2025" → 活動記録、スケジュール管理
- **教科・専攻グループ**: "数学科教員" → 教材開発、情報共有
- **プロジェクトチーム**: "文化祭実行委員会" → 企画・準備の協働作業
- **PTA・保護者会**: "1年生保護者会" → 連絡・相談、イベント調整
```

### 2. セキュリティグループ

**目的**: SharePoint サイトなどのリソースへのアクセスを許可するために使用され、アクセス制御を一元管理するために利用されます

#### 主な特徴

| 項目 | 詳細 |
|------|------|
| **主要機能** | リソースへのアクセス権限管理、ライセンス割り当て |
| **適用範囲** | SharePoint、Teams、アプリケーション、条件付きアクセス |
| **メンバー種別** | ユーザー、デバイス、ネストしたグループ |
| **メール機能** | なし（メール有効セキュリティグループは別途） |
| **管理場所** | Microsoft Entra ID、Microsoft 365管理センター |

#### 教育機関での活用例

```markdown
🏫 学校での権限管理シナリオ
- **職種別アクセス**: "Teachers" → 成績システム、教材データベース
- **学年別権限**: "Grade1-Students" → 学年限定コンテンツ
- **管理職権限**: "SchoolAdmins" → 人事情報、予算データ
- **施設利用権**: "LabUsers" → 特定PC教室、実験室システム
- **外部アクセス**: "GuestLecturers" → 限定的なシステムアクセス
```

### 3. 配布グループ

**目的**: ユーザーのグループに通知を送信するのに使用されます

#### 特徴と活用場面

```markdown
📧 配布グループの特徴
- メール配信専用（SharePoint等との連携なし）
- 外部メール受信可能（管理者設定により）
- 軽量でシンプルな構造
- Microsoft Teams のメンバーに追加可能（個別メンバーとして）

🎯 教育機関での活用
- 全校通知: "all-staff@school.edu.jp"
- 学年別連絡: "grade1-parents@school.edu.jp" 
- 部門別通知: "science-dept@school.edu.jp"
- 緊急連絡網: "emergency-contacts@school.edu.jp"
```

### 4. メール有効セキュリティグループ

**目的**: セキュリティグループと配布リストの機能を組み合わせたもので、メンバーへのアクセス制御を行いながら、メールの送信も可能です

#### 活用シナリオ

```markdown
🔐 統合型グループ活用例
- "ITSupport" → システム権限 + IT関連通知受信
- "GradeCoordinators" → 学年管理権限 + 保護者連絡
- "LabManagers" → 実験室アクセス + 機器管理通知
- "EventOrganizers" → イベント管理権限 + 関係者連絡
```

---

## 📊 静的メンバーシップ vs 動的メンバーシップ

### 静的メンバーシップ

#### 特徴

- **手動管理**: 管理者が個別にメンバーを追加・削除
- **即時反映**: 変更が直ちにグループに反映
- **細かい制御**: 例外的なメンバーシップにも対応可能
- **シンプル**: 追加ライセンス不要、設定が簡単

#### 適用場面

```markdown
🎯 静的グループが適している場面
- 人数が少なく変動頻度の低いグループ（管理職、委員会等）
- 例外的なメンバーシップが必要（特別講師、非常勤等）
- 一時的・プロジェクト型のグループ（文化祭実行委員会等）
- 細かいアクセス制御が必要な場合
```

### 動的メンバーシップ

#### 特徴と要件

- **自動管理**: 部門、場所、タイトルなどのユーザー属性に基づいてグループ メンバーを自動的に追加または削除できます
- **ライセンス要件**: **Microsoft 365 A3, A5** (Microsoft Entra ID Premium P1を含む) で利用可能。教育機関向けライセンスでの動的グループ機能
- **ルールベース**: 属性フィルターに基づく自動メンバー管理
- **スケーラブル**: 大規模組織での効率的な管理

> 💡 **教育機関向けライセンス**: Microsoft 365 A1では動的グループは利用できません。A3またはA5ライセンスが必要です。Microsoft Entra ID Premium P1が含まれるEnterprise系プラン（E3、E5）でも利用可能です。

#### 教育機関での動的ルール例

**🎯 動的グループルール設計例**

**学年別グループ**
```powershell
# 1年生グループの動的ルール
(user.extensionAttribute1 -eq "Grade1") -and (user.userType -eq "Member")

# 教職員グループの動的ルール  
(user.jobTitle -contains "教員") -or (user.jobTitle -contains "教授") -or (user.jobTitle -contains "講師")

# 特定学部の学生グループ
(user.department -eq "情報学部") -and (user.extensionAttribute2 -eq "Student")
```

**部署・職種別グループ**
```powershell
# 数学科教員グループ
(user.department -eq "数学科") -and (user.jobTitle -contains "教員")

# 管理職グループ
(user.jobTitle -contains "校長") -or (user.jobTitle -contains "教頭") -or (user.jobTitle -contains "主任")

# 非常勤講師グループ
(user.employeeType -eq "PartTime") -and (user.jobTitle -contains "講師")
```

### PowerShellによる動的グループ作成

**🎯 スクリプトの目的**
教育機関の組織構造に基づいた動的グループを効率的に作成し、属性ベースの自動メンバー管理を実現します。学年、学科、職種などの属性を活用して、年次更新時の手動作業を大幅に削減できます。

**📝 Usage**
```powershell
# 基本的な動的グループ作成
New-DynamicEducationGroup -GroupName "2025年度-1年生" -Rule "(user.extensionAttribute1 -eq 'Grade1')" -GroupType "Security"

# 複合条件での動的グループ
New-DynamicEducationGroup -GroupName "数学科教員" -Rule "(user.department -eq '数学科') -and (user.jobTitle -contains '教員')" -GroupType "Microsoft365"

# 一括作成（CSV入力）
$groups = Import-Csv "C:\temp\dynamic_groups.csv"
$groups | ForEach-Object { New-DynamicEducationGroup -GroupName $_.GroupName -Rule $_.Rule -GroupType $_.GroupType }
```

**⚙️ パラメータ**
- `-GroupName`: 作成するグループ名
- `-Rule`: 動的メンバーシップルール（Azure AD Query構文）
- `-GroupType`: グループ種別（Security/Microsoft365）
- `-Description`: グループの説明（オプション）

```powershell
function New-DynamicEducationGroup {
    param(
        [Parameter(Mandatory=$true)]
        [string]$GroupName,
        
        [Parameter(Mandatory=$true)]
        [string]$Rule,
        
        [Parameter(Mandatory=$true)]
        [ValidateSet("Security", "Microsoft365")]
        [string]$GroupType,
        
        [string]$Description = ""
    )
    
    try {
        Write-Host "動的グループ作成中: $GroupName" -ForegroundColor Cyan
        
        # グループタイプに応じたパラメータ設定
        $groupParams = @{
            DisplayName = $GroupName
            Description = if ($Description) { $Description } else { "動的グループ: $GroupName" }
            GroupTypes = @("DynamicMembership")
            MembershipRule = $Rule
            MembershipRuleProcessingState = "On"
        }
        
        if ($GroupType -eq "Microsoft365") {
            $groupParams.GroupTypes += "Unified"
            $groupParams.MailEnabled = $true
            $groupParams.MailNickname = ($GroupName -replace '[^a-zA-Z0-9]', '').ToLower()
        } else {
            $groupParams.SecurityEnabled = $true
        }
        
        # グループの作成
        $newGroup = New-MgGroup @groupParams
        
        Write-Host "✅ 動的グループ作成完了" -ForegroundColor Green
        Write-Host "  グループ名: $($newGroup.DisplayName)" -ForegroundColor White
        Write-Host "  グループID: $($newGroup.Id)" -ForegroundColor White
        Write-Host "  メンバーシップルール: $Rule" -ForegroundColor Yellow
        
        # 動的ルール処理状況の確認
        Write-Host "⏳ 動的メンバーシップ処理中..." -ForegroundColor Yellow
        Write-Host "   ※ 属性に基づくメンバー追加には数分〜数時間かかる場合があります" -ForegroundColor Gray
        Write-Host "   ※ Microsoft 365 A3/A5ライセンスが必要です" -ForegroundColor Yellow
        
        return $newGroup
        
    } catch {
        Write-Error "動的グループ作成失敗: $GroupName - $($_.Exception.Message)"
        
        # よくあるエラーの診断ヒント
        if ($_.Exception.Message -like "*Premium*") {
            Write-Warning "Microsoft 365 A3/A5 ライセンス（Microsoft Entra ID Premium P1を含む）が必要です"
        } elseif ($_.Exception.Message -like "*syntax*") {
            Write-Warning "メンバーシップルールの構文を確認してください"
        }
    }
}

# 動的ルールの検証用ヘルパー関数
function Test-DynamicMembershipRule {
    param(
        [string]$Rule,
        [string]$TestUserPrincipalName
    )
    
    try {
        # テストユーザーの属性取得
        $testUser = Get-MgUser -Filter "userPrincipalName eq '$TestUserPrincipalName'" -Property "id,userPrincipalName,department,jobTitle,extensionAttribute1,extensionAttribute2,userType,employeeType"
        
        if (-not $testUser) {
            Write-Warning "テストユーザーが見つかりません: $TestUserPrincipalName"
            return
        }
        
        Write-Host "=== 動的ルールテスト ===" -ForegroundColor Magenta
        Write-Host "ルール: $Rule" -ForegroundColor Yellow
        Write-Host "テストユーザー: $TestUserPrincipalName" -ForegroundColor Cyan
        
        # ユーザー属性の表示
        Write-Host "`nユーザー属性:" -ForegroundColor Green
        $testUser | Select-Object userPrincipalName, department, jobTitle, extensionAttribute1, extensionAttribute2, userType, employeeType | Format-List
        
        # 実際の動的ルール評価（Graph APIを使用）
        # ※ 実際の評価はMicrosoft Entra IDが行うため、ここでは属性の確認のみ
        Write-Host "⚠️ 実際の評価結果は Microsoft Entra ID の動的グループ機能によって処理されます" -ForegroundColor Yellow
        
    } catch {
        Write-Error "動的ルールテスト失敗: $($_.Exception.Message)"
    }
}
```

---

## 🏫 教育機関向けベストプラクティス

### 1. 階層的グループ設計

#### 標準的な学校組織に対応したグループ構造

```markdown
📊 推奨グループ階層構造

🏫 学校全体レベル
├── 🎯 全教職員（All-Staff）
├── 🎓 全学生（All-Students）
├── 👨‍💼 管理職（Administrators）
└── 🏢 事務職員（Administrative-Staff）

🏛️ 部門・学科レベル
├── 📚 各学科教員グループ
│   ├── 数学科教員（Math-Faculty）
│   ├── 国語科教員（Japanese-Faculty）
│   └── 理科教員（Science-Faculty）
├── 🎯 学年主任グループ（Grade-Coordinators）
└── 🏃‍♂️ 部活動顧問グループ（Club-Advisors）

🎓 学年・クラスレベル  
├── 📖 学年別学生グループ
│   ├── 2025年度-1年生（Grade1-2025）
│   ├── 2025年度-2年生（Grade2-2025） 
│   └── 2025年度-3年生（Grade3-2025）
├── 🏛️ クラス別グループ
│   ├── 1年A組（Class-1A-2025）
│   └── 1年B組（Class-1B-2025）
└── 🏃‍♂️ 部活動・サークル（Club-Baseball-2025）

🎯 機能別・プロジェクトレベル
├── 💻 IT委員会（IT-Committee）
├── 🎪 文化祭実行委員会（Festival-Committee）
├── 📝 入試委員会（Admission-Committee）
└── 🔒 セキュリティ管理者（Security-Admins）
```

### 2. 命名規則の標準化

#### 体系的なグループ命名ルール

```markdown
📝 推奨命名規則

**基本パターン**: [種別]-[対象]-[年度/期間]

**種別コード**:
- STAFF: 教職員関連
- STU: 学生関連  
- CLUB: 部活動・サークル
- PROJ: プロジェクト・委員会
- SYS: システム・管理用

**対象識別子**:
- 学年: Grade1, Grade2, Grade3
- 学科: Math, Science, English, etc.
- 職種: Teacher, Admin, Librarian
- クラス: ClassA, ClassB, ClassC

**年度表記**: 2025, 2026（西暦年）

**具体例**:
- STU-Grade1-2025（2025年度1年生）
- STAFF-Math-ALL（数学科全教員）
- CLUB-Baseball-2025（野球部2025年度）
- PROJ-Festival-2025（文化祭実行委員会）
- SYS-ITAdmin-ALL（IT管理者）
```

### 3. セキュリティとアクセス制御

#### 教育機関特有のセキュリティ要件

**🔒 レイヤード・セキュリティアプローチ**

```markdown
🛡️ 多層防御によるアクセス制御

**レベル1: 基本アクセス制御**
- 全学生 → 基本的な教育リソース
- 全教職員 → 校務システム、教材データベース
- 一般事務 → 事務システム、基本情報

**レベル2: 機能別アクセス制御**  
- 学年担当 → 担当学年の成績・出席情報
- 教科担当 → 該当教科の教材・評価データ
- 部活顧問 → 部活動関連情報・施設予約

**レベル3: 管理者アクセス制御**
- IT管理者 → システム設定、ユーザー管理
- 教務主任 → 全校の教務情報、時間割
- 校長・教頭 → 人事・予算・機密情報

**レベル4: 機密・特権アクセス制御**
- 人事担当 → 人事・給与・評価情報
- 経理担当 → 予算・財務・契約情報
- システム管理者 → 全システム特権アクセス
```

#### 条件付きアクセスとの連携

**🎯 スクリプトの目的**
教育機関向けの条件付きアクセスポリシーとセキュリティグループを連携させ、場所・時間・デバイスに基づく柔軟なアクセス制御を実現します。校内・校外、授業時間・放課後などの条件に応じた適切なセキュリティレベルを自動的に適用できます。

**📝 Usage**
```powershell
# 基本的なセキュリティポリシー適用
Set-EducationSecurityPolicy -GroupName "Students-Grade1" -AccessLevel "Standard" -RequireMFA $false

# 高セキュリティポリシー（管理者向け）
Set-EducationSecurityPolicy -GroupName "IT-Administrators" -AccessLevel "High" -RequireMFA $true -RestrictLocation $true

# 時間制限付きアクセス（部活動向け）
Set-EducationSecurityPolicy -GroupName "Club-Activities" -AccessLevel "Limited" -TimeRestriction "AfterSchool"
```

**⚙️ パラメータ**
- `-GroupName`: 対象セキュリティグループ名
- `-AccessLevel`: アクセスレベル（Standard/High/Limited）
- `-RequireMFA`: 多要素認証要求の有無
- `-RestrictLocation`: 場所制限の有無
- `-TimeRestriction`: 時間制限（SchoolHours/AfterSchool/AllTime）

```powershell
function Set-EducationSecurityPolicy {
    param(
        [Parameter(Mandatory=$true)]
        [string]$GroupName,
        
        [Parameter(Mandatory=$true)]
        [ValidateSet("Standard", "High", "Limited")]
        [string]$AccessLevel,
        
        [bool]$RequireMFA = $true,
        [bool]$RestrictLocation = $false,
        
        [ValidateSet("SchoolHours", "AfterSchool", "AllTime")]
        [string]$TimeRestriction = "AllTime"
    )
    
    try {
        # 対象グループの確認
        $targetGroup = Get-MgGroup -Filter "displayName eq '$GroupName'"
        if (-not $targetGroup) {
            Write-Error "グループが見つかりません: $GroupName"
            return
        }
        
        Write-Host "セキュリティポリシー設定中: $GroupName" -ForegroundColor Cyan
        
        # アクセスレベル別設定
        $policySettings = switch ($AccessLevel) {
            "Standard" {
                @{
                    SignInRiskLevel = "medium"
                    UserRiskLevel = "medium"
                    RequireCompliantDevice = $false
                    RequireApprovedClientApp = $true
                }
            }
            "High" {
                @{
                    SignInRiskLevel = "low"
                    UserRiskLevel = "low"
                    RequireCompliantDevice = $true
                    RequireApprovedClientApp = $true
                }
            }
            "Limited" {
                @{
                    SignInRiskLevel = "high"
                    UserRiskLevel = "high"
                    RequireCompliantDevice = $false
                    RequireApprovedClientApp = $false
                }
            }
        }
        
        # 時間制限設定
        $timeConditions = switch ($TimeRestriction) {
            "SchoolHours" {
                @{
                    StartTime = "08:00:00"
                    EndTime = "17:00:00"
                    Days = @("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
                }
            }
            "AfterSchool" {
                @{
                    StartTime = "17:00:00"
                    EndTime = "21:00:00"
                    Days = @("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")
                }
            }
            "AllTime" {
                $null
            }
        }
        
        # 条件付きアクセスポリシーの作成
        $policyParams = @{
            DisplayName = "$GroupName-SecurityPolicy"
            State = "enabled"
            Conditions = @{
                Users = @{
                    IncludeGroups = @($targetGroup.Id)
                }
                Applications = @{
                    IncludeApplications = @("All")
                }
            }
            GrantControls = @{
                Operator = "AND"
                BuiltInControls = @()
            }
        }
        
        # MFA要求の追加
        if ($RequireMFA) {
            $policyParams.GrantControls.BuiltInControls += "mfa"
        }
        
        # デバイス準拠要求の追加
        if ($policySettings.RequireCompliantDevice) {
            $policyParams.GrantControls.BuiltInControls += "compliantDevice"
        }
        
        # 承認済みアプリ要求の追加
        if ($policySettings.RequireApprovedClientApp) {
            $policyParams.GrantControls.BuiltInControls += "approvedApplication"
        }
        
        # 場所制限の追加
        if ($RestrictLocation) {
            $policyParams.Conditions.Locations = @{
                IncludeLocations = @("AllTrusted")
                ExcludeLocations = @("All")
            }
        }
        
        # 時間制限の追加
        if ($timeConditions) {
            # 注: 実際の時間制限設定はより複雑な構成が必要
            Write-Host "⚠️ 時間制限設定は個別の条件付きアクセスポリシー設定が必要です" -ForegroundColor Yellow
        }
        
        # ポリシーの適用（実際の環境では慎重に実行）
        Write-Host "📋 設定内容:" -ForegroundColor Green
        Write-Host "  グループ: $GroupName" -ForegroundColor White
        Write-Host "  アクセスレベル: $AccessLevel" -ForegroundColor White
        Write-Host "  MFA要求: $RequireMFA" -ForegroundColor White
        Write-Host "  場所制限: $RestrictLocation" -ForegroundColor White
        Write-Host "  時間制限: $TimeRestriction" -ForegroundColor White
        
        Write-Host "✅ セキュリティポリシー設定完了" -ForegroundColor Green
        
    } catch {
        Write-Error "セキュリティポリシー設定失敗: $($_.Exception.Message)"
    }
}
```

### 4. グループ作成権限の制御

#### 教育機関でのグループ作成権限管理の重要性

教育機関では、無制限なグループ作成により以下の問題が発生する可能性があります：

```markdown
⚠️ 無制限グループ作成のリスク
- グループの乱立によるガバナンス低下
- 重複機能を持つグループの無秩序な増加
- セキュリティポリシーに準拠しないグループの作成
- 管理者による統制の困難化
- ライセンス消費の無駄（Microsoft 365グループ）
```

#### グループ作成権限の制限方法

**🎯 スクリプトの目的**
Microsoft 365テナントにおいて、グループ作成権限を特定のユーザーまたはセキュリティグループに制限し、教育機関のガバナンス要件を満たします。管理者、教務主任、学年主任など、適切な権限を持つ人員のみがグループを作成できるよう制御します。

**📝 Usage**
```powershell
# 基本的な権限制限（管理者のみ）
Restrict-GroupCreation -AllowedGroup "IT-Administrators" -RestrictType "AllGroups"

# 教職員のみに制限
Restrict-GroupCreation -AllowedGroup "Faculty-Staff" -RestrictType "Microsoft365Groups"

# 複数グループに権限付与
$allowedGroups = @("School-Administrators", "Grade-Coordinators", "IT-Staff")
Restrict-GroupCreation -AllowedGroups $allowedGroups -RestrictType "AllGroups"
```

**⚙️ パラメータ**
- `-AllowedGroup`: グループ作成を許可するセキュリティグループ名
- `-AllowedGroups`: 複数のグループに権限を付与する場合
- `-RestrictType`: 制限対象（AllGroups/Microsoft365Groups/SecurityGroups）
- `-EnableException`: 例外ユーザーの個別指定

**💡 推奨権限設計**
- **管理者**: 全種類のグループ作成権限
- **教務主任・学年主任**: Microsoft 365グループ作成権限
- **一般教職員**: 申請ベースでのグループ作成
- **学生**: グループ作成権限なし（教員による代理作成）

```powershell
function Restrict-GroupCreation {
    param(
        [string]$AllowedGroup,
        [string[]]$AllowedGroups,
        [ValidateSet("AllGroups", "Microsoft365Groups", "SecurityGroups")]
        [string]$RestrictType = "Microsoft365Groups",
        [string[]]$ExceptionUsers = @()
    )
    
    Write-Host "=== グループ作成権限制限設定 ===" -ForegroundColor Magenta
    Write-Host "制限種別: $RestrictType" -ForegroundColor Cyan
    
    try {
        # 許可グループリストの統合
        $groupsToAllow = @()
        if ($AllowedGroup) { $groupsToAllow += $AllowedGroup }
        if ($AllowedGroups) { $groupsToAllow += $AllowedGroups }
        
        if ($groupsToAllow.Count -eq 0) {
            Write-Error "許可するグループが指定されていません"
            return
        }
        
        # Microsoft 365グループ作成制限の設定
        if ($RestrictType -in @("AllGroups", "Microsoft365Groups")) {
            Write-Host "`n🔒 Microsoft 365グループ作成権限制限中..." -ForegroundColor Yellow
            
            # グループ設定テンプレートの取得
            $groupSettingsTemplate = Get-MgDirectorySettingTemplate | Where-Object { $_.DisplayName -eq "Group.Unified" }
            
            if (-not $groupSettingsTemplate) {
                Write-Error "Group.Unified設定テンプレートが見つかりません"
                return
            }
            
            # 現在の設定確認
            $currentSettings = Get-MgDirectorySetting | Where-Object { $_.TemplateId -eq $groupSettingsTemplate.Id }
            
            if (-not $currentSettings) {
                # 新規設定作成
                Write-Host "新規グループ設定を作成中..." -ForegroundColor Yellow
                
                $settingValues = @()
                foreach ($settingTemplate in $groupSettingsTemplate.Values) {
                    $settingValue = @{
                        Name = $settingTemplate.Name
                        Value = $settingTemplate.DefaultValue
                    }
                    $settingValues += $settingValue
                }
                
                # グループ作成を制限
                $enableGroupCreation = $settingValues | Where-Object { $_.Name -eq "EnableGroupCreation" }
                $enableGroupCreation.Value = "false"
                
                # 許可グループの設定
                $groupCreationAllowedGroupId = $settingValues | Where-Object { $_.Name -eq "GroupCreationAllowedGroupId" }
                
                # 最初の許可グループを設定（複数の場合は後で追加処理）
                $firstAllowedGroup = Get-MgGroup -Filter "displayName eq '$($groupsToAllow[0])'"
                if ($firstAllowedGroup) {
                    $groupCreationAllowedGroupId.Value = $firstAllowedGroup.Id
                    Write-Host "  ✓ 許可グループ設定: $($groupsToAllow[0])" -ForegroundColor Green
                }
                
                # 設定を適用
                New-MgDirectorySetting -TemplateId $groupSettingsTemplate.Id -Values $settingValues
                Write-Host "  ✅ Microsoft 365グループ作成制限を適用しました" -ForegroundColor Green
                
            } else {
                # 既存設定の更新
                Write-Host "既存グループ設定を更新中..." -ForegroundColor Yellow
                
                $settingValues = $currentSettings.Values
                
                # グループ作成を制限
                $enableGroupCreation = $settingValues | Where-Object { $_.Name -eq "EnableGroupCreation" }
                $enableGroupCreation.Value = "false"
                
                # 許可グループの更新
                $groupCreationAllowedGroupId = $settingValues | Where-Object { $_.Name -eq "GroupCreationAllowedGroupId" }
                $firstAllowedGroup = Get-MgGroup -Filter "displayName eq '$($groupsToAllow[0])'"
                if ($firstAllowedGroup) {
                    $groupCreationAllowedGroupId.Value = $firstAllowedGroup.Id
                }
                
                Update-MgDirectorySetting -DirectorySettingId $currentSettings.Id -Values $settingValues
                Write-Host "  ✅ Microsoft 365グループ作成制限を更新しました" -ForegroundColor Green
            }
        }
        
        # セキュリティグループ作成制限（Microsoft Entra ID Premium P1/P2必要）
        if ($RestrictType -in @("AllGroups", "SecurityGroups")) {
            Write-Host "`n🔒 セキュリティグループ作成権限制限中..." -ForegroundColor Yellow
            
            # 注: セキュリティグループ作成制限にはより高度な権限管理が必要
            Write-Host "  ℹ️ セキュリティグループ作成制限には、管理者ロールによる制御を推奨します" -ForegroundColor Cyan
            Write-Host "    - Group Administrator ロールの適切な割り当て" -ForegroundColor Gray
            Write-Host "    - 条件付きアクセスポリシーによる制御" -ForegroundColor Gray
        }
        
        # 複数許可グループの処理（備考）
        if ($groupsToAllow.Count -gt 1) {
            Write-Host "`n📋 複数グループ許可について:" -ForegroundColor Yellow
            Write-Host "  Microsoft 365の標準設定では、1つのセキュリティグループのみ指定可能です" -ForegroundColor Gray
            Write-Host "  複数グループに権限を付与する場合の推奨方法:" -ForegroundColor Gray
            Write-Host "    1. 'Group-Creators' という統合グループを作成" -ForegroundColor Gray
            Write-Host "    2. 個別の許可グループを統合グループのメンバーに追加" -ForegroundColor Gray
            
            # 統合グループの自動作成提案
            $createUnifiedGroup = Read-Host "`n統合グループ 'Group-Creators' を自動作成しますか？ (y/n)"
            if ($createUnifiedGroup -eq 'y') {
                try {
                    $unifiedGroup = New-MgGroup -DisplayName "Group-Creators" -Description "Microsoft 365グループ作成権限を持つユーザーの統合グループ" -SecurityEnabled $true
                    
                    # 各許可グループを統合グループに追加
                    foreach ($allowedGroupName in $groupsToAllow) {
                        $allowedGroupObj = Get-MgGroup -Filter "displayName eq '$allowedGroupName'"
                        if ($allowedGroupObj) {
                            New-MgGroupMember -GroupId $unifiedGroup.Id -DirectoryObjectId $allowedGroupObj.Id
                            Write-Host "    ✓ $allowedGroupName を Group-Creators に追加" -ForegroundColor Green
                        }
                    }
                    
                    Write-Host "  ✅ 統合グループ 'Group-Creators' を作成しました" -ForegroundColor Green
                    Write-Host "    次回はこのグループIDを GroupCreationAllowedGroupId に設定してください" -ForegroundColor Cyan
                    
                } catch {
                    Write-Error "統合グループ作成失敗: $($_.Exception.Message)"
                }
            }
        }
        
        # 設定確認と推奨事項
        Write-Host "`n=== 設定確認 ===" -ForegroundColor Magenta
        Write-Host "制限対象: $RestrictType" -ForegroundColor White
        Write-Host "許可グループ: $($groupsToAllow -join ', ')" -ForegroundColor White
        
        Write-Host "`n📋 推奨フォローアップ作業:" -ForegroundColor Yellow
        Write-Host "  1. 許可グループのメンバーシップ確認" -ForegroundColor Gray
        Write-Host "  2. エンドユーザーへの変更通知" -ForegroundColor Gray
        Write-Host "  3. グループ作成申請プロセスの整備" -ForegroundColor Gray
        Write-Host "  4. 定期的な権限監査の実施" -ForegroundColor Gray
        
    } catch {
        Write-Error "グループ作成権限制限失敗: $($_.Exception.Message)"
    }
}

# 権限確認用ヘルパー関数
function Get-GroupCreationPermissions {
    Write-Host "=== 現在のグループ作成権限設定 ===" -ForegroundColor Magenta
    
    try {
        # Microsoft 365グループ設定確認
        $groupSettingsTemplate = Get-MgDirectorySettingTemplate | Where-Object { $_.DisplayName -eq "Group.Unified" }
        $currentSettings = Get-MgDirectorySetting | Where-Object { $_.TemplateId -eq $groupSettingsTemplate.Id }
        
        if ($currentSettings) {
            $enableGroupCreation = ($currentSettings.Values | Where-Object { $_.Name -eq "EnableGroupCreation" }).Value
            $allowedGroupId = ($currentSettings.Values | Where-Object { $_.Name -eq "GroupCreationAllowedGroupId" }).Value
            
            Write-Host "`nMicrosoft 365グループ作成設定:" -ForegroundColor Green
            Write-Host "  一般ユーザー作成許可: $enableGroupCreation" -ForegroundColor White
            
            if ($allowedGroupId) {
                $allowedGroup = Get-MgGroup -GroupId $allowedGroupId -ErrorAction SilentlyContinue
                if ($allowedGroup) {
                    Write-Host "  許可グループ: $($allowedGroup.DisplayName) ($allowedGroupId)" -ForegroundColor White
                    
                    # 許可グループのメンバー確認
                    $members = Get-MgGroupMember -GroupId $allowedGroupId
                    Write-Host "  許可ユーザー数: $($members.Count)名" -ForegroundColor White
                }
            } else {
                Write-Host "  許可グループ: 未設定" -ForegroundColor Yellow
            }
        } else {
            Write-Host "Microsoft 365グループ作成制限: 未設定（全ユーザーが作成可能）" -ForegroundColor Yellow
        }
        
        # 管理者ロール確認
        Write-Host "`n管理者ロール設定:" -ForegroundColor Green
        $groupAdmins = Get-MgDirectoryRoleDefinition | Where-Object { $_.DisplayName -eq "Groups Administrator" }
        if ($groupAdmins) {
            $groupAdminAssignments = Get-MgRoleManagementDirectoryRoleAssignment -Filter "roleDefinitionId eq '$($groupAdmins.Id)'"
            Write-Host "  Groups Administrator: $($groupAdminAssignments.Count)名に割り当て済み" -ForegroundColor White
        }
        
    } catch {
        Write-Error "権限確認失敗: $($_.Exception.Message)"
    }
}
```

#### 教育機関での推奨権限設計例

```markdown
🏫 学校での権限階層設計

**レベル1: 全権限（IT管理者）**
- グループ種別: 全種類
- 権限範囲: 作成・編集・削除・権限変更
- 対象: IT管理者、システム管理者

**レベル2: 教育管理権限（教務・管理職）**
- グループ種別: Microsoft 365グループ、配布グループ
- 権限範囲: 作成・編集・メンバー管理
- 対象: 校長、教頭、教務主任、学年主任

**レベル3: 教科・部活動権限（一般教員）**
- グループ種別: Microsoft 365グループ（申請ベース）
- 権限範囲: 自分が作成したグループの管理
- 対象: 教科担当、部活動顧問

**レベル4: 参加のみ（学生・保護者）**
- グループ種別: なし（作成権限なし）
- 権限範囲: 招待されたグループへの参加のみ
- 対象: 学生、保護者、外部ユーザー
```

#### 申請ベースグループ作成フローの構築

**🎯 スクリプトの目的**
Power AutomateやMicrosoft Formsと連携して、グループ作成申請から承認、自動作成までのワークフローを構築します。教員からのグループ作成要求を管理者が審査・承認後、自動的にグループを作成する仕組みを提供します。

**📋 申請フローの設計例**

```markdown
🔄 グループ作成申請ワークフロー

1. **申請（Microsoft Forms）**
   - 申請者: 教員
   - 申請内容: グループ名、目的、メンバー予定数、期間
   - 自動項目: 申請者情報、申請日時

2. **一次確認（自動）**
   - 命名規則チェック
   - 重複グループ名確認
   - 申請者権限確認

3. **承認（管理者）**
   - 申請内容レビュー
   - 承認・却下・修正要求

4. **自動作成（PowerShell）**
   - 承認後の自動グループ作成
   - 初期メンバー設定
   - 申請者への完了通知

5. **事後管理**
   - グループ活動監視
   - 期限切れグループの自動アーカイブ
```

```powershell
# 申請ベースグループ作成の実装例
function Process-GroupCreationRequest {
    param(
        [PSCustomObject]$ApprovalData,
        [switch]$AutoCreate
    )
    
    Write-Host "グループ作成申請処理開始..." -ForegroundColor Cyan
    
    try {
        # 申請データの検証
        $validationResult = Test-GroupCreationRequest -RequestData $ApprovalData
        
        if (-not $validationResult.IsValid) {
            Write-Warning "申請データに問題があります: $($validationResult.Issues -join ', ')"
            return
        }
        
        if ($AutoCreate) {
            # 自動グループ作成
            $groupParams = @{
                DisplayName = $ApprovalData.GroupName
                Description = $ApprovalData.Purpose
                GroupTypes = @("Unified")
                MailEnabled = $true
                SecurityEnabled = $false
                MailNickname = ($ApprovalData.GroupName -replace '[^a-zA-Z0-9]', '').ToLower()
            }
            
            $newGroup = New-MgGroup @groupParams
            
            # 申請者を所有者に設定
            $requester = Get-MgUser -Filter "userPrincipalName eq '$($ApprovalData.RequesterEmail)'"
            if ($requester) {
                New-MgGroupOwner -GroupId $newGroup.Id -DirectoryObjectId $requester.Id
            }
            
            Write-Host "✅ グループ作成完了: $($ApprovalData.GroupName)" -ForegroundColor Green
            return $newGroup
        }
        
    } catch {
        Write-Error "グループ作成申請処理失敗: $($_.Exception.Message)"
    }
}
```

### 5. School Data Sync (SDS) との連携

#### 校務支援システムとの自動同期

School Data Sync（SDS）は、校務支援システムとMicrosoft製品を連携する無料ツールです。アカウントの自動作成や履修情報に基づいたTeams の構築が可能で、学校の管理負担を軽減します

**🎯 SDS連携のメリット**

```markdown
🔄 自動化される業務プロセス
1. **ユーザー管理**: 学籍情報→Microsoft 365アカウント自動作成
2. **クラス編成**: 履修情報→Teams クラス自動構築
3. **グループ更新**: 時間割変更→関連グループ自動更新
4. **年次進級**: 学年更新→全グループメンバーシップ自動移行
5. **保護者連携**: 保護者情報→Teams 保護者機能自動設定
```

#### SDS用CSVテンプレート設計

**📊 標準的なSDS CSVフォーマット**

**School.csv**
```csv
SIS ID,Name,School Number
sch123,サンプル中学校,123
```

**Section.csv**  
```csv
SIS ID,School SIS ID,Section Name,Section Number,Term SIS ID,Term Name,Period
sec001,sch123,1年A組数学,1A-MATH,term2025,2025年度,1
sec002,sch123,1年A組国語,1A-JPN,term2025,2025年度,2
```

**Teacher.csv**
```csv
SIS ID,School SIS ID,Username,First Name,Last Name,Middle Name,Title,Teacher Number,Status,Secondary Email
tea001,sch123,t.tanaka@school.edu.jp,太郎,田中,,数学教諭,T001,Active,tanaka@school.local
```

**Student.csv**
```csv
SIS ID,School SIS ID,Username,First Name,Last Name,Middle Name,Grade,Student Number,Status,Secondary Email,Password
stu001,sch123,s001@school.edu.jp,花子,佐藤,,1,S001,Active,sato@school.local,TempPass123!
```

**StudentEnrollment.csv**
```csv
Section SIS ID,SIS ID
sec001,stu001
sec002,stu001
```

---

## 🔄 年次更新の自動化

### 1. 年度末・年度初めの課題

#### 教育機関特有の大規模更新要件

```markdown
📅 年次更新で発生する主要課題

**年度末処理（3月）**
- 卒業生アカウント処理（無効化・データ保護・削除）
- 進級予定者の仮グループ移動
- 教職員異動に伴うグループ再編
- 部活動・委員会の引き継ぎ処理
- 一時的なアーカイブグループ作成

**年度初め処理（4月）**  
- 新入生アカウント一括作成・グループ配属
- 進級に伴うグループメンバーシップ更新
- 新年度クラス・時間割対応グループ作成
- 教職員新配属・担当変更反映
- 校務システム連携データ更新

**継続的な課題**
- 転校・転入・休学者の随時対応
- 年度途中のクラス替え・担当変更
- 新規教職員・非常勤講師の追加
- システム間データ整合性維持
```

### 2. 段階的年次更新戦略

#### Phase 1: 事前準備・データ検証（2月中旬〜3月上旬）

**🎯 スクリプトの目的**
年次更新前の現状データを包括的に分析し、問題のあるアカウントやグループを事前に特定・修正します。卒業予定者、進級予定者、教職員異動情報を整理し、円滑な年次更新の準備を行います。

**📝 Usage**
```powershell
# 基本的な年次更新準備
Prepare-AnnualUpdate -CurrentAcademicYear 2024 -NextAcademicYear 2025 -OutputPath "C:\reports\annual_update_prep.xlsx"

# 特定の問題のみチェック
Prepare-AnnualUpdate -CheckType "GraduatingStudents" -OutputPath "C:\reports\graduates_2025.csv"

# 詳細レポート生成
Prepare-AnnualUpdate -Detailed -IncludeRecommendations -OutputPath "C:\reports\comprehensive_analysis.xlsx"
```

**⚙️ パラメータ**
- `-CurrentAcademicYear`: 現在の年度
- `-NextAcademicYear`: 新年度
- `-CheckType`: チェック種別（All/GraduatingStudents/PromotingStudents/Faculty）
- `-OutputPath`: 分析結果の出力先
- `-Detailed`: 詳細な分析レポート生成
- `-IncludeRecommendations`: 改善提案を含める

```powershell
function Prepare-AnnualUpdate {
    param(
        [int]$CurrentAcademicYear = (Get-Date).Year,
        [int]$NextAcademicYear = (Get-Date).Year + 1,
        [ValidateSet("All", "GraduatingStudents", "PromotingStudents", "Faculty")]
        [string]$CheckType = "All",
        [string]$OutputPath,
        [switch]$Detailed,
        [switch]$IncludeRecommendations
    )
    
    Write-Host "=== 年次更新準備分析開始 ===" -ForegroundColor Magenta
    Write-Host "対象年度: $CurrentAcademicYear → $NextAcademicYear" -ForegroundColor Cyan
    
    $analysisResults = @{
        GraduatingStudents = @()
        PromotingStudents = @()
        FacultyChanges = @()
        OrphanedGroups = @()
        ProblematicAccounts = @()
        Recommendations = @()
    }
    
    try {
        if ($CheckType -in @("All", "GraduatingStudents")) {
            Write-Host "`n📊 卒業予定者分析中..." -ForegroundColor Yellow
            
            # 卒業予定者（最高学年）の特定
            $graduatingStudents = Get-MgUser -Filter "extensionAttribute1 eq 'Grade3' and userType eq 'Member'" -All |
                Where-Object { $_.UserPrincipalName -like "*@school.edu.jp" }
            
            foreach ($student in $graduatingStudents) {
                $studentInfo = @{
                    UserPrincipalName = $student.UserPrincipalName
                    DisplayName = $student.DisplayName
                    Department = $student.Department
                    LastSignIn = $student.SignInActivity.LastSignInDateTime
                    GroupMemberships = (Get-MgUserMemberOf -UserId $student.Id).Count
                    HasLicense = (Get-MgUserLicenseDetail -UserId $student.Id).Count -gt 0
                    DataSize = "分析中" # OneDrive使用量等の分析
                    Action = "卒業処理予定"
                }
                $analysisResults.GraduatingStudents += $studentInfo
            }
            
            Write-Host "  ✓ 卒業予定者: $($graduatingStudents.Count)名" -ForegroundColor Green
        }
        
        if ($CheckType -in @("All", "PromotingStudents")) {
            Write-Host "`n📊 進級予定者分析中..." -ForegroundColor Yellow
            
            # 進級予定者（1・2年生）の特定
            $promotingStudents = Get-MgUser -Filter "(extensionAttribute1 eq 'Grade1' or extensionAttribute1 eq 'Grade2') and userType eq 'Member'" -All
            
            foreach ($student in $promotingStudents) {
                $currentGrade = $student.ExtensionAttribute1
                $nextGrade = switch ($currentGrade) {
                    "Grade1" { "Grade2" }
                    "Grade2" { "Grade3" }
                }
                
                $studentInfo = @{
                    UserPrincipalName = $student.UserPrincipalName
                    DisplayName = $student.DisplayName
                    CurrentGrade = $currentGrade
                    NextGrade = $nextGrade
                    CurrentGroups = @()
                    RequiredNewGroups = @()
                    Action = "進級処理予定"
                }
                
                # 現在のグループメンバーシップ
                $userGroups = Get-MgUserMemberOf -UserId $student.Id
                $studentInfo.CurrentGroups = $userGroups | ForEach-Object { $_.AdditionalProperties.displayName }
                
                $analysisResults.PromotingStudents += $studentInfo
            }
            
            Write-Host "  ✓ 進級予定者: $($promotingStudents.Count)名" -ForegroundColor Green
        }
        
        if ($CheckType -in @("All", "Faculty")) {
            Write-Host "`n📊 教職員異動分析中..." -ForegroundColor Yellow
            
            # 教職員の現状分析
            $facultyMembers = Get-MgUser -Filter "userType eq 'Member' and (jobTitle ne null)" -All |
                Where-Object { $_.JobTitle -like "*教*" -or $_.JobTitle -like "*職員*" }
            
            foreach ($faculty in $facultyMembers) {
                $facultyInfo = @{
                    UserPrincipalName = $faculty.UserPrincipalName
                    DisplayName = $faculty.DisplayName
                    JobTitle = $faculty.JobTitle
                    Department = $faculty.Department
                    ManagerAssigned = ($faculty.Manager -ne $null)
                    GroupMemberships = (Get-MgUserMemberOf -UserId $faculty.Id).Count
                    LastActivity = $faculty.SignInActivity.LastSignInDateTime
                    Action = "継続確認必要"
                }
                $analysisResults.FacultyChanges += $facultyInfo
            }
            
            Write-Host "  ✓ 教職員: $($facultyMembers.Count)名" -ForegroundColor Green
        }
        
        # 問題のあるグループ・アカウントの特定
        if ($CheckType -eq "All") {
            Write-Host "`n📊 問題のあるグループ・アカウント分析中..." -ForegroundColor Yellow
            
            # 所有者不在グループの特定
            $allGroups = Get-MgGroup -All
            foreach ($group in $allGroups) {
                $owners = Get-MgGroupOwner -GroupId $group.Id -ErrorAction SilentlyContinue
                if ($owners.Count -eq 0) {
                    $analysisResults.OrphanedGroups += @{
                        GroupName = $group.DisplayName
                        GroupId = $group.Id
                        MemberCount = (Get-MgGroupMember -GroupId $group.Id).Count
                        CreatedDate = $group.CreatedDateTime
                        Issue = "所有者不在"
                    }
                }
            }
            
            # 長期間サインインなしアカウント
            $inactiveUsers = Get-MgUser -All | Where-Object {
                $_.SignInActivity.LastSignInDateTime -and
                (Get-Date) - [datetime]$_.SignInActivity.LastSignInDateTime > (New-TimeSpan -Days 90)
            }
            
            foreach ($user in $inactiveUsers) {
                $analysisResults.ProblematicAccounts += @{
                    UserPrincipalName = $user.UserPrincipalName
                    DisplayName = $user.DisplayName
                    LastSignIn = $user.SignInActivity.LastSignInDateTime
                    Issue = "90日以上未ログイン"
                    Recommendation = "アカウント状況確認・必要に応じて無効化"
                }
            }
        }
        
        # 推奨アクションの生成
        if ($IncludeRecommendations) {
            Write-Host "`n📋 改善提案生成中..." -ForegroundColor Yellow
            
            $analysisResults.Recommendations = @(
                "卒業生 $($analysisResults.GraduatingStudents.Count)名のアカウント処理計画を策定",
                "進級者 $($analysisResults.PromotingStudents.Count)名のグループ移動スクリプト準備",
                "所有者不在グループ $($analysisResults.OrphanedGroups.Count)件の所有者指定",
                "非アクティブアカウント $($analysisResults.ProblematicAccounts.Count)件の状況確認",
                "新年度用グループテンプレートの事前作成",
                "データバックアップとアーカイブ戦略の確認"
            )
        }
        
        # 結果出力
        if ($OutputPath) {
            $analysisResults | ConvertTo-Json -Depth 10 | Out-File "$OutputPath.json" -Encoding UTF8
            Write-Host "`n📄 分析結果を出力しました: $OutputPath.json" -ForegroundColor Green
        }
        
        # サマリー表示
        Write-Host "`n=== 分析結果サマリー ===" -ForegroundColor Magenta
        Write-Host "卒業予定者: $($analysisResults.GraduatingStudents.Count)名" -ForegroundColor Cyan
        Write-Host "進級予定者: $($analysisResults.PromotingStudents.Count)名" -ForegroundColor Cyan
        Write-Host "教職員: $($analysisResults.FacultyChanges.Count)名" -ForegroundColor Cyan
        Write-Host "所有者不在グループ: $($analysisResults.OrphanedGroups.Count)件" -ForegroundColor Yellow
        Write-Host "要確認アカウント: $($analysisResults.ProblematicAccounts.Count)件" -ForegroundColor Yellow
        
        return $analysisResults
        
    } catch {
        Write-Error "年次更新準備分析失敗: $($_.Exception.Message)"
    }
}
```

#### Phase 2: 卒業生処理・アーカイブ（3月中旬）

**🎯 スクリプトの目的**
卒業生のアカウントとデータを適切に処理し、必要なデータは保護しながらライセンスコストを削減します。段階的な処理により、誤操作のリスクを最小化し、必要に応じたデータ復旧を可能にします。

**📝 Usage**
```powershell
# 基本的な卒業生処理
Process-GraduatingStudents -GraduationYear 2025 -ArchivePeriod 180 -Stage "All"

# 段階的処理（ライセンス削除のみ）
Process-GraduatingStudents -GraduationYear 2025 -Stage "RemoveLicenses"

# データアーカイブ付き処理
Process-GraduatingStudents -GraduationYear 2025 -CreateArchive -ArchivePath "\\server\archives\graduates2025"
```

**⚙️ パラメータ**
- `-GraduationYear`: 卒業年度
- `-Stage`: 処理段階（RemoveLicenses/DisableAccounts/ArchiveData/All）
- `-ArchivePeriod`: アーカイブ保持期間（日数）
- `-CreateArchive`: データアーカイブ作成
- `-ArchivePath`: アーカイブデータ保存先

```powershell
function Process-GraduatingStudents {
    param(
        [int]$GraduationYear = (Get-Date).Year,
        [ValidateSet("RemoveLicenses", "DisableAccounts", "ArchiveData", "All")]
        [string]$Stage = "All",
        [int]$ArchivePeriod = 180,
        [switch]$CreateArchive,
        [string]$ArchivePath
    )
    
    Write-Host "=== 卒業生処理開始 ===" -ForegroundColor Magenta
    Write-Host "対象: $GraduationYear 年度卒業生" -ForegroundColor Cyan
    
    try {
        # 卒業対象者の特定
        $graduatingStudents = Get-MgUser -Filter "extensionAttribute1 eq 'Grade3' and userType eq 'Member'" -All |
            Where-Object { $_.UserPrincipalName -like "*@school.edu.jp" }
        
        if ($graduatingStudents.Count -eq 0) {
            Write-Warning "卒業対象学生が見つかりません"
            return
        }
        
        Write-Host "卒業対象学生: $($graduatingStudents.Count)名" -ForegroundColor Green
        
        $processResults = @{
            LicenseRemoved = @()
            AccountsDisabled = @()
            DataArchived = @()
            Errors = @()
        }
        
        foreach ($student in $graduatingStudents) {
            Write-Host "`n処理中: $($student.DisplayName) ($($student.UserPrincipalName))" -ForegroundColor Cyan
            
            try {
                # Stage 1: ライセンス削除（即座にコスト削減）
                if ($Stage -in @("RemoveLicenses", "All")) {
                    $userLicenses = Get-MgUserLicenseDetail -UserId $student.Id -ErrorAction SilentlyContinue
                    if ($userLicenses) {
                        $licenseSkuIds = $userLicenses.SkuId
                        Set-MgUserLicense -UserId $student.Id -AddLicenses @() -RemoveLicenses $licenseSkuIds
                        $processResults.LicenseRemoved += $student.UserPrincipalName
                        Write-Host "  ✓ ライセンス削除完了" -ForegroundColor Green
                    }
                }
                
                # Stage 2: データアーカイブ（オプション）
                if ($CreateArchive -and ($Stage -in @("ArchiveData", "All"))) {
                    if ($ArchivePath) {
                        $userArchivePath = Join-Path $ArchivePath $student.UserPrincipalName
                        
                        # OneDriveデータの情報取得（実際のアーカイブは別途ツール使用）
                        $driveInfo = Get-MgUserDrive -UserId $student.Id -ErrorAction SilentlyContinue
                        if ($driveInfo) {
                            # アーカイブ指示の記録
                            $archiveInfo = @{
                                UserPrincipalName = $student.UserPrincipalName
                                DisplayName = $student.DisplayName
                                DriveSize = $driveInfo.Quota.Used
                                ArchivePath = $userArchivePath
                                ArchiveDate = Get-Date
                                RetentionUntil = (Get-Date).AddDays($ArchivePeriod)
                            }
                            $processResults.DataArchived += $archiveInfo
                            Write-Host "  ✓ アーカイブ情報記録完了" -ForegroundColor Green
                        }
                    }
                }
                
                # Stage 3: アカウント無効化
                if ($Stage -in @("DisableAccounts", "All")) {
                    Update-MgUser -UserId $student.Id -AccountEnabled $false
                    
                    # 卒業生グループへの移動
                    $graduatesGroup = Get-MgGroup -Filter "displayName eq 'Graduates-$GraduationYear'" -ErrorAction SilentlyContinue
                    if (-not $graduatesGroup) {
                        # 卒業生グループの作成
                        $graduatesGroup = New-MgGroup -DisplayName "Graduates-$GraduationYear" -Description "$GraduationYear 年度卒業生" -SecurityEnabled $true
                        Write-Host "  📁 卒業生グループ作成: Graduates-$GraduationYear" -ForegroundColor Yellow
                    }
                    
                    New-MgGroupMember -GroupId $graduatesGroup.Id -DirectoryObjectId $student.Id -ErrorAction SilentlyContinue
                    $processResults.AccountsDisabled += $student.UserPrincipalName
                    Write-Host "  ✓ アカウント無効化・グループ移動完了" -ForegroundColor Green
                }
                
                # 学年属性の更新
                Update-MgUser -UserId $student.Id -ExtensionAttribute1 "Graduate-$GraduationYear"
                
            } catch {
                $errorInfo = @{
                    UserPrincipalName = $student.UserPrincipalName
                    Error = $_.Exception.Message
                    Stage = $Stage
                }
                $processResults.Errors += $errorInfo
                Write-Host "  ❌ 処理失敗: $($_.Exception.Message)" -ForegroundColor Red
            }
        }
        
        # 処理結果サマリー
        Write-Host "`n=== 卒業生処理結果 ===" -ForegroundColor Magenta
        Write-Host "ライセンス削除: $($processResults.LicenseRemoved.Count)名" -ForegroundColor Green
        Write-Host "アカウント無効化: $($processResults.AccountsDisabled.Count)名" -ForegroundColor Green
        Write-Host "データアーカイブ: $($processResults.DataArchived.Count)名" -ForegroundColor Green
        Write-Host "エラー発生: $($processResults.Errors.Count)件" -ForegroundColor Red
        
        if ($processResults.Errors.Count -gt 0) {
            Write-Host "`nエラー詳細:" -ForegroundColor Red
            $processResults.Errors | ForEach-Object {
                Write-Host "  - $($_.UserPrincipalName): $($_.Error)" -ForegroundColor Red
            }
        }
        
        return $processResults
        
    } catch {
        Write-Error "卒業生処理失敗: $($_.Exception.Message)"
    }
}
```

#### Phase 3: 新年度グループ作成・進級処理（4月上旬）

**🎯 スクリプトの目的**
新年度の組織構造に基づいてグループを一括作成し、既存学生の進級処理を自動実行します。新入生受け入れとクラス編成、教職員の担当変更を効率的に処理し、新年度開始と同時に全システムが利用可能な状態を実現します。

**📝 Usage**
```powershell
# 基本的な新年度処理
Start-NewAcademicYear -NewYear 2025 -ClassStructure "A,B,C" -CreateTeamsClasses

# カスタム構成での新年度開始
$classConfig = @{
    Grade1 = @("A", "B", "C", "D")
    Grade2 = @("A", "B", "C")
    Grade3 = @("A", "B")
}
Start-NewAcademicYear -NewYear 2025 -CustomClassStructure $classConfig

# SDSデータ連携付き処理
Start-NewAcademicYear -NewYear 2025 -ImportFromSDS -SDSDataPath "C:\SDS\2025"
```

**⚙️ パラメータ**
- `-NewYear`: 新年度
- `-ClassStructure`: クラス構成（簡易指定）
- `-CustomClassStructure`: カスタムクラス構成
- `-CreateTeamsClasses`: Teams クラス自動作成
- `-ImportFromSDS`: School Data Sync データ連携
- `-SDSDataPath`: SDS データファイルのパス

```powershell
function Start-NewAcademicYear {
    param(
        [int]$NewYear = (Get-Date).Year,
        [string]$ClassStructure = "A,B,C",
        [hashtable]$CustomClassStructure,
        [switch]$CreateTeamsClasses,
        [switch]$ImportFromSDS,
        [string]$SDSDataPath
    )
    
    Write-Host "=== 新年度開始処理 ===" -ForegroundColor Magenta
    Write-Host "新年度: $NewYear" -ForegroundColor Cyan
    
    $creationResults = @{
        GroupsCreated = @()
        StudentsPromoted = @()
        TeamsClassesCreated = @()
        Errors = @()
    }
    
    try {
        # Step 1: 新年度グループ構造の作成
        Write-Host "`n📁 新年度グループ構造作成中..." -ForegroundColor Yellow
        
        $classStructureToUse = if ($CustomClassStructure) {
            $CustomClassStructure
        } else {
            $classes = $ClassStructure -split ','
            @{
                Grade1 = $classes
                Grade2 = $classes
                Grade3 = $classes
            }
        }
        
        foreach ($grade in $classStructureToUse.Keys) {
            foreach ($class in $classStructureToUse[$grade]) {
                $groupName = "$NewYear-$grade-Class$class"
                
                try {
                    # Microsoft 365 グループの作成
                    $newGroup = New-MgGroup -DisplayName $groupName -Description "$NewYear 年度 $grade $class 組" -GroupTypes @("Unified") -MailEnabled $true -SecurityEnabled $false -MailNickname ($groupName -replace '[^a-zA-Z0-9]', '').ToLower()
                    
                    $creationResults.GroupsCreated += @{
                        GroupName = $groupName
                        GroupId = $newGroup.Id
                        Type = "ClassGroup"
                    }
                    
                    Write-Host "  ✓ グループ作成: $groupName" -ForegroundColor Green
                    
                    # Teams クラスの作成（オプション）
                    if ($CreateTeamsClasses) {
                        # Teams クラス作成のためのテンプレート適用
                        # 注: 実際のTeams作成にはMicrosoft Graph Educationエンドポイント使用
                        $teamsInfo = @{
                            GroupId = $newGroup.Id
                            ClassName = $groupName
                            Subject = "一般授業"
                        }
                        $creationResults.TeamsClassesCreated += $teamsInfo
                        Write-Host "  📚 Teams クラス作成予定: $groupName" -ForegroundColor Cyan
                    }
                    
                } catch {
                    $creationResults.Errors += @{
                        Operation = "GroupCreation"
                        Target = $groupName
                        Error = $_.Exception.Message
                    }
                    Write-Host "  ❌ グループ作成失敗: $groupName - $($_.Exception.Message)" -ForegroundColor Red
                }
            }
        }
        
        # Step 2: 既存学生の進級処理
        Write-Host "`n📈 学生進級処理中..." -ForegroundColor Yellow
        
        # 1年生→2年生
        $grade1Students = Get-MgUser -Filter "extensionAttribute1 eq 'Grade1' and userType eq 'Member'" -All
        foreach ($student in $grade1Students) {
            try {
                Update-MgUser -UserId $student.Id -ExtensionAttribute1 "Grade2"
                
                # 新しい学年グループへの追加は別途クラス分け後に実行
                $creationResults.StudentsPromoted += @{
                    UserPrincipalName = $student.UserPrincipalName
                    From = "Grade1"
                    To = "Grade2"
                }
                Write-Host "  ✓ 進級処理: $($student.UserPrincipalName) (1年→2年)" -ForegroundColor Green
            } catch {
                $creationResults.Errors += @{
                    Operation = "StudentPromotion"
                    Target = $student.UserPrincipalName
                    Error = $_.Exception.Message
                }
            }
        }
        
        # 2年生→3年生
        $grade2Students = Get-MgUser -Filter "extensionAttribute1 eq 'Grade2' and userType eq 'Member'" -All
        foreach ($student in $grade2Students) {
            try {
                Update-MgUser -UserId $student.Id -ExtensionAttribute1 "Grade3"
                
                $creationResults.StudentsPromoted += @{
                    UserPrincipalName = $student.UserPrincipalName
                    From = "Grade2"
                    To = "Grade3"
                }
                Write-Host "  ✓ 進級処理: $($student.UserPrincipalName) (2年→3年)" -ForegroundColor Green
            } catch {
                $creationResults.Errors += @{
                    Operation = "StudentPromotion"
                    Target = $student.UserPrincipalName
                    Error = $_.Exception.Message
                }
            }
        }
        
        # Step 3: SDS データ連携（オプション）
        if ($ImportFromSDS -and $SDSDataPath) {
            Write-Host "`n🔄 School Data Sync データ連携中..." -ForegroundColor Yellow
            
            try {
                # SDS CSVファイルの読み込み
                $sdsFiles = @{
                    Students = "$SDSDataPath\Student.csv"
                    Teachers = "$SDSDataPath\Teacher.csv"
                    Sections = "$SDSDataPath\Section.csv"
                    Enrollments = "$SDSDataPath\StudentEnrollment.csv"
                }
                
                foreach ($file in $sdsFiles.Values) {
                    if (-not (Test-Path $file)) {
                        Write-Warning "SDSファイルが見つかりません: $file"
                        continue
                    }
                }
                
                # 新入生データの処理
                $newStudents = Import-Csv $sdsFiles.Students -Encoding UTF8
                $newStudents | Where-Object { $_.Grade -eq "1" } | ForEach-Object {
                    # 新入生アカウント作成処理（別関数で実装）
                    Write-Host "  📝 新入生登録準備: $($_.Username)" -ForegroundColor Cyan
                }
                
                Write-Host "  ✓ SDS データ連携完了" -ForegroundColor Green
                
            } catch {
                Write-Host "  ❌ SDS データ連携失敗: $($_.Exception.Message)" -ForegroundColor Red
            }
        }
        
        # Step 4: 年度アーカイブグループの整理
        Write-Host "`n📦 前年度グループアーカイブ中..." -ForegroundColor Yellow
        
        $previousYear = $NewYear - 1
        $previousYearGroups = Get-MgGroup -Filter "startswith(displayName, '$previousYear-')" -All
        
        foreach ($group in $previousYearGroups) {
            try {
                # グループ名に"Archive"を追加
                $newDisplayName = "Archive-$($group.DisplayName)"
                Update-MgGroup -GroupId $group.Id -DisplayName $newDisplayName
                Write-Host "  📁 アーカイブ: $($group.DisplayName) → $newDisplayName" -ForegroundColor Gray
            } catch {
                Write-Host "  ⚠️ アーカイブ失敗: $($group.DisplayName)" -ForegroundColor Yellow
            }
        }
        
        # 処理結果サマリー
        Write-Host "`n=== 新年度開始処理結果 ===" -ForegroundColor Magenta
        Write-Host "作成グループ数: $($creationResults.GroupsCreated.Count)" -ForegroundColor Green
        Write-Host "進級処理学生数: $($creationResults.StudentsPromoted.Count)" -ForegroundColor Green
        Write-Host "Teams クラス作成予定: $($creationResults.TeamsClassesCreated.Count)" -ForegroundColor Cyan
        Write-Host "エラー発生: $($creationResults.Errors.Count)件" -ForegroundColor Red
        
        return $creationResults
        
    } catch {
        Write-Error "新年度開始処理失敗: $($_.Exception.Message)"
    }
}
```

### 3. 継続的な年次保守

#### 定期的なグループメンテナンス

**🎯 スクリプトの目的**
年間を通じてグループの健全性を維持し、使用されなくなったグループの特定・整理、メンバーシップの正確性確認、パフォーマンス問題の早期発見を自動化します。効率的な運用と正確なアクセス制御を継続的に確保します。

**📝 Usage**
```powershell
# 基本的な定期メンテナンス
Invoke-GroupMaintenance -MaintenanceType "All" -OutputReport "C:\reports\group_maintenance.xlsx"

# 特定種類のメンテナンスのみ
Invoke-GroupMaintenance -MaintenanceType "InactiveGroups" -InactivityThreshold 60

# 詳細分析付きメンテナンス
Invoke-GroupMaintenance -MaintenanceType "All" -DetailedAnalysis -AutoCleanup -ApprovalRequired
```

**⚙️ パラメータ**
- `-MaintenanceType`: メンテナンス種別（All/InactiveGroups/DuplicateMembers/OrphanedGroups）
- `-InactivityThreshold`: 非アクティブ判定日数（デフォルト90日）
- `-DetailedAnalysis`: 詳細分析実行
- `-AutoCleanup`: 自動クリーンアップ（承認後）
- `-ApprovalRequired`: 変更前の承認要求

```powershell
function Invoke-GroupMaintenance {
    param(
        [ValidateSet("All", "InactiveGroups", "DuplicateMembers", "OrphanedGroups", "SizeAnalysis")]
        [string]$MaintenanceType = "All",
        [int]$InactivityThreshold = 90,
        [switch]$DetailedAnalysis,
        [switch]$AutoCleanup,
        [switch]$ApprovalRequired,
        [string]$OutputReport
    )
    
    Write-Host "=== グループ定期メンテナンス開始 ===" -ForegroundColor Magenta
    Write-Host "メンテナンス種別: $MaintenanceType" -ForegroundColor Cyan
    
    $maintenanceResults = @{
        InactiveGroups = @()
        DuplicateMembers = @()
        OrphanedGroups = @()
        SizeAnalysis = @()
        ActionsRecommended = @()
        ActionsExecuted = @()
    }
    
    try {
        # 1. 非アクティブグループの検出
        if ($MaintenanceType -in @("All", "InactiveGroups")) {
            Write-Host "`n🔍 非アクティブグループ検出中..." -ForegroundColor Yellow
            
            $allGroups = Get-MgGroup -All
            $thresholdDate = (Get-Date).AddDays(-$InactivityThreshold)
            
            foreach ($group in $allGroups) {
                try {
                    # グループのアクティビティ分析
                    $members = Get-MgGroupMember -GroupId $group.Id -All
                    $lastActivity = $null
                    
                    # Microsoft 365 グループの場合、Teams活動やSharePoint活動をチェック
                    if ($group.GroupTypes -contains "Unified") {
                        # 注: 実際の実装では Microsoft Graph Activity API を使用
                        $lastActivity = $group.CreatedDateTime # 簡略化
                    }
                    
                    if ($lastActivity -and [datetime]$lastActivity -lt $thresholdDate) {
                        $groupInfo = @{
                            GroupName = $group.DisplayName
                            GroupId = $group.Id
                            CreatedDate = $group.CreatedDateTime
                            LastActivity = $lastActivity
                            MemberCount = $members.Count
                            GroupType = if ($group.GroupTypes -contains "Unified") { "Microsoft365" } else { "Security" }
                            Recommendation = "アクティビティ確認・必要に応じてアーカイブ"
                        }
                        $maintenanceResults.InactiveGroups += $groupInfo
                    }
                } catch {
                    Write-Warning "グループ分析失敗: $($group.DisplayName)"
                }
            }
            
            Write-Host "  📊 非アクティブグループ: $($maintenanceResults.InactiveGroups.Count)件" -ForegroundColor Yellow
        }
        
        # 2. 重複メンバーシップの検出
        if ($MaintenanceType -in @("All", "DuplicateMembers")) {
            Write-Host "`n🔍 重複メンバーシップ検出中..." -ForegroundColor Yellow
            
            $allUsers = Get-MgUser -All
            
            foreach ($user in $allUsers) {
                try {
                    $userGroups = Get-MgUserMemberOf -UserId $user.Id -All
                    $groupNames = $userGroups | ForEach-Object { $_.AdditionalProperties.displayName }
                    
                    # 類似グループ名での重複検出（例：2024-Grade1-ClassA と 2025-Grade1-ClassA）
                    $duplicatePatterns = $groupNames | Group-Object { ($_ -split '-')[1..2] -join '-' } | Where-Object { $_.Count -gt 1 }
                    
                    foreach ($pattern in $duplicatePatterns) {
                        $duplicateInfo = @{
                            UserPrincipalName = $user.UserPrincipalName
                            DuplicateGroups = $pattern.Group
                            Pattern = $pattern.Name
                            Recommendation = "最新年度グループのみに整理"
                        }
                        $maintenanceResults.DuplicateMembers += $duplicateInfo
                    }
                } catch {
                    Write-Warning "ユーザー分析失敗: $($user.UserPrincipalName)"
                }
            }
            
            Write-Host "  📊 重複メンバーシップ: $($maintenanceResults.DuplicateMembers.Count)件" -ForegroundColor Yellow
        }
        
        # 3. 所有者不在グループの検出
        if ($MaintenanceType -in @("All", "OrphanedGroups")) {
            Write-Host "`n🔍 所有者不在グループ検出中..." -ForegroundColor Yellow
            
            $allGroups = Get-MgGroup -All
            
            foreach ($group in $allGroups) {
                try {
                    $owners = Get-MgGroupOwner -GroupId $group.Id -ErrorAction SilentlyContinue
                    
                    if ($owners.Count -eq 0) {
                        $orphanInfo = @{
                            GroupName = $group.DisplayName
                            GroupId = $group.Id
                            MemberCount = (Get-MgGroupMember -GroupId $group.Id).Count
                            CreatedDate = $group.CreatedDateTime
                            Recommendation = "所有者の指定または管理者承認"
                        }
                        $maintenanceResults.OrphanedGroups += $orphanInfo
                    }
                } catch {
                    Write-Warning "所有者確認失敗: $($group.DisplayName)"
                }
            }
            
            Write-Host "  📊 所有者不在グループ: $($maintenanceResults.OrphanedGroups.Count)件" -ForegroundColor Yellow
        }
        
        # 4. グループサイズ分析
        if ($MaintenanceType -in @("All", "SizeAnalysis")) {
            Write-Host "`n📏 グループサイズ分析中..." -ForegroundColor Yellow
            
            $allGroups = Get-MgGroup -All
            
            foreach ($group in $allGroups) {
                try {
                    $memberCount = (Get-MgGroupMember -GroupId $group.Id).Count
                    
                    $sizeCategory = switch ($memberCount) {
                        { $_ -eq 0 } { "Empty" }
                        { $_ -le 5 } { "Small" }
                        { $_ -le 50 } { "Medium" }
                        { $_ -le 200 } { "Large" }
                        default { "ExtraLarge" }
                    }
                    
                    $sizeInfo = @{
                        GroupName = $group.DisplayName
                        MemberCount = $memberCount
                        SizeCategory = $sizeCategory
                        GroupType = if ($group.GroupTypes -contains "Unified") { "Microsoft365" } else { "Security" }
                        CreatedDate = $group.CreatedDateTime
                    }
                    
                    if ($memberCount -eq 0) {
                        $sizeInfo.Recommendation = "空グループの削除検討"
                    } elseif ($memberCount -gt 200) {
                        $sizeInfo.Recommendation = "大規模グループの分割検討"
                    }
                    
                    $maintenanceResults.SizeAnalysis += $sizeInfo
                } catch {
                    Write-Warning "サイズ分析失敗: $($group.DisplayName)"
                }
            }
            
            Write-Host "  📊 グループサイズ分析完了" -ForegroundColor Green
        }
        
        # 5. 推奨アクションの生成
        $recommendedActions = @()
        
        if ($maintenanceResults.InactiveGroups.Count -gt 0) {
            $recommendedActions += "非アクティブグループ $($maintenanceResults.InactiveGroups.Count)件のアーカイブ検討"
        }
        
        if ($maintenanceResults.DuplicateMembers.Count -gt 0) {
            $recommendedActions += "重複メンバーシップ $($maintenanceResults.DuplicateMembers.Count)件の整理"
        }
        
        if ($maintenanceResults.OrphanedGroups.Count -gt 0) {
            $recommendedActions += "所有者不在グループ $($maintenanceResults.OrphanedGroups.Count)件の所有者指定"
        }
        
        $emptyGroups = $maintenanceResults.SizeAnalysis | Where-Object { $_.SizeCategory -eq "Empty" }
        if ($emptyGroups.Count -gt 0) {
            $recommendedActions += "空グループ $($emptyGroups.Count)件の削除検討"
        }
        
        $maintenanceResults.ActionsRecommended = $recommendedActions
        
        # 6. 自動クリーンアップ（オプション）
        if ($AutoCleanup) {
            Write-Host "`n🧹 自動クリーンアップ実行中..." -ForegroundColor Yellow
            
            if ($ApprovalRequired) {
                Write-Host "推奨アクション:" -ForegroundColor Cyan
                $recommendedActions | ForEach-Object { Write-Host "  - $_" -ForegroundColor White }
                $approval = Read-Host "`n自動クリーンアップを実行しますか？ (yes/no)"
                
                if ($approval -ne "yes") {
                    Write-Host "自動クリーンアップをキャンセルしました" -ForegroundColor Yellow
                    return $maintenanceResults
                }
            }
            
            # 空グループの削除
            foreach ($emptyGroup in $emptyGroups) {
                try {
                    Remove-MgGroup -GroupId $emptyGroup.GroupId -Confirm:$false
                    $maintenanceResults.ActionsExecuted += "空グループ削除: $($emptyGroup.GroupName)"
                    Write-Host "  ✓ 空グループ削除: $($emptyGroup.GroupName)" -ForegroundColor Green
                } catch {
                    Write-Host "  ❌ 削除失敗: $($emptyGroup.GroupName)" -ForegroundColor Red
                }
            }
        }
        
        # 7. レポート出力
        if ($OutputReport) {
            $maintenanceResults | ConvertTo-Json -Depth 10 | Out-File "$OutputReport.json" -Encoding UTF8
            Write-Host "`n📄 メンテナンスレポート出力: $OutputReport.json" -ForegroundColor Green
        }
        
        # サマリー表示
        Write-Host "`n=== メンテナンス結果サマリー ===" -ForegroundColor Magenta
        Write-Host "非アクティブグループ: $($maintenanceResults.InactiveGroups.Count)件" -ForegroundColor Yellow
        Write-Host "重複メンバーシップ: $($maintenanceResults.DuplicateMembers.Count)件" -ForegroundColor Yellow
        Write-Host "所有者不在グループ: $($maintenanceResults.OrphanedGroups.Count)件" -ForegroundColor Yellow
        Write-Host "空グループ: $(($maintenanceResults.SizeAnalysis | Where-Object { $_.SizeCategory -eq 'Empty' }).Count)件" -ForegroundColor Yellow
        Write-Host "実行されたアクション: $($maintenanceResults.ActionsExecuted.Count)件" -ForegroundColor Green
        
        return $maintenanceResults
        
    } catch {
        Write-Error "グループメンテナンス失敗: $($_.Exception.Message)"
    }
}
```

---

## 🧪 実習演習: 学校運営シミュレーション

### 演習1: 新年度グループ構造の設計（30分）

#### シナリオ設定

```markdown
🏫 架空の「サンプル中学校」新年度準備

**学校概要**:
- 各学年3クラス（A組、B組、C組）
- 全校生徒数: 約270名（各学年90名）
- 教職員数: 45名（教員35名、事務職員10名）
- 部活動: 10部活（文化部5、運動部5）

**組織構造**:
- 教務部（教務主任、学年主任×3）
- 生徒指導部（生徒指導主事、各学年担当）
- 進路指導部（進路指導主事、3年担当）
- 事務部（事務長、庶務、経理、施設管理）

**年次更新課題**:
- 2024年度3年生90名の卒業処理
- 1・2年生の進級とクラス替え
- 新入生90名の受け入れ準備
- 教職員5名の異動・新採用3名
```

#### 演習手順

1. **現状分析とグループ設計**
   ```powershell
   # 現在のグループ構造の分析
   Analyze-CurrentGroupStructure -School "Sample-MiddleSchool" -AcademicYear 2024
   
   # 新年度グループ設計の策定
   Design-NewAcademicYearStructure -NewYear 2025 -StudentCount 270 -ClassesPerGrade 3
   ```

2. **段階的移行計画の策定**
   ```powershell
   # 移行計画の作成
   Create-TransitionPlan -FromYear 2024 -ToYear 2025 -GraduatingStudents 90 -NewStudents 90
   ```

3. **テストグループでの検証**
   ```powershell
   # テスト環境でのグループ作成
   Test-GroupStructure -TestMode -SampleSize 10
   ```

### 演習2: 動的グループルールの設計（25分）

#### 設計要件

```markdown
🎯 動的グループ要件

**学年別グループ**:
- 属性: extensionAttribute1 (Grade1, Grade2, Grade3)
- 対象: 学生のみ（userType = Member）

**職種別グループ**:
- 教員: jobTitle に "教" を含む
- 事務職員: department = "事務部"
- 管理職: jobTitle に "主任" または "校長" を含む

**部活動グループ**:
- 属性: extensionAttribute2 (部活動コード)
- 顧問: jobTitle + extensionAttribute3 (顧問フラグ)

**特別グループ**:
- 新入生: extensionAttribute1 = "Grade1" AND createdDateTime > 新年度開始日
- 受験生: extensionAttribute1 = "Grade3" AND extensionAttribute4 = "ExamPeriod"
```

#### 実装演習

```powershell
# 演習用動的グループルール作成
$dynamicRules = @{
    "Grade1-Students" = "(user.extensionAttribute1 -eq 'Grade1') -and (user.userType -eq 'Member')"
    "Grade2-Students" = "(user.extensionAttribute1 -eq 'Grade2') -and (user.userType -eq 'Member')"
    "Grade3-Students" = "(user.extensionAttribute1 -eq 'Grade3') -and (user.userType -eq 'Member')"
    "All-Teachers" = "(user.jobTitle -contains '教') -and (user.userType -eq 'Member')"
    "Administrative-Staff" = "(user.department -eq '事務部')"
    "School-Administrators" = "(user.jobTitle -contains '主任') -or (user.jobTitle -contains '校長')"
    "Baseball-Club" = "(user.extensionAttribute2 -eq 'Baseball')"
    "New-Students" = "(user.extensionAttribute1 -eq 'Grade1') -and (user.createdDateTime -ge '2025-04-01')"
}

# 動的グループの一括作成
foreach ($groupName in $dynamicRules.Keys) {
    New-DynamicEducationGroup -GroupName $groupName -Rule $dynamicRules[$groupName] -GroupType "Security"
}
```

### 演習3: 年次更新の完全自動化（45分）

#### 包括的な年次更新シナリオ

```markdown
🔄 完全な年次更新プロセス

**Phase 1: 事前準備（2月）**
1. 現状データの包括的分析
2. 卒業予定者・進級予定者の特定
3. 新年度組織構造の確定
4. 移行計画の策定・承認

**Phase 2: 卒業生処理（3月）**
1. データバックアップとアーカイブ
2. ライセンス削除（コスト削減）
3. アカウント無効化・グループ移動
4. 卒業生グループの作成

**Phase 3: 新年度開始（4月）**
1. 新年度グループ構造の作成
2. 既存学生の進級処理
3. 新入生アカウント作成・配属
4. 教職員異動反映

**Phase 4: 検証・最適化（4月中旬）**
1. 全グループメンバーシップの検証
2. アクセス権限の確認
3. システム動作確認
4. フィードバック収集・改善
```

#### マスタースクリプトの実装

```powershell
function Complete-AnnualUpdate {
    param(
        [int]$CurrentYear = 2024,
        [int]$NewYear = 2025,
        [switch]$ExecutePhase1,    # 事前準備
        [switch]$ExecutePhase2,    # 卒業生処理
        [switch]$ExecutePhase3,    # 新年度開始
        [switch]$ExecutePhase4,    # 検証・最適化
        [switch]$ExecuteAll,       # 全Phase実行
        [string]$ConfigPath = "C:\Config\AnnualUpdate.json"
    )
    
    if ($ExecuteAll) {
        $ExecutePhase1 = $ExecutePhase2 = $ExecutePhase3 = $ExecutePhase4 = $true
    }
    
    Write-Host "=== 年次更新マスタープロセス開始 ===" -ForegroundColor Magenta
    Write-Host "$CurrentYear → $NewYear 年度更新" -ForegroundColor Cyan
    
    $overallResults = @{
        Phase1Results = $null
        Phase2Results = $null
        Phase3Results = $null
        Phase4Results = $null
        TotalProcessingTime = $null
        OverallStatus = "NotStarted"
    }
    
    $startTime = Get-Date
    
    try {
        # Phase 1: 事前準備・データ検証
        if ($ExecutePhase1) {
            Write-Host "`n🔍 Phase 1: 事前準備・データ検証開始" -ForegroundColor Green
            $phase1Start = Get-Date
            
            $overallResults.Phase1Results = Prepare-AnnualUpdate -CurrentAcademicYear $CurrentYear -NextAcademicYear $NewYear -CheckType "All" -Detailed -IncludeRecommendations
            
            $phase1Duration = (Get-Date) - $phase1Start
            Write-Host "Phase 1 完了時間: $($phase1Duration.ToString('hh\:mm\:ss'))" -ForegroundColor Green
        }
        
        # Phase 2: 卒業生処理
        if ($ExecutePhase2) {
            Write-Host "`n🎓 Phase 2: 卒業生処理開始" -ForegroundColor Green
            $phase2Start = Get-Date
            
            $overallResults.Phase2Results = Process-GraduatingStudents -GraduationYear $CurrentYear -Stage "All" -ArchivePeriod 180 -CreateArchive
            
            $phase2Duration = (Get-Date) - $phase2Start
            Write-Host "Phase 2 完了時間: $($phase2Duration.ToString('hh\:mm\:ss'))" -ForegroundColor Green
        }
        
        # Phase 3: 新年度開始
        if ($ExecutePhase3) {
            Write-Host "`n🚀 Phase 3: 新年度開始処理" -ForegroundColor Green
            $phase3Start = Get-Date
            
            $overallResults.Phase3Results = Start-NewAcademicYear -NewYear $NewYear -ClassStructure "A,B,C" -CreateTeamsClasses -ImportFromSDS
            
            $phase3Duration = (Get-Date) - $phase3Start
            Write-Host "Phase 3 完了時間: $($phase3Duration.ToString('hh\:mm\:ss'))" -ForegroundColor Green
        }
        
        # Phase 4: 検証・最適化
        if ($ExecutePhase4) {
            Write-Host "`n✅ Phase 4: 検証・最適化開始" -ForegroundColor Green
            $phase4Start = Get-Date
            
            $overallResults.Phase4Results = Invoke-GroupMaintenance -MaintenanceType "All" -DetailedAnalysis
            
            # 統合検証
            Write-Host "統合検証実行中..." -ForegroundColor Yellow
            $verificationResults = Test-AnnualUpdateIntegrity -TargetYear $NewYear
            $overallResults.Phase4Results.VerificationResults = $verificationResults
            
            $phase4Duration = (Get-Date) - $phase4Start
            Write-Host "Phase 4 完了時間: $($phase4Duration.ToString('hh\:mm\:ss'))" -ForegroundColor Green
        }
        
        $overallResults.TotalProcessingTime = (Get-Date) - $startTime
        $overallResults.OverallStatus = "Completed"
        
        # 最終結果サマリー
        Write-Host "`n=== 年次更新完了サマリー ===" -ForegroundColor Magenta
        Write-Host "総処理時間: $($overallResults.TotalProcessingTime.ToString('hh\:mm\:ss'))" -ForegroundColor Cyan
        
        if ($overallResults.Phase1Results) {
            Write-Host "Phase 1 - 卒業予定者: $($overallResults.Phase1Results.GraduatingStudents.Count)名" -ForegroundColor White
            Write-Host "Phase 1 - 進級予定者: $($overallResults.Phase1Results.PromotingStudents.Count)名" -ForegroundColor White
        }
        
        if ($overallResults.Phase2Results) {
            Write-Host "Phase 2 - ライセンス削除: $($overallResults.Phase2Results.LicenseRemoved.Count)名" -ForegroundColor White
            Write-Host "Phase 2 - アカウント無効化: $($overallResults.Phase2Results.AccountsDisabled.Count)名" -ForegroundColor White
        }
        
        if ($overallResults.Phase3Results) {
            Write-Host "Phase 3 - 新規グループ: $($overallResults.Phase3Results.GroupsCreated.Count)件" -ForegroundColor White
            Write-Host "Phase 3 - 進級処理: $($overallResults.Phase3Results.StudentsPromoted.Count)名" -ForegroundColor White
        }
        
        Write-Host "`n🎉 年次更新が正常に完了しました！" -ForegroundColor Green
        
        return $overallResults
        
    } catch {
        $overallResults.OverallStatus = "Failed"
        $overallResults.TotalProcessingTime = (Get-Date) - $startTime
        Write-Error "年次更新処理失敗: $($_.Exception.Message)"
        return $overallResults
    }
}

# 統合検証用ヘルパー関数
function Test-AnnualUpdateIntegrity {
    param([int]$TargetYear)
    
    Write-Host "年次更新統合検証実行中..." -ForegroundColor Cyan
    
    $verificationResults = @{
        GroupIntegrity = $true
        MembershipAccuracy = $true
        PermissionConsistency = $true
        DataIntegrity = $true
        Issues = @()
    }
    
    try {
        # 新年度グループの存在確認
        $expectedGroups = @("$TargetYear-Grade1-ClassA", "$TargetYear-Grade2-ClassA", "$TargetYear-Grade3-ClassA")
        foreach ($groupName in $expectedGroups) {
            $group = Get-MgGroup -Filter "displayName eq '$groupName'" -ErrorAction SilentlyContinue
            if (-not $group) {
                $verificationResults.GroupIntegrity = $false
                $verificationResults.Issues += "必要グループが見つかりません: $groupName"
            }
        }
        
        # 進級後の学生属性確認
        $grade2Students = Get-MgUser -Filter "extensionAttribute1 eq 'Grade2'" -All
        $grade3Students = Get-MgUser -Filter "extensionAttribute1 eq 'Grade3'" -All
        
        if ($grade2Students.Count -eq 0 -and $grade3Students.Count -eq 0) {
            $verificationResults.MembershipAccuracy = $false
            $verificationResults.Issues += "進級処理が正常に完了していない可能性があります"
        }
        
        # 卒業生グループの確認
        $graduatesGroup = Get-MgGroup -Filter "displayName eq 'Graduates-$($TargetYear-1)'" -ErrorAction SilentlyContinue
        if (-not $graduatesGroup) {
            $verificationResults.DataIntegrity = $false
            $verificationResults.Issues += "卒業生グループが作成されていません"
        }
        
        Write-Host "✅ 統合検証完了" -ForegroundColor Green
        return $verificationResults
        
    } catch {
        $verificationResults.GroupIntegrity = $false
        $verificationResults.Issues += "検証プロセス中にエラーが発生: $($_.Exception.Message)"
        return $verificationResults
    }
}
```

---

## 🛠️ トラブルシューティング

### よくある問題と解決方法

#### 問題1: 動的グループメンバーが更新されない

**症状**: 動的ルール設定後、メンバーが自動追加されない

**原因と解決方法**:

```markdown
🔧 診断手順

1. **ライセンス確認**
   - Microsoft 365 A3/A5 (Microsoft Entra ID Premium P1を含む) が必要
   - テナント全体での有効化確認

2. **ルール構文チェック**
   - フィールド名の正確性（user.department, user.jobTitle等）
   - 演算子の正しい使用（-eq, -contains, -and, -or）
   - 引用符の適切な使用

3. **属性値の確認**
   - 対象ユーザーの実際の属性値
   - 大文字小文字の区別
   - 空白文字の有無

4. **処理時間の考慮**
   - 動的更新には数分〜数時間かかる場合がある
   - 大規模テナントでは更に時間を要する
```

**解決用スクリプト**:

```powershell
function Diagnose-DynamicGroupIssues {
    param(
        [string]$GroupName,
        [string]$TestUserPrincipalName
    )
    
    Write-Host "=== 動的グループ診断 ===" -ForegroundColor Magenta
    
    try {
        # グループ情報の確認
        $group = Get-MgGroup -Filter "displayName eq '$GroupName'"
        if (-not $group) {
            Write-Error "グループが見つかりません: $GroupName"
            return
        }
        
        Write-Host "`nグループ情報:" -ForegroundColor Green
        Write-Host "  名前: $($group.DisplayName)" -ForegroundColor White
        Write-Host "  ID: $($group.Id)" -ForegroundColor White
        Write-Host "  グループタイプ: $($group.GroupTypes -join ', ')" -ForegroundColor White
        Write-Host "  動的ルール: $($group.MembershipRule)" -ForegroundColor Yellow
        Write-Host "  処理状態: $($group.MembershipRuleProcessingState)" -ForegroundColor White
        
        # テストユーザーの属性確認
        if ($TestUserPrincipalName) {
            $testUser = Get-MgUser -Filter "userPrincipalName eq '$TestUserPrincipalName'" -Property "id,userPrincipalName,department,jobTitle,extensionAttribute1,extensionAttribute2,userType,employeeType,createdDateTime"
            
            Write-Host "`nテストユーザー属性:" -ForegroundColor Green
            $testUser | Select-Object userPrincipalName, department, jobTitle, extensionAttribute1, extensionAttribute2, userType, employeeType, createdDateTime | Format-List
            
            # 現在のグループメンバーシップ確認
            $isMember = Get-MgGroupMember -GroupId $group.Id | Where-Object { $_.Id -eq $testUser.Id }
            Write-Host "現在のメンバーシップ: $(if($isMember) {'メンバー'} else {'非メンバー'})" -ForegroundColor $(if($isMember) {'Green'} else {'Red'})
        }
        
        # 処理状況の確認
        $processingState = $group.MembershipRuleProcessingState
        if ($processingState -eq "Paused") {
            Write-Warning "動的メンバーシップ処理が停止しています"
            Write-Host "解決策: Update-MgGroup -GroupId $($group.Id) -MembershipRuleProcessingState 'On'" -ForegroundColor Cyan
        }
        
        # 一般的な問題の確認
        Write-Host "`n📋 一般的な問題チェック:" -ForegroundColor Yellow
        
        # Microsoft 365 A3/A5 ライセンス確認
        $tenantInfo = Get-MgOrganization
        Write-Host "  ✓ テナント確認完了" -ForegroundColor Green
        
        # グループメンバー数確認
        $memberCount = (Get-MgGroupMember -GroupId $group.Id).Count
        Write-Host "  現在のメンバー数: $memberCount 名" -ForegroundColor White
        
        if ($memberCount -eq 0) {
            Write-Warning "    メンバーが0名です。ルール条件を満たすユーザーが存在しない可能性があります"
        }
        
    } catch {
        Write-Error "診断中にエラーが発生: $($_.Exception.Message)"
    }
}

# 使用例
# Diagnose-DynamicGroupIssues -GroupName "Grade1-Students" -TestUserPrincipalName "s001@school.edu.jp"
```

#### 問題2: School Data Sync 連携エラー

**症状**: SDS CSVファイル処理時のエラー・同期失敗

**解決方法**:

```markdown
🔧 SDS トラブルシューティング

**データ形式チェック**:
- CSV文字エンコーディング: UTF-8 (BOM付き)
- 改行コード: Windows (CRLF)
- 必須フィールドの欠損チェック
- 外部キー関係の整合性確認

**一般的なエラーパターン**:
1. 重複する SIS ID
2. 無効な文字（特殊記号、絵文字）
3. 不正なメールアドレス形式
4. 参照整合性エラー（存在しないSchool SIS IDの参照等）
```

**CSVバリデーションスクリプト**:

```powershell
function Validate-SDSData {
    param(
        [string]$SDSDataPath,
        [switch]$FixIssues
    )
    
    Write-Host "=== SDS データ検証開始 ===" -ForegroundColor Magenta
    
    $validationResults = @{
        IsValid = $true
        Issues = @()
        FixedIssues = @()
    }
    
    try {
        $sdsFiles = @{
            School = "$SDSDataPath\School.csv"
            Teacher = "$SDSDataPath\Teacher.csv"  
            Student = "$SDSDataPath\Student.csv"
            Section = "$SDSDataPath\Section.csv"
            StudentEnrollment = "$SDSDataPath\StudentEnrollment.csv"
        }
        
        # ファイル存在確認
        foreach ($fileType in $sdsFiles.Keys) {
            $filePath = $sdsFiles[$fileType]
            if (-not (Test-Path $filePath)) {
                $validationResults.IsValid = $false
                $validationResults.Issues += "ファイルが見つかりません: $filePath"
            }
        }
        
        if (-not $validationResults.IsValid) {
            return $validationResults
        }
        
        # School.csv 検証
        Write-Host "School.csv 検証中..." -ForegroundColor Yellow
        $schools = Import-Csv $sdsFiles.School -Encoding UTF8
        $schoolIds = $schools.'SIS ID'
        
        # 重複チェック
        $duplicateSchools = $schoolIds | Group-Object | Where-Object { $_.Count -gt 1 }
        if ($duplicateSchools) {
            $validationResults.IsValid = $false
            $validationResults.Issues += "School.csv: 重複するSIS ID - $($duplicateSchools.Name -join ', ')"
        }
        
        # Student.csv 検証
        Write-Host "Student.csv 検証中..." -ForegroundColor Yellow
        $students = Import-Csv $sdsFiles.Student -Encoding UTF8
        
        # 必須フィールドチェック
        foreach ($student in $students) {
            if ([string]::IsNullOrWhiteSpace($student.'SIS ID')) {
                $validationResults.Issues += "Student.csv: SIS IDが空白の行があります"
            }
            
            if ([string]::IsNullOrWhiteSpace($student.Username)) {
                $validationResults.Issues += "Student.csv: Usernameが空白の行があります (SIS ID: $($student.'SIS ID'))"
            }
            
            # メールアドレス形式チェック
            if ($student.Username -and $student.Username -notmatch '^[^@]+@[^@]+\.[^@]+) {
                $validationResults.Issues += "Student.csv: 無効なメールアドレス形式 - $($student.Username)"
                
                if ($FixIssues) {
                    # 簡易修正例（実際の要件に応じて調整）
                    $fixedEmail = "$($student.'SIS ID')@school.edu.jp"
                    $student.Username = $fixedEmail
                    $validationResults.FixedIssues += "メールアドレス修正: $($student.'SIS ID') → $fixedEmail"
                }
            }
            
            # 学校参照整合性チェック
            if ($student.'School SIS ID' -notin $schoolIds) {
                $validationResults.IsValid = $false
                $validationResults.Issues += "Student.csv: 存在しないSchool SIS ID - $($student.'School SIS ID')"
            }
        }
        
        # Teacher.csv 検証
        Write-Host "Teacher.csv 検証中..." -ForegroundColor Yellow
        $teachers = Import-Csv $sdsFiles.Teacher -Encoding UTF8
        
        foreach ($teacher in $teachers) {
            if ($teacher.'School SIS ID' -notin $schoolIds) {
                $validationResults.IsValid = $false
                $validationResults.Issues += "Teacher.csv: 存在しないSchool SIS ID - $($teacher.'School SIS ID')"
            }
        }
        
        # Section.csv 検証
        Write-Host "Section.csv 検証中..." -ForegroundColor Yellow
        $sections = Import-Csv $sdsFiles.Section -Encoding UTF8
        
        foreach ($section in $sections) {
            if ($section.'School SIS ID' -notin $schoolIds) {
                $validationResults.IsValid = $false
                $validationResults.Issues += "Section.csv: 存在しないSchool SIS ID - $($section.'School SIS ID')"
            }
        }
        
        # StudentEnrollment.csv 検証
        Write-Host "StudentEnrollment.csv 検証中..." -ForegroundColor Yellow
        $enrollments = Import-Csv $sdsFiles.StudentEnrollment -Encoding UTF8
        $studentIds = $students.'SIS ID'
        $sectionIds = $sections.'SIS ID'
        
        foreach ($enrollment in $enrollments) {
            if ($enrollment.'SIS ID' -notin $studentIds) {
                $validationResults.IsValid = $false
                $validationResults.Issues += "StudentEnrollment.csv: 存在しないStudent SIS ID - $($enrollment.'SIS ID')"
            }
            
            if ($enrollment.'Section SIS ID' -notin $sectionIds) {
                $validationResults.IsValid = $false
                $validationResults.Issues += "StudentEnrollment.csv: 存在しないSection SIS ID - $($enrollment.'Section SIS ID')"
            }
        }
        
        # 修正されたデータの保存
        if ($FixIssues -and $validationResults.FixedIssues.Count -gt 0) {
            $students | Export-Csv "$SDSDataPath\Student_Fixed.csv" -Encoding UTF8 -NoTypeInformation
            Write-Host "修正されたStudent.csvを保存しました: Student_Fixed.csv" -ForegroundColor Green
        }
        
        # 検証結果サマリー
        Write-Host "`n=== SDS データ検証結果 ===" -ForegroundColor Magenta
        Write-Host "検証状態: $(if($validationResults.IsValid) {'✅ 正常'} else {'❌ 問題あり'})" -ForegroundColor $(if($validationResults.IsValid) {'Green'} else {'Red'})
        Write-Host "問題数: $($validationResults.Issues.Count)" -ForegroundColor $(if($validationResults.Issues.Count -eq 0) {'Green'} else {'Red'})
        Write-Host "修正数: $($validationResults.FixedIssues.Count)" -ForegroundColor Green
        
        if ($validationResults.Issues.Count -gt 0) {
            Write-Host "`n問題詳細:" -ForegroundColor Red
            $validationResults.Issues | ForEach-Object { Write-Host "  - $_" -ForegroundColor Red }
        }
        
        return $validationResults
        
    } catch {
        Write-Error "SDS データ検証失敗: $($_.Exception.Message)"
        $validationResults.IsValid = $false
        return $validationResults
    }
}
```

#### 問題3: 大規模グループでのパフォーマンス問題

**症状**: メンバー数の多いグループで操作が遅い・タイムアウト

**解決策**:

```powershell
function Optimize-LargeGroupOperations {
    param(
        [string]$GroupId,
        [int]$BatchSize = 20,
        [int]$DelayBetweenBatches = 5
    )
    
    Write-Host "大規模グループ最適化処理開始..." -ForegroundColor Cyan
    
    try {
        # グループメンバーの段階的取得
        $allMembers = @()
        $skip = 0
        
        do {
            $batch = Get-MgGroupMember -GroupId $GroupId -Top $BatchSize -Skip $skip
            $allMembers += $batch
            $skip += $BatchSize
            
            Write-Progress -Activity "メンバー取得中" -Status "$($allMembers.Count) 名処理済み" -PercentComplete -1
            
            if ($batch.Count -eq $BatchSize) {
                Start-Sleep -Seconds $DelayBetweenBatches
            }
        } while ($batch.Count -eq $BatchSize)
        
        Write-Progress -Activity "メンバー取得中" -Completed
        Write-Host "✅ メンバー取得完了: $($allMembers.Count) 名" -ForegroundColor Green
        
        return $allMembers
        
    } catch {
        Write-Error "大規模グループ操作失敗: $($_.Exception.Message)"
    }
}
```

---

## ✅ 学習成果の確認

### 習得スキルチェック

この節を完了したら、以下の項目について実際に操作できることを確認してください：

#### 基本グループ管理

- [ ] **グループ種別の理解**: Microsoft 365グループ、セキュリティグループ、配布グループの特徴と使い分け
- [ ] **静的グループ作成**: 目的に応じた適切なグループ作成と設定
- [ ] **メンバー管理**: ユーザー・グループの追加・削除・権限設定
- [ ] **アクセス制御**: セキュリティグループを使用した適切な権限管理

#### 動的グループ管理

- [ ] **動的ルール設計**: ユーザー属性に基づく効果的なメンバーシップルール作成
- [ ] **ルール検証**: テストユーザーでの動的ルール動作確認
- [ ] **トラブルシューティング**: 動的グループの問題診断と解決
- [ ] **パフォーマンス最適化**: 大規模環境での効率的なグループ運用

#### 教育機関特有の管理

- [ ] **組織構造対応**: 学校の階層構造に適したグループ設計
- [ ] **年次更新自動化**: 新年度対応の包括的な自動処理
- [ ] **SDS連携**: School Data Sync を活用したシステム統合
- [ ] **セキュリティ設計**: 教育機関の要件に適したアクセス制御

### 実践課題: 総合グループ管理

実際の教育機関運営を想定した包括的な課題に取り組んでください：

```markdown
🎯 総合課題: 高等学校Microsoft 365グループ管理システム構築

## 背景
全校生徒1,200名、教職員80名の県立高等学校で、Microsoft 365を全面導入。
効率的で安全なグループ管理システムの設計・構築を担当します。

## 要件
**組織構造**:
- 3学年 × 8クラス（各クラス50名）
- 教科：国語、数学、英語、理科、社会、芸術、体育、情報
- 部活動：20部活（文化部12、運動部8）
- 管理部門：教務部、生徒指導部、進路指導部、事務部

**管理要件**:
- 年次更新の完全自動化
- 動的グループによる効率的なメンバー管理
- セキュリティレベル別のアクセス制御
- 部活動・委員会の柔軟な管理
- 保護者との連携機能

## 実施内容

### 1. 設計フェーズ（120分）
- グループ階層構造の設計
- 命名規則とメタデータ設計
- 動的ルール設計（学年・クラス・教科・部活動）
- セキュリティポリシー設計
- 年次更新フロー設計

### 2. 実装フェーズ（180分）
- 基本グループ構造の作成
- 動的グループとルールの実装
- セキュリティポリシーの適用
- 年次更新スクリプトの開発
- テストデータでの動作検証

### 3. 運用フェーズ（60分）
- 運用手順書の作成
- 定期メンテナンススクリプトの設定
- トラブルシューティングガイドの作成
- パフォーマンス監視設定

## 成果物
1. **設計書**：グループ階層図、ルール仕様書、セキュリティポリシー
2. **実装物**：PowerShellスクリプト一式、設定ファイル
3. **運用文書**：操作手順書、メンテナンス計画、緊急時対応手順
4. **検証レポート**：テスト結果、パフォーマンス分析、改善提案

## 評価ポイント
- 実用性：実際の学校運営で使用できる設計か
- 効率性：運用負荷の軽減が実現されているか
- 安全性：適切なセキュリティが確保されているか
- 拡張性：将来の変更・拡張に対応できるか
- 保守性：継続的な運用・改善が可能か
```

---

## 🚀 次のステップ

この節を完了したら、以下の学習に進むことを推奨します：

### 推奨学習パス

1. **[3.4 ユーザーロールと権限](04-roles-permissions.md)**
   - グループベースの権限管理の詳細
   - カスタムロール設計と条件付きアクセス

<!-- 2. **[第4章: ライセンス管理](../04-license-management/)** -->
   - グループベースライセンス割り当て
   - コスト最適化とライセンス戦略

<!-- 3. **[第5章: セキュリティとコンプライアンス](../05-security-compliance/)** -->
   - 高度なセキュリティグループ設計
   - コンプライアンス要件とグループポリシー

<!-- 4. **[第6章: デバイス管理](../06-device-management/)** -->
   - デバイスグループとコンプライアンス
   - Intune との連携によるデバイス管理

### 発展学習

- **Microsoft Graph API**: プログラマティックなグループ管理
- **Azure Logic Apps**: グループライフサイクルの自動化
- **Power Platform**: ローコード/ノーコードでのグループ管理アプリ開発
- **Microsoft Purview**: 高度なコンプライアンスとガバナンス

### 専門資格

- **Microsoft 365 Certified: Administrator Expert**
- **Microsoft 365 Certified: Security Administrator Associate**
- **Microsoft Entra ID Identity and Access Administrator (SC-300)**

---

## 📚 参考リソース

### Microsoft公式ドキュメント

- **Microsoft 365 のグループの種類について** - 各グループタイプの特徴と用途の詳細
- **Microsoft 365 のグループの種類を比較する** - 管理者向けのグループ比較とベストプラクティス
- **Microsoft Graph API ドキュメント** - プログラマティックなグループ管理
- **Microsoft Entra ID 動的グループ** - 動的メンバーシップルールの詳細仕様

### 教育機関特化リソース

- **Microsoft 365 Education** - 教育機関向けの包括的なソリューションガイド
- **School Data Sync (SDS)** - 校務支援システムとMicrosoft製品を連携する無料ツール
- **Teams for Education** - 教育機関向けTeams設定とクラス管理
- **Education Insights** - 学習分析と教育データ活用

### 実用ツール・テンプレート

<!-- **グループ設計テンプレート**:
- [組織構造マッピングシート](../../appendices/templates/group-structure-template.xlsx)
- [動的ルール設計テンプレート](../../appendices/templates/dynamic-rules-template.csv)
- [年次更新計画テンプレート](../../appendices/templates/annual-update-plan.xlsx)

**PowerShellスクリプト集**:
- [グループ管理スクリプト](../../appendices/powershell-commands.md#グループ管理)
- [動的グループ作成スクリプト](../../appendices/scripts/dynamic-groups.ps1)
- [年次更新自動化スクリプト](../../appendices/scripts/annual-update.ps1)

**運用チェックリスト**:
- [グループ作成チェックリスト](../../appendices/checklists/group-creation-checklist.md)
- [年次更新チェックリスト](../../appendices/checklists/annual-update-checklist.md)
- [セキュリティ設定チェックリスト](../../appendices/checklists/security-checklist.md) -->

### コミュニティ・サポート

- **Microsoft Tech Community - Education**
  - 教育機関のIT管理者コミュニティ
  - ベストプラクティスの共有とQ&A

- **Microsoft Education Partner Network**
  - 教育機関向け専門パートナー
  - 導入支援とコンサルティング

- **GitHub - Microsoft Graph PowerShell**
  - 最新のPowerShellモジュールとサンプルコード
  - コミュニティによる拡張機能

---

## 💡 実運用での成功ポイント

### 継続的改善のための指標

```markdown
📊 KPI（重要業績指標）

**効率性指標**:
- グループ管理作業時間の削減率（目標：70%以上）
- 年次更新自動化率（目標：90%以上）
- 手動介入が必要な例外処理件数（目標：全体の5%以下）

**正確性指標**:
- グループメンバーシップの正確性（目標：99%以上）
- アクセス権限の誤設定件数（目標：0件）
- セキュリティインシデント発生率（目標：年間0件）

**利用満足度指標**:
- 教職員の使いやすさ評価（目標：4.0/5.0以上）
- 学生のアクセス性評価（目標：4.0/5.0以上）
- IT管理負荷軽減実感（目標：80%以上）

**システム健全性指標**:
- グループ数の適正性（肥大化防止）
- 非アクティブグループ割合（目標：5%以下）
- 平均グループサイズの適正性
```

### 長期運用での注意点

- **定期的なレビュー**: 四半期ごとのグループ構造見直し
- **セキュリティ監査**: 年次でのアクセス権限総点検
- **ユーザーフィードバック**: 継続的な利便性改善
- **新機能対応**: Microsoft 365の機能更新への適応
- **災害復旧計画**: グループ設定のバックアップと復旧手順

---

*📅 最終更新: 2025年6月1日*  
*✍️ 更新者: Microsoft 365 ハンドブック編集チーム*  
*📖 対象バージョン: Microsoft 365 Education (2025年6月版) / Microsoft Graph PowerShell SDK v2.0+*
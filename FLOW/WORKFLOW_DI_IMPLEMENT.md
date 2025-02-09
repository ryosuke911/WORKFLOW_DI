# 依存性注入実装計画ワークフロー

## 1. 入力データ形式

評価フェーズからの入力を受け取り、実装計画を立案するための基礎情報として使用します。

### 1.1 評価結果の受け取り

評価フェーズから以下の形式で情報を受け取ります：

```yaml
implementation_input:
  component_id: string
  selected_patterns:
    primary:
      pattern_id: int
      name: string
      reason: string
      constraints: string[]
    secondary:
      - pattern_id: int
        name: string
        reason: string
        constraints: string[]
  
  pattern_constraints:
    dependencies: 
      - dependent: int
        requires: int[]
        reason: string
    conflicts:
      - patterns: int[]
        reason: string
  
  layer_constraints:
    type: string
    required_patterns: int[]
    avoid_patterns: int[]
```

#### 入力データの検証ポイント

1. **パターン選択の妥当性**
   - プライマリパターンとセカンダリパターンの組み合わせが適切か
   - パターン間の依存関係や制約が考慮されているか

2. **レイヤー制約の確認**
   - 選択されたパターンがレイヤー制約に違反していないか
   - 必須パターンが含まれているか

3. **制約条件の整合性**
   - パターン間の依存関係が循環していないか
   - コンフリクトするパターンの組み合わせがないか

## 2. 出力形式

### 2.1 実装計画書

実装計画は以下の要素を含む包括的なドキュメントとして出力します：

1. **概要**
   - 対象コンポーネント
   - 全体フェーズ数
   - 想定期間
   - 主要リスク

2. **フェーズ定義**
   - 各フェーズの目的
   - 具体的なタスク
   - 品質ゲート
   - ロールバック計画

3. **品質基準**
   - テストカバレッジ要件
   - 依存性メトリクス目標
   - コード品質基準

```yaml
implementation_plan:
  overview:
    component_id: string
    total_phases: int
    estimated_duration: string
    key_risks: string[]
  
  phases:
    - phase_id: int
      name: string
      patterns: int[]
      tasks: 
        - task_id: string
          name: string
          steps: string[]
          dependencies: string[]
          estimated_effort: string
      quality_gates:
        - metric: string
          threshold: number
      rollback_plan:
        steps: string[]
        verification: string[]
  
  quality_criteria:
    test_coverage:
      unit_test: number
      integration_test: number
    dependency_metrics:
      coupling_threshold: number
      circular_dependencies: boolean
  
  risk_management:
    key_risks: string[]
    mitigation_strategies: string[]
    rollback_triggers: string[]
```

### 2.2 実装ガイドライン

各パターンの実装に関する具体的なガイドラインを提供します：

1. **パターン別ガイド**
   - 実装手順
   - ベストプラクティス
   - 一般的な問題点

2. **テスト戦略**
   - テストタイプ
   - カバレッジ要件
   - モック戦略

3. **レビューチェックリスト**
   - レビューポイント
   - 受け入れ基準

```yaml
implementation_guidelines:
  patterns:
    - pattern_id: int
      name: string
      implementation_steps: string[]
      best_practices: string[]
      common_pitfalls: string[]
  
  testing_strategy:
    - phase_id: int
      test_types: string[]
      coverage_requirements: number
      mock_strategies: string[]
  
  review_checklist:
    - phase_id: int
      review_points: string[]
      acceptance_criteria: string[]
```

## 3. 実装計画の生成

実装計画は、影響範囲分析、フェーズ定義、品質基準の3つの主要要素から構成されます。

### 3.1 影響範囲分析

コードベースへの影響を以下の観点で分析します：

1. **直接的な影響**
   - コード変更が必要なファイル
   - インターフェース定義の追加・変更
   - テストコードへの影響

2. **間接的な影響**
   - 依存コンポーネントへの影響
   - 使用箇所への影響

3. **テストへの影響**
   - 修正が必要なテスト
   - 新規作成が必要なテスト
   - モックオブジェクトの要件

```yaml
impact_analysis:
  direct_impacts:
    - component: string
      type: "CODE" | "TEST" | "INTERFACE"
      risk_level: "HIGH" | "MEDIUM" | "LOW"
      affected_files: string[]
  
  indirect_impacts:
    - component: string
      type: "DEPENDENCY" | "USAGE"
      risk_level: "HIGH" | "MEDIUM" | "LOW"
      affected_components: string[]
  
  test_impacts:
    - test_type: "UNIT" | "INTEGRATION"
      affected_tests: string[]
      required_mocks: string[]
```

### 3.2 実装フェーズの定義

実装は以下の3つのフェーズに分けて計画します：

1. **基盤整備フェーズ**
   - グローバル状態の整理
   - 基本的な依存関係の整理
   - 初期テストの整備

2. **抽象化導入フェーズ**
   - インターフェースの抽出
   - 依存関係の抽象化
   - テスト容易性の向上

3. **テスト容易性向上フェーズ**
   - モックオブジェクトの導入
   - テストケースの拡充
   - カバレッジの向上

```yaml
implementation_phases:
  - phase_id: 1
    name: "基盤整備フェーズ"
    patterns: int[]
    tasks:
      - task_id: string
        name: string
        steps: string[]
        dependencies: string[]
        estimated_effort: string
    quality_gates:
      - metric: string
        threshold: number
    rollback_plan:
      steps: string[]
      verification: string[]

  - phase_id: 2
    name: "抽象化導入フェーズ"
    patterns: int[]
    tasks: []
    quality_gates: []
    rollback_plan: {}

  - phase_id: 3
    name: "テスト容易性向上フェーズ"
    patterns: int[]
    tasks: []
    quality_gates: []
    rollback_plan: {}
```

### 3.3 品質基準の定義

各フェーズで達成すべき品質基準を定義します：

1. **テストカバレッジ**
   - ユニットテストカバレッジ
   - 統合テストカバレッジ

2. **依存性メトリクス**
   - 結合度の閾値
   - 循環依存の有無

3. **コード品質**
   - 複雑度の閾値
   - コードの重複率

```yaml
quality_criteria:
  test_coverage:
    unit_test: number
    integration_test: number
  
  dependency_metrics:
    coupling_threshold: number
    circular_dependencies: boolean
  
  code_quality:
    complexity_threshold: number
    duplication_threshold: number
```

## 4. 検証フェーズ

### 4.1 計画の整合性チェック

実装計画の整合性を以下の観点でチェックします：

1. **フェーズシーケンス**
   - フェーズの順序が適切か
   - 並行タスクの制約

2. **リソース制約**
   - フェーズごとの工数
   - テスト時間の確保

3. **品質ゲート**
   - 必須メトリクス
   - 最低閾値

```yaml
validation_rules:
  phase_sequence:
    - required_order: int[]
    - max_parallel_tasks: int
  
  resource_constraints:
    - max_effort_per_phase: string
    - min_testing_time: string
  
  quality_gates:
    - required_metrics: string[]
    - minimum_thresholds: number[]
```

### 4.2 実現可能性の評価

計画の実現可能性を以下の観点で評価します：

1. **技術的実現可能性**
   - 技術スタックの適合性
   - チームのスキルセット
   - 技術的な制約

2. **リソース的実現可能性**
   - 工数の妥当性
   - スケジュールの現実性
   - チームの対応能力

```yaml
feasibility_assessment:
  technical_feasibility:
    - aspect: string
      score: number
      concerns: string[]
  
  resource_feasibility:
    - aspect: string
      score: number
      constraints: string[]
```

## 5. リスク管理計画

### 5.1 リスク評価

以下の観点でリスクを評価し、対策を計画します：

1. **技術的リスク**
   - パターン適用の複雑さ
   - 既存機能への影響
   - パフォーマンスへの影響

2. **運用リスク**
   - チームの学習曲線
   - デプロイメントの影響
   - 移行期間中の運用

```yaml
risk_assessment:
  technical_risks:
    - risk_id: string
      description: string
      probability: "HIGH" | "MEDIUM" | "LOW"
      impact: "HIGH" | "MEDIUM" | "LOW"
      mitigation: string[]
  
  operational_risks:
    - risk_id: string
      description: string
      probability: "HIGH" | "MEDIUM" | "LOW"
      impact: "HIGH" | "MEDIUM" | "LOW"
      mitigation: string[]
```

### 5.2 ロールバック計画

各フェーズでの問題発生時に備えたロールバック計画を策定します：

1. **ロールバックトリガー**
   - 品質基準の未達
   - 重大な機能障害
   - パフォーマンス低下

2. **ロールバック手順**
   - コード変更の取り消し
   - テストの復元
   - 設定の巻き戻し

```yaml
rollback_plan:
  triggers:
    - condition: string
      metrics: string[]
      thresholds: number[]
  
  procedures:
    - phase_id: int
      steps: string[]
      verification: string[]
``` 
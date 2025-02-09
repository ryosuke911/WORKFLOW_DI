# 依存性注入実装ワークフロー

## 1. 入力データ形式

実装計画フェーズ（WORKFLOW_IMPLEMENT_DI.md）からの入力を受け取り、実際のコード実装を行うための情報として使用します。

### 1.1 実装計画の受け取り

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

#### 入力データの検証ポイント

1. **実装計画の妥当性**
   - フェーズ定義が明確か
   - タスクの依存関係が適切か
   - 品質基準が具体的か

2. **ガイドラインの完全性**
   - 各パターンの実装手順が具体的か
   - テスト戦略が明確か
   - レビュー基準が具体的か

## 2. 出力形式

### 2.1 コード変更出力
```yaml
code_changes:
  component_id: string
  changes:
    - pattern_id: int
      name: string
      files:
        - path: string
          changes:
            - type: "ADD" | "MODIFY" | "DELETE"
              location:
                start_line: int
                end_line: int
              code: string
              reason: string
```

### 2.2 実装レポート出力
```yaml
implementation_report:
  component_id: string
  summary:
    start_time: datetime
    end_time: datetime
    total_changes: int
    affected_files: int
    
  pattern_results:
    - pattern_id: int
      name: string
      status: "SUCCESS" | "PARTIAL" | "FAILED"
      changes_made:
        - file: string
          type: string
          description: string
      test_results:
        coverage: float
        passing: int
        failing: int
      
  quality_metrics:
    coupling_score:
      before: float
      after: float
    cohesion_score:
      before: float
      after: float
    test_coverage:
      before: float
      after: float
      
  issues:
    - severity: "HIGH" | "MEDIUM" | "LOW"
      description: string
      mitigation: string
```

## 3. 実装プロセス

### 3.1 入力データの利用

1. **パターン情報の解釈**
   - `implementation_guidelines.patterns`から実装手順を特定
     - パターンIDと名前に基づいて適用手順を決定
     - `best_practices`と`common_pitfalls`を確認
   
   - `implementation_plan.phases`から実装順序を特定
     - フェーズごとのパターン適用順序を決定
     - タスクの依存関係（`dependencies`）に基づいて実装順序を決定

2. **品質基準の確認**
   - `quality_criteria.test_coverage`に基づくテスト要件の確認
   - `quality_criteria.dependency_metrics`に基づく結合度の目標値確認
   - `quality_gates`に基づく各フェーズでの品質チェックポイントの特定

### 3.2 実装準備

1. **対象コードの解析**
   - `component_id`で指定されたファイルの構文解析を実行
   - メソッドの依存関係を特定
   - 現在の結合度と凝集度を測定

2. **実装計画の確認**
   - `phases[].tasks`に基づいて具体的な実装タスクを特定
   - `estimated_effort`を考慮した実装スケジュールの確認
   - `testing_strategy`に基づくテスト計画の立案

### 3.3 パターン適用手順

各パターンの実装は`implementation_guidelines.patterns[].implementation_steps`に従って実行します：

#### インターフェース抽出パターン（Pattern ID: 9）の場合

1. **インターフェース定義**
   - 対象クラスの公開メソッドを抽出
   - インターフェースファイルを新規作成
   - メソッドシグネチャを定義

2. **実装クラスの修正**
   - インターフェースの実装を追加
   - 必要に応じてメソッドの可視性を調整
   - 依存するクラスの参照方法を修正

3. **依存性注入の導入**
   - コンストラクタパラメータの追加
   - DIコンテナの設定更新
   - 既存の参照箇所の修正

### 3.4 テスト実装

`testing_strategy`に基づいて以下を実施：

1. **単体テストの作成**
   - `mock_strategies`に従ったモックオブジェクトの定義
   - テストケースの実装
   - `coverage_requirements`に基づくテスト範囲の確保

2. **結合テストの更新**
   - 既存テストの修正
   - 新規テストケースの追加
   - テストカバレッジの確認

### 3.5 品質検証

`review_checklist`に基づいて以下を実施：

1. **コードレビュー項目**
   - `review_points`に基づく実装の確認
   - `acceptance_criteria`との照合
   - 既存機能への影響確認

2. **メトリクス評価**
   - `quality_criteria`との比較
     - 結合度の変化
     - 凝集度の変化
     - テストカバレッジ

## 4. 出力生成

### 4.1 コード変更の記録
[コード変更出力のYAML形式は現状と同じ]

### 4.2 実装レポートの生成
[実装レポート出力のYAML形式は現状と同じ]

## 5. エラーハンドリング

### 5.1 想定されるエラー

1. **コンパイルエラー**
   - 原因の特定
   - ロールバック手順の実行
   - レポートへの記録

2. **テスト失敗**
   - 失敗したテストの分析
   - 修正または代替案の検討
   - 部分的成功の記録

[エラーハンドリングのYAML形式は現状と同じ] 
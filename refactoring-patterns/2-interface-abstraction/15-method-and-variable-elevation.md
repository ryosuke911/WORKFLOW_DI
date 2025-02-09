## 15. メソッドと変数の引き上げ

### 概要

「メソッドと変数の引き上げ」とは、複数のサブクラスで共通して使用されるメソッドや変数を抽出し、スーパークラスに移動させるリファクタリング手法です。この手法により、コードの重複を排除し、再利用性と可読性を向上させることができます。

### 使用すべき場面

- サブクラス間で同一のロジックを持つメソッドや変数が存在する場合。
- コードの重複を減らし、保守性を向上させたい場合。
- 共通のロジックをスーパークラスに集約し、サブクラスの責務を明確化したい場合。

### Javaコード例

以下のコードを例に、具体的な適用手順を説明します。

### 変更前のコード

```java
public class Scheduler {
    private List<ScheduleItem> items;

    public int getDeadTime() {
        int result = 0;
        for (ScheduleItem item : items) {
            if (!item.isTransient()) {
                result += item.getDuration();
            }
        }
        return result;
    }
}

```

### 変更後のコード

スーパークラス `SchedulingServices` に共通のメソッド `getDeadTime` を引き上げる。

```java
public abstract class SchedulingServices {
    protected List<ScheduleItem> items;

    public int getDeadTime() {
        int result = 0;
        for (ScheduleItem item : items) {
            if (!item.isTransient()) {
                result += item.getDuration();
            }
        }
        return result;
    }
}

public class Scheduler extends SchedulingServices {
    // Scheduler固有のロジックを記述
}

```

### 手順

1. **引き上げたいメソッドを特定する**
    - サブクラス間で共通して使用されるロジックを含むメソッドを選定。
2. **スーパークラスを作成**
    - 該当するサブクラスの共通スーパークラスがない場合は、新たに作成する。
3. **メソッドをスーパークラスに移動**
    - 共通ロジックをスーパークラスに移動し、サブクラスからは削除する。
4. **メソッド内の依存関係を調整**
    - メソッドが依存する変数や他のメソッドをスーパークラスに移動させる。
5. **テスト**
    - 移動後のスーパークラスとサブクラスの機能が正しく動作するか確認する。

### 利点

- コードの重複を削減し、保守性を向上。
- スーパークラスに共通のロジックを集約することで、サブクラスの責務が明確化。
- 再利用性の高い設計を実現。

### 注意点

- スーパークラスに責務が集中しすぎると、オーバーヘッドが発生する可能性があるため、適切な設計バランスが必要です。
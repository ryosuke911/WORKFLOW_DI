## 16. 依存関係の押し出し

### **概要**

「依存関係の押し出し（Push Down Dependency）」は、あるクラスが持つ依存関係の一部が、他のクラスやテスト環境で扱いづらい場合に、その依存関係を新しいサブクラスに押し下げることで、元のクラスをシンプルにし、テスト可能性や拡張性を向上させるリファクタリング手法です。

- **目的**:
    - クラス内の依存関係を明確化し、テスト可能な設計にする。
    - UIやデータベースのような依存度の高い要素を分離することで、テストやメンテナンスの負担を軽減する。
    - 再利用性の高いコードを作る。

---

### **使用すべき場面**

1. **テストの困難さ**:
    - クラスが特定の依存関係（例: UIや外部サービス）に強く依存しており、モックやスタブでの代替が難しい場合。
2. **クラスの責務が不明確**:
    - 単一責務の原則（Single Responsibility Principle）を満たしていないクラスがある場合。
3. **コードの再利用性向上**:
    - 共通機能と依存関係を分離することで、他のプロジェクトやモジュールでの再利用を目指す場合。

---

### **具体的な例**

### **Before: 元のクラス**

```java
public class TradeValidator {
    private Trade trade;
    private boolean flag;

    public TradeValidator(Trade trade) {
        this.trade = trade;
        this.flag = false;
    }

    public boolean isValid() {
        if (inRange(trade.getDate()) &&
            validDestination(trade.getDestination()) &&
            inHours(trade)) {
            flag = true;
        }
        showMessage();
        return flag;
    }

    private void showMessage() {
        // UIメッセージ表示処理
        System.out.println("This is a valid trade.");
    }

    private boolean inRange(Date date) {
        // 日付の範囲チェック
        return true;
    }

    private boolean validDestination(String destination) {
        // 目的地のチェック
        return true;
    }

    private boolean inHours(Trade trade) {
        // 営業時間のチェック
        return true;
    }
}

```

### **問題点**

- `showMessage` メソッドがUI操作に依存しているため、ユニットテストが難しい。
- `isValid` メソッドのロジックをテストする際に、UIメッセージが干渉する。

---

### **After: 依存関係の押し出し後**

```java
public abstract class TradeValidator {
    protected Trade trade;
    protected boolean flag;

    public TradeValidator(Trade trade) {
        this.trade = trade;
        this.flag = false;
    }

    public boolean isValid() {
        if (inRange(trade.getDate()) &&
            validDestination(trade.getDestination()) &&
            inHours(trade)) {
            flag = true;
        }
        showMessage();
        return flag;
    }

    protected abstract void showMessage();

    protected boolean inRange(Date date) {
        return true;
    }

    protected boolean validDestination(String destination) {
        return true;
    }

    protected boolean inHours(Trade trade) {
        return true;
    }
}

public class WindowsTradeValidator extends TradeValidator {
    public WindowsTradeValidator(Trade trade) {
        super(trade);
    }

    @Override
    protected void showMessage() {
        // Windows環境用のメッセージ表示処理
        System.out.println("Windows Trade Message");
    }
}

public class TestTradeValidator extends TradeValidator {
    public TestTradeValidator(Trade trade) {
        super(trade);
    }

    @Override
    protected void showMessage() {
        // テスト用のメッセージ処理（空実装）
    }
}

```

---

### **メリット**

1. **テストの容易化**:
    - `TestTradeValidator` を使うことで、`showMessage` メソッドを無効化し、純粋にロジックのみをテスト可能。
2. **責務の分離**:
    - UIに関する依存関係をサブクラスに押し下げ、元のクラスの責務を明確化。
3. **拡張性の向上**:
    - 他のプラットフォームやテストシナリオに応じたサブクラスを容易に追加可能。

---

### **手順**

1. 依存関係を含むメソッドや変数を特定。
2. 新しい抽象クラスを作成し、依存関係を表すメソッドを `abstract` に設定。
3. 元のクラスを抽象クラスに変更し、共通ロジックを抽象クラスに移動。
4. サブクラスを作成し、プラットフォームやテストケースに応じた依存処理を実装。
5. 新しい構造が期待通りに動作することをテストで確認。
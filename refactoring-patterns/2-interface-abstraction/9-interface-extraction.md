## 9. インタフェースの抽出

### **概要**

「インタフェースの抽出」とは、既存のクラスから特定の機能群を分離し、新しいインタフェースとして定義するリファクタリング手法です。この手法を使用することで、特定の実装に依存せず、柔軟性と再利用性を向上させる設計が可能になります。また、テストやモックの作成が容易になり、システムのメンテナンス性が向上します。

---

### **使用すべき場面**

以下のような状況で特に有効です：

1. **テストの効率化**
    - テスト環境で使いたい特定の機能を簡単にモック化したい場合。
    - 例えば、データベースやログ記録のような重い処理を持つクラスをテスト用の軽量な実装で置き換えたいとき。
2. **依存関係の削減**
    - あるクラスが別のクラスに強く依存している場合、その依存を軽減するためにインタフェースを抽出。
    - これにより、依存先を柔軟に変更できるようになります。
3. **コードの再利用性向上**
    - 共通のインタフェースを持つ異なるクラスに対して同じ処理を適用したい場合。
4. **将来的な拡張性**
    - 新しい実装を追加したいときに、既存のコードを変更せずに済む設計を構築したい場合。

---

---

### **Javaでの具体例**

以下に、`TransactionLog` クラスのリファクタリングを示します。

### **Before: 元のクラス**

```java
public class TransactionLog {
    public void saveTransaction(Transaction transaction) {
        // 保存処理
    }
    public void recordError(int code) {
        // エラー処理
    }
}

```

### **Step 1: インタフェースの作成**

```java
public interface TransactionRecorder {
    void saveTransaction(Transaction transaction);
    void recordError(int code);
}

```

### **Step 2: クラスにインタフェースを実装**

```java
public class TransactionLog implements TransactionRecorder {
    @Override
    public void saveTransaction(Transaction transaction) {
        // 保存処理
    }

    @Override
    public void recordError(int code) {
        // エラー処理
    }
}

```

### **Step 3: テスト用のモッククラスを追加**

```java
public class FakeTransactionLog implements TransactionRecorder {
    @Override
    public void saveTransaction(Transaction transaction) {
        // テスト用の保存処理
    }

    @Override
    public void recordError(int code) {
        // テスト用のエラー処理
    }
}

```

### **Step 4: 使用箇所をインタフェースに変更**

```java
public void processTransaction(TransactionRecorder recorder) {
    Transaction transaction = new Transaction();
    recorder.saveTransaction(transaction);
}

```

### **Step 5: テストコードの作成**

```java
public void testTransaction() {
    TransactionRecorder fakeRecorder = new FakeTransactionLog();
    processTransaction(fakeRecorder);
    // アサーション処理
}

```

---

### **ポイントと注意点**

1. **単一責任の原則**
    - インタフェースには、特定の機能に限定したメソッドのみを定義する。
2. **モックの活用**
    - テストでは、本物のクラスを使わずにモックを使うことで軽量なテストを実現。
3. **過剰な抽出を避ける**
    - すべてのクラスにインタフェースを抽出するのではなく、本当に必要な場合のみ適用する。

### **手順**

### **1. 適切なインタフェース名を考える**

- まず、抽出するべき機能群を明確にし、それにふさわしい名前を付けたインタフェースを定義します。
例: `TransactionRecorder`

### **2. クラスにインタフェースを実装させる**

- 対象のクラス（例: `TransactionLog`）に、新しいインタフェースを実装させます。この段階ではメソッドの再編成は行わず、既存の実装をそのまま維持します。

### **3. 使用箇所を変更する**

- 他のクラスが対象クラス（例: `TransactionLog`）に直接依存している場合、それをインタフェース（例: `TransactionRecorder`）を使用する形に置き換えます。

### **4. インタフェースに必要なメソッドを追加**

- インタフェースに必要なメソッドを定義し、元のクラスがそれを実装するように変更します。
- 必要に応じて、既存のクラスに未定義の抽象メソッドを実装。

### **5. テスト用の実装を追加**

- テスト目的でモックやスタブといったテスト用クラス（例: `FakeTransactionLog`）を作成し、新しいインタフェースを実装します。

### **6. コンパイルとテスト**

- 最終的にコンパイルし、すべての依存関係が解決され、コードが正常に動作することを確認します。
## 7. getメソッドの抽出とオーバーライド
### **概要**

「getメソッドの抽出とオーバーライド」とは、クラス内で使用されるインスタンス変数やオブジェクト生成の依存関係を分離するために、`get` メソッドを導入し、そのメソッドをサブクラスでオーバーライドするリファクタリング手法です。

### **特徴**

1. **遅延初期化**: 必要なタイミングで初めてオブジェクトを生成する。
2. **柔軟性の向上**: サブクラスでオーバーライドすることで、異なるオブジェクトを返すことが可能。
3. **依存関係の削減**: オブジェクトの生成処理を一元管理することで、依存を分離。

---

### **使用すべき場面**

1. **オブジェクトの生成処理が複雑な場合**:
    - コンストラクタ内で直接生成しているオブジェクトを分離したい場合。
    - 例: 複数の依存オブジェクトが必要な場合。
2. **テスト用に特定のモックやスタブを注入したい場合**:
    - テスト時にオリジナルのオブジェクトではなく、カスタマイズされたオブジェクトを返したい場合。
3. **遅延初期化が必要な場合**:
    - 不要なオブジェクトを事前に生成するのではなく、必要になった時点で生成したい場合。

---

### **Javaコード例**

### **Before: オブジェクト生成が埋め込まれたコード**

```java
java
コピーする編集する
public class WorkflowEngine {
    private TransactionManager tm;

    public WorkflowEngine() {
        Reader reader = new ModelReader(AppConfiguration.getDryConfiguration());
        Persister persister = new XMLStore(AppConfiguration.getDryConfiguration());
        tm = new TransactionManager(reader, persister);
    }

    public TransactionManager getTransactionManager() {
        return tm;
    }
}

```

- **課題**:
    - コンストラクタ内で直接オブジェクトを生成しており、テストや拡張が難しい。
    - `TransactionManager` のモックやカスタムバージョンを注入するにはコードを改変する必要がある。

---

### **After: getメソッドの抽出とオーバーライドを適用**

1. **getメソッドの抽出**

```java
public class WorkflowEngine {
    private TransactionManager tm;

    public WorkflowEngine() {
        this.tm = null; // 遅延初期化のためにnullで初期化
    }

    protected TransactionManager getTransactionManager() {
        if (tm == null) { // 遅延初期化
            Reader reader = new ModelReader(AppConfiguration.getDryConfiguration());
            Persister persister = new XMLStore(AppConfiguration.getDryConfiguration());
            tm = new TransactionManager(reader, persister);
        }
        return tm;
    }
}

```

1. **サブクラスでオーバーライド**

```java
public class TestingWorkflowEngine extends WorkflowEngine {
    private final FakeTransactionManager fakeTransactionManager = new FakeTransactionManager();

    @Override
    protected TransactionManager getTransactionManager() {
        return fakeTransactionManager; // テスト用のオブジェクトを返す
    }
}

```

1. **使用例**

```java
public class Main {
    public static void main(String[] args) {
        // 本番環境用
        WorkflowEngine productionEngine = new WorkflowEngine();
        TransactionManager tm = productionEngine.getTransactionManager();

        // テスト環境用
        WorkflowEngine testEngine = new TestingWorkflowEngine();
        TransactionManager testTm = testEngine.getTransactionManager();
    }
}

```

---

### **手順**

1. **依存オブジェクトの特定**:
    - `WorkflowEngine` クラスで使用している生成済みのオブジェクトを確認。
2. **getメソッドの導入**:
    - 依存オブジェクトを返す `get` メソッドを作成し、オブジェクト生成処理をその中に移動。
3. **遅延初期化の実装**:
    - `get` メソッド内でオブジェクトが `null` の場合に初めて生成するロジックを追加。
4. **サブクラスの作成**:
    - サブクラスで `get` メソッドをオーバーライドし、テスト用オブジェクトやカスタムオブジェクトを返す実装を追加。
5. **全体の検証**:
    - サブクラスで期待通りのオブジェクトが返されることを確認する。

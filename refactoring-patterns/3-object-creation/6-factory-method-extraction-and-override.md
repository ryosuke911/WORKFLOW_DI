## 6. Factory Methodの抽出とオーバーライド

### **概要**

「Factory Methodの抽出とオーバーライド」とは、コンストラクタ内で直接オブジェクトを生成している場合に、その生成処理をファクトリメソッドとして切り出し、サブクラスでオーバーライドすることにより依存関係を分離するリファクタリング手法です。

### **目的**

- コンストラクタの中に埋め込まれた生成処理を外部化し、依存を分離する。
- テストコードで特定のモックやスタブを使用できるようにする。
- コードの柔軟性と拡張性を向上させる。

---

### **使用すべき場面**

1. **コンストラクタで直接オブジェクトを生成している場合**:
    - 例: コンストラクタ内で `new` を使用して依存オブジェクトを作成している。
2. **テスト環境で特定のオブジェクトを注入したい場合**:
    - 生成するオブジェクトをモックやスタブに差し替える必要がある場合。
3. **オブジェクト生成処理が複雑で再利用したい場合**:
    - 生成処理がコンストラクタを肥大化させている場合。

---

### **Javaコード例**

### **Before: コンストラクタ内で直接オブジェクト生成**

```java
java
コピーする編集する
public class WorkflowEngine {
    private final TransactionManager tm;

    public WorkflowEngine() {
        // コンストラクタ内で直接オブジェクトを生成
        Reader reader = new ModelReader(AppConfiguration.getDryConfiguration());
        Persister persister = new XMLStore(AppConfiguration.getDryConfiguration());
        this.tm = new TransactionManager(reader, persister);
    }
}

```

- **課題**:
    - `Reader` や `Persister` の生成処理がコンストラクタに埋め込まれており、テストが難しい。
    - モックやスタブを使用するにはコード全体を変更する必要がある。

---

### **After: Factory Methodの抽出とオーバーライド**

1. **Factory Methodの抽出**

```java
java
コピーする編集する
public class WorkflowEngine {
    private final TransactionManager tm;

    public WorkflowEngine() {
        // Factory Methodで生成
        this.tm = makeTransactionManager();
    }

    // Factory Methodとして切り出し
    protected TransactionManager makeTransactionManager() {
        Reader reader = new ModelReader(AppConfiguration.getDryConfiguration());
        Persister persister = new XMLStore(AppConfiguration.getDryConfiguration());
        return new TransactionManager(reader, persister);
    }
}

```

1. **サブクラスでオーバーライド**

```java
java
コピーする編集する
public class TestingWorkflowEngine extends WorkflowEngine {
    @Override
    protected TransactionManager makeTransactionManager() {
        // テスト用のモックオブジェクトを返す
        return new FakeTransactionManager();
    }
}

```

1. **使用例**

```java
java
コピーする編集する
public class Main {
    public static void main(String[] args) {
        // 本番環境用
        WorkflowEngine productionEngine = new WorkflowEngine();

        // テスト環境用
        WorkflowEngine testEngine = new TestingWorkflowEngine();
    }
}

```

---

### **手順**

1. **コンストラクタ内の生成処理を特定**:
    - コンストラクタで直接生成されているオブジェクトを抽出対象として選定。
2. **Factory Methodを定義**:
    - 抽出した生成処理を独立したメソッドとして切り出し。
    - メソッドを `protected` に設定してサブクラスでオーバーライド可能にする。
3. **コンストラクタでFactory Methodを呼び出す形に変更**:
    - コンストラクタ内の `new` を削除し、代わりにFactory Methodを呼び出す。
4. **サブクラスでFactory Methodをオーバーライド**:
    - 特定のテスト用オブジェクトやモックオブジェクトを返す処理をサブクラスに実装。
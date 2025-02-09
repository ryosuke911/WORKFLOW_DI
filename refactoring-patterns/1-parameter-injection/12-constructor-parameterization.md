## 12. コンストラクタのパラメータ化

### **概要**

「コンストラクタのパラメータ化」は、クラスのコンストラクタでオブジェクトを生成している場合に、テストの容易さや柔軟性を向上させるために、生成されるオブジェクトを外部から注入できるように設計を変更する手法です。これにより、依存関係の置き換えやテスト環境でのモック化が可能になります。

例えば、クラス内で直接生成される依存オブジェクトを削除し、それをコンストラクタの引数として外部から渡すことで、コードの可読性とテスト容易性が向上します。

---

### **使用すべき場面**

- **テストコードの改善**: クラスが依存しているオブジェクトを簡単にモックやスタブに置き換えられるようにする必要がある場合。
- **柔軟な設計**: コンストラクタ内で特定の実装クラスを直接生成することで発生する硬直性を取り除きたい場合。
- **依存性注入（Dependency Injection）を導入**: テストや本番環境で異なる依存オブジェクトを使用する設計が求められる場合。

---

### **Javaコード例**

### **リファクタリング前のコード**

```java
public class MailChecker {
    private MailReceiver receiver;
    private int checkPeriodSeconds;

    public MailChecker(int checkPeriodSeconds) {
        this.receiver = new MailReceiver(); // コンストラクタ内で直接生成
        this.checkPeriodSeconds = checkPeriodSeconds;
    }

    // メソッド
}

```

### **リファクタリング後のコード**

```java
public class MailChecker {
    private MailReceiver receiver;
    private int checkPeriodSeconds;

    // パラメータ化されたコンストラクタ
    public MailChecker(MailReceiver receiver, int checkPeriodSeconds) {
        this.receiver = receiver;
        this.checkPeriodSeconds = checkPeriodSeconds;
    }

    // デフォルトの挙動を維持するためのコンストラクタ
    public MailChecker(int checkPeriodSeconds) {
        this(new MailReceiver(), checkPeriodSeconds); // 既存のコードを利用
    }

    // メソッド
}

```

---

### **メリット**

1. **テストの容易さ**: テスト時に依存オブジェクトをモック化することで、ユニットテストが簡単に行える。
2. **設計の柔軟性**: 新たなオブジェクト生成の処理を外部に委譲することで、クラスの変更が不要になる。
3. **再利用性の向上**: 依存オブジェクトを外部から注入できるため、異なる文脈で同じクラスを活用しやすい。

---

### **手順**

1. **パラメータ化したいコンストラクタを特定**。
2. **依存オブジェクトを生成する処理を削除し、それをコンストラクタの引数に追加**。
3. **引数を通じて依存オブジェクトを注入する新しいコンストラクタを作成**。
4. **既存のコンストラクタを新しいコンストラクタを呼び出す形式に変更して後方互換性を維持**。

---

### **注意点**

- **テストの有無を確認**: リファクタリング後にテストが壊れていないか確認する。
- **依存オブジェクトのライフサイクル管理**: 外部から注入されたオブジェクトのライフサイクルを適切に管理する必要がある。
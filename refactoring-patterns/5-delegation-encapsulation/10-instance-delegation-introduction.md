## 10. インスタンス委譲の導入

### **概要**

「インスタンス委譲の導入」は、クラス内の`static`メソッドをインスタンスメソッドに変換することで、テスト可能性を向上させるリファクタリング手法です。この方法により、`static`メソッドの呼び出しに依存していたコードがインスタンスを通じて柔軟に動作するようになります。

`static`メソッドはグローバルな性質を持つため、特定の動作をテスト環境で置き換えたりモック化したりするのが困難です。この手法では、該当する`static`メソッドをインスタンスメソッドに移行し、依存性を注入することでテストの容易性を向上させます。

---

### **使用すべき場面**

- テストコードでモックやスタブを使用したい場合。
- `static`メソッドがコードの再利用や拡張を阻害している場合。
- グローバルな`static`メソッドの依存を排除し、より柔軟な設計を目指す場合。
- `static`メソッドを複数の異なる動作で置き換える必要がある場合（例: テスト用モック、プロダクション用実装の切り替え）。

---

### **手順**

1. **問題となる`static`メソッドの特定**
    
    テスト環境や再利用性の観点から、改善が必要な`static`メソッドを特定します。
    
2. **インスタンスメソッドの作成**
    
    該当する`static`メソッドの処理を同じクラスまたは新しいクラスのインスタンスメソッドに移行します。
    
3. **呼び出し箇所の変更**
    
    元の`static`メソッドを呼び出していた箇所を、新しいインスタンスメソッドを利用する形に変更します。この際に、必要であれば依存性注入（Dependency Injection）を活用します。
    
4. **`static`メソッドの削除**
    
    元の`static`メソッドが使用されなくなったことを確認し、削除します。
    

---

### **Javaコード例**

### **リファクタリング前**

```java
public class BankingServices {
    public static void updateAccountBalance(int userId, Money amount) {
        // 処理
    }
}

public class SomeClass {
    public void someMethod() {
        BankingServices.updateAccountBalance(12345, new Money(1000));
    }
}

```

### **リファクタリング後**

```java
public class BankingServices {
    public void updateBalance(int userId, Money amount) {
        // 元のstaticメソッドの処理を移行
    }
}

public class SomeClass {
    private final BankingServices services;

    public SomeClass(BankingServices services) {
        this.services = services;
    }

    public void someMethod() {
        services.updateBalance(12345, new Money(1000));
    }
}

```

---
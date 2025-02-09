## 17. getメソッドによるグローバル参照の置き換え

### **概要**

「getメソッドによるグローバル参照の置き換え」とは、コードの中で直接グローバル変数や静的メソッドを参照している箇所を、`getメソッド`を通してアクセスする形に変更する手法です。この手法を用いることで、以下のようなメリットがあります：

1. **依存関係の明確化**
    
    グローバルな依存関係を`getメソッド`を介して抽象化することで、コードがどのオブジェクトやクラスに依存しているのかが明確になります。
    
2. **テスト可能性の向上**
    
    グローバル参照に依存しているコードでは、モックやスタブを利用したテストが困難です。`getメソッド`を使うことで、テスト用の依存オブジェクトを注入できるようになり、テストが容易になります。
    
3. **柔軟な設計**
    
    サブクラスで`getメソッド`をオーバーライドすることで、動作を簡単にカスタマイズできるようになります。これにより、コードの拡張性が向上します。
    
4. **グローバル依存の解消**
    
    グローバル変数や静的メソッドの直接使用を避け、コード全体をよりモジュール化し、依存関係を緩和できます。
    

---

## **この手法が必要になる場面**

### **1. グローバル変数や静的メソッドの参照が多い場合**

コードの中でグローバル変数や静的メソッドに直接依存している箇所が多いと、コードの再利用性やテスト可能性が低下します。この手法を用いることで、こうした依存関係を緩和できます。

### **2. テストを容易にする必要がある場合**

特定のオブジェクトやクラスの動作をテストする際、グローバル依存があると動作が固定化され、テスト用のダミーオブジェクト（モックやスタブ）を差し込むことが難しくなります。この手法を導入することで、テスト環境で自由に挙動を制御できます。

### **3. コードの設計を柔軟に保つ必要がある場合**

静的メソッドやグローバル変数に依存していると、変更や拡張の際に多くのコード修正が必要になることがあります。この手法では依存関係を間接化するため、変更の影響を最小限に抑えられます。

---

## **手順**

1. **置き換えたいグローバル参照を特定する**
    
    コード内で直接グローバル変数や静的メソッドを呼び出している箇所を確認します。
    
2. **`getメソッド`を記述する**
    
    グローバル参照を返す`getメソッド`をクラス内に作成します。このメソッドはサブクラスでオーバーライド可能にするために`protected`として定義します。
    
3. **すべてのグローバル参照を`getメソッド`の呼び出しに置き換える**
    
    グローバル参照を直接使用している箇所を、`getメソッド`を通じたアクセスに変更します。
    
4. **テスト用サブクラスを作成し、`getメソッド`をオーバーライドする**
    
    テスト用に依存関係を差し替えたサブクラスを作成し、動作を検証します。
    

---

## **Javaコード例**

---

### **改善前のコード**

このコードでは、`Inventory`クラスの静的メソッドを直接使用しており、依存関係が強く、テストが難しい状態です。

```java
public class RegisterSale {
    public void addItem(Barcode code) {
        Item newItem = Inventory.getInventory().itemForBarcode(code);
        items.add(newItem);
    }
}

```

- `Inventory.getInventory()`が静的な呼び出しであり、グローバル参照となっています。
- テスト時に`Inventory`クラスの振る舞いを変更することができません。

---

### **改善後のコード**

以下のコードは、`getメソッド`を用いてグローバル参照を抽象化しています。

```java
java
コピーする編集する
public class RegisterSale {
    public void addItem(Barcode code) {
        Item newItem = getInventory().itemForBarcode(code); // getメソッドを利用
        items.add(newItem);
    }

    // Inventoryを取得するためのgetメソッド
    protected Inventory getInventory() {
        return Inventory.getInventory(); // 静的メソッドをラップ
    }
}

```

- `getInventory()`メソッドを使用することで、`Inventory`への依存を間接化しました。
- サブクラスでこの`getメソッド`をオーバーライドすることで、テスト用のモックを注入できます。

---

### **テスト用サブクラス**

以下は、テスト用にモックした`Inventory`クラスを利用する例です。

```java
java
コピーする編集する
// テスト用のモッククラス
public class FakeInventory extends Inventory {
    @Override
    public Item itemForBarcode(Barcode code) {
        // テスト用のデータを返す
        return new Item("TestItem");
    }
}

// RegisterSaleをテスト用に拡張
public class TestingRegisterSale extends RegisterSale {
    private final Inventory inventory = new FakeInventory();

    @Override
    protected Inventory getInventory() {
        return inventory; // テスト用のモックを返す
    }
}

```

- `FakeInventory`は、テスト用に作成したモッククラスです。
- `TestingRegisterSale`は`RegisterSale`を拡張し、`getInventory()`をオーバーライドしてモックを返します。
## 11. 静的setメソッドの導入

### **概要**

静的setメソッドの導入は、クラス内で保持されているSingletonのインスタンスやグローバル参照を、テスト時に柔軟に置き換えることを目的とした設計手法です。この手法では、静的setメソッドを通じて、オブジェクトの参照を外部から注入する仕組みを提供します。これにより、テスト環境に適したモックオブジェクトや特定の環境に適合するカスタムオブジェクトを使用することが可能になります。

---

### **使用すべき場面**

1. **Singletonパターンの柔軟性の向上**
    
    Singletonインスタンスが複数の場所で使用されており、その置き換えが必要な場合。この方法を用いると、コード全体に影響を与えずにインスタンスを変更できます。
    
2. **テスト時の依存性の解消**
    
    実際のオブジェクトの代わりにモックを利用してテストを実行したい場合。この手法を適用すると、環境依存の部分をモックで置き換え、独立したテストが可能になります。
    
3. **柔軟な設計が求められる場面**
    
    システムが複数の環境で動作し、それぞれの環境に応じて異なるオブジェクト参照を設定する必要がある場合。
    

---

### **Javaコード例**

### **元のコード**

```java
java
コピーする編集する
public class RouterFactory {
    private static RouterServer server = new RouterServer();

    public static Router makeRouter() {
        return server.makeRouterInstance();
    }
}

```

### **静的setメソッド導入後のコード**

```java
java
コピーする編集する
public class RouterFactory {
    private static RouterServer server;

    // 静的setメソッドの導入
    public static void setServer(RouterServer newServer) {
        server = newServer;
    }

    public static Router makeRouter() {
        return server.makeRouterInstance();
    }
}

```

### **テスト時の利用例**

```java
java
コピーする編集する
@Test
public void testRouterFactory() {
    RouterServer mockServer = new MockRouterServer(); // モックオブジェクト
    RouterFactory.setServer(mockServer); // モックを注入

    Router router = RouterFactory.makeRouter();
    assertEquals("MockRouter", router.getName());
}

```

---

### **手順**

1. Singletonクラスのコンストラクタのアクセス制限を下げてサブクラス化を可能にする。
2. Singletonクラスに静的setメソッドを追加し、外部からインスタンスを注入可能にする。
3. テスト環境や特定の目的に応じて、外部から注入されたオブジェクトを利用する。
4. 必要に応じてtearDownメソッドを使い、テスト終了後に元の状態に戻す。
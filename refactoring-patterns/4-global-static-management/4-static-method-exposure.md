## 4. 静的メソッドの公開
## **概要**

「静的メソッドの公開」とは、静的メソッドやグローバル参照を直接使用しているコードをリファクタリングし、それを他の部分から分離してカスタマイズ可能にする手法です。これにより、特定の依存関係を柔軟に変更できるようになり、特にテストコードの作成が容易になります。

この手法では、以下の手順を踏みます：

1. **静的メソッドの呼び出しをインスタンスメソッドに委譲する**
    
    直接静的メソッドを呼び出す代わりに、その処理をインスタンスメソッドに切り出します。
    
2. **インスタンスメソッドをオーバーライド可能にする**
    
    サブクラスでカスタマイズできるよう、メソッドの可視性を調整します。
    
3. **テスト用サブクラスを導入する**
    
    本番環境のロジックを変更せずに、テスト用のメソッドを簡単に差し替えることができます。
    

---

## **この手法が必要になる場面**

1. **静的メソッドへの依存を分離したい場合**
    
    コードの柔軟性を高めたい場合や、特定のメソッドに依存している部分をモジュール化したい場合に役立ちます。
    
2. **テストを容易にしたい場合**
    
    直接的な静的メソッドの呼び出しではテストが難しく、モックやスタブを利用したテストを行いたい場合に使用します。
    
3. **複数の場面で異なる動作を求める場合**
    
    本番環境とテスト環境、あるいは異なる条件で処理を変えたい場合に、この手法を使うことで切り替えが容易になります。
    

---

## **手順**

1. **抽出したい静的メソッドを特定する**
    
    コード内で頻繁に呼び出される静的メソッドを見つけます。
    
2. **静的メソッドのロジックを新しいメソッドに切り出す**
    
    新しいインスタンスメソッドを定義し、そこに静的メソッドのロジックを移します。
    
3. **元のコードをインスタンスメソッドの呼び出しに置き換える**
    
    静的メソッドの呼び出しを、作成したインスタンスメソッドに変更します。
    
4. **サブクラスでメソッドをオーバーライドする**
    
    サブクラスで必要に応じてメソッドをオーバーライドし、動作をカスタマイズします。
    

---

## **Javaコード例**

以下に、静的メソッドを分離してテスト可能にする具体例を示します。

---

### **リファクタリング前のコード**

以下のコードでは、`StyleMaster.formStyles()`という静的メソッドが直接呼び出されています。

```java
public class PageLayout {
    private int id = 0;
    private List styles;
    private StyleTemplate template;

    protected void rebindStyles() {
        styles = StyleMaster.formStyles(template, id);
    }
}

```

- `StyleMaster.formStyles()`は静的メソッドであり、`PageLayout`クラスが`StyleMaster`に強く依存しています。
- テスト時に`StyleMaster.formStyles()`の挙動を変更することができません。

---

### **リファクタリング後のコード**

静的メソッドの呼び出しをインスタンスメソッドに分離し、サブクラスでオーバーライド可能にしました。

```java
public class PageLayout {
    private int id = 0;
    private List styles;
    private StyleTemplate template;

    protected void rebindStyles() {
        styles = formStyles(template, id);
    }

    // 静的メソッドの呼び出しをラップ
    protected List formStyles(StyleTemplate template, int id) {
        return StyleMaster.formStyles(template, id);
    }
}

```

- `formStyles()`メソッドをインスタンスメソッドとして定義しました。
- `StyleMaster.formStyles()`の呼び出しをこのメソッド内に移動することで、依存関係を間接化しました。

---

### **テスト用サブクラスの導入**

以下は、`PageLayout`をテストするためにカスタマイズしたサブクラスの例です。

```java
public class TestingPageLayout extends PageLayout {
    @Override
    protected List formStyles(StyleTemplate template, int id) {
        // テスト用に空のリストを返す
        return new ArrayList();
    }
}

```

- `formStyles()`メソッドをオーバーライドし、テスト用の挙動を実現しています。
- 本番コードを変更せずに、テスト用のロジックを差し込むことができます。

---

### **テストコードの例**

以下は、`TestingPageLayout`を利用したテストコードの例です。

```java
public class PageLayoutTest {
    public static void main(String[] args) {
        PageLayout layout = new TestingPageLayout();
        layout.rebindStyles();

        // stylesが空のリストであることを確認
        System.out.println("Styles: " + layout.styles);
    }
}

```

- `TestingPageLayout`クラスを利用することで、本番環境と異なる動作を容易にテストできます。

## 5. 呼び出しの抽出とオーバーライド

### **概要**

「呼び出しの抽出とオーバーライド」とは、特定の呼び出しをメソッドとして抽出し、そのメソッドをサブクラスでオーバーライドすることにより依存関係を分離するリファクタリング手法です。この手法は、以下のような状況で役立ちます：

1. **依存の削減**: グローバル変数や静的メソッドに直接依存するコードを柔軟に変更可能。
2. **テスト容易性**: オーバーライドを用いて異なる動作を注入することでテストを容易にする。
3. **拡張性向上**: サブクラスを用いることで個別の処理に対応できる。

---

### **使用すべき場面**

1. **グローバル変数や静的メソッドを利用している場合**:
    - 依存する静的メソッドの影響を抑えたい場合。
    - 例: 静的メソッド `StyleMaster.formStyles()` に依存するコードを柔軟に管理したい。
2. **特定のメソッドの動作をカスタマイズしたい場合**:
    - サブクラスごとに異なる振る舞いを持たせたい場合。
    - 例: テスト用のクラスで異なる戻り値を提供する必要がある。
3. **テストコードでモックを利用したい場合**:
    - テスト時に特定のメソッドの振る舞いを切り替えたい場合。
    - 例: 外部依存を排除し、単純なデータを返すようにする。

---

### **Javaコード例**

### Before: 静的メソッドに直接依存する場合

```java
java
コピーする編集する
public class PageLayout {
    private int id = 0;
    private List<String> styles;
    private StyleTemplate template;

    // スタイルを再バインド
    protected void rebindStyles() {
        // 静的メソッドを直接利用
        styles = StyleMaster.formStyles(template, id);
    }
}

```

- **課題**:
    - 静的メソッド `StyleMaster.formStyles()` に直接依存しているため、テストや拡張が困難。

---

### After: 呼び出しを抽出してオーバーライド可能にする

1. **抽出されたメソッド**

```java
java
コピーする編集する
public class PageLayout {
    private int id = 0;
    private List<String> styles;
    private StyleTemplate template;

    // スタイルを再バインド
    protected void rebindStyles() {
        // 抽出されたメソッドを利用
        styles = formStyles(template, id);
    }

    // 抽出した呼び出し
    protected List<String> formStyles(StyleTemplate template, int id) {
        return StyleMaster.formStyles(template, id);
    }
}

```

1. **サブクラスでのオーバーライド**

```java
java
コピーする編集する
public class TestingPageLayout extends PageLayout {
    @Override
    protected List<String> formStyles(StyleTemplate template, int id) {
        // テスト用に空のリストを返す
        return new ArrayList<>();
    }
}

```

1. **使用例**

```java
public class Main {
    public static void main(String[] args) {
        // 本番用のPageLayout
        PageLayout pageLayout = new PageLayout();
        pageLayout.rebindStyles();

        // テスト用のPageLayout
        PageLayout testLayout = new TestingPageLayout();
        testLayout.rebindStyles();
    }
}

```

---

### **手順**

1. **呼び出しを特定**:
    - 対象のメソッド内で依存している呼び出しを特定。
2. **新しいメソッドに抽出**:
    - 抽出した呼び出し部分を独立したメソッドとして定義。
    - シグネチャを維持し、他の部分への影響を抑える。
3. **元のコードを新しいメソッドを使う形に修正**:
    - 元のメソッドで直接呼び出していた部分を、抽出したメソッドの呼び出しに置き換える。
4. **サブクラスを作成してオーバーライド**:
    - 必要に応じてサブクラスでメソッドをオーバーライドし、振る舞いをカスタマイズ。
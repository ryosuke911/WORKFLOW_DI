## 8. 実装の抽出

### 概要:

「実装の抽出」とは、既存クラスの具体的な実装部分をサブクラスに移動し、元のクラスをインターフェースに変えるリファクタリング手法です。この手法は、コードを分割して依存関係を整理しやすくするために使用されます。抽出後、元のクラスは純粋なインターフェースとして機能し、具体的な実装はサブクラスで行われるようになります。

---

### 使用すべき場面:

- **依存関係の削減:** 元のクラスが他の多くのクラスに依存している場合。
- **テスト容易性の向上:** 元のクラスの依存部分をモックやスタブに置き換えたい場合。
- **柔軟性の向上:** 複数の実装を容易に切り替えられるようにしたい場合。
- **再利用性の向上:** 特定の実装に依存せず、汎用的なインターフェースを提供したい場合。

---

### 手順:

1. **元のクラスの宣言をコピーする:**
元のクラスの宣言を新しいヘッダーファイル（または別ファイル）にコピーし、そのクラス名を異なる名前に変更する。
2. **元のクラスをインターフェースにする:**
元のクラスのpublicなメソッドのみを残し、非publicな変数や実装部分をすべて削除する。残ったメソッドを抽象メソッドに変更。
3. **新しいサブクラスを作成する:**
元のクラスを継承した新しいクラスを作り、具体的な実装をそこに移動する。
4. **システム全体を再コンパイルし、インターフェースの影響範囲を確認:**
影響範囲を確認し、必要に応じて依存関係を整理する。
5. **動作確認とテスト:**
システムの再テストを行い、動作が問題なく移行されたことを確認する。

---

### Javaコード例:

**元のクラス:**

```java
public class ModelNode {
    private List<ModelNode> interiorNodes;
    private List<ModelNode> exteriorNodes;

    public void addInteriorNode(ModelNode node) {
        interiorNodes.add(node);
    }

    public void addExteriorNode(ModelNode node) {
        exteriorNodes.add(node);
    }

    public void colorize() {
        // 具体的な実装
    }
}

```

**ステップ1: 元のクラスをインターフェースに変換**

```java
public interface ModelNode {
    void addInteriorNode(ModelNode node);
    void addExteriorNode(ModelNode node);
    void colorize();
}

```

**ステップ2: 実装を新しいサブクラスに移動**

```java
public class ProductionModelNode implements ModelNode {
    private List<ModelNode> interiorNodes = new ArrayList<>();
    private List<ModelNode> exteriorNodes = new ArrayList<>();

    @Override
    public void addInteriorNode(ModelNode node) {
        interiorNodes.add(node);
    }

    @Override
    public void addExteriorNode(ModelNode node) {
        exteriorNodes.add(node);
    }

    @Override
    public void colorize() {
        // 具体的な実装
    }
}

```

**ステップ3: 使用箇所を修正**

```java
ModelNode node = new ProductionModelNode();
node.addInteriorNode(new ProductionModelNode());
node.colorize();

```

---
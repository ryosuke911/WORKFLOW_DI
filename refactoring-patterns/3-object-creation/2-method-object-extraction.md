## 2. メソッドオブジェクトの取り出し
### **概要**

「メソッドオブジェクトの取り出し」は、長大で複雑なメソッドを専用のクラスに切り出すことで、次のようなメリットを得るリファクタリング手法です：

1. **メソッドの複雑性を軽減**:
    - メソッド内のロジックを整理して読みやすくする。
2. **テスト可能性の向上**:
    - 切り出されたクラスを単独でテストできるようにする。
3. **責務の分離**:
    - クラスやメソッドの役割を限定し、単一責任の原則（SRP）を満たしやすくする。
4. **依存の整理**:
    - メソッドが依存しているローカル変数や他のクラスを明示的に扱える。

---

### **使用すべき場面**

1. **長大なメソッドをリファクタリングしたい場合**:
    - 100行以上あるような長いメソッド。
    - 複数の責務が混在しているメソッド。
2. **ローカル変数が多く、分割が難しい場合**:
    - メソッド内で複数のローカル変数を扱っているため、簡単に関数に分割できない場合。
3. **メソッドが外部依存を持ち、テストが困難な場合**:
    - フレームワークや外部サービスに依存しているロジックを分割したい場合。
4. **コードの可読性が低下している場合**:
    - メソッド内で複数のロジックが混在しているため、処理の流れが追いづらい場合。

---

### **Javaコード例**

### Before: 複雑で長大なメソッド

```java
java
コピーする編集する
class GDIBrush {
    public void draw(Vector<Point> renderingRoots, ColorMatrix colorMatrix, Vector<Point> selection) {
        for (Vector<Point>::iterator it = renderingRoots.begin(); it != renderingRoots.end(); ++it) {
            Point p = *it;

            // 追加処理1: 描画するポイントの条件判定
            if (selection.contains(p)) {
                continue;
            }

            // 追加処理2: 色の適用
            Color color = colorMatrix.getColorForPoint(p);

            // 描画処理
            drawPoint(p.x, p.y, color);
        }
    }

    private void drawPoint(int x, int y, Color color) {
        // 実際の描画処理
    }
}

```

- **課題**:
    - `draw` メソッドが長く、複数の処理（条件判定、色適用、描画）が混在している。
    - `drawPoint` への依存が強いため、テストが難しい。

---

### After: メソッドオブジェクトの取り出し

1. **専用クラスを作成**

```java
java
コピーする編集する
class Renderer {
    private final GDIBrush brush;
    private final Vector<Point> renderingRoots;
    private final ColorMatrix colorMatrix;
    private final Vector<Point> selection;

    public Renderer(GDIBrush brush, Vector<Point> renderingRoots, ColorMatrix colorMatrix, Vector<Point> selection) {
        this.brush = brush;
        this.renderingRoots = renderingRoots;
        this.colorMatrix = colorMatrix;
        this.selection = selection;
    }

    public void render() {
        for (Vector<Point>::iterator it = renderingRoots.begin(); it != renderingRoots.end(); ++it) {
            Point p = *it;

            // 条件判定
            if (selection.contains(p)) {
                continue;
            }

            // 色適用
            Color color = colorMatrix.getColorForPoint(p);

            // 描画
            brush.drawPoint(p.x, p.y, color);
        }
    }
}

```

1. **元のクラスを修正**

```java
java
コピーする編集する
class GDIBrush {
    public void draw(Vector<Point> renderingRoots, ColorMatrix colorMatrix, Vector<Point> selection) {
        Renderer renderer = new Renderer(this, renderingRoots, colorMatrix, selection);
        renderer.render();
    }

    public void drawPoint(int x, int y, Color color) {
        // 実際の描画処理
    }
}

```

---

### **メリット**

1. **複雑性の軽減**:
    - `Renderer` クラスに処理を移動することで、`GDIBrush` の責務を減らし、可読性が向上。
2. **テストのしやすさ**:
    - `Renderer` クラスを独立してテスト可能。
3. **再利用性の向上**:
    - 描画ロジックを他のクラスやシステムで再利用しやすい。

---

### **手順**

1. **新しいクラスを作成する**:
    - 取り出すメソッドのコードを移動するためのクラスを新規作成。
    - クラス名は、メソッドの責務や役割を反映したものにする。
2. **コンストラクタを作成し、シグネチャを維持する**:
    - 作成したクラスにコンストラクタを定義し、元のメソッドで使用しているインスタンス変数やローカル変数を引数として渡す。
    - **目的**: シグネチャの維持によりコードの整合性を確保。
3. **元のメソッドをそのままコピー**:
    - 新しいクラスに、元のメソッドのロジックをそのまま移動。
    - 必要に応じて、移動元のインスタンス変数やローカル変数を新しいクラス内でインスタンス変数として宣言。
4. **空の実行用メソッドを作成する**:
    - 新しいクラス内に、移動した処理を呼び出すための空のメソッドを作成。
    - メソッド名は元のメソッド名（例: `run` や `execute` など）を反映。
5. **移動元のメソッド本体を新しいクラスに移行し、コンパイルを行う**:
    - 元のクラスのメソッドから処理を削除し、新しいクラスの実行用メソッドに移行。
    - コンパイルが通ることを確認。
6. **必要に応じてアクセス修飾子を調整する**:
    - 新しいクラスで呼び出されるメソッドが `private` な場合は、`public` や `protected` に変更。
    - インスタンス変数やメソッドへのアクセスが必要ならば `getter` や `setter` を追加。
7. **元のクラスを更新**:
    - 元のクラスに新しいクラスのインスタンスを生成するコードを追加し、そのクラスに処理を委譲。

8. **インターフェースの抽出を行う（必要に応じて）**:


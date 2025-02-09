## 1. パラーメータの適合
### **概要**

「パラメータの適合」とは、既存のメソッドが特定の具体的な依存（例: `HttpServletRequest`）に縛られている場合に、その依存を抽象化することでメソッドの柔軟性を高める手法です。これにより、テスト可能性を向上させ、コードの再利用性を高めることが目的です。

具体的には、新しいインタフェース（例: `ParameterSource`）を導入し、そのインタフェースに基づいた実装クラス（例: `ServletParameterSource` や `FakeParameterSource`）を切り替えて使用します。

---

### **使用すべき場面**

1. **既存のメソッドがフレームワークやライブラリに強く依存している場合**:
    - 例: `HttpServletRequest` のような特定のWebフレームワークに縛られているメソッド。
    - 解決方法: 特定の依存を抽象化し、テスト環境でも簡単に利用できるようにする。
2. **テストのためのモックやスタブが準備しづらい場合**:
    - 特定のフレームワークオブジェクトを直接扱っているため、テスト用にモックを作るのが難しい場合。
    - 解決方法: フレームワーク依存のないモック用クラスを用意。
3. **コードの変更による影響範囲を最小限に抑えたい場合**:
    - 直接的に依存を削除するよりも、間接的に依存を扱う方法でリファクタリングする。

---

### **Javaコード例**

### Before: フレームワーク依存のコード

```java
java
コピーする編集する
public class ARMDispatcher {
    public void populate(HttpServletRequest request) {
        String[] values = request.getParameterValues("pageStateName");
        if (values != null && values.length > 0) {
            marketBindings.put("pageStateName" + getDateStamp(), values[0]);
        }
    }
}

```

- **課題**:
    - `HttpServletRequest` に強く依存しており、フレームワークの環境でしか動作しない。
    - テストが困難（モックやスタブを使うにも複雑）。

---

### After: パラメータの適合を導入

```java
// 1. 新しいインタフェース
public interface ParameterSource {
    String getParameterForName(String name);
}

// 2. 本番用の実装クラス
public class ServletParameterSource implements ParameterSource {
    private final HttpServletRequest request;

    public ServletParameterSource(HttpServletRequest request) {
        this.request = request;
    }

    @Override
    public String getParameterForName(String name) {
        String[] values = request.getParameterValues(name);
        if (values == null || values.length < 1) {
            return null;
        }
        return values[0];
    }
}

// 3. テスト用のモッククラス
public class FakeParameterSource implements ParameterSource {
    private final String value;

    public FakeParameterSource(String value) {
        this.value = value;
    }

    @Override
    public String getParameterForName(String name) {
        return value;
    }
}

// 4. 修正されたメソッド
public class ARMDispatcher {
    public void populate(ParameterSource source) {
        String value = source.getParameterForName("pageStateName");
        if (value != null) {
            marketBindings.put("pageStateName" + getDateStamp(), value);
        }
    }
}

```

- **改善点**:
    - `ParameterSource` を介して依存が抽象化され、テスト用クラスで差し替え可能に。
    - 本番では `ServletParameterSource` を使用することで既存動作を維持。

---

### **使用手順**

1. **新しいインタフェースを作成**:
    - 例: `ParameterSource` のように、依存を抽象化するインタフェースを定義。
    - 単純でわかりやすく、必要最低限のメソッドを設計。
2. **本番用クラスの実装**:
    - 既存の具体的な依存（例: `HttpServletRequest`）を使って、インタフェースを実装するクラスを作成。
3. **テスト用クラスの作成**:
    - テスト環境で使用するモックやスタブを実装。単純な動作を保証するクラスを作る。
4. **既存メソッドを修正**:
    - 具体的な型の代わりに、抽象化されたインタフェースを使用する形にメソッドを変更。
5. **テストケースの追加**:
    - テスト用のモッククラスを利用して、リファクタリング後の動作を検証。

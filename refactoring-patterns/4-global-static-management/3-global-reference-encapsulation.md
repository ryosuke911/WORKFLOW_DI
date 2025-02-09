## 3. グローバル参照のカプセル化

## **概要**

グローバル変数や関数の直接的な利用は、コードの依存関係を複雑にし、テストや拡張を困難にする原因となります。「グローバル参照のカプセル化」は、こうした問題を解消するための設計手法の一つであり、以下を目的とします：

1. **依存関係の分離**
    
    グローバル参照をクラスの責務としてカプセル化し、アクセスをそのクラス経由に限定します。
    
2. **柔軟性の向上**
    
    カプセル化されたクラスをテスト用や本番用に切り替えることで、コードの動作を柔軟に変更できます。
    
3. **保守性の改善**
    
    グローバル参照がどこで使用されているかを明確化し、変更の影響範囲を限定します。
    

---

## **この手法が必要になる場面**

1. **グローバル変数が多数の場所で参照されている場合**
    
    直接的な参照が多いと、変更やテストが難しくなります。この手法を使えば、グローバル参照を一箇所に集約できます。
    
2. **依存関係の切り替えが必要な場合**
    
    例えば、テスト環境でモックを利用したり、本番環境で異なる動作をさせたりする必要がある場合です。
    
3. **コードの再利用性を高めたい場合**
    
    グローバル変数への依存を減らすことで、コードの再利用が容易になります。
    

---

## **「グローバル参照のカプセル化」の手順**

1. **カプセル化したいグローバル参照を特定する**
使用されているグローバル変数や関数をすべて確認し、どれをカプセル化するか選定します。
2. **新しいクラスを作成して、グローバル参照をカプセル化する**
グローバル参照のアクセス方法を管理するクラスを作成します。このクラスが責務を持つことで、グローバル参照が隠蔽されます。
3. **元のグローバル変数を削除し、カプセル化クラスを利用する**
既存のコードを変更し、グローバル参照へのアクセスを新しいクラス経由に切り替えます。
4. **テストやメンテナンスを実施する**
カプセル化クラスが正しく機能するかをテストし、必要に応じて調整します。

---

## **Javaコード例**

以下に、「グローバル参照のカプセル化」を適用した例を示します。

### **カプセル化前のコード**

以下のコードは、グローバル変数を直接利用しており、依存関係が強く、テストが困難です。

```java
public class AGGController {
    private static final int AGG230_SIZE = 256;
    private static boolean[] activeFrame = new boolean[AGG230_SIZE];
    private static boolean[] suspendedFrame = new boolean[AGG230_SIZE];

    public void suspendFrame() {
        System.arraycopy(activeFrame, 0, suspendedFrame, 0, AGG230_SIZE);
        clear(activeFrame);
    }

    private void clear(boolean[] frame) {
        for (int i = 0; i < frame.length; i++) {
            frame[i] = false;
        }
    }
}

```

- `activeFrame`と`suspendedFrame`はグローバルに定義されています。
- この実装では、テスト時に`activeFrame`や`suspendedFrame`を切り替えることができません。

---

### **カプセル化後のコード**

以下は、グローバル変数をクラスにカプセル化した例です。

```java
public class Frame {
    private boolean[] activeFrame;
    private boolean[] suspendedFrame;

    public Frame(int size) {
        activeFrame = new boolean[size];
        suspendedFrame = new boolean[size];
    }

    public boolean[] getActiveFrame() {
        return activeFrame;
    }

    public boolean[] getSuspendedFrame() {
        return suspendedFrame;
    }

    public void clearActiveFrame() {
        for (int i = 0; i < activeFrame.length; i++) {
            activeFrame[i] = false;
        }
    }

    public void copyActiveToSuspended() {
        System.arraycopy(activeFrame, 0, suspendedFrame, 0, activeFrame.length);
    }
}

```

- グローバル変数を`Frame`クラスにカプセル化し、その操作をメソッドとして提供します。

---

### **利用するクラスの例**

カプセル化クラスを利用するように変更したコードです。

```java
public class AGGController {
    private final Frame frame;

    public AGGController(Frame frame) {
        this.frame = frame;
    }

    public void suspendFrame() {
        frame.copyActiveToSuspended();
        frame.clearActiveFrame();
    }
}

```

- `AGGController`クラスでは、`Frame`オブジェクトを介してフレームを操作します。
- これにより、`Frame`をモックに差し替えることでテストが容易になります。

---

### **テスト用のコード例**

テスト時に`Frame`クラスをモックとして利用する例です。

```java
public class MockFrame extends Frame {
    public MockFrame() {
        super(256); // テスト用の固定サイズ
    }

    @Override
    public void copyActiveToSuspended() {
        System.out.println("Mock copyActiveToSuspended called");
    }

    @Override
    public void clearActiveFrame() {
        System.out.println("Mock clearActiveFrame called");
    }
}

public class AGGControllerTest {
    public static void main(String[] args) {
        Frame mockFrame = new MockFrame();
        AGGController controller = new AGGController(mockFrame);

        controller.suspendFrame(); // テスト用のモックメソッドが呼び出される
    }
}

```

---

## **まとめ**

「グローバル参照のカプセル化」は、依存関係を明確にし、テスト可能性を向上させる強力な手法です。この手法を活用することで、コードの保守性が大幅に改善され、変更や拡張が容易になります。また、特に大規模プロジェクトでは、グローバル参照の管理が課題となるため、このようなカプセル化が設計上非常に重要です。
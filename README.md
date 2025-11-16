# 🟦 Battle Simulator 檢討檔

---

# ① buff_factory 重點整理

### 🔹 為何 buff_factory 必須「回傳一個函數」？

- buff 並不是「立即修改數值」  
- buff 是「一個未來用來修改數值的操作」  
- 因此 buff_factory 必須產生 **一個函數物件**  
- 之後才能被 compose_buffs 合併 → 最後 battle 時才執行  
- **本質名稱：函數工廠（function factory）**

---

### 🔹 為何 lambda 不能寫 `*args`？

- 我們必須回傳 `(atk, dfs, hp)` 三個值  
- `*args` 無法保證解構順序
- 簡單來說，輸入值須為解包狀態  
- buff function 必須明確定義：

```
lambda atk, dfs, hp: (atk + something, dfs, hp)
```
---
### 🔹 Functional Programming 核心概念（濃縮版）

| 原則 | 精簡描述 |
|------|----------|
| 函數也是資料 | 函數可以被存進變數、傳入參數、從函數回傳 |
| 無副作用 | 理想函數只接受輸入 → 回傳輸出，不修改外部狀態 |
| 可組合 | 複雜行為可由多個小函數組合（compose / pipe） |
| 延遲執行 | 函數本身代表「行為」，並不會在宣告時執行 |

---

### 🔹 lambda 與 def 「本質等價」

#### ✔ 兩者都會：

- 建立一個 **callable function object**
- 接收參數 → 回傳值
- 可以被指派給變數後名稱呼叫

#### ✔ 差異只有：

| 項目 | `def` | `lambda` |
|------|------|----------|
| 語法 | 多行 | 單行 |
| 是否匿名 | 有名稱 | 原本匿名，賦值後即變具名 |
| 可讀性 | 高 | 適合小型即時使用 |

---

### 🔹 等價示例

```python
# def 版本
def f(x, y):
    return x + y
```

```python
# lambda 版本（完全等價）
f = lambda x, y: x + y
```

✔ 兩者建立的 `f` 都是函數物件  
✔ `f(3,4)` → 都會回傳 `7`

---

### 🔹 為何 Functional Programming 愛用 lambda？

| 理由 | 說明 |
|------|------|
| 函數可直接內嵌 | 不必額外起名字 |
| 適合當回傳值 | 符合「行為即資料」模式 |
| 與 compose 呼應 | λ 表達式本來就是函數組合的語法起源 |

---

### 🔹 一句話總結

> **在 Python，`lambda` 與 `def` 產生的都是同一種「函數物件」，  
>   只是寫法不同。  
>   Functional Programming 的重點不是 lambda 本身，  
>   而是：把函數視為可組裝、可傳遞的資料。**

---
### 🔹 Decimal 相關結論

- Decimal 不能跟 float 混用  
- 若要使用 Decimal → **所有 hp/atk/dfs 都必須是 Decimal**  
- 題目不要求高精度 → 改回 float 是正確決策

---

# ② compose_buffs（完整 6 步對照表）

| ChatGPT 引導內容 | 你的濃縮回覆 | 為何正確（重寫註解） |
|------------------|-------------|----------------------|
| 1️⃣「你覺得 buff 是不是應該在讀取時就直接修改 atk/dfs？」 | 不應該，buff 要在 battle 過程中才真正生效 | ✔ 正確認知 buff 是「延遲效果」，不是初始化時直接修改資料 |
| 2️⃣「那 buff 應該存『數字』還是『操作』？」 | 應該存操作，也就是函數 | ✔ 這就是 functional programming：**把行為（function）當作資料保存** |
| 3️⃣「為什麼 compose 需要 closure 而不是 list？」 | closure 可以把 buff 函數序列包在一個 callable 裡，達到中間有形式輸入的可能 | ✔ closure = 函數保留外層變數，讓 buff pipeline 能被一次呼叫 |
| 4️⃣「你覺得多個 buff 應該怎麼依序合併？」 | 依序套用：上一個輸出變下一個輸入 | ✔ 這叫 **function pipeline**（函數管線），函數依序套用 |
| 5️⃣「那 lambda atk,dfs,hp: inner(...) 這層包裹意義是？」 | 讓 compose 回傳「可直接呼叫」的函數，而不是生資料 | ✔ functional style → factory function 必須 return **callable**，而不是 return 結果 |
| 6️⃣「buff pipeline 與 decimal 問題有衝突嗎？」 | 如果使用 Decimal → pipeline 中所有 buff 都得是 Decimal | ✔ 正確認知：functional pipeline *不會自動轉型* → 所有階段必須一致類型 |

---

# ③ Battle 規則完整整理

---

## 🔹 初始資料

每個角色有：

```
HP       → 血量
Attack   → 攻擊力
Defense  → 防禦力
```

---

## 🔹 Buff 資料

buff 有四種：

```
atk_add x  → attack += x
atk_mul x  → attack *= x
dfs_add x  → defense += x
dfs_mul x  → defense *= x
```

所有 buff **只會影響攻擊或防禦，沒有 HP buff**

---

## 🔹 Buff 應用規則

- buff **會在每一回合開始前套用一次**
- buff **會永久累積**（新增攻擊＝新的攻擊值參與下一輪運算）

---

## 🔹 戰鬥順序

1️⃣ **c1 先攻擊**  
2️⃣ 攻擊順序永遠是：  

```
回合 0：c1 攻擊 c2
回合 1：c2 攻擊 c1
回合 2：c1 攻擊 c2
回合 3：c2 攻擊 c1
⋯⋯
```

---

## 🔹 攻擊公式

```
damage = max(1 , attacker.attack - defender.defense)
defender.hp -= damage
```

➡ **至少扣 1，因此戰鬥必定結束**

---

## 🔹 勝利條件

當其中一人 `hp <= 0`  
➡ 立即回傳：

```
勝者名字 + 剩餘 HP（到小數點一位）
```

---

# ④ Error 是什麼？何時會發生？
  (1) infinte loop
      忘了函數呼叫要 c1.is_alive()
  (2) Runtime Error
      處理非預期指令，已達安全程式
---

##  Runtime Error = 程式執行期錯誤  
（不是邏輯錯誤，是 Python 直接崩潰）

在這題，真正 RE 來源：

```
buff_factory(cmd,x) 收到一個未定義 command
➡ return None
➡ compose_buffs 在 inner() 呼叫 None(...)
➡ TypeError: 'NoneType' object is not callable
```

⚠ **這是你看到 OJ 顯示「Runtime Error」的真實原因**

---

# ⑤ 何時需要防呆？如何補強？（條列）

---

## 🔹 為何需要？

- 測資不保證只會出現四種指令  
- 存在 `hp_mul` 的可能性（題目自己寫過一次示例）  
- **任何未知 buff 會造成 RE，不會造成 WA**

因此 ➜ **必須加防呆**

---

## 🔹 防呆程式碼（必須加入）

```python
else:
    return lambda atk, dfs, hp: (atk, dfs, hp)
```

➡ 意思：  
**「遇到未知 buff，就當它不存在」**

---

## 🔹 什麼情況必須做防呆？

| 來源 | 是否必須防呆？ | 原因 |
|------|---------------|------|
| 學術題目測資 100% 保證 | ❌ 可省略 |
| OJ / 助教出題 | ✔ 必須寫 | 題目文字與測資常不同步 |
| 真實工程 API | ✔ 必須寫 | 永遠不能信任輸入格式 |

---

# 🔚 完整 battle 程式碼（含防呆 buff_factory）

```python
class Character:
    def __init__(self, name, hp, attack, defense):
        self.name = name
        self.hp = hp
        self.attack = attack
        self.defense = defense
        self.buff_func = lambda atk, dfs, hp: (atk, dfs, hp)

    def apply_buffs(self):
        self.attack, self.defense, self.hp = self.buff_func(
            self.attack, self.defense, self.hp
        )

    def is_alive(self):
        return self.hp > 0

    def take_damage(self, amount):
        self.hp -= max(1, amount - self.defense)

    def attack_target(self, other):
        other.take_damage(self.attack)


def compose_buffs(*funcs):
    def inner(atk, dfs, hp):
        for f in funcs:
            atk, dfs, hp = f(atk, dfs, hp)
        return atk, dfs, hp

    return lambda atk, dfs, hp: inner(atk, dfs, hp)


def buff_factory(cmd, x):
    dx = float(x)
    if cmd == "atk_add":
        return lambda atk, dfs, hp: (atk + dx, dfs, hp)
    elif cmd == "atk_mul":
        return lambda atk, dfs, hp: (atk * dx, dfs, hp)
    elif cmd == "dfs_add":
        return lambda atk, dfs, hp: (atk, dfs + dx, hp)
    elif cmd == "dfs_mul":
        return lambda atk, dfs, hp: (atk, dfs * dx, hp)
    else:
        return lambda atk, dfs, hp: (atk, dfs, hp)   # 防呆
def battle(c1, c2):
    """
    #TODO
    Turn-based battle: c1 attacks first, alternate until one dies.
    returns the winner's name and hp
    
    """
       
       
    while True:
      
      c1.apply_buffs()
      c2.apply_buffs()
      
      c1.attack_target(c2)
      if not c2.is_alive():
        
        return c1.name , c1.hp
      c1.apply_buffs()
      c2.apply_buffs()
      
      c2.attack_target(c1)
      if not c1.is_alive():
        
        return c2.name , c2.hp
        
        
        
  
if __name__ == "__main__":
    # Input format:
    # name1 hp1 atk1 dfs1
    # name2 hp2 atk2 dfs2
    # n
    # Then n lines:
    # target cmd value
    # (target is either name1 or name2)
    n1, h1, a1, d1 = input().split()
    n2, h2, a2, d2 = input().split()
    c1 = Character(n1, int(h1), int(a1), int(d1))
    c2 = Character(n2, int(h2), int(a2), int(d2))
  
    n = int(input())
    buffs_c1 = []
    buffs_c2 = []
    for _ in range(n):
        target, cmd, val = input().split()
        buff = buff_factory(cmd, float(val))
        if target == c1.name:
            buffs_c1.append(buff)
        elif target == c2.name:
            buffs_c2.append(buff)
  
    c1.buff_func = compose_buffs(*buffs_c1)
    c2.buff_func = compose_buffs(*buffs_c2)
    # print(c1.buff_func)
    winner, hp_left = battle(c1, c2)
    print(f"{winner} wins with {hp_left:.1f} HP left.")
```

# Passband Modulation 与 Carrier

## 1. 为什么需要 Passband Modulation

课件在物理层里把 `passband modulation` 作为重点之一。

核心原因是：

- 有些传输媒介或系统并不适合直接传 baseband digital signal
- 这时需要借助一个 `carrier signal` 承载信息

也就是先有 carrier，再把 bit 信息“调制”到 carrier 上。

更准确地说：

- `baseband`：bit 直接映射成链路上的电平变化
- `passband`：bit 先映射到高频 carrier 的属性变化

常见 carrier 公式：

```text
s(t) = A cos(2πft + φ)
```

其中：

- `A` = amplitude
- `f` = frequency
- `φ` = phase

## 2. Carrier Signal 可以改什么

课件要求你知道 carrier 的三个典型可调元素：

- `Amplitude` 幅度
- `Frequency` 频率
- `Phase` 相位

这也是考试里“用图说明 carrier 的哪些元素可修改”的直接落点。

对应典型调制方式：

- `ASK`：改变幅度
- `FSK`：改变频率
- `PSK`：改变相位

## 3. 和 Baseband 的区别

### Baseband

- 更像直接在链路上传 bit 对应的信号变化
- 典型上下文常见于 NRZ、NRZI、Manchester

### Passband

- 不直接拿 bit 去驱动物理媒介
- 先准备 carrier，再调制 carrier

## 4. 常见 Baseband 编码

`NRZ`、`NRZI`、`Manchester` 都属于 `baseband line coding`。它们不是 passband modulation，但经常和 passband 一起在物理层考。

### 4.1 [[NRZ ]]

`NRZ (Non-Return to Zero)`：

- `1` 用固定高电平表示
- `0` 用固定低电平表示
- bit 中间不回到零

过程图：

```text
Bits: 1   0   1   1   0
NRZ : ___     ___ ___
         |___|       |___
```

判定规则：

- 看当前 bit 槽的电平
- 高电平 = `1`
- 低电平 = `0`

特点：

- 优点：简单
- 缺点：长串相同比特时缺少跳变，不利于 clock recovery

### 4.2 NRZI

`NRZI (Non-Return to Zero Inverted)` 常见约定：

- `1`：在 bit 开始时跳变
- `0`：在 bit 开始时不跳变

过程图，假设初始电平为低：

```text
Initial level: Low

Bits : | 1 | 0 | 1 | 1 | 0 |
NRZI :   ___ ___     ___ ___
        |       |___|       

```

步骤法：

```text
Initial level: Low

Bit 1 = 1 -> transition
Bit 2 = 0 -> no transition
Bit 3 = 1 -> transition
Bit 4 = 1 -> transition
Bit 5 = 0 -> no transition
```

判定规则：

- 看 bit 开始处有没有跳变
- 有跳变 = `1`
- 无跳变 = `0`

特点：

- 比 NRZ 更容易恢复时钟
- 但长串 `0` 仍可能没有跳变

### 4.3 Manchester

`Manchester encoding`：

- 每个 bit 中间一定有一次跳变
- 常见约定之一：
  - `0`：低到高(上升沿)
  - `1`：高到低(下降沿)

不同教材可能把 `0/1` 方向写反，考试时先声明采用的定义即可。

过程图：
![[Pasted image 20260427152633.png]]



判定规则：

- 看每个 bit 中间的跳变方向
- `low -> high` 和 `high -> low` 分别对应不同 bit 值

特点：

- 每位都有跳变，clock recovery 最好
- 带宽开销更大

### 4.4 三者对比

| 编码 | 1 的表示 | 0 的表示 | 是否依赖跳变 | 时钟恢复 |
|---|---|---|---|---|
| `NRZ` | 高电平 | 低电平 | 否 | 较差 |
| `NRZI` | 跳变 | 不跳变 | 是 | 一般 |
| `Manchester` | 中间跳变方向之一 | 中间跳变方向相反 | 是 | 最好 |

## 5. QAM 与 bits per symbol

`QAM = Quadrature Amplitude Modulation`

课件里的关键点：

- QAM varies amplitude and phase
- constellation diagram 用点表示不同 symbols
- symbols 越多，每个 symbol 可承载的 bits 越多

| Scheme | Symbols | Bits per symbol |
|---|---:|---:|
| BPSK | 2 | 1 |
| QPSK | 4 | 2 |
| QAM-16 | 16 | 4 |
| QAM-64 | 64 | 6 |

公式：

`bits per symbol = log2(symbol count)`

增加 bits per symbol 的意义：

- 同样 symbol rate 下可以传更多 bits
- 但 constellation 点更密，对噪声更敏感
- 通常需要更高信噪比才能可靠区分 symbols

## 6. 课件里的位置

- [[02 物理层]]
- `L5 The Physical Layer`
- `L21 Review`

## 7. 易错点辨析

- passband modulation 不只是“增强信号强度”
- carrier 不是只有 amplitude 一个维度可以改
- Manchester / NRZI / 4B5B 更偏编码与 clock recovery 语境，不等于 passband modulation 本身
- `NRZ` 是看电平，`NRZI` 是看是否跳变，`Manchester` 是看 bit 中间跳变方向

# Internet Checksum 与 Hamming Code

## 1. 为什么这两个常一起出现

因为它们都属于链路/网络通信中的差错处理主题，但功能并不一样：

- `Internet Checksum` 更偏差错检测
- `Hamming Code` 可以做到单比特纠错

对应章节：

- [[03 链路层]]

## 2. Internet Checksum

### 课件定义

课件指出：

- 使用 `1's complement arithmetic`
- checksum 是所有 16-bit words 的和做 1's complement 之后的结果

### 发送步骤

1. 把数据按 `16-bit words` 排列
2. checksum 位置先放 `0000`
3. 把所有 16-bit words 相加
4. 若有进位，回卷加到低位
5. 对结果取反，得到 checksum

### 接收步骤

1. 把收到的数据连同 checksum 一起相加
2. 处理回卷进位
3. 对结果取反
4. 若结果为 `0`，则校验通过

### 手算例题

例 1：

```text
words = 0001 f4a2 c3d2 a2a4 f8d6

0001 + f4a2 = f4a3
f4a3 + c3d2 = 1b875 -> wrap = b876
b876 + a2a4 = 15b1a -> wrap = 5b1b
5b1b + f8d6 = 153f1 -> wrap = 53f2

sum = 53f2
checksum = ~53f2 = ac0d
```

例 2：

```text
words = A9BC 45FE A9C5 53DF

A9BC + 45FE = EFBA
EFBA + A9C5 = 1997F -> wrap = 9980
9980 + 53DF = ED5F

sum = ED5F
checksum = ~ED5F = 12A0
```

例 3：

```text
words = 4500 003C 1234 5678 9ABC

4500 + 003C = 453C
453C + 1234 = 5770
5770 + 5678 = ADE8
ADE8 + 9ABC = 148A4 -> wrap = 48A5

sum = 48A5
checksum = ~48A5 = B75A
```

考试写法重点：

- 每一步保留 16-bit 十六进制结果。
- 只要超过 `FFFF`，就把最高位 carry 回卷加到低 16 位。
- 最后再取 one's complement。

### 课件强调的局限

- checksum 并不是无懈可击
- 某些对应的数值变化可能彼此抵消
- 它通常比 parity 强，但不如 CRC 强

## 3. Hamming Distance

回顾关系：

- 若 `HD = d + 1`，则最多检测 `d` 位错误
- 若 `HD = 2d + 1`，则最多纠正 `d` 位错误

### Code Space 例题

给定 codewords：

```text
01010010
10001011
11110011
00000000
```

pairwise distances：

| Pair | Distance |
|---|---:|
| `01010010` vs `10001011` | 5 |
| `01010010` vs `11110011` | 3 |
| `01010010` vs `00000000` | 3 |
| `10001011` vs `11110011` | 4 |
| `10001011` vs `00000000` | 4 |
| `11110011` vs `00000000` | 6 |

所以最小 Hamming Distance 是 `3`：

- 可检测 `2` 位错误：`HD = d + 1`
- 可纠正 `1` 位错误：`HD = 2d + 1`

再给一组常见题：

```text
1: 10101100
2: 11001101
3: 11100010
4: 10011011
5: 01110100
```

| Pair | Distance |
|---|---:|
| 1-2 | 3 |
| 1-3 | 4 |
| 1-4 | 5 |
| 1-5 | 4 |
| 2-3 | 5 |
| 2-4 | 4 |
| 2-5 | 5 |
| 3-4 | 5 |
| 3-5 | 4 |
| 4-5 | 7 |

## 4. Hamming Code

### 课件构造方法

对 `4 data bits`：

- 增加 `3 check bits`
- check bits 放在位置 `1, 2, 4`
- 形成 `7-bit code`

### 覆盖规则

- check 1 覆盖位置 `1, 3, 5, 7`
- check 2 覆盖位置 `2, 3, 6, 7`
- check 4 覆盖位置 `4, 5, 6, 7`

### 解码步骤

1. 接收端重新计算 check bits
2. 把结果写成一个二进制数 `syndrome`
3. 若 syndrome = `000`，表示无错
4. 若 syndrome 非 0，则其值就是出错 bit 的位置
5. 翻转该 bit，恢复原 codeword
6. 再抽出 data bits

### 7-bit 解码例题

约定：

- 位置从左到右编号为 `1..7`
- check bit 位置是 `1, 2, 4`
- data bit 位置是 `3, 5, 6, 7`
- 使用偶校验：每组 XOR 后应为 `0`

收到 `1010010`：

| Check | 覆盖位置 | XOR 结果 |
|---|---|---:|
| `p1` | `1,3,5,7` | 0 |
| `p2` | `2,3,6,7` | 0 |
| `p4` | `4,5,6,7` | 1 |

syndrome = `100₂ = 4`，第 4 位错，翻转后：

```text
1010010 -> 1011010
data positions 3,5,6,7 -> 1010
```

收到 `1110110`：

| Check | 覆盖位置 | XOR 结果 |
|---|---|---:|
| `p1` | `1,3,5,7` | 1 |
| `p2` | `2,3,6,7` | 1 |
| `p4` | `4,5,6,7` | 0 |

syndrome = `011₂ = 3`，第 3 位错，翻转后：

```text
1110110 -> 1100110
data positions 3,5,6,7 -> 0110
```

所以 received message `10100101110110` 可拆成：

```text
1010010 1110110 -> data = 1010 0110
```

另一个 received message：

```text
0111101 0101110 -> data = 1100 0010
```

## 5. 为什么 Hamming Code 能纠错

课件给出的核心直觉是：

- 若码字之间最小距离至少为 3
- 单 bit error 后，收到的序列仍会离原正确码字最近
- 因此可以映射回最近的合法码字

## 6. 易错点辨析

- checksum 和 CRC 不是同一种算法
- Hamming distance 是性质，不等于 Hamming code 本身
- Hamming code 的 syndrome 不是“随便写的校验值”，而是错误位置索引

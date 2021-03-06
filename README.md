# Mahjong

本项目为一个麻将服务器，供AI比赛使用。本文档分为两部分，第一部分为此服务器使用的[麻将规则](#规则)，第二部分为[交互规范](#交互规则)。编译后的服务器程序下载：

- [Mac](https://github.com/wormful/Mahjong/releases/download/v1.0.0/server_v1.0.0_Mac_OS_X)
- [Win_x86_64](https://github.com/wormful/Mahjong/releases/download/v1.0.0/server_v1.0.0_Win_x86_64.exe)
- [Ubuntu_x86_64](https://github.com/wormful/Mahjong/releases/download/v1.0.0/server_v1.0.0_Ubuntu_x86_64)

其他系统需要下载源码，使用 Rust Nightly 版本的编译器自行编译。

### 服务器使用说明

请在终端中打开服务器。用法：

```
# *nix
$ ./server_bin <AI1> <AI2> <AI3> <AI4> [-v] [-d]
```

```
# Windows
server_bin <AI1> <AI2> <AI3> <AI4> [-v] [-d]
```

开启`-d`选项后，服务器会在当前文件夹下的log文件夹中生成每盘的日志文件。

## 规则

为降低复杂度，本规则以国标麻将为基础进行了大量的简化并进行一定修改。

### 注意点

* 麻将牌共136张，即在国标麻将的基础上去掉了八张花牌。
* 一局定为4盘。去除了圈风、门风、庄家的概念。
* 没有牌城、掷骰与开牌的概念，AI拿到的牌全部直接由服务器发放。
* 出牌顺序按id为0、1、2、3，但一盘内第一个行牌的AI不确定。
* 整场比赛的第一盘时，第一个行牌的AI由服务器随机决定，从第二盘开始第一个行牌的AI为最近和牌的AI。
* 没有起和番数
* 轮到AI出牌的时候，AI必须在1秒内打出牌，否则以100毫秒1分（四舍五入）进行罚分。
* 一方打出牌后，AI若要吃牌、碰牌、杠牌或是和牌，需要在0.5秒内向服务器发出指令，超过0.5秒按每100毫秒1分（四舍五入）罚分。
* 测试共有6组，每组分为4局。不同组测试中AI相对位置不同。同组内四局使用的随机数种子不变，一局结束后每位AI的位置逆时针旋转一位。

### 番种

国标麻将的番种非常多，本规则将番种大大精简。

##### 88番

* 大四喜（不计碰碰和）
* 大三元（不计箭刻）
* 十三幺（不计五门齐、单钓将、门前清、混幺九）
* 绿一色（无发记清一色，*有发记混一色*）
* 四杠（不计碰碰和、三杠，暗杠另计）

##### 64番

* 小四喜
* 小三元（不计箭刻）
* 字一色（不计碰碰和）
* 四暗刻（不计门前清、碰碰和）
* 清幺九（不计碰碰和）

##### 48番

* 一色四同顺（不计一色三同顺、一般高）

##### 32番

* 三杠（暗杠另计）
* 混幺九（不计碰碰和）

##### 24番

* 七对（不计门前清、单钓将）
* 清一色
* 一色三同顺（不计一般高）

##### 16番

* 三同刻
* 三暗刻

##### 8番

* 三色三同顺（不计喜相逢）

##### 6番

* 碰碰和
* 混一色
* 五门齐

##### 2番

* 门前清（因取消了不求人，自摸和牌及和他家打出的牌都记）
* 断幺
* 平和
* 箭刻（可算多次）
* 暗杠（可算多次）

##### 1番

* 自摸
* 一般高（可算多次）
* 喜相逢（可算多次）
* 明杠（可算多次）
* 单钓将

### 记分规则

最开始每位AI的分数都是0分。底分为4分，基本分和牌后各个番种分数的总和。

若和牌方自摸，则另外三家都需付给和牌方（底分＋基本分）的分数。

否则，点炮方付给和牌方（底分＋基本分）的分数，其余两家付给和牌方底分的分数。

### 结束与排名

1. 当有任何一方AI程序意外退出时比赛结束，意外关闭的AI排名最后，其余AI按分数高低排名。
2. 当总共96盘牌局结束后比赛结束，AI按分数高低排名。

## 交互规则

服务器与AI通过标准输入和标准输出进行通信，每次通信的内容为一行文本，。

示例中的“标准输入”、“标准输出”都是以AI的角度上说的。“标准输入”中的内容即AI需要读取的内容，“标准输出”中的内容为AI需要打印的内容。

### 骨牌代号

* 字牌：东（E）、南（S）、西（W）、北（N）、中（Z）、发（F）、白（B）
* 万子（M）：1M、2M、3M、4M、5M、6M、7M、8M、9M
* 索子（S）：1S、2S、3S、4S、5S、6S、7S、8S、9S
* 筒子（T）：1T、2T、3T、4T、5T、6T、7T、8T、9T

~~PS：关于万的缩写的问题，是取自日语读音`man`…………………………~~

### 加入比赛

AI程序由服务器启动。AI先发送加入信息。

标准输出：

```
join
```

当四位AI都加入比赛之后，服务器会把id号发送给AI。

标准输入：

```
id 3
```

说明：3即为AI被分配到的id号。

### 开始比赛

每盘牌局开始时，服务器会广播第一个行牌的AI的id。

标准输入：

```
first 3
```

说明：id为3的AI为第一个行牌的AI。

然后每个AI会收到13张起手牌。

标准输入：

```
init 1S 1S 2T 5M 9S 4T W Z 1T 8M 9M 5S 6T
```

说明：起手13张牌分别为一索、一索、二筒、五万、九索、四筒、西风、红中、一筒、八万、九万、五索、六筒。

### 行牌

要注意行牌是有时间限制的。

##### 拿牌

到AI拿牌的时候，服务器会将拿牌信息发给AI，同时向其他AI广播。

标准输入（拿牌AI）：

```
pick 5T
```

说明：拿到的牌为五筒。

标准输入（其他AI）：

```
mpick 0
```

说明：id为0的AI拿牌了。

##### 出牌

出牌需要AI在1秒内发送指令给服务器，然后服务器将出牌信息广播给其他AI。

标准输出（出牌AI）：

```
out 9M
```

说明：需要打出的牌为九万。

标准输入（其他AI）：

```
mout 2 9M
```

说明：id为2的AI打出了一张九万。

这时三位AI有0.5秒的时间向服务器发出吃、碰、杠或和的指令。超过0.5秒按每100毫秒1分罚分。

如果不发出吃、碰、杠或和的指令的话：

标准输出：

```
pass
```

###### 吃/碰牌

由于吃的方式不唯一，所以吃的指令需要另外指定形成顺子中最小的一张牌。

标准输出（AI1）：

```
chi 7M
```

说明：报吃，形成七万、八万、九万一副顺子。
注1：吃不一定成功。
注2：如上例，即使吃的方式唯一，也需要另指定牌。

标准输出（AI2）：

```
peng
```

说明：报碰。

当吃和碰的指令都发送给服务器时，碰优先于吃。然后服务器会广播此消息。

标准输入（所有）：

```
mpeng 2 9M
```

说明：id为2的AI碰了九万。

吃没有成功（发送在`mpeng`/`mgang`消息之后）：

标准输入（AI1）：

```
mfail
```

若是吃成功的情况：

标准输入（所有）：

```
mchi 1 7M
```

说明：id为1的AI吃成功，形成七万、八万、九万一副顺子。

###### 大明杠

标准输出（AI3）：

```
gang
```

若杠成功：

标准输入（所有）：

```
mgang 3 5S
```

说明：id为3的AI明杠五索。

###### 暗杠

标准输出（AI3）：

```
agang W
```

标准输出（所有）：

```
magang 3
```

说明：id为3的AI暗杠了。

###### 加杠

标准输出：（AI3）：

```
jgang 5S
```

说明：需要加杠五索。

标准输入（所有）：

```
mjgang 3 5S
```

说明：id为3的AI加杠五索。

###### 抢杠和

当接到有别的AI加杠的消息时候，如要抢杠，需要在0.5秒内发出抢杠和指令。

标准输出：（AI2）：

```
qgang
```

抢杠成功的话，和牌结算同下。

###### 和牌

在出牌阶段和别家打完牌后都可以发出和牌指令：

标准输出（AI3）：

```
hu
```

当一方有效和牌后，会广播该AI和牌的番数。分数将自动扣除并加到和牌方。

当有人和牌或流局后即关闭所有AI程序。
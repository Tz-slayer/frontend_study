# 可倒放动效代码架构

## 目标

这个文档记录一种更容易维护的动效写法：让入场和退场复用同一套结构。

核心目标不是让所有动画机械倒放，而是让“首尾呼应”的元素能够用同一组状态表达：

- 入场：从隐藏态进入完成态
- 退场：从完成态回到隐藏态

这样可以避免在 `stage-name` 和 `stage-outro` 里重复写两套不一致的 `transform`。

## 核心原则

动画代码可以拆成三层：

1. 元素层：定义这个元素怎么动
2. 入场阶段：只把变量改成可见态
3. 退场阶段：只把变量改回隐藏态

推荐结构：

```css
/* 1. 元素层：定义动画能力和默认隐藏态 */
.line-top {
  --line-opacity: 0;
  --line-scale: 0;

  opacity: var(--line-opacity);
  transform-origin: center top;
  transform: translateY(-5px) scaleY(var(--line-scale));
  transition:
    opacity 520ms ease,
    transform 520ms ease;
}

/* 2. 入场阶段：只改变量 */
body.stage-name .line-top {
  --line-opacity: 1;
  --line-scale: 1;
}

/* 3. 退场阶段：回到同一组隐藏变量 */
body.stage-outro .line-top {
  --line-opacity: 0;
  --line-scale: 0;
}
```

这里真正的视觉逻辑只写在 `.line-top` 上。阶段类只负责声明“现在应该是什么状态”。

## 为什么用变量

如果不用变量，代码容易变成这样：

```css
body.is-preload .line-top {
  opacity: 0;
  transform: translateY(-5px) scaleY(0);
}

body.stage-name .line-top {
  opacity: 1;
  transform: translateY(-5px) scaleY(1);
}

body.stage-outro .line-top {
  opacity: 0;
  transform: translateY(-5px) scaleY(0);
}
```

这种写法能工作，但问题是：

- `translateY(-5px)` 被写了三次
- 入场和退场的目标态容易不小心写偏
- 后续调整一个元素时，要同时检查多个阶段
- 文件变大后，很难看出哪些是元素能力，哪些是阶段状态

变量写法可以把重复部分收回元素本身：

```css
.line-top {
  opacity: var(--line-opacity, 0);
  transform: translateY(-5px) scaleY(var(--line-scale, 0));
}
```

阶段只负责改：

```css
body.stage-name .line-top {
  --line-opacity: 1;
  --line-scale: 1;
}

body.stage-outro .line-top {
  --line-opacity: 0;
  --line-scale: 0;
}
```

## 横向线条的倒放

如果线条入场是从左到右展开，可以这样写：

```css
.line-horizontal {
  --line-opacity: 0;
  --line-scale-x: 0;

  opacity: var(--line-opacity);
  transform-origin: left center;
  transform: scaleX(var(--line-scale-x));
  transition:
    opacity 520ms ease,
    transform 520ms ease;
}

body.stage-enter .line-horizontal {
  --line-opacity: 1;
  --line-scale-x: 1;
}

body.stage-outro .line-horizontal {
  --line-opacity: 0;
  --line-scale-x: 0;
}
```

这个退场效果会表现为：右边缘向左收回。

关键点是：不要在退场时把 `transform-origin` 改成 `right center`。
入场和退场使用同一个 `transform-origin`，退场才会像入场的倒放。

## 竖向线条的倒放

当前页面里的名字标记更接近竖向线条。

上方线条从上方 `dot` 向下生长：

```css
.line-top {
  --line-opacity: 0;
  --line-scale-y: 0;

  opacity: var(--line-opacity);
  transform-origin: center top;
  transform: translateY(-5px) scaleY(var(--line-scale-y));
}
```

入场：

```css
body.stage-name .line-top {
  --line-opacity: 1;
  --line-scale-y: 1;
}
```

退场：

```css
body.stage-outro .line-top {
  --line-opacity: 0;
  --line-scale-y: 0;
}
```

因为 `transform-origin` 是 `center top`，所以退场时线条会从下往上收回到上方 `dot`。

下方线条则相反：

```css
.line-bottom {
  --line-opacity: 0;
  --line-scale-y: 0;

  opacity: var(--line-opacity);
  transform-origin: center bottom;
  transform: translateY(5px) scaleY(var(--line-scale-y));
}
```

因为 `transform-origin` 是 `center bottom`，所以退场时线条会从上往下收回到下方 `dot`。

## 位移元素的倒放

对于 `frame`、`crown` 这种有位移和旋转的元素，也可以用变量表达隐藏态和完成态。

示例：

```css
.frame {
  --frame-opacity: 0;
  --frame-y: 18px;
  --frame-scale: 0.98;

  opacity: var(--frame-opacity);
  transform: translateY(var(--frame-y)) scale(var(--frame-scale));
  transition:
    opacity 700ms ease,
    transform 700ms ease;
}

body.stage-frame .frame {
  --frame-opacity: 1;
  --frame-y: 0px;
  --frame-scale: 1;
}

body.stage-outro .frame {
  --frame-opacity: 0;
  --frame-y: 18px;
  --frame-scale: 0.98;
}
```

这样 `frame` 的入场和退场会自然互为反向。

## 延迟也应该分层管理

变量主要解决“状态复用”，但时间节奏仍然要单独控制。

推荐继续把阶段时间写在阶段规则里：

```css
body.stage-name .line-top {
  --line-opacity: 1;
  --line-scale-y: 1;
  transition-delay: 580ms;
}

body.stage-outro .line-top {
  --line-opacity: 0;
  --line-scale-y: 0;
  transition-delay: 120ms;
}
```

也可以把延迟抽成变量：

```css
.line-top {
  transition-delay: var(--line-delay, 0ms);
}

body.stage-name .line-top {
  --line-delay: 580ms;
}

body.stage-outro .line-top {
  --line-delay: 120ms;
}
```

如果一个元素的入场和退场都需要频繁调时，使用延迟变量会更清晰。

## 什么时候不要强行倒放

不是所有动画都适合严格倒放。

适合倒放的元素：

- 线条延展
- 点的缩放
- frame 的位移显隐
- 皇冠落位和离场
- 背景层渐显渐隐

不一定适合倒放的元素：

- 雷电击中效果
- 文字被点亮的瞬间
- 人物从皇冠中释放的遮罩动效
- 有冲击感、触发感、仪式感的瞬时特效

这些元素可以保持“首尾呼应”，但不一定要严格 `reverse`。例如雷电入场是劈下，退场可以是能量熄灭，而不是把雷电倒着收回去。

## 推荐落地规则

后续修改动画时，优先按这个顺序组织代码：

1. 在元素选择器里定义默认变量、`transition`、`transform-origin`、最终 `transform` 结构
2. 在入场阶段里只修改变量和 `transition-delay`
3. 在退场阶段里把变量改回隐藏态
4. 只有一次性冲击特效才使用 `@keyframes`
5. 如果入场和退场状态不一致，优先检查变量是否没有复用同一套隐藏态

判断一段动画代码是否清晰，可以看这个标准：

- 元素本体能不能看出“它怎么动”
- 阶段规则能不能看出“它什么时候变成什么状态”
- 退场规则是不是能明显看出“回到入场初始态”

如果三点都满足，这段动画就比较容易继续维护。

# Markdown 小记


> 创建于\_2020-02-29 00:56 by uchin  
> 修改于\_2020-08-01 19:00 by uchin  
> 修改于\_2020-12-13 21:27 by uchin

## 基本语法

语法参考：<https://www.markdown.xyz/basic-syntax/>

### 插入表格

```markdown
| Title1 | Title2 |
| ------ | ------ |
|        |        |
```

### 插入链接

```markdown
[引用标题](引用地址)

快速插入链接  
<https://www.runoob.com>

注脚[^01]
[01]: https://www.runoob.com "菜鸟"

引用式链接 [网址变量][02]
[02]: https://www.runoob.com

<!-- 上标注脚，typora 支持 -->

10[^x]

<a href="#bottom">去底部</a>
```

### 代码块

````markdown
```Java
public class Main {
    public static void main(String[] args) {
        System.out.println("hello");
    }
}
```
````

### TODO list

```markdown
- [ ] 任务清单
- [x] vs
```

### 注释

```markdown
<!-- 这是注释 -->
```

### 其他

```markdown
<!-- 下标 -->

H~2~O~2~

<u>下划线</u>

~~划掉内容~~
```

### 插入公式

1. 行内公式语法：

   ```markdown
   $公式内容$
   ```

2. 块公式语法：

   ```markdown
   \$\$
   \sum\_{i=0}^{\infty}
   \$\$
   ```

## 公式符号对照表

### 上下标

| 算式                | Markdown           |
| :------------------ | :----------------- |
| $x^2 $              | x^2                |
| $y_1 $              | y_1                |
| $x_{k+\frac{1}{2}}$ | x\_{k+\frac{1}{2}} |

### 分式

| 算式          | Markdown    |
| :------------ | :---------- |
| $1/2$         | 1/2         |
| $\frac{1}{2}$ | \frac{1}{2} |

### 省略号

| 省略号   | Markdown |
| :------- | :------- |
| $\cdots$ | \cdots   |

### 开根号

| 算式       | Markdown |
| :--------- | :------- |
| $\sqrt{2}$ | \sqrt{2} |

### 矢量

| 算式      | Markdown |
| :-------- | :------- |
| $\vec{a}$ | \vec{a}  |

### 积分

| 算式                | Markdown           |
| :------------------ | :----------------- |
| $\int{x}dx$         | \int{x}dx          |
| $\int_{1}^{2}{x}dx$ | \int\_{1}^{2}{x}dx |

### 极限

| 算式                        | Markdown                    |
| :-------------------------- | :-------------------------- |
| $\lim{a+b}$                 | \lim{a+b}                   |
| $lim_{n\rightarrow+\infty}$ | \lim\_{n\rightarrow+\infty} |

### 累加

| 算式                                  | Markdown                             |
| :------------------------------------ | :----------------------------------- |
| $\sum_{i=1}^{\rightarrow +\infty}{a}$ | \sum\_{i=1}^{\rightarrow +\infty}{a} |
| $\sum_{n=1}^{100}{a_n}$               | \sum\_{n=1}^{100}{a_n}               |

### 累乘

| 算式                    | Markdown               |
| :---------------------- | :--------------------- |
| $\prod{x}$              | \prod{x}               |
| $\prod_{n=1}^{99}{x_n}$ | \prod\_{n=1}^{99}{x_n} |

### 希腊字母

| 大写       | Markdown | 小写          | Markdown    |
| :--------- | :------- | :------------ | :---------- |
| $A$        | A        | $\alpha$      | \alpha      |
| $B$        | B        | $\beta$       | \beta       |
| $\Gamma$   | \Gamma   | $\gamma$      | \gamma      |
| $\Delta$   | \Delta   | $\delta$      | \delta      |
| $E$        | E        | $\epsilon$    | \epsilon    |
|            |          | $\varepsilon$ | \varepsilon |
| $Z$        | Z        | $\zeta$       | \zeta       |
| $H$        | H        | $\eta$        | \eta        |
| $\Theta$   | \Theta   | $\theta$      | \theta      |
| $I$        | I        | $\iota$       | \iota       |
| $K$        | K        | $\kappa$      | \kappa      |
| $\Lambda$  | \Lambda  | $\lambda$     | \lambda     |
| $M$        | M        | $\mu$         | \mu         |
| $N$        | N        | $\nu$         | \nu         |
| $\Xi$      | \Xi      | $\xi$         | \xi         |
| $O$        | O        | $\omicron$    | \omicron    |
| $\Pi$      | \Pi      | $\pi$         | \pi         |
| $P$        | P        | $\rho$        | \rho        |
| $\Sigma$   | \Sigma   | $\sigma$      | \sigma      |
| $T$        | T        | $\tau$        | \tau        |
| $\Upsilon$ | \Upsilon | $\upsilon$    | \upsilon    |
| $\Phi$     | \Phi     | $\phi$        | \phi        |
|            |          | $\varphi$     | \varphi     |
| $X$        | X        | $\chi$        | \chi        |
| $\Psi$     | \Psi     | $\psi$        | \psi        |
| $\Omega$   | \Omega   | $\omega$      | \omega      |

### 三角函数

| 三角函数 | Markdown |
| :------- | :------- |
| $\sin$   | \sin     |

### 对数函数

| 算式       | Markdown |
| :--------- | :------- |
| $ln2$      | \ln2     |
| $\log_2 8$ | \log_28  |
| $\lg 10$   | \lg10    |

### 关系运算符

| 运算符    | Markdown |
| :-------- | :------- |
| $\pm$     | \pm      |
| $\times$  | \times   |
| $\cdot$   | \cdot    |
| $\div$    | \div     |
| $\neq$    | \neq     |
| $\equiv$  | \equiv   |
| $\leq$    | \leq     |
| $\geq$    | \geq     |
| $\approx$ | \approx  |

### 其它特殊字符

| 符号         | Markdown   |
| :----------- | :--------- |
| $\forall$    | \forall    |
| $\infty$     | \infty     |
| $\emptyset$  | \emptyset  |
| $\exists$    | \exists    |
| $\nabla$     | \nabla     |
| $\bot$       | \bot       |
| $\angle$     | \angle     |
| $\because$   | \because   |
| $\therefore$ | \therefore |

### 大括号

```markdown
$$
c(u)=\begin{cases}

\sqrt\frac{1}{N}，u=0\\
\sqrt\frac{2}{N}，u\neq0

\end{cases}
$$
```

### 矩阵

```markdown
$$
 a =
 \left[
 \matrix{
 \alpha_1 & test1\\
 \alpha_2 & test2\\
 \alpha_3 & test3
 }
 \right]
$$
```

### 空格

`$a\quad b$`

<font color='red'>注意：行内公式属于 LaTeX 扩展语法，并非 Markdown 的通用标准，需要在 Typora 的“文件”-“偏好设置”中，勾选“内联公式”一项，Typora 才会予以解析。**在很多其他的 Markdown 编辑器中很可能还是不认识无法解析**</font>

## 换行问题

typora 换行问题（以下 typora 均在所见即所得模式 下操作，源码模式下和一般 markdown 编辑器相同）

1. 软换行
   所谓软换行就是将其解析成空格的换行，在一般 markdown 编辑器里点击以此 enter 就是软换行，而在 typora 里是 shift+enter 是软换行

2. 硬换行
   硬换行就是在解析后真正有换行效果的换行，在一般 markdown 里是末尾两个空格之后再敲击回车达到硬换行效果

3. 换段
   在一般 markdown 编辑器里，段落之间使用一行空行进行区分，因此需要在某段落结尾（含两个空格，事实上不加两个空格换段效果也会有）连续敲击两次回车
   在 typora 中，敲击以此回车就是换段，即做了两次软换行 shift+enter

---

参考：

- [LaTeX 数学符号大全](https://www.moonpapers.com/blog/5f9bc075e342e0f73c0a49bc)
- [LaTeX 各种命令，符号](https://blog.csdn.net/garfielder007/article/details/51646604)
- [Markdown 基本语法](https://www.markdown.xyz/basic-syntax/)
- [在 VSCode 中高效编辑 markdown 的正确方式](https://www.thisfaner.com/p/edit-markdown-efficiently-in-vscode/)


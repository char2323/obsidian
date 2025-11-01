### 🧩 一、文本模式常用符号

|      显示效果      | 输入命令             | 说明            |
| :------------: | :--------------- | :------------ |
|       $        | `\$`             | 美元符号          |
|       %        | `\%`             | 百分号           |
|       _        | `\_`             | 下划线（需转义）      |
|       #        | `\#`             | 井号            |
|       &        | `\&`             | 与号            |
|       {        | `\{`             | 左花括号          |
|       }        | `\}`             | 右花括号          |
| \textbackslash | `\textbackslash` | 反斜杠           |
|     ``…''      | ````text''``     | 英文引号（左双右双）    |
|      `--`      | `--`             | 短破折号（en dash） |
|     `---`      | `---`            | 长破折号（em dash） |
|    `\LaTeX`    | `\LaTeX`         | LaTeX 标志      |
|     `\TeX`     | `\TeX`           | TeX 标志        |

---

### 🔢 二、数学模式常用符号

> 使用方式：放在 `$...$` 或 `\[...\]` 中

#### 1. 希腊字母

|显示|命令|显示|命令|
|:-:|:--|:-:|:--|
|$\alpha$|`\alpha`|$\beta$|`\beta`|
|$\gamma$|`\gamma`|$\Gamma$|`\Gamma`|
|$\delta$|`\delta`|$\Delta$|`\Delta`|
|$\epsilon$|`\epsilon`|$\varepsilon$|`\varepsilon`|
|$\theta$|`\theta`|$\Theta$|`\Theta`|
|$\lambda$|`\lambda`|$\Lambda$|`\Lambda`|
|$\mu$|`\mu`|$\nu$|`\nu`|
|$\pi$|`\pi`|$\Pi$|`\Pi`|
|$\rho$|`\rho`|$\sigma$|`\sigma`|
|$\Sigma$|`\Sigma`|$\phi$|`\phi`|
|$\Phi$|`\Phi`|$\omega$|`\omega`|
|$\Omega$|`\Omega`|||

---

#### 2. 上下标与运算符

|显示|命令|说明|
|:-:|:--|:--|
|$a^2$|`a^2`|上标|
|$a_i$|`a_i`|下标|
|$x^{n+1}$|`x^{n+1}`|复合上标|
|$\sum_{i=1}^{n} i$|`\sum_{i=1}^{n} i`|求和符号|
|$\prod_{i=1}^{n} i$|`\prod_{i=1}^{n} i`|连乘符号|
|$\int_a^b f(x),dx$|`\int_a^b f(x)\,dx`|积分符号|
|$\lim_{x\to 0}$|`\lim_{x\to 0}`|极限符号|
|$\frac{a}{b}$|`\frac{a}{b}`|分数|
|$\sqrt{x}$|`\sqrt{x}`|平方根|
|$\sqrt[n]{x}$|`\sqrt[n]{x}`|n 次方根|

---

#### 3. 关系符与运算符

|显示|命令|显示|命令|
|:-:|:--|:-:|:--|
|$=$|`=`|$\neq$|`\neq`|
|$\approx$|`\approx`|$\equiv$|`\equiv`|
|$<$|`<`|$>$|`>`|
|$\leq$|`\leq`|$\geq$|`\geq`|
|$\pm$|`\pm`|$\times$|`\times`|
|$\div$|`\div`|$\cdot$|`\cdot`|
|$\propto$|`\propto`|$\infty$|`\infty`|

---

#### 4. 集合与逻辑符号

|显示|命令|显示|命令|
|:-:|:--|:-:|:--|
|$\in$|`\in`|$\notin$|`\notin`|
|$\subset$|`\subset`|$\subseteq$|`\subseteq`|
|$\supset$|`\supset`|$\cup$|`\cup`|
|$\cap$|`\cap`|$\emptyset$|`\emptyset`|
|$\forall$|`\forall`|$\exists$|`\exists`|
|$\neg$|`\neg`|$\Rightarrow$|`\Rightarrow`|
|$\Leftrightarrow$|`\Leftrightarrow`|$\therefore$|`\therefore`|

---

#### 5. 其他常见符号

|显示|命令|说明|
|:-:|:--|:--|
|$\ldots$|`\ldots`|省略号|
|$\cdots$|`\cdots`|居中省略号|
|$\overline{AB}$|`\overline{AB}`|上划线|
|$\underline{AB}$|`\underline{AB}`|下划线|
|$\vec{v}$|`\vec{v}`|向量符号|
|$\hat{x}$|`\hat{x}`|单位向量/估计符号|
|$\bar{x}$|`\bar{x}`|均值/上横线|
|$\angle ABC$|`\angle ABC`|角符号|
|$\degree$|`\degree`|度（需 `gensymb` 宏包）|

---

要不要我帮你生成一个可以直接放进 LaTeX 文档的表格版本（带 `\begin{tabular}` 环境）？这样你可以直接复制粘贴进 `.tex` 文件里。
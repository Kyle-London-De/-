# Frenet-Serret 标架

![](D:/Obsidian/picture/Frenet-Serret-1.jpg)

由图1可得：
$$
\Delta s\approx|\Delta \vec{r}|=|\vec{r}(t+\Delta t)-\vec{r}(t)|
$$
有泰勒公式：
$$
f(x)=f(x_0)+f'(x_0)(x-x_0)+\frac{f''(x_0)(x-x_0)^2}{2!}+\dots+\frac{f^{(n)}(x_0)(x-x_0)^n}{n!}+o(x-x_0)^n
$$
由泰勒公式展开：
$$
\Delta s\approx|\Delta \vec{r}|=|\vec{r}(t+\Delta t)-\vec{r}(t)|=\left|\frac{d\vec{r}}{dt}\Delta t+\frac{1}{2}\frac{d^2\vec{r}}{dt^2}(\Delta t)^2\right|\approx\left|\frac{d\vec{r}}{dt}\right|\Delta t
$$

当 $\Delta s\to 0$ 时，$\Delta s$ 可由 $ds$ 表示，$\Delta t$ 可由 $dt$ 表示：
$$
ds=\left|\frac{d\vec{r}}{dt}\right|dt=\left|\dot{\vec{r}}\right|dt\quad\Longrightarrow\quad\frac{ds}{dt}=\left|\dot{\vec{r}}\right|
$$
引入单位切向量 $\vec{t}$ :
$$
\vec{t}=\frac{\dot{\vec{r}}}{\left|\dot{\vec{r}}\right|}=\frac{d\vec{r}}{dt}/\frac{ds}{dt}=\frac{d\vec{r}}{ds}=\dot{\vec{r}}(s)
$$
又有：
$$
\ddot{\vec{r}}(s)=\lim_{\Delta s\to0}\frac{\dot{\vec{r}}(s+\Delta s)-\dot{\vec{r}}(s)}{\Delta s}
$$
而图1(b)中，当 $\Delta s\to0$ 时，$\dot{\vec{r}}(s+\Delta s)-\dot{\vec{r}}(s)$ 与 $\vec{t}$ 趋近于正交，因而引入单位主法向量 $\vec{n}$ :
$$
\vec{n}=\frac{\ddot{\vec{r}}(s)}{\left|\ddot{\vec{r}}(s)\right|}=\frac{\dot{\vec{t}}(s)}{\left|\dot{\vec{t}}(s)\right|}
$$
再由右手定则引入单位副法向量 $\vec{b}$ :
$$
\vec{b}=\vec{t}\times\vec{n}
$$
因而得到三个单位向量分别为：
$$
\begin{cases}
单位切向量:\vec{t}=\frac{d\vec{r}}{ds}=\dot{\vec{r}}(s)\\
单位主法向量:\vec{n}=\frac{\ddot{\vec{r}}(s)}{\left|\ddot{\vec{r}}(s)\right|}=\frac{\dot{\vec{t}}(s)}{\left|\dot{\vec{t}}(s)\right|}\\
单位副法向量:\vec{b}=\vec{t}\times\vec{n}
\end{cases}
$$
其空间关系如下图所示：

![](F:/Obsidian/picture/Frenet-Serret-2.jpg)

单位切向量 $\vec{t}$ 总是与曲线相切，单位主法向量 $\vec{n}$ 总是指向该处曲率圆的圆心，单位副法向量 $\vec{b}$ 由 $\vec{t}\times\vec{n}$ 右手定则确定。
$\vec{n}$ 与 $\vec{b}$ 确定正交平面，曲线垂直于正交平面，$\vec{n}$ 与 $\vec{t}$ 确定密切平面，也是曲率圆所在平面，$\vec{b}$ 与 $\vec{t}$ 确定从切平面，垂直于其余两平面。
# Frenet-Serret 公式

![](F:/Obsidian/picture/Frenet-Serret-3.jpg)

由于 $\vec{t}=\dot{\vec{r}}(s)\approx\dot{\vec{r}}(s+\Delta s)$ 为单位向量，结合图1(b)可知：
$$
\left|\dot{\vec{r}}(s)-\dot{\vec{r}}(s+\Delta s)\right|=\Delta\theta\cdot1
$$
同时：
$$
\ddot{\vec{r}}(s)=\lim_{\Delta s\to0}\frac{\dot{\vec{r}}(s+\Delta s)-\dot{\vec{r}}(s)}{\Delta s}
$$
两边取模：
$$
\left|\ddot{\vec{r}}(s)\right|=\lim_{\Delta s\to0}\frac{\left|\dot{\vec{r}}(s+\Delta s)-\dot{\vec{r}}(s)\right|}{\Delta s}
$$
两式相消可得：
$$
\left|\ddot{\vec{r}}(s)\right|=\lim_{\Delta s\to 0}\frac{\Delta\theta}{\Delta s}=\lim_{\Delta s\to 0}\frac{\Delta\theta}{\rho\Delta\theta}=\frac{1}{\rho}=\kappa
$$
其中 $\rho$ 为曲率半径，$\kappa$ 为曲率。
可得到单位切向量关于弧长的微分：
$$
\frac{d\vec{t}}{ds}=\dot{\vec{t}}(s)=\left|\dot{\vec{t}}(s)\right|\vec{n}=\left|\ddot{\vec{r}}(s)\right|\vec{n}=\kappa\vec{n}
$$
单位副法向量关于弧长的微分：
$$
\frac{d\vec{b}}{ds}=\frac{d}{ds}(\vec{t}\times\vec{n})=\frac{d\vec{t}}{ds}\times\vec{n}+\vec{t}\times\frac{d\vec{n}}{ds}=\kappa\vec{n}\times\vec{n}+\vec{t}\times\dot{\vec{n}}(s)=\vec{t}\times\dot{\vec{n}}(s)
$$
又因为 $\vec{n}^2=1\Rightarrow\vec{n}\dot{\vec{n}}(s)=0$ ，故 $\vec{n}$ 与 $\dot{\vec{n}}(s)$ 垂直，$\dot{\vec{n}}(s)$ 位于 $\vec{b}$ 与 $\vec{t}$ 确定的从切平面内，可表示为：
$$
\dot{\vec{n}}(s)=\mu\vec{t}+\tau\vec{b}
$$
则有：
$$
\frac{d\vec{b}}{ds}=\vec{t}\times\dot{\vec{n}}(s)=\mu\vec{t}\times\vec{t}+\tau\vec{t}\times\vec{b}=-\tau\vec{n}
$$
单位主法向量关于弧长的微分：
$$
\frac{d\vec{n}}{ds}=\frac{d(\vec{b}\times\vec{t})}{ds}=-\tau\vec{n}\times\vec{t}+\kappa\vec{b}\times\vec{n}=-\kappa\vec{t}+\tau\vec{b}
$$
因此三个单位向量关于弧长的微分可表示为：
$$
\begin{cases}
\frac{d\vec{t}}{ds}=\kappa\vec{n}\\
\frac{d\vec{n}}{ds}=-\kappa\vec{t}+\tau\vec{b}\\
\frac{d\vec{b}}{ds}=-\tau\vec{n}
\end{cases}
$$
其中，$\kappa$ 为曲线的曲率，其绝对值度量了曲线在从切平面垂直方向的变化率，$\tau$ 为曲线的挠率，其绝对值度量了曲线在密切平面垂直方向的变化率。
整理后 Frenet-Serret 公式可写成：
$$
\begin{bmatrix}\dot{\vec{t}}\\\dot{\vec{n}}\\\dot{\vec{b}}\end{bmatrix}=\begin{bmatrix}0&\kappa&0\\-\kappa&0&\tau\\0&-\tau&0\end{bmatrix}\cdot\begin{bmatrix}\vec{t}\\\vec{n}\\\vec{b}\end{bmatrix}
$$

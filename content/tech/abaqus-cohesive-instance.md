---
title: "ABAQUS Cohesive 单元失效分析实例"
date: 2020-04-26T22:00:46+08:00
draft: false
align: normal
slug: abaqus-cohesive-instance
tags: ["ABAQUS","adhesive", "academic"]
indent: false
dropCap: false
comments: true

toc: true
---


本文是最近学习使用 ABAQUS 的 Cohesive 单元模拟胶接连接失效的简单算例，主要记录 CAE 软件的操作步骤。

<!--more-->

## ABAQUS Cohesive失效分析的两种方法

1. 建立 Cohesive 单元，给单元赋 Cohesive 属性；
2. 使用 Cohesive 接触。

## Cohesive 单元分析实例

### 模型描述

两块金属板胶接连接，在法向作用力下分开。

金属板尺寸：10mm&times;10mm&times;1mm

胶层厚度 0.1mm

材料属性：

* 不锈钢：*E*=199GPa, <i>&nu;</i>=0.3
* EC3448 胶：*E*=1GPa，<i>&nu;</i>=0.3，*G*=385MPa，[<i>&sigma;</i>]=6.8MPa，[<i>&tau;</i>]=35MPa

单位制：N, mm, MPa

### 建立模型(线性刚度折减模型)

1. 从 3D shell 建立一个 part，划分网格；

2. 从网格创建网格部件（Mesh→Create Mesh Part）;

3. 编辑网格（Mesh→Edit Mesh），用 Offset 方法创建实体层（create solid layers）。选取相应的网格和偏移方向，设置厚度（Thickness）1 和层数（Number of Layers）1，勾选「删除基础壳单元」（Delete base shell elements），从而创建出金属板底板；

4. 用同样的 Offset 方式创建出胶层（Thickness=0，勾选 Share nodes with base shell/surface）和金属板顶板（Thickness=1）；

5. 赋单元属性（Assign Element Type）：金属板使用 C3D8R 单元，胶层使用 COH3D8 单元；

6. 创建材料：金属板创建线弹性各项同性材料（Mechanical→Elasticity→Elastic, Type: Isotropic），设置 E=198e3 和 &nu;=0.3；
   胶层 Type: Traction，设置材料参数 *E<sub>nn</sub>*=1000, *E<sub>ss</sub>*=385, *E<sub>tt</sub>*=385；
   设置胶层二次名义应力准则（Mechanical→Damage for Traction Separation Laws→Quads Damage），破坏参数（Nominal Stress Normal-only Mode,Nominal Stress First Direction, Nominal Stress Second Direction ）*t<sub>n</sub>*=6.8, *t<sub>s</sub>*= 35, *t<sub>t</sub>*=35；
   设置损伤演化方式（Suboptions→Damage Evolution），Type 选择 Displacement，软化方式（Softening）选择 Linear，失效位移（Displacement at Failure）0.001；

   二次应力准则：

   $$
    \left \lbrace \frac{<t_n>}{t^0_n}  \right \rbrace ^2 + \left \lbrace \frac{t_s}{t^0_s}  \right \rbrace ^2 + \left \lbrace \frac{t_t}{t^0_t} \right \rbrace ^2 = 0
   $$

7. 创建截面：金属板 Category=Solid，Type=Homogeneous，材料选择金属的材料；胶层 Category=Other，Type=Cohesive，材料选择胶层材料，Response=Traction Separation，**Initial thickness**=0.1；分别给金属板和胶层赋予截面属性。

8. 装配：在金属板顶面和底面中心各创建一个参考点（Reference Point），参考点和金属板的单元创建刚体约束（Rigid Body）；

9. 创建分析步：设置最大迭代数（Maximum number of increments）1000，步长（Increment size）设置，初始步长（Initial）0.01，最小步长（Minimum）1E-015，最大步长（Maximum）0.01；

10. 设置输出变量，Field Output Requests 输出 E, S, SDEG, U；History Output Requests 输出两个参考点的 U3 和 RF3；

11. 创建边界条件：Initial Step 中选择金属底板的参考点设置 ENCASTRE 约束（Mechanical→Symmetry→ENCASTRE）；Step-1 选择金属顶板的参考点设置 Z 方向位移约束 U<sub>3</sub>=0.002；

12. 提交分析；

13. 结果后处理，通过 Create XY Data 分别绘制顶板参考点的支反力-位移曲线，Cohesive 单元的应力-应变曲线、应力-位移曲线和应变-位移曲线。

### 仿真结果

#### 支反力-位移曲线
<img src="/abaqus-cohesive-instance/F-U.png" width="500"  title="F-U 曲线">

#### 应力-应变曲线

<img src="/abaqus-cohesive-instance/S-E.png" width="500"  title="S-E 曲线">

#### 应力，应变-位移曲线

<img src="/abaqus-cohesive-instance/S,E-U.png" width="500"  title="S,E-U 曲线">

### 结果分析

#### 理论计算

胶层只受法向拉伸载荷，开始失效时的应变如下

<div>
$$
\begin{aligned}
\varepsilon _{f0} &= [\sigma]/E \\
&= 6.8 \mathrm{MPa} / 1 \mathrm{GPa} \\
&= 0.0068
\end{aligned}
$$
</div>
破坏载荷为

<div>
$$
\begin{aligned}
F_{f0} &= [\sigma]\cdot A \\
&= 6.8 \mathrm{MPa} \times 1000 \mathrm{mm} ^2 \\
&= 680 \mathrm N
\end{aligned}
$$
</div>
开始失效时胶层上下表面的相对位移为
<div>
$$
\begin{aligned}
U_{f0} &= \varepsilon _{f0} \cdot t \\
&= 0.0068 \times 0.1 \mathrm{mm} \\
&= 0.00068 \mathrm{mm}
\end{aligned}
$$
</div>
由于设置胶层的失效位移（指胶层的塑性变形位移，不包括弹性变形位移）为0.001，即胶层从发生破坏到最终断裂，上下表面相对位移的增加量，对应 
S-U 曲线中最高点到右侧与横轴焦点之间的横轴投影距离，因此胶层断裂时上下表面的位移为
<div>
$$
\begin{aligned}
U_f &= U_{f0} + U_{\Delta f} \\
&= 0.00068 \mathrm{mm} + 0.001 \mathrm{mm} \\
&= 0.00168 \mathrm{mm}
\end{aligned}
$$
</div>
从而胶层的失效应变为
<div>
$$
\begin{aligned}
\varepsilon _f &= \varepsilon _{f0} + \varepsilon _{\Delta f} \\
&= \varepsilon _{f0} + U_{\Delta f} / t \\
&= 0.0068 + 0.001 \mathrm{mm}/0.1\mathrm{mm} \\
&=0.0168
\end{aligned}
$$
</div>
因此当所用胶层材料的应力-应变数据和厚度已知时，可根据求出仿真所需的失效位移值

<div>
$$
U_{\Delta f} = \varepsilon _{\Delta f} \cdot t
$$
</div>
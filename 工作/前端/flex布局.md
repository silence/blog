### 前言

flex 布局出现好久了，但是**每次都需要重新查阅文档**，每次都没能记住。固写篇浅显易懂的文章稍微巩固下。

1. 在 flex 容器中默认存在两条轴，水平主轴 (main axis) 和 垂直的交叉轴 （cross axis），这是默认的设置，当然可以通过修改使垂直方向变为主轴，水平方向变为交叉轴，这个后面再说。

2. 在容器中的每个单元块被称为 flex item，每个项目占据的主轴空间为（main size），占据的交叉轴的空间为（cross size）。

3. 可以通过

   ```css
   .contain {
     display: flex | inline-flex; 
   }
   ```

   来生成块状或行内的 flex 容器盒子

   **需要注意的是：当设置 flex 布局之后，子元素的 float 、clear 、vertical-align 的属性将会失效。**

### flex 容器属性

有下面六种属性可以设置在容器上，它们分别是：

1. flex-direction

2. flex-wrap

3. flex-flow

4. justify-content

5. align-items

6. align-content

1. **flex-direction** ：决定主轴的方向

   ```css
   .container {
     flex-direction: row | row-reverse | column | column-reverse;
   }
   ```

   默认值：row ，主轴为水平方向，起点在左端。

   row-reverse：主轴为水平方向，起点在右端。

   column：主轴为垂直方向，起点在上沿。

   column-reverse：主轴为垂直方向，起点在下沿。

2. **flex-wrap**：决定容器类项目是否可以换行。

   ```css
   .container {
     flex-wrap: nowrap | wrap | wrap-reverse
   }
   ```

   默认值：nowrap 不换行，即当主轴尺寸固定时，**当空间不足时，项目尺寸会随之调整而并不会挤到下一行**。

   wrap：项目主轴总尺寸超出容器时换行，第一行在上方。

   wrap-reverse：项目主轴总尺寸超出容器时换行，第一行在下方。

3. flex-flow：flex-direction 和 flex-wrap 的简写形式

   ```css
   .container {
     flex-flow: <flew-direction> || <flex-wrap>
   }
   ```

   默认值为：row nowrap，感觉没什么卵用，老老实实分开写就好了。

4. **Justfy-content**：定义了项目在主轴的对齐方式。

   ```css
   .container {
     justify-content: flex-start | flex-end | center | space-between | space-around;
   }
   ```

   默认值：flex-start 左对齐

   flex-end 右对齐

   center 居中

   space-between：两端对齐，项目之间的间隔相等，即剩余空间等分成间隙。

   space-around：每个项目两侧的间隔相等，所以项目之间的间隔比项目与边缘的间隔大一倍。

5. **align-items**：定义了项目在交叉轴的对齐方式

   ```css
   .container {
     align-items: flex-start | flex-end | center | baseline | stretch;
   }
   ```

   默认值为 stretch 即如果项目未设置高度或者设为 auto，将占满整个容器的高度。

   flex-start：交叉轴的起点对齐

   flex-end：交叉轴的终点对齐

   center：交叉轴的中点对齐

   baseline：项目的第一行文字的基线对齐

6. align-content：定义了多根轴线的对齐方式，如果项目只有一根轴线，那么该属性将不起作用

   ```css
   .container {
     align-content: flex-start | flex-end | center | space-between | space-around | stretch
   }
   ```

   当 flex-wrap 设置为 no-wrap 的时候，容器仅存在一根轴线，因为项目不会换行，就不会产生多条轴线。

   当 flex-wrap 设置为 wrap 的时候，容器可能会出现多条轴线，这时候就需要设置多条轴线之间的对齐方式了。

   下图建立在主轴为水平方向时测试，即 flex-direction: row，flew-wrap：wrap

   默认值 stretch ：平分垂直轴上的空间。

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304155119.jpg)

   flex-start：轴线全部在交叉轴上的起点对齐

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304155210.jpg)

   flex-end：轴线全部在交叉轴上的终点对齐

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304155254.jpg)

   center：轴线全部在交叉轴上的中间对齐

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304155346.jpg)

   space-between：轴线两端对齐，之间的间隔相等，即剩余空间等分成间隙。

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304155442.jpg)

   space-around：每个轴线两侧的间隔相等，所以轴线之间的间隔比轴线与边缘的间隔大一倍。

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304155542.jpg)

7. 到此关于容器的所有属性都讲完了，接下来讲讲关于在flex item 上的属性。

### flex 项目属性

有六种属性可运用在 item 项目上：

1. order
2. flex-basis
3. flex-grow
4. flex-shrink
5. flex
6. align-self



1. **order：定义项目在容器中的排列顺序，数值越小，排列越靠前，默认值为 0 。**

   ```css
   .item {
     order: <integer>;
   }
   ```

2. **flex-basis：定义了在分配多余空间之前，项目占据的主轴空间，浏览器根据这个属性，计算主轴是否有多余空间**

   ```css
   .item {
     flex-basis: <length> | auto;
   }
   ```

   默认值：auto，即项目本来的大小，这时候 item 的宽高取决于 width 和 height 的值。

   **当主轴为水平方向的时候，当设置了 flex-basis，项目的宽度设置值会失效，flex-basis 需要跟 flex-grow 和 flex-shrink 配合使用才能发挥效果**。

   - 当 flex-basis 值为 0 % 时，是把该项目视为零尺寸的，故即使声明该尺寸为 140px，也并没有什么用。
   - 当 flex-basis 值为 auto 时，则根据尺寸的设定值（假如为 100px），则这 100px 不会纳入剩余空间。

3. **flex-grow：定义项目的放大比例**

   ```css
   .item {
     flex-grow: <number>;
   }
   ```

   默认值为 0 ，即如果存在剩余空间，也不放大

   ![img](https://raw.githubusercontent.com/silence/blog/assets/assets/20210304161114.jpg)

   当所有的项目都以 flex-basis 的值进行排列后，仍有剩余空间，那么这时候 flex-grow 就会发挥作用了。

   如果所有项目的 flex-grow 属性都为 1 ，则它们将等分剩余空间。（如果有的话）

   如果一个项目的 flex-basis 的值排列完后发现空间不够了，且 flex-wrap：nowrap 时，此时 flex-grow 则不起作用了，这时候就需要接下来的这个属性。

4. **flex-shrink：定义了项目的缩小比例**

   ```css
   .item {
     flex-shrink: <number>;
   }
   ```

   默认值：1，即如果空间不足，该项目该缩小，负值对该属性无效。

   - 如果所有项目的 flex-shrink 属性都为 1，当空间不足时，都将等比例缩小。
   - 如果所有项目的 flex-shrink 属性为 0，其他项目都为 1，则空间不足时，前者不缩小。

5. **flex：flex-grow，flex-shrink 和 flex-basis 的简写**

   ```css
   .item {
     flex: none | [<'flex-grow'> <'flex-shrink'>? || <'flex-basis'>]
   }
   ```

   flex 的默认值是以上三个属性值的组合。假设以上三个属性同样取默认值，则 flex 的默认值是 0 1 auto。

   - 有关快捷值 
     - auto (1 1 auto) 即响应式，既允许放大也允许缩小
     - none (0 0 auto) 不允许放大，也不允许缩小
   - 当 flex 取值为一个非负数字，则该数字为 flex-grow 值，flex-shrink 取 1，flex-basis 取 0%

      - 即 （1 1 0%）
- 当 flex 取值为 0 时，对应的三个值分别为 (0 1 0%) 即不允许放大
  
   - 当 flex 取值为一个长度或百分比，则视为 flex-basis 值，flex-grow 取 1，flex-shrink 取 1。即 (1 1 (0% | 24px))
- 当 flex 取值为两个非负数字，着分别视为 flex-grow 和 flex shrink 的值，flex-basis 取 0% 。即（2，3，0%）
   - 当 flex 取值为一个非负数字和一个长度或百分比，则分别视为 flex-grow 和 flex-basis 的值，flex-shrink 取 1。即（11 1 32px）

   grow 和 shrink 是一对双胞胎，grow 表示伸张因子，shrink 表示是收缩因子。

6. **align-self：允许单个项目有与其他项目不一样的对齐方式**

   单个项目覆盖 align-items 定义的属性

   默认值为 auto，表示继承父元素的 align-items 属性，如果没有父元素，则等同于 stretch。

   ```css
   .item {
     align-self : auto | flex-start | flex-end | center | baseline | stretch;
   }
   ```

   这个跟 align-items 属性时一样的，只不过 align-self 是对单个项目生效的，而 align-items 则是对容器下的所有项目生效的。

### Reference

https://zhuanlan.zhihu.com/p/25303493
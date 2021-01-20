# ConstraintLayout

### 官方文档

> https://developer.android.google.cn/reference/android/support/constraint/ConstraintLayout

### 相对定位属性

- layout_constraintBaseline_toBaselineOf
- layout_constraintLeft_toLeftOf
- layout_constraintLeft_toRightOf
- layout_constraintRight_toLeftOf
- layout_constraintRight_toRightOf
- layout_constraintTop_toTopOf
- layout_constraintTop_toBottomOf
- layout_constraintBottom_toTopOf
- layout_constraintBottom_toBottomOf
- layout_constraintStart_toEndOf
- layout_constraintStart_toStartOf
- layout_constraintEnd_toStartOf
- layout_constraintEnd_toEndOf

> 以上这些属性意思就是约束控件的某边到指定控件的某边
>
> 值是另一个控件的ID或parent
>
> Baseline指的是文本基线

### 偏移属性

- layout_constraintHorizontal_bias 水平偏移
- layout_constraintVertical_bias 垂直偏移

> 居中时设置偏移，值在0.0~1.0

### 角度定位属性

- layout_constraintCircle 参照物控件ID
- layout_constraintCircleAngle 角度
- layout_constraintCircleRadius 距离

> 角度定位指的是可以用一个角度和一个距离来约束两个空间的中心

### 边距属性

- layout_marginStart
- layout_marginEnd
- layout_marginLeft
- layout_marginTop
- layout_marginRight
- layout_marginBottom

> 使用边距属性控件必须在布局里约束一个相对位置，margin只能大于等于0

### Gone边距属性

- layout_goneMarginStart
- layout_goneMarginEnd
- layout_goneMarginLeft
- layout_goneMarginTop
- layout_goneMarginRight
- layout_goneMarginBottom

> goneMargin主要用于约束的控件可见性被设置为gone的时候使用的margin值

### 尺寸约束属性

- minWidth
- minHeight
- maxWidth
- maxHeight
- constrainedWidth
- constrainedHeight

### 链风格属性

- layout_constraintHorizontal_chainStyle

> spread(默认)、spread_inside(展开元素)、packet(打包元素)

### 比例约束属性

- layout_constraintDimensionRatio

> 值x:x，某一边需要为0dp。如果两边都是0dp，需要指定计算一边长度，如果W,2:1

### 百分比约束属性

- layout_constraintWidth_percent
- layout_constraintHeight_percent
- layout_constraintGuide_percent

### 布局辅组工具

- androidx.constraintlayout.widget.Barrier 栅栏
- androidx.constraintlayout.widget.Constraints辅助线
- androidx.constraintlayout.widget.Group 归组
- androidx.constraintlayout.widget.Guideline 辅助线
- androidx.constraintlayout.widget.Placeholder 占位符
- androidx.constraintlayout.helper.widget.Layer
- androidx.constraintlayout.helper.widget.Flow

### 自定义ConstraintHelper




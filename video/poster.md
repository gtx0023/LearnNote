poster属性图片铺满video

video{
width: 100%;
object-fit:fill;
}

标签定义及使用说明
object-fit 属性指定元素的内容应该如何去适应指定容器的高度与宽度。

object-fit 一般用于 img 和 video 标签，一般可以对这些元素进行保留原始比例的剪切、缩放或者直接进行拉伸等。

您可以通过使用 object-position 属性来切换被替换元素的内容对象在元素框内的对齐方式。

语法
object-fit: fill|contain|cover|scale-down|none|initial|inherit;

属性值
值	描述	尝试一下
fill	默认，不保证保持原有的比例，内容拉伸填充整个内容容器。	尝试一下 »
contain	保持原有尺寸比例。内容被缩放。	尝试一下 »
cover	保持原有尺寸比例。但部分内容可能被剪切。	尝试一下 »
none	保留原有元素内容的长度和宽度，也就是说内容不会被重置。	尝试一下 »
scale-down	保持原有尺寸比例。内容的尺寸与 none 或 contain 中的一个相同，取决于它们两个之间谁得到的对象尺寸会更小一些。	尝试一下 »
initial	设置为默认值，关于 initial	
inherit	从该元素的父元素继承属性。 关于 inherit

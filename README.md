1. 在RecyclerView中使用GridLayoutManager实现Item均匀分布
2. 详细说明请见https://www.jianshu.com/p/1d323775e2f5

# GridLayoutManger分配Item空间的机制
**GridLayoutManger在绘制子View的时候，会先为它们分配固定的空间**。
比如，我们这里是4列，则会为每个图片分配父布局宽度（这里是屏幕宽度） / 4的宽度大小的空间。
1. 每个图片会占据自己的空间，且不会超出这个空间。如果超出了，比如设置了很大的固定宽度，则超出的部分会被遮盖。
2. 如果使用ItemDecoration设置间隔，则间隔会占用当前图片的空间，图片会被压缩。

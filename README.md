1. 在RecyclerView中使用GridLayoutManager实现Item均匀分布
2. 详细说明请见https://www.jianshu.com/p/1d323775e2f5

# 背景描述
最近开发中遇到需求，需要实现图片的格子分布效果，如下图所示
![格子图片](https://upload-images.jianshu.io/upload_images/2658578-22632ab28c5aedd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中要求：
1. 图片与图片的间隔，图片与屏幕的左边距，以及图片与屏幕的右边距，都为固定大小，比如`10dp`。
2. 图片宽高相等。

<br>
# 代码实现
主要代码如下
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val images = listOf(...)
        val imageAdapter = ImageAdapter(images)
        rv_images.apply {
            layoutManager = GridLayoutManager(context, 4, RecyclerView.VERTICAL, false)
            addItemDecoration(EvenItemDecoration(dp2px(context, 10), 4))
            adapter = imageAdapter
        }
    }

    class ImageAdapter(private val images: List<Int> = listOf()) : RecyclerView.Adapter<ImageAdapter.ViewHolder>() {
        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val view = LayoutInflater.from(parent.context).inflate(R.layout.item_image, parent, false)
            // 动态设置图片大小 保证宽高相等
            val ivSize = (getScreenContentWidth(parent.context) - dp2px(parent.context, 10) * 5) / 4
            view.iv_image.layoutParams.height = ivSize
            return ViewHolder(view)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            holder.view.iv_image.setImageResource(images[position])
        }
        ...
    }

    class EvenItemDecoration(private val space: Int, private val column: Int) : RecyclerView.ItemDecoration() {
        override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State) {
            val position = parent.getChildAdapterPosition(view)
            // 每个span分配的间隔大小
            val spanSpace = space * (column + 1) / column
            // 列索引
            val colIndex = position % column
            // 列左、右间隙
            outRect.left = space * (colIndex + 1) - spanSpace * colIndex
            outRect.right = spanSpace * (colIndex + 1) - space * (colIndex + 1)
            // 行间距
            if (position >= column) {
                outRect.top = space
            }
        }
    }
}
```
布局`item_image.xml`如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<ImageView
    android:id="@+id/iv_image"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scaleType="fitXY"
    tools:src="@mipmap/ic_launcher"
    android:background="#ffccff"/>
```
其中需要注意是
1. 图片的宽度在布局和代码中不能写死，需要自适应。
2. 在代码（Adapter）中根据屏幕宽度来计算图片高度。
3. 在ItemDecoration中计算间隔。

<br>
# 分析-GridLayoutManger分配Item空间的机制
**GridLayoutManger在绘制子View的时候，会先为它们分配固定的空间**。
比如，我们这里是4列，则会为每个图片分配父布局宽度（这里是屏幕宽度） / 4的宽度大小的空间。
1. 每个图片会占据自己的空间，且不会超出这个空间。如果超出了，比如设置了很大的固定宽度，则超出的部分会被遮盖。
2. 如果使用ItemDecoration设置间隔，则间隔会占用当前图片的空间，图片会被压缩。

**示例一-设置固定大宽度**
在Adapter代码中设置图片的宽度并在去掉ItemDecoration
```
// ImageAdapter
...
val ivSize = (getScreenContentWidth(parent.context) - dp2px(parent.context, 10) * 5) / 4
view.iv_image.layoutParams.width = 1000
view.iv_image.layoutParams.height = ivSize
...

// MainActivity 中注释掉EvenItemDecoration
...
// addItemDecoration(EvenItemDecoration(dp2px(context, 10), 4))
...
```
结果如下：
![宽度设置为1000px的图片](https://upload-images.jianshu.io/upload_images/2658578-0d38e5c7cf5fd001.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过Studio自带`Layout Inspector`可以查看每个图片的左右位置，第一行图片的`mLeft`分为0、360、720、1080，对应的`mRight`是`mLeft` + 1000。
以上示例说明
1. 每个图片占据的父布局的宽度是一样的，为父布局宽度除以列数，这里是1440 / 4 = 360。
2. 每个图片超出所占据空间宽度后的部分会被后面的图片或屏幕遮盖。


**示例二-设置较大的左右间隔**
修改EvenItemDecoration代码如下：
```
    class EvenItemDecoration(private val space: Int, private val column: Int) : RecyclerView.ItemDecoration() {
        override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State) {
            outRect.left = space
            outRect.right = space
        }
    }
```
运行结果如下：
![有间隔的Grid](https://upload-images.jianshu.io/upload_images/2658578-e1a49131148182be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
通过`Layout Inspector`查看每个图片的左右位置，第一行图片的`mLeft`分为35（0 +35）、395（360+35）、755（720+35）、1115（1080+35），对应的mRight分别是325（360-35）、685（720-35）、1045（1080-35）、1405（1440-35）。
说明设置的间隔的确占用了对应图片之前分配的空间。

最后，根据以上机制，不难理解`EvenItemDecoration`中的代码。
源代码
<br>
> 提示：文章首发[我的个人博客](https://zhangliangnbu.github.io/)和[简书](https://www.jianshu.com/)
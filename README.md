This libraries has been customized from `drakeet`. 
Thanks for your awesome libraries.

## Getting started
In your `build.gradle`:
```groovy
dependencies {
  implementation 'co.legato.libraries:adapter:1.1.4'
}
```

## Supported

- `FlexibleAdapter`: Normal adapter without paging3
- `Paging3DataAdapter`: Include paging3 libraries of google.
- `RecyclerView Selection Library of google`: It uses the selection library to provide extra features the user can interact with, namely:\
        1. Pressing an item crosses it from the list.\
        2. Long-pressing an item triggers multiple-selection mode.

## Usage
#### Step 1. Create a Kotlin `class` or `data class` and extend `IFlexibleItem`, for example:

```kotlin
data class Item(val name: String) : IFlexibleItem {
    override fun areItemsTheSame(iFlexibleItem: IFlexibleItem): Boolean {
        return iFlexibleItem is Item && iFlexibleItem.name == name
    }

    override fun areContentsTheSame(iFlexibleItem: IFlexibleItem): Boolean {
        return iFlexibleItem is Item && iFlexibleItem.name == name
    }
}
```

#### Step 2. Create a class extends `ItemViewDelegate<T, VH : ViewHolder>`, for example:

```kotlin
class FooViewDelegate: ItemViewDelegate<Item, FooViewDelegate.ViewHolder>() {

  override fun onCreateViewHolder(context: Context, parent: ViewGroup): ViewHolder {
    // If you want a LayoutInflater parameter instead of a Context,
    // you can use ItemViewBinder as the parent of this class.
    return ViewHolder(FooView(context))
  }

  override fun onBindViewHolder(holder: ViewHolder, item: Item) {
    holder.fooView.text = item.value

    Log.d("ItemViewDelegate API", "position: ${holder.bindingAdapterPosition}")
    Log.d("ItemViewDelegate API", "items: $adapterItems")
    Log.d("ItemViewDelegate API", "adapter: $adapter")
    Log.d("More", "Context: ${holder.itemView.context}")
  }

  class ViewHolder(itemView : View): RecyclerView.ViewHolder(itemView) {
    val fooView: TextView = itemView.findViewById(R.id.foo)
  }
}
```

##### Or if you are using a custom View instead of XML layout, you can use `ViewDelegate`:

> The `ViewDelegate` is a simple `ItemViewDelegate` that does not require to declare and provide a `RecyclerView.ViewHolder`.

```kotlin
class FooViewDelegate : ViewDelegate<Item, FooView>() {

  override fun onCreateView(context: Context): FooView {
    return FooView(context).apply { layoutParams = LayoutParams(MATCH_PARENT, WRAP_CONTENT) }
  }

  override fun onBindView(view: FooView, item: Item) {
    view.imageView.setImageResource(item.imageResId)
    view.textView.text = item.text

    view.textView.text = """
      |${item.text}
      |viewHolder: ${view.holder}
      |layoutPosition: ${view.layoutPosition}
      |absoluteAdapterPosition: ${view.absoluteAdapterPosition}
      |bindingAdapterPosition: ${view.bindingAdapterPosition}
    """.trimMargin()
  }
}
```

##### Or if you are using a  ViewBingding, you can use `ViewBindingDelegate`:
```kotlin
class TextViewBindingDelegate : ViewBindingDelegate<Item, BinderItemViewBinding>() {

    override fun onCreateView(inflater: LayoutInflater, parent: ViewGroup): BinderItemViewBinding {
        return BinderItemViewBinding.inflate(inflater, parent, false)
    }

    override fun onBindView(binding: BinderItemViewBinding, item: Item) {
        val isSelected = tracker?.isSelected(item.getSelectionIndex()) ?: false
        binding.tvName.text = item.name
        binding.root.setBackgroundResource(
            if (isSelected) R.color.purple_200
            else R.color.white
        )
    }
}
```

#### Step 3. `register` your types and setup your `RecyclerView`, for example:

```kotlin
class SampleActivity : AppCompatActivity() {

  private val adapter = MultiTypeAdapter()
  private val items = ArrayList<Any>()

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_list)
    val recyclerView = findViewById<RecyclerView>(R.id.list)

    adapter.register(TextItemViewDelegate())
    adapter.register(ImageItemViewDelegate())
    adapter.register(RichItemViewDelegate())
    recyclerView.adapter = adapter

    val textItem = TextItem("world")
    val imageItem = ImageItem(R.mipmap.ic_launcher)
    val richItem = RichItem("小艾大人赛高", R.drawable.img_11)

    for (i in 0..19) {
      items.add(textItem)
      items.add(imageItem)
      items.add(richItem)
    }
    adapter.items = items
    adapter.notifyDataSetChanged()
  }
}
```

**That's all, you're good to go!**

## Advanced usage 

**1. One to many**:  

```kotlin
adapter.register(Data::class).to(
  DataType1ViewDelegate(),
  DataType2ViewDelegate()
).withKotlinClassLinker { _, data ->
  when (data.type) {
    Data.TYPE_2 -> DataType2ViewDelegate::class
    else -> DataType1ViewDelegate::class
  }
}
```

**2. Recyclerview selection tracker**:  

Step 1: We need to override `getSelectionIndex` and return index for it:
```kotlin
data class Item(val name: String) : IFlexibleItem {
    //... More info here
    
    override fun getSelectionIndex(): Long {
        return name.hashCode().toLong()
    }
    
    // optional function
    fun canSetStateForKey(nextState: Boolean): Boolean = true
    fun canSetStateAtPosition(nextState: Boolean): Boolean = true
}
```

Step 2: Enable selection tracker feature of adapter at activity, fragment:
```kotlin
adapter.setSelectionTracker(
            rv = mRootView.recyclerView,
            isSingleTapEnabled = true
        ) {
            // add more function of recyclerview selection feature libs here.
            this.withOnItemActivatedListener { item, e ->
                return@withOnItemActivatedListener true
            }
        }
```


**More methods that you can override from [ItemViewDelegate](library/src/main/kotlin/me/drakeet/multitype/ItemViewDelegate.kt)**:

```kotlin
open fun onBindViewHolder(holder: VH, item: T, payloads: List<Any>)
open fun getItemId(item: T): Long
open fun onViewRecycled(holder: VH)
open fun onFailedToRecycleView(holder: VH): Boolean
open fun onViewAttachedToWindow(holder: VH)
open fun onViewDetachedFromWindow(holder: VH)

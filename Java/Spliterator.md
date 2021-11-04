<!--
 * @Author: QingHui Meng
 * @Date: 2021-06-23 16:21:32
-->
Spliterator，可拆分迭代器，是Java为了**并行**遍历数据源中的元素而设计的迭代器，和Iterator类似，只不过一个是顺序遍历，一个是并行遍历。java已经为集合框架中包含的所有数据结构提供了一个默认的Spliterator实现：集合实现了Spliterator接口，接口提供了一个spliterator方法。

## Spliterator方法

* boolean tryAdvance(Consumer<? super T> action): 顺序处理每个元素，如果还有元素要处理，返回true，否则返回false。

* void forEachRemaining(Consumer<?super T> action): 对剩余元素执行给定的操作，知道所有元素都被处理或者被异常终止。默认调用tryAdvance方法：
    ```java
    default void forEachRemaining(Consumer<? super T> action) {
        do { } while (tryAdvance(action));
    }
    ```

* Spliterator<T> trySplit(): 专为Spliterator设计的方法，区分与普通的Iterator，该方法会把当前元素的一部分划分出去创建一个心得Spliterator作为返回，两个Spliterator会变为并行执行，如果元素小到无法划分则返回null。

* long estimateSize(): 估算还剩下多少个元素需要遍历。

* int characteristics(): 返回Spliterator的特性，Spliterator有八大特性，一个Spliterator可以有多个特性，特性可以进行or运算，最后得到最终的characteristics。
    ```java
    public static final int ORDERED    = 0x00000010;//表示元素是有序的（每一次遍历结果相同）
    public static final int DISTINCT   = 0x00000001;//表示元素不重复
    public static final int SORTED     = 0x00000004;//表示元素是按一定规律进行排列（有指定比较器）
    public static final int SIZED      = 0x00000040;//表示大小是固定的
    public static final int NONNULL    = 0x00000100;//表示没有null元素
    public static final int IMMUTABLE  = 0x00000400;//表示元素不可变
    public static final int CONCURRENT = 0x00001000;//表示迭代器可以多线程操作
    public static final int SUBSIZED   = 0x00004000;//表示子Spliterators都具有SIZED特性
    ```
* boolean hasCharacteristics(int characteristics): 是否具有当前特征值


## 实现类 ArrayListSpliterator
通过查看源码可以发现：
1. ArrayListSpliterator本质上还是对原list进行操作，只是通过index和fence来控制每次处理的范围。
2. ArrayListSpliterator遍历元素时，不能对list进行结构变更，否则会报错。
```java
static final class ArrayListSpliterator<E> implements Spliterator<E> {
   //用于存放ArrayList对象
   private final ArrayList<E> list;
   //起始位置（包含），tryAdvance/trySplit操作时会修改
   private int index; 
   //结束位置（不包含），-1 表示到最后一个元素
   private int fence; 
   //用于存放list的modCount
   private int expectedModCount; 

   ArrayListSpliterator(ArrayList<E> list, int origin, int fence,
                             int expectedModCount) {
            this.list = list; 
            this.index = origin;
            this.fence = fence;
            this.expectedModCount = expectedModCount;
    }

   //获取结束位置（存在意义：首次初始化需对fence和expectedModCount进行赋值）
   private int getFence() { 
        int hi; 
        ArrayList<E> lst;
        //fence<0时（第一次初始化时，fence才会小于0）：
        if ((hi = fence) < 0) {
            //list 为 null时，fence=0
            if ((lst = list) == null)
                hi = fence = 0;
            else {
                //否则，fence = list的长度。
                expectedModCount = lst.modCount;
                hi = fence = lst.size;
            }
        }
        return hi;
    }

    //分割list，返回一个新分割出的spliterator实例
    public ArrayListSpliterator<E> trySplit() {
        //hi为当前的结束位置
        //lo 为起始位置
        //计算中间的位置
        int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
        //当lo>=mid,表示不能在分割，返回null
        //当lo<mid时,可分割，切割（lo，mid）出去，同时更新index=mid
        return (lo >= mid) ? null : 
            new ArrayListSpliterator<E>(list, lo, index = mid, expectedModCount);
    }

    //返回true 时，表示还有元素未处理
    public boolean tryAdvance(Consumer<? super E> action) {
        if (action == null)
            throw new NullPointerException();
        //hi为当前的结束位置
        //i 为起始位置
        int hi = getFence(), i = index;
        //还有剩余元素未处理时
        if (i < hi) {
            //处理i位置，index+1
            index = i + 1;
            @SuppressWarnings("unchecked") E e = (E)list.elementData[i];
            action.accept(e);
            //遍历时，结构发生变更，抛错
            if (list.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            return true;
        }
        return false;
    }

    //顺序遍历处理所有剩下的元素
    public void forEachRemaining(Consumer<? super E> action) {
        int i, hi, mc;
        ArrayList<E> lst; Object[] a;
        if (action == null)
            throw new NullPointerException();
        if ((lst = list) != null && (a = lst.elementData) != null) {
            //当fence<0时，表示fence和expectedModCount未初始化
            if ((hi = fence) < 0) {
                mc = lst.modCount;
                hi = lst.size;
            }
            else
                mc = expectedModCount;
            if ((i = index) >= 0 && (index = hi) <= a.length) {
                for (; i < hi; ++i) {
                    @SuppressWarnings("unchecked") E e = (E) a[i];
                    //调用action.accept处理元素
                    action.accept(e);
                }
                //遍历时发生结构变更时抛出异常
                if (lst.modCount == mc)
                    return;
            }
        }
        throw new ConcurrentModificationException();
    }

    public long estimateSize() {
        return (long) (getFence() - index);
    }

    public int characteristics() {
        //返回特征值
        return Spliterator.ORDERED | Spliterator.SIZED | Spliterator.SUBSIZED;
    }
}
```


## Spliterator使用
```java
    List<Integer> list = Lists.newArrayList(0,1,2,3,4,5,6,7);
    Spliterator<Integer> s = list.spliterator();
    // index：spliterator起始位置   fence：结束位置(不包含)
    // 此时结果：s: index-0 fence-8
    Spliterator<Integer> s1 = s.trySplit();
    // 此时结果 s:index-4 fence-8 s1:index-0 fence-4
    Spliterator<Integer> s2 = s.trySplit();
    // 此时结果 s:6-8 s2:4-6 s1:0-4

    // tryAdvance方法进行单个处理
    while (s.tryAdvance(i -> i++)){

    }
    // 处理剩余的所有元素
    s.forEachRemaining(i -> i++);
```
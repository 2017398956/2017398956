# HashMap 从 0 到 1

## 前言

**阅读提醒：**关于 HashMap 的原理网上有很多内容，这里提供一个全新的视角来看待 HashMap ，请耐心看下去，保证你彻底理解 HashMap。

首先，按照一般的讲解模式会说 “HashMap 的意义就是实现一种快速的查找并且插入、删除性能都不错的一种数据结构。”，然后接着讲解 HashMap 的数据结构和实现快速的查找、插入、删除的方式。这种方式虽然理清了 HashMap 的原理，但总觉得和 HashMap 间，隔着一层若有若无的东西；所以，我们不妨换个角度，先不管 HashMap 的原理及实现，仅从需求的角度提出一个问题：

	现在有一堆数据怎么才能快速的对其增删改查呢？

## **方案一**

用链表存储这些数据，但是在查找的环节需要从头遍历。

## **方案二**

用数组（记为：table）存储这些数据，当查找某个位置的数据时比较方便，删除的时候直接将该位置的数据置为 null ，数组不进行移动即可。看起来可以增删改查都很方便，**但是这里存在一个问题**：

	当处理大量数据的时候，位置信息对用户没有任何意义，更常用的是删除具有某个特征（记为：K）的数据，
	这时就需要从头遍历这些数据，找到特征 K 所在的位置（这与链表的查找何异）。
	那么有没有办法将 K 与所在的位置关联起来？
	如果数据的特征 K 和数据处于数组的位置强相关且不同的 K 对应不同的位置，不就直接根据 K 定位到了吗，这样就不再需要遍历了。

由于数组的位置都是整数，所以我们的问题变成了将 K 转换成一个整数。在 Java 中万物皆对象，特征 K 也是个对象，而对象的 hashCode 就满足不同的对象其 hashCode 一般不相同的条件。如果我们直接用 hashcode 的值作为 position 来存储的话就碰到了这个问题：hashcode 是个 int 类型的数据，那么它有可能很大（例如第一个数据的 hashcode 就是 Int.MAX_VALUE），这样就要在内存中开辟一块很大的空间，这显然是不合适的。

**怎么解决 hashcode 在过大的时候占用很大内存的问题呢？**

我们可以这样：先为这个 table 设置个长度，记为：capacity，在添加数据的时候用 hashcode % capacity 取余数，用 **余数** 来表示这个数据应该放在数组的哪个位置，这样就不需要开辟大量空间了。既然每次添加数据都需要取余运算，要是还能优化取余的操作就更好了，性能提升一点是一点。我们知道计算机都是二进制的所以位运算会比直接用数学计算更高效，而且我们还知道位运算中 

（2<sup>n</sup> - 1）& hashcode 就等效于 hashcode % 2<sup>n</sup>

那么只要 <b>capacity 的值都取 2<sup>n</sup></b> 就好了。为了更加灵活，开发者是可以自己设置 capacity 的值的，当开发者设置了一个不是 2<sup>n</sup> 的值时就需要我们自动为其扩展到 2<sup>n</sup> ，下面是扩展的方法:

	// 注意：capacity 是 int 类型，32位数据，所以 >>> 16 即可
	int MAXIMUM_CAPACITY = 1 << 30
	int tableSizeFor(int capacity) {
        int n = capacity - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

虽然开辟大空间的问题解决了，但是取余这个操作会获得 **很多** K 不同，但 余数 相同的数据（余数只和低位有关，这种情况我们可以叫作：**hash 冲突**），这些数据都要放在 table 的同一个位置。所以，我们的问题又变成了：

**怎么让 hashcode % capacity 的值尽量不相等？**

我们一开始用 hashcode 作为 position 是因为 不同的 key ，hashcode 一般不等的性质。而余数的值只和低位的数据有关，如果我们把 hashcode 高位的数据和低位的数据按照某种方式组合成一个全新的数据，那么这个新的数据就兼具 hashcode 的全部特征，是不是就不容易产生相同的余数了？由于 hashcode 是 32 位的，我们可以把低 16 位和 高 16 位复合起来：

	// 这里用的是 ^ ,当然用 | 、 & 也可以，但是你会发现用 ^ 时，
	// hash(hashcode) % capacity 得到的余数更分散，更符合我们的要求
    int hash(int hashcode) {
        return hashcode ^ (hashcode >>> 16);
    }

虽然上面的方法解决了余数过于集中的问题，但是还是 **无法避免余数相同的情况**，此时我们应该怎么办？

当余数相同时我们需要在 table 的同一位置放置不同的数据，那么就要求 table 中的每个元素都应该是个数组或者是个链表。当是数组时，那 table 就变成了一个二维数组，因为 java 不支持变长数组所以这种情况有可能导致很大的空间浪费；当是链表时，则不会出现这种情况，**但是用链表的话则这个链表不能过长**，极端情况下 capacity = 1 ，那这就是个链表结构，查询数据依然会很慢。

**那么怎么控制 table 每个元素的链表结构不至于过长呢？**

通过上面的 hash(hashcode) % capacity 方法计算后，我们暂时认为每向 table 中添加一个数据，落在任何位置的可能性都是一样的。那么当有远少于 capacity 个数据添加后，可近似认为 table 每个元素链表的长度都不大于 1，随着添加的数据越来越多，则 table 的空闲位置越来越少，从经验和概率上来说当 table 被充满 75%（记作：loadFactor） 后，再次添加新的数据，那么这个新数据就极易落在已经存在数据的位置上。如果此时我们将 table 的长度扩大一倍（如果扩大的过小，有可能需要反反复复扩大），**把所有的元素重新添加到这个新的 table 中**，然后再新加数据时，新的 table 还是很空旷的，这样就不容易落在已有数据的位置。到此我们就解决了所有的问题 。

## 总结

理论上上面的解决方案是可行的，现在可以将这个方案简单的概况一下了。需要的数据有

1. 特征值 key
2. 数据 value
3. table 的长度 capacity
4. table 的填充率 loadFactor

不妨给这个方案起名叫 HashMap 然后对应的 java 类可以简写为

    public class HashMap<Key , Value>{
        ...
        public HashMap(int capacity , float loadFactor){
            ...
        }
    }

到此，我们就实现了一个用于快速增删改查的 HashMap。
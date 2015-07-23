title: 集合应用：LinkedHashMap与LRUcache
date: 2015-07-03 7:43:15
categories: Java集合
tags: Java集合
description: 本文是对Java常用的集合的学习，通过借鉴摘抄转载网上同样主题的博客进行学习。主要以学习记录为主，作为后期复习的一个材料。
---
#LRU缓存介绍
---
我们平时总会有一个电话本记录所有朋友的电话，但是，如果有朋友经常联系，那些朋友的电话号码不用翻电话本我们也能记住，但是，如果长时间没有联系了，要再次联系那位朋友的时候，我们又不得不求助电话本，但是，通过电话本查找还是很费时间的。但是，我们大脑能够记住的东西是一定的，我们只能记住自己最熟悉的，而长时间不熟悉的自然就忘记了。

其实，计算机也用到了同样的一个概念，我们用缓存来存放以前读取的数据，而不是直接丢掉，这样，再次读取的时候，可以直接在缓存里面取，而不用再重新查找一遍，这样系统的反应能力会有很大提高。但是，当我们读取的个数特别大的时候，我们不可能把所有已经读取的数据都放在缓存里，毕竟内存大小是一定的，我们一般把最近常读取的放在缓存里（相当于我们把最近联系的朋友的姓名和电话放在大脑里一样）。
<!--more-->
LRU缓存利用了这样的一种思想。LRU是Least Recently Used 的缩写，翻译过来就是“最近最少使用”，也就是说，LRU缓存把最近最少使用的数据移除，让给最新读取的数据。而往往最常读取的，也是读取次数最多的，所以，利用LRU缓存，我们能够提高系统的performance
#实现
要实现LRU缓存，我们首先要用到一个类LinkedHashMap。

用这个类有两大好处：一是它本身已经实现了按照访问顺序的存储，也就是说，最近读取的会放在最前面，最最不常读取的会放在最后（当然，它也可以实现按照插入顺序存储）。第二，LinkedHashMap本身有一个方法用于判断是否需要移除最不常读取的数，但是，原始方法默认不需要移除（这是，LinkedHashMap相当于一个linkedlist），所以，我们需要override这样一个方法，使得当缓存里存放的数据个数超过规定个数后，就把最不常用的移除掉。关于LinkedHashMap中已经有详细的介绍。

代码如下：（可直接复制，也可以通过[LRUcache-Java](https://github.com/tracylihui/LRUcache-Java)下载）
```java
import java.util.LinkedHashMap;
import java.util.Collection;
import java.util.Map;
import java.util.ArrayList;

/**
 * An LRU cache, based on <code>LinkedHashMap</code>.
 *
 * <p>
 * This cache has a fixed maximum number of elements (<code>cacheSize</code>).
 * If the cache is full and another entry is added, the LRU (least recently
 * used) entry is dropped.
 *
 * <p>
 * This class is thread-safe. All methods of this class are synchronized.
 *
 * <p>
 * Author: Christian d'Heureuse, Inventec Informatik AG, Zurich, Switzerland<br>
 * Multi-licensed: EPL / LGPL / GPL / AL / BSD.
 */
public class LRUCache<K, V> {
	private static final float hashTableLoadFactor = 0.75f;
	private LinkedHashMap<K, V> map;
	private int cacheSize;

	/**
	 * Creates a new LRU cache. 在该方法中，new LinkedHashMap<K,V>(hashTableCapacity,
	 * hashTableLoadFactor, true)中，true代表使用访问顺序
	 *
	 * @param cacheSize
	 *            the maximum number of entries that will be kept in this cache.
	 */
	public LRUCache(int cacheSize) {
		this.cacheSize = cacheSize;
		int hashTableCapacity = (int) Math
				.ceil(cacheSize / hashTableLoadFactor) + 1;
		map = new LinkedHashMap<K, V>(hashTableCapacity, hashTableLoadFactor,
				true) {
			// (an anonymous inner class)
			private static final long serialVersionUID = 1;

			@Override
			protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
				return size() > LRUCache.this.cacheSize;
			}
		};
	}

	/**
	 * Retrieves an entry from the cache.<br>
	 * The retrieved entry becomes the MRU (most recently used) entry.
	 *
	 * @param key
	 *            the key whose associated value is to be returned.
	 * @return the value associated to this key, or null if no value with this
	 *         key exists in the cache.
	 */
	public synchronized V get(K key) {
		return map.get(key);
	}

	/**
	 * Adds an entry to this cache. The new entry becomes the MRU (most recently
	 * used) entry. If an entry with the specified key already exists in the
	 * cache, it is replaced by the new entry. If the cache is full, the LRU
	 * (least recently used) entry is removed from the cache.
	 *
	 * @param key
	 *            the key with which the specified value is to be associated.
	 * @param value
	 *            a value to be associated with the specified key.
	 */
	public synchronized void put(K key, V value) {
		map.put(key, value);
	}

	/**
	 * Clears the cache.
	 */
	public synchronized void clear() {
		map.clear();
	}

	/**
	 * Returns the number of used entries in the cache.
	 *
	 * @return the number of entries currently in the cache.
	 */
	public synchronized int usedEntries() {
		return map.size();
	}

	/**
	 * Returns a <code>Collection</code> that contains a copy of all cache
	 * entries.
	 *
	 * @return a <code>Collection</code> with a copy of the cache content.
	 */
	public synchronized Collection<Map.Entry<K, V>> getAll() {
		return new ArrayList<Map.Entry<K, V>>(map.entrySet());
	}

	// Test routine for the LRUCache class.
	public static void main(String[] args) {
		LRUCache<String, String> c = new LRUCache<String, String>(3);
		c.put("1", "one"); // 1
		c.put("2", "two"); // 2 1
		c.put("3", "three"); // 3 2 1
		c.put("4", "four"); // 4 3 2
		if (c.get("2") == null)
			throw new Error(); // 2 4 3
		c.put("5", "five"); // 5 2 4
		c.put("4", "second four"); // 4 5 2
		// Verify cache content.
		if (c.usedEntries() != 3)
			throw new Error();
		if (!c.get("4").equals("second four"))
			throw new Error();
		if (!c.get("5").equals("five"))
			throw new Error();
		if (!c.get("2").equals("two"))
			throw new Error();
		// List cache content.
		for (Map.Entry<String, String> e : c.getAll())
			System.out.println(e.getKey() + " : " + e.getValue());
	}
}
```

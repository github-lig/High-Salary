HashMap#put
---

	put(int hash, K key, V value, boolean onlyIfAbsent) {
		1.数组无初始化，调用resize()进行初始化
		if (tab == null || tab.length == 0) {
			resize();	
		}
		
		2.通过计算出的hash值和数组长度 - 1 进行与运算（相当于求余）计算对应的下标
		n = tab.length;
		i = (hash & (n - 1))
	
		3.如果对应的数组下标为空，将新插入的元素放到数组的下标位置
		if (p = tab[i] == null) {
			tab[i] = new Node();
		} else {
		4. 如果对应的数组下标不为空，首先判断hash值是否相等，然后使用==和equals判断。
		   这也是为什么重写了equals方法，也需要重写hashCode的原因，要不然equals相等，但是hash值不等
			if (p.hash == hash && (p.key == key || (p.key != null && p.key.equals(key)))) {
				e = p;
			} else if (p instanceof TreeNode) {
		5.如果下标对应的结构是红黑树，调用红黑树的插入方法进行插入，如果存在相同的key，返回对应节点，否则插入红黑树
				e = 红黑树的插入方法;
			} else {
		6.如果是链表，并且不存在相同的key，插入尾结点的后方，插入完成之后，如果链表的长度>树化阈值，进行树化的逻辑。如果存在相同的key，返回对应的节点e
				
			}
		}
		
		7.判断是否有相同的key，如果有的话，根据方法入参判断是否是覆盖
		if (e != null) {
			return 历史值;
		}
		
		8.判断元素数量>阈值，进行扩容
	}

	
	
	
垃圾回收：  
	
	1.
	void createObject(Long size) {
		if (老年代剩余空间 > 新生代所有对象大小) {
			进行 monor gc;		
		}
		//判断是否是大对象
		if (大对象 && 老年代剩余空间找不到足够大的连续空间分配该大对象) {
			进行full gc;
		}
		
		//正常minor gc 流程
		//先走空间担保
		if (老年代剩余空间 > 新生代的所有对象大小)
			进行minor gc;
		if (往次minor gc晋升老年代的对象平均大小 > 老年代剩余空间)
			进行 full gc;
		minor gc;
	}
	
	void minorGc() {
		if (晋升老年代的对象大小 > 老年代剩余空间) 
			进行 full gc;
	}

	2.
	System.gc() full gc;
	
	3.
	使用cms进行并发回收的阶段，程序分配对象时内存不足 full gc;
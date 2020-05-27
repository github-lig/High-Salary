redis主从复制-数据同步阶段逻辑：
	
	//从节点发送psync逻辑
	if (从节点之前未执行过slaveof || 从节点执行过slaveof no one) {
		从节点发送psync(?, -1)请求全量复制
	} else {
		从节点发送psync(runid, offset)到主节点，主节点如果返回+CONTINUE ？，那么执行增量复制，否则执行全量复制。
	}

	//主节点收到psync(runid, offset)之后
	if (runid == self.runid && offset之后的数据在复制积压缓冲区) {
		return +CONTINUE 进行增量复制
	} else {
		return +FULLRESYNC<RUNID><OFFSET> 进行全量复制
	}
	

	
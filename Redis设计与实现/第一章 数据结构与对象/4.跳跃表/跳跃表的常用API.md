		    含义          操作      时间复杂度
	 	1. 创建新的跳表       zslCreate     O(1)
	 	2. 添加新的节点       zslInsert      平均：O(logN) 最坏O(N)
	 	3. 删除节点           zslDetele      平均：O(logN) 最坏O(N)
	 	4. 返回跳跃表中查找的索引 zslGetRank     平均：O(logN) 最坏O(N)
	 	5. 返回索引上的节点     zslGetElementByRank  平均：O(logN) 最坏O(N)
	 	6. 返回范围中第一个满足的节点  zslFirstInRange   平均：O(logN) 最坏O(N)
	 	7. 返回范围最后一个满足的节点 zslLastInRange    平均：O(logN) 最坏O(N) 
	 	其中N代表的是链表的长度
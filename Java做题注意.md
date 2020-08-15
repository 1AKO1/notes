文件必须是Main类



做题的开始！

```java
Arrays.fill(int[], value);//填充数组

//获取输入方法
Scanner scanner = new Scanner(System.in);
while(scanner.hasNext()){
    int a = scanner.next();
		//一直把一组输出样例全部get到
}
```



常用数据结构

```java
//栈
Stack stack = new Stack();
boolean empty();							//检查是否为空
Object peek();								//查看栈顶对象，不移除
Object pop();									//移除栈顶对象，并作为函数返回值
Object push(Object element);	//把element压入栈顶
int search(Object element);		//返回对象在栈中的位置，以1为基数
```

```java
//队列
Queue<E> queue = new LinkedList<E>();
queue.offer(E element); //向队列中加入一个元素，若队列满了，会返回false
queue.poll();						//删除队首元素，并且作为返回值，若没有返回null
queue.peek();						//查询头部元素，如果为空则返回null
//可以通过for each来遍历
```


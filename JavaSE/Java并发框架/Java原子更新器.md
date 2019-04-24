AtomicReferenceFieldUpdater

    一个基于反射的工具类，它能对指定类的指定的volatile引用字段进行原子更新。(注意这个字段不能是private的) 

newUpdater方法

	public static <U,W> AtomicReferenceFieldUpdater<U,W> newUpdater(Class<U> tclass,
	                                                                Class<W> vclass,
	                                                                String fieldName) {
	    return new AtomicReferenceFieldUpdaterImpl<U,W>
	        (tclass, vclass, fieldName, Reflection.getCallerClass());
	}

compareAndSet方法

	public final boolean compareAndSet(T obj, V expect, V update) {
		accessCheck(obj);
		valueCheck(update);
		return U.compareAndSwapObject(obj, offset, expect, update);
	}


BufferedInputStream extends FilterInputStream

	AutoCloseable(jdk1.7)对象的close()方法在退出try- with-resources块时被自动调用，
	该块在资源规范头中声明了该对象。此构造确保及时释放，避免资源耗尽异常和可能出现的错误。 

close方法

```java
public void close() throws IOException {
	byte[] buffer;
	while ( (buffer = buf) != null) {
		if (bufUpdater.compareAndSet(this, buffer, null)) {
			InputStream input = in;
			in = null;
			if (input != null)
				input.close();
			return;
		}
		// Else retry in case a new buf was CASed in fill()
	}
}
```

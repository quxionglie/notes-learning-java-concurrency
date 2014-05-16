不变模式immutable
======

当对象产生后，不发生任何改变。

适合场景：

(1)	当对象被创建后，其内部状态和数据不再发生变化。

(2)	对象需要共享、被多线程频繁访问。

```java
public final class Product {
	private final String  no;
	private final String name;
	private final double price;
	
	public Product(String no, String name, double price) {
		super();
		this.no = no;
		this.name = name;
		this.price = price;
	}
	
	public String getNo() {
		return no;
	}
	public String getName() {
		return name;
	}
	public double getPrice() {
		return price;
	}
}
```

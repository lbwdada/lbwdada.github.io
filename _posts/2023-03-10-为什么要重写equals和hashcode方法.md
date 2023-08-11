---
title: 为什么要重写equals和hashcode方法
categories:
- java
tags:
- 学习记录
---

在使用HashMap的时候一次突发奇想

<!--  more  -->

## 举例说明

```java
import java.util.HashMap;

class User{
private Integer id;private Stirng name;

public User(Integer id, String name) {
this.id = id;this.name=name;
}

public Integer getId() {
return id;
}

public Integer getName() {
return name;
}

// @Override
// public int hashCode() {
// final int prime = 31;
// int result = 1;
// result = prime * result + ((id == null) ? 0 : id.hashCode());
// result = prime * result + ((name == null) ? 0 : name.hashCode());
// return result;
// }
//
// @Override
// public boolean equals(Object obj) {
// if (this == obj)
// return true;
// if (obj == null)
// return false;
// if (getClass() != obj.getClass())
// return false;
// User other = (User) obj;
// if (id == null) {
// if (other.id != null)
// return false;
// } else if (!id.equals(other.id))
// return false;
// if (name == null) {
// if (other.name != null)
// return false;
// } else if (!name.equals(other.name))
// return false;
// return true;
// }
}

public class TestHashCode {
public static void main(String[] args) {
User user1= new User(1,”zhangsan”);
User user2= new User(1,”zhangsan”);
HashMap<User, String> userMap= new HashMap<User, String>();
userMap.put(user1, "我是第一个用户");
System.out.println(userMap.get(user2));
}}
```

> 在main方法里面定义了两个对象user1和user2，但是user里面的属性对象是一样的，接着创建一个HashMap叫做userMap，将user1存进去，然后用user2来来取值，取出来的值为null。
>
> 这里的原因很简单，因为没有重写hash和equals方法，当我们用user2来取直的时候，就会用user2的地址值生成一个hash值，因为user1和user2是完全两个对象所以生成的hash肯定是不同的。

举个例子，效果如下图所示。

![](https://cdn.jsdelivr.net/gh/lbwdada/Mybolg_img/2023-03-10/%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E9%87%8D%E5%86%99equals%E5%92%8Chashcode%E6%96%B9%E6%B3%95/Snipaste_2023-03-10_19-16-40.png)

> 当我们把user1存进去的时候，根据地址值生成的索引是1010，user1就存放在对应的位置，接着我们用user2来取值的时候，根据user2来生成的地址值是3030的索引值所对应的位置，所以就返回为null了。由于user1和user2是完全不同的对象，两份地址值是不一样的，因为没有重写hashCode方法，就会沿用父类Object中的hashCode的方法来生成Hash值，而Object中的hashCode方法生成的hash值是根据对象的地址值直接生成的，所以两个对象的hash就会完全不一样。

**当我们把hashCode方法的注释去点后，但是equals的方法不去掉，也进行测试，返回的结果还是为空，这又是为什么呢？**

  原因很简单，因为当我们把user1存进去的时候，hash值是根据user1中的属性生成hash值，因为user1和user2有相同的属性值，所以生成的hash值是一样的，当用user2来取值同样也能找到user1，但是此时还会进行equals方法的比较，而由于没有进行重写equals，所以使用的是Object的equals的方法进行的比较，而Object中的equals方法是直接使用==来比较，实际还是比较的是对象的地址值，这样比较肯定user1不等于user2啦，所以就返回空，这个就是HashMap的完整的取值过程。这也就是为什么要重写hashCode和equals方法了，在HashMap中存储了自定义对象后，要进行取值的过程，会会进行hashCode和equals方法的比较，假如不重写，那么使用的就是Object中的hashCode和equals中的方法，都是利用地址值来比较，这样就会出问题了，我们更期望的是，当来两个不同的对象，但是他们的属性值是完全一样的，我们认为这是同一个东西，这个才是我们真正期望的。

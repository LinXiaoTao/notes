# 覆盖和隐藏方法



## 实例方法

具有相同签名（名称，加上其参数的数量和类型）的子类中的实例方法以及作为超类中的实例方法的返回类型将覆盖超类的方法。



## 静态方法

如果一个子类在超类中定义了一个与静态方法具有相同签名的静态方法，那么子类中的方法隐藏超类中的方法。隐藏静态方法和重写实例方法之间的区别具有重要意义：

* 被调用的重写实例方法的版本是子类中的一个。
* 被调用的隐藏静态方法的版本取决于它是从超类还是子类调用。



## 接口方法

接口中的缺省方法和抽象方法像实例方法一样被继承。但是，当一个类或接口的超类型提供了具有相同签名的多个缺省方法时，Java 编译器遵循继承规则来解决名称冲突。这些规则是由以下两个原则驱动的：

* 实例方法优先于接口默认方法。

  ``` java
  public class Horse {
      public String identifyMyself() {
          return "I am a horse.";
      }
  }
  public interface Flyer {
      default public String identifyMyself() {
          return "I am able to fly.";
      }
  }
  public interface Mythical {
      default public String identifyMyself() {
          return "I am a mythical creature.";
      }
  }
  public class Pegasus extends Horse implements Flyer, Mythical {
      public static void main(String... args) {
          Pegasus myApp = new Pegasus();
        //实例方法优先于接口方法
          System.out.println(myApp.identifyMyself());
        	//I am a horse.
      }
  }
  ```

* 已被其他候选人覆盖的方法将被忽略。当超类型共享一个共同的祖先时，就会出现这种情况。

  ``` java
  public interface Animal {
      default public String identifyMyself() {
          return "I am an animal.";
      }
  }
  public interface EggLayer extends Animal {
      default public String identifyMyself() {
          return "I am able to lay eggs.";
      }
  }
  public interface FireBreather extends Animal { }
  public class Dragon implements EggLayer, FireBreather {
      public static void main (String... args) {
          Dragon myApp = new Dragon();
        	//就近原则
          System.out.println(myApp.identifyMyself());
        //I am able to lay eggs.
      }
  }
  ```

  如果两个或多个独立定义的默认方法冲突，或者缺省方法与抽象方法冲突，则Java编译器会产生编译器错误。您必须显式重写超类型方法。

  当实现多个接口时，可以使用 `superface.super.method` 调用直接实现的接口定义或者继承的一个默认调用方法。

  ``` java
  public interface OperateCar {
      // ...
      default public int startEngine(EncryptedKey key) {
          // Implementation
      }
  }
  public interface FlyCar {
      // ...
      default public int startEngine(EncryptedKey key) {
          // Implementation
      }
  }

  public class FlyingCar implements OperateCar, FlyCar {
      // ...
      public int startEngine(EncryptedKey key) {
          FlyCar.super.startEngine(key);
          OperateCar.super.startEngine(key);
      }
  }
  ```

  继承于实例类的方法可以覆盖实现的抽象接口方法：

  ``` java
  public interface Mammal {
      String identifyMyself();
  }
  public class Horse {
      public String identifyMyself() {
          return "I am a horse.";
      }
  }
  public class Mustang extends Horse implements Mammal {
      public static void main(String... args) {
          Mustang myApp = new Mustang();
        	//I am a horse.
          System.out.println(myApp.identifyMyself());
      }
  }
  ```

  注意：接口中的静态方法永远不会被继承。



## 修饰符

重写方法的访问说明符可以允许比重写的方法更多，但不是更少的访问。例如，超类中的 protected 实例方法可以在子类中 public，但不是 private。

如果您尝试将超类中的实例方法更改为子类中的静态方法，则会收到编译时错误，反之亦然。
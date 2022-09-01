# SOLID

#### SINGLE RESPONSIBILITY PRINCIPLE

Abbreviated with SRP, this principle states (very intuitively) that a class should only have a single reason to change. In other words, you should create classes with a single “responsibility” so that they’re easier to maintain and harder to break.

```dart
class Shapes {
  List < String > cache = List < > ();

  // Calculations
  double squareArea(double l) {
    /* ... */ }
  double circleArea(double r) {
    /* ... */ }
  double triangleArea(double b, double h) {
    /* ... */ }

  // Paint to the screen
  void paintSquare(Canvas c) {
    /* ... */ }
  void paintCircle(Canvas c) {
    /* ... */ }
  void paintTriangle(Canvas c) {
    /* ... */ }

  // GET requests
  String wikiArticle(String figure) {
    /* ... */ }
  void _cacheElements(String text) {
    /* ... */ }
}
```

This class totally destroys the SRP as it handles internet requests, paintings, and calculations all in one place. This class is going to change very often in the future: whenever a method requires maintenance, you’ll have to change some code and this may potentially break other parts of the class. What about this?

```dart
// Calculations and logic
abstract class Shape {
  double area();
}

class Square extends Shape {}
class Circle extends Shape {}
class Rectangle extends Shape {}

// UI painting
class ShapePainter {
  void paintSquare(Canvas c) {
    /* ... */ }
  void paintCircle(Canvas c) {
    /* ... */ }
  void paintTriangle(Canvas c) {
    /* ... */ }
}

// Networking
class ShapesOnline {
  String wikiArticle(String figure) {
    /* ... */ }
  void _cacheElements(String text) {
    /* ... */ }
}
```

Now each class has consistent methods and overall they have a single responsibility (respectively: making calculations, painting to the UI and fetching data from the Internet).

### OPEN-CLOSED PRINCIPLE

The open-closed principle states that in good architecture the developer should add new behaviors without changing the existing source code. This concept is notoriously described as “classes should be open for extension and closed for changes”. Look at this example:

```dart
class Rectangle {
  final double width;
  final double height;
  const Rectangle(this.width, this.height);
}

class Circle {
  final double radius;
  const Circle(this.radius);

  double get PI => 3.1415;
}

class AreaCalculator {
  double calculate(Object shape) {
    if (shape is Rectangle) {
      return r.width * r.height;
    } else {
      final c = shape as Circle;
      return c.radius * c.radius * c.PI;
    }
  }
}
```

Note that Rectangle and Circle respect the SRP as they only have a single responsibility. The problem is inside AreaCalculator, because if we added other shapes, we would have to add more if conditions, and thus the calculate method would require maintenance.

```dart
class AreaCalculator {
  double calculate(Object shape) {
    if (shape is Rectangle) {
      return shape.width * shape.height;
    } else if (shape is Triangle) {
      return shape.base * shape.height / 2;
    } else if (shape is Square) {
      return shape.side * shape.side;
    } else {
      final c = shape as Circle;
      return c.radius * c.radius * c.PI;
    }
  }
}
```

The code itself may also become hard to read. Thanks to abstraction and dependency injection, we can do much better.

```dart
abstract class Area {
  const Area();

  double computeArea();
}

class Rectangle extends Area {
  final double width;
  final double height;
  const Rectangle(this.width, this.height);

  @override
  double computeArea() => width * height;
}

class Circle extends Area {
  final double radius;
  const Circle(this.radius);

  @override
  double computeArea() => radius * radius * 3.1415;
}

class AreaCalculator {
  double calculate(Area shape) => shape.computeArea();
}
```

Thanks to inheritance, now we have the possibility to add or remove as many classes as we want without changing AreaCalculator. For example, if we added class Triangle extends Area {}, we wouldn’t need to update the AreaCalculator.calculate(double) because the overridden method is inherited.

#### LISKOV SUBSTITUTION PRINCIPLE

The LSP states that subclasses should be replaced with superclasses without changing the logical correctness of the program. In simpler terms, it means that a subtype must guarantee the “usage conditions” of its supertype plus some more behaviors. For example:

```dart
class Rectangle {
  double width;
  double height;
  const Rectangle(this.width, this.height);
}

class Square extends Rectangle {
  const Square(double length): super(length, length);
}
```

The code compiles but there is a bad logic error. All the sides of a square must have the same length but a rectangle doesn’t have this restriction. However, we can do this at the moment:

```dart
void main() {
  final Rectangle error = Square(3);

  // Creating a square with various sides lengths... what??
  error.width = 4;
  error.height = 8;
}
```

Whoops! Now we have a square with different length sides (which is… impossible). The LSP is broken because this architecture does not guarantee that the subclass will maintain the logical correctness of the hierarchy. The solution is to simply make Rectangle and Square two separate classes and remove the inheritance relationship.

--------------

Năm 1988, Barbara Liskov đã phát biểu những điều sau như một cách để định nghĩa các subtype:

> Nếu đối với mỗi đối tượng *o1* thuộc kiểu **S** có một đối tượng *o2* thuộc kiểu **T** sao cho tất cả các chương trình **P** được định nghĩa theo **T**, thì behaviour của **P** không thay đổi khi *o1* được thay thế bằng *o2*, thì **S** được coi là một subtype của **T**.

Nghe mấy cái này mình lại nhớ về mấy cái định lý toán học ngày xưa, nó hàn lâm quá nhưng đôi khi việc định nghĩa cần phải chặt chẽ nên nó vậy.

Nếu chúng ta liên hệ đến tính kế thừa trong lập trình hướng đối tượng rồi đọc lại định nghĩa trên thì sẽ hiểu ngay thôi. Vì đây là một nguyên tắc trong thiết kế nên nó cần được phát biểu tổng quát để cho những trường hợp rộng hơn.

Dịch ra từ chính cái tên, thì đây về cơ bản là nguyên tắc của việc thay thế được tạo ra bởi ông Liskov. Để hiểu về nguyên tắc này, chúng ta hãy cùng xem xét một số ví dụ sau.

## Cách sử dụng đúng đắn tính Kế thừa (Inheritance)

Giả sử chúng ta có một class tên là `License` như hình dưới đây. Class này có một method tên là `calcFee()`, mà được gọi bởi ứng dụng `Billing`. Chúng ta có 2 subtype của `License`: `PersonalLicense` và `BusinessLicense`. Các class này sử dụng các thuật toán khác nhau để tính phí license (kiểu phí bản quyền/giấy phép cho bạn nào không biết license là cái gì).

![](https://images.viblo.asia/ad1928d5-706c-44bd-91b0-04c1c3be4051.png)

Thiết kế này được nói là tuân theo LSP (Liskov Substitution Principle) vì behaviour của ứng dụng `Billing` không bị phụ thuộc vào việc ta sử dụng subtype nào trong hai cái trên. Cả hai đều có thể thay thế cho `License`.

## Vấn đề về Square và Rectangle

Một ví dụ kinh điển về vi phạm LSP là mối liên kết giữa Square (hình vuông) và Rectangle (hình chữ nhật).

![](https://images.viblo.asia/2278bf9a-eff1-41e2-914c-fe4718824e1f.png)Chúng ta đều biết trong toán học thì hình vuông được coi là một hình chữ nhật. Cách hiểu này gây nên cách định nghĩa subtype như hình trên.

Thực tế, việc định nghĩa `Square` là một subtype của `Rectangle` là không đúng cho lắm. Bởi vì `width` và `height` của `Rectangle` có thể khác nhau còn của `Square` thì không thể.

Và vì `User` nghĩ rằng đang tương tác với một `Rectangle` nên sẽ rất dễ để dẫn đến sai lầm như đoạn code dưới đây:

```none
Rectangle r = ...

r.setW(5);

r.setH(2);

assert(r.area() == 10);
```

Nếu cái đoạn `...` kia khởi tạo một `Square`, thì lệnh assert sẽ trả fail. Bởi vì với hình vuông thì khi ta thay đổi `width`, thì `height` sẽ phải đổi theo.

Cách duy nhất để đối phó với kiểu vi phạm LSP này là thêm một đoạn logic check kiểu ở trong `User` để xem là đang tương tác với `Square` hay là `Rectangle`. Bởi vì behaviour của `User` thay đổi dựa trên kiểu mà nó tương tác, theo LSP, `Square` không phải là một subtype phù hợp của `Rectangle`.

## LSP và Architecture

Trong những năm đầu của cuộc cách mạng hướng đối tượng, người ta nghĩ rằng LSP là một cách để định hướng cho việc sử dụng tính kế thừa, như ta vừa nói ở phần trước. Tuy nhiên qua thời gian, LSP đã biến thành một nguyên tắc thiết kế phần mềm rộng hơn liên quan đến interface và implementation.

Các interface được nhắc đến ở đây có thể ở nhiều dạng. Đó có thể là kiểu Java interface mà được implement bởi các class. Hoặc có thể là các Ruby class chia sẻ cùng những method signature. Hoặc có thể là một tập các service đều để respond cho một REST interface.

Trong tất cả những trường hợp đó, LSP được áp dụng bởi vì có những người dùng phụ thuộc vào các interface được định nghĩa rõ ràng, và dựa vào khả năng thay thế của các implementation của các interface đó.

Cách tốt nhất để hiểu LSP từ góc nhìn kiến trúc là hãy xem xem điều gì sẽ xảy ra với kiến trúc của hệ thống khi nguyên tắc này bị vi phạm.

## Ví dụ về vi phạm LSP

Giả sử rằng chúng ta đang xây dựng một ứng dụng tổng hợp cho nhiều dịch vụ đưa đón taxi. Khách hàng sử dụng website của chúng ta để gọi chiếc xe taxi thích hợp nhất để sử dụng, mà không cần quan tâm đến hãng taxi. Một khi khách hàng tạo một lựa chọn, hệ thống của chúng ta sẽ gửi taxi được chọn dựa vào một restful service.

Giờ giả dụ rằng một URI cho restful dispatch service (dispatch là ý là gọi xe đó) là một phần của thông tin chứa trong cơ sở dữ liệu của các tài xế. Khi hệ thống của chúng ta chọn một tài xế thích hợp cho khách hàng, nó sẽ lấy URI đó từ bản ghi của tài xế và sau đó sử dụng nó để gửi lời gọi xe đến tài xế.

Giả sử tài xế Bob có một dispatch URI trông như thế này:

```none
purplecab.com/driver/Bob
```

Hệ thống của chúng ta sẽ thêm thông tin gọi xe vào URI này và gửi đi với method PUT, như sau:

```none
purplecab.com/driver/Bob
        /pickupAddress/24 Maple St.
        /pickupTime/153
        /destination/ORD
```

Rõ ràng thì điều này có nghĩa là tất cả các dịch vụ gọi xe, ở tất cả các công ty khác nhau, phải tuân theo một REST interface chung. Chúng phải xét các trường `pickupAddress`, `pickupTime`, `destination` theo một cách giống nhau.

Giờ giả sử rằng công ty taxi Acme tuyển một vài lập trình viên mà không chịu đọc specs kỹ càng. Họ viết tắt trường `destination` thành `dest`. Giả dụ như Acme là một công ty taxi lớn nhất vùng, và vợ cũ của CEO của Acme là vợ mới của CEO chúng ta... Well, bạn hiểu đấy, đại thể là với nhiều lý do mà chúng ta không bắt thằng Acme nó đổi được. Vậy thì điều gì sẽ xảy ra với kiến trúc của hệ thống này?

Tất nhiên, giờ chúng ta lại phải thêm vào logic của một case đặc biệt. Yêu cầu gọi xe cho bất cứ tài xế Acme nào sẽ phải được cấu trúc sử dụng một tập các rule khác so với tất cả những tài xế còn lại.

Cách đơn giản nhất thì là thêm một lệnh rẽ nhánh `if` vào module mà sử dụng đế cấu trúc lên lệnh gọi xe:

```none
if (driver.getDispatchUri().startsWith("acme.com"))...
```

Nhưng tất nhiên là chẳng có kiến trúc sư nào cho phép một cấu trúc như vậy tồn tại trong hệ thống của mình. Việc đưa từ "acme" vào trong code sẽ tạo cơ hội cho những lỗi rất kinh khủng và ma giáo xảy ra, còn chưa kể đến vấn đề bảo mật.

Ví dụ, sẽ thế nào nếu Acme làm ăn ra tiền và mua lại công ty Purple Taxi. Sẽ thế nào nếu việc sát nhập công ty vẫn duy trì hai thương hiệu riêng biệt, nhưng lại hợp nhất hệ thống của Purple vào Acme? Liệu của chúng ta có phải thêm một lệnh `if` nữa cho "purple"?

Người kiến trúc sư cần phải cách ly hệ thống khỏi những bug như thế này bằng việc tạo ra một số module tạo lệnh gọi xe mà có thể điều phối được bằng một cấu hình database dựa trên khoá bởi các dispatch URI. Dữ liệu cấu hình có thể trông như sau:

| URI        | Dispatch Format                                  |
| ---------- | ------------------------------------------------ |
| `Acme.com` | `/pickupAddress/%s/pickupTime/%s/dest/%s`        |
| `*.*`      | `/pickupAddress/%s/pickupTime/%s/destination/%s` |

Như vậy, người kiến trúc sư sẽ phải thêm một cơ chế quan trọng và phức tạp để đối phó với một thực tế là các interface của restful service không hoàn toàn thay thế được.

## Tóm lại

LSP có thể, và nên, được mở rộng tới cấp độ kiến trúc. Một vi phạm đơn giản về khả năng thay thế, có thể dẫn đến một kiến trúc hệ thống "thối" với một lượng đáng kể các cơ chế phụ để xử lý.

---

*Dịch và tham khảo từ [Clean Architecture: A Craftsman's Guide to Software Structure and Design (Robert C. Martin Series)](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)*

-----------

#### INTERFACE SEGREGATION PRINCIPLE

This principle states that clients don’t have to implement a behavior they don’t need. The gist of this principle is: you should create small interfaces with minimal methods. Let’s look at this example:

```dart
abstract class Worker {
  void work();
  void sleep();
}

class Human implements Worker {
  void work() => print("I do a lot of work");
  void sleep() => print("I need 10 hours per night...");
}

class Robot implements Worker {
  void work() => print("I always work");
  void sleep() {} // ??
}
```

The problem here is that robots don’t need to sleep so we don’t know how to implement that method. Worker represents an entity able to make a job and it assumes that if you can work, then you can also sleep. In our case this assumption is not always valid, so let’s split the class:

```dart
abstract class Worker {
  void work();
}

abstract class Sleeper {
  void sleep();
}

class Human implements Worker, Sleeper {
  void work() => print("I do a lot of work");
  void sleep() => print("I need 10 hours per night...");
}

class Robot implements Worker {
  void work() => print("I always work");
}
```

This is much better. Now Robot is not forced to implement sleep() anymore and we have loosened the constraints for implementers.

#### DEPENDENCY INVERSION PRINCIPLE

The DIP states that we should favor abstractions over implementations. Extending an abstract class or implementing an interface is good but descending from a concrete class is bad. We’ve already seen why in the open-closed principle:    

```dart
abstract class EncryptionAlgorithm {
  const EncryptionAlgorithm();

  String encrypt(); // <-- abstraction
}

class AlgoAES implements EncryptionAlgorithm {
  /* .. */ }
class AlgoRSA implements EncryptionAlgorithm {
  /* .. */ }
class AlgoSHA implements EncryptionAlgorithm {
  /* .. */ }
The usage of abstractions gives the freedom to be independent from the implementation.The point is that anyone using the EncryptionAlgorithm only knows about the encrypt() method and the other internal details of the class don 't matter at all!
class FileManager {
  void secureFile(EncryptionAlgorithm algo) => algo.encrypt()
}
```

FileManager doesn’t care if we add one or more methods to AlgoRSA, for example, because it only cares about the encryption() method. It only knows that encrypt() exists: no matter how it works internally, FileManager only cares about the return type (if any).

```dart
void main() {
  const fm = FileManager();

  fm.secureFile(const AlgoAES());
  fm.secureFile(const AlgoRSA());
}
```

[docs](https://www.topcoder.com/thrive/articles/solid-principles-in-dart)

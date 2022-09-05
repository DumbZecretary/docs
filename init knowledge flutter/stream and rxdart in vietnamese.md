# Stream and RxDart

# 1. Stream là gì, Event là gì

Có thể hình dung cái băng chuyền này là 1 `Stream`.

![](https://images.viblo.asia/0d036b0a-1c12-44a7-a704-2222b3d0b447.gif)

Định nghĩa nguyên văn từ doc:

> A stream is a sequence of asynchronous events

Dịch nôm na ra thì Stream là một chuỗi các `events` bất đồng bộ. Các khối hộp đang trượt trên băng kia chính là các `events`. `Event` có thể là một `data`, cũng có thể là một `error`, hoặc một trạng thái `done` (Trong phần Tạo ra Stream ở dưới mình sẽ nói rõ hơn). Các `events` này được xử lý bất đồng bộ. Như vậy `Stream` là một chuỗi các `events` được xử lý bất đồng bộ.

Thực tế Stream nó không có chạy mãi như cái ảnh gif đâu nha. Nó có thể được tạo ra, cũng có thể bị đóng lại và ta cũng có thể tạm dừng cái băng chuyền đó (pause) và tiếp trục cho băng chuyền chạy (resume)

Trong quá trình truyền data, có thể qua 1 số khâu chế biến trung gian để cho ra output. Như cái ảnh dưới đây, dữ liệu đầu vào là các `chữ thường` qua một hàm xử lý gọi là `map` để cho ra những data đầu ra là các `chữ in hoa`. Các hàm như vậy thì trong lớp `Stream` của Dart cũng có tuy nhiên còn hạn chế. Chính vì vậy mà chúng ta mới cần đến `RxDart` - một thư viện bổ sung sức mạnh thêm cho `Stream`.

![](https://images.viblo.asia/e3a6c928-9fe3-4825-900e-36f7977aa59c.gif)

Nếu các bạn đã từng biết RxJava hay RxJs,... thì định nghĩa `Stream` chính là định nghĩa của `Observable` trong Rx.

# 2. Tạo ra Stream, lắng nghe Stream và giải thích thuật ngữ

## emit là gì

Hành động phát ra `Event` của Stream người ta gọi là `emit`

## Tạo Stream

Có rất là nhiều cách để tạo ra 1 stream. Ở đây mình tạm giới thiệu 1 cách đơn giản nhất để giải thích thuật ngữ.

```dart
var stream = Stream.value('gấu bông'); // tạo ra 1 stream và đồng thời emit 1 data là 'gấu bông'
```

Ở đây mình đã tạo ra 1 Stream và emit 1 data là 'gấu bông' nhưng khi run chương trình thì sẽ ko xảy ra gì cả. Là vì mình phát ra con 'gấu bông' nhưng không có ai nhận hết. Muốn nhận được con gấu bông phải đăng ký Stream để được nhận nó khi 'gấu bông' được phát ra. Tiếp theo chúng ta sẽ tiến hành đăng ký Stream trên (subscribe a Stream).

## Subscribe một Stream

Chúng ta sử dụng hàm `listen` để đăng ký nhận events từ stream.

```dart
var stream = Stream.value('gấu bông'); // tạo ra 1 stream và đồng thời emit 1 data là 'gấu bông'
stream.listen((event) { // đăng ký nhận event từ stream
    print('Tôi đã nhận được $event');
  });
```

Output:

```none
Tôi đã nhận được gấu bông
```

Trong RxJava, ..., người ta còn gọi cái lambda này `(event) { print('Tôi đã nhận được $event'); }` là 1 `Observer`

Trong hàm `listen` chúng ta có thêm các [optional parameter](https://viblo.asia/p/hoc-nhanh-ngon-ngu-dart-flutter-nho-ngon-ngu-kotlin-phan-3-5-Qbq5Qm935D8#_5-named-parameters-4) như `onDone`, `onError`

Một Stream khi emit xong tất cả event thì nó sẽ emit event tại `onDone`, nếu gặp lỗi thì nó sẽ emit một error và `onError` sẽ giúp ta nhận được event đó.

```dart
var stream = Stream.value('gấu bông'); // tạo ra 1 stream và đồng thời emit 1 data là 'gấu bông' và done luôn
  stream.listen((event) {
    print('Tôi đã nhận được $event');
  }, onDone: () => print('Done rồi'), 
      onError: (error) => print('Lỗi rồi $error'));
```

Output: Do không gặp lỗi nên output sẽ là thế này.

```none
Tôi đã nhận được gấu bông
Done rồi
```

## Thêm nhiều cách tạo Stream

Bây giờ mình sẽ giới thiệu thêm nhiều cách tạo ra Stream nữa:

- Tạo ra một Stream đồng thời emit 1 error và done luôn.

```dart
Stream.error(FormatException('lỗi nè')).listen(print, onError: print, onDone: () => print('Done!'));
```

Output:

```none
FormatException: lỗi nè
Done!
```

- Tạo ra một Stream đồng thời emit 1 List data xong rồi Done.

```dart
Stream.fromIterable([1, 2, 3]).listen(print, onError: print, onDone: () => print('Done!'));
```

Output:

```none
1
2
3
Done!
```

- Tạo ra một Stream từ 1 [Future](https://viblo.asia/p/hoc-nhanh-ngon-ngu-dart-flutter-nho-ngon-ngu-kotlin-phan-5-5-Ljy5Vqjolra#_async-va-future-5).

```dart
void main() {
  Stream.fromFuture(testFuture()).listen(print, onError: print, onDone: () => print('Done!'));
}

Future<int> testFuture() async {
  return 3;
}
```

Output:

```none
3
Done!
```

- Tạo ra một stream cứ 1 giây sẽ emit ra 1 event. Các bạn chú ý là `error` nó cũng là `event` nên khi Stream emit ra error thì Stream đó không dừng lại.

```dart
Stream.periodic(Duration(seconds: 1), (int i) {
    if(i == 5) {
      throw Exception('lỗi');
    }
    if (i % 2 == 0) {
      return '$i là số chẵn';
    } else {
      return '$i là số lẻ';
    }
  }).listen(print, onError: print, onDone: () => print('Done!'));
```

Output:

```none
0 là số chẵn
1 là số lẻ
2 là số chẵn
3 là số lẻ
4 là số chẵn
Exception: lỗi
6 là số chẵn
7 là số lẻ
8 là số chẵn
.......
```

Stream này nó bất tử lun ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png). Bây giờ mình sẽ sử dụng StreamSubscription để trảm nó, không cho nó bất tử nữa!

# 3. StreamSubscription là gì

Hàm `listen` ở trên sẽ trả về một đối tượng StreamSubscription. StreamSubscription giúp ta điều khiển stream. Ta có thể pause stream, resume stream và hủy stream.

```dart
void main() async {
  var subscription = Stream.periodic(Duration(seconds: 1), (int i) {
    if(i == 5) {
      throw Exception('lỗi');
    }
    if (i % 2 == 0) {
      return '$i là số chẵn';
    } else {
      return '$i là số lẻ';
    }
  }).listen(println, onError: println, onDone: () => println('Done!'));

  // Sau 3 giây kể từ lúc run chương trình ta sẽ pause stream
  await Future.delayed(Duration(seconds: 3), () {
    println('pause stream');
    subscription.pause();
  });

  // Sau 2 giây kể từ lúc pause stream ta sẽ resume stream
  await Future.delayed(Duration(seconds: 2), () {
    println('resume stream');
    subscription.resume();
  });

  // Sau 3 giây kể từ lúc resume stream ta sẽ cancel stream
  await Future.delayed(Duration(seconds: 3), () {
    println('cancel stream');
    subscription.cancel();
  });
}

// mình sử dụng hàm println này để in ra thời gian hiện tại cho dễ quan sát output
void println(Object value) {
  print('${DateTime.now()}: $value');
}
```

Output:

```none
2020-08-29 21:53:32.882979: 0 là số chẵn
2020-08-29 21:53:33.881757: 1 là số lẻ
2020-08-29 21:53:34.880876: 2 là số chẵn
2020-08-29 21:53:34.884870: pause stream
2020-08-29 21:53:36.888422: resume stream
2020-08-29 21:53:37.890725: 3 là số lẻ
2020-08-29 21:53:38.891863: 4 là số chẵn
2020-08-29 21:53:39.891114: cancel stream

Process finished with exit code 0
```

Và chúng ta đã có thể stop được stream bất tử ở trên thành công bằng hàm `cancel()`. Người ta thường gọi hành động này là `unsubscribe một Stream`.

# 4. Stream Builder, async*, yield và yield*

Trong thực tế, người ta lại rất ít khi dùng các cách trên để tạo stream mà họ sử dụng cái gọi là `stream builder`, ở đó họ có thể emit data bất cứ thời điểm nào họ muốn.

Để tạo một stream builder ta tạo 1 function với `async*` và trả về một `Stream`. Để emit một data ta có thể sử dụng `yield`, để emit tất cả data trong 1 stream ta có thể sử dụng `yield*`

```dart
void main() {
  testStream().listen(println, onError: println, onDone: () => println('Done!'));
}

Stream<int> testStream() async* {
  yield 10; // emit 10
  await Future.delayed(Duration(seconds: 2)); // delay 2s
  yield* Stream.fromIterable([1, 2, 3]); // emit nguyên 1 stream
  await Future.delayed(Duration(seconds: 3)); // delay 3s
  throw FormatException('lỗi');
  yield 13; // hàm này đã xảy ra Exception nên số 13 không được phát ra
}

// mình sử dụng hàm println này để in ra thời gian hiện tại cho dễ quan sát output
void println(Object value) {
  print('${DateTime.now()}: $value');
}
```

Output:

```none
2020-08-30 09:23:23.965913: 10
2020-08-30 09:23:25.975207: 1
2020-08-30 09:23:25.975207: 2
2020-08-30 09:23:25.975207: 3
2020-08-30 09:23:28.977575: FormatException: lỗi
2020-08-30 09:23:28.978575: Done!
```

# Kết luận

Qua phần 1 của series này, hy vọng các bạn đã nắm được các thuật ngữ quan trọng trong Stream cũng như trong lập trình bất đồng bộ của Dart. Các thuật ngữ này rất quan trọng đối với loạt bài tiếp theo trong series này. Trong bài tiếp theo mình sẽ giới thiệu đến các bạn các toán tử của Stream và Stream Controller, Broadcast Stream. Hy vọng các bạn đón đọc ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png).

Nguồn tham khảo: [Asynchronous programming: Streams | Dart](https://dart.dev/tutorials/language/streams)

Đọc bài tiếp theo: [Chinh phục RxDart Flutter trong 3 nốt nhạc. Nốt nhạc thứ 2: Tiếp tục với 'Sờ tream'](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W)

----

# 1. Stream Operators

Như mình đã giới thiệu ở bài 1, operators là những function được cung cấp để giúp ta dễ dàng xử lý `event` trong Stream. Có thể từ 1 Stream chế biến ra 1 Stream khác (`Stream transformation`), hoặc chỉ đơn giản là get 1 giá trị nào đó trong Stream (`Receiving stream events`), khi đó giá trị đó sẽ được trả dưới dạng một `Future`. Vì vậy mình sẽ chia thành 2 nhóm nhỏ: nhóm trả về `Future` và nhóm trả về `Stream` khác. Vì số function trong mỗi nhóm là rất nhiều nên mình sẽ giới thiệu 5 function điển hình, hay dùng.

![](https://images.viblo.asia/e3a6c928-9fe3-4825-900e-36f7977aa59c.gif)

## I. Nhóm function trả về Future

### 1. elementAt

Hàm này giúp ta get được phần tử tại index bất kỳ trong Stream

```dart
void main() async {
  var future = await testStream().elementAt(2); // get data tại index = 2
  print(future); // print ra: 15
}

// xuyên suốt các ví dụ mình sẽ sử dụng Stream này
Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

### 2. length

Hàm này giúp ta get được số lượng phần tử có trong stream.

```dart
void main() async {
  print(await testStream().length); // print: 4
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

### 3. firstWhere

Hàm này giúp ta get được phần tử đầu tiên thỏa mãn một điều kiện được chúng ta cho trước. Ví dụ dưới đây mình muốn get ra phần tử đầu tiên có giá trị lớn hơn 11. Đó chính là 15

```dart
void main() async {
  print(await testStream().firstWhere((element) => (element > 11))); // print: 15
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

### 4. join

Hàm này nó nối tất cả element thành 1 `String` duy nhất, giữa các element được ngăn cách bởi 1 ký tự separator do chúng ta truyền vào.

```dart
void main() async {
  print(await testStream().join(',')); // print: '5,10,15,20'
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

### 5. await for

Cú pháp `await for` giúp chúng ta tính tổng các phần tử trong Stream một cách bất đồng bộ. Vì là tính bất đồng bộ nên nó sẽ tính nhanh hơn dùng for thông thường rất nhiều.

```dart
void main() async {
  var sum = 0;
  await for (final value in testStream()) {
    sum += value;
  }

  print(sum); // print: 50
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

Mình thấy trong lớp `Stream` không có hàm `sum` mà dùng cú pháp `await for` đó thì nó dài dòng quá nên chúng ta có thể viết thêm 1 [extension function](https://viblo.asia/p/hoc-nhanh-ngon-ngu-dart-flutter-nho-ngon-ngu-kotlin-phan-3-5-Qbq5Qm935D8#_12-extension-function-13) cho nó:

```dart
void main() async {
  print(await testStream().sum()); // chỉ cần gọi hàm sum là print: 50
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}

// extension function của Stream dành để tính tổng
extension StreamExtensions on Stream<int> {
  Future<int> sum() async {
    var sum = 0;
    await for (final value in this) {
      sum += value;
    }

    return sum;
  }
}
```

***Bonus***: Xem thêm các function khác [tại đây](https://github.com/boriszv/Programming-Addict-Code-Examples/blob/master/12.%20Reactive%20Programming%20Course/3%20-%20Getting%20data%20from%20a%20Stream/main.dart)

## II. Nhóm function trả về Stream

Do các hàm này nó transform (biến đổi) một Stream sang một Stream khác nên để tránh nhầm lẫn, chúng ta sẽ thống nhất đặt cho chúng 2 cái tên là Source Stream và Output Stream như sau:

```none
Source Stream -> transform -> Output Stream
```

### 1. map

Hàm này cho phép chúng ta truyền vào 1 hàm, hàm này sẽ giúp ta biến đổi từng element của Source Stream.

```dart
void main() {
  var outputStream = testStream().map((event) => event / 5); // chia từng phần tử của Source Stream cho 5
  outputStream.listen(print); // subscribe outputStream để nhận kết quả
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

Output:

```none
1.0
2.0
3.0
4.0
```

### 2. where

Hàm này lọc ra những phần tử trong Source Stream thỏa mãn điều kiện cho trước.

```dart
void main() {
  testStream().where((event) => event > 11).listen(print); // lọc ra những phần tử > 11
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

Output:

```none
15
20
```

### 3. takeWhile

Hàm này nó sẽ emit ra các phần tử **CÓ** thỏa mãn điều kiện cho trước cho đến khi có một phần tử **KHÔNG** thỏa mãn điều kiện thì nó dừng Stream.

```dart
void main() {
  testStream().takeWhile((event) => event < 11).listen(print, onDone: () => print('Done!'));
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15; // tới đây gặp 15 KHÔNG pass điều kiện nên dừng stream, ko emit ra 15
  yield 20;
}
```

Output:

```none
5
10
Done!
```

Và như vậy có nghĩa là nếu Source Stream `yield 15` trước tiên thì Output Stream sẽ không có phần tử nào.

```dart
void main() {
  testStream().takeWhile((event) => event < 11).listen(print, onDone: () => print('Done!'));
}

Stream<int> testStream() async* {
  yield 15; // tới đây gặp 15 KHÔNG pass điều kiện nên dừng stream, ko emit ra 15
  yield 10;
  yield 5; 
  yield 20;
}
```

Output:

```none
Done!
```

### 4. skipWhile

Hàm này nó sẽ bỏ qua, không emit ra các phần tử **CÓ** thỏa mãn điều kiện cho trước cho đến khi có một phần tử **KHÔNG** thỏa mãn điều kiện được emit ra.

```dart
void main() {
  testStream().skipWhile((event) => event < 11).listen(print); // bỏ qua các phần tử < 11
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15;
  yield 20;
}
```

Output:

```none
15
20
```

Các bạn chú ý là: `skipWhile` không có lọc theo điều kiện `where` nhé. Thằng `skipWhile` một khi đã emit được 1 phần tử đầu tiên rồi thì nó sẽ không skip bất cứ phần tử nào nữa. Ví dụ như case này:

```dart
void main() {
  testStream().skipWhile((event) => event < 11).listen(print);
}

Stream<int> testStream() async* {
  yield 5; // thằng 5 CÓ pass điều kiện nên bị skip
  yield 15; // thằng 15 KHÔNG pass điều kiện nên được emit ra đầu tiên
  yield 6; // nhờ vậy mà tất cả phần tử sau 15 đều được emit kể cả 6
  yield 20;
}
```

Output:

```none
15
6
20
```

### 5. distinct

Hàm này skip (bỏ qua) các phần tử mà chúng equal (bằng) với phần tử đã được emit trước đó (the previous element).

```dart
void main() {
  testStream().distinct().listen(print, onDone: () => print('Done!'));
}

Stream<int> testStream() async* {
  yield 5; // emit 5
  yield 5; // bỏ qua vì phần tử liền trước là 5
  yield 5; // bỏ qua vì phần tử liền trước là 5
  yield 10; // emit 10 (vì 10 khác thằng liền trước đã emit là 5)
  yield 5; // emit 5 (vì 5 khác thằng liền trước đã emit là 10)
  yield 10; // emit 10 (vì 10 khác thằng liền trước đã emit là 5)
}
```

Output:

```none
5
10
5
10
Done!
```

Ngoài ra, `distinct` còn cho phép chúng ta có thể định nghĩa lại thế nào là bằng nhau.

```dart
void main() {
   // định nghĩa lại: 2 số bằng nhau là khi 2 số đó chia lấy phần nguyên với 10 ra kết quả bằng nhau 
  testStream().distinct((int previous, int next) => previous ~/ 10 == next ~/ 10)
      .listen(print, onDone: () => print('Done!'));
}

Stream<int> testStream() async* {
  yield 5;
  yield 10;
  yield 15; // không được emit ra vì 10 ~/ 10 = 15 ~/10 = 1
  yield 20;
}
```

Output:

```none
5
10
20
Done!
```

***Bonus***: Xem thêm các function khác [tại đây](https://github.com/boriszv/Programming-Addict-Code-Examples/blob/master/12.%20Reactive%20Programming%20Course/4%20-%20Operators/main.dart)

# 2. Handle errors

Stream có một function giúp ta bắt lỗi trong Stream là `handleError`. Hàm này nó lợi hại hơn [onError](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-phan-1-stream-va-giai-thich-cac-thuat-ngu-Ljy5Vq6blra#_subscribe-mot-stream-5) trong hàm `listen` mà mình đã giới thiệu ở bài 1 ở chỗ. Nó cung cấp cho ta 1 optional param là hàm `test`, ở hàm `test` ta có thể check xem cái lỗi đó có đáng để ta `catch` hay không, nếu không đáng thì ta có thể cho ném lỗi đó luôn không cần `catch` (tương tự [rethrow](https://viblo.asia/p/hoc-nhanh-ngon-ngu-dart-flutter-nho-ngon-ngu-kotlin-phan-2-5-L4x5x3vwlBM#_10-try--catch--finally-11) lỗi).

Hàm `test` này `return true` thì đồng nghĩa với việc chúng ta `catch` lỗi đó, ngược lại `return false` thì chúng ta `rethrow` lỗi đó.

```dart
void main() {
  testStream().handleError(print, test: (error) {
    if (error is HttpException) {
      return true; // return true thì catch lỗi đó lại
    } else {
      return false; // return false thì rethrow lỗi đó lun
    }
  }).listen(print);
}

Stream<int> testStream() async* {
  yield 5;
  throw FormatException('lỗi rồi');
  yield 10;
}
```

Vì `FormatException` không phải là `HttpException` nên hậu quả là máu nhuộm *cờn sô lơ* ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

![](https://images.viblo.asia/722c5167-7374-49a7-b798-32212c1b4687.PNG)

Vì hàm `test` là [optional param](https://viblo.asia/p/hoc-nhanh-ngon-ngu-dart-flutter-nho-ngon-ngu-kotlin-phan-3-5-Qbq5Qm935D8#_5-named-parameters-4) nên nếu chúng ta không truyền vào thì mặc định Dart sẽ hiểu tất cả lỗi trong Stream sẽ được `catch`

```dart
void main() {
  testStream().handleError(print).listen(print);
}

Stream<int> testStream() async* {
  yield 5;
  throw FormatException('lỗi rồi');
  yield 10;
}
```

Output:

```none
5
FormatException: lỗi rồi

Process finished with exit code 0
```

# 3. Stream Controlller

`StreamController` đơn giản chỉ là 1 controller bên trong có 1 stream và controller này giúp ta điều khiển stream đó dễ dàng. Chúng ta có thể thoải mái push các events vào stream, lắng nghe nhận data từ stream đó. Nói cách khác, đây là một con đường khác giúp ta tạo ra 1 Stream và thoải mái emit events và nhận events bất cứ thời điểm nào chúng ta muốn.

Trong mỗi `StreamController` đều có 2 đối tượng là `sink`(giúp chúng ta push events đến `stream`) và `stream` (giúp chúng ta nhận events từ `sink`)

![](https://images.viblo.asia/cfcd1e56-becd-44ca-a2cc-39d1d60e7e8e.png)

```dart
void main() async {
  // tạo stream controller
  var streamController = StreamController();

  // lắng nghe
  streamController.stream.listen(print);

  // push events
  streamController.sink.add('Tui là Minh');
  streamController.sink.add(100);

  // Khi không cần sử dụng controller này nữa thì nên close controller
  await Future.delayed(Duration(seconds: 2)); // sau 2 giây ta sẽ close controller
  await streamController.close();

  // sau khi close mà chúng ta vẫn cố push event sẽ gặp Exception: 
  // Unhandled exception: Bad state: Cannot add new events after calling close
  // streamController.sink.add(11); // cố push event sau khi controller đã close
}
```

Output:

```none
Tui là Minh
100
```

# 4. Broadcast StreamController

Giả sử chúng ta muốn có 2 nguồn lắng nghe events từ `sink`

```dart
void main() {
  // tạo broadcast stream controller
  var streamController = StreamController();

  // subscription thứ nhất
  streamController.stream.listen((event) {
    print('subscription thứ 1: $event');
  });

  // subscription thứ hai sẽ double giá trị của event lên
  streamController.stream.listen((event) {
    print('subscription thứ 2: ${event + event}'); // double value lên
  });

  // push events
  streamController.sink.add('Tui là Minh');
  streamController.sink.add(100);
}
```

Cái mà chúng ta nhận được là máu nhuộm cờn sô lơ:

![](https://images.viblo.asia/2fe507c2-62c1-401f-bd7d-b69ec5ae5337.PNG)

Nó báo lỗi là Stream này đã được lắng nghe rồi, vậy không có cách nào để có nhiều hơn 1 thằng lắng nghe Stream à?

Đừng lo, vì chúng ta đã có `Broadcast StreamController`. Với Broadcast StreamController chúng ta có thể tạo ra bao nhiêu thằng lắng nghe Stream đó cũng được.

Để tạo ra một Broadcast StreamController rất đơn giản, thay vì sử dụng constructor : `StreamController()` để tạo `streamController` thì ta sử dụng constructor `StreamController.broadcast()`. Và đối tượng `stream` trong `streamController` bây giờ người ta gọi là một `Broadcast Stream`.

```dart
void main() {
  // tạo Broadcast StreamController
  var streamController = StreamController.broadcast();

  // subscription thứ nhất
  streamController.stream.listen((event) { // streamController.stream là 1 broadcast stream
    print('subscription thứ 1: $event');
  });

  // subscription thứ hai sẽ double giá trị của event lên
  streamController.stream.listen((event) { // streamController.stream là 1 broadcast stream
    print('subscription thứ 2: ${event + event}');
  });

  // push events
  streamController.sink.add('Tui là Minh');
  streamController.sink.add(100);
}
```

Output:

```none
subscription thứ 1: Tui là Minh
subscription thứ 2: Tui là MinhTui là Minh
subscription thứ 1: 100
subscription thứ 2: 200
```

# Kết luận

Như vậy, chúng ta đã đi hết phần Stream. Sau khi hiểu được Stream bạn đã có thể học được Bloc Pattern mà không cần thiết phải học RxDart. Nếu bạn đang cần học Bloc Pattern thì bạn có thể dễ dàng học ở nguồn khác, có rất nhiều người chia sẻ về nó. Tất nhiên mình vẫn tiếp tục viết về RxDart và cả Bloc Pattern. Cơ mà có bỏ đi thì đừng quên tặng cho mình 1 vote nếu thấy những chia sẻ của mình có giá trị nhé =))

Nguồn tham khảo: [https://github.com/boriszv/Programming-Addict-Code-Examples/tree/master/12. Reactive Programming Course](https://github.com/boriszv/Programming-Addict-Code-Examples/tree/master/12.%20Reactive%20Programming%20Course)

Đọc bài cuối cùng: [Chinh phục RxDart Flutter trong 3 nốt nhạc. Nốt nhạc cuối: RxDart không đáng sợ như bạn nghĩ](https://viblo.asia/p/chinh-phuc-rxdart-flutter-trong-3-not-nhac-not-nhac-cuoi-rxdart-khong-dang-so-nhu-ban-nghi-bWrZn0qp5xw)

-------

# 1. Thêm package vào dự án

Link package: [rxdart | Dart Package](https://pub.dev/packages/rxdart)

Thời điểm mình viết bài này thì latest version của RxDart là 0.24.1 nên mình sẽ sử dụng version này để code demo.

Thêm package vào file `pubspec.yaml` rồi chạy `pub get` để nạp package vào dự án

```dart
dev_dependencies:
  rxdart: ^0.24.1
```

Thêm thư viện xong rồi thì vào file source code import RxDart vào:

```dart
import 'package:rxdart/rxdart.dart';
```

# 2. Tổng quan về RxDart

RxDart là một thư viện cung cấp một lượng class và function rất đồ sộ cho Stream. Vì vậy để dễ thấm được tất cả tuyệt đỉnh kungfu từ RxDart, ta sẽ chia ra 3 nhóm nhỏ để học. Mỗi nhóm nhỏ mình sẽ giới thiệu 1 số class, function đại diện và từ đó các bạn sẽ có nền để chinh phục được tất cả class, function trong mỗi nhóm.

**Nhóm 1 - Những class Stream do RxDart cung cấp**: nó cung cấp những lớp Stream đặc biệt mạnh mẽ hơn lớp `Stream` của Dart rất nhiều như `ZipStream`, `MergeStream`, `ConcatStream`, ....

**Nhóm 2 - Những extension function dành cho lớp Stream**: Ở bài 2 mình đã chia sẻ [rất nhiều function của Stream](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W#_1-stream-operators-1) rồi nhưng nhiêu đó chưa đủ sài đâu, RxDart nó cung cấp thêm cho chúng ta hơn 50 function nữa. Học xong đống function của nó có mà xưng bá võ lâm ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

**Nhóm 3 - Subjects**: Đây là thuật ngữ mới nhưng bản chất nó chính là các [StreamController](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W#_3-stream-controlller-15) trong Dart với nhiều sức mạnh lợi hại hơn ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

# 3. Những class Stream của RxDart

Ở đây, mình sẽ giới thiệu một số class đại diện và hay dùng để từ đó các bạn sẽ có nền để chinh phục được tất cả class. Trong dự án thực tế, chỉ có 1 số ít class là hay sử dụng thôi chứ cũng không cần phải học hết đâu ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

Trước tiên, mình muốn giới thiệu 1 trang webapp: [RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#merge) .Trang này cực kỳ tiện để hiểu các function này hoạt động thế nào bằng cách di chuyển các data (1, 2, 3, 4,...) trong 1 stream. Mặc dù nó sử dụng RxJs tuy nhiên RxJs hay RxDart hay RxJava thì nó cũng giống nhau hết. Hôm nay các bạn học RxDart thì cũng tức là các bạn đã học được cả RxJs và RxJava rồi. Ví dụ link trên là giúp bạn hiểu hàm `merge`. Có rất nhiều hàm trong đó đang chờ bạn khám phá và sẽ còn được tác giả cập nhật thêm nữa ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

## 3.1. MergeStream

Mình mượn cái ảnh trong [RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#merge) rồi chú thích vào, Stream ở chỗ nào, tạo ra Output Stream chỗ nào, event ở chỗ nào, các function sau cứ thế mà khám phá nhá ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

![](https://images.viblo.asia/900e145f-3e5c-4f3c-a517-9ef96ddeced5.jpg)

Nhìn hình là các bạn đủ hiểu rồi phải ko ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png). Ở đây ta có 2 stream gọi là `Stream 1` và `Stream 2` chạy song song nhau và 2 stream đó được **merge lại thành 1 MergeStream, các event trong MergeStream được emit theo thứ tự đúng y hệt dòng thời gian của Stream 1 và Stream 2**. Thực tế bạn thích truyền bao nhiêu Stream vào `MergeStream` để merge lại cũng được. Ở đây mình sẽ merge 2 stream lại bằng cách dựng code giống hệt cái ảnh trên cho các bạn dễ hiểu.

```dart
void main() {
  // biến này để làm cột mốc thời gian lúc bắt đầu run chương trình
  var currentTime = DateTime.now();

  var mergeStream = MergeStream([firstStream(), secondStream()]); // truyền 2 stream vào đây để merge lại
  mergeStream.listen((event) => println(event, currentTime));
}

// stream 1
Stream<int> firstStream() async* {
  await Future.delayed(Duration(seconds: 1)); // 1s sau sẽ emit event đầu tiên
  yield 20; // emit 20 vào giây thứ 1
  await Future.delayed(Duration(seconds: 1)); // nghỉ 1 giây
  yield 40; // emit 40 vào giây thứ 2
  await Future.delayed(Duration(seconds: 2));
  yield 60; // emit 60 vào giây thứ 4
  await Future.delayed(Duration(seconds: 6));
  yield 80; // emit 80 vào giây thứ 10
  await Future.delayed(Duration(seconds: 3));
  yield 100; // emit 100 vào giây thứ 13
}

// stream 2
Stream<int> secondStream() async* {
  await Future.delayed(Duration(seconds: 7)); // sau 7s sẽ emit event đầu tiên
  yield 1;
  await Future.delayed(Duration(seconds: 9));
  yield 1;
}

// mình sử dụng hàm println này để in kèm thời điểm emit event cho dễ quan sát output
void println(Object value, DateTime currentTime) {
  print('Emit $value vào giây thứ ${DateTime.now().difference(currentTime).inSeconds}');
}
```

Output giống hệt `MergeStream` trong cái ảnh trên ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png).

```none
Emit 20 vào giây thứ 1
Emit 40 vào giây thứ 2
Emit 60 vào giây thứ 4
Emit 1 vào giây thứ 7
Emit 80 vào giây thứ 10
Emit 100 vào giây thứ 13
Emit 1 vào giây thứ 16
```

Ngoài cách trên ra, RxDart lúc nào cũng đi kèm 1 hàm static để bạn thay đổi khẩu vị: `Rx.merge()`. Output chả khác gì cách dùng class `MergeStream` cả. Bạn có thể kiểm chứng bằng cách replace dòng code `var mergeStream = MergeStream([firstStream(), secondStream()]);` bằng dòng code rồi run lại chương trình:

```dart
var mergeStream = Rx.merge([firstStream(), secondStream()]);
```

## 3.2. ZipStream

Lại thêm 1 cái ảnh kèm đường link rxmarbles là đủ hiểu: [RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#zip) ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

![](https://images.viblo.asia/53c3f78b-e8ea-48dd-804f-d49780d30cbb.jpg)

Nhìn vào ảnh có thể hiểu có 2 Stream là `Stream 1` và `Stream 2` chạy song song, 2 Stream này cũng được merge lại thành 1 `ZipStream`.

- Phần tử thứ 1 của Stream 1 sẽ kết hợp với phần tử thứ 1 bên Stream 2
- Phần tử thứ 2 của Stream 1 sẽ kết hợp với phần tử thứ 2 bên Stream 2
- Phần tử thứ 3 của Stream 1 sẽ kết hợp với phần tử thứ 3 bên Stream 2
- Phần tử thứ 4 của Stream 1 sẽ kết hợp với phần tử thứ 4 bên Stream 2
- Phần tử thứ 5 của Stream 1 không có ai để kết hợp vì Stream 2 đã hết event.

Mỗi khi có 1 sự kết hợp thành công thì ZipStream sẽ emit event tổng hợp từ 2 event của Stream 1 và Stream 2. Hàm tổng hợp sẽ do chúng ta truyền vào.

```dart
// truyền vào 1 mảng Stream và 1 hàm kết hợp (zipper)
ZipStream(
    [
      Stream.fromIterable(['1', '2', '3', '4', '5']),
      Stream.fromIterable(['A', 'B', 'C', 'D']),
    ], (values) => values.join(), // hàm kết hợp zipper
  ).listen(print); // prints 1A, 2B, 3C, 4D
```

Ngoài ra, nếu bạn biết rõ số lượng stream cần zip lại thì bạn có thể sử dụng các constructor khác của `ZipStream` như `ZipStream.zip2`, `ZipStream.zip3`, ..., `ZipStream.zip9`. Ví dụ ở đây mình zip 3 stream:

```dart
ZipStream.zip3(
    Stream.fromIterable([1, 2]),
    Stream.fromIterable([3, 4, 5]),
    Stream.fromIterable([6, 7, 8]),
        (a, b, c) => a + b + c, // tính tổng 3 số lại
  ).listen(print);
```

Output:

```none
10 // vì 1 + 3 + 6
13 // vì 2 + 4 + 7
```

Mình khuyến khích sử dụng cách này thay cho cách trên. Vì dùng cách này thì lambda `(a, b, c) => a + b + c` cho ta cả 3 biến độc lập `a, b, c` và mình thoải mái sử dụng thích làm gì cũng được. Còn cách trên thì lambda chỉ có 1 biến kiểu List là `values`: `(values) => values.join()`, sẽ không được thoải mái lắm.

Cũng giống như merge, RxDart cũng cung cấp hàm static `Rx.zip` để bạn đổi khẩu vị:

```none
Rx.zip(
    [
      Stream.fromIterable(['1', '2', '3', '4', '5']),
      Stream.fromIterable(['A', 'B', 'C', 'D']),
    ], (values) => values.join(), // hàm kết hợp zipper
  ).listen(print); // prints 1A, 2B, 3C, 4D

  // hoặc dùng hàm zip2, zip3, ..., zip9
  Rx.zip2(Stream.fromIterable(['1', '2', '3', '4', '5']),
      Stream.fromIterable(['A', 'B', 'C', 'D']), 
          (a, b) => a + b) // hàm kết hợp zipper
      .listen(print); // prints 1A, 2B, 3C, 4D
```

## 3.3. TimerStream

Truyền vào 1 value, và 1 duration. Sau 1 khoảng thời gian duration đó Stream mới emit value được truyền vào.

```dart
TimerStream('hi', Duration(seconds: 5))
    .listen((i) => print(i)); // print 'hi' sau 5 giây
```

Tương tự các hàm trên ta cũng có hàm static: `Rx.timer`

## 3.4. RangeStream

Truyền vào 2 biến: `startInclusive` và `endInclusive` để tạo thành 1 range. `RangeStream` sẽ emit một chuỗi các số nguyên liên tiếp trong range đó. Ví dụ sau đây mình sẽ emit một chuỗi số nguyên từ 1 đến 6.

```dart
RangeStream(1, 6).listen((i) => print(i)); // Print: 1, 2, 3, 4, 5, 6
```

Tương tự các hàm trên ta cũng có hàm static: `Rx.range`

## 3.5. RetryStream

Truyền vào 1 stream. Khi stream được truyền vào gặp lỗi `RetryStream` sẽ cho retry lại nó. Chúng ta có thể truyền thêm 1 biến `count` chính là số lần được phép retry. Nếu không truyền biến `count` thì nó sẽ retry mãi mãi.

```dart
void main() {
  RetryStream(() { return errorStream(); }, 2) // truyền vào số 2 có nghĩa là được phép retry 2 lần
      .listen(print, onError: print);
}

Stream<int> errorStream() async* {
  yield 10;
  throw FormatException('abc'); // lỗi ở đây
  yield 11; // 11 mãi ko được in ra
}
```

Output:

```none
10 // lần 1 in ra gặp lỗi nên sẽ retry lần 1
10 // retry lại 1 lần vẫn gặp lỗi nên sẽ retry lần 2
10 // retry lại 2 lần vẫn gặp lỗi nên cho ném lỗi luôn
Received an error after attempting 2 retries // bắn lỗi ra sau 2 lần retry thất bại
```

Tương tự các hàm trên ta cũng có hàm static: `Rx.retry`

Như vậy là mình đã giới thiệu qua các class lợi hại của RxDart. Bạn có thể học thêm những class khác [tại đây](https://pub.dev/packages/rxdart#stream-classes).

# 4. Những extension function của RxDart

Đối với extention function cũng vậy, mình sẽ giới thiệu một số function đại diện và hay dùng để từ đó các bạn sẽ có nền để chinh phục được tất cả function. Và mình sẽ cố gắng chia sẻ những function không được chia sẻ ở phần 3 (ví dụ như function [zipWith](https://pub.dev/documentation/rxdart/latest/rx/ZipWithExtension/zipWith.html) có cùng công dụng với class `ZipStream` ở trên).

## 4.1. debounceTime

Link rxmarbles: [RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#debounceTime)

![](https://images.viblo.asia/9d605627-8640-43cf-9e4a-f2e728a91952.jpg)

Hàm này cho phép chúng ta truyền vào 1 Duration, cụ thể trong hình là 10s. Nếu Source Stream emit ra 1 event mà sau 10s nó không emit tiếp 1 event khác thì Output Stream mới emit event đó ra. Hàm này cực kỳ hữu ích để làm tính năng live search cho app (real time search) bởi vì app sẽ tránh spam server do call API quá nhiều. Cụ thể ví dụ sau mình sẽ mô phỏng lại quá trình live search của user.

```dart
void main() {
   // 500ms sau mà user không gõ nữa thì sẽ call API search
  demoStream().debounceTime(Duration(milliseconds: 500)).listen((event) { 
    // call API search với keyword
    print('call API search với keyword: $event');
  });
}


// stream mô phỏng lúc user đang gõ text vào TextField để search
Stream<String> demoStream() async* {
  yield 'L'; // gỡ chữ L
  yield 'Le'; // gõ thêm chữ e
  yield 'Lea'; // gõ thêm chữ a
  yield 'Lear'; // gõ thêm chữ r
  yield 'Learn'; // gõ thêm chữ n
  await Future.delayed(Duration(seconds: 1)); // dừng gõ 1 giây
  yield 'Learn R'; // tiếp tục gõ dấu space và chữ R
  yield 'Learn Rx'; // gõ tiếp chữ x
  yield 'Learn RxD'; // gõ tiếp chữ D
  yield 'Learn RxDa'; // gõ tiếp chữ a
  yield 'Learn RxDar'; // gõ tiếp chữ r
  yield 'Learn RxDart'; // gõ tiếp chữ t
}
```

Output:

```none
call API search với keyword: Learn
call API search với keyword: Learn RxDart
```

## 4.2. onErrorResumeNext

Hàm này truyền vào 1 Stream gọi là `recoveryStream` đi ha. Khi Source Stream gặp lỗi thay vì nó emit cái `event lỗi` đó thì nó lại vào emit các phần tử trong `recoveryStream`. Các bạn để ý code dưới đây, `onError` không print ra gì cả.

```dart
void main() {
  errorStream().onErrorResumeNext(Stream.fromIterable([11, 22, 33])).listen(print, onError: print);
}

Stream<int> errorStream() async* {
  yield 1;
  yield 2;
  throw FormatException('lỗi rồi');
  yield 3;
}
```

Output.

```none
1
2
11
22
33
```

## 4.3. interval

Hàm này truyền vào 1 duration. Cứ sau khoảng thời gian duration đó thì nó mới emit ra 1 event.

```dart
Stream.fromIterable([1, 2, 3])
  .interval(Duration(seconds: 1)) // cứ 1 giây emit ra 1 event
  .listen((i) => print('$i giây'); // prints 1 giây, 2 giây, 3 giây
```

## 4.4. concatWith

Link rxmarbles: [RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#concat)

![](https://images.viblo.asia/45069a64-88b6-4b19-8f9e-08ab70381f0d.JPG)

Merge 2 stream lại thành 1 ConcatStream và emit tất cả event trong lần lượt từng stream một. Sau khi emit xong tất cả event của Stream 1, thì ConcatStream vẫn chờ `x giây` (trong hình) mới emit phần tử đầu tiên của Stream 2 chứ không emit ngay lập tức. Chính vì vậy tổng thời gian của `t1` (thời gian stream 1 dừng) + `t2` (thời gian stream 2 dừng) = `t3` (thời gian ConcatStream dừng).

```dart
void main() {
  // biến này để làm cột mốc thời gian lúc bắt đầu run chương trình
  var currentTime = DateTime.now();

  var mergeStream = firstStream().concatWith([secondStream()]); // truyền vào 1 List Stream
  mergeStream.listen((event) => println(event, currentTime));
}

// stream 1
Stream<int> firstStream() async* {
  await Future.delayed(Duration(seconds: 1));
  yield 20; // emit 20 vào giây thứ 1
  await Future.delayed(Duration(seconds: 1));
  yield 40; // emit 40 vào giây thứ 2
  await Future.delayed(Duration(seconds: 2));
  yield 60; // emit 60 vào giây thứ 4
  await Future.delayed(Duration(seconds: 6));
  yield 80; // emit 80 vào giây thứ 10
  await Future.delayed(Duration(seconds: 3));
  yield 100; // emit 100 vào giây thứ 13
}

// stream 2
Stream<int> secondStream() async* {
  await Future.delayed(Duration(seconds: 7)); // emit 100 vào giây thứ 20
  yield 1;
  await Future.delayed(Duration(seconds: 9)); // emit 100 vào giây thứ 29
  yield 1;
}

// mình sử dụng hàm println này để in kèm thời điểm emit event cho dễ quan sát output
void println(Object value, DateTime currentTime) {
  print('Emit $value vào giây thứ ${DateTime.now().difference(currentTime).inSeconds}');
}
```

Output. Mất 13s để emit hết event trong stream 1 và mất tiếp 16s để emit hết event trong stream 2. Vậy mất tất cả 29s

```none
Emit 20 vào giây thứ 1
Emit 40 vào giây thứ 2
Emit 60 vào giây thứ 4
Emit 80 vào giây thứ 10
Emit 100 vào giây thứ 13
Emit 1 vào giây thứ 20
Emit 1 vào giây thứ 29
```

## 4.5. distinctUnique

Link rxmarbles: [RxMarbles: Interactive diagrams of Rx Observables](https://rxmarbles.com/#distinct)

![](https://images.viblo.asia/e25fe46f-dfde-4bd6-aa9e-f105da402539.JPG)

Mình từng giới thiệu về [hàm distinct có sẵn trong lớp Stream của Dart](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W#_5-distinct-13) rồi. Còn hàm `distinctUnique` là thuộc thư viện RxDart. Hai hàm này khác nhau nhé. Khác nhau ở chỗ:

Hàm `distinct` của Dart dùng để skip (bỏ qua) các phần tử mà chúng equal (bằng) với phần tử đã được emit trước đó (the previous element). Vâng nó chỉ so sánh với đúng 1 phần tử đứng liền trước đó thôi. Nếu bằng nhau thì nó skip

Còn hàm `distinctUnique` của RxDart dùng để emit các phần tử không trùng nhau trong Source Stream.

Ok, giờ mình sẽ dựng code giống hệt cái ảnh trên dùng cả 2 hàm là `distinct` và `distinctUnique`

```dart
void main() {
  // dùng distinct
  Stream.fromIterable([1, 2, 2, 1, 3]).distinct()
      .listen(print); // print: 1, 2, 1, 3

  // dùng distinctUnique
  Stream.fromIterable([1, 2, 2, 1, 3]).distinctUnique()
      .listen(print); // print: 1, 2, 3
}
```

Như vậy là mình đã giới thiệu qua các extension function lợi hại của RxDart. Bạn có thể học thêm những extension function khác [tại đây](https://pub.dev/packages/rxdart#extension-methods)

# 5. Subjects

Như mình đã giới thiệu ở trên, Subjects bản chất là [StreamController](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W#_3-stream-controlller-15). Trong RxDart có 3 loại Subjects là: `BehaviorSubject`, `ReplaySubject` và `PublishSubject`

## 5.1. BehaviorSubject

Đây cũng là một [Broadcast StreamController](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W#_4-broadcast-streamcontroller-16). Đặc điểm của thằng này là mỗi khi có một listener mới subscribe thì nó lập tức emit phần tử mới nhất vừa được add vào controller (the latest item that has been added to the controller)

![](https://images.viblo.asia/acf3e451-4eb9-41dd-9e39-deb6e12685f2.png)

Xem hình khó hiểu quá thì xem code, xem code khó hiểu quá thì lại lên xem hình sẽ hiểu ngay =))

```dart
// tạo BehaviorSubject
var streamController = BehaviorSubject();

void main() {
  // subscription thứ nhất
  streamController.stream.listen((event) {
    println('thứ 1: $event');
  });

  // 2 giây sau sẽ tạo subscription thứ hai
  createSecondSubscription();

  // cứ mỗi 600 ms sẽ push 1 data từ 0 cho đến 4
  streamController.sink.addStream(RangeStream(0, 4).interval(Duration(milliseconds: 600)));
}

void createSecondSubscription() async {
  Future.delayed(Duration(seconds: 2), () {
    println('tạo ra subscription thứ 2');
    streamController.stream.listen((event) {
      println('subscription thứ 2: ${event + event}'); // double value lên
    });
  });
}

// mình sử dụng hàm println này để in ra thời gian hiện tại cho dễ quan sát output
void println(Object value) {
  print('${DateTime.now()}: $value');
}
```

Output: Quan sát output có thể thấy ngay thời điểm 8 phút 51 giây thì subscription thứ 2 mới được tạo ra mà nó vừa tạo ra là nó chụp luôn cái data `số 2` để double lên thành `số 4`. Mặc dù data `số 2` được sinh ra trước khi `subscription thứ 2` chào đời gần 1 giây. Đúng là vừa sinh ra đã đi ăn cướp ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

```none
2020-08-30 15:08:49.728767: thứ 1: 0
2020-08-30 15:08:50.330743: thứ 1: 1
2020-08-30 15:08:50.932222: thứ 1: 2
2020-08-30 15:08:51.117602: tạo ra subscription thứ 2
2020-08-30 15:08:51.118510: subscription thứ 2: 4
2020-08-30 15:08:51.533267: thứ 1: 3
2020-08-30 15:08:51.533267: subscription thứ 2: 6
2020-08-30 15:08:52.133810: thứ 1: 4
2020-08-30 15:08:52.133810: subscription thứ 2: 8
```

Ngoài ra, BehaviorSubject còn có 1 constuctor khác là `BehaviorSubject.seeded`. Khi sử dụng constructor này thì mặc định mới vào controller sẽ được add 1 phần tử được truyền vào hàm `seeded`.

```dart
// tạo BehaviorSubject sử dụng seeded
var streamController = BehaviorSubject.seeded(333);

void main() {
  // subscription
  streamController.stream.listen(print);

  // cứ mỗi 600 ms sẽ push 1 data từ 0 cho đến 2
  streamController.sink.addStream(RangeStream(0, 2).interval(Duration(milliseconds: 600)));
}
```

Output:

```none
333
0
1
2
```

## 5.2. ReplaySubject

Đây cũng là một Broadcast StreamController. Thằng này còn bá đạo hơn thằng BehaviorSubject là vừa chào đời không những nó chụp 1 data mới nhất mà nó chụp tất cả data đã được add vào controller luôn.

![](https://images.viblo.asia/be65d365-78c9-4803-96d1-0df2a7af1241.png)

```dart
// tạo ReplaySubject
var streamController = ReplaySubject();

void main() {
  // subscription thứ nhất
  streamController.stream.listen((event) {
    println('thứ 1: $event');
  });

  // 2 giây sau sẽ tạo subscription thứ hai
  createSecondSubscription();

  // cứ mỗi 600 ms sẽ push 1 data từ 0 cho đến 4
  streamController.sink.addStream(RangeStream(0, 4).interval(Duration(milliseconds: 600)));
}

void createSecondSubscription() async {
  Future.delayed(Duration(seconds: 2), () {
    println('tạo ra subscription thứ 2');
    streamController.stream.listen((event) {
      println('subscription thứ 2: ${event + event}'); // double value lên
    });
  });
}

// mình sử dụng hàm println này để in ra thời gian hiện tại cho dễ quan sát output
void println(Object value) {
  print('${DateTime.now()}: $value');
}
```

Output. Ngay thời điểm `tạo ra subscription thứ 2` nó chụp luôn tất cả data từ 0 đến 2 để double lên `0, 2, 4`.

```none
2020-08-30 15:18:57.738695: thứ 1: 0
2020-08-30 15:18:58.338941: thứ 1: 1
2020-08-30 15:18:58.940654: thứ 1: 2
2020-08-30 15:18:59.124769: tạo ra subscription thứ 2
2020-08-30 15:18:59.126769: subscription thứ 2: 0
2020-08-30 15:18:59.126769: subscription thứ 2: 2
2020-08-30 15:18:59.126769: subscription thứ 2: 4
2020-08-30 15:18:59.542143: thứ 1: 3
2020-08-30 15:18:59.542143: subscription thứ 2: 6
2020-08-30 15:19:00.143433: thứ 1: 4
2020-08-30 15:19:00.143433: subscription thứ 2: 8
```

## 5.3. PublishSubject

![](https://images.viblo.asia/79af63f5-e3ed-45f9-adc9-09d814c8bde8.png)

Thằng này chính xác là nó rất giống một [Broadcast StreamController](https://viblo.asia/p/flutter-tu-rxdart-den-bloc-pattern-bai-2-tiep-tuc-voi-so-tream-oOVlYMX4l8W#_4-broadcast-streamcontroller-16) thông thường. Có nghĩa là lúc subscription thứ 2 được tạo ra nó sẽ không chôm chĩa bất cứ data nào trước đó của controller mà nó chờ có data mới được add vào controller nó mới nhận.

```dart
// tạo PublishSubject
var streamController = PublishSubject();

void main() {
  // subscription thứ nhất
  streamController.stream.listen((event) {
    println('thứ 1: $event');
  });

  // 2 giây sau sẽ tạo subscription thứ hai
  createSecondSubscription();

  // cứ mỗi 600 ms sẽ push 1 data từ 0 cho đến 4
  streamController.sink.addStream(RangeStream(0, 4).interval(Duration(milliseconds: 600)));
}

void createSecondSubscription() async {
  Future.delayed(Duration(seconds: 2), () {
    println('tạo ra subscription thứ 2');
    streamController.stream.listen((event) {
      println('subscription thứ 2: ${event + event}'); // double value lên
    });
  });
}

// mình sử dụng hàm println này để in ra thời gian hiện tại cho dễ quan sát output
void println(Object value) {
  print('${DateTime.now()}: $value');
}
```

Thay vì sử dụng `PublishSubject`: `var streamController = PublishSubject();` thì các bạn sử dụng Broadcast StreamController thông thường thế này: `var streamController = StreamController.broadcast();` nó đều sẽ cho ra cùng 1 output:

```none
2020-08-30 15:27:19.845036: thứ 1: 0
2020-08-30 15:27:20.446674: thứ 1: 1
2020-08-30 15:27:21.047935: thứ 1: 2
2020-08-30 15:27:21.230446: tạo ra subscription thứ 2
2020-08-30 15:27:21.649195: thứ 1: 3
2020-08-30 15:27:21.649195: subscription thứ 2: 6
2020-08-30 15:27:22.249643: thứ 1: 4
2020-08-30 15:27:22.249643: subscription thứ 2: 8
```

# 6. CompositeSubscription

`CompositeSubscription` giống như một cái vùng chứa tất cả StreamSubscription, để sau này chúng ta tiện cancel tất cả StreamSubscription cùng một lúc.

```dart
void main() async {
  // giả sử chúng ta có 3 subscription
  var subscription1 = testStream().listen(println);
  var subscription2 = testStream().listen(println);
  var subscription3 = testStream().listen(println);

  // tạo ra 1 compositeSubscription
  var compositeSubscription = CompositeSubscription();

  // add 3 subscription đó vào compositeSubscription
  compositeSubscription.add(subscription1);
  compositeSubscription.add(subscription2);
  compositeSubscription.add(subscription3);

  // 2 giây sau cancel tất cả 3 subscription đó
  await Future.delayed(Duration(seconds: 2));
  compositeSubscription.clear();

  // cũng có thể dùng hàm dispose để cancel tất cả 3 subscription đó
  compositeSubscription.dispose();
}

Stream<int> testStream() async* {
  yield 10;
  await Future.delayed(Duration(seconds: 2));
  yield 11;
  await Future.delayed(Duration(seconds: 3));
  throw FormatException('lỗi');
  yield 13; // hàm này đã xảy ra Exception nên số 13 không được phát ra
}
```

Output:

```none
2020-08-29 22:33:19.276475: 10
2020-08-29 22:33:19.278475: 10
2020-08-29 22:33:19.278475: 10
```

Hàm `clear()` giống hàm `dispose()` đều là để cancel tất cả subscription. Nhưng khác ở chỗ là khi sử dụng hàm `clear` thì chúng ta có thể tái sử dụng lại `compositeSubscription`, còn một khi đã sử dụng hàm `dispose` thì không thể tái sử dụng `compositeSubscription` nữa.

# Kết luận

Đúng như lúc đầu mình nói là RxDart tuy đồ sộ nhưng không hề đáng sợ đúng không nào, mình đâu có chém ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png). Nó thật sự chỉ đáng sợ với những bạn chưa hiểu về `Stream` mà thôi ![😄](https://twemoji.maxcdn.com/2/72x72/1f604.png)

Tham khảo: [rxdart - Dart API docs](https://pub.dev/documentation/rxdart/latest/)

[Flutter &#8211; RxDart &#8211; Hoàng Cửu Long](https://hoangcuulong.com/flutter/flutter-rxdart/)

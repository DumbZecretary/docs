# How does Flutter work?

[part 1](https://medium.com/vinid/flutter-ho%E1%BA%A1t-%C4%91%E1%BB%99ng-nh%C6%B0-n%C3%A0o-ph%E1%BA%A7n-1-ac13e289ed17)

[part 2](https://medium.com/vinid/flutter-ho%E1%BA%A1t-%C4%91%E1%BB%99ng-nh%C6%B0-th%E1%BA%BF-n%C3%A0o-ph%E1%BA%A7n-2-b2e46a257e0e)

Phần lớn chúng ta khi phát triển ứng dụng bằng Flutter sẽ chỉ sử dụng đến Widget nên rất ít khi quan tâm đến những khái niệm như Element, BuildContext, RenderObject và Binding. Đã bao giờ bạn thắc mắc Flutter hoạt động như thế nào chưa?

Nếu bạn tò mò thì đây chính là bài viết dành cho bạn.

## Thiết bị hiển thị hoạt động như nào

Khi chúng ta quan sát bề mặt hiển thị của một thiết bị (smartphone, TV,…) thì thưc chất chúng ta đang quan sát một tập các pixel được tập hợp với nhau nhằm tạo ra một ảnh phẳng 2 chiều.

Việc dựng ảnh để hiển thị trên màn hình được chịu trách bởi thiết bị hiển thị (TBHT). Các TBHT thường có một thông số gọi là tần số làm tươi (refresh rate) thể hiện số khung hình được làm tươi trên màn hình (thường là 60 khung hình/s).

TBHT nhận data để hiển thị từ GPU (bộ xử lý đồ họa). Số lần mà GPU có thể generate ra một hình ảnh (frame buffer) để gửi cho TBHT được gọi là frame rate, nó được đo lường bằng đơn vị fps (số khung hình/giây).

Lý do mình đề cập đến cách ảnh 2D được render là bởi mục tiêu chính của Flutter là giúp chúng ta dựng giao diện bằng các ảnh phẳng 2D có thể tương tác được (chạm, vuốt). Không chỉ có vậy, Flutter cũng rất quan tâm đến việc TBHT sẽ có thể refresh nhanh và **đúng thời điểm** để đảm bảo app chạy mượt mà không giật lag.

## Interface giữa code và thiết bị vật lý

Phía dưới là hình ảnh kiến trúc high-level của Flutter

![](https://miro.medium.com/proxy/1*vKZMWmCo_VjVaXfdDvCEig.png)

Flutter high level architecture

Khi chúng ta dùng Dart để viết một app Flutter thì tức là chúng ta đang dùng đến tầng Framework của kiến trúc (màu xanh lá).

Tầng Framework sẽ giao tiếp với tầng Engine (xanh dương) thông qua một lớp abstract gọi là [Window](https://api.flutter.dev/flutter/dart-ui/Window-class.html). Lớp abstract này sẽ cung cấp một tập các API để giao tiếp, một cách không trực tiếp, với thiết bị.

Cũng thông qua lớp abstract này mà tầng Engine sẽ có hành động notify tầng Framework khi có các sự kiện sau:

- Cấu hình thiết bị thi thay đổi (orientation, setting, trạng thái chạy của ứng dụng…)
- Có tương tác của user với màn hình (gesture).
- Platform channel truyền dữ liệu lên tầng Framework.
- Và phần lớn là khi tầng Engine sẵn sàng để render frame mới.

## Tầng Framework thực chất được điều khiển bởi tầng Engine

Thật khó tin đúng không? Nhưng đó là sự thật.

Ngoại trừ một số ngoại lệ (liệt kê bên dưới) thì không có trường hợp nào mà code ở tầng Framework được execute mà không phải là do được trigger bởi hành động render frame ở tầng Engine.

Các ngoại lệ đó là:

- Gesture: cử chỉ tay của user trên mặt kính của thiết bị.
- Platform message: Message được emit từ platform của thiết bị, ví dụ như GPS
- Device message: Các message liên quan đến các state của thiết bị, như orientation, app bị đưa vào background, setting của thiết bị…
- Các response của http hay `Future`.

> Tầng Framework sẽ ko apply sự thay đổi UI nào nếu đó không phải là request đến từ việc render frame của Engine. Thực tế vẫn có cách tạo sự thay đổi trên UI mà ko cần phải là điều được request từ Engine, nhưng cách đó **không được khuyến nghị**.

Một số bạn sẽ thắc mắc là giả sử nếu sự thay đổi UI đến từ gesture hoặc từ kết quả của một timer task nào đó thì rốt cuộc điều gì đã diễn ra?

Ảnh dưới mô tả cách mọi thứ diễn ra under the hood như nào:

![](https://miro.medium.com/max/1428/1*2KB3VJUyHcz3jmLWOr3Qug.gif)

Giải thích một cách ngắn gọn thì:

- Một số external event (gesture, http response, `Future` response) có thể launch một số task dẫn đến việc cần cập nhật việc render. Một message sẽ được gửi đến Engine với mục đích thông báo việc này (*Schedule Frame*).
- Khi Engine đã sẵn sàng cho việc update rendering thì nó sẽ emit một request là *Begin Frame*.
- Request Begin Frame sẽ bị intercept bởi Framework để chạy một số task chủ yếu liên quan đến `Tickers` như `Animations`.
- Các task được chạy có thể sẽ lại emit request render frame khác. Ví dụ như khi có một animation đang chạy thì tầng Framework sẽ phải gửi một message schedule frame cho Engine để nó render frame tiếp theo của animation.
- Sau đấy thì Flutter Engine sẽ emit message `Draw Frame`.
- Message này lại bị intercept bởi tầng `Framework` nhằm chạy những task liên quan đến việc cập nhật lại cấu trúc và size của layout.
- Sau đấy thì các task liên quan đến việc cập nhật paint của layout sẽ được thực hiện.
- Nếu có thứ cần được vẽ lên screen thì tầng `Framework` sẽ gửi một `Scene` mới cần được render cho `Engine` để nó thực hiện việc cập nhật lại screen.
- Sau đấy thì tầng `Framework` sẽ thực hiện các task cần được chạy sau sự kiện render (thông qua `PostFrame` callback) và các task sau đó mà không liên quan đến việc render.
- Và mọi việc cứ thế lặp lại từ đầu.

## `RenderView` và `RenderObject`

Khi làm UI thì từ những dòng code mà chúng ta viết sẽ cho output là một tập các pixel được compose. Hay nói cách khác là từ các `Widget` mà chúng ta sử dụng sẽ cho output là các “visual part” (một phần hiển thị trên màn hình tương ưng với `Widget` đó).

Các visual part được render trên screen này tương ứng với các object được gọi là `RenderObject`, được sử dụng để:

- Định nghĩa một khu vực trên screen để chứa nội dung được render. Để xác định khu vực thì nó sẽ sử dụng đến các thông tin như kích thước, vị trí, hình dạng hình học.
- Xác định các khu vực trên screen có thể bắt các sự kiện gesture.

Một set các `RenderObject` sẽ hình thành một tree, được gọi là `RenderTree`. Ở gốc của tree ấy sẽ luôn là một `RenderView`.

`RenderView` đại diện cho toàn bộ bề mặt của screen mà `RenderTree` sẽ dùng để hiển thị UI. Có thể coi `RenderView` là một phiên bản đặc biệt của `RenderObject`.

Để dễ hình dung các bạn có thể xem ảnh dưới

![](https://miro.medium.com/proxy/1*BBGJ4wG507T9w5T3DzTrqA.png)

Về mối quan hệ giữa `RenderObject` và `Widget` sẽ được đề cập sau.

## Binding

Khi một app Flutter được start thì hệ thống sẽ invoke hàm `main()`, kéo theo việc hàm `runApp(Widget app)` được gọi theo.

Trong quá trình gọi đến hàm `runApp()` thì tầng Framework sẽ khởi tạo các interface giữa tầng `Framework` và tầng `Engine`. Các interface này được gọi là **binding**.

## Giới thiệu về binding

Các **binding** được sinh ra như là một chất keo kết dính giữa tầng Engine và Framework. Hai tầng này chỉ có thể giao tiếp với nhau thông qua các **binding** này.

Mỗi một binding sẽ chịu trách nhiệm cho một số task, action và event nhất định, được nhóm lại dựa trên một tiêu chí cụ thể.

Tại thời điểm của bài viết thì đang có 8 loại binding.

Chúng ta sẽ chỉ đi sâu hơn về các loại binding chính sau:

- *SchedulerBinding*
- *GestureBinding*
- *RendererBinding*
- *WidgetBinding*

Các binding còn lại là:

- ServicesBinding: Chịu trách nhiệm xử lý các message được gửi bởi platform channel.
- PaintingBinding: Chịu trách nhiệm cho việc cache ảnh.
- SemanticsBinding: Chịu trách nhiệm cho tất cả các task liên quan đến `Semantics`.
- TestWidgetsFlutterBinding: được sử dụng bởi lib widget test.

Ngoài ra còn có cả `WidgetsFlutterBinding`, tuy nhiên nó giống một *binding initializer* hơn là một binding.

Hình bên dưới mô tả tương tác giữa một số các binding:

![](https://miro.medium.com/proxy/1*OwTxZ-q9loJT_ZQt6-24Gw.png)

Chúng ta hãy cùng tìm hiểu kỹ hơn về các binding chính nhé

**SchedulerBinding**

Binding này có 2 nhiệm vụ chính:

- Nhiệm vụ đầu tiên là để nói với tầng Engine rằng: “Trong lần rảnh tiếp theo của mày thì nhớ đánh thức tao dậy để tao xem mày có phải render cái gì không nhé, hoặc nếu tao cần mày gọi lại sau”.
- Nhiệm vụ thứ hai là lắng nghe và xử lý những lời “đánh thức” ấy.

Vậy khi nào thì *SchedulerBinding* sẽ yêu cầu được “đánh thức”?

- Là khi mà `Ticker` cần được *tick*. Ví dụ như là khi có một Animation và chúng ta start nó. Lúc này animation khi chạy sẽ được điều khiển bởi một `Ticker`, là cái mà cứ sau một khoảng thời gian (tick) thì nó sẽ được gọi để chạy một callback. Để chạy callback này thì chúng ta sẽ phải bảo Flutter Engine đánh thức tại lần refresh màn hình tiếp theo (*BeginFrame*). Việc đánh thức này sẽ invoke callback `ticker` để nó thực hiện task của mình. Sau khi hoàn thành task, nếu *ticker* cần được gọi tiếp thì nó sẽ bảo *SchedulerBinding* schedule frame tiếp theo.
- Khi có sự thay đổi ở layout: Ví dụ như là khi có một event nào đó xảy ra thì chúng ta cần thay đổi UI (thay đổi màu, scroll, thêm hoặc xóa cài gì đấy trên màn hình). Trong trường hợp này thì tầng Framework sẽ gọi SchedulerBinding để nhờ nó schedule frame khác với tầng Engine.

**GestureBinding**

Binding này sẽ lắng nghe sự tương tác của gesture (cử chỉ tay) với tầng Engine. Cụ thể thì nó sẽ chịu trách nhiệm trong việc tiếp nhận data liên quan đến gesture và xác định phần nào của screen sẽ bị tác động bởi gesture đấy. Sau đó nó sẽ gửi thông báo đến các thành phần bị ảnh hưởng.

**RendererBinding**

Binding này kết nối tầng Engine với *Render Tree*. Nó có 2 trách nhiệm sau:

- Đầu tiên là lắng nghe các sự kiện được emit bởi tầng Engine để thông báo về các thay đổi liên quan đến setting của thiết bị mà có thể tác động đến UI hoặc *semantic*.
- Thứ hai là cung cấp những sự thay đổi về hiển thị cần thực hiện cho tầng Engine.

Để cho những sự thay đổi này có thể được render lên screen thì *RendererBinding* sẽ phải chịu trách nhiệm điều khiển [PipelineOwner](https://docs.flutter.io/flutter/rendering/PipelineOwner-class.html) và khởi tạo **RenderView**.

*PipelineOwner* có thể coi như là một “nhạc trưởng” biết RenderObject nào cần làm gì liên quan đến layout và điều phối các hành động đó của các RenderObject.

**WIdgetsBinding**

Binding này cũng lắng nghe sự thay đổi về setting của device được tạo bởi người dùng mà ảnh hưởng tới ngôn ngữ (locale) và *semantics*. Ngoài ra thì nó cũng là thành phần kết nối Widget và *Flutter Engine*. Nó có 2 trách nhiệm sau:

- Đầu tiên là điều khiển tiến trình liên quan đến việc xử lý khi có sự kiện cấu trúc của Widget bị thay đổi.
- Thứ hai là trigger việc render của tầng Engine.

Việc xử lý khi cấu trúc của Widget bị thay đổi được thực hiện thông qua [BuildOwner](https://docs.flutter.io/flutter/widgets/BuildOwner-class.html). *BuildOwner* sẽ theo dõi xem WIdget nào cần rebuild và đảm nhiệm các task có tầm ảnh hưởng đến toàn bộ cấu trúc của Widget.

## Kết

Trong phần 1 này thì mình đã giới thiệu sơ lược cơ chế hoạt động bên trong của Flutter. Trong phần sau chúng ta sẽ tìm hiểu kỹ hơn về Widget, Element, RenderObject và mối quan hệ giữa chúng nhé.

Happy coding!

*Nguồn tham khảo:*

1. https://medium.com/saugo360/flutters-rendering-engine-a-tutorial-part-1-e9eff68b825d
2. [BuildOwner class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/BuildOwner-class.html)
3. [Flutter - Flutter internals](https://www.didierboelens.com/2019/09/flutter-internals/)
4. [Flutter&#39;s Rendering Pipeline - YouTube](https://www.youtube.com/watch?v=UUfXWzp0-DU)
5. [PipelineOwner class - rendering library - Dart API](https://api.flutter.dev/flutter/rendering/PipelineOwner-class.html)

Trên trang homepage của Flutter có nhấn mạnh rằng *mọi thứ đều là Widget*.

Nói thế này chưa chuẩn lắm, theo mình sẽ phải là:

> *Dưới góc nhìn của lập trình viên thì mọi thứ liên quan đến UI ở khía cạnh layout và tương tác người dùng đều được thực hiện thông qua Widget.*

Widget cho phép chúng ta định nghĩa một phần trên MHHT (màn hình hiển thị) về các khía cạnh như kích thước, nội dung, cách bố cục và tương tác. **Tuy nhiên đó chưa phải là tất cả**. Vậy thực sự Widget là gì?

# Cấu hình bất biến (Immutable configuration)

Khi lục lọi source code của Flutter, các bạn sẽ thấy Widget được định nghĩa như sau:

```dart
@immutable
abstract class Widget extends DiagnosticableTree {  
  const Widget({ this.key });final Key key;...  
}
```

Vậy là sao?

Annotation ***@immutable*** ở đây đóng vai trò rất quan trọng: **Nó nói cho chúng ta biết là tất cả các variable trong class Widget đều phải là final**. Nói cách khác thì tất cả các variable đều chỉ được định nghĩa và gán giá trị cho nó **một lần duy nhất**. Như vậy là một khi đã được khởi tạo thì Widget sẽ không cho phép chúng ta thay đổi các thuộc tính của nó nữa.

> *Có thể coi Widget như một dạng cấu hình do nó cho phép chúng ta định nghĩa cách UI, UX được thể hiện thông qua việc thiết lập cấu hình cho nó. Tuy nhiên do việc thiết lập chỉ được phép thực hiện 1 lần duy nhất nên chúng ta sẽ gọi nó là cấu hình bất biến.*

# Cấu trúc phân cấp của Widget

Khi định nghĩa cấu trúc cho một screen bằng Widget thì chúng ta sẽ có đoạn code kiểu như sau:

```dart
Widget build(BuildContext context){  
    return SafeArea(  
        child: Scaffold(  
            appBar: AppBar(  
                title: Text('My title'),  
            ),  
            body: Container(  
                child: Center(  
                    child: Text('Centered Text'),  
                ),  
            ),  
        ),  
    );  
}
```

Ví dụ ở trên sử dụng 7 widget, cùng nhau chúng tạo nên một cấu trúc phân cấp. Phía dưới là thể hiện dưới dạng diagram dễ nhìn hơn:

![](https://miro.medium.com/max/642/0*vy1VBm1lsp7B3wsB)

Như các bạn thấy đấy, nó là một cấu trúc dạng cây với root là `SafeArea`.

## Giấu rừng trong cây 😕

Tiêu đề không sai đâu các bạn. Bởi nếu các bạn chưa biết thì tự bản thân một widget nó đã có thể là tổng hợp của rất nhiều widget khác (cũng dưới dạng tree). Hay nói dễ hiểu hơn thì một widget có thể là đại diện cho một widget tree khác. Ví dụ chúng ta có thể viết lại đoạn code trên như sau:

```dart
Widget build(BuildContext context){  
    return MyOwnWidget();  
}
```

Ở đây thì `MyOwnWidget` đại diện cho cả cái widget tree bao gồm `SafeArea`, `Scaffold`...

> *Một widget có thể đại diện cho một widget tree. Nếu được dùng như một node của widget tree nào đó khác thì cũng có thể coi nó như đại diện cho một subtree.*

## Khái niệm element trong tree

Cái này rất quan trọng.

Thực tế thì khái niệm widget tree được sinh ra để đơn giản hoá việc đọc hiểu cho dev. Ở trên mình có nói widget là một dạng thông tin cấu hình.

Vậy thông tin cấu hình đó được dùng để làm gì?

Nó được dùng để sinh ra một kiểu đối tượng khác tương ứng có tên gọi là [Element](https://docs.flutter.io/flutter/widgets/Element-class.html).

> *Với mỗi một widget sẽ có tương ứng cho nó một element. Cũng như widget thì các element sẽ được liên kết với nhau để tạo thành dạng cây.*

Để dễ hình dung thì giả sử chúng ta có 1 element tên X như hình bên dưới. X là một node không phải leaf cũng không phải root trong 1 element tree nên nó sẽ có cả node cha và node con. **Ngoài ra thì nó có thể tham chiếu đến một widget và cả một** `**RenderObject**`.

![](https://miro.medium.com/max/1376/0*pDl_T1_KjOnmex0q)

Bên dưới là hình minh họa ánh xạ giữa 3 loại tree:

![](https://miro.medium.com/max/1400/0*HO_FOU6CloljUqAX)

Mình vẫn sẽ chưa nói đến việc tại sao lại sinh ra cái gọi là `Element`.

# Phân loại Widget

Mình xin phân loại nó ra làm 3 loại theo tài liệu không chính thống nhé:

- Proxy: Vai trò của loại widget này là để nắm giữ và cung cấp data mà widget khác (là node của subtree mà proxy làm root) cần. Một ví dụ điển hình là `InheritedWidget` hoặc `LayoutId`. Cũng chính vì chỉ dùng để cung cấp data cho widget khác mà loại widget này không gây tác động trực tiếp đến UI.
- Renderer: Loại widget này liên quan trực tiếp đến cách bố cục giao diện trên màn. Ví dụ tiêu biểu như: `Row`, `Column`, `Stack`, ngoài ra có cả `Padding`, `Align`, `Opacity`...
- Component: Đây là những Widget không trực tiếp cung cấp thông tin cuối cùng liên quan đến kích thước, vị trí, hình thức. Chúng chỉ cung cấp thông tin phụ trợ góp phần đưa ra thông tin cuối cùng. Ví dụ tiêu biểu cho loại này là : `RaisedButton`, `Scaffold`, `Text`, `GestureDetector`.

![](https://miro.medium.com/max/1400/0*DxCkHUxb2MCJ8h6T)

Các bạn có thể tham khảo bảng tổng hợp phân loại ở [đây](https://www.didierboelens.com/images/blog/internals_widget_categories.pdf).

Việc phân loại Widget như này rất quan trọng do với mỗi loại `Widget` thì sẽ có một loại `Element` tương ứng.

# Các loại Element

Dưới đây là liệt kê các loại element hiện có:

![](https://miro.medium.com/max/1400/0*TKv_c82ODBoOcl4a)

Như các bạn có thể thấy thì Element đang được chia thành 2 loại chính:

- `ComponentElement`: Các element loại này không trực tiếp tương ứng với phần giao diện được render nào.
- `RenderObjectElement`: Các element này tương ứng với một phần giao diện được render trên màn.

Giờ thì chúng ta hãy cùng tìm hiểu mối quan hệ giữa `Widget` và `Element` là thế nào nhé.

# Widget và Element hoạt động với nhau dư nào

> *Trong Flutter, việc trigger render frame mới có thể thực hiện qua việc invalidate (đánh dấu vô hiệu) element hoặc render object.*

Việc invalidate một element có thể thực hiện bằng nhiều cách khác nhau như:

- Sử dụng `setState`. Bằng cách này thì toàn bộ `StatefulElement` (không phải StatefulWidget nha) sẽ bị invalidate.
- Thông qua các notification (thông báo) của các `proxyElement` khác (ví dụ như `InheritedWidget`), `proxyElement` sẽ invalidate bất cứ element nào phụ thuộc vào nó.

Kết quả của việc invalidate element sẽ là một hoặc nhiều element tương ứng sẽ được tham chiếu tới từ một list được gọi là dirty elements.

Việc invalidate một `renderObject` đồng nghĩa với việc không có sự thay đổi cấu trúc nào xảy ra trên element tree. Sự thay đổi này đến từ level của `renderObject`, ví dụ như:

- Thay đổi về mặc kích thước, vị trí, hình dạng hình học…
- Cần thực hiện việc repaint (sơn lại), ví dụ như khi chúng ta đổi màu background, font style…

Kết quả của việc invalidate render object cũng sẽ là các `renderObject` tương ứng sẽ được tham chiếu tới bởi list các `renderObject` cần rebuild hay repaint.

Bất kể loại invalidation nào xảy ra thì đều dẫn đến việc `SchedulerBinding` (bạn còn nhớ?) được yêu cầu để "xin xỏ" `Flutter Engine` schedule một frame mới.

Và thời khắc mà `Flutter Engine` đánh thức `SchedulerBinding` chính là lúc các phép màu xuất hiện...

## onDrawFrame()

Trong [phần 1](https://medium.com/vinid/flutter-ho%E1%BA%A1t-%C4%91%E1%BB%99ng-nh%C6%B0-n%C3%A0o-ph%E1%BA%A7n-1-ac13e289ed17) thì mình có đề cập đến việc `SchedulerBinding` có 2 trách nhiệm chính, một trong số đó là thực hiện các request liên quan đến việc rebuild lại frame đến từ `Flutter Engine`. Bây giờ chúng ta sẽ nói kỹ hơn về vấn đề này nhé...

Lưu đồ trình tự phía dưới sẽ cho các bạn thấy điều gì sẽ xảy ra khi `SchedulerBinding` tiếp nhận một request `onDrawFrame()` từ `Flutter Engine`.

![](https://miro.medium.com/max/1400/0*z9azA1Cf7ll_iVBk)

**Bước 1:**

Do **BuildOwner** chịu trách nhiệm cho việc handle element tree nên `WidgetBinding` sẽ invoke hàm `buildScope` của `BuildOwner`. Hàm này sẽ duyệt qua danh sách các element bị đánh dấu invalidate (=dirty) và sẽ yêu cầu chúng phải rebuild.

Quá trình từ khi gọi `rebuild()` sẽ là:

1. Yêu cầu element (giả sử gọi là X) phải `rebuild()` hầu như đều dẫn đến việc gọi đến hàm `build()` (chính là hàm `Widget build(BuildContext context){...}`) của widget tham chiếu đến bởi X.
2. Nếu X không có child thì widget mới sẽ được inflate (xem hình bên dưới).
3. Trong trường hợp element có child (giả sử child đó là Y ) thì widget mới sẽ được so sánh với widget được tham chiếu tới bởi Y:  
   - Nếu chúng có thể hoán đổi được cho nhau (= cùng kiểu widget và key) thì việc update sẽ được thực hiện, nhưng Y sẽ được giữ nguyên.  
   - Nếu chúng không hoán đổi được cho nhau thì Y sẽ bị unmount (bị bỏ) và widget mới sẽ được inflate.
4. Việc inflate một widget mới dẫn đến việc một element mới cũng sẽ được khởi tạo, và được mount như là một child mới của Y.

Các bạn có thể xem hình dưới để dễ hình dung hơn.

![](https://miro.medium.com/proxy/1*axYhku0Ue0g6mPO_OL8uKA.gif)

**Một số lưu ý về việc inflate widget**

Khi widget được inflate thì nó sẽ được yêu cầu tạo một element với một type nhất định, được định nghĩa từ bảng phân loại widget, cụ thể:

- `InheritedWidget` sẽ sinh ra một `InheritedElement`.
- `StatefulWidget` sẽ sinh ra một `StatefulElement`.
- `StatelessWidget` sẽ sinh ra một `StatelessElement`.
- `InheritedModel` sẽ sinh ra một `InheritedModelElement`.
- `InheritedNotifier` sẽ sinh ra một `InheritedNotifierElement`.
- `LeafRenderObjectWidget` sẽ sinh ra một `LeafRenderObjectElement`.
- `SingleChildRenderObjectWidget` sẽ sinh ra một `SingleChildRenderObjectElement`
- `MultiChildRenderObjectWidget` sẽ sinh ra một `MultiChildRenderObjectElement`.
- `ParentDataWidget` sẽ sinh ra một `ParentDataElement`

Mỗi một loại element này sẽ có behavior khác nhau. Ví dụ:

- Một `StatefulElement` sẽ invoke hàm `widget.createState()` tại thời điểm khởi tạo, dẫn đến việc `State` sẽ được khởi tạo và link đến element.
- Kiểu `RenderObjectElement` sẽ tạo một `RenderObject` khi mà element đó sẽ được mount, `renderObject` sẽ được thêm vào `render tree` và link đến element.

**Bước 2:**

Sau khi tất cả các hành động liên quan đến các dirty element đã được thực hiện thì lúc này công việc liên quan đến element tree coi như đã xong. Quá trình tiếp theo sẽ được thực hiện là quá trình rendering.

`RendererBinding` chịu trách nhiệm việc handle *rendering tree*, *WidgetBinding* sẽ invoke hàm *drawFrame* của *RendererBinding*.

Biểu đồ phía dưới sẽ mô tả chi tiết chuỗi các hành động được thực hiện sau khi hàm *drawFrame* được gọi.

![](https://miro.medium.com/max/1400/0*q8sox_07IZLTD6F8)

Cụ thể, sẽ có các action sau diễn ra:

- Các `renderObject` được đánh dấu là `dirty` sẽ thực hiện lại quá trình layout (tính toán lại kích thước và hình dáng hình học).
- Các `renderObject` được đánh dấu là `need paint` sẽ được tô màu lại bằng cách sử dụng layer của renderObject.
- Kết quả chúng ta sẽ có 1 **scene** mới được build và gửi đến *Flutter Engine* để hiển thị lên màn hình.
- Cuối cùng thì *Semantics (mình sẽ viết bài khác về nó)* cũng sẽ được update và gửi đến *Flutter Engine.*

Sau chuỗi action trên thì màn hình hiển thị sẽ được cập nhật.

# Phần 3: Gesture handling

Các gesture (sự kiện liên quan đến viết vuốt chạm màn hình) sẽ được handle bởi **GestureBinding**.

Khi *Flutter Engine* gửi các thông tin liên quan đến sự kiện gesture thì **GestureBinding** sẽ intercept các thông tin này bằng cách sử dụng API *window.onPointerDataPacket*, thực hiện việc buffering và:

1. Chuyển đổi các toạ độ được emit bởi Flutter Engine sao cho khớp với tỉ lệ pixel của device.
2. Yêu cầu **renderView** cung cấp **TẤT CẢ** *RenderObjects* phụ trách phần màn hình có chứa toạ độ của sự kiện gesture.
3. Duyệt qua list *renderObject* đó để lần lượt dispatch event gesture tương ứng cho mỗi phần tử của list.
4. Khi *renderObject* nhận được event thì nó sẽ bắt đầu xử lý.

# Phần 4: Animation

Phần này mình sẽ làm rõ hơn về khái niệm **Animation** và **Ticker**.

Bình thường khi start một Animation thì chúng ta sẽ sử dụng một **AnimationController** hoặc Widget tương tự bất kỳ nào đấy.

Trong Flutter thì toàn bộ mọi thứ liên quan đến animation đều dính dáng đến một khái niệm là [Ticker](https://docs.flutter.io/flutter/scheduler/Ticker-class.html).

Ticker chỉ làm một nhiệm vụ duy nhất, đó là nó sẽ yêu cầu **SchedulerBinding** đăng ký một one-time callback cho nó. Callback này sẽ được invoke khi Flutter Engine đã sẵn sàng để tiếp nhận request vẽ một frame mới.

Flutter Engine sẽ báo hiệu thời điểm này cho SchedulerBinding biết thông qua việc nó invoke callback `onBeginFrame`. SchedulerBinding khi này sẽ duyệt qua và invoke callback của tất cả các ticker (việc này gọi là tick) được đăng ký.

Mỗi sự kiện tick này sẽ được animation controller intercept để cung cấp thông tin liên quan đến frame tiếp theo của animation. Khi animation kết thúc thì ticker sẽ bị *“disabled”*. Trường hợp chưa kết thúc thì ticker sẽ request SchedulerBinding schedule một callback tiếp theo, và cứ thế lặp lại cho đến khi animation kết thúc…

## BuildContext

Nếu các bạn chưa xem qua class `Element` thì cụ tỉ signature của nó là dư này:

```dart
abstract class Element extends DiagnosticableTree implements BuildContext {  
    ...  
}
```

Nhìn lướt qua thì chắc mọi người sẽ nhận ra thứ khá quen thuộc: [BuildContext](https://docs.flutter.io/flutter/widgets/BuildContext-class.html). Chúng ta chủ yếu chỉ dùng `BuildContext` trong hàm `build()` của StatelessWidget và StatefulWidget, hoặc là trong object **State** của StatefulWidget

> *Chúng ta có thể coi* `*BuildContext*` *như là đại diện cho* `*Element*` *có thể tham chiếu đến từ Widget.*

Vậy là phần lớn chúng ta đã vô thức dùng Element mà không hề hay biết, cả mình cũng vậy =))

## BuildContext hữu dụng như nào?

Chúng ta có thể dùng `BuildContext` của một widget để làm những việc như:

- Lấy tham chiếu của *RenderObject* tương ứng với *widget* (hoặc nếu widget đó không thuộc loại *renderer* thì sẽ lấy được của widget con).
- Lấy được size của *RenderObject*.
- Lấy được thông tin của các ancestor widget (các widget có level thấp hơn nó, tức là gần root hơn).

# Kết luận

Cảm ơn bạn đã đọc đến cuối bài viết. Hi vọng qua bài viết này các bạn đã có cái nhìn rõ ràng hơn về cách mà Flutter hoạt động. Đặc biệt là những khái niệm như *Widget*, *Element*, *BuildContext* và *RenderObject* — những thứ mà chúng ta sẽ phải đụng chạm đến rất nhiều.

Gặp lại các bạn ở những bài viết sau.

*Happy coding~*

***Nguồn tham khảo****:* [Flutter - Flutter internals](https://www.didierboelens.com/2019/09/flutter-internals/)

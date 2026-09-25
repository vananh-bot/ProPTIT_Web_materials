# Spring Boot


## 1. Khởi tạo dự án Spring Boot

### 1.1. Cách tạo dự án

Cách phổ biến nhất là dùng [Spring Initializr](https://start.spring.io) hoặc tạo trực tiếp trong IntelliJ IDEA (New Project → Spring Boot).

| Mục | Giá trị gợi ý |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | 3.x (bản ổn định mới nhất) |
| Java | 17 hoặc 21 |
| Dependencies | Spring Web, Thymeleaf, Lombok |

### 1.2. Cấu trúc thư mục cơ bản

```
src/
├── main/
│   ├── java/com/example/demo/
│   │   ├── DemoApplication.java
│   │   ├── controller/
│   │   ├── model/
│   │   └── service/
│   └── resources/
│       ├── application.properties
│       ├── static/          (CSS, JS, ảnh)
│       └── templates/       (file Thymeleaf .html)
└── test/
```

### 1.3. Chạy ứng dụng

Chạy method `main` trong class có `@SpringBootApplication`, sau đó truy cập `http://localhost:8080`.

---

## 2. Bean trong Spring Boot

### 2.1. Bean là gì?

Bean là một đối tượng do **Spring IoC Container** khởi tạo, quản lý vòng đời và cung cấp cho các thành phần khác thông qua **Dependency Injection (DI)**. Thay vì tự `new` đối tượng, ta để Spring làm việc đó.

Spring Boot có thể tự nhận diện bean theo hai cách chính: quét tự động bằng `@Component` (và các annotation kế thừa từ nó) hoặc khai báo thủ công bằng `@Bean`.

### 2.2. Cách 1: `@Component` (quét tự động)

Dùng cho các class do mình viết và có quyền sửa source code. Spring sẽ tự phát hiện nhờ **component scanning**.

```java
@Component("collegeBean")
public class College {
    public void test() {
        System.out.println("Test College Method");
    }
}
```

### 2.3. Cách 2: `@Bean` (khai báo thủ công)

Dùng khi class đến từ thư viện bên thứ ba (bạn không sửa được source), hoặc khi cần khởi tạo với logic tùy biến. Khai báo trong một class `@Configuration`:

```java
@Configuration
public class CollegeConfig {

    @Bean
    public College collegeBean() {
        return new College();
    }
}
```

Lưu ý quan trọng:

- **Tên bean mặc định là tên method** (ở đây là `collegeBean`). Nếu method không có `@Bean`, Spring sẽ không tạo bean và bạn sẽ gặp lỗi `NoSuchBeanDefinitionException` khi gọi `getBean("collegeBean", ...)`.
- **Đổi tên bean:** `@Bean(name = "myCollegeBean")`.
- **Nhiều tên cho cùng một bean:** `@Bean(name = {"myCollegeBean", "yourCollegeBean"})`.
- Khi đã dùng `@Bean` trong class `@Configuration`, không cần dùng `@ComponentScan` cho việc tạo bean đó.

### 2.4. Lấy bean từ container

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context =
                new AnnotationConfigApplicationContext(CollegeConfig.class);

        College college = context.getBean("collegeBean", College.class);
        college.test();
    }
}
```

Trong Spring Boot thực tế, bạn thường không cần gọi `getBean` thủ công mà để Spring tự inject bằng `@Autowired`.

### 2.5. Dependency Injection với `@Bean`

Giả sử class `College` cần một dependency là `Principal`. Có hai cách inject phổ biến.

**Constructor Injection (khuyến khích):**

```java
public class College {
    private Principal principal;

    public College(Principal principal) {
        this.principal = principal;
    }

    public void test() {
        principal.principalInfo();
        System.out.println("Test College Method");
    }
}
```

```java
@Configuration
public class CollegeConfig {

    @Bean
    public Principal principalBean() {
        return new Principal();
    }

    @Bean
    public College collegeBean() {
        return new College(principalBean());
    }
}
```

**Setter Injection:**

```java
public class College {
    private Principal principal;

    public void setPrincipal(Principal principal) {
        this.principal = principal;
    }
}
```

```java
@Bean
public College collegeBean() {
    College college = new College();
    college.setPrincipal(principalBean());
    return college;
}
```

Constructor injection thường được ưu tiên vì đảm bảo object luôn có đủ dependency ngay khi được tạo.

### 2.6. BeanFactory và ApplicationContext

| Tiêu chí | BeanFactory | ApplicationContext |
|---|---|---|
| Vai trò | Interface gốc của IoC container | Mở rộng từ BeanFactory |
| Khởi tạo bean | Lazy (khi được gọi) | Eager (khởi tạo ngay khi start) |
| Tính năng thêm | Cơ bản | Event, i18n, load message, AOP... |
| Dùng trong Spring Boot | Ít khi dùng trực tiếp | Dùng phổ biến |

### 2.7. Vòng đời của một Bean

Quá trình khởi tạo và hủy một bean diễn ra theo các bước:

1. **Instantiation:** Spring tạo đối tượng bean (gọi constructor).
2. **Populate properties:** Spring inject các dependency (DI).
3. **Aware callbacks:** gọi các interface như `BeanNameAware`, `ApplicationContextAware` nếu có.
4. **BeanPostProcessor (before):** xử lý trước khi khởi tạo.
5. **Initialization:** gọi method có `@PostConstruct`, `InitializingBean.afterPropertiesSet()` hoặc `initMethod`.
6. **BeanPostProcessor (after):** xử lý sau khi khởi tạo.
7. **Sẵn sàng sử dụng.**
8. **Destruction:** khi container đóng, gọi method có `@PreDestroy` hoặc `DisposableBean.destroy()`.

```java
@Component
public class MyService {

    @PostConstruct
    public void init() {
        System.out.println("Bean đã được khởi tạo");
    }

    @PreDestroy
    public void cleanup() {
        System.out.println("Bean sắp bị hủy");
    }
}
```

---

## 3. Các Annotation quan trọng

Annotation là metadata cung cấp chỉ dẫn cho Spring khi chạy hoặc khi biên dịch. Thay vì cấu hình XML phức tạp, ta có thể cấu hình trực tiếp trong code Java. Nhờ đó:

- Giảm cấu hình XML.
- Code dễ đọc và dễ bảo trì hơn.
- Hỗ trợ auto-configuration.
- Đơn giản hóa dependency injection.
- Tăng tốc độ phát triển ứng dụng.

### 3.1. Annotation cấu hình ứng dụng

| Annotation | Ý nghĩa |
|---|---|
| `@SpringBootApplication` | Đánh dấu class chính của ứng dụng. Gộp ba annotation: `@SpringBootConfiguration`, `@EnableAutoConfiguration`, `@ComponentScan` |
| `@SpringBootConfiguration` | Đánh dấu class cung cấp cấu hình cho Spring Boot (phiên bản chuyên biệt của `@Configuration`) |
| `@EnableAutoConfiguration` | Bật cơ chế auto-configuration, tự cấu hình bean dựa trên dependency trong classpath và file properties |
| `@ComponentScan` | Chỉ định package cần quét để tìm các component như `@Controller`, `@Service`, `@Repository` |
| `@Configuration` | Đánh dấu class chứa các định nghĩa bean, thay thế cho cấu hình XML |

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

### 3.2. Annotation đánh dấu bean

| Annotation | Ý nghĩa | Tầng sử dụng |
|---|---|---|
| `@Component` | Component chung do Spring quản lý | Bất kỳ |
| `@Service` | Chứa logic nghiệp vụ | Service layer |
| `@Repository` | Tương tác với database, tự chuyển đổi exception database thành exception của Spring | DAO layer |
| `@Controller` | Xử lý request trong MVC | Web layer |
| `@Bean` | Khai báo bean từ giá trị trả về của method, cho phép kiểm soát hoàn toàn việc tạo bean | Class `@Configuration` |

Ví dụ `@Bean` để tạo `RestTemplate`:

```java
@Configuration
public class AppConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

### 3.3. Annotation Dependency Injection

| Annotation | Ý nghĩa |
|---|---|
| `@Autowired` | Tự động inject dependency, không cần tạo object thủ công |
| `@Qualifier("tên")` | Chỉ định bean cụ thể khi có nhiều bean cùng kiểu |
| `@Primary` | Đánh dấu bean mặc định khi có nhiều ứng viên và không có `@Qualifier` |

```java
@Autowired
@Qualifier("emailService")
private NotificationService service;
```

```java
@Primary
@Component
public class SmsService implements NotificationService {
}
```

Từ Spring 4.3 trở đi, nếu class chỉ có một constructor thì không bắt buộc phải viết `@Autowired`.

### 3.4. Annotation Web và REST API

| Annotation | Ý nghĩa |
|---|---|
| `@RestController` | Tạo web service RESTful, tự động trả về dữ liệu dạng JSON hoặc XML |
| `@RequestMapping("/api")` | Ánh xạ request HTTP tới controller hoặc method, định nghĩa đường dẫn gốc |
| `@GetMapping` | Lấy dữ liệu (HTTP GET) |
| `@PostMapping` | Gửi dữ liệu lên server (HTTP POST) |
| `@PutMapping` | Cập nhật dữ liệu (HTTP PUT) |
| `@DeleteMapping` | Xóa dữ liệu (HTTP DELETE) |
| `@PathVariable` | Lấy giá trị từ đường dẫn URI |
| `@RequestParam` | Lấy tham số query trên URL (dùng cho lọc, tìm kiếm) |
| `@RequestBody` | Chuyển phần body của request thành object Java (thường dùng với POST, PUT) |

Ví dụ:

```java
@RestController
@RequestMapping("/api")
public class UserController {

    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.getAllUsers();
    }

    @GetMapping("/users/{id}")
    public User getUser(@PathVariable int id) {
        return userService.getUser(id);
    }

    @GetMapping("/search")
    public String search(@RequestParam String keyword) {
        return keyword;
    }

    @PostMapping("/users")
    public User saveUser(@RequestBody User user) {
        return userService.save(user);
    }
}
```

### 3.5. Annotation cấu hình và thuộc tính

| Annotation | Ý nghĩa |
|---|---|
| `@Value("${server.port}")` | Inject một giá trị đơn lẻ từ `application.properties` hoặc `application.yml` |
| `@ConfigurationProperties(prefix = "app")` | Gom nhóm nhiều thuộc tính liên quan vào một class POJO theo kiểu an toàn |

```java
@Value("${server.port}")
private String port;
```

```java
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    private String name;
}
```

### 3.6. Annotation validation

Dùng để kiểm tra tính đúng đắn của dữ liệu đầu vào.

| Annotation | Ý nghĩa |
|---|---|
| `@NotNull` | Giá trị không được null |
| `@NotBlank` | Chuỗi phải có ít nhất một ký tự không phải khoảng trắng |
| `@Email` | Kiểm tra định dạng email |
| `@Size` | Giới hạn độ dài hoặc kích thước |

```java
public class User {
    @NotBlank
    private String username;

    @Email
    private String email;
}
```

### 3.7. Annotation xử lý exception

| Annotation | Ý nghĩa |
|---|---|
| `@ExceptionHandler` | Xử lý exception cụ thể trong một controller |
| `@ControllerAdvice` | Xử lý exception tập trung cho tất cả controller |

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public String handleException() {
        return "Error occurred";
    }
}
```

### 3.8. Annotation JPA và database

| Annotation | Mục đích |
|---|---|
| `@Entity` | Đánh dấu class là một JPA entity (ánh xạ với bảng) |
| `@Table` | Chỉ định tên bảng |
| `@Id` | Đánh dấu khóa chính |
| `@GeneratedValue` | Tự sinh giá trị khóa chính |
| `@Column` | Ánh xạ field với cột trong bảng |

---

## 4. Mô hình thiết kế MVC

### 4.1. MVC là gì?

MVC (Model–View–Controller) là mô hình thiết kế chia ứng dụng thành ba thành phần riêng biệt. Sự phân tách này giúp cải thiện việc tổ chức code, khả năng bảo trì và khả năng mở rộng. Mỗi thành phần đảm nhiệm một trách nhiệm cụ thể nên việc sửa đổi hoặc mở rộng ứng dụng dễ dàng hơn.

### 4.2. Ba thành phần

| Thành phần | Vai trò |
|---|---|
| **Model** | Quản lý dữ liệu ứng dụng và logic nghiệp vụ. Xử lý các quy tắc nghiệp vụ và phản hồi yêu cầu thông tin từ View, Controller |
| **View** | Hiển thị dữ liệu từ Model cho người dùng và gửi dữ liệu nhập của người dùng đến Controller. Hoạt động thụ động, không tương tác trực tiếp với Model |
| **Controller** | Trung gian giữa Model và View. Nhận dữ liệu đầu vào, cập nhật Model, cập nhật View và chứa logic ứng dụng như kiểm tra dữ liệu, chuyển đổi dữ liệu |

### 4.3. Ví dụ thực tế

Một trang web thương mại điện tử áp dụng MVC:

- **Model:** quản lý dữ liệu sản phẩm, tài khoản người dùng và thông tin đơn hàng từ cơ sở dữ liệu.
- **View:** hiển thị trang sản phẩm, giỏ hàng và chi tiết đơn hàng.
- **Controller:** xử lý yêu cầu như đăng nhập hoặc thanh toán, rồi cập nhật Model và View tương ứng.

### 4.4. Luồng giao tiếp giữa các thành phần

1. Người dùng tương tác với giao diện (click nút, nhập văn bản vào form).
2. View nhận dữ liệu nhập và chuyển tiếp đến Controller.
3. Controller diễn giải dữ liệu, thực hiện thao tác cần thiết (ví dụ cập nhật Model) và quyết định cách phản hồi.
4. Controller cập nhật Model dựa trên dữ liệu đầu vào hoặc logic ứng dụng.
5. Nếu Model thay đổi, nó thông báo cho View.
6. View yêu cầu dữ liệu từ Model để cập nhật hiển thị.
7. Controller cập nhật View dựa trên thay đổi của Model hoặc phản hồi của người dùng.
8. View hiển thị giao diện đã được cập nhật.

### 4.5. Ví dụ triển khai bằng Java thuần

Bài toán: quản lý sinh viên, Model lưu tên và mã sinh viên, View hiển thị thông tin, Controller điều phối giữa hai thành phần.

**Model (`Student`):**

```java
class Student {
    private String rollNo;
    private String name;

    public String getRollNo() { return rollNo; }
    public void setRollNo(String rollNo) { this.rollNo = rollNo; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```

**View (`StudentView`):**

```java
class StudentView {
    public void printStudentDetails(String studentName, String studentRollNo) {
        System.out.println("Student:");
        System.out.println("Name: " + studentName);
        System.out.println("Roll No: " + studentRollNo);
    }
}
```

**Controller (`StudentController`):**

```java
class StudentController {
    private Student model;
    private StudentView view;

    public StudentController(Student model, StudentView view) {
        this.model = model;
        this.view = view;
    }

    public void setStudentName(String name) { model.setName(name); }
    public String getStudentName() { return model.getName(); }

    public void setStudentRollNo(String rollNo) { model.setRollNo(rollNo); }
    public String getStudentRollNo() { return model.getRollNo(); }

    public void updateView() {
        view.printStudentDetails(model.getName(), model.getRollNo());
    }
}
```

**Chạy thử:**

```java
public class MVCPattern {
    public static void main(String[] args) {
        Student model = retrieveStudentFromDatabase();
        StudentView view = new StudentView();
        StudentController controller = new StudentController(model, view);

        controller.updateView();

        controller.setStudentName("Vikram Sharma");
        controller.updateView();
    }

    private static Student retrieveStudentFromDatabase() {
        Student student = new Student();
        student.setName("Lokesh Sharma");
        student.setRollNo("15UCS157");
        return student;
    }
}
```

**Kết quả:**

```
Student:
Name: Lokesh Sharma
Roll No: 15UCS157
Student:
Name: Vikram Sharma
Roll No: 15UCS157
```

Ví dụ này minh họa đúng tinh thần MVC: khi Controller đổi tên trong Model, View chỉ cần được gọi lại để hiển thị dữ liệu mới, còn Model và View không phụ thuộc trực tiếp vào nhau.

### 4.6. Ưu điểm và nhược điểm

**Ưu điểm:**

- Phân tách rõ ràng giúp cải thiện khả năng bảo trì.
- Cho phép phát triển song song giao diện và logic nghiệp vụ.
- Dễ kiểm thử, đặc biệt là unit test.
- Hỗ trợ mở rộng và thêm tính năng mới mượt mà hơn.

**Nhược điểm:**

- Độ phức tạp tăng lên với ứng dụng nhỏ hoặc đơn giản.
- Cần nhiều công đoạn thiết kế và lập kế hoạch ban đầu.
- Có thể khó hiểu và khó áp dụng đối với người mới bắt đầu.

---

## 5. Spring MVC và Thymeleaf

### 5.1. Ánh xạ MVC vào Spring Boot

| Thành phần MVC | Trong Spring Boot |
|---|---|
| Model | Class Java (POJO), Service, Entity |
| View | File HTML Thymeleaf trong `templates/` |
| Controller | Class `@Controller` |

### 5.2. `@Controller` và `@GetMapping`

```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home(Model model) {
        model.addAttribute("message", "Xin chào!");
        return "index";  // trả về templates/index.html
    }
}
```

Lưu ý: `@Controller` trả về **tên view** (Spring sẽ tìm file HTML tương ứng), khác với `@RestController` trả về **dữ liệu** (JSON hoặc XML).

### 5.3. Thymeleaf

Thymeleaf là template engine giúp nhúng dữ liệu vào HTML.

| Cú pháp | Ý nghĩa |
|---|---|
| `th:text="${message}"` | Hiển thị giá trị biến |
| `th:each="item : ${list}"` | Lặp qua danh sách |
| `th:if="${condition}"` | Hiển thị có điều kiện |
| `th:href="@{/path}"` | Tạo đường dẫn |

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Trang cá nhân</title>
</head>
<body>
    <h1 th:text="${message}">Tiêu đề</h1>
</body>
</html>
```

---

## 6. Lombok

Lombok giúp giảm code lặp lại (getter, setter, constructor, toString...) thông qua annotation.

Thêm dependency:

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

| Annotation | Tác dụng |
|---|---|
| `@Getter` | Sinh getter cho tất cả field |
| `@Setter` | Sinh setter cho tất cả field |
| `@ToString` | Sinh method `toString()` |
| `@EqualsAndHashCode` | Sinh `equals()` và `hashCode()` |
| `@Data` | Gộp `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor` |
| `@NoArgsConstructor` | Sinh constructor không tham số |
| `@AllArgsConstructor` | Sinh constructor với tất cả field |
| `@RequiredArgsConstructor` | Sinh constructor với các field `final` |
| `@Builder` | Sinh cơ chế builder pattern |

Ví dụ:

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Person {
    private String fullName;
    private int age;
    private String email;
}
```

Sử dụng builder:

```java
Person p = Person.builder()
        .fullName("Nguyễn Văn A")
        .age(22)
        .email("a@example.com")
        .build();
```

---

## 7. Logging trong Spring Boot

### 7.1. Vì sao cần logging?

Logging ghi lại thông tin, hành động và sự kiện xảy ra trong ứng dụng. Nhờ log, bạn có thể theo dõi hoạt động, giám sát hiệu năng, hiểu hành vi của ứng dụng và tìm nguyên nhân khi có lỗi.

### 7.2. Các thành phần của một framework logging

| Thành phần | Vai trò |
|---|---|
| **Logger** | Ghi nhận các thông điệp (message) |
| **Formatter** | Định dạng thông điệp do Logger ghi nhận |
| **Handler** | Xuất thông điệp ra console, file hoặc gửi qua email... |

### 7.3. Các mức log (Log level)

| Level | Mục đích |
|---|---|
| **ERROR** | Lỗi không thể phục hồi |
| **WARN** | Lỗi có thể phục hồi |
| **INFO** | Thông tin phục vụ kiểm toán, theo dõi hoạt động |
| **DEBUG** | Thông tin phục vụ điều tra lỗi |
| **TRACE** | Thông tin chi tiết để điều tra sâu |

Thứ tự từ mức cao đến thấp: ERROR > WARN > INFO > DEBUG > TRACE.

### 7.4. Logging mặc định (không cần cấu hình)

Spring Boot hiển thị log ra console ngay cả khi không có cấu hình nào. Lý do là Spring Boot dùng **Logback** làm framework mặc định. Khi dùng `spring-boot-starter-web`, starter này kéo theo `spring-boot-starter-logging` và tự động đưa Logback vào project.

Ví dụ một controller ghi log:

```java
@RestController
public class LogController {

    Logger logger = LoggerFactory.getLogger(LogController.class);

    @RequestMapping("/log")
    public String log() {
        logger.trace("Log level: TRACE");
        logger.debug("Log level: DEBUG");
        logger.info("Log level: INFO");
        logger.warn("Log level: WARN");
        logger.error("Log level: ERROR");

        return "Hey! You can check the output in the logs";
    }
}
```

Chạy ứng dụng và truy cập `http://localhost:8080/log` để xem log.

Mỗi dòng log mặc định gồm các thành phần:

- Ngày ghi log
- Thời gian (có độ chính xác đến mili-giây)
- Mức log (mặc định hiển thị INFO, WARN, ERROR)
- Process ID
- Dấu `---` ngăn cách
- Tên thread (đặt trong dấu ngoặc vuông)
- Tên logger (thường là tên class nguồn)
- Nội dung log

### 7.5. Bật mức DEBUG hoặc TRACE

Mặc định chỉ ERROR, WARN và INFO được in ra console. Để bật DEBUG hoặc TRACE, có thể:

Khi chạy bằng dòng lệnh:

```bash
java -jar target/log-0.0.1-SNAPSHOT.jar --debug
java -jar target/log-0.0.1-SNAPSHOT.jar --trace
```

Hoặc thêm vào `application.properties`:

```properties
debug=true
trace=true
```

### 7.6. Log có màu

Nếu terminal hỗ trợ ANSI, có thể bật log có màu bằng cách thêm vào `application.properties`:

```properties
spring.output.ansi.enabled=always
```

### 7.7. Ghi log ra file

Mặc định Spring Boot chỉ ghi log ra console. Để ghi thêm vào file, thêm vào `application.properties`:

```properties
logging.file.path=logs/
logging.file.name=logs/application.log
```

Sau khi chạy, thư mục `logs/` sẽ được tạo và chứa file `application.log`.

### 7.8. Cấu hình bằng Logback (`logback-spring.xml`)

Khi cần tùy biến định dạng, màu sắc, đường dẫn file và chính sách xoay vòng (rolling policy), tạo file cấu hình trong `src/main/resources`. Spring Boot tự động ghi đè cấu hình mặc định nếu tìm thấy một trong các file:

- `logback-spring.xml` (khuyến khích)
- `logback.xml`
- `logback-spring.groovy`
- `logback.groovy`

Ví dụ `logback-spring.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Đường dẫn và tên file log -->
    <property name="LOG_PATH" value="./logs" />
    <property name="LOG_FILE_NAME" value="application_logback" />

    <!-- Định dạng log cho console -->
    <appender name="ConsoleOutput" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %white(%d{ISO8601}) %highlight(%-5level) [%yellow(%t)] : %msg%n%throwable
            </Pattern>
        </layout>
    </appender>

    <!-- Định dạng log cho file, có rolling policy -->
    <appender name="LogFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/${LOG_FILE_NAME}.log</file>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level : %msg%n</Pattern>
        </encoder>

        <!-- Xoay file theo ngày hoặc khi đạt 10MB -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/archived/${LOG_FILE_NAME}-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>

    <!-- Mức INFO cho toàn bộ ứng dụng -->
    <root level="info">
        <appender-ref ref="LogFile" />
        <appender-ref ref="ConsoleOutput" />
    </root>

    <!-- Mức TRACE riêng cho package com.log -->
    <logger name="com.log" level="trace" additivity="false">
        <appender-ref ref="LogFile" />
        <appender-ref ref="ConsoleOutput" />
    </logger>

</configuration>
```

Ý nghĩa các phần chính:

- **Appender:** nơi log được ghi (console hoặc file).
- **Pattern:** định dạng của từng dòng log.
- **Rolling policy:** chia nhỏ file log theo ngày hoặc theo dung lượng để tránh file quá lớn.
- **root:** cấu hình mức log mặc định cho toàn ứng dụng.
- **logger:** cấu hình mức log riêng cho một package.

### 7.9. Cấu hình bằng Log4j2

Log4j2 là framework thay thế Logback. Để dùng, cần loại bỏ Logback khỏi starter và thêm dependency của Log4j2:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

Tạo file `log4j2-spring.xml` (hoặc `log4j2.xml`) trong `src/main/resources`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>

    <Properties>
        <property name="LOG_PATH" value="./logs" />
        <property name="LOG_FILE_NAME" value="application-log4j2" />
    </Properties>

    <Appenders>
        <Console name="ConsoleOutput" target="SYSTEM_OUT">
            <PatternLayout
                pattern="%style{%d{ISO8601}}{white} %highlight{%-5level} [%style{%t}{bright,yellow}] : %msg%n%throwable"
                disableAnsi="false" />
        </Console>

        <RollingFile name="LogFile"
            fileName="${LOG_PATH}/${LOG_FILE_NAME}.log"
            filePattern="${LOG_PATH}/$${date:yyyy-MM}/application-log4j2-%d{dd-MMMM-yyyy}-%i.log.gz">
            <PatternLayout>
                <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level : %msg%n</pattern>
            </PatternLayout>
            <Policies>
                <OnStartupTriggeringPolicy />
                <SizeBasedTriggeringPolicy size="10 MB" />
                <TimeBasedTriggeringPolicy />
            </Policies>
        </RollingFile>
    </Appenders>

    <Loggers>
        <Root level="info">
            <AppenderRef ref="ConsoleOutput" />
            <AppenderRef ref="LogFile" />
        </Root>

        <logger name="com.log" level="trace" additivity="false">
            <appender-ref ref="LogFile" />
            <appender-ref ref="ConsoleOutput" />
        </logger>
    </Loggers>

</Configuration>
```

Cấu trúc tương tự Logback: `Appenders` định nghĩa nơi ghi log, `Loggers` định nghĩa mức log cho root và từng package.

### 7.10. Logging với Lombok và `@Slf4j`

Thay vì khai báo logger thủ công, Lombok cung cấp annotation để tự sinh logger. Thêm dependency Lombok (đã có ở mục 6), sau đó:

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class Main {
    public static void main(String[] args) {
        log.info("Info level");
        log.error("Error level");
    }
}
```

`@Slf4j` tự động thêm một field tên `log` vào class, tương đương với:

```java
private static final Logger log = LoggerFactory.getLogger(Main.class);
```

Ngoài `@Slf4j`, còn có `@CommonsLog` (dùng Apache Commons Logging). Điểm lợi của `@Slf4j` là **code không phụ thuộc vào framework log cụ thể**, bạn có thể đổi từ Logback sang Log4j2 mà không cần sửa code.

---

## 8. Bài tập bắt buộc

### 8.1. Đề bài

Xây dựng một trang web giới thiệu thông tin cá nhân sử dụng **Spring Boot**, **Thymeleaf** và mô hình **MVC**.

Yêu cầu:

- Thông tin hiển thị (họ tên, tuổi, email, địa chỉ, sở thích...) phải được lấy từ một **Object** (Model). Đầu vào có thể hardcode.
- Tách rõ Model, View và Controller.
- Sử dụng Lombok để giảm code lặp lại.
- Ghi log bằng `@Slf4j` khi truy cập trang.

### 8.2. Cấu trúc gợi ý

```
src/main/java/com/example/demo/
├── DemoApplication.java
├── controller/
│   └── ProfileController.java
└── model/
    └── Profile.java
src/main/resources/templates/
└── profile.html
```

### 8.3. Bước 1: Tạo Model

```java
package com.example.demo.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Profile {
    private String fullName;
    private int age;
    private String email;
    private String address;
    private String hobby;
}
```

### 8.4. Bước 2: Tạo Controller

```java
package com.example.demo.controller;

import com.example.demo.model.Profile;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
public class ProfileController {

    @GetMapping("/profile")
    public String showProfile(Model model) {
        Profile profile = Profile.builder()
                .fullName("Nguyễn Văn A")
                .age(22)
                .email("nguyenvana@example.com")
                .address("Hà Nội, Việt Nam")
                .hobby("Lập trình, đọc sách, du lịch")
                .build();

        model.addAttribute("profile", profile);
        log.info("Đã hiển thị trang thông tin cá nhân: {}", profile.getFullName());
        return "profile";
    }
}
```

### 8.5. Bước 3: Tạo View (`templates/profile.html`)

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Thông tin cá nhân</title>
</head>
<body>
    <h1>Thông tin cá nhân</h1>
    <ul>
        <li>Họ và tên: <span th:text="${profile.fullName}"></span></li>
        <li>Tuổi: <span th:text="${profile.age}"></span></li>
        <li>Email: <span th:text="${profile.email}"></span></li>
        <li>Địa chỉ: <span th:text="${profile.address}"></span></li>
        <li>Sở thích: <span th:text="${profile.hobby}"></span></li>
    </ul>
</body>
</html>
```

### 8.6. Bước 4: Chạy và kiểm tra

1. Chạy `DemoApplication`.
2. Truy cập `http://localhost:8080/profile`.
3. Kiểm tra thông tin hiển thị đúng và dòng log xuất hiện trên console.

Nâng cao (tùy chọn): thêm file `logback-spring.xml` để ghi log ra file `logs/application.log`.

### 8.7. Tiêu chí đánh giá

| Tiêu chí | Yêu cầu |
|---|---|
| Cấu trúc MVC | Có đủ Model, View, Controller tách biệt |
| Dữ liệu | Lấy từ Object, không hardcode trực tiếp trong HTML |
| Lombok | Dùng `@Data`, `@Builder` hoặc tương đương |
| Log | Có `@Slf4j` và ghi log khi truy cập |
| Giao diện | Hiển thị đúng bằng Thymeleaf |

## 1. Eureka Server

#### 1. **Tổng quan về Eureka**
- **Eureka** là một **service registry** do Netflix phát triển, chủ yếu sử dụng trong các hệ thống microservices.
- Nó được thiết kế để quản lý và theo dõi trạng thái của các dịch vụ trong hệ thống phân tán.
- Với Eureka, các dịch vụ có thể **tự động đăng ký** và **khám phá lẫn nhau**, điều này rất quan trọng trong môi trường **cloud-native** hoặc **containerized**.

#### 2. **Cách hoạt động của Eureka Server và Eureka Client**
- **Eureka Server** lưu trữ thông tin về các dịch vụ đã đăng ký như tên dịch vụ, địa chỉ mạng (host, port) và trạng thái (health).
- **Eureka Client** (các dịch vụ) sẽ:
    - **Đăng ký** vào Eureka Server khi khởi động.
    - Gửi **heartbeat** định kỳ để thông báo rằng dịch vụ vẫn hoạt động.
    - **Tự động hủy đăng ký** khi dịch vụ ngừng hoạt động.
- Các dịch vụ khác có thể sử dụng Eureka Server để **khám phá** vị trí của các dịch vụ đã đăng ký, từ đó giao tiếp mà không cần biết trước địa chỉ cụ thể.

#### 3. **Cách cấu hình một Eureka Server**
- Để tạo một **Eureka Server**, bạn cần cài đặt dependency cho `spring-cloud-starter-netflix-eureka-server` vào dự án **Spring Boot**:
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  </dependency>
  ```
- Trong class chính của ứng dụng, thêm annotation `@EnableEurekaServer`:
  ```java
  @SpringBootApplication
  @EnableEurekaServer
  public class EurekaServerApplication {
      public static void main(String[] args) {
          SpringApplication.run(EurekaServerApplication.class, args);
      }
  }
  ```
- Cấu hình tệp `application.properties`:
  ```properties
  spring.application.name=eureka-server
  eureka.client.register-with-eureka=false
  eureka.client.fetch-registry=false
  server.port=8761
  ```
- **Port 8761** thường được sử dụng cho Eureka Server.

#### 4. **Cách cấu hình một Eureka Client**
- **Eureka Clients** là các dịch vụ khác nhau trong hệ thống sẽ đăng ký với **Eureka Server**.
- Để cấu hình client, thêm dependency `spring-cloud-starter-netflix-eureka-client` vào dự án:
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  ```
- Cấu hình tệp `application.properties` cho client:
  ```properties
  spring.application.name=my-service
  eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
  eureka.instance.prefer-ip-address=true
  ```
- Client sẽ đăng ký vào **Eureka Server** tại địa chỉ `http://localhost:8761/eureka/`.

#### 5. **Health Check và Heartbeat**
- **Eureka Client** sẽ gửi heartbeat theo khoảng thời gian cố định (thường là 30 giây) để báo hiệu rằng dịch vụ vẫn đang hoạt động.
- **Eureka Server** sẽ kiểm tra trạng thái của các dịch vụ thông qua các heartbeat này.
- Nếu không nhận được heartbeat trong thời gian cho phép (timeout), **Eureka Server** sẽ loại bỏ dịch vụ khỏi danh sách đăng ký (deregister).

#### 6. **Chế độ tự bảo vệ (Self-preservation Mode)**
- Khi **Eureka Server** không nhận được số lượng heartbeat dự kiến trong một thời gian nhất định, nó sẽ kích hoạt **chế độ tự bảo vệ**.
- Thay vì loại bỏ các dịch vụ ngay lập tức, server sẽ giữ lại thông tin đăng ký để tránh mất dữ liệu quan trọng trong trường hợp hệ thống tạm thời gặp vấn đề.
- Điều này giúp hệ thống có thể tự ổn định sau khi sự cố được khắc phục.

#### 7. **Caching và Load Balancing**
- **Eureka Clients** có khả năng **caching** thông tin về các dịch vụ khác để giảm tải cho Eureka Server.
- Các dịch vụ có thể **load balance** giữa các instance khác nhau của cùng một dịch vụ thông qua thông tin lấy từ **Eureka Server**.

#### 8. **Tính nhất quán và sẵn sàng**
- Eureka ưu tiên **high availability** hơn là tính **consistency**. Điều này có nghĩa là các dịch vụ vẫn có thể hoạt động ngay cả khi thông tin về dịch vụ khác có thể không hoàn toàn chính xác.
- Trong môi trường phân tán, sự nhất quán hoàn toàn là khó khăn và không cần thiết. Do đó, Eureka chỉ đảm bảo rằng các dịch vụ có thể tiếp tục giao tiếp với nhau ngay cả khi server gặp vấn đề tạm thời.

#### 9. **Replication giữa các Eureka Server**
- Khi có nhiều **Eureka Server**, chúng sẽ tự động **replicate** thông tin dịch vụ lẫn nhau để đảm bảo tính sẵn sàng cao hơn.
- Điều này đảm bảo rằng các **Eureka Client** luôn có thể đăng ký và khám phá dịch vụ ngay cả khi một hoặc vài server bị ngừng hoạt động.

#### 10. **Sử dụng với Spring Cloud Gateway**
- **Spring Cloud Gateway** có thể tích hợp với **Eureka** để thực hiện **dynamic routing** và **load balancing**.
- Thay vì cấu hình tĩnh các route, **Gateway** có thể tự động lấy danh sách các dịch vụ từ **Eureka** để định tuyến các yêu cầu đến đúng dịch vụ dựa trên **service discovery**.

#### 11. **Eureka và các mô hình kiến trúc microservices**
- Trong kiến trúc **microservices**, số lượng dịch vụ có thể rất lớn và địa chỉ mạng của chúng có thể thay đổi liên tục do **scaling** hoặc **container orchestration**.
- Eureka cung cấp cơ chế **tự động phát hiện** và **đăng ký động** giúp các dịch vụ có thể giao tiếp linh hoạt và chính xác hơn.

#### 12. **Kiến trúc HA của Eureka Server**
- Eureka Server có thể được triển khai theo kiến trúc **high availability** bằng cách thiết lập nhiều instance.
- Các Eureka Server sẽ **replicate** dữ liệu lẫn nhau để đảm bảo rằng khi một server bị ngắt, các server còn lại vẫn có đầy đủ thông tin về các dịch vụ.

#### 13. **Ưu và nhược điểm của Eureka**
- **Ưu điểm**:
    - Giúp các dịch vụ có thể đăng ký và khám phá lẫn nhau tự động.
    - Cung cấp khả năng **scalability** và **fault tolerance** tốt.
    - Phù hợp cho các hệ thống **dynamic scaling**.
- **Nhược điểm**:
    - Yêu cầu cấu hình phức tạp khi triển khai trong môi trường lớn.
    - Độ trễ trong việc cập nhật thông tin dịch vụ có thể xảy ra trong hệ thống phân tán lớn.

### Kết luận
**Eureka Server** là một giải pháp hiệu quả cho việc quản lý và khám phá dịch vụ trong các hệ thống **microservices**. Nó cung cấp khả năng **service discovery**, **health monitoring**, và **dynamic scaling**, giúp các dịch vụ có thể giao tiếp và quản lý trạng thái một cách tự động, đồng thời đảm bảo tính sẵn sàng của hệ thống trong môi trường phân tán.

#### Tài liệu tham khảo:
- [Spring Cloud Netflix Documentation](https://spring.io/projects/spring-cloud-netflix)
- [Eureka Wiki](https://github.com/Netflix/eureka)

## 2. Config Server

#### 1. **Tổng quan về Spring Cloud Config**
- **Spring Cloud Config** là một giải pháp cho phép quản lý cấu hình tập trung, đặc biệt hữu ích trong môi trường **microservices**.
- Trong các hệ thống microservices, mỗi dịch vụ thường có các tệp cấu hình riêng, điều này có thể gây khó khăn khi phải đồng bộ và quản lý. **Config Server** giúp giải quyết vấn đề này bằng cách lưu trữ các tệp cấu hình tập trung và cung cấp chúng cho các dịch vụ client qua API.
- Cấu hình có thể được lưu trữ trong các kho lưu trữ bên ngoài như **Git**, **SVN**, **File System**, giúp dễ dàng cập nhật và quản lý.

#### 2. **Cách cấu hình một Spring Cloud Config Server**
- Để bắt đầu, bạn cần cài đặt dependency sau vào `pom.xml`:
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
  </dependency>
  ```
- Sau đó, kích hoạt **Config Server** bằng cách thêm annotation `@EnableConfigServer` vào class chính:
  ```java
  @SpringBootApplication
  @EnableConfigServer
  public class ConfigServerApplication {
      public static void main(String[] args) {
          SpringApplication.run(ConfigServerApplication.class, args);
      }
  }
  ```
- **Config Server** cần được chỉ định một nguồn cấu hình bên ngoài để đọc các tệp cấu hình từ đó. Ví dụ, để sử dụng **Git**, bạn có thể cấu hình trong `application.properties`:
  ```properties
  spring.cloud.config.server.git.uri=https://github.com/your-repo/config-repo
  ```
- **Config Server** sẽ lấy các tệp cấu hình từ kho Git và phục vụ chúng thông qua API để các ứng dụng client có thể truy xuất.

#### 3. **Cách cấu hình một Config Client**
- Các dịch vụ microservices cần lấy cấu hình từ **Config Server** được gọi là **Config Clients**. Để một dịch vụ trở thành client, bạn cần thêm dependency `spring-cloud-starter-config` vào `pom.xml`:
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
  </dependency>
  ```
- Sau đó, trong tệp `application.properties` hoặc `bootstrap.properties` của client, bạn phải chỉ ra URL của **Config Server**:
  ```properties
  spring.application.name=my-service
  spring.cloud.config.uri=http://localhost:8888
  ```
- Ở đây, `spring.application.name` được dùng để chỉ ra tên dịch vụ, và **Config Server** sẽ dựa vào tên này để trả về cấu hình tương ứng cho dịch vụ đó.

#### 4. **Cấu trúc thư mục và tệp cấu hình**
- Các tệp cấu hình của dịch vụ được lưu trữ trong Git hoặc các kho lưu trữ khác dưới dạng YAML hoặc properties.
- Các tệp có thể được chia làm ba loại chính:
  - `application.yml`: Áp dụng cho tất cả các ứng dụng, là cấu hình mặc định.
  - `my-service.yml`: Áp dụng cho ứng dụng cụ thể tên là **my-service**.
  - `application-{profile}.yml`: Áp dụng cho một profile cụ thể, ví dụ: `application-dev.yml` cho môi trường phát triển, `application-prod.yml` cho môi trường sản xuất.

#### 5. **Cấu hình Profile-based**
- **Profile-based configuration** cho phép cấu hình tùy theo môi trường như **dev**, **test**, **prod**.
- Ví dụ: tệp `application-prod.yml` có thể chứa thông tin cấu hình khác so với `application-dev.yml` như cơ sở dữ liệu hoặc các API khác nhau.
- Client có thể truy vấn cấu hình dựa trên profile bằng cách gửi yêu cầu với đường dẫn có chứa profile tương ứng, ví dụ:
  ```
  http://localhost:8888/my-service/dev
  ```

#### 6. **Lấy cấu hình từ Config Server**
- **Config Clients** lấy cấu hình từ **Config Server** khi khởi động hoặc khi được yêu cầu.
- Cấu hình có thể được truy xuất dưới dạng JSON bằng REST API:
  ```bash
  http://localhost:8888/my-service/default
  ```
- Trong đó, `my-service` là tên của ứng dụng, và `default` là profile mặc định. **Config Server** sẽ trả về tệp cấu hình dưới dạng JSON.

#### 7. **Cập nhật cấu hình động (Hot Reload)**
- **Spring Cloud Bus** có thể được tích hợp để hỗ trợ việc **cập nhật cấu hình động** cho các dịch vụ mà không cần phải khởi động lại.
- Khi có sự thay đổi trong kho cấu hình (ví dụ Git), một lệnh **refresh** sẽ được gửi tới tất cả các client để chúng cập nhật cấu hình mới mà không phải dừng hệ thống.
- Bạn có thể kích hoạt tính năng này bằng cách thêm dependency cho Spring Cloud Bus:
  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
  </dependency>
  ```

#### 8. **Bảo mật trong Spring Cloud Config**
- **Spring Cloud Config** cho phép mã hóa các thông tin nhạy cảm trong tệp cấu hình như mật khẩu, khóa API, bằng cách sử dụng cơ chế mã hóa tích hợp.
- Bạn có thể mã hóa một giá trị bằng cách sử dụng lệnh mã hóa:
  ```bash
  curl http://localhost:8888/encrypt -d mySecret
  ```
- Giá trị đã mã hóa có thể được đặt trong tệp cấu hình dưới dạng:
  ```yaml
  password: {cipher}encrypted_value
  ```
- **Config Server** sẽ tự động giải mã các giá trị khi cung cấp cho client.

#### 9. **Khả năng mở rộng của Config Server**
- **Config Server** có thể hỗ trợ nhiều kho cấu hình khác nhau cùng một lúc, như **Git**, **SVN**, **database**, hoặc **local file system**.
- Điều này giúp cho việc quản lý cấu hình trở nên linh hoạt hơn, đặc biệt là trong các hệ thống phân tán lớn.

#### 10. **Config Server trong môi trường thực tế**
- Trong môi trường **production**, **Config Server** thường được triển khai với nhiều instance để đảm bảo **high availability**.
- Các instance này có thể được **load-balanced** để giảm tải và đảm bảo tính ổn định cho toàn bộ hệ thống.

### Kết luận
**Spring Cloud Config Server** là một công cụ mạnh mẽ giúp quản lý và phân phối cấu hình một cách tập trung cho các dịch vụ trong hệ thống **microservices**. Với khả năng hỗ trợ **cấu hình động**, **profile-based configuration**, và **bảo mật**, Config Server giúp đơn giản hóa và tối ưu hóa quy trình quản lý cấu hình trong môi trường phân tán.
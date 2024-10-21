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
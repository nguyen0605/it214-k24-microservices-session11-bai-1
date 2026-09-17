# BÀI TẬP 1: KHỞI TẠO NỀN TẢNG NON-BLOCKING VÀ KẾT NỐI KAFKA CLUSTER

## 1. Tổng quan giải pháp
Dự án `order-service` được chuyển đổi sang kiến trúc Phản ứng (Reactive Architecture) sử dụng Spring WebFlux dựa trên máy chủ Embedded Netty và kết nối Apache Kafka làm Event Broker.

## 2. Giải quyết các Edge Cases & Tiêu chí nghiệm thu

### BUG-01: Khắc phục xung đột Tomcat / Netty
- **Nguyên nhân:** Thư viện `spring-boot-starter-web` sử dụng Apache Tomcat làm Servlet Container mặc định (Blocking I/O).
- **Khắc phục:** Loại bỏ hoàn toàn `spring-boot-starter-web` khỏi `pom.xml` và chỉ sử dụng `spring-boot-starter-webflux`. Đồng thời đặt `spring.main.web-application-type: reactive` trong `application.yml`. Khi ứng dụng khởi động, máy chủ Netty sẽ được kích hoạt trên cổng `8080`.

### BUG-02: Cấu hình Serializer cho Kafka Value
- **Khắc phục:** Sử dụng `org.springframework.kafka.support.serializer.JsonSerializer` làm Value Serializer thay vì `StringSerializer`. Cấu hình được thiết lập đồng bộ cả ở `application.yml` và trong `KafkaProducerConfig.java` để đảm bảo các đối tượng Java Object được tuần tự hóa thành chuỗi JSON chuẩn khi gửi qua Kafka.

## 3. Cấu hình WebClient với Timeout
- Tạo Bean `WebClient` dùng chung trong `WebClientConfig.java` sử dụng Reactor Netty `HttpClient` với timeout 5 giây (Connect timeout, Response timeout, Read/Write timeout) nhằm tránh hiện tượng treo luồng khi dịch vụ phụ thuộc phản hồi chậm.

# Amazon VPC (Virtual Private Cloud): Kiến trúc, Vai trò và Ứng dụng trong Hệ sinh thái AWS

---

## 1. Tổng quan về Amazon VPC

### 1.1 Định nghĩa

- **Amazon VPC là gì:** Amazon Virtual Private Cloud (VPC) là một dịch vụ mạng cốt lõi của Amazon Web Services (AWS), cho phép người dùng khởi tạo một vùng mạng tách biệt về mặt logic (logically isolated section) trên hạ tầng AWS, nơi họ có thể triển khai các tài nguyên đám mây trong một mạng ảo do chính họ định nghĩa.
- **Khái niệm "mạng riêng ảo" trong Cloud Computing:** Trong bối cảnh điện toán đám mây công cộng (Public Cloud), "mạng riêng ảo" là một phân vùng không gian địa chỉ IP được giới hạn quyền truy cập, tách biệt hoàn toàn với các khách hàng khác (multi-tenant isolation) thông qua công nghệ ảo hóa mạng.
- **Vị trí của VPC trong kiến trúc AWS:** VPC nằm ở lớp Hạ tầng (Infrastructure Layer), đóng vai trò là lớp vỏ bọc mạng (Network Boundary) cho phần lớn các dịch vụ tính toán và lưu trữ của AWS.

### 1.2 Mục tiêu thiết kế

- **Isolation (Cô lập tài nguyên):** Đảm bảo dữ liệu và tài nguyên của các tenant (khách hàng) khác nhau không thể can thiệp lẫn nhau.
- **Security (Bảo mật):** Cung cấp các công cụ kiểm soát luồng dữ liệu nghiêm ngặt ở cả cấp độ mạng con và cấp độ máy chủ.
- **Scalability (Khả năng mở rộng):** Hỗ trợ khả năng mở rộng không gian IP linh hoạt mà không bị giới hạn bởi hạ tầng phần cứng vật lý.
- **Availability (Tính sẵn sàng):** Được thiết kế trải rộng trên nhiều Availability Zones (Khu vực sẵn sàng) để đảm bảo hệ thống không có điểm mù chịu lỗi đơn lẻ (Single Point of Failure).

### 1.3 Vai trò trong hệ sinh thái AWS

- **VPC là lớp Network Foundation của AWS:** Hầu hết các dịch vụ hiện đại của AWS đều yêu cầu một nền tảng mạng để vận hành, và VPC chính là nền tảng đó.
- **Mối liên hệ giữa VPC với các dịch vụ:**
  - **EC2 (Elastic Compute Cloud):** Máy chủ ảo bắt buộc phải chạy bên trong một Subnet của VPC.
  - **RDS (Relational Database Service):** Đặt trong các Private Subnet để bảo vệ cơ sở dữ liệu khỏi Internet.
  - **ECS/EKS:** Các container và pod được cấp phát địa chỉ IP trực tiếp từ dải mạng của VPC.
  - **Lambda:** Chạy trong VPC để truy cập an toàn các tài nguyên nội bộ (như RDS, ElastiCache).
  - **Load Balancer (ELB):** Phân phối traffic vào các máy chủ nằm ở các Subnet/AZ khác nhau trong VPC.
  - **S3 & Route 53:** S3 tích hợp qua VPC Endpoint; Route 53 quản lý phân giải tên miền nội bộ (Private Hosted Zones) cho VPC.

### 1.4 Use Case thực tế

- **Web Application:** Phân phối các ứng dụng web với lượng truy cập lớn qua Internet một cách an toàn.
- **Microservices:** Xây dựng hệ thống phân tán với sự cô lập mạng lưới giữa các service.
- **Hybrid Cloud:** Mở rộng trung tâm dữ liệu nội bộ (On-premise) lên AWS.
- **Enterprise Infrastructure:** Xây dựng hệ thống mạng chuẩn doanh nghiệp với các quy tắc tuân thủ bảo mật khắt khe.
- **Multi-tier Architecture:** Kiến trúc n-tier (Web-App-DB) kinh điển.

---

## 2. Kiến trúc mạng trong AWS

### 2.1 Khái niệm Software-Defined Networking (SDN)

- **Định nghĩa SDN:** Mạng định nghĩa bằng phần mềm (SDN) là một kiến trúc mạng tách biệt Control Plane (mặt phẳng điều khiển) khỏi Data Plane (mặt phẳng dữ liệu), cho phép quản lý mạng tập trung và có thể lập trình được.
- **AWS triển khai SDN như thế nào:** AWS sử dụng hệ thống **AWS Nitro System** và thẻ mạng **Elastic Network Adapter (ENA)**. Các quy tắc định tuyến, bảo mật được xử lý tại lớp Hypervisor bằng phần mềm, thay vì dựa vào các switch/router vật lý truyền thống.
- **Vai trò của Virtual Networking:** Cung cấp tính linh hoạt (Agility), cho phép người dùng khởi tạo mạng lưới phức tạp chỉ bằng vài API calls thay vì mất hàng tháng để nối cáp vật lý.

### 2.2 Kiến trúc mạng truyền thống vs Cloud Networking

| Tiêu chí         | Traditional Network (Mạng truyền thống) | AWS VPC (Cloud Networking)           |
| ---------------- | --------------------------------------- | ------------------------------------ |
| **Hardware**     | Physical (Router, Switch, Cáp mạng)     | Virtualized (SDN, API-driven)        |
| **Scaling**      | Thủ công (Mua thêm thiết bị, lắp đặt)   | Tự động (Chỉnh sửa qua console/code) |
| **Provisioning** | Chậm (Vài tuần đến vài tháng)           | Nhanh (Vài giây đến vài phút)        |
| **Security**     | Firewall vật lý trung tâm (Appliance)   | Security Group / NACL phân tán       |

### 2.3 Shared Responsibility Model trong Networking

- **AWS chịu trách nhiệm gì:** "Security OF the Cloud". AWS bảo vệ phần cứng, cáp quang, hệ thống định tuyến lõi, chống DDoS ở lớp hạ tầng toàn cầu, và đảm bảo sự cô lập logic giữa các VPC.
- **Người dùng chịu trách nhiệm gì:** "Security IN the Cloud". Người dùng phải cấu hình đúng dải IP, Route Table, quy tắc Security Group/NACL, quản lý VPN và mã hóa dữ liệu truyền tải.

---

## 3. Cấu trúc tổng thể của Amazon VPC

### 3.1 CIDR Block

- **Định nghĩa:** Classless Inter-Domain Routing (CIDR) là phương pháp cấp phát địa chỉ IP và định tuyến IP. Một CIDR Block xác định kích thước mạng của VPC (VD: `10.0.0.0/16`).
- **Nguyên lý hoạt động:** `/16` đại diện cho số bit cố định của network, để lại 16 bit cho host (tương đương 65,536 IP).
- **IPv4 và IPv6:** VPC hỗ trợ cả IPv4 (tối đa /16) và IPv6 (cấp phát dải /56 cố định từ AWS). Có thể chạy ở chế độ Dual-stack.
- **Thiết kế địa chỉ mạng:** Cần tuân thủ RFC 1918 (Private IPv4) để tránh xung đột mạng.
- **Vai trò:**
  - Phân vùng mạng logic toàn cục cho VPC.
  - Tránh xung đột IP khi kết nối Peering hoặc VPN.
  - Hỗ trợ mở rộng hạ tầng (Có thể thêm Secondary CIDR).
- **Ưu điểm:** Cấp phát linh hoạt, tính tiêu chuẩn cao.
- **Nhược điểm:** Không thể thay đổi Primary CIDR Block sau khi đã khởi tạo VPC. Không thể chọn CIDR quá lớn (lớn hơn /16).

### 3.2 Subnet

- **Định nghĩa:** Sự phân mảnh logic một VPC thành các khối mạng nhỏ hơn, nằm gọn trong 1 Availability Zone (AZ). AWS bảo lưu 5 IP đầu/cuối trong mỗi subnet.
  - **Public Subnet:** Có tuyến đường (route) dẫn thẳng ra Internet Gateway (IGW).
  - **Private Subnet:** Không có route ra IGW.
- **Chức năng:** Phân tách tầng ứng dụng, tăng tính bảo mật, kiểm soát chi tiết luồng traffic ở mức nhỏ.
- **Mối liên kết:** Gắn liền với 1 Route Table, chứa các tài nguyên (EC2, RDS) và định tuyến qua NAT/IGW.
- **Kiến trúc Multi-tier:**
  - **Web Tier:** Ở Public Subnet (Load Balancer, Bastion Host).
  - **Application Tier:** Ở Private Subnet (Backend EC2, Container).
  - **Database Tier:** Ở Private Subnet sâu hơn (RDS, Aurora, không thể ra Internet).
- **Ưu điểm:** Kiểm soát luồng mạng độc lập theo vùng, giới hạn vùng ảnh hưởng khi có sự cố.
- **Nhược điểm:** Subnet bị ràng buộc vào 1 AZ duy nhất, nếu AZ sập, tài nguyên trong subnet cũng ngưng hoạt động.

### 3.3 Route Table

- **Định nghĩa:** Bảng định tuyến là tập hợp các quy tắc (routes) quyết định traffic từ Subnet/VPC sẽ đi đến đâu (Destination - Target).
- **Cách hoạt động:** Dựa trên Longest Prefix Match (Luật khớp dài nhất). Ví dụ: Route `/24` sẽ được ưu tiên hơn route `/16`.
- **Routing Decision Process:** Đánh giá IP đích -> So sánh với bảng định tuyến -> Đẩy traffic tới Target tương ứng. Mặc định luôn có Local Route để các Subnet trong cùng VPC giao tiếp với nhau.
- **Mối liên hệ:** Gắn với Subnet. Target có thể là IGW, NAT Gateway, Transit Gateway, hoặc Virtual Private Gateway (VPN).
- **Vai trò:** "Bộ não" điều hướng traffic của mạng, kiểm soát tính chất Public/Private của subnet.
- **Ưu điểm:** Cấu hình linh hoạt, áp dụng ngay lập tức.
- **Hạn chế:** Giới hạn số lượng route (Mặc định 50 routes/table), có thể gây phức tạp trong quản trị nếu có quá nhiều kết nối ngoại vi.

---

## 4. Internet Connectivity trong VPC

### 4.1 Internet Gateway (IGW)

- **Định nghĩa:** Một thành phần mạng mở rộng theo chiều ngang (horizontally scaled), có tính sẵn sàng cao, kết nối VPC với Internet.
- **Kiến trúc hoạt động:** Hoạt động như một router logic lưu trữ bảng ánh xạ (mapping) giữa Private IP và Public IP/Elastic IP của EC2.
- **Luồng traffic:** Xử lý cả traffic đi vào (inbound) và đi ra (outbound) từ Internet.
- **Vai trò:** Cho phép Public Access cho dịch vụ web, cập nhật phần mềm từ Internet.
- **Mối liên hệ:** Gắn vào VPC -> Cấu hình Route Table của Public Subnet chỉ `/0` về IGW -> Map Elastic IP cho EC2.
- **Ưu điểm:** Không tính phí duy trì, không bị nghẽn cổ chai (No bandwidth constraint).
- **Nhược điểm:** Phơi bày trực tiếp (expose) tài nguyên ra public internet, yêu cầu Security Group phải cực kỳ chặt chẽ.

### 4.2 NAT Gateway

- **Định nghĩa:** Dịch vụ Managed NAT (Network Address Translation) của AWS.
- **Nguyên lý NAT:** Dịch Private IP của các EC2 trong Private Subnet thành Public IP của NAT Gateway khi ra Internet, và ghi nhớ luồng để trả kết quả về đúng máy chủ ban đầu.
- **Mục tiêu sử dụng:** Cho phép outbound internet từ Private Subnet (để tải patch, call public API) nhưng chặn hoàn toàn kết nối khởi tạo từ bên ngoài vào.
- **Mối liên hệ:** Đặt tại **Public Subnet**, gắn với Elastic IP, và được trỏ tới bởi Route Table của **Private Subnet**.

#### So sánh NAT Gateway vs NAT Instance

| Tiêu chí            | NAT Gateway                           | NAT Instance (EC2 tự build NAT)          |
| ------------------- | ------------------------------------- | ---------------------------------------- |
| **Managed Service** | Có (AWS tự quản lý)                   | Không (Người dùng tự duy trì OS, Vá lỗi) |
| **Scalability**     | Tự động mở rộng đến 100 Gbps          | Thủ công (Phải đổi size EC2)             |
| **Availability**    | Rất cao (Tích hợp HA trong 1 AZ)      | Phụ thuộc vào vòng đời của 1 máy chủ EC2 |
| **Cost**            | Cao hơn (Phí giờ + Phí Data Transfer) | Thấp hơn (Chỉ tính phí chạy EC2)         |

- **Ưu điểm:** Không tốn công quản trị (Zero-maintenance), bảo mật cực cao, hiệu năng xuất sắc.
- **Nhược điểm:** Phí duy trì và phí xử lý dữ liệu qua NAT (Data Processing Charge) khá đắt đỏ. Đòi hỏi 1 NAT GW mỗi AZ để đạt HA cao nhất.

---

## 5. Security Architecture trong Amazon VPC

### 5.1 Security Group (SG)

- **Định nghĩa:** Là tường lửa ảo cấp độ máy chủ (Instance-level virtual firewall).
- **Stateful Firewall (Tường lửa trạng thái):** Ghi nhớ trạng thái kết nối. Nếu bạn cho phép Request đi ra (Outbound), Response đi vào sẽ tự động được cho phép bất kể luật Inbound là gì.
- **Cơ chế hoạt động:**
  - Mặc định: Chặn mọi Inbound, Cho phép mọi Outbound.
  - Chỉ hỗ trợ quy tắc **ALLOW** (Không có DENY).
- **Vai trò:** Bảo vệ lớp ứng dụng, chỉ cho phép đúng IP/Port thiết yếu.
- **Mối liên kết:** Gắn trực tiếp lên Card mạng (ENI) của EC2, RDS, ELB, ECS...
- **Ưu điểm:** Tính linh hoạt cực cao, có thể thiết lập Source là một Security Group khác (rất hữu ích trong kiến trúc Multi-tier).
- **Hạn chế:** Không thể chặn một IP cụ thể (Blacklisting), giới hạn số lượng rule mỗi group.

### 5.2 Network ACL (NACL)

- **Định nghĩa:** Danh sách kiểm soát truy cập ở cấp độ mạng con (Subnet-level firewall).
- **Stateless Firewall (Tường lửa phi trạng thái):** Không ghi nhớ kết nối. Mọi traffic đi vào hay đi ra đều phải được xác minh lại bằng cả rule Inbound và Outbound.
- **Cơ chế hoạt động:**
  - Hỗ trợ cả luật **ALLOW** và **DENY**.
  - Đánh giá tuần tự theo số Rule Number (từ nhỏ đến lớn). Chạm rule nào thỏa mãn là dừng lại (Rule Evaluation).
- **Vai trò:** Đóng vai trò như lớp phòng thủ thứ hai, chuyên dùng để chặn (block) các IP độc hại hoặc chống DDoS ở cấp mạng lưới.
- **Mối liên hệ:** Gắn vào Subnet. Mọi traffic ra/vào Subnet đều bị kiểm soát.
- **Ưu điểm:** Cho phép Blacklist IP, bảo vệ cả một dải Subnet cùng lúc.
- **Nhược điểm:** Khó cấu hình, phải khai báo kỹ các dải cổng tạm thời (Ephemeral Ports), rủi ro chặn nhầm traffic rất cao.

### 5.3 So sánh Security Group và NACL

| Tiêu chí            | Security Group                   | NACL                                                 |
| ------------------- | -------------------------------- | ---------------------------------------------------- |
| **Level**           | Lớp máy chủ (Instance / ENI)     | Lớp mạng con (Subnet)                                |
| **Stateful**        | Có (Stateful)                    | Không (Stateless)                                    |
| **Rule Type**       | Chỉ Allow (Allow only)           | Cả Allow & Deny                                      |
| **Rule Evaluation** | Đánh giá đồng thời toàn bộ Rules | Đánh giá tuần tự theo số thứ tự (Rule #)             |
| **Complexity**      | Thấp, dễ cấu hình                | Cao, dễ xảy ra lỗi nếu không am hiểu Ephemeral Ports |

---

## 6. High Availability (HA) & Scalability

### 6.1 Availability Zone (AZ)

- **Định nghĩa:** AZ là một hoặc nhiều trung tâm dữ liệu vật lý riêng biệt với nguồn điện, mạng và làm mát độc lập trong một AWS Region.
- **Vai trò trong HA:** Ngăn chặn sự cố cơ sở hạ tầng vật lý làm sập toàn bộ hệ thống.
- **Mối liên hệ:** Subnet bị ràng buộc với 1 AZ. Để tận dụng HA, VPC phải trải dài tài nguyên trên tối thiểu 2 AZ.

### 6.2 Multi-AZ Architecture

- **Mô hình triển khai:** Subnet A (AZ-a) và Subnet B (AZ-b). ELB phân phối traffic tới các EC2 ở cả hai.
- **Fault Tolerance:** Nếu AZ-a mất điện, ELB lập tức điều hướng 100% traffic sang AZ-b.
- **Disaster Recovery:** RDS có tính năng Multi-AZ tự động đồng bộ dữ liệu (synchronous replication) sang AZ phụ.
- **Ưu điểm:** Tính chống chịu lỗi ở cấp độ trung tâm dữ liệu, thời gian Uptime lên đến 99.99%.
- **Nhược điểm:** Phí truyền dữ liệu (Data Transfer Cross-AZ) tăng chi phí vận hành.

---

## 7. Kết nối giữa các mạng

### 7.1 VPC Peering

- **Định nghĩa:** Kết nối mạng logic giữa 2 VPC sử dụng IP nội bộ.
- **Kiến trúc hoạt động:** Trao đổi dữ liệu qua hệ thống cáp quang backbone của AWS, không đi qua public internet.
- **Use Cases:** Microservices nằm ở 2 VPC khác nhau, chia sẻ chung hệ thống Active Directory (Multi-account).
- **Hạn chế:** Non-transitive Routing (VPC A nối B, B nối C -> A không thể nói chuyện với C). Scaling kém khi có hàng trăm VPC (Full-mesh complexity).

### 7.2 Transit Gateway (TGW)

- **Định nghĩa:** Dịch vụ định tuyến lõi đám mây (Cloud Router).
- **Hub-and-Spoke Architecture:** TGW đóng vai trò là Hub trung tâm, các VPC và VPN cắm vào như các nan hoa (Spoke).
- **Vai trò:** Quản lý định tuyến tập trung, giải quyết bài toán Non-transitive của Peering.
- **Mối liên kết:** Nối hàng ngàn VPC, Direct Connect, VPN.
- **Ưu điểm:** Đơn giản hóa kiến trúc mạng cực kỳ phức tạp.
- **Nhược điểm:** Chi phí xử lý dữ liệu qua TGW cao hơn so với VPC Peering.

### 7.3 VPN Connection

- **Site-to-Site VPN:** Kết nối IPsec mã hóa giữa Datacenter nội bộ (Customer Gateway) và AWS VPC (Virtual Private Gateway / Transit Gateway).
- **Client VPN:** Cho phép nhân viên làm việc từ xa kết nối an toàn vào VPC bằng OpenVPN client.
- **Vai trò:** Cầu nối cốt lõi trong kiến trúc Hybrid Cloud.

### 7.4 AWS Direct Connect (DX)

- **Định nghĩa:** Đường truyền cáp quang vật lý chuyên dụng (Dedicated physical connection) nối thẳng từ mạng của khách hàng tới AWS.
- **Use Cases:** Yêu cầu bảo mật tuyệt đối, độ trễ cực thấp (Low Latency), băng thông ổn định (High Throughput).
- **So sánh VPN vs DX:**
  - **VPN:** Nhanh chóng thiết lập, truyền qua Internet (băng thông biến động), chi phí thấp.
  - **Direct Connect:** Thiết lập mất nhiều tuần (lắp cáp vật lý), Không qua Internet (độ trễ ổn định 100%), chi phí CAPEX/OPEX cao.

---

## 8. Private Connectivity

### 8.1 VPC Endpoint

- **Định nghĩa:** Công nghệ kết nối các EC2 trong VPC với các dịch vụ AWS bên ngoài VPC (như S3, DynamoDB, SNS) bằng IP nội bộ.
- **Mục tiêu thiết kế:** Ngăn chặn việc truyền dữ liệu nhạy cảm ra Internet công cộng.
- **Gateway Endpoint:** Dành riêng cho S3 và DynamoDB. Hoạt động bằng cách chèn 1 Route vào Route Table. Cấu hình miễn phí.
- **Interface Endpoint (AWS PrivateLink):** Dành cho các dịch vụ còn lại (SQS, SNS, Kinesis...). Khởi tạo một card mạng ảo (ENI) với Private IP trực tiếp trong Subnet của người dùng.
- **Vai trò trong bảo mật:** Không đi qua Internet Gateway, Giảm Attack Surface (bề mặt tấn công), chống đánh cắp dữ liệu.
- **Ưu điểm:** An toàn tuyệt đối, hiệu năng cao.
- **Nhược điểm:** Interface Endpoint có tính phí duy trì theo giờ và phí truyền tải.

---

## 9. Monitoring & Observability

### 9.1 VPC Flow Logs

- **Định nghĩa:** Tính năng thu thập metadata (siêu dữ liệu) của dòng traffic IP đi qua các ENI trong VPC.
- **Kiến trúc logging:** Log Src IP, Dest IP, Port, Protocol, Bytes, Packet, Action (ACCEPT/REJECT).
- **Vai trò:**
  - **Monitoring:** Theo dõi băng thông.
  - **Security Analysis:** Phát hiện botnet quét port (dựa trên tỷ lệ REJECT cao).
  - **Troubleshooting:** Phân tích lý do vì sao EC2 không kết nối được DB (kiểm tra SG/NACL).
- **Mối liên hệ:** Xuất logs ra CloudWatch Logs (để xem real-time) hoặc Amazon S3 (để lưu trữ rẻ) -> Dùng Athena để truy vấn bằng SQL.

### 9.2 CloudWatch Integration

Tích hợp VPC Flowlogs, đo lường băng thông NAT Gateway, số lượng kết nối VPN bị rớt, tạo Alarms cảnh báo tự động.

### 9.3 CloudTrail Integration

Ghi lại mọi lệnh gọi API liên quan đến cấu hình mạng (Ví dụ: Ai đã xóa Security Group? Ai đã thay đổi Route Table vào lúc mấy giờ?). Phục vụ cho Compliance (Tuân thủ kiểm toán).

---

## 10. Tích hợp VPC với các AWS Services

### 10.1 Amazon EC2

- EC2 được khởi tạo (Placement) vào 1 Subnet xác định.
- Kế thừa CIDR của subnet và áp dụng Security Group để vận hành Firewall.

### 10.2 Amazon RDS

- Sử dụng **DB Subnet Group** (nhóm các subnet nằm trên tối thiểu 2 AZ) để triển khai HA.
- Kiến trúc Private Database luôn không cấp Public IP cho RDS.

### 10.3 Elastic Load Balancer (ELB)

- **Public/Internet-facing ALB:** Đặt tại Public Subnet, nhận traffic từ Internet, đẩy xuống Backend ở Private Subnet.
- **Internal ALB:** Đặt ở Private Subnet, dùng để giao tiếp nội bộ giữa tầng Web và tầng App (Microservices).

### 10.4 Amazon ECS / EKS

- Sử dụng **awsvpc network mode** (ECS) hoặc **Amazon VPC CNI plugin** (EKS) để cấp mỗi container/pod một địa chỉ IP nội bộ thật sự từ dải mạng VPC (Native Container Networking).

### 10.5 AWS Lambda

- Mặc định chạy ngoài VPC. Khi gắn vào VPC (VPC-enabled Lambda), AWS sẽ tạo một ENI (Hyperplane ENI) trong Subnet của người dùng để Lambda có thể query vào RDS nội bộ một cách an toàn.

---

## 11. Thiết kế kiến trúc thực tế

### 11.1 Three-Tier Architecture

- **Web Tier (Tầng trình diễn):** Public Subnet chứa ALB, Bastion Host. Chỉ nhận kết nối HTTP/HTTPS (Port 80/443).
- **Application Tier (Tầng logic):** Private Subnet chứa EC2 Auto Scaling Group. Security Group chỉ cho phép nhận traffic từ SG của ALB.
- **Database Tier (Tầng dữ liệu):** Private Subnet chứa RDS/Aurora. SG chỉ nhận cổng 3306/5432 từ SG của tầng Application.
- **Luồng hoạt động:** User -> Internet -> IGW -> ALB -> EC2 App -> RDS.

### 11.2 Microservices Architecture

- **Service Isolation:** Chia nhỏ mạng thành các Subnet/VPC riêng cho từng domain nghiệp vụ.
- **Internal Communication:** Giao tiếp qua Internal Load Balancers, API Gateway (Private) hoặc AWS Cloud Map (Service Discovery).
- **Security Segmentation:** Dùng Security Group khóa chặt traffic, Service A chỉ được gọi Service B, không được gọi Service C.

### 11.3 Hybrid Cloud Architecture

- **On-Premise + AWS:** Kết nối văn phòng chính với AWS qua AWS Direct Connect để làm đường truyền chính, cấu hình Site-to-Site VPN làm đường truyền dự phòng (Failover).

---

## 12. Best Practices trong thiết kế VPC

### 12.1 Security Best Practices

- **Least Privilege:** SG chỉ mở đúng cổng cần thiết, đúng IP cần thiết. Không dùng `0.0.0.0/0` bừa bãi.
- **Private-by-default:** Mặc định đưa mọi thứ vào Private Subnet, chỉ những gì thật sự cần ra internet (LB, Bastion) mới đặt ở Public.
- **Segmentation:** Sử dụng NACL như một bức tường lửa lớp ngoài để chặn các dải IP từ các quốc gia bị cấm.

### 12.2 Scalability Best Practices

- **CIDR Planning:** Không thiết kế VPC CIDR quá chật. Để dành lại 1 số Subnet / AZ dự phòng chưa sử dụng để mở rộng trong tương lai.
- **Multi-AZ Deployment:** Phân bổ tài nguyên đồng đều giữa ít nhất 2 đến 3 AZ.

### 12.3 Cost Optimization

- **NAT Gateway Optimization:** Giao tiếp S3/DynamoDB qua VPC Endpoint (miễn phí/rẻ) thay vì dùng qua NAT Gateway (chi phí Data transfer cực cao).
- **Traffic Control:** Cố gắng giữ luồng dữ liệu (Data Traffic) ở bên trong cùng một AZ để tránh phí Cross-AZ Data Transfer.

---

## 13. Ưu điểm và hạn chế của Amazon VPC

### 13.1 Ưu điểm

- **Isolation (Cô lập tuyệt đối):** An toàn cao nhất trên môi trường Public Cloud.
- **Elastic Scalability:** Mở rộng linh hoạt mà không cần đổi phần cứng.
- **Fine-grained Security:** Kiểm soát bảo mật chi tiết qua 2 lớp (NACL và SG).
- **Hybrid Connectivity:** Kết nối dễ dàng với hạ tầng On-premise.
- **High Availability:** Triển khai Multi-AZ với tính chống chịu thảm họa tốt.

### 13.2 Hạn chế

- **Complexity (Độ phức tạp):** Yêu cầu kiến thức cực kỳ vững chắc về TCP/IP, Routing, Subnetting.
- **Cost (Chi phí tiềm ẩn):** Phí Data transfer, phí duy trì NAT, Endpoint, TGW có thể vượt ngoài kiểm soát nếu thiết kế sai.
- **Routing Complexity:** Khi scale lên hàng trăm VPC, việc quản lý IP Overlap và Routing rules vô cùng khó khăn.
- **Operational Overhead:** Cần nhân sự mạng đám mây (Cloud Network Engineer) để duy trì cấu trúc.

---

## 14. So sánh Amazon VPC với mô hình truyền thống

| Tiêu chí         | Traditional Data Center (TT Dữ liệu truyền thống)     | Amazon VPC                                                     |
| ---------------- | ----------------------------------------------------- | -------------------------------------------------------------- |
| **Deployment**   | Manual (Lắp đặt cáp vật lý, config CLI từng thiết bị) | Automated (Infrastructure as Code - Terraform, CloudFormation) |
| **Scalability**  | Limited (Bị giới hạn bởi cổng Switch, Router)         | Elastic (Thay đổi cấp phát IP linh hoạt vô hạn)                |
| **Security**     | Physical Appliance (Tường lửa mạng chu vi)            | Software-defined (Tường lửa bao bọc quanh mỗi máy chủ)         |
| **Cost**         | CapEx (Chi phí đầu tư mua đứt thiết bị)               | OpEx (Chi trả theo giờ dùng NAT, VPN, Data)                    |
| **Availability** | Phụ thuộc hardware (Phải mua thiết bị Active-Standby) | Tích hợp Multi-AZ bẩm sinh, HA phần mềm                        |

---

## 15. Tổng kết

### 15.1 Vai trò chiến lược của Amazon VPC

VPC không chỉ là một dịch vụ phụ trợ, mà là **nền tảng networking cốt lõi** không thể thiếu của toàn bộ hệ sinh thái AWS. Nó là "trung tâm kết nối" giúp các dịch vụ rời rạc hợp nhất thành một hệ thống thông tin hoàn chỉnh.

### 15.2 Đóng góp trong kiến trúc Cloud tổng thể

- **Security Foundation:** Cung cấp hàng rào vật lý-logic đầu tiên chống lại các nguy cơ bảo mật mạng.
- **Connectivity Layer:** Là lớp kết nối liên kết Internet, Intranet, và Hybrid Cloud.
- **Isolation Layer:** Đảm bảo môi trường Sandbox cho việc thử nghiệm cũng như môi trường Production khắc nghiệt.

### 15.3 Tầm quan trọng trong Cloud Architecture

Đối với kiến trúc sư đám mây (Cloud Architect), việc thấu hiểu và thiết kế chuẩn xác hạ tầng VPC từ đầu là chìa khóa quyết định sự thành bại của dự án Enterprise, quyết định khả năng chống chịu thảm họa của Distributed Systems và mở đường cho các kiến trúc vĩ mô như Hybrid Cloud trong tương lai.

---

## 16. Tài liệu tham khảo

**AWS Official Documentation:**

- _Amazon VPC User Guide_ - AWS.
- _AWS Networking Fundamentals_ - AWS Whitepapers.
- _AWS Well-Architected Framework: Reliability and Security Pillar_ - AWS.

**Sách / Sách chuyên ngành:**

- _AWS Certified Advanced Networking - Specialty Exam Guide_.
- _Cloud Networking Fundamentals: Simplifying the network for modern applications_.

**Whitepapers & Hướng dẫn kỹ thuật:**

- _AWS Security Best Practices_ (Bảo mật chuyên sâu trên AWS).
- _Hybrid Connectivity Whitepaper_ (Xây dựng đám mây lai với DX và VPN).
- _Building Scalable VPC Architectures_ (Thiết kế kiến trúc VPC mở rộng quy mô lớn).

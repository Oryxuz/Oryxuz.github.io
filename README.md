# **4 Domain trong đề thi SAA-C03:**   
Domain 1: Design Secure Architectures (30%) — Bảo mật, phân quyền, mã hóa.   
Domain 2: Design Resilient Architectures (26%) — Chịu lỗi, khả năng phục hồi (HA/DR).   
Domain 3: Design High-Performing Architectures (24%) — Tối ưu hiệu năng, tốc độ xử lý.   
Domain 4: Design Cost-Optimized Architectures (20%) — Tối ưu chi phí, không lãng phí tài nguyên.

## 1. Pillar: Secure Architectures (Bảo mật - 30%)
Bức tranh này trả lời câu hỏi: Làm sao để hệ thống không bị chui lén, dữ liệu không bị lộ và mọi thao tác đều được kiểm soát?

- Lý thuyết cốt lõi:
    + Identity & Access (IAM): Ai (Principal) được làm gì (Action) trên tài nguyên nào (Resource)?
    + Network Isolation (VPC): Phân vùng mạng công cộng (Public) và mạng nội bộ (Private).
    + Data Protection (KMS/Secrets): Mã hóa dữ liệu khi truyền (In-transit) và khi lưu trữ (At-rest).

- Cách ghép Service:
    + Truy cập ứng dụng: User ➔ WAF (chặn SQLi, XSS) ➔ ALB trong Public Subnet ➔ EC2 nằm trong Private Subnet.
    + Truy cập dịch vụ AWS an toàn: EC2 trong Private Subnet muốn gọi S3/DynamoDB không qua Internet ➔ Ghép VPC Endpoints (Gateway/Interface).
    + Quản lý mật khẩu/Khóa: Ứng dụng cần connection string vào DB ➔ Luân chuyển tự động bằng AWS Secrets Manager ➔ Mã hóa bằng KMS.
## 2. Pillar: Resilient Architectures (Chịu lỗi & Phục hồi - 26%)
Bức tranh này trả lời câu hỏi: Nếu 1 con server sập, 1 Data Center (AZ) bị cháy, hoặc cả 1 Region bị thiên tai, hệ thống làm sao sống tiếp?

- Lý thuyết cốt lõi:High Availability (HA):
    + Luôn có bản sao chạy song song ở nhiều AZ.
    + Decoupling (Bất đồng bộ): Tách rời các component để 1 cái sập không kéo theo toàn bộ hệ thống sập.
    + Disaster Recovery (DR): Backup và đồng bộ dữ liệu sang Region khác.

- Cách ghép Service:
    + Chịu lỗi lớp Compute: Route 53 (DNS Failover) ➔ ALB (Chia tải) ➔ Auto Scaling Group (ASG) rải EC2 đều ra Multi-AZ.
    + Chịu lỗi lớp Database: RDS Multi-AZ (Chạy 1 con Primary, 1 con Standby đồng bộ). Khi con chính sập, AWS tự động Failover sang con phụ.
    + Chịu lỗi bằng tin nhắn (Decoupling): Web Server nhận request ➔ Đẩy tin nhắn vào SQS Queue ➔ Worker đọc tin nhắn xử lý. Nếu Worker sập, tin nhắn vẫn nằm nguyên trong SQS, không bị mất request của khách.
    + Khôi phục sau thảm họa (Cross-Region DR): S3 Cross-Region Replication (CRR) hoặc Aurora Global Database (đồng bộ dữ liệu từ Singapore sang Tokyo với độ trễ < 1s).

## 3. Pillar: High-Performing Architectures (Hiệu năng cao - 24%)
Bức tranh này trả lời câu hỏi: Làm sao để hệ thống xử lý hàng triệu request với độ trễ (latency) thấp nhất?

- Lý thuyết cốt lõi:
    + Caching (Bộ nhớ tạm): Không bắt Database hay Server làm việc lại những việc đã làm.
    + Offloading: Chuyển các file tĩnh (ảnh, video) ra gần người dùng nhất.
    + Parallel Processing: Xử lý song song.

- Cách ghép Service:
    + Tăng tốc nội dung tĩnh & API: CloudFront (CDN) đứng trước S3/ALB để cache nội dung ở các Edge Location gần người dùng nhất.
    + Tăng tốc Database truy vấn nhiều: Đặt ElastiCache (Redis/Memcached) hoặc DynamoDB Accelerator (DAX) phía trước Database để lưu kết quả query hay dùng.
    + Tăng tốc lưu trữ tệp: Chọn ổ đĩa EBS Provisioned IOPS (io2) cho DB cần IOPS cực lớn, hoặc EFS cho nhiều server đọc/ghi đồng thời.

## 4. Pillar: Cost-Optimized Architectures (Tối ưu chi phí - 20%)
Bức tranh này trả lời câu hỏi: Làm sao để đạt được mục tiêu kinh doanh với số tiền AWS trả hằng tháng là nhỏ nhất?

- Lý thuyết cốt lõi:
    + Right-sizing: Chọn đúng kích thước tài nguyên, không mua thừa.
    + Storage Lifecycle: Chuyển dữ liệu ít dùng sang kho lưu trữ rẻ hơn.
    + Pricing Models: Kết hợp On-Demand, Reserved, Savings Plans và Spot Instances.

- Cách ghép Service:
    + Tối ưu Storage: Thiết kế S3 Lifecycle Rules: Dữ liệu mới lưu ở S3 Standard ➔ Sau 30 ngày chuyển sang S3 Standard-IA ➔ Sau 90 ngày đẩy vào S3 Glacier Flexible / Deep Archive (rẻ hơn hàng chục lần).
    + Tối ưu Compute: Dùng Spot Instances cho các task batch processing (chạy ẩn, chấp nhận ngắt đột ngột) để giảm tới 90% chi phí. Dùng Savings Plans / Reserved Instances cho các EC2/RDS chạy 24/7.
    + Tối ưu Architecture: Chuyển từ EC2 chạy liên tục sang kiến trúc Serverless (API Gateway + Lambda + DynamoDB) — chỉ trả tiền đúng theo mi-li-giây khi có request chạy.

---

# ** BỨC TRANH TỔNG QUAN: 13 CATEGORY CỐT LÕI CỦA AWS (SAA-C03)**
1. *Compute (Máy chủ, Serverless, Container)*
2. *Networking & Content Delivery (Mạng ảo, Điều hướng, CDN, Kết nối lai Hybrid)**
3. *Storage (Lưu trữ File, Object, Block, Hybrid)*
4. *Databases (Relational, NoSQL, In-Memory, Graph)*
5. *Security, Identity & Compliance (Định danh, Phân quyền, Mã hóa, Chống tấn công)*
6. *Application Integration & Messaging (Hàng chờ, Pub/Sub, Workflows)*
7. *Analytics & Big Data (Xử lý, Query, Streaming - Cầu nối sang DEA-C01)*
8. *Management, Governance & Monitoring (Ghi Log, Giám sát, Quản trị tài nguyên)*
9. *Migration & Transfer (Chuyển đổi dữ liệu và hệ thống lên Cloud)*
10. *Developer Tools & CI/CD (Quản lý code, Pipeline, Deploy)*
11. *Cost Management (Quản lý & Tối ưu chi phí)*
12. *Disaster Recovery & Business Continuity (Sao lưu, Khôi phục thảm họa)*
13. *Front-End Web & Mobile (API Gateway, Cognito - Phục vụ app Client)*

# **A. COMPUTE**
### 1. Amazon EC2 (Elastic Compute Cloud)
- **Khái niệm**: Máy chủ ảo (Virtual Machine - IaaS) chạy trên hạ tầng AWS.
- **Tác dụng**: Cung cấp tài nguyên tính toán có khả năng thay đổi quy mô linh hoạt.

- **Cách sử dụng**:
    + Chọn AMI (Amazon Machine Image: Linux, Windows, Ubuntu...).
    + Chọn Instance Type (Ví dụ: t3.micro, c5.large, r5.xlarge).
    + Cấu hình EBS Volume (ổ đĩa boot).
    + Cấu hình Security Group (Firewall cấp Instance).
    + Kết nối qua SSH (Linux) hoặc RDP (Windows) / EC2 Instance Connect / Systems Manager.

- **Phân loại Instance Types (Cần nhớ cho SAA)**:
    + General Purpose (t3, m5, m6g): Bằng phẳng giữa CPU & RAM (Web server, Small DB).
    + Compute Optimized (c5, c6g): CPU cao (Media encoding, High-performance web server, Gaming).
    + Memory Optimized (r5, x1): RAM cực lớn (In-memory DBs như Redis, High-performance Relational DB).
    + Storage Optimized (i3, d2): Sequential Read/Write cao (Data warehousing, Distributed file systems).

- **Phân loại Mua (Pricing Models - CỰC KỲ QUAN TRỌNG)**:
    + *On-Demand*: Trả tiền theo giây/giờ, không cam kết. Dùng cho: App mới, workload không đoán trước được.
    + *Reserved Instances (RI) / Savings Plans*: Cam kết 1 năm hoặc 3 năm. Giảm tới 72%. Dùng cho: App chạy liên tục 24/7 (Baseline workload).
    + *Spot Instances*: Đấu giá tài nguyên thừa của AWS. Giảm tới 90%. AWS có thể thu hồi bất kỳ lúc nào (báo trước 2 phút). Dùng cho: Batch jobs, CI/CD, Stateless workloads chấp nhận gián đoạn.
    + *Dedicated Hosts / Dedicated Instances*: Chạy trên phần cứng vật lý riêng biệt. Dùng cho: Tuân thủ pháp lý (Compliance) hoặc ứng dụng có license gắn theo CPU vật lý (BYOL).

- **Pros**: Toàn quyền kiểm soát (Full Root/Admin OS access), tương thích mọi phần mềm truyền thống.
- **Cons**: Tốn công vận hành (Phải tự OS patching, cài Antivirus, quản lý backup).
- **Nên dùng khi**: Đưa ứng dụng Monolith cũ lên Cloud, ứng dụng cần cấu hình OS sâu, workload chạy liên tục nặng.
- **KHÔNG nên dùng khi**: Ứng dụng nhỏ chạy theo sự kiện (nên dùng Lambda) hoặc muốn rảnh tay không quản server.

### 2. Auto Scaling Group (ASG)
- **Khái niệm**: Tập hợp các EC2 Instance được tự động tăng hoặc giảm số lượng dựa trên nhu cầu (Demand).
- **Tác dụng**: Đảm bảo tính Sẵn sàng cao (Availability) và Tối ưu chi phí (Cost).

- **Cách sử dụng**:
    + Tạo Launch Template (Định nghĩa EC2 chạy bằng AMI nào, Instance type gì, Security group nào).
    + Cấu hình Min, Max, Desired Capacity.
    + Đặt Scaling Policy (Dynamic, Target Tracking, Scheduled, Predictive).

- **Các cơ chế Scale**:
    + *Target Tracking Scaling*: Duy trì CPU trung bình ở mức 60%.
    + *Step/Simple Scaling*: Nếu CPU > 80% thì thêm 2 instance; nếu CPU < 20% thì bớt 1 instance.
    + *Scheduled Scaling*: Tăng lên 10 instance vào lúc 8 giờ sáng mỗi T2 (Khung giờ cao điểm).

- **Pros**: Tự động khôi phục Instance lỗi (Health Check Replacement), trải rộng EC2 ra nhiều AZ (High Availability).
- **Cons**: Cần thiết kế App dạng Stateless (không lưu Session trực tiếp trên EC2) thì Scale mới chuẩn.
- **Nên dùng khi**: Mọi ứng dụng EC2 chạy sau Load Balancer có lượng traffic biến động.
- **KHÔNG nên dùng khi**: Ứng dụng Stateful không thể nhân bản nhiều con.

### 3. AWS Lambda (Serverless Compute)
- **Khái niệm**: Dịch vụ thực thi code dạng Serverless (FaaS - Function as a Service).
- **Tác dụng**: Chạy code tự động khi có sự kiện (Event-driven) mà hoàn toàn không cần dựng server.

- **Cách sử dụng**: Viết hàm (Python, Node.js, Go, Java...) -> Đặt Trigger (S3, API Gateway, DynamoDB Streams, EventBridge) -> Lambda tự chạy.

- **Thông số kỹ thuật quan trọng**:
    + Thời gian chạy tối đa (Timeout): 15 phút.
    + RAM cấu hình: Từ 128MB đến 10,240MB (CPU scale tỉ lệ thuận theo RAM).
    + Dung lượng bộ nhớ tạm /tmp: Từ 512MB đến 10GB.

- **Pros**: Chi phí cực rẻ (Tính theo số lần gọi + thời gian chạy tính bằng millisecond), Tự động Scale từ 0 đến hàng ngàn request đồng thời.
- **Cons**:
    + Cold Start: Lần chạy đầu tiên bị trễ vài trăm ms (Khắc phục bằng Provisioned Concurrency).
    + Giới hạn 15 phút.

- **Nên dùng khi**: API backend (kết hợp API Gateway), xử lý dữ liệu realtime khi upload file lên S3, Cronjob tự động.
- **KHÔNG nên dùng khi**: Task tính toán phức tạp chạy liên tục nhiều giờ liền.

### 4. Elastic Container Service (ECS)
- **Khái niệm**: Dịch vụ điều phối Container (Container Orchestration) độc quyền của AWS.
- **Tác dụng**: Chạy và quản lý hàng loạt Docker Container.

- **Cấu trúc**:
    + Task Definition: File JSON khai báo Docker Image, CPU/RAM, Port, Environment variables.
    + Task: Bản thể đang chạy của Task Definition (tương đương 1 Container).
    + Service: Quản lý duy trì số lượng Task nhất định (tương tự ReplicaSet). 
    ```  
        ECS
    ├── Task
    │    └── Container
    └── Task
        └── Container
    ```
    + Nhưng container thực sự phải chạy trên một compute capacity nào đó:
    ```
                 ECS
              │
       ┌──────┴──────┐
       ↓             ↓
     EC2          Fargate
   (server)     (serverless)
    ```
- 2 Launch Types (Mô hình chạy):
    + *ECS EC2 Launch Type*: Bạn tự quản lý dàn máy chủ EC2 để làm Cluster chứa Container. (Rẻ hơn nếu tận dụng hết capacity).
    + *ECS Fargate Launch Type*: Serverless Container. Không quản lý EC2, chỉ khai báo CPU/RAM cho Task, AWS lo phần còn lại.

- **Pros**: Tích hợp cực sâu với IAM Role cho từng Task, Load Balancer (ALB), CloudWatch.
- **Cons**: Vendor lock-in (Khó chuyển sang Cloud khác so với Kubernetes).
- **Nên dùng khi**: Ứng dụng Microservices chạy Docker, muốn quản lý đơn giản không rườm rà.
- **KHÔNG nên dùng khi**: Đội ngũ bắt buộc dùng hệ sinh thái Kubernetes chuẩn mở.

### 5. Elastic Kubernetes Service (EKS)
- **Khái niệm**: Dịch vụ quản lý Kubernetes (K8s) chuẩn Cloud Native do AWS vận hành Control Plane.
- **Tác dụng**: Chạy ứng dụng Kubernetes mà không cần tự cài Control Plane (API Server, etcd).
- **Mô hình triển khai**: Control Plane do AWS quản lý 100% (High Availability qua nhiều AZ). Worker Nodes có thể là EC2 hoặc Fargate.
    ```
        EKS
    │
    ├── Kubernetes control plane
    │
    └── Worker compute
        ├── EC2
        └── Fargate
    ```
- **Pros**: Chuẩn mở Kubernetes, dễ dàng di chuyển ứng dụng (Portable) giữa On-premises và AWS.
- **Cons**: Chi phí cố định ~$0.10/giờ cho Control Plane, cấu hình phức tạp, đòi hỏi kiến thức K8s sâu.
- **Nên dùng khi**: Công ty đã và đang xài Kubernetes ở On-premise, muốn bê nguyên sang AWS.
- **KHÔNG nên dùng khi**: Đội ngũ chưa biết K8s và chỉ muốn chạy vài container đơn giản (Nên dùng ECS + Fargate).

### 6. AWS Fargate (Serverless Compute cho Containers)
- **Khái niệm**: Engine tính toán Serverless dùng chung cho cả ECS và EKS.
- **Tác dụng**: Loại bỏ hoàn toàn việc provision, scale và quản lý máy chủ EC2 khi chạy Container.
- **Cách sử dụng**: Trong ECS/EKS, chọn Launch Type = FARGATE. Chỉ định bao nhiêu vCPU và GB RAM cho Container.
- **Pros**: Không lo OS patching, cô lập bảo mật cấp VM cho từng Task, chỉ trả tiền đúng tài nguyên Container tiêu thụ.
- **Cons**: Chi phí trên mỗi vCPU/RAM đắt hơn một chút so với việc tối ưu hóa 100% trên EC2 Instance.
- **Nên dùng khi**: Muốn tập trung 100% vào code/container, không muốn tốn nhân sự quản lý EC2 Cluster.

### 7. AWS Elastic Beanstalk (PaaS - Platform as a Service)
- **Khái niệm**: Nền tảng hỗ trợ Deploy ứng dụng nhanh chóng mà không cần bận tâm thiết kế hạ tầng bên dưới.
- **Tác dụng**: Bạn đưa Code (Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker) -> Beanstalk tự động dựng EC2, ASG, Load Balancer, Database và Security Group.
- **Pros**: Triển khai cực nhanh (Developer friendly), vẫn cho phép "jump" vào hạ tầng bên dưới để tùy chỉnh nếu cần.
- **Cons**: Mất đi tính linh hoạt soát sâu nếu cấu hình kiến trúc phức tạp.
- **Nên dùng khi**: Muốn demo app nhanh, đội phát triển không có chuyên viên DevOps/Cloud Engineer.
- **KHÔNG nên dùng khi**: Hệ thống lớn có quy trình kiến trúc phức tạp, tùy biến nhiều.

### 8. AWS Batch
- **Khái niệm**: Dịch vụ quản lý và lập lịch chạy các công việc tính toán theo lô (Batch Processing) ở mọi quy mô.
- **Tác dụng**: Tự động chia nhỏ, xếp hàng (Queue) và phân phối hàng ngàn job tính toán lên EC2 / Spot Instances / Fargate.
- **Pros**: Tự động scale từ 0 lên hàng ngàn node và tự hạ xuống 0 khi chạy xong. Đơn giản hóa việc chạy Batch jobs.
- **Nên dùng khi**: Xử lý hình ảnh/video số lượng lớn, phân tích tài chính cuối ngày, mô phỏng khoa học.

### 9. AWS App Runner
- **Khái niệm**: Dịch vụ Serverless hoàn toàn dành cho Web App và API đơn giản.
- **Tác dụng**: Đưa trực tiếp Source Code (GitHub) hoặc Container Image -> App Runner tự build, deploy, cân bằng tải và cấp HTTPS URL trong vài phút.
- **Pros**: Siêu đơn giản, tự động scale traffic từ ít đến nhiều.
- **Cons**: Thiếu các tính năng tùy biến hạ tầng mạng phức tạp.
- **Nên dùng khi**: Cần deploy web app REST API nhanh nhất có thể.

### 10. HPC — AWS Parallel Computing Service
```
HPC workload
    ↓
AWS PCS
    ↓
compute cluster
    ↓
parallel jobs
```

# **B. NETWORKING**
```
=======================================================================================================================
 AWS GLOBAL INFRASTRUCTURE
=======================================================================================================================

 🌐 PUBLIC INTERNET & EXTERNAL USERS
      │
      ├───────────────────────────────────┬───────────────────────────────────┐
      │                                   │                                   │
      ▼                                   ▼                                   ▼
 ┌────────────────────────┐      ┌────────────────────────┐      ┌────────────────────────┐
 │   Amazon Route 53      │      │   Amazon CloudFront    │      │    AWS WAF / Shield    │
 │ (DNS Global Routing)   │      │ (CDN Edge Locations)   │      │ (L7 & DDoS Protection) │
 └───────────┬────────────┘      └───────────┬────────────┘      └───────────┬────────────┘
             │                               │                               │
 ────────────┴───────────────────────────────┴───────────────────────────────┴─────────────────────────────────────────
  AWS REGION (e.g., ap-southeast-1 Singapore)
 ──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
      │
      │  [ Internet Gateway (IGW) ]  ◄── Cổng giao tiếp chính ra/vào Internet
      │               │
 ┌────┴───────────────┴───────────────────────────────────────────────────────────────────────────────────────────────┐
 │  AWS VPC (Virtual Private Cloud)  -  CIDR: 10.0.0.0/16                                                             │
 │                                                                                                                    │
 │   ┌─────────────────────────────────────────────────────────┬─────────────────────────────────────────────────┐    │
 │   │ Availability Zone A (az1)                               │ Availability Zone B (az2)                       │    │
 │   │                                                         │                                                 │    │
 │   │   ┌─────────────────────────────────────────────────┐   │   ┌─────────────────────────────────────────┐   │    │
 │   │   │ PUBLIC SUBNET A (10.0.1.0/24)                   │   │   │ PUBLIC SUBNET B (10.0.2.0/24)           │   │    │
 │   │   │                                                 │   │   │                                         │   │    │
 │   │   │   [ ALB - Application Load Balancer ] ──────────┼───┼───┤ (Multi-AZ Load Balancing)               │   │    │
 │   │   │                  │                              │   │   │                                         │   │    │
 │   │   │   [ NAT Gateway ] ───┐                          │   │   │   [ EIP - Elastic IP ]                  │   │    │
 │   │   └──────────────────┼───┼──────────────────────────┘   │   └─────────────────────────────────────────┘   │    │
 │   │                      │   │                                                         │                      │    │
 │   │                      │   └────────────────────────────────────────┐                │                      │    │
 │   │                      ▼                                            ▼                │                      │    │
 │   │   ┌─────────────────────────────────────────────────┐   ┌──────────────────────────┴──────────────┐       │    │
 │   │   │ PRIVATE SUBNET A (10.0.10.0/24)                 │   │ PRIVATE SUBNET B (10.0.20.0/24)         │       │    │
 │   │   │                                                 │   │                                         │       │    │
 │   │   │   ┌─────────────────────────────────────────┐   │   │   ┌─────────────────────────────────┐   │       │    │
 │   │   │   │ Security Group (Stateful Firewall)      │   │   │   │ Security Group                  │   │       │    │
 │   │   │   │  ┌───────────────────────────────────┐  │   │   │   │  ┌───────────────────────────┐  │   │       │    │
 │   │   │   │  │ App Server (EC2 / ECS / Fargate)  │  │   │   │   │  │ App Server (EC2 / Fargate)│  │   │       │    │
 │   │   │   │  └───────────────────────────────────┘  │   │   │   │  └───────────────────────────┘  │   │       │    │
 │   │   │   └─────────────────────────────────────────┘   │   │   └─────────────────────────────────┘   │       │    │
 │   │   │                        │                        │   │                        │                │       │    │
 │   │   │                        ▼                        │   │                        ▼                │       │    │
 │   │   │   ┌─────────────────────────────────────────┐   │   │   ┌─────────────────────────────────┐   │       │    │
 │   │   │   │ Primary DB (Amazon RDS / Aurora)        │───┼───┼───┤ Standby DB (Auto Failover Sync) │   │       │    │
 │   │   │   └─────────────────────────────────────────┘   │   │   └─────────────────────────────────┘   │       │    │
 │   │   └─────────────────────────────────────────────────┘   └─────────────────────────────────────────┘       │    │
 │   │                                                                                                           │    │
 │   │   NACL (Network Access Control List - Stateless Firewall) gác ở cửa mỗi Subnet                            │    │
 │   └───────────────────────────────────────────────────────────────────────────────────────────────────────────┘    │
 │                                                                                                                    │
 │   ┌───────────────────────────────────────────────────────────────────────────────────────────────────────────┐    │
 │   │ VPC ENDPOINTS (Private Access - Không đi qua Internet)                                                    │    │
 │   │   ├── Gateway Endpoint  ──► Amazon S3 / DynamoDB (Miễn phí, đi qua Route Table)                           │    │
 │   │   └── Interface Endpoint ──► SQS / KMS / Secrets Manager (Dùng ENI Private IP)                            │    │
 │   └───────────────────────────────────────────────────────────────────────────────────────────────────────────┘    │
 └──────────────────────────────┬──────────────────────────────────────────────┬──────────────────────────────────────┘
                                │                                              │
 ───────────────────────────────┼──────────────────────────────────────────────┼──────────────────────────────────────
  EXTENDED HYBRID / EDGE        │                                              │
 ───────────────────────────────┼──────────────────────────────────────────────┼──────────────────────────────────────
                                │                                              │
      ┌─────────────────────────┴──────────────┐                    ┌──────────┴──────────────────────────────┐
      │ AWS TRANSIT GATEWAY (TGW) / PEERING    │                    │ AWS LOCAL ZONE / OUTPOSTS               │
      │ (Hub-and-Spoke Router cho nhiều VPC)   │                    │ (Đặt gần user/On-premise để Ultra-low   │
      └───────────┬────────────────┬───────────┘                    │  latency)                               │
                  │                │                                └─────────────────────────────────────────┘
                  │                │
                  ▼                ▼
     ┌──────────────────┐    ┌───────────────────────────────────┐
     │ Site-to-Site VPN │    │ AWS Direct Connect (DX)           │
     │ (IPSec Internet) │    │ (Đường cáp riêng vật lý 1-100Gbps)│
     └────────┬─────────┘    └─────────────────┬─────────────────┘
              │                                │
              └────────────────┐───────────────┘
                               ▼
              ┌─────────────────────────────────┐
              │ ON-PREMISES DATA CENTER         │
              │ (Doanh nghiệp tự vận hành)      │
              └─────────────────────────────────┘
```
## I. MẠNG NỘI BỘ VÀ ĐIỀU HƯỚNG (VPC CORE)

### 1. Amazon VPC (Virtual Private Cloud)
- **Khái niệm**: Mạng riêng ảo isolated hoàn toàn dành riêng cho bạn trên Cloud.
- **Các thành phần cốt lõi bên trong VPC**:
    + *CIDR Block*: Dải IP nội bộ gán cho VPC (Ví dụ: 10.0.0.0/16 -> 65,536 IP). Không thể thay đổi size sau khi tạo.
    + *Subnet*: Phân vùng dải IP của VPC vào từng Availability Zone (AZ).
        * *Public Subnet*: Có Route Table trỏ ra Internet Gateway.
        * *Private Subnet*: KHÔNG có đường trỏ trực tiếp ra Internet Gateway.
    + *Internet Gateway (IGW)*: Cổng nối giữa VPC và Internet. Mang tính HA, tự động scale. 1 VPC chỉ gắn được tối đa 1 IGW.
    + *NAT Gateway / NAT Instance*: Đặt ở Public Subnet, cho phép các máy chủ ở Private Subnet đi ra ngoài Internet (để tải patch, update) nhưng chặn chiều ngược lại từ Internet chui vào Private Subnet.
        * *NAT Gateway*: Quản lý bởi AWS, HA tự động, băng thông tới 100 Gbps, tốn tiền theo GB dữ liệu qua cổng.
        * *NAT Instance*: Tự dựng EC2 chạy NAT (Legacy), phải tự quản lý HA, rẻ hơn nếu traffic ít.
    + *Egress-Only Internet Gateway*: Tương tự NAT Gateway nhưng dành riêng cho lượng truy cập IPv6.
    + *Route Table*: Bảng quy định luồng đi của traffic (Destination -> Target).
- **Pros**: Cô lập hoàn toàn hạ tầng, linh hoạt thiết kế kiến trúc bảo mật nhiều lớp.
- **Cons**: Thiết kế sai CIDR ban đầu sẽ gây rắc rối lớn khi cần kết nối (Overlap IP) với mạng khác.
- **Nên dùng khi**: BẮT BUỘC cho mọi kiến trúc hạ tầng trên AWS.

### 2. Elastic IP (EIP)
- **Khái niệm**: Địa chỉ IPv4 tĩnh công cộng (Static Public IPv4) không bị đổi khi bạn Stop/Start EC2.
- **Quy tắc chi phí (Cực hay hỏi)**:
    + Miễn phí nếu gắn vào 1 EC2 đang chạy.
    + BỊ TÍNH PHÍ nếu tạo ra mà KHÔNG gán vào đâu, hoặc gán vào 1 EC2 đang trong trạng thái Stopped. (AWS phạt để tránh lãng phí IPv4).

## II. KẾT NỐI LIÊN VPC VÀ HYBRID CLOUD (KẾT NỐI LAI)

### 3. VPC Peering
- **Khái niệm**: Kết nối trực tiếp 2 VPC lại với nhau bằng đường truyền riêng của AWS.
- **Đặc điểm quan trọng (Cần nhớ để làm bài thi)**:
    + KHÔNG có tính bắc cầu (Non-Transitive Routing): Nếu VPC A peer với B, B peer với C ➔ A KHÔNG THỂ nói chuyện với C qua B. Muốn A nối C phải tạo thêm đường Peer A-C.
    + Hai VPC KHÔNG ĐƯỢC trùng dải IP (No Overlapping CIDR).
    + Có thể Peer giữa các AWS Account khác nhau và giữa các Region khác nhau (Inter-Region VPC Peering).
Nên dùng khi: Kết nối một vài VPC đơn lẻ với nhau (dưới 10-20 VPC).

### 4. AWS Transit Gateway (TGW)
- **Khái niệm**: Bộ tuyến trung tâm (Hub-and-Spoke router) cho phép kết nối hàng ngàn VPC và mạng On-premises lại với nhau.
- **Tác dụng**: Giải quyết sự phức tạp của VPC Peering khi số lượng VPC tăng lên quá nhiều (loại bỏ mạng lưới Peering chằng chịt mesh network).
- **Tính năng**: Hỗ trợ tính bắc cầu (Transitive Routing), hỗ trợ IP Multicast.
- **Pros**: Quản lý tập trung mạng quy mô lớn vô cùng đơn giản.
- **Cons**: Chi phí đắt (tính tiền theo TGW Attachment + lượng Data qua cổng).
- **Nên dùng khi**: Doanh nghiệp có hàng chục/hàng trăm VPC cần kết nối liên thông với On-premise.

### 5. AWS Site-to-Site VPN
- **Khái niệm**: Kết nối mã hóa an toàn qua môi trường Internet công cộng giữa Data Center On-premise và AWS VPC.
- Thành phần:
    + *Customer Gateway (CGW)*: Thiết bị mạng hoặc phần mềm đặt tại On-premise.
    + *Virtual Private Gateway (VGW) hoặc Transit Gateway (TGW)*: Đầu nối phía trên AWS.

- **Pros**: Triển khai nhanh, mã hóa IPSec an toàn, chi phí rẻ hơn Direct Connect.
- **Cons**: Tốc độ và độ trễ phụ thuộc vào chất lượng đường truyền Internet công cộng.

### 6. AWS Direct Connect (DX)
- **Khái niệm**: Đường dây cáp vật lý riêng biệt (Dedicated Physical Connection) nối trực tiếp từ Data Center của bạn đến AWS.
- **Pros**: Băng thông cực lớn (1Gbps - 100Gbps), độ trễ siêu thấp, đường truyền ổn định không qua Internet, giảm chi phí Data Transfer Out.
- **Cons**: Tốn thời gian triển khai (vài tuần/tháng), chi phí đầu tư ban đầu rất cao.
- **Mô hình kết hợp chuẩn cho HA (High Availability)**:
    + Đường chính: Direct Connect.
    + Đường dự phòng (Backup): Site-to-Site VPN (nếu DX bị đứt cáp thì VPN nhảy vào gánh).

### 7. AWS Client VPN
- **Khái niệm**: Dịch vụ Managed VPN dựa trên OpenVPN dành cho nhân viên (End-users) kết nối an toàn từ máy tính cá nhân vào tài nguyên bên trong VPC.

## III. BẢO MẬT MẠNG VÀ TƯỜNG LỬA (NETWORK SECURITY)

### 8. Security Groups (SG)
- **Khái niệm**: Tường lửa ảo kiểm soát traffic ở cấp độ Instance (NIC/ENI).
- **Đặc điểm bắt buộc thuộc lòng**:
    +Stateful: Nếu Inbound rule cho phép traffic vào ➔ Outbound rule tự động mở để phản hồi ra (không cần cấu hình Outbound).
    +Chỉ có quy tắc ALLOW (Cho phép), KHÔNG có quy tắc DENY (Cấm).
    +Mặc định: Chặn toàn bộ Inbound, Mở toàn bộ Outbound.
    +Có thể trỏ Source/Destination tới CIDR IP hoặc tới một Security Group khác (SG Referencing).

### 9. Network Access Control Lists (NACL)
- **Khái niệm**: Tường lửa ảo kiểm soát traffic ở cấp độ Subnet.
- **Đặc điểm bắt buộc thuộc lòng**:
    + Stateless: Inbound và Outbound độc lập hoàn toàn. Cho phép Inbound vào thì PHẢI cấu hình mở Outbound để gói tin đi ra.
    + Hỗ trợ cả quy tắc ALLOW và DENY. (Ví dụ: Muốn chặn 1 IP hacker cụ thể 1.2.3.4 ➔ Dùng NACL DENY, Security Group KHÔNG làm được việc này).
    + Được xử lý theo thứ tự số Rule (Rule number từ nhỏ đến lớn, gặp rule khớp đầu tiên là thực thi ngay).

### 10. AWS WAF (Web Application Firewall)
- *Khái niệm*: Tường lửa ứng dụng Layer 7 (Application Layer).
- *Tác dụng*: Bảo vệ ứng dụng khỏi các lỗ hổng web phổ biến (SQL Injection, Cross-Site Scripting - XSS, HTTP Flood/DDoS).
- *Vị trí gắn*: Gắn được vào Amazon CloudFront, Application Load Balancer (ALB), Amazon API Gateway, hoặc AWS AppSync.
- *Cách hoạt động*: Cho phép viết Web ACL (Rules) dựa trên IP, Header, Body, URI, hoặc dùng Managed Rules có sẵn của AWS.

### 11. AWS Shield (Standard & Advanced)
- **Khái niệm**: Dịch vụ chống tấn công từ chối dịch vụ (DDoS Protection) ở Layer 3 và Layer 4.
- **2 Cấp độ**:
    + *AWS Shield Standard*: Miễn phí 100%, bật tự động cho mọi khách hàng AWS (Chống SYN Floods, UDP Floods).
    + *AWS Shield Advanced*: Dịch vụ trả phí cao ($3,000/tháng), chống DDoS nâng cao cho CloudFront, Route53, ALB, Route53. Có đội ngũ DRT (DDoS Response Team) hỗ trợ 24/7 và bảo hiểm chi phí khi bị DDoS gây bùng nổ bill.

### 12. AWS Network Firewall
- **Khái niệm**: Tường lửa mạng Stateful/Stateless nâng cao cấp độ VPC (Layer 3 - Layer 7).
- **Tác dụng**: Kiểm tra chuyên sâu gói tin (Deep Packet Inspection), lọc URL/Domain name, phát hiện và ngăn chặn xâm nhập (IDS/IPS) cho toàn bộ VPC.

## IV. BẢO MẬT KẾT NỐI RIÊNG VỚI DỊCH VỤ AWS (VPC ENDPOINTS)
- Tránh việc EC2 ở Private Subnet phải đi qua NAT Gateway + Internet để gọi các dịch vụ AWS khác.

### 13. VPC Endpoint (AWS PrivateLink)
- **Tác dụng**: Cho phép tài nguyên trong VPC kết nối tới các dịch vụ AWS (như S3, DynamoDB, SQS...) qua đường mạng nội bộ riêng của AWS, không bao giờ đi qua Internet.
- **2 Loại VPC Endpoint (CỰC KỲ HAY HỎI THI)**:
    - *Gateway Endpoint*:
        + Chỉ hỗ trợ cho đúng 2 dịch vụ: Amazon S3 và Amazon DynamoDB.
        + Cách dùng: Đính kèm trực tiếp vào Route Table của Subnet.
        + Chi phí: MIỄN PHÍ.
    - *Interface Endpoint (PrivateLink)*:
        + Hỗ trợ hầu hết các dịch vụ AWS còn lại (SQS, SNS, Kinesis, KMS...) và cả Custom Services.
        + Cách dùng: Tạo ra một Elastic Network Interface (ENI) có IP nội bộ cắm trực tiếp vào Subnet.
        + Chi phí: TÍNH PHÍ theo giờ + dung lượng data truyền qua.

## V. ĐIỀU HƯỚNG VÀ PHÂN PHỐI TRAFFIC (LOAD BALANCING & CDN)

### 14. Elastic Load Balancing (ELB)
- **Khái niệm**: Cân bằng tải tự động phân phối lượng traffic truy cập vào dàn target (EC2, Container, IP, Lambda).
- **3 Loại Load Balancer chính**:
    + *Application Load Balancer (ALB) - Layer 7*:
        * Hiểu nội dung HTTP/HTTPS.
        * Routing thông minh: Route theo Path (/api, /users), theo Hostname (app.domain.com), theo Header.
        * Hỗ trợ Redirect HTTP sang HTTPS, hỗ trợ WebSockets.
    + *Network Load Balancer (NLB) - Layer 4*:
        * Chạy ở tầng TCP/UDP/TLS.
        * Hiệu năng cực cao: Xử lý hàng triệu request/giây với độ trễ cực thấp (Ultra-low latency).
        * Có Static IP / Elastic IP cố định cho mỗi AZ (ALB không có IP cố định).
    + *Gateway Load Balancer (GWLB) - Layer 3*:
        * Dùng để cân bằng tải cho các thiết bị mạng bên thứ 3 (Firewall, IDS/IPS ảo) trước khi traffic chui vào hệ thống của bạn.

### 15. Amazon Route 53 (DNS & Traffic Routing)
- **Khái niệm**: Dịch vụ DNS Quản lý Tên miền và Điều hướng Traffic toàn cầu.
- **Routing Policies (Các chiến lược điều hướng - THI RẤT NHIỀU)**:
    + *Simple Routing*: Trỏ domain về 1 tài nguyên duy nhất (không hỗ trợ health check).
    + *Weighted Routing*: Chia % traffic (Ví dụ: 80% vào bản cũ, 20% vào bản mới để thử nghiệm A/B Testing).
    + *Latency-based Routing*: Trỏ người dùng về Region nào có độ trễ (latency) thấp nhất với họ.
    + *Failover Routing*: Dùng cho Disaster Recovery (Primary - Secondary). Nếu Primary sập Health Check ➔ Tự chuyển traffic sang Secondary.
    + *Geolocation Routing*: Trỏ dựa trên vị trí địa lý của người dùng (Ví dụ: Khách ở VN trỏ về Server VN, khách ở Mỹ trỏ về Server US).
    + *Geoproximity Routing*: Điều hướng dựa trên vị trí địa lý của cả người dùng VÀ tài nguyên (dùng Bias).
    + *Multi-Value Answer Routing*: Trả về ngẫu nhiên tối đa 8 IP khỏe mạnh (Healthy Records).

### 16. Amazon CloudFront (CDN)
- **Khái niệm**: Mạng phân phối nội dung (Content Delivery Network) phủ khắp thế giới qua hệ thống Edge Locations.
- **Tác dụng**: Cache nội dung tĩnh/động gần người dùng nhất để giảm độ trễ (Latency) và bảo vệ Server gốc (Origin).
- *Bảo mật với CloudFront*:
    + Origin Access Control (OAC) / OAI: Đảm bảo người dùng BẮT BUỘC phải đi qua CloudFront mới lấy được file trên S3, chặn không cho ai truy cập trực tiếp URL của S3 Bucket.
    + Signed URLs / Signed Cookies: Chỉ cho phép những người dùng đã trả phí/đăng nhập mới được tải nội dung riêng tư.

### 17. AWS Global Accelerator
- **Khái niệm**: Dịch vụ tối ưu mạng dùng đường truyền cáp quang riêng toàn cầu của AWS để tăng tốc độ truy cập cho ứng dụng.
- **Cách hoạt động**: Cấp cho bạn 2 Static Anycast IP. Traffic của người dùng chui vào Edge Location gần nhất của AWS ➔ Chạy qua đường cáp ngầm riêng của AWS tới ALB/EC2 ở Region đích.
- *So sánh CloudFront vs Global Accelerator*:
    + *CloudFront*: Dùng cho cả HTTP/HTTPS, CÓ CACHE dữ liệu tĩnh (Video, ảnh, HTML).
    + *Global Accelerator*: Dùng cho cả TCP/UDP (Game, VoIP, HTTP), KHÔNG CACHE, chỉ đơn thuần là tăng tốc độ kết nối mạng cho dữ liệu động.

# C. STORAGE 
Mô hình lưu trữ trên AWS chia làm 3 dạng chính: **Block Storage** (dành cho OS/Database), **File Storage** (dùng chung cho nhiều máy chủ), và **Object Storage** (lưu tệp tin khổng lồ qua Internet).

## I. OBJECT STORAGE (LƯU TRỮ ĐỐI TƯỢNG)

### 1. Amazon S3 (Simple Storage Service)
- **Khái niệm:** Dịch vụ lưu trữ tệp tin (ảnh, video, backup, log, static website) không giới hạn dung lượng với độ bền cực cao (Durability 11 số 9: $99.999999999\%$).
- **Đặc điểm quan trọng:**
    + Tệp tin gọi là **Object** (Dung lượng tối đa 1 object = 5TB).
    + Chứa trong các thùng gọi là **Bucket** (Tên Bucket phải là **duy nhất trên toàn thế giới / Globally unique**).
    + Mã hóa mặc định: Server-Side Encryption (SSE-S3, SSE-KMS, SSE-C).

- **Các lớp lưu trữ (Storage Classes - CỰC HAY HỎI TÍNH CHI PHÍ):**
    + *S3 Standard:* Dùng cho dữ liệu truy cập thường xuyên, độ trễ milliseconds.
    + *S3 Intelligent-Tiering:* Tự động chuyển đổi giữa các tier để tối ưu chi phí dựa trên tần suất truy cập (không tốn phí lấy dữ liệu).
    + *S3 Standard-IA (Infrequent Access):* Dữ liệu ít truy cập nhưng khi cần lấy phải có ngay. Rẻ hơn Standard về tiền lưu trữ, nhưng tốn phí khi đọc dữ liệu.
    + *S3 One Zone-IA:* Tương tự Standard-IA nhưng chỉ lưu ở 1 AZ (Rẻ hơn 20%, mất dữ liệu nếu AZ đó sập).
    + *S3 Glacier Instant Retrieval:* Lưu trữ архив, lấy dữ liệu ra trong vài miligiây.
    + *S3 Glacier Flexible Retrieval:* Lưu архив, lấy dữ liệu từ vài phút đến 3-5 giờ.
    + *S3 Glacier Deep Archive:* Rẻ nhất toàn bộ AWS! Dùng cho lưu trữ pháp lý/tuân thủ (Compliance), lấy dữ liệu mất 12 - 48 giờ.

- **Tính năng nâng cao hay thi:**
    + *S3 Lifecycle Policies:* Tự động chuyển tệp từ Standard $\rightarrow$ IA $\rightarrow$ Glacier $\rightarrow$ Xóa sau X ngày.
    + *S3 Versioning:* Lưu nhiều phiên bản của 1 tệp (chống ghi đè/xóa nhầm).
    + *S3 Object Lock / WORM (Write Once Read Many):* Chặn xóa/sửa file trong một khoảng thời gian (chống Ransomware).
    + *Cross-Region Replication (CRR) / Same-Region Replication (SRR):* Tự động nhân bản Bucket sang Region/AZ khác (Cần bật Versioning trước).

## II. BLOCK STORAGE (Ổ ĐĨA GẮN VÀO MÁY CHỦ)

### 2. Amazon EBS (Elastic Block Store)
- **Khái niệm:** Ổ cứng ảo (Network Drive) gắn vào 1 EC2 Instance để cài OS hoặc Database.
- **Đặc điểm:**
    + Nằm cố định ở **1 AZ duy nhất** (muốn chuyển sang AZ khác phải chụp Snapshot rồi Restore sang AZ mới).
    + Mặc định khi xóa EC2, ổ root EBS sẽ bị xóa theo (trừ khi tắt thuộc tính `DeleteOnTermination`).

- **Các loại EBS Volumes (Hay hỏi trong đề thi):**
    + **gp2 / gp3 (General Purpose SSD):** Dùng chung cho OS, dev/test, DB vừa và nhỏ. Cân bằng giữa giá thành và hiệu năng.
    + **io1 / io2 Block Express (Provisioned IOPS SSD):** Dùng cho ứng dụng siêu gắt về hiệu năng (Mission-critical DB, OLTP như Oracle, SQL Server). Cho phép chỉnh IOPS độc lập với dung lượng.
    + **st1 (Throughput Optimized HDD):** Dùng cho Big Data, Data Warehousing, Log processing (truy cập tuần tự - Sequential Read/Write).
    + **sc1 (Cold HDD):** Rẻ nhất trong nhóm EBS, dùng cho file lưu trữ ít truy cập.

| Volume | Size | Baseline IOPS | Max IOPS | Max Throughput | Điểm cần nhớ |
|---|---:|---:|---:|---:|---|
| **gp3** | **1 GiB – 64 TiB** | **3,000** | **80,000** | **2,000 MiB/s** | IOPS & throughput provision độc lập |
| **gp2** | **1 GiB – 64 TiB** | **3 IOPS/GiB** | **16,000** | **250 MiB/s** | IOPS phụ thuộc size |
| **io1** | **4 GiB – 64 TiB** | Provisioned | **64,000** | **1,000 MiB/s** | Provisioned IOPS |
| **io2** | **4 GiB – 64 TiB** | Provisioned | **64,000** | **1,000 MiB/s** | Durability cao hơn io1 |
| **io2 Block Express** | **4 GiB – 64 TiB** | Provisioned | **256,000** | **4,000 MiB/s** | Mission-critical / extreme IOPS |
| **st1** | **125 GiB – 16 TiB** | **40 MiB/s/TiB** | **500 IOPS** | **500 MiB/s** | Sequential throughput |
| **sc1** | **125 GiB – 16 TiB** | **12 MiB/s/TiB** | **250 IOPS** | **250 MiB/s** | Cold / lowest-cost HDD |

- **EBS Multi-Attach:** Cho phép gắn 1 EBS volume (`io1/io2`) vào **nhiều EC2 instances trong cùng 1 AZ** cùng lúc (dùng cho Cluster application).

### 3. EC2 Instance Store (Ephemeral Storage)
- **Khái niệm:** Ổ cứng vật lý cắm trực tiếp vào máy chủ vật lý chứa EC2.
- **Pros:** Băng thông và IOPS cực khủng (hàng trăm ngàn IOPS), độ trễ siêu thấp.
- **Cons (ĐIỂM YẾU CHÍNH):** **Tạm thời (Ephemeral)**. Nếu EC2 bị `Stop` hoặc `Terminate`, **dữ liệu bị MẤT HOÀN TOÀN**. (Chỉ giữ được dữ liệu khi `Reboot`).
- **Nên dùng khi:** Làm ổ Cache, Buffer, Temp data, Scratch data không sợ mất.

## III. FILE STORAGE (LƯU TRỮ TỆP DÙNG CHUNG)

### 4. Amazon EFS (Elastic File System)
- **Khái niệm:** Hệ thống File System quản lý hoàn toàn (Managed Network File System - NFSv4) có thể gắn vào **hàng trăm EC2 instances ở NHIỀU AZ khác nhau** cùng một lúc.
- **Đặc điểm:**
    + Chỉ tương thích với hệ điều hành **Linux** (POSIX-compliant).
    + Tự động scale dung lượng (không cần phân bổ trước dung lượng như EBS).
    + Đắt hơn EBS nhưng hỗ trợ kết nối đồng thời từ nhiều AZ.



### 5. Amazon FSx (Managed File Shares cho các OS/File System đặc thù)

Nếu ứng dụng cần File System dùng chung phức tạp hơn EFS, dùng FSx:

+ **FSx for Windows File Server:** Chạy native SMB protocol, tích hợp Microsoft Active Directory (Dành riêng cho máy chủ Windows).
+ **FSx for Lustre:** Dùng cho High Performance Computing (HPC), Machine Learning, tính toán song song dữ liệu lớn.
+ **FSx for NetApp ONTAP & FSx for OpenZFS:** Dùng để migrate các hạ tầng Storage On-premises chạy NetApp/ZFS lên AWS mà không cần sửa code.

## IV. HYBRID STORAGE & DATA MIGRATION (CHUYỂN ĐỔI VÀ KẾT NỐI LAI)

### 6. AWS Storage Gateway
- **Khái niệm:** Cầu nối giữa Data Center On-premises và bộ lưu trữ Cloud (S3/EBS).
- **3 Loại Storage Gateway:**
    + *S3 File Gateway:* Giúp ứng dụng On-premise lưu file trực tiếp lên S3 qua giao thức NFS/SMB.
    + *Volume Gateway:* Mở rộng ổ đĩa On-premise lên AWS (iSCSI). Lưu bản sao Snapshot lên S3.
    + *Tape Gateway:* Thay thế hệ thống băng đĩa từ (Physical Tapes) On-premise bằng cách lưu backup sang S3/Glacier.

### 7. AWS Snow Family (Chuyển dữ liệu vật lý quy mô lớn)

Dùng khi dung lượng dữ liệu quá lớn (vài TB đến hàng PB) mà đường truyền Internet không thể tải nổi (tốn hàng tháng/hàng năm). AWS sẽ gửi thiết bị vật lý đến tận nhà bạn để chép data rồi đem về Data Center của AWS.

+ **Snowcone:** Thiết bị nhỏ gọn (8TB usable), có thể xách tay.
+ **Snowball Edge:**
+ *Snowball Edge Storage Optimized:* Lưu trữ dữ liệu lớn (80TB+).
+ *Snowball Edge Compute Optimized:* Chứa vCPU + GPU để tính toán tại chỗ (Edge Computing).
+ **Snowmobile:** Một chiếc **xe container khổng lồ** chở thiết bị lưu trữ tới 100 Petabytes (PB) chạy thẳng tới Data Center của bạn.

# D. DATABASES

Mảng này chia thành 2 nhóm chính: **Relational Databases (SQL)** dành cho dữ liệu có cấu trúc chặt chẽ và **Non-Relational Databases (NoSQL / In-Memory / Analytical)** dành cho độ trễ siêu thấp, scale không giới hạn hoặc phân tích dữ liệu lớn.

## I. CƠ SỞ DỮ LIỆU QUAN HỆ (RELATIONAL DATABASES - SQL)

Dùng cho dữ liệu có cấu trúc, cần tính toàn vẹn **ACID**, hỗ trợ các câu lệnh JOIN phức tạp (Dùng cho E-commerce, Banking, ERP).

### 1. Amazon RDS (Relational Database Service)
- **Khái niệm:** Dịch vụ CSDL quan hệ do AWS quản lý hoàn toàn (Managed SQL).
- **Hỗ trợ 6 Database Engines:** PostgreSQL, MySQL, MariaDB, Oracle, Microsoft SQL Server, và Amazon Aurora.
- **Các tính năng cốt lõi bắt buộc thuộc lòng cho bài thi:**
    + **Multi-AZ Deployment (Dùng cho High Availability & Disaster Recovery):**
        * Tự động đồng bộ dữ liệu (Synchronous replication) sang một **Standby DB** ở AZ khác.
        * Nếu Primary DB bị sập, AWS tự động chuyển hướng (Auto-failover) sang Standby DB mà không bị đổi Endpoint connection.
        * *Chú ý:* Standby DB **không dùng để đọc/ghi dữ liệu được** (chỉ đứng chờ dự phòng).

    + **Read Replicas (Dùng cho Scaling hiệu năng đọc - Read Performance):**
        * Tạo ra các bản sao bất đồng bộ (Asynchronous replication) để gánh tải cho các câu lệnh đếm/đọc/báo cáo (`SELECT`).
        * Có thể tạo tối đa 15 Read Replicas, nằm khác AZ hoặc **khác Region (Cross-Region Read Replica)**.
        * Read Replicas **CÓ THỂ truy cập để đọc dữ liệu**, và có thể nâng cấp thành Primary DB độc lập khi cần.
    + **Backups:**
        * *Automated Backups:* Tự động sao lưu hàng ngày + lưu transaction logs, hỗ trợ khôi phục về từng giây (Point-In-Time Restore - PITR) trong Retention Period (1 - 35 ngày).
        * *Manual DB Snapshots:* Khôi phục thủ công do người dùng chủ động bấm tạo, giữ bao lâu tùy thích kể cả khi xóa RDS instance.

### 2. Amazon Aurora
- **Khái niệm:** CSDL quan hệ do chính AWS thiết kế lại tối ưu riêng cho Cloud, hoàn toàn tương thích với **MySQL** và **PostgreSQL**.
- **Hiệu năng:** Nhanh gấp **5 lần** MySQL thường và **3 lần** PostgreSQL thường, nhưng chi phí chỉ bằng 1/10 các CSDL thương mại (Oracle/SQL Server).
- **Kiến trúc lưu trữ độc quyền (Cực kỳ hay hỏi):**
    + Dữ liệu được chia nhỏ và nhân bản thành **6 bản sao (6 copies) trải rộng trên 3 Availability Zones (AZs)**.
    + Chỉ cần 4/6 bản sao ghi thành công là tính thành công (Chịu lỗi mất 1 AZ + 1 copy mà không mất dữ liệu).
    + Tự động mở rộng dung lượng ổ đĩa (Auto-scaling storage) lên tới 128TB.

- **Các biến thể của Aurora:**
    + **Aurora Serverless (v2):** Tự động scale dung lượng tính toán (ACUs) lên/ลง theo traffic thực tế milliseconds. Tối ưu chi phí cho ứng dụng có lượng truy cập thất thường/không dự đoán được.
    + **Aurora Global Database:** 1 Primary Region (Ghi/Đọc) và tối đa 5 Secondary Regions (Chỉ đọc). Độ trễ đồng bộ liên Region < 1 giây, hỗ trợ Disaster Recovery toàn cầu.
    + **Aurora Multi-Master:** Cho phép tất cả các DB Instances ở nhiều AZ cùng có quyền Ghi (Read/Write) để đạt HA cấp độ ghi.

## II. CƠ SỞ DỮ LIỆU KHÔNG QUAN HỆ & IN-MEMORY (NOSQL & CACHING)

### 3. Amazon DynamoDB
- **Khái niệm:** CSDL NoSQL dạng **Key-Value** và **Document** hoàn toàn Serverless, quản lý hoàn toàn.
- **Đặc điểm nổi bật:**
    + Độ trễ siêu thấp dưới 10 miligiây (**Single-digit millisecond latency**) ở bất kỳ quy mô dữ liệu nào.
    + Scale vô tận, tự động chia tải (Partitioning) mà không cần quản lý Server.

- **Tính năng hay thi:**
    + **DynamoDB Accelerator (DAX):** Bộ nhớ Cache In-Memory dành riêng cho DynamoDB, giúp giảm độ trễ đọc xuống mức **Microseconds** (micro-giây) - nhanh gấp 10 lần.
    + **DynamoDB Streams:** Ghi lại dòng sự kiện thay đổi dữ liệu (Insert/Update/Delete) theo thời gian thực ➔ Thường kết hợp với **AWS Lambda** để làm kiến trúc Event-Driven.
    + **Global Tables:** CSDL Active-Active đa vùng (Multi-Region), cho phép ghi và đọc dữ liệu từ nhiều Region khác nhau với độ trễ thấp.
    + **TTL (Time To Live):** Tự động xóa các item đã hết hạn để tiết kiệm dung lượng.
    + **Read/Write Capacity Modes:** *On-Demand Mode* (Trả tiền theo request, hợp với tải thất thường) và *Provisioned Mode* (Đặt trước RCU/WCU, hỗ trợ Auto Scaling, rẻ hơn nếu tải ổn định).

### 4. Amazon ElastiCache
- **Khái niệm:** Dịch vụ lưu trữ bộ nhớ đệm In-memory (In-Memory Caching) hoàn toàn quản lý.
- **Tác dụng:** Đặt phía trước CSDL quan hệ (RDS/Aurora) để Cache các câu truy vấn phổ biến, giảm tải cho Database chính và tăng tốc độ đọc cho ứng dụng.
- **2 Engines phổ biến:**
    + **Redis:** Hỗ trợ cấu trúc dữ liệu phức tạp (Lists, Sets, Geospatial), hỗ trợ Multi-AZ, Read Replicas, Persistent data (lưu được đĩa) và Backup.
    + **Memcached:** Bộ đệm đơn thuần (Pure In-memory cache), multithreaded, không hỗ trợ replication hay persistent data.

* *Lưu ý:* Nếu ứng dụng dùng **DynamoDB**, hãy dùng **DAX**. Nếu dùng **RDS/Aurora**, hãy dùng **ElastiCache**.

## III. CƠ SỞ DỮ LIỆU CHUYÊN DỤNG (SPECIALIZED DATABASES)

### 5. Amazon Redshift (Data Warehousing / OLAP)
- **Khái niệm:** Kho dữ liệu (Data Warehouse) quy mô lớn dùng cho phân tích, báo cáo kinh doanh (Business Intelligence - BI) và xử lý câu lệnh SQL phức tạp trên tập dữ liệu hàng Terabyte đến Petabyte.
- **Công nghệ:** Lưu trữ theo dạng cột (**Columnar Storage**) và xử lý song song khối lượng lớn (Massively Parallel Processing - MPP) giúp truy vấn analytical siêu nhanh.
- **Redshift Spectrum:** Cho phép chạy truy vấn SQL trực tiếp lên dữ liệu thô dạng file (Parquet, CSV, JSON) đang nằm ở **Amazon S3** mà không cần load dữ liệu vào Redshift.

### 6. Amazon DocumentDB
- CSDL Document quản lý hoàn toàn, tương thích native với **MongoDB**.

### 7. Amazon Neptune
- CSDL dạng đồ thị (**Graph Database**) dùng để lưu trữ và truy vấn các mối quan hệ phức tạp giữa các mạng lưới (Mạng xã hội, Hệ thống phát hiện gian lận tài chính Fraud Detection, Knowledge Graphs).

### 8. Amazon Timestream
- CSDL chuỗi thời gian (**Time-Series Database**) Serverless dành riêng cho dữ liệu IoT, chỉ số giám sát (Metrics) và Event tracking theo mốc thời gian.

# E. SECURITY, IDENTITY & COMPLIANCE

## I. IDENTITY & ACCESS MANAGEMENT

### 1. AWS IAM (Identity and Access Management)
- **Khái niệm:** Dịch vụ kiểm soát **AI** có quyền làm **GÌ** trên tài nguyên nào của AWS.
- **Các thành phần cốt lõi:**
    + **Users:** Đại diện cho 1 người dùng hoặc 1 ứng dụng.
    + **Groups:** Tập hợp các User để gán quyền chung (không thể chứa Group khác).
    + **Roles:** Quyền hạn tạm thời (Temporary Credentials). **BẮT BUỘC dùng Role khi cấp quyền cho EC2, Lambda hoặc ứng dụng gọi dịch vụ AWS khác** (tuyệt đối không hardcode Access Key/Secret Key vào code).
    + **Policies:** Các tài liệu JSON định nghĩa quyền (Allow/Deny).

- **Quy tắc đánh giá Policy (CỰC KỲ HAY HỎI THI):**
    + Mặc định: **Explicit Deny** (Mọi hành động đều bị cấm nếu không được khai báo).
    + Nếu có 1 quy tắc **DENY** xuất hiện $\rightarrow$ Nó sẽ ghi đè (Override) tất cả các quy tắc ALLOW khác.

### 2. AWS Organizations & Service Control Policies (SCPs)
- **AWS Organizations:** Quản lý tập trung nhiều tài khoản AWS (Multi-Account Setup) của doanh nghiệp.
- **Service Control Policies (SCPs):** Tường lửa quy định quyền hạn tối đa ở cấp độ Tài khoản (Account-level / OU-level).
- *Chú ý:* SCP **KHÔNG cấp quyền (Allow)**, nó chỉ đặt ra **Ranh giới quyền hạn tối đa (Boundary)**. Dù IAM User có quyền `AdministratorAccess`, nhưng nếu SCP của Account bị Deny dịch vụ S3 $\rightarrow$ User đó vẫn KHÔNG THỂ dùng S3.

### 3. AWS IAM Identity Center (Successor to AWS Single Sign-On)
- Quản lý đăng nhập tập trung (SSO) cho nhiều tài khoản AWS và các ứng dụng doanh nghiệp (Salesforce, Office 365) thông qua Active Directory hoặc Okta.

### 4. Amazon Cognito
- Dịch vụ quản lý đăng ký/đăng nhập cho **người dùng cuối (End-users)** của ứng dụng Web/Mobile.
- **User Pools:** Quản lý cơ sở dữ liệu User (Đăng ký, Đăng nhập, MFA, Reset Password).
- **Identity Pools (Federated Identities):** Cấp quyền tạm thời (AWS Credentials) cho người dùng cuối truy cập trực tiếp vào tài nguyên AWS (như Upload ảnh thẳng lên S3).

## II. DATA PROTECTION & ENCRYPTION

### 5. AWS KMS (Key Management Service)
- **Khái niệm:** Dịch vụ tạo và quản lý khóa mã hóa (KMS Keys / CMK) do AWS quản lý hoàn toàn.
- **Các loại KMS Keys:**
    + *AWS Owned Keys:* Miễn phí, do AWS tự tạo và quản lý ngầm.
    + *AWS Managed Keys:* Miễn phí, tự động tạo cho từng dịch vụ (`aws/s3`, `aws/ebs`).
    + *Customer Managed Keys:* Trả phí, do bạn tự tạo, tự cài đặt chính sách xoay khóa (Key Rotation) và cấp quyền truy cập.

- **Đặc điểm:** Tích hợp sẵn với EBS, S3, RDS, DynamoDB... Khóa KMS là **Regional** (không thể chuyển trực tiếp sang Region khác, muốn dùng phải Re-encrypt hoặc Export).

### 6. AWS CloudHSM (Hardware Security Module)
- **Khái niệm:** Thiết bị phần cứng mã hóa dedicated chuyên dụng, đạt chuẩn FIPS 140-2 Level 3.
- **So sánh KMS vs CloudHSM:**
    + *KMS:* Shared hardware, rẻ, dễ dùng, AWS quản lý phần cứng.
    + *CloudHSM:* Dedicated hardware cho riêng bạn, đắt, bạn toàn quyền quản lý chìa khóa và hệ thống (phục vụ yêu cầu tuân thủ khắt khe khâu pháp lý).

### 7. AWS Certificate Manager (ACM)
- Dịch vụ cấp phát và tự động gia hạn chứng chỉ bảo mật **SSL/TLS (HTTPS)** miễn phí cho các dịch vụ AWS như CloudFront, ALB, API Gateway.

## III. THREAT DETECTION & AUDIT

### 8. Amazon GuardDuty
- **Khái niệm:** Hệ thống phát hiện mối đe dọa thông minh (Intelligent Threat Detection) dựa trên Machine Learning.
- **Cơ chế:** Đọc log ngầm (CloudTrail Logs, VPC Flow Logs, DNS Logs, EKS Audit Logs) mà **không làm chậm hệ thống**.
- **Tác dụng:** Cảnh báo khi bị lộ IAM Key, EC2 bị chiếm dụng để đào coin, hoặc S3 bị rò rỉ dữ liệu.

### 9. Amazon Macie
- **Khái niệm:** Dịch vụ dùng AI/Machine Learning để tự động quét **Amazon S3 Buckets**.
- **Mục đích:** Phát hiện các dữ liệu nhạy cảm, thông tin định danh cá nhân (**PII** - Personally Identifiable Information) như số thẻ tín dụng, số CMND/CCCD, Passport... bị để lộ ra ngoài.

### 10. Amazon Inspector
- **Khái niệm:** Dịch vụ tự động **dò quét lỗ hổng bảo mật (Vulnerability Assessment)**.
- **Phạm vi quét:** Tự động quét các gói phần mềm cài đặt trên **EC2 instances**, hình ảnh ứng dụng trong **Amazon ECR (Container Registry)** và **AWS Lambda Functions**.

## IV. SECRETS & SECURITY MANAGEMENT

### 11. AWS Secrets Manager vs. SSM Parameter Store
Rất hay ra câu hỏi so sánh lựa chọn giữa 2 dịch vụ lưu trữ cấu hình bảo mật này:

- **AWS Systems Manager Parameter Store:** Lưu chuỗi cấu hình, tham số, password dạng Plaintext hoặc Encrypted (KMS). **Miễn phí/Rẻ**, không tự động xoay mật khẩu.
- **AWS Secrets Manager:** Chuyên dùng lưu DB Credentials, API Keys. Đắt hơn, hỗ trợ **TỰ ĐỘNG XOAY MẬT KHẨU (Automatic Secret Rotation)** định kỳ bằng cách tích hợp trực tiếp với AWS Lambda và RDS/Aurora.

### 12. AWS Security Hub
- Trung tâm gom tụ (Aggregator) hiển thị bức tranh toàn cảnh về trạng thái an toàn bảo mật (Security Posture) của toàn bộ tài khoản AWS. Gom cảnh báo từ GuardDuty, Macie, Inspector, Firewall Manager... về 1 Dashboard duy nhất.

# F. APPLICATION INTEGRATION & MESSAGING

## I. QUEUING & NOTIFICATION

### 1. Amazon SQS (Simple Queue Service)
- **Khái niệm:** Dịch vụ hàng đợi tin nhắn (Message Queue) hoàn toàn quản lý, giúp giải mã liên kết (Decouple) giữa bên gửi (Producer) và bên nhận (Consumer).
- **2 Loại Queue bắt buộc thuộc lòng:**
    + **Standard Queue:** Tốc độ/Throughput gần như **không giới hạn**, đảm bảo tin nhắn được giao *ít nhất 1 lần* (At-least-once delivery), nhưng **thứ tự tin nhắn có thể bị xáo trộn**.
    + **FIFO Queue (First-In-First-Out):** Đảm bảo **đúng thứ tự** tin nhắn đến trước xử lý trước, đảm bảo *chỉ giao đúng 1 lần* (Exactly-once delivery), nhưng giới hạn Throughput (300 req/s hoặc 3000 req/s nếu dùng batching).

- **Các tham số tính năng CỰC HAY HỎI THI:**
    + **Visibility Timeout:** Khoảng thời gian một tin nhắn bị "ẩn" đi sau khi 1 Consumer lấy nó về xử lý (mặc định 30 giây). Nếu Consumer xử lý thất bại/sập trước khi xóa file, hết khoảng thời gian này tin nhắn sẽ xuất hiện lại trong Queue để Consumer khác lấy xử lý lại.
    + **Dead Letter Queue (DLQ):** Nếu một tin nhắn bị lỗi và xử lý lại quá số lần quy định (`ReceiveCount > maxReceiveCount`), nó sẽ được chuyển sang DLQ để kỹ sư vào kiểm tra sau, tránh làm tắc nghẽn Queue chính.
    + **Short Polling vs Long Polling:**
        * *Short Polling:* Trả về kết quả ngay lập tức (dù Queue rỗng) $\rightarrow$ Tốn nhiều request, đắt tiền.
        * *Long Polling (`WaitTimeSeconds > 0`, max 20s):* Đợi cho đến khi có tin nhắn mới trả về $\rightarrow$ **Tiết kiệm chi phí và giảm độ trễ tối đa**.

### 2. Amazon SNS (Simple Notification Service)
- **Khái niệm:** Dịch vụ truyền tin theo mô hình **Pub/Sub (Publisher / Subscriber)** dạng 1-to-Many (Gửi 1 thông điệp tới hàng ngàn người nhận cùng lúc).
- **Cơ chế:**
    + Publisher gửi tin nhắn vào một **Topic**.
    + Các Subscriber đăng ký Topic đó sẽ nhận được tin nhắn ngay lập tức.

- **Các loại Subscriber:** Email, SMS, Mobile Push Notification, HTTP/HTTPS Endpoints, **AWS Lambda**, và đặc biệt là **Amazon SQS**.
- **Mô hình kinh điển: Fan-out Pattern (SNS + SQS):**
    + Gửi 1 tin nhắn vào SNS Topic $\rightarrow$ SNS tự đẩy tin nhắn đó vào **nhiều SQS Queues khác nhau** cùng lúc. Mỗi SQS Queue lại có các tập hợp Worker riêng xử lý các nhiệm vụ song song. Ví dụ:   
        * Bài toán: Khi khách hàng đặt hàng thành công trên website, hệ thống cần làm 3 việc độc lập cùng lúc:
            * Trừ tiền / Tạo hóa đơn.
            * Gửi lệnh tới kho xuất hàng.
            * Gửi email xác nhận cho khách hàng.
        * Giải pháp Pub/Sub Fan-out:
            * App bán hàng (Publisher) chỉ cần gửi 1 event OrderCreated vào SNS Topic.
            * Tạo 3 SQS Queues tương ứng với 3 dịch vụ trên và cho 3 Queues này Subscribe vào SNS Topic.
            * SNS sẽ tự động nhân bản tin nhắn thành 3 bản và đẩy vào 3 SQS Queues. Mỗi SQS Queue có nhóm Worker riêng tự đọc và xử lý độc lập mà không ảnh hưởng lẫn nhau.

## II. ORCHESTRATION & EVENT BUS

### 3. Amazon EventBridge (Tên cũ: CloudWatch Events)
- **Khái niệm:** Dịch vụ **Event Bus** Serverless giúp kết nối các ứng dụng bằng dữ liệu sự kiện (Event-Driven Architecture).
- **Cách hoạt động:** Lắng nghe các sự kiện (Events) phát ra từ các dịch vụ AWS, ứng dụng SaaS (Zendesk, Datadog...), hoặc custom app $\rightarrow$ Khớp với các quy tắc (**Rules**) $\rightarrow$ Tự động kích hoạt **Target** (Lambda, Step Functions, SQS, SNS...).
- **Tính năng nổi bật:**
    + *Event Bus Rules:* Lọc sự kiện theo cấu trúc JSON.
    + *Scheduled Events:* Đóng vai trò như một **Cron job** trên Cloud (ví dụ: Chạy Lambda vào 12h đêm mỗi ngày).
    + *Schema Registry:* Tự động phân tích và lưu trữ cấu trúc file sự kiện.

### 4. AWS Step Functions
- **Khái niệm:** Dịch vụ điều phối (Orchestration) luồng công việc dưới dạng **Visual Workflows / State Machines**.
- **Tác dụng:** Dùng để chuỗi các **AWS Lambda Functions** và các dịch vụ khác thành một quy trình nghiệp vụ phức tạp (có các bước rẽ nhánh `Choice`, vòng lặp `Loop`, xử lý lỗi `Retry/Catch`, đợi phản hồi `Wait`).
- **Khi nào dùng?** Khi quy trình quá dài/phức tạp mà 1 hàm Lambda không thể làm hết (do Lambda bị giới hạn thời gian chạy tối đa 15 phút).

## III. XỬ LÝ DỮ LIỆU CHUỖI THỜI GIAN THỰC (REAL-TIME STREAMING)

### 5. Amazon Kinesis
Dùng cho bài toán thu thập, xử lý và phân tích dữ liệu luồng thời gian thực (Real-time Streaming Data) quy mô lớn (IoT logs, clickstreams, financial transactions).

- **Kinesis Data Streams (KDS):**
    + Thu thập dữ liệu luồng độ trễ cực thấp (milliseconds).
    + Dữ liệu được lưu trữ lại trong Stream từ **24 giờ đến 365 ngày** (có thể đọc đi đọc lại).
    + Mở rộng bằng cách tăng/giảm số lượng **Shards**.
- **Kinesis Data Firehose:**
    + Dùng để **Load (Đẩy) dữ liệu luồng trực tiếp vào các kho chứa** như Amazon S3, Redshift, OpenSearch, Splunk.
    + Serverless hoàn toàn (không có khái niệm Shards), tự động scale.
    + **Không lưu trữ dữ liệu** (Dữ liệu chui qua Firehose là biến mất, chỉ ghi vào đích).
- **Kinesis Data Analytics:** Dùng câu lệnh SQL hoặc Apache Flink để phân tích dữ liệu ngay trên luồng Kinesis Streams/Firehose đang chạy.

## IV. CÁC DỊCH VỤ TÍCH HỢP KHÁC

### 6. Amazon API Gateway
- **Khái niệm:** Dịch vụ Serverless giúp các nhà phát triển dễ dàng tạo, xuất bản, bảo trì, giám sát và bảo vệ các API (REST, HTTP, WebSocket) ở bất kỳ quy mô nào.
- **Vị trí trong kiến trúc:** Đóng vai trò là **"Cửa ngõ" (Front-Door)** REST/HTTP/WebSocket API nhận mọi request từ client (App/Web/IoT), sau đó điều hướng traffic tới các service phía sau (như **AWS Lambda**, **EC2**, **ECS**, hoặc gọi trực tiếp tới các **AWS Services khác**), tích hợp với Lambda, Cognito, WAF, hỗ trợ Throttling, Usage Plans, và Caching.

```
 Client (Web/Mobile) ──► [ Amazon API Gateway ] ──┬──► AWS Lambda (Serverless Backend)
                                                 ├──► EC2 / ECS / Load Balancer
                                                 └──► Direct AWS Service (Kinesis, DynamoDB...)
```

- **Các loại API trong API Gateway (CỰC HAY HỎI THI)**
    + **HTTP API:**
        * Mới hơn, tối ưu riêng cho RESTful API.
        * **Tốc độ nhanh hơn, chi phí rẻ hơn ~70%** so với REST API.
        * Hỗ trợ OIDC, OAuth2, CORS mặc định.

    + **REST API:**
        * Dành cho RESTful API cần các tính năng nâng cao: **API Keys**, **Usage Plans (Throttling/Quota)**, **WAF Integration**, **Request Validation**, **Transformation**.

    + **WebSocket API:**
        * Dành cho ứng dụng thời gian thực (Real-time 2-way communication) như Chat App, Dashboard chứng khoán, Game trực tuyến.

- **Authentication & Authorization**
    + **IAM Roles & Policies:** Dành cho người dùng/máy chủ *nội bộ trên AWS* gọi API.
    + **Amazon Cognito User Pools:** Dành cho *người dùng cuối (End-users)* đã đăng nhập vào App/Web (dùng OAuth2 / JWT Tokens).
    + **Lambda Authorizer (Custom Authorizer):** Dùng code Lambda để tự viết logic kiểm tra Token/Header (dùng khi tích hợp với bên thứ 3 như OAuth2 bên ngoài, SAML, Auth0).

- **Các tính năng quản lý Traffic & Cache**
    + **API Gateway Caching:**
        * Cache kết quả trả về của API để giảm số lượng request gọi vào Backend (Lambda/EC2).
        * Giúp tăng tốc phản hồi và giảm chi phí chạy Lambda.
    + **Throttling & Rate Limiting:**
        * Giới hạn số lượng request/giây (TPS - Transactions Per Second) để bảo vệ Backend không bị quá tải hay bị tấn công dồn dập.
        * *Sử dụng Usage Plans & API Keys* để phân hạng khách hàng (ví dụ: Tài khoản Free được 10 req/s, tài khoản VIP được 1000 req/s).
    + **API Versioning & Stages:**
        * Quản lý các môi trường (`dev`, `test`, `prod`) và các phiên bản API (`v1`, `v2`) dễ dàng.

- **Loại Endpoint Deployment (Deployment Types)**
    + **Edge-Optimized Endpoint:** Traffic chạy qua mạng lưới **CloudFront Edge Locations** toàn cầu của AWS trước khi chui vào API Gateway (Tối ưu cho người dùng truy cập từ khắp nơi trên thế giới).
    + **Regional Endpoint:** Dành cho người dùng tập trung ở cùng một Region với API Gateway (Có thể tự gắn CloudFront thủ công ở phía trước nếu muốn).
    + **Private Endpoint:** API chỉ truy cập được từ bên trong **AWS VPC** qua **VPC Endpoint (PrivateLink)**, hoàn toàn không mở ra Internet.

### 7. Amazon MQ
- **Khái niệm:** Dịch vụ tin nhắn quản lý hoàn toàn cho các giao thức chuẩn ngành như **ActiveMQ** và **RabbitMQ**.
- **Khi nào dùng?** Dùng khi muốn di chuyển (Migrate) các ứng dụng On-premises cũ (Legacy Applications) đang dùng chuẩn tin nhắn MQTT, AMQP, STOMP lên Cloud **mà không muốn sửa code để chuyển sang SQS/SNS**.

Bạn nói **chính xác 100%**! Mình nhận sai sót ở điểm này.

Trong cấu trúc chuẩn 13 Category mà chúng ta đã thống nhất từ đầu:

* **AWS Lambda, ECS, EKS, ECR, Fargate** thuộc về **CATEGORY 1: COMPUTE** (mở rộng mảng Container & Serverless).
* **API Gateway** và **Cognito** nằm ở **CATEGORY 13: FRONT-END WEB & MOBILE** (và một chân ở Category 6 / Category 5).

Việc mình xé lẻ *Serverless & Containers* thành một "Category 7" riêng ở câu trả lời trước là bị **lệch so với khung 13 Category chuẩn** của bạn. Cảm ơn bạn đã nhặt lỗi rất tinh!

---

Để đưa lộ trình về đúng quỹ đạo chuẩn 13 Category, tiếp theo Category 6 sẽ là:

# G. ANALYTICS & BIG DATA

Mảng này tập trung vào bài toán: *Làm sao để truy vấn, biến đổi, khai phá và hiển thị báo cáo từ lượng dữ liệu khổng lồ (Big Data) trên AWS?*

## I. QUERY & ETL

### 1. Amazon Athena
- **Khái niệm:** Dịch vụ truy vấn dữ liệu Serverless bằng câu lệnh **SQL chuẩn** trực tiếp trên các file nằm ở **Amazon S3**.
- **Đặc điểm cốt lõi:**
    + **Serverless:** Không cần bật/quản lý server, không cần load dữ liệu vào Database. Chỉ cần trỏ Athena tới S3 Bucket và chạy SQL `SELECT`.
    + **Tính phí:** Trả tiền dựa trên số lượng dữ liệu bị quét (Data Scanned).
- **Cách tối ưu chi phí (CỰC HAY HỎI THI):**
    1. Chuyển đổi định dạng file sang dạng cột (**Columnar Format**) như **Apache Parquet** hoặc **ORC** (giúp Athena chỉ đọc đúng cột cần thiết thay vì đọc toàn bộ file).
    2. Nén dữ liệu (GZIP, Snappy).
    3. Chia nhỏ folder theo ngày/tháng/vùng (**Partitioning** dữ liệu trên S3).

### 2. AWS Glue
- **Khái niệm:** Dịch vụ **ETL (Extract, Transform, Load)** hoàn toàn Serverless.
- **Các thành phần chính:**
    + **Glue Data Catalog:** Kho lưu trữ Metadata (chứa thông tin về Schema, bảng, cột của dữ liệu nằm ở S3, RDS, DynamoDB...). Đóng vai trò là Central Catalog cho Athena, Redshift Spectrum, EMR.
    + **Glue Crawlers:** Tự động quét các nguồn dữ liệu (như S3) để phân tích định dạng và tự động tạo Schema lưu vào Glue Data Catalog.
    + **Glue ETL Jobs:** Chạy các script (Python/PySpark) để biến đổi, làm sạch dữ liệu và đẩy sang kho chứa khác.

## II. BIG DATA PROCESSING

### 3. Amazon EMR (Elastic MapReduce)
- **Khái niệm:** Dịch vụ quản lý cụm máy chủ (Cluster) chuyên chạy các framework Big Data mã nguồn mở như **Apache Spark, Hadoop, Hive, Presto, HBase**.
- **Khi nào dùng?** Khi bạn có các bài toán tính toán Big Data cực lớn, machine learning, hoặc muốn di chuyển (Migrate) cụm Hadoop/Spark sẵn có từ On-premises lên AWS.
- **Thành phần cụm EMR:**
    + *Master Node:* Quản lý cụm.
    + *Core Node:* Chạy nhiệm vụ và lưu trữ dữ liệu (HDFS).
    + *Task Node:* Chỉ chạy tính toán (không lưu trữ), có thể dùng **EC2 Spot Instances** ở đây để tiết kiệm 90% chi phí.

### 4. Amazon OpenSearch Service (Tên cũ: Amazon Elasticsearch Service)
- **Khái niệm:** Dịch vụ quản lý tìm kiếm và phân tích log thời gian thực.
- **Tác dụng:** Chuyên dùng cho bài toán **Tìm kiếm văn bản (Text Search)**, **Phân tích Log (Application Logging)** và hiển thị Dashboard giám sát (qua OpenSearch Dashboards / Kibana).

## III. STREAMING & BI

### 5. Amazon MSK (Managed Streaming for Apache Kafka)
- **Khái niệm:** Dịch vụ Apache Kafka do AWS quản lý hoàn toàn.
- **So sánh MSK vs Kinesis:**
    + Dùng **Kinesis** nếu muốn dịch vụ Native của AWS, hoàn toàn Serverless, tích hợp sẵn với các AWS Services khác.
    + Dùng **MSK** nếu ứng dụng hiện tại On-premises đã viết sẵn cho **Apache Kafka mã nguồn mở** và muốn Bê-nguyên-xi (Lift-and-Shift) lên Cloud mà không cần sửa code.

### 6. Amazon QuickSight
- **Khái niệm:** Dịch vụ tạo Báo cáo, Biểu đồ trực quan hóa dữ liệu (**Business Intelligence - BI**) Serverless do AWS cung cấp (Tương tự PowerBI hay Tableau).
- **Tính năng:** Tích hợp trực tiếp với Athena, Redshift, S3, RDS... Sử dụng công nghệ **SPICE** (Super-fast, Parallel, In-memory Calculation Engine) để load biểu đồ siêu nhanh.

# H. MANAGEMENT, GOVERNANCE & MONITORING

## I. GIÁM SÁT VÀ GHI LOG (MONITORING & LOGGING)

### 1. Amazon CloudWatch
- **Khái niệm:** Dịch vụ giám sát hiệu năng (Performance Monitoring) và ghi nhận nhật ký hoạt động (Logging) cho toàn bộ tài nguyên trên AWS.
- **Các thành phần cốt lõi (BẮT BUỘC THUỘC LÒNG):**
    + **CloudWatch Metrics:** Theo dõi các chỉ số hiệu năng (CPUUtilization, NetworkIn/Out, DiskReadOps...).
        * *Mặc định:* Thu thập Metric mỗi 5 phút (Basic) hoặc 1 phút (Detailed Monitoring).
        * *Lưu ý quan trọng:* CloudWatch **KHÔNG thu thập mặc định** các chỉ số bên trong OS như: **RAM usage (Memory), Swap Space, Free Disk Space**. Muốn có các chỉ số này, bạn phải cài **CloudWatch Agent** lên EC2.
    + **CloudWatch Logs:** Tập trung lưu trữ log từ EC2, Lambda, ECS, Route 53...
    + **CloudWatch Alarms:** Đặt ngưỡng cảnh báo (ví dụ: CPU > 80% trong 5 phút).
        * *Hành động kích hoạt (Actions):* Gửi thông báo SNS, kích hoạt **EC2 Auto Scaling** (thêm/bớt máy chủ), hoặc tự động **Stop/Terminate/Reboot EC2 Instance**.
    + **CloudWatch Synthetics / Canaries:** Chạy các kịch bản test tự động định kỳ để kiểm tra xem API/Website của bạn có đang hoạt động tốt hay không.

### 2. AWS CloudTrail
- **Khái niệm:** Dịch vụ ghi lại toàn bộ **lịch sử cuộc gọi API / nhật ký hành vi (Audit Trail / API Logging)** xảy ra trong tài khoản AWS.
- **Công dụng:** Giúp trả lời các câu hỏi bảo mật: *Ai (Who) đã làm gì (What), ở đâu (Where), lúc nào (When) và bằng IP nào (From)?*
- **Phân biệt CloudWatch vs CloudTrail (Kinh điển trong đề thi):**
    + **CloudWatch:** Giám sát **Hiệu năng & Sức khỏe** hệ thống (*"Server có bị quá tải CPU không?"*).
    + **CloudTrail:** Giám sát **Hành vi & Sự thay đổi** qua API (*"Ai đã bấm nút xóa S3 Bucket này?"*).

- *Tính năng:* **CloudTrail Insights** dùng AI để tự động phát hiện các hành vi bất thường (như lượng call API tăng vọt đột biến).

## II. CONFIGURATION & OPERATIONS

### 3. AWS Config
- **Khái niệm:** Dịch vụ ghi lại lịch sử thay đổi cấu hình tài nguyên và kiểm tra mức độ tuân thủ quy định bảo mật (**Compliance & Auditing**).
- **Tác dụng:**
    + Cho biết trạng thái cấu hình của tài nguyên thay đổi như thế nào theo thời gian.
    + **AWS Config Rules:** Đặt quy tắc kiểm tra tự động (ví dụ: Quy định tất cả S3 Bucket không được mở Public, tất cả EBS phải mã hóa). Nếu phát hiện vi phạm, AWS Config sẽ báo lỗi `NON_COMPLIANT` và có thể tự kích hoạt quy trình sửa lỗi tự động (**Auto-remediation** qua SSM Automation).

### 4. AWS Systems Manager (SSM)
- **Khái niệm:** Trung tâm quản lý vận hành (Operational Hub) cho máy chủ (EC2 và On-premises) ở quy mô lớn.
- **Các tính năng nổi bật hay hỏi thi:**
    + **SSM Session Manager:** Cho phép SSH / RDP an toàn vào EC2 Instance **mà không cần mở Port 22 / 3389**, không cần SSH Key, không cần Bastion Host và không cần Public IP.
    + **SSM Run Command:** Chạy lệnh bash/powershell hàng loạt trên hàng trăm máy chủ cùng lúc.
    + **SSM Patch Manager:** Tự động quét và cài đặt bản vá bảo mật (OS Patches) cho tập hợp máy chủ.
    + **SSM Parameter Store:** Kho lưu trữ cấu hình/mật khẩu dạng Plaintext hoặc Mã hóa (đã nói ở Cat 5).

## III. ORGANIZATIONAL GOVERNANCE

### 5. AWS Control Tower
- **Khái niệm:** Dịch vụ giúp dựng nhanh một môi trường đa tài khoản AWS an toàn, chuẩn chỉnh theo Best Practices (gọi là **Landing Zone**).
- **Đặc điểm:** Tự động thiết lập AWS Organizations, áp dụng các quy tắc bảo mật bắt buộc (**Guardrails** - dạng Preventative/SCP hoặc Detective/AWS Config Rules).

### 6. AWS Service Catalog
- Dịch vụ cho phép doanh nghiệp tạo ra danh mục các dịch vụ/mô hình AWS đã được phê duyệt sẵn (IT-approved services) để các nhân viên/đội ngũ phát triển tự vào chọn và khởi tạo mà không vi phạm chính sách công ty.

### 7. AWS Health Dashboard
- **Service Health Dashboard:** Báo cáo trạng thái hoạt động chung của toàn bộ các dịch vụ AWS trên tất cả Region (dành cho cộng đồng).
- **Personal Health Dashboard (PHD / AWS Health):** Báo cáo trạng thái riêng liên quan trực tiếp đến các tài nguyên **đang chạy trong tài khoản của BẠN** (ví dụ: cảnh báo EC2 instance của bạn đang nằm trên phần cứng bị lỗi cần bảo trì).

# I. MIGRATION & TRANSFER

## I. CHIẾN LƯỢC MIGRATION (6 Rs OF MIGRATION)

Trong đề thi SAA-C03, bạn sẽ hay gặp các kịch bản yêu cầu chọn chiến lược dịch chuyển phù hợp:

1. **Rehost ("Lift-and-Shift"):** Bê nguyên xi máy chủ ảo/vật lý lên EC2 mà không sửa đổi ứng dụng. Nhanh nhất, tốn ít công sức nhất.
2. **Replatform ("Lift-tinker-and-shift"):** Đưa ứng dụng lên Cloud và thực hiện một vài tối ưu nhỏ mà không đổi kiến trúc cốt lõi (ví dụ: Chuyển DB tự cài trên EC2 sang dùng **Amazon RDS** quản lý hoàn toàn).
3. **Refactor / Re-architect:** Viết lại hoàn toàn ứng dụng để tận dụng tối đa các dịch vụ Native Cloud (Serverless, Microservices, DynamoDB...).
4. **Repurchase ("Drop-and-Shop"):** Bỏ ứng dụng cũ, chuyển sang dùng giải pháp SaaS mới (ví dụ: Chuyển CRM tự làm sang Salesforce).
5. **Retain:** Giữ lại một số ứng dụng ở On-premises do yêu cầu tuân thủ pháp lý hoặc chưa sẵn sàng chuyển đổi.
6. **Retire:** Tắt/bỏ hẳn các ứng dụng không còn sử dụng.

## II. SYSTEM & DATABASE MIGRATION

### 1. AWS Application Migration Service (AWS MGN)
- **Khái niệm:** Dịch vụ chuyển đổi máy chủ (Server Migration) chủ đạo của AWS.
- **Cách hoạt động:** Cài một Agent nhẹ lên máy chủ On-premises (vật lý hoặc ảo hóa VMware/Hyper-V) $\rightarrow$ Tự động sao chép từng block dữ liệu (Block-level replication) liên tục lên AWS $\rightarrow$ Chuyển đổi thành EC2 Instance.
- **Ưu điểm:** **Downtime cực thấp** (chỉ vài phút để thực hiện Cutover chính thức).

### 2. AWS Database Migration Service (AWS DMS)
- **Khái niệm:** Dịch vụ dịch chuyển Cơ sở dữ liệu lên AWS an toàn, tin cậy và liên tục.
- **Đặc điểm quan trọng:** Cơ sở dữ liệu nguồn (Source DB) **vẫn hoạt động bình thường và xử lý giao dịch trong suốt quá trình migration**.
- **2 Kịch bản chuyển đổi:**
    + *Homogeneous Migration (Cùng loại DB):* Ví dụ từ Oracle On-prem sang Oracle RDS.
    + *Heterogeneous Migration (Khác loại DB):* Ví dụ từ Oracle sang Amazon Aurora / PostgreSQL.

- **AWS Schema Conversion Tool (AWS SCT):** Khi chuyển đổi **Khác loại DB**, bắt buộc dùng AWS SCT trước để chuyển đổi Schema, Stored Procedures, Views từ DB nguồn sang cấu trúc của DB đích, sau đó mới dùng DMS để di chuyển dữ liệu.

## III. DATA TRANSFER

Khi dung lượng dữ liệu cần chuyển lên AWS lên tới hàng Terabytes (TB) hoặc Petabytes (PB), việc truyền qua đường truyền Internet truyền thống sẽ mất nhiều tháng hoặc nhiều năm.

### 3. AWS Snow Family (Chuyển dữ liệu Offline bằng phần cứng)
AWS cung cấp các thiết bị phần cứng vật lý siêu bền, bảo mật cao để gửi tới văn phòng/datacenter của bạn, bạn copy dữ liệu vào rồi gửi ship về AWS để nạp vào S3:

- **AWS Snowcone:** Thiết bị nhỏ gọn (~2.1 kg), lưu trữ tối đa **8 TB** (có bản SSD 14 TB).
- **AWS Snowball Edge:** Thiết bị dạng vali chống va đập:
- *Snowball Edge Storage Optimized:* Dung lượng lưu trữ cực lớn (~**80 TB** usable).
- *Snowball Edge Compute Optimized:* Dung lượng ~40 TB kèm sức mạnh tính toán (vCPU/RAM) để xử lý dữ liệu ngay tại chỗ (Edge Computing).
- **AWS Snowmobile:** Xe container 18 bánh khổng lồ, chuyển tới **100 PB** (Petabytes) dữ liệu cho các đại dự án.
- *Mẹo thi:* Nếu dung lượng lớn và mất **trên 1 tuần** để upload qua mạng $\rightarrow$ Hãy chọn **Snowball Edge**.

### 4. AWS DataSync (Chuyển dữ liệu Online tự động)
- **Khái niệm:** Dịch vụ đẩy dữ liệu online tốc độ cao giữa hạ tầng On-premises (NFS, SMB, HDFS) và các dịch vụ lưu trữ AWS (**S3, EFS, FSx**).
- **Đặc điểm:** Tối ưu hóa đường truyền mạng, nhanh gấp **10 lần** so với các công cụ truyền file thông thường, tự động mã hóa và kiểm tra tính toàn vẹn của file.

### 5. AWS Transfer Family
- Dịch vụ giúp chuyển file trực tiếp vào/ra khỏi Amazon S3 hoặc Amazon EFS thông qua các giao thức truyền file truyền thống: **SFTP, FTPS, và FTP**.

# J. DEVELOPER TOOLS & CI/CD

## I. AWS CI/CD NATIVE TOOLS (AWS CODE SUITE)

### 1. AWS CodeCommit
- **Khái niệm:** Dịch vụ quản lý mã nguồn (Source Control) hoàn toàn quản lý, dựa trên chuẩn **Git**.
- **Đặc điểm:** Tương tự GitHub / GitLab, nhưng dữ liệu mã nguồn được lưu trữ an toàn trong hạ tầng AWS và tích hợp sẵn với IAM để phân quyền truy cập.

### 2. AWS CodeBuild
- **Khái niệm:** Dịch vụ biên dịch mã nguồn và chạy các bài kiểm thử tự động (Automated Testing) hoàn toàn Serverless.
- **Cách hoạt động:**
    + Tự động khởi tạo một Container tạm thời, kéo code về, biên dịch thành các file chạy được (Artifacts).
    + Cấu hình các bước build thông qua file **`buildspec.yml`** đặt tại thư mục gốc của code.
    + Scale tự động, bạn chỉ trả tiền theo từng phút chạy build.

### 3. AWS CodeDeploy
- **Khái niệm:** Dịch vụ tự động hóa quá trình triển khai (Deploy) ứng dụng lên các môi trường tính toán như **EC2, AWS Fargate, AWS Lambda**, hoặc ngay cả máy chủ On-premises.
- **Cấu hình:** Sử dụng file cấu hình **`appspec.yml`**.
- **Các chiến lược Deployment quan trọng (CỰC HAY HỎI THI):**
    + **In-Place Deployment:** Dừng ứng dụng cũ trên từng EC2 $\rightarrow$ Cài bản mới $\rightarrow$ Chạy lại. (Tốn chút ít downtime, tiết kiệm chi phí vì không cần tạo EC2 mới).
    + **Blue/Green Deployment:** Tạo ra một tập hợp máy chủ mới hoàn toàn (Green) chạy bản code mới bên cạnh hệ thống cũ (Blue). Sau khi kiểm tra thành công, đổi hướng traffic từ Blue sang Green (qua Load Balancer hoặc Route 53). **Zero-downtime** và cho phép **Rollback tức thì** nếu có lỗi.
    + **Canary Deployment (Dành cho Lambda/ECS):** Chuyển một lượng nhỏ traffic (ví dụ 10%) sang bản code mới để theo dõi, nếu ổn định mới chuyển nốt 90% còn lại.

### 4. AWS CodePipeline
- **Khái niệm:** Dịch vụ điều phối toàn bộ quy trình phát triển phần mềm (CI/CD Workflow Orchestration).
- **Tác dụng:** Đóng vai trò như "nhạc trưởng" kết nối các khâu:

$$\text{Source (CodeCommit/GitHub)} \longrightarrow \text{Build (CodeBuild)} \longrightarrow \text{Deploy (CodeDeploy)}$$



---

## II. INFRASTRUCTURE AS CODE - IaC

### 5. AWS CloudFormation
- **Khái niệm:** Công cụ giúp khai báo và khởi tạo toàn bộ bộ khung hạ tầng AWS (VPC, EC2, RDS, S3...) tự động thông qua file cấu hình (định dạng **JSON** hoặc **YAML**).
- **Các thuật ngữ cốt lõi:**
    + **Template:** File mã nguồn mô tả các tài nguyên AWS cần khởi tạo.
    + **Stack:** Tập hợp các tài nguyên AWS được tạo ra từ 1 Template (Tạo/Xóa toàn bộ môi trường chỉ bằng 1 cú click).
    + **Change Sets:** Cho phép xem trước các thay đổi đối với tài nguyên trước khi áp dụng thực tế (tránh rủi ro ghi đè/xóa nhầm DB).
    + **StackSets:** Cho phép triển khai 1 Template CloudFormation lên **nhiều Tài khoản AWS (Multi-Account)** và **nhiều Vùng (Multi-Region)** cùng lúc chỉ bằng một thao tác.

### 6. AWS Cloud Development Kit (AWS CDK)
- **Khái niệm:** Công cụ cho phép các lập trình viên định nghĩa hạ tầng AWS bằng các ngôn ngữ lập trình quen thuộc (TypeScript, Python, Java, C#) thay vì viết JSON/YAML.
- **Cơ chế:** Code CDK sau đó sẽ tự động dịch (Synthesize) ra file CloudFormation Template để triển khai lên AWS.

# K. COST MANAGEMENT

## I. PAYMENT MODELS

Đề thi SAA-C03 rất hay hỏi việc chọn hình thức trả tiền phù hợp cho EC2, RDS, DynamoDB... dựa trên đặc tính của ứng dụng:

### 1. EC2 Pricing Models (Thuộc lòng cho bài thi)

- **On-Demand:** Trả tiền theo từng giây/giờ sử dụng thực tế.
    + *Khi nào dùng:* Ứng dụng mới ra mắt, tải không dự đoán được, test ngắn hạn, không thể gián đoạn.

- **Reserved Instances (RI):** Cam kết sử dụng dịch vụ cố định trong **1 năm hoặc 3 năm** (Giảm tới 72% so với On-Demand).
    + *Standard RI:* Giảm giá sâu nhất, cố định thuộc tính (Region, OS, Instance Type).
    + *Convertible RI:* Cho phép đổi loại máy chủ (Instance Family) nếu nhu cầu thay đổi.
    + *Khi nào dùng:* Tải ổn định, chạy 24/7 cố định (ví dụ: Core Database, Production Web Server).

- **Savings Plans:** Cam kết chi tiêu **X USD/giờ** trong 1 hoặc 3 năm (Linh hoạt hơn RI).
    + *Compute Savings Plans:* Áp dụng cho cả **EC2, Fargate, và Lambda** không phân biệt Region hay OS (Giảm tới 66%).

- **Spot Instances:** Đặt mua dung lượng phần cứng nhàn rỗi của AWS (Giảm tới **90%** chi phí!).
    + *Đặc điểm sinh tử:* AWS có thể lấy lại máy chủ bất kỳ lúc nào và chỉ báo trước **2 phút** (2-minute warning notification).
    + *Khi nào dùng:* Tải chịu được ngắt quãng (Interruptible workloads), Batch processing, Big Data (EMR Task Nodes), CI/CD Jobs.

- **Dedicated Hosts / Dedicated Instances:** Thuê riêng máy chủ vật lý.
    + *Khi nào dùng:* Yêu cầu tuân thủ bảo mật khắt khe hoặc cần dùng lại giấy phép phần mềm cũ (BYOL - Bring Your Own License). Đắt nhất.

## II. COST ANALYSIS & BUDGETING

### 2. AWS Organizations & Consolidated Billing
- **Consolidated Billing:** Gom hóa đơn của tất cả các tài khoản con (Member Accounts) về một tài khoản chính (Payer Account).
- **Lợi ích:** Được hưởng mức chiết khấu theo dung lượng (Volume Discounts) cho cả S3 và EC2 nhờ gộp chung lưu lượng sử dụng của toàn bộ công ty.

### 3. AWS Cost Explorer
- **Khái niệm:** Công cụ trực quan hóa (Visual Tool) cho phép phân tích, báo cáo và dự báo (Forecast) chi phí AWS trong quá khứ và tương lai.
- **Tác dụng:** Giúp trả lời câu hỏi: *Dịch vụ nào đang tốn tiền nhất? Chi phí tháng sau dự kiến là bao nhiêu?*

### 4. AWS Budgets
- **Khái niệm:** Công cụ đặt **ngưỡng ngân sách** (Budget Limits) và phát cảnh báo.
- **Cơ chế cảnh báo (Alerts):** Gửi Email / SNS Notification khi chi phí thực tế **hoặc chi phí dự báo (Forecasted cost)** vượt quá ngưỡng cho phép (ví dụ: Cảnh báo khi hóa đơn vượt $1000/tháng).
- **AWS Budgets Actions:** Tự động thực thi hành động (như gắn SCP cấm tạo EC2 mới) nếu chi phí vượt ngưỡng.

### 5. AWS Cost Anomaly Detection
- Sử dụng Machine Learning để liên tục giám sát và tự động phát hiện các khoản chi phí **tăng vọt bất thường** (ví dụ: Do lỡ tay bật nhầm máy chủ lớn hoặc bị dính mã độc).

## III. COST OPTIMIZATION TOOLS

### 6. AWS Compute Optimizer
- **Khái niệm:** Dịch vụ dùng Machine Learning để phân tích dữ liệu lịch sử sử dụng (CPU, Memory, Disk...) của EC2, Auto Scaling Groups, EBS volumes, và Lambda.
- **Tác dụng:** Đưa ra khuyến nghị **Rightsizing** (Tăng/giảm/đổi loại máy chủ) để bạn không mua thừa dung lượng.

### 7. AWS Trusted Advisor
- **Khái niệm:** Dịch vụ đóng vai trò như một "chuyên gia tư vấn ảo" tự động kiểm tra tài khoản AWS của bạn theo 5 khía cạnh Best Practices:
    1. *Cost Optimization* (Phát hiện các EBS Volume không gắn vào EC2, Idle Load Balancers, Unassociated Elastic IPs).
    2. *Performance*
    3. *Security* (Phát hiện S3 công khai, thiếu MFA).
    4. *Fault Tolerance*
    5. *Service Limits*

# L. DISASTER RECOVERY & BUSINESS CONTINUITY

## I. HAI CHỈ SỐ CỐT LÕI TRONG DISASTER RECOVERY (DR)

Mọi bài toán DR trong bài thi đều xoay quanh 2 chỉ số này để đo lường mức độ chịu đựng rủi ro của doanh nghiệp:

1. **RPO (Recovery Point Objective - Mục tiêu điểm khôi phục):**
    * *Ý nghĩa:* Mức độ **mất mát dữ liệu tối đa** có thể chấp nhận được (tính bằng thời gian).
    * *Câu hỏi đặt ra:* *"Từ thời điểm hệ thống bị sập lùi về quá khứ, chúng ta chấp nhận mất dữ liệu trong bao lâu?"*
    * *Ví dụ:* RPO = 5 phút nghĩa là dữ liệu phải được backup liên tục, nếu sập chỉ được phép mất tối đa 5 phút dữ liệu gần nhất.


2. **RTO (Recovery Time Objective - Mục tiêu thời gian khôi phục):**
    * *Ý nghĩa:* Mức độ **downtime/thời gian gián đoạn tối đa** có thể chấp nhận được để khôi phục hệ thống hoạt động trở lại.
    * *Câu hỏi đặt ra:* *"Hệ thống bị sập thì mất bao lâu để dựng lại hoàn toàn cho người dùng truy cập?"*

## II. 4 STRATEGIES KHÔI PHÚC THẢM HỌA TRÊN AWS (TỪ RẺ/CHẬM ĐẾN ĐẮT/NHANH)

Đề thi sẽ đưa ra ngân sách và yêu cầu RTO/RPO để bạn chọn 1 trong 4 chiến lược kinh điển sau:

```text
Chi phí tăng dần -------------------------------------------------------->
[ Backup & Restore ] ---> [ Pilot Light ] ---> [ Warm Standby ] ---> [ Multi-Site Active-Active ]
RTO/RPO lớn nhất (giờ/ngày)                                          RTO/RPO gần như = 0
```

### 1. Backup & Restore (Chi phí thấp nhất - RTO/RPO cao nhất)
- **Cách hoạt động:** Sao lưu dữ liệu (Snapshots, S3 Backups) định kỳ từ Region chính sang Region phụ. Không chạy bất kỳ máy chủ nào ở Region phụ.
- **Khi xảy ra thảm họa:** Khởi tạo hạ tầng từ đầu ở Region phụ bằng CloudFormation/AMIs và restore dữ liệu từ backup.
- **Thời gian khôi phục (RTO):** Tính bằng **giờ hoặc ngày**.

### 2. Pilot Light (Đèn báo sự cố)
- **Cách hoạt động:** Dữ liệu cốt lõi (Database) luôn được **đồng bộ liên tục** sang Region phụ (ví dụ: RDS Cross-Region Read Replica). Các thành phần tính toán (EC2/Auto Scaling) được cấu hình sẵn nhưng ở trạng thái **TẮT** hoặc số lượng = 0.
- **Khi xảy ra thảm họa:** Bật các máy chủ EC2 lên, scale out hệ thống và promote Database ở Region phụ lên làm Master.
- **Thời gian khôi phục (RTO):** Tính bằng **vài chục phút**.

### 3. Warm Standby (Bản sao thu nhỏ)
- **Cách hoạt động:** Dựng một phiên bản **hoàn chỉnh nhưng ở quy mô tối thiểu (Minimal size)** của toàn bộ hệ thống tại Region phụ (ví dụ: 1 EC2 instance chạy 24/7 đứng sau Load Balancer).
- **Khi xảy ra thảm họa:** Chỉ cần Scale Out số lượng EC2 ở Region phụ lên đủ tải và chuyển DNS qua Route 53.
- **Thời gian khôi phục (RTO):** Tính bằng **vài phút**.

### 4. Multi-Site Active-Active (RTO/RPO gần như bằng 0 - Chi phí cao nhất)
- **Cách hoạt động:** Toàn bộ hệ thống chạy full công suất song song ở **2 hoặc nhiều Region cùng lúc**.
- **Traffic:** Route 53 phân phối tải (Latency-based hoặc Geolocation) tới cả 2 Region. Nếu 1 Region sập, Route 53 Health Check tự động loại bỏ Region lỗi, người dùng không nhận ra sự cố.
- **Thời gian khôi phục (RTO/RPO):** gần như **bằng 0 (Real-time)**.

## III. DỊCH VỤ SAO LƯU VÀ KHÔI PHÚC CHUYÊN DỤNG CỦA AWS

### 1. AWS Backup
- **Khái niệm:** Dịch vụ quản lý sao lưu tập trung (Centralized Backup Management) tự động hóa quy trình sao lưu trên nhiều dịch vụ AWS (EBS, RDS, DynamoDB, EFS, S3, FSx...).
- **Tính năng quan trọng:**
    + Cho phép đặt lịch sao lưu tự động (Backup Plans) và chính sách lưu trữ (Retention Policy).
     Hỗ trợ **Cross-Region & Cross-Account Backup** (sao lưu sang tài khoản AWS khác để chống Ransomware/Xóa nhầm).
    + **AWS Backup Vault Lock:** Khóa bản backup để không ai (kể cả Root account) có thể xóa hoặc sửa đổi trong thời gian quy định (Tuân thủ chuẩn WORM - Write Once, Read Many).

### 2. AWS Elastic Disaster Recovery (AWS DRS)
- **Khái niệm:** Dịch vụ DR chuyên dụng giúp khôi phục máy chủ (Vật lý, Virtual Machines, hoặc EC2) sang AWS sử dụng cơ chế sao lưu liên tục ở cấp độ Block (Block-level replication).
- **Ứng dụng:** Giúp giảm thiểu chi phí khôi phục thảm họa cho máy chủ On-premise lên AWS mà vẫn đảm bảo RPO tính bằng giây và RTO tính bằng phút.

# M. FRONT-END WEB & MOBILE

## I. AUTHENTICATION & USER MANAGEMENT

### 1. Amazon Cognito
- **Khái niệm:** Dịch vụ quản lý đăng ký, đăng nhập và phân quyền hoàn toàn Serverless dành riêng cho **người dùng cuối (End-Users)** của ứng dụng Web/Mobile.
- **2 Thành phần cốt lõi (CỰC HAY HỎI THI):**
    + **User Pools (Xác thực - Authentication):**
        * Đóng vai trò là danh bạ/Cơ sở dữ liệu người dùng (User Directory).
        * Xử lý việc Đăng ký, Đăng nhập, Quên mật khẩu, Xác thực 2 yếu tố (MFA).
        * Hỗ trợ **Federated Identity / Social Login**: Cho phép người dùng đăng nhập qua Google, Facebook, Apple, hoặc SAML/OIDC của doanh nghiệp. Trả về cho Client một chuỗi **JWT Token**.
    + **Identity Pools / Federated Identities (Phân quyền - Authorization):**
        * Quy đổi JWT Token (hoặc người dùng ẩn danh) thành **Quyền tạm thời của AWS (Temporary AWS IAM Credentials)**.
        * Giúp ứng dụng phía Client có thể **upload/download file trực tiếp từ Amazon S3** hoặc **ghi dữ liệu thẳng vào DynamoDB** mà không cần phải đi qua một Server trung gian.

## II. API & DATA SYNC

### 2. Amazon API Gateway (Chân Front-End)
*(Dịch vụ này đứng ở ranh giới giữa Cat 6 và Cat 13)*:
- **Vai trò ở Front-End:** Đóng vai trò là điểm tiếp nhận duy nhất (Single Point of Entry) cho mọi request HTTP/REST/WebSocket từ Web/Mobile Client gửi lên.
- **Sử dụng với Cognito:** Tích hợp trực tiếp **Cognito User Pools Authorizer** để chặn và kiểm tra JWT Token ngay tại API Gateway trước khi cho phép request đi vào Lambda hay EC2 backend.

### 3. AWS AppSync
- **Khái niệm:** Dịch vụ quản lý API dạng **GraphQL** hoàn toàn Serverless.
- **So sánh REST API (API Gateway) vs GraphQL (AppSync) trong bài thi:**
    + *API Gateway (REST):* Mỗi request gọi đến 1 Endpoint cụ thể, dữ liệu trả về có thể bị thừa hoặc thiếu (Over-fetching / Under-fetching).
    + *AppSync (GraphQL):* Client chỉ cần gửi **1 request duy nhất** và chỉ định chính xác các trường dữ liệu (fields) cần lấy. Giúp tiết kiệm tối đa băng thông cho thiết bị di động.

- **Tính năng ăn tiền của AppSync:**
    + **Real-time Data Sync:** Tự động đồng bộ dữ liệu theo thời gian thực tới hàng triệu Client qua giao thức WebSocket (dùng tính năng **GraphQL Subscriptions**).
    + **Offline Data Access:** Cho phép app di động thao tác dữ liệu khi mất mạng (Offline), tự động đồng bộ/giải quyết xung đột (Conflict Resolution) khi có mạng trở lại.

## III. DEVELOPMENT FRAMEWORK

### 4. AWS Amplify
- **Khái niệm:** Bộ công cụ (CLI, Client SDK, UI Components) và dịch vụ Hosting giúp các Lập trình viên Front-End/Mobile xây dựng và deploy ứng dụng Full-Stack trên AWS một cách cực kỳ nhanh chóng mà không cần kiến thức chuyên sâu về Cloud Infrastructure.
- **2 Thành phần chính:**
    + **Amplify Studio / Framework:** Cho phép tạo backend (Cognito, DynamoDB, S3, API Gateway) chỉ bằng vài dòng lệnh CLI hoặc thao tác kéo thả trên UI.
    + **Amplify Hosting:** Dịch vụ quản lý Hosting hoàn toàn Serverless cho Static Website và SSR Apps (React, Vue, Next.js, iOS, Android...). Hỗ trợ tự động deploy từ GitHub (CI/CD), tích hợp sẵn CDN toàn cầu và cấp SSL miễn phí.

---

# SUMMARY
**Bộ Kim Chỉ Nam & Khung Tư Duy Kiến Trúc** tập trung vào **mẫu hình đề thi (Exam Patterns), thuật toán loại trừ đáp án, và cách ghép 13 Categories thành các bức tranh kiến trúc thực tế.**

# PHẦN 1: BỨC TRANH KIẾN TRÚC TỔNG HỢP (THỰC TẾ & TÌNH HUỐNG)

Trừ các câu hỏi đơn lẻ, đa số câu hỏi tình huống trong đề thi SAA-C03 đều là việc ghép nối các mảnh ghép từ 13 Categories vào **3 Mô hình Kiến trúc Chuẩn** dưới đây:

```text
[MÔ HÌNH 1: SERVERLESS FULL-STACK]
User (Web/App) 
  ──> CloudFront (Edge) + S3 (Static Content)
  ──> Cognito (Auth) ──> API Gateway (REST) / AppSync (GraphQL)
  ──> AWS Lambda (Compute)
  ──> DynamoDB (NoSQL) / Aurora Serverless (Relational)
  ──> SQS/SNS/EventBridge (Decoupling) ──> Step Functions (Orchestration)

[MÔ HÌNH 2: TRADITIONAL HIGH-AVAILABILITY WEB APP]
User 
  ──> Route 53 (DNS) ──> CloudFront (CDN) ──> WAF (Security)
  ──> ALB (Public Subnet) 
  ──> EC2 Auto Scaling Group (Private Subnet - Multi-AZ)
  ──> Multi-AZ RDS / Aurora (Private Subnet) + ElastiCache (Caching)
  ──> Shared Storage: EFS (Linux) hoặc FSx (Windows)

[MÔ HÌNH 3: HYBRID & DATA ANALYTICS PIPELINE]
On-Premises Datacenter ──(Direct Connect / VPN)──> AWS VPC
Data Source ──> Kinesis Data Streams / MSK (Real-time Streaming)
            ──> S3 Data Lake (Storage) ──> AWS Glue (ETL Catalog)
            ──> Athena (Ad-hoc Query) / EMR (Big Data) / Redshift (Data Warehouse)
            ──> QuickSight (Visualization)

```

# PHẦN 2: BẢN ĐỒ TỰ VỰNG - NHẬN DIỆN DOMAIN QUA KEYWORDS

Khi đọc đề, **Keyword chính trong yêu cầu của đề thi** sẽ quyết định bạn đang ở Domain nào và tiêu chí chọn đáp án là gì:

```text
                     ┌── "least operational overhead" / "serverless" / "managed" 
                     │    └─> Domain 1 / Domain 2 (Tối ưu vận hành)
                     │
                     ├── "resilient" / "high availability" / "multi-AZ" / "RTO/RPO"
                     │    └─> Domain 1: Design Resilient Architectures (26%)
                     │
KEYWORD TRONG ĐỀ ────┼── "encrypt" / "least privilege" / "compliance" / "audit"
                     │    └─> Domain 2: Design Secure Architectures (30%)
                     │
                     ├── "low latency" / "high throughput" / "caching" / "HPC"
                     │    └─> Domain 3: Design High-Performing Architectures (24%)
                     │
                     └── "most cost-effective" / "minimize cost" / "unattached"
                          └─> Domain 4: Design Cost-Optimized Architectures (20%)

```


# PHẦN 3: BẢNG NGUYÊN TẮC VÀNG TỪNG DOMAIN (CORE KEYNOTES)

### DOMAIN 1: DESIGN RESILIENT ARCHITECTURES (26%)

- **Multi-AZ luôn đánh bại Single-AZ:** Bất kỳ kiến trúc nào chỉ nằm trên 1 Availability Zone đều **SAI** nếu đề hỏi "High Availability" hoặc "Fault Tolerant".
- **Decoupling (Phân rã hệ thống):** Khi đề cập tới "spike in traffic", "system overload", "loose coupling" $\rightarrow$ Bắt buộc chọn **SQS** hoặc **EventBridge**.
- **RTO/RPO quyết định DR Strategy:**
    * RTO/RPO = phút $\rightarrow$ **Pilot Light** / **Warm Standby**.
    * RTO/RPO = 0 / Real-time $\rightarrow$ **Multi-Site Active-Active**.
    * RTO/RPO = giờ/ngày $\rightarrow$ **Backup & Restore**.

* **State Management:** Hạn chế lưu Session trực tiếp trên EC2. Đưa Session ra **ElastiCache** hoặc **DynamoDB** để EC2 có thể Scale Out/In thoải mái.

### DOMAIN 2: DESIGN SECURE ARCHITECTURES (30%) - TRỌNG SỐ CAO NHẤT

- **Chính sách "Least Privilege":** Luôn cấp quyền tối thiểu. Không bao giờ dùng `*` trong IAM Policy nếu có đáp án ghi rõ Resource Arn/Action.
- **Bảo mật dữ liệu (Data Encryption):**
    * *At-Rest (Đã lưu):* KMS (SSE-KMS, SSE-S3). Đề hỏi "nhà mạng/khách hàng tự quản lý key" $\rightarrow$ **KMS SSE-C** hoặc **CloudHSM**.
    * *In-Transit (Đang truyền):* TLS/HTTPS.

- **Chống DDoS & Web Attack:**
    + Tầng 3/4 (DDoS) $\rightarrow$ **AWS Shield Standard / Advanced**.
    + Tầng 7 (SQL Injection, XSS) $\rightarrow$ **AWS WAF** (gắn vào CloudFront, ALB, API Gateway).

- **Public vs Private:** Database và Backend EC2 **LUÔN LUÔN** đặt ở **Private Subnet**. Chỉ ALB hoặc Bastion Host/NAT Gateway mới ở Public Subnet.

### DOMAIN 3: DESIGN HIGH-PERFORMING ARCHITECTURES (24%)

- **Đọc/Ghi nhanh (Low Latency Read/Write):**
    + Caching đệm dữ liệu DB $\rightarrow$ **ElastiCache (Redis/Memcached)** hoặc **DynamoDB Accelerator (DAX)**.
    + Caching nội dung tĩnh/động toàn cầu $\rightarrow$ **CloudFront**.


- **Tải lớn/Xử lý song song:**
    + HPC (High-Performance Computing) $\rightarrow$ **EC2 Cluster Placement Group** + **EFA (Elastic Fabric Adapter)** + **FSx for Lustre**.
    + Stream dữ liệu thời gian thực lượng lớn $\rightarrow$ **Kinesis Data Streams** / **Amazon MSK**.

- **Network Performance:**
    + Upload file dung lượng lớn toàn cầu nhanh hơn vào S3 $\rightarrow$ **S3 Transfer Acceleration**.
    + Kết nối trực tiếp On-premise băng thông cố định $\rightarrow$ **AWS Direct Connect**.

### DOMAIN 4: DESIGN COST-OPTIMIZED ARCHITECTURES (20%)

- **EC2 Pricing Selection Rule:**
    + Biết trước tải, chạy 24/7 liên tục 1-3 năm $\rightarrow$ **Reserved Instances (RI)** / **Savings Plans**.
    + Tải chịu gián đoạn được (Batch processing, Big Data) $\rightarrow$ **Spot Instances** (rẻ nhất).
    + Tải không dự đoán được, test ngắn hạn $\rightarrow$ **On-Demand**.


- **Storage Tiering:**
    + S3 Lifecycle Policies: Chuyển dữ liệu ít truy cập sang **S3 Standard-IA** $\rightarrow$ **S3 Glacier Flexible** $\rightarrow$ **S3 Glacier Deep Archive**.
    + Dữ liệu không rõ tần suất truy cập $\rightarrow$ **S3 Intelligent-Tiering** (Tự động tối ưu chi phí).

- **Loại bỏ tài nguyên thừa:** Tìm và xóa Unattached EBS volumes, Idle Elastic IPs, Unassociated Load Balancers thông qua **AWS Trusted Advisor**.

# PHẦN 4: TƯ DUY KHI BỊ "KẸT" GIỮA 2 ĐÁP ÁN

Khi bạn đọc đề, loại được 2 đáp án vô lý và **còn lại 2 đáp án cực kỳ giống nhau**, hãy soi 2 đáp án đó qua các **Thước đo Ưu tiên** sau theo thứ tự từ trên xuống dưới:

```text
THƯỚC ĐO 1: TIÊU CHÍ CỦA CÂU HỎI (CRITICAL REQUIREMENT)
  │
  ├── Đề hỏi "LEAST OPERATIONAL OVERHEAD" (Tốn ít công vận hành nhất)?
  │    └─> Ưu tiên: Serverless (Lambda/DynamoDB) > Managed Service (RDS/EFS) > Unmanaged (Tự cài trên EC2).
  │
  ├── Đề hỏi "MOST COST-EFFECTIVE" (Tối ưu chi phí nhất)?
  │    └─> Ưu tiên: S3 Standard-IA/Glacier > Spot Instances > Serverless > On-Demand EC2.
  │
  ├── Đề hỏi "LEAST DOWNTIME / HIGH AVAILABILITY"?
  │    └─> Ưu tiên: Multi-AZ / Multi-Region > Auto Scaling > Single Instance.
  │
  └── Đề hỏi "LEAST ARCHITECTURAL CHANGE / FASTEST MIGRATION"?
       └─> Ưu tiên: Rehost (Lift-and-Shift) / MGN > Replatform / Refactor.

```

## ELIMINATION RULES

1. **Quy tắc Native/Managed > Self-Managed:**
    * *Ví dụ:* Nếu cần chạy Kubernetes, chọn **EKS** thay vì "Tự cài K8s trên EC2". Nếu cần Queue, chọn **SQS** thay vì "Tự dựng RabbitMQ trên EC2".


2. **Quy tắc IAM Role > Access Keys:**
    * Ứng dụng trên EC2/Lambda muốn gọi S3/DynamoDB $\rightarrow$ Luôn chọn **Gán IAM Role**, tuyệt đối không chọn "Hardcode AWS Access Key / Secret Key vào code/file config".


3. **Quy tắc KMS > Self-Encryption:**
    * Bảo mật dữ liệu trên S3/EBS $\rightarrow$ Chọn **AWS KMS** thay vì "Tự mã hóa bằng script riêng trước khi upload".


4. **Quy tắc VPC Endpoint > Internet Gateway (Bảo mật nội bộ):**
    * EC2 nằm trong Private Subnet muốn gọi S3/DynamoDB mà **KHÔNG đi qua Internet** $\rightarrow$ Chọn **VPC Gateway Endpoint**.


5. **Quy tắc SSM Parameter Store / Secrets Manager > Hardcode:**
    * Lưu trữ Database Passwords / API Keys $\rightarrow$ Luôn chọn **Secrets Manager** (có tính năng tự động xoay kms key - Auto Rotation) hoặc **SSM Parameter Store**, không lưu ở Environment Variables hay file code.

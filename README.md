**4 Domain trong đề thi SAA-C03:**   
Domain 1: Design Secure Architectures (30%) — Bảo mật, phân quyền, mã hóa.   
Domain 2: Design Resilient Architectures (26%) — Chịu lỗi, khả năng phục hồi (HA/DR).   
Domain 3: Design High-Performing Architectures (24%) — Tối ưu hiệu năng, tốc độ xử lý.   
Domain 4: Design Cost-Optimized Architectures (20%) — Tối ưu chi phí, không lãng phí tài nguyên.

1. Pillar: Secure Architectures (Bảo mật - 30%)
Bức tranh này trả lời câu hỏi: Làm sao để hệ thống không bị chui lén, dữ liệu không bị lộ và mọi thao tác đều được kiểm soát?

- Lý thuyết cốt lõi:
    + Identity & Access (IAM): Ai (Principal) được làm gì (Action) trên tài nguyên nào (Resource)?
    + Network Isolation (VPC): Phân vùng mạng công cộng (Public) và mạng nội bộ (Private).
    + Data Protection (KMS/Secrets): Mã hóa dữ liệu khi truyền (In-transit) và khi lưu trữ (At-rest).

- Cách ghép Service:
    + Truy cập ứng dụng: User ➔ WAF (chặn SQLi, XSS) ➔ ALB trong Public Subnet ➔ EC2 nằm trong Private Subnet.
    + Truy cập dịch vụ AWS an toàn: EC2 trong Private Subnet muốn gọi S3/DynamoDB không qua Internet ➔ Ghép VPC Endpoints (Gateway/Interface).
    + Quản lý mật khẩu/Khóa: Ứng dụng cần connection string vào DB ➔ Luân chuyển tự động bằng AWS Secrets Manager ➔ Mã hóa bằng KMS.
2. Pillar: Resilient Architectures (Chịu lỗi & Phục hồi - 26%)
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

3. Pillar: High-Performing Architectures (Hiệu năng cao - 24%)
Bức tranh này trả lời câu hỏi: Làm sao để hệ thống xử lý hàng triệu request với độ trễ (latency) thấp nhất?

- Lý thuyết cốt lõi:
    + Caching (Bộ nhớ tạm): Không bắt Database hay Server làm việc lại những việc đã làm.
    + Offloading: Chuyển các file tĩnh (ảnh, video) ra gần người dùng nhất.
    + Parallel Processing: Xử lý song song.

- Cách ghép Service:
    + Tăng tốc nội dung tĩnh & API: CloudFront (CDN) đứng trước S3/ALB để cache nội dung ở các Edge Location gần người dùng nhất.
    + Tăng tốc Database truy vấn nhiều: Đặt ElastiCache (Redis/Memcached) hoặc DynamoDB Accelerator (DAX) phía trước Database để lưu kết quả query hay dùng.
    + Tăng tốc lưu trữ tệp: Chọn ổ đĩa EBS Provisioned IOPS (io2) cho DB cần IOPS cực lớn, hoặc EFS cho nhiều server đọc/ghi đồng thời.

4. Pillar: Cost-Optimized Architectures (Tối ưu chi phí - 20%)
Bức tranh này trả lời câu hỏi: Làm sao để đạt được mục tiêu kinh doanh với số tiền AWS trả hằng tháng là nhỏ nhất?

- Lý thuyết cốt lõi:
    + Right-sizing: Chọn đúng kích thước tài nguyên, không mua thừa.
    + Storage Lifecycle: Chuyển dữ liệu ít dùng sang kho lưu trữ rẻ hơn.
    + Pricing Models: Kết hợp On-Demand, Reserved, Savings Plans và Spot Instances.

- Cách ghép Service:
    + Tối ưu Storage: Thiết kế S3 Lifecycle Rules: Dữ liệu mới lưu ở S3 Standard ➔ Sau 30 ngày chuyển sang S3 Standard-IA ➔ Sau 90 ngày đẩy vào S3 Glacier Flexible / Deep Archive (rẻ hơn hàng chục lần).
    + Tối ưu Compute: Dùng Spot Instances cho các task batch processing (chạy ẩn, chấp nhận ngắt đột ngột) để giảm tới 90% chi phí. Dùng Savings Plans / Reserved Instances cho các EC2/RDS chạy 24/7.
    + Tối ưu Architecture: Chuyển từ EC2 chạy liên tục sang kiến trúc Serverless (API Gateway + Lambda + DynamoDB) — chỉ trả tiền đúng theo mi-li-giây khi có request chạy.

**BỨC TRANH TỔNG QUAN: 13 CATEGORY CỐT LÕI CỦA AWS (SAA-C03)**
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

**I. COMPUTE**
1. Amazon EC2 (Elastic Compute Cloud)
- Khái niệm: Máy chủ ảo (Virtual Machine - IaaS) chạy trên hạ tầng AWS.
- Tác dụng: Cung cấp tài nguyên tính toán có khả năng thay đổi quy mô linh hoạt.

- Cách sử dụng:
    + Chọn AMI (Amazon Machine Image: Linux, Windows, Ubuntu...).
    + Chọn Instance Type (Ví dụ: t3.micro, c5.large, r5.xlarge).
    + Cấu hình EBS Volume (ổ đĩa boot).
    + Cấu hình Security Group (Firewall cấp Instance).
    + Kết nối qua SSH (Linux) hoặc RDP (Windows) / EC2 Instance Connect / Systems Manager.

- Phân loại Instance Types (Cần nhớ cho SAA):
    + General Purpose (t3, m5, m6g): Bằng phẳng giữa CPU & RAM (Web server, Small DB).
    + Compute Optimized (c5, c6g): CPU cao (Media encoding, High-performance web server, Gaming).
    + Memory Optimized (r5, x1): RAM cực lớn (In-memory DBs như Redis, High-performance Relational DB).
    + Storage Optimized (i3, d2): Sequential Read/Write cao (Data warehousing, Distributed file systems).

- Phân loại Mua (Pricing Models - CỰC KỲ QUAN TRỌNG):
    + On-Demand: Trả tiền theo giây/giờ, không cam kết. Dùng cho: App mới, workload không đoán trước được.
    + Reserved Instances (RI) / Savings Plans: Cam kết 1 năm hoặc 3 năm. Giảm tới 72%. Dùng cho: App chạy liên tục 24/7 (Baseline workload).
    + Spot Instances: Đấu giá tài nguyên thừa của AWS. Giảm tới 90%. AWS có thể thu hồi bất kỳ lúc nào (báo trước 2 phút). Dùng cho: Batch jobs, CI/CD, Stateless workloads chấp nhận gián đoạn.
    + Dedicated Hosts / Dedicated Instances: Chạy trên phần cứng vật lý riêng biệt. Dùng cho: Tuân thủ pháp lý (Compliance) hoặc ứng dụng có license gắn theo CPU vật lý (BYOL).

- Pros: Toàn quyền kiểm soát (Full Root/Admin OS access), tương thích mọi phần mềm truyền thống.
- Cons: Tốn công vận hành (Phải tự OS patching, cài Antivirus, quản lý backup).
- Nên dùng khi: Đưa ứng dụng Monolith cũ lên Cloud, ứng dụng cần cấu hình OS sâu, workload chạy liên tục nặng.
- KHÔNG nên dùng khi: Ứng dụng nhỏ chạy theo sự kiện (nên dùng Lambda) hoặc muốn rảnh tay không quản server.

2. Auto Scaling Group (ASG)
Khái niệm: Tập hợp các EC2 Instance được tự động tăng hoặc giảm số lượng dựa trên nhu cầu (Demand).
Tác dụng: Đảm bảo tính Sẵn sàng cao (Availability) và Tối ưu chi phí (Cost).

Cách sử dụng:
    Tạo Launch Template (Định nghĩa EC2 chạy bằng AMI nào, Instance type gì, Security group nào).
    Cấu hình Min, Max, Desired Capacity.
    Đặt Scaling Policy (Dynamic, Target Tracking, Scheduled, Predictive).

Các cơ chế Scale:
Target Tracking Scaling: Duy trì CPU trung bình ở mức 60%.
Step/Simple Scaling: Nếu CPU > 80% thì thêm 2 instance; nếu CPU < 20% thì bớt 1 instance.
Scheduled Scaling: Tăng lên 10 instance vào lúc 8 giờ sáng mỗi T2 (Khung giờ cao điểm).

Pros: Tự động khôi phục Instance lỗi (Health Check Replacement), trải rộng EC2 ra nhiều AZ (High Availability).
Cons: Cần thiết kế App dạng Stateless (không lưu Session trực tiếp trên EC2) thì Scale mới chuẩn.
Nên dùng khi: Mọi ứng dụng EC2 chạy sau Load Balancer có lượng traffic biến động.
KHÔNG nên dùng khi: Ứng dụng Stateful không thể nhân bản nhiều con.

3. AWS Lambda (Serverless Compute)
Khái niệm: Dịch vụ thực thi code dạng Serverless (FaaS - Function as a Service).
Tác dụng: Chạy code tự động khi có sự kiện (Event-driven) mà hoàn toàn không cần dựng server.

Cách sử dụng: Viết hàm (Python, Node.js, Go, Java...) -> Đặt Trigger (S3, API Gateway, DynamoDB Streams, EventBridge) -> Lambda tự chạy.

Thông số kỹ thuật quan trọng:
    Thời gian chạy tối đa (Timeout): 15 phút.
    RAM cấu hình: Từ 128MB đến 10,240MB (CPU scale tỉ lệ thuận theo RAM).
    Dung lượng bộ nhớ tạm /tmp: Từ 512MB đến 10GB.

Pros: Chi phí cực rẻ (Tính theo số lần gọi + thời gian chạy tính bằng millisecond), Tự động Scale từ 0 đến hàng ngàn request đồng thời.
Cons:
    Cold Start: Lần chạy đầu tiên bị trễ vài trăm ms (Khắc phục bằng Provisioned Concurrency).
    Giới hạn 15 phút.

Nên dùng khi: API backend (kết hợp API Gateway), xử lý dữ liệu realtime khi upload file lên S3, Cronjob tự động.
KHÔNG nên dùng khi: Task tính toán phức tạp chạy liên tục nhiều giờ liền.

4. Elastic Container Service (ECS)
Khái niệm: Dịch vụ điều phối Container (Container Orchestration) độc quyền của AWS.
Tác dụng: Chạy và quản lý hàng loạt Docker Container.

Cấu trúc:
    Task Definition: File JSON khai báo Docker Image, CPU/RAM, Port, Environment variables.
    Task: Bản thể đang chạy của Task Definition (tương đương 1 Container).
    Service: Quản lý duy trì số lượng Task nhất định (tương tự ReplicaSet).

2 Launch Types (Mô hình chạy):

ECS EC2 Launch Type: Bạn tự quản lý dàn máy chủ EC2 để làm Cluster chứa Container. (Rẻ hơn nếu tận dụng hết capacity).

ECS Fargate Launch Type: Serverless Container. Không quản lý EC2, chỉ khai báo CPU/RAM cho Task, AWS lo phần còn lại.

Pros: Tích hợp cực sâu với IAM Role cho từng Task, Load Balancer (ALB), CloudWatch.

Cons: Vendor lock-in (Khó chuyển sang Cloud khác so với Kubernetes).

Nên dùng khi: Ứng dụng Microservices chạy Docker, muốn quản lý đơn giản không rườm rà.

KHÔNG nên dùng khi: Đội ngũ bắt buộc dùng hệ sinh thái Kubernetes chuẩn mở.

5. Elastic Kubernetes Service (EKS)
Khái niệm: Dịch vụ quản lý Kubernetes (K8s) chuẩn Cloud Native do AWS vận hành Control Plane.

Tác dụng: Chạy ứng dụng Kubernetes mà không cần tự cài Control Plane (API Server, etcd).

Mô hình triển khai: Control Plane do AWS quản lý 100% (High Availability qua nhiều AZ). Worker Nodes có thể là EC2 hoặc Fargate.

Pros: Chuẩn mở Kubernetes, dễ dàng di chuyển ứng dụng (Portable) giữa On-premises và AWS.

Cons: Chi phí cố định ~$0.10/giờ cho Control Plane, cấu hình phức tạp, đòi hỏi kiến thức K8s sâu.

Nên dùng khi: Công ty đã và đang xài Kubernetes ở On-premise, muốn bê nguyên sang AWS.

KHÔNG nên dùng khi: Đội ngũ chưa biết K8s và chỉ muốn chạy vài container đơn giản (Nên dùng ECS + Fargate).

6. AWS Fargate (Serverless Compute cho Containers)
Khái niệm: Engine tính toán Serverless dùng chung cho cả ECS và EKS.

Tác dụng: Loại bỏ hoàn toàn việc provision, scale và quản lý máy chủ EC2 khi chạy Container.

Cách sử dụng: Trong ECS/EKS, chọn Launch Type = FARGATE. Chỉ định bao nhiêu vCPU và GB RAM cho Container.

Pros: Không lo OS patching, cô lập bảo mật cấp VM cho từng Task, chỉ trả tiền đúng tài nguyên Container tiêu thụ.

Cons: Chi phí trên mỗi vCPU/RAM đắt hơn một chút so với việc tối ưu hóa 100% trên EC2 Instance.

Nên dùng khi: Muốn tập trung 100% vào code/container, không muốn tốn nhân sự quản lý EC2 Cluster.

7. AWS Elastic Beanstalk (PaaS - Platform as a Service)
Khái niệm: Nền tảng hỗ trợ Deploy ứng dụng nhanh chóng mà không cần bận tâm thiết kế hạ tầng bên dưới.

Tác dụng: Bạn đưa Code (Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker) -> Beanstalk tự động dựng EC2, ASG, Load Balancer, Database và Security Group.

Pros: Triển khai cực nhanh (Developer friendly), vẫn cho phép "jump" vào hạ tầng bên dưới để tùy chỉnh nếu cần.

Cons: Mất đi tính linh hoạt soát sâu nếu cấu hình kiến trúc phức tạp.

Nên dùng khi: Muốn demo app nhanh, đội phát triển không có chuyên viên DevOps/Cloud Engineer.

KHÔNG nên dùng khi: Hệ thống lớn có quy trình kiến trúc phức tạp, tùy biến nhiều.

8. AWS Batch
Khái niệm: Dịch vụ quản lý và lập lịch chạy các công việc tính toán theo lô (Batch Processing) ở mọi quy mô.

Tác dụng: Tự động chia nhỏ, xếp hàng (Queue) và phân phối hàng ngàn job tính toán lên EC2 / Spot Instances / Fargate.

Pros: Tự động scale từ 0 lên hàng ngàn node và tự hạ xuống 0 khi chạy xong. Đơn giản hóa việc chạy Batch jobs.

Nên dùng khi: Xử lý hình ảnh/video số lượng lớn, phân tích tài chính cuối ngày, mô phỏng khoa học.

9. AWS App Runner
Khái niệm: Dịch vụ Serverless hoàn toàn dành cho Web App và API đơn giản.

Tác dụng: Đưa trực tiếp Source Code (GitHub) hoặc Container Image -> App Runner tự build, deploy, cân bằng tải và cấp HTTPS URL trong vài phút.

Pros: Siêu đơn giản, tự động scale traffic từ ít đến nhiều.

Cons: Thiếu các tính năng tùy biến hạ tầng mạng phức tạp.

Nên dùng khi: Cần deploy web app REST API nhanh nhất có thể.

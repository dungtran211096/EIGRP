# Giao thức định tuyến EIGRP

[1. Định nghĩa EIGRP](#definition)

[2. Ưu điểm và nhược điểm](#comparation)

[3. Nguyên tắc hoạt động](#work)

====================================

<a name="definition"></a>
### 1. Định nghĩa EIGRP

EIGRP là phiên bản nâng cao của IGRP, IGRP là giao thức dạng classfull, còn EIGRP là giao thức dạng classless, nghĩa là có mang theo subnetmask trong các lần cập nhật.

EIGRP là giao thức định tuyến hỗn hợp(hybrid routinng), là sự kết hợp của Distance vector và link states.

EIGRP là giao thức định tuyến theo distance vector nâng cao nhưng khi cập nhật và bảo trì thông tin node láng giềng và thông tin định tuyến thì nó làm việc như giao thức định tuyến linkstates.

<a name="comparation"></a>
### 2. Ưu điểm và nhược điểm

EIGRP với distance vector.**Ưu điểm:**

- Khả năng hội tụ nhanh: vì chúng sử dụng [DUAL](#dual). DUAL bảo đảm hoạt động không bị lặp vòng khi tính toán đường đi, cho phép mọi Router trong hệ thống mạng thực hiện đồng bộ cùng lúc khi có sự thay đổi xảy ra.
- Bảo tồn băng thông và sử dụng băng thông một cách hiểu quả: vì nó chỉ gửi thông tin cập nhật một phần và giới hạn chứ không gửi toàn bộ bảng định tuyến. Nhờ vậy nó chỉ tốn một lượng băng thông tối thiểu khi hệ thống mạng đã ổn định. Điều này tương tự như hoạt động cập nhật của OSPF, Router EIGRP chỉ gửi thong tin cập nhật một phần cho Router nào cần thông tin đó mà thôi chứ không gửi mọi Router khác trong vùng như OSPF. Chính vì hoạt động cập nhật theo chu kỳ, các Router EIGRP giữ liên lạc với nhau bằng các gói hello rất nhỏ. Việc trao đổi các gói hello theo định kỳ không chiếm nhiều băng thông đường truyền.
- Hỗ trợ VLSM (Veriable Length Subnet Mask) và CIDA(Classles Inter Domain Routing). Không giống như IGRP,EIGRP có thể trao đổi thông tin ở các IP khác lớp mạng
- Gửi toàn bộ bảng định tuyến qua cho các neighbor giống như Rip. Nhưng nó chỉ gửi cho đến khi mạng hội tụ thì ngưng không gửi nữa.
- Hỗ trợ IP, IPX, Apple talk: vì Talk nhờ có cấu trúc từng phần theo giao thức (PDMs – Protocok dependent modules). EIGRP có thể phân phối thông tin của IPX,RIP để cải tiến hoạt động toàn diện. Trên thực tế, EIGRP có thể điều khiển giao thức này. Router EIGRP nhận thông tin định tuyến và dịch vụ, chỉ cập nhật cho các Router khác khi thông tin trong bảng định tuyến thay đổi.
- Chạy trực tiếp trên IP và protocol number là 88.
- Load balancing cho những mạng bằng nhau. Có thể có nhiều đường đi và nhiều đường dự phòng.
- Hổ trợ tất cả các giao thức và cấu trúc dữ liệu ở layer 2.
- Chỉ hỗ trợ Multicast hoặc Unicast trong từng trường hợp cụ thể.
- Hổ trợ việc chứng thực
+ Manual Summary trên bất kỳ interface nào.

**Nhược điểm:**
EIGRP là một giao thức với rất nhiều ưu điểm và có thể được sử dụng trong những mô hình mạng vừa và lớn tuy nhiên vì đây là giao thức độc quyền của Cisco nên nó chỉ chạy trên thiết bị của cisco, trong khi đó không phải một tổ chức nào cũng có thể dùng toàn đồ Cisco mà còn các dòng sản phẩm khác nữa. Chính vì vậy, đây là một bất lợi của giao thức định tuyến EIGRP.

<a name="work"></a>
### 3. Nguyên tắc hoạt động

Để tìm ra đường đi tốt nhất nó phải trải qua 3 giao đoạn:

- Thiết lập [neighbor](#neighbor)
- Đưa ra [bảng topology](#topo)
- Dùng thuật toán [Dual](#dual) để tìm ra đường đi tốt nhất trong bảng định tuyến.

=> Giống OSPF

####3.1 Thiết lập neighbor
EIGRP thiết lập neighbor giống như OSPF.

- B1: trao đổi gói tin hello 5s/1 lần có IP(224.0.0.10)
- B2: các thông số trong hello với khớp với 1 vài thông số trên router thì mới được làm neighbor của nhau:
	- AS (Autonomous system: hệ thống tự trị): giống AS của IEEE nhưng ý nghĩa nhỏ hơn nhiều so với AS trên. Nó chỉ tương đương với 1 vùng hay 1 domain chạy EIGRP.
	- Cùng Subnet(không cần cùng subnet-mask).
	- Cùng loại xác thực. Chỉ hỗ trợ duy nhất xác thực MD5
=> Thỏa 3 đk trên thì 2 router sẽ là neighbor của nhau

Thành phần neighbor table bao gồm thông tin được yêu cầu bởi cơ chế truyền tin tin cậy. Sequence number được triển khai để khớp với acks của packets. Sequence number cuối cùng được nhân từ neighbor sẽ được lưu lại để cho gói tin ngoài thứ tư có thể biết được. Danh sách truyền đi thường sử dụng hàng đợi để có thể truyền packet mỗi neighbor. ROUND TRIP TIME lưu giữ cấu trúc dữ liệu neighbor để ước tính khoảng thời gian tối ưu lưu truyền của gói tin.

#### 2. Topology table

**FD(Feasible Distance):** là thông tin định tuyến nhỏ nhất mà EIGRP tính được từ router của mình tới mạng đích.

**AD(Advertised Distance):**  thông tin định tuyến được đo từ neighbor của mình đến mạng đích

# LLMrosa – Hướng dẫn tiếng Việt

**Điều khiển robot bằng AI (LLM) kết hợp ROS 2, Nav2, SLAM và Gazebo**

> **Phân biệt phạm vi:** Repository \`xulytiengviet/LLMrosa\` hiện chứa mã ROSA của NASA JPL. Bài mô phỏng robot Summit XL sử dụng ROSA + Nav2 + Gazebo thuộc repository ZHAW riêng [RAP-2025-Project-Group-2](https://github.com/RAP-2025-Project-Group-2/RAP-2025-Project-Group-2). Trang \`index.html\` bổ sung trong repository này là **bài học và mô phỏng 2D độc lập**, không phải cổng điều khiển robot thật.

## 1. Xem trang Việt hóa

Trang chủ nằm tại [index.html](index.html). Có thể mở trực tiếp bằng trình duyệt hoặc triển khai qua GitHub Pages.

Để bật GitHub Pages:

1. Vào **Settings → Pages** trong repository.
2. Ở mục **Build and deployment**, chọn **Deploy from a branch**.
3. Chọn **main**, thư mục **/(root)** rồi lưu.
4. Khi GitHub báo triển khai thành công, truy cập **https://xulytiengviet.github.io/LLMrosa/**.

Trạng thái công bố Pages tùy thuộc cấu hình repository và kết quả deployment; việc commit \`index.html\` **không tự bảo đảm** website đã trực tuyến.

## 2. Thử mô phỏng 2D (không cần ROS)

Mở trang, đến mục **Phòng thí nghiệm tương tác** và thử:

| Câu lệnh tiếng Việt | Phản hồi mô phỏng |
|---|---|
| Hãy đi đến nhà bếp | Chọn \`navigate_to_location_by_name("kitchen")\`, lập đường A* |
| Đi đến phòng khách | Điều hướng tới đích đã định nghĩa |
| Đi đến phòng ngủ | Lập tuyến đi qua các ô trống |
| Lùi lại 1,5 mét | Tính đích tương đối theo ô 0,5 m |
| Khám phá bản đồ | Ghé các waypoint mẫu; **không chạy SLAM** |
| Lưu bản đồ | Minh họa thao tác; **không ghi file thật** |
| Dừng lại | Hủy tuyến và dừng mô phỏng |
| Hãy bay lên | Từ chối vì lệnh không nằm trong danh sách cho phép |

**Giới hạn mô phỏng:** Không có LLM/ROSA/ROS 2/Nav2/Gazebo thực sự chạy trong trang web; các tên công cụ và phản hồi là mô phỏng có chủ đích. Tọa độ và chướng ngại vật được định nghĩa cục bộ. Chỉ dùng cho giảng dạy thuật toán điều hướng, phân tích ý định và kiểm tra lệnh.

## 3. Chạy ROSA Summit gốc (ROS 2 Jazzy + Gazebo)

Ví dụ ZHAW sử dụng môi trường container cụ thể \`robopaas/rap-jazzy:cuda12.5.0\`, ROS 2 Jazzy, Nav2 và thế giới Gazebo Summit XL. Các bước dưới đây phải chạy **bên trong container/workspace đã được cấu hình tương thích**, không phải trong GitHub Pages.

Sao chép nguồn:

\`\`\`bash
git clone https://github.com/RAP-2025-Project-Group-2/RAP-2025-Project-Group-2.git ~/rap/Gruppe2
# Đọc init.sh trước khi thực thi (script có thao tác cài gói và build).
source ~/rap/Gruppe2/init.sh
\`\`\`

Chạy mô phỏng bản đồ sẵn có:

\`\`\`bash
ros2 launch rosa_summit summit.launch.py
\`\`\`

Hoặc chạy SLAM:

\`\`\`bash
ros2 launch rosa_summit summit.launch.py slam:=True
\`\`\`

Mở terminal mới đã source đúng ROS workspace và chạy agent:

\`\`\`bash
ros2 run rosa_summit rosa_summit
\`\`\`

Nhập lệnh mẫu theo tài liệu gốc: \`take me to the kitchen\`, \`stop\`, \`start autonomous exploration\`. Muốn sử dụng tiếng Việt với LLM thật cần kiểm tra prompt, chuẩn hóa tên phòng và xác nhận kết quả ánh xạ công cụ.

**Lưu ý:** Phần code Summit không phải thành phần đã cài mặc định trong \`LLMrosa\`; cần tải repository ZHAW riêng. Lệnh trên phụ thuộc môi trường và các package Gazebo/Nav2 tương ứng.

## 4. Kiến trúc mở rộng an toàn cho robot thật

\`\`\`text
Giọng nói / chữ tiếng Việt
          ↓
ASR / LLM (hiểu ý định)
          ↓
ROSA + danh sách tool cho phép
          ↓
Task IR có kiểu / Safety Gate đề xuất
          ↓
ROS 2 Action / Nav2 / SLAM
          ↓
Robot thật hoặc Gazebo → telemetry & stop
\`\`\`

Safety Gate đề xuất nên kiểm tra giới hạn vận tốc, khoảng cách, trạng thái map/TF, vùng cấm, timeout, mất kết nối, xác thực người dùng và quyền can thiệp khẩn cấp. Không cho phép LLM gửi mã Python/ROS shell tùy ý đến máy điều khiển.

Có thể thử tích hợp CVNSS4.0 vào phần chuẩn hóa/hậu kiểm lệnh tiếng Việt; đây là **hướng nghiên cứu dự kiến**, không phải chức năng có sẵn của ROSA.

## 5. Bảo vệ API key

Repository có file \`.env\` đang được Git theo dõi. **Không điền API key thật vào file này rồi commit lên GitHub.** Khi phát triển, tạo cấu hình riêng nằm ngoài Git hoặc thêm quy tắc bỏ qua, dùng biến môi trường/secrets manager phía backend. Nếu đã từng commit key thật, phải thu hồi/luân chuyển key; chỉ xóa file khỏi commit mới không xóa dữ liệu trong lịch sử Git.

GitHub Pages là nơi phục vụ **website tĩnh**, không lưu khóa và không tiếp xúc trực tiếp với ROS network. Để điều khiển robot thật qua website, cần backend có xác thực và kiểm soát bảo mật.

## 6. Tài liệu gốc và ghi nhận công lao

- [NASA JPL – ROSA](https://github.com/nasa-jpl/rosa) (giấy phép Apache-2.0; xem LICENSE của repo này)
- [ROSA Summit – nhóm ZHAW](https://github.com/RAP-2025-Project-Group-2/RAP-2025-Project-Group-2)
- [Mike Likes Robots – Controlling Robots using a Large Language Model](https://mikelikesrobots.github.io/blog/llm-robot-control/)
- [NASA ROSA Wiki](https://github.com/nasa-jpl/rosa/wiki)

Bản Việt hóa là tài liệu bổ sung, không phải sản phẩm chính thức của NASA JPL hay nhóm ZHAW.

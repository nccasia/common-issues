Khi lưu dữ liệu vào DB, nhiều dữ liệu được lưu dưới dạng Binary

Ví dụ: Hòm đồ của người dùng sẽ dùng n bit đầu tiên cho Mũ, m bit tiếp theo cho Vũ khí, ... 
& khi truyền về client sẽ phải Serialize ra.
Nảy sinh ra tình huống: update bản mới, nếu có thay đổi gì về chuỗi Binary khia, thì nhiều chức năng ở client cũ sẽ bị hỏng / sai lệch do không nhận được đúng thông tin.

- Kiểu lưu này cũng rất khó để query & check bằng các tool visualize DB
- Khi người chơi bị lỗi đồ, không check được history để bù đắp.
- Có lần GM phát nhầm đồ chưa được implement, chỉ có cách xóa Hòm đồ đi, thay vì chỉ cần xóa bản ghi đồ lỗi đó
# Rule hiệu chỉnh AI label 

## 1. Kết luận từ benchmark 

AI label ban đầu **khá tốt**, đặc biệt ở sentiment

Benchmark AI trên gold set:

- Relevance Accuracy ≈ 0.734, Macro-F1 ≈ 0.697.
- Relevance có recall lớp relevant rất cao, nhưng còn nhiều false positive: AI giữ nhầm comment không liên quan.
- Sentiment Accuracy ≈ 0.753, Macro-F1 ≈ 0.755 trên các dòng mà cả AI và human đều relevant.
- Sentiment đủ tốt để làm pseudo-label nền; chỉ cần sửa các lỗi sentiment rõ.

## 2. Phân tích lỗi relevance

Tổng số relevance mismatch: 133 dòng.

| Error type | Số dòng |
|---|---:|
| AI giữ nhầm - liên quan yếu/gián tiếp, human chốt không liên quan | 50 |
| AI giữ nhầm - comment quá ngắn/chung chung, thiếu ngữ cảnh | 16 |
| AI giữ nhầm - nội dung hướng vào video/reviewer/AD/nhu cầu phụ trợ, không trực tiếp vào topic | 9 |
| AI giữ nhầm - nhắc đối tượng/sản phẩm khác, thiếu tín hiệu topic chính | 9 |
| AI giữ nhầm - mua bán/quảng cáo/link/shop, không phải đánh giá topic | 8 |
| AI giữ nhầm - chỉ nhắc đội/cầu thủ khác, thiếu tín hiệu topic chính | 6 |
| AI giữ nhầm - hỏi/mua bán giá chung, thiếu tín hiệu topic rõ | 6 |
| AI giữ nhầm - fanwar/tranh cãi chung, không trực tiếp đánh giá topic | 6 |
| AI loại nhầm - có topic nhưng bị nhiễu bởi video/BLV/đối tượng khác | 5 |
| AI giữ nhầm - thông tin giá/thị trường/linh kiện chung, thiếu trọng tâm topic | 5 |
| AI giữ nhầm - tiếng Anh/ngoại ngữ ngoài phạm vi phân tích | 4 |
| AI giữ nhầm - hỏi phụ kiện/kỹ thuật phụ trợ, không đủ trọng tâm VF3 | 3 |
| AI loại nhầm - có alias/từ khóa topic nhưng AI không nhận diện | 2 |
| AI loại nhầm - comment ngắn/implicit nhưng human xác định liên quan theo ngữ cảnh topic | 2 |
| AI giữ nhầm - nhắc sản phẩm khác, thiếu tín hiệu topic chính | 1 |
| AI loại nhầm - nhắc bộ phận/tính năng/sản phẩm nhưng thiếu keyword chính | 1 |


### Rule relevance đề xuất

#### R1. Giảm false positive relevance của AI
Nếu AI gắn `is_relevant = 1`, chỉ chuyển về `0` khi có dấu hiệu rõ:

- Comment quá ngắn/chung chung, thiếu ngữ cảnh: “chịu”, “ăn hên thôi”, “còn k”, “chuẩn bác”…
- Chỉ nhắc đối tượng ngoài topic:
  - Arsenal: chỉ nói MC/Man City/Pep/Haaland/Cherki/City mà không có Arsenal/Pháo/Phấn/alias Arsenal.
  - Ronaldo: chỉ nói Messi/M10/Pele/MP10/HL9 mà không đánh giá Ronaldo/CR7.
  - S25: chỉ nói iPhone/S23/S24/điện thoại khác mà không có S25/Samsung/Galaxy.
  - VF3: chỉ nói VF5/Mercedes/xe Trung Quốc chung mà không có VF3/VinFast/bộ phận VF3.
- iPhone: chỉ nói giá/thị trường/linh kiện chung như RAM, SSD, tỉ giá, VND, hàng xách tay, giữ giá mà không nhắc rõ iPhone/Apple/17/16/Air.
- Nội dung hướng vào video/AD/reviewer/BLV hoặc nhu cầu phụ trợ như xin địa chỉ dán phim, hỏi camera hành trình, hỏi shop/link, không trực tiếp đánh giá topic.
- Quảng cáo, link bán hàng, shop, comment mua bán không thể hiện đánh giá topic.
- Tiếng Anh/ngoại ngữ ngoài phạm vi phân tích tiếng Việt nếu nhóm đã thống nhất loại.

#### R2. Khôi phục false negative relevance của AI
Nếu AI gắn `is_relevant = 0`, chuyển về `1` khi có dấu hiệu rõ:

- Có alias/từ khóa topic:
  - Arsenal: Arsenal, Ars, Pháo, Phấn, Phú Ngao, Phốn, Asean/A sơ nan, Arteta…
  - iPhone17series: iPhone, IP, 17, 16 Pro/16prm, Apple, Air, Pro Max…
  - Ronaldo: Ronaldo, CR7, R7, Anh 7, Cu Đô, 1000 bàn…
  - S25series: S25, S25U, Samsung, Galaxy, One UI, màn, pin, sọc…
  - VinFast VF3: VF3, VinFast, xe, pin, sạc, cốp, vô lăng, cần số, firmware…
- Có topic nhưng bị nhiễu bởi video/BLV/đối tượng khác, ví dụ vẫn nhắc Arsenal/Phấn/Ars nhưng câu có thêm MC/BLV.
- Topic sản phẩm: nhắc bộ phận/tính năng/trải nghiệm đặc thù như pin, camera, màn hình, sạc, cần số, vô lăng, cốp.

## 3. Phân tích lỗi sentiment/manual_label

Tổng số sentiment mismatch: {len(sent_rows)} dòng.

| Error type | Số dòng |
|---|---:|
| AI làm mềm negative - sắc thái tiêu cực ngầm/gián tiếp bị gắn neutral | 11 |
| AI làm mềm negative - mỉa mai/cà khịa bị gắn neutral | 10 |
| AI gắn positive nhầm - câu mỉa mai/chê rõ chứa từ/cấu trúc tích cực | 6 |
| AI bỏ sót positive - khen/ủng hộ rõ bị gắn neutral | 6 |
| AI diễn giải quá mức negative - tín hiệu tiêu cực yếu/mơ hồ, human chốt neutral | 6 |
| AI làm mềm negative - chê/lỗi/trải nghiệm tiêu cực rõ bị gắn neutral | 5 |
| AI diễn giải quá mức positive - câu mixed/khen nhẹ nhưng chưa đủ nghiêng positive | 5 |
| AI diễn giải quá mức positive - câu hỏi/thông tin/lựa chọn bị hiểu là khen | 3 |
| AI diễn giải quá mức positive - tín hiệu tích cực yếu/mơ hồ, human chốt neutral | 3 |
| AI diễn giải quá mức negative - câu hỏi/lo ngại thông tin bị hiểu là chê | 3 |
| AI diễn giải quá mức negative - câu vừa khen vừa chê, human chốt neutral | 2 |
| AI diễn giải quá mức positive - quảng cáo/bán hàng/link, không phải sentiment topic | 1 |
| AI gắn positive nhầm - bị nhiễu bởi từ khóa tích cực trong câu tiêu cực | 1 |
| AI gắn positive nhầm - không nhận ra hướng tiêu cực của câu | 1 |
| AI gắn negative nhầm - cảm xúc tiêu cực hướng vào người khác, không phải topic | 1 |
| AI diễn giải quá mức negative - cảm xúc tiêu cực hướng vào người khác/video, không phải topic | 1 |
| AI bỏ sót positive - sắc thái tích cực nhẹ/gián tiếp bị gắn neutral | 1 |
| AI diễn giải quá mức positive - cảm xúc hướng đối tượng khác/video/reviewer | 1 |


### Rule sentiment đề xuất

#### S1. Sửa negative bị AI làm mềm thành neutral
Chuyển `neutral -> negative` khi có dấu hiệu rõ:

- Mỉa mai/cà khịa: phốn lào, phú ngao, kẹt pháo, bàn danh dự, trắng tay, hạng nhì, 6.9 điểm, chị 7, cười/haha/😂 kèm chê.
- Chê rõ sản phẩm/người/topic: không đáng mua, thất bại, tệ nhất, pin yếu, sọc màn, camera cùi, xe dỏm, của nợ, phá sản, khuyên bán, hút máu, lùa gà.
- Câu có cấu trúc khen giả/chê thật: “giỏi trong trận nhỏ… đá chả ra”, “tuyệt vời hahaha” nhưng ngữ cảnh mỉa mai.

#### S2. Sửa positive bị AI bỏ sót
Chuyển `neutral -> positive` khi có khen/ủng hộ rõ:

- Đá hay hơn, chơi tốt, xứng đáng, chúc mừng, hời, ngon, đẹp, ổn, đáng mua.
- Với sản phẩm: pin tốt, camera đẹp, xài ổn, dùng tốt, giá hời.
- Với Ronaldo/Arsenal: chúc mừng, đẳng cấp, vĩ đại, đá hay, chơi tốt.

Không dùng từ yếu đơn lẻ như “được”, “hay”, “ổn” để sửa nếu câu là câu hỏi/lựa chọn.

#### S3. Đưa về neutral khi AI diễn giải quá mức
Chuyển AI positive/negative về neutral khi:

- Câu hỏi thông tin/lựa chọn: “so với…”, “có nên…”, “giá bao nhiêu”, “pin ổn không”, “A hay B”.
- Reply thiếu ngữ cảnh, không rõ khen/chê topic.
- Cảm xúc hướng vào video/reviewer/AD/BLV/người nói, không phải topic.
- Quảng cáo/link bán hàng không thể hiện đánh giá topic.
- Câu mixed sentiment cân bằng: vừa khen vừa chê, không nghiêng rõ.
- Câu chỉ thể hiện tiếc/thiếu may mắn/cảm thông, không chê rõ.




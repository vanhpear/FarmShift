FarmShift — Hỗ trợ lập kế hoạch luân canh từ dữ liệu NASA
NASA Space Apps Challenge 2026 · Adapting Farms with NASA Data

FarmShift là nguyên mẫu web hỗ trợ so sánh 6 lựa chọn cây trồng qua 3 vụ dựa trên dữ liệu khí hậu NASA POWER, đánh giá mức phù hợp cây trồng, phân tích đa tiêu chí và tối ưu hóa tổ hợp. Hệ thống kết hợp Classical Exhaustive Search với QAOA mô phỏng để kiểm tra chất lượng nghiệm.
Phạm vi: Đây là nguyên mẫu nghiên cứu cho hackathon, không phải công cụ dự báo năng suất, đo độ phì đất hoặc khuyến nghị canh tác đã được kiểm chứng ngoài thực địa. Các điểm Soil HP, Water HP, Profit HP, Overall và Rank là điểm heuristic theo mô hình hiện tại, không phải xác suất thành công hay phép đo vật lý.

1. Bài toán và cách FarmShift giải quyết
Lựa chọn cây trồng không chỉ phụ thuộc vào một mùa: nhiệt độ, lượng mưa, độ ẩm, rét, nhu cầu nước, lợi nhuận và ảnh hưởng của cây vụ trước đều có thể thay đổi quyết định. Với 6 lựa chọn và 3 vụ, mô hình xét tối đa 6³ = 216 chuỗi luân canh ban đầu.

Quy trình chính:
1. Nhận vị trí và ưu tiên đất/nước/lợi nhuận của người dùng.
2. Lấy dữ liệu NASA POWER, kiểm tra đơn vị, dữ liệu thiếu, nguồn gốc và điều kiện sử dụng.
3. Đánh giá từng cây theo từng vụ bằng mô hình phù hợp sinh thái kiểu fuzzy; kiểm tra các điều kiện rét và loại trừ theo mô hình.
4. Tính điểm các phương án theo một hàm mục tiêu thống nhất.
5. Tối ưu và đối chiếu: Exhaustive Search tìm nghiệm tốt nhất theo mô hình; QAOA mô phỏng tìm nghiệm gần tốt nhất và được so sánh với nghiệm cổ điển.
6. Hiển thị kết quả và cảnh báo kèm giới hạn nguồn dữ liệu; không đưa ra khuyến nghị tại ruộng khi thiếu điều kiện an toàn dữ liệu.
Các mùa hiện được chia minh họa thành T1–T4, T5–T8, T9–T12; đây chưa phải lịch gieo trồng được hiệu chuẩn theo địa phương, giống và kỹ thuật canh tác.
2. Chạy dự án
Yêu cầu Node.js >= 18. Bản sửa từng được kiểm tra trên Node.js v26.13.1/Windows; chưa kiểm tra mọi phiên bản Node.
npm ci
npm start
Mở http://localhost:3000.

Các lệnh bổ sung:
npm run dev         # Chạy với Node watch mode
npm test            # Kiểm thử offline
npm run check       # Kiểm tra cú pháp
npm run benchmark   # Benchmark thuật toán
npm run check:live  # Kiểm tra nguồn online; cần mạng
Trong PowerShell, có thể đổi cổng bằng $env:PORT='3001' trước npm start; bật log bằng $env:DEBUG='1'. npm run dev không tự bật DEBUG.
3. Dữ liệu và kiểm soát chất lượng
- NASA POWER climatology: khí hậu lưới, chủ yếu từ MERRA-2; bức xạ từ sản phẩm CERES. Không đồng nghĩa đo trực tiếp tại ruộng hay dự báo thời tiết vụ tiếp theo.
- Đơn vị: lượng mưa mm/day được đổi sang mm/month; dữ liệu lịch sử dùng số ngày thực (cả năm nhuận), climatology dùng 28,25 ngày cho tháng Hai. Bức xạ kWh/m²/day được đổi sang MJ/m²/day khi cần.
- QC: giữ giá trị 0 hợp lệ; null, NaN, vô cực, sentinel, chuỗi rỗng và boolean được xem là thiếu. Có nội suy tuyến tính tối đa 2 tháng liên tiếp theo quy tắc của pipeline, với nhãn interpolated.
- Nguồn gốc: metadata từng biến/tháng ghi source, product, unit, period, status và limitations; confidence chưa hiệu chuẩn được để null, không gán xác suất tùy ý.
- Không trộn DEMO với dữ liệu thật: mockNasa và module fallback chỉ dùng demo/test. Khi NASA thiếu biến cốt lõi sau QC, API trả HTTP 503, recommendation_allowed=false, scenarios=null.
- SPI: sử dụng lịch sử mưa theo tháng/năm, tách khỏi climatology; loader lấy 31 năm hoàn chỉnh kết thúc năm trước để đảm bảo tối thiểu 30 mẫu theo tháng. Thiếu hoặc không đạt điều kiện: SPI trả null/unavailable, không tự tạo số liệu. SPI hiện được khớp và đánh giá trên cùng lịch sử, chưa phải dự báo ngoài mẫu.
- Chỉ số viễn thám trên dashboard: NDVI/ESI/erosion và phenology hiện là proxy từ khí hậu, chưa phải ảnh MODIS, ECOSTRESS hoặc dữ liệu SoilGrids thực. Bộ xử lý reflectance riêng yêu cầu surface reflectance, QA phù hợp và thông tin source/date.
- Machine learning: Random Forest đang dùng dữ liệu tổng hợp để thử nghiệm và không nằm trong /api/analyze; tỷ lệ vote không phải xác suất nông nghiệp đã hiệu chuẩn.
Nguồn tham khảo: NASA POWER Services, Climatology API, Monthly API, NCAR SPI.
4. Rét, độ cao và giới hạn địa lý
Dự án sử dụng T2M_MIN khí hậu, lịch sử nhiệt độ tối thiểu ngày 5 năm từ NASA POWER và DEM NASA/NGA SRTM thông qua Open Topo Data. Giao diện phân biệt rét theo tháng/ngày, Tmin âm, nguy cơ theo vụ và chênh cao giữa điểm chọn với ô khí hậu.

Chênh cao lớn hoặc thiếu đánh giá rét sẽ chặn khuyến nghị tại ruộng, nhưng có thể vẫn cho xem phân tích theo ô khí hậu. Nguy cơ rét không có nghĩa xác nhận cây chết. Mô hình giả định cây ngoài trời, không được bảo vệ, trong các vụ minh họa 4 tháng.

Phân loại biển/băng hiện dùng đa giác/hộp giới hạn đơn giản; chưa có bản đồ bờ biển/độ dốc hoàn chỉnh. Xem [báo cáo rét/độ cao](docs/RET_VA_DO_CAO_NASA.md) và [snapshot nguồn thật](docs/live-cold-check.json). Kiểm tra online bằng npm run check:live không thuộc npm test.
5. Tính điểm và tối ưu luân canh
Một hàm mục tiêu thống nhất — planning_rotation_v5
Các thành phần scoreScenario(), Classical Exhaustive Search và QAOA mô phỏng đều sử dụng evaluateRotationObjective() từ src/algorithms/13_crop_rotation_optimization_qaoa_milp.js.

Điểm mục tiêu kết hợp: điểm thành phần đất/nước/lợi nhuận theo hệ số cây, điểm phù hợp cây với điều kiện mùa, và thưởng/phạt chuyển cây giữa các vụ. Các trọng số đất/nước/lợi nhuận được chuẩn hóa; nếu tất cả bằng 0 thì chia đều. Các hệ số, ngưỡng và phân hạng vẫn là giả định heuristic chưa hiệu chuẩn nông học.

Soil HP và Water HP chủ yếu phản ánh hệ số đặc tính cây trong mô hình, không phải phép đo trực tiếp độ phì hay lượng nước sẵn có tại ruộng. Profit HP không phải dự báo lợi nhuận bằng tiền. Overall dùng cùng objective với bộ tối ưu; các điểm thành phần và Rank giúp giải thích kết quả nhưng không phải chỉ số xác suất.
Classical Exhaustive Search
Duyệt toàn bộ 216 chuỗi ban đầu, loại các phương án vi phạm ràng buộc của mô hình, sau đó chọn điểm objective cao nhất. Đây là nghiệm chuẩn chính xác theo mô hình hiện tại, không phải bằng chứng phương án tối ưu ngoài thực tế.
QUBO và QAOA mô phỏng
QUBO mã hóa lựa chọn cây theo biến nhị phân one-hot (18 biến = 3 vụ × 6 cây), cùng các tương tác chuyển vụ và hình phạt ràng buộc. Quy ước năng lượng:
E(x) = offset + Σ Q[i][i] x[i] + Σ(i<j) Q[i][j] x[i]x[j]
Với nghiệm one-hot hợp lệ, E = -objective_score / 100. Hình phạt được chọn để vi phạm không làm nghiệm toàn cục hấp dẫn hơn nghiệm hợp lệ. Ánh xạ Ising dùng x=(1-Z)/2; code có kiểm tra quan hệ năng lượng/pha trên 262.144 bitstring.

QAOA hiện chạy mô phỏng statevector trong không gian con one-hot khả thi, tối đa 216 amplitudes; khởi tạo đều, cost phase và mixer complete-graph theo từng vụ. Đây không phải mạch lượng tử 18 qubit được chạy trên phần cứng.

Thuật toán tìm tham số bằng lưới 24 × 16 cặp góc, tối thiểu hóa expected energy, rồi lấy 1.024 shots mặc định bằng bộ sinh ngẫu nhiên có seed. Báo cáo có objective, expected energy, feasibility, gap và thời gian chạy; so sánh với nghiệm Exhaustive Search. Gap 0 trên một fixture không chứng minh quantum advantage. Khi dùng fallback cổ điển, các cờ quantum_available=false, quantum_feasible=false, fallback_used=true; metric quantum không được giả lập thành kết quả thực.
6. API
POST /api/analyze nhận:
{"lat":21.03,"lon":105.85,"w_water":0.33,"w_soil":0.33,"w_profit":0.34,"user_crops":["Rice","Soybean","Maize"]}
lat/lon phải là số hữu hạn trong giới hạn tọa độ; các trọng số tùy chọn trong [0,1]; user_crops nếu cung cấp phải có đúng 3 tên hợp lệ. Trả HTTP 400 nếu đầu vào sai, HTTP 503 khi dữ liệu khí hậu cốt lõi không khả dụng.

Phản hồi giữ scenarios.baseline/user/quantum để tương thích giao diện. Kết quả Quantum ghi rõ hardware_executed=false, quantum_available, quantum_feasible, fallback_used và các metric mô phỏng. Metadata nguồn nằm trong variables; proxy có dependencies; SPI có status/reason/source/period/details. cold_risk, terrain và season_results giải thích rủi ro theo vụ. recommendation_allowed=false có thể xảy ra ngay cả khi ok=true để chỉ cho phép xem phân tích theo ô khí hậu.

GET /api/random_coords chỉ trả tọa độ gần vùng mẫu, không xác nhận đó là đất canh tác phù hợp. API cũng có endpoint kiểm tra runtime và nhận diện vị trí.
7. Cấu trúc mã nguồn
server.js                  Express và giao diện tĩnh
src/routes/api.js          Các endpoint API
src/services/              NASA/QC/provenance, scoring, rét/địa hình
src/algorithms/01..03       Provider, demo riêng, kiểm soát chất lượng
src/algorithms/04..07       QA, chỉ số viễn thám và chuỗi thời gian
src/algorithms/08..10       SPI, clustering, Random Forest thử nghiệm
src/algorithms/11..14       Fuzzy, MCDA, shared objective/QUBO/QAOA, validation
src/quantum/               Statevector và adapter QAOA
src/config/crops.js        Cây trồng, mùa vụ, hệ số và objective version
src/utils/                Helpers, phân loại địa lý, seeded RNG
public/                    Giao diện HTML/CSS/JS
tests/                     Kiểm thử offline và fixtures
scripts/                   Kiểm tra cú pháp, benchmark, đóng gói
docs/                      Báo cáo kỹ thuật và snapshot
8. Kiểm thử, độ tin cậy và lộ trình
npm test sử dụng fixture offline, bao gồm kiểm tra API, scoring, QUBO/QAOA và các trường hợp khí hậu đặc biệt. Một số test UI dùng DOM giả lập, không thay thế kiểm thử trình duyệt thực. SPI đã được đối chiếu với công thức gamma giải tích và tích phân số độc lập, chưa đối chiếu toàn bộ với ground truth thực địa. Các fixture vùng cực đoan không phải phép đo tại các tọa độ tương ứng.

Những gì nguyên mẫu chứng minh: pipeline xử lý dữ liệu có kiểm soát, hàm mục tiêu nhất quán giữa UI và bộ tối ưu, khả năng so sánh QAOA với nghiệm cổ điển theo cùng mô hình và khả năng báo cáo giới hạn dữ liệu.

Những gì chưa chứng minh: độ chính xác khuyến nghị ngoài đồng ruộng, dự báo năng suất/lợi nhuận, hiệu quả sử dụng nước đo thực tế, hoặc lợi thế tính toán lượng tử. Khi triển khai thực tế cần hiệu chuẩn theo giống/vùng/vụ, bổ sung dữ liệu đất và năng suất, kiểm chứng ngoài mẫu và theo dõi sai số.

Để tạo ZIP mã nguồn trên Windows:
npm run package:source
Lệnh tạo dist/FarmShiftRemake-fixed.zip từ danh sách nguồn cho phép; không đóng gói .git, node_modules hoặc .env. Người nhận cài lại bằng npm ci.
# Individual Report: Lab 3 - Chatbot vs ReAct Agent

- **Student Name**: Nguyễn Anh Tài
- **Student ID**: 2A202600388
- **Date**: 2026-04-06

---

## I. Technical Contribution (15 Points)

_Describe your specific contribution to the codebase (e.g., implemented a specific tool, fixed the parser, etc.)._

- **Modules Implementated**: [src/tools/weather_tool.py]
- **Code Highlights**: [
  days_diff = (target_date - today).days

  if days_diff == 0:
  return self.\_get_current_weather(city)
  elif 1 <= days_diff <= 13:
  return self.\_get_forecast_weather(city, days=days_diff)
  elif 14 <= days_diff <= 365:
  return self.\_get_future_weather(city, target_date)
  ]

- **Documentation**: [Công cụ này tương tác với vòng lặp ReAct thông qua cơ chế phản hồi lỗi chi tiết. Nếu LLM cung cấp định dạng ngày sai, thay vì làm chương trình bị crash, Tool trả về một thông báo lỗi JSON. Điều này ép Agent phải đọc Observation, nhận ra sai lầm trong bước Action trước đó và thực hiện lại bước Thought để sửa định dạng ngày]

---

## II. Debugging Case Study (10 Points)

_Analyze a specific failure event you encountered during the lab using the logging system._

- **Problem Description**: [Chỉ lấy được thời tiết ngày hôm nay thành công, nhưng khi người dùng hỏi về "ngày mai" hoặc một ngày cụ thể trong tương lai, Agent vẫn chỉ trả về dữ liệu của ngày hiện tại hoặc báo lỗi không tìm thấy dữ liệu Forecast.]

- **Log Source**: [Action: get_weather("Ho Chi Minh", "2026-04-07")

Observation: {"location": "Ho Chi Minh", "date": "2026-04-06", "temp": 30.5 ...}

Thought: Tôi thấy kết quả trả về vẫn là ngày 2026-04-06, tôi sẽ thử gọi lại..]

- **Diagnosis**: Lỗi nằm ở việc xử lý tham số days trong API Forecast của WeatherAPI.
  \*\* Tool Spec: Tham số days trong API yêu cầu số lượng ngày tính từ hôm nay. Nếu Agent truyền ngày 2026-04-07 nhưng Tool lại mặc định gọi endpoint current.json, dữ liệu sẽ luôn là ngày hôm nay.

  \*\* Logic Code: Trong phiên bản đầu tiên, hàm \_get_forecast_weather chưa cộng thêm 1 vào tham số days (API yêu cầu days=2 để lấy được dữ liệu của ngày mai), dẫn đến việc Observation trả về mảng dữ liệu không chứa ngày mục tiêu.

- **Solution**: Cập nhật logic trong WeatherTool để tính toán chính xác days_diff. Nếu days_diff > 0, hệ thống bắt buộc chuyển sang Endpoint forecast.json và truyền tham số days = days_diff + 1. Đồng thời thêm bước kiểm tra trong mảng forecastday để trích xuất đúng object có date khớp với yêu cầu của Agent.

---

## III. Personal Insights: Chatbot vs ReAct (10 Points)

_Reflect on the reasoning capability difference._

1.  **Reasoning**: Khối Thought đóng vai trò là "bộ não" kiểm soát. So với Chatbot thông thường (chỉ đoán từ tiếp theo), ReAct Agent sử dụng Thought để phân tách thực thể. Ví dụ: Khi nghe "cuối tuần tới", Agent phải suy luận ra ngày YYYY-MM-DD cụ thể trước khi điền vào Action. Đây là sự khác biệt giữa "nói chuyện phiếm" và "giải quyết vấn đề".

2.  **Reliability**: Agent thực hiện tệ hơn Chatbot trong trường hợp các câu hỏi đơn giản, mang tính chất hội thoại (ví dụ: "Chào bạn"). Khi đó, cấu trúc ReAct trở nên quá cồng kềnh, Agent có thể cố gắng tìm một Action không cần thiết, gây lãng phí tài nguyên và tăng độ trễ (latency).

3.  **Observation**: Phản hồi từ môi trường (Observation) là yếu tố thực tế giúp Agent "tỉnh ngộ". Trong Lab này, dữ liệu Mock (khi chưa config API Key) giúp tôi quan sát rõ nhất: Agent nhận thấy ghi chú "MOCK DATA", từ đó nó điều chỉnh câu trả lời Final để cảnh báo người dùng rằng đây chỉ là dữ liệu giả lập.

---

## IV. Future Improvements (5 Points)

_How would you scale this for a production-level AI agent system?_

- **Scalability**:Triển khai cơ chế Connection Pooling cho các request API. Khi hệ thống có hàng ngàn Agent chạy cùng lúc, việc mở/đóng kết nối HTTP liên tục sẽ gây nghẽn cổ chai.
- **Safety**: Xây dựng một lớp Input Validation sử dụng Pydantic để kiểm tra tên thành phố và định dạng ngày trước khi gọi API, giúp tiết kiệm chi phí gọi API và tránh các lỗi injection vào URL.
- **Performance**: Tích hợp Semantic Caching. Nếu nhiều người dùng cùng hỏi về thời tiết TP.HCM ngày mai, Agent có thể lấy kết quả từ Cache thay vì thực hiện lại toàn bộ vòng lặp ReAct và gọi API bên thứ ba.

---

> [!NOTE]
> Submit this report by renaming it to `REPORT_[YOUR_NAME].md` and placing it in this folder.

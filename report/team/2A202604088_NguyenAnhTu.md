# Báo cáo cá nhân: Nguyễn Anh Tú

## 1. Thông tin thành viên

| Thông tin | Chi tiết |
|---|---|
| Họ tên | Nguyễn Anh Tú |
| MSSV | 2A202604088 |
| Email | comtrang378@gmail.com |
| Vai trò | Phát triển giao diện người dùng (UI) |

## 2. Phần việc phụ trách

Phụ trách thiết kế và phát triển giao diện người dùng (UI) cho hệ thống Research Agent bằng Streamlit. Xây dựng giao diện tương tác giữa người dùng và AI Agent, hỗ trợ cấu hình kết nối với các nhà cung cấp mô hình ngôn ngữ lớn (LLM), gửi câu hỏi và hiển thị phản hồi từ Agent.

Bên cạnh đó, phụ trách hiển thị lịch sử hội thoại, thông tin về quá trình sử dụng công cụ (Tool Calling), quản lý trạng thái phiên làm việc và hỗ trợ tạo giao diện demo cho sản phẩm.

## 3. Đóng góp cụ thể

| STT | Công việc | Nội dung thực hiện |
|---:|---|---|
| 1 | Thiết kế giao diện | Sử dụng Streamlit xây dựng giao diện Research Agent với bố cục rộng, khu vực chat và thanh sidebar cấu hình. |
| 2 | Kết nối LLM Provider | Xây dựng chức năng lựa chọn và kết nối các Provider: OpenRouter, OpenAI, Anthropic, Gemini và Ollama. |
| 3 | Xây dựng giao diện Chat | Sử dụng `st.chat_input()` và `st.chat_message()` để tiếp nhận câu hỏi và hiển thị nội dung hội thoại giữa người dùng với Agent. |
| 4 | Quản lý hội thoại | Sử dụng `st.session_state` để lưu trữ lịch sử tin nhắn, trạng thái kết nối và thông tin phiên hội thoại. |
| 5 | Hiển thị Tool Calling | Xây dựng khu vực mở rộng bằng `st.expander()` để hiển thị chi tiết các công cụ mà Agent đã sử dụng. |
| 6 | Cấu hình Agent | Cho phép người dùng lựa chọn phiên bản prompt, điều chỉnh số lượt hội thoại được giữ trong ngữ cảnh và số vòng gọi Tool tối đa. |
| 7 | Hỗ trợ Demo | Bổ sung khu vực hướng dẫn truy cập ứng dụng thông qua localhost và hướng dẫn triển khai lên Streamlit Community Cloud. |

Ngoài ra, giao diện được tích hợp với các module xử lý phía Agent thông qua `run_model_tool_loop()`, giúp kết nối phần giao diện với quá trình xử lý yêu cầu và gọi công cụ của hệ thống.

## 4. Bằng chứng

### File triển khai chính

- `app.py`, 

### AI/công cụ đã dùng và cách tự kiểm tra kết quả

- Python: Ngôn ngữ lập trình chính để phát triển ứng dụng.
- Streamlit: Xây dựng giao diện người dùng và các thành phần tương tác.
- LLM Providers: Hỗ trợ kết nối với OpenRouter, OpenAI, Anthropic, Gemini và Ollama.
- GitHub: Quản lý mã nguồn và tích hợp sản phẩm trong repository của nhóm.


## 5. Tự đánh giá

- Trong quá trình thực hiện, tôi đã hoàn thành phần phát triển giao diện người dùng cho hệ thống Research Agent, tập trung vào việc xây dựng giao diện trực quan, dễ sử dụng và hỗ trợ kết nối với các mô hình AI khác nhau.
- Thông qua công việc này, tôi có cơ hội củng cố kiến thức về Python, Streamlit, quản lý trạng thái ứng dụng và cách tích hợp giao diện với hệ thống AI Agent có sử dụng cơ chế Tool Calling.
- Tuy nhiên, phần triển khai giao diện vẫn có thể được cải thiện thêm về trải nghiệm người dùng, khả năng hiển thị trạng thái xử lý và tối ưu giao diện khi triển khai trên môi trường thực tế.
- Tôi đánh giá phần việc của mình đã đóng góp vào việc hoàn thiện sản phẩm ở khía cạnh giao diện và khả năng tương tác giữa người dùng với hệ thống Research Agent.

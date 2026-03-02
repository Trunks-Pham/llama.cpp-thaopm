# llama.cpp Technical Demonstration Edition

Hướng dẫn build và chạy Local LLM Server sử dụng `llama.cpp` trên môi trường Linux, tương thích OpenAI API.

Model sử dụng: **LFM2.5 1.2B Instruct (GGUF Q8_0)**
Chạy hoàn toàn offline.

Dự án này là phiên bản cá nhân hóa và tối ưu triển khai từ llama.cpp (MIT License) do ggml-org phát triển.
README hiện tại tập trung vào cấu hình, triển khai và định hướng sử dụng cho AI Server.
Tài liệu kỹ thuật gốc của dự án ban đầu được giữ nguyên để developer có thể tham khảo đầy đủ:

[![Xem tài liệu llama.cpp](https://img.shields.io/badge/Xem%20tài%20liệu-llama.cpp-blue?style=for-the-badge)](./ALLSERVER.md)

## **Upstream Project**

*This repository is derived from the original llama.cpp project:*  
*https://github.com/ggml-org/llama.cpp*

*All original credits remain with the respective authors under the MIT License.*
---

# Yêu cầu hệ thống

* Ubuntu 22.04+ (hoặc Debian tương đương)
* RAM tối thiểu: 4GB
* CPU: 2–4 cores
* Không yêu cầu GPU (CPU mode)

---

# 1. Cài đặt dependencies

```bash
sudo apt update
sudo apt install -y build-essential cmake git curl
```

Kiểm tra CMake:

```bash
cmake --version
```

---

# 2. Clone llama.cpp

```bash
git clone https://github.com/ggml-org/llama.cpp.git
cd llama.cpp
```

---

# 3. Build llama.cpp (CPU mode)

```bash
cmake -B build
cmake --build build --config Release -j$(nproc)
```

Sau khi build xong, binary sẽ nằm ở:

```
build/bin/llama-server
```

---

# 4. Thêm Model GGUF

Tạo thư mục models:

```bash
mkdir models
```

Copy file model `.gguf` vào thư mục:

```
models/LFM2.5-1.2B-Instruct-Q8_0.gguf
```

Cấu trúc thư mục:

```
llama.cpp/
 ├── build/
 ├── models/
 │    └── LFM2.5-1.2B-Instruct-Q8_0.gguf
```

---

# 5. Chạy LLM Server

```bash
./build/bin/llama-server \
  -m models/LFM2.5-1.2B-Instruct-Q8_0.gguf \
  -c 2048 \
  -t $(nproc) \
  --host 0.0.0.0 \
  --port 8000
```

Nếu thành công sẽ thấy:

```
server is listening on http://0.0.0.0:8000
```

---

# 6. Test API

Kiểm tra model:

```bash
curl http://localhost:8000/v1/models
```

Test chat completion:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "LFM2.5-1.2B-Instruct-Q8_0.gguf",
    "messages": [
      {"role": "user", "content": "Xin chào, bạn là ai?"}
    ]
  }'
```

---

# 7. Gọi từ Python (OpenAI Compatible)

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="not-needed"
)

resp = client.chat.completions.create(
    model="LFM2.5-1.2B-Instruct-Q8_0.gguf",
    messages=[{"role":"user","content":"Hello"}]
)

print(resp.choices[0].message.content)
```

---

# Tối ưu hiệu năng

| Tùy chọn         | Ý nghĩa                |
| ---------------- | ---------------------- |
| `-t $(nproc)`    | Dùng toàn bộ CPU cores |
| `-c 2048`        | Context size           |
| `--host 0.0.0.0` | Cho phép truy cập LAN  |
| `--port 8000`    | Cổng server            |

---

# Yêu cầu RAM

Model 1.2B Q8_0:

* Model: ~1.2GB
* KV Cache: ~24MB
* Compute buffer: ~136MB
* Tổng: ~1.4GB RAM

Khuyến nghị: 4GB RAM trở lên.

---

# Truy cập từ máy khác trong LAN

Tìm IP Ubuntu:

```bash
ip a
```

Truy cập từ Windows:

```
http://<UBUNTU-IP>:8000
```

---

# Cấu trúc dự án

```
llama.cpp/
 ├── build/
 ├── models/
 │    └── LFM2.5-1.2B-Instruct-Q8_0.gguf
 ├── README.md
```

---

# Kết luận

Bạn đã triển khai thành công một Local LLM Server sử dụng llama.cpp trên Ubuntu VM, hoạt động tương thích OpenAI API và chạy hoàn toàn offline.

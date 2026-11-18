# Bài thực hành 1 – Tổng quan hệ thống và MEP

## Thông tin sinh viên

| Nội dung | Thông tin |
|---|---|
| Họ và tên | Đặng Quốc Trung |
| Mã sinh viên | DTC245200016 |
| Lớp | CNTT K23E |
| Học phần | Phát triển ứng dụng phân tán |
| Repository fork | `trungdtt123-cmd/CNTTK23E` |
| Nhánh | `bai1-dtc245200016` |
| Pull Request | `#16` – `Bai1 - dtc245200016 - Đặng Quốc Trung` |

Nhật ký thực hành chi tiết, phần trả lời câu hỏi và toàn bộ mô tả ảnh được trình bày tại [docs/bai1.md](docs/bai1.md).

## Mục tiêu bài thực hành

- Làm quen với .NET, C#, Git, REST Client và quy trình chạy một ứng dụng phân tán đơn giản.
- Tìm hiểu các mẫu trao đổi thông điệp (MEP): Request–Response, One-way, Queue-based, Publish–Subscribe và Streaming/Push.
- Xây dựng hệ thống EduSubmit gồm API và Client chạy ở hai tiến trình độc lập, giao tiếp qua HTTP/JSON.
- Thực hành thiết kế endpoint REST, kiểm tra dữ liệu đầu vào và sử dụng đúng các mã trạng thái HTTP.
- Kiểm thử trường hợp thành công, dữ liệu không hợp lệ, tài nguyên không tồn tại và lỗi kết nối.

## Công nghệ sử dụng

- C# và .NET SDK `10.0.102` (`net10.0`).
- ASP.NET Core Minimal API.
- `HttpClient` và `System.Net.Http.Json` trong ứng dụng Console.
- JSON, HTTP và REST.
- `ConcurrentDictionary<Guid, SubmissionResult>` để lưu tạm lượt nộp trong RAM.
- Git, GitHub, Visual Studio Code và extension REST Client.

## Chức năng đã thực hiện

### Submission.Api

- Kiểm tra trạng thái hoạt động của dịch vụ.
- Trả về danh sách hai bài tập mẫu.
- Tiếp nhận lượt nộp bài, kiểm tra các trường bắt buộc và trả về GUID.
- Trả `201 Created` kèm header `Location` khi tạo lượt nộp thành công.
- Trả `400 Bad Request` khi `assignmentId` không hợp lệ hoặc MSSV/URL repository bị rỗng.
- Lưu lượt nộp trong RAM và tra cứu lại theo GUID.
- Trả `404 Not Found` khi GUID không tồn tại hoặc không khớp ràng buộc route.

### Submission.Client

- Nhận URL API từ tham số dòng lệnh; mặc định là `http://localhost:5000`.
- Gọi API để lấy và hiển thị danh sách bài tập.
- Gửi một lượt nộp hợp lệ và hiển thị GUID, trạng thái trả về.
- Tra cứu lại lượt nộp vừa tạo theo GUID.
- Bắt `HttpRequestException` và hiển thị thông báo khi không kết nối được API.

## Cấu trúc thư mục

```text
Bai1/
├── README.md
├── EduSubmit.slnx
├── requests.http
├── docs/
│   ├── bai1.md
│   └── *.png
└── src/
    ├── Submission.Api/
    │   ├── Program.cs
    │   ├── Submission.Api.csproj
    │   ├── appsettings.json
    │   ├── appsettings.Development.json
    │   └── Properties/launchSettings.json
    └── Submission.Client/
        ├── Program.cs
        └── Submission.Client.csproj
```

Hai project được khai báo trong `EduSubmit.slnx` và đều nhắm đến `net10.0`.

## Yêu cầu môi trường

- Windows, Linux hoặc macOS có .NET SDK 10.
- Terminal PowerShell, Command Prompt hoặc shell tương đương.
- Visual Studio Code có extension REST Client nếu muốn chạy trực tiếp `requests.http`.
- Cổng `5000` còn trống khi chạy theo các lệnh bên dưới.

Các lệnh phải được thực hiện tại thư mục gốc `Bai1`.

## Khôi phục dependencies và build solution

Khôi phục dependencies:

```powershell
dotnet restore .\EduSubmit.slnx
```

Build toàn bộ solution:

```powershell
dotnet build .\EduSubmit.slnx --no-restore
```

Kết quả kiểm tra ngày 18/11/2026 trên .NET SDK `10.0.102`:

```text
Submission.Client -> ...\src\Submission.Client\bin\Debug\net10.0\Submission.Client.dll
Submission.Api    -> ...\src\Submission.Api\bin\Debug\net10.0\Submission.Api.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)
```

## Chạy chương trình

### 1. Khởi động API

Mở terminal thứ nhất tại thư mục `Bai1` và chạy:

```powershell
dotnet run --project .\src\Submission.Api\Submission.Api.csproj -- --urls http://localhost:5000
```

Kết quả mong đợi:

```text
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

`launchSettings.json` khai báo cổng HTTP mặc định `5126`. Tham số `--urls` trong lệnh trên chủ động chuyển API sang cổng `5000` để đồng bộ với `requests.http` và URL mặc định của Client.

### 2. Khởi động Client

Giữ API đang chạy, mở terminal thứ hai tại thư mục `Bai1` và chạy:

```powershell
dotnet run --project .\src\Submission.Client\Submission.Client.csproj -- http://localhost:5000
```

Client có thể chạy không cần tham số vì URL mặc định cũng là `http://localhost:5000`:

```powershell
dotnet run --project .\src\Submission.Client\Submission.Client.csproj
```

Kết quả thực tế gồm hai bài tập, thông báo nộp bài thành công, GUID mới, trạng thái `received` và thông tin lượt nộp được tra cứu lại theo cùng GUID.

## Kiểm thử bằng `requests.http`

1. Khởi động API tại `http://localhost:5000`.
2. Mở [requests.http](requests.http) bằng Visual Studio Code.
3. Nhấn **Send Request** lần lượt ở từng request.
4. Kiểm tra status code, JSON phản hồi và header `Location` của request POST hợp lệ.
5. Sau khi POST hợp lệ, sao chép trường `id` trong JSON trả về và thay GUID ở request tra cứu tồn tại trước khi gửi request đó.

Lưu ý: lượt nộp chỉ được lưu trong RAM. Mỗi lần khởi động lại API, GUID của phiên chạy trước không còn tồn tại; vì vậy GUID mẫu đang có trong `requests.http` có thể trả `404` và cần được thay bằng GUID vừa tạo.

## Danh sách endpoint thực tế

| Phương thức | Endpoint | Chức năng | Kết quả chính |
|---|---|---|---|
| `GET` | `/` | Kiểm tra trạng thái API | `200 OK` |
| `GET` | `/api/assignments` | Lấy danh sách bài tập | `200 OK` |
| `POST` | `/api/submissions` | Tạo lượt nộp bài | `201 Created` hoặc `400 Bad Request` |
| `GET` | `/api/submissions/{id:guid}` | Tra cứu lượt nộp theo GUID | `200 OK` hoặc `404 Not Found` |

Dữ liệu POST hợp lệ có dạng:

```json
{
  "assignmentId": 1,
  "studentCode": "DTC245200016",
  "repositoryUrl": "https://github.com/trungdtt123-cmd/EduSubmit-DTC245200016"
}
```

## Kết quả kiểm thử thực tế

Các kiểm thử dưới đây được chạy ngày 18/11/2026 với API tại `http://localhost:5000`.

| Trường hợp | Kết quả thực tế | Đánh giá |
|---|---|---|
| `GET /` | `200 OK`; `{"service":"Submission.Api","status":"running"}` | Đúng |
| `GET /api/assignments` | `200 OK`; trả hai bài tập có ID `1` và `2` | Đúng |
| POST dữ liệu hợp lệ | `201 Created`; có `Location`, GUID, `status: "received"`, `receivedAt` | Đúng |
| POST `assignmentId = 0`, MSSV và URL rỗng | `400 Bad Request`; `Du lieu nop bai khong hop le` | Đúng |
| GET theo GUID vừa tạo | `200 OK`; dữ liệu khớp lượt nộp vừa tạo | Đúng |
| GET GUID toàn số `0` | `404 Not Found`; body rỗng | Đúng |
| GET với `not-a-guid` | `404 Not Found` do không khớp route `{id:guid}` | Đúng |
| Chạy Client khi API hoạt động | Hiển thị hai bài tập, nộp bài và tra cứu lại thành công | Đúng |
| Chạy Client khi API đã dừng | Hiển thị `Loi HTTP: No connection could be made... actively refused it` | Đúng |

Sau kiểm thử, tiến trình API thử nghiệm đã được dừng và cổng `5000` được giải phóng.

## Ảnh minh chứng

Một số ảnh chính:

![Build solution EduSubmit thành công](docs/edusubmit-khoitao.png)

![API và Client giao tiếp thành công](docs/edusubmit-api-client.png)

![POST hợp lệ trả 201 Created](docs/edusubmit-rest-201.png)

![POST không hợp lệ trả 400 Bad Request](docs/edusubmit-rest-400.png)

![Tra cứu GUID không tồn tại trả 404](docs/edusubmit-get-submission-404.png)

Toàn bộ 23 ảnh minh chứng hiện có trong `docs/`:

- MEP: [Queue-based](docs/mep-queue.png), [Publish–Subscribe](docs/mep-pubsub.png).
- BasicWebApi: [GET 200](docs/basic-rest-200.png), [POST 201](docs/basic-rest-201.png), [POST 400](docs/basic-rest-400.png), [tiếng Nhật](docs/basicwebapi-ja.png), [request debug](docs/debug-request.png), [breakpoint](docs/debug-breakpoint.png).
- MiniDistributedSystem và thí nghiệm lỗi: [API–Client](docs/mini-api-client.png), [thêm bài tập](docs/loi-a-them-baitap.png), [tắt API](docs/loi-b-tat-api.png), [MSSV rỗng](docs/loi-c-studentcode-rong.png), [sai cổng](docs/loi-d-sai-cong.png), [khắc phục sai cổng](docs/loi-d-khac-phuc.png).
- EduSubmit: [khởi tạo và build](docs/edusubmit-khoitao.png), [GET danh sách](docs/edusubmit-api-get.png), [API–Client](docs/edusubmit-api-client.png), [GET 200](docs/edusubmit-rest-200.png), [POST 201](docs/edusubmit-rest-201.png), [POST 400](docs/edusubmit-rest-400.png), [Client tra cứu](docs/edusubmit-tra-cuu-client.png), [tra cứu 200](docs/edusubmit-get-submission-200.png), [tra cứu 404](docs/edusubmit-get-submission-404.png).

## Tổng kết

Bài thực hành đã xây dựng được một hệ thống phân tán nhỏ gồm API và Client độc lập, giao tiếp bằng HTTP/JSON theo MEP Request–Response. Các chức năng GET, POST, kiểm tra dữ liệu và tra cứu theo GUID hoạt động đúng trong lần kiểm thử hiện tại; solution restore/build thành công với `0 Warning` và `0 Error`. Dữ liệu lượt nộp đang được lưu trong RAM để phù hợp phạm vi bài thực hành, nên sẽ bị xóa khi API dừng.

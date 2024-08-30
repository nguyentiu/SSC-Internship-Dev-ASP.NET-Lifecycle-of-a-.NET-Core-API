# SSC-Internship-Dev-ASP.NET-Lifecycle-of-a-.NET-Core-API
# Hướng Dẫn Về Lifecycle của một .NET Core API
## Giới Thiệu
Khi phát triển một ứng dụng web hoặc API với ASP.NET Core, hiểu rõ về vòng đời của một API là điều quan trọng để có thể tối ưu hóa, xử lý lỗi, và quản lý tài nguyên hiệu quả. Mỗi yêu cầu (request) gửi đến API của bạn đều trải qua nhiều bước từ khi khởi tạo cho đến khi phản hồi được trả về.

Tổng Quan Về Vòng Đời của .NET Core API

Một .NET Core API có vòng đời bao gồm các bước chính sau đây:

1. `Application Startup`: Quá trình khởi động ứng dụng, bao gồm việc thiết lập cấu hình, đăng ký các dịch vụ, và khởi tạo các middleware.
2. `Request Pipeline`: Xử lý yêu cầu thông qua các middleware trước khi đến controller.
3. `Controller Action Execution`: Thực thi logic trong các hành động của controller và trả về kết quả.
4. `Response Generation`: Tạo và trả về phản hồi cho client.
5. `Application Shutdown`: Quá trình tắt ứng dụng và giải phóng tài nguyên.
## 1. Application Startup
Quá trình khởi động của một ứng dụng .NET Core được quản lý bởi hai tệp chính: `Program.cs` và `Startup.cs`. Trong phiên bản .NET 6, Program.cs được sử dụng để thiết lập và cấu hình ứng dụng.

Ví Dụ: Program.cs Cơ Bản

```csharp
var builder = WebApplication.CreateBuilder(args);

// Đăng ký các dịch vụ vào DI container
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Cấu hình middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```
Trong đoạn mã trên:

- `CreateBuilder` tạo một đối tượng `WebApplicationBuilder` dùng để cấu hình ứng dụng.
- `AddControllers` đăng ký dịch vụ MVC để sử dụng các controller.
- `UseSwagger` và `UseSwaggerUI` chỉ được kích hoạt trong môi trường phát triển để cung cấp tài liệu API.
## 2. Request Pipeline
Sau khi ứng dụng được khởi động, mỗi yêu cầu HTTP sẽ đi qua một chuỗi các middleware được đăng ký trong `Program.cs`. Middleware có thể thực hiện các tác vụ như xác thực, logging, xử lý lỗi, hoặc chuyển tiếp yêu cầu đến các thành phần tiếp theo trong pipeline.

Ví Dụ: Thêm Middleware Tùy Chỉnh

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Request: " + context.Request.Path);
    await next();
    Console.WriteLine("Response: " + context.Response.StatusCode);
});
```
Đoạn middleware này ghi lại đường dẫn của yêu cầu và mã trạng thái của phản hồi.

## 3. Controller Action Execution
Khi yêu cầu đã qua các middleware và đến controller, một hành động cụ thể trong controller sẽ được thực thi dựa trên URL và phương thức HTTP (GET, POST, PUT, DELETE). Controller là nơi chứa các logic chính để xử lý yêu cầu và chuẩn bị phản hồi.

Ví Dụ: Controller Cơ Bản

```csharp
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        })
        .ToArray();
    }
}
```
Đoạn mã này định nghĩa một hành động GET trả về một danh sách các đối tượng `WeatherForecast`.

## 4. Response Generation
Sau khi hành động của controller được thực thi, kết quả sẽ được chuyển đổi thành phản hồi HTTP và gửi lại cho client. Điều này có thể bao gồm mã trạng thái HTTP, headers, và nội dung phản hồi.

Ví Dụ: Trả Về Response Với Status Code

```csharp
[HttpGet("{id}")]
public IActionResult Get(int id)
{
    if (id < 0)
        return BadRequest("Invalid ID");

    var forecast = new WeatherForecast
    {
        Date = DateTime.Now,
        TemperatureC = 25,
        Summary = "Warm"
    };
    
    return Ok(forecast);
}
```
Trong ví dụ này, nếu `id` không hợp lệ, API trả về `BadRequest`; nếu hợp lệ, trả về `Ok` với đối tượng `WeatherForecast`.

## 5. Application Shutdown
Cuối cùng, khi ứng dụng cần được tắt hoặc khởi động lại, .NET Core sẽ thực hiện quy trình tắt ứng dụng và giải phóng các tài nguyên đã sử dụng. Điều này có thể bao gồm việc đóng kết nối cơ sở dữ liệu, hủy bỏ các tác vụ đang chạy, và ghi lại các thông tin log cuối cùng.
## 6. Tổng kết
Hiểu rõ vòng đời của một .NET Core API là bước đầu tiên quan trọng để phát triển các ứng dụng web mạnh mẽ và hiệu quả. Từ khởi tạo ứng dụng, xử lý yêu cầu, đến quản lý phản hồi và tài nguyên, mỗi bước trong vòng đời đều đóng vai trò quan trọng trong hiệu suất và khả năng mở rộng của ứng dụng.

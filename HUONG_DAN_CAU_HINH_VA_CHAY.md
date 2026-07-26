# Hướng dẫn cấu hình, setup và chạy ứng dụng

Hướng dẫn này dành cho cách chạy trực tiếp trên máy host, đặc biệt là Windows. Không cần Docker.

## 1. Yêu cầu

- JDK 21.
- MySQL 8.x đang chạy và có thể truy cập bằng TCP.
- Quyền truy cập Azure DevOps và PAT phù hợp.
- Kết nối mạng/VPN tới Azure DevOps nội bộ, Internal AI và Teams Workflow.
- Maven không bắt buộc vì project có Maven Wrapper.
- MySQL Workbench có thể dùng để tạo schema và chạy SQL.

H2 trong `pom.xml` chỉ có scope `test`; app chạy thật dùng MySQL. Việc Workbench kết nối được nhưng terminal báo `mysql: command not found` chỉ có nghĩa MySQL CLI chưa nằm trong `PATH`, không có nghĩa MySQL Server bị lỗi.

## 2. Thư mục làm việc

Module chạy ứng dụng:

```text
work_item_summary/
├── app_backend/
│   └── notifications/
│       ├── .env
│       ├── .env.example
│       ├── pom.xml
│       └── src/
├── docs/
└── script/
```

`application.yml` import `.env` bằng đường dẫn `./.env`. Vì vậy working directory phải là:

```text
<repository>\app_backend\notifications
```

Đây là nguyên nhân thường gặp khi chạy được bằng `java -jar` trong terminal nhưng lỗi bằng IntelliJ: IDE đang dùng working directory khác hoặc có environment variable cũ ghi đè `.env`.

## 3. Tạo lại MySQL schema

Project chỉ có một migration baseline:

```text
src/main/resources/db/migration/V1__create_application_schema.sql
```

Khi chủ động làm mới toàn bộ dữ liệu, chạy bằng MySQL Workbench:

```sql
DROP DATABASE IF EXISTS azure_bug_notifier;
CREATE DATABASE azure_bug_notifier
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

Khuyến nghị dùng user riêng:

```sql
CREATE USER IF NOT EXISTS 'azure_app'@'localhost'
  IDENTIFIED BY 'replace-with-strong-password';

GRANT ALL PRIVILEGES ON azure_bug_notifier.*
  TO 'azure_app'@'localhost';

FLUSH PRIVILEGES;
```

Nếu user đã tồn tại nhưng password cần thay:

```sql
ALTER USER 'azure_app'@'localhost'
  IDENTIFIED BY 'replace-with-strong-password';
```

Khi app start, Flyway tự tạo bốn bảng:

- `work_item_current`
- `work_item_processing`
- `polling_checkpoint`
- `ai_secret`

Không chạy `ddl-auto=create`; cấu hình hiện tại là `validate` để schema do Flyway quản lý.

## 4. Tạo file `.env`

Trong `app_backend\notifications`:

```powershell
Copy-Item .env.example .env
```

Mẫu đầy đủ theo source hiện tại:

```properties
# Server và MySQL
DB_URL=jdbc:mysql://127.0.0.1:3306/azure_bug_notifier?useSSL=false&allowPublicKeyRetrieval=true&connectionTimeZone=UTC&forceConnectionTimeZoneToSession=true&preserveInstants=true
DB_USERNAME=azure_app
DB_PASSWORD=replace-with-strong-password
SERVER_PORT=8080
APP_TIMEZONE=Asia/Ho_Chi_Minh
APP_LOG_LEVEL=INFO
MOCK_EXTERNAL_SERVICES=false

# Azure DevOps
AZURE_DEVOPS_BASE_URL=https://your-tfs-host.example/ads
AZURE_DEVOPS_COLLECTION=your-collection
AZURE_DEVOPS_PROJECT=your-project
AZURE_DEVOPS_PAT=replace-with-personal-access-token
AZURE_DEVOPS_API_VERSION=6.0
AZURE_DEVOPS_BUG_DESCRIPTION_FIELD=Nttdata.TERASOLUNA.Common.Description
AZURE_DEVOPS_ERROR_DESCRIPTION_FIELD=Nttdata.TERASOLUNA.Error.Issue

# Nhận diện team/module
MODULE_CATALOG_LOCATION=classpath:module-catalog.csv
SUB_TEAM_KEYWORD=電算
SUB_TEAM_LEADER_DISPLAY_NAME=trinh vu
SUB_TEAM_LEADER_COMPACT_NAME=trinhvu
SUB_TEAM_LEADER_ACCOUNT=bttrinhvu
SUB_TEAM_LEADER_DOMAIN_ACCOUNT=macx\\bttrinhvu

# Internal AI
INTERNAL_AI_BASE_URL=https://d3aq9ev4ohhkz3.cloudfront.net
INTERNAL_AI_RESPONSES_ENDPOINT=/v1/responses
INTERNAL_AI_REFRESH_ENDPOINT=/v1/auth/refresh
INTERNAL_AI_USER_EMAIL=replace-with-token-email
INTERNAL_AI_WORKSPACE=work_item_summary
INTERNAL_AI_INSTANCE_ID=work_item_summary
INTERNAL_AI_SESSION=
INTERNAL_AI_MODE=auto
INTERNAL_AI_REFRESH_ENABLED=true
INTERNAL_AI_REFRESH_BEFORE_EXPIRY=72h
INTERNAL_AI_REFRESH_RETRY_DELAY=15m
INTERNAL_AI_REFRESH_CHECK_DELAY_MS=3600000
INTERNAL_AI_REFRESH_CHECK_INITIAL_DELAY_MS=30000
INTERNAL_AI_AVAILABLE_FROM=08:00
INTERNAL_AI_AVAILABLE_UNTIL=18:30
INTERNAL_AI_LATEST_REQUEST_START=18:25

# Microsoft Teams
TEAMS_WORKFLOW_WEBHOOK_URL=https://replace-with-full-signed-workflow-url
TEAMS_PIC_ASSIGNMENT_CATALOG_LOCATION=classpath:project_PIC.csv
TEAMS_PROFILE_CATALOG_LOCATION=classpath:ms_teams_profile_info.csv
TEAMS_MAX_MENTIONS=20

# Scheduler và retry
WORK_ITEM_POLLING_ENABLED=true
WORK_ITEM_POLLING_CRON=0 */5 8-18 * * *
WORK_ITEM_POLLING_BATCH_SIZE=200
MORNING_RECOVERY_ENABLED=true
MORNING_RECOVERY_CRON=0 0 8 * * *
WORK_ITEM_PROCESSING_ENABLED=true
WORK_ITEM_PROCESSING_DELAY_MS=1000
WORK_ITEM_PROCESSING_BATCH_SIZE=50
AI_MAX_CONCURRENT_REQUESTS=5
AI_ANALYSIS_VERSION=v1
TEAMS_DELIVERY_DELAY_MS=1000
MAX_RETRIES=3

# API nội bộ
INTERNAL_API_KEY=replace-with-random-internal-api-key
```

Lưu ý:

- Không commit `.env`. File này đã được ignore.
- Không thêm dấu cách quanh `=`.
- Trong `.env`, domain account cần `\\` như mẫu.
- `INTERNAL_AI_USER_EMAIL` phải trùng chính xác claim `email` trong JWT, không phải email PIC Teams.
- `INTERNAL_AI_SESSION` rỗng sẽ tự thành `{instance-id}/{workspace}`.
- `TEAMS_WORKFLOW_WEBHOOK_URL` phải là URL đầy đủ, gồm cả phần sau dấu `?`.
- `TEAMS_WEBHOOK_URL` chỉ là alias tương thích cấu hình cũ; ưu tiên dùng `TEAMS_WORKFLOW_WEBHOOK_URL`.
- Giá trị `TEAMS_SEND_FALLBACK_ON_AI_FAILURE` hiện chưa được sử dụng trong code; không dựa vào biến này để gửi card khi AI lỗi.

Spring ưu tiên environment variable của process cao hơn file `.env`. Nếu IntelliJ từng cấu hình `DB_PASSWORD`, `AZURE_DEVOPS_PAT` hoặc URL cũ trong Run Configuration, giá trị đó sẽ ghi đè file.

## 5. Kiểm tra hai field description

Mặc định hiện tại:

```properties
AZURE_DEVOPS_BUG_DESCRIPTION_FIELD=Nttdata.TERASOLUNA.Common.Description
AZURE_DEVOPS_ERROR_DESCRIPTION_FIELD=Nttdata.TERASOLUNA.Error.Issue
```

Nếu response trên server thật dùng prefix khác, lấy đúng key trong object `fields` và thay biến tương ứng. Không dùng label hiển thị trên giao diện Azure; phải dùng reference name trong JSON.

Nếu field sai hoặc không tồn tại, description trở thành rỗng và Work Item bị `IGNORED_EMPTY_DESCRIPTION`.

## 6. Cấu hình catalog module và PIC

Ba file mặc định nằm trong `src/main/resources`.

### 6.1 `module-catalog.csv`

```csv
repository,module
lmvis-link,a01
lmvis-online,ServerLib
```

Quy tắc:

- đúng hai cột;
- không để trống;
- dòng đầu là `repository,module`;
- parser file này không hỗ trợ repository/module chứa dấu phẩy.

### 6.2 `project_PIC.csv`

```csv
repository,module,pic
"lmvis-link","a01","Linh Phạm"
"lmvis-online","ServerLib","Trung"
```

Quy tắc:

- đúng ba cột;
- mọi `(repository,module)` phải tồn tại trong `module-catalog.csv`;
- không được trùng `(repository,module)`;
- `pic` là khóa logic để nối sang profile;
- CSV có hỗ trợ quoted field và dấu `""` để escape dấu `"`.

Một module có trong catalog nhưng thiếu assignment vẫn cho app start; khi gửi card, app mention toàn bộ profile.

### 6.3 `ms_teams_profile_info.csv`

```csv
Pic,DisplayName,Email
Linh Phạm,"Teams Display Name",account@company.example
Trung,"Another Display Name",another@company.example
```

`Email` phải là UPN/email của chính account Teams muốn tag. `DisplayName` phải là tên hiển thị mong muốn trong `<at>...</at>`.

Quy tắc:

- đúng ba cột;
- `Pic` không trùng sau khi chuẩn hóa;
- `Email`/UPN không trùng và phải chứa `@`;
- số profile không vượt `TEAMS_MAX_MENTIONS`, tối đa 20.

Catalog được đọc khi start. Sửa CSV cần build/restart lại app nếu dùng file nằm trong JAR.

## 7. Tạo và nạp token AI

### 7.1 Lấy token bằng device login

Từ repository root trên Windows PowerShell, nên lưu file tạm ngoài Git:

```powershell
$tokenFile = Join-Path $env:TEMP 'clauxet.token.txt'
$metaFile = Join-Path $env:TEMP 'clauxet.token.json'

.\script\gen_jwt_clauxet.ps1 `
  -TokenFile $tokenFile `
  -MetaFile $metaFile
```

Mở URL được script in ra, đăng nhập và chờ `Login success`.

Ứng dụng không đọc token từ file. Copy giá trị token vào DB bằng Workbench, sau đó xóa file tạm:

```sql
UPDATE ai_secret
SET status = 'REVOKED'
WHERE status = 'ACTIVE';

INSERT INTO ai_secret (token, status)
VALUES ('paste-full-jwt-here', 'ACTIVE');
```

```powershell
Remove-Item $tokenFile, $metaFile
```

Không gửi token vào log, ảnh chụp, commit hoặc tài liệu.

Ở lần đọc đầu, app decode payload và tự điền `user_email`, `issued_at`, `expires_at`. Token không dùng được nếu email không trùng cấu hình hoặc `exp` đã hết hạn.

### 7.2 Refresh tự động

Mặc định app:

- kiểm tra token sau 30 giây từ lúc start;
- sau đó kiểm tra mỗi giờ;
- refresh khi token còn không quá 72 giờ;
- nếu refresh chủ động lỗi nhưng token cũ còn hạn, thử lại sau 15 phút;
- ép refresh ngay nếu Responses API trả 401.

Theo dõi:

```sql
SELECT
  id,
  status,
  user_email,
  issued_at,
  expires_at,
  last_refreshed_at,
  next_refresh_attempt_at,
  refresh_failure_count,
  last_refresh_error
FROM ai_secret
ORDER BY id DESC;
```

## 8. Tạo internal API key

`INTERNAL_API_KEY` bảo vệ endpoint `/api/internal/**`; nó không liên quan Azure PAT, AI token hoặc Teams webhook.

Từ repository root:

```powershell
.\script\gen_internal_api_key.ps1
```

Copy chuỗi sinh ra vào `.env`:

```properties
INTERNAL_API_KEY=generated-random-value
```

Không dùng giá trị mặc định `change-me` hoặc placeholder trong `.env.example`.

## 9. Build và test

Mở PowerShell tại module:

```powershell
Set-Location .\app_backend\notifications
.\mvnw.cmd clean test
.\mvnw.cmd clean package
```

Test dùng H2 in-memory ở MySQL compatibility mode, tắt Flyway và tất cả scheduler. H2 không được đóng gói làm production database.

JAR:

```text
target\azure-bug-notifier-0.0.1-SNAPSHOT.jar
```

## 10. Chạy bằng terminal

Tại `app_backend\notifications`:

```powershell
java -jar .\target\azure-bug-notifier-0.0.1-SNAPSHOT.jar
```

Không cần truyền `--spring.profiles.active=local`. Profile `local` đặt `MOCK_EXTERNAL_SERVICES=true`, chỉ phù hợp khi cố ý không gọi dịch vụ thật.

Sau khi thấy `Started AzureBugNotifierApplication`, kiểm tra:

```powershell
$headers = @{ 'X-Internal-Api-Key' = 'your-internal-api-key' }
Invoke-RestMethod `
  -Uri 'http://localhost:8080/api/internal/health/dependencies' `
  -Headers $headers
```

## 11. Chạy bằng IntelliJ IDEA

Tạo Spring Boot/Application Run Configuration:

| Mục | Giá trị |
|---|---|
| Main class | `com.example.azurebugnotifier.AzureBugNotifierApplication` |
| JRE/SDK | Java 21 |
| Working directory | `<repository>\app_backend\notifications` |
| Active profiles | Để trống khi gọi dịch vụ thật |

Kiểm tra phần Environment variables và xóa giá trị cũ không cần thiết. Đặc biệt, biến môi trường tại Run Configuration sẽ ưu tiên hơn `.env`.

Nếu terminal chạy được nhưng IntelliJ báo MySQL 1045, gần như chắc chắn cần đối chiếu:

- working directory;
- `DB_USERNAME`/`DB_PASSWORD` trong Run Configuration;
- profile đang active;
- JRE đang sử dụng.

## 12. Khi nào job chạy

App không poll ngay chỉ vì vừa start.

Với cấu hình mặc định:

- polling chạy mỗi 5 phút trong các giờ 08–18 theo `APP_TIMEZONE`;
- morning recovery chạy 08:00, reset job treo quá 30 phút rồi poll;
- AI và Teams dispatcher bắt đầu kiểm tra DB ngay, sau đó lặp theo fixed delay;
- token maintenance bắt đầu sau 30 giây.

Tại 08:00, polling và morning recovery có thể cùng được trigger nhưng `poll()` được đồng bộ trong một JVM nên không chạy song song.

Nếu muốn poll từ 08:00 đến trước 23:00:

```properties
WORK_ITEM_POLLING_CRON=0 */5 8-22 * * *
INTERNAL_AI_AVAILABLE_FROM=08:00
INTERNAL_AI_AVAILABLE_UNTIL=23:00
INTERNAL_AI_LATEST_REQUEST_START=22:55
```

Cron trên chạy lần cuối lúc 22:55. `AVAILABLE_UNTIL` và `LATEST_REQUEST_START` đều có biên trên loại trừ: đúng 23:00 không còn trong window.

Nếu chỉ muốn tạm tắt từng phần trước khi start:

```properties
WORK_ITEM_POLLING_ENABLED=false
MORNING_RECOVERY_ENABLED=false
WORK_ITEM_PROCESSING_ENABLED=false
INTERNAL_AI_REFRESH_ENABLED=false
```

`WORK_ITEM_PROCESSING_ENABLED=false` tắt cả AI dispatcher và Teams dispatcher.

## 13. Theo dõi bằng DB

### 13.1 Polling

```sql
SELECT
  job_name,
  status,
  last_started_at,
  last_completed_at,
  cursor_changed_date,
  cursor_work_item_id,
  error_message
FROM polling_checkpoint;
```

`cursor_changed_date` là `System.ChangedDate` cuối cùng đã xử lý, không phải thời điểm app chạy.

`polling_checkpoint.status`:

| Status | Ý nghĩa |
|---|---|
| `RUNNING` | Một lần polling đã bắt đầu |
| `COMPLETED` | Lần polling kết thúc thành công |
| `FAILED` | Lần polling lỗi; xem `error_message` |

### 13.2 Work Item hiện tại

```sql
SELECT
  work_item_id,
  latest_revision,
  work_item_type,
  state,
  sub_team_detection_status,
  repository_name,
  module_name,
  module_detection_status,
  tracking_status,
  changed_date
FROM work_item_current
ORDER BY changed_date DESC;
```

Các record cần kiểm tra thủ công:

```sql
SELECT
  work_item_id,
  title,
  sub_team_detection_sources,
  sub_team_match_evidence,
  module_detection_status,
  module_match_evidence,
  work_item_url
FROM work_item_current
WHERE tracking_status = 'AWAITING_MANUAL_REVIEW';
```

### 13.3 Hàng đợi AI/Teams

```sql
SELECT
  status,
  COUNT(*) AS total
FROM work_item_processing
GROUP BY status
ORDER BY status;
```

```sql
SELECT
  id,
  work_item_id,
  revision,
  analysis_version,
  status,
  retry_count,
  teams_retry_count,
  next_processing_at,
  ai_processed_at,
  teams_sent_at,
  error_message
FROM work_item_processing
ORDER BY id DESC
LIMIT 100;
```

Ý nghĩa nhanh:

| Status | Cách hiểu |
|---|---|
| `PENDING` | Chờ AI |
| `PROCESSING` | AI worker đang xử lý |
| `WAITING_FOR_AI` | Ngoài khung giờ AI |
| `WAITING_FOR_AI_AUTH` | Không có JWT dùng được |
| `AI_COMPLETED` | AI và payload xong, chờ Teams |
| `TEAMS_SENDING` | Đang gọi webhook |
| `TEAMS_FAILED` | Teams lỗi |
| `FAILED` | AI lỗi |
| `COMPLETED` | Webhook đã trả HTTP không lỗi |
| `SUPERSEDED` | Description/ownership không còn là bản hiện tại |

## 14. Quy ước ngày giờ

- Mọi `Instant` được ghi DB theo UTC.
- JDBC ép MySQL session về `+00:00`.
- Scheduler, log và card Teams dùng `APP_TIMEZONE`.
- Card hiển thị dạng `dd/MM/yyyy HH:mm:ss +offset (ZoneId)`.

Trong Workbench:

```sql
SELECT @@session.time_zone, @@global.time_zone;
SET time_zone = '+00:00';
```

Đổi UTC sang giờ Việt Nam để xem:

```sql
SELECT
  created_at AS created_at_utc,
  CONVERT_TZ(created_at, '+00:00', '+07:00') AS created_at_vietnam
FROM work_item_processing
ORDER BY id DESC
LIMIT 20;
```

Không tự cộng thêm 7 giờ vào dữ liệu khi insert. Với schema cũ từng ghi local time, app không tự chuyển đổi các row lịch sử; cách an toàn của quá trình setup hiện tại là drop schema và để Flyway tạo lại.

## 15. Dừng ứng dụng

Nhấn Stop trong IntelliJ hoặc `Ctrl+C` bắt đầu graceful shutdown:

- scheduler không nhận thêm vòng mới;
- AI pool chờ tối đa 30 giây cho request đang chạy;
- sau đó interrupt và chờ thêm tối đa 10 giây.

Một Teams webhook đã trả accepted có thể tiếp tục chạy trong Power Automate, nên message vẫn có thể xuất hiện sau khi Java process đã dừng. Điều đó không chứng minh app còn chạy.

Nếu nghi còn process Java khác:

```powershell
Get-CimInstance Win32_Process |
  Where-Object { $_.Name -eq 'java.exe' } |
  Select-Object ProcessId, CommandLine
```

Chỉ dừng đúng PID của ứng dụng:

```powershell
Stop-Process -Id <PID>
```

## 16. Xử lý sự cố thường gặp

### MySQL `Access denied for user`

- Test đúng host/port/user/password bằng Workbench.
- Kiểm tra biến môi trường IntelliJ có ghi đè `.env`.
- Kiểm tra working directory.
- Kiểm tra account MySQL theo host:

```sql
SELECT user, host, plugin
FROM mysql.user
WHERE user IN ('root', 'azure_app');
```

### Terminal báo `mysql: command not found`

Workbench vẫn có thể kết nối bình thường. Dùng Workbench hoặc cài/thêm MySQL CLI vào `PATH`; lỗi này không liên quan JDBC của app.

### Azure báo API version ngoài phạm vi

Đặt:

```properties
AZURE_DEVOPS_API_VERSION=6.0
```

Không dùng 7.0 trên server chỉ hỗ trợ tối đa 6.1.

### Postman gọi được nhưng Java lỗi DNS trên Windows

Source hiện đã ép cả Azure, AI và Teams WebClient dùng JVM/OS resolver thay cho Netty async resolver. Vẫn cần:

- kết nối đúng VPN;
- kiểm tra DNS của Windows;
- restart app sau khi đổi mạng/VPN;
- so sánh hostname trong startup log.

Không cần thêm JVM DNS resolver flag cho phiên bản source hiện tại.

### AI job ở `WAITING_FOR_AI_AUTH`

Kiểm tra:

- có record `ACTIVE`;
- JWT có đúng ba segment;
- claim `email` trùng `INTERNAL_AI_USER_EMAIL`;
- `exp` còn hạn;
- `last_refresh_error`.

### AI job ở `WAITING_FOR_AI`

Đây là trạng thái bình thường ngoài khoảng từ `INTERNAL_AI_AVAILABLE_FROM` đến trước `INTERNAL_AI_LATEST_REQUEST_START`.

### AI trả về nhưng job `FAILED`

Kiểm tra `error_message` và log. Các lỗi thường gặp:

- stream đóng trước event `done`;
- output không phải JSON;
- thiếu `summary` hoặc `suggestion`;
- timeout;
- HTTP 429/5xx đã retry hết.

### Teams trả 401

- Copy nguyên signed URL, gồm `api-version`, `sp`, `sv`, `sig`.
- Không dùng URL đã revoke hoặc workflow đã tắt.
- Kiểm tra startup log có `teamsWebhookConfigured=true` và `teamsWebhookHasQuery=true`.
- Nếu set URL trực tiếp trong PowerShell, bọc toàn URL bằng dấu nháy đơn để `&` không bị shell xử lý.

### DB đã `COMPLETED` nhưng Teams chưa thấy message

`COMPLETED` chỉ nghĩa webhook trả HTTP không lỗi. Kiểm tra Run History của Power Automate, bước post card vào Teams, channel/chat đích và quyền của workflow owner.

### CSV làm app không start

Đọc exception đầu tiên liên quan catalog. Thường do:

- header sai;
- cột rỗng;
- duplicate module/PIC/UPN;
- assignment trỏ tới module không có trong `module-catalog.csv`;
- số profile vượt 20.

### Warning `sun.misc.Unsafe::allocateMemory`

Đây là warning từ Netty/JDK về API sẽ bị loại bỏ trong tương lai, không phải lỗi MySQL và không làm app stop. Dùng đúng JDK 21 của project và chỉ nâng dependency cùng phiên bản Spring Boot đã kiểm thử.

## 17. Checklist trước khi chạy thật

- [ ] JDK 21 và working directory đúng.
- [ ] MySQL schema mới đã được tạo.
- [ ] `.env` không còn placeholder.
- [ ] IntelliJ không có environment variable cũ ghi đè.
- [ ] Azure API version là 6.0 và PAT còn hạn.
- [ ] Hai description field đúng với JSON thật.
- [ ] Ba CSV hợp lệ và mapping PIC đúng.
- [ ] Full signed Teams webhook URL đã được cấu hình.
- [ ] `INTERNAL_API_KEY` là giá trị ngẫu nhiên.
- [ ] Có JWT `ACTIVE`, email và `exp` hợp lệ.
- [ ] VPN/DNS truy cập được ba external service.
- [ ] `.\mvnw.cmd clean test` thành công.
- [ ] Dependency health trả kết quả mong đợi.

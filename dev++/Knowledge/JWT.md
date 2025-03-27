# JWT Toàn Tập: Hướng dẫn Chi tiết từ A-Z cho Lập trình viên

Trong kỷ nguyên của các ứng dụng web phân tán, microservices và single-page applications (SPA), việc xác thực và ủy quyền người dùng một cách hiệu quả và an toàn trở thành một thách thức lớn. Các phương pháp truyền thống dựa trên session lưu trữ phía server đôi khi bộc lộ những hạn chế về khả năng mở rộng và quản lý trạng thái. Đây là lúc **JSON Web Token (JWT)** bước vào và cung cấp một giải pháp mạnh mẽ, linh hoạt.

Bài viết này là cẩm nang toàn diện, đi từ những khái niệm cơ bản nhất đến các kỹ thuật bảo mật nâng cao, giúp bạn hiểu sâu và áp dụng JWT một cách chính xác và an toàn trong các dự án của mình. Hãy coi JWT như một **"chứng minh thư kỹ thuật số" (digital ID card)**: nó chứa thông tin về bạn (claims), được cấp bởi một cơ quan uy tín (server), và có thể được xác minh dễ dàng nhờ con dấu chống giả mạo (chữ ký số).

## Phần 1: Khái niệm Cơ bản và Cấu trúc của JWT

### 1. JWT là gì và Dùng để làm gì?

JWT (phát âm là /dʒɒt/) là một **tiêu chuẩn mở (RFC 7519)**, định nghĩa một phương thức **nhỏ gọn (compact)** và **khép kín (self-contained)** để truyền tải thông tin an toàn giữa các bên dưới dạng một đối tượng JSON. "Khép kín" nghĩa là bản thân token đã chứa đủ thông tin cần thiết (ví dụ: thông tin người dùng) mà không yêu cầu server phải tra cứu thêm ở đâu khác (mặc dù có thể). "An toàn" ở đây chủ yếu đề cập đến việc thông tin có thể được **xác minh tính toàn vẹn** và **nguồn gốc** nhờ vào chữ ký số.

Các trường hợp sử dụng chính của JWT bao gồm:

*   **Xác thực (Authentication):** Đây là ứng dụng phổ biến nhất. Sau khi người dùng đăng nhập thành công, server cấp một JWT. Client lưu lại và gửi kèm JWT này trong các yêu cầu sau để chứng minh mình đã được xác thực. Server chỉ cần kiểm tra chữ ký để tin tưởng yêu cầu đó.
*   **Ủy quyền (Authorization):** JWT có thể chứa thông tin về vai trò (roles) hoặc quyền hạn (permissions) của người dùng. Server có thể dựa vào đó để quyết định người dùng có được phép truy cập tài nguyên hoặc thực hiện hành động cụ thể hay không.
*   **Trao đổi thông tin (Information Exchange):** JWT là một cách an toàn để truyền thông tin giữa các dịch vụ (ví dụ: trong kiến trúc microservices) vì chữ ký đảm bảo dữ liệu không bị thay đổi và đến từ nguồn đáng tin cậy.

### 2. Giải phẫu JWT: Cấu trúc 3 phần Vàng

Mọi JWT đều có cấu trúc gồm 3 phần, ngăn cách bởi dấu chấm (`.`), mỗi phần được mã hóa bằng **Base64Url Encoding** (an toàn cho URL, khác Base64 chuẩn một chút ở ký tự `+`, `/` và padding `=`):

`HEADER.PAYLOAD.SIGNATURE`

**Ví dụ một chuỗi JWT điển hình:**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJleHAiOjE1MTYyNDI2MjJ9.4BGe9hAduz0Yg-Eq3-aPAsn3GP14qh4UTbxJ53X0j58
```

Hãy cùng "mổ xẻ" từng thành phần:

#### 2.1. Header (Tiêu đề)

*   **Mục đích:** Chứa thông tin về cách tạo ra chữ ký và loại token.
*   **Nội dung:** Là một đối tượng JSON, thường có 2 trường chính:
    *   `alg` (Algorithm): Thuật toán được sử dụng để tạo chữ ký (Signature). Ví dụ: `HS256`, `RS256`, `ES256`. Đây là trường **bắt buộc**.
    *   `typ` (Type): Loại token, thường luôn là `"JWT"`.
*   **Ví dụ (JSON gốc):**
    ```json
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```
*   **Mã hóa:** JSON này được mã hóa Base64Url thành phần đầu tiên:
    `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

#### 2.2. Payload (Tải trọng - Nơi chứa Claims)

*   **Mục đích:** Chứa các "Claims" (tuyên bố) - là các mẩu thông tin về chủ thể (thường là người dùng) và các dữ liệu ngữ cảnh khác.
*   **Nội dung:** Là một đối tượng JSON chứa các cặp key-value (claims). Có 3 loại claims:
    *   **Registered Claims:** Các tên claim được định nghĩa sẵn trong chuẩn RFC 7519. Chúng không bắt buộc nhưng được *khuyến nghị mạnh mẽ* vì tính chuẩn hóa và tương thích. Một số claims quan trọng:
        *   `iss` (Issuer): Ai là người/hệ thống đã phát hành token này (ví dụ: `"https://myauth.service.com"`).
        *   `sub` (Subject): **Ai** là chủ thể mà token này đại diện (thường là User ID duy nhất, ví dụ: `"auth0|123456789"` hoặc một UUID). Đây là claim **then chốt** để định danh người dùng.
        *   `aud` (Audience): Token này dành cho **ai/dịch vụ nào** (ví dụ: URL của API mà client sẽ gọi: `"https://api.mydomain.com"`). Server nhận token *nên* kiểm tra xem mình có nằm trong danh sách `aud` không.
        *   `exp` (Expiration Time): Thời điểm (Unix Timestamp tính bằng giây) mà token **hết hạn**. Server **phải** kiểm tra claim này và từ chối token đã hết hạn. Ví dụ: `1711533600`.
        *   `nbf` (Not Before): Thời điểm (Unix Timestamp) mà token bắt đầu có hiệu lực. Token sẽ bị từ chối *trước* thời điểm này.
        *   `iat` (Issued At): Thời điểm (Unix Timestamp) mà token được phát hành. Hữu ích để biết "tuổi" của token. Ví dụ: `1711530000`.
        *   `jti` (JWT ID): Một định danh duy nhất (UUID) cho chính token này. Rất hữu ích cho việc theo dõi và đặc biệt là cho các chiến lược thu hồi token (revocation).
    *   **Public Claims:** Các claim do người dùng tự định nghĩa nhưng cần tránh xung đột tên. Cách tốt nhất là đặt tên dưới dạng URI hoặc sử dụng namespace. Ví dụ: `"https://mycompany.com/claims/user_level": "gold"`.
    *   **Private Claims:** Các claim tự định nghĩa, được tạo ra để chia sẻ thông tin giữa các bên đã có thỏa thuận ngầm với nhau (ví dụ: giữa client và server của cùng một ứng dụng). Có thể đặt tên tùy ý nhưng nên tránh trùng với Registered Claims. Ví dụ: `"role": "administrator"`, `"department": "Sales"`.
*   **Ví dụ (JSON gốc):**
    ```json
    {
      "sub": "user-9876",
      "name": "Jane Doe", // Private Claim
      "role": ["editor", "publisher"], // Private Claim
      "iss": "https://secure.myapp.com",
      "aud": "https://api.myapp.com",
      "iat": 1678886400, // March 15, 2023 12:00:00 PM UTC
      "exp": 1678890000, // March 15, 2023 01:00:00 PM UTC (1 hour later)
      "jti": "f8a4b1e2-c3d4-5e6f-7a8b-9c0d1e2f3a4b"
    }
    ```
*   **Mã hóa:** JSON này được mã hóa Base64Url thành phần thứ hai:
    `eyJzdWIiOiJ1c2VyLTk4NzYiLCJuYW1lIjoiSmFuZSBEb2UiLCJyb2xlIjpbImVkaXRvciIsInB1Ymxpc2hlciJdLCJpc3MiOiJodHRwczovL3NlY3VyZS5teWFwcC5jb20iLCJhdWQiOiJodHRwczovL2FwaS5teWFwcC5jb20iLCJpYXQiOjE2Nzg4ODY0MDAsImV4cCI6MTY3ODg5MDAwMCwianRpIjoiZjhhNGIxZTItYzNkNC01ZTZmLTdhOGItOWMwZDFlMmYzYTRiIn0`
*   🚨 **CẢNH BÁO BẢO MẬT TỐI QUAN TRỌNG:** Payload chỉ được mã hóa Base64Url, nghĩa là **bất kỳ ai có được token đều có thể giải mã và đọc nội dung của nó một cách dễ dàng**. Giống như thông tin ghi trên bưu thiếp vậy. Do đó:
    *   **TUYỆT ĐỐI KHÔNG BAO GIỜ** lưu trữ thông tin cực kỳ nhạy cảm như: mật khẩu (kể cả hash), khóa API bí mật, số thẻ tín dụng, mã PIN, thông tin định danh cá nhân (PII) chi tiết và nhạy cảm (số an sinh xã hội, chi tiết tài khoản ngân hàng...).
    *   Chỉ nên chứa thông tin định danh (`sub`), thông tin phân quyền cơ bản (roles - nếu không quá nhiều), thông tin quản lý phiên (`exp`, `iat`, `jti`), và các dữ liệu định danh ngữ cảnh không quá nhạy cảm khác.

---

## Phần 2: Chữ ký số, Bảo mật và So sánh với Session

### 2.3. Signature (Chữ ký số) - Con dấu Bảo đảm

Đây là phần thứ ba và cũng là phần quan trọng nhất đảm bảo tính bảo mật cốt lõi của JWT. Nếu Header và Payload giống như nội dung của tấm hộ chiếu, thì Signature chính là con dấu và các yếu tố bảo an chống làm giả.

*   **Mục đích:** Chữ ký đóng hai vai trò sống còn:
    1.  **Đảm bảo Tính toàn vẹn (Integrity):** Nó xác nhận rằng dữ liệu trong Header và Payload **không hề bị thay đổi** kể từ khi token được cấp phát. Bất kỳ sự sửa đổi nào, dù nhỏ nhất, cũng sẽ làm cho chữ ký không còn khớp khi được xác minh.
    2.  **Xác thực Nguồn gốc / Người gửi (Authenticity):** Vì việc tạo ra chữ ký yêu cầu một khóa bí mật (Secret Key) hoặc khóa riêng tư (Private Key) mà chỉ người/hệ thống cấp phát token hợp lệ mới sở hữu, nên một chữ ký hợp lệ chứng minh rằng token đó **thực sự được tạo ra bởi nguồn đáng tin cậy**, không phải giả mạo.

*   **Cách tạo chữ ký:** Quá trình này sử dụng ba thành phần đầu vào:
    1.  **Encoded Header:** Phần Header đã mã hóa Base64Url.
    2.  **Encoded Payload:** Phần Payload đã mã hóa Base64Url.
    3.  **Khóa bí mật (Secret) hoặc Khóa riêng tư (Private Key):**
        *   Với thuật toán đối xứng (HMAC): Một chuỗi bí mật (Secret Key) duy nhất.
        *   Với thuật toán bất đối xứng (RSA, ECDSA): Khóa riêng tư (Private Key) của người cấp phát.
    Quá trình diễn ra như sau:
    ```
    // 1. Ghép nối Header và Payload đã mã hóa
    dataToSign = base64UrlEncode(Header) + "." + base64UrlEncode(Payload)

    // 2. Áp dụng thuật toán ký đã chọn trong Header (`alg`) với Khóa tương ứng
    rawSignature = signFunction(dataToSign, Key)
    // Ví dụ: rawSignature = HMACSHA256(dataToSign, secretKey)
    // Hoặc: rawSignature = SignWithRSA_SHA256(dataToSign, privateKey)

    // 3. Mã hóa kết quả chữ ký thô bằng Base64Url
    signature = base64UrlEncode(rawSignature)
    ```
    Chuỗi `signature` này chính là phần thứ ba của JWT.

*   **Các Thuật toán Ký Phổ biến (`alg`):**
    *   **HMAC (vd: `HS256`, `HS384`, `HS512`)**
        *   **Loại:** Đối xứng (Symmetric).
        *   **Cơ chế:** Sử dụng **một khóa bí mật (Secret Key)** duy nhất cho cả việc *ký* và *xác minh*. Giống như một mật khẩu chung.
        *   **Ưu điểm:** Nhanh và đơn giản hơn về mặt tính toán.
        *   **Nhược điểm:** Cả bên ký và bên xác minh phải biết và bảo vệ cùng một khóa bí mật. Nếu khóa bị lộ ở bất kỳ đâu, kẻ tấn công có thể tạo token giả mạo. Việc chia sẻ khóa bí mật an toàn cho nhiều bên xác minh có thể phức tạp.
        *   **Khi nào dùng:** Phù hợp khi việc ký và xác minh diễn ra trong cùng một ứng dụng/hệ thống, hoặc giữa các microservice nội bộ rất tin cậy nhau và có kênh chia sẻ bí mật an toàn.
    *   **RSA / ECDSA (vd: `RS256`, `RS384`, `RS512`, `ES256`, `ES384`, `ES512`)**
        *   **Loại:** Bất đối xứng (Asymmetric).
        *   **Cơ chế:** Sử dụng một cặp khóa: **Khóa riêng tư (Private Key)** và **Khóa công khai (Public Key)**.
            *   Chỉ **Private Key** (phải được giữ tuyệt mật bởi người cấp phát) mới có thể *tạo* chữ ký.
            *   **Public Key** (có thể chia sẻ rộng rãi) được dùng để *xác minh* chữ ký.
        *   **Ưu điểm:** An toàn hơn cho hệ thống phân tán. Bên xác minh (ví dụ: các API servers) chỉ cần Public Key, không thể tạo token giả. Cho phép bên thứ ba dễ dàng xác minh token mà không cần biết bí mật của người cấp phát.
        *   **Nhược điểm:** Thường chậm hơn HMAC về mặt tính toán. Việc quản lý cặp khóa (tạo, lưu trữ private key an toàn, phân phối public key) phức tạp hơn.
        *   **Khi nào dùng:** Lý tưởng cho kiến trúc microservices nơi một Identity Provider (IdP) trung tâm cấp token và nhiều Resource Server khác nhau cần xác minh; Các kịch bản Single Sign-On (SSO); Khi cần cấp token cho các đối tác hoặc ứng dụng bên thứ ba.

*   **Tầm quan trọng của việc Bảo mật Khóa:** Đây là điểm yếu chí mạng nếu không được quản lý cẩn thận.
    *   Mất **Secret Key (HMAC)** hoặc **Private Key (RSA/ECDSA)** đồng nghĩa với việc **toàn bộ hệ thống xác thực JWT bị vô hiệu hóa**. Kẻ tấn công có thể tự do tạo ra bất kỳ token giả mạo nào với chữ ký hợp lệ, mạo danh người dùng và chiếm quyền truy cập.
    *   Việc bảo vệ các khóa này là ưu tiên hàng đầu (sẽ được đề cập kỹ hơn ở phần bảo mật).

### 3. JWT: Ký (Signing) ≠ Mã hóa (Encryption)

Một lần nữa, cần phải nhấn mạnh sự khác biệt cơ bản này:

*   **JWT (JWS - JSON Web Signature):** Mục đích chính là **xác thực** và **đảm bảo tính toàn vẹn** thông qua **Chữ ký số (Signing)**. Nó **KHÔNG** che giấu nội dung Payload theo mặc định.
*   **Base64Url Encoding:** Chỉ là phương thức mã hóa dữ liệu để truyền tải an toàn qua các kênh text-based (như HTTP header, URL), **hoàn toàn có thể giải mã (decode)** dễ dàng.
*   **Encryption (Mã hóa bảo mật):** Là quá trình làm cho dữ liệu không thể đọc được nếu không có khóa giải mã (decryption key). Nếu bạn thực sự cần bảo vệ *nội dung* của JWT khỏi bị đọc trộm, bạn phải sử dụng một chuẩn khác là **JWE (JSON Web Encryption)**, phức tạp hơn đáng kể.

**=> Hãy luôn coi Payload của JWS là thông tin có thể đọc được.**

### 4. So sánh JWT với Xác thực dựa trên Session

Để thấy rõ ưu điểm của JWT, hãy so sánh nó với phương pháp xác thực truyền thống dựa trên Session:

| Tính năng              | Session-Based Authentication                                     | JWT-Based Authentication                                          |
| :--------------------- | :--------------------------------------------------------------- | :---------------------------------------------------------------- |
| **Lưu trữ trạng thái** | **Stateful:** Server phải lưu trữ dữ liệu session (trong bộ nhớ, DB, cache). | **Stateless (chủ yếu):** Server không cần lưu trạng thái phiên. Thông tin nằm trong token. |
| **Dữ liệu phiên**     | ID phiên (thường trong cookie) được gửi từ client. Server tra cứu dữ liệu session tương ứng. | Toàn bộ thông tin cần thiết (claims) nằm trong Payload của token. |
| **Scalability**        | Khó scale ngang hơn. Cần cơ chế đồng bộ session hoặc sticky session (binding client tới 1 server). | Dễ dàng scale ngang. Bất kỳ server nào có khóa đều xác thực được token. |
| **Hiệu suất Server**  | Cần I/O (đọc/ghi) để truy cập dữ liệu session trên server.       | Chỉ cần tính toán CPU để xác thực chữ ký (thường nhanh hơn I/O). |
| **Microservices/APIs** | Khó khăn hơn trong việc chia sẻ session giữa các dịch vụ khác nhau. | **Rất phù hợp.** Token dễ dàng truyền giữa các dịch vụ.             |
| **Mobile Apps**        | Quản lý cookie có thể phức tạp hơn.                                | Dễ dàng lưu và gửi token trong header `Authorization`.             |
| **Cross-Domain (CORS)**| Cookie có thể gặp vấn đề với chính sách same-origin.            | Dễ dàng hơn khi gửi token trong header `Authorization`.            |
| **CSRF**               | Dễ bị tấn công CSRF nếu chỉ dùng session cookie. Cần thêm token chống CSRF. | Nếu lưu token trong `localStorage`, ít bị CSRF hơn cookie. Nếu dùng cookie, vẫn cần cờ `SameSite`. |
| **XSS**                | Rủi ro nếu session ID bị lộ qua XSS (nhưng khó hơn).              | **Rủi ro cao** nếu token lưu trong `localStorage`/`sessionStorage` bị lộ qua XSS. `HttpOnly` cookie an toàn hơn. |
| **Kích thước Token**    | Session ID thường nhỏ.                                           | JWT có thể lớn hơn nếu chứa nhiều claims. Header HTTP có giới hạn kích thước. |
| **Thu hồi (Revocation)**| **Dễ dàng:** Chỉ cần xóa session trên server.                      | **Khó khăn hơn:** Token hợp lệ cho đến khi hết hạn. Cần các chiến lược phức tạp (blacklist, refresh token...). |
| **Chuẩn hóa**          | Ít chuẩn hóa hơn về cách lưu trữ/quản lý session.               | **Chuẩn hóa (RFC 7519)**, nhiều thư viện hỗ trợ.                   |

**Tóm lại:** JWT tỏa sáng nhờ tính **stateless**, **scalability**, và sự phù hợp với kiến trúc **microservices/APIs**, nhưng đi kèm với những thách thức về **bảo mật lưu trữ token phía client (XSS)**, **kích thước token**, và **độ phức tạp của việc thu hồi token**.

---

## Phần 3: Luồng Hoạt động Xác thực JWT trong Thực tế

Bây giờ chúng ta đã hiểu cấu trúc của JWT, hãy xem cách nó được sử dụng trong một quy trình xác thực đăng nhập và truy cập tài nguyên được bảo vệ.

**Bối cảnh:** Người dùng (Alice) muốn đăng nhập vào ứng dụng web `myapp.com` và truy cập vào trang hồ sơ cá nhân (`/api/profile`), một tài nguyên yêu cầu xác thực.

**Các bước diễn ra như sau:**

**Bước 1: Yêu cầu Đăng nhập (Client → Server)**

1.  **Alice nhập thông tin:** Alice mở trang đăng nhập của `myapp.com` và nhập username (`alice@example.com`) cùng mật khẩu (`herSecretPassword`).
2.  **Client gửi yêu cầu:** Khi Alice nhấn nút "Đăng nhập", JavaScript phía client (hoặc ứng dụng di động) tạo một yêu cầu `POST` gửi đến endpoint xác thực của server (ví dụ: `/api/auth/login`).
    *   **Quan trọng:** Yêu cầu này **phải** được gửi qua **HTTPS** để mã hóa username và mật khẩu trên đường truyền.
    *   Credentials thường được gửi trong phần body của request dưới dạng JSON.

    ```http
    POST /api/auth/login HTTP/1.1
    Host: myapp.com
    Content-Type: application/json
    Connection: keep-alive
    Accept: application/json

    {
      "email": "alice@example.com",
      "password": "herSecretPassword"
    }
    ```

**Bước 2: Xác thực Thông tin & Tạo JWT (Server)**

1.  **Server nhận yêu cầu:** Server nhận yêu cầu `POST` tại `/api/auth/login`.
2.  **Xác thực Credentials:**
    *   Server lấy `email` và `password` từ body request.
    *   Truy vấn cơ sở dữ liệu để tìm người dùng có `email` tương ứng.
    *   Nếu tìm thấy người dùng: Server sử dụng một thuật toán hash mạnh (như bcrypt, Argon2) để so sánh `password` người dùng gửi lên với **hash của mật khẩu** được lưu trữ an toàn trong database. **Tuyệt đối không lưu mật khẩu dạng plain text!**
    *   Nếu không tìm thấy người dùng hoặc mật khẩu không khớp: Server trả về lỗi xác thực (ví dụ: HTTP status `401 Unauthorized`).
3.  **Tạo JWT (Nếu xác thực thành công):**
    *   Server quyết định thuật toán ký (ví dụ: `HS256`).
    *   **Tạo Header:** `{ "alg": "HS256", "typ": "JWT" }`.
    *   **Tạo Payload:** Bao gồm các claims cần thiết. Tối thiểu nên có `sub` (User ID của Alice lấy từ DB), `exp` (thời gian hết hạn ngắn, ví dụ 15 phút sau), `iat` (thời điểm hiện tại). Có thể thêm `role`, `name`...
        ```json
        {
          "sub": "user-alice-1a2b3c",
          "name": "Alice Example",
          "role": "member",
          "iss": "https://myapp.com", // Issuer là chính server này
          "aud": "https://myapp.com/api", // Audience là API của server
          "iat": 1711540000, // Thời điểm tạo
          "exp": 1711540900  // Hết hạn sau 15 phút (900 giây)
        }
        ```
    *   **Tạo Signature:** Server lấy Khóa bí mật (Secret Key) đã được cấu hình an toàn, áp dụng thuật toán `HS256` lên `base64UrlEncode(Header) + "." + base64UrlEncode(Payload)` để tạo chữ ký.
    *   **Ghép nối:** Server kết hợp 3 phần đã mã hóa Base64Url thành chuỗi JWT hoàn chỉnh.

**Bước 3: Gửi JWT về Client (Server → Client)**

1.  **Server phản hồi:** Sau khi tạo JWT thành công, server gửi lại một response `200 OK` cho client.
2.  **Gửi Token:** Chuỗi JWT (thường gọi là `accessToken`) được đặt trong body của response, thường là một đối tượng JSON.
    *   **Khuyến nghị:** Server cũng nên tạo và gửi kèm một `refreshToken` (sẽ nói rõ ở phần sau) có thời hạn dài hơn và được xử lý khác với `accessToken`.

    ```http
    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store
    Pragma: no-cache
    // Set-Cookie: refreshToken=xyzabc...; HttpOnly; Secure; SameSite=Strict; Path=/api/auth/refresh; Max-Age=... (Nếu dùng cookie cho refresh token)

    {
      "message": "Login successful",
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyLWFsaWNlLTFhMmIzYyIsIm5hbWUiOiJBbGljZSBFeGFtcGxlIiwicm9sZSI6Im1lbWJlciIsImlzcyI6Imh0dHBzOi8vbXlhcHAuY29tIiwiYXVkIjoiaHR0cHM6Ly9teWFwcC5jb20vYXBpIiwiaWF0IjoxNzExNTQwMDAwLCJleHAiOjE3MTE1NDA5MDB9.SOME_VALID_HS256_SIGNATURE"
      // , "refreshToken": "def456..." // Nếu không dùng cookie
    }
    ```

**Bước 4: Lưu trữ JWT (Client)**

1.  **Client nhận Token:** Client (JavaScript trong trình duyệt của Alice) nhận được response chứa `accessToken` (và có thể cả `refreshToken`).
2.  **Lưu trữ an toàn:** Client cần lưu trữ (các) token này để sử dụng cho các yêu cầu sau. Đây là một điểm **rất quan trọng về bảo mật**. Các lựa chọn phổ biến và đánh đổi:
    *   **`localStorage` / `sessionStorage` (Web Storage):**
        *   *Ưu:* API đơn giản (`localStorage.setItem('accessToken', token)`), `localStorage` giữ lại token ngay cả khi đóng trình duyệt.
        *   *Nhược:* **Cực kỳ dễ bị tấn công XSS (Cross-Site Scripting).** Nếu trang web có lỗ hổng XSS, mã độc có thể chạy và đọc *toàn bộ* nội dung trong `localStorage/sessionStorage`, bao gồm cả token, dẫn đến việc bị đánh cắp phiên đăng nhập. **Không khuyến nghị cho việc lưu trữ token nhạy cảm như Refresh Token.**
    *   **`HttpOnly` Cookie:**
        *   *Ưu:* **An toàn hơn nhiều trước XSS** vì JavaScript phía client không thể truy cập đọc/ghi cookie được đánh dấu `HttpOnly`. Trình duyệt tự động đính kèm cookie vào các yêu cầu gửi đến cùng domain (và path) đã được cấu hình.
        *   *Nhược:* Có thể bị **tấn công CSRF (Cross-Site Request Forgery)** nếu không cấu hình cờ `SameSite` (`Strict` hoặc `Lax`) một cách chính xác. Có giới hạn kích thước (khoảng 4KB). Việc cấu hình `Set-Cookie` header từ server phức tạp hơn một chút.
    *   **In-Memory (Biến JavaScript):**
        *   *Ưu:* Tương đối an toàn trước XSS sau khi token đã được load vào biến.
        *   *Nhược:* Token sẽ bị mất hoàn toàn khi người dùng refresh trang (F5) hoặc đóng tab/trình duyệt. Thường chỉ phù hợp cho `accessToken` trong các SPA phức tạp, và luôn cần đi kèm cơ chế dùng `refreshToken` để lấy lại `accessToken` mới khi cần.
    *   **Khuyến nghị phổ biến:**
        *   Lưu **`refreshToken`** trong **`HttpOnly`, `Secure`, `SameSite=Strict` Cookie**.
        *   Lưu **`accessToken`**: Có thể lưu trong **biến JavaScript (in-memory)** (yêu cầu cơ chế refresh khi cần) hoặc cũng có thể lưu trong **`HttpOnly` Cookie** (đơn giản hơn cho việc gửi tự động, nhưng cần cẩn thận với CSRF và giới hạn kích thước). Lưu `accessToken` trong `localStorage` nên được cân nhắc kỹ lưỡng về rủi ro XSS.

**Bước 5: Gửi JWT trong các Yêu cầu Tiếp theo (Client → Server)**

1.  **Alice truy cập trang Profile:** Alice nhấn vào link để xem trang hồ sơ cá nhân (`/profile`) trên `myapp.com`. Trình duyệt cần gọi API `/api/profile` để lấy dữ liệu.
2.  **Client đính kèm Token:** Trước khi gửi yêu cầu đến `/api/profile`, client cần lấy `accessToken` đã lưu.
    *   **Cách phổ biến nhất:** Đặt token vào **HTTP Header `Authorization`** sử dụng schema (kiểu) `Bearer`.
    ```http
    GET /api/profile HTTP/1.1
    Host: myapp.com
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyLWFsaWNlLTFhMmIzYyIsIm5hbWUiOiJBbGljZSBFeGFtcGxlIiwicm9sZSI6Im1lbWJlciIsImlzcyI6Imh0dHBzOi8vbXlhcHAuY29tIiwiYXVkIjoiaHR0cHM6Ly9teWFwcC5jb20vYXBpIiwiaWF0IjoxNzExNTQwMDAwLCJleHAiOjE3MTE1NDA5MDB9.SOME_VALID_HS256_SIGNATURE
    Connection: keep-alive
    Accept: application/json
    ```
    *   **Nếu dùng `HttpOnly` Cookie:** Trình duyệt sẽ tự động đính kèm cookie chứa token vào yêu cầu nếu các thuộc tính domain, path, secure khớp. Client-side code không cần làm gì thêm ở bước này. Server sẽ cần đọc token từ cookie thay vì header `Authorization`.

**Bước 6: Xác minh JWT (Server)**

1.  **Middleware/Filter chặn yêu cầu:** Yêu cầu `GET /api/profile` đến server. Một lớp trung gian (middleware trong Node.js/Express, filter trong Java/Spring, ...) được cấu hình để bảo vệ endpoint này sẽ chặn yêu cầu lại trước tiên.
2.  **Trích xuất Token:** Middleware tìm và trích xuất chuỗi JWT từ header `Authorization` (bỏ đi phần `Bearer `) hoặc từ cookie (nếu dùng cách đó). Nếu không tìm thấy token -> Trả lỗi `401 Unauthorized`.
3.  **Thực hiện Xác minh (Verification):** Đây là bước quan trọng nhất phía server:
    *   **Kiểm tra định dạng:** Token có đúng cấu trúc 3 phần ngăn cách bởi dấu chấm không?
    *   **Giải mã Header:** Decode Base64Url phần header để đọc `alg`.
    *   **Kiểm tra Thuật toán (`alg`):** Thuật toán trong header (`HS256`) có nằm trong danh sách các thuật toán được server cho phép không? **Tuyệt đối không chấp nhận `alg: "none"`!**
    *   **Giải mã Payload:** Decode Base64Url phần payload.
    *   **Kiểm tra Thời gian hết hạn (`exp`):** Lấy giá trị `exp` từ payload. So sánh với thời gian hiện tại của server. Nếu `exp` <= `thời_gian_hiện_tại` -> Token đã hết hạn, trả lỗi `401 Unauthorized`.
    *   *(Tùy chọn)* **Kiểm tra các claims khác:** `iss`, `aud` có khớp với cấu hình của server không? `nbf` (nếu có) đã qua chưa?
    *   *(Quan trọng nhất)* **Xác thực Chữ ký (Verify Signature):**
        *   Lấy `encodedHeader` và `encodedPayload` từ token nhận được.
        *   Lấy **đúng Khóa bí mật (Secret Key)** (cho HS256) hoặc **Khóa công khai (Public Key)** (cho RS256/ES256) mà server đang giữ và tương ứng với `alg` đã xác định.
        *   Tính toán lại chữ ký dự kiến: `expectedSignature = HMACSHA256(encodedHeader + "." + encodedPayload, secretKey)` (ví dụ với HS256).
        *   So sánh `expectedSignature` (sau khi mã hóa Base64Url) với phần `signature` nhận được từ token.
4.  **Kết quả Xác minh:**
    *   **Thành công:** Chữ ký khớp và mọi kiểm tra khác đều qua! Token hợp lệ. Middleware cho phép yêu cầu đi tiếp vào bộ xử lý logic chính của endpoint `/api/profile`. Thông tin người dùng (ví dụ `sub` từ payload) thường được middleware gắn vào đối tượng request (vd: `req.user = { id: 'user-alice-1a2b3c', role: 'member' }`) để logic phía sau sử dụng.
    *   **Thất bại:** Chữ ký không khớp hoặc một kiểm tra nào đó thất bại (hết hạn, sai `alg`...). Middleware ngay lập tức trả về lỗi cho client, thường là `401 Unauthorized` (token không hợp lệ/hết hạn) hoặc `403 Forbidden` (token hợp lệ nhưng không đủ quyền - kiểm tra này thường diễn ra sau khi xác thực thành công).

Luồng hoạt động này, khi được triển khai đúng cách, đảm bảo rằng chỉ những yêu cầu từ người dùng đã được xác thực và có token hợp lệ mới có thể truy cập vào các tài nguyên được bảo vệ.

---

## Phần 4: Bảo mật JWT - Các Vấn đề Then chốt và Thực tiễn Tốt nhất

JWT mang lại nhiều lợi ích, nhưng nếu không được triển khai cẩn thận, nó có thể mở ra những lỗ hổng bảo mật nghiêm trọng. Dưới đây là các vấn đề cần lưu ý và cách giảm thiểu rủi ro:

### 1. Vấn đề: Đánh cắp Token qua Kênh truyền không Mã hóa (HTTP)

*   **Mô tả:** Nếu JWT được gửi qua kết nối HTTP thông thường (không phải HTTPS), kẻ tấn công ở giữa (Man-in-the-Middle - MitM) có thể dễ dàng nghe lén và chặn bắt toàn bộ gói tin, bao gồm cả header `Authorization` chứa token.
*   **Hậu quả:** Kẻ tấn công có được token và có thể sử dụng nó để mạo danh người dùng hợp lệ cho đến khi token hết hạn.
*   **✅ Thực tiễn tốt nhất:** **LUÔN LUÔN và CHỈ SỬ DỤNG HTTPS** cho *tất cả* các giao tiếp liên quan đến JWT:
    *   Endpoint đăng nhập (gửi credentials).
    *   Endpoint trả về token.
    *   Tất cả các endpoint API yêu cầu token.
    *   Endpoint làm mới token (refresh token).
    *   Cấu hình HSTS (HTTP Strict Transport Security) để buộc trình duyệt luôn sử dụng HTTPS.

### 2. Vấn đề: Lộ Khóa Bí mật (Secret Key / Private Key)

*   **Mô tả:** Đây là "gót chân Achilles" của JWT. Nếu Secret Key (dùng cho HMAC) hoặc Private Key (dùng cho RSA/ECDSA) bị lộ, kẻ tấn công có thể:
    *   Tự tạo ra bất kỳ JWT giả mạo nào với chữ ký hợp lệ.
    *   Mạo danh bất kỳ người dùng nào.
    *   Gán cho mình bất kỳ quyền hạn nào được định nghĩa trong claims.
*   **Hậu quả:** Toàn bộ hệ thống xác thực dựa trên JWT bị phá vỡ hoàn toàn.
*   **✅ Thực tiễn tốt nhất:**
    *   **Sử dụng khóa mạnh:** Dài, phức tạp, ngẫu nhiên.
    *   **Không bao giờ hardcode khóa:** Tránh nhúng khóa trực tiếp vào mã nguồn hoặc commit vào Git.
    *   **Lưu trữ an toàn:**
        *   Sử dụng biến môi trường (environment variables).
        *   Sử dụng file cấu hình được bảo vệ quyền truy cập nghiêm ngặt.
        *   **Tốt nhất:** Sử dụng các hệ thống quản lý bí mật chuyên dụng (Secret Management Systems) như HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, Google Secret Manager.
    *   **Giới hạn quyền truy cập:** Chỉ cấp quyền truy cập khóa cho các tiến trình/dịch vụ thực sự cần thiết.
    *   **Xoay vòng khóa (Key Rotation):** Định kỳ thay đổi khóa (ví dụ: vài tháng một lần) để giảm thiểu tác động nếu khóa cũ bị lộ.

### 3. Vấn đề: Lỗ hổng Thuật toán (`alg`)

*   **Mô tả:** Kẻ tấn công có thể cố gắng thao túng trường `alg` trong Header để lừa server bỏ qua hoặc xác minh chữ ký sai cách.
    *   **`alg: "none"`:** Kẻ tấn công sửa `alg` thành `"none"` và xóa bỏ phần Signature. Nếu server không kiểm tra `alg` và chấp nhận `"none"`, nó sẽ coi token giả mạo là hợp lệ mà không cần xác minh chữ ký.
    *   **Nhầm lẫn Thuật toán (Algorithm Confusion):** Tấn công tinh vi hơn, ví dụ lừa server dùng Public Key (của RS256) như là Secret Key (cho HS256). Nếu kẻ tấn công biết Public Key (thường là công khai), họ có thể tạo token giả mạo với chữ ký HS256 hợp lệ.
*   **✅ Thực tiễn tốt nhất:**
    *   **Server PHẢI xác minh `alg`:** Luôn kiểm tra giá trị `alg` trong Header nhận được.
    *   **Sử dụng Whitelist:** Chỉ chấp nhận một danh sách các thuật toán cụ thể mà server hỗ trợ và mong đợi (ví dụ: chỉ `["HS256"]` hoặc `["RS256", "ES256"]`).
    *   **Tuyệt đối TỪ CHỐI `alg: "none"`**.
    *   **Sử dụng thư viện JWT uy tín:** Chọn các thư viện được cập nhật thường xuyên và đã được kiểm tra về các lỗ hổng phổ biến. Cấu hình thư viện đúng cách để chỉ định rõ thuật toán và khóa tương ứng khi xác minh.

### 4. Vấn đề: Token Bị Lộ và Thời Gian Hết Hạn (`exp`)

*   **Mô tả:** Nếu một `accessToken` bị đánh cắp (ví dụ qua XSS, malware, hoặc người dùng vô tình làm lộ), kẻ tấn công có thể sử dụng nó cho đến khi token tự hết hạn. Nếu thời gian hết hạn (`exp`) quá dài (ngày, tuần, tháng), rủi ro sẽ kéo dài tương ứng.
*   **✅ Thực tiễn tốt nhất:**
    *   **Luôn đặt `exp` cho `accessToken`:** Không bao giờ cấp token không có thời hạn.
    *   **Thời hạn `accessToken` NGẮN:** Đặt `exp` ngắn một cách hợp lý (5 phút, 15 phút, tối đa 1 giờ). Điều này giới hạn "cửa sổ cơ hội" cho kẻ tấn công nếu token bị lộ.
    *   **Kết hợp với Refresh Token:** Để tránh bắt người dùng đăng nhập lại liên tục do `accessToken` hết hạn ngắn, hãy sử dụng cơ chế Refresh Token (xem mục 7).
    *   **Server PHẢI kiểm tra `exp`:** Luôn xác minh token chưa hết hạn trước khi xử lý yêu cầu.

### 5. Vấn đề: Đánh cắp Token phía Client (XSS)

*   **Mô tả:** Nếu ứng dụng web có lỗ hổng Cross-Site Scripting (XSS), kẻ tấn công có thể tiêm mã JavaScript độc hại vào trang web. Mã này có thể chạy trong trình duyệt của người dùng và:
    *   Đọc trộm token nếu nó được lưu trong `localStorage` hoặc `sessionStorage`.
    *   Gửi token bị đánh cắp về máy chủ của kẻ tấn công.
*   **Hậu quả:** Mất phiên đăng nhập của người dùng.
*   **✅ Thực tiễn tốt nhất:**
    *   **Phòng chống XSS triệt để:** Đây là biện pháp quan trọng nhất. Sử dụng các framework hiện đại có cơ chế chống XSS sẵn có, sanitize (làm sạch) mọi dữ liệu đầu vào từ người dùng trước khi hiển thị, sử dụng Content Security Policy (CSP).
    *   **Lưu trữ Token An toàn:**
        *   **Ưu tiên `HttpOnly` Cookie:** Đặc biệt đối với `refreshToken` và có thể cả `accessToken`. Cookie `HttpOnly` không thể bị truy cập bởi JavaScript, giảm thiểu đáng kể nguy cơ bị đánh cắp qua XSS.
        *   **Cấu hình Cookie chặt chẽ:** Luôn dùng cờ `Secure` (chỉ gửi qua HTTPS) và `SameSite=Strict` hoặc `SameSite=Lax` (để chống CSRF).
        *   Nếu bắt buộc dùng `localStorage` cho `accessToken`: Phải cực kỳ chú trọng vào việc vá mọi lỗ hổng XSS.

### 6. Vấn đề: Khó Thu hồi Token (Revocation)

*   **Mô tả:** Do tính chất stateless, một JWT khi đã được cấp phát sẽ được coi là hợp lệ cho đến khi hết hạn (`exp`). Không có cơ chế tích hợp sẵn để "vô hiệu hóa" một token cụ thể ngay lập tức nếu nó bị lộ hoặc người dùng muốn logout/đổi mật khẩu.
*   **✅ Thực tiễn tốt nhất (Thường phải đánh đổi một phần tính stateless):**
    *   **Chiến lược phổ biến nhất: `accessToken` ngắn hạn + `refreshToken` dài hạn:**
        *   Khi cần thu hồi (logout, đổi pass, khóa tài khoản): Chỉ cần **vô hiệu hóa `refreshToken`** phía server (ví dụ: xóa khỏi DB lưu trữ refresh token, hoặc thêm vào blacklist).
        *   Khi `accessToken` hiện tại hết hạn, client sẽ cố gắng dùng `refreshToken` đã bị vô hiệu hóa để lấy `accessToken` mới -> Server từ chối -> Người dùng bị buộc logout hoặc phải xác thực lại.
    *   **Blacklisting (Danh sách đen):**
        *   Server duy trì một danh sách (thường trong cache nhanh như Redis) chứa ID (`jti`) của các token đã bị thu hồi.
        *   Mỗi lần xác thực token, server phải kiểm tra xem `jti` của token đó có nằm trong blacklist không.
        *   *Nhược điểm:* Thêm state vào server, tăng độ trễ do phải kiểm tra cache/DB. Cần cơ chế dọn dẹp blacklist.
    *   **Kiểm tra Phiên bản/Timestamp:**
        *   Lưu một giá trị (ví dụ: `password_version`, `last_logout_timestamp`) cho mỗi user trong DB.
        *   Thêm giá trị này vào Payload của JWT khi cấp phát.
        *   Khi xác thực, server lấy giá trị mới nhất từ DB và so sánh với giá trị trong token. Nếu không khớp -> Từ chối token.
        *   *Nhược điểm:* Yêu cầu truy vấn DB mỗi lần xác thực token, gần giống cơ chế session.

### 7. Vai trò Quan trọng của Refresh Token

Refresh Token là một "người bạn đồng hành" không thể thiếu của Access Token trong các hệ thống JWT hiện đại, giúp cân bằng giữa bảo mật và trải nghiệm người dùng.

*   **Khái niệm:** Là một credential đặc biệt, được cấp cùng lúc với `accessToken` nhưng có **thời gian sống dài hơn nhiều** (ví dụ: vài ngày, vài tuần, hoặc đánh dấu là "cho đến khi bị thu hồi"). Nó **chỉ dùng cho một mục đích duy nhất**: lấy `accessToken` mới.
*   **Luồng hoạt động với Refresh Token:**
    1.  Người dùng đăng nhập -> Nhận cả `accessToken` (ngắn hạn) và `refreshToken` (dài hạn).
    2.  Client dùng `accessToken` để gọi các API.
    3.  Khi `accessToken` hết hạn (server trả lỗi 401), client **không** bắt người dùng đăng nhập lại ngay.
    4.  Thay vào đó, client **âm thầm** gửi `refreshToken` (được lưu trữ an toàn, ví dụ trong HttpOnly cookie) đến một **endpoint đặc biệt** trên server (ví dụ: `/api/auth/refresh`).
    5.  Server xác thực `refreshToken` (kiểm tra xem nó có tồn tại, còn hạn, và chưa bị thu hồi trong DB/blacklist không).
    6.  Nếu `refreshToken` hợp lệ: Server cấp một **`accessToken` mới** (và tùy chọn, một **`refreshToken` mới** - gọi là **rotation**) và gửi về cho client.
    7.  Client nhận `accessToken` mới và tự động thử lại yêu cầu API ban đầu đã thất bại. Người dùng không hề hay biết quá trình này.
    8.  Nếu `refreshToken` không hợp lệ (đã hết hạn, bị thu hồi): Server từ chối cấp token mới, client biết rằng phiên đăng nhập đã thực sự kết thúc và chuyển hướng người dùng về trang đăng nhập.
*   **Lợi ích:**
    *   **Bảo mật Tốt hơn:** Giảm thiểu rủi ro từ `accessToken` bị lộ (do thời hạn ngắn). `refreshToken` ít được truyền đi hơn và có thể được quản lý/thu hồi phía server.
    *   **Trải nghiệm Người dùng (UX) Tốt hơn:** Người dùng không bị bắt đăng nhập lại thường xuyên.
    *   **Cung cấp Cơ chế Thu hồi Hiệu quả:** Vô hiệu hóa `refreshToken` phía server là cách đáng tin cậy để chấm dứt phiên đăng nhập.
*   **Lưu trữ Refresh Token:** **Cực kỳ quan trọng!** Vì `refreshToken` rất mạnh, phải lưu trữ nó cực kỳ an toàn. **Khuyến nghị mạnh mẽ:** Lưu trong **`HttpOnly`, `Secure`, `SameSite` Cookie**.
*   **Refresh Token Rotation (Nên dùng):** Mỗi khi `refreshToken` được sử dụng thành công, server nên cấp một `refreshToken` *mới* và vô hiệu hóa `refreshToken` cũ. Điều này giúp phát hiện nếu `refreshToken` bị đánh cắp và sử dụng lại (vì kẻ tấn công và người dùng thật sẽ có `refreshToken` khác nhau, và chỉ một cái được chấp nhận ở lần dùng tiếp theo).

### 8. Tổng kết các Thực tiễn Tốt nhất

1.  **HTTPS Everywhere.**
2.  **Bảo vệ Secret/Private Key như tài sản quý giá nhất.**
3.  **Xác minh `alg` nghiêm ngặt, từ chối `none`.**
4.  **Đặt `exp` ngắn cho `accessToken`.**
5.  **Không bao giờ đặt dữ liệu nhạy cảm trong Payload.**
6.  **Phòng chống XSS triệt để.**
7.  **Ưu tiên lưu token (đặc biệt là Refresh Token) trong `HttpOnly`, `Secure`, `SameSite` Cookie.**
8.  **Sử dụng Refresh Token để cân bằng bảo mật và UX, đồng thời có cơ chế thu hồi.**
9.  **Xem xét Refresh Token Rotation.**
10. **Sử dụng thư viện JWT uy tín và cập nhật.**
11. **Thêm các claim `iss`, `aud`, `jti` để tăng cường bảo mật và quản lý.**

---

**Hoàn thành.**

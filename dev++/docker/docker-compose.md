# Làm Chủ Docker Compose: Từ Cơ Bản Đến Quản Lý Ứng Dụng Hiệu Quả

Docker đã cách mạng hóa cách chúng ta đóng gói và triển khai ứng dụng. Tuy nhiên, khi ứng dụng trở nên phức tạp, bao gồm nhiều thành phần (như web server, database, cache, worker...), việc quản lý từng container riêng lẻ bằng lệnh `docker run` trở nên cồng kềnh và dễ lỗi. Đây là lúc Docker Compose tỏa sáng.

Docker Compose là một công cụ mạnh mẽ giúp đơn giản hóa việc định nghĩa, chạy và quản lý các ứng dụng Docker đa container. Bài viết này sẽ hướng dẫn bạn từ những khái niệm cơ bản nhất đến các kỹ thuật nâng cao và thực tiễn tốt nhất khi làm việc với Docker Compose.

## Phần 1: Docker Compose là gì và Tại sao bạn cần nó?

### 1.1. Định nghĩa và Mục đích

*   **Docker Compose là gì?** Là một công cụ dòng lệnh (CLI tool) của Docker, cho phép bạn sử dụng một tệp cấu hình YAML (thường là `docker-compose.yml`) để định nghĩa toàn bộ *stack* ứng dụng đa container của mình.
*   **Mục đích chính:** Thay vì phải gõ nhiều lệnh `docker run` phức tạp với vô số tùy chọn để khởi chạy từng container và kết nối chúng lại với nhau, bạn chỉ cần mô tả trạng thái mong muốn của ứng dụng trong tệp YAML. Sau đó, với một lệnh duy nhất (`docker compose up`), Compose sẽ lo phần còn lại: xây dựng images (nếu cần), tạo và khởi chạy containers, thiết lập networks, quản lý volumes.

### 1.2. Lợi ích chính so với `docker run`

Việc sử dụng Docker Compose mang lại nhiều lợi ích đáng kể so với việc quản lý thủ công từng container:

*   **Đơn giản hóa quản lý:** Một tệp YAML duy nhất định nghĩa toàn bộ ứng dụng phức tạp. Các lệnh `up`, `down`, `ps`, `logs` áp dụng cho toàn bộ stack, giúp dễ dàng khởi chạy, dừng, kiểm tra trạng thái và gỡ lỗi.
*   **Nhất quán môi trường:** Đảm bảo ứng dụng chạy theo cùng một cách trên các môi trường khác nhau (development, staging, production) chỉ bằng cách sử dụng cùng một tệp cấu hình.
*   **Kết nối dễ dàng:** Compose tự động tạo một mạng riêng cho ứng dụng. Các container (services) trong cùng tệp Compose có thể dễ dàng giao tiếp với nhau bằng cách sử dụng tên service làm hostname, loại bỏ nhu cầu quản lý network thủ công phức tạp.
*   **Quản lý Volume tiện lợi:** Việc định nghĩa và sử dụng volumes (đặc biệt là named volumes) để lưu trữ dữ liệu persistent trở nên đơn giản và rõ ràng.
*   **Tái sử dụng và Chia sẻ:** Tệp `docker-compose.yml` dễ đọc, dễ hiểu, dễ dàng đưa vào hệ thống quản lý phiên bản (như Git) và chia sẻ với đồng nghiệp, giúp cộng tác hiệu quả hơn.

## Phần 2: Bắt đầu với `docker-compose.yml` và Các Lệnh Cơ Bản

Tệp `docker-compose.yml` là trung tâm của Docker Compose, nơi bạn định nghĩa mọi thứ về ứng dụng của mình.

### 2.1. Vai trò và Cấu trúc cơ bản

*   **Vai trò:** Là bản thiết kế chi tiết cho ứng dụng đa container của bạn, mô tả các thành phần (services), cách chúng kết nối (networks), và cách chúng lưu trữ dữ liệu (volumes).
*   **Cấu trúc:** Sử dụng định dạng YAML, dễ đọc và viết. Cấu trúc cơ bản thường bao gồm:
    *   `version`: (Tùy chọn nhưng khuyến nghị) Chỉ định phiên bản schema của tệp Compose.
    *   `services`: Nơi định nghĩa các container riêng lẻ của ứng dụng. Mỗi mục con là một service.
    *   `volumes`: (Tùy chọn) Nơi định nghĩa các named volumes để lưu trữ dữ liệu bền vững.
    *   `networks`: (Tùy chọn) Nơi định nghĩa các mạng tùy chỉnh (custom networks).

### 2.2. Ví dụ: Tạo file `docker-compose.yml` cho Web (Nginx) + Database (Postgres)

Hãy tạo một ứng dụng đơn giản gồm Nginx làm web server và PostgreSQL làm database.

```yaml
# docker-compose.yml

version: '3.8' # Sử dụng phiên bản schema 3.8

services:
  # Định nghĩa service database
  db:
    image: postgres:14-alpine # Sử dụng image Postgres chính thức, phiên bản 14-alpine
    container_name: my_postgres_db # (Tùy chọn) Đặt tên cụ thể cho container
    restart: unless-stopped    # Tự khởi động lại nếu container bị dừng không mong muốn
    volumes:
      - db_data:/var/lib/postgresql/data # Sử dụng named volume 'db_data' để lưu trữ dữ liệu Postgres
    environment:               # Các biến môi trường để cấu hình Postgres
      POSTGRES_DB: myappdb
      POSTGRES_USER: myappuser
      POSTGRES_PASSWORD: mysecretpassword # !!! Thay mật khẩu mạnh trong thực tế

  # Định nghĩa service web server
  web:
    image: nginx:alpine        # Sử dụng image Nginx chính thức, phiên bản alpine
    container_name: my_nginx_web # (Tùy chọn) Đặt tên cụ thể cho container
    restart: unless-stopped
    ports:
      - "80:80"                # Ánh xạ cổng 80 trên host vào cổng 80 của container Nginx
    # depends_on:              # (Tùy chọn) Nếu Nginx cần DB chạy trước (ví dụ: reverse proxy cho app cần DB)
    #   - db                   # Bỏ comment dòng này và dòng dưới

# Định nghĩa named volume ở cấp cao nhất
volumes:
  db_data: {}                  # Khai báo volume 'db_data', Docker sẽ quản lý nó
```

**Giải thích:**

*   Chúng ta định nghĩa hai `services`: `db` và `web`.
*   Service `db` sử dụng image `postgres:14-alpine`, lưu dữ liệu vào named volume `db_data` và được cấu hình qua các biến môi trường.
*   Service `web` sử dụng image `nginx:alpine` và ánh xạ cổng 80 của host tới cổng 80 của container, cho phép truy cập Nginx từ trình duyệt qua `http://localhost`.
*   Named volume `db_data` được khai báo ở cuối file.

### 2.3. Các Lệnh Docker Compose Cơ Bản

Sau khi có tệp `docker-compose.yml`, bạn sử dụng các lệnh sau trong terminal (tại thư mục chứa tệp):

*   **Khởi chạy ứng dụng (ở chế độ nền):**
    ```bash
    docker compose up -d
    ```
    *   `up`: Tạo và khởi động containers, networks, volumes. Kéo images nếu chưa có.
    *   `-d` (detached): Chạy ở chế độ nền.

*   **Xem trạng thái các container:**
    ```bash
    docker compose ps
    ```
    *   Liệt kê các container được quản lý bởi Compose và trạng thái của chúng (running, exited...).

*   **Xem logs:**
    ```bash
    docker compose logs
    ```
    *   Hiển thị logs tổng hợp từ tất cả các services.
    *   `docker compose logs <tên_service>`: Xem log của một service cụ thể.
    *   `docker compose logs -f`: Theo dõi logs trực tiếp (live tail).
    *   `docker compose logs --tail=N`: Xem N dòng log cuối cùng.

*   **Dừng và xóa tài nguyên:**
    ```bash
    docker compose down
    ```
    *   Dừng và xóa các containers, networks được tạo bởi Compose.
    *   **Lưu ý:** Mặc định không xóa named volumes. Dùng `docker compose down --volumes` để xóa cả volumes (cẩn thận!).

*   **Khởi động lại services:**
    ```bash
    docker compose restart [tên_service...]
    ```

*   **Tạm dừng/Tiếp tục services:**
    ```bash
    docker compose pause [tên_service...]
    docker compose unpause [tên_service...]
    ```

---

## Phần 3: Tìm Hiểu Sâu Về Các Thành Phần Cấu Hình `docker-compose.yml`

Để kiểm soát ứng dụng đa container một cách hiệu quả, bạn cần hiểu rõ các tùy chọn cấu hình chính trong tệp `docker-compose.yml`.

### 3.1. `volumes`: Quản Lý Dữ Liệu Bền Vững

Volumes là cơ chế để lưu trữ dữ liệu bên ngoài vòng đời của container, đảm bảo dữ liệu không bị mất khi container bị xóa.

*   **Named Volumes (Khuyến nghị):**
    *   **Cách hoạt động:** Docker quản lý hoàn toàn volume này. Dữ liệu được lưu trữ trong một khu vực đặc biệt trên host, độc lập với container.
    *   **Ưu điểm:** Dễ quản lý, độc lập với host, dễ backup/di chuyển.
    *   **Cấu hình:**
        ```yaml
        services:
          database:
            image: mysql:8.0
            volumes:
              # Cú pháp: <tên_named_volume>:<đường_dẫn_trong_container>
              - mysql_data:/var/lib/mysql

        volumes:
          mysql_data: {} # Khai báo named volume ở cấp cao nhất
        ```

*   **Bind Mounts (Thường dùng cho Development):**
    *   **Cách hoạt động:** Ánh xạ trực tiếp một thư mục/tệp từ máy *host* vào container.
    *   **Ưu điểm:** Thay đổi trên host phản ánh ngay trong container (live-reload code). Tiện lợi cho việc mount code hoặc tệp cấu hình.
    *   **Nhược điểm:** Phụ thuộc đường dẫn host, có thể gây vấn đề về quyền, kém linh hoạt.
    *   **Cấu hình:**
        ```yaml
        services:
          web:
            build: .
            volumes:
              # Cú pháp: <đường_dẫn_host>:<đường_dẫn_container>[:ro]
              - ./app_code:/usr/src/app      # Mount code từ thư mục hiện tại
              - ./config/settings.ini:/etc/app/settings.ini:ro # Mount file config (read-only)
        ```

### 3.2. `networks`: Kết Nối Các Services

Mạng là yếu tố then chốt để các container giao tiếp với nhau.

*   **Mạng Bridge Mặc Định:**
    *   Compose tự động tạo một mạng bridge riêng cho các service trong cùng tệp `yml`.
    *   Các service trong mạng này có thể giao tiếp với nhau qua *tên service* làm hostname (ví dụ: `web` có thể gọi `db` qua hostname `db`).

*   **Custom Networks (Mạng Tùy Chỉnh):**
    *   **Khi nào cần:** Khi muốn kiểm soát kết nối chi tiết hơn (ví dụ: chia lớp frontend/backend), kết nối với mạng Docker bên ngoài, hoặc sử dụng driver mạng khác.
    *   **Cấu hình:**
        ```yaml
        services:
          proxy:
            image: traefik:v2.5
            networks:
              - webnet # Chỉ tham gia mạng webnet
            ports:
              - "80:80"
              - "443:443"
          api:
            image: my_api:latest
            networks:
              - webnet # Có thể truy cập từ proxy
              - internal # Có thể truy cập database
          database:
            image: redis:alpine
            networks:
              - internal # Chỉ tham gia mạng internal, không bị lộ ra webnet

        networks:
          webnet: {}   # Khai báo mạng webnet (dùng driver bridge mặc định)
          internal: {} # Khai báo mạng internal
        ```

### 3.3. `ports` vs `expose`: Công Khai Cổng Container

*   **`ports` (Ánh xạ cổng):**
    *   **Mục đích:** Cho phép truy cập service từ *bên ngoài máy host*.
    *   **Cách hoạt động:** Tạo quy tắc chuyển tiếp từ `HOST_PORT` đến `CONTAINER_PORT`.
    *   **Cú pháp:** `"HOST_PORT:CONTAINER_PORT"` (ví dụ: `"8080:80"`).
    *   **Ví dụ:**
        ```yaml
        services:
          grafana:
            image: grafana/grafana
            ports:
              - "3000:3000" # Truy cập Grafana UI qua http://localhost:3000
        ```

*   **`expose`:**
    *   **Mục đích:** *Chỉ khai báo* cổng mà container dự định lắng nghe, chủ yếu mang tính tài liệu. **Không** làm cho cổng đó truy cập được từ host.
    *   **Cách hoạt động:** Các service khác trong cùng mạng Docker có thể kết nối tới cổng này (nhưng thường không cần `expose` để làm việc này).
    *   **Cú pháp:** List các cổng (ví dụ: `expose: ["8000"]`).
    *   **Tóm lại:** Dùng `ports` khi cần truy cập từ host. Dùng `expose` (hoặc không dùng gì) nếu chỉ cần giao tiếp nội bộ giữa các container.

### 3.4. `environment` và `env_file`: Truyền Biến Môi Trường

Đây là cách phổ biến để cấu hình ứng dụng bên trong container.

*   **`environment`:**
    *   **Cách hoạt động:** Định nghĩa biến môi trường trực tiếp trong tệp `yml`.
    *   **Cú pháp:** Dạng list (`- VAR=value`) hoặc map (`VAR: value`).
    *   **Ví dụ:**
        ```yaml
        services:
          backend:
            image: myapi
            environment:
              DATABASE_URL: "postgresql://user:pass@db:5432/mydb"
              CACHE_ENABLED: "true"
              LOG_LEVEL: "info"
        ```

*   **`env_file`:**
    *   **Cách hoạt động:** Chỉ định một hoặc nhiều tệp `.env` chứa danh sách biến môi trường (`VAR=value`). Compose sẽ đọc các tệp này.
    *   **Ưu điểm:** Giữ tệp `yml` gọn gàng, tách biệt cấu hình, **tránh commit secrets vào Git**.
    *   **Cấu hình:**
        ```yaml
        # File: .env
        SECRET_KEY=a_very_secret_value
        API_TIMEOUT=30

        # File: docker-compose.yml
        services:
          worker:
            image: myworker
            env_file:
              - .env # Tải biến từ file .env
            environment:
              # Biến ở đây sẽ ghi đè biến cùng tên trong env_file
              DEBUG_MODE: "false"
        ```
    *   **Thứ tự ưu tiên:** Biến trong shell > Biến trong `environment` > Biến trong `env_file`.

### 3.5. `depends_on`: Thứ Tự Khởi Động Cơ Bản

*   **Mục đích:** Đảm bảo một service chỉ khởi động *sau khi* các service mà nó phụ thuộc đã được khởi động.
*   **Quan trọng - Giới hạn:** `depends_on` **chỉ đợi container được khởi động**, **không đợi ứng dụng bên trong sẵn sàng** (ví dụ: DB chưa khởi tạo xong).
*   **Cấu hình:**
    ```yaml
    services:
      webapp:
        build: .
        depends_on:
          - database # Webapp sẽ khởi động sau khi container 'database' khởi động
          - cache    # và sau khi container 'cache' khởi động
        # ...
      database:
        image: mysql
        # ...
      cache:
        image: redis
        # ...
    ```
*   **Giải pháp cho việc chờ sẵn sàng:** Sử dụng các kỹ thuật như `healthcheck` (sẽ đề cập ở phần sau), script `wait-for-it.sh`, hoặc xử lý logic retry trong ứng dụng.

---

## Phần 4: Tính Năng Nâng Cao và Kỹ Thuật Hữu Ích

Khi ứng dụng trở nên lớn hơn hoặc bạn cần quy trình làm việc phức tạp hơn, các tính năng nâng cao của Docker Compose sẽ phát huy tác dụng.

### 4.1. `profiles`: Chạy Nhóm Service Tùy Chọn

*   **Mục đích:** Cho phép bạn nhóm các service lại với nhau và chỉ khởi động những nhóm (profiles) cần thiết. Rất hữu ích để tách biệt các service chỉ dùng cho development, testing hoặc các tính năng tùy chọn.
*   **Cách dùng:** Thêm thuộc tính `profiles` (dạng list) vào service. Service không có `profiles` sẽ luôn chạy mặc định.
    ```yaml
    services:
      # ... các service luôn chạy ...
      debugger:
        image: mydebugger
        profiles: ["debug"] # Chỉ chạy khi profile 'debug' được kích hoạt
      mailhog:
        image: mailhog/mailhog
        profiles: ["dev"]   # Chỉ chạy khi profile 'dev' được kích hoạt
      tests:
        image: mytests
        profiles: ["test"]  # Chỉ chạy khi profile 'test' được kích hoạt
    ```
*   **Chạy với profiles:**
    *   `docker compose up -d`: Chạy các service mặc định (không có profile).
    *   `docker compose --profile dev up -d`: Chạy service mặc định + service có profile `dev`.
    *   `docker compose --profile dev --profile debug up -d`: Chạy service mặc định + service có profile `dev` + service có profile `debug`.
*   **Lợi ích:** Giữ tệp Compose gọn gàng, linh hoạt chọn service cần chạy, dễ tích hợp công cụ phụ trợ.

### 4.2. `extends`: Tái Sử Dụng Cấu Hình (DRY)

*   **Mục đích:** Giảm lặp lại cấu hình bằng cách cho phép một service kế thừa cấu hình từ một service khác (trong cùng tệp hoặc tệp khác).
*   **Cách dùng:** Sử dụng từ khóa `extends` chỉ định `file` (tùy chọn) và `service` nguồn.
    ```yaml
    # File: common.yml
    services:
      base-app:
        image: python:3.9-slim
        working_dir: /app
        environment:
          - PYTHONUNBUFFERED=1

    # File: docker-compose.yml
    services:
      web:
        extends:
          file: common.yml
          service: base-app
        build: ./web_app
        ports: ["8000:8000"]
        command: python manage.py runserver 0.0.0.0:8000

      worker:
        extends:
          file: common.yml
          service: base-app
        build: ./worker_app
        command: celery -A tasks worker --loglevel=info
    ```
*   **Lợi ích:** Dễ bảo trì (sửa một nơi, áp dụng nhiều nơi), tạo sự nhất quán.
*   **Lưu ý:** `extends` có một số hạn chế, không phải mọi thuộc tính đều có thể kế thừa hoặc ghi đè một cách trực quan.

### 4.3. Biến Thay Thế và `.env`: Linh Hoạt Hóa Cấu Hình

*   **Mục đích:** Sử dụng biến (`${VAR}` hoặc `$VAR`) trong tệp Compose để lấy giá trị từ biến môi trường của shell hoặc tệp `.env`. Giúp quản lý cấu hình khác nhau giữa các môi trường và bảo mật secrets.
*   **Cách dùng:**
    *   Tạo tệp `.env` cùng cấp `docker-compose.yml`.
    *   Đặt biến: `KEY=VALUE`.
    *   Sử dụng trong `yml`: `${KEY}`.
    *   Cú pháp hữu ích:
        *   `${VAR:-default}`: Dùng giá trị của `VAR`, nếu không có thì dùng `default`.
        *   `${VAR?error message}`: Báo lỗi nếu `VAR` chưa được đặt.
    ```yaml
    # .env
    WEB_IMAGE_TAG=1.5
    EXTERNAL_PORT=8080

    # docker-compose.yml
    services:
      frontend:
        image: myfrontend:${WEB_IMAGE_TAG:-latest}
        ports:
          - "${EXTERNAL_PORT}:80"
    ```
*   **Lợi ích:** Linh hoạt, tách biệt cấu hình môi trường, quản lý secrets tốt hơn.

### 4.4. `healthcheck`: Đảm Bảo Service Sẵn Sàng

*   **Mục đích:** Định nghĩa cách Docker kiểm tra xem ứng dụng bên trong container có thực sự hoạt động hay không, khắc phục hạn chế của `depends_on`.
*   **Cách dùng:** Thêm khối `healthcheck` vào service.
    ```yaml
    services:
      api:
        image: myapi
        healthcheck:
          test: ["CMD", "curl", "--fail", "http://localhost:5000/health"]
          interval: 1m30s  # Kiểm tra mỗi 1 phút 30 giây
          timeout: 10s     # Thời gian chờ tối đa cho lệnh test
          retries: 3       # Số lần thử lại nếu thất bại
          start_period: 40s # Chờ 40s sau khi container start mới bắt đầu kiểm tra
      web:
        image: myfrontend
        depends_on:
          api:
            condition: service_healthy # Chỉ start web khi api 'healthy'
    ```
*   **Lợi ích:** Khởi động ứng dụng ổn định hơn, tích hợp tốt với cơ chế tự phục hồi của orchestrator.

### 4.5. `build` Context và `dockerfile`: Tùy Chỉnh Quá Trình Build

Khi dùng `build` để Compose tự xây dựng image:

*   **`context`:** Đường dẫn đến thư mục chứa tài nguyên build (code, Dockerfile...). Toàn bộ thư mục này sẽ được gửi đến Docker daemon.
*   **`dockerfile`:** Tên tệp Dockerfile (nếu khác `Dockerfile`), đường dẫn tương đối so với `context`.
*   **Cấu hình chi tiết:**
    ```yaml
    services:
      backend:
        build:
          context: ./backend_src # Thư mục chứa code backend
          dockerfile: Dockerfile.prod # Dùng file Dockerfile.prod
          args: # Truyền build-time arguments
            - BUILD_VERSION=${VERSION:-dev}
          target: production # Build đến stage 'production' (cho multi-stage)
    ```
*   **Lợi ích:** Tổ chức dự án linh hoạt, dùng Dockerfile khác nhau cho các môi trường/mục đích, tối ưu build.

## Phần 5: Thực Tiễn Tốt Nhất (Best Practices) Trong Môi Trường Development

Áp dụng các thực tiễn sau giúp quy trình phát triển với Docker Compose hiệu quả và dễ dàng hơn.

### 5.1. Cấu trúc Dự án và File Compose (`.yml` vs `.override.yml`)

*   **Thực tiễn:**
    *   `docker-compose.yml`: Chứa cấu hình cốt lõi, ổn định, gần giống production.
    *   `docker-compose.override.yml`: Chứa cấu hình *chỉ dành cho dev* (mount code, expose port debug, chạy lệnh dev, service phụ trợ dev...). Compose tự động merge file này.
*   **Lợi ích:** Tách biệt rõ ràng, giảm lặp lại, dễ bảo trì, nhất quán.

### 5.2. Quản lý Cấu hình và Secrets với `.env`

*   **Thực tiễn:**
    *   Dùng `.env` cho các biến cấu hình (port, user/pass dev...).
    *   **KHÔNG commit `.env` chứa secret vào Git.** Dùng `.env.example` làm mẫu.
*   **Lợi ích:** Bảo mật, linh hoạt, tùy chỉnh cá nhân.

### 5.3. Tối ưu Build Image

*   **Thực tiễn:**
    *   `.dockerignore`: Loại bỏ file không cần thiết khỏi build context (`.git`, `node_modules`...).
    *   Tận dụng cache Dockerfile: Đặt lệnh ít thay đổi lên đầu, `COPY` code gần cuối.
    *   Multi-stage builds: Giảm kích thước image cuối cùng.
*   **Lợi ích:** Build nhanh hơn, image nhỏ hơn, bảo mật hơn.

### 5.4. Quản lý Volumes và Networks

*   **Thực tiễn:**
    *   Bind mount code (`./code:/app`) trong `override.yml` để live-reload.
    *   Named volumes cho data bền vững (DB, uploads...). Đặt tên rõ ràng.
    *   Hiểu và thường chỉ cần dùng mạng bridge mặc định của Compose trong dev.
    *   Dọn dẹp volume cũ bằng `docker volume prune` (cẩn thận).
*   **Lợi ích:** Phát triển hiệu quả, bảo toàn dữ liệu, kết nối tin cậy.

### 5.5. Logging Hiệu Quả

*   **Thực tiễn:**
    *   Cấu hình ứng dụng log ra STDOUT/STDERR.
    *   Sử dụng `docker compose logs [-f] [--tail=N] [service_name]` để xem và theo dõi log.
*   **Lợi ích:** Debug dễ dàng, chuẩn mực cho container.

---

**Hoàn thành!**

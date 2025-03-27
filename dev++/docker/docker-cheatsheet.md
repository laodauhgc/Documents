# Tổng Hợp Các Lệnh Docker Chi Tiết

## Quản Lý Container

### `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`
Tạo và khởi chạy một container mới từ một image. Đây là lệnh phức tạp và mạnh mẽ nhất.
*   `-d, --detach`: Chạy container ở chế độ nền (detached mode).
*   `-it`: Chạy container ở chế độ tương tác (interactive) và cấp phát một pseudo-TTY. Thường dùng để truy cập shell của container.
*   `-p, --publish hostPort:containerPort`: Ánh xạ cổng từ máy host tới container.
*   `-v, --volume hostPath:containerPath`: Mount một volume hoặc bind mount một thư mục từ host vào container.
*   `--name string`: Đặt tên cho container.
*   `--rm`: Tự động xóa container khi nó dừng.
*   `--network string`: Kết nối container vào một mạng Docker cụ thể.
*   `-e, --env list`: Thiết lập biến môi trường bên trong container.
*   `--restart string`: Chính sách khởi động lại container (ví dụ: `no`, `on-failure[:max-retries]`, `unless-stopped`, `always`).
*   `--entrypoint string`: Ghi đè entrypoint mặc định của image.
*   `--memory bytes`: Giới hạn bộ nhớ RAM cho container (ví dụ: `512m`, `1g`).
*   `--cpus decimal`: Giới hạn số lượng CPU core (ví dụ: `0.5`, `2`).
*   `--link container_name:alias`: (Cũ, nên dùng network thay thế) Liên kết với container khác.

**Ví dụ:** `docker run -d -p 8080:80 --name my-web-server nginx`

### `docker ps [OPTIONS]`
Liệt kê các container đang chạy.
*   `-a, --all`: Hiển thị tất cả container (bao gồm cả những container đã dừng).
*   `-q, --quiet`: Chỉ hiển thị ID của container.
*   `-s, --size`: Hiển thị kích thước file của container.
*   `--filter key=value`: Lọc kết quả (ví dụ: `--filter "status=exited"`, `--filter "name=my-web"`).

### `docker start [OPTIONS] CONTAINER [CONTAINER...]`
Khởi động một hoặc nhiều container đã dừng.

### `docker stop [OPTIONS] CONTAINER [CONTAINER...]`
Dừng một hoặc nhiều container đang chạy (gửi tín hiệu SIGTERM, sau đó SIGKILL nếu không dừng).
*   `-t, --time int`: Số giây chờ trước khi gửi SIGKILL (mặc định 10).

### `docker restart [OPTIONS] CONTAINER [CONTAINER...]`
Khởi động lại một hoặc nhiều container.

### `docker pause CONTAINER [CONTAINER...]`
Tạm dừng tất cả các tiến trình trong một hoặc nhiều container.

### `docker unpause CONTAINER [CONTAINER...]`
Tiếp tục các tiến trình đã bị tạm dừng trong một hoặc nhiều container.

### `docker kill [OPTIONS] CONTAINER [CONTAINER...]`
Gửi tín hiệu SIGKILL để buộc dừng một hoặc nhiều container ngay lập tức.

### `docker rm [OPTIONS] CONTAINER [CONTAINER...]`
Xóa một hoặc nhiều container đã dừng.
*   `-f, --force`: Buộc xóa container đang chạy (sử dụng SIGKILL).
*   `-v, --volumes`: Xóa các anonymous volume liên kết với container.

### `docker exec [OPTIONS] CONTAINER COMMAND [ARG...]`
Thực thi một lệnh bên trong một container đang chạy.
*   `-it`: Chạy lệnh ở chế độ tương tác và TTY (thường dùng để vào shell: `docker exec -it my-container /bin/bash`).
*   `-d, --detach`: Chạy lệnh ở chế độ nền.
*   `-u, --user string`: Chạy lệnh với user/UID cụ thể.

### `docker logs [OPTIONS] CONTAINER`
Xem log (stdout/stderr) của một container.
*   `-f, --follow`: Theo dõi log output liên tục.
*   `--tail string`: Hiển thị số dòng log cuối cùng (mặc định "all").
*   `--since string`: Hiển thị log từ một mốc thời gian (ví dụ: `10m`, `2023-10-27T10:00:00`).
*   `--until string`: Hiển thị log đến một mốc thời gian.
*   `-t, --timestamps`: Hiển thị timestamp cho mỗi dòng log.

### `docker inspect [OPTIONS] NAME|ID [NAME|ID...]`
Hiển thị thông tin chi tiết (dạng JSON) về container, image, network, volume,...
*   `--format string`: Định dạng output bằng Go template.

### `docker stats [OPTIONS] [CONTAINER...]`
Hiển thị thống kê tài nguyên (CPU, RAM, Network I/O, Disk I/O) của các container đang chạy theo thời gian thực.

### `docker top CONTAINER [ps OPTIONS]`
Hiển thị các tiến trình đang chạy bên trong một container.

### `docker attach [OPTIONS] CONTAINER`
Gắn kết (attach) local standard input, output, và error streams vào một container đang chạy.

### `docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-`
### `docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH`
Sao chép file/thư mục giữa container và máy host.

### `docker diff CONTAINER`
Kiểm tra các thay đổi trong filesystem của container so với image gốc.

### `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
Tạo một image mới từ các thay đổi của một container. (Thường không khuyến khích sử dụng, nên dùng Dockerfile).

### `docker rename CONTAINER NEW_NAME`
Đổi tên một container.

### `docker update [OPTIONS] CONTAINER [CONTAINER...]`
Cập nhật cấu hình của một hoặc nhiều container (ví dụ: giới hạn tài nguyên).

### `docker wait CONTAINER [CONTAINER...]`
Block cho đến khi một hoặc nhiều container dừng, sau đó in ra exit code của chúng.

### `docker export [OPTIONS] CONTAINER`
Xuất filesystem của một container thành một file tar (không bao gồm metadata, layer).

### `docker port CONTAINER [PRIVATE_PORT[/PROTO]]`
Liệt kê các ánh xạ cổng hoặc ánh xạ cụ thể cho một container.

## Quản Lý Image

### `docker images [OPTIONS] [REPOSITORY[:TAG]]`
Liệt kê các image trên máy local.
*   `-a, --all`: Hiển thị tất cả image (bao gồm cả image trung gian).
*   `-q, --quiet`: Chỉ hiển thị ID của image.
*   `--filter key=value`: Lọc kết quả (ví dụ: `--filter "dangling=true"`).
*   `--format string`: Định dạng output.

### `docker build [OPTIONS] PATH | URL | -`
Xây dựng một image từ Dockerfile.
*   `-t, --tag list`: Đặt tên và tag cho image (ví dụ: `my-app:1.0`).
*   `-f, --file string`: Chỉ định đường dẫn đến Dockerfile (mặc định là `PATH/Dockerfile`).
*   `--build-arg list`: Thiết lập các biến build-time arguments.
*   `--no-cache`: Không sử dụng cache khi build.
*   `--pull`: Luôn cố gắng pull image base mới nhất.
*   `.`: Dấu chấm (`.`) thường được dùng cuối lệnh để chỉ định build context là thư mục hiện tại.
*   `--platform string`: Chỉ định nền tảng để build (cho multi-platform).
*   **BuildKit:** Thường được bật mặc định trong các phiên bản Docker mới, cung cấp hiệu năng build tốt hơn. Có thể bật/tắt bằng biến môi trường `DOCKER_BUILDKIT=1` hoặc `0`.

### `docker pull [OPTIONS] NAME[:TAG|@DIGEST]`
Tải một image hoặc repository từ registry (mặc định là Docker Hub).
*   `-a, --all-tags`: Tải tất cả các tag của repository.
*   `--platform string`: Chỉ định nền tảng của image cần pull.

### `docker push [OPTIONS] NAME[:TAG]`
Đẩy một image hoặc repository lên registry.
*   `-a, --all-tags`: Đẩy tất cả các tag của repository.

### `docker rmi [OPTIONS] IMAGE [IMAGE...]`
Xóa một hoặc nhiều image khỏi máy local.
*   `-f, --force`: Buộc xóa image (ngay cả khi đang được sử dụng bởi container đã dừng).

### `docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`
Tạo một tag mới tham chiếu đến một image đã có.

### `docker history [OPTIONS] IMAGE`
Hiển thị lịch sử (các layer) của một image.

### `docker save [OPTIONS] IMAGE [IMAGE...]`
Lưu một hoặc nhiều image thành một file tar archive (bao gồm các layer và metadata).
*   `-o, --output string`: Ghi output vào file thay vì STDOUT.

### `docker load [OPTIONS]`
Nạp một image từ một file tar archive (tạo bởi `docker save`).
*   `-i, --input string`: Đọc từ file tar thay vì STDIN.
*   `-q, --quiet`: Im lặng.

### `docker search [OPTIONS] TERM`
Tìm kiếm image trên Docker Hub.
*   `--filter key=value`: Lọc kết quả (ví dụ: `--filter is-official=true`).
*   `--limit int`: Giới hạn số lượng kết quả (mặc định 25).

### `docker image prune [OPTIONS]`
Xóa các image không sử dụng (dangling images - không có tag và không được container nào tham chiếu).
*   `-a, --all`: Xóa tất cả image không được sử dụng bởi ít nhất một container (kể cả image có tag).
*   `--filter key=value`: Lọc image cần xóa.
*   `-f, --force`: Không yêu cầu xác nhận.

### `docker scan [OPTIONS] IMAGE`
(Sử dụng Docker Scout hoặc Snyk) Quét image để tìm lỗ hổng bảo mật.

## Quản Lý Mạng (Network)

### `docker network ls [OPTIONS]`
Liệt kê các mạng Docker.
*   `-q, --quiet`: Chỉ hiển thị ID mạng.
*   `--filter key=value`: Lọc kết quả.

### `docker network create [OPTIONS] NETWORK`
Tạo một mạng Docker mới.
*   `-d, --driver string`: Chỉ định driver mạng (ví dụ: `bridge`, `overlay`, `macvlan`, mặc định `bridge`).
*   `--subnet string`: Chỉ định subnet cho mạng (ví dụ: `172.18.0.0/16`).
*   `--gateway string`: Chỉ định gateway cho subnet.
*   `--attachable`: Cho phép container độc lập kết nối vào mạng overlay.

### `docker network inspect [OPTIONS] NETWORK [NETWORK...]`
Hiển thị thông tin chi tiết về một hoặc nhiều mạng.
*   `--format string`: Định dạng output.

### `docker network connect [OPTIONS] NETWORK CONTAINER`
Kết nối một container vào một mạng.

### `docker network disconnect [OPTIONS] NETWORK CONTAINER`
Ngắt kết nối một container khỏi một mạng.
*   `-f, --force`: Buộc ngắt kết nối.

### `docker network rm NETWORK [NETWORK...]`
Xóa một hoặc nhiều mạng.

### `docker network prune [OPTIONS]`
Xóa tất cả các mạng không được sử dụng bởi bất kỳ container nào.
*   `--filter key=value`: Lọc mạng cần xóa.
*   `-f, --force`: Không yêu cầu xác nhận.

## Quản Lý Volume

### `docker volume ls [OPTIONS]`
Liệt kê các volume.
*   `-q, --quiet`: Chỉ hiển thị tên volume.
*   `--filter key=value`: Lọc kết quả (ví dụ: `--filter "dangling=true"`).

### `docker volume create [OPTIONS] [VOLUME]`
Tạo một volume mới.
*   `-d, --driver string`: Chỉ định volume driver (mặc định `local`).
*   `-o, --opt map`: Thiết lập các tùy chọn cho driver.
*   `--label list`: Gán metadata (label) cho volume.

### `docker volume inspect [OPTIONS] VOLUME [VOLUME...]`
Hiển thị thông tin chi tiết về một hoặc nhiều volume.
*   `--format string`: Định dạng output.

### `docker volume rm [OPTIONS] VOLUME [VOLUME...]`
Xóa một hoặc nhiều volume.
*   `-f, --force`: Buộc xóa volume (hữu ích nếu có lỗi).

### `docker volume prune [OPTIONS]`
Xóa tất cả các local volume không được sử dụng bởi bất kỳ container nào.
*   `--filter key=value`: Lọc volume cần xóa.
*   `-f, --force`: Không yêu cầu xác nhận.

## Hệ Thống Docker & Registry

### `docker info [OPTIONS]`
Hiển thị thông tin toàn hệ thống Docker (phiên bản, số container, số image, storage driver, OS,...).
*   `--format string`: Định dạng output.

### `docker version [OPTIONS]`
Hiển thị thông tin phiên bản Docker client và server.
*   `--format string`: Định dạng output.

### `docker system df [OPTIONS]`
Hiển thị dung lượng ổ đĩa Docker đang sử dụng (bởi images, containers, volumes, build cache).
*   `-v, --verbose`: Hiển thị chi tiết hơn.

### `docker system prune [OPTIONS]`
Dọn dẹp tài nguyên không sử dụng:
*   Mặc định: Xóa tất cả container đã dừng, tất cả mạng không sử dụng, tất cả dangling image, tất cả build cache.
*   `-a, --all`: Xóa thêm tất cả image không được sử dụng bởi ít nhất một container.
*   `--volumes`: Xóa thêm tất cả local volume không được sử dụng.
*   `-f, --force`: Không yêu cầu xác nhận.

### `docker system events [OPTIONS]`
Xem các sự kiện hệ thống Docker theo thời gian thực.
*   `--since string`: Lấy sự kiện từ một mốc thời gian.
*   `--until string`: Lấy sự kiện đến một mốc thời gian.
*   `--filter key=value`: Lọc sự kiện (ví dụ: `--filter 'type=container'`).

### `docker login [OPTIONS] [SERVER]`
Đăng nhập vào một Docker registry.
*   `-u, --username string`: Tên đăng nhập.
*   `-p, --password string`: Mật khẩu.
*   `--password-stdin`: Đọc mật khẩu từ STDIN.

### `docker logout [SERVER]`
Đăng xuất khỏi một Docker registry.

## Docker Compose (Lệnh mới tích hợp)

**Lưu ý:** Trước đây dùng `docker-compose` (dấu gạch nối), nay tích hợp thành `docker compose` (dấu cách). Chức năng tương tự nhưng được khuyến khích dùng lệnh mới.

### `docker compose up [OPTIONS] [SERVICE...]`
Tạo và khởi chạy các service được định nghĩa trong file `compose.yaml` (hoặc `docker-compose.yml`).
*   `-d`: Chạy ở chế độ nền.
*   `--build`: Build image trước khi khởi chạy service.
*   `--force-recreate`: Tạo lại container ngay cả khi cấu hình không thay đổi.
*   `--no-deps`: Không khởi chạy các service phụ thuộc.
*   `--scale SERVICE=NUM`: Scale số lượng container cho một service.
*   `-f, --file FILE`: Chỉ định file compose thay thế.

### `docker compose down [OPTIONS]`
Dừng và xóa các container, network, volume được tạo bởi `up`.
*   `-v, --volumes`: Xóa các named volume được định nghĩa trong `volumes`.
*   `--rmi type`: Xóa image (type: `all` hoặc `local`).
*   `--remove-orphans`: Xóa các container của service không còn được định nghĩa trong file compose.

### `docker compose ps [OPTIONS] [SERVICE...]`
Liệt kê các container của các service.

### `docker compose logs [OPTIONS] [SERVICE...]`
Xem log output từ các service.
*   `-f, --follow`: Theo dõi log.

### `docker compose exec [OPTIONS] SERVICE COMMAND [ARGS...]`
Thực thi một lệnh trong một container của service đang chạy.

### `docker compose build [OPTIONS] [SERVICE...]`
Build hoặc rebuild các service.

### `docker compose pull [OPTIONS] [SERVICE...]`
Pull image cho các service.

### `docker compose push [OPTIONS] [SERVICE...]`
Push image cho các service.

### `docker compose config [OPTIONS]`
Kiểm tra và xem cấu hình compose đã được tổng hợp.

### `docker compose start [SERVICE...]`
Khởi chạy các service đã tồn tại (đã được tạo nhưng đang dừng).

### `docker compose stop [OPTIONS] [SERVICE...]`
Dừng các service đang chạy mà không xóa chúng.

### `docker compose restart [OPTIONS] [SERVICE...]`
Khởi động lại các container của service.

### `docker compose rm [OPTIONS] [SERVICE...]`
Xóa các container đã dừng của service.

### `docker compose run [OPTIONS] SERVICE [COMMAND] [ARGS...]`
Chạy một lệnh một lần trong một container mới cho một service (thường dùng để chạy task ngắn hạn).

## Docker Swarm (Orchestration)

### `docker swarm init [OPTIONS]`
Khởi tạo một swarm mới (biến node hiện tại thành manager đầu tiên).
*   `--advertise-addr string`: Địa chỉ IP để các node khác kết nối tới.

### `docker swarm join --token <TOKEN> <MANAGER-IP>:<PORT>`
Tham gia vào một swarm đã tồn tại với vai trò worker hoặc manager. Token được lấy từ manager.

### `docker swarm join-token [OPTIONS] (worker|manager)`
Hiển thị hoặc xoay vòng token để tham gia swarm.

### `docker swarm leave [OPTIONS]`
Rời khỏi swarm (cho worker hoặc manager).
*   `--force`: Buộc rời khỏi, ngay cả khi là manager cuối cùng.

### `docker swarm update [OPTIONS]`
Cập nhật cấu hình của swarm.

### `docker node ls [OPTIONS]`
Liệt kê các node trong swarm (chỉ chạy trên manager).

### `docker node inspect [OPTIONS] NODE [NODE...]`
Hiển thị thông tin chi tiết về một hoặc nhiều node.

### `docker node update [OPTIONS] NODE`
Cập nhật thông tin node (ví dụ: role, availability, label).

### `docker node promote NODE [NODE...]`
Nâng cấp một hoặc nhiều worker thành manager.

### `docker node demote NODE [NODE...]`
Giáng cấp một hoặc nhiều manager thành worker.

### `docker node rm [OPTIONS] NODE [NODE...]`
Xóa một hoặc nhiều node khỏi swarm (node phải ở trạng thái `down`).

### `docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]`
Tạo một service mới trong swarm.
*   `--name string`: Đặt tên cho service.
*   `--replicas uint`: Số lượng task (container) cần chạy.
*   `--publish mode=ingress,target=...,published=...`: Ánh xạ cổng cho service.
*   `--network string`: Kết nối service vào mạng overlay.
*   `--mount type=volume,...`: Mount volume vào service.
*   `--secret source=...,target=...`: Cung cấp secret cho service.
*   `--config source=...,target=...`: Cung cấp config cho service.
*   `--mode (replicated|global)`: Chế độ deployment (chạy số lượng replicas hoặc chạy 1 task trên mỗi node).

### `docker service ls [OPTIONS]`
Liệt kê các service trong swarm.

### `docker service inspect [OPTIONS] SERVICE [SERVICE...]`
Hiển thị thông tin chi tiết về một hoặc nhiều service.

### `docker service ps [OPTIONS] SERVICE [SERVICE...]`
Liệt kê các task (container) của một hoặc nhiều service.

### `docker service logs [OPTIONS] SERVICE|TASK`
Xem log của một service hoặc một task cụ thể.

### `docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]`
Scale số lượng replica cho một hoặc nhiều service.

### `docker service update [OPTIONS] SERVICE`
Cập nhật cấu hình của một service đang chạy.

### `docker service rm SERVICE [SERVICE...]`
Xóa một hoặc nhiều service.

### `docker stack deploy [OPTIONS] STACK`
Deploy một stack mới hoặc cập nhật stack đã có từ file compose.
*   `-c, --compose-file string`: Đường dẫn đến file compose.
*   `--with-registry-auth`: Gửi thông tin xác thực registry đến các node swarm.

### `docker stack ls [OPTIONS]`
Liệt kê các stack đang chạy.

### `docker stack ps [OPTIONS] STACK`
Liệt kê các task trong một stack.

### `docker stack services [OPTIONS] STACK`
Liệt kê các service trong một stack.

### `docker stack rm STACK [STACK...]`
Xóa một hoặc nhiều stack.

### `docker secret create [OPTIONS] SECRET file|-`
Tạo một secret từ file hoặc STDIN.

### `docker secret ls [OPTIONS]`
Liệt kê các secret.

### `docker secret inspect [OPTIONS] SECRET [SECRET...]`
Hiển thị thông tin chi tiết về secret.

### `docker secret rm SECRET [SECRET...]`
Xóa một hoặc nhiều secret.

### `docker config create [OPTIONS] CONFIG file|-`
Tạo một config từ file hoặc STDIN.

### `docker config ls [OPTIONS]`
Liệt kê các config.

### `docker config inspect [OPTIONS] CONFIG [CONFIG...]`
Hiển thị thông tin chi tiết về config.

### `docker config rm CONFIG [CONFIG...]`
Xóa một hoặc nhiều config.

## Context Management

### `docker context create [OPTIONS] CONTEXT`
Tạo một context mới (để kết nối đến Docker daemon khác, ví dụ: remote host qua SSH, Docker Desktop).

### `docker context ls [OPTIONS]`
Liệt kê các context.

### `docker context use CONTEXT`
Chuyển sang sử dụng một context khác.

### `docker context inspect [OPTIONS] [CONTEXT...]`
Hiển thị thông tin chi tiết về context.

### `docker context rm [CONTEXT...]`
Xóa một hoặc nhiều context.

## Plugin Management

### `docker plugin ls [OPTIONS]`
Liệt kê các plugin đã cài đặt.

### `docker plugin install [OPTIONS] PLUGIN [PLUGIN_OPTS]`
Cài đặt một plugin.

### `docker plugin enable [OPTIONS] PLUGIN`
Kích hoạt một plugin đã cài đặt.

### `docker plugin disable [OPTIONS] PLUGIN`
Vô hiệu hóa một plugin.

### `docker plugin rm [OPTIONS] PLUGIN`
Xóa một plugin.

# Bun Cheatsheet (Tổng hợp lệnh Bun)

Bun là một bộ công cụ JavaScript & TypeScript tất-cả-trong-một cực nhanh, bao gồm trình chạy (runtime), trình quản lý gói (package manager), trình đóng gói (bundler), và trình chạy thử nghiệm (test runner).

## 🚀 Chạy File & Script

*   **Chạy file JavaScript/TypeScript:**
    ```bash
    bun run <file>.[js|ts|jsx|tsx|mjs|mts]
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun <file>.[js|ts|jsx|tsx|mjs|mts]
    ```
    *Ví dụ:* `bun run index.ts` hoặc `bun index.ts`

*   **Chạy file với chế độ theo dõi (watch mode):** Tự động chạy lại khi file thay đổi.
    ```bash
    bun --watch run <file>
    ```
    *Hoặc:*
    ```bash
    bun run --watch <file>
    ```

*   **Chạy file với chế độ hot-reload (thường dùng cho server):** Tải lại module mà không cần khởi động lại toàn bộ tiến trình.
    ```bash
    bun --hot <server_file>.ts
    ```

*   **Chạy script từ `package.json`:**
    ```bash
    bun run <script_name>
    ```
    *Hoặc ngắn gọn (nếu tên script không trùng tên file):*
    ```bash
    bun <script_name>
    ```
    *Ví dụ:* `bun run dev` hoặc `bun dev`

*   **Các script thông dụng:** Bun tự động nhận diện các script phổ biến.
    ```bash
    bun start
    bun test
    bun dev
    ```

*   **Chạy file/script với chế độ Debug:**
    ```bash
    bun --inspect run <file_or_script>
    bun --inspect-wait run <file_or_script> # Chờ debugger kết nối
    ```

*   **Truyền tham số vào script:** Dùng `--` để ngăn Bun xử lý các tham số tiếp theo.
    ```bash
    bun run <script_name> -- --arg1 --arg2=value
    ```

## 📦 Quản lý Dependency (Package Management)

*   **Khởi tạo dự án mới:** Tạo file `package.json`.
    ```bash
    bun init
    ```
    *Tạo với giá trị mặc định, không cần hỏi:*
    ```bash
    bun init -y
    ```

*   **Cài đặt tất cả dependency từ `package.json`:**
    ```bash
    bun install
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun i
    ```
    *Cài đặt từ `bun.lockb` nếu tồn tại (nhanh hơn, đảm bảo tính nhất quán):*
    ```bash
    bun install --frozen-lockfile
    ```

*   **Thêm dependency:** Mặc định thêm vào `dependencies`.
    ```bash
    bun add <package_name>
    bun add <package1> <package2>
    ```

*   **Thêm dev dependency:**
    ```bash
    bun add --dev <package_name>
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun add -d <package_name>
    ```

*   **Thêm optional dependency:**
    ```bash
    bun add --optional <package_name>
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun add -o <package_name>
    ```

*   **Thêm dependency với phiên bản chính xác:**
    ```bash
    bun add --exact <package_name>
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun add -E <package_name>
    ```

*   **Xóa dependency:**
    ```bash
    bun remove <package_name>
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun rm <package_name>
    ```

*   **Cập nhật dependency:**
    *Cập nhật một package cụ thể:*
    ```bash
    bun update <package_name>
    ```
    *Cập nhật nhiều package:*
    ```bash
    bun update <package1> <package2>
    ```
    *Cập nhật tất cả các package lên phiên bản mới nhất cho phép trong `package.json`:*
    ```bash
    bun update
    ```

*   **Liệt kê các dependency đã cài:**
    ```bash
    bun pm ls
    # Hoặc chi tiết hơn
    bun pm ls --all
    ```

*   **Link package local (để phát triển):**
    *Trong thư mục package cần link:*
    ```bash
    bun link
    ```
    *Trong thư mục dự án muốn sử dụng package đã link:*
    ```bash
    bun link <package_name>
    ```

*   **Hủy link package:**
    *Trong thư mục dự án đang sử dụng package:*
    ```bash
    bun unlink <package_name>
    ```
    *Để hủy đăng ký package khỏi global links (trong thư mục package gốc):*
    ```bash
    bun unlink
    ```

*   **Xóa cache của Bun:**
    ```bash
    bun pm cache rm
    ```

## ✨ Tạo Dự án Nhanh (Scaffolding)

*   **Tạo dự án mới từ template:** Tương tự `npm create`, `yarn create`.
    ```bash
    bun create <template> [destination_folder]
    ```
    *Ví dụ:*
    ```bash
    bun create react ./my-react-app
    bun create next ./my-next-app
    bun create elysia ./my-elysia-app # Elysia là một framework phổ biến với Bun
    bun create vite ./my-vite-app    # Sử dụng template Vite
    ```
    
## 🛠️ Build & Đóng Gói (Bundling)

*   **Build dự án:**
    ```bash
    bun build <entrypoint_file(s)> --outdir <output_directory>
    ```
    *Ví dụ:*
    ```bash
    bun build ./src/index.ts --outdir ./dist
    ```

*   **Các tùy chọn build phổ biến:**
    *   `--target`: Chỉ định môi trường đích (`browser`, `node`, `bun`).
        ```bash
        bun build ./index.ts --outdir ./dist --target browser
        ```
    *   `--format`: Định dạng output (`esm`, `cjs`, `iife`). Mặc định Bun sẽ tự suy luận.
    *   `--minify`: Nén và tối ưu code output.
    *   `--sourcemap`: Tạo sourcemap (`inline` hoặc `external`).
    *   `--external <package>`: Đánh dấu package là external, không đóng gói vào bundle.
        ```bash
        bun build ./index.ts --outdir ./dist --external react --external react-dom
        ```
    *   `--splitting`: Tự động tách code thành các chunk nhỏ hơn.
    *   `--define <key=value>`: Thay thế biến global trong code.
    *   `--loader <.ext:loader>`: Chỉ định loader cho các loại file cụ thể.

## ✅ Chạy Test (Test Runner)

*   **Chạy tất cả các file test:** Bun tự động tìm các file có đuôi `.test.{js,jsx,ts,tsx}`, `.spec.{js,jsx,ts,tsx}` hoặc trong thư mục `test/`, `tests/`.
    ```bash
    bun test
    ```

*   **Chạy các file test cụ thể hoặc theo pattern:**
    ```bash
    bun test src/utils.test.ts
    bun test "src/**/*.spec.ts"
    ```

*   **Chạy test ở chế độ theo dõi (watch mode):**
    ```bash
    bun test --watch
    ```

*   **Tạo báo cáo độ bao phủ code (coverage):**
    ```bash
    bun test --coverage
    ```

*   **Chạy test với file preload:** Thực thi một script trước khi chạy test.
    ```bash
    bun test --preload ./setupTests.ts
    ```

*   **Lọc test theo tên:** Chỉ chạy các test case có tên khớp với pattern.
    ```bash
    bun test -t "user login"
    ```

## 💻 REPL (Read-Eval-Print Loop)

*   **Khởi động REPL:** Môi trường tương tác để chạy code JavaScript/TypeScript với Bun.
    ```bash
    bun repl
    ```
    *Hoặc đơn giản là chạy `bun` mà không có tham số:*
    ```bash
    bun
    ```

## ⚙️ Tiện ích & Thông tin

*   **Kiểm tra phiên bản Bun:**
    ```bash
    bun --version
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun -v
    ```

*   **Hiển thị trợ giúp chung:**
    ```bash
    bun --help
    ```
    *Hoặc ngắn gọn:*
    ```bash
    bun -h
    ```

*   **Hiển thị trợ giúp cho một lệnh cụ thể:**
    ```bash
    bun <command> --help
    ```
    *Ví dụ:* `bun install --help`, `bun build --help`

*   **Nâng cấp Bun lên phiên bản mới nhất:**
    ```bash
    bun upgrade
    ```

*   **Tạo gợi ý lệnh cho shell (bash, zsh, fish):**
    ```bash
    bun completions
    ```

## 💡 Lưu ý

*   Nhiều lệnh của Bun được thiết kế để tương thích với các lệnh tương ứng của `npm`, `yarn`, và `npx`, giúp việc chuyển đổi dễ dàng hơn.
*   Bun thường nhanh hơn đáng kể so với các công cụ Node.js truyền thống.
*   `bun run <script>` có thể được rút gọn thành `bun <script>` nếu không có file nào trùng tên trong thư mục hiện tại.
*   Sử dụng `bun.lockb` để đảm bảo cài đặt dependency nhất quán và nhanh chóng.

Hy vọng Cheatsheet này hữu ích cho bạn!

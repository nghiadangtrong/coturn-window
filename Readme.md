# This is the build for windows from https://github.com/coturn/coturn

## Infomation

**Build version: 4.6.3**

## Hướng dẫn build 

**Bước 0: Chuẩn Bị Môi Trường (Đảm bảo bạn đã cài đặt)**

1.  **Visual Studio Community:** (Ví dụ: 2022) Đã cài đặt với workload **"Desktop development with C++"**.
2.  **Git for Windows:** Đã cài đặt và có thể truy cập từ dòng lệnh.
3.  **Command Prompt (cmd) hoặc PowerShell:** Sẵn sàng để sử dụng.

---

**Bước 1: Cài đặt và Cấu hình `vcpkg`**

1.  **Chọn thư mục cài đặt:** Ví dụ, bạn sẽ cài `vcpkg` vào `C:\dev\vcpkg`.
2.  **Mở Command Prompt hoặc PowerShell với quyền Administrator:** Nhấn chuột phải vào biểu tượng và chọn "Run as administrator". Điều này cần thiết cho bước `integrate install`.
3.  **Tải mã nguồn vcpkg:**
    ```bash
    git clone https://github.com/microsoft/vcpkg.git C:\dev\vcpkg
    ```
4.  **Chạy script bootstrap:**
    ```bash
    cd C:\dev\vcpkg
    .\bootstrap-vcpkg.bat
    ```
5.  **Tích hợp vcpkg với Visual Studio:** (Vẫn trong cửa sổ Administrator)
    ```bash
    .\vcpkg integrate install
    ```
    Lệnh này giúp Visual Studio (MSBuild) tự động nhận diện các thư viện bạn cài qua vcpkg.

---

**Bước 2: Cài đặt Các Thư Viện Phụ Thuộc (Dependencies)**

1.  **Vẫn trong thư mục `vcpkg` (`C:\dev\vcpkg`) với quyền Administrator.**
2.  **Cài đặt các thư viện cần thiết:** Coturn cần ít nhất `openssl` và `libevent`. Bạn cũng có thể cài thêm các thư viện CSDL nếu cần. Chúng ta sẽ cài cho kiến trúc 64-bit (`x64-windows`).
    * **Cài đặt bắt buộc:**
        ```bash
        .\vcpkg install openssl:x64-windows libevent:x64-windows
        ```
    * **Cài đặt tùy chọn (nếu bạn cần hỗ trợ CSDL tương ứng):**
        ```bash
        .\vcpkg install sqlite3:x64-windows
        .\vcpkg install hiredis:x64-windows       # Hỗ trợ Redis
        .\vcpkg install libpq:x64-windows         # Hỗ trợ PostgreSQL
        .\vcpkg install mariadb-connector-c:x64-windows # Hỗ trợ MySQL/MariaDB
        ```
    * **Tìm kiếm thư viên tương ứng**
        ```bash
        .\vcpkg search redis
        .\vcpkg search mariadb
        ```
    * **Lưu ý:** Quá trình cài đặt các thư viện này có thể mất khá nhiều thời gian tùy thuộc vào tốc độ mạng và máy tính của bạn.

---

**Bước 3: Tải Mã Nguồn Coturn**

1.  **Chọn thư mục chứa mã nguồn dự án:** Ví dụ: `C:\dev\projects`.
2.  **Mở một cửa sổ Command Prompt hoặc PowerShell mới** (không cần quyền Administrator).
3.  **Di chuyển đến thư mục dự án:**
    ```bash
    cd C:\dev\projects
    ```
4.  **Tải mã nguồn Coturn từ GitHub:**
    ```bash
    git clone https://github.com/coturn/coturn.git
    ```
5.  **Đi vào thư mục mã nguồn vừa tải về:**
    ```bash
    cd coturn
    ```

---

**Bước 4: Cấu hình Build bằng CMake**

1.  **Tạo thư mục build riêng biệt:** (Để giữ mã nguồn sạch sẽ)
    ```bash
    mkdir build
    ```
2.  **Đi vào thư mục build:**
    ```bash
    cd build
    ```
3.  **Chạy CMake để tạo project Visual Studio:**
    * Thay thế `"Visual Studio 17 2022"` nếu bạn dùng phiên bản Visual Studio khác (ví dụ: `"Visual Studio 16 2019"`).
    * Đảm bảo đường dẫn đến `vcpkg.cmake` là chính xác (nếu bạn cài vcpkg ở thư mục khác `C:\dev\vcpkg`).
    ```bash
    cmake .. -G "Visual Studio 17 2022" -A x64 -DCMAKE_TOOLCHAIN_FILE=C:/dev/vcpkg/scripts/buildsystems/vcpkg.cmake
    ```
    * **Giải thích lệnh:**
        * `cmake ..`: Gọi CMake để cấu hình dự án từ thư mục cha (`..`).
        * `-G "Visual Studio 17 2022"`: Chỉ định tạo project cho Visual Studio 2022.
        * `-A x64`: Chỉ định kiến trúc là 64-bit.
        * `-DCMAKE_TOOLCHAIN_FILE=...`: **Rất quan trọng**, chỉ cho CMake biết cách tìm các thư viện đã cài bằng vcpkg.
    * *(Tùy chọn)* Nếu bạn đã cài thư viện CSDL và muốn bật hỗ trợ đó, bạn có thể thêm cờ `-DENABLE_<DB>=ON`, ví dụ: `-DENABLE_SQLITE=ON`. Xem `CMakeLists.txt` gốc của Coturn để biết các tùy chọn.

4.  Nếu lệnh CMake chạy thành công, nó sẽ tạo ra các tệp project Visual Studio (như `coturn.sln`) trong thư mục `build`.

---

**Bước 5: Biên dịch Dự án (Compile)**

Bạn có thể chọn một trong hai cách sau:

* **Cách 1: Dùng dòng lệnh (Khuyến nghị):**
    * **Vẫn ở trong thư mục `build`**.
    * Chạy lệnh build:
        ```bash
        cmake --build . --config Release
        ```
    * `--config Release`: Build phiên bản tối ưu hóa (optimized). Dùng `--config Debug` nếu bạn cần gỡ lỗi.

* **Cách 2: Dùng Visual Studio IDE:**
    * Mở tệp `coturn.sln` (nằm trong thư mục `build`) bằng cáchดับเบิลคลิก vào nó.
    * Trong Visual Studio, chọn cấu hình là `Release` và nền tảng là `x64` (thường ở thanh công cụ phía trên).
    * Trong cửa sổ "Solution Explorer" (thường ở bên phải), tìm đến `ALL_BUILD`, nhấn chuột phải vào đó và chọn `Build`.

* **Lưu ý:** Quá trình biên dịch có thể mất vài phút tùy thuộc vào cấu hình máy tính.

---

**Bước 6: Tìm Tệp Thực Thi (.exe)**

* Nếu quá trình biên dịch thành công không có lỗi, các tệp thực thi (`.exe`) sẽ được tạo ra.
* Bạn có thể tìm thấy chúng trong thư mục con của `build`, thường là: `C:\dev\projects\coturn\build\bin\Release\` (hoặc `...\bin\Debug\` nếu bạn build ở chế độ Debug).
* Tệp chính bạn cần là `turnserver.exe`. Ngoài ra còn có các tiện ích khác như `turnutils_stunclient.exe`, `turnutils_uclient.exe`,...

---

**Bước 7: Bước Tiếp Theo (Cấu hình và Chạy)**

1.  Để chạy `turnserver.exe`, bạn cần một tệp cấu hình, thường được đặt tên là `turnserver.conf`.
2.  Bạn có thể tìm thấy một tệp cấu hình mẫu trong mã nguồn đã tải về tại: `C:\dev\projects\coturn\examples\etc\turnserver.conf`.
3.  Hãy sao chép tệp mẫu này ra một vị trí khác, đổi tên thành `turnserver.conf` và chỉnh sửa các thông số bên trong cho phù hợp với nhu cầu của bạn.
4.  Sau đó, bạn có thể chạy server từ dòng lệnh, ví dụ: `.\turnserver.exe -c đường_dẫn_đến_tệp\turnserver.conf`.

<p align="center">
  <img src="https://i.pinimg.com/originals/fd/91/b1/fd91b1715061efc79dbb6678aea0f9b9.gif" width="220" alt="Ngọc Rồng Online">
</p>

<h1 align="center">dragonboy-db-backups</h1>

<p align="center">
  <em>Automated Database Backups · MySQL · PostgreSQL · MongoDB · Redis · 3-Layer Storage</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Bash-Script-4EAA25?style=flat&logo=gnubash&logoColor=white" alt="Bash"/>
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=flat&logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/MySQL-dump-4479A1?style=flat&logo=mysql&logoColor=white" alt="MySQL"/>
  <img src="https://img.shields.io/badge/PostgreSQL-dump-4169E1?style=flat&logo=postgresql&logoColor=white" alt="PostgreSQL"/>
  <img src="https://img.shields.io/badge/MongoDB-archive-47A248?style=flat&logo=mongodb&logoColor=white" alt="MongoDB"/>
  <img src="https://img.shields.io/badge/Redis-RDB-DC382D?style=flat&logo=redis&logoColor=white" alt="Redis"/>
  <img src="https://img.shields.io/badge/rclone-Google_Drive-3970E4?style=flat&logo=googledrive&logoColor=white" alt="rclone"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/cron-daily_4_AM-FF6B35?style=flat&logo=clockify&logoColor=white" alt="Schedule"/>
  <img src="https://img.shields.io/badge/retention-7_days-yellow?style=flat" alt="Retention"/>
  <img src="https://img.shields.io/badge/repo-private-lightgrey?style=flat&logo=github&logoColor=white" alt="Private"/>
</p>

<p align="center">
  <a href="https://github.com/DANG-PH/dragonboy-nginx-service">
    <img src="https://img.shields.io/badge/⚙_Mã_nguồn_script-dragonboy--nginx--service-181717?style=for-the-badge&logo=github&logoColor=white" alt="Source scripts"/>
  </a>
  &nbsp;
  <a href="https://ngocrongdark.com">
    <img src="https://img.shields.io/badge/▶_CHƠI_NGAY-ngocrongdark.com-FF6B35?style=for-the-badge&logoColor=white" alt="Play Now"/>
  </a>
</p>

<p align="center">
  Kho lưu trữ <strong>backup database</strong> cho dự án <strong>Ngọc Rồng Online</strong> – tựa game MMORPG<br>
  lấy cảm hứng từ bộ truyện <strong>Dragon Ball (7 Viên Ngọc Rồng)</strong> của tác giả Akira Toriyama.<br>
  Backup chạy <strong>tự động hằng ngày lúc 4h sáng</strong>, lưu đồng thời ở 3 nơi để an toàn nhiều lớp.
</p>

---

## Tổng quan

Repo này chứa các bản dump database được tạo **tự động qua cron** trên VPS. Mỗi đêm, script `backup.sh` dump cả 4 database đang chạy trong Docker, nén bằng `gzip`, rồi phân phối tới ba đích lưu trữ độc lập:

| Lớp lưu trữ | Vai trò | Tốc độ khôi phục |
|---|---|---|
| **Local (VPS)** | Bản gần nhất, restore nhanh nhất | ⚡ Nhanh nhất |
| **GitHub (repo này)** | Lưu lịch sử theo commit, versioned | 🌐 Cần mạng |
| **Google Drive** | Phòng khi VPS hỏng hoàn toàn | ☁️ Off-site |

Việc tách làm 3 lớp tuân theo nguyên tắc backup **3-2-1**: nhiều bản sao, trên các phương tiện khác nhau, và ít nhất một bản nằm ngoài máy chủ (off-site). Khi VPS gặp sự cố, vẫn còn GitHub và Drive để khôi phục; khi mạng có vấn đề, bản local vẫn restore được ngay.

Quá trình dump được thiết kế để **không gây downtime**: MySQL dùng `--single-transaction`, Redis dùng `BGSAVE` (lưu nền, không chặn), nên các container vẫn phục vụ người chơi bình thường trong lúc backup.

Mỗi lần backup là một commit `backup: <ngày_giờ>`. Bản cũ hơn **7 ngày** sẽ tự động bị dọn ở cả ba nơi (commit `cleanup: remove backups older than 7 days`).

> **Lưu ý bảo mật:** Repo này **private** vì chứa dữ liệu thật (tài khoản, dữ liệu game...). Không công khai, không chia sẻ deploy key / SSH key dùng để push.

---

## Hai repo, hai vai trò

Hệ thống tách làm hai repo riêng:

| Repo | Vai trò | Nội dung |
|---|---|---|
| **dragonboy-db-backups** *(repo này)* | Nơi **lưu trữ** dữ liệu | Chỉ chứa thư mục `data/` với các file backup được push lên mỗi đêm |
| **[DANG-PH/dragonboy-nginx-service](https://github.com/DANG-PH/dragonboy-nginx-service)** | Nơi chứa **mã nguồn** | Toàn bộ script `.sh`, `docker-compose.yml`, cấu hình nginx... |

Tức là **mọi lệnh `*.sh` bên dưới đều nằm và chạy ở repo `dragonboy-nginx-service`**, không phải repo này. Repo này thuần tuý là đích để script đẩy backup lên (qua biến `GITHUB_BACKUP_DIR`).

Muốn xem chi tiết 4 script điều khiển toàn bộ vòng đời backup, đọc trực tiếp trên repo script:

| Script | Chức năng | Link |
|---|---|---|
| `bootstrap.sh` | Setup VPS mới từ A–Z (14 bước) | [xem file](https://github.com/DANG-PH/dragonboy-nginx-service/blob/main/backup/scripts/bootstrap.sh) |
| `setup.sh` | Cài rclone + đăng ký cron backup | [xem file](https://github.com/DANG-PH/dragonboy-nginx-service/blob/main/backup/scripts/setup.sh) |
| `backup.sh` | Dump 4 DB → local → Drive → GitHub | [xem file](https://github.com/DANG-PH/dragonboy-nginx-service/blob/main/backup/scripts/backup.sh) |
| `restore.sh` | Khôi phục DB từ local / GitHub / Drive | [xem file](https://github.com/DANG-PH/dragonboy-nginx-service/blob/main/backup/scripts/restore.sh) |

> Đường dẫn `backup/scripts/...` ở trên là theo cấu trúc thư mục trong repo script. Nếu bạn để script ở vị trí khác thì chỉnh lại link cho đúng.

---

## Định dạng file backup

Mỗi loại database có một định dạng riêng. Tên file theo mẫu `<db>_<YYYY-MM-DD>_<HHMM>.<ext>`:

| File | Database | Cách tạo | Nội dung sau khi giải nén |
|---|---|---|---|
| `mysql_*.sql.gz` | MySQL | `mysqldump --all-databases` | File `.sql` text (lệnh SQL) |
| `pg_*.sql.gz` | PostgreSQL | `pg_dumpall` | File `.sql` text (lệnh SQL) |
| `mongo_*.archive.gz` | MongoDB | `mongodump --archive --gzip` | Định dạng nhị phân riêng của Mongo |
| `redis_*.rdb.gz` | Redis | `BGSAVE` + copy `dump.rdb` | File RDB nhị phân |

> Vì chỉ `.sql.gz` giải nén ra là SQL text thật nên repo gán nhãn ngôn ngữ **SQL** (xem `.gitattributes`). Mongo archive và Redis RDB là binary, không thuộc ngôn ngữ nào nên cố ý không gán.

Muốn **xem nhanh nội dung** một bản dump SQL mà không cần restore vào database:

```bash
# Xem toàn bộ
zcat mysql_2026-05-22_0400.sql.gz | less

# Tìm một bảng cụ thể
zcat pg_2026-05-22_0400.sql.gz | grep -i "CREATE TABLE players"
```

(File `.archive.gz` của Mongo và `.rdb.gz` của Redis là nhị phân nên `zcat` chỉ ra ký tự rác — chúng chỉ đọc được qua `mongorestore` / Redis.)

---

## Kiến trúc backup

```
                    ┌─────────────────────────┐
                    │   cron (4:00 hằng ngày)  │
                    └────────────┬────────────┘
                                 │
                         ┌───────▼────────┐
                         │   backup.sh    │
                         └───────┬────────┘
                                 │  docker exec → dump 4 DB → gzip
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
        ┌──────────┐      ┌────────────┐     ┌─────────────┐
        │  MySQL   │      │ PostgreSQL │     │   MongoDB   │   + Redis
        └──────────┘      └────────────┘     └─────────────┘
                                 │
                    backup/data/*_<DATE>.*  (Local trên VPS)
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                                      ▼
     ┌─────────────────┐                  ┌─────────────────────┐
     │  GitHub (push)  │                  │ Google Drive (rclone)│
     │   repo này      │                  │      off-site        │
     └─────────────────┘                  └─────────────────────┘
```

---

## Cài đặt VPS mới

> Mọi lệnh dưới đây chạy trong repo script [**dragonboy-nginx-service**](https://github.com/DANG-PH/dragonboy-nginx-service), **không phải repo data này**.

**Bước 1 — Clone repo script về VPS** (qua SSH, vì repo private):

```bash
git clone git@github.com:DANG-PH/dragonboy-nginx-service.git
cd dragonboy-nginx-service
```

**Bước 2 — Chạy bootstrap** với quyền root. Toàn bộ quá trình setup được tự động hoá trong một lệnh:

```bash
sudo ./backup/scripts/bootstrap.sh
```

Script sẽ tự làm 14 bước (mất ~15–20 phút): cài tool cơ bản, set timezone `Asia/Ho_Chi_Minh`, tạo swap 2GB, cài Docker, cấu hình UFW firewall, tạo `.env` + `htpasswd`, lấy SSL qua certbot, bật `docker compose`, verify nginx + DB container, cài rclone + cấu hình Google Drive, logrotate, và đăng ký cron backup. Chính `bootstrap.sh` cũng tự `git clone` repo data này về (qua biến `GITHUB_BACKUP_DIR`) để có nơi push backup.

> **Trước khi chạy:** đảm bảo các domain đã trỏ DNS về IP VPS, và bạn có sẵn máy local có trình duyệt để hoàn tất bước OAuth của rclone (Google Drive).

Nếu chỉ cần cài lại riêng phần backup (rclone + cron) mà không setup lại toàn bộ VPS:

```bash
cd dragonboy-nginx-service
./backup/scripts/setup.sh
```

---

## Cấu hình `.env`

File `.env` nằm ở **gốc repo script** (`dragonboy-nginx-service`), chứa mọi mật khẩu và tham số (chmod `600`). Copy từ mẫu rồi điền:

```bash
cd dragonboy-nginx-service
cp .env.example .env
nano .env
```

| Biến | Ý nghĩa | Bắt buộc |
|---|---|:---:|
| `MYSQL_CONTAINER` | Tên container MySQL (mặc định `mysql-nro`) | |
| `MYSQL_PASS` | Mật khẩu root MySQL | ✅ |
| `MONGO_CONTAINER` | Tên container MongoDB (mặc định `mongo`) | |
| `MONGO_USER` / `MONGO_PASS` | Tài khoản MongoDB (auth db `admin`) | ✅ |
| `POSTGRES_CONTAINER` | Tên container Postgres (mặc định `postgres`) | |
| `PG_USER` | User PostgreSQL dùng cho `pg_dumpall` | |
| `REDIS_CONTAINER` | Tên container Redis (mặc định `redis`) | |
| `RCLONE_REMOTE` | Đích Google Drive, dạng `remote:folder` | ✅ |
| `GITHUB_BACKUP_DIR` | Đường dẫn local tới thư mục `data/` của repo này | |
| `HTPASSWD_USER` / `HTPASSWD_PASS` | Tài khoản basic-auth cho nginx | ✅ |
| `LOCAL_RETENTION_DAYS` | Số ngày giữ bản local + GitHub | |
| `DRIVE_RETENTION_DAYS` | Số ngày giữ bản trên Google Drive | |
| `CRON_HOUR` / `CRON_MINUTE` | Giờ/phút chạy cron (mặc định `4` / `0`) | |

> Mật khẩu có ký tự đặc biệt (`@`, `$`, `!`) phải bọc trong nháy đơn, ví dụ: `MYSQL_PASS='Phamhaidang112@'`

---

## Sao lưu thủ công

Chạy backup ngay lập tức (ngoài lịch cron), từ trong repo script:

```bash
cd dragonboy-nginx-service
./backup/scripts/backup.sh
```

Một lượt backup gồm: dump 4 DB → lưu local → upload Google Drive → push GitHub → dọn file cũ hơn `LOCAL_RETENTION_DAYS` ngày.

Xem cron đã đăng ký và theo dõi log:

```bash
crontab -l
tail -f backup/backup.log
```

---

## Khôi phục (Restore)

> ⚠️ Restore sẽ **ghi đè** dữ liệu hiện tại. Script sẽ hỏi xác nhận (gõ `yes`) trước khi thực hiện.

Cú pháp (chạy trong repo script `dragonboy-nginx-service`):

```bash
cd dragonboy-nginx-service
./backup/scripts/restore.sh <mysql|mongo|pg|redis> <tên_file> [--from-github|--from-drive]
```

Nếu không truyền nguồn, script tìm file ngay trong `backup/data/` (bản local). Truyền `--from-github` hoặc `--from-drive` để kéo file về từ nguồn tương ứng trước khi restore.

Ví dụ:

```bash
# Restore MySQL từ bản local
./backup/scripts/restore.sh mysql mysql_2026-05-22_0400.sql.gz

# Kéo từ GitHub về rồi restore
./backup/scripts/restore.sh pg pg_2026-05-22_0400.sql.gz --from-github

# Kéo từ Google Drive về rồi restore
./backup/scripts/restore.sh mongo mongo_2026-05-22_0400.archive.gz --from-drive
```

Cơ chế restore từng loại:

| DB | Cách khôi phục |
|---|---|
| **MySQL** | `gunzip` → pipe vào `mysql` trong container |
| **PostgreSQL** | `gunzip` → pipe vào `psql` trong container |
| **MongoDB** | `mongorestore --archive --gzip --drop` |
| **Redis** | Stop container → thay `dump.rdb` → start lại |

Chạy không tham số để xem danh sách file local đang có sẵn:

```bash
./backup/scripts/restore.sh
```

---

## Lịch & vòng đời dữ liệu

- **Backup:** chạy lúc `CRON_HOUR:CRON_MINUTE` (mặc định **4:00**) mỗi ngày.
- **Auto-renew SSL:** certbot renew lúc 3:00 mỗi ngày (cấu hình bởi `bootstrap.sh`).
- **Dọn dẹp:** file cũ hơn `LOCAL_RETENTION_DAYS` (mặc định 7) bị xoá ở local + GitHub; bản trên Drive xoá theo `DRIVE_RETENTION_DAYS`.
- **Log:** `backup/backup.log`, xoay vòng hằng tuần qua logrotate, giữ 4 tuần.

---

## Xử lý sự cố thường gặp

| Triệu chứng | Nguyên nhân thường gặp | Cách kiểm tra |
|---|---|---|
| Cron không chạy đúng giờ | Daemon cron còn dùng timezone cũ | `timedatectl` xem timezone; bootstrap đã restart cron sau khi đổi sang `Asia/Ho_Chi_Minh` |
| Push GitHub báo lỗi (rejected) | Lịch sử local lệch remote (vì đã sửa file trên web) | `cd` vào repo data rồi `git pull origin main`, sau đó chạy lại |
| `Permission denied (publickey)` | SSH key chưa nhận hoặc sai user | `ssh -T git@github.com`; kiểm tra `~/.ssh/config` trỏ đúng key |
| Upload Drive fail | Token rclone hết hạn | `rclone lsd <remote>:`; nếu lỗi thì `rclone config reconnect <remote>:` |
| Dump rỗng / 0 byte | Sai mật khẩu DB hoặc tên container | Xem `backup/backup.log`; đối chiếu `*_CONTAINER`, `*_PASS` trong `.env` |
| File trên 100MB không push được | Giới hạn file đơn lẻ của GitHub | Cân nhắc chỉ dựa vào Drive cho DB lớn; xem mục dưới |

Khi cron "im lặng" không rõ lý do, nơi đầu tiên cần xem luôn là log:

```bash
tail -n 50 backup/backup.log
```

> **Về lâu dài:** GitHub giới hạn 100MB/file và repo phình theo lịch sử commit (kể cả sau khi cleanup, bản cũ vẫn nằm trong git history). Nếu database lớn dần, nên ưu tiên Google Drive hoặc object storage (Backblaze B2, Cloudflare R2...) cho các bản dump nhị phân lớn, và để GitHub giữ vai trò versioning cho phần nhẹ.

---

<p align="center">
  <sub>Dự án cá nhân · Ngọc Rồng Online · <a href="https://ngocrongdark.com">ngocrongdark.com</a></sub>
</p>

# Căn phòng giấu tim — hướng dẫn deploy lên Vercel

Game pixel "tìm tim giấu trong phòng", gói gọn trong **một file HTML tĩnh duy nhất**
(`can-phong-bi-mat.html`, ~125KB): HTML + CSS + JS + ảnh đều nằm trong file đó.

Không có bước build, không `package.json`, không dependency. Vercel chỉ việc phục vụ file tĩnh.

---

## 1. File nào cần deploy

| File | Vai trò | Có cần đẩy lên không |
|---|---|---|
| `can-phong-bi-mat.html` | **Chính là game.** Chứa luôn ảnh cuối game (base64) | ✅ Có |
| `Gemini_Generated_Image_*.png` | Ảnh gốc 2.3MB, đã nhúng sẵn vào game nên không dùng lúc chạy | ❌ Không |
| `pixel-*.html`, `pixel-data.js` | Các bản nháp lúc thử hiệu ứng pixel | ❌ Không |

Tạo file `.vercelignore` để Vercel bỏ qua những thứ không cần (và cũng để ảnh gốc
không bị công khai trên mạng):

```
Gemini_Generated_Image_*.png
pixel-*.html
pixel-data.js
```

## 2. Đặt game làm trang chủ

Vercel mở `/` là tìm `index.html`. Hiện chưa có file đó, nên chọn **một** trong hai cách:

**Cách A — đổi tên (đơn giản nhất):**

```bash
cp can-phong-bi-mat.html index.html
```

Từ đó mỗi lần sửa game thì copy lại trước khi deploy.

**Cách B — giữ nguyên tên file, thêm `vercel.json`:**

```json
{
  "cleanUrls": true,
  "rewrites": [{ "source": "/", "destination": "/can-phong-bi-mat.html" }]
}
```

Vào `/` là ra game, đồng thời `/can-phong-bi-mat` (không cần đuôi `.html`) cũng chạy.

## 3. Deploy bằng Vercel CLI

Cần Node 18 trở lên (máy đang có v18.20.8 — hợp lệ).

```bash
# lần đầu: cài CLI và đăng nhập
npm i -g vercel
vercel login

# đứng trong thư mục này
cd ~/Desktop/code/ai/pixel-HH

# bản thử (preview) — CLI sẽ hỏi vài câu, cứ Enter theo mặc định
vercel

# bản chính thức
vercel --prod
```

Những câu CLI hỏi và câu trả lời:

- *Set up and deploy?* → `y`
- *Which scope?* → tài khoản cá nhân
- *Link to existing project?* → `n` (lần đầu)
- *What's your project's name?* → ví dụ `can-phong-giau-tim`
- *In which directory is your code located?* → `./`
- *Want to modify these settings?* → `n` (không có build command, không có output directory)

Xong sẽ có link dạng `https://can-phong-giau-tim.vercel.app`.

## 4. Deploy qua GitHub (nếu muốn tự động cập nhật)

```bash
cd ~/Desktop/code/ai/pixel-HH
git init
git add .
git commit -m "Căn phòng giấu tim"
gh repo create can-phong-giau-tim --private --source=. --push
```

Rồi vào [vercel.com/new](https://vercel.com/new) → **Import** repo vừa tạo:

- Framework Preset: **Other**
- Build Command: để trống
- Output Directory: để trống (hoặc `.`)
- Install Command: để trống

Sau đó mỗi lần `git push` là Vercel tự deploy lại.

## 5. Cập nhật sau này

```bash
# sửa can-phong-bi-mat.html xong:
cp can-phong-bi-mat.html index.html   # nếu dùng Cách A ở mục 2
vercel --prod
```

Hoặc `git push` nếu deploy qua GitHub.

## 6. Lưu ý riêng của game này

- **Ảnh cá nhân sẽ công khai.** Link `.vercel.app` ai có cũng mở được, và ảnh nằm
  ngay trong HTML. Nếu muốn giới hạn người xem: Vercel Dashboard → Project →
  Settings → **Deployment Protection** → bật *Vercel Authentication* (chỉ người được
  mời mới xem được), hoặc đặt tên project khó đoán rồi chỉ gửi link cho đúng người.
- **Không cần Vercel vẫn chạy được.** File là HTML tĩnh tự chứa, mở trực tiếp bằng
  trình duyệt hoặc AirDrop sang iPhone là chơi được luôn, kể cả khi không có mạng.
- **Đã tối ưu sẵn cho hai khổ máy:** MacBook 14" (khung cảnh 872×654) và iPhone 14
  Plus cả dọc (400×400) lẫn ngang (506×316) — không phải cuộn trang ở khổ nào.
- **Sửa nội dung** ở đầu thẻ `<script>`, phần `const CONFIG`: câu hỏi mở quà, lời
  nhắn cuối, và ảnh (chuỗi base64). Muốn đổi ảnh thì thay chuỗi `photo` — code tự
  nhận độ phân giải mới, không phải sửa gì thêm.

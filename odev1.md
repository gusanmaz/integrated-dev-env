# Ödev 1 — Flask Hello World (Tema + Port + Docker Hub)

**Ders:** AYG110 Tümleşik Geliştirme Ortamları  
**Konu:** [Linux Komut Satırı Eğitimi — Bölüm 29–30](https://gusanmaz.github.io/bash-tutorial/#home)  
**Teslim yeri:** [Mergen](https://mergen.anadolu.edu.tr/) — tek satır Docker Hub URL'si

**Ödev metnine erişim:** Bu içerik Mergen'de de yayınlanır. Mergen bazen Markdown dosyalarını (kod blokları, tablolar vb.) düzgün göstermeyebilir. Aynı metne GitHub üzerinden de ulaşabilirsiniz:

https://github.com/gusanmaz/integrated-dev-env/blob/main/odev1.md

---

## Ödevin Amacı

1. **Flask** ile basit bir web uygulaması yazmak  
2. Uygulamayı **kendi portunuzda** (`app.py` + `Dockerfile`) dinletmek  
3. **`THEME` ortam değişkeni** ile açık/koyu tema  
4. Uygulamayı **Docker imajı** yapıp **Docker Hub**'a yüklemek  
5. Mergen'e yalnızca imaj linkini yazmak  

Değerlendirme: Öğrenci numaranızdan port hesaplanır (`8000 + son 2 hane`), imaj çekilir, `-p PORT:PORT -e THEME=...` ile konteyner çalıştırılır.

---

## Ne Yapacaksınız? (Özet)

| Gereksinim | Açıklama |
|------------|----------|
| Metin | Sayfada **yalnızca:** `Merhaba AYG110 Tümleşik Geliştirme Ortamları` |
| Port | `8000 + son 2 hane` → `app.py` ve `Dockerfile EXPOSE` (örn. **8045**) |
| `THEME` | `-e THEME=light` → açık, `-e THEME=dark` → koyu tema |
| Teslim | Mergen'e Docker Hub URL'si |

**PORT değeriniz:**

```
PORT = 8000 + (öğrenci numaranızın son 2 hanesi)
```

Örnek: numara `...2345` → port **8045** — bunu `app.py` ve `Dockerfile`'a yazarsınız.

---

## Adım Adım Nasıl Yapılır?

### 1) Proje klasörü oluşturun

```bash
mkdir odev-01-flask
cd odev-01-flask
```

### 2) Dosyaları oluşturun

Aşağıdaki **dört dosyayı** aynen oluşturun. Kodları kopyalayıp yapıştırabilirsiniz; sonra isteğe bağlı küçük değişiklikler yapabilirsiniz.

---

#### `requirements.txt`

```
flask==3.0.0
```

---

#### `app.py` — ana uygulama

Bu dosya:

1. Sayfada zorunlu metni gösterir  
2. `THEME` env'sine göre arka plan / yazı rengini değiştirir  
3. **Sabit `PORT`** değerinde dinler (öğrenci numaranıza göre)

```python
import os
from flask import Flask

app = Flask(__name__)

MESSAGE = "Merhaba AYG110 Tümleşik Geliştirme Ortamları"

PORT = 8045   # ← kendi portunuz: 8000 + son 2 hane

THEMES = {
    "light": ("#f5f5f5", "#222222"),
    "dark": ("#1a1a1a", "#eeeeee"),
}


def get_theme():
    value = os.getenv("THEME", "light").lower()
    return "dark" if value == "dark" else "light"


@app.route("/")
def index():
    theme = get_theme()
    bg, fg = THEMES[theme]
    return f"""<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="utf-8">
  <title>AYG110</title>
</head>
<body style="margin:2rem;font-family:sans-serif;background:{bg};color:{fg}">
  <p>{MESSAGE}</p>
</body>
</html>"""


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=PORT)
```

---

#### `Dockerfile`

`EXPOSE` değeri `app.py`'deki `PORT` ile **aynı** olmalı (örnek: **8045**).

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 8045

CMD ["python", "app.py"]
```

---

#### `.dockerignore`

```
__pycache__
*.pyc
.git
.env
.venv
```

---

### 3) Yerelde imaj oluşturun (Docker olmadan test — isteğe bağlı)

```bash
python3 -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Uygulamayı bir terminalde çalıştırın; **başka bir terminalde** `curl` ile test edin.

**Case 1 — `THEME=light` (açık tema):**

```bash
THEME=light python app.py
```

```bash
curl -s http://localhost:8045
```

Beklenen:

```html
<body style="margin:2rem;font-family:sans-serif;background:#f5f5f5;color:#222222">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

**Case 2 — `THEME=dark` (koyu tema):**

Uygulamayı durdurup (`Ctrl+C`) yeniden başlatın:

```bash
THEME=dark python app.py
```

```bash
curl -s http://localhost:8045
```

Beklenen — **aynı metin**, farklı renkler:

```html
<body style="margin:2rem;font-family:sans-serif;background:#1a1a1a;color:#eeeeee">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

**Case 3 — `THEME` verilmedi (varsayılan = light):**

```bash
python app.py
```

```bash
curl -s http://localhost:8045
```

Beklenen: Case 1 ile aynı (varsayılan açık tema):

```html
<body style="margin:2rem;font-family:sans-serif;background:#f5f5f5;color:#222222">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

### 4) Docker imajı build edin

**İmaj adında ne değişir, ne değişmez?**

| | Öğrenci A | Öğrenci B |
|---|-----------|-----------|
| Öğrenci no | `2024012345` | `2024012303` |
| Port (`app.py` + `EXPOSE`) | **8045** | **8003** |
| Docker Hub kullanıcı adı | `ahmetk` | `zeynepy` |
| Repo adı (herkeste aynı) | `ayg110-odev1` | `ayg110-odev1` |
| Tam imaj adı | `ahmetk/ayg110-odev1:1.0` | `zeynepy/ayg110-odev1:1.0` |

Yani: **port** numaranıza göre kodda değişir; **repo adı** (`ayg110-odev1`) herkeste aynı kalır; **kullanıcı adı** sizin Docker Hub hesabınızdır.

**Öğrenci A** (`ahmetk`, numara `...2345`, port `8045`):

```bash
docker build -t ayg110-odev1:1.0 .
docker tag ayg110-odev1:1.0 ahmetk/ayg110-odev1:1.0
```

**Öğrenci B** (`zeynepy`, numara `...2303`, port `8003` — `app.py` ve `EXPOSE` buna göre):

```bash
docker build -t ayg110-odev1:1.0 .
docker tag ayg110-odev1:1.0 zeynepy/ayg110-odev1:1.0
```

### 5) Kendi makinenizde konteyner testi

#### **Öğrenci A** (`ahmetk`, port **8045**):

```bash
# Light
docker run -d --rm --name test \
  -p 8045:8045 \
  -e THEME=light \
  ahmetk/ayg110-odev1:1.0

curl -s http://localhost:8045
```

Beklenen: HTML döner; içinde metin ve **açık tema** renkleri:

```html
<body style="...background:#f5f5f5;color:#222222...">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

```bash
docker rm -f test

# Dark
docker run -d --rm --name test \
  -p 8045:8045 \
  -e THEME=dark \
  ahmetk/ayg110-odev1:1.0

curl -s http://localhost:8045
```

Beklenen: **aynı metin**, **koyu tema** renkleri:

```html
<body style="...background:#1a1a1a;color:#eeeeee...">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

#### **Öğrenci B** (`zeynepy`, port **8003**):

```bash
# Light
docker run -d --rm --name test \
  -p 8003:8003 \
  -e THEME=light \
  zeynepy/ayg110-odev1:1.0

curl -s http://localhost:8003
```

Beklenen: HTML döner; içinde metin ve **açık tema** renkleri:

```html
<body style="...background:#f5f5f5;color:#222222...">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

```bash
docker rm -f test

# Dark
docker run -d --rm --name test \
  -p 8003:8003 \
  -e THEME=dark \
  zeynepy/ayg110-odev1:1.0

curl -s http://localhost:8003
```

Beklenen: **aynı metin**, **koyu tema** renkleri:

```html
<body style="...background:#1a1a1a;color:#eeeeee...">
  <p>Merhaba AYG110 Tümleşik Geliştirme Ortamları</p>
</body>
```

Tarayıcıda da kontrol edin: light/dark arka plan rengi değişmeli, metin aynı kalmalı.

### 6) Docker Hub'a yükleyin

**Öğrenci A:**

```bash
docker login
docker push ahmetk/ayg110-odev1:1.0
```

Kontrol: https://hub.docker.com/r/ahmetk/ayg110-odev1

**Öğrenci B:**

```bash
docker login
docker push zeynepy/ayg110-odev1:1.0
```

Kontrol: https://hub.docker.com/r/zeynepy/ayg110-odev1

Her iki durumda da repo **Public** olmalı ve tag **`1.0`** görünmeli.

### 7) Mergen'e teslim edin

Ödev açıklaması Mergen'de okunaklı görünmüyorsa: [github.com/gusanmaz/integrated-dev-env/blob/main/odev1.md](https://github.com/gusanmaz/integrated-dev-env/blob/main/odev1.md)

1. [mergen.anadolu.edu.tr](https://mergen.anadolu.edu.tr) → giriş  
2. **AYG110** → **Ödevler** → **Ödev 1**  
3. **Ödev Gönder**  
4. Metin alanına **yalnızca** Docker Hub URL'sini yapıştırın:

```
https://hub.docker.com/r/ahmetk/ayg110-odev1
```

5. **Gönder**

GitHub, zip veya ekran görüntüsü **göndermeyin**.

---

## Kurallar (Tekrar)

### `PORT`

| Son 2 hane | Port |
|------------|------|
| `45` | **8045** |
| `03` | **8003** |
| `00` | **8000** |

- `app.py` → `PORT = 8000 + son 2 hane`  
- `Dockerfile` → `EXPOSE` aynı port  
- Run anında **yalnızca** `-e THEME=...` verilir; `-e PORT=...` **yok**

### `THEME`

| Değer | Görünüm |
|-------|---------|
| yok / `light` | Açık arka plan |
| `dark` | Koyu arka plan |

- Tema **ortam değişkeni** ile; `?theme=dark` veya JavaScript **kabul edilmez**.

### Metin

Sayfada **sadece:**

```
Merhaba AYG110 Tümleşik Geliştirme Ortamları
```

---

## Değerlendirme (Öğretmen)

Mergen'deki öğrenci numarasından port hesaplanır: `PORT = 8000 + son 2 hane`. Örnek: `...2345` → **8045**.

```bash
docker pull ahmetk/ayg110-odev1:1.0

docker run -d --name ayg110-test \
  -p 8045:8045 -e THEME=light \
  ahmetk/ayg110-odev1:1.0
curl -s http://localhost:8045

docker stop ayg110-test && docker rm ayg110-test

docker run -d --name ayg110-test \
  -p 8045:8045 -e THEME=dark \
  ahmetk/ayg110-odev1:1.0
curl -s http://localhost:8045

docker stop ayg110-test && docker rm ayg110-test
```

| Kriter | Puan |
|--------|------|
| URL geçerli, `docker pull :1.0` OK | 25 |
| Zorunlu metin doğru | 25 |
| Doğru portta çalışıyor (numaraya uygun) | 25 |
| `THEME=light` / `dark` çalışıyor | 25 |

---

## İlgili Eğitim Bölümleri

- **Bölüm 29:** `docker run`, `-p`, `-e`  
- **Bölüm 30:** Flask + Dockerfile + Docker Hub  
- **Bölüm 32:** Ortam değişkeniyle davranış değiştirme  

---

*Son güncelleme: Mayıs 2026*


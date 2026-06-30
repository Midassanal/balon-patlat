# Balon Patlat — Dünya Sıralaması Kurulumu (Supabase)

Oyun şu an çalışıyor. Sıralama, Supabase anahtarlarını girene kadar **bu cihazda yerel** olarak tutulur.
Aşağıdaki adımları yapınca, hangi dilde oynanırsa oynansın **tüm dünya tek bir global sıralamada** birikir.

Süre: ~5–10 dakika. Ücretsiz plan yeterli.

---

## 1. Supabase projesi aç
1. https://supabase.com adresine git, ücretsiz hesap aç.
2. **New project** → bir isim ver, bir veritabanı şifresi belirle, bölge seç → **Create**.
3. Proje hazırlanana kadar ~1 dakika bekle.

## 2. Tabloyu oluştur
Sol menüden **SQL Editor**'ü aç, aşağıdaki kodu yapıştırıp **Run** de:

```sql
create table if not exists public.scores (
  id bigint generated always as identity primary key,
  name text not null,
  score integer not null,
  level integer not null,
  lang text,
  created_at timestamptz default now()
);

-- Güvenlik (RLS): herkes okuyabilsin, herkes makul puan ekleyebilsin
alter table public.scores enable row level security;

create policy "read_all" on public.scores
  for select using (true);

create policy "insert_sane" on public.scores
  for insert with check (
    score >= 0 and score <= 100000000
    and level >= 1 and level <= 100000
    and char_length(name) <= 16
  );

-- Sıralama sorgusu hızlı olsun
create index if not exists scores_score_idx on public.scores (score desc);
```

## 3. Anahtarları al
Sol menüden **Project Settings → API**:
- **Project URL** (örn. `https://abcdefgh.supabase.co`)
- **Project API keys → `anon` `public`** anahtarı

> Not: `anon public` anahtarı tarayıcıda görünmesi için tasarlanmıştır, paylaşmak güvenlidir.
> Asıl gizli olan `service_role` anahtarını ASLA buraya koyma.

## 4. Anahtarları oyuna gir
`balon-patlat.html` dosyasını bir metin düzenleyiciyle aç. En üstte, `<script>` içinde şu iki satırı bul:

```js
const SUPABASE_URL = ""; // e.g. https://abcdefgh.supabase.co
const SUPABASE_KEY = ""; // the anon public key
```

Tırnakların içine kendi değerlerini yapıştır:

```js
const SUPABASE_URL = "https://abcdefgh.supabase.co";
const SUPABASE_KEY = "eyJhbGciOi....(uzun anon anahtar)....";
```

Kaydet. Dosyayı tarayıcıda aç — artık puanlar global tabloya yazılır ve **🏆 Sıralama** ekranı dünya genelini gösterir.

---

## 5. Oyunu internette yayınla (herkes oynayabilsin)
Tek dosya bilgisayarda açılınca sadece sende çalışır. Başkalarının oynaması için dosyayı bir web adresine koyman gerekir. En kolay ücretsiz yollar:

- **Netlify Drop:** https://app.netlify.com/drop → `balon-patlat.html` dosyasını sürükle-bırak, anında bir link verir. (İstersen dosyayı `index.html` olarak adlandır.)
- **Vercel** veya **GitHub Pages** de olur.

Yayınladıktan sonra linki İngilizce/İspanyolca/Çince/Türkçe konuşan herkese gönderebilirsin; hepsi aynı dünya sıralamasında yarışır.

---

## Notlar
- **Diller:** Oyun tarayıcı diline göre otomatik açılır (İngilizce, İspanyolca, Çince, Türkçe). Oyuncu başlangıç ekranındaki EN / ES / 中文 / TR düğmeleriyle de değiştirebilir. Sıralama ortaktır, sadece arayüz çevrilir.
- **Çevrimdışı / anahtar yoksa:** Oyun yine çalışır, sıralama o cihazda yerel tutulur ve listede "(çevrimdışı — yerel puanlar)" yazar.
- **İleride istersen:** İsim küfür filtresi, ülke bayrağı, haftalık sıralama, hile önleme gibi eklemeler yapılabilir. Söyle, kurarım.

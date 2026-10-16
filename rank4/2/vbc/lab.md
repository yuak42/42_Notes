---
title: "42 Exam Rank 04 — VBC: Sıfırdan Recursive Descent Parser"
tags: [42, exam-rank-04, c, parser, recursion, lab]
status: tamamlandı
---
---
title: "42 Exam Rank 04 — VBC: Sıfırdan Recursive Descent Parser"
tags: [42, exam-rank-04, c, parser, recursion, lab]
status: tamamlandı
---

# 42 Exam Rank 04 — VBC Lab

> [!abstract] Hedef
> Tek basamaklı rakamlar, `+`, `*` ve parantezlerden oluşan matematiksel ifadeleri hesaplayan bir C programını **sıfırdan** yazmak. Her görevde önce kendin kod yaz, sonra **Çözüm** başlığındaki C koduyla karşılaştır. Kod blokları Obsidian ve GitHub Markdown ile uyumlu, doğrudan kopyalanabilir biçimdedir.

## Problem ve kurallar

Program komut satırından **tek bir matematiksel ifade** alır. Rakamlar `0`–`9` arasındadır; çok basamaklı sayılar ve boşluklar desteklenmez. Toplama (`+`), çarpma (`*`) ve iç içe parantezler desteklenir. Çarpma toplamadan önceliklidir.

- Geçerli ifade: yalnızca sonucu ve satır sonunu yazdır, çıkış kodu `0`.
- Geçersiz karakter: `Unexpected token '%c'\n` yazdır, çıkış kodu `1`.
- Beklenmedik string sonu: `Unexpected end of input\n` yazdır, çıkış kodu `1`.
- Argüman sayısı yanlışsa `1` döndür.
- İzin verilen fonksiyonlar (paylaşılan soru metni): `malloc`, `calloc`, `realloc`, `free`, `printf`, `isdigit`, `write`. **`exit()` izinli değil.** Bu çözüm yalnızca `printf` ve `isdigit` kullanır.

> [!warning] Dosya düzeni
> Bu lab başlangıçta fonksiyonları ayrı dosyalarda ve `vbc.h` ile yazmayı öğretiyor. Son bölümde sınav için **tek dosyalık `vbc.c`** sürümü bulunuyor. Çözümler HTML açılır kutular yerine standart Markdown kod bloklarında tutulmuştur; böylece C sözdizimi renklendirmesi ve kopyalama düzgün çalışır. Sorunun `*.c *.h` dosyalarına izin vermesi ayrı dosya kullanmayı da mümkün kılar.

## Öğrenme haritası

```text
main
  └── parse_sum       (+)
       └── parse_term  (*)
            └── parse_factor (rakam veya parantez)
                 ├── parse_number
                 └── parse_sum  ← parantezde recursive çağrı
```

Parser **sayı sonucunu** ve **okuma konumunu** birlikte ilerletir. Daha sonra üçüncü bilgi olarak **hata durumu** eklenir.

---

## Görev 1 — Program iskeleti

**Amaç:** Komut satırı argümanlarını anlamak.

**Yapılacaklar**
- [ ] `main(int ac, char **av)` yaz.
- [ ] `ac != 2` ise `1` döndür.
- [ ] Tek argümanı ekrana yazdır.
- [ ] Başarılı durumda `0` döndür.

**Test:** `./vbc '3+4*5'` → `3+4*5`.


### Çözüm — Görev 1


```c
#include <stdio.h>

int main(int ac, char **av)
{
    if (ac != 2)
        return (1);
    printf("%s\n", av[1]);
    return (0);
}
```


**Kritik bilgi:** `ac`, program adını da sayar; bu yüzden tek kullanıcı argümanı için `ac == 2` gerekir.

## Görev 2 — String'i karakter karakter oku

**Amaç:** Parser'ın bir karakterde durup sonra ilerlemesini öğrenmek.

**Yapılacaklar**
- [ ] `char *str = av[1]` ve `int i = 0` oluştur (istersen ayrı satırlarda ata).
- [ ] `while (str[i])` ile ilerle.
- [ ] Her karakteri `%c\n` ile yazdır.

**Test:** `./vbc '3+4'` → alt alta `3`, `+`, `4`.


### Çözüm — Görev 2


```c
char *str;
int i;

str = av[1];
i = 0;
while (str[i])
{
    printf("%c\n", str[i]);
    i++;
}
```


**Kritik bilgi:** C string'inin sonundaki `\0` karakteri döngüyü durdurur.

## Görev 3 — Karakter türünü belirle

**Amaç:** Geçerli token türlerini ayırmak.

**Yapılacaklar**
- [ ] `void print_type(char c)` yaz.
- [ ] Rakam → `DIGIT`; `+` veya `*` → `OPERATOR`.
- [ ] `(` → `OPEN`; `)` → `CLOSE`; diğerleri → `INVALID`.
- [ ] Görev 2 döngüsünden çağır.

**Test:** `3+(4*5)` → sırasıyla `DIGIT OPERATOR OPEN DIGIT OPERATOR DIGIT CLOSE`.


### Çözüm — Görev 3


```c
#include <ctype.h>

void print_type(char c)
{
    if (isdigit((unsigned char)c))
        printf("DIGIT");
    else if (c == '+' || c == '*')
        printf("OPERATOR");
    else if (c == '(')
        printf("OPEN");
    else if (c == ')')
        printf("CLOSE");
    else
        printf("INVALID");
}
```

Döngüde `print_type(str[i]); printf("\n");` çağır.


**Kritik bilgi:** Bu fonksiyon bir **öğrenme egzersizi**; nihai parser'da kullanılmayacak.

## Görev 4 — `parse_number`: ilk gerçek parser fonksiyonu

**Amaç:** Okunan rakamı tamsayıya dönüştürürken çağıranın pointer'ını ilerletmek.

**Prototip:** `int parse_number(char **str);`

**Yapılacaklar**
- [ ] `**str - '0'` ile sayısal değeri hesapla.
- [ ] `(*str)++` ile pointer'ı ilerlet.
- [ ] Sayıyı döndür.

**Test:** `str = "3+4"` iken fonksiyon `3` döndürmeli ve `str` artık `"+4"` göstermeli.


### Çözüm — Görev 4


```c
int parse_number(char **str)
{
    int num;

    num = **str - '0';
    (*str)++;
    return (num);
}
```


> [!important] Neden `char **str`?
> `char *str` ile pointer'ın kopyasını göndeririz; fonksiyon içindeki `str++` çağıranın pointer'ını değiştirmez. `&str` gönderip parametreyi `char **` yaparsak `(*str)++` çağıranın pointer'ını ilerletir. `**str` mevcut karakterdir, `*str` karakterin adresidir. Parantez önemlidir: `(*str)++` pointer'ı ilerletir; `*str++` ise yerel çift pointer'ı ilerletmeye çalışır.

## Görev 5 — İki rakamı topla

**Amaç:** Parser fonksiyonunu `main` içinde kullanmak.

**Yapılacaklar**
- [ ] `parse_number(&str)` ile ilk rakamı al.
- [ ] `*str == '+'` ise `str++` yap.
- [ ] İkinci rakamı alıp toplama ekle.
- [ ] Sonucu yazdır.

**Test:** `3+4` → `7`. Bu aşamada yalnızca geçerli `rakam+rakam` biçimini test et.


### Çözüm — Görev 5, main'in ilgili kısmı


```c
str = av[1];
total = parse_number(&str);
if (*str == '+')
{
    str++;
    total += parse_number(&str);
}
printf("%d\n", total);
```


**Kritik bilgi:** Bu tasarım sadece bir `+` işleyebilir; tekrarlanan işlemler için döngü gerekecek.

## Görev 6 — `parse_sum`: sınırsız sayıda toplama

**Amaç:** `1+2+3+4` gibi ifadeleri çözmek.

**Prototip:** `int parse_sum(char **str);`

**Yapılacaklar**
- [ ] İlk sayıyı oku.
- [ ] Mevcut karakter `+` olduğu sürece pointer'ı ilerlet, sıradaki sayıyı topla.
- [ ] Sonucu döndür.

**Test:** `9` → `9`, `1+2+3+4` → `10`.


### Çözüm — Görev 6


```c
int parse_sum(char **str)
{
    int sum;

    sum = parse_number(str);
    while (**str == '+')
    {
        (*str)++;
        sum += parse_number(str);
    }
    return (sum);
}
```


**Kritik bilgi:** `parse_sum` içinde `str` zaten `char **`; `parse_number(str)` çağırılır, `parse_number(&str)` değil.

## Görev 7 — `parse_term`: çarpma ve işlem önceliği

**Amaç:** `3+4*5` sonucunun `35` değil `23` olmasını sağlamak.

**Prototip:** `int parse_term(char **str);`

**Yapılacaklar**
- [ ] `parse_term` ilk sayıyı okusun, `*` gördükçe çarpsın.
- [ ] `parse_sum` içindeki **iki** `parse_number` çağrısını `parse_term` ile değiştir.
- [ ] `parse_sum` döngüsünde **yalnızca `+`** kontrol et.

**Test:** `2*3*4` → `24`, `2+3*4` → `14`, `2*3+4*5` → `26`.


### Çözüm — Görev 7


```c
int parse_term(char **str)
{
    int result;

    result = parse_number(str);
    while (**str == '*')
    {
        (*str)++;
        result *= parse_number(str);
    }
    return (result);
}

int parse_sum(char **str)
{
    int sum;

    sum = parse_term(str);
    while (**str == '+')
    {
        (*str)++;
        sum += parse_term(str);
    }
    return (sum);
}
```


**Düşün:** `parse_term("2+3")` neden `2` döndürüp pointer'ı `+` üzerinde bırakmalı? Çünkü `+` operatörünü üst seviyedeki `parse_sum` işleyecek.

## Görev 8 — `parse_factor`: parantez ve recursion

**Amaç:** `2*(3+4)` ve iç içe parantezleri hesaplamak.

**Prototip:** `int parse_factor(char **str);`

**Yapılacaklar**
- [ ] `(` yoksa `parse_number(str)` döndür (şimdilik geçerli girdi varsay).
- [ ] `(` varsa onu geç, `parse_sum(str)` ile içeriyi hesapla, `)` karakterini geç.
- [ ] `parse_term` içindeki iki `parse_number` çağrısını `parse_factor` ile değiştir.

**Test:** `(3+4)` → `7`, `2*(3+(4*5))` → `46`.


### Çözüm — Görev 8 (yalnızca geçerli girdiler için)


```c
int parse_factor(char **str)
{
    int result;

    if (**str != '(')
        return (parse_number(str));
    (*str)++;
    result = parse_sum(str);
    (*str)++;
    return (result);
}
```

`parse_term` içinde her iki `parse_number(str)` çağrısını `parse_factor(str)` yap.


> [!tip] Recursion neden duruyor?
> `parse_factor` parantez gördüğünde tekrar `parse_sum` çağırır. Fakat rakam gördüğünde **base case** olarak `parse_number` çağırır; yeniden `parse_sum` çağırmaz. Geçerli ve sonlu bir ifade için her çağrı yeni karakterler tüketir.

> [!warning] Bu aşamanın bilinçli eksiği
> Kapanış `)` var mı kontrol edilmedi. Bu kod **yalnızca geçerli girdilerde** kullanılmalı; hata yönetimi sonraki aşamada eklenir.

## Görev 9 — Rakam / parantez / geçersiz karakter ayrımı

**Amaç:** `a`, `+`, `)` gibi karakterleri yanlışlıkla sayı olarak okumamak.

**Yapılacaklar**
- [ ] `parse_factor` içinde rakamsa `parse_number` çağır.
- [ ] `(` ise parantez işleme dalına gir.
- [ ] Diğer durumları hata dalına ayır.

**Test:** `3+a` → hatalı; `()` → hatalı; `3+4` → geçerli.

**İpucu:** `isdigit((unsigned char)**str)` kullanılabilir. Geçersiz dalda başlatılmamış `result` döndürme.


### Çözüm iskeleti — Görev 9


```c
if (isdigit((unsigned char)**str))
    return (parse_number(str));
else if (**str == '(')
{
    /* parantezi aç, içeriği hesapla, kapanışı doğrula */
}
else
{
    /* hata durumunu ele al */
}
```


## Görev 10 — Doğru hata mesajı ve kapanış parantezi

**Amaç:** Eksik `)` ile beklenmedik karakteri ayırmak.

**Yapılacaklar**
- [ ] `void print_error(char c)` oluştur.
- [ ] `c == '\0'` ise `Unexpected end of input` yaz.
- [ ] Değilse `Unexpected token '%c'` yaz.
- [ ] Parantez içindeki `parse_sum` sonrasında **gerçekten `)` var mı** kontrol et.

**Test:** `(3+4` → `Unexpected end of input`; `(3+4a` → `Unexpected token 'a'`.


### Çözüm — ortak hata yazdırma


```c
void print_error(char c)
{
    if (c == '\0')
        printf("Unexpected end of input\n");
    else
        printf("Unexpected token '%c'\n", c);
}
```


> [!danger] Sınav kuralı
> `exit(1)` kullanma; paylaşılan soru metninde izinli değil. Hata yazdırmak programı durdurmaz. Bir sonraki adımda hata durumunu çağıran fonksiyonlara ileteceğiz.

## Görev 11 — `int *error` ile hatayı yukarı taşı

**Amaç:** Hesaplanan `0` ile hata durumunu birbirinden ayırmak.

**Yeni prototipler**

```c
int parse_sum(char **str, int *error);
int parse_term(char **str, int *error);
int parse_factor(char **str, int *error);
```

**Yapılacaklar**
- [ ] `main` içinde `int error = 0` tanımla.
- [ ] `parse_sum(&str, &error)` çağır.
- [ ] `parse_factor` hata bulunca `print_error(**str)` ve `*error = 1` yapsın, `0` döndürsün.
- [ ] `parse_term` ve `parse_sum`, her alt çağrıdan sonra `*error` kontrol etsin.
- [ ] Parantez içindeki `parse_sum` hata verirse kapanış parantezi kontrolüne devam etme.

**Test:** `2**3`, `3+`, `()` doğru hata vermeli; `0*5` geçerli olarak `0` vermeli.


### Çözüm — Görev 11, parser fonksiyonları


```c
int parse_sum(char **str, int *error)
{
    int sum;

    sum = parse_term(str, error);
    if (*error)
        return (0);
    while (**str == '+')
    {
        (*str)++;
        sum += parse_term(str, error);
        if (*error)
            return (0);
    }
    return (sum);
}

int parse_term(char **str, int *error)
{
    int result;

    result = parse_factor(str, error);
    if (*error)
        return (0);
    while (**str == '*')
    {
        (*str)++;
        result *= parse_factor(str, error);
        if (*error)
            return (0);
    }
    return (result);
}

int parse_factor(char **str, int *error)
{
    int result;

    if (*error)
        return (0);
    if (isdigit((unsigned char)**str))
        return (parse_number(str));
    if (**str == '(')
    {
        (*str)++;
        result = parse_sum(str, error);
        if (*error)
            return (0);
        if (**str == ')')
        {
            (*str)++;
            return (result);
        }
    }
    print_error(**str);
    *error = 1;
    return (0);
}
```


**Kritik bilgi:** `0` geçerli sonuçtur; bu nedenle yalnızca `return (0)` ile hata sinyali verilemez. `*error` ayrı bir kanaldır.

## Görev 12 — `main` içinde kalan karakterleri denetle

**Amaç:** `3+4abc` veya `3+4)` gibi yarım çözümlenen girdileri reddetmek.

**Yapılacaklar**
- [ ] `str = av[1]`, `error = 0`.
- [ ] `sum = parse_sum(&str, &error)`.
- [ ] `error` varsa `1` döndür (hata zaten yazdırıldı).
- [ ] `*str != '\0'` ise `print_error(*str)` ve `1` döndür.
- [ ] Aksi halde **yalnızca** `printf("%d\n", sum)` ve `0` döndür.

**Test:** `3+4abc` → `Unexpected token 'a'`; `3+4)` → `Unexpected token ')'`.


### Çözüm — Görev 12


```c
int main(int ac, char **av)
{
    char *str;
    int sum;
    int error;

    if (ac != 2)
        return (1);
    str = av[1];
    error = 0;
    sum = parse_sum(&str, &error);
    if (error)
        return (1);
    if (*str != '\0')
    {
        print_error(*str);
        return (1);
    }
    printf("%d\n", sum);
    return (0);
}
```


---

# Tam çözüm — sınav için tek dosya `vbc.c`

> [!note] Önce kendin yaz
> Aşağıdaki kod **referans çözüm**dür. İlk çalışmada değil, 12 görevi tamamladıktan sonra aç.


### Referans kodu aç


```c
#include <stdio.h>
#include <ctype.h>

int parse_sum(char **str, int *error);
int parse_term(char **str, int *error);
int parse_factor(char **str, int *error);

void print_error(char c)
{
    if (c == '\0')
        printf("Unexpected end of input\n");
    else
        printf("Unexpected token '%c'\n", c);
}

int parse_number(char **str)
{
    int num;

    num = **str - '0';
    (*str)++;
    return (num);
}

int parse_factor(char **str, int *error)
{
    int result;

    if (*error)
        return (0);
    if (isdigit((unsigned char)**str))
        return (parse_number(str));
    if (**str == '(')
    {
        (*str)++;
        result = parse_sum(str, error);
        if (*error)
            return (0);
        if (**str == ')')
        {
            (*str)++;
            return (result);
        }
    }
    print_error(**str);
    *error = 1;
    return (0);
}

int parse_term(char **str, int *error)
{
    int result;

    result = parse_factor(str, error);
    if (*error)
        return (0);
    while (**str == '*')
    {
        (*str)++;
        result *= parse_factor(str, error);
        if (*error)
            return (0);
    }
    return (result);
}

int parse_sum(char **str, int *error)
{
    int sum;

    sum = parse_term(str, error);
    if (*error)
        return (0);
    while (**str == '+')
    {
        (*str)++;
        sum += parse_term(str, error);
        if (*error)
            return (0);
    }
    return (sum);
}

int main(int ac, char **av)
{
    char *str;
    int sum;
    int error;

    if (ac != 2)
        return (1);
    str = av[1];
    error = 0;
    sum = parse_sum(&str, &error);
    if (error)
        return (1);
    if (*str != '\0')
    {
        print_error(*str);
        return (1);
    }
    printf("%d\n", sum);
    return (0);
}
```


## Derleme ve test

```bash
gcc -Wall -Wextra -Werror vbc.c -o vbc
./vbc '3+4*5'
echo $?
```

**Beklenen:** Önce `23`, sonra `0`.

| İfade | Beklenen çıktı | Çıkış kodu |
|---|---|---:|
| `1` | `1` | 0 |
| `1+2*3` | `7` | 0 |
| `(1+2)*3` | `9` | 0 |
| `2*(3+(4*5))` | `46` | 0 |
| `9*0+8` | `8` | 0 |
| `((1))` | `1` | 0 |
| `1++2` | `Unexpected token '+'` | 1 |
| `2**3` | `Unexpected token '*'` | 1 |
| `(1+2` | `Unexpected end of input` | 1 |
| `1+2)` | `Unexpected token ')'` | 1 |
| `()` | `Unexpected token ')'` | 1 |
| `""` | `Unexpected end of input` | 1 |
| `3+4abc` | `Unexpected token 'a'` | 1 |
| `(3+4a` | `Unexpected token 'a'` | 1 |

**Ek testler:** `1(2)`, `1  +2`, `2+`, `2*`, `((2+3)*4)`, `0`, `9*9*9`.

> [!important] İfade sınırları
> Bu problemde boşluklar geçersiz kabul edilir; `12` gibi yan yana rakamlar iki basamaklı sayı olarak yorumlanmaz. Parser ilk rakamdan sonra kalanı kontrol ederek bu tür girdileri reddeder. C `int` taşması kapsam dışı olabilir; çok uzun ifadeler için taşma davranışı bu labın hedefi değildir.

## Sınav öncesi kontrol listesi

- [ ] `char **str` ile `(*str)++` arasındaki ilişkiyi açıklayabiliyorum.
- [ ] `parse_number` sadece bir rakam tüketiyor.
- [ ] `parse_term` yalnızca `*` işliyor.
- [ ] `parse_sum` yalnızca `+` işliyor.
- [ ] `parse_factor` rakam veya parantez işliyor.
- [ ] Parantez için recursive `parse_sum` çağrısı yapıyorum.
- [ ] Hata durumunu `int *error` ile yukarı iletiyorum.
- [ ] Hata sonrası başka işlem yapmıyorum.
- [ ] `main` sonunda tüm string'in tüketildiğini doğruluyorum.
- [ ] Çıktıda `Total:` gibi ek metin yok.
- [ ] Hata durumunda çıkış kodu `1`.
- [ ] `exit()` veya izin verilmeyen başka fonksiyon kullanmıyorum.
- [ ] `-Wall -Wextra -Werror` ile derleniyor.

## Kendini sınama — çözüme bakmadan cevapla

1. Neden `parse_sum`, `parse_number` yerine `parse_term` çağırıyor?
2. `2+3*4` için hangi fonksiyon hangi karakterde duruyor?
3. `(*str)++` ile `*str++` neden farklı?
4. `parse_factor` içinden `parse_sum` çağrısı neden sonsuza kadar devam etmiyor?
5. Neden `return (0)` tek başına hata belirtmek için yeterli değil?
6. `(1+2` ifadesinde hangi fonksiyon hangi hatayı buluyor?
7. `1+2)` neden `parse_factor` değil `main` tarafından reddediliyor?

> [!success] Tekrar hedefi
> Dosyayı kapat. Yalnızca görev başlıklarına bakarak `vbc.c` dosyasını **sıfırdan** yaz. Sonra hata testlerini ve çıkış kodlarını doğrula.

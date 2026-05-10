---
parent: Architectural Decisions
nav_order: 9
---

# Menggunakan Plain JUnit5 untuk Assertion Testing Lanjutan

## Konteks dan Permasalahan

Bagaimana cara menulis assertion testing yang mudah dibaca?  
Bagaimana cara menjaga agar assertion pada pengujian yang kompleks tetap mudah dipahami dan dirawat?

---

## Opsi yang Dipertimbangkan

- Plain JUnit5
- Hamcrest
- AssertJ

---

## Hasil Keputusan

Opsi yang dipilih: **"Plain JUnit5"**, karena memberikan keseimbangan terbaik antara kesederhanaan, keterbacaan, dan kemudahan penggunaan dalam ekosistem Java.

---

## Konsekuensi

### Dampak Positif

- Test menjadi lebih mudah dipahami oleh developer Java pada umumnya.
- Tidak memerlukan library tambahan untuk assertion.
- Mengurangi kompleksitas dependency project.
- Menjaga konsistensi gaya penulisan testing di seluruh project.

### Dampak Negatif

- Assertion yang kompleks dapat menjadi lebih panjang dan sulit dibaca.
- Validasi lanjutan dapat menghasilkan kode testing yang verbose.
- Tidak memiliki fluent API seperti AssertJ.

---

# Kelebihan dan Kekurangan dari Setiap Opsi

## Plain JUnit5

**Homepage:**  
<https://junit.org/junit5/docs/current/user-guide/>

**Panduan Testing:**  
<https://devdocs.jabref.org/getting-into-the-code/code-howtos#test-cases>

### Contoh

```java
String actual = markdownFormatter.format(source);

assertTrue(actual.contains("Markup<br />"));
assertTrue(actual.contains("<li>list item one</li>"));
assertTrue(actual.contains("<li>list item 2</li>"));
assertTrue(actual.contains("> rest"));

assertFalse(actual.contains("\n"));
```

### Kelebihan

- Sudah menjadi pengetahuan umum dalam pengembangan Java
- Tidak memerlukan dependency tambahan
- Sintaks sederhana dan mudah dipahami
- Mempermudah onboarding developer baru

### Kekurangan

- Assertion kompleks dapat menjadi sulit dibaca
- Tidak memiliki fluent API
- Banyak assertion berulang dapat membuat kode verbose

---

## Hamcrest

**Homepage:**  
<https://github.com/hamcrest/JavaHamcrest>

### Kelebihan

- Menyediakan matcher yang lebih fleksibel dan ekspresif
- Lebih baik untuk validasi kondisi tertentu
- Membantu meningkatkan keterbacaan dibanding assertion dasar

### Kekurangan

- Bukan fluent API sepenuhnya
- Menambah learning curve bagi developer baru
- Menambah kompleksitas project

---

## AssertJ

**Homepage:**  
<https://joel-costigliola.github.io/assertj/>

### Contoh

```java
assertThat(markdownFormatter.format(source))
        .contains("Markup<br />")
        .contains("<li>list item one</li>")
        .contains("<li>list item 2</li>")
        .contains("> rest")
        .doesNotContain("\n");
```

### Kelebihan

- Menggunakan fluent assertion yang lebih mudah dibaca
- Cocok untuk pengujian string parsial
- Sangat ekspresif untuk assertion kompleks
- Membuat assertion lebih terstruktur

### Kekurangan

- Tidak digunakan secara umum di semua project Java
- Developer perlu mempelajari API tambahan
- Menambah hambatan onboarding
- Gaya assertion dapat berbeda antar unit test

---
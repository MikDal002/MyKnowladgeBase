---
autor: Mikołaj Dalecki <mikdal_opensource@outlook.com>
data: 23.03.2018
tytuł: Trochę mojej wiedzy
---

<link href="prism.css" rel="stylesheet"/>
<script src="prism.js"></script>

# Trochę mojej wiedzy

## Wstęp

Witam w moim repozytorium, z którego robię pewien erazc bloga.
Wrzucam tutaj takie rzeczy, które często wypadają mi z głowy.
Jeżeli uważasz, że któreś z moich rozwiązań jest błędne, to proszę, daj mi znać.

Ważną dla mnie rzeczą jest to, aby cały blog był bardzo czytelny i elegancki zarazem.
Dodatkowo chciałbym, na ile to możliwe, posługiwać się językiem polskim przy wyjaśnianiu wszelakich pojęć.

Dokument, który teraz czytasz jest dla mnie wzorcem, pokazującym jak powinienem organizować wszystkie inne pliki.

## Wstawianie elementów

Poniżej prezentuję w jaki sposób powieniem wstawiać wszystkie elementy do pliku, oraz jak powinno to wyglądać.

### Kod

Czas na mały przykładowy blok kodu, wemy na przykład taki z Qt:

<figure>
<pre class="language-cpp"><code>
#include <QDebug>
#include <QString>
QString name = "Trochę mojej wiedzy";
if (name.lenght() > 5)
{
    qDebug() << "Ho, ho, ale się rozpisałeś!";
}</code></pre>
<figcaption>Krótki przykładowy kodzik.</figcaption>
</figure>


Jak widać, również i tutaj cały blok z kodem źródłowym zajął całą dostępną szerokość kolumny, to bardzo miłe z jego strony.
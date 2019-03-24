---
autor: Mikołaj Dalecki <mikdal_opensource@outlook.com>
data: 23.03.2018
tytuł: Przechowywanie konfiguracji oprogramowania
---

# Przechowywanie konfiguracji oprogramowania

- [Przechowywanie konfiguracji oprogramowania](#przechowywanie-konfiguracji-oprogramowania)
  - [Wstęp](#wst%C4%99p)
  - [Realizacja obsługi pliku konfiguracyjnego](#realizacja-obs%C5%82ugi-pliku-konfiguracyjnego)
    - [Prosta implementacja w kodzie](#prosta-implementacja-w-kodzie)
    - [Wersjonowanie](#wersjonowanie)

## Wstęp

W końcu, po długich miesiącach poszukiwań, w mojej głowie wyklarowało się pojęcie „konfiguracji aplikacji”.

Aby do tego doszło musiałem uświadomić sobię różnicę pomiędzy *stanem* aplikacji a jej *konfiguracją*.
Na czym polega ta różnica według mnie?

- **Konfiguracja** — pewne parametry aplikacji ustawione raz, i takie, jakie ustawił użytkownik, powinny pozostać nawet pomiędzy aktualizacjami. Parametry te powinny być również łatwe do przenoszenia pomiędzy różnymi maszynami oraz platformami, oraz warto pomyśleć nad łatwym dostępem do pliku konfiguracyjnego dla użytkownika bardziej zaawansowanego.

- **Stan** — są to wszystkie parametry aplikacji, które zostały ustawione przez użytkownika (tak samo jak w przypadku konfiguracji) jednak NIE powinny być one przenoszone między maszynami czy platformami (pewnym wyjątkiem może być tutaj funkcjonalność, którą można nazwać „kontynuuj na innym urządzeniu”). Do tego użytkownik nie powinien w tym pliku w ogóle grzebać.

Jakie konsekwencje niosą ze sobą powyższe spostrzeżenia?
Najważniejsze z nich to fakt, że nad zapisem/odczytem konfiguracji programista powinien mieć *pełną kontrolę* – także pomiędzy obiektami w aplikacji a samym plikiem konfiguracyjnym MUSI znaleźć się warstwa abstrakcji odpowiedzialna za przełożenie wartości obiektów na linijki w pliku.
Natomiast jeśli chodzi o zapis stanu, to warstwa abstrakcji może być zmniejszona do granic możliwości i opierać się głównie na refleksji – w takim przypadku wraz z zmianą nazwy jakiejść właściwości nie przywrócimy poprzedniego stanu, jednak w tym przypadku nie będzie to poważne zaniedbanie.

***Dobre praktyki przy pracy z plikiem konfiguracyjnym:***

1. Plik konfiguracyjny posiada **numer wersji**,
2. Wszystkie nazwy pól pliku konfiguracyjnego są zawarte, **jako stałe** w module odpowiedzialnym za odczyt/zapis danych.
3. Moduł odczytu/zapisu danych ma świadomość (w **postaci stałej**) jakiego typu dane i w jakim formacie znajdują się w każdym polu pliku o konfiguracyjnego

Także wszystko co najważniejsze zostało zawarte powyżej.
Jednak dalsze szczegóły zawieram w poniższych sekcjach:

## Realizacja obsługi pliku konfiguracyjnego

### Prosta implementacja w kodzie

Skoro jest to blog programistyczny, to nie może zabraknąć jednak kodu.
Dlatego poniżej przedstawiam uproszczoną obsługę pliku konfiguracyjnego, która skupia się w okół trzech przedstawionych wcześniej założeń:

<figure>

```xml
<?xml version="1.0"?>
<Konfiguracja Wersja="1.0.0">
    <Imię>Franek</Imię>
    <Wiek>7</Wiek>
    <ID>0x7AB</ID>
</Konfiguracja>
```

<figcaption>Wynikowy plik konfiguracyjny w postaci XML</figcaption>
</figure>

<figure>

```C++
    const string RootElementName = "Konfiguracja";
    const string VersionAttributeName = "Wersja";
    const string NamePropertyFieldName = "Imię";
    const string AgePropertyFieldName = "Wiek";
    const string IdPropertyFieldName = "ID";
```

<figcaption>Elementy wspólne dla odczytu i zapisu danych z pliku</figcaption>
</figure>

<figure>

```c++
void saveToFile(string filePath, User userToStore)
{
    XmlWriter xml = XmlWriter(filePath);
    xml.startElement(RootElementName);
    xml.writeAttribute(VersionAttributeName, "1.0.0");
    xml.writeTextElement(NamePropertyFieldName, userToStore.name);
    xml.writeTextElement(AgePropertyFieldName, StringHelper::toDecimalString(userToStore.age);
    xml.writeTextElement(IdPropertyFieldName, StringHelper::toHexString(userToStore.id);
    xml.endElement();
}
```

<figcaption>Prosty zapis do pliku. Podobieństwo do QXmlStreamWriter jest zamierzone.</figcaption>
</figure>

<figure>

```c++
User readFromFile(string filePath)
{
    User retUsr();
    auto xml = XmlReader(filePath);
    string version = xml.readAttribute(VersionAttributeName);
    retUsr.name = xml.readTextElement(NamePropertyFieldName);
    retUsr.age = StringHelper::fromDecimalString(AgePropertyFieldName);
    retUsr.id = StringHelper::fromHexString(IdPropertyFieldName);
    return retUsr;
}
```

<figcaption>Prosty odczyt do pliku – w postaci niemalże pseudokodu</figcaption>
</figure>

Tak, zdaję sobie sprawę z tego, że w powyższych kodach jest bardzo dużo uproszczeń.
Jednak są one aż nadto wystarczające aby zaprezentować o co mi chodzi.

> [!INFO]
> W C++ można uzyskać nazwę pola poprzez proste makro `#define nameof(var) #var`, w Qt mamy `MetaObject`, nie wspominając o refleksji w językach podobnych do C#.
> Także istnieje realne ryzyko, że ktoś użyje nazwy pola klasy jako nazwę pola w pliku! Co, swoją drogą, zalecam podczas przechowywania informacji na temat stanu aplikacji.

Więc od początku.
Najpierw stworzyłem stałe reprezentujące nazwy pól w pliku – co jest chyba oczywiste, że powinny być w takiej postaci, a nie napisane na sztywno w kodzie.
Jednak wytłuściłem to, aby pokazać, że **nie biorę** nazw pól w pliku z nazw cech klasy!
Bo w rzeczywistości mógłbym zrobić na przykład coś takiego: `retUser.name = xml.readTextElement(nameof(retUser.name));`.
Jednak kompletnie mijałoby się to z celem, ponieważ, jak zostało już wcześniej powiedziane, konfiguracja musi być przenośna między wersjami, a między wersjami produktu mogą zmienić się nazwy pól klasy. Ba! Klasa może nawet zniknąć!

Drugi listing to zapis danych do pliku.
Jak widzisz zapisałem numer wersji – mimo iż wynosi on 1.0.0 i nie wiem, czy kiedykolwiek będę tu coś zmieniał, to warto to zrobić. Jednak ważniejsze jest coś innego, chodzi o spełnienie warunku numer 3: **świadomość typów danych i ich formatów w pliku konfiguracyjnym**. W tym przypadku jest to realizowane przez takie fragmenty kodu jak `StringHelper::toDecimalString()` czy `StringHelper::toHexString()` (pamiętaj, że klasy typu „helper” nie są najszczęśliwsze – użyłem ich tutaj tylko dla prostoty).
Dzięki zastosowaniu jawnej kowersji z typu liczbowego na ciąg znaków mamy informację o tym jak dane są przechowywane w pliku.

### Wersjonowanie

*P.S. Ten akapit znajduje się tutaj tylko dla porządku i nie zawiera nic odkrywczego.
Może być on co najwyżej przydatny dla osób, które nie mają bladego pojęcia o nadawaniu numeru wersji.*
<br/><br/>

Przy imporcie i eksporcie ustawień pojawia się problemu związany z tym, że różne wydania tej samej aplikacji mogą korzystać z odmiennego zestawu danych w plikach konfiguracyjnych.
Wersjonowanie pliku pozwala nam określić, czy nasza aplikacja na pewno zrozumie, to co jest zapisane w pliku konfiguracyjnym.

> [!TŁUMACZENIE]
> **Wersjonowanie sematyczne** – brzmi to co najmniej enigmatycznie i mi, jako polakowi, kompletnie nic nie mówi.
> Znacznie łatwiej jest mi zapamiętać, gdy powiem sobie „wersjonowanie oparte na systemie pozycyjnym”, „wersjonowanie pozycyjne” i kompletnie nie rozumiem dlaczego autor użył słowa „semantyczne”.

Jest kilka popularnych sposobów na wersjonowanie wszelakich rzeczy.
Można opierać się na dacie, nazwa kodowych, bądź na tzw. „wersjonowaniu semantycznym“.
O plusach i minusach poszczególnych rozwiązaniach nie będę się tutaj rozwodził.
Według mnie do wersjonowania plików konfiguracyjnych najlepiej nadaje się właśnie semantyczne.

Zgodnie z zamysłem autora wersjonowanie semantyczne powinno składać się z około trzech pól reprezentujących wersję: x.y.z, gdzie x – kolejny numer publicznego API, y – numer aktualizacji tegoż api (bez zmian łamiących wsteczną kompatybilność), z – kolejny numer poprawki usuwającej błedy. To tak w skrócie.
Pamiętaj, że naszym publicznym API jest właśnie zbiór zestawów: `nazwa pola – wartość w postaci tekstu`.
Tak więc, kiedy zmianiam poszczególne numery?

- X - numer publicznego API, zmieniamy zawsze wtedy, kiedy • zmienia się nazwa pola, • zmienia się typ wartości, • zmienia się format wartości, • bądź usuwamy jakieś zestaw.
- Y – numer aktualizacji publicznego API – zmieniamy zawsze wtedy, gdy dodajemy nowe pole do pliku konfiguracyjnego. Dodanie nowego pola nie psuje nam kompatybilności wstecznej, ponieważ starsze czytniki mogą po prostu **pominąć** takie nieznane im pole.
- Z – przyznam się, że mogę tylko gdybać kiedy należy zatualizować tę wartość dlatego przeważnie jej nie używam.

Z powyższych rozważań rodzą się proste wnioski:

1. Wersja czytnik w wersji  2.0 odczyta plik w wersjach 2.x i 1.y,
2. Czytnik w wersji 1.7 odczyta wszystkie pliki w wersjach 1.x, z tym, że gdy x > 7, to pominie nieznane wartości, najlepiej komunikując to ostrzeżeniem.
3. Czytnik w wersji 1.x nie powinien odczytać nic z wersji 2.x, 3.y itd., nawet, jeśli niektóre pola są dokładnie takie same.
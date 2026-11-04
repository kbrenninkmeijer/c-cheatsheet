# C# OOP Cheat Sheet

Snel naslagwerk voor de OOP-toets. Gebruik `Ctrl+F` om een begrip te zoeken.

## Inhoud
1. [Git basics](#1-git-basics)
2. [Classes, objects, fields, methods](#2-classes-objects-fields-methods)
3. [Overloading](#3-overloading)
4. [Constructors & chaining](#4-constructors--chaining)
5. [Object state & reference types](#5-object-state--reference-types)
6. [Access modifiers](#6-access-modifiers)
7. [Encapsulation, getters/setters, properties](#7-encapsulation-getterssetters-properties)
8. [Cohesion](#8-cohesion)
9. [Inheritance](#9-inheritance)
10. [is-a vs has-a](#10-is-a-vs-has-a)
11. [Constructors bij inheritance](#11-constructors-bij-inheritance)
12. [Overriding vs hiding](#12-overriding-vs-hiding)
13. [Generics & constraints](#13-generics--constraints)
14. [Virtual / abstract](#14-virtual--abstract)
15. [Polymorfisme & dynamic dispatch](#15-polymorfisme--dynamic-dispatch)
16. [Upcasting / downcasting](#16-upcasting--downcasting)
17. [Interfaces](#17-interfaces)
18. [Interface vs abstract class](#18-interface-vs-abstract-class)
19. [Dependency inversion & injection](#19-dependency-inversion--injection)
20. [Exceptions](#20-exceptions)
21. [Logging](#21-logging)
22. [Exception anti-patterns](#22-exception-anti-patterns)
23. [Design patterns](#23-design-patterns)
24. [Veelgemaakte fouten](#24-veelgemaakte-fouten)
25. [Output-vragen: stappenplan](#25-output-vragen-stappenplan)

---

## 1. Git basics

| Commando | Doet |
|---|---|
| `git init` | nieuwe repo maken |
| `git clone <url>` | repo downloaden |
| `git status` | wat is gewijzigd? |
| `git add .` | alle wijzigingen klaarzetten (staging) |
| `git commit -m "bericht"` | snapshot opslaan |
| `git log --oneline` | geschiedenis bekijken |
| `git branch <naam>` | nieuwe branch |
| `git switch <naam>` / `git checkout <naam>` | naar branch gaan |
| `git switch -c <naam>` | branch maken + erheen gaan |
| `git merge <naam>` | branch samenvoegen in huidige |
| `git pull` | ophalen + mergen van remote |
| `git push` | lokale commits naar remote |

- **Working directory → staging (`add`) → repository (`commit`) → remote (`push`)**
- **Merge conflict**: twee branches wijzigden dezelfde regel → handmatig oplossen tussen `<<<<<<<`, `=======`, `>>>>>>>`, daarna `add` + `commit`.
- `.gitignore`: bestanden die Git moet negeren (bijv. `bin/`, `obj/`).

---

## 2. Classes, objects, fields, methods

- **Class** = blauwdruk. **Object** = concreet exemplaar, gemaakt met `new`.
- **Field (attribute)** = variabele in de class (data).
- **Method** = functie in de class (gedrag).

```csharp
public class Hond
{
    public string Naam;                 // instance field: elk object eigen waarde
    public static int Aantal;           // static field: gedeeld door ALLE objecten

    public void Blaf() => Console.WriteLine($"{Naam}: Woef!");   // method
}

var h = new Hond();   // object maken
h.Naam = "Rex";
h.Blaf();             // Rex: Woef!
Hond.Aantal++;        // static via de CLASSnaam, niet via het object
```

| | Instance | Static |
|---|---|---|
| Hoort bij | elk object apart | de class zelf (één keer) |
| Aanroepen | `obj.Field` | `ClassNaam.Field` |
| Kan bij instance-members? | ja | **nee** (geen `this`) |

Method-opbouw: `access returntype Naam(parameters) { body }` — `void` = geeft niets terug.

---

## 3. Overloading

**Zelfde methodenaam, andere parameters** (aantal / type / volgorde), in **dezelfde class**.

```csharp
public int    Tel(int a, int b)        => a + b;
public double Tel(double a, double b)  => a + b;
public int    Tel(int a, int b, int c) => a + b + c;
```

- Compiler kiest **tijdens compileren** op basis van aantal + types van de argumenten.
- Beste (exacte) match wint; anders automatische conversie (`int` → `double` mag, andersom niet).
- **Telt NIET mee**: return type, parameternamen.

```csharp
public int    Tel(int a, int b) ...
public double Tel(int a, int b) ...   // ❌ CS0111: zelfde signature
```

**Signature = naam + parameterlijst (aantal, types, volgorde).**

Integer-deling valkuil:
```csharp
int som = 10; int n = 4;
som / n            // 2    (int / int → rest valt weg)
(double)som / n    // 2.5  ✅ eerst casten
```

---

## 4. Constructors & chaining

- Zelfde naam als de class, **geen return type** (ook geen `void`!).
- Wordt uitgevoerd bij `new`. Doel: object meteen in geldige toestand.
- **Geen** constructor geschreven → je krijgt een lege default `ClassNaam()`.
- **Wel** een constructor geschreven → default verdwijnt! (`new Student()` compileert dan niet.)
- `: this(...)` = andere constructor van **dezelfde** class aanroepen (chaining).

```csharp
public class Tijdstip
{
    public int Uren { get; }
    public int Minuten { get; }

    public Tijdstip(int uren, int minuten)   // de "echte" constructor
    {
        Uren = uren;        // parameter (klein) → field/property (groot)
        Minuten = minuten;
    }
    public Tijdstip(int uren) : this(uren, 0) { }
    public Tijdstip()         : this(0, 0)    { }

    public override string ToString() => $"{Uren:00}:{Minuten:00}";  // 08:05
}
```

- Parameter `uren` bestaat alleen tijdens de constructor; `Uren` blijft bij het object.
- Zelfde naam? Gebruik `this.naam = naam;`

---

## 5. Object state & reference types

- **State** = huidige waarden van de fields van een object.
- Classes zijn **reference types**: variabele bevat een *verwijzing*, niet het object zelf.
- `int`, `double`, `bool`, `struct` zijn **value types**: worden gekopieerd.

```csharp
var a = new Lamp();
var b = a;          // b wijst naar HETZELFDE object
b.Aan = true;
Console.WriteLine(a.Aan);   // True!

int x = 5;
int y = x;          // kopie
y = 10;
Console.WriteLine(x);       // 5
```

- `null` = verwijst naar niets → member aanroepen geeft `NullReferenceException`.
- `==` op objecten vergelijkt standaard of het **hetzelfde object** is (referentie), niet de inhoud.

---

## 6. Access modifiers

| Modifier | Zichtbaar in |
|---|---|
| `public` | overal |
| `private` | alleen binnen dezelfde class (**default voor members**) |
| `protected` | zelfde class + afgeleide classes |
| `internal` | zelfde project/assembly (**default voor classes**) |
| `protected internal` | zelfde assembly OF afgeleide class |
| `private protected` | afgeleide class binnen zelfde assembly |

---

## 7. Encapsulation, getters/setters, properties

**Encapsulation** = data `private` maken en alleen gecontroleerd toegang geven → object bewaakt zijn eigen geldige toestand (invarianten).

```csharp
// Java-stijl getter/setter (werkt, maar in C# gebruik je properties)
private int _leeftijd;
public int GetLeeftijd() => _leeftijd;
public void SetLeeftijd(int value) { if (value >= 0) _leeftijd = value; }
```

### Properties

```csharp
public class Persoon
{
    // 1. Auto-property
    public string Naam { get; set; }

    // 2. Read-only (alleen in constructor te zetten)
    public string Bsn { get; }

    // 3. Private setter (van buiten alleen lezen)
    public decimal Saldo { get; private set; }

    // 4. Full property met validatie (backing field)
    private int _leeftijd;
    public int Leeftijd
    {
        get => _leeftijd;
        set
        {
            if (value < 0) throw new ArgumentOutOfRangeException(nameof(value));
            _leeftijd = value;      // 'value' = de toegewezen waarde
        }
    }

    // 5. Berekende (computed) property: geen opslag
    public bool IsVolwassen => Leeftijd >= 18;

    // 6. init: alleen te zetten bij aanmaken
    public string Land { get; init; }
}
```

**Waarom geen `public` field?** Geen validatie mogelijk, je kunt de interne opslag later niet veranderen zonder alle code die het gebruikt te breken, iedereen kan ongeldige waarden zetten.

---

## 8. Cohesion

- **Hoge cohesie (goed)**: class heeft één duidelijke verantwoordelijkheid; alle members horen bij elkaar.
- **Lage cohesie (slecht)**: class doet van alles (berekenen + printen + opslaan + mailen).
- Hangt samen met **Single Responsibility Principle (SRP)**: een class heeft maar één reden om te veranderen.
- Oplossing: opsplitsen, bijv. `Bestelling` → `Bestelling`, `FactuurPrinter`, `BestellingRepository`, `EmailService`.
- (Tegenhanger: **coupling** = afhankelijkheid tussen classes → liefst **laag**.)

---

## 9. Inheritance

```csharp
public class Voertuig                       // base class / superclass
{
    public string Merk { get; set; }
    public virtual string Info() => $"Voertuig van {Merk}";
}
public class Auto : Voertuig                // derived class / subclass
{
    public int AantalDeuren { get; set; }
    public override string Info() => $"{base.Info()} met {AantalDeuren} deuren";
}
```

- `:` = "erft van". C# heeft **single inheritance**: max. **één** base class (wel meerdere interfaces).
- Afgeleide class krijgt alle (niet-private) members van de base.
- `private` members worden wel geërfd maar zijn **niet** bereikbaar in de subclass → gebruik `protected`.
- Alles erft uiteindelijk van **`object`** → elke class heeft `ToString()`, `Equals()`, `GetHashCode()`, `GetType()`.
- `sealed class X` → er mag niet van X geërfd worden. `sealed override` → verder overriden verboden.
- `base.Methode()` = versie van de base class aanroepen.

---

## 10. is-a vs has-a

| | is-a | has-a |
|---|---|---|
| Relatie | inheritance | composition (field) |
| Voorbeeld | `Hond : Dier` ("hond **is een** dier") | `Auto` heeft een `Motor` field |
| Code | `class Hond : Dier` | `private Motor _motor;` |

**Vuistregel: "favor composition over inheritance"** — gebruik inheritance alleen bij een echte is-a relatie. Twijfel? Composition is flexibeler (losser gekoppeld).

---

## 11. Constructors bij inheritance

- Constructors worden **niet** geërfd.
- **Base-constructor draait EERST**, daarna die van de subclass.
- Heeft base geen parameterloze constructor → subclass **moet** `: base(...)` aanroepen.

```csharp
class A { public A() { Console.Write("A "); } }
class B : A { public B() { Console.Write("B "); } }
class C : B { public C() { Console.Write("C "); } }

new C();   // Output: A B C
```

```csharp
public class Dier
{
    public string Naam { get; }
    public Dier(string naam) { Naam = naam; }
}
public class Kat : Dier
{
    public Kat(string naam) : base(naam) { }   // verplicht!
    // public Kat() { }  ❌ compileert niet: Dier() bestaat niet
}
```

`: this(...)` = zelfde class. `: base(...)` = parent class.

---

## 12. Overriding vs hiding

| | **override** | **new** (hiding) |
|---|---|---|
| Base-method moet zijn | `virtual` / `abstract` / `override` | maakt niet uit |
| Welke versie draait? | type van het **object** (runtime) | type van de **variabele** (compile-time) |
| Polymorf? | ✅ ja | ❌ nee |

```csharp
class Basis
{
    public virtual string A() => "Basis.A";
    public string B() => "Basis.B";
}
class Afgeleid : Basis
{
    public override string A() => "Afgeleid.A";
    public new string B() => "Afgeleid.B";
}

Basis x = new Afgeleid();
Afgeleid y = new Afgeleid();
x.A();  // "Afgeleid.A"  ← override: kijkt naar object
x.B();  // "Basis.B"     ← new: kijkt naar variabele-type (Basis)
y.A();  // "Afgeleid.A"
y.B();  // "Afgeleid.B"
```

- Vergeet je `override` én `new` → compiler-**warning**, gedrag = `new` (hiding).
- **Overloading** ≠ **overriding**:

| Overloading | Overriding |
|---|---|
| zelfde naam, **andere parameters** | zelfde naam, **zelfde parameters** |
| zelfde class | subclass vervangt base-versie |
| compile-time | runtime |
| geen keyword | `virtual` + `override` |

---

## 13. Generics & constraints

**Generics** = code voor "een type T" schrijven → **type safety** (fouten bij compileren) + **geen casting / boxing** + herbruikbaar.

```csharp
// Generic class
public class Doos<T>
{
    public T Inhoud { get; set; }
}
var d = new Doos<int> { Inhoud = 5 };

// Meerdere type-parameters
public class Paar<TEerste, TTweede>
{
    public TEerste Eerste { get; }
    public TTweede Tweede { get; }
    public Paar(TEerste e, TTweede t) { Eerste = e; Tweede = t; }
    public override string ToString() => $"({Eerste}, {Tweede})";
}

// Generic method
public static T Laatste<T>(List<T> lijst) => lijst[lijst.Count - 1];
var x = Laatste(new List<string> { "a", "b" });   // T wordt afgeleid: type inference
```

`ArrayList` (oud, slaat `object` op) vs `List<T>`: `List<T>` is type-safe (geen verkeerd type erin) en sneller (geen casting/boxing).

### Constraints (`where`)

| Constraint | Betekenis |
|---|---|
| `where T : class` | T moet reference type zijn |
| `where T : struct` | T moet value type zijn |
| `where T : new()` | T moet parameterloze constructor hebben (`new T()` mag) |
| `where T : Dier` | T moet `Dier` zijn of ervan erven |
| `where T : IComparable<T>` | T moet interface implementeren |
| `where T : notnull` | T mag niet nullable zijn |

```csharp
public static T Grootste<T>(T a, T b) where T : IComparable<T>
    => a.CompareTo(b) > 0 ? a : b;
// Zonder constraint: a > b of a.CompareTo(b) compileert NIET,
// want de compiler weet niet dat T vergelijkbaar is.

public static void PrintAlleNamen<T>(List<T> items) where T : Dier
{
    foreach (var i in items) Console.WriteLine(i.Naam);   // mag door constraint
}
```

Meerdere: `where T : Dier, IComparable<T>, new()` (`new()` altijd als laatste).

---

## 14. Virtual / abstract

| | `virtual` | `abstract` |
|---|---|---|
| Body? | ja (standaardgedrag) | **nee** |
| Override in subclass | **mag** | **moet** |
| Class moet abstract zijn? | nee | **ja** |

```csharp
public abstract class Vorm
{
    public string Naam { get; set; }
    public abstract double Oppervlakte();                 // geen body, verplicht override
    public virtual string Beschrijf() => $"{Naam}: {Oppervlakte():F2}";   // mag override
}
public class Cirkel : Vorm
{
    public double Straal { get; set; }
    public override double Oppervlakte() => Math.PI * Straal * Straal;
}
```

- `new Vorm()` → ❌ abstract class is niet te instantiëren.
- Abstract class mag wél fields, constructors en gewone methods hebben.
- Vergeet subclass een abstract method → compileerfout (tenzij subclass zelf ook abstract is).

---

## 15. Polymorfisme & dynamic dispatch

**Polymorfisme** = één variabele/lijst van base-type, elk object gedraagt zich volgens zijn **eigen** type.

**Dynamic dispatch (late binding)** = bij `virtual`/`abstract` wordt **tijdens runtime** gekozen welke implementatie draait, op basis van het **werkelijke object**, niet het variabele-type.

```csharp
List<Vorm> vormen = new() { new Cirkel { Straal = 1 }, new Rechthoek { B = 2, H = 3 } };
foreach (Vorm v in vormen)
    Console.WriteLine(v.Oppervlakte());   // elk object eigen versie

double totaal = vormen.Sum(v => v.Oppervlakte());
```

**Open/Closed principle**: open voor uitbreiding, gesloten voor wijziging → nieuwe vorm = nieuwe class, bestaande code blijft ongewijzigd.

```csharp
// ❌ Code smell: type-checks i.p.v. polymorfisme
if (v is Cirkel c) ... else if (v is Rechthoek r) ...
// ✅ Gewoon: v.Oppervlakte();
```

---

## 16. Upcasting / downcasting

```csharp
Cirkel c = new Cirkel();
Vorm v = c;                 // UPcasting: impliciet, altijd veilig (Cirkel IS een Vorm)
// v.Straal                 ❌ via Vorm-variabele zie je alleen Vorm-members

Cirkel c1 = (Cirkel)v;      // DOWNcasting: expliciet, InvalidCastException als het fout is
Cirkel? c2 = v as Cirkel;   // null als het mislukt (geen exception)
if (v is Cirkel c3)         // ✅ veiligst: check + cast in één
    Console.WriteLine(c3.Straal);
```

| Manier | Mislukt → |
|---|---|
| `(Cirkel)v` | `InvalidCastException` |
| `v as Cirkel` | `null` (alleen voor reference types) |
| `v is Cirkel c` | `false`, geen exception |

Upcasting verandert het **object** niet, alleen hoe je ernaar kijkt. Overridden methods blijven de subclass-versie gebruiken.

---

## 17. Interfaces

**Interface = contract**: beschrijft **wat** een class kan, niet **hoe**. Naam begint met `I`.

```csharp
public interface IOpslag
{
    void Opslaan(string sleutel, string waarde);
    string Laden(string sleutel);
}

public class GeheugenOpslag : IOpslag
{
    private readonly Dictionary<string, string> _data = new();
    public void Opslaan(string k, string w) => _data[k] = w;   // public, GEEN override
    public string Laden(string k) => _data[k];
}
```

- Class moet **alle** members **`public`** implementeren; geen `override` nodig.
- Kan geen object van maken: `new IOpslag()` ❌, wel `IOpslag o = new GeheugenOpslag();` ✅

### Multiple interfaces

```csharp
public class Factuur : IPrintbaar, IExporteerbaar { ... }
public class Eend : Dier, IVliegbaar, IZwembaar { ... }   // base class EERST, dan interfaces
```

`List<IPrintbaar>` kan alles bevatten wat `IPrintbaar` implementeert (Factuur én Rapport).

### Explicit implementation (twee interfaces, zelfde methodnaam)

```csharp
public class Robot : IVliegbaar, IRijdbaar
{
    void IVliegbaar.Start() => Console.WriteLine("Opstijgen");
    void IRijdbaar.Start()  => Console.WriteLine("Wegrijden");
}
Robot r = new Robot();
// r.Start();                 ❌ niet direct aanroepbaar
((IVliegbaar)r).Start();      // "Opstijgen"
```

Expliciete implementatie: geen access modifier, alleen bereikbaar via de interface.

---

## 18. Interface vs abstract class

| | Interface | Abstract class |
|---|---|---|
| Hoeveel per class | **meerdere** | maar **één** |
| Fields / state | ❌ | ✅ |
| Constructors | ❌ | ✅ |
| Implementatie (code) | nee (behalve default methods C# 8+) | ✅ gedeelde code |
| Access modifiers members | standaard `public` | alles mogelijk |
| Relatie | "**kan** iets" (can-do) | "**is een**" (is-a) |
| Gebruik bij | ongerelateerde classes met zelfde vermogen | verwante classes met gedeelde code/state |

**Vuistregel:** begin met een interface; kies abstract class als subclasses echt code/state delen. Combineren kan: `abstract class Rekening : IRekening`.

Voorbeelden:
- `ISorteerbaar` voor `Student` én `Product` → **interface** (niet verwant)
- `Spaarrekening` + `Betaalrekening` met gedeeld saldo/IBAN → **abstract class**
- `ILogger` met `ConsoleLogger`, `FileLogger` → **interface**

---

## 19. Dependency inversion & injection

**Dependency Inversion Principle (de D van SOLID):**
1. High-level modules hangen niet af van low-level modules; beide hangen af van **abstracties** (interfaces).
2. Abstracties hangen niet af van details; details hangen af van abstracties.

```csharp
// ❌ FOUT: hard gekoppeld
public class OrderService
{
    private readonly EmailSender _sender = new EmailSender();   // 'new' = vaste afhankelijkheid
}

// ✅ GOED: afhankelijk van interface + Dependency Injection
public interface IMessageSender { void Send(string bericht); }
public class EmailSender : IMessageSender { public void Send(string b) => Console.WriteLine($"Mail: {b}"); }
public class SmsSender   : IMessageSender { public void Send(string b) => Console.WriteLine($"SMS: {b}"); }

public class OrderService
{
    private readonly IMessageSender _sender;
    public OrderService(IMessageSender sender) => _sender = sender;   // constructor injection
    public void Plaats() => _sender.Send("Order geplaatst");
}

// Samenstellen (composition root, bijv. in Main):
var service = new OrderService(new SmsSender());
```

**Dependency Injection** = een class maakt zijn afhankelijkheden niet zelf (`new`), maar krijgt ze van buitenaf.

| Soort DI | Hoe |
|---|---|
| **Constructor injection** (meest gebruikt) | via constructor-parameter |
| Property injection | via een publieke property |
| Method injection | als parameter van een method |

**Voordelen:** testbaar (nep-implementatie meegeven, bijv. `NepDatabase`), uitwisselbaar zonder class te wijzigen, losse koppeling.

ASP.NET: DI-container → `builder.Services.AddScoped<IMessageSender, EmailSender>();`

**DIP = het principe, DI = de techniek om het toe te passen.**

---

## 20. Exceptions

### Hiërarchie (catch is polymorf!)

```
System.Exception
├── SystemException
│   ├── ArgumentException
│   │   ├── ArgumentNullException
│   │   └── ArgumentOutOfRangeException
│   ├── InvalidOperationException
│   ├── NullReferenceException
│   ├── IndexOutOfRangeException
│   ├── InvalidCastException
│   ├── FormatException
│   ├── ArithmeticException
│   │   └── DivideByZeroException
│   └── IOException → FileNotFoundException
└── (jouw custom exceptions)
```

| Situatie | Exception |
|---|---|
| Verkeerd argument (bijv. negatief bedrag) | `ArgumentException` / `ArgumentOutOfRangeException` |
| Argument is `null` | `ArgumentNullException` |
| Actie mag niet in huidige toestand (bijv. fiets al verhuurd) | `InvalidOperationException` |
| Member aanroepen op `null` | `NullReferenceException` |
| `int.Parse("abc")` | `FormatException` |
| `10 / 0` (int) | `DivideByZeroException` (bij `double` → `∞`, geen exception!) |
| `arr[10]` buiten array | `IndexOutOfRangeException` |
| `list[10]` buiten List | `ArgumentOutOfRangeException` |
| Foute cast | `InvalidCastException` |
| Bestand bestaat niet | `FileNotFoundException` |

Properties: `ex.Message`, `ex.StackTrace`, `ex.InnerException`.

### try / catch / finally

```csharp
try
{
    // code die mis kan gaan
}
catch (DivideByZeroException)      // SPECIFIEK eerst
{
}
catch (Exception ex)               // ALGEMEEN laatst (anders compileerfout)
{
    Console.WriteLine(ex.Message);
}
finally
{
    // draait ALTIJD: ook na return of onafgevangen exception → opruimen
}
```

- Na een exception in `try` wordt de **rest van `try` overgeslagen**.
- Slechts **één** catch-blok wordt uitgevoerd (de eerste die past).
- Exception filter: `catch (Exception ex) when (ex.Message.Contains("x"))`
- `using` = automatisch `Dispose()` (verkorte try/finally).

**Voorbeeld trace** (`A` → parse → deel → `B`, catch C=DivideByZero, D=Exception, finally E):

| Invoer | Output |
|---|---|
| `"5"` | `A 2 B E` |
| `"0"` | `A C E` |
| `"x"` | `A D E` (FormatException → algemene catch) |

### Throwen

```csharp
if (bedrag <= 0)
    throw new ArgumentOutOfRangeException(nameof(bedrag), "Bedrag moet positief zijn");

try { ... }
catch (Exception ex)
{
    _logger.LogError(ex, "Fout");
    throw;        // ✅ rethrow: originele stack trace blijft behouden
    // throw ex;  // ❌ stack trace wordt gereset naar deze regel
}

// Wrappen met context:
catch (IOException ex)
{
    throw new OpslagException("Kon niet opslaan", ex);   // ex wordt InnerException
}
```

### Custom exceptions

```csharp
public class FietsNietGevondenException : Exception
{
    public int FietsId { get; }

    public FietsNietGevondenException(int fietsId)
        : base($"Fiets {fietsId} niet gevonden")
    {
        FietsId = fietsId;
    }

    public FietsNietGevondenException(string message, Exception inner)
        : base(message, inner) { }
}
```

Regels: erf van `Exception`, naam eindigt op `Exception`, geef een duidelijke message mee via `base(...)`, voeg evt. extra properties toe. Maak er alleen één als het **domeinspecifiek** is en iemand er apart op wil kunnen catchen.

---

## 21. Logging

Logging legt vast wat een applicatie doet → terugkijken bij fouten in productie (`Console.WriteLine` is niet persistent, niet filterbaar, geen niveaus).

| Niveau | Gebruik |
|---|---|
| `Trace` | extreem gedetailleerd |
| `Debug` | info voor ontwikkelaars |
| `Information` | normale gebeurtenissen (bijv. "Fiets 12 verhuurd") |
| `Warning` | onverwacht, maar app werkt door (bijv. ongeldige invoer, retry) |
| `Error` | actie mislukt (exception) |
| `Critical` | app/systeem ligt plat |

```csharp
public class VerhuurService
{
    private readonly ILogger<VerhuurService> _logger;
    public VerhuurService(ILogger<VerhuurService> logger) => _logger = logger;   // via DI

    public void Verhuur(int id)
    {
        _logger.LogInformation("Fiets {FietsId} verhuurd", id);   // structured logging
        try { ... }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Verhuur van fiets {FietsId} mislukt", id);
            throw;
        }
    }
}
```

Richtlijnen: juiste niveau kiezen, exception-object meegeven (`LogError(ex, ...)`), **geen gevoelige data** (wachtwoorden, BSN), gebruik placeholders `{Naam}` i.p.v. string-interpolatie.

---

## 22. Exception anti-patterns

| Anti-pattern | Waarom slecht | Beter |
|---|---|---|
| Leeg catch-blok `catch { }` | fout verdwijnt spoorloos ("swallowing") | loggen en/of opnieuw gooien |
| Overal `catch (Exception)` | vangt ook bugs die je niet verwacht | specifieke exceptions vangen |
| `throw ex;` | stack trace kwijt | `throw;` |
| Exceptions voor normale flow | traag en onduidelijk | `if`-check, `int.TryParse(...)` |
| Log-and-throw op elk niveau | dezelfde fout N keer in de log | op één plek loggen |
| `throw new Exception("fout")` | te algemeen, niet specifiek te vangen | specifiek/custom type |
| Vage message | niemand weet wat er mis is | context meegeven (id, waarde) |
| Exception catchen en `null` teruggeven | probleem verschuift naar later (NullReference) | exception laten doorgaan |

```csharp
// ✅ Geen exception voor normale flow
if (int.TryParse(invoer, out int getal)) { ... } else { Console.WriteLine("Geen geldig getal"); }
```

---

## 23. Design patterns

**Design pattern** = herbruikbare, bewezen oplossing voor een veelvoorkomend ontwerpprobleem. Geen kant-en-klare code maar een **sjabloon**. Voordelen: gedeelde woordenschat, bewezen, onderhoudbaar/flexibel. Let op: niet overal patterns forceren (overengineering).

| Soort | Gaat over | Patterns |
|---|---|---|
| **Creational** | hoe objecten **gemaakt** worden | **Factory**, Singleton, Builder |
| **Structural** | hoe classes/objecten **samengesteld** worden | **Adapter**, Decorator, Facade |
| **Behavioral** | hoe objecten **communiceren** / verantwoordelijkheid verdelen | **Strategy**, **Observer**, Command |

### Strategy (behavioral)
**Probleem:** algoritme moet uitwisselbaar zijn, anders lange `if/switch`. **Oplossing:** elk algoritme in eigen class achter een interface; de *context* krijgt er één (via DI) en kan **tijdens runtime** wisselen.

```csharp
public interface IKortingStrategie { decimal Bereken(decimal prijs); }
public class GeenKorting      : IKortingStrategie { public decimal Bereken(decimal p) => p; }
public class StudentenKorting : IKortingStrategie { public decimal Bereken(decimal p) => p * 0.8m; }

public class Kassa                                    // context
{
    private IKortingStrategie _strategie;
    public Kassa(IKortingStrategie s) => _strategie = s;
    public void ZetStrategie(IKortingStrategie s) => _strategie = s;   // wisselen
    public decimal Afrekenen(decimal p) => _strategie.Bereken(p);
}
```

### Factory (creational)
**Probleem:** `new` met keuzelogica overal verspreid. **Oplossing:** één plek die beslist welk object gemaakt wordt; caller kent alleen de interface.

```csharp
public static class BetaalmethodeFactory
{
    public static IBetaalmethode Maak(string type) => type switch
    {
        "ideal"      => new IdealBetaling(),
        "creditcard" => new CreditcardBetaling(),
        _ => throw new ArgumentException($"Onbekend type: {type}")
    };
}
IBetaalmethode b = BetaalmethodeFactory.Maak("ideal");
```
Nieuw type (bijv. `"paypal"`) → alleen de factory + nieuwe class aanpassen, de rest niet.
**Factory Method**-variant: base class met `abstract IProduct MaakProduct();` die subclasses invullen.

### Observer (behavioral)
**Probleem:** meerdere objecten moeten reageren als één object verandert, zonder harde koppeling. **Oplossing:** *subject* houdt lijst van *observers* bij en meldt wijzigingen.

```csharp
public interface IObserver { void Update(string product, int nieuw); }

public class Voorraad                                  // subject
{
    private readonly List<IObserver> _observers = new();
    public void Aanmelden(IObserver o) => _observers.Add(o);
    public void Afmelden(IObserver o)  => _observers.Remove(o);
    public void Wijzig(string p, int n)
    {
        foreach (var o in _observers) o.Update(p, n);   // notify
    }
}

// C#-manier met events:
public class Voorraad2
{
    public event Action<string, int>? VoorraadGewijzigd;
    public void Wijzig(string p, int n) => VoorraadGewijzigd?.Invoke(p, n);
}
voorraad.VoorraadGewijzigd += (p, n) => Console.WriteLine($"{p}: {n}");   // aanmelden
// -= om af te melden
```
Observers worden aangeroepen in **volgorde van aanmelden**; afgemelde observers krijgen niks meer.

### Adapter (structural)
**Probleem:** bestaande class (bijv. externe library) heeft een interface die niet past bij wat jouw code verwacht. **Oplossing:** adapter-class implementeert jouw interface en vertaalt naar de bestaande class.

```csharp
public interface IPrinter { void Print(string tekst); }      // wat onze code verwacht

public class OudePrinter                                       // bestaand, niet aan te passen
{
    public void PrintXml(string xml) => Console.WriteLine(xml);
}

public class PrinterAdapter : IPrinter                         // adapter
{
    private readonly OudePrinter _oud;
    public PrinterAdapter(OudePrinter oud) => _oud = oud;      // composition (has-a)
    public void Print(string tekst) => _oud.PrintXml($"<tekst>{tekst}</tekst>");
}

IPrinter p = new PrinterAdapter(new OudePrinter());
```

### Welk pattern?

| Situatie | Pattern |
|---|---|
| Gedrag/algoritme wisselen tijdens runtime | Strategy |
| Object maken op basis van input, zonder `new`-logica overal | Factory |
| Meerdere objecten op de hoogte brengen van een wijziging | Observer |
| Bestaande class past niet op jouw interface | Adapter |

**Strategy vs Factory:** Strategy = *hoe gedraagt* een object zich (gedrag wisselen). Factory = *welk* object wordt *gemaakt*. Vaak samen: factory maakt de juiste strategy.

---

## 24. Veelgemaakte fouten

| Fout | Oplossing |
|---|---|
| `public void Methode();` + `{ }` eronder | **geen `;`** na een methodekop |
| `public string Tijdstip(...)` als constructor | constructor heeft **geen return type** |
| `return som / n;` geeft 2 i.p.v. 2,5 | `(double)som / n` |
| `new Student()` compileert niet | default constructor verdwijnt als je zelf een constructor schrijft |
| Subclass-constructor zonder `: base(...)` | verplicht als base geen parameterloze constructor heeft |
| Method overriden zonder `virtual` in base | base-method `virtual`/`abstract` maken |
| `override` vergeten | warning → gedraagt zich als `new` (geen polymorfisme) |
| `catch (Exception)` vóór specifieke catch | specifiek eerst, algemeen laatst |
| `throw ex;` | `throw;` |
| Interface-method niet `public` | interface-members altijd `public` implementeren |
| `new AbstracteClass()` | abstract class niet instantieerbaar → subclass gebruiken |
| Array-index vanaf 1 | arrays beginnen bij **0**, laatste = `Length - 1` |
| `arr.Count` / `list.Length` | array → `.Length`, List → `.Count` |

---

## 25. Output-vragen: stappenplan

1. **Welk type heeft de variabele, welk type heeft het object?** (`Basis x = new Afgeleid();` → variabele `Basis`, object `Afgeleid`)
2. **Method `virtual` + `override`?** → versie van het **object**.
   **Geen `virtual` of met `new`?** → versie van het **variabele-type**.
3. **Constructors:** base eerst, dan subclass. Bij `: this(...)` eerst de aangeroepen constructor, dan de body van de huidige.
4. **Reference types:** `b = a` → zelfde object; wijziging via `b` zie je ook via `a`.
5. **Static field:** één waarde voor alle objecten.
6. **try/catch/finally:** na exception rest van `try` overslaan → eerste passende `catch` → `finally` altijd → code na de try loopt door (tenzij opnieuw gegooid).
7. **Overloading:** tel argumenten + kijk naar types → beste match. Binnen-naar-buiten bij geneste aanroepen.
8. **Integer-deling:** `int / int` = afgerond naar beneden (richting 0).
9. **Observer:** observers in volgorde van aanmelden; na afmelden geen melding meer.

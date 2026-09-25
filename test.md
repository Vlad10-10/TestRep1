# MSTest – testowanie jednostkowe w C#

## 1. Czym jest test jednostkowy?

**Test jednostkowy (unit test)** to test, który sprawdza działanie małego fragmentu programu, najczęściej pojedynczej metody lub klasy.

Jego zadaniem jest sprawdzenie, czy kod działa zgodnie z oczekiwaniami.

Przykład: jeśli mamy metodę:

```csharp
public int Add(int a, int b)
{
    return a + b;
}
```

możemy napisać test sprawdzający, czy `Add(2, 3)` zwraca `5`.

Testy jednostkowe pomagają:

* wykrywać błędy,
* sprawdzać, czy zmiany w kodzie nie zepsuły istniejących funkcji,
* łatwiej utrzymywać i rozwijać program.

---

## 2. Czym jest MSTest?

**MSTest** to framework do tworzenia i uruchamiania testów jednostkowych dla aplikacji .NET i C#.

Pozwala między innymi:

* tworzyć metody testowe,
* sprawdzać wyniki działania kodu za pomocą asercji,
* przygotowywać środowisko przed testami,
* wykonywać czynności po zakończeniu testów,
* sprawdzać, czy program zgłasza oczekiwane wyjątki.

---

# 3. Podstawowe elementy MSTest

## `[TestClass]`

Oznacza klasę, która zawiera testy.

```csharp
[TestClass]
public class CalculatorTests
{
}
```

## `[TestMethod]`

Oznacza metodę, która jest pojedynczym testem.

```csharp
[TestMethod]
public void Add_TwoNumbers_ReturnsSum()
{
    // test
}
```

---

## 4. Arrange – Act – Assert

Test jednostkowy często tworzy się według schematu **AAA**:

### Arrange

Przygotowanie danych i obiektów potrzebnych do testu.

### Act

Wykonanie testowanej operacji.

### Assert

Sprawdzenie, czy otrzymany wynik jest prawidłowy.

Przykład:

```csharp
[TestMethod]
public void Add_TwoNumbers_ReturnsSum()
{
    // Arrange
    Calculator calculator = new Calculator();

    // Act
    int result = calculator.Add(2, 3);

    // Assert
    Assert.AreEqual(5, result);
}
```

---

# 5. Asercje

**Asercja (`Assert`)** służy do sprawdzania, czy wynik testu jest zgodny z oczekiwaniami.

### `Assert.AreEqual`

Sprawdza, czy dwie wartości są takie same.

```csharp
Assert.AreEqual(5, result);
```

### `Assert.IsTrue`

Sprawdza, czy warunek jest prawdziwy.

```csharp
Assert.IsTrue(result > 0);
```

### `Assert.IsNull`

Sprawdza, czy wartość jest `null`.

```csharp
Assert.IsNull(result);
```

Istnieją również między innymi `Assert.IsFalse`, `Assert.IsNotNull` oraz `Assert.AreNotEqual`.

---

# 6. Atrybuty MSTest

## `[TestInitialize]`

Metoda oznaczona tym atrybutem uruchamia się **przed każdym testem**.

```csharp
[TestInitialize]
public void Setup()
{
    // przygotowanie danych
}
```

Przydaje się, gdy każdy test wymaga takiego samego przygotowania.

---

## `[TestCleanup]`

Uruchamia się **po każdym teście**.

```csharp
[TestCleanup]
public void Cleanup()
{
    // sprzątanie po teście
}
```

---

## `[ClassInitialize]`

Uruchamia się **raz przed wszystkimi testami w danej klasie**.

```csharp
[ClassInitialize]
public static void ClassSetup(TestContext context)
{
    // przygotowanie wspólne dla klasy
}
```

## `[ClassCleanup]`

Uruchamia się **po zakończeniu wszystkich testów w klasie**.

```csharp
[ClassCleanup]
public static void ClassCleanup()
{
    // sprzątanie
}
```

---

## `[Ignore]`

Pozwala tymczasowo pominąć test.

```csharp
[TestMethod]
[Ignore("Test wymaga poprawy")]
public void MyTest()
{
}
```

Taki test nie zostanie wykonany.

---

## `[ExpectedException]`

Służy do sprawdzania, czy wykonanie kodu spowoduje oczekiwany wyjątek.

```csharp
[TestMethod]
[ExpectedException(typeof(ArgumentException))]
public void TestInvalidArgument()
{
    throw new ArgumentException();
}
```

Test przejdzie, jeśli podczas jego wykonywania zostanie rzucony `ArgumentException`.

---

# 7. Przykład kompletnego testu

Kod testowanej klasy:

```csharp
public class Calculator
{
    public int Add(int a, int b)
    {
        return a + b;
    }
}
```

Test:

```csharp
[TestClass]
public class CalculatorTests
{
    [TestMethod]
    public void Add_TwoNumbers_ReturnsSum()
    {
        // Arrange
        Calculator calculator = new Calculator();

        // Act
        int result = calculator.Add(2, 3);

        // Assert
        Assert.AreEqual(5, result);
    }
}
```

### Najważniejsze do zapamiętania

* **Test jednostkowy** sprawdza mały fragment kodu.
* **MSTest** jest frameworkiem do tworzenia testów w C#.
* `[TestClass]` oznacza klasę z testami.
* `[TestMethod]` oznacza konkretny test.
* **Arrange → Act → Assert** to podstawowa struktura testu.
* `Assert` służy do sprawdzania wyników.
* `[TestInitialize]` i `[TestCleanup]` działają przed i po każdym teście.
* `[ClassInitialize]` i `[ClassCleanup]` działają raz dla całej klasy.
* `[Ignore]` pomija test.
* `[ExpectedException]` pozwala testować oczekiwane wyjątki.

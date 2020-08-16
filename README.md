# Mejores prácticas en PHP utilizadas en Tiendanube.

Este documento es una referencia para los desarrolladores de Tiendanube y para la comunidad de PHP en general y tiene como objetivo ayudar y guiar a los mismos a escribir código limpio.

# Tabla de Contenidos

- [Introducción](#introduccin)
    - [¿Por qué seguir buenas prácticas?](#por-qu-seguir-buenas-prcticas)
- [Clean Code](#clean-code)
  - [Naming](#naming)
  - [Variables](#variables)
  - [Métodos y Funciones](#mtodos-y-funciones)
  - [Condicionales](#condicionales)
  - [Clases y Objetos](#clases-y-objetos)
- [Principios del Diseño de Software](#principios-del-diseo-de-software)
  - [SOLID](#solid)
  - [Principio DRY](#principio-dry)
  - [Principio Tell don't ask](#principio-tell-dont-ask)
- [Testing](#testing)

# Introducción

![Humorous image of software quality estimation as a count of how many expletives you shout when reading code](http://www.osnews.com/images/comics/wtfm.jpg)

En esta guia recopilamos varios principios de la Ingeniería de Software introducidos en diferentes libros y experiencias propias que creemos que ayudarán a escribir mejor código.

No todos los principios deben seguirse estrictamente. Y puede que no estes de acuerdo con todos ellos. Si es así, eres bienvenido a enviar tu solicitud de cambio.

Tener en cuenta que la mayoría de los ejemplos en este artículo funcionan solo en PHP 7+.

Inspirado en [clean-code-dotnet](https://github.com/thangchung/clean-code-dotnet/blob/master/README.md).

## ¿Por qué seguir buenas prácticas?
Como desarrolladores, a veces nos sentimos tentados a escribir código de cierta manera que sea conveniente para terminar en el momento sin tener en cuenta las mejores prácticas. Esto dificulta las revisiones y las pruebas de código. 
Podríamos decir que estamos _codificando_ y, al hacerlo, hacemos más difícil para otros _decodificar_ nuestro trabajo. Pero queremos que nuestro código sea reutilizable, legible y mantenible. Y eso requiere desarrollar de la manera correcta, no de la manera fácil.

# Clean Code

## Naming

<details>
  <summary><b>Evite usar nombres malos</b></summary>
Un buen nombre permite que los desarrolladores entiendan fácilmente el código. El nombre debe reflejar lo que hace y dar contexto.

**Mal:**

```php
$ymdstr = $moment->format('y-m-d');
```

**Bien:**

```php
$currentDate = $moment->format('y-m-d');
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Usar notación Camel Case</b></summary>

Usar la [notación CamelCase](https://en.wikipedia.org/wiki/Camel_case) para variables, clases, métodos y funciones.

**Mal:**

```php
$customerphone = '1134971828';

public function calculate_product_price(int $productid, int $quantity)
{
    // some logic
}
```

**Bien:**

```php
$customerPhone = '1134971828';

public function calculateProductPrice(int $productId, int $quantity)
{
    // some logic
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

## Variables

<details>
  <summary><b>Evitar valores magicos</b></summary>

Los valores mágicos son valores constantes que se especifican directamente dentro del código. Con frecuencia, dichos valores terminan duplicados dentro del sistema y se convierten en una fuente común de errores cuando se realizan cambios en algunos valores pero no en otros. 

**Mal:**

```php
class Payment 
{
    public function isPending()
    {
        if ($this->status === "pending") 
        { 
            // lógica aquí 
        }
    }
}
```

**Bien:**

```php
class Payment 
{
    const STATUS_PENDING = 'pending';

    public function isPending()
    {
        if ($this->status === self::STATUS_PENDING) 
        { 
            // lógica aquí 
        }
    }
}
```
De esta manera solo tenemos que cambiar en un lugar centralizado.`

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Usar variables explicativas</b></summary>

No fuerce al lector de su código a traducir lo que significa la variable. **Explícito es mejor que implícito**.

**Mal:**

```php
$address = 'Avenida Rivadavia 1678, 1406';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{4})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches[1], $matches[2]);
```

**No tan mal:**

```php
$address = 'Avenida Rivadavia 1678, 1406';
$cityZipCodeRegex = '/^[^,]+,\s*(.+?)\s*(\d{4})$/';
preg_match($cityZipCodeRegex, $address, $matches);

[, $street, $zipCode] = $matches;
saveCityZipCode($street, $zipCode);
```

**Bien:**
Al utilizar subpatrones es más fácil entender la expresión regular.
```php
$address = 'Avenida Rivadavia 1678, 1406';
$cityZipCodeRegex = '/^[^,]+,\s*(?<street>.+?)\s*(?<zipCode>\d{4})$/';
preg_match($cityZipCodeRegex, $address, $matches);

saveCityZipCode($matches['street'], $matches['zipCode']);
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar anidación profunda.</b></summary>

Debemos minimizar al máximo los niveles de indentación. Muchas declaraciones if-else anidadas pueden hacer tu código difícil de seguir. 

**Bad**

```php
function isShopOpen($day): bool
{
    if ($day) {
        if (is_string($day)) {
            $day = strtolower($day);
            if ($day === 'friday') {
                return true;
            } elseif ($day === 'saturday') {
                return true;
            } elseif ($day === 'sunday') {
                return true;
            } else {
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}
```

**Good**

```php
function isShopOpen(string $day): bool
{
    if (empty($day)) {
        return false;
    }

    $openingDays = [
        'friday', 'saturday', 'sunday'
    ];

    return in_array(strtolower($day), $openingDays, true);
}
```

Using this we only have to change in centralize place and others will adapt it.

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar utilizar sentencia else.</b></summary>

La sentencia `else` puede dificultar la lectura del código si la sentencia `if` contiene demasiado código. 

**Bad**

```php
if ($product->hasStock()) {
    // código...
} else {
    return false;
}
```

**Good**

Para evitar utilizar la sentencia `else` podemos utilizar Cláusulas de Guarda.
La cláusula de guarda es una técnica sensilla que consiste en negar la condición de la consulta y eliminar la sentencia `else`.
```php
if (!$product->hasStock()) {
    return false;
}

// código...
```
**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>No agregar contexto innecesario</b></summary>

Si el nombre de tu clase/objeto te dice algo, no lo repitas en el nombre del atributo.

**Mal:**

```php
class Product
{
    private $productPrice;
    private $productStock;
    private $productDescription;

    //...
}
```

**Bien:**

```php
class Product
{
    private $price;
    private $stock;
    private $description;

    //...
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

## Métodos y Funciones

<details>
  <summary><b>Declaración de tipos en parámetros y retornos</b></summary>
  
  Desde PHP 7 podemos utilizar la declaración de tipos para los argumentos de entrada y salida de nuestras funciones y métodos. 

**Mal:**

```php
function sum($val1, $val2)
{
    return $val1 + $val2;
}
```

**Bien:**

```php
function sum(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Argumentos de la función (idealmente no más de 2)</b></summary>

El número ideal de argumentos es cero, después uno (monádico) y después dos (diádico). Siempre debes evitar desarrollar funciones o métodos con tres o más argumentos (poliádico). Ya que, si tenes más de dos argumentos significa que estás intentando hacer demasiado en la función.
En los casos que necesitas establecer más de dos argumentos, es mejor utilizar un objeto como argumento. 

**Mal:**

```php
function createProduct(string $title, string $description, float $price, int $stock): void
{
    // ...
}
```

**Bien:**

```php
class ProductConfig
{
    private string $title;
    private string $description;
    private float $price;
    private int $stock;

    public function __construct(string $title, string $description, float $price, int $stock) 
    {
        $this->title = $title;
        $this->description = $description;
        $this->price = $price;
        $this->stock = $stock;
    }
}

$config = new ProductConfig('foo', 'bar', 10.0, 100);
createProduct($config);

function createProduct(ProductConfig $config): void
{
    // ...
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Las funciones deben hacer una cosa</b></summary>

Cuando las funciones hacen más de una acción, se vuelven difíciles de razonar y probar. Cuando puedes aislar una función en una sola acción, ellas pueden ser refactorizadas con facilidad y tu código será mucho más fácil de leer y reutilizar.

**Mal:**

```php
function emailClients(array $clients): void
{
    foreach ($clients as $client) {
        $clientRecord = $db->find($client);
        if ($clientRecord->isActive()) {
            email($client);
        }
    }
}
```

**Bien:**

```php
function emailClients(array $clients): void
{
    $activeClients = activeClients($clients);
    array_walk($activeClients, 'email');
}

function activeClients(array $clients): array
{
    return array_filter($clients, 'isClientActive');
}

function isClientActive(int $client): bool
{
    $clientRecord = $db->find($client);

    return $clientRecord->isActive();
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Tamaño reducido</b></summary>

La primera regla de las funciones es que deben ser de tamaño reducido. La segunda regla es que deben ser todavía más reducidas.
La experiencia cuenta que las funciones de más de 20 líneas de código aproximadamente, esconden más de una acción.

Los programas que cuentan con funciones de pocas líneas de código, son obvias. Cada función cuenta una historia y cada una lleva a la siguiente de una forma natural. ¡A eso debes aspirar!

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Los nombres de las funciones deben indicar lo que hacen</b></summary>

**Mal:**

```php
class Email
{
    //...

    public function handle(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// ¿Qué es esto? ¿Un "manejador" para los mensajes? 🤔
$message->handle();
```

**Bien:**

```php
class Email 
{
    //...

    public function send(): void
    {
        mail($this->to, $this->subject, $this->body);
    }
}

$message = new Email(...);
// Limpio y obvio
$message->send();
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Las funciones deben tener sólo un nivel de abstracción</b></summary>
  
Cuando tienes más de un nivel de abstracción usualmente es porque tu función está haciendo demasiado. Separarlas en funciones lleva a la reutilización y facilita las pruebas.

**Mal:**

```php
function parseBetterJSAlternative(string $code): void
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            // ...
        }
    }

    $ast = [];
    foreach ($tokens as $token) {
        // lex...
    }

    foreach ($ast as $node) {
        // convertir...
    }
}
```

**También mal:**

Hemos separado algunas de las funcionalidades, pero la función parseBetterJSAlternative() todavía es muy compleja e imposible de probar.

```php
function tokenize(string $code): array
{
    $regexes = [
        // ...
    ];

    $statements = explode(' ', $code);
    $tokens = [];
    foreach ($regexes as $regex) {
        foreach ($statements as $statement) {
            $tokens[] = /* ... */;
        }
    }

    return $tokens;
}

function lexer(array $tokens): array
{
    $ast = [];
    foreach ($tokens as $token) {
        $ast[] = /* ... */;
    }

    return $ast;
}

function parseBetterJSAlternative(string $code): void
{
    $tokens = tokenize($code);
    $ast = lexer($tokens);
    foreach ($ast as $node) {
        // convertir...
    }
}
```

**Bien:**

Lo mejor es sacar las dependencias de la función `parseBetterJSAlternative()`.

```php
class Tokenizer
{
    public function tokenize(string $code): array
    {
        $regexes = [
            // ...
        ];

        $statements = explode(' ', $code);
        $tokens = [];
        foreach ($regexes as $regex) {
            foreach ($statements as $statement) {
                $tokens[] = /* ... */;
            }
        }

        return $tokens;
    }
}

class Lexer
{
    public function lexify(array $tokens): array
    {
        $ast = [];
        foreach ($tokens as $token) {
            $ast[] = /* ... */;
        }

        return $ast;
    }
}

class BetterJSAlternative
{
    private $tokenizer;
    private $lexer;

    public function __construct(Tokenizer $tokenizer, Lexer $lexer)
    {
        $this->tokenizer = $tokenizer;
        $this->lexer = $lexer;
    }

    public function parse(string $code): void
    {
        $tokens = $this->tokenizer->tokenize($code);
        $ast = $this->lexer->lexify($tokens);
        foreach ($ast as $node) {
            // convertir...
        }
    }
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>No usar flags como parámetros de funciones</b></summary>
  
Las banderas le dicen al usuario que la función hace más de una cosa. Divide tus funciones si siguen diferentes caminos basados en un valor booleano.

**Mal:**

```php
function createFile(string $name, bool $temp = false): void
{
    if ($temp) {
        touch('./temp/'.$name);
    } else {
        touch($name);
    }
}
```

**Bien:**

```php
function createFile(string $name): void
{
    touch($name);
}

function createTempFile(string $name): void
{
    touch('./temp/'.$name);
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar efectos secundarios</b></summary>

Una función produce un efecto secundario si hace algo más que tomar un valor y devolver otros. Un efecto secundario puede ser escribir en un archivo, modificar alguna variable global, o accidentalmente darle todo tu dinero a un extraño.

Ahora, ocasionalmente necesitaras los efectos secundarios en un programa. Como los ejemplos anteriores, necesitarás escribir en un archivo. Lo que quieres hacer en esos casos es centralizar donde realizarlos. No tengas muchas funciones y clases que escriban un archivo en particular. Ten un servicio que lo haga. Uno y sólo uno.

El punto principal es evitar trampas comunes como compartir estados entre objetos sin alguna estructura, usar tipos de datos mutables que puedan ser escritos por cualquiera, y no centralizar donde el efecto paralelo ocurre. Si puedes hacerlo, serás más feliz que la vasta mayoría de los demás programadores.

**Mal:**

```php
function createUser(string $name, string $email, string $password)
{
    $user = new User($name, $email, $password);
    $user->create();
    $message = new Email($email);
    $message->send();
}
```

**Bien:**

```php
function createUser(string $name, string $email, string $password)
{
    $user = new User($name, $email, $password);
    $user->create();
}

function sendEmail($email)
{
    $message = new Email($email);
    $message->send();
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar retornar null</b></summary>

Al retornar `null` podríamos estar obligando al cliente de la función o método a realizar una validación adicional.
Basta con que falte la comprobación de `null` para que la aplicación pierda el control.

Si siente la tentación de devolver `null`, pruebe con lanzar una excepción, devolver una estructura vacía o un objeto vacío ([Null Object Pattern](https://en.wikipedia.org/wiki/Null_object_pattern)). 

**Mal:**

```php
function getItems()
{
    //...

    return null;
}

$items = getItems();
if ($items !== null) {
    foreach ($items as $item) {
        // ...
    }
}
```

**Bien:**

```php
function getItems()
{
    //...

    return [];
}

$items = getItems();
foreach ($items as $item) {
    // ...
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

## Condicionales

<details>
  <summary><b>Evitar condicionales</b></summary>

Al tener declaraciones `if` dentro de nuestras funciones, estamos diciendo que nuestra función puede hacer más de una cosa.

  Podemos utilizar polimorfismo para evitar condicionales.

**Mal:**

```php
class Airplane
{
    public function getCruisingAltitude(): int
    {
        switch ($this->type) {
            case '777':
                return $this->getMaxAltitude() - $this->getPassengerCount();
            case 'Air Force One':
                return $this->getMaxAltitude();
            case 'Cessna':
                return $this->getMaxAltitude() - $this->getFuelExpenditure();
        }
    }
}
```

**Bien:**

```php
interface Airplane
{
    public function getCruisingAltitude(): int;
}

class Boeing777 implements Airplane
{
    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getPassengerCount();
    }
}

class AirForceOne implements Airplane
{
    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude();
    }
}

class Cessna implements Airplane
{
    public function getCruisingAltitude(): int
    {
        return $this->getMaxAltitude() - $this->getFuelExpenditure();
    }
}
```
**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar condicionales negativos</b></summary>

**Mal:**

```php
if (!haveNotPromotions($product))
{
    // ...
}
```

**Bien:**

```php
if (havePromotions($product))
{
    // ...
}
```
**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar verificación de tipo</b></summary>

Hacer verificaciones de tipos da una falsa "seguridad de tipado" además de afectar la legibilidad del código.

**Mal:**

```php
function travelToTexas($vehicle): void
{
    if ($vehicle instanceof Bicycle) {
        $vehicle->pedalTo(new Location('texas'));
    } elseif ($vehicle instanceof Car) {
        $vehicle->driveTo(new Location('texas'));
    }
}
```

**Bien:**

```php
function travelToTexas(Traveler $vehicle): void
{
    $vehicle->travelTo(new Location('texas'));
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Evitar revisión de tipos primitivos</b></summary>
  
  El punto anterior también aplica a los tipos primitivos. 

**Mal:**

```php
function combine($val1, $val2): int
{
    if (!is_numeric($val1) || !is_numeric($val2)) {
        throw new \Exception('Must be of type Number');
    }

    return $val1 + $val2;
}
```

**Bien:**

```php
function combine(int $val1, int $val2): int
{
    return $val1 + $val2;
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Encapsular condicionales</b></summary>
  
  Encapsula condiciones dentro de métodos o funciones brinda un unico punto de modificación, mayor cohesión y promueve la reutilización. 

**Mal:**

```php
if ($payment->status === 'pending') {
    // ...
}
```

**Bien:**

```php
if ($payment->isPending()) {
    // ...
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

## Clases y Objetos

<details>
  <summary><b>Clases finales por defecto</b></summary>
  
  Al momento de pensar y crear una clase debemos partir de la premisa que son clases finales. De esta forma promovemos la idea de que deben funcionar como una unidad del sistema.
  Permitiendo mayor entendimiento, mantenimiento y reutilización de la clase.
  
  **La visibilidad de los objetos se debe ir abriendo según evolucione el código o en casos que lo justifique.**

**No tan bien:**

```php
class Employee
{
    // código...
}
```

**Bien:**

```php
final class Employee
{
    // código...
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Propiedades privadas por defecto</b></summary>

La visibilidad de las propiedades debe ser `private` por defecto. Ya que el objeto no debe revelar nada de si mismo, 
excepto lo que sea necesario para que otras partes del sistema interactúen con él.

**Mal:**

```php
final class Product
{
    public float $price = 1000.00;
}
```

**Bien:**

```php
final class Product
{
    private float $price = 1000.00;
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Constructor sobre setters públicos</b></summary>
  
  Siempre debemos esconder los detalles de implementación de los objetos. 
  Para eso, debemos construir los objetos por medio de su constructor y no por setters públicos.

**Mal:**

```php
final class Product
{
    private string $description;
    private float $price;

    public function setDescription(string $value): void
    {
        $this->description = $value;
    }

    public function setPrice(float $value): void
    {
        $this->price = $value;
    }
}

$product = new Product();
$product->setDescription('Foo');
$product->setPrice(1000.00);
```

**Bien:**

```php
final class Product
{
    private string $description;
    private float $price;

    public function __construct(string $description, float $price)
    {
        $this->description = $description;
        $this->price = $price;
    }
}

$product = new Product('Foo', 1000.00);
```

**O también:**

Note que los setters son `private`. 
Esta forma es útil cuando deseas agregar lógica al momento de establecer una propiedad.

```php
final class Product
{
    private string $description;
    private float $price;

    public function __construct(string $description, float $price)
    {
        $this->setDescription($description);
        $this->setPrice($price);
    }
    
    private function setDescription(string $value): void
    {
        $this->description = $value;
    }

    private function setPrice(float $value): void
    {
        $this->price = $value;
    }   
}

$product = new Product('Foo', 1000.00);
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Preferir la composición sobre la herencia</b></summary>

Composición sobre herencia es un principio de diseño que permite una mayor flexibilidad.
Es más natural construir clases de dominio a partir de varios componentes 
que tratar de encontrar puntos en común entre ellos y crear un árbol genealógico, como así ocurre con la herencia.
La composición también proporciona un dominio del negocio más estable a largo plazo, 
ya que es menos propenso a las peculiaridades clases hijas de hijas de hijas.

**Mal:**

```php
class Vehicle
{    
    public function move()
    {
        echo "Move the car";
    }    
}

class Car extends Vehicle
{
    public function accelarate()
    {    
        $this->move();    
    }
}

$car = new Car();
$car->accelarate();
```
En este ejemplo existe un acoplamiento muy estrecho entre las clases `Vehicle` y `Car`.
i algo cambia en la clase `Vehicle`, específicamente en el método `move()`, la clase `Car` puede romperse fácilmente
ya que la superclase `Vehicle` no tiene idea de para qué la utilizan las clases hijas.

**Bien:**

```php
interface MoveVehicle
{
    public function move();
}

final class Vehicle implements MoveVehicle
{    
    public function move()
    {
        echo "Move the car";
    }    
}

final class Car
{
    private MoveVehicle $vehicle;

    public function __construct(MoveVehicle $vehicle)
    {
        $this->vehicle = $vehicle;
    }

    public function accelarate()
    {    
        $this->vehicle->move();    
    }
}

$vehicle = new Vehicle();
$car = new Car($vehicle);
$car->accelarate();
```

Ahora nuestra clase `Car` se compone de cualquier clase que implemente la interfaz. Esto brinda una gran flexibilidad y 
rompe con el acoplamiento estrecho que teníamos en la herencia.

**[⬆ Volver](#tabla-de-contenidos)**

</details>

# Principios del Diseño de Software

<details>
  <summary><b>¿Qué es el Diseño de Software?</b></summary>
  
  Los principios de diseño son un conjunto de diseños y buenas prácticas que se emplean en OOD y OOP (diseño y programación orientada a objetos).
  
  Robert C. Martin dice que la mayoría de nosotros usamos lenguajes orientados a objetos sin saber por qué, y sin saber cómo obtener el beneficio máximo de ellos. Por este motivo es que surgen estos patrones de diseño.
  
  Aplicar estos principios nos ayudaran, entre otras cosas, a crear un código que sea más legible, simple, reusable, escalable y fácil de mantener. Es decir, nos ayudaran a evitar los problemas con los que nos solemos encontrar a medida que nuestro código va creciendo.
  
  Algunos de ellos son:
</details>

## SOLID

<details>
  <summary><b>¿Qué es SOLID?</b></summary>
  
  SOLID es un acrónimo donde cada letra representa un principio del diseño orientado a objetos. 
  Los cinco principios están muy relacionados entre si y nos brindan distintas técnicas para crear código de alta calidad.

- [S: Principio de responsabilidad única (SRP)](#single-responsibility-principle-srp)
- [O: Principio abierto/cerrado (OCP)](#openclosed-principle-ocp)
- [L: Principio de sustitución de Liskov (LSP)](#liskov-substitution-principle-lsp)
- [I: Principio de la segregación de la interfaz (ISP)](#interface-segregation-principle-isp)
- [D: Principio de la inversión de dependencias (DIP)](#dependency-inversion-principle-dip)

</details>

<details>
  <summary><b>Principio de responsabilidad única (SRP)</b></summary>

El Principio de Responsabilidad Única (SRP) es la S del Principio SOLID y nos dice que una clase debe tener un único motivo 
por el cual debe ser modificada.

**Mal:**

```php
class Page 
{
    private string $title;
    
    public function title(): string 
    {
        return $this->title;
    }
    
    public function formatJson() 
    {
        return json_encode($this->title());
    }
}
```
Si necesitamos agregar otro tipo de formato debemos modificar la clase para agregar otro método. 
Eso está bien para una clase tan simple como esta, pero si la clase tuviera más propiedades, el formato sería más complejo de cambiar.

**Bien:**

```php
class Page 
{
    private string $title;
 
    public function title(): string
    {
        return $this->title;    
    }
}
 
class JsonPageFormatter 
{
    public function format(Page $page)
    {
        return  json_encode($page->title());
    }
}
```

Hacer esto significa que si quisiéramos crear un formato XML, simplemente podríamos agregar una clase llamada `XmlPageFormatter` 
y escribir un código simple para generar XML. Ahora solo tenemos una razón para cambiar la clase `Page`.

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Principio abierto/cerrado (OCP)</b></summary>

Las entidades de software (clases, módulos, funciones, etc) deben ser abiertas para ser extendidas, pero cerradas para modificarlas.
Este principio establece que puedes permitir a los usuarios agregar nuevas funcionalidades pero sin cambiar el código existente.

Parece complicado, no? Pero gracias a las interfaces y abstracciones se convierte en algo muy simple.

**Mal:**

```php
class Customer
{
    public function pay(float $total, CreditCardPayment $paymentMethod)
    {
        $paymentMethod->execute($total);
    }
}

class CreditCardPayment
{
    public function execute(float $total)
    {
        // lógica para pagar con tarjeta de crédito...
    }
}

$customer = new Customer();
$customer->pay(1000.00, new CreditCardPayment());
```

De esta forma, el cliente solamente puede pagar con tarjeta de crédito. Si quisiéramos agregar un nuevo método de pago 
deberíamos modificar el método `pay()`.

**Bien:**

```php
interface PaymentMethod
{
    public function execute(float $total);
}

class Customer
{
    public function pay(float $total, PaymentMethod $paymentMethod)
    {
        $paymentMethod->execute($total);
    }
}

class CreditCardPayment implements PaymentMethod
{
    public function execute(float $total)
    {
        // lógica para pagar con tarjeta de crédito...
    }
}

class CashPayment implements PaymentMethod
{
    public function execute(float $total)
    {
        // lógica para pagar con dinero...
    }
}

$customer = new Customer();
$customer->pay(1000.00, new CreditCardPayment());
// o
$customer->pay(1000.00, new CashPayment());
```

Gracias a la interfaz `PaymentMethod`, el método `pay()` quedo cerrado para modificaciones pero abierto a nuevas formas de pago.
**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Principio de sustitución de Liskov (LSP)</b></summary>

El principio de Liskov establece que si tienes una clase padre y una clase hija, entonces la clase padre y la clase hija 
pueden intercambiarse sin obtener resultados incorrectos.

**Mal:**

```php
class Rectangle
{
    // ...

    public function setWidth(int $width): void
    {
        $this->width = $width;
    }

    public function setHeight(int $height): void
    {
        $this->height = $height;
    }

    public function getArea(): int
    {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle
{
    //... 

    public function setWidth(int $width): void
    {
        $this->width = $this->height = $width;
    }

    public function setHeight(int $height): void
    {
        $this->width = $this->height = $height;
    }
}
```

Debido a que un `Square` es un poco diferente de un `Rectangle`, necesitamos anular parte del código para permitir 
que un cuadrado exista correctamente.

Anular mucho código en clases para adaptarse a situaciones específicas puede provocar problemas de mantenimiento.

**Bien:**

```php
interface Quadrilateral {
  public function setHeight(int $height);
 
  public function setWidth(int $width);
 
  public function getArea();
}
 
class Rectangle implements Quadrilateral
{
    // ...
}
 
class Square implements Quadrilateral
{
    // ...
}
```

La conclusión aquí es que si encuentra que está anulando una gran cantidad de código, entonces tal vez su arquitectura 
sea incorrecta y debería pensar en el principio de sustitución de Liskov.

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Principio de la segregación de la interfaz (ISP)</b></summary>

 El principio de segregación de la interfaz dice que un cliente solo debe conocer los métodos que van a utilizar y no aquellos que no utilizarán.

**Mal:**

```php
interface Worker 
{
  public function takeBreak();
  public function code();
  public function callToClient();
  public function attendMeetings();
  public function getPaid();
}

class Manager implements Worker 
{
    // ...

    public function code() 
    {
        return false;
    }
}

class Developer implements Worker 
{
    // ...

    public function callToClient() 
    {
        echo "I'll ask my manager.";
    }
}
```
En este ejemplo vemos como la interfaz `Worker` abarca muchas funcionalidades y nos vemos obligados a implementar 
el método `callToClient()` en la clase `Developer` o devolver falso en el método `code()` para `Manager` ya que los managers no desarrollan. 

**Bien:**

La solución es dividir en varias interfaces especificas.

```php
interface Worker 
{
    public function takeBreak();
    public function getPaid();
}
 
interface Coder 
{
    public function code();
}
 
interface ClientFacer 
{
    public function callToClient();
    public function attendMeetings();
}

class Developer implements Worker, Coder 
{
    // ...
}
 
class Manager implements Worker, ClientFacer 
{
    // ...
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

<details>
  <summary><b>Principio de la inversión de dependencias (DIP)</b></summary>

El principio de la inversión de dependencias nos dice que las clases no deben depender de clases concretas, sino que deben depender de interfaces.

**Mal:**

```php
class MySqlConnection 
{
    public function connect() {}
}
 
class PageLoader 
{
    private MySqlConnection $dbConnection;

    public function __construct(MySqlConnection $dbConnection) 
    {
        $this->dbConnection = $dbConnection;
    }
}
```

**Bien:**

```php
interface DbConnectionInterface 
{
    public function connect();
} 
 
class MySqlConnection implements DbConnectionInterface 
{
    public function connect() {}
}
 
class PageLoader 
{
    private DbConnectionInterface $dbConnection;

    public function __construct(DbConnectionInterface $dbConnection) 
    {
        $this->dbConnection = $dbConnection;
    }
}
```

Otro ejemplo de este principio lo vimos en la sección de [Composición sobre Herencia]

**[⬆ Volver](#tabla-de-contenidos)**

</details>

## Principio DRY
<details>
  <summary><b>Principio DRY: Don’t Repeat Yourself</b></summary>

Según este principio toda pieza de información nunca debería ser duplicada debido a que la duplicación incrementa la dificultad 
en los cambios y evolución posterior, puede perjudicar la claridad y crear un espacio para posibles inconsistencias.

A menudo tienes código duplicado porque tienes dos o más cosas ligeramente diferentes, que comparten mucho en común, 
pero sus diferencias te fuerzan a tener dos o más funciones separadas haciendo mucho de lo mismo. 
Remover código duplicado significa crear una abstracción que puedan manejar diferentes conjuntos de cosas en una función/modulo/clase. 

**Mal:**

```php
function showDeveloperList(array $developers): void
{
    foreach ($developers as $developer) {
        $expectedSalary = $developer->calculateExpectedSalary();
        $experience = $developer->getExperience();
        $githubLink = $developer->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}

function showManagerList(array $managers): void
{
    foreach ($managers as $manager) {
        $expectedSalary = $manager->calculateExpectedSalary();
        $experience = $manager->getExperience();
        $githubLink = $manager->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**Bien:**

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        $expectedSalary = $employee->calculateExpectedSalary();
        $experience = $employee->getExperience();
        $githubLink = $employee->getGithubLink();
        $data = [
            $expectedSalary,
            $experience,
            $githubLink
        ];

        render($data);
    }
}
```

**Very good:**

Mejor aún si eliminamos variables temporales.

```php
function showList(array $employees): void
{
    foreach ($employees as $employee) {
        render([
            $employee->calculateExpectedSalary(),
            $employee->getExperience(),
            $employee->getGithubLink()
        ]);
    }
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

## Principio Tell don't ask
<details>
  <summary><b>¿Qué es el principio Tell don't ask?</b></summary>

El principio Tell Don’t Ask nos dice que no debemos preguntar a un objeto sobre su estado para luego realizar una acción. 
Por el contrario, debemos realizar la acción directamente.

**Mal:**

```php
$order = new Order();
if ($order->hasFreeShipping()) {
    $order->shippingCost = 0;
}
```

**Bien:**

```php
$order = new Order();
$order->updateShippingPrice();

class Order
{
    public function updateShippingPrice()
    {
        if ($this->hasFreeShipping()) {
            $this->shippingCost = 0;
        }

        // ...
    }
}
```

**[⬆ Volver](#tabla-de-contenidos)**

</details>

# Testing
WIP

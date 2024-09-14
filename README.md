<h1>SOLID POO</h1>

## Single Responsiblity Principle (Princípio da responsabilidade única) - SRP

Uma classe deve conter somente uma responsabilidade e ter um e somente um motivo para mudar.

* Exemplo de uma classe que viola o SRP, ela possui muitas responsabilidades.

```php
class User {
    private $name;
    private $email;
    private $password;

    public function __construct($name, $email, $password) {
        $this->name = $name;
        $this->email = $email;
        $this->password = $password;
    }

    // Métodos relacionados ao usuário
    public function getName() {
        return $this->name;
    }

    public function getEmail() {
        return $this->email;
    }

    public function getPassword() {
        return $this->password;
    }

    // Método para salvar usuário no banco de dados
    public function saveToDatabase() {
        $connection = new DatabaseConnection();
        $connection->query("INSERT INTO users (name, email, password) VALUES ('$this->name', '$this->email', '$this->password')");
    }

    // Método para enviar email de boas-vindas
    public function sendWelcomeEmail() {
        $mailer = new Mailer();
        $mailer->send($this->email, "Bem-vindo!", "Obrigado por se registrar, $this->name!");
    }
}
```

* Exemplo de como ficaria respeitando o princípio SRP.

```php
class User {
    private $name;
    private $email;
    private $password;

    public function __construct($name, $email, $password) {
        $this->name = $name;
        $this->email = $email;
        $this->password = $password;
    }

    public function getName() {
        return $this->name;
    }

    public function getEmail() {
        return $this->email;
    }

    public function getPassword() {
        return $this->password;
    }
}

// Classe que salva o usuário

class UserRepository {
    public function save(User $user) {
        // Conecta ao banco de dados e salva o usuário
        $connection = new DatabaseConnection();
        $connection->query("INSERT INTO users (name, email, password) VALUES ('{$user->getName()}', '{$user->getEmail()}', '{$user->getPassword()}')");
    }
}

// Classe que envia o e-mail

class UserMailer {
    public function sendWelcomeEmail(User $user) {
        // Configuração e envio do email
        $mailer = new Mailer();
        $mailer->send($user->getEmail(), "Bem-vindo!", "Obrigado por se registrar, {$user->getName()}!");
    }
}

```

* Atenção: O mesmo se aplica para funções e métodos.

## Open-Closed Principle (Princípio Aberto-Fechado) - OCP

Objetos e entidades devem estar apertos para extensão, mas fechados para modificação.

* Exemplo de uso de OCP.

```php
interface ReportGenerator {
    public function generate($data);
}

class PdfReportGenerator implements ReportGenerator {
    public function generate($data) {
        // Lógica para gerar relatório em PDF
    }
}

class ExcelReportGenerator implements ReportGenerator {
    public function generate($data) {
        // Lógica para gerar relatório em Excel
    }
}

class Report {
    private $generator;

    public function __construct(ReportGenerator $generator) {
        $this->generator = $generator;
    }

    public function generate($data) {
        return $this->generator->generate($data);
    }
}
```

## Liskov Substitution Principle (Princípio da substituição de Liskov) - LSP

Uma classe derivada pode ser substituível por sua classe base. Por exemplo, um objeto B pode ser substituível pelo objeto A sem que as propriedades desse programa sejam alteradas.

* Exemplo de classe que viola o LSP

```php
class Rectangle {
    protected $width;
    protected $height;

    public function setWidth($width) {
        $this->width = $width;
    }

    public function setHeight($height) {
        $this->height = $height;
    }

    public function getArea() {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle {
    public function setWidth($width) {
        $this->width = $width;
        $this->height = $width; // Mantendo a propriedade de um quadrado
    }

    public function setHeight($height) {
        $this->width = $height; // Mantendo a propriedade de um quadrado
        $this->height = $height;
    }
}

// Função que espera um Retângulo
function calculateArea(Rectangle $rectangle) {
    $rectangle->setWidth(5);
    $rectangle->setHeight(10);
    return $rectangle->getArea();
}

$rectangle = new Rectangle();
echo calculateArea($rectangle); // Saída: 50

$square = new Square();
echo calculateArea($square); // Saída esperada: 50, Saída real: 100 (violação do LSP)

```

* Exemplo que respeita a LSP

```php
interface Shape {
    public function getArea();
}

class Rectangle implements Shape {
    protected $width;
    protected $height;

    public function __construct($width, $height) {
        $this->width = $width;
        $this->height = $height;
    }

    public function getArea() {
        return $this->width * $this->height;
    }
}

class Square implements Shape {
    protected $side;

    public function __construct($side) {
        $this->side = $side;
    }

    public function getArea() {
        return $this->side * $this->side;
    }
}

// Função que espera uma Forma
function calculateArea(Shape $shape) {
    return $shape->getArea();
}

$rectangle = new Rectangle(5, 10);
echo calculateArea($rectangle); // Saída: 50

$square = new Square(5);
echo calculateArea($square); // Saída: 25

```

## Interface Segregation Principle (Princípio da Segregação da Interface) - PSI

Uma classe não deve ser forçada a implementar funções e métodos que não irá usar. Ou seja, as interfaces devem ser mais específicas do que genéricas

* Exemplo de implementação que viola o PSI

```php
interface Bird {
    public function fly();
    public function swim();
}

class Sparrow implements Bird {
    public function fly() {
        // O pardal pode voar
        echo "Sparrow is flying";
    }

    public function swim() {
        // O pardal não pode nadar, mas é forçado a implementar esse método
        throw new Exception("Sparrow cannot swim");
    }
}

class Penguin implements Bird {
    public function fly() {
        // O pinguim não pode voar, mas é forçado a implementar esse método
        throw new Exception("Penguin cannot fly");
    }

    public function swim() {
        // O pinguim pode nadar
        echo "Penguin is swimming";
    }
}
```

* Exemplo de utilização correta com PSI

```php
interface FlyingBird {
    public function fly();
}

interface SwimmingBird {
    public function swim();
}

class Sparrow implements FlyingBird {
    public function fly() {
        // O pardal pode voar
        echo "Sparrow is flying";
    }
}

class Penguin implements SwimmingBird {
    public function swim() {
        // O pinguim pode nadar
        echo "Penguin is swimming";
    }
}
```

## Dependency Inversion Principle (Princípio da inversão da dependência) - D

Devemos depender de abstrações e não implementações

* Exemplo que viola o D

```php
class PaypalPayment {
    public function processPayment($amount) {
        // Lógica específica para processar o pagamento via PayPal
        echo "Processando o pagamento de $$amount via PayPal.";
    }
}

class OrderProcessor {
    private $payment;

    public function __construct() {
        // Depende diretamente da implementação concreta PaypalPayment
        $this->payment = new PaypalPayment();
    }

    public function processOrder($amount) {
        $this->payment->processPayment($amount);
    }
}

$orderProcessor = new OrderProcessor();
$orderProcessor->processOrder(100);
```

* Exemplo correto de utilização do D

```php
// Interface de abstração
interface PaymentMethod {
    public function processPayment($amount);
}

// Implementação concreta do PayPal
class PaypalPayment implements PaymentMethod {
    public function processPayment($amount) {
        echo "Processando o pagamento de $$amount via PayPal.";
    }
}

// Outra implementação concreta do Stripe
class StripePayment implements PaymentMethod {
    public function processPayment($amount) {
        echo "Processando o pagamento de $$amount via Stripe.";
    }
}

// Classe de alto nível que depende da abstração PaymentMethod
class OrderProcessor {
    private $payment;

    public function __construct(PaymentMethod $payment) {
        $this->payment = $payment;
    }

    public function processOrder($amount) {
        $this->payment->processPayment($amount);
    }
}

// Uso
$paypalPayment = new PaypalPayment();
$orderProcessor = new OrderProcessor($paypalPayment);
$orderProcessor->processOrder(100);

// Ou, se quisermos mudar para Stripe, fazemos isso sem alterar OrderProcessor
$stripePayment = new StripePayment();
$orderProcessor = new OrderProcessor($stripePayment);
$orderProcessor->processOrder(200);
```


## Referência

Código fonte TV

* [SOLID (O básico para você programar melhor)](https://www.youtube.com/watch?v=mkx0CdWiPRA)


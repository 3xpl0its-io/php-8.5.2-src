# PHP 8.5 Moderno: Tópicos Avançados - Parte 2

**Continuação dos capítulos complementares**

---

## Capítulo 29: OOP - Abstract Classes Avançado

### 29.1 Classes Abstratas com Métodos Concretos

```php
<?php

declare(strict_types=1);

abstract class Forma
{
    // Propriedades concretas
    protected string $cor;
    
    public function __construct(string $cor)
    {
        $this->cor = $cor;
    }
    
    // Métodos abstratos (devem ser implementados)
    abstract public function calcularArea(): float;
    abstract public function calcularPerimetro(): float;
    
    // Métodos concretos (já implementados)
    public function obterCor(): string
    {
        return $this->cor;
    }
    
    public function definirCor(string $cor): void
    {
        $this->cor = $cor;
    }
    
    public function exibirInfo(): string
    {
        return sprintf(
            "%s - Cor: %s, Área: %.2f, Perímetro: %.2f",
            get_class($this),
            $this->cor,
            $this->calcularArea(),
            $this->calcularPerimetro()
        );
    }
    
    // Template Method Pattern
    final public function renderizar(): string
    {
        $html = "<div style='color: {$this->cor}'>";
        $html .= $this->renderizarConteudo();
        $html .= "</div>";
        
        return $html;
    }
    
    // Método protegido que subclasses podem sobrescrever
    protected function renderizarConteudo(): string
    {
        return $this->exibirInfo();
    }
}

class Retangulo extends Forma
{
    public function __construct(
        string $cor,
        private float $largura,
        private float $altura
    ) {
        parent::__construct($cor);
    }
    
    public function calcularArea(): float
    {
        return $this->largura * $this->altura;
    }
    
    public function calcularPerimetro(): float
    {
        return 2 * ($this->largura + $this->altura);
    }
}

class Circulo extends Forma
{
    public function __construct(
        string $cor,
        private float $raio
    ) {
        parent::__construct($cor);
    }
    
    public function calcularArea(): float
    {
        return pi() * ($this->raio ** 2);
    }
    
    public function calcularPerimetro(): float
    {
        return 2 * pi() * $this->raio;
    }
}

// Uso
$retangulo = new Retangulo('azul', 5, 10);
echo $retangulo->exibirInfo();
// Retangulo - Cor: azul, Área: 50.00, Perímetro: 30.00

$circulo = new Circulo('vermelho', 5);
echo $circulo->exibirInfo();
// Circulo - Cor: vermelho, Área: 78.54, Perímetro: 31.42
```

### 29.2 Template Method Pattern

```php
<?php

declare(strict_types=1);

abstract class ProcessadorPagamento
{
    // Template Method: define o esqueleto do algoritmo
    final public function processar(float $valor): bool
    {
        // 1. Validar
        if (!$this->validar($valor)) {
            return false;
        }
        
        // 2. Preparar transação
        $dadosTransacao = $this->prepararTransacao($valor);
        
        // 3. Executar pagamento (implementado pela subclasse)
        $resultado = $this->executarPagamento($dadosTransacao);
        
        // 4. Registrar resultado
        $this->registrarTransacao($resultado);
        
        return $resultado['sucesso'];
    }
    
    // Métodos com implementação padrão (podem ser sobrescritos)
    protected function validar(float $valor): bool
    {
        return $valor > 0;
    }
    
    protected function prepararTransacao(float $valor): array
    {
        return [
            'valor' => $valor,
            'timestamp' => time(),
            'gateway' => static::class
        ];
    }
    
    protected function registrarTransacao(array $resultado): void
    {
        $status = $resultado['sucesso'] ? 'SUCESSO' : 'FALHA';
        error_log("Transação: $status - " . json_encode($resultado));
    }
    
    // Método abstrato (cada gateway implementa diferente)
    abstract protected function executarPagamento(array $dados): array;
}

class PagamentoCartao extends ProcessadorPagamento
{
    protected function executarPagamento(array $dados): array
    {
        // Lógica específica de cartão de crédito
        return [
            'sucesso' => true,
            'transacao_id' => 'CC_' . uniqid(),
            'valor' => $dados['valor']
        ];
    }
}

class PagamentoPix extends ProcessadorPagamento
{
    protected function validar(float $valor): bool
    {
        // PIX tem valor mínimo de R$ 1
        return parent::validar($valor) && $valor >= 1.0;
    }
    
    protected function executarPagamento(array $dados): array
    {
        // Lógica específica de PIX
        return [
            'sucesso' => true,
            'transacao_id' => 'PIX_' . uniqid(),
            'valor' => $dados['valor'],
            'qr_code' => 'data:image/png;base64,...'
        ];
    }
}

// Uso
$pagamentoCartao = new PagamentoCartao();
$sucesso = $pagamentoCartao->processar(100.00);

$pagamentoPix = new PagamentoPix();
$sucesso = $pagamentoPix->processar(50.00);
```

### 29.3 Abstract Classes vs Interfaces

```php
<?php

declare(strict_types=1);

// Interface: apenas contrato
interface PagavelInterface
{
    public function processar(float $valor): bool;
    public function obterTaxa(): float;
}

// Abstract Class: contrato + implementação parcial
abstract class ProcessadorBase
{
    protected array $logs = [];
    
    // Método concreto
    public function registrarLog(string $mensagem): void
    {
        $this->logs[] = [
            'timestamp' => time(),
            'mensagem' => $mensagem
        ];
    }
    
    public function obterLogs(): array
    {
        return $this->logs;
    }
    
    // Método abstrato
    abstract public function processar(float $valor): bool;
}

// Classe pode implementar interface E estender classe abstrata
class PagamentoBoleto extends ProcessadorBase implements PagavelInterface
{
    public function processar(float $valor): bool
    {
        $this->registrarLog("Processando boleto: R$ $valor");
        
        // Lógica de processamento
        return true;
    }
    
    public function obterTaxa(): float
    {
        return 2.50;  // Taxa fixa
    }
}

// Quando usar cada um:
// - Interface: quando precisa de contrato sem implementação
// - Abstract Class: quando tem código reutilizável entre subclasses
// - Ambos: quando precisa do contrato E da implementação base
```

### 29.4 Hierarquia de Classes Abstratas

```php
<?php

declare(strict_types=1);

abstract class Veiculo
{
    public function __construct(
        protected string $marca,
        protected string $modelo
    ) {}
    
    abstract public function ligar(): string;
    
    public function obterDescricao(): string
    {
        return "{$this->marca} {$this->modelo}";
    }
}

abstract class VeiculoMotorizado extends Veiculo
{
    public function __construct(
        string $marca,
        string $modelo,
        protected int $cilindradas
    ) {
        parent::__construct($marca, $modelo);
    }
    
    abstract public function acelerar(): string;
    
    public function obterPotencia(): int
    {
        // Estimativa: 1 cilindrada ≈ 0.5 CV
        return (int) ($this->cilindradas * 0.5);
    }
}

class Carro extends VeiculoMotorizado
{
    public function ligar(): string
    {
        return "Carro ligado: motor {$this->cilindradas}cc";
    }
    
    public function acelerar(): string
    {
        return "Carro acelerando com {$this->obterPotencia()} CV";
    }
}

class Moto extends VeiculoMotorizado
{
    public function ligar(): string
    {
        return "Moto ligada: motor {$this->cilindradas}cc";
    }
    
    public function acelerar(): string
    {
        return "Moto acelerando com {$this->obterPotencia()} CV";
    }
}

// Uso
$carro = new Carro('Toyota', 'Corolla', 2000);
echo $carro->ligar();      // Carro ligado: motor 2000cc
echo $carro->acelerar();   // Carro acelerando com 1000 CV

$moto = new Moto('Honda', 'CB 500', 500);
echo $moto->ligar();       // Moto ligada: motor 500cc
```

### 📝 Exercícios do Capítulo 29

1. Crie hierarquia abstrata para diferentes tipos de relatórios
2. Implemente Template Method para processamento de pedidos
3. Faça classes abstratas para sistema de notificações
4. Crie base abstrata para diferentes tipos de validadores
5. Implemente Strategy pattern com classes abstratas

---

## Capítulo 30: OOP - Class Constants

### 30.1 Constantes de Classe Básicas

```php
<?php

declare(strict_types=1);

class Database
{
    // Constantes públicas
    public const HOST = 'localhost';
    public const PORT = 3306;
    public const CHARSET = 'utf8mb4';
    
    // Constantes privadas (PHP 7.1+)
    private const USERNAME = 'root';
    private const PASSWORD = 'secret';
    
    public static function conectar(): PDO
    {
        $dsn = sprintf(
            'mysql:host=%s;port=%d;charset=%s',
            self::HOST,
            self::PORT,
            self::CHARSET
        );
        
        return new PDO($dsn, self::USERNAME, self::PASSWORD);
    }
}

// Acessar constantes públicas
echo Database::HOST;     // localhost
echo Database::PORT;     // 3306

// Database::USERNAME;  // ERRO: constante privada
```

### 30.2 Constantes com Tipos (PHP 8.3+)

```php
<?php

declare(strict_types=1);

class Configuracao
{
    // Constantes tipadas
    public const string APP_NAME = 'Meu Sistema';
    public const string VERSION = '1.0.0';
    public const int MAX_UPLOAD_SIZE = 5242880;  // 5MB em bytes
    public const float TAX_RATE = 0.15;
    public const bool DEBUG = true;
    public const array SUPPORTED_LANGUAGES = ['pt-BR', 'en-US', 'es-ES'];
}

echo Configuracao::APP_NAME;          // Meu Sistema
echo Configuracao::MAX_UPLOAD_SIZE;   // 5242880
print_r(Configuracao::SUPPORTED_LANGUAGES);
```

### 30.3 Constantes em Enums

```php
<?php

declare(strict_types=1);

enum StatusPedido: string
{
    case PENDENTE = 'pendente';
    case APROVADO = 'aprovado';
    case ENVIADO = 'enviado';
    case ENTREGUE = 'entregue';
    case CANCELADO = 'cancelado';
    
    // Constantes dentro de enum
    public const ESTADOS_EDITAVEIS = [
        self::PENDENTE,
        self::APROVADO
    ];
    
    public const ESTADOS_FINAIS = [
        self::ENTREGUE,
        self::CANCELADO
    ];
    
    public function podeEditar(): bool
    {
        return in_array($this, self::ESTADOS_EDITAVEIS, true);
    }
    
    public function ehFinal(): bool
    {
        return in_array($this, self::ESTADOS_FINAIS, true);
    }
}

// Uso
$status = StatusPedido::PENDENTE;
echo $status->podeEditar() ? 'Pode editar' : 'Não pode editar';  // Pode editar
```

### 30.4 Constantes Herdadas

```php
<?php

declare(strict_types=1);

class Animal
{
    public const TIPO = 'Animal';
    protected const HABITAT = 'Terrestre';
    
    public static function obterTipo(): string
    {
        return self::TIPO;
    }
    
    public static function obterHabitat(): string
    {
        return static::HABITAT;  // Late static binding
    }
}

class Cachorro extends Animal
{
    // Sobrescrever constante
    public const TIPO = 'Cachorro';
    protected const HABITAT = 'Doméstico';
}

class Peixe extends Animal
{
    public const TIPO = 'Peixe';
    protected const HABITAT = 'Aquático';
}

// Uso
echo Animal::TIPO;           // Animal
echo Cachorro::TIPO;         // Cachorro
echo Peixe::TIPO;            // Peixe

// Late static binding em ação
echo Cachorro::obterTipo();     // Cachorro
echo Cachorro::obterHabitat();  // Doméstico
```

### 30.5 Constantes Mágicas

```php
<?php

declare(strict_types=1);

class Logger
{
    public function log(string $mensagem): void
    {
        echo sprintf(
            "[%s] [%s::%s] %s\n",
            date('Y-m-d H:i:s'),
            __CLASS__,      // Nome da classe
            __METHOD__,     // Nome do método
            $mensagem
        );
    }
    
    public function debug(): void
    {
        echo "Arquivo: " . __FILE__ . "\n";      // Caminho do arquivo
        echo "Linha: " . __LINE__ . "\n";        // Número da linha
        echo "Diretório: " . __DIR__ . "\n";     // Diretório do arquivo
        echo "Função: " . __FUNCTION__ . "\n";   // Nome da função
        echo "Classe: " . __CLASS__ . "\n";      // Nome da classe
        echo "Método: " . __METHOD__ . "\n";     // Classe::método
        echo "Namespace: " . __NAMESPACE__ . "\n"; // Namespace atual
        echo "Trait: " . __TRAIT__ . "\n";       // Nome do trait
    }
}

$logger = new Logger();
$logger->log('Teste');
// [2026-02-09 15:30:00] [Logger::log] Teste
```

### 30.6 Uso Prático: Configuração com Constantes

```php
<?php

declare(strict_types=1);

class AppConfig
{
    // Ambiente
    public const string ENVIRONMENT = 'production';
    
    // Database
    public const string DB_HOST = 'localhost';
    public const int DB_PORT = 3306;
    public const string DB_NAME = 'app_db';
    
    // Upload
    public const int MAX_FILE_SIZE = 5 * 1024 * 1024;  // 5MB
    public const array ALLOWED_EXTENSIONS = ['jpg', 'png', 'pdf'];
    
    // Cache
    public const int CACHE_TTL = 3600;  // 1 hora
    
    // Paginação
    public const int ITEMS_PER_PAGE = 20;
    
    // API
    public const string API_VERSION = 'v1';
    public const int RATE_LIMIT = 100;  // requests por minuto
    
    // Verificar ambiente
    public static function isProduction(): bool
    {
        return self::ENVIRONMENT === 'production';
    }
    
    public static function isDevelopment(): bool
    {
        return self::ENVIRONMENT === 'development';
    }
    
    // Obter DSN do banco
    public static function getDatabaseDSN(): string
    {
        return sprintf(
            'mysql:host=%s;port=%d;dbname=%s',
            self::DB_HOST,
            self::DB_PORT,
            self::DB_NAME
        );
    }
}

// Uso
if (AppConfig::isProduction()) {
    ini_set('display_errors', '0');
}

$pdo = new PDO(AppConfig::getDatabaseDSN(), 'user', 'pass');

if ($_FILES['upload']['size'] > AppConfig::MAX_FILE_SIZE) {
    die('Arquivo muito grande');
}
```

### 📝 Exercícios do Capítulo 30

1. Crie classe de configuração com constantes tipadas
2. Implemente sistema de permissões com constantes
3. Faça enum de status com constantes auxiliares
4. Use constantes mágicas para logging avançado
5. Crie hierarquia de classes com constantes herdadas

---

## Capítulo 31: OOP - Access Modifiers Detalhado

### 31.1 Public, Private, Protected

```php
<?php

declare(strict_types=1);

class Veiculo
{
    // public: acessível de qualquer lugar
    public string $marca;
    
    // protected: acessível na classe e subclasses
    protected string $modelo;
    
    // private: acessível apenas na própria classe
    private string $chassi;
    
    public function __construct(string $marca, string $modelo, string $chassi)
    {
        $this->marca = $marca;
        $this->modelo = $modelo;
        $this->chassi = $chassi;
    }
    
    // Método público
    public function obterDescricao(): string
    {
        // Pode acessar todas as propriedades dentro da classe
        return "{$this->marca} {$this->modelo} - Chassi: {$this->chassi}";
    }
    
    // Método protegido
    protected function obterModelo(): string
    {
        return $this->modelo;
    }
    
    // Método privado
    private function obterChassi(): string
    {
        return $this->chassi;
    }
}

class Carro extends Veiculo
{
    public function exibirInfo(): string
    {
        // Pode acessar public e protected
        $info = "Marca: {$this->marca}\n";
        $info .= "Modelo: {$this->obterModelo()}\n";
        
        // Não pode acessar private da classe pai
        // $info .= $this->chassi;  // ERRO
        // $info .= $this->obterChassi();  // ERRO
        
        return $info;
    }
}

// Uso
$carro = new Carro('Toyota', 'Corolla', 'ABC123');

echo $carro->marca;  // OK (public)
// echo $carro->modelo;  // ERRO (protected)
// echo $carro->chassi;  // ERRO (private)

echo $carro->obterDescricao();  // OK (método público)
// echo $carro->obterModelo();  // ERRO (método protegido)
```

### 31.2 Constructor Property Promotion com Visibility

```php
<?php

declare(strict_types=1);

class Usuario
{
    // Constructor property promotion com diferentes visibilidades
    public function __construct(
        public int $id,
        public string $nome,
        protected string $email,
        private string $senhaHash
    ) {}
    
    // Getter para email (protected)
    public function obterEmail(): string
    {
        return $this->email;
    }
    
    // Método para verificar senha (private)
    public function verificarSenha(string $senha): bool
    {
        return password_verify($senha, $this->senhaHash);
    }
}

$usuario = new Usuario(
    id: 1,
    nome: 'João',
    email: 'joao@example.com',
    senhaHash: password_hash('senha123', PASSWORD_DEFAULT)
);

echo $usuario->id;     // OK
echo $usuario->nome;   // OK
// echo $usuario->email;  // ERRO
// echo $usuario->senhaHash;  // ERRO

echo $usuario->obterEmail();  // OK
echo $usuario->verificarSenha('senha123') ? 'Válida' : 'Inválida';
```

### 31.3 Readonly Properties (PHP 8.1+)

```php
<?php

declare(strict_types=1);

class Produto
{
    // Readonly: pode ser atribuído apenas no construtor
    public function __construct(
        public readonly int $id,
        public readonly string $codigo,
        public string $nome,  // Não readonly, pode mudar
        public float $preco   // Não readonly, pode mudar
    ) {}
}

$produto = new Produto(
    id: 1,
    codigo: 'PROD001',
    nome: 'Notebook',
    preco: 2500.00
);

// Pode modificar propriedades normais
$produto->nome = 'Notebook Dell';
$produto->preco = 2800.00;

// Não pode modificar readonly
// $produto->id = 2;  // ERRO
// $produto->codigo = 'PROD002';  // ERRO
```

### 31.4 Asymmetric Visibility (PHP 8.4+)

```php
<?php

declare(strict_types=1);

class ContaBancaria
{
    // Leitura pública, escrita privada
    public private(set) float $saldo = 0.0;
    
    public function depositar(float $valor): void
    {
        if ($valor > 0) {
            $this->saldo += $valor;
        }
    }
    
    public function sacar(float $valor): bool
    {
        if ($valor > 0 && $valor <= $this->saldo) {
            $this->saldo -= $valor;
            return true;
        }
        
        return false;
    }
}

$conta = new ContaBancaria();

// Pode ler
echo $conta->saldo;  // 0.0

// Pode modificar via métodos
$conta->depositar(1000);
echo $conta->saldo;  // 1000.0

// Não pode modificar diretamente
// $conta->saldo = 5000;  // ERRO: set é private
```

### 31.5 Visibilidade de Constantes

```php
<?php

declare(strict_types=1);

class Configuracao
{
    // Constante pública
    public const APP_NAME = 'Meu Sistema';
    
    // Constante protegida (PHP 7.1+)
    protected const SECRET_KEY = 'chave-secreta-123';
    
    // Constante privada (PHP 7.1+)
    private const INTERNAL_VERSION = '2.1.0';
    
    public static function obterVersao(): string
    {
        // Pode acessar constante privada dentro da classe
        return self::INTERNAL_VERSION;
    }
}

class ConfiguracaoExtendida extends Configuracao
{
    public static function obterChave(): string
    {
        // Pode acessar constante protected da classe pai
        return self::SECRET_KEY;
    }
    
    public static function obterVersaoInterna(): string
    {
        // Não pode acessar constante private da classe pai
        // return self::INTERNAL_VERSION;  // ERRO
        
        // Mas pode via método público
        return parent::obterVersao();
    }
}

echo Configuracao::APP_NAME;  // OK
// echo Configuracao::SECRET_KEY;  // ERRO
// echo Configuracao::INTERNAL_VERSION;  // ERRO

echo Configuracao::obterVersao();  // OK
```

### 31.6 Visibility em Traits

```php
<?php

declare(strict_types=1);

trait Loggable
{
    // Propriedades do trait
    protected array $logs = [];
    
    // Método privado do trait
    private function registrarLog(string $mensagem): void
    {
        $this->logs[] = [
            'timestamp' => time(),
            'mensagem' => $mensagem
        ];
    }
    
    // Método protegido do trait
    protected function log(string $mensagem): void
    {
        $this->registrarLog($mensagem);
    }
    
    // Método público do trait
    public function obterLogs(): array
    {
        return $this->logs;
    }
}

class Servico
{
    use Loggable;
    
    public function processar(): void
    {
        $this->log('Processando...');  // OK (protected)
        // $this->registrarLog('teste');  // ERRO (private do trait)
    }
}

$servico = new Servico();
$servico->processar();
print_r($servico->obterLogs());  // OK (public)
```

### 📝 Exercícios do Capítulo 31

1. Crie classe com todos os níveis de visibilidade
2. Implemente encapsulamento completo em classe de domínio
3. Use readonly properties para value objects
4. Experimente asymmetric visibility em entidade
5. Controle acesso a constantes em hierarquia de classes

---

## Capítulo 32: Regular Expressions Completo

### 32.1 Sintaxe Básica de Regex

```php
<?php

declare(strict_types=1);

// Metacaracteres básicos
// .  - Qualquer caractere (exceto quebra de linha)
// ^  - Início da string
// $  - Fim da string
// *  - 0 ou mais
// +  - 1 ou mais
// ?  - 0 ou 1
// [] - Classe de caracteres
// () - Grupo de captura
// |  - OU lógico
// \  - Escape

// Exemplos básicos
$padrao = '/php/i';  // i = case-insensitive
preg_match($padrao, 'PHP é legal');  // 1 (match)

$padrao = '/^PHP/';  // Começa com PHP
preg_match($padrao, 'PHP 8.5');  // 1

$padrao = '/5$/';    // Termina com 5
preg_match($padrao, 'PHP 8.5');  // 1

$padrao = '/PHP.*/';  // PHP seguido de qualquer coisa
preg_match($padrao, 'PHP é incrível');  // 1
```

### 32.2 Classes de Caracteres

```php
<?php

declare(strict_types=1);

// Classes predefinidas
// \d - Dígito [0-9]
// \D - Não dígito [^0-9]
// \w - Palavra [a-zA-Z0-9_]
// \W - Não palavra
// \s - Espaço em branco [ \t\n\r]
// \S - Não espaço

// Validar número
$padrao = '/^\d+$/';
preg_match($padrao, '12345');  // 1
preg_match($padrao, '123a5');  // 0

// Validar alfanumérico
$padrao = '/^\w+$/';
preg_match($padrao, 'usuario123');  // 1

// Classe customizada
$padrao = '/^[a-zA-Z]+$/';  // Apenas letras
preg_match($padrao, 'abc');  // 1
preg_match($padrao, 'abc123');  // 0

// Negação
$padrao = '/^[^0-9]+$/';  // Nenhum número
preg_match($padrao, 'abc');  // 1
preg_match($padrao, 'abc1');  // 0
```

### 32.3 Quantificadores

```php
<?php

declare(strict_types=1);

// {n}   - Exatamente n vezes
// {n,}  - n ou mais vezes
// {n,m} - Entre n e m vezes

// CEP: 5 dígitos + hífen + 3 dígitos
$padrao = '/^\d{5}-\d{3}$/';
preg_match($padrao, '12345-678');  // 1

// CPF: 3 dígitos + ponto + 3 dígitos + ponto + 3 dígitos + hífen + 2 dígitos
$padrao = '/^\d{3}\.\d{3}\.\d{3}-\d{2}$/';
preg_match($padrao, '123.456.789-00');  // 1

// Telefone: 2 dígitos entre parênteses + espaço + 4 ou 5 dígitos + hífen + 4 dígitos
$padrao = '/^\(\d{2}\) \d{4,5}-\d{4}$/';
preg_match($padrao, '(11) 98765-4321');  // 1
preg_match($padrao, '(11) 3456-7890');   // 1

// Senha forte: mínimo 8 caracteres
$padrao = '/^.{8,}$/';
preg_match($padrao, 'senha123');  // 1
```

### 32.4 Grupos e Capturas

```php
<?php

declare(strict_types=1);

// Grupos de captura ()
$texto = 'João Silva tem 25 anos';
$padrao = '/(\w+) (\w+) tem (\d+) anos/';

preg_match($padrao, $texto, $matches);
print_r($matches);
/*
Array (
    [0] => João Silva tem 25 anos  (match completo)
    [1] => João                     (grupo 1)
    [2] => Silva                    (grupo 2)
    [3] => 25                       (grupo 3)
)
*/

// Grupos nomeados (?<nome>...)
$padrao = '/(?<nome>\w+) (?<sobrenome>\w+) tem (?<idade>\d+) anos/';

preg_match($padrao, $texto, $matches);
echo $matches['nome'];       // João
echo $matches['sobrenome'];  // Silva
echo $matches['idade'];      // 25

// Grupos não-capturantes (?:...)
$padrao = '/(?:Sr\.|Sra\.) (\w+)/';

preg_match($padrao, 'Sr. Silva', $matches);
print_r($matches);
/*
Array (
    [0] => Sr. Silva
    [1] => Silva
)
*/
```

### 32.5 Validações Práticas

```php
<?php

declare(strict_types=1);

class Validador
{
    // Email
    public static function email(string $email): bool
    {
        $padrao = '/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/';
        return preg_match($padrao, $email) === 1;
    }
    
    // URL
    public static function url(string $url): bool
    {
        $padrao = '/^https?:\/\/(www\.)?[a-zA-Z0-9-]+(\.[a-zA-Z]{2,})+/';
        return preg_match($padrao, $url) === 1;
    }
    
    // CPF
    public static function cpf(string $cpf): bool
    {
        $padrao = '/^\d{3}\.\d{3}\.\d{3}-\d{2}$/';
        return preg_match($padrao, $cpf) === 1;
    }
    
    // Telefone BR
    public static function telefone(string $tel): bool
    {
        $padrao = '/^\(\d{2}\) \d{4,5}-\d{4}$/';
        return preg_match($padrao, $tel) === 1;
    }
    
    // Senha forte (min 8 chars, maiúscula, minúscula, número, especial)
    public static function senhaForte(string $senha): bool
    {
        if (strlen($senha) < 8) {
            return false;
        }
        
        $temMaiuscula = preg_match('/[A-Z]/', $senha);
        $temMinuscula = preg_match('/[a-z]/', $senha);
        $temNumero = preg_match('/[0-9]/', $senha);
        $temEspecial = preg_match('/[^A-Za-z0-9]/', $senha);
        
        return $temMaiuscula && $temMinuscula && $temNumero && $temEspecial;
    }
    
    // Cartão de crédito
    public static function cartaoCredito(string $numero): bool
    {
        // Remove espaços e hífens
        $numero = preg_replace('/[\s-]/', '', $numero);
        
        // Verifica se são 16 dígitos
        $padrao = '/^\d{16}$/';
        return preg_match($padrao, $numero) === 1;
    }
    
    // Data BR (dd/mm/yyyy)
    public static function dataBR(string $data): bool
    {
        $padrao = '/^(0[1-9]|[12][0-9]|3[01])\/(0[1-9]|1[0-2])\/\d{4}$/';
        return preg_match($padrao, $data) === 1;
    }
    
    // Placa de carro (formato Mercosul)
    public static function placaCarro(string $placa): bool
    {
        // Formato: ABC1D23 ou ABC-1D23
        $padrao = '/^[A-Z]{3}-?\d[A-Z]\d{2}$/';
        return preg_match($padrao, strtoupper($placa)) === 1;
    }
}

// Testes
echo Validador::email('joao@example.com') ? 'Válido' : 'Inválido';  // Válido
echo Validador::cpf('123.456.789-00') ? 'Válido' : 'Inválido';      // Válido
echo Validador::telefone('(11) 98765-4321') ? 'Válido' : 'Inválido'; // Válido
echo Validador::senhaForte('Senha@123') ? 'Válido' : 'Inválido';    // Válido
```

### 32.6 Funções de Regex Avançadas

```php
<?php

declare(strict_types=1);

// preg_match_all: encontrar todas as ocorrências
$texto = 'PHP 8.5 e PHP 8.4 e PHP 8.3';
$padrao = '/PHP \d\.\d/';

preg_match_all($padrao, $texto, $matches);
print_r($matches[0]);
// Array ( [0] => PHP 8.5, [1] => PHP 8.4, [2] => PHP 8.3 )

// preg_replace: substituir
$texto = 'O PHP é legal. PHP rocks!';
$novo = preg_replace('/PHP/', 'JavaScript', $texto);
// O JavaScript é legal. JavaScript rocks!

// Com grupos de captura
$texto = 'João Silva, 25 anos';
$padrao = '/(\w+) (\w+), (\d+) anos/';
$novo = preg_replace($padrao, 'Nome: $1, Sobrenome: $2, Idade: $3', $texto);
// Nome: João, Sobrenome: Silva, Idade: 25

// preg_replace_callback: substituir com função
$texto = 'Preço: 100, 200, 300';
$novo = preg_replace_callback(
    '/\d+/',
    fn($match) => (int)$match[0] * 2,
    $texto
);
// Preço: 200, 400, 600

// preg_split: dividir string
$texto = 'maçã;banana,laranja|uva';
$frutas = preg_split('/[;,|]/', $texto);
print_r($frutas);
// Array ( [0] => maçã, [1] => banana, [2] => laranja, [3] => uva )

// preg_grep: filtrar array
$palavras = ['PHP', 'JavaScript', 'Python', 'Ruby'];
$comP = preg_grep('/^P/', $palavras);
print_r($comP);
// Array ( [0] => PHP, [2] => Python )
```

### 32.7 Lookahead e Lookbehind

```php
<?php

declare(strict_types=1);

// Positive Lookahead (?=...)
// Verifica se seguido por algo (sem capturar)
$texto = 'preço100';
$padrao = '/\d+(?=\D)/';  // Dígitos seguidos de não-dígito

preg_match($padrao, 'preço100', $m);  // Não match (100 no final)
preg_match($padrao, 'preço100reais', $m);  // Match: 100

// Negative Lookahead (?!...)
// Verifica se NÃO seguido por algo
$padrao = '/\d+(?!\.)/';  // Números não seguidos de ponto

preg_match($padrao, '123.45', $m);  // Match: 45
preg_match($padrao, '123', $m);     // Match: 123

// Positive Lookbehind (?<=...)
// Verifica se precedido por algo
$padrao = '/(?<=R\$)\d+/';  // Números precedidos de R$

preg_match($padrao, 'R$100', $m);  // Match: 100
print_r($m[0]);  // 100

// Negative Lookbehind (?<!...)
$padrao = '/(?<!-)\\d+/';  // Números NÃO precedidos de -

preg_match($padrao, '-50', $m);  // Não match
preg_match($padrao, '50', $m);   // Match: 50

// Exemplo prático: Extrair preço
$texto = 'O produto custa R$ 99,90';
$padrao = '/(?<=R\$ )\d+,\d{2}/';

preg_match($padrao, $texto, $m);
echo $m[0];  // 99,90
```

### 📝 Exercícios do Capítulo 32

1. Valide CNPJ com regex
2. Extraia todos os emails de um texto
3. Valide senha com requisitos complexos
4. Substitua números de cartão parcialmente (****1234)
5. Faça parser de markdown simples com regex

---

**Continua na próxima parte com:**
- Capítulo 33: Validação de Forms E-mail e URL
- Capítulo 34: cURL - Requisições HTTP

Deseja que eu continue?

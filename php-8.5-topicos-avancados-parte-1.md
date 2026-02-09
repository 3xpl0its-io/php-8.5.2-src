# PHP 8.5 Moderno: Tópicos Avançados Complementares

**Capítulos adicionais para domínio completo do PHP 8.5**

---

## Índice

1. [Iterables e Iteradores](#capítulo-24-iterables-e-iteradores)
2. [Namespaces Avançado](#capítulo-25-namespaces-avançado)
3. [OOP: Properties e Methods Estáticos](#capítulo-26-oop-properties-e-methods-estáticos)
4. [OOP: Traits Avançado](#capítulo-27-oop-traits-avançado)
5. [OOP: Interfaces Avançado](#capítulo-28-oop-interfaces-avançado)
6. [OOP: Abstract Classes Avançado](#capítulo-29-oop-abstract-classes-avançado)
7. [OOP: Class Constants](#capítulo-30-oop-class-constants)
8. [OOP: Access Modifiers Detalhado](#capítulo-31-oop-access-modifiers-detalhado)
9. [Regular Expressions Completo](#capítulo-32-regular-expressions-completo)
10. [Validação de Forms E-mail e URL](#capítulo-33-validação-de-forms-e-mail-e-url)
11. [cURL - Requisições HTTP](#capítulo-34-curl-requisições-http)

---

## Capítulo 24: Iterables e Iteradores

### 24.1 O que são Iterables?

```php
<?php

declare(strict_types=1);

// Iterable: qualquer coisa que pode ser iterada com foreach
// Arrays e objetos que implementam Traversable

function processarIterable(iterable $items): void
{
    foreach ($items as $item) {
        echo $item . "\n";
    }
}

// Funciona com arrays
$array = [1, 2, 3, 4, 5];
processarIterable($array);

// Funciona com Generator
function gerarNumeros(): Generator
{
    for ($i = 1; $i <= 5; $i++) {
        yield $i;
    }
}

processarIterable(gerarNumeros());

// Type hint iterable pode retornar array ou Traversable
function obterDados(): iterable
{
    return [1, 2, 3];
    // ou
    // return gerarNumeros();
}
```

### 24.2 Generators

```php
<?php

declare(strict_types=1);

// Generator: cria iterador sem criar array em memória
function lerArquivoGrande(string $arquivo): Generator
{
    $handle = fopen($arquivo, 'r');
    
    while (($linha = fgets($handle)) !== false) {
        yield $linha;  // Retorna um valor por vez
    }
    
    fclose($handle);
}

// Uso: processa linha por linha sem carregar tudo na memória
foreach (lerArquivoGrande('arquivo_enorme.txt') as $linha) {
    echo $linha;
}

// Generator com chave e valor
function gerarPares(): Generator
{
    yield 'um' => 1;
    yield 'dois' => 2;
    yield 'tres' => 3;
}

foreach (gerarPares() as $chave => $valor) {
    echo "$chave: $valor\n";
}

// Generator pode receber valores (send)
function contador(): Generator
{
    $i = 0;
    
    while (true) {
        $reset = yield $i;
        
        if ($reset) {
            $i = 0;
        } else {
            $i++;
        }
    }
}

$gen = contador();

echo $gen->current();  // 0
$gen->next();
echo $gen->current();  // 1
$gen->send(true);      // Reset
echo $gen->current();  // 0
```

### 24.3 Iterator Interface

```php
<?php

declare(strict_types=1);

// Implementar Iterator manualmente
class MeuIterador implements Iterator
{
    private array $items;
    private int $posicao = 0;
    
    public function __construct(array $items)
    {
        $this->items = $items;
    }
    
    // Retorna elemento atual
    public function current(): mixed
    {
        return $this->items[$this->posicao];
    }
    
    // Retorna chave atual
    public function key(): int
    {
        return $this->posicao;
    }
    
    // Avança para próximo elemento
    public function next(): void
    {
        $this->posicao++;
    }
    
    // Volta para o início
    public function rewind(): void
    {
        $this->posicao = 0;
    }
    
    // Verifica se posição atual é válida
    public function valid(): bool
    {
        return isset($this->items[$this->posicao]);
    }
}

// Uso
$iterador = new MeuIterador([10, 20, 30, 40, 50]);

foreach ($iterador as $valor) {
    echo $valor . "\n";
}

// Exemplo prático: Iterar sobre intervalo numérico
class IntervaloIterador implements Iterator
{
    private int $atual;
    
    public function __construct(
        private int $inicio,
        private int $fim,
        private int $passo = 1
    ) {
        $this->atual = $inicio;
    }
    
    public function current(): int
    {
        return $this->atual;
    }
    
    public function key(): int
    {
        return $this->atual;
    }
    
    public function next(): void
    {
        $this->atual += $this->passo;
    }
    
    public function rewind(): void
    {
        $this->atual = $this->inicio;
    }
    
    public function valid(): bool
    {
        return $this->passo > 0 
            ? $this->atual <= $this->fim
            : $this->atual >= $this->fim;
    }
}

// Uso
$range = new IntervaloIterador(1, 10, 2);

foreach ($range as $numero) {
    echo $numero . " ";  // 1 3 5 7 9
}
```

### 24.4 IteratorAggregate Interface

```php
<?php

declare(strict_types=1);

// IteratorAggregate: delega iteração para outro objeto
class Colecao implements IteratorAggregate
{
    private array $items = [];
    
    public function adicionar(mixed $item): void
    {
        $this->items[] = $item;
    }
    
    // Retorna um iterador
    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->items);
    }
}

// Uso
$colecao = new Colecao();
$colecao->adicionar('Item 1');
$colecao->adicionar('Item 2');
$colecao->adicionar('Item 3');

foreach ($colecao as $item) {
    echo $item . "\n";
}

// Exemplo avançado: Filtrar durante iteração
class ColecaoFiltrada implements IteratorAggregate
{
    public function __construct(
        private array $items,
        private ?callable $filtro = null
    ) {}
    
    public function getIterator(): Traversable
    {
        $items = $this->items;
        
        if ($this->filtro !== null) {
            $items = array_filter($items, $this->filtro);
        }
        
        return new ArrayIterator($items);
    }
}

// Uso
$numeros = new ColecaoFiltrada(
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    fn($n) => $n % 2 === 0  // Apenas pares
);

foreach ($numeros as $numero) {
    echo $numero . " ";  // 2 4 6 8 10
}
```

### 24.5 Exemplo Prático: Paginação com Iterator

```php
<?php

declare(strict_types=1);

class PaginadorIterador implements Iterator
{
    private int $paginaAtual = 1;
    private array $paginaAtualDados = [];
    
    public function __construct(
        private array $todosOsDados,
        private int $itensPorPagina
    ) {
        $this->carregarPagina();
    }
    
    private function carregarPagina(): void
    {
        $offset = ($this->paginaAtual - 1) * $this->itensPorPagina;
        $this->paginaAtualDados = array_slice(
            $this->todosOsDados,
            $offset,
            $this->itensPorPagina
        );
    }
    
    public function current(): array
    {
        return $this->paginaAtualDados;
    }
    
    public function key(): int
    {
        return $this->paginaAtual;
    }
    
    public function next(): void
    {
        $this->paginaAtual++;
        $this->carregarPagina();
    }
    
    public function rewind(): void
    {
        $this->paginaAtual = 1;
        $this->carregarPagina();
    }
    
    public function valid(): bool
    {
        return !empty($this->paginaAtualDados);
    }
}

// Uso
$dados = range(1, 100);  // 100 itens
$paginador = new PaginadorIterador($dados, 10);  // 10 por página

foreach ($paginador as $numeroPagina => $itensPagina) {
    echo "Página $numeroPagina: " . implode(', ', $itensPagina) . "\n";
}
```

### 📝 Exercícios do Capítulo 24

1. Crie generator que lê CSV linha por linha
2. Implemente Iterator para árvore binária
3. Faça IteratorAggregate para coleção ordenada
4. Crie generator infinito de números Fibonacci
5. Implemente paginação customizada com Iterator

---

## Capítulo 25: Namespaces Avançado

### 25.1 Declaração e Organização

```php
<?php

declare(strict_types=1);

// Namespace simples
namespace App\Model;

class Usuario
{
    public string $nome;
}

// Múltiplas classes no mesmo namespace
namespace App\Service;

class UsuarioService
{
    public function criar(): void
    {
        // código
    }
}

class EmailService
{
    public function enviar(): void
    {
        // código
    }
}

// Sub-namespaces
namespace App\Service\Email;

class SMTPService
{
    // Implementação SMTP
}

class MailgunService
{
    // Implementação Mailgun
}
```

### 25.2 Importação e Aliases

```php
<?php

declare(strict_types=1);

namespace App\Controller;

// Importar classe específica
use App\Model\Usuario;
use App\Service\UsuarioService;
use App\Service\EmailService;

// Importar múltiplas classes do mesmo namespace
use App\Repository\{
    UsuarioRepository,
    ProdutoRepository,
    PedidoRepository
};

// Alias para evitar conflitos
use App\Model\Usuario as UsuarioModel;
use App\DTO\Usuario as UsuarioDTO;

class UsuarioController
{
    public function __construct(
        private UsuarioService $service,
        private UsuarioRepository $repository
    ) {}
    
    public function criar(array $dados): UsuarioModel
    {
        // Converter DTO para Model
        $dto = new UsuarioDTO($dados);
        
        return $this->service->criar($dto);
    }
}
```

### 25.3 Namespace Global

```php
<?php

declare(strict_types=1);

namespace App\Util;

// Acessar classes do namespace global com \
class DateHelper
{
    public function agora(): \DateTime
    {
        // \ antes de DateTime acessa classe global
        return new \DateTime();
    }
    
    public function formatarData(\DateTime $data): string
    {
        return $data->format('d/m/Y');
    }
}

// Funções globais também precisam de \
function calcular(): float
{
    return \sqrt(16);  // Função sqrt() global
}

// Constantes globais
class Config
{
    public function versaoPhp(): string
    {
        return \PHP_VERSION;  // Constante global
    }
}
```

### 25.4 Autoloading com PSR-4

```php
<?php
// autoload.php

declare(strict_types=1);

/**
 * PSR-4 Autoloader
 * 
 * Estrutura de diretórios:
 * 
 * projeto/
 * ├── src/
 * │   ├── Model/
 * │   │   └── Usuario.php (namespace App\Model)
 * │   ├── Service/
 * │   │   └── UsuarioService.php (namespace App\Service)
 * │   └── Repository/
 * │       └── UsuarioRepository.php (namespace App\Repository)
 * └── autoload.php
 */

spl_autoload_register(function (string $classe) {
    // Prefixo do namespace
    $prefixo = 'App\\';
    
    // Diretório base
    $diretorioBase = __DIR__ . '/src/';
    
    // Verificar se a classe usa o prefixo
    $tamanho = strlen($prefixo);
    if (strncmp($prefixo, $classe, $tamanho) !== 0) {
        return;
    }
    
    // Obter o nome relativo da classe
    $classeRelativa = substr($classe, $tamanho);
    
    // Converter namespace em caminho de arquivo
    $arquivo = $diretorioBase . str_replace('\\', '/', $classeRelativa) . '.php';
    
    // Se o arquivo existe, incluí-lo
    if (file_exists($arquivo)) {
        require $arquivo;
    }
});

// Agora pode usar classes sem require manual
$usuario = new App\Model\Usuario();
$service = new App\Service\UsuarioService();
```

### 25.5 Namespace e Funções/Constantes

```php
<?php

declare(strict_types=1);

namespace App\Helpers;

// Funções em namespace
function formatarMoeda(float $valor): string
{
    return 'R$ ' . number_format($valor, 2, ',', '.');
}

function limparCpf(string $cpf): string
{
    return preg_replace('/[^0-9]/', '', $cpf);
}

// Constantes em namespace
const VERSAO_APP = '1.0.0';
const AMBIENTE = 'producao';

// Usar funções e constantes
namespace App\Controller;

use function App\Helpers\formatarMoeda;
use function App\Helpers\limparCpf;
use const App\Helpers\VERSAO_APP;

class ProdutoController
{
    public function exibirPreco(float $preco): void
    {
        echo formatarMoeda($preco);  // R$ 1.234,56
    }
    
    public function versao(): string
    {
        return VERSAO_APP;  // 1.0.0
    }
}
```

### 25.6 Organização de Projeto Real

```php
<?php

declare(strict_types=1);

/**
 * Estrutura recomendada:
 * 
 * projeto/
 * ├── src/
 * │   ├── Controller/
 * │   │   ├── UsuarioController.php
 * │   │   └── ProdutoController.php
 * │   ├── Model/
 * │   │   ├── Usuario.php
 * │   │   └── Produto.php
 * │   ├── Repository/
 * │   │   ├── UsuarioRepository.php
 * │   │   └── ProdutoRepository.php
 * │   ├── Service/
 * │   │   ├── AuthService.php
 * │   │   └── EmailService.php
 * │   ├── Validator/
 * │   │   └── Validator.php
 * │   ├── Exception/
 * │   │   ├── ValidationException.php
 * │   │   └── NotFoundException.php
 * │   └── Helpers/
 * │       └── StringHelper.php
 * ├── config/
 * │   └── database.php
 * ├── public/
 * │   └── index.php
 * └── autoload.php
 */

// src/Controller/UsuarioController.php
namespace App\Controller;

use App\Service\AuthService;
use App\Repository\UsuarioRepository;
use App\Validator\Validator;
use App\Exception\ValidationException;

class UsuarioController
{
    public function __construct(
        private AuthService $auth,
        private UsuarioRepository $repository,
        private Validator $validator
    ) {}
}
```

### 📝 Exercícios do Capítulo 25

1. Organize projeto seguindo PSR-4
2. Crie autoloader customizado
3. Implemente helpers com funções em namespace
4. Resolva conflitos de nomes com aliases
5. Estruture aplicação MVC com namespaces

---

## Capítulo 26: OOP - Properties e Methods Estáticos

### 26.1 Static Properties (Propriedades Estáticas)

```php
<?php

declare(strict_types=1);

class Contador
{
    // Propriedade estática: compartilhada por todas as instâncias
    private static int $contagem = 0;
    
    public function __construct()
    {
        // Incrementar contador quando criar instância
        self::$contagem++;
    }
    
    public static function obterContagem(): int
    {
        return self::$contagem;
    }
    
    public static function resetar(): void
    {
        self::$contagem = 0;
    }
}

// Uso
echo Contador::obterContagem();  // 0

$obj1 = new Contador();
$obj2 = new Contador();
$obj3 = new Contador();

echo Contador::obterContagem();  // 3

Contador::resetar();
echo Contador::obterContagem();  // 0
```

### 26.2 Static Methods (Métodos Estáticos)

```php
<?php

declare(strict_types=1);

class Matematica
{
    // Métodos estáticos não acessam $this
    public static function somar(float $a, float $b): float
    {
        return $a + $b;
    }
    
    public static function multiplicar(float $a, float $b): float
    {
        return $a * $b;
    }
    
    public static function fatorial(int $n): int
    {
        if ($n <= 1) {
            return 1;
        }
        
        // Pode chamar outro método estático com self::
        return $n * self::fatorial($n - 1);
    }
}

// Uso sem criar objeto
echo Matematica::somar(5, 3);        // 8
echo Matematica::multiplicar(4, 7);   // 28
echo Matematica::fatorial(5);         // 120
```

### 26.3 Late Static Binding (static::)

```php
<?php

declare(strict_types=1);

class Animal
{
    protected static string $tipo = 'Animal genérico';
    
    public static function obterTipo(): string
    {
        // self:: refere à classe onde o método foi definido
        return self::$tipo;
    }
    
    public static function obterTipoCorreto(): string
    {
        // static:: refere à classe que chamou o método
        return static::$tipo;
    }
}

class Cachorro extends Animal
{
    protected static string $tipo = 'Cachorro';
}

class Gato extends Animal
{
    protected static string $tipo = 'Gato';
}

// Late static binding
echo Cachorro::obterTipo();         // Animal genérico (self)
echo Cachorro::obterTipoCorreto();  // Cachorro (static)

echo Gato::obterTipo();             // Animal genérico (self)
echo Gato::obterTipoCorreto();      // Gato (static)
```

### 26.4 Singleton Pattern

```php
<?php

declare(strict_types=1);

class Database
{
    private static ?Database $instancia = null;
    private PDO $conexao;
    
    // Construtor privado: não pode criar com new
    private function __construct()
    {
        $this->conexao = new PDO(
            'mysql:host=localhost;dbname=test',
            'user',
            'password'
        );
    }
    
    // Clone privado: não pode clonar
    private function __clone() {}
    
    // Unserialize privado: não pode deserializar
    public function __wakeup()
    {
        throw new Exception("Não pode deserializar singleton");
    }
    
    // Único método público para obter instância
    public static function obterInstancia(): self
    {
        if (self::$instancia === null) {
            self::$instancia = new self();
        }
        
        return self::$instancia;
    }
    
    public function getConexao(): PDO
    {
        return $this->conexao;
    }
}

// Uso
$db1 = Database::obterInstancia();
$db2 = Database::obterInstancia();

var_dump($db1 === $db2);  // true (mesma instância)

// $db = new Database();  // ERRO: construtor privado
// $clone = clone $db1;   // ERRO: clone privado
```

### 26.5 Factory Pattern com Static

```php
<?php

declare(strict_types=1);

interface LoggerInterface
{
    public function log(string $mensagem): void;
}

class FileLogger implements LoggerInterface
{
    public function __construct(
        private string $arquivo
    ) {}
    
    public function log(string $mensagem): void
    {
        file_put_contents(
            $this->arquivo,
            date('[Y-m-d H:i:s] ') . $mensagem . "\n",
            FILE_APPEND
        );
    }
}

class DatabaseLogger implements LoggerInterface
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function log(string $mensagem): void
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO logs (mensagem, criado_em) VALUES (?, NOW())"
        );
        $stmt->execute([$mensagem]);
    }
}

// Factory com métodos estáticos
class LoggerFactory
{
    public static function criarFileLogger(string $arquivo): LoggerInterface
    {
        return new FileLogger($arquivo);
    }
    
    public static function criarDatabaseLogger(PDO $pdo): LoggerInterface
    {
        return new DatabaseLogger($pdo);
    }
    
    public static function criar(string $tipo, mixed $config): LoggerInterface
    {
        return match($tipo) {
            'file' => self::criarFileLogger($config),
            'database' => self::criarDatabaseLogger($config),
            default => throw new InvalidArgumentException("Tipo inválido: $tipo")
        };
    }
}

// Uso
$logger = LoggerFactory::criar('file', 'app.log');
$logger->log('Aplicação iniciada');

$pdo = new PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
$dbLogger = LoggerFactory::criar('database', $pdo);
$dbLogger->log('Erro crítico');
```

### 26.6 Cache com Static

```php
<?php

declare(strict_types=1);

class Cache
{
    private static array $dados = [];
    
    public static function definir(string $chave, mixed $valor, int $ttl = 3600): void
    {
        self::$dados[$chave] = [
            'valor' => $valor,
            'expira' => time() + $ttl
        ];
    }
    
    public static function obter(string $chave): mixed
    {
        if (!isset(self::$dados[$chave])) {
            return null;
        }
        
        $item = self::$dados[$chave];
        
        // Verificar se expirou
        if (time() > $item['expira']) {
            unset(self::$dados[$chave]);
            return null;
        }
        
        return $item['valor'];
    }
    
    public static function existe(string $chave): bool
    {
        return self::obter($chave) !== null;
    }
    
    public static function deletar(string $chave): void
    {
        unset(self::$dados[$chave]);
    }
    
    public static function limpar(): void
    {
        self::$dados = [];
    }
    
    // Cache com callback
    public static function lembrar(string $chave, callable $callback, int $ttl = 3600): mixed
    {
        if (self::existe($chave)) {
            return self::obter($chave);
        }
        
        $valor = $callback();
        self::definir($chave, $valor, $ttl);
        
        return $valor;
    }
}

// Uso
Cache::definir('usuario:1', ['nome' => 'João', 'email' => 'joao@example.com']);

$usuario = Cache::obter('usuario:1');
print_r($usuario);

// Com callback
$produtos = Cache::lembrar('produtos:lista', function() {
    // Consulta pesada ao banco
    return [
        ['id' => 1, 'nome' => 'Produto 1'],
        ['id' => 2, 'nome' => 'Produto 2']
    ];
}, 3600);
```

### 📝 Exercícios do Capítulo 26

1. Implemente padrão Singleton para configuração
2. Crie Factory estática para diferentes tipos de notificação
3. Faça cache de consultas com métodos estáticos
4. Implemente contador de visitas com static
5. Use late static binding em hierarquia de classes

---

## Capítulo 27: OOP - Traits Avançado

### 27.1 Traits com Properties e Methods

```php
<?php

declare(strict_types=1);

trait Timestampable
{
    protected ?string $criadoEm = null;
    protected ?string $atualizadoEm = null;
    
    public function marcarComoCriado(): void
    {
        $this->criadoEm = date('Y-m-d H:i:s');
    }
    
    public function marcarComoAtualizado(): void
    {
        $this->atualizadoEm = date('Y-m-d H:i:s');
    }
    
    public function obterCriadoEm(): ?string
    {
        return $this->criadoEm;
    }
    
    public function obterAtualizadoEm(): ?string
    {
        return $this->atualizadoEm;
    }
}

trait SoftDeletable
{
    protected ?string $deletadoEm = null;
    
    public function deletar(): void
    {
        $this->deletadoEm = date('Y-m-d H:i:s');
    }
    
    public function restaurar(): void
    {
        $this->deletadoEm = null;
    }
    
    public function estaDeletado(): bool
    {
        return $this->deletadoEm !== null;
    }
}

// Usar múltiplos traits
class Usuario
{
    use Timestampable, SoftDeletable;
    
    public function __construct(
        public string $nome,
        public string $email
    ) {
        $this->marcarComoCriado();
    }
    
    public function atualizar(string $nome, string $email): void
    {
        $this->nome = $nome;
        $this->email = $email;
        $this->marcarComoAtualizado();
    }
}

// Uso
$usuario = new Usuario('João', 'joao@example.com');
echo $usuario->obterCriadoEm();  // 2026-02-09 15:30:00

$usuario->atualizar('João Silva', 'joao.silva@example.com');
echo $usuario->obterAtualizadoEm();  // 2026-02-09 15:31:00

$usuario->deletar();
echo $usuario->estaDeletado() ? 'Deletado' : 'Ativo';  // Deletado
```

### 27.2 Traits com Abstract Methods

```php
<?php

declare(strict_types=1);

trait Validavel
{
    protected array $erros = [];
    
    // Método abstrato que a classe deve implementar
    abstract protected function regrasValidacao(): array;
    
    public function validar(): bool
    {
        $this->erros = [];
        $regras = $this->regrasValidacao();
        
        foreach ($regras as $campo => $regra) {
            if (!$this->validarCampo($campo, $regra)) {
                $this->erros[$campo] = "Campo $campo inválido";
            }
        }
        
        return empty($this->erros);
    }
    
    private function validarCampo(string $campo, array $regra): bool
    {
        $valor = $this->$campo ?? null;
        
        if (in_array('required', $regra) && empty($valor)) {
            return false;
        }
        
        if (in_array('email', $regra)) {
            return filter_var($valor, FILTER_VALIDATE_EMAIL) !== false;
        }
        
        return true;
    }
    
    public function obterErros(): array
    {
        return $this->erros;
    }
}

class Usuario
{
    use Validavel;
    
    public function __construct(
        public string $nome,
        public string $email
    ) {}
    
    // Implementar método abstrato do trait
    protected function regrasValidacao(): array
    {
        return [
            'nome' => ['required'],
            'email' => ['required', 'email']
        ];
    }
}

// Uso
$usuario = new Usuario('', 'email-invalido');

if (!$usuario->validar()) {
    print_r($usuario->obterErros());
}
```

### 27.3 Resolução de Conflitos

```php
<?php

declare(strict_types=1);

trait Logger
{
    public function log(string $mensagem): void
    {
        echo "[LOG] $mensagem\n";
    }
    
    public function processar(): void
    {
        $this->log("Processando com Logger");
    }
}

trait Auditor
{
    public function log(string $mensagem): void
    {
        echo "[AUDIT] $mensagem\n";
    }
    
    public function processar(): void
    {
        $this->log("Processando com Auditor");
    }
}

class Servico
{
    use Logger, Auditor {
        // Resolver conflito: usar log() de Logger
        Logger::log insteadof Auditor;
        
        // Criar alias para log() de Auditor
        Auditor::log as auditLog;
        
        // Usar processar() de Logger
        Logger::processar insteadof Auditor;
    }
    
    public function executar(): void
    {
        $this->log("Mensagem normal");      // Logger::log
        $this->auditLog("Mensagem audit");  // Auditor::log
        $this->processar();                  // Logger::processar
    }
}

// Uso
$servico = new Servico();
$servico->executar();
/*
[LOG] Mensagem normal
[AUDIT] Mensagem audit
[LOG] Processando com Logger
*/
```

### 27.4 Traits Compostos

```php
<?php

declare(strict_types=1);

// Trait pode usar outro trait
trait Loggable
{
    protected function log(string $mensagem): void
    {
        echo "[LOG] $mensagem\n";
    }
}

trait Trackable
{
    use Loggable;
    
    protected array $operacoes = [];
    
    protected function rastrear(string $operacao): void
    {
        $this->operacoes[] = [
            'operacao' => $operacao,
            'timestamp' => time()
        ];
        
        $this->log("Operação rastreada: $operacao");
    }
    
    public function obterHistorico(): array
    {
        return $this->operacoes;
    }
}

class ContaBancaria
{
    use Trackable;
    
    private float $saldo = 0;
    
    public function depositar(float $valor): void
    {
        $this->saldo += $valor;
        $this->rastrear("Depósito de R$ $valor");
    }
    
    public function sacar(float $valor): void
    {
        $this->saldo -= $valor;
        $this->rastrear("Saque de R$ $valor");
    }
}

// Uso
$conta = new ContaBancaria();
$conta->depositar(1000);
$conta->sacar(200);

print_r($conta->obterHistorico());
```

### 27.5 Traits com Propriedades Estáticas

```php
<?php

declare(strict_types=1);

trait ContadorInstancias
{
    protected static int $numeroInstancias = 0;
    
    public function __construct()
    {
        self::$numeroInstancias++;
    }
    
    public static function obterNumeroInstancias(): int
    {
        return self::$numeroInstancias;
    }
    
    public static function resetarContador(): void
    {
        self::$numeroInstancias = 0;
    }
}

class Usuario
{
    use ContadorInstancias;
    
    public function __construct(
        public string $nome
    ) {
        // Chamar construtor do trait
        $this->__construct();
    }
}

class Produto
{
    use ContadorInstancias;
    
    public function __construct(
        public string $nome
    ) {
        $this->__construct();
    }
}

// Uso
$user1 = new Usuario('João');
$user2 = new Usuario('Maria');

echo Usuario::obterNumeroInstancias();  // 2

$prod1 = new Produto('Notebook');
$prod2 = new Produto('Mouse');
$prod3 = new Produto('Teclado');

echo Produto::obterNumeroInstancias();  // 3
```

### 📝 Exercícios do Capítulo 27

1. Crie trait Sluggable para gerar URLs amigáveis
2. Implemente trait Cacheable para cache automático
3. Faça trait Searchable com filtros e ordenação
4. Crie trait Exportable para exportar para JSON/CSV
5. Resolva conflitos complexos entre 3+ traits

---

## Capítulo 28: OOP - Interfaces Avançado

### 28.1 Interface Segregation

```php
<?php

declare(strict_types=1);

// ❌ Interface "gorda" (fat interface)
interface TrabalhadorGordoInterface
{
    public function trabalhar(): void;
    public function comer(): void;
    public function dormir(): void;
    public function programar(): void;
}

// ✅ Interfaces segregadas (pequenas e específicas)
interface TrabalhadorInterface
{
    public function trabalhar(): void;
}

interface AlimentavelInterface
{
    public function comer(): void;
}

interface DormivelInterface
{
    public function dormir(): void;
}

interface ProgramadorInterface
{
    public function programar(): void;
}

// Classes implementam apenas o que precisam
class Desenvolvedor implements TrabalhadorInterface, ProgramadorInterface
{
    public function trabalhar(): void
    {
        echo "Desenvolvedor trabalhando\n";
    }
    
    public function programar(): void
    {
        echo "Desenvolvedor programando\n";
    }
}

class Robo implements TrabalhadorInterface
{
    public function trabalhar(): void
    {
        echo "Robô trabalhando 24/7\n";
    }
    
    // Robô não come nem dorme!
}
```

### 28.2 Interface com Constantes

```php
<?php

declare(strict_types=1);

interface StatusPedidoInterface
{
    // Constantes públicas
    public const PENDENTE = 'pendente';
    public const PROCESSANDO = 'processando';
    public const ENVIADO = 'enviado';
    public const ENTREGUE = 'entregue';
    public const CANCELADO = 'cancelado';
    
    public function obterStatus(): string;
    public function atualizarStatus(string $novoStatus): void;
}

class Pedido implements StatusPedidoInterface
{
    private string $status = self::PENDENTE;
    
    public function obterStatus(): string
    {
        return $this->status;
    }
    
    public function atualizarStatus(string $novoStatus): void
    {
        $statusesValidos = [
            self::PENDENTE,
            self::PROCESSANDO,
            self::ENVIADO,
            self::ENTREGUE,
            self::CANCELADO
        ];
        
        if (!in_array($novoStatus, $statusesValidos)) {
            throw new InvalidArgumentException("Status inválido");
        }
        
        $this->status = $novoStatus;
    }
}

// Uso
$pedido = new Pedido();
$pedido->atualizarStatus(StatusPedidoInterface::PROCESSANDO);

echo $pedido->obterStatus();  // processando
```

### 28.3 Interfaces Fluentes

```php
<?php

declare(strict_types=1);

interface QueryBuilderInterface
{
    public function select(array $campos): self;
    public function from(string $tabela): self;
    public function where(string $condicao, mixed $valor): self;
    public function orderBy(string $campo, string $direcao = 'ASC'): self;
    public function limit(int $limite): self;
    public function executar(): array;
}

class QueryBuilder implements QueryBuilderInterface
{
    private array $campos = ['*'];
    private string $tabela = '';
    private array $wheres = [];
    private ?string $orderBy = null;
    private ?int $limit = null;
    
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function select(array $campos): self
    {
        $this->campos = $campos;
        return $this;
    }
    
    public function from(string $tabela): self
    {
        $this->tabela = $tabela;
        return $this;
    }
    
    public function where(string $condicao, mixed $valor): self
    {
        $this->wheres[] = [$condicao, $valor];
        return $this;
    }
    
    public function orderBy(string $campo, string $direcao = 'ASC'): self
    {
        $this->orderBy = "$campo $direcao";
        return $this;
    }
    
    public function limit(int $limite): self
    {
        $this->limit = $limite;
        return $this;
    }
    
    public function executar(): array
    {
        $sql = "SELECT " . implode(', ', $this->campos) . " FROM {$this->tabela}";
        
        if (!empty($this->wheres)) {
            $sql .= " WHERE " . $this->wheres[0][0];
        }
        
        if ($this->orderBy) {
            $sql .= " ORDER BY {$this->orderBy}";
        }
        
        if ($this->limit) {
            $sql .= " LIMIT {$this->limit}";
        }
        
        $stmt = $this->pdo->prepare($sql);
        
        if (!empty($this->wheres)) {
            $stmt->execute([$this->wheres[0][1]]);
        } else {
            $stmt->execute();
        }
        
        return $stmt->fetchAll();
    }
}

// Uso fluente
$pdo = new PDO('mysql:host=localhost;dbname=app', 'user', 'pass');
$qb = new QueryBuilder($pdo);

$usuarios = $qb
    ->select(['id', 'nome', 'email'])
    ->from('usuarios')
    ->where('ativo = ?', 1)
    ->orderBy('nome', 'ASC')
    ->limit(10)
    ->executar();
```

### 28.4 Contratos de Repositório

```php
<?php

declare(strict_types=1);

interface RepositorioInterface
{
    public function encontrar(int $id): ?array;
    public function encontrarTodos(): array;
    public function criar(array $dados): int;
    public function atualizar(int $id, array $dados): bool;
    public function deletar(int $id): bool;
}

interface RepositorioPesquisavelInterface
{
    public function pesquisar(array $criterios): array;
    public function pesquisarPorCampo(string $campo, mixed $valor): array;
}

interface RepositorioPaginavelInterface
{
    public function paginar(int $pagina, int $porPagina): array;
    public function contar(): int;
}

class UsuarioRepository implements 
    RepositorioInterface,
    RepositorioPesquisavelInterface,
    RepositorioPaginavelInterface
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function encontrar(int $id): ?array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM usuarios WHERE id = ?");
        $stmt->execute([$id]);
        $resultado = $stmt->fetch();
        
        return $resultado ?: null;
    }
    
    public function encontrarTodos(): array
    {
        return $this->pdo->query("SELECT * FROM usuarios")->fetchAll();
    }
    
    public function criar(array $dados): int
    {
        $stmt = $this->pdo->prepare(
            "INSERT INTO usuarios (nome, email) VALUES (?, ?)"
        );
        $stmt->execute([$dados['nome'], $dados['email']]);
        
        return (int) $this->pdo->lastInsertId();
    }
    
    public function atualizar(int $id, array $dados): bool
    {
        $stmt = $this->pdo->prepare(
            "UPDATE usuarios SET nome = ?, email = ? WHERE id = ?"
        );
        
        return $stmt->execute([$dados['nome'], $dados['email'], $id]);
    }
    
    public function deletar(int $id): bool
    {
        $stmt = $this->pdo->prepare("DELETE FROM usuarios WHERE id = ?");
        return $stmt->execute([$id]);
    }
    
    public function pesquisar(array $criterios): array
    {
        // Implementação de busca
        return [];
    }
    
    public function pesquisarPorCampo(string $campo, mixed $valor): array
    {
        $stmt = $this->pdo->prepare("SELECT * FROM usuarios WHERE $campo = ?");
        $stmt->execute([$valor]);
        
        return $stmt->fetchAll();
    }
    
    public function paginar(int $pagina, int $porPagina): array
    {
        $offset = ($pagina - 1) * $porPagina;
        
        $stmt = $this->pdo->prepare(
            "SELECT * FROM usuarios LIMIT ? OFFSET ?"
        );
        $stmt->bindValue(1, $porPagina, PDO::PARAM_INT);
        $stmt->bindValue(2, $offset, PDO::PARAM_INT);
        $stmt->execute();
        
        return $stmt->fetchAll();
    }
    
    public function contar(): int
    {
        return (int) $this->pdo->query("SELECT COUNT(*) FROM usuarios")->fetchColumn();
    }
}
```

### 📝 Exercícios do Capítulo 28

1. Crie interfaces para sistema de pagamento
2. Implemente Repository pattern com interfaces
3. Faça interfaces para diferentes tipos de cache
4. Crie interface fluente para email builder
5. Implemente Strategy pattern com interfaces

---

**Continua no próximo arquivo com:**
- Capítulo 29: Abstract Classes Avançado
- Capítulo 30: Class Constants
- Capítulo 31: Access Modifiers Detalhado
- Capítulo 32: Regular Expressions Completo
- Capítulo 33: Validação de Forms
- Capítulo 34: cURL

Deseja que eu continue criando o restante?

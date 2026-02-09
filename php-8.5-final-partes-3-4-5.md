# PHP 8.5 Moderno: Do Zero à Maestria
## Continuação - Partes III, IV e V (Final)

---

## Capítulo 14: Interfaces e Traits

### 14.1 Interfaces

```php
<?php

declare(strict_types=1);

// Interface define um contrato (apenas assinaturas de métodos)
interface Autenticavel
{
    public function autenticar(string $senha): bool;
    public function obterPermissoes(): array;
}

interface Notificavel
{
    public function enviarNotificacao(string $mensagem): void;
}

// Classe implementa interface(s)
class Usuario implements Autenticavel, Notificavel
{
    public function __construct(
        private string $nome,
        private string $senhaHash,
        private array $permissoes
    ) {}
    
    public function autenticar(string $senha): bool
    {
        return password_verify($senha, $this->senhaHash);
    }
    
    public function obterPermissoes(): array
    {
        return $this->permissoes;
    }
    
    public function enviarNotificacao(string $mensagem): void
    {
        echo "Enviando email para {$this->nome}: $mensagem\n";
    }
}

// Função que aceita qualquer Autenticavel
function realizarLogin(Autenticavel $entidade, string $senha): bool
{
    if ($entidade->autenticar($senha)) {
        $permissoes = $entidade->obterPermissoes();
        echo "Login realizado! Permissões: " . implode(', ', $permissoes);
        return true;
    }
    
    return false;
}

$usuario = new Usuario(
    "João",
    password_hash("senha123", PASSWORD_DEFAULT),
    ['ler', 'escrever']
);

realizarLogin($usuario, "senha123");
```

### 14.2 Traits

```php
<?php

declare(strict_types=1);

// Trait: código reutilizável que pode ser incluído em classes
trait Timestamp
{
    private ?string $criadoEm = null;
    private ?string $atualizadoEm = null;
    
    public function registrarCriacao(): void
    {
        $this->criadoEm = date('Y-m-d H:i:s');
    }
    
    public function registrarAtualizacao(): void
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

trait Validavel
{
    protected array $erros = [];
    
    abstract protected function validar(): bool;
    
    public function isValido(): bool
    {
        $this->erros = [];
        return $this->validar();
    }
    
    public function obterErros(): array
    {
        return $this->erros;
    }
    
    protected function adicionarErro(string $campo, string $mensagem): void
    {
        $this->erros[$campo] = $mensagem;
    }
}

// Usando múltiplos traits
class Usuario
{
    use Timestamp, Validavel;
    
    public function __construct(
        private string $nome,
        private string $email
    ) {
        $this->registrarCriacao();
    }
    
    public function atualizar(string $nome, string $email): void
    {
        $this->nome = $nome;
        $this->email = $email;
        $this->registrarAtualizacao();
    }
    
    protected function validar(): bool
    {
        if (strlen($this->nome) < 3) {
            $this->adicionarErro('nome', 'Nome muito curto');
            return false;
        }
        
        if (!filter_var($this->email, FILTER_VALIDATE_EMAIL)) {
            $this->adicionarErro('email', 'Email inválido');
            return false;
        }
        
        return true;
    }
}

$usuario = new Usuario("João", "joao@example.com");

if ($usuario->isValido()) {
    echo "Usuário válido!";
    echo "Criado em: " . $usuario->obterCriadoEm();
} else {
    print_r($usuario->obterErros());
}
```

### 14.3 Conflitos em Traits

```php
<?php

declare(strict_types=1);

trait TraitA
{
    public function hello(): string
    {
        return "Hello from A";
    }
    
    public function metodoUnico(): string
    {
        return "Único de A";
    }
}

trait TraitB
{
    public function hello(): string
    {
        return "Hello from B";
    }
}

class MinhaClasse
{
    use TraitA, TraitB {
        // Resolver conflito: usar método de TraitA
        TraitA::hello insteadof TraitB;
        
        // Criar alias para método de TraitB
        TraitB::hello as helloB;
    }
}

$obj = new MinhaClasse();
echo $obj->hello();   // Hello from A
echo $obj->helloB();  // Hello from B
echo $obj->metodoUnico();  // Único de A
```

### 14.4 Exemplo Prático Completo

```php
<?php

declare(strict_types=1);

// Trait para logging
trait Loggable
{
    private array $logs = [];
    
    protected function log(string $mensagem): void
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
}

// Trait para serialização
trait JsonSerializable
{
    public function paraJson(): string
    {
        return json_encode($this->toArray());
    }
    
    abstract protected function toArray(): array;
}

// Interface de repositório
interface RepositorioInterface
{
    public function salvar(mixed $entidade): void;
    public function buscar(int $id): mixed;
    public function deletar(int $id): void;
}

// Classe de domínio usando traits
class Produto
{
    use Loggable, JsonSerializable;
    
    public function __construct(
        private int $id,
        private string $nome,
        private float $preco
    ) {
        $this->log("Produto criado: {$nome}");
    }
    
    public function atualizarPreco(float $novoPreco): void
    {
        $precoAntigo = $this->preco;
        $this->preco = $novoPreco;
        $this->log("Preço alterado de {$precoAntigo} para {$novoPreco}");
    }
    
    protected function toArray(): array
    {
        return [
            'id' => $this->id,
            'nome' => $this->nome,
            'preco' => $this->preco
        ];
    }
}

// Uso
$produto = new Produto(1, "Notebook", 2500.00);
$produto->atualizarPreco(2800.00);

echo $produto->paraJson();
// {"id":1,"nome":"Notebook","preco":2800}

print_r($produto->obterLogs());
/*
Array (
    [0] => Array ( [timestamp] => 1704067200, [mensagem] => Produto criado: Notebook )
    [1] => Array ( [timestamp] => 1704067200, [mensagem] => Preço alterado de 2500 para 2800 )
)
*/
```

### 📝 Exercícios do Capítulo 14

1. Crie interface Pagavel com métodos processar e validar
2. Faça trait Sluggable que gera URL-friendly strings
3. Implemente trait SoftDelete para exclusão lógica de registros
4. Crie múltiplas interfaces para um sistema de e-commerce
5. Resolva conflitos entre traits com métodos duplicados

---

## Capítulo 15: OOP Avançado

### 15.1 Type Hinting e Return Types

```php
<?php

declare(strict_types=1);

class UsuarioService
{
    // Type hint para parâmetros
    public function criar(string $nome, int $idade, bool $ativo = true): Usuario
    {
        return new Usuario($nome, $idade, $ativo);
    }
    
    // Union types (PHP 8.0+)
    public function buscar(int|string $identificador): ?Usuario
    {
        if (is_int($identificador)) {
            // Buscar por ID
        } else {
            // Buscar por email
        }
        
        return null;
    }
    
    // Array de tipos específicos (PHPDoc)
    /**
     * @param array<Usuario> $usuarios
     * @return array<string>
     */
    public function extrairNomes(array $usuarios): array
    {
        return array_map(fn($u) => $u->nome, $usuarios);
    }
    
    // Never: função nunca retorna (sempre lança exceção ou termina)
    public function abortar(string $mensagem): never
    {
        throw new RuntimeException($mensagem);
    }
    
    // Void: não retorna nada
    public function notificar(Usuario $usuario): void
    {
        echo "Notificando {$usuario->nome}";
    }
}

class Usuario
{
    public function __construct(
        public string $nome,
        public int $idade,
        public bool $ativo
    ) {}
}
```

### 15.2 Enums (PHP 8.1+)

```php
<?php

declare(strict_types=1);

// Enum pura
enum Status
{
    case PENDENTE;
    case PROCESSANDO;
    case CONCLUIDO;
    case CANCELADO;
}

// Backed enum (com valores)
enum StatusPedido: string
{
    case PENDENTE = 'pendente';
    case PAGO = 'pago';
    case ENVIADO = 'enviado';
    case ENTREGUE = 'entregue';
    case CANCELADO = 'cancelado';
    
    // Métodos no enum
    public function label(): string
    {
        return match($this) {
            self::PENDENTE => 'Aguardando Pagamento',
            self::PAGO => 'Pagamento Confirmado',
            self::ENVIADO => 'Em Transporte',
            self::ENTREGUE => 'Entregue',
            self::CANCELADO => 'Cancelado',
        };
    }
    
    public function podeAlterar(): bool
    {
        return match($this) {
            self::PENDENTE, self::PAGO => true,
            default => false,
        };
    }
}

class Pedido
{
    public function __construct(
        private int $id,
        private StatusPedido $status
    ) {}
    
    public function atualizarStatus(StatusPedido $novoStatus): void
    {
        if (!$this->status->podeAlterar()) {
            throw new Exception("Não pode alterar status: " . $this->status->label());
        }
        
        $this->status = $novoStatus;
    }
    
    public function obterStatus(): StatusPedido
    {
        return $this->status;
    }
}

// Uso
$pedido = new Pedido(1, StatusPedido::PENDENTE);
echo $pedido->obterStatus()->label();  // Aguardando Pagamento

$pedido->atualizarStatus(StatusPedido::PAGO);
echo $pedido->obterStatus()->value;  // pago

// Iterar sobre enum
foreach (StatusPedido::cases() as $status) {
    echo $status->name . ": " . $status->label() . "\n";
}
```

### 15.3 Métodos Mágicos

```php
<?php

declare(strict_types=1);

class Usuario
{
    private array $dados = [];
    
    // __construct: construtor
    public function __construct(array $dados = [])
    {
        $this->dados = $dados;
    }
    
    // __get: acesso a propriedade inexistente
    public function __get(string $nome): mixed
    {
        return $this->dados[$nome] ?? null;
    }
    
    // __set: atribuição a propriedade inexistente
    public function __set(string $nome, mixed $valor): void
    {
        $this->dados[$nome] = $valor;
    }
    
    // __isset: verificar se propriedade existe
    public function __isset(string $nome): bool
    {
        return isset($this->dados[$nome]);
    }
    
    // __unset: remover propriedade
    public function __unset(string $nome): void
    {
        unset($this->dados[$nome]);
    }
    
    // __toString: converter para string
    public function __toString(): string
    {
        return json_encode($this->dados);
    }
    
    // __call: chamar método inexistente
    public function __call(string $metodo, array $argumentos): mixed
    {
        echo "Método $metodo não existe\n";
        return null;
    }
    
    // __invoke: usar objeto como função
    public function __invoke(string $mensagem): string
    {
        return "Usuário diz: $mensagem";
    }
    
    // __clone: clonar objeto
    public function __clone(): void
    {
        $this->dados['id'] = null;  // Remover ID do clone
    }
    
    // __serialize: controlar serialização
    public function __serialize(): array
    {
        return $this->dados;
    }
    
    // __unserialize: controlar desserialização
    public function __unserialize(array $dados): void
    {
        $this->dados = $dados;
    }
}

// Uso
$user = new Usuario(['nome' => 'João', 'idade' => 25]);

echo $user->nome;  // João (__get)
$user->email = 'joao@example.com';  // __set

echo $user;  // {"nome":"João","idade":25,"email":"joao@example.com"} (__toString)

echo $user("Olá!");  // Usuário diz: Olá! (__invoke)

$clone = clone $user;  // __clone
```

### 15.4 Namespaces

```php
<?php

declare(strict_types=1);

// Definir namespace
namespace App\Model;

class Usuario
{
    public string $nome;
}

// Em outro arquivo
namespace App\Service;

// Importar classe de outro namespace
use App\Model\Usuario;

class UsuarioService
{
    public function criar(string $nome): Usuario
    {
        $usuario = new Usuario();
        $usuario->nome = $nome;
        return $usuario;
    }
}

// Usar múltiplas classes
namespace App\Controller;

use App\Model\Usuario;
use App\Service\UsuarioService;
use App\Repository\UsuarioRepository;

// Alias para evitar conflito
use App\Model\Admin as AdminModel;
use App\Service\AdminService;

class UsuarioController
{
    public function __construct(
        private UsuarioService $service,
        private UsuarioRepository $repository
    ) {}
}

// Namespace global
namespace;

// Acessar classe sem namespace
$data = new DateTime();

// Acessar classe de namespace
$usuario = new App\Model\Usuario();
```

### 15.5 Autoloading (PSR-4)

```php
<?php

declare(strict_types=1);

// spl_autoload_register: registrar autoloader personalizado
spl_autoload_register(function (string $classe) {
    // App\Model\Usuario -> app/Model/Usuario.php
    
    // Remover namespace base
    $prefixo = 'App\\';
    
    if (strpos($classe, $prefixo) === 0) {
        $classeRelativa = substr($classe, strlen($prefixo));
        
        // Converter namespace para caminho
        $arquivo = __DIR__ . '/app/' . str_replace('\\', '/', $classeRelativa) . '.php';
        
        if (file_exists($arquivo)) {
            require $arquivo;
        }
    }
});

// Agora pode usar classes sem require manual
$usuario = new App\Model\Usuario();
$service = new App\Service\UsuarioService();
```

### 📝 Exercícios do Capítulo 15

1. Crie enums para dias da semana e meses do ano
2. Implemente métodos mágicos __get e __set com validação
3. Organize classes em namespaces seguindo PSR-4
4. Crie autoloader personalizado para seu projeto
5. Use union types em um sistema de pagamentos

---

# Parte IV: PHP 8.5 Moderno

## Capítulo 16: Recursos do PHP 8.0-8.4

### 16.1 PHP 8.0: Named Arguments

```php
<?php

declare(strict_types=1);

function criarUsuario(
    string $nome,
    string $email,
    int $idade = 18,
    bool $ativo = true,
    string $pais = 'Brasil'
): array {
    return compact('nome', 'email', 'idade', 'ativo', 'pais');
}

// Forma tradicional
$usuario1 = criarUsuario("João", "joao@example.com", 25, true, "Brasil");

// Named arguments: ordem não importa
$usuario2 = criarUsuario(
    email: "maria@example.com",
    nome: "Maria",
    pais: "Portugal",
    idade: 30
);

// Pular parâmetros opcionais
$usuario3 = criarUsuario(
    nome: "Pedro",
    email: "pedro@example.com",
    pais: "Argentina"
);
```

### 16.2 PHP 8.0: Match Expression

```php
<?php

declare(strict_types=1);

// Match: mais rigoroso que switch
$statusCode = 404;

$mensagem = match ($statusCode) {
    200 => 'OK',
    201 => 'Criado',
    400 => 'Requisição Inválida',
    401 => 'Não Autorizado',
    404 => 'Não Encontrado',
    500 => 'Erro do Servidor',
    default => 'Código Desconhecido'
};

echo $mensagem;  // Não Encontrado

// Match com expressões
$idade = 20;

$categoria = match (true) {
    $idade < 13 => 'Criança',
    $idade < 18 => 'Adolescente',
    $idade < 60 => 'Adulto',
    default => 'Idoso'
};

// Match com múltiplos valores
$dia = 'sabado';

$tipo = match ($dia) {
    'segunda', 'terca', 'quarta', 'quinta', 'sexta' => 'Dia útil',
    'sabado', 'domingo' => 'Final de semana',
};
```

### 16.3 PHP 8.0: Nullsafe Operator

```php
<?php

declare(strict_types=1);

class Usuario
{
    public function __construct(
        public ?Endereco $endereco = null
    ) {}
}

class Endereco
{
    public function __construct(
        public ?string $cidade = null
    ) {}
}

$usuario = new Usuario();

// Forma antiga
$cidade = null;
if ($usuario->endereco !== null) {
    $cidade = $usuario->endereco->cidade;
}

// Nullsafe operator (?->)
$cidade = $usuario->endereco?->cidade;

// Encadeamento
$usuario = new Usuario(new Endereco("São Paulo"));
$cidade = $usuario->endereco?->cidade ?? "Não informada";
echo $cidade;  // São Paulo
```

### 16.4 PHP 8.1: Readonly Properties

```php
<?php

declare(strict_types=1);

class Produto
{
    public function __construct(
        public readonly int $id,
        public readonly string $nome,
        public float $preco  // Não readonly
    ) {}
}

$produto = new Produto(1, "Notebook", 2500.00);

echo $produto->id;  // 1
$produto->preco = 2800.00;  // OK

// $produto->id = 2;  // ERRO: não pode modificar readonly
// $produto->nome = "Mouse";  // ERRO
```

### 16.5 PHP 8.1: Enumerations

```php
<?php

declare(strict_types=1);

enum TipoPagamento: string
{
    case CREDITO = 'credito';
    case DEBITO = 'debito';
    case PIX = 'pix';
    case BOLETO = 'boleto';
    
    public function temJuros(): bool
    {
        return $this === self::CREDITO;
    }
    
    public function prazoMaximo(): int
    {
        return match($this) {
            self::CREDITO => 12,
            self::DEBITO, self::PIX => 1,
            self::BOLETO => 30,
        };
    }
}

function processarPagamento(TipoPagamento $tipo, float $valor): void
{
    echo "Processando {$tipo->value} no valor de R$ {$valor}\n";
    
    if ($tipo->temJuros()) {
        echo "Este pagamento pode ter juros\n";
    }
    
    echo "Prazo máximo: {$tipo->prazoMaximo()} dias\n";
}

processarPagamento(TipoPagamento::CREDITO, 1000.00);
```

### 16.6 PHP 8.2: Readonly Classes

```php
<?php

declare(strict_types=1);

// Todas as propriedades são readonly
readonly class Configuracao
{
    public function __construct(
        public string $appName,
        public string $version,
        public bool $debug,
        public array $database
    ) {}
}

$config = new Configuracao(
    appName: "Meu App",
    version: "1.0.0",
    debug: true,
    database: ['host' => 'localhost', 'port' => 3306]
);

// Nenhuma propriedade pode ser modificada
// $config->appName = "Outro";  // ERRO
// $config->debug = false;  // ERRO
```

### 16.7 PHP 8.4: Property Hooks

```php
<?php

declare(strict_types=1);

class Usuario
{
    // Property hook get
    public string $nomeCompleto {
        get => strtoupper($this->nomeCompleto);
    }
    
    // Property hook set com validação
    public string $email {
        set {
            if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
                throw new InvalidArgumentException("Email inválido");
            }
            $this->email = $value;
        }
    }
    
    // Ambos hooks
    private float $salario;
    
    public float $salarioFormatado {
        get => $this->salario;
        set {
            if ($value < 0) {
                throw new InvalidArgumentException("Salário não pode ser negativo");
            }
            $this->salario = $value;
        }
    }
    
    public function __construct(string $nomeCompleto, string $email, float $salario)
    {
        $this->nomeCompleto = $nomeCompleto;
        $this->email = $email;
        $this->salarioFormatado = $salario;
    }
}

$user = new Usuario("joão silva", "joao@example.com", 5000.00);
echo $user->nomeCompleto;  // JOÃO SILVA (hook get)
```

### 📝 Exercícios do Capítulo 16

1. Refatore código switch usando match
2. Use nullsafe operator em cadeia de objetos nullable
3. Crie enums para status de pedido com métodos
4. Implemente classes readonly para configuração
5. Use property hooks para normalizar dados

---

## Capítulo 17: Novidades do PHP 8.5

### 17.1 Pipe Operator (|>)

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
// Composição funcional da esquerda para direita

// Forma tradicional (difícil de ler)
$resultado = array_map(
    fn($x) => $x * 2,
    array_filter(
        array_map('trim', $valores),
        fn($x) => $x !== ''
    )
);

// Com pipe operator (leitura natural)
$resultado = $valores
    |> array_map('trim', ...)
    |> array_filter(..., fn($x) => $x !== '')
    |> array_map(fn($x) => $x * 2, ...);

// Exemplo prático: processamento de texto
function processarTexto(string $texto): string
{
    return $texto
        |> trim(...)
        |> strtolower(...)
        |> ucfirst(...);
}

echo processarTexto("  OLÁ MUNDO  ");  // Olá mundo

// Com funções personalizadas
function removerAcentos(string $texto): string
{
    return iconv('UTF-8', 'ASCII//TRANSLIT', $texto);
}

function criarSlug(string $texto): string
{
    return $texto
        |> trim(...)
        |> strtolower(...)
        |> removerAcentos(...)
        |> preg_replace('/[^a-z0-9]+/', '-', ...)
        |> trim(..., '-');
}

echo criarSlug("Título do Artigo!");  // titulo-do-artigo
```

### 17.2 array_first() e array_last()

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
$produtos = [
    ['id' => 1, 'nome' => 'Notebook', 'preco' => 2500],
    ['id' => 2, 'nome' => 'Mouse', 'preco' => 50],
    ['id' => 3, 'nome' => 'Teclado', 'preco' => 150],
];

// Forma antiga (verbosa e pode dar erro)
$primeiro = !empty($produtos) ? $produtos[0] : null;
$ultimo = !empty($produtos) ? $produtos[count($produtos) - 1] : null;

// Nova forma (limpa e segura)
$primeiro = array_first($produtos);
$ultimo = array_last($produtos);

print_r($primeiro);  // ['id' => 1, 'nome' => 'Notebook', ...]
print_r($ultimo);    // ['id' => 3, 'nome' => 'Teclado', ...]

// Retorna null se vazio
$vazio = [];
$resultado = array_first($vazio);
var_dump($resultado);  // NULL

// Uso prático
function obterPrimeiroUsuario(): ?array
{
    $usuarios = buscarUsuarios();
    return array_first($usuarios);
}

function obterUltimoLog(): ?string
{
    $logs = file('app.log');
    return array_last($logs);
}
```

### 17.3 Clone with

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
// Clonar objeto com propriedades modificadas

readonly class Usuario
{
    public function __construct(
        public int $id,
        public string $nome,
        public string $email,
        public int $idade
    ) {}
}

$usuario = new Usuario(1, "João", "joao@example.com", 25);

// Forma antiga (não funciona com readonly)
// $atualizado = clone $usuario;
// $atualizado->idade = 26;  // ERRO

// Nova forma: clone with
$atualizado = clone($usuario, ['idade' => 26]);

echo $usuario->idade;     // 25 (original)
echo $atualizado->idade;  // 26 (clone)

// Múltiplas propriedades
$modificado = clone($usuario, [
    'nome' => 'João Silva',
    'idade' => 26
]);

// Uso prático: padrão Builder
class ProdutoBuilder
{
    private function __construct(
        private readonly string $nome = '',
        private readonly float $preco = 0.0,
        private readonly int $estoque = 0,
        private readonly array $tags = []
    ) {}
    
    public static function criar(): self
    {
        return new self();
    }
    
    public function comNome(string $nome): self
    {
        return clone($this, ['nome' => $nome]);
    }
    
    public function comPreco(float $preco): self
    {
        return clone($this, ['preco' => $preco]);
    }
    
    public function comEstoque(int $estoque): self
    {
        return clone($this, ['estoque' => $estoque]);
    }
    
    public function construir(): Produto
    {
        return new Produto($this->nome, $this->preco, $this->estoque);
    }
}

// Uso fluente
$produto = ProdutoBuilder::criar()
    ->comNome("Notebook")
    ->comPreco(2500.00)
    ->comEstoque(10)
    ->construir();
```

### 17.4 Atributo #[\NoDiscard]

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
// Força uso do valor de retorno

class PaymentGateway
{
    /**
     * Processa pagamento - retorno DEVE ser verificado
     */
    #[\NoDiscard]
    public function processPayment(float $amount): PaymentResult
    {
        // Simular processamento
        return new PaymentResult(
            success: true,
            transactionId: 'TXN' . rand(1000, 9999)
        );
    }
    
    /**
     * Validar cartão - retorno DEVE ser verificado
     */
    #[\NoDiscard]
    public function validateCard(string $cardNumber): bool
    {
        return strlen($cardNumber) === 16;
    }
}

class PaymentResult
{
    public function __construct(
        public readonly bool $success,
        public readonly string $transactionId
    ) {}
}

$gateway = new PaymentGateway();

// ❌ PHPStan/Psalm alertarão sobre isso
$gateway->processPayment(100.00);  // Warning: Resultado não usado!

// ✅ Forma correta
$result = $gateway->processPayment(100.00);
if (!$result->success) {
    throw new Exception("Pagamento falhou");
}

// ✅ Ou usar diretamente em condição
if ($gateway->validateCard("1234567890123456")) {
    echo "Cartão válido";
}
```

### 17.5 URI Extension Nativa

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
use Uri\Rfc3986\Uri;

// Forma antiga
$partes = parse_url('https://example.com/path?query=1#hash');
$scheme = $partes['scheme'] ?? null;  // Pode falhar

// Nova forma: OOP com validação
$uri = new Uri('https://user:pass@example.com:8080/path?query=1#hash');

echo $uri->getScheme();      // "https"
echo $uri->getUserInfo();    // "user:pass"
echo $uri->getHost();        // "example.com"
echo $uri->getPort();        // 8080
echo $uri->getPath();        // "/path"
echo $uri->getQuery();       // "query=1"
echo $uri->getFragment();    // "hash"

// Manipulação imutável
$novaUri = $uri
    ->withHost('novo.example.com')
    ->withPath('/novo-path')
    ->withQuery('nova=query');

echo (string) $novaUri;
// https://user:pass@novo.example.com:8080/novo-path?nova=query#hash

// Validação automática
try {
    $invalida = new Uri('http://exemplo inválido.com');
} catch (InvalidArgumentException $e) {
    echo "URI inválida: " . $e->getMessage();
}

// Uso prático
function construirUrlApi(string $endpoint, array $params = []): string
{
    $base = new Uri('https://api.exemplo.com/v1');
    
    $uri = $base
        ->withPath($base->getPath() . '/' . $endpoint);
    
    if (!empty($params)) {
        $uri = $uri->withQuery(http_build_query($params));
    }
    
    return (string) $uri;
}

echo construirUrlApi('usuarios', ['page' => 1, 'limit' => 10]);
// https://api.exemplo.com/v1/usuarios?page=1&limit=10
```

### 17.6 Closures em Expressões Constantes

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
// Closures podem ser usadas em constantes de classe

class Calculadora
{
    // Closure em constante
    public const DOBRAR = fn(int $x) => $x * 2;
    public const TRIPLICAR = fn(int $x) => $x * 3;
    
    public const OPERACOES = [
        'dobrar' => self::DOBRAR,
        'triplicar' => self::TRIPLICAR,
        'quadrado' => fn(int $x) => $x * $x
    ];
}

// Uso
$dobrar = Calculadora::DOBRAR;
echo $dobrar(5);  // 10

$operacao = Calculadora::OPERACOES['quadrado'];
echo $operacao(5);  // 25

// Exemplo prático: Validadores
class Validadores
{
    public const EMAIL = fn(string $v) => filter_var($v, FILTER_VALIDATE_EMAIL) !== false;
    public const CPF = fn(string $v) => preg_match('/^\d{3}\.\d{3}\.\d{3}-\d{2}$/', $v);
    public const TELEFONE = fn(string $v) => preg_match('/^\(\d{2}\) \d{4,5}-\d{4}$/', $v);
    
    public static function validar(string $tipo, string $valor): bool
    {
        return match($tipo) {
            'email' => (self::EMAIL)($valor),
            'cpf' => (self::CPF)($valor),
            'telefone' => (self::TELEFONE)($valor),
            default => false
        };
    }
}

echo Validadores::validar('email', 'teste@example.com') ? 'Válido' : 'Inválido';
```

### 📝 Exercícios do Capítulo 17

1. Refatore código aninhado usando pipe operator
2. Implemente funções seguras de array com array_first/last
3. Crie value objects imutáveis usando clone with
4. Adicione #[\NoDiscard] em métodos críticos
5. Parse e manipule URLs usando a URI extension

---

## Capítulo 18: Sistema de Tipos Avançado

### 18.1 Union Types

```php
<?php

declare(strict_types=1);

// Aceita múltiplos tipos
function processar(int|float|string $valor): int|float
{
    return is_string($valor) ? (int) $valor : $valor * 2;
}

echo processar(5);      // 10
echo processar(5.5);    // 11.0
echo processar("10");   // 10

// Em propriedades
class Configuracao
{
    public function __construct(
        public int|string $porta,
        public bool|int $debug
    ) {}
}

$config = new Configuracao(porta: 8080, debug: true);
$config2 = new Configuracao(porta: "8080", debug: 1);
```

### 18.2 Intersection Types (PHP 8.1+)

```php
<?php

declare(strict_types=1);

interface Autenticavel
{
    public function autenticar(): bool;
}

interface Rastreavel
{
    public function obterLogs(): array;
}

// Deve implementar AMBAS interfaces
function processar(Autenticavel&Rastreavel $objeto): void
{
    if ($objeto->autenticar()) {
        $logs = $objeto->obterLogs();
        // processar
    }
}

class Usuario implements Autenticavel, Rastreavel
{
    public function autenticar(): bool
    {
        return true;
    }
    
    public function obterLogs(): array
    {
        return ['login', 'logout'];
    }
}

$user = new Usuario();
processar($user);  // OK
```

### 18.3 Mixed Type

```php
<?php

declare(strict_types=1);

// mixed: aceita qualquer tipo (exceto void)
function processar(mixed $valor): mixed
{
    if (is_int($valor)) {
        return $valor * 2;
    }
    
    if (is_string($valor)) {
        return strtoupper($valor);
    }
    
    if (is_array($valor)) {
        return count($valor);
    }
    
    return null;
}

echo processar(5);           // 10
echo processar("olá");       // OLÁ
echo processar([1, 2, 3]);   // 3
```

### 18.4 Never Type

```php
<?php

declare(strict_types=1);

// never: função nunca retorna normalmente
function abortar(string $mensagem): never
{
    throw new RuntimeException($mensagem);
}

function loopInfinito(): never
{
    while (true) {
        // código
    }
}

function sair(): never
{
    exit(1);
}

// Útil para indicar que código após nunca será executado
function processar(mixed $valor): int
{
    if (!is_int($valor)) {
        abortar("Valor deve ser inteiro");
        // Código aqui nunca executará
    }
    
    return $valor * 2;
}
```

### 📝 Exercícios do Capítulo 18

1. Use union types em função que aceita ID numérico ou string
2. Crie intersection type para objeto que é Serializable e Comparable
3. Implemente função com mixed que processa diferentes tipos
4. Use never em funções de erro e validação
5. Combine union e nullable types em sistema de configuração

---

# Parte V: Projetos Práticos

## Capítulo 19: Formulários e Validação

### 19.1 Processamento de Formulário Básico

```php
<?php

declare(strict_types=1);

// formulario.html
/*
<!DOCTYPE html>
<html>
<body>
    <form method="POST" action="processar.php">
        <input type="text" name="nome" required>
        <input type="email" name="email" required>
        <input type="number" name="idade" min="18" max="100">
        <button type="submit">Enviar</button>
    </form>
</body>
</html>
*/

// processar.php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Obter dados
    $nome = $_POST['nome'] ?? '';
    $email = $_POST['email'] ?? '';
    $idade = (int) ($_POST['idade'] ?? 0);
    
    // Validar
    $erros = [];
    
    if (strlen($nome) < 3) {
        $erros['nome'] = 'Nome deve ter pelo menos 3 caracteres';
    }
    
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $erros['email'] = 'Email inválido';
    }
    
    if ($idade < 18 || $idade > 100) {
        $erros['idade'] = 'Idade deve estar entre 18 e 100';
    }
    
    // Se tiver erros, exibir
    if (!empty($erros)) {
        foreach ($erros as $campo => $mensagem) {
            echo "<p><strong>$campo:</strong> $mensagem</p>";
        }
    } else {
        echo "Dados válidos!<br>";
        echo "Nome: $nome<br>";
        echo "Email: $email<br>";
        echo "Idade: $idade<br>";
    }
}
```

### 19.2 Classe de Validação Reutilizável

```php
<?php

declare(strict_types=1);

class Validator
{
    private array $erros = [];
    
    public function validar(array $dados, array $regras): bool
    {
        $this->erros = [];
        
        foreach ($regras as $campo => $regrasCampo) {
            $valor = $dados[$campo] ?? null;
            
            foreach ($regrasCampo as $regra) {
                $this->aplicarRegra($campo, $valor, $regra);
            }
        }
        
        return empty($this->erros);
    }
    
    private function aplicarRegra(string $campo, mixed $valor, string $regra): void
    {
        [$nomeRegra, $parametro] = $this->parseRegra($regra);
        
        $valido = match($nomeRegra) {
            'required' => !empty($valor),
            'email' => filter_var($valor, FILTER_VALIDATE_EMAIL) !== false,
            'min' => strlen($valor) >= (int) $parametro,
            'max' => strlen($valor) <= (int) $parametro,
            'numeric' => is_numeric($valor),
            'integer' => filter_var($valor, FILTER_VALIDATE_INT) !== false,
            default => true
        };
        
        if (!$valido) {
            $this->erros[$campo][] = $this->getMensagem($nomeRegra, $campo, $parametro);
        }
    }
    
    private function parseRegra(string $regra): array
    {
        if (strpos($regra, ':') !== false) {
            return explode(':', $regra, 2);
        }
        
        return [$regra, null];
    }
    
    private function getMensagem(string $regra, string $campo, ?string $param): string
    {
        return match($regra) {
            'required' => "O campo $campo é obrigatório",
            'email' => "O campo $campo deve ser um email válido",
            'min' => "O campo $campo deve ter no mínimo $param caracteres",
            'max' => "O campo $campo deve ter no máximo $param caracteres",
            'numeric' => "O campo $campo deve ser numérico",
            'integer' => "O campo $campo deve ser um número inteiro",
            default => "O campo $campo é inválido"
        };
    }
    
    public function obterErros(): array
    {
        return $this->erros;
    }
}

// Uso
$validator = new Validator();

$dados = [
    'nome' => 'João',
    'email' => 'joao@example.com',
    'idade' => '25',
    'senha' => '123'
];

$regras = [
    'nome' => ['required', 'min:3', 'max:100'],
    'email' => ['required', 'email'],
    'idade' => ['required', 'integer'],
    'senha' => ['required', 'min:8']
];

if ($validator->validar($dados, $regras)) {
    echo "Todos os dados são válidos!";
} else {
    print_r($validator->obterErros());
}
```

### 19.3 Proteção CSRF

```php
<?php

declare(strict_types=1);

session_start();

class CsrfProtection
{
    private const TOKEN_KEY = 'csrf_token';
    
    public static function gerarToken(): string
    {
        if (!isset($_SESSION[self::TOKEN_KEY])) {
            $_SESSION[self::TOKEN_KEY] = bin2hex(random_bytes(32));
        }
        
        return $_SESSION[self::TOKEN_KEY];
    }
    
    public static function validarToken(string $token): bool
    {
        if (!isset($_SESSION[self::TOKEN_KEY])) {
            return false;
        }
        
        return hash_equals($_SESSION[self::TOKEN_KEY], $token);
    }
    
    public static function campoInput(): string
    {
        $token = self::gerarToken();
        return sprintf(
            '<input type="hidden" name="csrf_token" value="%s">',
            htmlspecialchars($token, ENT_QUOTES, 'UTF-8')
        );
    }
}

// No formulário
/*
<form method="POST">
    <?php echo CsrfProtection::campoInput(); ?>
    <input type="text" name="nome">
    <button type="submit">Enviar</button>
</form>
*/

// No processamento
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $token = $_POST['csrf_token'] ?? '';
    
    if (!CsrfProtection::validarToken($token)) {
        die("Token CSRF inválido!");
    }
    
    // Processar formulário
    echo "Token válido, processando...";
}
```

### 📝 Exercícios do Capítulo 19

1. Crie formulário de cadastro com validação completa
2. Implemente validação de CPF personalizada
3. Adicione proteção CSRF a todos os formulários
4. Crie validador de senha forte (maiúsculas, números, símbolos)
5. Faça formulário de upload com validação de tipo e tamanho

---

## Capítulo 20: Sessões e Cookies

### 20.1 Trabalhando com Sessões

```php
<?php

declare(strict_types=1);

// Iniciar sessão
session_start();

// Armazenar dados
$_SESSION['usuario_id'] = 123;
$_SESSION['usuario_nome'] = 'João Silva';
$_SESSION['usuario_email'] = 'joao@example.com';

// Ler dados
$nome = $_SESSION['usuario_nome'] ?? 'Visitante';
echo "Olá, $nome!";

// Verificar se está logado
function estaLogado(): bool
{
    return isset($_SESSION['usuario_id']);
}

if (estaLogado()) {
    echo "Usuário está logado";
} else {
    echo "Usuário não está logado";
}

// Destruir sessão (logout)
function logout(): void
{
    $_SESSION = [];
    
    if (ini_get('session.use_cookies')) {
        $params = session_get_cookie_params();
        setcookie(
            session_name(),
            '',
            time() - 42000,
            $params['path'],
            $params['domain'],
            $params['secure'],
            $params['httponly']
        );
    }
    
    session_destroy();
}
```

### 20.2 Sistema de Autenticação Simples

```php
<?php

declare(strict_types=1);

session_start();

class Auth
{
    private const SESSION_USER_KEY = 'usuario_autenticado';
    
    public static function login(int $userId, array $userData): void
    {
        // Regenerar ID de sessão (segurança)
        session_regenerate_id(true);
        
        $_SESSION[self::SESSION_USER_KEY] = [
            'id' => $userId,
            'nome' => $userData['nome'],
            'email' => $userData['email'],
            'timestamp' => time()
        ];
    }
    
    public static function logout(): void
    {
        unset($_SESSION[self::SESSION_USER_KEY]);
        session_destroy();
    }
    
    public static function estaLogado(): bool
    {
        return isset($_SESSION[self::SESSION_USER_KEY]);
    }
    
    public static function obterUsuario(): ?array
    {
        return $_SESSION[self::SESSION_USER_KEY] ?? null;
    }
    
    public static function obterUserId(): ?int
    {
        return $_SESSION[self::SESSION_USER_KEY]['id'] ?? null;
    }
    
    public static function requererLogin(): void
    {
        if (!self::estaLogado()) {
            header('Location: login.php');
            exit;
        }
    }
}

// Uso em página de login
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $email = $_POST['email'] ?? '';
    $senha = $_POST['senha'] ?? '';
    
    // Buscar usuário no banco (simulado)
    $usuario = [
        'id' => 1,
        'nome' => 'João Silva',
        'email' => 'joao@example.com',
        'senha_hash' => password_hash('senha123', PASSWORD_DEFAULT)
    ];
    
    if ($email === $usuario['email'] && password_verify($senha, $usuario['senha_hash'])) {
        Auth::login($usuario['id'], $usuario);
        header('Location: dashboard.php');
        exit;
    } else {
        echo "Credenciais inválidas!";
    }
}

// Uso em página protegida
// dashboard.php
Auth::requererLogin();

$usuario = Auth::obterUsuario();
echo "Bem-vindo, {$usuario['nome']}!";
```

### 20.3 Cookies

```php
<?php

declare(strict_types=1);

// Criar cookie
setcookie(
    'preferencia_tema',
    'dark',
    time() + (86400 * 30),  // 30 dias
    '/',                     // Path
    '',                      // Domain
    true,                    // Secure (HTTPS only)
    true                     // HttpOnly (não acessível por JS)
);

// Ler cookie
$tema = $_COOKIE['preferencia_tema'] ?? 'light';
echo "Tema atual: $tema";

// Deletar cookie
setcookie('preferencia_tema', '', time() - 3600, '/');

// Classe para gerenciar cookies
class Cookie
{
    public static function definir(
        string $nome,
        string $valor,
        int $diasExpiracao = 30,
        bool $httpOnly = true,
        bool $secure = true
    ): void {
        setcookie(
            $nome,
            $valor,
            time() + ($diasExpiracao * 86400),
            '/',
            '',
            $secure,
            $httpOnly
        );
    }
    
    public static function obter(string $nome, string $padrao = ''): string
    {
        return $_COOKIE[$nome] ?? $padrao;
    }
    
    public static function existe(string $nome): bool
    {
        return isset($_COOKIE[$nome]);
    }
    
    public static function deletar(string $nome): void
    {
        setcookie($nome, '', time() - 3600, '/');
        unset($_COOKIE[$nome]);
    }
}

// Uso
Cookie::definir('idioma', 'pt-BR', 365);
$idioma = Cookie::obter('idioma', 'en-US');

if (Cookie::existe('ultima_visita')) {
    echo "Última visita: " . Cookie::obter('ultima_visita');
}

Cookie::deletar('cookie_antigo');
```

### 📝 Exercícios do Capítulo 20

1. Implemente sistema completo de login/logout
2. Crie "lembrar-me" usando cookies seguros
3. Faça carrinho de compras usando sessões
4. Implemente timeout de sessão por inatividade
5. Crie sistema de preferências do usuário com cookies

---

## Capítulo 21: Banco de Dados com PDO

### 21.1 Conexão e Configuração

```php
<?php

declare(strict_types=1);

class Database
{
    private static ?PDO $conexao = null;
    
    public static function conectar(): PDO
    {
        if (self::$conexao === null) {
            $host = 'localhost';
            $db = 'meu_banco';
            $user = 'usuario';
            $pass = 'senha';
            
            $dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";
            
            self::$conexao = new PDO(
                $dsn,
                $user,
                $pass,
                [
                    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                    PDO::ATTR_EMULATE_PREPARES => false,
                ]
            );
        }
        
        return self::$conexao;
    }
}

// Uso
$pdo = Database::conectar();
```

### 21.2 CRUD Completo

```php
<?php

declare(strict_types=1);

class UsuarioRepository
{
    private PDO $pdo;
    
    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    // CREATE
    public function criar(array $dados): int
    {
        $sql = "INSERT INTO usuarios (nome, email, senha_hash, criado_em) 
                VALUES (:nome, :email, :senha_hash, NOW())";
        
        $stmt = $this->pdo->prepare($sql);
        
        $stmt->execute([
            'nome' => $dados['nome'],
            'email' => $dados['email'],
            'senha_hash' => password_hash($dados['senha'], PASSWORD_DEFAULT)
        ]);
        
        return (int) $this->pdo->lastInsertId();
    }
    
    // READ (um)
    public function buscarPorId(int $id): ?array
    {
        $sql = "SELECT * FROM usuarios WHERE id = :id";
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['id' => $id]);
        
        $resultado = $stmt->fetch();
        
        return $resultado ?: null;
    }
    
    // READ (todos)
    public function buscarTodos(): array
    {
        $sql = "SELECT * FROM usuarios ORDER BY nome ASC";
        
        $stmt = $this->pdo->query($sql);
        
        return $stmt->fetchAll();
    }
    
    // READ (com filtro)
    public function buscarPorEmail(string $email): ?array
    {
        $sql = "SELECT * FROM usuarios WHERE email = :email";
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['email' => $email]);
        
        $resultado = $stmt->fetch();
        
        return $resultado ?: null;
    }
    
    // UPDATE
    public function atualizar(int $id, array $dados): bool
    {
        $sql = "UPDATE usuarios 
                SET nome = :nome, email = :email, atualizado_em = NOW() 
                WHERE id = :id";
        
        $stmt = $this->pdo->prepare($sql);
        
        return $stmt->execute([
            'id' => $id,
            'nome' => $dados['nome'],
            'email' => $dados['email']
        ]);
    }
    
    // DELETE
    public function deletar(int $id): bool
    {
        $sql = "DELETE FROM usuarios WHERE id = :id";
        
        $stmt = $this->pdo->prepare($sql);
        
        return $stmt->execute(['id' => $id]);
    }
    
    // Paginação
    public function paginar(int $pagina = 1, int $porPagina = 10): array
    {
        $offset = ($pagina - 1) * $porPagina;
        
        $sql = "SELECT * FROM usuarios 
                ORDER BY id DESC 
                LIMIT :limit OFFSET :offset";
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->bindValue(':limit', $porPagina, PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
        $stmt->execute();
        
        return $stmt->fetchAll();
    }
    
    // Contagem total
    public function contar(): int
    {
        $sql = "SELECT COUNT(*) FROM usuarios";
        
        $stmt = $this->pdo->query($sql);
        
        return (int) $stmt->fetchColumn();
    }
}

// Uso
$pdo = Database::conectar();
$repo = new UsuarioRepository($pdo);

// Criar
$id = $repo->criar([
    'nome' => 'João Silva',
    'email' => 'joao@example.com',
    'senha' => 'senha123'
]);

// Buscar
$usuario = $repo->buscarPorId($id);
print_r($usuario);

// Atualizar
$repo->atualizar($id, [
    'nome' => 'João Silva Santos',
    'email' => 'joao.santos@example.com'
]);

// Listar com paginação
$usuarios = $repo->paginar(1, 10);
$total = $repo->contar();

// Deletar
$repo->deletar($id);
```

### 21.3 Transações

```php
<?php

declare(strict_types=1);

class PedidoService
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function criarPedido(int $usuarioId, array $itens): int
    {
        try {
            // Iniciar transação
            $this->pdo->beginTransaction();
            
            // 1. Criar pedido
            $sql = "INSERT INTO pedidos (usuario_id, total, status) 
                    VALUES (:usuario_id, :total, 'pendente')";
            
            $stmt = $this->pdo->prepare($sql);
            
            $total = array_sum(array_column($itens, 'preco'));
            
            $stmt->execute([
                'usuario_id' => $usuarioId,
                'total' => $total
            ]);
            
            $pedidoId = (int) $this->pdo->lastInsertId();
            
            // 2. Inserir itens do pedido
            $sql = "INSERT INTO pedido_itens (pedido_id, produto_id, quantidade, preco) 
                    VALUES (:pedido_id, :produto_id, :quantidade, :preco)";
            
            $stmt = $this->pdo->prepare($sql);
            
            foreach ($itens as $item) {
                $stmt->execute([
                    'pedido_id' => $pedidoId,
                    'produto_id' => $item['produto_id'],
                    'quantidade' => $item['quantidade'],
                    'preco' => $item['preco']
                ]);
                
                // 3. Atualizar estoque
                $this->atualizarEstoque($item['produto_id'], $item['quantidade']);
            }
            
            // Confirmar transação
            $this->pdo->commit();
            
            return $pedidoId;
            
        } catch (Exception $e) {
            // Reverter em caso de erro
            $this->pdo->rollBack();
            
            throw new RuntimeException(
                "Erro ao criar pedido: " . $e->getMessage(),
                0,
                $e
            );
        }
    }
    
    private function atualizarEstoque(int $produtoId, int $quantidade): void
    {
        $sql = "UPDATE produtos 
                SET estoque = estoque - :quantidade 
                WHERE id = :id AND estoque >= :quantidade";
        
        $stmt = $this->pdo->prepare($sql);
        
        $stmt->execute([
            'id' => $produtoId,
            'quantidade' => $quantidade
        ]);
        
        if ($stmt->rowCount() === 0) {
            throw new RuntimeException("Estoque insuficiente para produto $produtoId");
        }
    }
}
```

### 📝 Exercícios do Capítulo 21

1. Crie sistema CRUD completo para produtos
2. Implemente busca com múltiplos filtros
3. Faça sistema de relacionamento (usuário tem pedidos)
4. Implemente soft delete (exclusão lógica)
5. Crie relatório com agrupamento e somas

---

## Capítulo 22: Segurança

### 22.1 Prevenção de SQL Injection

```php
<?php

declare(strict_types=1);

// ❌ NUNCA FAÇA ISSO
function buscarUsuarioInseguro(string $email): ?array
{
    $pdo = Database::conectar();
    
    // VULNERÁVEL A SQL INJECTION!
    $sql = "SELECT * FROM usuarios WHERE email = '$email'";
    
    return $pdo->query($sql)->fetch() ?: null;
}

// Atacante pode passar: ' OR '1'='1
// Resultado: SELECT * FROM usuarios WHERE email = '' OR '1'='1'
// Retorna TODOS os usuários!

// ✅ SEMPRE USE PREPARED STATEMENTS
function buscarUsuarioSeguro(string $email): ?array
{
    $pdo = Database::conectar();
    
    $sql = "SELECT * FROM usuarios WHERE email = :email";
    $stmt = $pdo->prepare($sql);
    $stmt->execute(['email' => $email]);
    
    return $stmt->fetch() ?: null;
}
```

### 22.2 Prevenção de XSS

```php
<?php

declare(strict_types=1);

// ❌ NUNCA FAÇA ISSO
$nome = $_GET['nome'];
echo "Olá, $nome!";

// Atacante pode passar: <script>alert('XSS')</script>

// ✅ SEMPRE ESCAPE OUTPUT
$nome = $_GET['nome'] ?? 'Visitante';
echo "Olá, " . htmlspecialchars($nome, ENT_QUOTES, 'UTF-8') . "!";

// Função helper
function e(string $texto): string
{
    return htmlspecialchars($texto, ENT_QUOTES, 'UTF-8');
}

echo "Olá, " . e($nome) . "!";
```

### 22.3 Senhas Seguras

```php
<?php

declare(strict_types=1);

class SenhaService
{
    // Hash de senha
    public function hash(string $senha): string
    {
        return password_hash($senha, PASSWORD_ARGON2ID, [
            'memory_cost' => 65536,
            'time_cost' => 4,
            'threads' => 3
        ]);
    }
    
    // Verificar senha
    public function verificar(string $senha, string $hash): bool
    {
        return password_verify($senha, $hash);
    }
    
    // Verificar se precisa rehash (algoritmo mudou)
    public function precisaRehash(string $hash): bool
    {
        return password_needs_rehash($hash, PASSWORD_ARGON2ID, [
            'memory_cost' => 65536,
            'time_cost' => 4,
            'threads' => 3
        ]);
    }
    
    // Validar força da senha
    public function validarForca(string $senha): array
    {
        $erros = [];
        
        if (strlen($senha) < 8) {
            $erros[] = 'Senha deve ter pelo menos 8 caracteres';
        }
        
        if (!preg_match('/[A-Z]/', $senha)) {
            $erros[] = 'Senha deve conter pelo menos uma letra maiúscula';
        }
        
        if (!preg_match('/[a-z]/', $senha)) {
            $erros[] = 'Senha deve conter pelo menos uma letra minúscula';
        }
        
        if (!preg_match('/[0-9]/', $senha)) {
            $erros[] = 'Senha deve conter pelo menos um número';
        }
        
        if (!preg_match('/[^A-Za-z0-9]/', $senha)) {
            $erros[] = 'Senha deve conter pelo menos um caractere especial';
        }
        
        return $erros;
    }
}

// Uso no cadastro
$senhaService = new SenhaService();

$senha = $_POST['senha'] ?? '';

$erros = $senhaService->validarForca($senha);

if (!empty($erros)) {
    foreach ($erros as $erro) {
        echo $erro . "<br>";
    }
} else {
    $hash = $senhaService->hash($senha);
    // Salvar $hash no banco
}

// Uso no login
$senhaDigitada = $_POST['senha'] ?? '';
$hashDoBanco = '...';  // Buscar do banco

if ($senhaService->verificar($senhaDigitada, $hashDoBanco)) {
    // Login bem-sucedido
    
    // Verificar se precisa atualizar hash
    if ($senhaService->precisaRehash($hashDoBanco)) {
        $novoHash = $senhaService->hash($senhaDigitada);
        // Atualizar no banco
    }
}
```

### 22.4 Rate Limiting Simples

```php
<?php

declare(strict_types=1);

class RateLimiter
{
    private string $diretorioLogs = 'logs/rate_limit';
    
    public function __construct()
    {
        if (!is_dir($this->diretorioLogs)) {
            mkdir($this->diretorioLogs, 0755, true);
        }
    }
    
    public function permitir(
        string $identificador,
        int $maxTentativas = 5,
        int $janelaTempo = 60
    ): bool {
        $arquivo = $this->obterArquivo($identificador);
        
        $tentativas = $this->lerTentativas($arquivo);
        
        // Remover tentativas antigas
        $agora = time();
        $tentativasValidas = array_filter(
            $tentativas,
            fn($timestamp) => ($agora - $timestamp) < $janelaTempo
        );
        
        // Verificar limite
        if (count($tentativasValidas) >= $maxTentativas) {
            return false;
        }
        
        // Registrar nova tentativa
        $tentativasValidas[] = $agora;
        $this->salvarTentativas($arquivo, $tentativasValidas);
        
        return true;
    }
    
    private function obterArquivo(string $identificador): string
    {
        $hash = md5($identificador);
        return $this->diretorioLogs . '/' . $hash . '.json';
    }
    
    private function lerTentativas(string $arquivo): array
    {
        if (!file_exists($arquivo)) {
            return [];
        }
        
        $json = file_get_contents($arquivo);
        return json_decode($json, true) ?? [];
    }
    
    private function salvarTentativas(string $arquivo, array $tentativas): void
    {
        file_put_contents($arquivo, json_encode($tentativas));
    }
}

// Uso em formulário de login
$limiter = new RateLimiter();

$ip = $_SERVER['REMOTE_ADDR'];

if (!$limiter->permitir("login:$ip", 5, 300)) {  // 5 tentativas em 5 minutos
    die("Muitas tentativas de login. Aguarde 5 minutos.");
}

// Processar login...
```

### 📝 Exercícios do Capítulo 22

1. Implemente validação completa de entrada de dados
2. Crie sistema de token de redefinição de senha
3. Adicione rate limiting em API
4. Faça auditoria de ações do usuário (log de atividades)
5. Implemente 2FA (autenticação de dois fatores) simples

---

## Capítulo 23: Projeto Final - Sistema Completo

### 23.1 Estrutura do Projeto

```
projeto/
├── public/
│   ├── index.php
│   ├── login.php
│   ├── logout.php
│   ├── produtos/
│   │   ├── index.php
│   │   ├── criar.php
│   │   ├── editar.php
│   │   └── deletar.php
│   └── css/
│       └── estilo.css
├── src/
│   ├── Database.php
│   ├── Auth.php
│   ├── Validator.php
│   ├── Repository/
│   │   ├── UsuarioRepository.php
│   │   └── ProdutoRepository.php
│   └── Service/
│       ├── AuthService.php
│       └── ProdutoService.php
├── config/
│   └── database.php
└── autoload.php
```

### 23.2 Autoloader

```php
<?php
// autoload.php

declare(strict_types=1);

spl_autoload_register(function (string $classe) {
    $arquivo = __DIR__ . '/src/' . str_replace('\\', '/', $classe) . '.php';
    
    if (file_exists($arquivo)) {
        require $arquivo;
    }
});
```

### 23.3 Configuração

```php
<?php
// config/database.php

declare(strict_types=1);

return [
    'host' => 'localhost',
    'database' => 'meu_sistema',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4'
];
```

### 23.4 Classe Database

```php
<?php
// src/Database.php

declare(strict_types=1);

class Database
{
    private static ?PDO $conexao = null;
    
    public static function conectar(): PDO
    {
        if (self::$conexao === null) {
            $config = require __DIR__ . '/../config/database.php';
            
            $dsn = sprintf(
                "mysql:host=%s;dbname=%s;charset=%s",
                $config['host'],
                $config['database'],
                $config['charset']
            );
            
            self::$conexao = new PDO(
                $dsn,
                $config['username'],
                $config['password'],
                [
                    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                    PDO::ATTR_EMULATE_PREPARES => false,
                ]
            );
        }
        
        return self::$conexao;
    }
}
```

### 23.5 Sistema de Templates Simples

```php
<?php
// src/View.php

declare(strict_types=1);

class View
{
    public static function render(string $template, array $dados = []): void
    {
        extract($dados);
        
        require __DIR__ . "/../views/header.php";
        require __DIR__ . "/../views/{$template}.php";
        require __DIR__ . "/../views/footer.php";
    }
}
```

### 23.6 Página Inicial

```php
<?php
// public/index.php

declare(strict_types=1);

require __DIR__ . '/../autoload.php';

session_start();

Auth::requererLogin();

$usuario = Auth::obterUsuario();

View::render('dashboard', [
    'usuario' => $usuario,
    'titulo' => 'Dashboard'
]);
```

### 📝 Projeto Final Completo

**Crie um sistema de gerenciamento de produtos com:**

1. ✅ Autenticação (login/logout)
2. ✅ CRUD de produtos
3. ✅ Validação de dados
4. ✅ Proteção CSRF
5. ✅ Paginação de listagem
6. ✅ Upload de imagem de produto
7. ✅ Busca e filtros
8. ✅ Sessões seguras
9. ✅ Prepared statements
10. ✅ Interface responsiva

---

## Conclusão

Parabéns! Você completou o guia completo de **PHP 8.5 Moderno**! 🎉

### O que você aprendeu:

✅ **Fundamentos:** Variáveis, tipos, operadores, estruturas de controle  
✅ **Arrays e Funções:** Manipulação de dados e reutilização de código  
✅ **Strings e Datas:** Processamento de texto e temporalidade  
✅ **Arquivos e JSON:** Persistência e troca de dados  
✅ **OOP Completo:** Classes, herança, interfaces, traits  
✅ **PHP 8.5 Features:** Pipe operator, array_first/last, clone with, URI  
✅ **Projetos Práticos:** Formulários, sessões, banco de dados, segurança  

### Próximos Passos:

1. 📚 **Pratique:** Construa projetos reais  
2. 🔒 **Segurança:** Aprofunde-se em OWASP Top 10  
3. 🧪 **Testes:** Aprenda PHPUnit  
4. 🚀 **Frameworks:** Explore Laravel ou Symfony  
5. 📊 **Performance:** Estude otimização e caching  

**Continue programando e evoluindo! 💻🚀**

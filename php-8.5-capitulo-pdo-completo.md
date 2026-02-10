# PHP 8.5 Moderno: PDO - PHP Data Objects (Completo)

**Capítulo Especial: Dominando PDO para Banco de Dados**

---

## Índice do Capítulo

1. [Introdução ao PDO](#351-introdução-ao-pdo)
2. [PDO::__construct - Conexão](#352-pdo__construct---conexão)
3. [PDO::connect - Conexão Alternativa](#353-pdoconnect---conexão-alternativa)
4. [PDO::getAttribute e setAttribute](#354-pdogetattribute-e-setattribute)
5. [PDO::getAvailableDrivers](#355-pdogetavailabledrivers)
6. [PDO::prepare - Prepared Statements](#356-pdoprepare---prepared-statements)
7. [PDO::query - Queries Diretas](#357-pdoquery---queries-diretas)
8. [PDO::exec - Execução Direta](#358-pdoexec---execução-direta)
9. [PDO::quote - Escapar Strings](#359-pdoquote---escapar-strings)
10. [PDO::lastInsertId](#3510-pdolastinsertid)
11. [PDO::errorCode e errorInfo](#3511-pdoerrorcode-e-errorinfo)
12. [Transações (beginTransaction, commit, rollBack)](#3512-transações)
13. [PDO::inTransaction](#3513-pdointransaction)
14. [Projeto Prático Completo](#3514-projeto-prático-completo)

---

## Capítulo 35: PDO - PHP Data Objects

### 35.1 Introdução ao PDO

PDO (PHP Data Objects) é uma extensão que fornece uma interface consistente para acesso a diferentes bancos de dados.

```php
<?php

declare(strict_types=1);

/**
 * Por que usar PDO?
 * 
 * ✅ Funciona com 12+ bancos diferentes (MySQL, PostgreSQL, SQLite, etc)
 * ✅ Prepared Statements nativos (proteção contra SQL Injection)
 * ✅ Orientado a objetos
 * ✅ Tratamento de erros robusto
 * ✅ Transações
 * ✅ Melhor performance
 */

// Bancos suportados:
// - MySQL
// - PostgreSQL
// - SQLite
// - Oracle
// - Microsoft SQL Server
// - Firebird
// - ODBC
// - IBM DB2
// E outros...
```

---

### 35.2 PDO::__construct - Conexão

O construtor cria uma conexão com o banco de dados.

```php
<?php

declare(strict_types=1);

/**
 * Sintaxe:
 * new PDO(string $dsn, ?string $username = null, ?string $password = null, ?array $options = null)
 * 
 * DSN (Data Source Name): string de conexão específica do banco
 */

// Exemplo 1: MySQL
try {
    $pdo = new PDO(
        'mysql:host=localhost;dbname=meu_banco;charset=utf8mb4',
        'usuario',
        'senha'
    );
    
    echo "Conectado ao MySQL com sucesso!\n";
} catch (PDOException $e) {
    die("Erro na conexão: " . $e->getMessage());
}

// Exemplo 2: PostgreSQL
$pdo = new PDO(
    'pgsql:host=localhost;port=5432;dbname=meu_banco',
    'usuario',
    'senha'
);

// Exemplo 3: SQLite (arquivo local)
$pdo = new PDO('sqlite:/caminho/para/banco.db');

// Exemplo 4: SQLite (em memória)
$pdo = new PDO('sqlite::memory:');

// Exemplo 5: Com opções de configuração
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
    PDO::ATTR_PERSISTENT => false,
];

$pdo = new PDO(
    'mysql:host=localhost;dbname=test;charset=utf8mb4',
    'root',
    'senha',
    $options
);
```

### Classe de Conexão Reutilizável

```php
<?php

declare(strict_types=1);

class Database
{
    private static ?PDO $instance = null;
    
    private function __construct()
    {
        // Construtor privado (Singleton)
    }
    
    public static function getConnection(): PDO
    {
        if (self::$instance === null) {
            $dsn = 'mysql:host=localhost;dbname=app;charset=utf8mb4';
            $username = 'root';
            $password = '';
            
            $options = [
                // Modo de erro: lançar exceções
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                
                // Modo de busca padrão: array associativo
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                
                // Desabilitar emulação de prepared statements
                PDO::ATTR_EMULATE_PREPARES => false,
                
                // Não usar conexão persistente
                PDO::ATTR_PERSISTENT => false,
                
                // Timeout de conexão
                PDO::ATTR_TIMEOUT => 5,
            ];
            
            try {
                self::$instance = new PDO($dsn, $username, $password, $options);
            } catch (PDOException $e) {
                error_log("Erro de conexão PDO: " . $e->getMessage());
                throw new RuntimeException("Não foi possível conectar ao banco de dados");
            }
        }
        
        return self::$instance;
    }
    
    public static function disconnect(): void
    {
        self::$instance = null;
    }
}

// Uso
$pdo = Database::getConnection();
```

---

### 35.3 PDO::connect - Conexão Alternativa

```php
<?php

declare(strict_types=1);

/**
 * PDO::connect() é um método estático introduzido no PHP 8.4
 * para criar subclasses PDO específicas de drivers
 */

// MySQL específico
$mysql = PDO::connect('mysql:host=localhost;dbname=test', 'user', 'pass');

// PostgreSQL específico
$pgsql = PDO::connect('pgsql:host=localhost;dbname=test', 'user', 'pass');

// Equivalente ao new PDO() mas retorna subclasse quando disponível
```

---

### 35.4 PDO::getAttribute e setAttribute

Recuperar e definir atributos de configuração da conexão.

```php
<?php

declare(strict_types=1);

$pdo = new PDO('mysql:host=localhost;dbname=test', 'root', '');

// ========================================
// DEFINIR ATRIBUTOS (setAttribute)
// ========================================

// 1. Modo de erro
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
// Opções:
// - PDO::ERRMODE_SILENT (padrão, retorna false)
// - PDO::ERRMODE_WARNING (emite E_WARNING)
// - PDO::ERRMODE_EXCEPTION (lança PDOException)

// 2. Modo de busca padrão
$pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
// Opções:
// - PDO::FETCH_ASSOC (array associativo)
// - PDO::FETCH_NUM (array numérico)
// - PDO::FETCH_BOTH (ambos)
// - PDO::FETCH_OBJ (objeto stdClass)
// - PDO::FETCH_CLASS (instância de classe)

// 3. Emulação de prepared statements
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
// false = usar prepared statements nativos (recomendado)
// true = emular no PHP

// 4. Case das colunas
$pdo->setAttribute(PDO::ATTR_CASE, PDO::CASE_NATURAL);
// - PDO::CASE_NATURAL (mantém original)
// - PDO::CASE_LOWER (minúsculas)
// - PDO::CASE_UPPER (maiúsculas)

// 5. Conversão de NULL
$pdo->setAttribute(PDO::ATTR_ORACLE_NULLS, PDO::NULL_NATURAL);
// - PDO::NULL_NATURAL (mantém NULL)
// - PDO::NULL_EMPTY_STRING (converte "" para NULL)
// - PDO::NULL_TO_STRING (converte NULL para "")

// 6. Autocommit (MySQL)
$pdo->setAttribute(PDO::ATTR_AUTOCOMMIT, true);

// 7. Timeout
$pdo->setAttribute(PDO::ATTR_TIMEOUT, 30);

// ========================================
// OBTER ATRIBUTOS (getAttribute)
// ========================================

// Modo de erro atual
$errorMode = $pdo->getAttribute(PDO::ATTR_ERRMODE);
echo "Modo de erro: $errorMode\n";

// Nome do driver
$driver = $pdo->getAttribute(PDO::ATTR_DRIVER_NAME);
echo "Driver: $driver\n";  // mysql, pgsql, sqlite, etc

// Versão do servidor
$serverVersion = $pdo->getAttribute(PDO::ATTR_SERVER_VERSION);
echo "Versão do servidor: $serverVersion\n";

// Versão do cliente
$clientVersion = $pdo->getAttribute(PDO::ATTR_CLIENT_VERSION);
echo "Versão do cliente: $clientVersion\n";

// Status da conexão
$connectionStatus = $pdo->getAttribute(PDO::ATTR_CONNECTION_STATUS);
echo "Status: $connectionStatus\n";

// Informações do servidor
$serverInfo = $pdo->getAttribute(PDO::ATTR_SERVER_INFO);
echo "Info: $serverInfo\n";

// Case das colunas
$case = $pdo->getAttribute(PDO::ATTR_CASE);
echo "Case: $case\n";

// Exemplo completo de configuração
function configurarPDO(PDO $pdo): void
{
    // Exceções para erros
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    
    // Array associativo por padrão
    $pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
    
    // Prepared statements nativos
    $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
    
    // Manter case original
    $pdo->setAttribute(PDO::ATTR_CASE, PDO::CASE_NATURAL);
    
    // Manter NULL como NULL
    $pdo->setAttribute(PDO::ATTR_ORACLE_NULLS, PDO::NULL_NATURAL);
}
```

---

### 35.5 PDO::getAvailableDrivers

Retorna array com drivers PDO disponíveis.

```php
<?php

declare(strict_types=1);

// Obter drivers disponíveis
$drivers = PDO::getAvailableDrivers();

echo "Drivers PDO instalados:\n";
foreach ($drivers as $driver) {
    echo "- $driver\n";
}

// Exemplo de saída:
// - mysql
// - sqlite
// - pgsql

// Verificar se driver específico está disponível
function driverDisponivel(string $driver): bool
{
    return in_array($driver, PDO::getAvailableDrivers(), true);
}

if (driverDisponivel('mysql')) {
    echo "MySQL disponível!\n";
} else {
    echo "MySQL não está instalado\n";
}

// Classe helper
class PDODriverChecker
{
    public static function verificarRequisitos(array $driversNecessarios): array
    {
        $disponiveis = PDO::getAvailableDrivers();
        $faltando = [];
        
        foreach ($driversNecessarios as $driver) {
            if (!in_array($driver, $disponiveis, true)) {
                $faltando[] = $driver;
            }
        }
        
        return $faltando;
    }
}

// Uso
$necessarios = ['mysql', 'pgsql', 'sqlite'];
$faltando = PDODriverChecker::verificarRequisitos($necessarios);

if (!empty($faltando)) {
    echo "Instale os drivers: " . implode(', ', $faltando) . "\n";
}
```

---

### 35.6 PDO::prepare - Prepared Statements

Prepara uma instrução SQL para execução. **SEMPRE use para queries com dados do usuário!**

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// PREPARED STATEMENTS COM PLACEHOLDERS NOMEADOS
// ========================================

// 1. Preparar
$sql = "SELECT * FROM usuarios WHERE email = :email AND ativo = :ativo";
$stmt = $pdo->prepare($sql);

// 2. Executar com dados
$stmt->execute([
    'email' => 'joao@example.com',
    'ativo' => 1
]);

// 3. Buscar resultados
$usuario = $stmt->fetch();
print_r($usuario);

// ========================================
// PLACEHOLDERS POSICIONAIS (?)
// ========================================

$sql = "SELECT * FROM usuarios WHERE email = ? AND ativo = ?";
$stmt = $pdo->prepare($sql);

$stmt->execute(['joao@example.com', 1]);

$usuario = $stmt->fetch();

// ========================================
// BIND DE VALORES INDIVIDUAL
// ========================================

$sql = "INSERT INTO usuarios (nome, email, idade) VALUES (:nome, :email, :idade)";
$stmt = $pdo->prepare($sql);

// Bind um por um
$stmt->bindValue(':nome', 'João Silva');
$stmt->bindValue(':email', 'joao@example.com');
$stmt->bindValue(':idade', 25, PDO::PARAM_INT);

$stmt->execute();

// ========================================
// BIND DE VARIÁVEIS POR REFERÊNCIA
// ========================================

$sql = "INSERT INTO usuarios (nome, email) VALUES (:nome, :email)";
$stmt = $pdo->prepare($sql);

// Bind por referência
$nome = '';
$email = '';

$stmt->bindParam(':nome', $nome);
$stmt->bindParam(':email', $email);

// Executar múltiplas vezes
$usuarios = [
    ['João', 'joao@example.com'],
    ['Maria', 'maria@example.com'],
    ['Pedro', 'pedro@example.com']
];

foreach ($usuarios as [$nome, $email]) {
    $stmt->execute();
}

// ========================================
// TIPOS DE BIND
// ========================================

$sql = "SELECT * FROM produtos WHERE id = :id AND ativo = :ativo";
$stmt = $pdo->prepare($sql);

$stmt->bindValue(':id', 1, PDO::PARAM_INT);           // Integer
$stmt->bindValue(':ativo', true, PDO::PARAM_BOOL);    // Boolean
$stmt->bindValue(':nome', 'Produto', PDO::PARAM_STR); // String (padrão)

$stmt->execute();

// Tipos disponíveis:
// - PDO::PARAM_NULL
// - PDO::PARAM_BOOL
// - PDO::PARAM_INT
// - PDO::PARAM_STR (padrão)
// - PDO::PARAM_LOB (large object)

// ========================================
// BUSCAR RESULTADOS
// ========================================

$sql = "SELECT * FROM usuarios WHERE ativo = :ativo";
$stmt = $pdo->prepare($sql);
$stmt->execute(['ativo' => 1]);

// Fetch: um registro
$usuario = $stmt->fetch();

// FetchAll: todos os registros
$usuarios = $stmt->fetchAll();

// FetchColumn: primeira coluna
$sql = "SELECT COUNT(*) FROM usuarios";
$stmt = $pdo->prepare($sql);
$stmt->execute();
$total = $stmt->fetchColumn();

// FetchObject: como objeto
$usuario = $stmt->fetchObject();
echo $usuario->nome;

// ========================================
// EXEMPLO COMPLETO: CLASSE REPOSITORY
// ========================================

class UsuarioRepository
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function findById(int $id): ?array
    {
        $sql = "SELECT * FROM usuarios WHERE id = :id";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['id' => $id]);
        
        $resultado = $stmt->fetch();
        
        return $resultado ?: null;
    }
    
    public function findByEmail(string $email): ?array
    {
        $sql = "SELECT * FROM usuarios WHERE email = :email";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['email' => $email]);
        
        $resultado = $stmt->fetch();
        
        return $resultado ?: null;
    }
    
    public function findAll(): array
    {
        $sql = "SELECT * FROM usuarios ORDER BY nome ASC";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute();
        
        return $stmt->fetchAll();
    }
    
    public function create(array $dados): int
    {
        $sql = "INSERT INTO usuarios (nome, email, senha_hash, criado_em) 
                VALUES (:nome, :email, :senha_hash, NOW())";
        
        $stmt = $this->pdo->prepare($sql);
        
        $stmt->execute([
            'nome' => $dados['nome'],
            'email' => $dados['email'],
            'senha_hash' => password_hash($dados['senha'], PASSWORD_ARGON2ID)
        ]);
        
        return (int) $this->pdo->lastInsertId();
    }
    
    public function update(int $id, array $dados): bool
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
    
    public function delete(int $id): bool
    {
        $sql = "DELETE FROM usuarios WHERE id = :id";
        $stmt = $this->pdo->prepare($sql);
        
        return $stmt->execute(['id' => $id]);
    }
}

// Uso
$pdo = Database::getConnection();
$repository = new UsuarioRepository($pdo);

$id = $repository->create([
    'nome' => 'João Silva',
    'email' => 'joao@example.com',
    'senha' => 'senha123'
]);

$usuario = $repository->findById($id);
print_r($usuario);
```

---

### 35.7 PDO::query - Queries Diretas

Executa query diretamente e retorna PDOStatement. **Use apenas para queries seguras sem dados do usuário!**

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// QUERY SIMPLES (sem dados do usuário)
// ========================================

// SELECT
$stmt = $pdo->query("SELECT * FROM usuarios");
$usuarios = $stmt->fetchAll();

// Iterar sobre resultados
foreach ($pdo->query("SELECT * FROM produtos") as $produto) {
    echo $produto['nome'] . "\n";
}

// ========================================
// QUERY COM FETCH MODE
// ========================================

// Fetch como objeto
$stmt = $pdo->query("SELECT * FROM usuarios", PDO::FETCH_OBJ);
foreach ($stmt as $usuario) {
    echo $usuario->nome . "\n";
}

// Fetch como classe específica
class Usuario
{
    public int $id;
    public string $nome;
    public string $email;
}

$stmt = $pdo->query("SELECT * FROM usuarios", PDO::FETCH_CLASS, Usuario::class);
$usuarios = $stmt->fetchAll();

// ========================================
// QUANDO USAR query() vs prepare()
// ========================================

// ✅ USE query() para:
// - Queries estáticas sem variáveis
// - Operações administrativas
$pdo->query("TRUNCATE TABLE logs");
$pdo->query("OPTIMIZE TABLE usuarios");

// ❌ NUNCA use query() com dados do usuário:
$email = $_POST['email'];
// VULNERÁVEL!
// $pdo->query("SELECT * FROM usuarios WHERE email = '$email'");

// ✅ Use prepare() com dados do usuário:
$stmt = $pdo->prepare("SELECT * FROM usuarios WHERE email = :email");
$stmt->execute(['email' => $email]);
```

---

### 35.8 PDO::exec - Execução Direta

Executa statement e retorna número de linhas afetadas.

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// INSERT
// ========================================

$sql = "INSERT INTO logs (mensagem, criado_em) VALUES ('Sistema iniciado', NOW())";
$linhasAfetadas = $pdo->exec($sql);

echo "Linhas inseridas: $linhasAfetadas\n";  // 1

// ========================================
// UPDATE
// ========================================

$sql = "UPDATE usuarios SET ativo = 1 WHERE ativo = 0";
$linhasAfetadas = $pdo->exec($sql);

echo "Usuários ativados: $linhasAfetadas\n";

// ========================================
// DELETE
// ========================================

$sql = "DELETE FROM logs WHERE criado_em < DATE_SUB(NOW(), INTERVAL 30 DAY)";
$linhasAfetadas = $pdo->exec($sql);

echo "Logs antigos deletados: $linhasAfetadas\n";

// ========================================
// OPERAÇÕES DDL (Data Definition Language)
// ========================================

// Criar tabela
$sql = "CREATE TABLE IF NOT EXISTS temp_data (
    id INT PRIMARY KEY AUTO_INCREMENT,
    valor VARCHAR(255),
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)";

$pdo->exec($sql);

// Deletar tabela
$pdo->exec("DROP TABLE IF EXISTS temp_data");

// Alterar tabela
$pdo->exec("ALTER TABLE usuarios ADD COLUMN telefone VARCHAR(20)");

// ========================================
// IMPORTANTE: exec() não retorna resultados
// ========================================

// ❌ Não funciona para SELECT
// $resultado = $pdo->exec("SELECT * FROM usuarios");

// ✅ Use query() ou prepare() para SELECT
$stmt = $pdo->query("SELECT * FROM usuarios");
```

---

### 35.9 PDO::quote - Escapar Strings

Adiciona aspas e escapa caracteres especiais. **NÃO RECOMENDADO - use prepare() ao invés!**

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// quote() - Escapar string manualmente
// ========================================

$nome = "O'Reilly";
$nomeEscapado = $pdo->quote($nome);

echo $nomeEscapado;  // 'O\'Reilly' (com aspas e escape)

// Pode usar em query direta (mas não recomendado)
$sql = "SELECT * FROM usuarios WHERE nome = $nomeEscapado";
$stmt = $pdo->query($sql);

// ========================================
// IMPORTANTE: SEMPRE PREFIRA prepare()
// ========================================

// ❌ Não recomendado:
$email = $pdo->quote($_POST['email']);
$sql = "SELECT * FROM usuarios WHERE email = $email";
$pdo->query($sql);

// ✅ Recomendado:
$stmt = $pdo->prepare("SELECT * FROM usuarios WHERE email = :email");
$stmt->execute(['email' => $_POST['email']]);

// ========================================
// Quando quote() pode ser útil
// ========================================

// Construção dinâmica de queries complexas (último recurso)
function construirFiltro(PDO $pdo, array $valores): string
{
    $escapados = array_map(
        fn($v) => $pdo->quote($v),
        $valores
    );
    
    return implode(', ', $escapados);
}

$ids = [1, 2, 3, 4, 5];
$lista = implode(', ', $ids);
$sql = "SELECT * FROM usuarios WHERE id IN ($lista)";
```

---

### 35.10 PDO::lastInsertId

Retorna o ID da última linha inserida.

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// OBTER ID APÓS INSERT
// ========================================

$sql = "INSERT INTO usuarios (nome, email) VALUES (:nome, :email)";
$stmt = $pdo->prepare($sql);

$stmt->execute([
    'nome' => 'João Silva',
    'email' => 'joao@example.com'
]);

$ultimoId = $pdo->lastInsertId();
echo "Usuário criado com ID: $ultimoId\n";

// ========================================
// COM SEQUÊNCIA ESPECÍFICA (PostgreSQL)
// ========================================

// PostgreSQL usa sequências
$ultimoId = $pdo->lastInsertId('usuarios_id_seq');

// ========================================
// RETORNO TIPADO
// ========================================

$ultimoId = (int) $pdo->lastInsertId();

// ========================================
// CLASSE HELPER
// ========================================

class InsertHelper
{
    public static function insertAndGetId(
        PDO $pdo,
        string $tabela,
        array $dados
    ): int {
        $campos = array_keys($dados);
        $placeholders = array_map(fn($c) => ":$c", $campos);
        
        $sql = sprintf(
            "INSERT INTO %s (%s) VALUES (%s)",
            $tabela,
            implode(', ', $campos),
            implode(', ', $placeholders)
        );
        
        $stmt = $pdo->prepare($sql);
        $stmt->execute($dados);
        
        return (int) $pdo->lastInsertId();
    }
}

// Uso
$id = InsertHelper::insertAndGetId($pdo, 'usuarios', [
    'nome' => 'Maria',
    'email' => 'maria@example.com'
]);

// ========================================
// IMPORTANTE: lastInsertId() retorna string!
// ========================================

$id = $pdo->lastInsertId();
var_dump($id);  // string "123"

// Sempre converta para int
$id = (int) $pdo->lastInsertId();
var_dump($id);  // int 123
```

---

### 35.11 PDO::errorCode e errorInfo

Obter informações sobre erros.

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// errorCode() - Código SQLSTATE
// ========================================

$sql = "INSERT INTO usuarios (nome, email) VALUES ('João', 'joao@example.com')";
$pdo->exec($sql);

$codigo = $pdo->errorCode();

if ($codigo !== '00000') {
    echo "Erro: $codigo\n";
}

// ========================================
// errorInfo() - Informação detalhada
// ========================================

$info = $pdo->errorInfo();

print_r($info);
/*
Array (
    [0] => SQLSTATE (5 caracteres)
    [1] => Código de erro específico do driver
    [2] => Mensagem de erro
)
*/

// Exemplo com erro
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT);

$sql = "INSERT INTO tabela_inexistente (campo) VALUES ('valor')";
$pdo->exec($sql);

$info = $pdo->errorInfo();
echo "SQLSTATE: {$info[0]}\n";
echo "Código: {$info[1]}\n";
echo "Mensagem: {$info[2]}\n";

// ========================================
// MODO DE ERRO: EXCEPTION (recomendado)
// ========================================

$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

try {
    $sql = "SELECT * FROM tabela_inexistente";
    $pdo->query($sql);
} catch (PDOException $e) {
    echo "Erro: " . $e->getMessage() . "\n";
    echo "Código: " . $e->getCode() . "\n";
    
    // Informação adicional
    $info = $pdo->errorInfo();
    print_r($info);
}

// ========================================
// CLASSE DE LOG DE ERROS
// ========================================

class PDOErrorLogger
{
    public static function log(PDO $pdo, string $contexto = ''): void
    {
        $info = $pdo->errorInfo();
        
        if ($info[0] !== '00000') {
            $mensagem = sprintf(
                "[%s] SQLSTATE: %s | Código: %s | Mensagem: %s | Contexto: %s",
                date('Y-m-d H:i:s'),
                $info[0],
                $info[1] ?? 'N/A',
                $info[2] ?? 'N/A',
                $contexto
            );
            
            error_log($mensagem);
        }
    }
}

// Uso
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_SILENT);
$pdo->exec("INSERT ERRO PROPOSITAL");

PDOErrorLogger::log($pdo, 'Teste de insert');
```

---

### 35.12 Transações

Agrupar múltiplas operações em uma unidade atômica.

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// ========================================
// TRANSAÇÃO BÁSICA
// ========================================

try {
    // Iniciar transação
    $pdo->beginTransaction();
    
    // Operação 1
    $pdo->exec("INSERT INTO usuarios (nome, email) VALUES ('João', 'joao@example.com')");
    
    // Operação 2
    $pdo->exec("INSERT INTO logs (mensagem) VALUES ('Usuário criado')");
    
    // Confirmar transação
    $pdo->commit();
    
    echo "Transação concluída com sucesso!\n";
    
} catch (Exception $e) {
    // Reverter em caso de erro
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }
    
    echo "Erro: " . $e->getMessage() . "\n";
}

// ========================================
// TRANSFERÊNCIA BANCÁRIA
// ========================================

class TransferenciaService
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function transferir(int $deContaId, int $paraContaId, float $valor): bool
    {
        try {
            $this->pdo->beginTransaction();
            
            // Debitar da conta origem
            $sql = "UPDATE contas SET saldo = saldo - :valor WHERE id = :id";
            $stmt = $this->pdo->prepare($sql);
            $stmt->execute(['valor' => $valor, 'id' => $deContaId]);
            
            // Verificar se saldo ficou negativo
            $sql = "SELECT saldo FROM contas WHERE id = :id";
            $stmt = $this->pdo->prepare($sql);
            $stmt->execute(['id' => $deContaId]);
            $saldo = (float) $stmt->fetchColumn();
            
            if ($saldo < 0) {
                throw new Exception("Saldo insuficiente");
            }
            
            // Creditar na conta destino
            $sql = "UPDATE contas SET saldo = saldo + :valor WHERE id = :id";
            $stmt = $this->pdo->prepare($sql);
            $stmt->execute(['valor' => $valor, 'id' => $paraContaId]);
            
            // Registrar transação
            $sql = "INSERT INTO transacoes (de_conta_id, para_conta_id, valor) 
                    VALUES (:de, :para, :valor)";
            $stmt = $this->pdo->prepare($sql);
            $stmt->execute([
                'de' => $deContaId,
                'para' => $paraContaId,
                'valor' => $valor
            ]);
            
            $this->pdo->commit();
            
            return true;
            
        } catch (Exception $e) {
            if ($this->pdo->inTransaction()) {
                $this->pdo->rollBack();
            }
            
            error_log("Erro na transferência: " . $e->getMessage());
            
            return false;
        }
    }
}

// ========================================
// CRIAR PEDIDO COM ITENS
// ========================================

class PedidoService
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function criar(int $usuarioId, array $itens): int
    {
        try {
            $this->pdo->beginTransaction();
            
            // Calcular total
            $total = array_sum(array_column($itens, 'preco'));
            
            // Criar pedido
            $sql = "INSERT INTO pedidos (usuario_id, total, status) 
                    VALUES (:usuario_id, :total, 'pendente')";
            $stmt = $this->pdo->prepare($sql);
            $stmt->execute(['usuario_id' => $usuarioId, 'total' => $total]);
            
            $pedidoId = (int) $this->pdo->lastInsertId();
            
            // Inserir itens
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
                
                // Atualizar estoque
                $this->atualizarEstoque($item['produto_id'], $item['quantidade']);
            }
            
            $this->pdo->commit();
            
            return $pedidoId;
            
        } catch (Exception $e) {
            if ($this->pdo->inTransaction()) {
                $this->pdo->rollBack();
            }
            
            throw new RuntimeException("Erro ao criar pedido: " . $e->getMessage(), 0, $e);
        }
    }
    
    private function atualizarEstoque(int $produtoId, int $quantidade): void
    {
        $sql = "UPDATE produtos SET estoque = estoque - :quantidade WHERE id = :id";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['quantidade' => $quantidade, 'id' => $produtoId]);
        
        // Verificar se estoque ficou negativo
        $sql = "SELECT estoque FROM produtos WHERE id = :id";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['id' => $produtoId]);
        $estoque = (int) $stmt->fetchColumn();
        
        if ($estoque < 0) {
            throw new Exception("Estoque insuficiente para produto $produtoId");
        }
    }
}

// ========================================
// SAVEPOINTS (pontos de salvamento)
// ========================================

try {
    $pdo->beginTransaction();
    
    // Operação 1
    $pdo->exec("INSERT INTO usuarios (nome) VALUES ('João')");
    
    // Criar savepoint
    $pdo->exec("SAVEPOINT sp1");
    
    // Operação 2
    $pdo->exec("INSERT INTO logs (mensagem) VALUES ('Log 1')");
    
    // Voltar para savepoint (cancela apenas operação 2)
    $pdo->exec("ROLLBACK TO SAVEPOINT sp1");
    
    // Operação 3
    $pdo->exec("INSERT INTO logs (mensagem) VALUES ('Log 2')");
    
    $pdo->commit();
    
} catch (Exception $e) {
    $pdo->rollBack();
}
```

---

### 35.13 PDO::inTransaction

Verifica se está dentro de uma transação.

```php
<?php

declare(strict_types=1);

$pdo = Database::getConnection();

// Verificar se está em transação
var_dump($pdo->inTransaction());  // false

$pdo->beginTransaction();
var_dump($pdo->inTransaction());  // true

$pdo->commit();
var_dump($pdo->inTransaction());  // false

// ========================================
// USO PRÁTICO: GARANTIR ROLLBACK
// ========================================

function executarComTransacao(PDO $pdo, callable $operacao): mixed
{
    try {
        $pdo->beginTransaction();
        
        $resultado = $operacao();
        
        $pdo->commit();
        
        return $resultado;
        
    } catch (Exception $e) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }
        
        throw $e;
    }
}

// Uso
$resultado = executarComTransacao($pdo, function() use ($pdo) {
    $pdo->exec("INSERT INTO usuarios (nome) VALUES ('João')");
    $pdo->exec("INSERT INTO logs (mensagem) VALUES ('Criado')");
    
    return $pdo->lastInsertId();
});

// ========================================
// CLASSE TRANSACTION MANAGER
// ========================================

class TransactionManager
{
    private int $transactionLevel = 0;
    
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function begin(): void
    {
        if ($this->transactionLevel === 0) {
            $this->pdo->beginTransaction();
        } else {
            $this->pdo->exec("SAVEPOINT level_{$this->transactionLevel}");
        }
        
        $this->transactionLevel++;
    }
    
    public function commit(): void
    {
        $this->transactionLevel--;
        
        if ($this->transactionLevel === 0) {
            $this->pdo->commit();
        } else {
            $this->pdo->exec("RELEASE SAVEPOINT level_{$this->transactionLevel}");
        }
    }
    
    public function rollback(): void
    {
        $this->transactionLevel--;
        
        if ($this->transactionLevel === 0) {
            if ($this->pdo->inTransaction()) {
                $this->pdo->rollBack();
            }
        } else {
            $this->pdo->exec("ROLLBACK TO SAVEPOINT level_{$this->transactionLevel}");
        }
    }
    
    public function inTransaction(): bool
    {
        return $this->transactionLevel > 0;
    }
}

// Uso
$tm = new TransactionManager($pdo);

$tm->begin();
// operação 1
$tm->begin();  // Nested transaction (savepoint)
// operação 2
$tm->commit();
$tm->commit();
```

---

### 35.14 Projeto Prático Completo

Sistema completo de gerenciamento de usuários com PDO.

```php
<?php

declare(strict_types=1);

// ========================================
// 1. CLASSE DE CONEXÃO
// ========================================

class Database
{
    private static ?PDO $instance = null;
    
    public static function getConnection(): PDO
    {
        if (self::$instance === null) {
            $config = [
                'host' => 'localhost',
                'dbname' => 'sistema',
                'username' => 'root',
                'password' => '',
                'charset' => 'utf8mb4'
            ];
            
            $dsn = sprintf(
                'mysql:host=%s;dbname=%s;charset=%s',
                $config['host'],
                $config['dbname'],
                $config['charset']
            );
            
            $options = [
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
                PDO::ATTR_EMULATE_PREPARES => false,
            ];
            
            self::$instance = new PDO($dsn, $config['username'], $config['password'], $options);
        }
        
        return self::$instance;
    }
}

// ========================================
// 2. ENTITY
// ========================================

class Usuario
{
    public function __construct(
        public ?int $id = null,
        public string $nome = '',
        public string $email = '',
        public bool $ativo = true,
        public ?string $criadoEm = null,
        public ?string $atualizadoEm = null
    ) {}
}

// ========================================
// 3. REPOSITORY
// ========================================

class UsuarioRepository
{
    public function __construct(
        private PDO $pdo
    ) {}
    
    public function find(int $id): ?Usuario
    {
        $sql = "SELECT * FROM usuarios WHERE id = :id";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['id' => $id]);
        
        $dados = $stmt->fetch();
        
        if (!$dados) {
            return null;
        }
        
        return $this->hydrate($dados);
    }
    
    public function findByEmail(string $email): ?Usuario
    {
        $sql = "SELECT * FROM usuarios WHERE email = :email";
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute(['email' => $email]);
        
        $dados = $stmt->fetch();
        
        return $dados ? $this->hydrate($dados) : null;
    }
    
    public function findAll(int $limit = 100, int $offset = 0): array
    {
        $sql = "SELECT * FROM usuarios ORDER BY nome ASC LIMIT :limit OFFSET :offset";
        $stmt = $this->pdo->prepare($sql);
        $stmt->bindValue(':limit', $limit, PDO::PARAM_INT);
        $stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
        $stmt->execute();
        
        $usuarios = [];
        foreach ($stmt->fetchAll() as $dados) {
            $usuarios[] = $this->hydrate($dados);
        }
        
        return $usuarios;
    }
    
    public function save(Usuario $usuario): int
    {
        if ($usuario->id === null) {
            return $this->insert($usuario);
        }
        
        $this->update($usuario);
        return $usuario->id;
    }
    
    private function insert(Usuario $usuario): int
    {
        $sql = "INSERT INTO usuarios (nome, email, ativo, criado_em) 
                VALUES (:nome, :email, :ativo, NOW())";
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute([
            'nome' => $usuario->nome,
            'email' => $usuario->email,
            'ativo' => $usuario->ativo ? 1 : 0
        ]);
        
        $usuario->id = (int) $this->pdo->lastInsertId();
        
        return $usuario->id;
    }
    
    private function update(Usuario $usuario): void
    {
        $sql = "UPDATE usuarios 
                SET nome = :nome, email = :email, ativo = :ativo, atualizado_em = NOW() 
                WHERE id = :id";
        
        $stmt = $this->pdo->prepare($sql);
        $stmt->execute([
            'id' => $usuario->id,
            'nome' => $usuario->nome,
            'email' => $usuario->email,
            'ativo' => $usuario->ativo ? 1 : 0
        ]);
    }
    
    public function delete(int $id): bool
    {
        $sql = "DELETE FROM usuarios WHERE id = :id";
        $stmt = $this->pdo->prepare($sql);
        
        return $stmt->execute(['id' => $id]);
    }
    
    public function count(): int
    {
        $sql = "SELECT COUNT(*) FROM usuarios";
        return (int) $this->pdo->query($sql)->fetchColumn();
    }
    
    private function hydrate(array $dados): Usuario
    {
        return new Usuario(
            id: (int) $dados['id'],
            nome: $dados['nome'],
            email: $dados['email'],
            ativo: (bool) $dados['ativo'],
            criadoEm: $dados['criado_em'],
            atualizadoEm: $dados['atualizado_em']
        );
    }
}

// ========================================
// 4. SERVICE
// ========================================

class UsuarioService
{
    public function __construct(
        private UsuarioRepository $repository
    ) {}
    
    public function criar(string $nome, string $email, string $senha): Usuario
    {
        // Validar email
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException("Email inválido");
        }
        
        // Verificar se email já existe
        if ($this->repository->findByEmail($email) !== null) {
            throw new RuntimeException("Email já cadastrado");
        }
        
        // Criar usuário
        $usuario = new Usuario(
            nome: $nome,
            email: $email
        );
        
        $this->repository->save($usuario);
        
        return $usuario;
    }
    
    public function atualizar(int $id, array $dados): Usuario
    {
        $usuario = $this->repository->find($id);
        
        if ($usuario === null) {
            throw new RuntimeException("Usuário não encontrado");
        }
        
        if (isset($dados['nome'])) {
            $usuario->nome = $dados['nome'];
        }
        
        if (isset($dados['email'])) {
            $usuario->email = $dados['email'];
        }
        
        if (isset($dados['ativo'])) {
            $usuario->ativo = (bool) $dados['ativo'];
        }
        
        $this->repository->save($usuario);
        
        return $usuario;
    }
    
    public function deletar(int $id): bool
    {
        return $this->repository->delete($id);
    }
    
    public function listar(int $pagina = 1, int $porPagina = 10): array
    {
        $offset = ($pagina - 1) * $porPagina;
        
        return [
            'usuarios' => $this->repository->findAll($porPagina, $offset),
            'total' => $this->repository->count(),
            'pagina' => $pagina,
            'porPagina' => $porPagina
        ];
    }
}

// ========================================
// 5. USO
// ========================================

// Criar conexão
$pdo = Database::getConnection();

// Criar repository e service
$repository = new UsuarioRepository($pdo);
$service = new UsuarioService($repository);

// Criar usuário
try {
    $usuario = $service->criar(
        nome: 'João Silva',
        email: 'joao@example.com',
        senha: 'senha123'
    );
    
    echo "Usuário criado com ID: {$usuario->id}\n";
} catch (Exception $e) {
    echo "Erro: " . $e->getMessage() . "\n";
}

// Listar usuários
$resultado = $service->listar(pagina: 1, porPagina: 10);

echo "Total de usuários: {$resultado['total']}\n";
foreach ($resultado['usuarios'] as $usuario) {
    echo "- {$usuario->nome} ({$usuario->email})\n";
}

// Atualizar usuário
$usuario = $service->atualizar(1, [
    'nome' => 'João Silva Santos',
    'ativo' => true
]);

// Deletar usuário
$service->deletar(1);
```

---

## 📝 Exercícios do Capítulo 35

### Nível Iniciante

1. Crie uma conexão PDO com tratamento de erros
2. Faça CRUD completo de produtos (create, read, update, delete)
3. Implemente paginação em listagem de registros
4. Use prepared statements para busca com filtros

### Nível Intermediário

5. Crie sistema de log de auditoria com transações
6. Implemente soft delete (exclusão lógica)
7. Faça relacionamento entre tabelas (usuário tem pedidos)
8. Crie repository genérico reutilizável

### Nível Avançado

9. Implemente sistema de transferência bancária com transações
10. Crie query builder orientado a objetos
11. Faça cache de consultas com invalidação inteligente
12. Implemente connection pooling

---

## 🎯 Resumo do Capítulo

### ✅ O que você aprendeu:

- ✅ **Conexão:** PDO::__construct e PDO::connect
- ✅ **Configuração:** getAttribute, setAttribute
- ✅ **Queries:** prepare (SEMPRE use!), query, exec
- ✅ **Resultados:** fetch, fetchAll, fetchColumn, fetchObject
- ✅ **Segurança:** Prepared statements, quote
- ✅ **IDs:** lastInsertId
- ✅ **Erros:** errorCode, errorInfo, exceções
- ✅ **Transações:** beginTransaction, commit, rollBack, inTransaction
- ✅ **Padrões:** Repository, Service Layer
- ✅ **Boas práticas:** SEMPRE use prepare(), NUNCA concatene SQL

### 🔒 Regras de Ouro do PDO:

1. **SEMPRE** use `PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION`
2. **SEMPRE** use prepared statements para dados do usuário
3. **NUNCA** use `PDO::query()` ou `PDO::exec()` com variáveis externas
4. **SEMPRE** use transações para operações múltiplas
5. **SEMPRE** trate exceções adequadamente
6. **SEMPRE** use type hints e conversões de tipo
7. **NUNCA** confie em dados não validados

### 🚀 Próximos Passos:

- Estude **Query Builders** (Doctrine DBAL)
- Aprenda **ORMs** (Doctrine ORM, Eloquent)
- Pratique **padrões de repositório**
- Explore **migrations** de banco de dados
- Implemente **connection pooling**

---

**Parabéns! Você dominou PDO! 🎉**

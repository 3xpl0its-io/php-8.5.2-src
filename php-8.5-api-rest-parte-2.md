# PHP 8.5 Moderno: Criando sua Própria API REST - Parte 2

**Continuação do Capítulo 36**

---

### 36.8 Autenticação JWT

```php
<?php
// src/Helpers/JWT.php

declare(strict_types=1);

namespace App\Helpers;

class JWT
{
    private const ALGORITHM = 'HS256';
    
    public function __construct(
        private string $secretKey
    ) {}
    
    /**
     * Criar token JWT
     */
    public function encode(array $payload, int $expiresIn = 3600): string
    {
        // Header
        $header = [
            'typ' => 'JWT',
            'alg' => self::ALGORITHM
        ];
        
        // Payload com expiração
        $payload['iat'] = time();
        $payload['exp'] = time() + $expiresIn;
        
        // Encode header e payload
        $headerEncoded = $this->base64UrlEncode(json_encode($header));
        $payloadEncoded = $this->base64UrlEncode(json_encode($payload));
        
        // Criar assinatura
        $signature = hash_hmac(
            'sha256',
            "$headerEncoded.$payloadEncoded",
            $this->secretKey,
            true
        );
        
        $signatureEncoded = $this->base64UrlEncode($signature);
        
        // Token final
        return "$headerEncoded.$payloadEncoded.$signatureEncoded";
    }
    
    /**
     * Decodificar e validar token
     */
    public function decode(string $token): ?array
    {
        // Separar partes
        $parts = explode('.', $token);
        
        if (count($parts) !== 3) {
            return null;
        }
        
        [$headerEncoded, $payloadEncoded, $signatureEncoded] = $parts;
        
        // Verificar assinatura
        $signature = hash_hmac(
            'sha256',
            "$headerEncoded.$payloadEncoded",
            $this->secretKey,
            true
        );
        
        $signatureValid = $this->base64UrlEncode($signature);
        
        if (!hash_equals($signatureValid, $signatureEncoded)) {
            return null;  // Assinatura inválida
        }
        
        // Decodificar payload
        $payload = json_decode($this->base64UrlDecode($payloadEncoded), true);
        
        // Verificar expiração
        if (isset($payload['exp']) && $payload['exp'] < time()) {
            return null;  // Token expirado
        }
        
        return $payload;
    }
    
    private function base64UrlEncode(string $data): string
    {
        return rtrim(strtr(base64_encode($data), '+/', '-_'), '=');
    }
    
    private function base64UrlDecode(string $data): string
    {
        return base64_decode(strtr($data, '-_', '+/'));
    }
}
```

### Controller de Autenticação

```php
<?php
// src/Controllers/AuthController.php

declare(strict_types=1);

namespace App\Controllers;

use App\Http\Request;
use App\Http\Response;
use App\Http\JsonResponse;
use App\Repositories\UserRepository;
use App\Helpers\JWT;

class AuthController
{
    public function __construct(
        private UserRepository $repository = new UserRepository(),
        private JWT $jwt = new JWT($_ENV['JWT_SECRET'] ?? 'sua-chave-secreta-aqui')
    ) {}
    
    /**
     * POST /api/v1/auth/register
     * Registrar novo usuário
     */
    public function register(Request $request): Response
    {
        // Validar dados
        $nome = $request->input('nome');
        $email = $request->input('email');
        $senha = $request->input('senha');
        
        if (empty($nome) || empty($email) || empty($senha)) {
            return JsonResponse::validationError([
                'nome' => 'Campo obrigatório',
                'email' => 'Campo obrigatório',
                'senha' => 'Campo obrigatório'
            ]);
        }
        
        // Verificar se email já existe
        if ($this->repository->findByEmail($email)) {
            return JsonResponse::conflict('Email já cadastrado');
        }
        
        // Criar usuário
        $id = $this->repository->create([
            'nome' => $nome,
            'email' => $email,
            'senha_hash' => password_hash($senha, PASSWORD_ARGON2ID)
        ]);
        
        $usuario = $this->repository->find($id);
        
        // Gerar token
        $token = $this->jwt->encode([
            'user_id' => $usuario['id'],
            'email' => $usuario['email']
        ]);
        
        return JsonResponse::success([
            'user' => [
                'id' => $usuario['id'],
                'nome' => $usuario['nome'],
                'email' => $usuario['email']
            ],
            'token' => $token
        ], 'Usuário registrado com sucesso', 201);
    }
    
    /**
     * POST /api/v1/auth/login
     * Fazer login
     */
    public function login(Request $request): Response
    {
        $email = $request->input('email');
        $senha = $request->input('senha');
        
        if (empty($email) || empty($senha)) {
            return JsonResponse::validationError([
                'email' => 'Campo obrigatório',
                'senha' => 'Campo obrigatório'
            ]);
        }
        
        // Buscar usuário
        $usuario = $this->repository->findByEmail($email);
        
        if (!$usuario) {
            return JsonResponse::unauthorized('Credenciais inválidas');
        }
        
        // Verificar senha
        if (!password_verify($senha, $usuario['senha_hash'])) {
            return JsonResponse::unauthorized('Credenciais inválidas');
        }
        
        // Gerar token
        $token = $this->jwt->encode([
            'user_id' => $usuario['id'],
            'email' => $usuario['email']
        ], 86400);  // 24 horas
        
        return JsonResponse::success([
            'user' => [
                'id' => $usuario['id'],
                'nome' => $usuario['nome'],
                'email' => $usuario['email']
            ],
            'token' => $token
        ], 'Login realizado com sucesso');
    }
    
    /**
     * GET /api/v1/auth/me
     * Obter dados do usuário autenticado
     */
    public function me(Request $request): Response
    {
        // Obter usuário do request (adicionado pelo middleware)
        $userId = $request->input('auth_user_id');
        
        $usuario = $this->repository->find($userId);
        
        return JsonResponse::success([
            'id' => $usuario['id'],
            'nome' => $usuario['nome'],
            'email' => $usuario['email'],
            'criado_em' => $usuario['criado_em']
        ]);
    }
}
```

---

### 36.9 Middleware

```php
<?php
// src/Middleware/AuthMiddleware.php

declare(strict_types=1);

namespace App\Middleware;

use App\Http\Request;
use App\Http\Response;
use App\Http\JsonResponse;
use App\Helpers\JWT;

class AuthMiddleware
{
    public function __construct(
        private JWT $jwt = new JWT($_ENV['JWT_SECRET'] ?? 'sua-chave-secreta')
    ) {}
    
    public function handle(Request $request): ?Response
    {
        // Obter token do header
        $token = $request->bearerToken();
        
        if (!$token) {
            return JsonResponse::unauthorized('Token não fornecido');
        }
        
        // Validar token
        $payload = $this->jwt->decode($token);
        
        if (!$payload) {
            return JsonResponse::unauthorized('Token inválido ou expirado');
        }
        
        // Adicionar user_id ao request
        $_REQUEST['auth_user_id'] = $payload['user_id'];
        
        // Continue para o próximo middleware/controller
        return null;
    }
}
```

### Middleware CORS

```php
<?php
// src/Middleware/CorsMiddleware.php

declare(strict_types=1);

namespace App\Middleware;

use App\Http\Request;
use App\Http\Response;

class CorsMiddleware
{
    public function handle(Request $request): ?Response
    {
        // Definir headers CORS
        header('Access-Control-Allow-Origin: *');
        header('Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS');
        header('Access-Control-Allow-Headers: Content-Type, Authorization');
        header('Access-Control-Max-Age: 86400');
        
        // Se for OPTIONS (preflight), retornar 200
        if ($request->getMethod() === 'OPTIONS') {
            http_response_code(200);
            exit;
        }
        
        return null;
    }
}
```

### Sistema de Middleware

```php
<?php
// src/Http/MiddlewareStack.php

declare(strict_types=1);

namespace App\Http;

class MiddlewareStack
{
    private array $middlewares = [];
    
    public function add(object $middleware): self
    {
        $this->middlewares[] = $middleware;
        return $this;
    }
    
    public function handle(Request $request, callable $next): Response
    {
        // Executar middlewares na ordem
        foreach ($this->middlewares as $middleware) {
            $response = $middleware->handle($request);
            
            // Se middleware retornar Response, interromper
            if ($response instanceof Response) {
                return $response;
            }
        }
        
        // Executar handler final (controller)
        return $next($request);
    }
}
```

---

### 36.10 Validação de Requisições

```php
<?php
// src/Services/ValidationService.php

declare(strict_types=1);

namespace App\Services;

use App\Repositories\UserRepository;

class ValidationService
{
    /**
     * Validar dados
     * 
     * @param array $data Dados a validar
     * @param array $rules Regras ['campo' => ['rule1', 'rule2']]
     * @return array Erros (vazio se válido)
     */
    public function validate(array $data, array $rules): array
    {
        $errors = [];
        
        foreach ($rules as $field => $fieldRules) {
            $value = $data[$field] ?? null;
            
            foreach ($fieldRules as $rule) {
                $error = $this->validateRule($field, $value, $rule, $data);
                
                if ($error) {
                    $errors[$field][] = $error;
                }
            }
        }
        
        return $errors;
    }
    
    private function validateRule(string $field, mixed $value, string $rule, array $allData): ?string
    {
        // Parse regra (ex: "min:3", "unique:usuarios,id,5")
        $parts = explode(':', $rule);
        $ruleName = $parts[0];
        $ruleParams = array_slice($parts, 1);
        
        return match($ruleName) {
            'required' => $this->validateRequired($value),
            'email' => $this->validateEmail($value),
            'min' => $this->validateMin($value, (int) $ruleParams[0]),
            'max' => $this->validateMax($value, (int) $ruleParams[0]),
            'numeric' => $this->validateNumeric($value),
            'url' => $this->validateUrl($value),
            'unique' => $this->validateUnique($value, $ruleParams),
            'confirmed' => $this->validateConfirmed($field, $value, $allData),
            default => null
        };
    }
    
    private function validateRequired(mixed $value): ?string
    {
        if (empty($value) && $value !== '0' && $value !== 0) {
            return 'Este campo é obrigatório';
        }
        
        return null;
    }
    
    private function validateEmail(mixed $value): ?string
    {
        if (empty($value)) {
            return null;  // Se vazio, deixar para 'required'
        }
        
        if (!filter_var($value, FILTER_VALIDATE_EMAIL)) {
            return 'Email inválido';
        }
        
        return null;
    }
    
    private function validateMin(mixed $value, int $min): ?string
    {
        if (empty($value)) {
            return null;
        }
        
        if (is_string($value) && strlen($value) < $min) {
            return "Deve ter no mínimo $min caracteres";
        }
        
        if (is_numeric($value) && $value < $min) {
            return "Deve ser no mínimo $min";
        }
        
        return null;
    }
    
    private function validateMax(mixed $value, int $max): ?string
    {
        if (empty($value)) {
            return null;
        }
        
        if (is_string($value) && strlen($value) > $max) {
            return "Deve ter no máximo $max caracteres";
        }
        
        if (is_numeric($value) && $value > $max) {
            return "Deve ser no máximo $max";
        }
        
        return null;
    }
    
    private function validateNumeric(mixed $value): ?string
    {
        if (empty($value)) {
            return null;
        }
        
        if (!is_numeric($value)) {
            return 'Deve ser um número';
        }
        
        return null;
    }
    
    private function validateUrl(mixed $value): ?string
    {
        if (empty($value)) {
            return null;
        }
        
        if (!filter_var($value, FILTER_VALIDATE_URL)) {
            return 'URL inválida';
        }
        
        return null;
    }
    
    private function validateUnique(mixed $value, array $params): ?string
    {
        if (empty($value)) {
            return null;
        }
        
        // params: ['usuarios', 'id', '5']
        $table = $params[0];
        $ignoreColumn = $params[1] ?? null;
        $ignoreValue = $params[2] ?? null;
        
        // Verificar se já existe (implementação simplificada)
        $repo = new UserRepository();
        
        if ($table === 'usuarios') {
            $existing = $repo->findByEmail($value);
            
            if ($existing) {
                // Se for update e for o mesmo registro, ignorar
                if ($ignoreColumn && $ignoreValue && $existing[$ignoreColumn] == $ignoreValue) {
                    return null;
                }
                
                return 'Este valor já está em uso';
            }
        }
        
        return null;
    }
    
    private function validateConfirmed(string $field, mixed $value, array $allData): ?string
    {
        $confirmationField = $field . '_confirmation';
        $confirmationValue = $allData[$confirmationField] ?? null;
        
        if ($value !== $confirmationValue) {
            return 'Os campos não coincidem';
        }
        
        return null;
    }
}
```

---

### 36.11 Rate Limiting

```php
<?php
// src/Middleware/RateLimitMiddleware.php

declare(strict_types=1);

namespace App\Middleware;

use App\Http\Request;
use App\Http\Response;
use App\Http\JsonResponse;

class RateLimitMiddleware
{
    private const CACHE_DIR = __DIR__ . '/../../storage/rate_limit';
    
    public function __construct(
        private int $maxRequests = 60,      // Máximo de requisições
        private int $windowSeconds = 60      // Janela de tempo (1 minuto)
    ) {
        if (!is_dir(self::CACHE_DIR)) {
            mkdir(self::CACHE_DIR, 0755, true);
        }
    }
    
    public function handle(Request $request): ?Response
    {
        $identifier = $this->getIdentifier($request);
        $cacheFile = self::CACHE_DIR . '/' . md5($identifier) . '.json';
        
        // Carregar histórico
        $history = $this->loadHistory($cacheFile);
        
        // Remover requisições antigas (fora da janela)
        $now = time();
        $history = array_filter($history, fn($timestamp) => ($now - $timestamp) < $this->windowSeconds);
        
        // Verificar limite
        if (count($history) >= $this->maxRequests) {
            $retryAfter = $this->windowSeconds - ($now - min($history));
            
            return JsonResponse::error(
                'RATE_LIMIT_EXCEEDED',
                'Muitas requisições. Tente novamente mais tarde.',
                ['retry_after' => $retryAfter],
                429
            );
        }
        
        // Adicionar requisição atual
        $history[] = $now;
        
        // Salvar histórico
        file_put_contents($cacheFile, json_encode($history));
        
        // Adicionar headers de rate limit
        header("X-RateLimit-Limit: {$this->maxRequests}");
        header("X-RateLimit-Remaining: " . ($this->maxRequests - count($history)));
        header("X-RateLimit-Reset: " . ($now + $this->windowSeconds));
        
        return null;
    }
    
    private function getIdentifier(Request $request): string
    {
        // Usar token se autenticado
        $token = $request->bearerToken();
        if ($token) {
            return "token:$token";
        }
        
        // Caso contrário, usar IP
        return "ip:" . ($_SERVER['REMOTE_ADDR'] ?? 'unknown');
    }
    
    private function loadHistory(string $file): array
    {
        if (!file_exists($file)) {
            return [];
        }
        
        $content = file_get_contents($file);
        return json_decode($content, true) ?? [];
    }
}
```

---

### 36.12 Versionamento

```php
<?php
// public/index.php

declare(strict_types=1);

require __DIR__ . '/../vendor/autoload.php';  // Se usar Composer
// ou
require __DIR__ . '/../src/autoload.php';     // Autoloader manual

use App\Http\Request;
use App\Http\Router;
use App\Middleware\CorsMiddleware;
use App\Middleware\RateLimitMiddleware;
use App\Middleware\AuthMiddleware;
use App\Controllers\V1\UserController as UserControllerV1;
use App\Controllers\V1\AuthController as AuthControllerV1;
use App\Controllers\V2\UserController as UserControllerV2;

$request = new Request();
$router = new Router();

// Middleware global
$cors = new CorsMiddleware();
$cors->handle($request);

$rateLimit = new RateLimitMiddleware(100, 60);
$rateLimitResponse = $rateLimit->handle($request);

if ($rateLimitResponse) {
    $rateLimitResponse->send();
}

// ========================================
// VERSÃO 1 da API
// ========================================

// Autenticação
$router->post('/api/v1/auth/register', [AuthControllerV1::class, 'register']);
$router->post('/api/v1/auth/login', [AuthControllerV1::class, 'login']);

// Rotas protegidas V1
$router->get('/api/v1/auth/me', function(Request $request) {
    $auth = new AuthMiddleware();
    $authResponse = $auth->handle($request);
    
    if ($authResponse) {
        return $authResponse;
    }
    
    $controller = new AuthControllerV1();
    return $controller->me($request);
});

// Usuários V1
$router->get('/api/v1/usuarios', [UserControllerV1::class, 'index']);
$router->get('/api/v1/usuarios/{id}', [UserControllerV1::class, 'show']);
$router->post('/api/v1/usuarios', [UserControllerV1::class, 'store']);
$router->put('/api/v1/usuarios/{id}', [UserControllerV1::class, 'update']);
$router->delete('/api/v1/usuarios/{id}', [UserControllerV1::class, 'destroy']);

// ========================================
// VERSÃO 2 da API (com melhorias)
// ========================================

$router->get('/api/v2/usuarios', [UserControllerV2::class, 'index']);
// ... outras rotas V2

// Dispatch
$response = $router->dispatch($request);
$response->send();
```

---

### 36.13 Documentação

```markdown
# API Documentation - v1

## Base URL
```
https://api.exemplo.com/api/v1
```

## Autenticação

Todas as rotas protegidas requerem token JWT no header:

```
Authorization: Bearer SEU_TOKEN_AQUI
```

---

## Endpoints

### Autenticação

#### POST /auth/register
Registrar novo usuário.

**Request:**
```json
{
  "nome": "João Silva",
  "email": "joao@example.com",
  "senha": "senha123"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "nome": "João Silva",
      "email": "joao@example.com"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  },
  "message": "Usuário registrado com sucesso"
}
```

---

#### POST /auth/login
Fazer login.

**Request:**
```json
{
  "email": "joao@example.com",
  "senha": "senha123"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "nome": "João Silva",
      "email": "joao@example.com"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

#### GET /auth/me
Obter dados do usuário autenticado.

**Headers:**
```
Authorization: Bearer TOKEN
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "nome": "João Silva",
    "email": "joao@example.com",
    "criado_em": "2026-02-09 10:30:00"
  }
}
```

---

### Usuários

#### GET /usuarios
Listar usuários (com paginação).

**Query Parameters:**
- `page` (int): Número da página (padrão: 1)
- `per_page` (int): Itens por página (padrão: 10)
- `search` (string): Buscar por nome ou email

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "nome": "João Silva",
      "email": "joao@example.com"
    }
  ],
  "meta": {
    "total": 100,
    "page": 1,
    "per_page": 10,
    "last_page": 10
  }
}
```

---

#### GET /usuarios/{id}
Buscar usuário específico.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "nome": "João Silva",
    "email": "joao@example.com",
    "criado_em": "2026-02-09 10:30:00"
  }
}
```

**Response 404:**
```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "Usuário não encontrado"
  }
}
```

---

### Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Requisição inválida |
| 401 | Não autenticado |
| 403 | Sem permissão |
| 404 | Recurso não encontrado |
| 422 | Validação falhou |
| 429 | Rate limit excedido |
| 500 | Erro interno do servidor |

---

### Rate Limiting

- **Limite:** 60 requisições por minuto
- **Headers:**
  - `X-RateLimit-Limit`: Limite total
  - `X-RateLimit-Remaining`: Requisições restantes
  - `X-RateLimit-Reset`: Timestamp do reset
```

---

### 36.14 API Completa - Projeto Final

```php
<?php
// public/index.php - ARQUIVO PRINCIPAL

declare(strict_types=1);

// Autoloader (se não usar Composer)
spl_autoload_register(function ($class) {
    $prefix = 'App\\';
    $baseDir = __DIR__ . '/../src/';
    
    $len = strlen($prefix);
    if (strncmp($prefix, $class, $len) !== 0) {
        return;
    }
    
    $relativeClass = substr($class, $len);
    $file = $baseDir . str_replace('\\', '/', $relativeClass) . '.php';
    
    if (file_exists($file)) {
        require $file;
    }
});

use App\Http\Request;
use App\Http\Response;
use App\Http\Router;
use App\Http\JsonResponse;
use App\Middleware\CorsMiddleware;
use App\Middleware\RateLimitMiddleware;
use App\Middleware\AuthMiddleware;
use App\Controllers\AuthController;
use App\Controllers\UserController;
use App\Controllers\ProductController;

// Tratamento de erros
set_error_handler(function($errno, $errstr, $errfile, $errline) {
    throw new ErrorException($errstr, 0, $errno, $errfile, $errline);
});

set_exception_handler(function($exception) {
    error_log($exception->getMessage());
    
    JsonResponse::error(
        'INTERNAL_ERROR',
        'Erro interno do servidor',
        null,
        500
    )->send();
});

// Criar request
$request = new Request();
$router = new Router();

// Middleware CORS
$cors = new CorsMiddleware();
$cors->handle($request);

// Middleware Rate Limit
$rateLimit = new RateLimitMiddleware(100, 60);
$rateLimitResponse = $rateLimit->handle($request);

if ($rateLimitResponse) {
    $rateLimitResponse->send();
}

// ========================================
// ROTAS PÚBLICAS
// ========================================

$router->post('/api/v1/auth/register', [AuthController::class, 'register']);
$router->post('/api/v1/auth/login', [AuthController::class, 'login']);

// ========================================
// ROTAS PROTEGIDAS (requerem autenticação)
// ========================================

$protectedRoutes = function(Router $router, Request $request) {
    $auth = new AuthMiddleware();
    
    // Verificar autenticação
    $authResponse = $auth->handle($request);
    if ($authResponse) {
        return $authResponse;
    }
    
    // Se autenticado, continuar
    return null;
};

// Auth
$router->get('/api/v1/auth/me', function(Request $request) use ($protectedRoutes) {
    $response = $protectedRoutes(new Router(), $request);
    if ($response) return $response;
    
    return (new AuthController())->me($request);
});

// Usuários
$router->get('/api/v1/usuarios', [UserController::class, 'index']);
$router->get('/api/v1/usuarios/{id}', [UserController::class, 'show']);

$router->post('/api/v1/usuarios', function(Request $request, array $params = []) use ($protectedRoutes) {
    $response = $protectedRoutes(new Router(), $request);
    if ($response) return $response;
    
    return (new UserController())->store($request);
});

$router->put('/api/v1/usuarios/{id}', function(Request $request, array $params) use ($protectedRoutes) {
    $response = $protectedRoutes(new Router(), $request);
    if ($response) return $response;
    
    return (new UserController())->update($request, $params);
});

$router->delete('/api/v1/usuarios/{id}', function(Request $request, array $params) use ($protectedRoutes) {
    $response = $protectedRoutes(new Router(), $request);
    if ($response) return $response;
    
    return (new UserController())->destroy($request, $params);
});

// Produtos (exemplo adicional)
$router->get('/api/v1/produtos', [ProductController::class, 'index']);
$router->get('/api/v1/produtos/{id}', [ProductController::class, 'show']);

// Rota de teste
$router->get('/api/v1/ping', function() {
    return JsonResponse::success(['message' => 'pong'], 'API está funcionando!');
});

// Dispatch
try {
    $response = $router->dispatch($request);
    $response->send();
} catch (Exception $e) {
    error_log($e->getMessage());
    
    JsonResponse::error(
        'INTERNAL_ERROR',
        'Erro ao processar requisição',
        null,
        500
    )->send();
}
```

### Exemplo de Uso da API (Cliente)

```php
<?php
// client_example.php - Como consumir a API

declare(strict_types=1);

class ApiClient
{
    private string $baseUrl;
    private ?string $token = null;
    
    public function __construct(string $baseUrl)
    {
        $this->baseUrl = rtrim($baseUrl, '/');
    }
    
    public function setToken(string $token): void
    {
        $this->token = $token;
    }
    
    public function register(string $nome, string $email, string $senha): array
    {
        $response = $this->post('/auth/register', [
            'nome' => $nome,
            'email' => $email,
            'senha' => $senha
        ]);
        
        if ($response['success']) {
            $this->token = $response['data']['token'];
        }
        
        return $response;
    }
    
    public function login(string $email, string $senha): array
    {
        $response = $this->post('/auth/login', [
            'email' => $email,
            'senha' => $senha
        ]);
        
        if ($response['success']) {
            $this->token = $response['data']['token'];
        }
        
        return $response;
    }
    
    public function getMe(): array
    {
        return $this->get('/auth/me');
    }
    
    public function getUsuarios(int $page = 1, int $perPage = 10): array
    {
        return $this->get("/usuarios?page=$page&per_page=$perPage");
    }
    
    public function getUsuario(int $id): array
    {
        return $this->get("/usuarios/$id");
    }
    
    private function get(string $endpoint): array
    {
        return $this->request('GET', $endpoint);
    }
    
    private function post(string $endpoint, array $data): array
    {
        return $this->request('POST', $endpoint, $data);
    }
    
    private function request(string $method, string $endpoint, ?array $data = null): array
    {
        $url = $this->baseUrl . $endpoint;
        
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, $method);
        
        $headers = ['Content-Type: application/json'];
        
        if ($this->token) {
            $headers[] = "Authorization: Bearer {$this->token}";
        }
        
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        
        if ($data) {
            curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
        }
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        return json_decode($response, true);
    }
}

// USO
$client = new ApiClient('http://localhost/api/v1');

// Registrar
$result = $client->register('João Silva', 'joao@example.com', 'senha123');
print_r($result);

// Login
$result = $client->login('joao@example.com', 'senha123');
print_r($result);

// Obter usuário autenticado
$me = $client->getMe();
print_r($me);

// Listar usuários
$usuarios = $client->getUsuarios(1, 10);
print_r($usuarios);
```

---

## 📝 Exercícios do Capítulo 36

### Nível Iniciante
1. Crie endpoint GET /produtos que retorna lista de produtos
2. Adicione endpoint POST /produtos para criar produto
3. Implemente validação de campos obrigatórios
4. Teste API com Postman ou Insomnia

### Nível Intermediário
5. Adicione filtros de busca em GET /produtos?nome=X&preco_max=Y
6. Implemente soft delete em produtos
7. Crie relacionamento: produto pertence a categoria
8. Adicione ordenação customizada (sort=preco, sort=-nome)

### Nível Avançado
9. Implemente refresh token JWT
10. Adicione upload de imagem de produto
11. Crie sistema de permissões (admin, user)
12. Implemente cache de respostas

---

## 🎯 Resumo do Capítulo

### ✅ O que você aprendeu:

- ✅ **Conceito de API:** Ponte entre sistemas
- ✅ **REST:** Recursos, métodos HTTP, status codes
- ✅ **Estrutura:** Roteamento, controladores, middleware
- ✅ **Autenticação JWT:** Stateless, seguro, escalável
- ✅ **Validação:** Service de validação reutilizável
- ✅ **Rate Limiting:** Proteção contra abuso
- ✅ **Versionamento:** Manter compatibilidade
- ✅ **Documentação:** Essencial para consumidores

### 🔒 Boas Práticas Aplicadas:

1. ✅ Respostas JSON padronizadas
2. ✅ Status codes corretos
3. ✅ Validação de entrada
4. ✅ Autenticação com JWT
5. ✅ Rate limiting
6. ✅ CORS configurado
7. ✅ Tratamento de erros
8. ✅ Versionamento de API

---

**Parabéns! Você criou sua primeira API REST profissional! 🎉**

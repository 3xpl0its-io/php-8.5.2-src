# PHP 8.5 Moderno: Criando sua Própria API REST

**Capítulo 36: APIs - Do Conceito à Implementação Completa**

---

## Índice do Capítulo

1. [O que é uma API?](#361-o-que-é-uma-api)
2. [Como APIs Funcionam](#362-como-apis-funcionam)
3. [REST API - Conceitos](#363-rest-api---conceitos)
4. [Estrutura de Projeto](#364-estrutura-de-projeto)
5. [Roteamento](#365-roteamento)
6. [Controladores](#366-controladores)
7. [Respostas JSON](#367-respostas-json)
8. [Autenticação JWT](#368-autenticação-jwt)
9. [Middleware](#369-middleware)
10. [Validação de Requisições](#3610-validação-de-requisições)
11. [Rate Limiting](#3611-rate-limiting)
12. [Versionamento](#3612-versionamento)
13. [Documentação](#3613-documentação)
14. [API Completa - Projeto Final](#3614-api-completa---projeto-final)

---

## Capítulo 36: Criando sua Própria API REST

### 36.1 O que é uma API?

```php
<?php

declare(strict_types=1);

/**
 * API (Application Programming Interface)
 * 
 * Uma API é uma "ponte" que permite que diferentes sistemas conversem entre si.
 * 
 * ANALOGIA DO MUNDO REAL:
 * =====================
 * 
 * Imagine um restaurante:
 * 
 * - VOCÊ (Cliente) = Aplicação Frontend
 * - GARÇOM = API
 * - COZINHA = Banco de Dados/Backend
 * 
 * Você não entra na cozinha para pegar comida.
 * Você fala com o GARÇOM (API), que:
 * 1. Recebe seu pedido
 * 2. Leva para a cozinha
 * 3. Traz de volta sua comida
 * 4. Você nunca precisa saber como a comida foi feita
 * 
 * EXEMPLO PRÁTICO:
 * ===============
 * 
 * Site de clima:
 * - Seu site (Frontend) faz requisição para API do clima
 * - API retorna dados: { "temperatura": 25, "cidade": "São Paulo" }
 * - Seu site exibe: "25°C em São Paulo"
 * 
 * Você NÃO precisa:
 * - Ter estações meteorológicas
 * - Medir temperatura
 * - Armazenar dados
 * 
 * Você SÓ precisa:
 * - Fazer uma requisição HTTP
 * - Receber a resposta JSON
 */

// Exemplo de consumo de API externa
function obterClima(string $cidade): array
{
    $url = "https://api.openweathermap.org/data/2.5/weather?q={$cidade}";
    
    $response = file_get_contents($url);
    
    return json_decode($response, true);
}

$clima = obterClima('Sao Paulo');
echo "Temperatura: {$clima['main']['temp']}°C\n";
```

### Por que criar sua própria API?

```php
<?php

declare(strict_types=1);

/**
 * MOTIVOS PARA CRIAR UMA API:
 * ==========================
 * 
 * 1. SEPARAÇÃO FRONTEND/BACKEND
 *    - React/Vue/Angular (Frontend)
 *    - PHP API (Backend)
 *    - Desenvolvimento independente
 * 
 * 2. MÚLTIPLOS CLIENTES
 *    - Site web
 *    - App mobile (iOS/Android)
 *    - App desktop
 *    - Todos usando a MESMA API
 * 
 * 3. INTEGRAÇÃO COM TERCEIROS
 *    - Permitir que outros desenvolvam usando seus dados
 *    - Exemplo: API do Twitter, Facebook, Google
 * 
 * 4. MICROSERVIÇOS
 *    - Dividir aplicação grande em serviços pequenos
 *    - Cada serviço com sua própria API
 * 
 * 5. REUTILIZAÇÃO
 *    - Mesma lógica de negócio
 *    - Múltiplas interfaces
 */
```

---

### 36.2 Como APIs Funcionam

```php
<?php

declare(strict_types=1);

/**
 * CICLO DE VIDA DE UMA REQUISIÇÃO API
 * ===================================
 * 
 * 1. CLIENTE faz REQUISIÇÃO HTTP
 *    ↓
 *    GET https://api.exemplo.com/usuarios/123
 *    Headers: Authorization: Bearer TOKEN
 *    
 * 2. SERVIDOR RECEBE
 *    ↓
 *    - Verifica autenticação
 *    - Valida dados
 *    - Processa lógica de negócio
 *    
 * 3. BANCO DE DADOS
 *    ↓
 *    SELECT * FROM usuarios WHERE id = 123
 *    
 * 4. SERVIDOR RESPONDE
 *    ↓
 *    Status: 200 OK
 *    Content-Type: application/json
 *    Body: { "id": 123, "nome": "João" }
 *    
 * 5. CLIENTE RECEBE e PROCESSA
 *    ↓
 *    Exibe dados na tela
 */

/**
 * MÉTODOS HTTP (Verbos)
 * =====================
 */

// GET - Buscar dados (não modifica nada)
// Exemplo: GET /usuarios/123
// Retorna dados do usuário 123

// POST - Criar novo recurso
// Exemplo: POST /usuarios
// Body: { "nome": "João", "email": "joao@example.com" }
// Cria novo usuário

// PUT - Atualizar recurso completo
// Exemplo: PUT /usuarios/123
// Body: { "nome": "João Silva", "email": "joao.silva@example.com" }
// Substitui todos os dados

// PATCH - Atualizar parcialmente
// Exemplo: PATCH /usuarios/123
// Body: { "nome": "João Silva" }
// Atualiza apenas o nome

// DELETE - Deletar recurso
// Exemplo: DELETE /usuarios/123
// Remove usuário 123

/**
 * STATUS CODES (Códigos de Resposta)
 * ==================================
 */

// 2xx - SUCESSO
const STATUS_OK = 200;                    // Sucesso geral
const STATUS_CREATED = 201;               // Recurso criado
const STATUS_NO_CONTENT = 204;            // Sucesso sem conteúdo (DELETE)

// 3xx - REDIRECIONAMENTO
const STATUS_MOVED_PERMANENTLY = 301;     // Recurso movido
const STATUS_NOT_MODIFIED = 304;          // Usar cache

// 4xx - ERRO DO CLIENTE
const STATUS_BAD_REQUEST = 400;           // Requisição inválida
const STATUS_UNAUTHORIZED = 401;          // Não autenticado
const STATUS_FORBIDDEN = 403;             // Sem permissão
const STATUS_NOT_FOUND = 404;             // Recurso não existe
const STATUS_METHOD_NOT_ALLOWED = 405;    // Método HTTP errado
const STATUS_CONFLICT = 409;              // Conflito (email duplicado)
const STATUS_UNPROCESSABLE = 422;         // Validação falhou
const STATUS_TOO_MANY_REQUESTS = 429;     // Rate limit excedido

// 5xx - ERRO DO SERVIDOR
const STATUS_INTERNAL_ERROR = 500;        // Erro genérico servidor
const STATUS_SERVICE_UNAVAILABLE = 503;   // Servidor indisponível

/**
 * HEADERS HTTP IMPORTANTES
 * ========================
 */

// Content-Type: Tipo do conteúdo
// application/json - JSON
// application/xml - XML
// multipart/form-data - Upload de arquivo

// Authorization: Autenticação
// Bearer TOKEN - JWT Token
// Basic BASE64 - Basic Auth

// Accept: O que o cliente aceita receber
// application/json

// Cache-Control: Controle de cache
// no-cache, max-age=3600

/**
 * FORMATO DE RESPOSTA JSON
 * ========================
 */

// Sucesso - Único recurso
$response = [
    'success' => true,
    'data' => [
        'id' => 123,
        'nome' => 'João Silva',
        'email' => 'joao@example.com'
    ]
];

// Sucesso - Lista de recursos
$response = [
    'success' => true,
    'data' => [
        ['id' => 1, 'nome' => 'João'],
        ['id' => 2, 'nome' => 'Maria']
    ],
    'meta' => [
        'total' => 100,
        'page' => 1,
        'per_page' => 10
    ]
];

// Erro - Validação
$response = [
    'success' => false,
    'error' => [
        'code' => 'VALIDATION_ERROR',
        'message' => 'Dados inválidos',
        'details' => [
            'email' => ['O campo email é obrigatório'],
            'nome' => ['O campo nome deve ter no mínimo 3 caracteres']
        ]
    ]
];

// Erro - Não encontrado
$response = [
    'success' => false,
    'error' => [
        'code' => 'NOT_FOUND',
        'message' => 'Usuário não encontrado'
    ]
];
```

---

### 36.3 REST API - Conceitos

```php
<?php

declare(strict_types=1);

/**
 * REST (Representational State Transfer)
 * ======================================
 * 
 * REST é um estilo arquitetural para APIs que usa:
 * - HTTP methods (GET, POST, PUT, DELETE)
 * - URLs para identificar recursos
 * - JSON para troca de dados
 * - Stateless (sem estado entre requisições)
 * 
 * PRINCÍPIOS REST:
 * ===============
 * 
 * 1. RECURSOS são substantivos (não verbos)
 *    ✅ GET /usuarios
 *    ✅ GET /produtos
 *    ❌ GET /obterUsuarios
 *    ❌ GET /listarProdutos
 * 
 * 2. USAR MÉTODOS HTTP corretamente
 *    ✅ GET /usuarios - Listar
 *    ✅ POST /usuarios - Criar
 *    ✅ GET /usuarios/123 - Buscar um
 *    ✅ PUT /usuarios/123 - Atualizar
 *    ✅ DELETE /usuarios/123 - Deletar
 * 
 * 3. STATELESS (sem estado)
 *    - Cada requisição é independente
 *    - Servidor não guarda informação entre requisições
 *    - Use tokens (JWT) para autenticação
 * 
 * 4. URLs HIERÁRQUICAS
 *    ✅ GET /usuarios/123/pedidos
 *    ✅ GET /usuarios/123/pedidos/456
 *    ✅ GET /categorias/5/produtos
 * 
 * 5. FILTROS via QUERY PARAMETERS
 *    ✅ GET /usuarios?ativo=true
 *    ✅ GET /produtos?categoria=eletronicos&preco_max=1000
 *    ✅ GET /pedidos?data_inicio=2024-01-01&data_fim=2024-12-31
 * 
 * 6. PAGINAÇÃO
 *    ✅ GET /usuarios?page=1&per_page=10
 *    ✅ GET /produtos?limit=20&offset=40
 * 
 * 7. ORDENAÇÃO
 *    ✅ GET /usuarios?sort=nome
 *    ✅ GET /produtos?sort=-preco (- = descendente)
 * 
 * 8. VERSIONAMENTO
 *    ✅ GET /v1/usuarios
 *    ✅ GET /v2/usuarios
 *    ✅ Header: Accept: application/vnd.api.v1+json
 */

/**
 * EXEMPLO DE ESTRUTURA REST COMPLETA
 * ==================================
 */

// USUÁRIOS
// --------
// GET    /api/v1/usuarios              - Listar todos
// GET    /api/v1/usuarios/123          - Buscar um
// POST   /api/v1/usuarios              - Criar
// PUT    /api/v1/usuarios/123          - Atualizar completo
// PATCH  /api/v1/usuarios/123          - Atualizar parcial
// DELETE /api/v1/usuarios/123          - Deletar

// PEDIDOS DO USUÁRIO
// ------------------
// GET    /api/v1/usuarios/123/pedidos  - Pedidos do usuário 123
// GET    /api/v1/pedidos?usuario_id=123 - Mesma coisa (alternativa)

// PRODUTOS
// --------
// GET    /api/v1/produtos              - Listar
// GET    /api/v1/produtos/456          - Buscar
// GET    /api/v1/produtos?categoria=eletronicos&preco_max=1000

// AUTENTICAÇÃO
// -----------
// POST   /api/v1/auth/login            - Login (retorna token)
// POST   /api/v1/auth/register         - Registro
// POST   /api/v1/auth/logout           - Logout
// GET    /api/v1/auth/me               - Dados do usuário autenticado
```

---

### 36.4 Estrutura de Projeto

```php
<?php

declare(strict_types=1);

/**
 * ESTRUTURA DE DIRETÓRIOS
 * =======================
 * 
 * api/
 * ├── public/
 * │   └── index.php              # Ponto de entrada único
 * ├── src/
 * │   ├── Config/
 * │   │   └── Database.php       # Configuração do banco
 * │   ├── Controllers/
 * │   │   ├── AuthController.php
 * │   │   ├── UserController.php
 * │   │   └── ProductController.php
 * │   ├── Models/
 * │   │   ├── User.php
 * │   │   └── Product.php
 * │   ├── Repositories/
 * │   │   ├── UserRepository.php
 * │   │   └── ProductRepository.php
 * │   ├── Services/
 * │   │   ├── AuthService.php
 * │   │   └── ValidationService.php
 * │   ├── Middleware/
 * │   │   ├── AuthMiddleware.php
 * │   │   └── RateLimitMiddleware.php
 * │   ├── Http/
 * │   │   ├── Request.php
 * │   │   ├── Response.php
 * │   │   └── Router.php
 * │   └── Helpers/
 * │       └── JWT.php
 * ├── .env                        # Variáveis de ambiente
 * ├── .htaccess                   # Reescrita de URL
 * └── composer.json               # Dependências (se usar)
 */
```

### Arquivo .htaccess (Apache)

```apache
# public/.htaccess

RewriteEngine On

# Redirecionar tudo para index.php
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]

# Headers CORS (se precisar)
Header set Access-Control-Allow-Origin "*"
Header set Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS"
Header set Access-Control-Allow-Headers "Content-Type, Authorization"
```

### Configuração Nginx

```nginx
# nginx.conf

server {
    listen 80;
    server_name api.exemplo.com;
    root /var/www/api/public;
    
    index index.php;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

---

### 36.5 Roteamento

```php
<?php
// src/Http/Router.php

declare(strict_types=1);

namespace App\Http;

class Router
{
    private array $routes = [];
    
    public function get(string $path, callable|array $handler): void
    {
        $this->addRoute('GET', $path, $handler);
    }
    
    public function post(string $path, callable|array $handler): void
    {
        $this->addRoute('POST', $path, $handler);
    }
    
    public function put(string $path, callable|array $handler): void
    {
        $this->addRoute('PUT', $path, $handler);
    }
    
    public function patch(string $path, callable|array $handler): void
    {
        $this->addRoute('PATCH', $path, $handler);
    }
    
    public function delete(string $path, callable|array $handler): void
    {
        $this->addRoute('DELETE', $path, $handler);
    }
    
    private function addRoute(string $method, string $path, callable|array $handler): void
    {
        // Converter /usuarios/{id} para regex
        $pattern = preg_replace('/\{([a-zA-Z]+)\}/', '(?P<$1>[^/]+)', $path);
        $pattern = '#^' . $pattern . '$#';
        
        $this->routes[] = [
            'method' => $method,
            'path' => $path,
            'pattern' => $pattern,
            'handler' => $handler
        ];
    }
    
    public function dispatch(Request $request): Response
    {
        $method = $request->getMethod();
        $uri = $request->getUri();
        
        foreach ($this->routes as $route) {
            if ($route['method'] !== $method) {
                continue;
            }
            
            if (preg_match($route['pattern'], $uri, $matches)) {
                // Extrair parâmetros da URL
                $params = array_filter($matches, 'is_string', ARRAY_FILTER_USE_KEY);
                
                $handler = $route['handler'];
                
                // Se handler é array [Controller, 'method']
                if (is_array($handler)) {
                    [$controller, $method] = $handler;
                    $controller = new $controller();
                    
                    return $controller->$method($request, $params);
                }
                
                // Se handler é closure
                return $handler($request, $params);
            }
        }
        
        // Rota não encontrada
        return Response::json([
            'success' => false,
            'error' => [
                'code' => 'NOT_FOUND',
                'message' => 'Rota não encontrada'
            ]
        ], 404);
    }
}
```

### Classe Request

```php
<?php
// src/Http/Request.php

declare(strict_types=1);

namespace App\Http;

class Request
{
    private array $query;
    private array $body;
    private array $headers;
    private string $method;
    private string $uri;
    
    public function __construct()
    {
        $this->query = $_GET;
        $this->method = $_SERVER['REQUEST_METHOD'];
        $this->uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
        
        // Parse body JSON
        $input = file_get_contents('php://input');
        $this->body = json_decode($input, true) ?? [];
        
        // Parse headers
        $this->headers = $this->parseHeaders();
    }
    
    private function parseHeaders(): array
    {
        $headers = [];
        
        foreach ($_SERVER as $key => $value) {
            if (str_starts_with($key, 'HTTP_')) {
                $header = str_replace('_', '-', substr($key, 5));
                $headers[strtolower($header)] = $value;
            }
        }
        
        return $headers;
    }
    
    public function getMethod(): string
    {
        return $this->method;
    }
    
    public function getUri(): string
    {
        return $this->uri;
    }
    
    public function query(string $key, mixed $default = null): mixed
    {
        return $this->query[$key] ?? $default;
    }
    
    public function input(string $key, mixed $default = null): mixed
    {
        return $this->body[$key] ?? $default;
    }
    
    public function all(): array
    {
        return $this->body;
    }
    
    public function header(string $key, mixed $default = null): mixed
    {
        return $this->headers[strtolower($key)] ?? $default;
    }
    
    public function bearerToken(): ?string
    {
        $header = $this->header('authorization');
        
        if ($header && str_starts_with($header, 'Bearer ')) {
            return substr($header, 7);
        }
        
        return null;
    }
}
```

### Classe Response

```php
<?php
// src/Http/Response.php

declare(strict_types=1);

namespace App\Http;

class Response
{
    public function __construct(
        private mixed $data,
        private int $status = 200,
        private array $headers = []
    ) {}
    
    public static function json(mixed $data, int $status = 200, array $headers = []): self
    {
        return new self($data, $status, $headers);
    }
    
    public function send(): void
    {
        // Definir código HTTP
        http_response_code($this->status);
        
        // Headers padrão
        header('Content-Type: application/json; charset=utf-8');
        
        // Headers customizados
        foreach ($this->headers as $key => $value) {
            header("$key: $value");
        }
        
        // Enviar JSON
        echo json_encode($this->data, JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES);
        
        exit;
    }
}
```

---

### 36.6 Controladores

```php
<?php
// src/Controllers/UserController.php

declare(strict_types=1);

namespace App\Controllers;

use App\Http\Request;
use App\Http\Response;
use App\Repositories\UserRepository;
use App\Services\ValidationService;

class UserController
{
    public function __construct(
        private UserRepository $repository = new UserRepository(),
        private ValidationService $validator = new ValidationService()
    ) {}
    
    /**
     * GET /api/v1/usuarios
     * Listar todos os usuários
     */
    public function index(Request $request): Response
    {
        // Query parameters
        $page = (int) $request->query('page', 1);
        $perPage = (int) $request->query('per_page', 10);
        $search = $request->query('search');
        
        // Buscar usuários
        $usuarios = $this->repository->paginate($page, $perPage, $search);
        $total = $this->repository->count($search);
        
        return Response::json([
            'success' => true,
            'data' => $usuarios,
            'meta' => [
                'total' => $total,
                'page' => $page,
                'per_page' => $perPage,
                'last_page' => (int) ceil($total / $perPage)
            ]
        ]);
    }
    
    /**
     * GET /api/v1/usuarios/{id}
     * Buscar um usuário
     */
    public function show(Request $request, array $params): Response
    {
        $id = (int) $params['id'];
        
        $usuario = $this->repository->find($id);
        
        if (!$usuario) {
            return Response::json([
                'success' => false,
                'error' => [
                    'code' => 'NOT_FOUND',
                    'message' => 'Usuário não encontrado'
                ]
            ], 404);
        }
        
        return Response::json([
            'success' => true,
            'data' => $usuario
        ]);
    }
    
    /**
     * POST /api/v1/usuarios
     * Criar novo usuário
     */
    public function store(Request $request): Response
    {
        // Validar dados
        $errors = $this->validator->validate($request->all(), [
            'nome' => ['required', 'min:3', 'max:100'],
            'email' => ['required', 'email', 'unique:usuarios'],
            'senha' => ['required', 'min:8']
        ]);
        
        if (!empty($errors)) {
            return Response::json([
                'success' => false,
                'error' => [
                    'code' => 'VALIDATION_ERROR',
                    'message' => 'Dados inválidos',
                    'details' => $errors
                ]
            ], 422);
        }
        
        // Criar usuário
        $dados = [
            'nome' => $request->input('nome'),
            'email' => $request->input('email'),
            'senha_hash' => password_hash($request->input('senha'), PASSWORD_ARGON2ID)
        ];
        
        $id = $this->repository->create($dados);
        $usuario = $this->repository->find($id);
        
        return Response::json([
            'success' => true,
            'data' => $usuario,
            'message' => 'Usuário criado com sucesso'
        ], 201);
    }
    
    /**
     * PUT /api/v1/usuarios/{id}
     * Atualizar usuário completo
     */
    public function update(Request $request, array $params): Response
    {
        $id = (int) $params['id'];
        
        // Verificar se existe
        if (!$this->repository->find($id)) {
            return Response::json([
                'success' => false,
                'error' => [
                    'code' => 'NOT_FOUND',
                    'message' => 'Usuário não encontrado'
                ]
            ], 404);
        }
        
        // Validar
        $errors = $this->validator->validate($request->all(), [
            'nome' => ['required', 'min:3'],
            'email' => ['required', 'email', "unique:usuarios,id,$id"]
        ]);
        
        if (!empty($errors)) {
            return Response::json([
                'success' => false,
                'error' => [
                    'code' => 'VALIDATION_ERROR',
                    'message' => 'Dados inválidos',
                    'details' => $errors
                ]
            ], 422);
        }
        
        // Atualizar
        $dados = [
            'nome' => $request->input('nome'),
            'email' => $request->input('email')
        ];
        
        $this->repository->update($id, $dados);
        $usuario = $this->repository->find($id);
        
        return Response::json([
            'success' => true,
            'data' => $usuario,
            'message' => 'Usuário atualizado com sucesso'
        ]);
    }
    
    /**
     * DELETE /api/v1/usuarios/{id}
     * Deletar usuário
     */
    public function destroy(Request $request, array $params): Response
    {
        $id = (int) $params['id'];
        
        if (!$this->repository->find($id)) {
            return Response::json([
                'success' => false,
                'error' => [
                    'code' => 'NOT_FOUND',
                    'message' => 'Usuário não encontrado'
                ]
            ], 404);
        }
        
        $this->repository->delete($id);
        
        return Response::json([
            'success' => true,
            'message' => 'Usuário deletado com sucesso'
        ], 204);
    }
}
```

---

### 36.7 Respostas JSON Padronizadas

```php
<?php
// src/Http/JsonResponse.php

declare(strict_types=1);

namespace App\Http;

class JsonResponse
{
    /**
     * Resposta de sucesso
     */
    public static function success(
        mixed $data,
        string $message = null,
        int $status = 200
    ): Response {
        $response = ['success' => true];
        
        if ($data !== null) {
            $response['data'] = $data;
        }
        
        if ($message !== null) {
            $response['message'] = $message;
        }
        
        return Response::json($response, $status);
    }
    
    /**
     * Resposta de erro
     */
    public static function error(
        string $code,
        string $message,
        mixed $details = null,
        int $status = 400
    ): Response {
        $response = [
            'success' => false,
            'error' => [
                'code' => $code,
                'message' => $message
            ]
        ];
        
        if ($details !== null) {
            $response['error']['details'] = $details;
        }
        
        return Response::json($response, $status);
    }
    
    /**
     * Resposta de validação
     */
    public static function validationError(array $errors): Response
    {
        return self::error(
            'VALIDATION_ERROR',
            'Os dados fornecidos são inválidos',
            $errors,
            422
        );
    }
    
    /**
     * Resposta de não encontrado
     */
    public static function notFound(string $resource = 'Recurso'): Response
    {
        return self::error(
            'NOT_FOUND',
            "$resource não encontrado",
            null,
            404
        );
    }
    
    /**
     * Resposta de não autorizado
     */
    public static function unauthorized(string $message = 'Não autorizado'): Response
    {
        return self::error(
            'UNAUTHORIZED',
            $message,
            null,
            401
        );
    }
    
    /**
     * Resposta de proibido
     */
    public static function forbidden(string $message = 'Acesso negado'): Response
    {
        return self::error(
            'FORBIDDEN',
            $message,
            null,
            403
        );
    }
    
    /**
     * Resposta de conflito
     */
    public static function conflict(string $message = 'Conflito de recursos'): Response
    {
        return self::error(
            'CONFLICT',
            $message,
            null,
            409
        );
    }
}
```

---

**Continua no próximo arquivo com:**
- Autenticação JWT
- Middleware
- Rate Limiting
- Versionamento
- API completa funcional

Deseja que eu continue criando a parte 2 com o resto do capítulo?

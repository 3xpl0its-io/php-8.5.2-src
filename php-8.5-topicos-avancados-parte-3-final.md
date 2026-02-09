# PHP 8.5 Moderno: Tópicos Avançados - Parte 3 (Final)

**Capítulos finais: Validação de Forms e cURL**

---

## Capítulo 33: Validação de Forms - E-mail e URL

### 33.1 Validação de E-mail com filter_var

```php
<?php

declare(strict_types=1);

class EmailValidator
{
    // Validação básica
    public static function validar(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
    
    // Validação com normalização
    public static function validarENormalizar(string $email): ?string
    {
        // Remover espaços
        $email = trim($email);
        
        // Converter para minúsculas
        $email = strtolower($email);
        
        // Validar
        if (filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
            return null;
        }
        
        return $email;
    }
    
    // Validação com sanitização
    public static function sanitizar(string $email): string
    {
        return filter_var($email, FILTER_SANITIZE_EMAIL);
    }
    
    // Validação completa
    public static function validarCompleto(string $email): array
    {
        $resultado = [
            'valido' => false,
            'email' => null,
            'erros' => []
        ];
        
        // Remover espaços
        $email = trim($email);
        
        if (empty($email)) {
            $resultado['erros'][] = 'Email não pode estar vazio';
            return $resultado;
        }
        
        // Sanitizar
        $emailLimpo = self::sanitizar($email);
        
        // Validar formato
        if (!self::validar($emailLimpo)) {
            $resultado['erros'][] = 'Formato de email inválido';
            return $resultado;
        }
        
        // Validar comprimento
        if (strlen($emailLimpo) > 254) {
            $resultado['erros'][] = 'Email muito longo (máx 254 caracteres)';
            return $resultado;
        }
        
        // Extrair domínio
        [$local, $dominio] = explode('@', $emailLimpo);
        
        // Validar parte local
        if (strlen($local) > 64) {
            $resultado['erros'][] = 'Parte local muito longa (máx 64 caracteres)';
            return $resultado;
        }
        
        // Verificar domínio com DNS (opcional)
        if (!self::validarDominioDNS($dominio)) {
            $resultado['erros'][] = 'Domínio não existe';
            return $resultado;
        }
        
        $resultado['valido'] = true;
        $resultado['email'] = strtolower($emailLimpo);
        
        return $resultado;
    }
    
    // Validar domínio com DNS
    private static function validarDominioDNS(string $dominio): bool
    {
        // Verificar registro MX
        if (checkdnsrr($dominio, 'MX')) {
            return true;
        }
        
        // Verificar registro A
        if (checkdnsrr($dominio, 'A')) {
            return true;
        }
        
        return false;
    }
}

// Testes
$emails = [
    'joao@example.com',           // Válido
    'JOAO@EXAMPLE.COM',           // Válido (será normalizado)
    'joao.silva@example.com.br',  // Válido
    'joao+tag@example.com',       // Válido
    'invalid@',                   // Inválido
    '@example.com',               // Inválido
    'invalid',                    // Inválido
];

foreach ($emails as $email) {
    $resultado = EmailValidator::validarCompleto($email);
    
    if ($resultado['valido']) {
        echo "✓ {$email} → {$resultado['email']}\n";
    } else {
        echo "✗ {$email} → " . implode(', ', $resultado['erros']) . "\n";
    }
}
```

### 33.2 Validação de E-mail com Regex

```php
<?php

declare(strict_types=1);

class EmailValidatorRegex
{
    // Regex básica
    private const PATTERN_BASIC = '/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/';
    
    // Regex mais rigorosa (RFC 5322 simplificada)
    private const PATTERN_RFC = '/^[a-zA-Z0-9!#$%&\'*+\/=?^_`{|}~-]+(?:\.[a-zA-Z0-9!#$%&\'*+\/=?^_`{|}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?$/';
    
    public static function validarBasico(string $email): bool
    {
        return preg_match(self::PATTERN_BASIC, $email) === 1;
    }
    
    public static function validarRFC(string $email): bool
    {
        return preg_match(self::PATTERN_RFC, $email) === 1;
    }
    
    // Extrair domínio
    public static function extrairDominio(string $email): ?string
    {
        if (!self::validarBasico($email)) {
            return null;
        }
        
        return explode('@', $email)[1];
    }
    
    // Extrair parte local
    public static function extrairParteLocal(string $email): ?string
    {
        if (!self::validarBasico($email)) {
            return null;
        }
        
        return explode('@', $email)[0];
    }
    
    // Mascarar email
    public static function mascarar(string $email): string
    {
        if (!self::validarBasico($email)) {
            return '***';
        }
        
        [$local, $dominio] = explode('@', $email);
        
        $tamanhoLocal = strlen($local);
        $mostrar = (int) ceil($tamanhoLocal / 3);
        
        $mascarado = substr($local, 0, $mostrar) 
                   . str_repeat('*', $tamanhoLocal - $mostrar)
                   . '@' 
                   . $dominio;
        
        return $mascarado;
    }
}

// Testes
echo EmailValidatorRegex::mascarar('joao.silva@example.com');  // joa*******@example.com
echo EmailValidatorRegex::extrairDominio('joao@example.com');  // example.com
```

### 33.3 Validação de URL

```php
<?php

declare(strict_types=1);

class URLValidator
{
    // Validação básica com filter_var
    public static function validar(string $url): bool
    {
        return filter_var($url, FILTER_VALIDATE_URL) !== false;
    }
    
    // Sanitizar URL
    public static function sanitizar(string $url): string
    {
        return filter_var($url, FILTER_SANITIZE_URL);
    }
    
    // Validação com protocolo específico
    public static function validarHTTP(string $url): bool
    {
        if (!self::validar($url)) {
            return false;
        }
        
        $esquema = parse_url($url, PHP_URL_SCHEME);
        
        return in_array($esquema, ['http', 'https'], true);
    }
    
    // Validação completa
    public static function validarCompleto(string $url): array
    {
        $resultado = [
            'valido' => false,
            'url' => null,
            'componentes' => [],
            'erros' => []
        ];
        
        // Remover espaços
        $url = trim($url);
        
        if (empty($url)) {
            $resultado['erros'][] = 'URL não pode estar vazia';
            return $resultado;
        }
        
        // Validar formato
        if (!self::validar($url)) {
            $resultado['erros'][] = 'Formato de URL inválido';
            return $resultado;
        }
        
        // Parse componentes
        $componentes = parse_url($url);
        
        if ($componentes === false) {
            $resultado['erros'][] = 'URL malformada';
            return $resultado;
        }
        
        // Validar esquema
        if (!isset($componentes['scheme'])) {
            $resultado['erros'][] = 'Esquema (http/https) obrigatório';
            return $resultado;
        }
        
        if (!in_array($componentes['scheme'], ['http', 'https'], true)) {
            $resultado['erros'][] = 'Apenas http e https são permitidos';
            return $resultado;
        }
        
        // Validar host
        if (!isset($componentes['host'])) {
            $resultado['erros'][] = 'Host obrigatório';
            return $resultado;
        }
        
        // Validar comprimento
        if (strlen($url) > 2048) {
            $resultado['erros'][] = 'URL muito longa (máx 2048 caracteres)';
            return $resultado;
        }
        
        $resultado['valido'] = true;
        $resultado['url'] = $url;
        $resultado['componentes'] = $componentes;
        
        return $resultado;
    }
    
    // Normalizar URL
    public static function normalizar(string $url): ?string
    {
        $url = trim($url);
        
        // Adicionar http:// se não tiver esquema
        if (!preg_match('/^https?:\/\//i', $url)) {
            $url = 'http://' . $url;
        }
        
        // Validar
        if (!self::validar($url)) {
            return null;
        }
        
        // Parse e reconstruir
        $componentes = parse_url($url);
        
        $esquema = strtolower($componentes['scheme']);
        $host = strtolower($componentes['host']);
        $caminho = $componentes['path'] ?? '/';
        $query = isset($componentes['query']) ? '?' . $componentes['query'] : '';
        $fragmento = isset($componentes['fragment']) ? '#' . $componentes['fragment'] : '';
        
        return "{$esquema}://{$host}{$caminho}{$query}{$fragmento}";
    }
    
    // Extrair domínio
    public static function extrairDominio(string $url): ?string
    {
        if (!self::validar($url)) {
            return null;
        }
        
        return parse_url($url, PHP_URL_HOST);
    }
    
    // Verificar se URL está acessível
    public static function verificarAcessivel(string $url): bool
    {
        if (!self::validarHTTP($url)) {
            return false;
        }
        
        $headers = @get_headers($url);
        
        if ($headers === false) {
            return false;
        }
        
        // Verificar status code 2xx ou 3xx
        return preg_match('/HTTP\/\d\.\d\s+[23]\d\d/', $headers[0]) === 1;
    }
}

// Testes
$urls = [
    'https://example.com',
    'http://example.com/path',
    'example.com',  // Sem esquema
    'ftp://example.com',  // Esquema não HTTP
    'not-a-url',
];

foreach ($urls as $url) {
    $resultado = URLValidator::validarCompleto($url);
    
    if ($resultado['valido']) {
        echo "✓ {$url}\n";
        echo "  Host: {$resultado['componentes']['host']}\n";
    } else {
        echo "✗ {$url} → " . implode(', ', $resultado['erros']) . "\n";
    }
}

// Normalização
echo URLValidator::normalizar('example.com');  // http://example.com/
echo URLValidator::normalizar('HTTPS://EXAMPLE.COM/PATH');  // https://example.com/PATH
```

### 33.4 Formulário Completo de Contato

```php
<?php

declare(strict_types=1);

class FormularioContatoValidator
{
    private array $erros = [];
    
    public function validar(array $dados): bool
    {
        $this->erros = [];
        
        // Validar nome
        if (empty($dados['nome'])) {
            $this->erros['nome'] = 'Nome é obrigatório';
        } elseif (strlen($dados['nome']) < 3) {
            $this->erros['nome'] = 'Nome deve ter pelo menos 3 caracteres';
        } elseif (strlen($dados['nome']) > 100) {
            $this->erros['nome'] = 'Nome muito longo (máx 100 caracteres)';
        }
        
        // Validar email
        if (empty($dados['email'])) {
            $this->erros['email'] = 'Email é obrigatório';
        } else {
            $resultadoEmail = EmailValidator::validarCompleto($dados['email']);
            
            if (!$resultadoEmail['valido']) {
                $this->erros['email'] = implode(', ', $resultadoEmail['erros']);
            }
        }
        
        // Validar telefone (opcional)
        if (!empty($dados['telefone'])) {
            $padrao = '/^\(\d{2}\) \d{4,5}-\d{4}$/';
            
            if (preg_match($padrao, $dados['telefone']) !== 1) {
                $this->erros['telefone'] = 'Formato inválido. Use (11) 98765-4321';
            }
        }
        
        // Validar website (opcional)
        if (!empty($dados['website'])) {
            if (!URLValidator::validarHTTP($dados['website'])) {
                $this->erros['website'] = 'URL inválida';
            }
        }
        
        // Validar assunto
        if (empty($dados['assunto'])) {
            $this->erros['assunto'] = 'Assunto é obrigatório';
        } elseif (strlen($dados['assunto']) < 5) {
            $this->erros['assunto'] = 'Assunto muito curto (mín 5 caracteres)';
        }
        
        // Validar mensagem
        if (empty($dados['mensagem'])) {
            $this->erros['mensagem'] = 'Mensagem é obrigatória';
        } elseif (strlen($dados['mensagem']) < 20) {
            $this->erros['mensagem'] = 'Mensagem muito curta (mín 20 caracteres)';
        } elseif (strlen($dados['mensagem']) > 1000) {
            $this->erros['mensagem'] = 'Mensagem muito longa (máx 1000 caracteres)';
        }
        
        return empty($this->erros);
    }
    
    public function obterErros(): array
    {
        return $this->erros;
    }
}

// Processar formulário
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $validator = new FormularioContatoValidator();
    
    if ($validator->validar($_POST)) {
        // Sanitizar dados
        $dados = [
            'nome' => htmlspecialchars($_POST['nome'], ENT_QUOTES, 'UTF-8'),
            'email' => filter_var($_POST['email'], FILTER_SANITIZE_EMAIL),
            'telefone' => $_POST['telefone'] ?? '',
            'website' => filter_var($_POST['website'] ?? '', FILTER_SANITIZE_URL),
            'assunto' => htmlspecialchars($_POST['assunto'], ENT_QUOTES, 'UTF-8'),
            'mensagem' => htmlspecialchars($_POST['mensagem'], ENT_QUOTES, 'UTF-8')
        ];
        
        // Processar (enviar email, salvar no banco, etc.)
        echo "Formulário válido! Dados: " . json_encode($dados);
    } else {
        echo "Erros de validação:\n";
        print_r($validator->obterErros());
    }
}
```

### 📝 Exercícios do Capítulo 33

1. Implemente validação de CNPJ com verificação de dígitos
2. Crie validador de domínio que verifica DNS
3. Faça sistema de whitelist/blacklist de domínios de email
4. Valide URLs permitindo apenas domínios específicos
5. Crie formulário de cadastro completo com todas as validações

---

## Capítulo 34: cURL - Requisições HTTP

### 34.1 GET Request Básico

```php
<?php

declare(strict_types=1);

class HttpClient
{
    // GET simples
    public static function get(string $url): string|false
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
        
        $response = curl_exec($ch);
        
        curl_close($ch);
        
        return $response;
    }
    
    // GET com verificação de erro
    public static function getComErro(string $url): array
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
        curl_setopt($ch, CURLOPT_TIMEOUT, 30);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        $erro = curl_error($ch);
        
        curl_close($ch);
        
        return [
            'sucesso' => $response !== false,
            'codigo_http' => $httpCode,
            'resposta' => $response,
            'erro' => $erro
        ];
    }
}

// Uso
$response = HttpClient::get('https://api.example.com/users');

if ($response !== false) {
    $dados = json_decode($response, true);
    print_r($dados);
}

// Com verificação de erro
$resultado = HttpClient::getComErro('https://api.example.com/users');

if ($resultado['sucesso'] && $resultado['codigo_http'] === 200) {
    echo "Sucesso!\n";
    $dados = json_decode($resultado['resposta'], true);
} else {
    echo "Erro: {$resultado['erro']}\n";
}
```

### 34.2 POST Request com JSON

```php
<?php

declare(strict_types=1);

class HttpClient
{
    // POST com JSON
    public static function postJSON(string $url, array $dados): array
    {
        $ch = curl_init();
        
        $json = json_encode($dados);
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $json);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, [
            'Content-Type: application/json',
            'Content-Length: ' . strlen($json)
        ]);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        return [
            'codigo_http' => $httpCode,
            'resposta' => json_decode($response, true)
        ];
    }
    
    // POST com form data
    public static function postForm(string $url, array $dados): array
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($dados));
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        return [
            'codigo_http' => $httpCode,
            'resposta' => $response
        ];
    }
}

// Uso POST JSON
$dados = [
    'nome' => 'João Silva',
    'email' => 'joao@example.com',
    'idade' => 25
];

$resultado = HttpClient::postJSON('https://api.example.com/users', $dados);

if ($resultado['codigo_http'] === 201) {
    echo "Usuário criado!\n";
    print_r($resultado['resposta']);
}

// Uso POST Form
$resultado = HttpClient::postForm('https://api.example.com/login', [
    'username' => 'joao',
    'password' => 'senha123'
]);
```

### 34.3 Headers e Autenticação

```php
<?php

declare(strict_types=1);

class HttpClient
{
    // GET com headers customizados
    public static function getComHeaders(string $url, array $headers): array
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        return [
            'codigo_http' => $httpCode,
            'resposta' => $response
        ];
    }
    
    // GET com Bearer Token
    public static function getComToken(string $url, string $token): array
    {
        return self::getComHeaders($url, [
            "Authorization: Bearer $token",
            'Accept: application/json'
        ]);
    }
    
    // GET com Basic Auth
    public static function getComBasicAuth(string $url, string $usuario, string $senha): array
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_USERPWD, "$usuario:$senha");
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        return [
            'codigo_http' => $httpCode,
            'resposta' => $response
        ];
    }
}

// Uso com token
$token = 'seu-token-aqui';
$resultado = HttpClient::getComToken(
    'https://api.example.com/protected',
    $token
);

// Uso com Basic Auth
$resultado = HttpClient::getComBasicAuth(
    'https://api.example.com/admin',
    'admin',
    'senha123'
);
```

### 34.4 Upload de Arquivos

```php
<?php

declare(strict_types=1);

class HttpClient
{
    // Upload de arquivo
    public static function uploadArquivo(string $url, string $caminhoArquivo): array
    {
        if (!file_exists($caminhoArquivo)) {
            throw new InvalidArgumentException("Arquivo não encontrado: $caminhoArquivo");
        }
        
        $ch = curl_init();
        
        $cFile = new CURLFile($caminhoArquivo);
        
        $dados = [
            'arquivo' => $cFile
        ];
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $dados);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        return [
            'codigo_http' => $httpCode,
            'resposta' => $response
        ];
    }
    
    // Upload múltiplo
    public static function uploadMultiplos(string $url, array $arquivos): array
    {
        $ch = curl_init();
        
        $dados = [];
        foreach ($arquivos as $nome => $caminho) {
            if (file_exists($caminho)) {
                $dados[$nome] = new CURLFile($caminho);
            }
        }
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $dados);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        
        curl_close($ch);
        
        return [
            'codigo_http' => $httpCode,
            'resposta' => $response
        ];
    }
}

// Uso
$resultado = HttpClient::uploadArquivo(
    'https://api.example.com/upload',
    '/path/to/file.jpg'
);

// Upload múltiplo
$resultado = HttpClient::uploadMultiplos(
    'https://api.example.com/upload',
    [
        'foto1' => '/path/to/foto1.jpg',
        'foto2' => '/path/to/foto2.jpg',
        'documento' => '/path/to/doc.pdf'
    ]
);
```

### 34.5 Download de Arquivos

```php
<?php

declare(strict_types=1);

class HttpClient
{
    // Download para string
    public static function download(string $url): string|false
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
        
        $conteudo = curl_exec($ch);
        
        curl_close($ch);
        
        return $conteudo;
    }
    
    // Download para arquivo
    public static function downloadParaArquivo(string $url, string $destino): bool
    {
        $ch = curl_init();
        
        $fp = fopen($destino, 'w+');
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_FILE, $fp);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
        
        curl_exec($ch);
        
        curl_close($ch);
        fclose($fp);
        
        return file_exists($destino);
    }
    
    // Download com barra de progresso
    public static function downloadComProgresso(string $url, string $destino): bool
    {
        $ch = curl_init();
        $fp = fopen($destino, 'w+');
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_FILE, $fp);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
        curl_setopt($ch, CURLOPT_NOPROGRESS, false);
        curl_setopt($ch, CURLOPT_PROGRESSFUNCTION, function(
            $resource,
            $downloadSize,
            $downloaded,
            $uploadSize,
            $uploaded
        ) {
            if ($downloadSize > 0) {
                $progresso = ($downloaded / $downloadSize) * 100;
                echo "\rDownload: " . number_format($progresso, 2) . "%";
            }
        });
        
        curl_exec($ch);
        
        curl_close($ch);
        fclose($fp);
        
        echo "\n";
        
        return file_exists($destino);
    }
}

// Uso
$conteudo = HttpClient::download('https://example.com/arquivo.txt');

if ($conteudo !== false) {
    file_put_contents('local.txt', $conteudo);
}

// Download direto para arquivo
$sucesso = HttpClient::downloadParaArquivo(
    'https://example.com/imagem.jpg',
    'imagem_local.jpg'
);

// Com progresso
$sucesso = HttpClient::downloadComProgresso(
    'https://example.com/arquivo_grande.zip',
    'arquivo_local.zip'
);
```

### 34.6 Cliente HTTP Completo

```php
<?php

declare(strict_types=1);

class HttpClientCompleto
{
    private array $headers = [];
    private int $timeout = 30;
    
    public function __construct(
        private ?string $baseUrl = null
    ) {}
    
    public function setHeader(string $nome, string $valor): self
    {
        $this->headers["$nome: $valor"] = "$nome: $valor";
        return $this;
    }
    
    public function setTimeout(int $segundos): self
    {
        $this->timeout = $segundos;
        return $this;
    }
    
    public function get(string $endpoint, array $params = []): array
    {
        $url = $this->construirUrl($endpoint, $params);
        
        $ch = $this->inicializarCurl($url);
        
        return $this->executar($ch);
    }
    
    public function post(string $endpoint, array $dados): array
    {
        $url = $this->construirUrl($endpoint);
        
        $ch = $this->inicializarCurl($url);
        
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($dados));
        
        return $this->executar($ch);
    }
    
    public function put(string $endpoint, array $dados): array
    {
        $url = $this->construirUrl($endpoint);
        
        $ch = $this->inicializarCurl($url);
        
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'PUT');
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($dados));
        
        return $this->executar($ch);
    }
    
    public function delete(string $endpoint): array
    {
        $url = $this->construirUrl($endpoint);
        
        $ch = $this->inicializarCurl($url);
        
        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, 'DELETE');
        
        return $this->executar($ch);
    }
    
    private function construirUrl(string $endpoint, array $params = []): string
    {
        $url = $this->baseUrl ? $this->baseUrl . $endpoint : $endpoint;
        
        if (!empty($params)) {
            $url .= '?' . http_build_query($params);
        }
        
        return $url;
    }
    
    private function inicializarCurl(string $url)
    {
        $ch = curl_init();
        
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
        curl_setopt($ch, CURLOPT_TIMEOUT, $this->timeout);
        curl_setopt($ch, CURLOPT_HTTPHEADER, array_values($this->headers));
        
        return $ch;
    }
    
    private function executar($ch): array
    {
        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        $erro = curl_error($ch);
        
        curl_close($ch);
        
        return [
            'sucesso' => $response !== false && $httpCode < 400,
            'codigo_http' => $httpCode,
            'resposta' => json_decode($response, true) ?? $response,
            'erro' => $erro
        ];
    }
}

// Uso
$client = new HttpClientCompleto('https://api.example.com');

$client
    ->setHeader('Authorization', 'Bearer TOKEN')
    ->setHeader('Accept', 'application/json')
    ->setTimeout(60);

// GET
$resultado = $client->get('/users', ['page' => 1, 'limit' => 10]);

if ($resultado['sucesso']) {
    print_r($resultado['resposta']);
}

// POST
$resultado = $client->post('/users', [
    'nome' => 'João',
    'email' => 'joao@example.com'
]);

// PUT
$resultado = $client->put('/users/1', [
    'nome' => 'João Silva'
]);

// DELETE
$resultado = $client->delete('/users/1');
```

### 📝 Exercícios do Capítulo 34

1. Crie cliente para consumir API REST completa (CRUD)
2. Implemente cache de requisições HTTP
3. Faça scraper de site com cURL (respeite robots.txt)
4. Crie sistema de retry automático para requisições falhadas
5. Implemente cliente para API GraphQL usando cURL

---

## Conclusão dos Tópicos Avançados

Parabéns! Você dominou tópicos avançados de PHP 8.5! 🎉

### O que você aprendeu nesta seção:

✅ **Iterables:** Generators, Iterators, IteratorAggregate  
✅ **Namespaces:** Organização PSR-4, autoloading, aliases  
✅ **Static:** Properties, methods, late static binding  
✅ **Traits:** Composição, resolução de conflitos, traits avançados  
✅ **Interfaces:** Segregation, constantes, contratos  
✅ **Abstract Classes:** Template method, hierarquias  
✅ **Constants:** Tipadas, visibilidade, em enums  
✅ **Access Modifiers:** Public, protected, private, readonly  
✅ **Regex:** Validações, extração, substituição  
✅ **Validação:** Email, URL, formulários completos  
✅ **cURL:** GET, POST, autenticação, upload, download  

### Próximos passos:

1. 🔨 **Pratique** com projetos reais
2. 🧪 **Teste** todo código importante
3. 📊 **Analise** com PHPStan/Psalm
4. 🚀 **Otimize** performance
5. 📚 **Estude** frameworks (Laravel/Symfony)

**Continue evoluindo como desenvolvedor PHP! 💻🔥**

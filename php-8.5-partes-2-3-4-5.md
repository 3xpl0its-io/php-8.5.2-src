# PHP 8.5 Moderno: Do Zero à Maestria

## Continuação - Partes II, III, IV e V

---

# Parte II: Manipulação de Dados

## Capítulo 7: Strings

### 7.1 Criação e Manipulação Básica

```php
<?php

declare(strict_types=1);

// Strings com aspas duplas (interpola variáveis)
$nome = "João";
$mensagem = "Olá, $nome!";
echo $mensagem;  // Olá, João!

// Strings com aspas simples (literal)
$mensagem = 'Olá, $nome!';
echo $mensagem;  // Olá, $nome!

// Concatenação
$primeiro = "João";
$ultimo = "Silva";
$completo = $primeiro . " " . $ultimo;
echo $completo;  // João Silva

// Interpolação com chaves
$produto = "notebook";
$preco = 2500;
echo "O {$produto} custa R$ {$preco},00";

// String heredoc (multilinha com interpolação)
$html = <<<HTML
    <div>
        <h1>Bem-vindo, $nome!</h1>
        <p>Este é um exemplo de heredoc</p>
    </div>
HTML;

// String nowdoc (multilinha sem interpolação)
$texto = <<<'TEXTO'
    Este é um texto literal.
    Variáveis como $nome não são interpretadas.
TEXTO;
```

### 7.2 Funções Essenciais de String

```php
<?php

declare(strict_types=1);

$texto = "  PHP é uma linguagem poderosa!  ";

// Comprimento
$tamanho = strlen($texto);
echo $tamanho;  // 35

// Remover espaços em branco
$limpo = trim($texto);
echo $limpo;  // "PHP é uma linguagem poderosa!"

$esquerda = ltrim($texto);  // Remove esquerda
$direita = rtrim($texto);   // Remove direita

// Maiúsculas e minúsculas
$texto = "PHP 8.5 Moderno";
echo strtoupper($texto);  // PHP 8.5 MODERNO
echo strtolower($texto);  // php 8.5 moderno
echo ucfirst($texto);     // Php 8.5 moderno (primeira letra maiúscula)
echo ucwords($texto);     // Php 8.5 Moderno (primeira de cada palavra)

// Substituição
$frase = "Eu gosto de Python";
$nova = str_replace("Python", "PHP", $frase);
echo $nova;  // Eu gosto de PHP

// Substituição case-insensitive
$texto = "PHP é LEGAL!";
$novo = str_ireplace("php", "JavaScript", $texto);
echo $novo;  // JavaScript é LEGAL!

// Posição de substring
$texto = "PHP é incrível";
$posicao = strpos($texto, "incrível");
echo $posicao;  // 7

// Verificar se contém
if (str_contains($texto, "incrível")) {
    echo "Texto contém 'incrível'";
}

// Verificar início
if (str_starts_with($texto, "PHP")) {
    echo "Começa com PHP";
}

// Verificar fim
if (str_ends_with($texto, "incrível")) {
    echo "Termina com incrível";
}
```

### 7.3 Substring e Divisão

```php
<?php

declare(strict_types=1);

$texto = "PHP 8.5 Moderno";

// Extrair substring
$parte = substr($texto, 0, 3);
echo $parte;  // PHP

$final = substr($texto, -7);
echo $final;  // Moderno

$meio = substr($texto, 4, 3);
echo $meio;  // 8.5

// Dividir string em array
$frase = "maçã,banana,laranja,uva";
$frutas = explode(",", $frase);
print_r($frutas);  // ['maçã', 'banana', 'laranja', 'uva']

// Juntar array em string
$lista = ["PHP", "JavaScript", "Python"];
$texto = implode(", ", $lista);
echo $texto;  // PHP, JavaScript, Python

// Dividir por caractere
$texto = "PHP";
$caracteres = str_split($texto);
print_r($caracteres);  // ['P', 'H', 'P']

// Quebrar em palavras
$frase = "PHP é uma linguagem poderosa";
$palavras = str_word_count($frase, 1);
print_r($palavras);  // ['PHP', 'é', 'uma', 'linguagem', 'poderosa']
```

### 7.4 Formatação de Strings

```php
<?php

declare(strict_types=1);

// sprintf: formata string
$nome = "João";
$idade = 25;
$mensagem = sprintf("Meu nome é %s e tenho %d anos", $nome, $idade);
echo $mensagem;  // Meu nome é João e tenho 25 anos

// printf: formata e exibe
printf("Preço: R$ %.2f", 19.5);  // Preço: R$ 19.50

// Formatação de números
$numero = 1234567.89;

echo number_format($numero);  // 1,234,568
echo number_format($numero, 2);  // 1,234,567.89
echo number_format($numero, 2, ',', '.');  // 1.234.567,89

// Padding (preenchimento)
$numero = "42";
echo str_pad($numero, 5, "0", STR_PAD_LEFT);  // 00042
echo str_pad($numero, 5, "0", STR_PAD_RIGHT);  // 42000
echo str_pad($numero, 6, "0", STR_PAD_BOTH);  // 004200
```

### 7.5 Expressões Regulares (Regex)

```php
<?php

declare(strict_types=1);

// Verificar padrão
$email = "joao@example.com";
$padrao = "/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/";

if (preg_match($padrao, $email)) {
    echo "Email válido";
}

// Buscar todas as ocorrências
$texto = "PHP 8.5 e PHP 8.4 são versões modernas";
preg_match_all("/PHP \d\.\d/", $texto, $matches);
print_r($matches[0]);  // ['PHP 8.5', 'PHP 8.4']

// Substituir com regex
$texto = "Meu telefone é (11) 98765-4321";
$limpo = preg_replace("/[^0-9]/", "", $texto);
echo $limpo;  // 11987654321

// Dividir com regex
$texto = "maçã;banana,laranja|uva";
$frutas = preg_split("/[;,|]/", $texto);
print_r($frutas);  // ['maçã', 'banana', 'laranja', 'uva']

// Validações comuns
function validarCPF(string $cpf): bool
{
    $padrao = "/^\d{3}\.\d{3}\.\d{3}-\d{2}$/";
    return preg_match($padrao, $cpf) === 1;
}

function validarTelefone(string $tel): bool
{
    $padrao = "/^\(\d{2}\) \d{4,5}-\d{4}$/";
    return preg_match($padrao, $tel) === 1;
}

echo validarCPF("123.456.789-00") ? "CPF válido" : "CPF inválido";
echo validarTelefone("(11) 98765-4321") ? "Tel válido" : "Tel inválido";
```

### 7.6 Multibyte Strings (UTF-8)

```php
<?php

declare(strict_types=1);

// Strings com caracteres especiais (acentos)
$texto = "São Paulo - Brasil";

// strlen conta bytes, não caracteres
echo strlen($texto);  // 22 (bytes)

// mb_strlen conta caracteres
echo mb_strlen($texto);  // 19 (caracteres)

// Substring com multibyte
$texto = "Programação";
echo substr($texto, 0, 7);  // Program (errado)
echo mb_substr($texto, 0, 7);  // Programa (correto)

// Maiúsculas/minúsculas com acentos
$texto = "josé da silva";
echo strtoupper($texto);  // JOSé DA SILVA (errado)
echo mb_strtoupper($texto);  // JOSÉ DA SILVA (correto)

// Sempre configure a codificação
mb_internal_encoding("UTF-8");
```

### 📝 Exercícios do Capítulo 7

1. Crie uma função que inverta uma string
2. Faça uma função que conte quantas vogais tem em uma string
3. Valide um email usando regex
4. Crie uma função que capitalize a primeira letra de cada palavra
5. Remova todos os números de uma string usando preg_replace

---

## Capítulo 8: Datas e Horas

### 8.1 Obtendo Data e Hora Atual

```php
<?php

declare(strict_types=1);

// Timestamp Unix (segundos desde 1/1/1970)
$timestamp = time();
echo $timestamp;  // Ex: 1704067200

// Data formatada
echo date('Y-m-d');  // 2026-02-09
echo date('d/m/Y');  // 09/02/2026
echo date('H:i:s');  // 14:30:45

// Data e hora completa
echo date('Y-m-d H:i:s');  // 2026-02-09 14:30:45

// Formatos comuns
echo date('l, d \de F \de Y');  // Monday, 09 de February de 2026
echo date('d/m/Y H:i');  // 09/02/2026 14:30

// Componentes individuais
echo date('Y');  // 2026 (ano)
echo date('m');  // 02 (mês)
echo date('d');  // 09 (dia)
echo date('H');  // 14 (hora)
echo date('i');  // 30 (minuto)
echo date('s');  // 45 (segundo)
```

### 8.2 Formatação de Datas

```php
<?php

declare(strict_types=1);

// Símbolos de formatação
$data = time();

// Ano
echo date('Y', $data);  // 2026 (4 dígitos)
echo date('y', $data);  // 26 (2 dígitos)

// Mês
echo date('m', $data);  // 02 (número com zero)
echo date('n', $data);  // 2 (número sem zero)
echo date('M', $data);  // Feb (abreviado)
echo date('F', $data);  // February (completo)

// Dia
echo date('d', $data);  // 09 (com zero)
echo date('j', $data);  // 9 (sem zero)
echo date('D', $data);  // Mon (abreviado)
echo date('l', $data);  // Monday (completo)

// Hora
echo date('H', $data);  // 14 (24h)
echo date('h', $data);  // 02 (12h)
echo date('a', $data);  // pm (am/pm)
echo date('A', $data);  // PM (AM/PM)

// Exemplos práticos
echo date('d/m/Y');  // 09/02/2026 (BR)
echo date('Y-m-d');  // 2026-02-09 (ISO)
echo date('d/m/Y H:i');  // 09/02/2026 14:30
```

### 8.3 DateTime Class (Orientado a Objetos)

```php
<?php

declare(strict_types=1);

// Criar DateTime
$agora = new DateTime();
echo $agora->format('Y-m-d H:i:s');

// DateTime com data específica
$natal = new DateTime('2026-12-25');
echo $natal->format('d/m/Y');  // 25/12/2026

// DateTime com hora
$reuniao = new DateTime('2026-02-10 15:30:00');
echo $reuniao->format('d/m/Y \à\s H:i');

// Modificar data
$data = new DateTime('2026-02-09');
$data->modify('+1 day');
echo $data->format('d/m/Y');  // 10/02/2026

$data->modify('+1 month');
echo $data->format('d/m/Y');  // 10/03/2026

$data->modify('+1 year');
echo $data->format('d/m/Y');  // 10/03/2027

// Métodos específicos
$data = new DateTime('2026-02-09');
$data->add(new DateInterval('P7D'));  // +7 dias
echo $data->format('d/m/Y');

$data->sub(new DateInterval('P1M'));  // -1 mês
echo $data->format('d/m/Y');
```

### 8.4 Cálculos com Datas

```php
<?php

declare(strict_types=1);

// Diferença entre datas
$inicio = new DateTime('2026-01-01');
$fim = new DateTime('2026-12-31');

$diferenca = $inicio->diff($fim);

echo $diferenca->days;  // 364 dias
echo $diferenca->m;     // 11 meses
echo $diferenca->format('%m meses e %d dias');

// Idade
function calcularIdade(string $dataNascimento): int
{
    $nascimento = new DateTime($dataNascimento);
    $hoje = new DateTime();
    
    $idade = $nascimento->diff($hoje);
    
    return $idade->y;
}

echo calcularIdade('1998-05-15');  // 27 (aproximadamente)

// Verificar se é final de semana
function isFimDeSemana(DateTime $data): bool
{
    $diaSemana = (int) $data->format('N');  // 1=segunda, 7=domingo
    return $diaSemana >= 6;
}

$hoje = new DateTime();
echo isFimDeSemana($hoje) ? "É fim de semana!" : "Dia útil";

// Adicionar dias úteis
function adicionarDiasUteis(DateTime $data, int $dias): DateTime
{
    $resultado = clone $data;
    $diasAdicionados = 0;
    
    while ($diasAdicionados < $dias) {
        $resultado->modify('+1 day');
        
        if (!isFimDeSemana($resultado)) {
            $diasAdicionados++;
        }
    }
    
    return $resultado;
}

$data = new DateTime('2026-02-09');
$prazo = adicionarDiasUteis($data, 5);
echo $prazo->format('d/m/Y');
```

### 8.5 Fusos Horários

```php
<?php

declare(strict_types=1);

// Definir fuso horário padrão
date_default_timezone_set('America/Sao_Paulo');

// DateTime com fuso horário específico
$sp = new DateTime('now', new DateTimeZone('America/Sao_Paulo'));
$ny = new DateTime('now', new DateTimeZone('America/New_York'));
$tokyo = new DateTime('now', new DateTimeZone('Asia/Tokyo'));

echo "São Paulo: " . $sp->format('H:i') . "\n";
echo "Nova York: " . $ny->format('H:i') . "\n";
echo "Tóquio: " . $tokyo->format('H:i') . "\n";

// Converter entre fusos
$saoPaulo = new DateTime('2026-02-09 15:00:00', new DateTimeZone('America/Sao_Paulo'));
$saoPaulo->setTimezone(new DateTimeZone('Europe/London'));
echo "Londres: " . $saoPaulo->format('H:i');  // 18:00 (diferença de +3h)

// Listar todos os fusos
$fusos = DateTimeZone::listIdentifiers();
echo count($fusos);  // ~400 fusos
```

### 8.6 DateTimeImmutable (Imutável)

```php
<?php

declare(strict_types=1);

// DateTime é mutável
$data1 = new DateTime('2026-02-09');
$data2 = $data1;
$data2->modify('+1 day');

echo $data1->format('d/m/Y');  // 10/02/2026 (mudou!)
echo $data2->format('d/m/Y');  // 10/02/2026

// DateTimeImmutable não muda
$data1 = new DateTimeImmutable('2026-02-09');
$data2 = $data1->modify('+1 day');

echo $data1->format('d/m/Y');  // 09/02/2026 (original)
echo $data2->format('d/m/Y');  // 10/02/2026 (novo objeto)

// Recomendado usar DateTimeImmutable para evitar bugs
function proximoDiaUtil(DateTimeImmutable $data): DateTimeImmutable
{
    $resultado = $data->modify('+1 day');
    
    while (isFimDeSemana($resultado)) {
        $resultado = $resultado->modify('+1 day');
    }
    
    return $resultado;
}
```

### 📝 Exercícios do Capítulo 8

1. Calcule quantos dias faltam para o próximo Natal
2. Crie uma função que retorne o nome do dia da semana em português
3. Calcule quantas semanas completas existem entre duas datas
4. Verifique se uma data é feriado (use array com feriados fixos)
5. Crie um calendário simples de um mês específico

---

## Capítulo 9: Arquivos e Diretórios

### 9.1 Leitura de Arquivos

```php
<?php

declare(strict_types=1);

// Ler arquivo completo
$conteudo = file_get_contents('arquivo.txt');
echo $conteudo;

// Verificar se arquivo existe
if (file_exists('arquivo.txt')) {
    echo "Arquivo existe!";
}

// Ler como array (uma linha por elemento)
$linhas = file('arquivo.txt');

foreach ($linhas as $numero => $linha) {
    echo "Linha $numero: $linha";
}

// Ler linha por linha (econômico para arquivos grandes)
$arquivo = fopen('arquivo.txt', 'r');

if ($arquivo) {
    while (($linha = fgets($arquivo)) !== false) {
        echo $linha;
    }
    
    fclose($arquivo);
}

// Ler quantidade específica de bytes
$arquivo = fopen('arquivo.txt', 'r');
$primeiros100 = fread($arquivo, 100);
fclose($arquivo);
echo $primeiros100;
```

### 9.2 Escrita de Arquivos

```php
<?php

declare(strict_types=1);

// Escrever (sobrescreve o arquivo)
file_put_contents('arquivo.txt', 'Novo conteúdo');

// Adicionar ao final (append)
file_put_contents('arquivo.txt', "\nLinha adicional", FILE_APPEND);

// Escrever com fopen
$arquivo = fopen('arquivo.txt', 'w');

if ($arquivo) {
    fwrite($arquivo, "Primeira linha\n");
    fwrite($arquivo, "Segunda linha\n");
    fclose($arquivo);
}

// Modos de abertura:
// 'r'  - Leitura
// 'w'  - Escrita (cria/sobrescreve)
// 'a'  - Append (adiciona no final)
// 'r+' - Leitura e escrita
// 'w+' - Leitura e escrita (cria/sobrescreve)
// 'a+' - Leitura e append

// Exemplo prático: Log
function registrarLog(string $mensagem): void
{
    $timestamp = date('Y-m-d H:i:s');
    $linha = "[$timestamp] $mensagem\n";
    
    file_put_contents('app.log', $linha, FILE_APPEND);
}

registrarLog('Aplicação iniciada');
registrarLog('Usuário fez login');
```

### 9.3 Informações sobre Arquivos

```php
<?php

declare(strict_types=1);

$arquivo = 'documento.txt';

// Verificações
echo file_exists($arquivo);   // Existe?
echo is_file($arquivo);       // É arquivo?
echo is_dir($arquivo);        // É diretório?
echo is_readable($arquivo);   // Pode ler?
echo is_writable($arquivo);   // Pode escrever?

// Informações
echo filesize($arquivo);      // Tamanho em bytes
echo filetype($arquivo);      // Tipo (file, dir, link)
echo filemtime($arquivo);     // Timestamp última modificação

// Data de modificação formatada
$modificado = filemtime($arquivo);
echo date('d/m/Y H:i:s', $modificado);

// Extensão do arquivo
$info = pathinfo($arquivo);
echo $info['dirname'];    // Diretório
echo $info['basename'];   // Nome completo
echo $info['extension'];  // Extensão
echo $info['filename'];   // Nome sem extensão

// Função auxiliar
function obterExtensao(string $arquivo): string
{
    return pathinfo($arquivo, PATHINFO_EXTENSION);
}

echo obterExtensao('documento.pdf');  // pdf
```

### 9.4 Manipulação de Arquivos

```php
<?php

declare(strict_types=1);

// Copiar arquivo
if (copy('origem.txt', 'destino.txt')) {
    echo "Arquivo copiado com sucesso";
}

// Renomear/mover arquivo
if (rename('antigo.txt', 'novo.txt')) {
    echo "Arquivo renomeado";
}

// Mover para outro diretório
rename('arquivo.txt', 'pasta/arquivo.txt');

// Deletar arquivo
if (unlink('arquivo.txt')) {
    echo "Arquivo deletado";
}

// Deletar com verificação
if (file_exists('arquivo.txt')) {
    unlink('arquivo.txt');
    echo "Arquivo deletado";
} else {
    echo "Arquivo não existe";
}
```

### 9.5 Trabalhando com Diretórios

```php
<?php

declare(strict_types=1);

// Criar diretório
if (mkdir('nova_pasta')) {
    echo "Diretório criado";
}

// Criar com permissões (Linux)
mkdir('pasta', 0755);

// Criar diretórios recursivamente
mkdir('pasta/subpasta/subsubpasta', 0755, true);

// Remover diretório vazio
if (rmdir('pasta_vazia')) {
    echo "Diretório removido";
}

// Listar arquivos em um diretório
$arquivos = scandir('.');

foreach ($arquivos as $arquivo) {
    if ($arquivo !== '.' && $arquivo !== '..') {
        echo $arquivo . "\n";
    }
}

// Recursivamente através de diretórios
function listarArquivos(string $diretorio): array
{
    $resultado = [];
    $items = scandir($diretorio);
    
    foreach ($items as $item) {
        if ($item === '.' || $item === '..') {
            continue;
        }
        
        $caminho = $diretorio . '/' . $item;
        
        if (is_dir($caminho)) {
            $resultado = array_merge($resultado, listarArquivos($caminho));
        } else {
            $resultado[] = $caminho;
        }
    }
    
    return $resultado;
}

$todosArquivos = listarArquivos('.');
print_r($todosArquivos);
```

### 9.6 Upload de Arquivos

```php
<?php

declare(strict_types=1);

// HTML do formulário (separado)
/*
<form method="POST" enctype="multipart/form-data">
    <input type="file" name="arquivo">
    <button type="submit">Enviar</button>
</form>
*/

// Processamento do upload
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_FILES['arquivo'])) {
    $arquivo = $_FILES['arquivo'];
    
    // Informações do arquivo
    $nome = $arquivo['name'];
    $tipo = $arquivo['type'];
    $tamanho = $arquivo['size'];
    $tmpNome = $arquivo['tmp_name'];
    $erro = $arquivo['error'];
    
    // Verificar se não houve erro
    if ($erro === UPLOAD_ERR_OK) {
        // Verificar tamanho (5MB máximo)
        if ($tamanho <= 5 * 1024 * 1024) {
            // Verificar tipo
            $tiposPermitidos = ['image/jpeg', 'image/png', 'image/gif'];
            
            if (in_array($tipo, $tiposPermitidos)) {
                // Gerar nome único
                $extensao = pathinfo($nome, PATHINFO_EXTENSION);
                $nomeUnico = uniqid() . '.' . $extensao;
                
                // Mover para diretório final
                $destino = 'uploads/' . $nomeUnico;
                
                if (move_uploaded_file($tmpNome, $destino)) {
                    echo "Upload realizado: $nomeUnico";
                } else {
                    echo "Erro ao mover arquivo";
                }
            } else {
                echo "Tipo de arquivo não permitido";
            }
        } else {
            echo "Arquivo muito grande (máx 5MB)";
        }
    } else {
        echo "Erro no upload: $erro";
    }
}
```

### 9.7 Trabalhar com CSV

```php
<?php

declare(strict_types=1);

// Ler arquivo CSV
$arquivo = fopen('dados.csv', 'r');

while (($linha = fgetcsv($arquivo)) !== false) {
    print_r($linha);
}

fclose($arquivo);

// Escrever arquivo CSV
$dados = [
    ['Nome', 'Idade', 'Cidade'],
    ['João', 25, 'São Paulo'],
    ['Maria', 30, 'Rio de Janeiro'],
    ['Pedro', 22, 'Belo Horizonte']
];

$arquivo = fopen('saida.csv', 'w');

foreach ($dados as $linha) {
    fputcsv($arquivo, $linha);
}

fclose($arquivo);

// Função auxiliar para ler CSV completo
function lerCSV(string $caminho): array
{
    $resultado = [];
    $arquivo = fopen($caminho, 'r');
    
    // Primeira linha são os cabeçalhos
    $cabecalhos = fgetcsv($arquivo);
    
    while (($linha = fgetcsv($arquivo)) !== false) {
        $resultado[] = array_combine($cabecalhos, $linha);
    }
    
    fclose($arquivo);
    
    return $resultado;
}

$usuarios = lerCSV('usuarios.csv');
/*
Array (
    [0] => ['nome' => 'João', 'idade' => 25, 'cidade' => 'São Paulo'],
    [1] => ['nome' => 'Maria', 'idade' => 30, 'cidade' => 'Rio de Janeiro']
)
*/
```

### 📝 Exercícios do Capítulo 9

1. Crie um contador de visitas salvando em arquivo
2. Leia um arquivo de texto e conte quantas palavras tem
3. Crie um sistema simples de log com níveis (INFO, WARNING, ERROR)
4. Faça backup de todos os arquivos .txt de um diretório
5. Leia um CSV e calcule a média de uma coluna numérica

---

## Capítulo 10: JSON e Serialização

### 10.1 Trabalhando com JSON

```php
<?php

declare(strict_types=1);

// Array PHP para JSON
$pessoa = [
    'nome' => 'João Silva',
    'idade' => 25,
    'email' => 'joao@example.com',
    'ativo' => true,
    'hobbies' => ['programação', 'música', 'esportes']
];

$json = json_encode($pessoa);
echo $json;
/*
{"nome":"João Silva","idade":25,"email":"joao@example.com",
 "ativo":true,"hobbies":["programação","música","esportes"]}
*/

// JSON formatado (pretty print)
$jsonFormatado = json_encode($pessoa, JSON_PRETTY_PRINT);
echo $jsonFormatado;
/*
{
    "nome": "João Silva",
    "idade": 25,
    "email": "joao@example.com",
    "ativo": true,
    "hobbies": [
        "programação",
        "música",
        "esportes"
    ]
}
*/

// Opções úteis
json_encode($pessoa, JSON_UNESCAPED_UNICODE);  // Não escapa acentos
json_encode($pessoa, JSON_UNESCAPED_SLASHES);  // Não escapa /
json_encode($pessoa, JSON_NUMERIC_CHECK);      // Converte strings numéricas
```

### 10.2 Decodificando JSON

```php
<?php

declare(strict_types=1);

$json = '{"nome":"João","idade":25,"email":"joao@example.com"}';

// Decodificar para array associativo (padrão)
$array = json_decode($json, true);
print_r($array);
/*
Array (
    [nome] => João
    [idade] => 25
    [email] => joao@example.com
)
*/

// Decodificar para objeto
$objeto = json_decode($json);
echo $objeto->nome;   // João
echo $objeto->idade;  // 25

// Verificar erros
$jsonInvalido = '{"nome": "João"';  // JSON malformado
$resultado = json_decode($jsonInvalido);

if (json_last_error() !== JSON_ERROR_NONE) {
    echo "Erro ao decodificar JSON: " . json_last_error_msg();
}
```

### 10.3 JSON com Arquivos

```php
<?php

declare(strict_types=1);

// Salvar dados em arquivo JSON
$usuarios = [
    ['id' => 1, 'nome' => 'João', 'email' => 'joao@example.com'],
    ['id' => 2, 'nome' => 'Maria', 'email' => 'maria@example.com']
];

file_put_contents(
    'usuarios.json',
    json_encode($usuarios, JSON_PRETTY_PRINT)
);

// Ler dados de arquivo JSON
$json = file_get_contents('usuarios.json');
$usuarios = json_decode($json, true);

foreach ($usuarios as $usuario) {
    echo $usuario['nome'] . "\n";
}

// Classe para gerenciar arquivo JSON
class JsonDatabase
{
    private string $arquivo;
    
    public function __construct(string $arquivo)
    {
        $this->arquivo = $arquivo;
        
        if (!file_exists($arquivo)) {
            file_put_contents($arquivo, '[]');
        }
    }
    
    public function ler(): array
    {
        $json = file_get_contents($this->arquivo);
        return json_decode($json, true) ?? [];
    }
    
    public function escrever(array $dados): void
    {
        $json = json_encode($dados, JSON_PRETTY_PRINT);
        file_put_contents($this->arquivo, $json);
    }
    
    public function adicionar(array $item): void
    {
        $dados = $this->ler();
        $dados[] = $item;
        $this->escrever($dados);
    }
}

// Uso
$db = new JsonDatabase('produtos.json');
$db->adicionar(['id' => 1, 'nome' => 'Notebook', 'preco' => 2500]);
$produtos = $db->ler();
```

### 10.4 Serialização PHP

```php
<?php

declare(strict_types=1);

// Serializar dados PHP
$dados = [
    'nome' => 'João',
    'idade' => 25,
    'hobbies' => ['programação', 'música']
];

$serializado = serialize($dados);
echo $serializado;
// a:3:{s:4:"nome";s:4:"João";s:5:"idade";i:25;s:7:"hobbies";a:2:{i:0;s:11:"programação";i:1;s:6:"música";}}

// Deserializar
$restaurado = unserialize($serializado);
print_r($restaurado);

// Serializar objetos
class Usuario
{
    public function __construct(
        public string $nome,
        public int $idade
    ) {}
}

$usuario = new Usuario('João', 25);
$serializado = serialize($usuario);

$restaurado = unserialize($serializado);
echo $restaurado->nome;  // João

// ⚠️ CUIDADO: Nunca deserialize dados não confiáveis!
// Pode executar código malicioso
```

### 10.5 API REST Simples

```php
<?php

declare(strict_types=1);

// Definir cabeçalho JSON
header('Content-Type: application/json; charset=utf-8');

// Método HTTP
$metodo = $_SERVER['REQUEST_METHOD'];

// Arquivo de dados
$arquivoDados = 'dados.json';

// Ler dados existentes
function lerDados(): array
{
    global $arquivoDados;
    
    if (!file_exists($arquivoDados)) {
        return [];
    }
    
    $json = file_get_contents($arquivoDados);
    return json_decode($json, true) ?? [];
}

// Salvar dados
function salvarDados(array $dados): void
{
    global $arquivoDados;
    
    $json = json_encode($dados, JSON_PRETTY_PRINT);
    file_put_contents($arquivoDados, $json);
}

// GET - Listar todos
if ($metodo === 'GET') {
    $dados = lerDados();
    echo json_encode($dados);
}

// POST - Criar novo
if ($metodo === 'POST') {
    $input = file_get_contents('php://input');
    $novoItem = json_decode($input, true);
    
    $dados = lerDados();
    $novoItem['id'] = count($dados) + 1;
    $dados[] = $novoItem;
    
    salvarDados($dados);
    
    http_response_code(201);
    echo json_encode($novoItem);
}

// DELETE - Remover
if ($metodo === 'DELETE') {
    $id = (int) $_GET['id'];
    
    $dados = lerDados();
    $dados = array_filter($dados, fn($item) => $item['id'] !== $id);
    
    salvarDados(array_values($dados));
    
    echo json_encode(['success' => true]);
}
```

### 📝 Exercícios do Capítulo 10

1. Crie um sistema de configurações usando JSON
2. Faça um CRUD completo (Create, Read, Update, Delete) com arquivo JSON
3. Converta um array multidimensional para JSON e vice-versa
4. Crie uma função que valide se uma string é JSON válido
5. Implemente um cache simples usando arquivos JSON

---

# Parte III: Orientação a Objetos

## Capítulo 11: Classes e Objetos

### 11.1 Criando sua Primeira Classe

```php
<?php

declare(strict_types=1);

// Definição da classe
class Pessoa
{
    // Propriedades (atributos)
    public string $nome;
    public int $idade;
    
    // Método (função da classe)
    public function apresentar(): string
    {
        return "Olá, meu nome é {$this->nome} e tenho {$this->idade} anos.";
    }
}

// Criar objeto (instância da classe)
$pessoa1 = new Pessoa();
$pessoa1->nome = "João";
$pessoa1->idade = 25;

echo $pessoa1->apresentar();
// Olá, meu nome é João e tenho 25 anos.

// Criar outro objeto
$pessoa2 = new Pessoa();
$pessoa2->nome = "Maria";
$pessoa2->idade = 30;

echo $pessoa2->apresentar();
// Olá, meu nome é Maria e tenho 30 anos.
```

### 11.2 Construtor

```php
<?php

declare(strict_types=1);

class Produto
{
    public string $nome;
    public float $preco;
    public int $estoque;
    
    // Construtor: executado ao criar objeto
    public function __construct(string $nome, float $preco, int $estoque = 0)
    {
        $this->nome = $nome;
        $this->preco = $preco;
        $this->estoque = $estoque;
    }
    
    public function obterDescricao(): string
    {
        return "{$this->nome} - R$ {$this->preco}";
    }
}

// Criar com construtor
$produto = new Produto("Notebook", 2500.00, 10);
echo $produto->obterDescricao();

// PHP 8.0+: Constructor Property Promotion
class ProdutoModerno
{
    public function __construct(
        public string $nome,
        public float $preco,
        public int $estoque = 0
    ) {}
    
    public function obterDescricao(): string
    {
        return "{$this->nome} - R$ {$this->preco}";
    }
}

$produto = new ProdutoModerno("Mouse", 50.00);
```

### 11.3 Métodos

```php
<?php

declare(strict_types=1);

class ContaBancaria
{
    public function __construct(
        private float $saldo = 0.0
    ) {}
    
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
    
    public function obterSaldo(): float
    {
        return $this->saldo;
    }
    
    public function transferir(ContaBancaria $destino, float $valor): bool
    {
        if ($this->sacar($valor)) {
            $destino->depositar($valor);
            return true;
        }
        
        return false;
    }
}

// Uso
$conta1 = new ContaBancaria(1000.00);
$conta2 = new ContaBancaria(500.00);

$conta1->sacar(100.00);
echo $conta1->obterSaldo();  // 900.00

$conta1->transferir($conta2, 200.00);
echo $conta1->obterSaldo();  // 700.00
echo $conta2->obterSaldo();  // 700.00
```

### 11.4 Propriedades e Métodos Estáticos

```php
<?php

declare(strict_types=1);

class Matematica
{
    // Constante da classe
    public const PI = 3.14159;
    
    // Propriedade estática (compartilhada por todas as instâncias)
    private static int $contadorChamadas = 0;
    
    // Método estático (pode ser chamado sem criar objeto)
    public static function somar(float $a, float $b): float
    {
        self::$contadorChamadas++;
        return $a + $b;
    }
    
    public static function areaCirculo(float $raio): float
    {
        return self::PI * ($raio ** 2);
    }
    
    public static function obterChamadas(): int
    {
        return self::$contadorChamadas;
    }
}

// Usar sem criar objeto
echo Matematica::somar(5, 3);  // 8
echo Matematica::areaCirculo(5);  // 78.53975
echo Matematica::PI;  // 3.14159
echo Matematica::obterChamadas();  // 2

// Exemplo prático: Singleton
class Configuracao
{
    private static ?Configuracao $instancia = null;
    
    private array $config = [];
    
    // Construtor privado (não pode criar com new)
    private function __construct()
    {
        $this->config = [
            'app_name' => 'Meu App',
            'version' => '1.0.0'
        ];
    }
    
    public static function obterInstancia(): Configuracao
    {
        if (self::$instancia === null) {
            self::$instancia = new Configuracao();
        }
        
        return self::$instancia;
    }
    
    public function obter(string $chave): mixed
    {
        return $this->config[$chave] ?? null;
    }
}

// Sempre retorna a mesma instância
$config1 = Configuracao::obterInstancia();
$config2 = Configuracao::obterInstancia();

var_dump($config1 === $config2);  // true
```

### 11.5 $this, self e parent

```php
<?php

declare(strict_types=1);

class Animal
{
    protected string $nome;
    
    public function __construct(string $nome)
    {
        // $this refere à instância atual
        $this->nome = $nome;
    }
    
    public function emitirSom(): string
    {
        return "Animal fazendo som";
    }
    
    public static function tipo(): string
    {
        // self refere à classe atual
        return "Animal";
    }
}

class Cachorro extends Animal
{
    public function emitirSom(): string
    {
        // parent refere à classe pai
        return parent::emitirSom() . " - Au au!";
    }
    
    public static function tipo(): string
    {
        return "Cachorro";
    }
    
    public function apresentar(): string
    {
        // $this para propriedades/métodos de instância
        return "Eu sou {$this->nome}, um " . self::tipo();
    }
}

$rex = new Cachorro("Rex");
echo $rex->emitirSom();  // Animal fazendo som - Au au!
echo $rex->apresentar();  // Eu sou Rex, um Cachorro
```

### 📝 Exercícios do Capítulo 11

1. Crie uma classe Retangulo com largura e altura, com método para calcular área
2. Faça uma classe Carro com propriedades marca, modelo, ano
3. Crie uma classe Calculadora com métodos estáticos
4. Implemente uma classe Usuario com validação de email no construtor
5. Faça uma classe Contador que conte quantas instâncias foram criadas

---

## Capítulo 12: Encapsulamento

### 12.1 Modificadores de Acesso

```php
<?php

declare(strict_types=1);

class Usuario
{
    // public: acessível de qualquer lugar
    public string $nome;
    
    // private: acessível apenas dentro da classe
    private string $senha;
    
    // protected: acessível na classe e subclasses
    protected string $email;
    
    public function __construct(string $nome, string $senha, string $email)
    {
        $this->nome = $nome;
        $this->senha = password_hash($senha, PASSWORD_DEFAULT);
        $this->email = $email;
    }
    
    // Método público para verificar senha
    public function verificarSenha(string $senha): bool
    {
        return password_verify($senha, $this->senha);
    }
    
    // Método privado (helper interno)
    private function validarEmail(string $email): bool
    {
        return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
    }
}

$usuario = new Usuario("João", "senha123", "joao@example.com");

echo $usuario->nome;  // OK (public)
// echo $usuario->senha;  // ERRO (private)
// echo $usuario->email;  // ERRO (protected)

$usuario->verificarSenha("senha123");  // OK
// $usuario->validarEmail("teste");  // ERRO (private)
```

### 12.2 Getters e Setters

```php
<?php

declare(strict_types=1);

class Produto
{
    private string $nome;
    private float $preco;
    private int $estoque;
    
    public function __construct(string $nome, float $preco, int $estoque)
    {
        $this->setNome($nome);
        $this->setPreco($preco);
        $this->setEstoque($estoque);
    }
    
    // Getter para nome
    public function getNome(): string
    {
        return $this->nome;
    }
    
    // Setter para nome com validação
    public function setNome(string $nome): void
    {
        if (strlen($nome) < 3) {
            throw new InvalidArgumentException("Nome deve ter pelo menos 3 caracteres");
        }
        
        $this->nome = $nome;
    }
    
    // Getter para preço
    public function getPreco(): float
    {
        return $this->preco;
    }
    
    // Setter para preço com validação
    public function setPreco(float $preco): void
    {
        if ($preco < 0) {
            throw new InvalidArgumentException("Preço não pode ser negativo");
        }
        
        $this->preco = $preco;
    }
    
    // Getter para estoque
    public function getEstoque(): int
    {
        return $this->estoque;
    }
    
    // Setter para estoque com validação
    public function setEstoque(int $estoque): void
    {
        if ($estoque < 0) {
            throw new InvalidArgumentException("Estoque não pode ser negativo");
        }
        
        $this->estoque = $estoque;
    }
    
    // Métodos de negócio
    public function vender(int $quantidade): bool
    {
        if ($quantidade > $this->estoque) {
            return false;
        }
        
        $this->estoque -= $quantidade;
        return true;
    }
}

$produto = new Produto("Notebook", 2500.00, 10);

echo $produto->getNome();  // Notebook
$produto->setPreco(2800.00);
echo $produto->getPreco();  // 2800.00

// $produto->setPreco(-100);  // ERRO: InvalidArgumentException
```

### 12.3 Readonly Properties (PHP 8.1+)

```php
<?php

declare(strict_types=1);

class Usuario
{
    // Propriedade readonly: pode ser atribuída apenas no construtor
    public function __construct(
        public readonly int $id,
        public readonly string $email,
        public string $nome  // Não readonly, pode mudar
    ) {}
}

$usuario = new Usuario(1, "joao@example.com", "João");

echo $usuario->id;  // 1
echo $usuario->email;  // joao@example.com

$usuario->nome = "João Silva";  // OK
// $usuario->id = 2;  // ERRO: não pode modificar readonly
// $usuario->email = "novo@email.com";  // ERRO

// Readonly Class (PHP 8.2+): todas propriedades são readonly
readonly class Configuracao
{
    public function __construct(
        public string $appName,
        public string $version,
        public bool $debug
    ) {}
}

$config = new Configuracao("Meu App", "1.0.0", true);
// Nenhuma propriedade pode ser modificada
```

### 12.4 Property Hooks (PHP 8.4+)

```php
<?php

declare(strict_types=1);

// Property Hooks: getters/setters integrados
class Produto
{
    // Hook get
    public string $nome {
        get => strtoupper($this->nome);
    }
    
    // Hook set com validação
    public float $preco {
        set {
            if ($value < 0) {
                throw new InvalidArgumentException("Preço inválido");
            }
            $this->preco = $value;
        }
    }
    
    // Ambos hooks
    public int $estoque {
        get => $this->estoque;
        set {
            if ($value < 0) {
                throw new InvalidArgumentException("Estoque inválido");
            }
            $this->estoque = $value;
        }
    }
    
    public function __construct(string $nome, float $preco, int $estoque)
    {
        $this->nome = $nome;
        $this->preco = $preco;
        $this->estoque = $estoque;
    }
}

$produto = new Produto("notebook", 2500.00, 10);
echo $produto->nome;  // NOTEBOOK (get hook aplicado)

// $produto->preco = -100;  // ERRO (validação no set hook)
```

### 📝 Exercícios do Capítulo 12

1. Crie uma classe Pessoa com propriedades privadas e getters/setters
2. Faça uma classe ContaBancaria que não permita saldo negativo
3. Implemente validação de CPF em um setter
4. Crie uma classe readonly para representar um endereço
5. Use property hooks para formatar um número de telefone automaticamente

---

## Capítulo 13: Herança e Polimorfismo

### 13.1 Herança Básica

```php
<?php

declare(strict_types=1);

// Classe base (pai/superclasse)
class Veiculo
{
    public function __construct(
        protected string $marca,
        protected string $modelo,
        protected int $ano
    ) {}
    
    public function obterDescricao(): string
    {
        return "{$this->marca} {$this->modelo} ({$this->ano})";
    }
    
    public function ligar(): string
    {
        return "Veículo ligado";
    }
}

// Classe derivada (filha/subclasse)
class Carro extends Veiculo
{
    public function __construct(
        string $marca,
        string $modelo,
        int $ano,
        private int $portas
    ) {
        // Chamar construtor da classe pai
        parent::__construct($marca, $modelo, $ano);
    }
    
    public function obterPortas(): int
    {
        return $this->portas;
    }
    
    // Sobrescrita de método (override)
    public function ligar(): string
    {
        return parent::ligar() . " - Motor do carro ligado";
    }
}

class Moto extends Veiculo
{
    public function __construct(
        string $marca,
        string $modelo,
        int $ano,
        private int $cilindradas
    ) {
        parent::__construct($marca, $modelo, $ano);
    }
    
    public function obterCilindradas(): int
    {
        return $this->cilindradas;
    }
    
    public function ligar(): string
    {
        return parent::ligar() . " - Motor da moto ligado";
    }
}

// Uso
$carro = new Carro("Toyota", "Corolla", 2024, 4);
echo $carro->obterDescricao();  // Toyota Corolla (2024)
echo $carro->ligar();  // Veículo ligado - Motor do carro ligado
echo $carro->obterPortas();  // 4

$moto = new Moto("Honda", "CB 500", 2024, 500);
echo $moto->ligar();  // Veículo ligado - Motor da moto ligado
echo $moto->obterCilindradas();  // 500
```

### 13.2 Classes Abstratas

```php
<?php

declare(strict_types=1);

// Classe abstrata: não pode ser instanciada diretamente
abstract class Forma
{
    // Método abstrato: deve ser implementado pelas subclasses
    abstract public function calcularArea(): float;
    abstract public function calcularPerimetro(): float;
    
    // Método concreto: pode ser usado pelas subclasses
    public function exibirInfo(): string
    {
        return sprintf(
            "Área: %.2f | Perímetro: %.2f",
            $this->calcularArea(),
            $this->calcularPerimetro()
        );
    }
}

class Retangulo extends Forma
{
    public function __construct(
        private float $largura,
        private float $altura
    ) {}
    
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
        private float $raio
    ) {}
    
    public function calcularArea(): float
    {
        return pi() * ($this->raio ** 2);
    }
    
    public function calcularPerimetro(): float
    {
        return 2 * pi() * $this->raio;
    }
}

// $forma = new Forma();  // ERRO: não pode instanciar classe abstrata

$retangulo = new Retangulo(5, 10);
echo $retangulo->exibirInfo();  // Área: 50.00 | Perímetro: 30.00

$circulo = new Circulo(5);
echo $circulo->exibirInfo();  // Área: 78.54 | Perímetro: 31.42
```

### 13.3 Polimorfismo

```php
<?php

declare(strict_types=1);

// Polimorfismo: objetos de classes diferentes respondendo à mesma interface

abstract class Animal
{
    public function __construct(
        protected string $nome
    ) {}
    
    abstract public function fazerSom(): string;
    
    public function apresentar(): string
    {
        return "{$this->nome} diz: " . $this->fazerSom();
    }
}

class Cachorro extends Animal
{
    public function fazerSom(): string
    {
        return "Au au!";
    }
}

class Gato extends Animal
{
    public function fazerSom(): string
    {
        return "Miau!";
    }
}

class Vaca extends Animal
{
    public function fazerSom(): string
    {
        return "Muuuu!";
    }
}

// Função que aceita qualquer Animal
function fazerAnimalFalar(Animal $animal): void
{
    echo $animal->apresentar() . "\n";
}

// Polimorfismo em ação
$animais = [
    new Cachorro("Rex"),
    new Gato("Mimi"),
    new Vaca("Mimosa")
];

foreach ($animais as $animal) {
    fazerAnimalFalar($animal);
}
/*
Rex diz: Au au!
Mimi diz: Miau!
Mimosa diz: Muuuu!
*/
```

### 13.4 Final Classes e Métodos

```php
<?php

declare(strict_types=1);

// Método final: não pode ser sobrescrito
class Usuario
{
    final public function getId(): int
    {
        return 1;
    }
    
    public function getNome(): string
    {
        return "Usuario";
    }
}

class Admin extends Usuario
{
    // public function getId(): int { }  // ERRO: não pode sobrescrever final
    
    public function getNome(): string  // OK
    {
        return "Admin";
    }
}

// Classe final: não pode ser herdada
final class Configuracao
{
    public string $valor = "teste";
}

// class MinhaConfig extends Configuracao { }  // ERRO: não pode herdar de final
```

### 📝 Exercícios do Capítulo 13

1. Crie hierarquia: Funcionario -> Gerente, Desenvolvedor com cálculo de salário
2. Faça classes abstratas: ContaBancaria -> ContaCorrente, ContaPoupanca
3. Implemente polimorfismo com diferentes formas de pagamento
4. Crie classes para veículos (Carro, Moto, Caminhão) herdando de Veiculo
5. Use final para proteger métodos críticos em uma classe de segurança

---

**Continua com Capítulos 14-23...**

(Por limitação de tamanho, continuarei na próxima parte!)

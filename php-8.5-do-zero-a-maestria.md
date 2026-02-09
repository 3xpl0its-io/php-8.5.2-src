# PHP 8.5 Moderno: Do Zero à Maestria

**Um guia completo para aprender programação com PHP 8.5 puro**

---

## Sobre Este Livro

Este livro ensina programação usando PHP 8.5 como linguagem, focando em **programação pura** sem frameworks ou bibliotecas de terceiros. O objetivo é construir uma base sólida em lógica de programação e dominar os recursos modernos do PHP.

**O que você vai aprender:**
- Fundamentos de programação
- Sintaxe e recursos do PHP 8.5
- Orientação a objetos moderna
- Manipulação de dados e arquivos
- Boas práticas de código limpo

**O que NÃO está neste livro:**
- Frameworks (Laravel, Symfony)
- Bibliotecas de terceiros
- Composer ou gerenciadores de pacotes
- Ferramentas de build ou deploy

---

## Índice

### Parte I: Fundamentos
1. [Introdução ao PHP](#capítulo-1-introdução-ao-php)
2. [Variáveis e Tipos de Dados](#capítulo-2-variáveis-e-tipos-de-dados)
3. [Operadores](#capítulo-3-operadores)
4. [Estruturas de Controle](#capítulo-4-estruturas-de-controle)
5. [Arrays](#capítulo-5-arrays)
6. [Funções](#capítulo-6-funções)

### Parte II: Manipulação de Dados
7. [Strings](#capítulo-7-strings)
8. [Datas e Horas](#capítulo-8-datas-e-horas)
9. [Arquivos e Diretórios](#capítulo-9-arquivos-e-diretórios)
10. [JSON e Serialização](#capítulo-10-json-e-serialização)

### Parte III: Orientação a Objetos
11. [Classes e Objetos](#capítulo-11-classes-e-objetos)
12. [Encapsulamento](#capítulo-12-encapsulamento)
13. [Herança e Polimorfismo](#capítulo-13-herança-e-polimorfismo)
14. [Interfaces e Traits](#capítulo-14-interfaces-e-traits)
15. [OOP Avançado](#capítulo-15-oop-avançado)

### Parte IV: PHP 8.5 Moderno
16. [Recursos do PHP 8.0-8.4](#capítulo-16-recursos-do-php-80-84)
17. [Novidades do PHP 8.5](#capítulo-17-novidades-do-php-85)
18. [Sistema de Tipos Avançado](#capítulo-18-sistema-de-tipos-avançado)

### Parte V: Aplicações Práticas
19. [Formulários e Validação](#capítulo-19-formulários-e-validação)
20. [Sessões e Cookies](#capítulo-20-sessões-e-cookies)
21. [Banco de Dados com PDO](#capítulo-21-banco-de-dados-com-pdo)
22. [Segurança](#capítulo-22-segurança)
23. [Projeto Final](#capítulo-23-projeto-final)

---

# Parte I: Fundamentos

## Capítulo 1: Introdução ao PHP

### 1.1 O que é PHP?

PHP (Hypertext Preprocessor) é uma linguagem de programação criada em 1995 por Rasmus Lerdorf. É executada no **servidor** (server-side), gerando HTML que é enviado ao navegador.

**Diferença importante:**
- **JavaScript** roda no navegador (client-side)
- **PHP** roda no servidor (server-side)

```php
<?php
// Este código roda no SERVIDOR
echo "Olá, mundo!";
// O navegador recebe apenas: Olá, mundo!
```

### 1.2 Primeiro Programa

Crie um arquivo chamado `ola.php`:

```php
<?php

// Todo arquivo PHP começa com <?php
// Este é um comentário de linha única

/*
   Este é um comentário
   de múltiplas linhas
*/

// echo exibe texto na tela
echo "Olá, mundo!";

// Cada instrução termina com ponto e vírgula
echo "Bem-vindo ao PHP!";
```

**Executando:**
```bash
php ola.php
```

**Saída:**
```
Olá, mundo!Bem-vindo ao PHP!
```

### 1.3 PHP com HTML

Você pode misturar PHP com HTML:

```php
<!DOCTYPE html>
<html>
<head>
    <title>Minha Página</title>
</head>
<body>
    <h1><?php echo "Olá, mundo!"; ?></h1>
    
    <?php
    $hora = date('H');
    
    if ($hora < 12) {
        echo "<p>Bom dia!</p>";
    } elseif ($hora < 18) {
        echo "<p>Boa tarde!</p>";
    } else {
        echo "<p>Boa noite!</p>";
    }
    ?>
</body>
</html>
```

### 1.4 Strict Types (Importante!)

**Sempre** use `declare(strict_types=1)` no início dos seus arquivos:

```php
<?php

declare(strict_types=1);

// Agora o PHP vai exigir tipos exatos
// Sem conversões automáticas
```

**Por quê?** Previne bugs causados por conversões automáticas de tipo.

### 1.5 Estrutura de um Arquivo PHP

```php
<?php

declare(strict_types=1);

// 1. Comentários e documentação
// 2. Definições de constantes
// 3. Funções ou classes
// 4. Código principal

// Exemplo:
const SITE_NAME = "Meu Site";

function saudacao(): string
{
    return "Bem-vindo ao " . SITE_NAME;
}

echo saudacao();
```

### 📝 Exercícios do Capítulo 1

1. Crie um arquivo que exiba seu nome e idade
2. Misture PHP com HTML para criar uma página com título dinâmico
3. Use `date()` para exibir a data atual

---

## Capítulo 2: Variáveis e Tipos de Dados

### 2.1 O que são Variáveis?

Variáveis são **"caixas"** que guardam informações. Em PHP, começam com `$`:

```php
<?php

declare(strict_types=1);

// Criando variáveis
$nome = "João";
$idade = 25;
$altura = 1.75;
$estudante = true;

// Exibindo variáveis
echo $nome;        // João
echo $idade;       // 25
echo $altura;      // 1.75
echo $estudante;   // 1 (true é exibido como 1)
```

### 2.2 Regras para Nomes de Variáveis

✅ **Permitido:**
```php
<?php
$nome = "João";
$idade = 25;
$nomeCompleto = "João Silva";
$nome_completo = "João Silva";
$_valor = 100;
$valor2 = 200;
```

❌ **NÃO permitido:**
```php
<?php
$2valor = 100;      // Não pode começar com número
$nome-completo = "João";  // Não pode ter hífen
$valor total = 100;  // Não pode ter espaço
```

### 2.3 Tipos de Dados Primitivos

#### 2.3.1 String (Texto)

```php
<?php

declare(strict_types=1);

// Aspas duplas: interpreta variáveis
$nome = "João";
echo "Olá, $nome!";  // Olá, João!

// Aspas simples: texto literal
echo 'Olá, $nome!';  // Olá, $nome!

// Concatenação com ponto (.)
$primeiroNome = "João";
$sobrenome = "Silva";
$nomeCompleto = $primeiroNome . " " . $sobrenome;
echo $nomeCompleto;  // João Silva

// String multilinha
$texto = "Esta é uma string
que ocupa múltiplas
linhas de código";
```

#### 2.3.2 Integer (Números Inteiros)

```php
<?php

declare(strict_types=1);

$idade = 25;
$ano = 2026;
$temperatura = -5;

// Operações matemáticas
$soma = 10 + 5;        // 15
$subtracao = 10 - 5;   // 5
$multiplicacao = 10 * 5;  // 50
$divisao = 10 / 5;     // 2

echo $soma;
```

#### 2.3.3 Float (Números Decimais)

```php
<?php

declare(strict_types=1);

$preco = 19.99;
$altura = 1.75;
$temperatura = -3.5;

// Cálculos com decimais
$total = $preco * 2;
echo $total;  // 39.98

// Formatação
$valor = 1234.56;
echo number_format($valor, 2, ',', '.');  // 1.234,56
```

#### 2.3.4 Boolean (Verdadeiro/Falso)

```php
<?php

declare(strict_types=1);

$ligado = true;
$desligado = false;

$maiorDeIdade = true;
$estudante = false;

// Usado em condições
if ($maiorDeIdade) {
    echo "Pode entrar";
}

// Operações lógicas
$podeBeber = $maiorDeIdade && !$estudante;
```

#### 2.3.5 Null (Ausência de Valor)

```php
<?php

declare(strict_types=1);

$variavel = null;

// Verificando se é null
if ($variavel === null) {
    echo "Variável está vazia";
}

// Valor padrão para null
$nome = null;
$nomeExibir = $nome ?? "Anônimo";
echo $nomeExibir;  // Anônimo
```

### 2.4 Verificando Tipos

```php
<?php

declare(strict_types=1);

$nome = "João";
$idade = 25;
$altura = 1.75;
$ativo = true;
$vazio = null;

// Funções para verificar tipo
var_dump($nome);    // string(4) "João"
var_dump($idade);   // int(25)
var_dump($altura);  // float(1.75)
var_dump($ativo);   // bool(true)
var_dump($vazio);   // NULL

// Verificações específicas
echo is_string($nome);    // 1 (true)
echo is_int($idade);      // 1 (true)
echo is_float($altura);   // 1 (true)
echo is_bool($ativo);     // 1 (true)
echo is_null($vazio);     // 1 (true)
```

### 2.5 Conversão de Tipos (Type Casting)

```php
<?php

declare(strict_types=1);

// String para inteiro
$textoNumero = "123";
$numero = (int) $textoNumero;
echo $numero + 10;  // 133

// Float para inteiro (perde casas decimais)
$decimal = 19.99;
$inteiro = (int) $decimal;
echo $inteiro;  // 19

// Qualquer coisa para string
$idade = 25;
$idadeTexto = (string) $idade;
echo "Idade: " . $idadeTexto;

// Para boolean
$zero = 0;
$vazio = "";
echo (bool) $zero;   // false
echo (bool) $vazio;  // false
echo (bool) "texto"; // true
echo (bool) 42;      // true
```

### 2.6 Constantes

Constantes são valores que **nunca mudam**:

```php
<?php

declare(strict_types=1);

// Definindo constantes
const PI = 3.14159;
const SITE_NAME = "Meu Site";
const MAX_ATTEMPTS = 3;

// Usando constantes
echo PI;  // 3.14159

// Constantes não usam $
// echo $PI;  // ERRO!

// Constantes não podem ser alteradas
// PI = 3.14;  // ERRO!

// Constante global usando define()
define('DB_HOST', 'localhost');
echo DB_HOST;  // localhost
```

### 2.7 Escopo de Variáveis

```php
<?php

declare(strict_types=1);

// Variável global
$global = "Eu sou global";

function testeEscopo(): void
{
    // Variável local
    $local = "Eu sou local";
    echo $local;  // OK
    
    // Para acessar global, precisa declarar
    global $global;
    echo $global;  // OK
}

testeEscopo();

echo $local;  // ERRO! $local não existe aqui
```

### 📝 Exercícios do Capítulo 2

1. Crie variáveis para armazenar nome, idade, altura e peso
2. Calcule o IMC usando: peso / (altura * altura)
3. Use `var_dump()` para verificar o tipo de cada variável
4. Crie constantes para valores que não mudam (ex: velocidade da luz)

---

## Capítulo 3: Operadores

### 3.1 Operadores Aritméticos

```php
<?php

declare(strict_types=1);

$a = 10;
$b = 3;

// Operações básicas
echo $a + $b;   // 13 (soma)
echo $a - $b;   // 7  (subtração)
echo $a * $b;   // 30 (multiplicação)
echo $a / $b;   // 3.333... (divisão)
echo $a % $b;   // 1  (resto da divisão - módulo)
echo $a ** $b;  // 1000 (potência - 10³)

// Precedência de operadores
$resultado = 5 + 3 * 2;
echo $resultado;  // 11 (multiplicação primeiro)

$resultado = (5 + 3) * 2;
echo $resultado;  // 16 (parênteses primeiro)
```

### 3.2 Operadores de Atribuição

```php
<?php

declare(strict_types=1);

// Atribuição simples
$x = 10;

// Atribuição com operação
$x += 5;   // $x = $x + 5  → 15
$x -= 3;   // $x = $x - 3  → 12
$x *= 2;   // $x = $x * 2  → 24
$x /= 4;   // $x = $x / 4  → 6
$x %= 4;   // $x = $x % 4  → 2

// Incremento e decremento
$i = 5;
$i++;  // $i = $i + 1  → 6
$i--;  // $i = $i - 1  → 5

// Pré vs pós incremento
$a = 5;
echo $a++;  // Exibe 5, depois incrementa para 6
echo ++$a;  // Incrementa para 7, depois exibe 7
```

### 3.3 Operadores de Comparação

```php
<?php

declare(strict_types=1);

$a = 10;
$b = "10";
$c = 20;

// Igualdade (compara valor)
echo $a == $b;   // true (10 == "10")

// Identidade (compara valor E tipo)
echo $a === $b;  // false (int !== string)

// Diferença
echo $a != $c;   // true
echo $a !== $b;  // true (tipos diferentes)

// Comparações
echo $a > $c;    // false
echo $a < $c;    // true
echo $a >= 10;   // true
echo $a <= 10;   // true

// Spaceship operator (retorna -1, 0 ou 1)
echo $a <=> $c;  // -1 (menor)
echo $a <=> $b;  // 0 (igual)
echo $c <=> $a;  // 1 (maior)
```

### 3.4 Operadores Lógicos

```php
<?php

declare(strict_types=1);

$idade = 20;
$temCarteira = true;
$possuiCarro = false;

// AND (E) - todas condições devem ser true
$podeDirigir = $idade >= 18 && $temCarteira;
echo $podeDirigir;  // true

// OR (OU) - pelo menos uma condição true
$podeViajar = $possuiCarro || $temCarteira;
echo $podeViajar;  // true

// NOT (NÃO) - inverte o valor
$naoTemCarro = !$possuiCarro;
echo $naoTemCarro;  // true

// Exemplos práticos
$maiorDeIdade = $idade >= 18;
$menorDeIdade = $idade < 18;
$idadeAdulto = $idade >= 18 && $idade < 60;
$semCarteira = !$temCarteira;
```

### 3.5 Operador Ternário

```php
<?php

declare(strict_types=1);

$idade = 20;

// Forma longa com if-else
if ($idade >= 18) {
    $status = "Maior de idade";
} else {
    $status = "Menor de idade";
}

// Forma curta com operador ternário
$status = $idade >= 18 ? "Maior de idade" : "Menor de idade";

// Exemplo prático
$nota = 7.5;
$resultado = $nota >= 7.0 ? "Aprovado" : "Reprovado";
echo $resultado;  // Aprovado

// Ternário aninhado (evite usar muito)
$nota = 8.5;
$conceito = $nota >= 9 ? "A" : ($nota >= 7 ? "B" : "C");
echo $conceito;  // B
```

### 3.6 Null Coalescing Operator

```php
<?php

declare(strict_types=1);

// Operador ?? retorna o primeiro valor não-null
$nome = null;
$nomeExibir = $nome ?? "Anônimo";
echo $nomeExibir;  // Anônimo

// Útil com arrays
$usuario = [];
$email = $usuario['email'] ?? "não informado";

// Encadeamento
$resultado = $a ?? $b ?? $c ?? "padrão";

// Comparação com ternário
// Forma antiga
$nome = isset($usuario['nome']) ? $usuario['nome'] : "Anônimo";

// Forma moderna
$nome = $usuario['nome'] ?? "Anônimo";
```

### 3.7 Operadores de String

```php
<?php

declare(strict_types=1);

// Concatenação com .
$primeiro = "João";
$ultimo = "Silva";
$completo = $primeiro . " " . $ultimo;
echo $completo;  // João Silva

// Concatenação com atribuição
$mensagem = "Olá, ";
$mensagem .= "mundo!";
echo $mensagem;  // Olá, mundo!

// Interpolação em aspas duplas
$nome = "Maria";
echo "Bem-vinda, $nome!";  // Bem-vinda, Maria!

// Com chaves para clareza
$produto = "notebook";
echo "Comprei um {$produto} novo";
```

### 3.8 Operadores de Array

```php
<?php

declare(strict_types=1);

// União de arrays
$a = [1, 2, 3];
$b = [4, 5, 6];
$c = $a + $b;
print_r($c);  // [1, 2, 3]

// Igualdade de arrays
$x = ['a' => 1, 'b' => 2];
$y = ['b' => 2, 'a' => 1];
echo $x == $y;   // true (mesmo conteúdo)
echo $x === $y;  // false (ordem diferente)
```

### 📝 Exercícios do Capítulo 3

1. Crie uma calculadora simples que soma, subtrai, multiplica e divide
2. Verifique se um número é par ou ímpar usando o operador módulo (%)
3. Use operador ternário para classificar uma nota (A, B, C, D, F)
4. Concatene strings para formar uma frase completa

---

## Capítulo 4: Estruturas de Controle

### 4.1 IF - ELSE

```php
<?php

declare(strict_types=1);

// IF simples
$idade = 20;

if ($idade >= 18) {
    echo "Maior de idade";
}

// IF-ELSE
$nota = 6.5;

if ($nota >= 7.0) {
    echo "Aprovado";
} else {
    echo "Reprovado";
}

// IF-ELSEIF-ELSE
$nota = 8.5;

if ($nota >= 9.0) {
    echo "Excelente";
} elseif ($nota >= 7.0) {
    echo "Bom";
} elseif ($nota >= 5.0) {
    echo "Regular";
} else {
    echo "Insuficiente";
}

// Condições múltiplas
$idade = 20;
$temCarteira = true;

if ($idade >= 18 && $temCarteira) {
    echo "Pode dirigir";
}

// IF aninhado
$salario = 5000;
$anosEmpresa = 3;

if ($salario > 4000) {
    if ($anosEmpresa >= 2) {
        echo "Elegível para promoção";
    } else {
        echo "Aguarde mais tempo na empresa";
    }
}
```

### 4.2 MATCH (Novo no PHP 8.0)

```php
<?php

declare(strict_types=1);

// Match é como switch, mas retorna um valor
$dia = 3;

$nomeDia = match ($dia) {
    1 => "Segunda",
    2 => "Terça",
    3 => "Quarta",
    4 => "Quinta",
    5 => "Sexta",
    6 => "Sábado",
    7 => "Domingo",
    default => "Inválido"
};

echo $nomeDia;  // Quarta

// Match com múltiplos valores
$nota = 7;

$conceito = match (true) {
    $nota >= 9 => "A",
    $nota >= 7 => "B",
    $nota >= 5 => "C",
    default => "D"
};

// Match com comparação estrita (===)
$valor = "1";

$resultado = match ($valor) {
    1 => "Número um",      // Não corresponde
    "1" => "String um",    // Corresponde!
};

echo $resultado;  // String um
```

### 4.3 SWITCH

```php
<?php

declare(strict_types=1);

// Switch básico
$diaSemana = 3;

switch ($diaSemana) {
    case 1:
        echo "Segunda-feira";
        break;
    case 2:
        echo "Terça-feira";
        break;
    case 3:
        echo "Quarta-feira";
        break;
    case 4:
        echo "Quinta-feira";
        break;
    case 5:
        echo "Sexta-feira";
        break;
    case 6:
    case 7:
        echo "Final de semana";
        break;
    default:
        echo "Dia inválido";
}

// Switch sem break (fall-through)
$mes = 2;

switch ($mes) {
    case 12:
    case 1:
    case 2:
        echo "Verão";
        break;
    case 3:
    case 4:
    case 5:
        echo "Outono";
        break;
    case 6:
    case 7:
    case 8:
        echo "Inverno";
        break;
    case 9:
    case 10:
    case 11:
        echo "Primavera";
        break;
}
```

### 4.4 FOR Loop

```php
<?php

declare(strict_types=1);

// Loop básico de 1 a 10
for ($i = 1; $i <= 10; $i++) {
    echo $i . " ";
}
// Saída: 1 2 3 4 5 6 7 8 9 10

// Contagem regressiva
for ($i = 10; $i >= 1; $i--) {
    echo $i . " ";
}
// Saída: 10 9 8 7 6 5 4 3 2 1

// Incremento de 2 em 2
for ($i = 0; $i <= 20; $i += 2) {
    echo $i . " ";
}
// Saída: 0 2 4 6 8 10 12 14 16 18 20

// Múltiplas variáveis
for ($i = 0, $j = 10; $i < $j; $i++, $j--) {
    echo "i=$i, j=$j\n";
}

// Loop com array
$numeros = [1, 2, 3, 4, 5];

for ($i = 0; $i < count($numeros); $i++) {
    echo $numeros[$i] . " ";
}
```

### 4.5 WHILE Loop

```php
<?php

declare(strict_types=1);

// While básico
$contador = 1;

while ($contador <= 5) {
    echo $contador . " ";
    $contador++;
}
// Saída: 1 2 3 4 5

// While com condição complexa
$numero = 1;

while ($numero <= 100 && $numero % 7 !== 0) {
    $numero++;
}

echo "Primeiro número divisível por 7: $numero";

// Loop infinito (use com cuidado!)
$tentativas = 0;

while (true) {
    $tentativas++;
    
    if ($tentativas >= 5) {
        break;  // Sai do loop
    }
    
    echo "Tentativa $tentativas\n";
}
```

### 4.6 DO-WHILE Loop

```php
<?php

declare(strict_types=1);

// Do-while executa pelo menos uma vez
$numero = 10;

do {
    echo $numero . " ";
    $numero++;
} while ($numero <= 5);

// Saída: 10 (executa mesmo com condição falsa)

// Exemplo prático: menu
$opcao = 0;

do {
    echo "Menu:\n";
    echo "1 - Ver produtos\n";
    echo "2 - Comprar\n";
    echo "3 - Sair\n";
    
    // Simulando entrada do usuário
    $opcao = 3;
    
} while ($opcao !== 3);
```

### 4.7 FOREACH Loop

```php
<?php

declare(strict_types=1);

// Foreach com array indexado
$frutas = ["Maçã", "Banana", "Laranja"];

foreach ($frutas as $fruta) {
    echo $fruta . "\n";
}

// Foreach com índice e valor
$cores = ["vermelho", "azul", "verde"];

foreach ($cores as $indice => $cor) {
    echo "Posição $indice: $cor\n";
}

// Foreach com array associativo
$pessoa = [
    'nome' => 'João',
    'idade' => 25,
    'cidade' => 'São Paulo'
];

foreach ($pessoa as $chave => $valor) {
    echo "$chave: $valor\n";
}

// Modificando valores (por referência)
$numeros = [1, 2, 3, 4, 5];

foreach ($numeros as &$numero) {
    $numero = $numero * 2;
}

print_r($numeros);  // [2, 4, 6, 8, 10]
```

### 4.8 BREAK e CONTINUE

```php
<?php

declare(strict_types=1);

// BREAK: sai do loop
for ($i = 1; $i <= 10; $i++) {
    if ($i === 5) {
        break;  // Para quando chegar em 5
    }
    echo $i . " ";
}
// Saída: 1 2 3 4

// CONTINUE: pula para próxima iteração
for ($i = 1; $i <= 10; $i++) {
    if ($i % 2 === 0) {
        continue;  // Pula números pares
    }
    echo $i . " ";
}
// Saída: 1 3 5 7 9

// Break com níveis (loops aninhados)
for ($i = 1; $i <= 3; $i++) {
    for ($j = 1; $j <= 3; $j++) {
        if ($i === 2 && $j === 2) {
            break 2;  // Sai de ambos os loops
        }
        echo "($i,$j) ";
    }
}
```

### 4.9 GOTO (Evite Usar!)

```php
<?php

declare(strict_types=1);

// GOTO permite pular para um label
// Mas torna o código difícil de entender
$i = 0;

inicio:
echo $i . " ";
$i++;

if ($i < 5) {
    goto inicio;
}

// Preferível usar loops normais!
```

### 📝 Exercícios do Capítulo 4

1. Use IF-ELSE para verificar se um ano é bissexto
2. Crie um menu com SWITCH que execute diferentes ações
3. Use FOR para calcular o fatorial de um número
4. Use WHILE para encontrar números primos até 50
5. Use FOREACH para somar todos os elementos de um array

---

## Capítulo 5: Arrays

### 5.1 O que são Arrays?

Arrays são estruturas que armazenam **múltiplos valores** em uma única variável.

```php
<?php

declare(strict_types=1);

// Array vazio
$vazio = [];

// Array com valores
$frutas = ["Maçã", "Banana", "Laranja"];

// Acessando elementos (índice começa em 0)
echo $frutas[0];  // Maçã
echo $frutas[1];  // Banana
echo $frutas[2];  // Laranja

// Modificando elementos
$frutas[1] = "Morango";
echo $frutas[1];  // Morango

// Adicionando elementos
$frutas[] = "Uva";  // Adiciona no final
echo $frutas[3];  // Uva
```

### 5.2 Arrays Indexados

```php
<?php

declare(strict_types=1);

// Criação
$numeros = [10, 20, 30, 40, 50];

// Tamanho do array
echo count($numeros);  // 5

// Percorrendo com for
for ($i = 0; $i < count($numeros); $i++) {
    echo $numeros[$i] . " ";
}

// Percorrendo com foreach
foreach ($numeros as $numero) {
    echo $numero . " ";
}

// Array multidimensional (matriz)
$matriz = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

echo $matriz[0][0];  // 1
echo $matriz[1][2];  // 6
echo $matriz[2][1];  // 8
```

### 5.3 Arrays Associativos

```php
<?php

declare(strict_types=1);

// Array com chaves personalizadas
$pessoa = [
    'nome' => 'João Silva',
    'idade' => 25,
    'email' => 'joao@example.com',
    'cidade' => 'São Paulo'
];

// Acessando por chave
echo $pessoa['nome'];   // João Silva
echo $pessoa['idade'];  // 25

// Modificando
$pessoa['idade'] = 26;

// Adicionando novos campos
$pessoa['telefone'] = '(11) 98765-4321';

// Percorrendo
foreach ($pessoa as $chave => $valor) {
    echo "$chave: $valor\n";
}
```

### 5.4 Arrays Mistos

```php
<?php

declare(strict_types=1);

// Array com índices numéricos e associativos
$misto = [
    'nome' => 'Produto X',
    'preco' => 99.90,
    0 => 'tag1',
    1 => 'tag2'
];

// Array aninhado
$usuarios = [
    [
        'id' => 1,
        'nome' => 'João',
        'email' => 'joao@example.com'
    ],
    [
        'id' => 2,
        'nome' => 'Maria',
        'email' => 'maria@example.com'
    ]
];

echo $usuarios[0]['nome'];  // João
echo $usuarios[1]['email']; // maria@example.com
```

### 5.5 Funções de Arrays Essenciais

#### 5.5.1 Adicionando e Removendo Elementos

```php
<?php

declare(strict_types=1);

$frutas = ['Maçã', 'Banana'];

// Adicionar no final
array_push($frutas, 'Laranja');
// ou simplesmente:
$frutas[] = 'Laranja';

// Adicionar no início
array_unshift($frutas, 'Morango');

// Remover do final
$ultima = array_pop($frutas);
echo $ultima;  // Laranja

// Remover do início
$primeira = array_shift($frutas);
echo $primeira;  // Morango

print_r($frutas);  // ['Maçã', 'Banana']
```

#### 5.5.2 Buscando e Verificando

```php
<?php

declare(strict_types=1);

$numeros = [10, 20, 30, 40, 50];

// Verificar se existe
if (in_array(30, $numeros)) {
    echo "30 está no array";
}

// Buscar posição
$posicao = array_search(40, $numeros);
echo $posicao;  // 3

// Verificar se chave existe
$pessoa = ['nome' => 'João', 'idade' => 25];

if (array_key_exists('email', $pessoa)) {
    echo $pessoa['email'];
} else {
    echo "Email não informado";
}

// Ou usando isset
if (isset($pessoa['email'])) {
    echo $pessoa['email'];
}
```

#### 5.5.3 Ordenação

```php
<?php

declare(strict_types=1);

// Ordenação crescente
$numeros = [5, 2, 8, 1, 9];
sort($numeros);
print_r($numeros);  // [1, 2, 5, 8, 9]

// Ordenação decrescente
rsort($numeros);
print_r($numeros);  // [9, 8, 5, 2, 1]

// Ordenar array associativo por valor
$idades = ['João' => 25, 'Maria' => 30, 'Pedro' => 20];
asort($idades);
print_r($idades);  // Pedro => 20, João => 25, Maria => 30

// Ordenar array associativo por chave
ksort($idades);
print_r($idades);  // João => 25, Maria => 30, Pedro => 20
```

#### 5.5.4 Manipulação

```php
<?php

declare(strict_types=1);

// Unir arrays
$a = [1, 2, 3];
$b = [4, 5, 6];
$c = array_merge($a, $b);
print_r($c);  // [1, 2, 3, 4, 5, 6]

// Dividir array
$pedaco = array_slice($c, 2, 3);
print_r($pedaco);  // [3, 4, 5]

// Extrair valores únicos
$numeros = [1, 2, 2, 3, 3, 3, 4];
$unicos = array_unique($numeros);
print_r($unicos);  // [1, 2, 3, 4]

// Inverter array
$original = [1, 2, 3, 4, 5];
$invertido = array_reverse($original);
print_r($invertido);  // [5, 4, 3, 2, 1]

// Contar valores
$cores = ['azul', 'verde', 'azul', 'vermelho', 'azul'];
$contagem = array_count_values($cores);
print_r($contagem);  // ['azul' => 3, 'verde' => 1, 'vermelho' => 1]
```

### 5.6 PHP 8.5: array_first() e array_last()

```php
<?php

declare(strict_types=1);

// NOVO NO PHP 8.5!
$numeros = [10, 20, 30, 40, 50];

// Pegar primeiro elemento
$primeiro = array_first($numeros);
echo $primeiro;  // 10

// Pegar último elemento
$ultimo = array_last($numeros);
echo $ultimo;  // 50

// Retorna null se vazio
$vazio = [];
$resultado = array_first($vazio);
var_dump($resultado);  // NULL

// Substitui código antigo:
// $primeiro = $array[0];  // Pode dar erro se vazio
// $ultimo = end($array);  // Modifica ponteiro interno
```

### 5.7 Funções Avançadas

```php
<?php

declare(strict_types=1);

// array_map: aplica função a cada elemento
$numeros = [1, 2, 3, 4, 5];
$dobrados = array_map(fn($n) => $n * 2, $numeros);
print_r($dobrados);  // [2, 4, 6, 8, 10]

// array_filter: filtra elementos
$numeros = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
$pares = array_filter($numeros, fn($n) => $n % 2 === 0);
print_r($pares);  // [2, 4, 6, 8, 10]

// array_reduce: reduz a um único valor
$numeros = [1, 2, 3, 4, 5];
$soma = array_reduce($numeros, fn($carry, $n) => $carry + $n, 0);
echo $soma;  // 15

// array_column: extrai coluna de array multidimensional
$usuarios = [
    ['id' => 1, 'nome' => 'João'],
    ['id' => 2, 'nome' => 'Maria'],
    ['id' => 3, 'nome' => 'Pedro']
];

$nomes = array_column($usuarios, 'nome');
print_r($nomes);  // ['João', 'Maria', 'Pedro']

// array_chunk: divide em pedaços
$numeros = [1, 2, 3, 4, 5, 6, 7, 8, 9];
$grupos = array_chunk($numeros, 3);
print_r($grupos);  // [[1,2,3], [4,5,6], [7,8,9]]
```

### 📝 Exercícios do Capítulo 5

1. Crie um array com notas de alunos e calcule a média
2. Ordene um array de nomes em ordem alfabética
3. Crie um array associativo representando um produto (nome, preço, estoque)
4. Use array_filter para encontrar números maiores que 50 em um array
5. Crie uma matriz 3x3 e percorra todos os elementos

---

## Capítulo 6: Funções

### 6.1 O que são Funções?

Funções são **blocos de código reutilizáveis** que executam uma tarefa específica.

```php
<?php

declare(strict_types=1);

// Definindo uma função
function saudacao(): void
{
    echo "Olá, mundo!";
}

// Chamando a função
saudacao();  // Olá, mundo!
saudacao();  // Olá, mundo!
```

### 6.2 Funções com Parâmetros

```php
<?php

declare(strict_types=1);

// Função com um parâmetro
function saudar(string $nome): void
{
    echo "Olá, $nome!";
}

saudar("João");   // Olá, João!
saudar("Maria");  // Olá, Maria!

// Função com múltiplos parâmetros
function somar(int $a, int $b): int
{
    return $a + $b;
}

$resultado = somar(5, 3);
echo $resultado;  // 8

// Parâmetros com valores padrão
function criarMensagem(string $texto, string $tipo = "info"): string
{
    return "[$tipo] $texto";
}

echo criarMensagem("Bem-vindo");  // [info] Bem-vindo
echo criarMensagem("Erro!", "error");  // [error] Erro!
```

### 6.3 Tipos de Retorno

```php
<?php

declare(strict_types=1);

// Retorna string
function getNome(): string
{
    return "João Silva";
}

// Retorna inteiro
function getIdade(): int
{
    return 25;
}

// Retorna float
function calcularMedia(array $notas): float
{
    $soma = array_sum($notas);
    return $soma / count($notas);
}

// Retorna boolean
function isAdulto(int $idade): bool
{
    return $idade >= 18;
}

// Retorna array
function getDados(): array
{
    return ['nome' => 'João', 'idade' => 25];
}

// Não retorna nada (void)
function exibirMensagem(string $msg): void
{
    echo $msg;
}

// Pode retornar null
function buscarUsuario(int $id): ?array
{
    if ($id === 1) {
        return ['id' => 1, 'nome' => 'João'];
    }
    
    return null;
}
```

### 6.4 Escopo de Variáveis

```php
<?php

declare(strict_types=1);

// Variável global
$global = "Eu sou global";

function testeEscopo(): void
{
    // Variável local
    $local = "Eu sou local";
    
    // Para acessar variável global
    global $global;
    echo $global;  // Eu sou global
    
    // Ou usando $GLOBALS
    echo $GLOBALS['global'];
}

testeEscopo();

// echo $local;  // ERRO! Não existe aqui

// Variável estática (mantém valor entre chamadas)
function contador(): void
{
    static $count = 0;
    $count++;
    echo "Chamada número: $count\n";
}

contador();  // Chamada número: 1
contador();  // Chamada número: 2
contador();  // Chamada número: 3
```

### 6.5 Passagem por Referência

```php
<?php

declare(strict_types=1);

// Por valor (padrão) - copia o valor
function incrementarPorValor(int $numero): void
{
    $numero++;
    echo "Dentro: $numero\n";
}

$x = 5;
incrementarPorValor($x);
echo "Fora: $x\n";  // 5 (não mudou)

// Por referência - modifica o original
function incrementarPorReferencia(int &$numero): void
{
    $numero++;
    echo "Dentro: $numero\n";
}

$y = 5;
incrementarPorReferencia($y);
echo "Fora: $y\n";  // 6 (mudou!)

// Útil para modificar arrays
function adicionarElemento(array &$arr, mixed $valor): void
{
    $arr[] = $valor;
}

$lista = [1, 2, 3];
adicionarElemento($lista, 4);
print_r($lista);  // [1, 2, 3, 4]
```

### 6.6 Funções Variádicas

```php
<?php

declare(strict_types=1);

// Aceita número variável de argumentos
function somar(...$numeros): int
{
    $total = 0;
    
    foreach ($numeros as $numero) {
        $total += $numero;
    }
    
    return $total;
}

echo somar(1, 2, 3);        // 6
echo somar(1, 2, 3, 4, 5);  // 15
echo somar(10);             // 10

// Combinando parâmetros normais com variádicos
function criarLista(string $titulo, string ...$itens): string
{
    $html = "<h3>$titulo</h3><ul>";
    
    foreach ($itens as $item) {
        $html .= "<li>$item</li>";
    }
    
    $html .= "</ul>";
    
    return $html;
}

echo criarLista("Frutas", "Maçã", "Banana", "Laranja");
```

### 6.7 Funções Anônimas (Closures)

```php
<?php

declare(strict_types=1);

// Função anônima atribuída a variável
$saudacao = function(string $nome): string {
    return "Olá, $nome!";
};

echo $saudacao("João");  // Olá, João!

// Closure que captura variável externa
$taxa = 0.1;

$calcularComTaxa = function(float $valor) use ($taxa): float {
    return $valor + ($valor * $taxa);
};

echo $calcularComTaxa(100);  // 110

// Arrow function (PHP 7.4+)
$dobrar = fn(int $n) => $n * 2;
echo $dobrar(5);  // 10

// Útil com array_map, array_filter, etc.
$numeros = [1, 2, 3, 4, 5];

$dobrados = array_map(fn($n) => $n * 2, $numeros);
$pares = array_filter($numeros, fn($n) => $n % 2 === 0);
```

### 6.8 Recursão

```php
<?php

declare(strict_types=1);

// Função que chama a si mesma
function fatorial(int $n): int
{
    // Caso base
    if ($n <= 1) {
        return 1;
    }
    
    // Caso recursivo
    return $n * fatorial($n - 1);
}

echo fatorial(5);  // 120 (5 * 4 * 3 * 2 * 1)

// Fibonacci recursivo
function fibonacci(int $n): int
{
    if ($n <= 1) {
        return $n;
    }
    
    return fibonacci($n - 1) + fibonacci($n - 2);
}

echo fibonacci(10);  // 55

// Contagem regressiva
function contagemRegressiva(int $n): void
{
    if ($n <= 0) {
        echo "Fim!\n";
        return;
    }
    
    echo "$n\n";
    contagemRegressiva($n - 1);
}

contagemRegressiva(5);
// 5
// 4
// 3
// 2
// 1
// Fim!
```

### 6.9 Documentação de Funções (PHPDoc)

```php
<?php

declare(strict_types=1);

/**
 * Calcula a área de um círculo
 * 
 * Esta função recebe o raio e retorna a área calculada
 * usando a fórmula: π * r²
 * 
 * @param float $raio O raio do círculo em metros
 * @return float A área do círculo em metros quadrados
 */
function calcularAreaCirculo(float $raio): float
{
    return pi() * ($raio ** 2);
}

/**
 * Busca um usuário pelo ID
 * 
 * @param int $id ID do usuário
 * @return array|null Dados do usuário ou null se não encontrado
 */
function buscarUsuario(int $id): ?array
{
    $usuarios = [
        1 => ['nome' => 'João', 'email' => 'joao@example.com'],
        2 => ['nome' => 'Maria', 'email' => 'maria@example.com']
    ];
    
    return $usuarios[$id] ?? null;
}
```

### 📝 Exercícios do Capítulo 6

1. Crie uma função que calcule o IMC (peso / altura²)
2. Faça uma função que receba um array e retorne o maior valor
3. Crie uma função recursiva para calcular a potência (x^n)
4. Use funções anônimas com array_map para converter temperaturas Celsius para Fahrenheit
5. Crie uma função variádica que encontre o maior número entre vários argumentos

---

**Continua na Parte II...**

(Este foi apenas o início! O livro continua com manipulação de strings, datas, arquivos, OOP, recursos modernos do PHP 8.5, e muito mais!)

---

## Resumo da Parte I

✅ **Capítulo 1:** Introdução ao PHP e primeiro programa  
✅ **Capítulo 2:** Variáveis, tipos de dados e constantes  
✅ **Capítulo 3:** Operadores aritméticos, lógicos e de comparação  
✅ **Capítulo 4:** Estruturas de controle (if, match, loops)  
✅ **Capítulo 5:** Arrays indexados, associativos e funções de array  
✅ **Capítulo 6:** Funções, parâmetros, retornos e recursão  

**Próximos capítulos:** Strings, arquivos, OOP, PHP 8.5 features, projetos práticos!

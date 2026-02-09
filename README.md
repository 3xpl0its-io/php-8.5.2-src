# 📚 PHP 8.5 Moderno: Do Zero à Maestria

<div align="center">

![PHP Version](https://img.shields.io/badge/PHP-8.5-777BB4?style=for-the-badge&logo=php&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completo-success?style=for-the-badge)
![Language](https://img.shields.io/badge/Idioma-Português%20BR-blue?style=for-the-badge)

**O guia mais completo e atualizado de PHP 8.5 em português brasileiro**

[📖 Começar a Ler](#-estrutura-do-curso) • [🎯 Características](#-características) • [🚀 Como Usar](#-como-usar) • [💡 Contribuir](#-contribuindo)

</div>

---

## 🌟 Sobre o Projeto

Este é um **curso completo de PHP 8.5** estruturado como um livro didático progressivo, levando você desde os fundamentos absolutos da programação até técnicas avançadas de desenvolvimento profissional.

Diferente de outros cursos, este material:

✅ **Foca em PHP puro** - sem frameworks ou bibliotecas de terceiros  
✅ **Prioriza PHP 8.5** - utilizando os recursos mais modernos da linguagem  
✅ **Ensina do zero** - não assume conhecimento prévio de programação  
✅ **É 100% prático** - todos os exemplos são executáveis e testados  
✅ **Está em português** - comentários, explicações e narrativa em PT-BR  
✅ **Segue padrões profissionais** - código em inglês seguindo PSR-12  

---

## 🎯 Características

### 📝 34 Capítulos Detalhados

- **Parte I:** Fundamentos (6 capítulos)
- **Parte II:** Manipulação de Dados (4 capítulos)
- **Parte III:** Orientação a Objetos Completa (5 capítulos)
- **Parte IV:** Recursos Modernos PHP 8.5 (3 capítulos)
- **Parte V:** Projetos Práticos (5 capítulos)
- **Complementos:** Tópicos Avançados (11 capítulos)

### 💻 Conteúdo Técnico

```php
<?php

declare(strict_types=1);

// Código moderno com:
// ✅ Strict types obrigatório
// ✅ Type hints completos
// ✅ Recursos do PHP 8.5 (pipe operator, array_first/last, clone with)
// ✅ Comentários explicativos em português
// ✅ Padrões PSR-12
```

### 🛡️ Segurança Desde o Início

Todos os exemplos ensinam segurança desde o primeiro capítulo:
- Prepared statements para SQL
- Sanitização e validação de inputs
- Proteção contra XSS, CSRF e SQL Injection
- Hashing seguro de senhas com Argon2ID
- Sessões seguras

### 🎓 Metodologia Pedagógica

- **Progressão gradual** de complexidade
- **Exercícios práticos** ao final de cada capítulo
- **Exemplos do mundo real** em todos os tópicos
- **Explicações detalhadas** do "porquê", não apenas "como"
- **Comparações** entre abordagens antigas vs modernas

---

## 📚 Estrutura do Curso

### 📂 Parte I: Fundamentos
**Arquivo:** `php-8.5-do-zero-a-maestria.md`

| Capítulo | Tópico | Conceitos |
|----------|--------|-----------|
| 1 | Introdução ao PHP | Execução server-side, primeiro programa, strict types |
| 2 | Variáveis e Tipos | String, int, float, bool, null, constantes |
| 3 | Operadores | Aritméticos, lógicos, comparação, ternário, null coalescing |
| 4 | Estruturas de Controle | if/else, match, for, while, foreach, break/continue |
| 5 | Arrays | Indexados, associativos, funções nativas, array_first/last |
| 6 | Funções | Parâmetros, retornos, closures, arrow functions, recursão |

### 📂 Parte II: Manipulação de Dados
**Arquivo:** `php-8.5-partes-2-3-4-5.md`

| Capítulo | Tópico | Conceitos |
|----------|--------|-----------|
| 7 | Strings | Manipulação, regex, multibyte (UTF-8) |
| 8 | Datas e Horas | DateTime, cálculos, fusos horários, DateTimeImmutable |
| 9 | Arquivos e Diretórios | Leitura, escrita, upload, CSV |
| 10 | JSON e Serialização | encode/decode, API REST básica |

### 📂 Parte III: Orientação a Objetos
**Arquivo:** `php-8.5-final-partes-3-4-5.md`

| Capítulo | Tópico | Conceitos |
|----------|--------|-----------|
| 11 | Classes e Objetos | Básico OOP, construtor, métodos, propriedades |
| 12 | Encapsulamento | Getters/setters, readonly, property hooks |
| 13 | Herança e Polimorfismo | extends, override, abstract, final |
| 14 | Interfaces e Traits | Contratos, composição, resolução de conflitos |
| 15 | OOP Avançado | Enums, métodos mágicos, namespaces, autoloading |

### 📂 Parte IV: PHP 8.5 Moderno
**Arquivo:** `php-8.5-final-partes-3-4-5.md`

| Capítulo | Tópico | Conceitos |
|----------|--------|-----------|
| 16 | Recursos PHP 8.0-8.4 | Named args, match, nullsafe, enums, property hooks |
| 17 | **Novidades PHP 8.5** | **Pipe operator, clone with, #[\NoDiscard], URI** |
| 18 | Sistema de Tipos | Union, intersection, mixed, never |

### 📂 Parte V: Projetos Práticos
**Arquivo:** `php-8.5-final-partes-3-4-5.md`

| Capítulo | Tópico | Conceitos |
|----------|--------|-----------|
| 19 | Formulários e Validação | Validadores, CSRF, sanitização |
| 20 | Sessões e Cookies | Autenticação, Auth class, segurança |
| 21 | Banco de Dados PDO | CRUD completo, transações, paginação |
| 22 | Segurança | SQL injection, XSS, senhas, rate limiting |
| 23 | Projeto Final | Sistema completo MVC |

### 📂 Tópicos Avançados Complementares

**Arquivos:** `php-8.5-topicos-avancados-parte-[1-3].md`

| Capítulo | Tópico | Arquivo |
|----------|--------|---------|
| 24 | Iterables e Iteradores | Parte 1 |
| 25 | Namespaces Avançado | Parte 1 |
| 26 | Static Properties/Methods | Parte 1 |
| 27 | Traits Avançado | Parte 1 |
| 28 | Interfaces Avançado | Parte 1 |
| 29 | Abstract Classes Avançado | Parte 2 |
| 30 | Class Constants | Parte 2 |
| 31 | Access Modifiers Detalhado | Parte 2 |
| 32 | Regular Expressions Completo | Parte 2 |
| 33 | Validação de E-mail e URL | Parte 3 |
| 34 | cURL - Requisições HTTP | Parte 3 |

---

## 🚀 Como Usar

### 📖 Leitura Sequencial (Recomendado para Iniciantes)

1. Comece pelo **Capítulo 1** do primeiro arquivo
2. Execute todos os exemplos de código
3. Faça os exercícios ao final de cada capítulo
4. Avance apenas quando dominar o conteúdo atual

### 🎯 Consulta Específica (Para Desenvolvedores)

Use a estrutura acima para ir direto ao tópico desejado:

```bash
# Exemplo: aprender sobre Enums
→ Arquivo: php-8.5-final-partes-3-4-5.md
→ Buscar: "Capítulo 16" ou "8.1: Enumerations"

# Exemplo: aprender Regex
→ Arquivo: php-8.5-topicos-avancados-parte-2.md
→ Buscar: "Capítulo 32"
```

### 💡 Prática Recomendada

1. **Crie um ambiente de testes:**
   ```bash
   # Verifique sua versão do PHP
   php -v  # Deve ser 8.5 ou superior
   
   # Crie diretório de prática
   mkdir php-pratica
   cd php-pratica
   ```

2. **Execute os exemplos:**
   ```bash
   # Crie arquivo de teste
   nano teste.php
   
   # Cole código do livro
   # Execute
   php teste.php
   ```

3. **Modifique e experimente:**
   - Altere valores
   - Adicione validações
   - Combine conceitos diferentes

---

## 🎓 Para Quem é Este Curso?

### ✅ Perfeito para:

- **Iniciantes absolutos** em programação
- Desenvolvedores que querem **migrar para PHP**
- Programadores PHP que querem **atualizar para 8.5**
- Estudantes de **Ciência da Computação**
- Profissionais que precisam de **referência completa**

### ❌ Não é ideal para:

- Quem busca tutoriais de **Laravel/Symfony** (este curso é PHP puro)
- Quem precisa de conteúdo em **inglês** (todo material é PT-BR)
- Quem quer apenas "copiar e colar" código (ensina conceitos, não receitas)

---

## 🏆 Diferenciais Deste Material

### 🆕 PHP 8.5 Prioritário

Este é um dos **primeiros cursos em português** a focar no PHP 8.5:

```php
// Pipe Operator (PHP 8.5)
$resultado = $valor
    |> trim(...)
    |> strtolower(...)
    |> ucfirst(...);

// array_first / array_last (PHP 8.5)
$primeiro = array_first($array);
$ultimo = array_last($array);

// Clone with (PHP 8.5)
$atualizado = clone($usuario, ['idade' => 26]);
```

### 📋 Padrões Profissionais

Todo código segue **PSR-12** e boas práticas:

```php
<?php

declare(strict_types=1);  // ✅ Sempre presente

namespace App\Service;     // ✅ Namespaces organizados

class UsuarioService       // ✅ PascalCase para classes
{
    // ✅ Type hints em tudo
    public function __construct(
        private readonly UsuarioRepository $repository
    ) {}
    
    // ✅ Retorno tipado
    public function buscar(int $id): ?Usuario
    {
        return $this->repository->find($id);
    }
}
```

### 🛡️ Segurança em Primeiro Lugar

Nunca ensinamos código inseguro:

```php
// ❌ Você NUNCA verá isto no curso:
$sql = "SELECT * FROM users WHERE email = '$email'";

// ✅ Sempre ensinamos o jeito correto:
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $email]);
```

---

## 📊 Estatísticas do Curso

| Métrica | Valor |
|---------|-------|
| **Capítulos** | 34 |
| **Linhas de Código** | ~5.000+ |
| **Exemplos Práticos** | 200+ |
| **Exercícios** | 100+ |
| **Páginas (estimado)** | ~500 |
| **Horas de Leitura** | 40-60h |
| **Nível** | Iniciante → Avançado |

---

## 🛠️ Tecnologias e Conceitos Cobertos

### Linguagem
- PHP 8.5 (recursos mais recentes)
- PHP 8.0-8.4 (compatibilidade)
- Strict Types
- Type System completo

### OOP
- Classes, Herança, Polimorfismo
- Interfaces, Traits, Abstract Classes
- Enums, Property Hooks
- Design Patterns (Factory, Singleton, Repository, Strategy)

### Dados
- Arrays, Strings, Datas
- JSON, Serialização
- Arquivos, CSV
- PDO, Prepared Statements

### Segurança
- SQL Injection Prevention
- XSS Protection
- CSRF Tokens
- Password Hashing (Argon2ID)
- Input Validation/Sanitization

### Avançado
- Regular Expressions
- Generators e Iterators
- cURL e HTTP Requests
- Namespaces e Autoloading PSR-4
- Static Analysis (conceitos)

---

## 💡 Contribuindo

Contribuições são bem-vindas! Veja como você pode ajudar:

### 🐛 Reportar Erros

Encontrou um erro? Abra uma [Issue](../../issues) com:
- Capítulo e seção
- Descrição do erro
- Correção sugerida (se tiver)

### ✨ Sugerir Melhorias

Tem ideias para melhorar o curso? Abra uma [Issue](../../issues) com:
- Tópico sugerido
- Por que seria útil
- Exemplo de conteúdo (opcional)

### 📝 Contribuir com Código

1. Fork o projeto
2. Crie uma branch (`git checkout -b melhoria/minha-contribuicao`)
3. Commit suas mudanças (`git commit -m 'Adiciona novo exemplo de X'`)
4. Push para a branch (`git push origin melhoria/minha-contribuicao`)
5. Abra um Pull Request

---

## 📜 Licença

Este projeto está sob a licença **MIT**. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

Isso significa que você pode:
- ✅ Usar comercialmente
- ✅ Modificar
- ✅ Distribuir
- ✅ Uso privado

Desde que mantenha o aviso de copyright e licença.

---

## 🙏 Agradecimentos

Este curso foi criado com dedicação para a comunidade brasileira de desenvolvedores PHP.

Agradecimentos especiais:
- **PHP-FIG** - pelos padrões PSR
- **Comunidade PHP Brasil** - pelo suporte e feedback
- **Você** - por escolher aprender PHP de forma profissional

---

## 📞 Contato e Suporte

- 📧 **Issues:** [GitHub Issues](../../issues)
- 💬 **Discussões:** [GitHub Discussions](../../discussions)
- ⭐ **Star o projeto** se este material foi útil!

---

## 🗺️ Roadmap

### ✅ Concluído
- [x] Fundamentos completos
- [x] OOP completo
- [x] PHP 8.5 features
- [x] Projetos práticos
- [x] Tópicos avançados

### 🚧 Futuro
- [ ] Vídeos explicativos
- [ ] Projetos adicionais
- [ ] Seção de performance
- [ ] Docker para desenvolvimento
- [ ] Testes com PHPUnit

---

<div align="center">

### ⭐ Se este material foi útil, considere dar uma estrela!

**Feito com ❤️ para a comunidade brasileira de PHP**

[⬆️ Voltar ao topo](#-php-85-moderno-do-zero-à-maestria)

</div>

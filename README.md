# PostgreSQL: desenvolva com PL/pgSQL

Implementando Stored Procedures; utilizando cursores para buscar dados; controlando o fluxo da procedure; tratando os erros corretamente; aplicando e praticando programação com Stored Procedures.

1. [Criando funções](#1-criando-funções)
2. [Tipos e funções](#2-tipos-e-funções)
3. [Linguagem Procedural](#3-linguagem-procedural)
4. [Estruturas de Controle](#4-estruturas-de-controle)
5. [Estruturas de repetição](#5-estruturas-de-repetição)
6. [Mão na massa](#6-mão-na-massa)

Saiba mais sobre o curso [aqui](https://cursos.alura.com.br/course/postgresql-procedures) ou acompanhe minhas anotações abaixo. ⬇️

## 1. Criando funções

Para programar com banco de dados, utilizam-se funções. As funções são uma extensão da linguagem SQL para complementar as consultas. A linguagem SQL não é uma linguagem de programação, pois não possui estruturas de decisão, repetição, variáveis, etc.

### **Primeira função**

O comando `CREATE FUNCTION` é utilizado para criar uma função que, em sua forma mais simples, não traz parâmetros e precisa retornar algo. Além disso, é preciso dizer a linguagem usada para criar a função.

> Por padrão, o nome da coluna retornada em uma função que nos entrega um único valor é o mesmo nome da função. Nós podemos facilmente mudar o nome dessa coluna utilizando o AS, que cria um alias.

```sql
-- Criando a função que retorna um inteiro usando a linguagem sql
CREATE FUNCTION primeira_funcao() RETURNS INTEGER AS '
	SELECT (5 - 3) * 2
' LANGUAGE SQL;

-- Chama a função e retorna o inteiro com uma coluna de mesmo nome
SELECT primeira_funcao();

-- Chama a função e retorna o inteiro com uma coluna renomeada
SELECT primeira_funcao() AS numero;

-- É possível chamar a função como uma tabela também
SELECT * FROM primeira_funcao();
```

### **Recebendo parâmetros**

Quando utiliza-se parâmetros na função, o nome é opcional, mas é necessário dizer o tipo de dado que será usado. O nome é opcional, pois pode-se utilizar a posição do parâmetro na função criada.

***Propósito do parâmetro no função:***

> Através de parâmetros nós podemos receber informações de fora da função (como o ID de um registro, por exemplo) e manipular estes dados dentro da função

```sql
-- Cria a função com dois parâmetros e seu tipo para utilizá-los inputando dados
CREATE FUNCTION soma_dois_numeros(numero_1 INTEGER, numero_2 INTEGER)
RETURNS INTEGER AS '
	SELECT numero_1 + numero_2
' LANGUAGE SQL;

SELECT soma_dois_numeros(2, 2);

-- Uma das formas de reutilizar uma função é excluir e recriar ela
DROP FUNCTION soma_dois_numeros;

-- Cria a função com dois parâmetros só com o tipo de dado e utilizando sua posição
CREATE FUNCTION soma_dois_numeros(INTEGER, INTEGER)
RETURNS INTEGER AS '
	SELECT $1 + $2
' LANGUAGE SQL;

SELECT soma_dois_numeros(3, 17);
```

### **Detalhes sobre funções**

O que define o valor que será retornado em uma função?

    Após realizar todas as instruções, a última query tem seu resultado obtido pelo PostgreSQL
    e, desse resultado, a primeira linha é retornada para quem chamou a função.

- O último comando de uma função precisa informar o valor, precisa trazer o valor que se quer retornar.
- O que vai ser utilizado como retorno é a primeira linha desse último comando. Sempre o primeiro item.

```sql
CREATE TABLE a (nome VARCHAR(255) NOT NULL);

CREATE FUNCTION cria_a(nome VARCHAR) RETURNS VARCHAR AS '
	INSERT INTO a (nome) VALUES (cria_a.nome);
	SELECT nome;
' LANGUAGE SQL;
```

***Restrição ao substituir uma função:***

> Nós não podemos alterar (REPLACE) uma função informando tipos diferentes em seus parâmetros ou retornos. Para atingir este objetivo precisamos excluir e criar esta função.

```sql
-- Alterando ou substituindo uma função, sem precisar excluir
CREATE OR REPLACE FUNCTION cria_a(nome VARCHAR) RETURNS VARCHAR AS '
	INSERT INTO a (nome) VALUES (cria_a.nome);
	SELECT nome;
' LANGUAGE SQL;

SELECT cria_a('Vinicius Dias');
```

Para criar funções que não necessitam de um retorno, basta substituir o comando após o `RETURNS` para `void`. Isso faz com que, ao chamar a função, ela retorne um valor nulo.

Importante saber que o comando void não permite replace, então é preciso excluir e recriar a função.

```sql
DROP FUNCTION cria_a;

CREATE FUNCTION cria_a(nome VARCHAR) RETURNS void AS '
	INSERT INTO a (nome) VALUES (cria_a.nome);
' LANGUAGE SQL;

SELECT cria_a('Vinicius Dias');
```

Quando se quer definir um valor no insert como varchar, por exemplo, as aspas simples da função conflitam com as aspas simples do valor. Para resolver isso, existe outra forma de definir a criação da função. Utiliza-se `$$`.

```sql
DROP FUNCTION cria_a;

CREATE FUNCTION cria_a(nome VARCHAR) RETURNS void AS $$
	INSERT INTO a (nome) VALUES ('Patrícia');
$$ LANGUAGE SQL;
```

Para saber mais: [Procedure](Para%20saber%20mais/Aula%201%20-%20Atividade%2012%20Para%20saber%20mais_%20Procedure.pdf)
Para saber mais: [Returning](Para%20saber%20mais/Aula%201%20-%20Atividade%2013%20Para%20saber%20mais_%20Returning.pdf)

## 2. Tipos e funções

### **Parâmetros compostos**

Ao utilizar uma tabela como parâmetro, define-se o campo que irá receber a função dentro da própria função.

```sql
CREATE TABLE instrutor (
	id SERIAL PRIMARY KEY,
	nome VARCHAR(255) NOT NULL,
	salario DECIMAL(10, 2)
);

INSERT INTO instrutor (nome, salario) VALUES ('Vinicius Dias', 100);

CREATE FUNCTION dobro_do_salario(instrutor) RETURNS DECIMAL AS $$
	SELECT $1.salario * 2 AS dobro;
$$ LANGUAGE SQL;

SELECT nome, dobro_do_salario(instrutor.*) FROM instrutor;
```

### **Retorno composto**

O retorno composto utiliza os campos da tabela para trazer a primeira linha como um valor ou como um conjunto de valores.

***Como um valor:***

> Usamos como um único valor quando o retorno da função for um tipo simples (após o SELECT).

```sql
CREATE OR REPLACE FUNCTION cria_instrutor_falso() RETURNS instrutor AS $$
	SELECT 22 AS id, 'Nome falso' AS nome, 200.00 AS salario;
$$ LANGUAGE SQL;

SELECT cria_instrutor_falso();
```

***Como um conjunto de valores:***

> Usamos como um conjunto de valores quando o retorno da função for um tipo composto (após o FROM).

```sql
CREATE OR REPLACE FUNCTION cria_instrutor_falso() RETURNS instrutor AS $$
	SELECT 22, 'Nome falso', 200::DECIMAL;
$$ LANGUAGE SQL;

SELECT * FROM cria_instrutor_falso();
```

- Ao criar a tabela, foi definido que o salário seria um valor decimal. Então, ao definir o salário na função é necessário que seja nesse formato. Acima estão duas formas de trazer tal notação.
- Como a função está retornando a tabela instrutor como tipo, não é necessário colocar o nome dos campos na função, porém fica mais legível se tiverem descritos.

### **Retornando conjuntos**

```sql
-- Inserindo mais dados na tabela
INSERT INTO instrutor (nome, salario) VALUES ('Diogo Mascarenhas', 200);
INSERT INTO instrutor (nome, salario) VALUES ('Nico Steppat', 300);
INSERT INTO instrutor (nome, salario) VALUES ('Juliana Oliveira', 400);
INSERT INTO instrutor (nome, salario) VALUES ('Príscila Saraiva', 500);

CREATE FUNCTION instrutores_bem_pagos(valor_salario DECIMAL)
RETURNS SETOF instrutor AS $$
	SELECT * FROM instrutor WHERE salario > valor_salario;
$$ LANGUAGE SQL;

SELECT * FROM instrutores_bem_pagos(300);
```

> Nós podemos definir a estrutura de uma tabela especificamente para o retorno de uma função. Isso é utilizado quando não queremos criar um tipo (ou tabela) específico para um retorno.

```sql
DROP FUNCTION instrutores_bem_pagos;

CREATE FUNCTION instrutores_bem_pagos(valor_salario DECIMAL)
RETURNS TABLE (id INTEGER, nome VARCHAR, salario DECIMAL) AS $$
	SELECT * FROM instrutor WHERE salario > valor_salario;
$$ LANGUAGE SQL;

SELECT * FROM instrutores_bem_pagos(300);
```

- O record retorna uma linha genérica para o registro. Com ele é necessário definir as colunas que se quer retornar.

```sql
DROP FUNCTION instrutores_bem_pagos;

CREATE FUNCTION instrutores_bem_pagos(valor_salario DECIMAL)
RETURNS SETOF record AS $$
	SELECT * FROM instrutor WHERE salario > valor_salario;
$$ LANGUAGE SQL;

SELECT * FROM instrutores_bem_pagos(300);
```

### **Parâmetros de saída**

Pode-se definir parâmetros de saída para retornar valores compostos. Além de definir os parâmetros de entrada no `IN`, o comando `OUT` funciona semelhante a criaçao de tipo para retornar valores compostos nos registros.

```sql
-- Utilizando parâmetros de saída para retornar valores compostos
CREATE FUNCTION soma_e_produto (
	IN numero_1 INTEGER,
	IN numero_2 INTEGER,
	OUT soma INTEGER,
	OUT produto INTEGER
) AS $$
	SELECT numero_1 + numero_2 AS soma, numero_1 * numero_2 AS produto;
$$ LANGUAGE SQL;

SELECT * FROM soma_e_produto(3, 3);

-- Criando tipo para retornar valores compostos
CREATE TYPE dois_valores AS (soma INTEGER, produto INTEGER);

DROP FUNCTION soma_e_produto;

CREATE FUNCTION soma_e_produto (IN numero_1 INTEGER, IN numero_2 INTEGER)
RETURNS dois_valores AS $$
	SELECT numero_1 + numero_2 AS soma, numero_1 * numero_2 AS produto;
$$ LANGUAGE SQL;

SELECT * FROM soma_e_produto(3, 3);
```

Então, para utilizar o `record` de maneira funcional, definindo as colunas que retornarão os registros, pode-se utilizar os parâmetros de saída em conjunto com ele.

```sql
DROP FUNCTION instrutores_bem_pagos;

CREATE FUNCTION instrutores_bem_pagos(
    valor_salario DECIMAL,
    OUT nome VARCHAR,
    OUT salario DECIMAL
)
RETURNS SETOF record AS $$
	SELECT nome, salario FROM instrutor WHERE salario > valor_salario;
$$ LANGUAGE SQL;

SELECT * FROM instrutores_bem_pagos(300);
```

## 3. Linguagem procedural

> A PLPGSQL ou PL/pgSQL é uma linguagem estrutural estendida da SQL que tem por objetivo auxiliar as tarefas de programação no PostgreSQL. Ela incorpora à SQL características procedurais, como os benefícios e facilidades de controle de fluxo de programas que as melhores linguagens possuem. - [Wikipédia](https://pt.wikipedia.org/wiki/PLPGSQL)

### **Estruturas de PLpgSQL**

A estrutura PLpgSQL difere da estrutura SQL porque não, necessariamente, o último comando é executado. Na verdade, a PL precisa do `RETURN` para realmente retornar o que foi definido. Precisa também, das cláusulas `BEGIN` e `END` delimitando o corpo da função.

```sql
CREATE OR REPLACE FUNCTION primeira_pl() RETURNS INTEGER AS $$
	BEGIN
        -- Vários comandos em SQL
		RETURN 1;
	END
$$ LANGUAGE plpgsql;

SELECT primeira_pl();
```

***Diferenças básicas entre SQL e PLpgSQL:***

- PLpgSQL informa de forma explícita o que vai retornar. Funções em SQL retornam o resultado da última query.
- PLpgSQL precisa ter blocos definidos, enquanto funções em SQL não.

### **Declarações de variáveis**

O símbolo `:=` atribui um valor para uma variável. Difere do igual porque serve apenas para atribuição, enquanto o `=` pode ser utilizado para comparação. O `DEFAULT` funciona da mesma maneira, mas pode ser utilizado apenas para declarar a variável, dentro do corpo da função não funciona.

Para saber mais: [Default](Para%20saber%20mais/Aula%203%20-%20Atividade%206%20Para%20saber%20mais_%20Default.pdf)

```sql
CREATE OR REPLACE FUNCTION primeira_pl() RETURNS INTEGER AS $$
	DECLARE
		primeira_variavel INTEGER DEFAULT 3;
	BEGIN
		primeira_variavel := primeira_variavel * 2;
		RETURN primeira_variavel;
	END
$$ LANGUAGE plpgsql;

SELECT primeira_pl();
```

### **Blocos**

Caso queira declarar uma variável dentro do bloco já existente, o PL entende como se fosse uma nova variável e apenas armazena ela.

```sql
CREATE OR REPLACE FUNCTION primeira_pl() RETURNS INTEGER AS $$
	DECLARE
		primeira_variavel INTEGER DEFAULT 3;
	BEGIN
		primeira_variavel := primeira_variavel * 2;
		
		DECLARE
			primeira_variavel INTEGER;
		BEGIN
			primeira_variavel := 7;
		END;
		
		RETURN primeira_variavel;
	END
$$ LANGUAGE plpgsql;

SELECT primeira_pl();
```

Para que essa variável se torne acessível dentro do bloco pai, basta não declará-la, apenas iniciar e finalizar já retorna seu valor.

```sql
CREATE OR REPLACE FUNCTION primeira_pl() RETURNS INTEGER AS $$
	DECLARE
		primeira_variavel INTEGER DEFAULT 3;
	BEGIN
		primeira_variavel := primeira_variavel * 2;
		
		BEGIN
			primeira_variavel := 7;
		END;
		
		RETURN primeira_variavel;
	END
$$ LANGUAGE plpgsql;

SELECT primeira_pl();
```

> Por padrão, por boas práticas, evite a todo custo definir duas variáveis com o mesmo nome em blocos diferentes.

O que é um escopo de variáveis?

    É o bloco onde uma variável “vive”. Fora deste bloco, ela não existe, ou seja, está fora de seu escopo.

> Uma variável declarada em um bloco mais interno não pode ser acessada em um bloco mais externo.
> Variáveis com o mesmo nome podem existir em diferentes escopos.

## 4. Estruturas de controle

### **Retornos em PLs**

***Diferença no retorno do registro que vem vazio ao invés de nulo:***

```sql
DROP FUNCTION cria_a;

CREATE OR REPLACE FUNCTION cria_a(nome VARCHAR) RETURNS void AS $$
	BEGIN
		INSERT INTO a (nome) VALUES ('Patricia');
	END
$$ LANGUAGE plpgsql;

SELECT cria_a('Vinicius Dias');
```

***Retorna uma linha com os registros, referenciando o tipo instrutor:***

```sql
CREATE OR REPLACE FUNCTION cria_instrutor_falso() RETURNS instrutor AS $$
	BEGIN
		RETURN ROW(22, 'Nome falso', 200::DECIMAL)::instrutor;
	END
$$ LANGUAGE plpgsql;

SELECT id, salario FROM cria_instrutor_falso();
```

***Declara uma variável para inserir na consulta e retorna ela:***

```sql
CREATE OR REPLACE FUNCTION cria_instrutor_falso() RETURNS instrutor AS $$
	DECLARE
		retorno instrutor;
	BEGIN
		SELECT 22, 'Nome falso', 200::DECIMAL INTO retorno;
		RETURN retorno;
	END
$$ LANGUAGE plpgsql;

SELECT id, salario FROM cria_instrutor_falso();
```

***Retorna a consulta do select em si***

```sql
DROP FUNCTION instrutores_bem_pagos;

CREATE FUNCTION instrutores_bem_pagos(valor_salario DECIMAL)
RETURNS SETOF instrutor AS $$
	BEGIN
		RETURN QUERY SELECT * FROM instrutor WHERE salario > valor_salario;
	END
$$ LANGUAGE plpgsql;

SELECT * FROM instrutores_bem_pagos(300);
```

Qual o propósito ou funcionalidade da instrução SELECT INTO?

    SELECT INTO atribui o resultado de uma query a uma variável.

Para saber mais: [Sintaxe](https://www.postgresql.org/docs/current/plpgsql-statements.html#PLPGSQL-STATEMENTS-SQL-ONEROW)

### **If - Else**

A consulta abaixo é mais rápida que a posterior, pois vai direto ao ponto. Com apenas uma consulta, chega ao mesmo resultado. Utiliza uma condição para retornar os registros.

```sql
CREATE FUNCTION salario_ok(instrutor instrutor)
RETURNS VARCHAR AS $$
	BEGIN
		-- Se o salário do instrutor for maior do que 200, ok
		IF instrutor.salario > 200 THEN
			RETURN 'Salário está ok';
		-- Senão, pode aumentar
		ELSE
			RETURN 'Salário pode aumentar';
		END IF;
	END
$$ LANGUAGE plpgsql;

SELECT nome, salario_ok(instrutor) FROM instrutor;
```

A query abaixo entrega o mesmo resultado, porém com perda de performance, pois executa mais de uma consulta na mesma função. Existe uma condição e uma consulta extra pelo id.

```sql
DROP FUNCTION salario_ok;

CREATE FUNCTION salario_ok(id_instrutor INTEGER)
RETURNS VARCHAR AS $$
	DECLARE
		instrutor instrutor;
	BEGIN
		SELECT * FROM instrutor WHERE id = id_instrutor INTO instrutor;
		
		-- Se o salário do instrutor for maior do que 200, ok
		IF instrutor.salario > 200 THEN
			RETURN 'Salário está ok';
		-- Senão, pode aumentar
		ELSE
			RETURN 'Salário pode aumentar';
		END IF;
	END
$$ LANGUAGE plpgsql;

SELECT nome, salario_ok(instrutor.id) FROM instrutor;
```

### **Elseif**

No caso de várias condições, pode-se utilizar o `ELSEIF` para reduzir e deixar o código mais legível.

```sql
CREATE OR REPLACE FUNCTION salario_ok(id_instrutor INTEGER)
RETURNS VARCHAR AS $$
	DECLARE
		instrutor instrutor;
	BEGIN
		SELECT * FROM instrutor WHERE id = id_instrutor INTO instrutor;
		
		-- Se o salário do instrutor for maior do que 300, ok
		IF instrutor.salario > 300 THEN
			RETURN 'Salário está ok';
			-- Se forem exatos 300 reais, pode aumentar
		ELSEIF instrutor.salario = 300 THEN
			RETURN 'Salário pode aumentar';
			-- Caso contrário, o salário está defasado
		ELSE
			RETURN 'Salário está defasado';
		END IF;
	END
$$ LANGUAGE plpgsql;

SELECT nome, salario_ok(instrutor.id) FROM instrutor;
```

Obs. Um bloco `IF` não precisa ter nem um bloco `ELSE` nem um bloco `ELSEIF`. Ambos são opcionais.

> Um bloco IF não precisa necessariamente ser acompanhado por ELSE ou ELSEIF. Inclusive é bastante comum na programação termos somente o IF. [Aqui](https://www.postgresql.org/docs/current/plpgsql-control-structures.html#PLPGSQL-CONDITIONALS) você pode conferir a sintaxe.

### **Case When**

O `CASE` funciona semelhante ao `IF`, contudo é ainda mais legível.

```sql
CREATE OR REPLACE FUNCTION salario_ok(id_instrutor INTEGER)
RETURNS VARCHAR AS $$
	DECLARE
		instrutor instrutor;
	BEGIN
		SELECT * FROM instrutor WHERE id = id_instrutor INTO instrutor;
		
		CASE
			WHEN instrutor.salario = 100 THEN
				RETURN 'Salário muito baixo';
			WHEN instrutor.salario = 200 THEN
				RETURN 'Salário baixo';
			WHEN instrutor.salario = 300 THEN
				RETURN 'Salário ok';
			ELSE
				RETURN 'Salário ótimo';
		END CASE;
	END
$$ LANGUAGE plpgsql;

SELECT nome, salario_ok(instrutor.id) FROM instrutor;
```

Para simplificar o case, pode-se utilizar a expressão de comparação ao case e diferenciar o when.

```sql
CREATE OR REPLACE FUNCTION salario_ok(id_instrutor INTEGER)
RETURNS VARCHAR AS $$
	DECLARE
		instrutor instrutor;
	BEGIN
		SELECT * FROM instrutor WHERE id = id_instrutor INTO instrutor;
		
		CASE instrutor.salario
			WHEN 100 THEN
				RETURN 'Salário muito baixo';
			WHEN 200 THEN
				RETURN 'Salário baixo';
			WHEN 300 THEN
				RETURN 'Salário ok';
			ELSE
				RETURN 'Salário ótimo';
		END CASE;
	END
$$ LANGUAGE plpgsql;

SELECT nome, salario_ok(instrutor.id) FROM instrutor;
```

Quando parece ser mais interessante usar CASE?

	Quando precisaríamos de muitos ELSEIFs

> Se nós precisarmos de muito blocos ELSEIF, normalmente podemos trocar por um CASE. Se a comparação for de igualdade, usamos o CASE simples. Senão, podemos usar o Searched case. Dá uma olhada [aqui](https://www.postgresql.org/docs/current/plpgsql-control-structures.html#id-1.8.8.8.6.6) pra mais detalhes.

## 5. Estruturas de repetição

### **Return Next**

O `RETURN NEXT` é utilizado para retornar múltiplas linhas de uma função.

```sql
CREATE OR REPLACE FUNCTION tabuada(numero INTEGER)
RETURNS SETOF VARCHAR AS $$
	BEGIN
		RETURN NEXT numero * 1;
		RETURN NEXT numero * 2;
		RETURN NEXT numero * 3;
		RETURN NEXT numero * 4;
		RETURN NEXT numero * 5;
		RETURN NEXT numero * 6;
		RETURN NEXT numero * 7;
		RETURN NEXT numero * 8;
		RETURN NEXT numero * 9;
	END
$$ LANGUAGE plpgsql;

SELECT tabuada(2);
```

Para saber mais: [SETOF PLpgSQL](https://www.postgresql.org/docs/current/plpgsql-control-structures.html#PLPGSQL-STATEMENTS-RETURNING)

### **Conhecendo Loop**

Criar um `LOOP` faz com que reduza a repetição e o código fique mais legível.

```sql
DROP FUNCTION tabuada;

CREATE OR REPLACE FUNCTION tabuada(numero INTEGER)
RETURNS SETOF INTEGER AS $$
	DECLARE
		-- multiplicador que começa com 1 e vai até 10
		multiplicador INTEGER DEFAULT 1;
	BEGIN
		LOOP
			-- multiplica numeros
			RETURN NEXT numero * multiplicador;

			-- aumenta o multiplicador
			multiplicador := multiplicador + 1;

			-- finaliza o multiplicador
			EXIT WHEN multiplicador = 10;
		END LOOP;
	END
$$ LANGUAGE plpgsql;

SELECT tabuada(3);
```

Pode-se ainda concatenar strings para mostrar o cálculo, não apenas o resultado dele.

```sql
DROP FUNCTION tabuada;

CREATE OR REPLACE FUNCTION tabuada(numero INTEGER)
RETURNS SETOF VARCHAR AS $$
	DECLARE
		multiplicador INTEGER DEFAULT 1;
	BEGIN
		LOOP
			RETURN NEXT numero || ' x ' || multiplicador || ' = ' || numero * multiplicador;
			multiplicador := multiplicador + 1;
			EXIT WHEN multiplicador = 10;
		END LOOP;
	END
$$ LANGUAGE plpgsql;

SELECT tabuada(4);
```

### **While**

O `WHILE` funciona como um contador, finalizando o loop quando alcança o que foi definido no while.

```sql
DROP FUNCTION tabuada;

CREATE OR REPLACE FUNCTION tabuada(numero INTEGER)
RETURNS SETOF VARCHAR AS $$
	DECLARE
		multiplicador INTEGER DEFAULT 1;
	BEGIN
		WHILE multiplicador < 10 LOOP
			RETURN NEXT numero || ' x ' || multiplicador || ' = ' || numero * multiplicador;
			multiplicador := multiplicador + 1;
		END LOOP;
	END
$$ LANGUAGE plpgsql;

SELECT tabuada(5);
```

Qual a principal diferença entre estes LOOP e WHILE … LOOP?

	Sem o WHILE nós precisamos utilizar o EXIT para informar a condição de saída dentro do loop.

> O WHILE já nos permite definir explicitamente a condição de saída. Sem o WHILE nós precisamos, dentro do loop, executar a instrução EXIT para verificar a condição de saída.

### **Conhecendo For**

O `FOR` assim como o while, funciona como um contador, porém ele também faz o incremento do loop. Fazendo com que não seja mais necessário incrementar o loop de 1 em 1 e, também, não mais necessário declarar a variável.

```sql
DROP FUNCTION tabuada;

CREATE OR REPLACE FUNCTION tabuada(numero INTEGER)
RETURNS SETOF VARCHAR AS $$
	BEGIN
		FOR multiplicador IN 1..9 LOOP
			RETURN NEXT numero || ' x ' || multiplicador || ' = ' || numero * multiplicador;
		END LOOP;
	END
$$ LANGUAGE plpgsql;

SELECT tabuada(6);
```

Com o for é possível realizar um loop, inclusive, com uma query.

```sql
CREATE FUNCTION instrutor_com_salario(
	OUT nome VARCHAR,
	OUT salario_ok VARCHAR
)
RETURNS SETOF record AS $$
	DECLARE
		instrutor instrutor;
	BEGIN
		FOR instrutor IN SELECT * FROM instrutor LOOP
			nome := instrutor.nome;
			salario_ok = salario_ok(instrutor.id);
			
			RETURN NEXT;
		END LOOP;
	END
$$ LANGUAGE plpgsql;

SELECT * FROM instrutor_com_salario();
```

> Estruturas de repetição (do inglês, loops) são muito utilizadas na programação para percorrer listas de valores.
> Sejam vetores de informações, registros de tabelas, sequências pré-definidas… Em cada caso vamos analisar qual a estrutura ideal de repetição para aplicar.

Para saber mais: [Loops](https://www.postgresql.org/docs/current/plpgsql-control-structures.html#PLPGSQL-CONTROL-STRUCTURES-LOOPS)

## 6. Mão na massa

### **Curso na categoria**

É possível criar funções para popular tabelas, inserindo novos registros ou atualizando os existentes. Para isso, é necessário relacionar a função à uma ou mais tabelas e declarar variáveis e campos. Podendo realizar uma verificação se o item já existe antes de realmente inserir como novo.

```sql
CREATE FUNCTION cria_curso(nome_curso VARCHAR, nome_categoria VARCHAR)
RETURNS void AS $$
	DECLARE
		id_categoria INTEGER;
	BEGIN
		SELECT id INTO id_categoria FROM categoria
		WHERE nome = nome_categoria;
		
		IF NOT FOUND THEN
			INSERT INTO categoria (nome) VALUES (nome_categoria)
			RETURNING id INTO id_categoria;
		END IF;
		
		INSERT INTO curso (nome, categoria_id)
		VALUES (nome_curso, id_categoria);
	END;
$$ LANGUAGE plpgsql;

SELECT cria_curso('PHP', 'Programação');
SELECT cria_curso('Java', 'Programação');

SELECT * FROM curso;
SELECT * FROM categoria;
```

O que é essa variável FOUND?

	Uma variável que informa se houve algum resultado produzido pela última query executada.

> Esta variável é definida automaticamente em toda função em PLpgSQL e começa como FALSE. Nesse [link](https://www.postgresql.org/docs/current/plpgsql-statements.html#PLPGSQL-STATEMENTS-DIAGNOSTICS) você confere em que situações ela será definida como TRUE.

Para saber mais: [Execute](Para%20saber%20mais/Aula%206%20-%20Atividade%204%20Para%20saber%20mais_%20Execute.pdf)

⬆️ [Voltar ao topo](#postgresql-desenvolva-com-plpgsql) ⬆️

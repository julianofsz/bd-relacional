# Atividade de Aprofundamento em Scripts SQL

**Banco de Dados Utilizado:** Sakila (MySQL)  
**Formato:** Conceito + 2 desafios (gerados por IA) + Soluções em SQL

---

## Item 1: Tipos de JOIN

**Conceito:** `JOIN`s são para conectar tabelas. Simples assim.

- **INNER JOIN:** Só pega o que tem nos dois lados. Tipo a interseção.
- **LEFT JOIN:** Pega tudo da tabela da esquerda e, se achar, a correspondência da direita. Se não achar, vem `NULL`.
- **RIGHT JOIN:** O contrário do LEFT. Pega tudo da direita e, se achar, a correspondência da esquerda. Se não, `NULL`.

**Desafio 1 (INNER JOIN):** Nomes de clientes e os títulos dos filmes que eles alugaram.

```sql
SELECT c.first_name, c.last_name, f.title
FROM customer c
INNER JOIN rental r ON c.customer_id = r.customer_id
INNER JOIN inventory i ON r.inventory_id = i.inventory_id
INNER JOIN film f ON i.film_id = f.film_id;
Desafio 2 (LEFT JOIN): Todos os clientes, mesmo quem nunca alugou filme. Pra quem alugou, mostra o título.

SQL

SELECT c.first_name, c.last_name, f.title
FROM customer c
LEFT JOIN rental r ON c.customer_id = r.customer_id
LEFT JOIN inventory i ON r.inventory_id = i.inventory_id
LEFT JOIN film f ON i.film_id = f.film_id;
Item 2: Múltiplos JOINs
Conceito: Quando a info tá espalhada, a gente vai JOINando uma tabela na outra. É tipo uma corrente: um JOIN junta duas, e o resultado já serve pro próximo JOIN.

Desafio 1: Relatório com nome do cliente, cidade, país e os filmes alugados.

SQL

SELECT c.first_name, c.last_name, ci.city, co.country, f.title
FROM customer c
JOIN address a ON c.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id
JOIN country co ON ci.country_id = co.country_id
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id;
Desafio 2: Atores, filmes que fizeram e categorias desses filmes.

SQL

SELECT a.first_name, a.last_name, f.title, c.name AS category
FROM actor a
JOIN film_actor fa ON a.actor_id = fa.actor_id
JOIN film f ON fa.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id;
Item 3: Funções na Cláusula SELECT
Conceito: Funções no SELECT servem pra arrumar, calcular ou mudar os dados na hora da consulta.

Texto: CONCAT, UPPER, LOWER, LENGTH, REPLACE (pra juntar, maiúscula, minúscula, tamanho, substituir).
Numéricas: ROUND, CEIL, FLOOR (pra arredondar).
NULLs: COALESCE, IFNULL (pra trocar NULL por outra coisa).
Desafio 1 (Strings): Nome completo dos clientes em maiúsculas e o tamanho desse nome.

SQL

SELECT UPPER(CONCAT(first_name, ' ', last_name)) AS full_name,
       LENGTH(CONCAT(first_name, ' ', last_name)) AS name_length
FROM customer;
Desafio 2 (NULLs e arredondamento): Pagamentos arredondados e NULLs como zero.

SQL

SELECT payment_id,
       ROUND(IFNULL(amount, 0), 2) AS rounded_amount
FROM payment;
Item 4: Funções de Data e Hora
Conceito: Funções pra mexer com datas e horas: NOW(), DATEDIFF(), DATE_FORMAT(), YEAR(), etc.

Desafio 1: Calcular os dias entre aluguel e devolução.

SQL

SELECT rental_id, DATEDIFF(return_date, rental_date) AS rental_days
FROM rental
WHERE return_date IS NOT NULL;
Desafio 2: Data de pagamento formatada 'dd/mm/yyyy'.

SQL

SELECT payment_id, DATE_FORMAT(payment_date, '%d/%m/%Y') AS formatted_date
FROM payment;
Item 5: Subconsultas (Subqueries)
Conceito: Uma consulta dentro da outra. Pode ser pra pegar um valor único (escalar), vários valores (IN), ou ser "correlacionada" (depende da consulta de fora).

Desafio 1: Clientes que alugaram algum filme de 'Action'.

SQL

SELECT DISTINCT c.first_name, c.last_name
FROM customer c
WHERE c.customer_id IN (
    SELECT r.customer_id
    FROM rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    JOIN film f ON i.film_id = f.film_id
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category cat ON fc.category_id = cat.category_id
    WHERE cat.name = 'Action'
);
Desafio 2 (correlacionada): Filmes e quantas vezes cada um foi alugado.

SQL

SELECT title,
       (SELECT COUNT(*) FROM inventory i
        JOIN rental r ON i.inventory_id = r.inventory_id
        WHERE i.film_id = f.film_id) AS rental_count
FROM film f;
Item 6: Subconsultas em Comandos DML
Conceito: Dá pra usar subconsultas em INSERT, UPDATE e DELETE pra definir quem ou o que vai ser afetado de um jeito dinâmico.

Desafio 1 (UPDATE): Dar 10% de desconto em pagamentos de clientes que alugaram mais de 40 filmes.

SQL

UPDATE payment
SET amount = amount * 0.9
WHERE customer_id IN (
    SELECT customer_id
    FROM rental
    GROUP BY customer_id
    HAVING COUNT(*) > 40
);
Desafio 2 (DELETE): Deletar pagamentos de clientes que nunca alugaram filme.

SQL

DELETE FROM payment
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM rental
);
Item 7: WITH AS (CTE) e DISTINCT
Conceito:

DISTINCT: Tira as linhas repetidas.
CTE (WITH AS): Ajuda a organizar queries grandes, criando "blocos" nomeados e temporários que você usa depois.
Desafio 1: Contar quantas cidades únicas tem no banco.

SQL

SELECT COUNT(DISTINCT city) AS unique_cities FROM city;
Desafio 2 (CTE): Listar os filmes mais alugados usando CTE.

SQL

WITH film_rentals AS (
    SELECT i.film_id, COUNT(*) AS total_rentals
    FROM rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    GROUP BY i.film_id
)
SELECT f.title, fr.total_rentals
FROM film_rentals fr
JOIN film f ON f.film_id = fr.film_id
ORDER BY fr.total_rentals DESC;
Item 8: Funções de Janela (Window Functions) para Ranking
Conceito: Funções de janela fazem cálculos em um "pedaço" de dados relacionados, sem agrupar tudo. O OVER() define esse pedaço. Ótimas pra ranking:

ROW_NUMBER(): Numera sequencialmente.
RANK(): Dá ranking, mas "pula" se tiver empate.
DENSE_RANK(): Dá ranking, não pula se tiver empate.
Desafio 1 (RANK): Clientes, gastos totais e ranking de gastos (pulando posições se tiver empate).

SQL

SELECT
    c.first_name,
    c.last_name,
    SUM(p.amount) AS total_spent,
    RANK() OVER (ORDER BY SUM(p.amount) DESC) AS ranking_gasto
FROM
    customer c
JOIN
    payment p ON c.customer_id = p.customer_id
GROUP BY
    c.customer_id, c.first_name, c.last_name
ORDER BY
    ranking_gasto, total_spent DESC;
Desafio 2 (DENSE_RANK): Os 5 filmes mais longos de cada categoria, sem pular ranking em caso de empate na duração.

SQL

SELECT
    c.name AS category_name,
    f.title,
    f.length,
    DENSE_RANK() OVER (PARTITION BY c.name ORDER BY f.length DESC) AS rank_duracao
FROM
    film f
JOIN
    film_category fc ON f.film_id = fc.film_id
JOIN
    category c ON fc.category_id = c.category_id
WHERE
    DENSE_RANK() OVER (PARTITION BY c.name ORDER BY f.length DESC) <= 5
ORDER BY
    c.name, rank_duracao;
Item 9: Lógica Condicional com CASE
Conceito: CASE WHEN ... THEN ... ELSE ... END é tipo um "if/then/else" no SQL. Cria novas colunas com valores baseados em condições.

Desafio 1: Classificar filmes por duração: 'Curto' (até 60 min), 'Médio' (61-120 min), 'Longo' (acima de 120 min).

SQL

SELECT
    title,
    length,
    CASE
        WHEN length <= 60 THEN 'Curto'
        WHEN length > 60 AND length <= 120 THEN 'Médio'
        ELSE 'Longo'
    END AS classificacao_duracao
FROM
    film
ORDER BY
    length;
Desafio 2: Nível de atividade do cliente por filmes alugados: 'Baixo' (até 10), 'Médio' (11-30), 'Alto' (mais de 30).

SQL

SELECT
    c.first_name,
    c.last_name,
    COUNT(r.rental_id) AS total_filmes_alugados,
    CASE
        WHEN COUNT(r.rental_id) <= 10 THEN 'Baixo'
        WHEN COUNT(r.rental_id) > 10 AND COUNT(r.rental_id) <= 30 THEN 'Médio'
        ELSE 'Alto'
    END AS nivel_atividade
FROM
    customer c
LEFT JOIN
    rental r ON c.customer_id = r.customer_id
GROUP BY
    c.customer_id, c.first_name, c.last_name
ORDER BY
    total_filmes_alugados DESC;
Item 10: Combinando Resultados com UNION
Conceito: UNION e UNION ALL juntam resultados de várias consultas SELECT.

UNION: Tira duplicatas. Mais lento.
UNION ALL: Mantém duplicatas. Mais rápido. As consultas precisam ter o mesmo número e tipo de colunas.
Desafio 1: Uma lista única com nomes completos de atores e clientes.

SQL

SELECT
    first_name,
    last_name
FROM
    actor
UNION
SELECT
    first_name,
    last_name
FROM
    customer
ORDER BY
    first_name, last_name;
Desafio 2: Filmes lançados em 2006 OU com mais de 180 min. (Pode incluir duplicatas se um filme se encaixar nas duas).

SQL

SELECT title, release_year, length
FROM film
WHERE release_year = 2006
UNION ALL
SELECT title, release_year, length
FROM film
WHERE length > 180
ORDER BY title;
Item 11: Visões (Views)
Conceito: Views são "tabelas virtuais" baseadas em uma SELECT. Não guardam dados, mas a query. Servem pra:

Simplificar consultas chatas.
Reaproveitar código.
Controlar acesso (segurança).
Desafio 1: Criar uma VIEW filmes_mais_alugados com título e total de aluguéis (do mais para o menos).

SQL

CREATE VIEW filmes_mais_alugados AS
SELECT
    f.title,
    COUNT(r.rental_id) AS total_alugueis
FROM
    film f
JOIN
    inventory i ON f.film_id = i.film_id
JOIN
    rental r ON i.inventory_id = r.inventory_id
GROUP BY
    f.title
ORDER BY
    total_alugueis DESC;
Desafio 2: Usar a VIEW filmes_mais_alugados pra listar os 5 mais alugados.

SQL

SELECT
    title,
    total_alugueis
FROM
    filmes_mais_alugados
LIMIT 5;
Item 12: Filtragem com WHERE vs. HAVING
Conceito: Ambos filtram, mas em momentos diferentes:

WHERE: Filtra linhas individuais antes de agrupar (GROUP BY). Pensa nele como o "primeiro corte".
HAVING: Filtra grupos de linhas depois que o GROUP BY e as agregações (COUNT, SUM) já rolaram. Ele filtra o "resumo".
Desafio 1: Categorias de filmes com média de duração acima de 120 minutos.

SQL

SELECT
    c.name AS category_name,
    AVG(f.length) AS average_length
FROM
    category c
JOIN
    film_category fc ON c.category_id = fc.category_id
JOIN
    film f ON fc.film_id = f.film_id
GROUP BY
    c.name
HAVING
    AVG(f.length) > 120
ORDER BY
    average_length DESC;
Desafio 2: Clientes que fizeram mais de 30 pagamentos E gastaram mais de 150.

SQL

SELECT
    c.first_name,
    c.last_name,
    COUNT(p.payment_id) AS total_pagamentos,
    SUM(p.amount) AS total_gasto
FROM
    customer c
JOIN
    payment p ON c.customer_id = p.customer_id
GROUP BY
    c.customer_id, c.first_name, c.last_name
HAVING
    COUNT(p.payment_id) > 30 AND SUM(p.amount) > 150
ORDER BY
    total_gasto DESC;
Item 13: Controle de Transações (COMMIT e ROLLBACK)
Conceito: Transação é um monte de operações SQL que funcionam como um pacote só. Ou tudo dá certo e salva (COMMIT), ou se der erro, tudo desfaz (ROLLBACK) e o banco volta ao que era. Essencial pra não bagunçar os dados.

START TRANSACTION/BEGIN: Começa o pacote.
COMMIT: Salva tudo de vez.
ROLLBACK: Desfaz tudo que foi feito desde o START TRANSACTION.
Desafio 1: Simular um aluguel: inserir na tabela rental e atualizar last_update do inventory. Tudo num bloco transacional.

SQL

-- Cenário de Sucesso (COMMIT)
START TRANSACTION;

-- 1. Inserir um novo aluguel (ex: customer_id = 1, inventory_id = 1, staff_id = 1)
INSERT INTO rental (rental_date, inventory_id, customer_id, staff_id, return_date, last_update)
VALUES (NOW(), 1, 1, 1, NULL, NOW());

-- 2. Atualizar o last_update do inventário
UPDATE inventory
SET last_update = NOW()
WHERE inventory_id = 1;

COMMIT;

-- Cenário de Falha (ROLLBACK) - Exemplo
-- START TRANSACTION;
-- INSERT INTO rental (rental_date, inventory_id, customer_id, staff_id, return_date, last_update)
-- VALUES (NOW(), 99999, 1, 1, NULL, NOW()); -- inventory_id inválido
-- ROLLBACK; -- Desfaz a inserção
Desafio 2: Simular exclusão de cliente, pagamentos e aluguéis. Tudo numa transação pra garantir que ou apaga tudo ou nada.

SQL

-- Cenário de Sucesso (COMMIT)
START TRANSACTION;

SET @customer_to_delete = 100; -- Cliente para apagar

DELETE FROM payment WHERE customer_id = @customer_to_delete;
DELETE FROM rental WHERE customer_id = @customer_to_delete;
DELETE FROM customer WHERE customer_id = @customer_to_delete;

COMMIT;

-- Cenário de Falha (ROLLBACK) - Exemplo
-- START TRANSACTION;
-- SET @customer_to_delete_fail = 99999;
-- DELETE FROM payment WHERE customer_id = @customer_to_delete_fail;
-- ROLLBACK;
Item 14: Gatilhos (Triggers)
Conceito: Um Trigger é um código SQL que roda automaticamente quando algo acontece em uma tabela (INSERT, UPDATE ou DELETE). Bom pra auditoria, validação complexa ou pra manter dados sincronizados.

Desafio 1: Criar um TRIGGER que, ao atualizar a tabela film, registra o film_id e a hora na tabela film_audit_log (precisa criar ela primeiro).

SQL

-- Primeiro, crie a tabela de log
CREATE TABLE film_audit_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    film_id SMALLINT NOT NULL,
    update_timestamp DATETIME NOT NULL
);

-- Agora, crie o TRIGGER
DELIMITER //

CREATE TRIGGER after_film_update
AFTER UPDATE ON film
FOR EACH ROW
BEGIN
    INSERT INTO film_audit_log (film_id, update_timestamp)
    VALUES (NEW.film_id, NOW());
END;
//

DELIMITER ;

-- Teste: UPDATE film SET rental_duration = 4 WHERE film_id = 1;
-- Verifique: SELECT * FROM film_audit_log;
Desafio 2: Criar um TRIGGER que, antes de inserir um aluguel, bloqueie se o filme tiver classificação 'NC-17' e mostre um erro.

SQL

DELIMITER //

CREATE TRIGGER before_rental_insert_check_rating
BEFORE INSERT ON rental
FOR EACH ROW
BEGIN
    DECLARE film_rating VARCHAR(10);

    SELECT f.rating INTO film_rating
    FROM inventory i
    JOIN film f ON i.film_id = f.film_id
    WHERE i.inventory_id = NEW.inventory_id;

    IF film_rating = 'NC-17' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Não é permitido alugar filmes com classificação NC-17.';
    END IF;
END;
//

DELIMITER ;

-- Teste (substitua o inventory_id por um de filme NC-17 ou PG):
-- INSERT INTO rental (rental_date, inventory_id, customer_id, staff_id, last_update)
-- VALUES (NOW(), (SELECT inventory_id FROM inventory i JOIN film f ON i.film_id = f.film_id WHERE f.rating = 'NC-17' LIMIT 1), 1, 1, NOW());
Item 15: Índices e Otimização (Indexes)
Conceito: Índice é tipo o índice de um livro: te ajuda a achar a informação rapidão, sem ter que ler tudo. No banco, acelera as consultas em tabelas grandes. Sem índice, o banco lê linha por linha (lento!). Com índice, ele vai direto. Usa CREATE INDEX.

Desafio 1: Otimizar busca por sobrenome em customer.

Consulta Lenta:

SQL

SELECT *
FROM customer
WHERE last_name = 'SMITH';
Por que melhora? Sem índice, pra achar 'SMITH' ele lê a tabela inteira. Com o índice em last_name, ele vai direto pros 'SMITH', tipo um atalho. Ganho gigante em tabela grande.

Criação do Índice:

SQL

CREATE INDEX idx_customer_last_name ON customer (last_name);
Desafio 2: Otimizar busca em rental por customer_id E rental_date.

Consulta a Otimizar:

SQL

SELECT rental_id, inventory_id, return_date
FROM rental
WHERE customer_id = 100 AND rental_date BETWEEN '2005-05-01' AND '2005-06-30';
Criação do Índice Composto:

SQL

CREATE INDEX idx_rental_customer_date ON rental (customer_id, rental_date);
Por que melhora? Pra buscar em duas colunas, um índice composto (customer_id, rental_date) é o ideal. Ele já organiza os dados pra encontrar o cliente e, dentro do cliente, as datas, tudo junto e rápido.
```

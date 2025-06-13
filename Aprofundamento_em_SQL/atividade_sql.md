# Atividade de Aprofundamento em Scripts SQL

**Banco de Dados Utilizado:** Sakila (MySQL)  
**Formato:** Conceito + 2 desafios (gerados por IA) + Soluções em SQL

---

## 1. Tipos de JOIN

**Conceito:** `JOIN`s são usados para combinar linhas de duas ou mais tabelas com base em uma coluna relacionada entre elas.

-   **`INNER JOIN`:** Retorna apenas os registros que têm valores correspondentes em ambas as tabelas (interseção).
-   **`LEFT JOIN`:** Retorna todos os registros da tabela da esquerda e os registros correspondentes da tabela da direita. Se não houver correspondência, o resultado é `NULL` no lado direito.
-   **`RIGHT JOIN`:** Retorna todos os registros da tabela da direita e os registros correspondentes da tabela da esquerda. Se não houver correspondência, o resultado é `NULL` no lado esquerdo.

### Desafio 1
Listar os nomes dos clientes e os títulos dos filmes que eles alugaram.

### Solução 1
```sql
SELECT
    c.first_name,
    c.last_name,
    f.title
FROM customer c
INNER JOIN rental r ON c.customer_id = r.customer_id
INNER JOIN inventory i ON r.inventory_id = i.inventory_id
INNER JOIN film f ON i.film_id = f.film_id;
```

### Desafio 2
Mostrar todos os clientes, mesmo aqueles que nunca alugaram um filme. Para os que alugaram, mostrar o título do filme.

### Solução 2
```sql
SELECT
    c.first_name,
    c.last_name,
    f.title
FROM customer c
LEFT JOIN rental r ON c.customer_id = r.customer_id
LEFT JOIN inventory i ON r.inventory_id = i.inventory_id
LEFT JOIN film f ON i.film_id = f.film_id;
```

---

## 2. Múltiplos JOINs

**Conceito:** Para obter dados que estão espalhados por várias tabelas, você pode encadear múltiplos `JOIN`s. Cada `JOIN` conecta uma nova tabela ao resultado do `JOIN` anterior.

### Desafio 1
Criar um relatório com o nome do cliente, cidade, país e os títulos dos filmes que eles alugaram.

### Solução 1
```sql
SELECT
    c.first_name,
    c.last_name,
    ci.city,
    co.country,
    f.title
FROM customer c
JOIN address a ON c.address_id = a.address_id
JOIN city ci ON a.city_id = ci.city_id
JOIN country co ON ci.country_id = co.country_id
JOIN rental r ON c.customer_id = r.customer_id
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id;
```

### Desafio 2
Listar os atores, os filmes em que atuaram e as categorias desses filmes.

### Solução 2
```sql
SELECT
    a.first_name,
    a.last_name,
    f.title,
    c.name AS category
FROM actor a
JOIN film_actor fa ON a.actor_id = fa.actor_id
JOIN film f ON fa.film_id = f.film_id
JOIN film_category fc ON f.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id;
```

---

## 3. Funções na Cláusula SELECT

**Conceito:** Funções no `SELECT` permitem manipular, formatar ou calcular dados diretamente na consulta.

-   **Texto:** `CONCAT`, `UPPER`, `LOWER`, `LENGTH`, `REPLACE`.
-   **Numéricas:** `ROUND`, `CEIL`, `FLOOR`.
-   **Tratamento de Nulos:** `COALESCE`, `IFNULL`.

### Desafio 1
Exibir o nome completo dos clientes em letras maiúsculas e o comprimento total do nome.

### Solução 1
```sql
SELECT
    UPPER(CONCAT(first_name, ' ', last_name)) AS full_name,
    LENGTH(CONCAT(first_name, ' ', last_name)) AS name_length
FROM customer;
```

### Desafio 2
Mostrar os valores de pagamento arredondados para duas casas decimais, tratando valores `NULL` como zero.

### Solução 2
```sql
SELECT
    payment_id,
    ROUND(IFNULL(amount, 0), 2) AS rounded_amount
FROM payment;
```

---

## 4. Funções de Data e Hora

**Conceito:** Funções específicas para manipular e extrair informações de tipos de dados de data e hora, como `NOW()`, `DATEDIFF()`, `DATE_FORMAT()`, `YEAR()`.

### Desafio 1
Calcular o número de dias entre a data de aluguel e a data de devolução.

### Solução 1
```sql
SELECT
    rental_id,
    DATEDIFF(return_date, rental_date) AS rental_days
FROM rental
WHERE return_date IS NOT NULL;
```

### Desafio 2
Formatar a data de pagamento para o padrão `dd/mm/yyyy`.

### Solução 2
```sql
SELECT
    payment_id,
    DATE_FORMAT(payment_date, '%d/%m/%Y') AS formatted_date
FROM payment;
```

---

## 5. Subconsultas (Subqueries)

**Conceito:** Uma `SELECT` dentro de outra `SELECT`. Pode ser usada para retornar um único valor (escalar), uma lista de valores (para usar com `IN`), ou de forma correlacionada (dependendo da consulta externa).

### Desafio 1
Listar os clientes que alugaram pelo menos um filme da categoria 'Action'.

### Solução 1
```sql
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
```

### Desafio 2
Para cada filme, contar quantas vezes ele foi alugado usando uma subconsulta correlacionada.

### Solução 2
```sql
SELECT
    title,
    (SELECT COUNT(*)
     FROM inventory i
     JOIN rental r ON i.inventory_id = r.inventory_id
     WHERE i.film_id = f.film_id) AS rental_count
FROM film f;
```

---

## 6. Subconsultas em Comandos DML

**Conceito:** Usar subconsultas em comandos de manipulação de dados (`INSERT`, `UPDATE`, `DELETE`) para definir dinamicamente os registros que serão afetados.

### Desafio 1
Aplicar um desconto de 10% nos pagamentos de clientes que alugaram mais de 40 filmes.

### Solução 1
```sql
UPDATE payment
SET amount = amount * 0.9
WHERE customer_id IN (
    SELECT customer_id
    FROM rental
    GROUP BY customer_id
    HAVING COUNT(*) > 40
);
```

### Desafio 2
Deletar todos os pagamentos de clientes que nunca realizaram um aluguel.

### Solução 2
```sql
DELETE FROM payment
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM rental
);
```

---

## 7. `WITH AS` (CTE) e `DISTINCT`

**Conceito:**
-   **`DISTINCT`:** Remove linhas duplicadas do resultado de uma consulta.
-   **`CTE (Common Table Expression)`:** Definida com `WITH AS`, permite criar um conjunto de resultados temporário e nomeado, que pode ser referenciado posteriormente na consulta principal. Ajuda a organizar queries complexas.

### Desafio 1
Contar quantas cidades únicas existem no banco de dados.

### Solução 1
```sql
SELECT COUNT(DISTINCT city) AS unique_cities FROM city;
```

### Desafio 2
Listar os filmes mais alugados em ordem decrescente, utilizando uma CTE.

### Solução 2
```sql
WITH film_rentals AS (
    SELECT
        i.film_id,
        COUNT(*) AS total_rentals
    FROM rental r
    JOIN inventory i ON r.inventory_id = i.inventory_id
    GROUP BY i.film_id
)
SELECT
    f.title,
    fr.total_rentals
FROM film_rentals fr
JOIN film f ON f.film_id = fr.film_id
ORDER BY fr.total_rentals DESC;
```

---

## 8. Funções de Janela (Window Functions)

**Conceito:** Realizam cálculos em um conjunto de linhas relacionadas (uma "janela"), sem agrupar o resultado em uma única linha. A cláusula `OVER()` define a janela. São ótimas para criar rankings.

-   **`ROW_NUMBER()`:** Numera as linhas sequencialmente.
-   **`RANK()`:** Cria um ranking, mas "pula" posições em caso de empate.
-   **`DENSE_RANK()`:** Cria um ranking, mas não pula posições em caso de empate.

### Desafio 1
Criar um ranking de clientes com base no total gasto, pulando posições em caso de empate.

### Solução 1
```sql
SELECT
    c.first_name,
    c.last_name,
    SUM(p.amount) AS total_spent,
    RANK() OVER (ORDER BY SUM(p.amount) DESC) AS ranking_gasto
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY ranking_gasto, total_spent DESC;
```

### Desafio 2
Listar os 5 filmes mais longos de cada categoria, sem pular posições no ranking em caso de empate na duração.

### Solução 2
```sql
WITH RankedFilms AS (
    SELECT
        c.name AS category_name,
        f.title,
        f.length,
        DENSE_RANK() OVER (PARTITION BY c.name ORDER BY f.length DESC) AS rank_duracao
    FROM film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
)
SELECT
    category_name,
    title,
    length,
    rank_duracao
FROM RankedFilms
WHERE rank_duracao <= 5
ORDER BY category_name, rank_duracao;
```

---

## 9. Lógica Condicional com `CASE`

**Conceito:** A expressão `CASE WHEN ... THEN ... ELSE ... END` funciona como uma estrutura `if/then/else` dentro do SQL, permitindo criar colunas com valores baseados em condições.

### Desafio 1
Classificar filmes por duração: 'Curto' (até 60 min), 'Médio' (61-120 min) e 'Longo' (acima de 120 min).

### Solução 1
```sql
SELECT
    title,
    length,
    CASE
        WHEN length <= 60 THEN 'Curto'
        WHEN length > 60 AND length <= 120 THEN 'Médio'
        ELSE 'Longo'
    END AS classificacao_duracao
FROM film
ORDER BY length;
```

### Desafio 2
Classificar o nível de atividade do cliente com base no número de filmes alugados: 'Baixo' (até 10), 'Médio' (11-30) e 'Alto' (mais de 30).

### Solução 2
```sql
SELECT
    c.first_name,
    c.last_name,
    COUNT(r.rental_id) AS total_filmes_alugados,
    CASE
        WHEN COUNT(r.rental_id) <= 10 THEN 'Baixo'
        WHEN COUNT(r.rental_id) > 10 AND COUNT(r.rental_id) <= 30 THEN 'Médio'
        ELSE 'Alto'
    END AS nivel_atividade
FROM customer c
LEFT JOIN rental r ON c.customer_id = r.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_filmes_alugados DESC;
```

---

## 10. Combinando Resultados com `UNION`

**Conceito:** `UNION` e `UNION ALL` combinam o resultado de duas ou mais consultas `SELECT` em um único conjunto de resultados.

-   **`UNION`:** Remove linhas duplicadas (mais lento).
-   **`UNION ALL`:** Mantém todas as linhas, incluindo duplicatas (mais rápido).
-   *As consultas devem ter o mesmo número e tipos de colunas compatíveis.*

### Desafio 1
Criar uma lista única com os nomes completos de atores e clientes.

### Solução 1
```sql
SELECT first_name, last_name FROM actor
UNION
SELECT first_name, last_name FROM customer
ORDER BY first_name, last_name;
```

### Desafio 2
Listar todos os filmes lançados em 2006 **OU** com duração superior a 180 minutos, permitindo duplicatas.

### Solução 2
```sql
SELECT title, release_year, length FROM film
WHERE release_year = 2006
UNION ALL
SELECT title, release_year, length FROM film
WHERE length > 180
ORDER BY title;
```

---

## 11. Visões (Views)

**Conceito:** Views são "tabelas virtuais" baseadas no resultado de uma consulta `SELECT`. Elas não armazenam dados fisicamente, mas simplificam consultas complexas, promovem a reutilização de código e podem ser usadas para controle de acesso.

### Desafio 1
Criar uma `VIEW` chamada `filmes_mais_alugados` que contenha o título do filme e o total de aluguéis.

### Solução 1
```sql
CREATE OR REPLACE VIEW filmes_mais_alugados AS
SELECT
    f.title,
    COUNT(r.rental_id) AS total_alugueis
FROM film f
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY f.title
ORDER BY total_alugueis DESC;
```

### Desafio 2
Usar a `VIEW` criada para listar os 5 filmes mais alugados.

### Solução 2
```sql
SELECT
    title,
    total_alugueis
FROM filmes_mais_alugados
LIMIT 5;
```

---

## 12. Filtragem com `WHERE` vs. `HAVING`

**Conceito:** Ambos filtram dados, mas em momentos diferentes do processamento da consulta.

-   **`WHERE`:** Filtra as linhas **antes** de qualquer agrupamento (`GROUP BY`). Atua sobre os dados brutos.
-   **`HAVING`:** Filtra os grupos de linhas **depois** do agrupamento e da aplicação de funções de agregação (`COUNT`, `SUM`, `AVG`, etc.). Atua sobre os dados já resumidos.

### Desafio 1
Listar as categorias de filmes cuja média de duração é superior a 120 minutos.

### Solução 1
```sql
SELECT
    c.name AS category_name,
    AVG(f.length) AS average_length
FROM category c
JOIN film_category fc ON c.category_id = fc.category_id
JOIN film f ON fc.film_id = f.film_id
GROUP BY c.name
HAVING AVG(f.length) > 120
ORDER BY average_length DESC;
```

### Desafio 2
Encontrar clientes que fizeram mais de 30 pagamentos **E** cujo total gasto foi superior a $150.

### Solução 2
```sql
SELECT
    c.first_name,
    c.last_name,
    COUNT(p.payment_id) AS total_pagamentos,
    SUM(p.amount) AS total_gasto
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
HAVING COUNT(p.payment_id) > 30 AND SUM(p.amount) > 150
ORDER BY total_gasto DESC;
```

---

## 13. Controle de Transações (`COMMIT` e `ROLLBACK`)

**Conceito:** Uma transação é um conjunto de operações SQL que devem ser executadas como uma unidade atômica. Ou todas são bem-sucedidas e salvas permanentemente (`COMMIT`), ou todas falham e são desfeitas (`ROLLBACK`), garantindo a integridade dos dados.

-   **`START TRANSACTION` ou `BEGIN`:** Inicia o bloco da transação.
-   **`COMMIT`:** Confirma e salva todas as alterações.
-   **`ROLLBACK`:** Desfaz todas as alterações desde o início da transação.

### Desafio 1
Simular um aluguel de filme (inserir em `rental` e atualizar `inventory`) dentro de uma transação.

### Solução 1
```sql
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

-- Cenário de Falha (ROLLBACK) - Exemplo conceitual
-- START TRANSACTION;
-- INSERT INTO rental (rental_date, inventory_id, customer_id, staff_id, return_date, last_update)
-- VALUES (NOW(), 99999, 1, 1, NULL, NOW()); -- inventory_id inválido causaria erro
-- ROLLBACK; -- A inserção seria desfeita
```

### Desafio 2
Simular a exclusão de um cliente e todos os seus registros relacionados (`payments` e `rentals`) usando uma transação.

### Solução 2
```sql
-- Cenário de Sucesso (COMMIT)
START TRANSACTION;

SET @customer_to_delete = 100; -- ID do cliente a ser apagado

DELETE FROM payment WHERE customer_id = @customer_to_delete;
DELETE FROM rental WHERE customer_id = @customer_to_delete;
DELETE FROM customer WHERE customer_id = @customer_to_delete;

COMMIT;
```

---

## 14. Gatilhos (Triggers)

**Conceito:** Um Trigger é um procedimento armazenado que é executado automaticamente em resposta a um evento `INSERT`, `UPDATE` ou `DELETE` em uma tabela específica. Útil para auditoria, validações complexas ou manter dados sincronizados.

### Desafio 1
Criar um `TRIGGER` que, ao atualizar a tabela `film`, registre o `film_id` e a data/hora da modificação em uma tabela de auditoria.

### Solução 1
```sql
-- 1. Crie a tabela de log
CREATE TABLE IF NOT EXISTS film_audit_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    film_id SMALLINT UNSIGNED NOT NULL,
    update_timestamp DATETIME NOT NULL,
    FOREIGN KEY (film_id) REFERENCES film(film_id) ON DELETE CASCADE
);

-- 2. Crie o TRIGGER
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

-- 3. Teste
-- UPDATE film SET rental_duration = 4 WHERE film_id = 1;
-- SELECT * FROM film_audit_log;
```

### Desafio 2
Criar um `TRIGGER` que, antes de inserir um novo aluguel, impeça a operação se o filme tiver a classificação 'NC-17', retornando uma mensagem de erro.

### Solução 2
```sql
DELIMITER //

CREATE TRIGGER before_rental_insert_check_rating
BEFORE INSERT ON rental
FOR EACH ROW
BEGIN
    DECLARE film_rating ENUM('G','PG','PG-13','R','NC-17');

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

-- Teste (exige um inventory_id de um filme NC-17)
-- INSERT INTO rental (rental_date, inventory_id, customer_id, staff_id, last_update)
-- VALUES (NOW(), (SELECT i.inventory_id FROM inventory i JOIN film f ON i.film_id = f.film_id WHERE f.rating = 'NC-17' LIMIT 1), 1, 1, NOW());
```

---

## 15. Índices (Indexes) e Otimização

**Conceito:** Um índice é uma estrutura de dados que melhora a velocidade das operações de busca em uma tabela. Em vez de percorrer a tabela inteira (full table scan), o banco de dados pode usar o índice para encontrar os registros rapidamente, como o índice de um livro.

### Desafio 1
Otimizar a busca de clientes por sobrenome (`last_name`).

#### Consulta a Otimizar
```sql
-- Esta consulta pode ser lenta em tabelas grandes sem um índice
SELECT * FROM customer WHERE last_name = 'SMITH';
```
#### Criação do Índice
```sql
-- Criar um índice na coluna last_name acelera a busca
CREATE INDEX idx_customer_last_name ON customer (last_name);
```
**Por que melhora?** Sem o índice, o banco de dados precisa ler cada linha da tabela `customer` para encontrar os 'SMITH'. Com o índice, ele vai diretamente para as localizações dos registros correspondentes, tornando a busca quase instantânea.

### Desafio 2
Otimizar uma busca que filtra aluguéis por `customer_id` e por um intervalo de `rental_date`.

#### Consulta a Otimizar
```sql
SELECT rental_id, inventory_id, return_date
FROM rental
WHERE customer_id = 100 AND rental_date BETWEEN '2005-05-01' AND '2005-06-30';
```
#### Criação do Índice Composto
```sql
-- Um índice em múltiplas colunas (composto) é ideal para esta query
CREATE INDEX idx_rental_customer_date ON rental (customer_id, rental_date);
```
**Por que melhora?** Um índice composto em `(customer_id, rental_date)` permite que o banco filtre eficientemente primeiro pelo cliente e, em seguida, pelo intervalo de datas dentro dos registros daquele cliente, otimizando drasticamente a consulta.

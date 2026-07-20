DROP DATABASE IF EXISTS controle_estoque_vendas;

CREATE DATABASE controle_estoque_vendas
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

USE controle_estoque_vendas;

-- =====================================================
-- TABELA DE CLIENTES
-- =====================================================

CREATE TABLE clientes (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    cpf CHAR(11) NOT NULL UNIQUE,
    email VARCHAR(150) UNIQUE,
    telefone VARCHAR(20),
    data_cadastro DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- =====================================================
-- TABELA DE FUNCIONÁRIOS
-- =====================================================

CREATE TABLE funcionarios (
    id_funcionario INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(120) NOT NULL,
    cargo VARCHAR(80) NOT NULL,
    salario DECIMAL(10,2) NOT NULL,
    data_admissao DATE NOT NULL,
    ativo BOOLEAN NOT NULL DEFAULT TRUE,

    CONSTRAINT chk_salario
        CHECK (salario >= 0)
);

-- =====================================================
-- TABELA DE CATEGORIAS
-- =====================================================

CREATE TABLE categorias (
    id_categoria INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(80) NOT NULL UNIQUE,
    descricao VARCHAR(255)
);

-- =====================================================
-- TABELA DE FORNECEDORES
-- =====================================================

CREATE TABLE fornecedores (
    id_fornecedor INT AUTO_INCREMENT PRIMARY KEY,
    razao_social VARCHAR(150) NOT NULL,
    nome_fantasia VARCHAR(120),
    cnpj CHAR(14) NOT NULL UNIQUE,
    email VARCHAR(150),
    telefone VARCHAR(20)
);

-- =====================================================
-- TABELA DE PRODUTOS
-- =====================================================

CREATE TABLE produtos (
    id_produto INT AUTO_INCREMENT PRIMARY KEY,
    id_categoria INT NOT NULL,
    nome VARCHAR(150) NOT NULL,
    descricao VARCHAR(255),
    preco_compra DECIMAL(10,2) NOT NULL,
    preco_venda DECIMAL(10,2) NOT NULL,
    estoque_atual INT NOT NULL DEFAULT 0,
    estoque_minimo INT NOT NULL DEFAULT 5,
    ativo BOOLEAN NOT NULL DEFAULT TRUE,

    CONSTRAINT fk_produto_categoria
        FOREIGN KEY (id_categoria)
        REFERENCES categorias(id_categoria),

    CONSTRAINT chk_preco_compra
        CHECK (preco_compra >= 0),

    CONSTRAINT chk_preco_venda
        CHECK (preco_venda >= 0),

    CONSTRAINT chk_estoque
        CHECK (estoque_atual >= 0)
);

-- =====================================================
-- TABELA DE ENTRADAS DE ESTOQUE
-- =====================================================

CREATE TABLE entradas_estoque (
    id_entrada INT AUTO_INCREMENT PRIMARY KEY,
    id_produto INT NOT NULL,
    id_fornecedor INT NOT NULL,
    quantidade INT NOT NULL,
    valor_unitario DECIMAL(10,2) NOT NULL,
    data_entrada DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    observacao VARCHAR(255),

    CONSTRAINT fk_entrada_produto
        FOREIGN KEY (id_produto)
        REFERENCES produtos(id_produto),

    CONSTRAINT fk_entrada_fornecedor
        FOREIGN KEY (id_fornecedor)
        REFERENCES fornecedores(id_fornecedor),

    CONSTRAINT chk_quantidade_entrada
        CHECK (quantidade > 0)
);

-- =====================================================
-- TABELA DE VENDAS
-- =====================================================

CREATE TABLE vendas (
    id_venda INT AUTO_INCREMENT PRIMARY KEY,
    id_cliente INT,
    id_funcionario INT NOT NULL,
    data_venda DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    valor_total DECIMAL(12,2) NOT NULL DEFAULT 0,
    status ENUM(
        'ABERTA',
        'FINALIZADA',
        'CANCELADA'
    ) NOT NULL DEFAULT 'ABERTA',

    CONSTRAINT fk_venda_cliente
        FOREIGN KEY (id_cliente)
        REFERENCES clientes(id_cliente),

    CONSTRAINT fk_venda_funcionario
        FOREIGN KEY (id_funcionario)
        REFERENCES funcionarios(id_funcionario)
);

-- =====================================================
-- TABELA DE ITENS DA VENDA
-- =====================================================

CREATE TABLE itens_venda (
    id_item INT AUTO_INCREMENT PRIMARY KEY,
    id_venda INT NOT NULL,
    id_produto INT NOT NULL,
    quantidade INT NOT NULL,
    preco_unitario DECIMAL(10,2) NOT NULL,

    subtotal DECIMAL(12,2)
        GENERATED ALWAYS AS (
            quantidade * preco_unitario
        ) STORED,

    CONSTRAINT fk_item_venda
        FOREIGN KEY (id_venda)
        REFERENCES vendas(id_venda)
        ON DELETE CASCADE,

    CONSTRAINT fk_item_produto
        FOREIGN KEY (id_produto)
        REFERENCES produtos(id_produto),

    CONSTRAINT chk_quantidade_item
        CHECK (quantidade > 0)
);

-- =====================================================
-- TABELA DE PAGAMENTOS
-- =====================================================

CREATE TABLE pagamentos (
    id_pagamento INT AUTO_INCREMENT PRIMARY KEY,
    id_venda INT NOT NULL,

    forma_pagamento ENUM(
        'DINHEIRO',
        'PIX',
        'CARTAO_CREDITO',
        'CARTAO_DEBITO',
        'BOLETO'
    ) NOT NULL,

    valor_pago DECIMAL(12,2) NOT NULL,
    data_pagamento DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_pagamento_venda
        FOREIGN KEY (id_venda)
        REFERENCES vendas(id_venda),

    CONSTRAINT chk_valor_pagamento
        CHECK (valor_pago > 0)
);

-- =====================================================
-- ÍNDICES
-- =====================================================

CREATE INDEX idx_produto_nome
ON produtos(nome);

CREATE INDEX idx_venda_data
ON vendas(data_venda);

CREATE INDEX idx_venda_cliente
ON vendas(id_cliente);

CREATE INDEX idx_item_produto
ON itens_venda(id_produto);

-- =====================================================
-- INSERÇÃO DE CATEGORIAS
-- =====================================================

INSERT INTO categorias (nome, descricao)
VALUES
('Informática', 'Computadores e acessórios'),
('Escritório', 'Materiais para escritório'),
('Eletrônicos', 'Equipamentos eletrônicos'),
('Limpeza', 'Produtos de limpeza');

-- =====================================================
-- INSERÇÃO DE CLIENTES
-- =====================================================

INSERT INTO clientes (
    nome,
    cpf,
    email,
    telefone
)
VALUES
(
    'Ana Beatriz Souza',
    '12345678901',
    'ana@email.com',
    '61999990001'
),
(
    'Bruno Henrique Lima',
    '12345678902',
    'bruno@email.com',
    '61999990002'
),
(
    'Carla Mendes Rocha',
    '12345678903',
    'carla@email.com',
    '61999990003'
);

-- =====================================================
-- INSERÇÃO DE FUNCIONÁRIOS
-- =====================================================

INSERT INTO funcionarios (
    nome,
    cargo,
    salario,
    data_admissao
)
VALUES
(
    'Marcos Paulo Ribeiro',
    'Vendedor',
    2500.00,
    '2025-01-10'
),
(
    'Juliana Ferreira Alves',
    'Gerente',
    4500.00,
    '2024-07-15'
),
(
    'Rafael Gomes Santos',
    'Vendedor',
    2600.00,
    '2025-03-01'
);

-- =====================================================
-- INSERÇÃO DE FORNECEDORES
-- =====================================================

INSERT INTO fornecedores (
    razao_social,
    nome_fantasia,
    cnpj,
    email,
    telefone
)
VALUES
(
    'Tech Distribuidora Ltda',
    'Tech Distribuidora',
    '11222333000101',
    'contato@tech.com',
    '6133331001'
),
(
    'Papelaria Central Ltda',
    'Papelaria Central',
    '11222333000102',
    'vendas@papelaria.com',
    '6133331002'
);

-- =====================================================
-- INSERÇÃO DE PRODUTOS
-- =====================================================

INSERT INTO produtos (
    id_categoria,
    nome,
    descricao,
    preco_compra,
    preco_venda,
    estoque_atual,
    estoque_minimo
)
VALUES
(
    1,
    'Mouse sem fio',
    'Mouse óptico USB sem fio',
    45.00,
    79.90,
    30,
    10
),
(
    1,
    'Teclado mecânico',
    'Teclado mecânico padrão ABNT2',
    180.00,
    289.90,
    15,
    5
),
(
    1,
    'Monitor 24 polegadas',
    'Monitor LED Full HD',
    650.00,
    899.90,
    8,
    3
),
(
    2,
    'Papel A4',
    'Resma com 500 folhas',
    22.00,
    34.90,
    50,
    15
),
(
    2,
    'Caneta azul',
    'Caneta esferográfica',
    1.20,
    2.50,
    100,
    30
);

-- =====================================================
-- TRIGGER PARA VALIDAR ESTOQUE
-- =====================================================

DELIMITER $$

CREATE TRIGGER trg_validar_estoque
BEFORE INSERT ON itens_venda
FOR EACH ROW
BEGIN

    DECLARE estoque_disponivel INT;

    SELECT estoque_atual
    INTO estoque_disponivel
    FROM produtos
    WHERE id_produto = NEW.id_produto;

    IF estoque_disponivel < NEW.quantidade THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT =
        'Estoque insuficiente para realizar a venda';

    END IF;

END$$

-- =====================================================
-- TRIGGER PARA BAIXAR ESTOQUE
-- =====================================================

CREATE TRIGGER trg_baixar_estoque
AFTER INSERT ON itens_venda
FOR EACH ROW
BEGIN

    UPDATE produtos
    SET estoque_atual =
        estoque_atual - NEW.quantidade
    WHERE id_produto = NEW.id_produto;

    UPDATE vendas
    SET valor_total = (

        SELECT COALESCE(SUM(subtotal), 0)
        FROM itens_venda
        WHERE id_venda = NEW.id_venda

    )
    WHERE id_venda = NEW.id_venda;

END$$

-- =====================================================
-- TRIGGER PARA DEVOLVER ESTOQUE
-- =====================================================

CREATE TRIGGER trg_devolver_estoque
AFTER DELETE ON itens_venda
FOR EACH ROW
BEGIN

    UPDATE produtos
    SET estoque_atual =
        estoque_atual + OLD.quantidade
    WHERE id_produto = OLD.id_produto;

    UPDATE vendas
    SET valor_total = (

        SELECT COALESCE(SUM(subtotal), 0)
        FROM itens_venda
        WHERE id_venda = OLD.id_venda

    )
    WHERE id_venda = OLD.id_venda;

END$$

-- =====================================================
-- TRIGGER PARA ENTRADA DE ESTOQUE
-- =====================================================

CREATE TRIGGER trg_entrada_estoque
AFTER INSERT ON entradas_estoque
FOR EACH ROW
BEGIN

    UPDATE produtos
    SET
        estoque_atual =
            estoque_atual + NEW.quantidade,

        preco_compra =
            NEW.valor_unitario

    WHERE id_produto = NEW.id_produto;

END$$

DELIMITER ;

-- =====================================================
-- PROCEDURE PARA CRIAR VENDA
-- =====================================================

DELIMITER $$

CREATE PROCEDURE sp_criar_venda (
    IN p_id_cliente INT,
    IN p_id_funcionario INT,
    OUT p_id_venda INT
)
BEGIN

    INSERT INTO vendas (
        id_cliente,
        id_funcionario,
        status
    )
    VALUES (
        p_id_cliente,
        p_id_funcionario,
        'ABERTA'
    );

    SET p_id_venda = LAST_INSERT_ID();

END$$

-- =====================================================
-- PROCEDURE PARA ADICIONAR ITEM
-- =====================================================

CREATE PROCEDURE sp_adicionar_item (
    IN p_id_venda INT,
    IN p_id_produto INT,
    IN p_quantidade INT
)
BEGIN

    DECLARE v_preco DECIMAL(10,2);
    DECLARE v_status VARCHAR(20);

    SELECT status
    INTO v_status
    FROM vendas
    WHERE id_venda = p_id_venda;

    IF v_status IS NULL THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Venda não encontrada';

    END IF;

    IF v_status <> 'ABERTA' THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT =
        'Apenas vendas abertas podem receber itens';

    END IF;

    SELECT preco_venda
    INTO v_preco
    FROM produtos
    WHERE id_produto = p_id_produto
    AND ativo = TRUE;

    INSERT INTO itens_venda (
        id_venda,
        id_produto,
        quantidade,
        preco_unitario
    )
    VALUES (
        p_id_venda,
        p_id_produto,
        p_quantidade,
        v_preco
    );

END$$

-- =====================================================
-- PROCEDURE PARA FINALIZAR VENDA
-- =====================================================

CREATE PROCEDURE sp_finalizar_venda (
    IN p_id_venda INT,
    IN p_forma_pagamento VARCHAR(30)
)
BEGIN

    DECLARE v_total DECIMAL(12,2);
    DECLARE v_status VARCHAR(20);

    SELECT
        valor_total,
        status
    INTO
        v_total,
        v_status
    FROM vendas
    WHERE id_venda = p_id_venda;

    IF v_status <> 'ABERTA' THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT =
        'A venda não está aberta';

    END IF;

    IF v_total <= 0 THEN

        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT =
        'A venda não possui produtos';

    END IF;

    INSERT INTO pagamentos (
        id_venda,
        forma_pagamento,
        valor_pago
    )
    VALUES (
        p_id_venda,
        p_forma_pagamento,
        v_total
    );

    UPDATE vendas
    SET status = 'FINALIZADA'
    WHERE id_venda = p_id_venda;

END$$

DELIMITER ;

-- =====================================================
-- VIEWS
-- =====================================================

CREATE VIEW vw_produtos_estoque_baixo AS
SELECT
    p.id_produto,
    p.nome AS produto,
    c.nome AS categoria,
    p.estoque_atual,
    p.estoque_minimo
FROM produtos p
INNER JOIN categorias c
ON c.id_categoria = p.id_categoria
WHERE p.estoque_atual <= p.estoque_minimo;

CREATE VIEW vw_resumo_vendas AS
SELECT
    v.id_venda,
    v.data_venda,
    COALESCE(
        c.nome,
        'Consumidor não identificado'
    ) AS cliente,
    f.nome AS funcionario,
    v.valor_total,
    v.status
FROM vendas v
LEFT JOIN clientes c
ON c.id_cliente = v.id_cliente
INNER JOIN funcionarios f
ON f.id_funcionario = v.id_funcionario;

CREATE VIEW vw_faturamento_mensal AS
SELECT
    YEAR(data_venda) AS ano,
    MONTH(data_venda) AS mes,
    COUNT(id_venda) AS quantidade_vendas,
    SUM(valor_total) AS faturamento
FROM vendas
WHERE status = 'FINALIZADA'
GROUP BY
    YEAR(data_venda),
    MONTH(data_venda);

-- =====================================================
-- EXEMPLO DE CRIAÇÃO DE VENDA
-- =====================================================

CALL sp_criar_venda(
    1,
    1,
    @id_nova_venda
);

SELECT @id_nova_venda;

CALL sp_adicionar_item(
    @id_nova_venda,
    1,
    2
);

CALL sp_adicionar_item(
    @id_nova_venda,
    4,
    1
);

CALL sp_finalizar_venda(
    @id_nova_venda,
    'PIX'
);

-- =====================================================
-- CONSULTAS PARA DEMONSTRAÇÃO
-- =====================================================

-- Listar produtos e categorias

SELECT
    p.id_produto,
    p.nome AS produto,
    c.nome AS categoria,
    p.preco_venda,
    p.estoque_atual
FROM produtos p
INNER JOIN categorias c
ON c.id_categoria = p.id_categoria
ORDER BY p.nome;

-- Mostrar todas as vendas

SELECT *
FROM vw_resumo_vendas;

-- Mostrar detalhes das vendas

SELECT
    v.id_venda,
    v.data_venda,
    c.nome AS cliente,
    f.nome AS funcionario,
    p.nome AS produto,
    iv.quantidade,
    iv.preco_unitario,
    iv.subtotal
FROM vendas v
LEFT JOIN clientes c
ON c.id_cliente = v.id_cliente
INNER JOIN funcionarios f
ON f.id_funcionario = v.id_funcionario
INNER JOIN itens_venda iv
ON iv.id_venda = v.id_venda
INNER JOIN produtos p
ON p.id_produto = iv.id_produto
ORDER BY v.id_venda;

-- Produtos mais vendidos

SELECT
    p.nome AS produto,
    SUM(iv.quantidade) AS quantidade_vendida,
    SUM(iv.subtotal) AS total_vendido
FROM itens_venda iv
INNER JOIN produtos p
ON p.id_produto = iv.id_produto
INNER JOIN vendas v
ON v.id_venda = iv.id_venda
WHERE v.status = 'FINALIZADA'
GROUP BY
    p.id_produto,
    p.nome
ORDER BY quantidade_vendida DESC;

-- Faturamento por mês

SELECT *
FROM vw_faturamento_mensal
ORDER BY ano, mes;

-- Produtos com estoque baixo

SELECT *
FROM vw_produtos_estoque_baixo;

-- Ticket médio

SELECT
    ROUND(
        AVG(valor_total),
        2
    ) AS ticket_medio
FROM vendas
WHERE status = 'FINALIZADA';

-- Total vendido por funcionário

SELECT
    f.nome AS funcionario,
    COUNT(v.id_venda) AS quantidade_vendas,
    COALESCE(
        SUM(v.valor_total),
        0
    ) AS total_vendido
FROM funcionarios f
LEFT JOIN vendas v
ON v.id_funcionario = f.id_funcionario
AND v.status = 'FINALIZADA'
GROUP BY
    f.id_funcionario,
    f.nome
ORDER BY total_vendido DESC;

-- Clientes que mais compraram

SELECT
    c.nome AS cliente,
    COUNT(v.id_venda) AS quantidade_compras,
    SUM(v.valor_total) AS total_gasto
FROM clientes c
INNER JOIN vendas v
ON v.id_cliente = c.id_cliente
WHERE v.status = 'FINALIZADA'
GROUP BY
    c.id_cliente,
    c.nome
ORDER BY total_gasto DESC;

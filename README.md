# biblioteca

## Tabela de Livros
```
CREATE TABLE Livros (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Titulo VARCHAR(255) NOT NULL,
    ISBN VARCHAR(13) NOT NULL,
    AnoPublicacao INT,
    EditoraID INT,
    FOREIGN KEY (EditoraID) REFERENCES Editoras(ID)
);
```
## Tabela de Autores
```
CREATE TABLE Autores (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Nome VARCHAR(255) NOT NULL,
    DataNascimento DATE,
    Nacionalidade VARCHAR(255)
);
```
## Tabela de Editoras
```
CREATE TABLE Editoras (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Nome VARCHAR(255) NOT NULL,
    Endereco VARCHAR(255)
);
```
## Tabela de Empréstimos
```
CREATE TABLE Emprestimos (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    DataEmprestimo DATE NOT NULL,
    DataDevolucaoDeterminada DATE NOT NULL,
    DataDevolucaoCliente DATE,
    StatusEmprestimo ENUM('pendente', 'devolvido', 'atrasado') NOT NULL,
    LivroID INT,
    ClienteID INT,
    FOREIGN KEY (LivroID) REFERENCES Livros(ID),
    FOREIGN KEY (ClienteID) REFERENCES Clientes(ID)
);
```
## Tabela de Clientes
```
CREATE TABLE Clientes (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Nome VARCHAR(255) NOT NULL,
    Idade INT
);
```
## Tabela de Relacionamentos 
```
CREATE TABLE LivrosAutores (
    LivroID INT,
    AutorID INT,
    PRIMARY KEY (LivroID, AutorID),
    FOREIGN KEY (LivroID) REFERENCES Livros(ID),
    FOREIGN KEY (AutorID) REFERENCES Autores(ID)
);
```

## Stored Procedure para registrar um novo empréstimo
```
DELIMITER //
CREATE PROCEDURE RegistrarEmprestimo (
    IN LivroID INT,
    IN ClienteID INT,
    IN DataDevolucaoDeterminada DATE
)
BEGIN
    DECLARE LivroDisponivel BOOLEAN;
    
    SET LivroDisponivel = FALSE;

    SELECT 1 INTO LivroDisponivel
    FROM LivrosDisponiveis
    WHERE ID = LivroID;
    
    IF LivroDisponivel THEN
        INSERT INTO Emprestimos (DataEmprestimo, DataDevolucaoDeterminada, StatusEmprestimo, LivroID, ClienteID)
        VALUES (CURDATE(), DataDevolucaoDeterminada, 'pendente', LivroID, ClienteID);
        SELECT 'Empréstimo registrado com sucesso.' AS Resultado;
    ELSE
        SELECT 'O livro não está disponível para empréstimo.' AS Resultado;
    END IF;
END;
//
DELIMITER ;
```

## View para mostrar livros disponíveis para empréstimos 
```
CREATE VIEW LivrosDisponiveis AS
SELECT L.ID, L.Titulo, L.ISBN, L.AnoPublicacao
FROM Livros L
LEFT JOIN Emprestimos E ON L.ID = E.LivroID
WHERE E.ID IS NULL OR (E.StatusEmprestimo = 'devolvido' AND L.ID NOT IN (SELECT LivroID FROM Emprestimos WHERE StatusEmprestimo = 'pendente' OR 'atrasado'));
```


## View para fornecer lista de todos os empréstimos atuais 
```
CREATE VIEW ListaEmprestimos AS
SELECT E.ID AS EmprestimoID, C.Nome AS Cliente, L.Titulo AS Livro, E.DataEmprestimo, E.DataDevolucaoDeterminada, E.DataDevolucaoCliente, E.StatusEmprestimo
FROM Emprestimos E
JOIN Clientes C ON E.ClienteID = C.ID
JOIN Livros L ON E.LivroID = L.ID;
```

## Trigger para definir o status do empréstimo no momento do registro
```
DELIMITER //
CREATE TRIGGER UpdateStatusAtrasado
BEFORE INSERT ON Emprestimos
FOR EACH ROW
BEGIN
    IF NEW.StatusEmprestimo = 'pendente' AND NEW.StatusEmprestimo != 'devolvido' AND NEW.DataDevolucaoDeterminada < CURDATE() THEN
        SET NEW.StatusEmprestimo = 'atrasado';
    END IF;
END;
//
DELIMITER ;
```

## Inserindo dados
```
INSERT INTO Livros (Titulo, ISBN, AnoPublicacao, EditoraID)
VALUES
    ('Livro 1', 'ISBN1', 1999, 1),
    ('Livro 2', 'ISBN2', 1540, 2),
    ('Livro 3', 'ISBN3', 2022, 3),
    ('Livro 4', 'ISBN4', 2007, 4),
    ('Livro 5', 'ISBN5', 2016, 5),
    ('Livro 6', 'ISBN6', 1220, 6),
    ('Livro 7', 'ISBN7', 1210, 7),
    ('Livro 8', 'ISBN8', 2018, 8),
    ('Livro 9', 'ISBN9', 2012, 9),
    ('Livro 10', 'ISBN10', 2011, 10),
    ('Livro 11', 'ISBN11', 2010, 11),
    ('Livro 12', 'ISBN12', 2009, 12),
    ('Livro 13', 'ISBN13', 2008, 13),
    ('Livro 14', 'ISBN14', 2007, 14),
    ('Livro 15', 'ISBN15', 2006, 15),
    ('Livro 16', 'ISBN16', 2005, 16),
    ('Livro 17', 'ISBN17', 2004, 17),
    ('Livro 18', 'ISBN18', 2000, 18),
    ('Livro 19', 'ISBN19', 2022, 19),
    ('Livro 20', 'ISBN20', 2023, 20),
    ('Livro 21', 'ISBN21', 2011, 1),
    ('Livro 22', 'ISBN22', 2010, 2),
    ('Livro 23', 'ISBN23', 2009, 3),
    ('Livro 24', 'ISBN24', 2008, 4),
    ('Livro 25', 'ISBN25', 2007, 5),
    ('Livro 26', 'ISBN26', 2006, 6),
    ('Livro 27', 'ISBN27', 2005, 7),
    ('Livro 28', 'ISBN28', 2004, 8),
    ('Livro 29', 'ISBN29', 2000, 9),
    ('Livro 30', 'ISBN30', 2022, 10);

INSERT INTO Autores (Nome, DataNascimento, Nacionalidade)
VALUES
    ('Autor 1', '1510-01-15', 'Brasileiro'),
    ('Autor 2', '1180-03-20', 'Americano'),
    ('Autor 3', '1182-05-10', 'Francês'),
    ('Autor 4', '1990-11-30', 'Alemão'),
    ('Autor 5', '1973-07-05', 'Inglês'),
    ('Autor 6', '1988-09-25', 'Italiano'),
    ('Autor 7', '1987-04-12', 'Espanhol'),
    ('Autor 8', '1985-12-07', 'Canadense'),
    ('Autor 9', '1978-03-28', 'Holandês'),
    ('Autor 10', '1992-06-14', 'Sueco'),
    ('Autor 11', '1970-08-10', 'Japonês'),
    ('Autor 12', '1981-09-02', 'Coreano'),
    ('Autor 13', '1995-04-18', 'Russo'),
    ('Autor 14', '1979-02-24', 'Chinês'),
    ('Autor 15', '1977-11-03', 'Mexicano'),
    ('Autor 16', '1986-06-29', 'Argentino'),
    ('Autor 17', '1974-10-20', 'Chileno'),
    ('Autor 18', '1993-07-15', 'Australiano'),
    ('Autor 19', '1984-03-06', 'Sul-Africano'),
    ('Autor 20', '1989-12-22', 'Indiano');

INSERT INTO Editoras (Nome, Endereco)
VALUES
    ('Editora X', 'Rua X, Cidade X'),
    ('Editora Y', 'Rua Y, Cidade Y'),
    ('Editora Z', 'Rua Z, Cidade Z'),
    ('Editora A', 'Rua A, Cidade A'),
    ('Editora B', 'Rua B, Cidade B'),
    ('Editora C', 'Rua C, Cidade C'),
    ('Editora D', 'Rua D, Cidade D'),
    ('Editora E', 'Rua E, Cidade E'),
    ('Editora F', 'Rua F, Cidade F'),
    ('Editora G', 'Rua G, Cidade G'),
    ('Editora H', 'Rua H, Cidade H'),
    ('Editora I', 'Rua I, Cidade I'),
    ('Editora J', 'Rua J, Cidade J'),
    ('Editora K', 'Rua K, Cidade K'),
    ('Editora L', 'Rua L, Cidade L'),
    ('Editora M', 'Rua M, Cidade M'),
    ('Editora N', 'Rua N, Cidade N'),
    ('Editora O', 'Rua O, Cidade O'),
    ('Editora P', 'Rua P, Cidade P'),
    ('Editora Q', 'Rua Q, Cidade Q');

INSERT INTO Clientes (Nome, Idade)
VALUES
    ('Cliente 1', 30),
    ('Cliente 2', 25),
    ('Cliente 3', 35),
    ('Cliente 4', 28),
    ('Cliente 5', 40),
    ('Cliente 6', 22),
    ('Cliente 7', 33),
    ('Cliente 8', 27),
    ('Cliente 9', 31),
    ('Cliente 10', 26),
    ('Cliente 11', 29),
    ('Cliente 12', 38),
    ('Cliente 13', 24),
    ('Cliente 14', 37),
    ('Cliente 15', 32),
    ('Cliente 16', 34),
    ('Cliente 17', 23),
    ('Cliente 18', 36),
    ('Cliente 19', 39),
    ('Cliente 20', 28);


INSERT INTO LivrosAutores (LivroID, AutorID)
VALUES
    (1, 5),
    (2, 1),
    (3, 6),
    (4, 4),
    (5, 7),
    (6, 3),
    (7, 2),
    (8, 8),
    (9, 9),
    (10, 10),
    (11, 11),
    (12, 12),
    (13, 13),
    (14, 14),
    (15, 15),
    (16, 16),
    (17, 17),
    (18, 18),
    (19, 19),
    (20, 20);


INSERT INTO Emprestimos (DataEmprestimo, DataDevolucaoDeterminada, StatusEmprestimo, LivroID, ClienteID)
VALUES
    ('2023-01-05', '2023-01-19', "devolvido", 1, 1),
    ('2023-02-10', '2023-02-24', "devolvido", 2, 2),
    ('2023-03-15', '2023-03-29', "pendente", 3, 3),
    ('2023-04-20', '2023-05-04', "pendente", 4, 4),
    ('2023-05-25', '2023-06-08', "pendente", 5, 5),
    ('2023-06-30', '2023-07-14', "pendente", 6, 6),
    ('2023-07-05', '2023-07-19', "pendente", 7, 7),
    ('2023-08-10', '2023-08-24', "pendente", 8, 8),
    ('2023-09-15', '2023-09-29', "pendente", 9, 9),
    ('2023-10-20', '2023-11-03', "pendente", 10, 10),
    ('2023-11-25', '2023-12-09', "pendente", 11, 11),
    ('2023-12-30', '2024-01-13', "pendente", 12, 12),
    ('2024-01-05', '2024-01-19', "pendente", 13, 13),
    ('2024-02-10', '2024-02-24', "pendente", 14, 14),
    ('2024-03-15', '2024-03-29', "pendente", 15, 15),
    ('2024-04-20', '2024-05-04', "pendente", 16, 16),
    ('2024-05-25', '2024-06-08', "pendente", 17, 17),
    ('2024-06-30', '2024-07-14', "pendente", 18, 18),
    ('2024-07-05', '2024-07-19', "pendente", 19, 19),
    ('2024-08-10', '2024-08-24',  "pendente",20, 20);
```
## Registrando empréstimos com Stored Procedure
```
call RegistrarEmprestimo(25,1, "2024-03-20");
call RegistrarEmprestimo(26, 2, "2024-03-20")
```

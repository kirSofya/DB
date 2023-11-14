# Задание №6 
### Кирьянова Софья, 214 
 
## Задача 1 

* Показать все названия книг вместе с именами издателей:
```sql
SELECT Book.Title, Publisher.PubName  
FROM Book  
JOIN Publisher ON Book.PubName = Publisher.PubName;  
```
* В какой книге наибольшее количество страниц?
```sql
SELECT Title  
FROM Book  
WHERE PagesNum = (SELECT MAX(PagesNum) FROM Book);  
```
* Какие авторы написали более 5 книг?
```sql
SELECT Author  
FROM Book  
GROUP BY Author  
HAVING COUNT(*) > 5;
```
* В каких книгах более чем в два раза больше страниц, чем среднее количество страниц для всех книг?
``` sql
SELECT Title  
FROM Book  
WHERE PagesNum > (SELECT AVG(PagesNum) * 2 FROM Book);
```  
* Какие категории содержат подкатегории?
```sql
SELECT DISTINCT c.CategoryName  
FROM Category c  
INNER JOIN Category p ON c.ParentCat = p.CategoryName;
```
* У какого автора (предположим, что имена авторов уникальны) написано максимальное количество книг?
```sql
SELECT Author  
FROM Book  
GROUP BY Author  
HAVING COUNT(*) = (SELECT MAX(BookCount) FROM (SELECT Author, COUNT(*) AS BookCount FROM Book GROUP BY Author) AS Counts);
```
* Какие читатели забронировали все книги (не копии), написанные "Марком Твеном"?
```sql
SELECT r.FirstName, r.LastName
FROM Reader r
JOIN Borrowing b ON r.number = b.ReaderNr
JOIN Book bk ON bk.isbn = b.ISBN
WHERE bk.Author = 'Марк Твен' AND b.ReturnDate IS NULL
GROUP BY r.FirstName, r.LastName
HAVING COUNT(DISTINCT bk.isbn) = (
    SELECT COUNT(DISTINCT isbn)
    FROM Book
    WHERE Author = 'Марк Твен'
)
```
* Какие книги имеют более одной копии?
```sql
SELECT b.Title  
FROM Book b  
INNER JOIN Copy c ON b.ISBN = c.ISBN  
GROUP BY b.Title  
HAVING COUNT(c.CopyNumber) > 1;
```
* ТОП 10 самых старых книг
```sql
SELECT Book.isbn  
FROM Book  
ORDER BY Book.PubYear  
LIMIT 10
``` 
* Перечислите все категории в категории “Спорт” (с любым уровнем вложености).
```sql
WITH RECURSIVE CategoryHierarchy (CategoryName) AS (  
  SELECT CategoryName FROM Category WHERE ParentCat = 'Спорт'  
  UNION ALL  
  SELECT c.CategoryName FROM Category c INNER JOIN CategoryHierarchy h ON c.ParentCat = h.CategoryName  
)  
SELECT CategoryName  
FROM CategoryHierarchy;
```
  
## Задача 2  
  
* Добавьте запись о бронировании читателем ‘Василеем Петровым’ книги с ISBN 123456 и номером копии 4.
```sql
INSERT INTO Borrowing (ReaderNr, ISBN, CopyNumber)  
VALUES ((SELECT number FROM Reader WHERE FirstName = 'Василий' AND LastName = 'Петров'), '123456', 4);
```
* Удалить все книги, год публикации которых превышает 2000 год.
```sql
DELETE FROM Book  
WHERE PubYear > 2000;
```
* Измените дату возврата для всех книг категории "Базы данных", начиная с 01.01.2016, чтобы они были в заимствовании на 30 дней дольше (предположим, что в SQL можно добавлять числа к датам).  
```sql
UPDATE Borrowing  
SET ReturnDate = ReturnDate + INTERVAL '30' DAY  
WHERE ISBN IN (SELECT ISBN FROM BookCat WHERE CategoryName = 'Базы данных')  
AND ReturnDate >= DATE '2016-01-01';
```  

## Задача 3  
  
1.
```sql
SELECT s.Name, s.MatrNr FROM Student s  
WHERE NOT EXISTS (  
SELECT * FROM Check c WHERE c.MatrNr = s.MatrNr AND c.Note >= 4.0 );
```
Этот запрос выберет имена и номера студентов, у которых оценка (Note) ниже 4.  
  
2.   
```sql
( SELECT p.ProfNr, p.Name, sum(lec.Credit)  
FROM Professor p, Lecture lec  
WHERE p.ProfNr = lec.ProfNr  
GROUP BY p.ProfNr, p.Name)  
UNION  
( SELECT p.ProfNr, p.Name, 0  
FROM Professor p  
WHERE NOT EXISTS (  
SELECT * FROM Lecture lec WHERE lec.ProfNr = p.ProfNr ));
```
Этот запрос выведет сумму кредитов за все лекции профессора. Если профессор не ведет лекции, будет выведен 0.   
  
3.   
```sql
SELECT s.Name, p.Note   
FROM Student s, Lecture lec, Check c  
WHERE s.MatrNr = c.MatrNr AND lec.LectNr = c.LectNr AND c.Note >= 4  
AND c.Note >= ALL (  
SELECT c1.Note FROM Check c1 WHERE c1.MatrNr = c.MatrNr )
```
Этот запрос выводит максимальную оценку для студентов, у которых нет оценок ниже 4.   

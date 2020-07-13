# eCentric SQL Assessment


## Section 1


### Question 1.1

    CREATE TABLE Employees (
	employeeNo INT IDENTITY(1, 1) PRIMARY KEY NOT NULL
	,lastName VARCHAR(50) NOT NULL
	,firstName VARCHAR(50) NOT NULL
	,gender CHAR(1) NOT NULL
	,IDNumber VARCHAR(20) NOT NULL
	,salaryLevelID INT
	,departmentID INT NOT NULL REFERENCES Department(departmentID)
	);

### Question 1.2

    ALTER TABLE Employees
    ALTER COLUMN lastName VARCHAR(60) NOT NULL;

### Question 1.3

    CREATE PROCEDURE dbo.pr_getEmployeeDetails @departmentName VARCHAR(50)
    AS
    BEGIN
	IF @departmentName IS NULL
		OR @departmentName = ''
	BEGIN
		SELECT *
		FROM Employees
	END
	ELSE
	BEGIN
		SELECT *
		FROM Employees E
		JOIN Department D ON D.departmentID = E.departmentID
			AND D.name = @departmentName
	END
	END;
	GO

### Question 1.4

    CREATE PROCEDURE dbo.pr_getDepartmentEmployeeCount
    AS
    BEGIN
	SELECT D.name AS DepartmentName
		,count(E.departmentID) AS EmployeeCount
	FROM Department D
	LEFT JOIN Employees E ON E.departmentID = D.departmentID
	GROUP BY D.name
	ORDER BY EmployeeCount;
	END;
	GO

### Question 1.5

    CREATE PROCEDURE dbo.pr_UpdateEmployeeSalary @departmentName VARCHAR(50)
    AS
    BEGIN
	UPDATE SalaryLevel
	SET SalaryLevel.amount = (S.amount + (S.amount * (S.IncreasePercentage) / 100))
	FROM Employees E
	JOIN Department D ON D.departmentID = E.departmentID
	JOIN SalaryLevel S ON S.salaryLevelID = E.salaryLevelID
	WHERE D.name = @departmentName
	END;
	GO

### Question 1.6

    We might update salary percentages for employees not associated with the chosen department.

### Question 1.7

    CREATE PROCEDURE dbo.pr_AddEmployee @lastName VARCHAR(50) /* NOT NULL */
	,@firstName VARCHAR(50) /* NOT NULL */
	,@gender CHAR(1) /* NOT NULL */
	,@idNumber VARCHAR(20) /* NOT NULL */
	,@departmentId INT /* NOT NULL */
	,@salaryLevelId INT
	AS
	BEGIN
	INSERT INTO Employees (
		lastName
		,firstName
		,gender
		,IDNumber
		,epartmentId
		,salaryLevelID
		)
	VALUES (
		@lastName
		,@firstName
		,UPPER(@gender)
		,@IDNumber
		,@departmentId
		,@salaryLevelID
		);
	END;


### Question 1.8

    CREATE PROCEDURE dbo.pr_DepartmentGenderCount @departmentName VARCHAR(50)
    AS
    BEGIN
	SELECT D.name
		,SUM(CASE 
				WHEN E.gender = 'M'
					THEN 1
				ELSE 0
				END) AS MaleCount
		,SUM(CASE 
				WHEN E.gender = 'F'
					THEN 1
				ELSE 0
				END) AS FemaleCount
	FROM Department D
	LEFT JOIN Employees E ON D.departmentID = E.departmentID
	WHERE D.name = @departmentName
	GROUP BY D.name;
	END;
	GO

### Question 1.9

    CREATE PROCEDURE dbo.pr_GetEmployeeSalaryandDepartment @employeeNo INT
    AS
    BEGIN
	SELECT E.employeeNo
		,E.firstName
		,(D.name) AS departmentName
		,(S.amount) AS SalaryAmount
	FROM Employees E
	JOIN Department D ON D.departmentID = E.departmentID
	JOIN SalaryLevel S ON S.salaryLevelID = E.salaryLevelID
	WHERE E.employeeNo = @employeeNo;
	END;
	GO

### Question 1.10

    CREATE PROCEDURE dbo.pr_GetSalaryLevel
    AS
    BEGIN
	SELECT E.employeeNo
	,E.lastName
	,E.salaryLevelID
	FROM Employees E
	LEFT JOIN SalaryLevel S ON S.salaryLevelID = E.salaryLevelID
	WHERE S.salaryLevelID IS NULL
	END


## Section 2

### Question 2.1.1

    Index on Employee, LastName column
    Index on Employee, IDNumber

### Question 2.1.2

    They decrease performance on inserts, updates and deletes
    They take up additiomal disk space
   

### Question 2.2

    Although similar, ISNULL and COALESCE can behave differently. 
    ISNULL is a function, can be evaluated only once and can accept only two parameters. 
    COALESCE is an expression, can be evaluated multiple times and can accept multiple parameters.
    These both handle data types differently.
    The datatype of an ISNULL function is the datatype of its first input. If this
    is NULL, the datatype of the result is the type of the second input. If this is
    also NULL then the type of the output is an INT. The ISNULL return value is
    awlays considered not nullable.
    Whereas COALESCE follows a case expression and returns a datatype with the
    highest precedence. If all inputs are NULL, then an error is returned.

### Question 2.3

    This is not valid because you are insering an ID value into an Identity column which are automatically added on insert. 
    If you wanted to manual enter an ID value you would need to set IDENTITY_INSERT value to ON beforehand.

### Question 2.4

    We use a relational table structure here in order to remove duplication and normalize the data. 
    For example the Employees table can have duplicated entries for Departments. Instead of allowing us to insert duplicate department names per Employee, we split up Employee and Department data and add a relationship between the two tables.
    This means the Employees table does not contain duplicates Department names for the employee records.


## Section 3

### Question 3.1

    CREATE FUNCTION dbo.MapEcentricTestTable (@name VARCHAR(255))
    RETURNS VARCHAR(255)
    AS
    BEGIN
	RETURN (
			SELECT ID
			FROM EcentricTestTable
			WHERE Name = @name
			)
	END

### Question 3.2

You could check whether the name exists in the table first, then select and run the function:

    IF EXISTS ( 
    SELECT *
    FROM dbo.EcentricTestTable
    WHERE Name = 'Test2'
    )
    BEGIN
    SELECT * FROM dbo.EcentricTestTable e 
    JOIN EcentricTestTable_NEW n ON n.ID = e.ID 
    WHERE e.ID = dbo.MapEcentricTestTable('Test2') 
    END

To further this you could also create another function that will take in the name parameter, check if its exists and then call the MapEcentricTestTable function,
    


## Section 4

### Question 4.1
    The result of the above query will be ‘No’
    NULL is stored as a reference type and no two NULLS are equal. 

### Question 4.2
    Add SET ANSI_NULLS off
    The = and <> operators will no longer follow the ISO standard, producing a result of ‘Yes’

### Question 4.3

    SELECT CASE 
		WHEN NULL IS NULL
			THEN 'Yes'
		ELSE 'No'
		END AS Result

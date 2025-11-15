# Student Management MVC

A simple Java MVC application for managing student records (CRUD) using Servlets, JSP, and MySQL.

This application follows a Model-View-Controller (MVC) pattern where:
* **Model:** `Student.java` (the data object)
* **View:** `.jsp` files (not provided, but referenced as `/views/student-list.jsp` and `/views/student-form.jsp`)
* **Controller:** `StudentController.java` (a single servlet that routes all requests)
* **DAO:** `StudentDAO.java` (handles all database logic)

## ⚙️ Technologies Used

* Java 17
* Jakarta Servlets 6.0
* Jakarta JSP 3.1
* Jakarta JSTL 3.0
* MySQL Connector/J
* Maven

## 🗄️ Database Setup

The application expects a MySQL database and table set up as follows. The connection details are hardcoded in `StudentDAO.java`:

* **URL:** `jdbc:mysql://localhost:3306/student_management`
* **User:** `root`
* **Password:** `Bensam^061506`

### SQL Table Schema

You must create the `student_management` database and the `students` table:

```sql
CREATE DATABASE IF NOT EXISTS student_management;
USE student_management;

CREATE TABLE students (
    id INT PRIMARY KEY AUTO_INCREMENT,
    student_code VARCHAR(20) UNIQUE NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL,
    major VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample data
INSERT INTO students (student_code, full_name, email, major) VALUES
('SV001', 'Nguyen Van A', 'a.nguyen@example.com', 'Computer Science'),
('SV002', 'Tran Thi B', 'b.tran@example.com', 'Information Technology'),
'SV003', 'Le Van C', 'c.le@example.com', 'Software Engineering');
```

## 🌐 Application Workflow (CRUD)

The entire application is controlled by the `StudentController` servlet, which is mapped to the URL pattern `/student`.

The logic is routed using a URL parameter named `action`.

---

### 1. List Students (Read)

This is the default action when no `action` parameter is provided.

1.  **Request:** A `GET` request is made to `/student` (or `/student?action=list`).
2.  **Controller (`StudentController.java`):**
    * `doGet()` is called. The `action` is "list".
    * The `switch` statement calls the `listStudents()` method.
3.  **Controller (`listStudents()`):**
    * Calls `studentDAO.getAllStudents()`.
4.  **DAO (`StudentDAO.java`):**
    * `getAllStudents()` connects to the database.
    * Executes `SELECT * FROM students ORDER BY id DESC`.
    * Returns a `List<Student>` to the controller.
5.  **Controller (`listStudents()`):**
    * The list of students is attached to the request: `request.setAttribute("students", students)`.
    * The request is forwarded to the view: `/views/student-list.jsp`.
6.  **View (`student-list.jsp`):**
    * The JSP uses JSTL (e.g., `<c:forEach>`) to iterate over the `${students}` list and display them in an HTML table.

---

### 2. Create Student (Create)

This is a two-part process: showing the form and handling the submission.

#### Part A: Show New Student Form

1.  **Request:** A `GET` request is made to `/student?action=new`.
2.  **Controller (`StudentController.java`):**
    * `doGet()` is called. The `action` is "new".
    * The `switch` statement calls `showNewForm()`.
3.  **Controller (`showNewForm()`):**
    * The request is forwarded to the view: `/views/student-form.jsp`.
4.  **View (`student-form.jsp`):**
    * The JSP displays an empty HTML form.
    * The form's `method` is `POST` and its `action` attribute points to `student?action=insert`.

#### Part B: Handle Form Submission

1.  **Request:** The user submits the form, sending a `POST` request to `/student?action=insert`.
2.  **Controller (`StudentController.java`):**
    * `doPost()` is called. The `action` is "insert".
    * The `switch` statement calls `insertStudent()`.
3.  **Controller (`insertStudent()`):**
    * Retrieves form data (`studentCode`, `fullName`, `email`, `major`) using `request.getParameter()`.
    * Creates a `new Student(...)` object.
    * Calls `studentDAO.addStudent(newStudent)`.
4.  **DAO (`StudentDAO.java`):**
    * `addStudent()` connects to the database.
    * Executes an `INSERT INTO students (...) VALUES (?, ?, ?, ?)` statement.
    * Returns `true` if the row was inserted.
5.  **Controller (`insertStudent()`):**
    * `response.sendRedirect()` is called, redirecting the user back to the student list (`/student?action=list`) with a success or error message in the URL.

---

### 3. Update Student (Update)

This is also a two-part process, very similar to Create.

#### Part A: Show Edit Student Form

1.  **Request:** A `GET` request is made from the list page, e.g., `/student?action=edit&id=5`.
2.  **Controller (`StudentController.java`):**
    * `doGet()` is called. The `action` is "edit".
    * The `switch` statement calls `showEditForm()`.
3.  **Controller (`showEditForm()`):**
    * Gets the `id` from the request.
    * Calls `studentDAO.getStudentById(id)` to fetch the existing data.
4.  **DAO (`StudentDAO.java`):**
    * `getStudentById()` executes `SELECT * FROM students WHERE id = ?`.
    * Returns the matching `Student` object.
5.  **Controller (`showEditForm()`):**
    * Attaches the fetched object to the request: `request.setAttribute("student", existingStudent)`.
    * The request is forwarded to the *same* view: `/views/student-form.jsp`.
6.  **View (`student-form.jsp`):**
    * The JSP detects that the `${student}` attribute is not empty.
    * It populates the form fields with the student's data (e.g., `<input value="${student.fullName}">`).
    * The form's `method` is `POST` and its `action` is `student?action=update`.
    * A hidden input is included: `<input type="hidden" name="id" value="${student.id}">`.

#### Part B: Handle Form Update

1.  **Request:** The user submits the modified form, sending a `POST` request to `/student?action=update`.
2.  **Controller (`StudentController.java`):**
    * `doPost()` is called. The `action` is "update".
    * The `switch` statement calls `updateStudent()`.
3.  **Controller (`updateStudent()`):**
    * Retrieves all form data, *including the hidden `id`*.
    * Creates a `Student` object and sets its ID.
    * Calls `studentDAO.updateStudent(student)`.
4.  **DAO (`StudentDAO.java`):**
    * `updateStudent()` connects to the database.
    * Executes an `UPDATE students SET ... WHERE id = ?` statement.
    * Returns `true` if the row was updated.
5.  **Controller (`updateStudent()`):**
    * `response.sendRedirect()` is called, redirecting the user back to the student list (`/student?action=list`).

---

### 4. Delete Student (Delete)

1.  **Request:** A `GET` request is made from the list page, e.g., `/student?action=delete&id=5`.
    * *(Note: Using `GET` for delete operations is not ideal practice—`POST` or `DELETE` methods are preferred—but it's how this controller is written.)*
2.  **Controller (`StudentController.java`):**
    * `doGet()` is called. The `action` is "delete".
    * The `switch` statement calls `deleteStudent()`.
3.  **Controller (`deleteStudent()`):**
    * Gets the `id` from the request.
    * Calls `studentDAO.deleteStudent(id)`.
4.  **DAO (`StudentDAO.java`):**
    * `deleteStudent()` connects to the database.
    * Executes a `DELETE FROM students WHERE id = ?` statement.
    * Returns `true` if the row was deleted.
5.  **Controller (`deleteStudent()`):**
    * `response.sendRedirect()` is called, redirecting the user back to the student list (`/student?action=list`) with a success or error message.
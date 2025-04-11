# Relationships

🔗 ## Relationships in Relational Databases

In a relational database, **relationships** define how data in one table is related to data in another. The purpose of these relationships is to maintain **data integrity** and reduce **data redundancy**.

There are three main types:

---

### 1. **One-to-One (1:1)**

Each row in Table A is linked to **only one** row in Table B, and vice versa.

**Example:**

- One `User` has one `Passport`
- Tables: `users(id, name)`, `passports(user_id, passport_number)`

> Useful when you want to split rarely used or sensitive data into a separate table.
> 

---

### 2. **One-to-Many / Many-to-One (1:N or N:1)**

One row in Table A can be linked to **many** rows in Table B.

But each row in Table B is linked to **only one** row in Table A.

**Example:**

- One `Author` has many `Books`
- Tables: `authors(id, name)`, `books(id, title, author_id)`

> This is the most common relationship.
> 

---

### 3. **Many-to-Many (M:N)**

Multiple rows in Table A relate to multiple rows in Table B.

To implement this, you need a **junction (association) table**.

**Example:**

- A `Student` can take many `Courses`, and each `Course` can have many `Students`.
- Tables:
    - `students(id, name)`
    - `courses(id, title)`
    - `student_courses(student_id, course_id)`

> The student_courses table is the bridge that holds foreign keys pointing to both sides.
> 

---

## 🛠 How Are Relationships Implemented?

Using **foreign keys**, which are constraints that ensure referential integrity.

```sql
sql
CopyEdit
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    author_id INTEGER REFERENCES authors(id)
);

```

You can define:

- `ON DELETE CASCADE` (delete related records)
- `ON UPDATE SET NULL`, etc.

---

## 🧠 Best Practices

- Always index foreign keys for performance.
- Use proper constraints to enforce integrity.
- Normalize wisely — but don’t be afraid to denormalize when necessary for performance.


# Examples with `sqlalchemy`
## 🧱 Setup (Boilerplate)

```python
python
CopyEdit
from sqlalchemy import Column, Integer, String, ForeignKey, Table
from sqlalchemy.orm import relationship, declarative_base

Base = declarative_base()

```

---

## 1. 🔗 One-to-One

```python
python
CopyEdit
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    passport = relationship("Passport", back_populates="user", uselist=False)

class Passport(Base):
    __tablename__ = 'passports'

    id = Column(Integer, primary_key=True)
    number = Column(String)
    user_id = Column(Integer, ForeignKey('users.id'))
    user = relationship("User", back_populates="passport")

```

> ⚠️ uselist=False tells SQLAlchemy this is a one-to-one, not one-to-many.
> 

---

## 2. 🔗 One-to-Many

```python
python
CopyEdit
class Author(Base):
    __tablename__ = 'authors'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    books = relationship("Book", back_populates="author")

class Book(Base):
    __tablename__ = 'books'

    id = Column(Integer, primary_key=True)
    title = Column(String)
    author_id = Column(Integer, ForeignKey('authors.id'))
    author = relationship("Author", back_populates="books")

```

> Here, one Author has many Books, and each Book belongs to one Author.
> 

---

## 3. 🔗 Many-to-Many

```python
python
CopyEdit
# Association table (no model class needed)
student_course = Table(
    'student_course',
    Base.metadata,
    Column('student_id', Integer, ForeignKey('students.id')),
    Column('course_id', Integer, ForeignKey('courses.id'))
)

class Student(Base):
    __tablename__ = 'students'

    id = Column(Integer, primary_key=True)
    name = Column(String)
    courses = relationship("Course", secondary=student_course, back_populates="students")

class Course(Base):
    __tablename__ = 'courses'

    id = Column(Integer, primary_key=True)
    title = Column(String)
    students = relationship("Student", secondary=student_course, back_populates="courses")

```

> secondary=student_course points to the junction table.
> 

---

### 🧠 Extra Tips

- Always define `back_populates` or `backref` for bi-directional access.
- You can add extra fields to the association table by converting it to a full model class.
- For one-to-one, ensure uniqueness on the FK at the DB level for full safety.
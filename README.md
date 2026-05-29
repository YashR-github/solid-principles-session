# SOLID Design Principles 

## Introduction : 
Let us start by understanding why maintainable software design matters in real-world development :<br><br>
**Q. How much time do you think developers spend each day on an average writing new code:** <br><br>
**Ans:** Less than 40% of developer's time goes in writing new code. In a company like Google, less than 15% of developer's time goes into writing fresh code.
<br>
Other time goes into :<br>
1. Reading other people's code.
2. Going through documentation
3. Discussion
4. Tutorials
5. Chatgpt/ Research
6. Stackoverflow
7. Code review
8. Collaboration and meetings (eg: Jira, Scrum)
9. Testing and QA
10. Documentation
11. Devops and CI/CD

Breaks: Play TT, Read a book, Coffee/tea, Relax etc

So the question is how to **maximize efficiency and accuracy** of the written code and ensure less time in maintaining and refactoring the code?<br>
**Ans:** By ensuring that whatever work we do, we do it right the first time.
<br> <br>
Things we must ensure in code:<br>

1. Readability
2. Testable
3. Maintainability
4. Extensible

The toolkit we use to achieve this is : SOLID design principles. <br>
The **SOLID** design principles were introduced by **Robert Martin** in his paper **'Design Principles and Design Patterns'** in 2000 and since then it has revolutionized how developers write software.<br>


SOLID is a set of five object-oriented design principles that focus on creating elegant, robust, and maintainable object-oriented code. 
Following are the 5 principles of SOLID:<br>

1. Single Responsibility Principle (SRP)
2. Open/Close principle (OCP)
3. Liskov's Substitution Principle (LSP) 
4. Interface Segregation Principle (ISP)
5. Dependency Inversion Principle (DIP)


Think of them as the pillars of good software architecture. Let's break each one down:


## 1. S : Single Responsibility Principle (SRP)
Simply put, the Single Responsibility Principle states that a class, module, or function should have only "1" job. The most common definition says every class should have only one reason to change. If a class contains the logic of performing more than one job, it won't be easy to debug and maintain as the code base increases in size. 
Let's understand this with an example:<br>
Let's say we have this class "UserService" that handles all the things related to the user in one single class:

```python
class UserService:

    def create_user(self, user):
        print("Creating user:", user)
        self.save_to_db(user)
        self.send_email(user)

    def save_to_db(self, user):
        print("Saving user to database")

    def send_email(self, user):
        print("Sending welcome email")
```

**Core Problems :**
- If email logic changes → class must change
- If DB changes → class must change
- Too many responsibilities in one place


New developers will have a tough time trying to read and understand the code. If we separate the logic and make each class perform one single job, not only will the code be easy to maintain and debug, but the code will also become reusable because each core functionality is performed by one class.

Hence, a better approach would be to separate different functionalities into separate classes as follows:
```python
class UserRepository:

    def save(self, user):
        print("Saving user to database")


class EmailService:

    def send_welcome_email(self, user):
        print("Sending welcome email")


class UserService:

    def __init__(self, user_repository, email_service):
        self.user_repository = user_repository
        self.email_service = email_service

    def create_user(self, user):
        print("Creating user:", user)

        self.user_repository.save(user)
        self.email_service.send_welcome_email(user)

```

## 2. O: Open/Closed Principle (OCP)
Stands for : Open for Extension , Closed for Modification.


Let's consider an example:
Consider our code provides the main functionality of returning the "area" of every 2-D shape.

```python
import math

class AreaCalculator:
    def calculate_area(self, shape):

        if isinstance(shape, Rectangle):
            return shape.width * shape.height

        elif isinstance(shape, Circle):
            return math.pi * shape.radius ** 2

        return 0


class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height


class Circle:
    def __init__(self, radius):
        self.radius = radius



```
### **The Problem :**

If you want to add new shapes later, like:<br>
- Triangle <br>
- Square <br>
- Pentagon <br>

You **must** keep modifying **AreaCalculator** using additional **'elif'** blocks, which break **"closed for modification"** principle.

A better approach would be to abstract out a **"Shape"** class and use it in **AreaCalculator** to calculate area for any shape without modifying existing code:

```python
import math
from abc import ABC, abstractmethod

class Shape(ABC):

    @abstractmethod
    def area(self):
        pass


class Rectangle(Shape):

    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height


class Circle(Shape):

    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * self.radius ** 2


class AreaCalculator:

    def calculate_area(self, shape: Shape):
        return shape.area()

```



Now a question may arise, how can a code always be **"closed for modification"**? What if a functionality is not needed and needs to be removed? <br><br>
Ans: OCP does not mean code can **never** be modified.
It means that a stable, well-tested code should not need frequent modification when adding new behavior. If a particular functionality is not needed, it can absolutely be removed. The better way to think of OCP is: <br><br>
**Open for extension → Be able to add new behavior without changing existing core logic.**<br>
**Closed for modification → Avoid modifying existing stable working code for every new feature.**

## 3. L : Liskov's Substitution Principle (LSP)
If a subclass does not properly behave like its Parent, substitution breaks.

```python
class Bird:
    def fly(self):
        return "Flying"
class Sparrow(Bird):
    def fly(self):
        return "Sparrow flying"
class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins can't fly")
```
**The Problem :**

Code expecting a Bird breaks when a Penguin is used:
```python
def make_bird_fly(bird: Bird):
    return bird.fly()

make_bird_fly(Penguin())  # Gives Runtime error
```
### Correct LSP Design (Better Abstraction)

Instead of forcing inheritance, separate capabilities:

```python
class Bird:
    pass
class FlyingBird(Bird):
    def fly(self):
        return "Flying"

class Sparrow(FlyingBird):
    pass
class Penguin(Bird):
    def swim(self):
        return "Swimming"
```
```python
Now usage is safe:
def make_bird_fly(bird: FlyingBird):
    return bird.fly()
```
<br>
Now:<br><br>

- Only flying birds are passed → no crash <br>
- Substitution works correctly
<br>

**Another example:** <br><br>
Square changes expected behavior of Rectangle → unexpected side effects.
```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def set_width(self, w):
        self.width = w

    def set_height(self, h):
        self.height = h

    def area(self):
        return self.width * self.height

class Square(Rectangle):
    def set_width(self, w):
        self.width = self.height = w

    def set_height(self, h):
        self.width = self.height = h
```
## Correct LSP Design (Better Abstraction)

```python
class Shape:
    def area(self):
        raise NotImplementedError()

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    def area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side
    def area(self):
        return self.side * self.side

```

Final Takeaway:

- Ensure subclasses never break parent expectations.
- Prefer composition or separate implementations.
- Don’t force inheritance just because of similiarity.
- If substituting a child class breaks logic, then the design violates LSP and needs to be redesigned.



## 4. I : Interface Segregation Principle (ISP)
The I in SOLID design principles stands for interface segregation, which indicates that bigger interfaces should be divided into smaller ones. By doing so, we can ensure that implementing classes is only concerned with the methods that are relevant to them. In other words, multiple client-specific interfaces are better than a single generic interface.

Key idea :
- Clients must never be forced to implement an interface or methods in an interface that they do not use.

```python
from abc import ABC, abstractmethod

class Shape(ABC):

    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def volume(self):
        pass


class Circle(Shape):

    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2

    def volume(self):
        # Not meaningful for Circle (2D shape)
        raise NotImplementedError("Circle has no volume")


class Square(Shape):

    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

    def volume(self):
        # Not meaningful for Square (2D shape)
        raise NotImplementedError("Square has no volume")
```
**Problems :**
- 2D shapes are forced to implement volume().
- Leads to dummy methods or exceptions.
- Violates clean design.
Better design:

```python
from abc import ABC, abstractmethod
import math


class Shape2D(ABC):

    @abstractmethod
    def area(self):
        pass


class Shape3D(ABC):

    @abstractmethod
    def volume(self):
        pass

# Now 2D shapes can implement only the Shape2D interface.

class Circle(Shape2D):

    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2

class Square(Shape2D):

    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side ** 2

# While 3D shapes can implement the Shape3D interface separately
class Cube(Shape3D):

    def __init__(self, side):
        self.side = side

    def volume(self):
        return self.side ** 3


class Sphere(Shape3D):

    def __init__(self, radius):
        self.radius = radius

    def volume(self):
        return (4/3) * math.pi * self.radius ** 3

```

**Why this is better :**
- No unnecessary methods<br>
- No fake implementations<br>
- Each class depends only on what it uses <br>
- Easier to maintain and extend<br>


The Interface Segregation Principle (ISP) can be viewed as applying the idea of 'Single Responsibility Principle (SRP)' at the interface level, where interfaces are kept focused so clients depend only on behaviors they actually use.

- SRP → separates responsibilities inside implementations/classes
- ISP → separates responsibilities inside interfaces/contracts



## 5. D : Dependency Inversion Principle (DIP)
Entities must depend on abstractions, not on concrete implementations. That is high-level module must not depend on low-level module, instead they should depend on abstractions.
<br> <br>
Here the high-level class depends directly on a concrete implementation:<br>

```python
class MySQLDatabase:
    def save(self, data):
        print("Saving to MySQL:", data)


class UserService:

    def __init__(self):
        self.db = MySQLDatabase()   

    def create_user(self, user):
        self.db.save(user)
```
**Problem:**

- UserService is locked to MySQL
- If we switch to MongoDB → we must modify UserService
- High-level logic depends on low-level details

Better approach :

```python
from abc import ABC, abstractmethod

class Database(ABC):

    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):

    def save(self, data):
        print("Saving to MySQL:", data)


class MongoDatabase(Database):

    def save(self, data):
        print("Saving to MongoDB:", data)

class UserService:

    def __init__(self, db: Database):
        self.db = db   # depends on abstraction, not concrete class

    def create_user(self, user):
        self.db.save(user)

mysql_service = UserService(MySQLDatabase())
mysql_service.create_user("Alice")

mongo_service = UserService(MongoDatabase())
mongo_service.create_user("Bob")
```
**Why this follows DIP:**<br>
- UserService does NOT know about MySQL or MongoDB<br>
- It only knows the Database interface<br>
- You can swap implementations without modifying UserService<br>


So, always strive to make classes as loosely connected as possible, which you can do through abstraction. 

## Important nuances:

**Q. Do SOLID principles apply to only classes?** <br>
**Ans:** No, SOLID principles apply to **wide range of Software entities**.
“Software entities” can include:

- Classes
- Functions / methods
- Modules
- Components
- Packages
- APIs
- Even larger architectural units like services


**Q. Which languages support SOLID design principles?**<br><br>
**Ans:** Any modern programming language that supports OOP, can follow SOLID principles, including: <br>
- Java
- Python
- C#
- C++
- Dart
- Ruby
- JavaScript
- PHP


**Q. Do all modern developers "always" follow SOLID principles fully?**
 <br><br>
**Ans:** No. SOLID is a guideline, not a strict rulebook. Modern developers don’t “always follow SOLID perfectly/ extensively”.

**Instead they:**
1. Apply SOLID when it adds value
2. Ignore or simplify it in small scripts or prototypes
3. Balance it with practicality (avoid over-engineering)

So the real approach is more dependent on the type of project, its expectations, future requirements and practicality.


**Q. Where SOLID is heavily used?**

- Microservices architecture
- Clean architecture / hexagonal architecture
- Large codebases with many contributors
- Long-lived production systems

**Q. Does Low Level Design (LLD) mean following SOLID principles only?**<br>
**Ans:** SOLID comes under LLD (Low level design) which is a vast topic. It covers other important things like:
- Understanding of OOP
- Design patterns (builder, singleton, factory, strategy etc)
- Database Schema design 
- Entities & Relationships (often represented through ER Diagram/Class diagram)



## Conclusion:
SOLID principles are not just theoretical software concepts — they are practical engineering guidelines that help developers build scalable, maintainable, flexible, and production-ready systems. As software systems grow in complexity, code that lacks proper structure becomes difficult to debug, extend, test, and collaborate on. SOLID helps solve these problems by encouraging separation of concerns, loose coupling, better abstraction, and safer extensibility.

By applying SOLID principles appropriately, developers can:

- Reduce code complexity
- Improve readability and maintainability
- Make systems easier to test and debug
- Extend features with lower risk of breaking existing code
- Build cleaner architectures suitable for long-term projects

However, SOLID should be treated as a **practical guideline** rather than a rigid rulebook. Overusing abstractions or applying principles unnecessarily in very small projects can lead to over-engineering. Strong software design comes from balancing SOLID principles with simplicity, project requirements, team size, scalability needs, and real-world practicality.

In modern backend systems, large-scale applications, microservices, APIs, and enterprise software, **SOLID principles remain one of the strongest foundations** for writing professional-quality code and designing robust software architectures.

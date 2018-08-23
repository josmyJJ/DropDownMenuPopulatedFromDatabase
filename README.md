# Lesson 10 - Drop Down list populated from database  
## The Walkthrough

1. Get copy of lesson10

2. Create a Class
	* Right click on com.example.demo and click New -> Class
	* Name it Subject.java
	* Edit it to look like this:

```java
import javax.persistence.*;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import java.util.Set;

@Entity
public class Subject {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private long id;

  @NotNull
  @Size(min=4)
  @Column(unique = true)
  private String title;

  @OneToMany(mappedBy = "subject")
  public Set<Course>courses;
}

```

3. Autogenerate getters and setters
  * Right-click on the word Course and select generate -> Getters and Setters
  * Select all the fields list and click OK

4. Create a Repository
	* Right click on com.example.demo and click New -> Class
	* Name it SubjectRepository.java
	* Edit it to look like this:

```java
import org.springframework.data.repository.CrudRepository;

public interface SubjectRepository extends CrudRepository<Subject, Long> {
  Subject findByTitle(String subject_name);
}
```

5. Edit a Course Class
	* Open Course.java
	* Add the relationship

```java

@ManyToOne
private Subject subject;

```
6. Auto generate getters and setters for the ManyToOne relationship


7. Edit Controller
	* Open HomeController.java
	* Edit the HomeController to look like this:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;

import javax.validation.Valid;

@Controller
public class HomeController {

  @Autowired
  CourseRepository courseRepository;

  @Autowired
  SubjectRepository subjectRepository;

  @RequestMapping("/")
  public String listCourses(Model model){
    model.addAttribute("courses", courseRepository.findAll());
    model.addAttribute("subjects", subjectRepository.findAll());
    return "list";
  }

  @GetMapping("/add")
  public String courseForm(Model model){
    model.addAttribute("course", new Course());
    model.addAttribute("subjects", subjectRepository.findAll());
    return "courseform";
  }

  @PostMapping("/process")
  public String processForm(@Valid Course course, BindingResult result, Model
          model){
    if (result.hasErrors()){
      model.addAttribute("subjects", subjectRepository.findAll());
      return "courseform";
    }
    courseRepository.save(course);
    return "redirect:/";
  }

  @GetMapping("/addsubject")
  public String subjectForm(Model model){
    model.addAttribute("subject", new Subject());
    return "subjectform";
  }


  @PostMapping("/processsubject")
  public String processSubject(@Valid Subject subject, BindingResult result,
                               Model model){
    if(result.hasErrors()){
      return "subjectform";
    }

    if(subjectRepository.findByTitle(subject.getTitle()) != null){
      model.addAttribute("message", "You already have a subject called " +
              subject.getTitle() + "!" + " Try something else.");
      return "subjectform";
    }

    subjectRepository.save(subject);
    return "redirect:/";
  }

  @RequestMapping("/detail/{id}")
  public String showCourse(@PathVariable("id") long id, Model model){
    model.addAttribute("course", courseRepository.findById(id).get());
    return "show";
  }

  @RequestMapping("/update/{id}")
  public String updateCourse(@PathVariable("id") long id, Model model){
    model.addAttribute("subjects", subjectRepository.findAll());
    model.addAttribute("course", courseRepository.findById(id).get());
    return "courseform";
  }

  @RequestMapping("/delete/{id}")
  public String delCourse(@PathVariable("id") long id){
    courseRepository.deleteById(id);
    return "redirect:/";
  }
}

```

8. Create a Template for the subject form
  * Right click on templates and click New -> Html
	* Name it subjectform.html
	* Edit it to look like this:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Subject Form</title>
</head>
<body>

<h1>Subject Form</h1>

<form action="#"
      th:action="@{/processsubject}"
      th:object="${subject}"
      method="post">

    <input type="hidden" th:field="*{id}" />
    Title :<input type="text" th:field="*{title}" />

    <span th:if="${#fields.hasErrors('title')}"
          th:errors="*{title}"></span><br />

    <span th:text="${message}"></span><br/>

    <input type="submit" value="Submit" />

</form>

</body>
</html>
```

9. Edit course form template
  * Open courseform.html
	* Add the following lines of codes inside the form

```html
Subject:<select th:field="*{subject}" class="form-control input-md">
           <option th:each="option: ${subjects}"
                   th:value="${option.id}">[[${option.title}]]</option>
       </select><br/>

```
  * The file should look like this:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Course Form</title>
</head>
<body>

<h1>Course Form</h1>
<form action="#"
      th:action="@{/process}"
      th:object="${course}"
      method="post">
    <input type="hidden" th:field="*{id}" />

    Subject:<select th:field="*{subject}" class="form-control input-md">
            <option th:each="option: ${subjects}"
                    th:value="${option.id}">[[${option.title}]]</option>
        </select><br/>

    Title :<input type="text" th:field="*{title}" />
    <span th:if="${#fields.hasErrors('title')}"
          th:errors="*{title}"></span><br />
    Instructor :<input type="text" th:field="*{instructor}" />
    <span th:if="${#fields.hasErrors('instructor')}"
          th:errors="*{instructor}"></span><br />
    Description :<textarea rows="3" th:field="*{description}" />
    <span th:if="${#fields.hasErrors('description')}"
          th:errors="*{description}"></span><br />
    Credits :<input type="text" th:field="*{credit}" />
    <span th:if="${#fields.hasErrors('credit')}"
          th:errors="*{credit}"></span><br />
    <br />
    <input type="submit" value="Submit" />
</form>
</body>
</html>

```

10. Edit the course listings template
  * Open list.html
	* Edit it to look like this:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Course List</title>
</head>
<body>

<h1>Course List</h1>
<a href="/addsubject">Add a Subject</a><br />
<a href="/add">Add a Course</a><br />
<table>
    <tr>
        <th>Subject</th>
        <th>Title</th>
        <th>Instructor</th>
        <th>Credits</th>
        <td>Actions</td>
    </tr>
    <tr th:each="course : ${courses}">
        <td th:text="${course.title}"></td>
        <td th:text="${course.subject.title}"></td>
        <td th:text="${course.instructor}"></td>
        <td th:text="${course.credit}"></td>
        <td>
            <a th:href="@{/update/{id}(id=${course.id})}">Update</a> -
            <a th:href="@{/detail/{id}(id=${course.id})}">Details</a> -
            <a th:href="@{/delete/{id}(id=${course.id})}">Delete</a>
        </td>
    </tr>
</table>
</body>
</html>

```

11. Edit course detail template
	* Open show.html
	* Edit it to look like this:

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Course Details</title>
</head>
<body>

<h1>Course Details</h1>

<a href="/">Show All Courses</a><br />
Subject :<span th:text="${course.subject.title}" ></span><br />
Title :<span th:text="${course.title}" ></span><br />
Instructor :<span th:text="${course.instructor}" ></span><br />
Description :<p th:text="${course.description}" ></p>
Credits :<span th:text="${course.credit}" ></span><br />

<a th:href="@{/delete/{id}(id=${course.id})}">Delete this course</a>

</body>
</html>
```

## What is Going On

#### CrudRepository
Subject repository finds the subject by title which will be used to check for duplicate entry in the subject form. 

### The Controller

#### The routes

##### Default Route (“/”)
When the user visits this route, the user will see a list of all the course entries and subject entries that have been made.

#### Add route ("/addsubject")
When a user visits this route, a new instance of the Subject class will be created and passed to the view. This will hold all values that the user enters into the form and return them to the controller at the route specified on the form by the POST method.

#### Process route ("/processsubject")
This route validates the subject for errors and check to see if a subject already exists in the database, saves it to the database (using the SubjectRepository object created by the @Autowired annotation), and redirects the user to the default route.

### The View

#### subjectform.html
This is the form that allows users to add new subject. It is tied to the course model (th:object="${subject}"), and has validation that uses the default error messaging for the fields that have been annotated in the model (e.g. title). The message object (th:text="${message}") displays a message if the user tries to enter a subject title that already exists in the database. 

#### courseform.html
Within the select tag thymeleaf loop through all the available subject and provides a dropdown menu that contains the subject title.



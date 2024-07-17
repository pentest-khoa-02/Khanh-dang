# SSTI

(**Server side template injection** ) 

- **WHAT IS THE BUG ?**
    
    SSTI is the vulnerable that malicious code embed to template in render processing instead of passing data static .
    
    Example : 
    
    ![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled.png)
    
- **Where do it occur in the web  ?**
    
    In the web using server-side-rendering  ?? 
    
    ?? single page ⇒ ok 
    
- **Root-cause**
    
    **Server-side template injection attacks can occur when user input is concatenated directly into a template**
    
- **How to detect ?**
    
    Trigger error and analysis response ( time, status code , error messages  )
    
    ⇒ **break syntax of template engine** 
    
    ```jsx
    Fuzzing  a lot of payload will be break syntax of template engine 
    
    Example :  ${{<%[%'"}}%\
    ```
    
    Two context:
    
    - **Plant text** : ⇒ Obvious XSS ⇒ payload with syntax expression statement
    
    ![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled%201.png)
    
    - **Code context** ⇒ escape ⇒ XSS ⇒ payload with object provider
    
    Example: `user.name%><h1>hi</h1>`  
    
    ![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled%202.png)
    
    Inside expression statement .
    
    A way to detect and identify using Template Injection Table 
    
    [https://cheatsheet.hackmanit.de/template-injection-table/index.html](https://cheatsheet.hackmanit.de/template-injection-table/index.html) 
    
    - Trigger Error based
    - Non-error based
- **IMPACT**
    - Extracting Sensitive information
    - RCE
    
    ![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled%203.png)
    
    - XSS
    
    ![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled%204.png)
    
    - File inclusion
    
    ![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled%205.png)
    
- **How to test ?**
    
    STEP  1:  Identify template engine
    
    STEP 2 :  analysis context ⇒ read documentation 
    
    STEP 3 :  Attempt call **os** module or module access to file 
    
    Build the RCE exploit 
    
    - Explore ⇒ try to discover all the object to which you have access
    
    `Target`  :  Find “self” or “environment”  object ⇒ like a namespace containing all objects , methods, and attributes that are supported by template engine .⇒ 
    
    - Lists of built-in methods, functions, filters, and variables.
- **How to exploit**
    
    **The first step is to identify objects and methods to which you have access .**
    
    - READ  :
    1. Learn basic syntax 
    2. security implications  < focus on warning section>
    - Explore
    1. Built-in object or Developer-supplied objects 
    
- **How to prevent**
    - Sanitize Data :  Before using a template , identify and remove potentially malicious content
    - Set up a sandbox environment  // **research**
    - passing user input as data , not make directly into template
    

### Note :

Dev-tool: ⇒ xem file js ⇒ search syntax  statement , 

`Java.lang.Runtime`  without import ,  search execution code in documentation 

 

## Sandbox

Is an environment safe , that is allowed some function or method is not dangerous execution in template engine .

![Untitled](SSTI%20f6f7ace2201b4bd9b58bcbe52f1f475f/Untitled%206.png)

### MainModule in Node js

module process in Nodejs is a  API core path and provide more function due to interact with and control process of NodeJS currently .  

Process meaning is a version of NodeJs program is  running .

Process is controlled by operation system and can interact with other part in system such as file system , network ,….

When you running one Node js application , you is initializing new process Nodejs 

```jsx
 <% import('fs').then(module => { global.hihi = (module.readFileSync('D:\\download\\test.txt',{ encoding: 'utf8', flag: 'r' })) }) %> <%= global.hihi %>
```
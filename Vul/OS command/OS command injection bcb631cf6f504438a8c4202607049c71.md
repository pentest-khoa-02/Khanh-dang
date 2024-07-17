# OS command injection

## What is the bug ?

Untrusted data is  directly chained in raw command .

```sql
$filename= $_GET['file_name']
system("cat $filename") 
```

## WHERE Occur in the web ?

In these feature using system call , example is : check host , remove file  ? 

## How to detect ?

Break syntax ⇒ analysis response (status code ,  time  , length)

- time delay
- out of band

## How to test  ?

Test objective : 

`Identify injection point` 

pipeline :  output of an command will be input of other command 

```sql
ls | wc -l 
ls => liệt kê tất cả thư mục , chuyển đầu vào cho lệnh thứ 2 => đếm số dòng 

```

- `stdin` : standard input  :  data is get by key broad ,  file , pipeline
- `stdout` : standard output
- `stderr` : standard error

? 

**Special character for Os command** 

| Name | Description |
| --- | --- |
| cm1 | cmd2 | uses of  | will make command2  is executed whether cmd1 execution is successful or not  |
| cmd1;cmd2  | uses of ; will make command2  is executed whether cmd1 execution is successful or not  |
| cmd1 || cmd2 | Uses of ||  will only  command2 is executed if command1 execution is fails |
| cmd1&&cmd2 | Uses of && will only execute command2 if cmd1 is successful |
| $(cmd) | Sub command  |
| `cmd` | Sub command |

**Process Substitution** :

?? 

Code review Dangerous : 

- Java ⇒  `Runtime.exec()`
- C/C++ : `system` , `exec` , `ShellExecute`
- Python : `eval` , `exec`, `os.popen` , `subprocess` , …
- PHP  :  `system` , `shell_exec` , `exec`, `proc_open` , `eval`

## How to prevent  ?

- Never call system from application layer
- Validate user input ⇒ whitelist

### Specific prevent point

- **Argument Injection**

In this type , user’s  input is passed as argument of command while executing specific command 

For example  : 

For example, if the user input is passed through an escape function to escape certain characters like `&`, `|`, `;`, etc.

```sql
system("curl " . escape($url));
```

which will prevent an attacker to run other commands.
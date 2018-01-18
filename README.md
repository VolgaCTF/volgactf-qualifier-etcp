# Themis Quals Checker Protocol (TQCP)
Themis Quals is a contest checking system for CTF contests (jeopardy). This document introduces a protocol to check an answer to a task.

## Call for a change
The current system has only one way to specify a correct answer / a limited number of correct answers to a task. Nonetheless, some types of problems (e.g. cryptography) may be solved in a variety of ways, which may result in a great many valid answers for a single task.

## A new protocol (proposal)
A Themis Quals task checker (TQTS) is a REST web service.

A TQTS runs on a public FQDN[:port]. For instance, the FQDN might look like `task.checker.quals.themis-project.com`.

A TQTS accepts requests from a Themis Quals master server (TQMS) and provides a response indicating whether a provided answer is correct or not.

### Authentication
The TQMS makes use of [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) to authenticate itself to a TQTS. Additionaly, a TQTS may require of the TQMS to start a secure (HTTPS) connection. Last but not the least, a TQTS may accept connections only from a limited set of IP adresses.

**Note.** The aforementioned measures may be implemented by a load-balancing server like Nginx. In a nutshell, a basic TQTS may lack authentication & security features at all.

### Heartbeat

This request may be made by the TQMS to check TQTS availability status.

*Request*

```
GET /check
Host: task.quals.finals.themis-project.com
Content-Type: application/json
```

*Response*

```
HTTP/1.1 204 No Content
```

### Check

*Request*

```
POST /check
Host: task.checker.quals.themis-project.com
Content-Type: application/json

{
  "answer": "SOME FLAG"
}
```

*Response*

```
HTTP/1.1 200 OK
Content-Type: application/json

{
  "answer": "SOME FLAG",
  "result": true|false,
  "comment": "Try harder!"
}
```

### Web server process(es)
There may be launched several instances (processes) of a TQTS. A process monitoring system (e.g. Supervisor) should provide a TQTS start script with the following environment variables:
- `HOST`
- `PORT`
- `INSTANCE`

`INSTANCE` number will be used to calculate a port number to listen to. For example, the first process will be launched with these environment variables: `HOST=127.0.0.1 PORT=3000 INSTANCE=0`, and the second one with these ones: `HOST=127.0.0.1 PORT=3000 INSTANCE=1`. According to these settings, processes will listen to ports 3000 and 3001 respectively.

**Note.** In a modern Linux system, one may take advantage of `REUSE_PORT` socket option.

### Other requirements
To have a similar interface, it is suggested to put all scripts in `script` folder. The following script names are reserved and should be used only for certain purposes:

1. [Required] `server` - web server launch script;  
1. [Optional] `bootstrap` - project setup;  
2. [Optional] `console` - CLI;  

## License
MIT @ [Alexander Pyatkin](https://github.com/aspyatkin)

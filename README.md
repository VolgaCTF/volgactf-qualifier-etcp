# VolgaCTF Qualifier External Task Checker Protocol  (ETCP)
VolgaCTF Qualifier is a checking system for CTF contests (jeopardy). This document introduces a protocol to check an answer to a task.

## Call for a change
The current system has only one way to specify a correct answer / a limited number of correct answers to a task. Nonetheless, some types of problems (e.g. cryptography) may be solved in a variety of ways, which may result in a great many valid answers for a single task.

## A new protocol
An external task checker (ETC) is a REST web service.

An ETC runs on a public FQDN[:port]. For instance, the FQDN might look like `task.checker.qualifier.volgactf.org`.

An ETC accepts requests from a VolgaCTF Qualifier master server (VQMS) and provides a response indicating whether a provided answer is correct or not.

### Authentication
The VQMS makes use of [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) to authenticate itself to an ETC. Additionaly, an ETC may require that VQMS starts a secure (HTTPS) connection. Last but not the least, an ETC may accept connections only from a limited set of IP adresses.

**Note.** The aforementioned measures may be implemented by a load-balancing server like Nginx. In a nutshell, a basic ETC may lack authentication & security features at all.

### Heartbeat

This request may be made by the VQMS to check an ETC availability status.

*Request*

```
GET /check
Host: task.checker.qualifier.volgactf.org
```

*Response*

```
HTTP/1.1 204 No Content
```

### Check

*Request*

```
POST /check
Host: task.checker.qualifier.volgactf.org
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
There may be launched several instances (processes) of an ETC. A process monitoring system (e.g. Supervisor) should provide an ETC start script with the following environment variables:
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
MIT @ [VolgaCTF](https://github.com/VolgaCTF)

## SSH port forwarding trick
allows to take a port on a remote machine which is not exposed and forward it through ssh to the local machine.

example, an application runs on the remote machine under localhost, not exposed to the internet, the remote machine also have ssh port 22. you want your machine to
talk to remote host application. you can forward the port to local machine via ssh.

```
ssh -L <local_port>:127.0.0.1:<remote_port> <ssh_login>
```

now local port `local_port` will talk to the application in the remote port `remote_port`

## Tools for network monitoring
`netstat -a` for seeing all open tcp ports
`ss -a -n -t` shows all opened tcp ports
`ss -a -n -u` shows all opened udp ports

we can use these to monitor if any port is open on the machine that is leaking information by viruses and malware.

## File Transfer

```bash
# On client
nc -w 10 -l -p 12345 | tar -x
```

```bash
# On server
tar -c ~/Desktop/HomeVideo1.mp4 | nc 192.168.0.3 12345
```

```go
func main() {
	cmd := exec.Command("sleep", "1000")
	_ = cmd.Start()
	time.Sleep(1000 * time.Second)
}

```
```
➜  ~ ps -o pid,ppid,pgid,command,state
  PID  PPID  PGID COMMAND          STAT
32494 32491 32494 /Applications/iT Ss
32497 32495 32497 -zsh             S
94518 32497 94518 vim main.go      S+
37521 32491 37521 /Applications/iT Ss
37523 37522 37523 -zsh             S
73036 37523 73036 vim .            S+
 1261 94518  1261 /bin/zsh         Ss
 1471  1261  1471 go run main.go   S+
 1486  1471  1471 /var/folders/kl/ S+
 1487  1486  1471 sleep 1000       S+
 1318 32491  1318 /Applications/iT Ss
 1320  1319  1320 -zsh             S
➜  ~ kill 1487
➜  ~ ps -o pid,ppid,pgid,command,state
  PID  PPID  PGID COMMAND          STAT
32494 32491 32494 /Applications/iT Ss
32497 32495 32497 -zsh             S
94518 32497 94518 vim main.go      S+
37521 32491 37521 /Applications/iT Ss
37523 37522 37523 -zsh             S
73036 37523 73036 vim .            S+
 1261 94518  1261 /bin/zsh         Ss
 1471  1261  1471 go run main.go   S+
 1486  1471  1471 /var/folders/kl/ S+
 1487  1486  1471 (sleep)          Z+
 1318 32491  1318 /Applications/iT Ss
 1320  1319  1320 -zsh             S
```

[Process State](https://blog.actorsfit.com/a?ID=00850-c0bb8998-ba0e-4598-a743-3bc7fbed42f6)

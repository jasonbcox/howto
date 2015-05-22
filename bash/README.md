
# howto/bash

This document covers the current processes, best practices, and tips/tricks that I use personally for bash.
These methods may not apply for everyone, and I will try to make notes on different implementations where necessary.

### Table of Contents
 * [Exit Codes](#exit-codes)
 * [PIDS & Background Processes](#pids)

## Exit Codes<a id="exit-codes"></a>
```
./myscript
EXIT_CODE=$?        # $? is the Exit Code of the last-run command
```

## PIDS & Background Processes<a id="pids"></a>
```
MY_PID=$$           # $$ is the PID of the current process

./myscript.sh &
MYSCRIPT_PID=$!     # $! is the PID of the last-run process
```


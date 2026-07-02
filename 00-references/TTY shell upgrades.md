
| Command                                           | Description                                                                                                                                    |
| ------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `python3 -c 'import pty; pty.spawn("/bin/bash")'` | Spawns a shell via Python's pty module. Most reliable method where Python is available.                                                        |
| `/bin/sh -i`                                      | Invokes sh in interactive mode directly. Simple but depends on shell flags being supported.                                                    |
| `perl -e 'exec "/bin/sh";'`                       | Spawns a shell by replacing the Perl process. Works where Perl is installed.                                                                   |
| `ruby: exec "/bin/sh"`                            | Replaces the Ruby process with a shell.                                                                                                        |
| `lua: os.execute('/bin/sh')`                      | Runs a shell as a child process from Lua.                                                                                                      |
| `awk 'BEGIN {system("/bin/sh")}'`                 | Executes a shell via awk's system call. Useful when scripting languages are unavailable.                                                       |
| `vim -c ':!/bin/sh'`                              | Drops into a shell from vim's command mode without exiting the editor.                                                                         |
| `vim :set shell=/bin/sh :shell`                   | Sets the shell inside vim and opens it. Alternative vim method if `-c` is restricted.                                                          |
| `stty raw -echo; fg`                              | Upgrades a shell to full TTY. Use `Ctrl + Z` to suspend the session on the target. Copy the command on your terminal and double click `Enter.` |

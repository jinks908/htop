# Forking Htop

## TODO
───────────────────────────────
- [ ] Change: Help menu mappings (reflect changes)


## Key Mappings
───────────────────────────────
### Action.c
- [Help menu display](~/code/repos/htop/Action.c:656)
- [Default k/l maps](~/code/repos/htop/Action.c:939)
- [Auto-send SIGKILL function](~/code/repos/htop/Action.c:541)
- [Cursor follows process](~/code/repos/htop/Action.c:686)

### Panel.c
- [Scroll up/down](~/code/repos/htop/Panel.c:364)
- ["PAUSED" Color](~/code/repos/htop/Panel.c:196)

### MainPanel.c
- [Bottom Panel w/ F-Key Bindings](~/code/repos/htop/MainPanel.c:29)

### Process.c
- [Check and Mark 'l'](~/code/repos/htop/Process.c:266)
- [Check and Mark 'l'](~/code/repos/htop/Process.c:281)

### Platform.c
- [SIGKILL -9](~/code/repos/htop/darwin/Platform.c:90)

### CRT.c
- [Terminal Supports](~/code/repos/htop/CRT.c:1164)


#### Debugging Functions
───────────────────────────────

```c
if (key < 256) {
  fprintf(stderr, "Received key: %d (char: %c)\n", key, key);
} else {
  fprintf(stderr, "Received key: %d\n", key);
}
```

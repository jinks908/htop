# Forking Htop

## TODO
────────────────────────────────────────────────────────────────
- [x] Feat: **Auto-follow / auto-tag processes on search select**
    - We may be able to do this using the boolean `isFilter` variable (See ~/code/repos/htop/IncSet.c:47)

- [ ] Feat: **Auto-follow process on selection (cursor move)**
    - We would have to ensure that the process is unfollowed on cursor move


### Follow highlight color:
**────────────────────────────────────────────────────────────────**
function `Panel_setSelectionColor`
provided by `"Panel.h"`
**────────────────────────────────────────────────────────────────**
→ `void`
Parameters:
- `Panel * this (aka struct Panel_ *)`
- `ColorElements colorId (aka enum ColorElements_)`
**────────────────────────────────────────────────────────────────**
```cpp
void Panel_setSelectionColor(Panel *this, ColorElements colorId)
```


#### Debugging Functions
**────────────────────────────────────────────────────────────────**
**Print key value received**
```c
if (key < 256) {
  fprintf(stderr, "Received key: %d (char: %c)\n", key, key);
} else {
  fprintf(stderr, "Received key: %d\n", key);
}
```


## Key Mappings
**────────────────────────────────────────────────────────────────**
**See Keymap Menu -->** ~/icloud/code/screens/code_screens/htop_custom_bindings.jpg

---

### Action.c
- [Help menu display](~/code/repos/htop/Action.c:656)
- [Default k/l maps](~/code/repos/htop/Action.c:939)
- [Auto-send SIGKILL function](~/code/repos/htop/Action.c:541)

#### Cursor follow process
- [Follow process (help menu)](~/code/repos/htop/Action.c:686)
- [Follow process (mapping)](~/code/repos/htop/Action.c:933)
- [Follow process (function)](~/code/repos/htop/Action.c:578)

---

### Panel.c
- [Scroll up/down](~/code/repos/htop/Panel.c:364)
- ["PAUSED" Color](~/code/repos/htop/Panel.c:196)

---

### MainPanel.c
- [Bottom Panel w/ F-Key Bindings](~/code/repos/htop/MainPanel.c:29)

---

### Process.c
- [Check and Mark 'l'](~/code/repos/htop/Process.c:266)
- [Check and Mark 'l'](~/code/repos/htop/Process.c:281)

---

### Platform.c
- [SIGKILL -9](~/code/repos/htop/darwin/Platform.c:90)

---

### CRT.c
- [Terminal Supports](~/code/repos/htop/CRT.c:1164)

---

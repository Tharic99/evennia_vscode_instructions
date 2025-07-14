# Copilot Instructions for Evennia MUD Development

1. Project Overview
2. Coding Style & Standards
    - 2.1 Python Style Guide (PEP8 + enhancements)
    - 2.2 Evennia Code Style & Naming Conventions
    - 2.3 Imports
    - 2.4 File Organization
3. Typeclass Design Patterns
    - 3.1 Inheritance & Hook Usage
    - 3.2 Attribute Management
    - 3.3 Shared Behavior via Mixins
4. Command System
    - 4.1 Command Design Rules
    - 4.2 CmdCustom Template (with switches & parse)
5. Script System
    - 5.1 Script Lifecycle
    - 5.2 Persistent & Interval Scripts
6. Menu System (EvMenu)
    - 6.1 Menu Pattern & Node Logic
    - 6.2 Menu Input Validation
7. Table System (EvTable)
    - 7.1 Basic Tables
    - 7.2 Dynamic, Paginated, and Combat Tables
8. Prototypes & Object Creation
    - 8.1 Defining Prototypes
    - 8.2 Using `create_object`, `search_object`
9. Web Integration
    - 9.1 Django Views & URLs
    - 9.2 REST API and Web Client Hooks
10. Testing & Validation
    - 10.1 Debug Logging
    - 10.2 Unit Test Patterns
    - 10.3 Integration Testing with Evennia
11. Security & Access Control
    - 11.1 Input Validation
    - 11.2 Locks, Permissions, and Rate Limiting
12. Performance Optimization
    - 12.1 Query Optimization
    - 12.2 Caching & Memory Efficiency
13. Documentation & Comments
    - 13.1 Google-style Docstrings
    - 13.2 README and Code Quality Checklist
14. Copilot Instruction Patterns
    - 14.1 Copilot-Friendly Coding Rules
    - 14.2 General Behavior Heuristics

Appendices (Optional)
- A. Full Command Example
- B. Full Script Example
- C. Full EvMenu Sequence
- D. Advanced EvTable Reference
- E. Evennia ANSI Color & Style Codes
- F. Combat Mixin / Shared Logic


## 1. Project Overview

You are working on an **Evennia-based Python MUD (Multi-User Dungeon)**.

- Evennia is a game server framework for online multiplayer text games (MUDs, MU*s).
- It is built on Django + Twisted + Python 3.
- All major game logic is implemented via:
  - **Typeclasses** (Characters, Objects, Rooms, etc.)
  - **Commands** (`MuxCommand`)
  - **Scripts** (timed or persistent background processes)
  - **Menus and Utilities** (`EvMenu`, `EvTable`, `EvEditor`, etc.)
- Object storage follows:
  - `obj.db.attrname` for **persistent data**
  - `obj.ndb.attrname` for **non-persistent, runtime data**

Your goals are to:
- Adhere to Python best practices (PEP8, typing, clean functions)
- Use Evennia's APIs and hook systems properly
- Structure all logic for clarity, extensibility, and security
- Document and test your systems clearly

---

## 2. Coding Style & Standards

### 2.1 Python Style Guide (PEP8 + Enhancements)

- Use **PEP8** for indentation, naming, and whitespace
- Maximum line length: **100 characters**
- Always use **type hints** for function signatures
- Prefer **f-strings** for formatting (`f"{name}"` over `.format()` or `%`)
- Use **Google-style docstrings** (see example below)
- Handle **specific exceptions** (`except ValueError`, not bare `except`)
- Log errors with `logger.log_err()` or info with `logger.log_info()`
- Keep functions **small and focused** (Single Responsibility Principle)
- Use **list/dict comprehensions** where clearer
- Mock external dependencies in tests using `unittest.mock`
- The top of each python file should include a brief module-level docstring.
- **NEVER** replace existing code that is outside of what you have specifically been told to replace. If in doubt, **ALWAYS** confirm.
- When moving code between files or functions, ensure all dependencies and context are preserved.
- When moving code between files or functions, ensure to not place the code to break up the previous comment blocks.
- When moving code between files or functions, do not place it above the top  of the module-level docstring.
- Maintain consistent formatting and style throughout the codebase using the VScode formatter configured.
- Use meaningful names for variables, functions, and classes to improve code readability.



### 2.2 Evennia-Specific Style Guide

#### Code Layout

- Indentation: 4 spaces (never tabs)
- String formatting: use **double quotes** by default (`"text"`)
- Trailing commas: use in multiline lists, dicts, tuples
- No space before colons in dict/slice syntax
- Blank lines:
  - 2 between top-level functions and classes
  - 1 between methods in a class

#### Naming Conventions

- **Classes**: `PascalCase` (e.g. `DragonBoss`)
- **Functions/variables**: `snake_case` (e.g. `get_room_tag`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g. `MAX_ITEMS`)
- **Private methods/vars**: `_leading_underscore`


#### Type Hints

- Use `from typing import Optional, List, Dict, Any, Union`
- Type all function arguments and return values
- For Evennia dynamic attributes (e.g., `self.caller`, `self.location`), use `Any` or `# type: ignore` as needed to avoid type checker errors.

Example:

```python
def take_damage(attacker: "Character", amount: int) -> bool:
    """
    Inflicts damage from an attacker.
    """
    ...
```


#### Docstring Format

Use **Google-style** docstrings for all functions and classes. Use triple quotes for Python docstrings, not triple backticks.

Example:

```python
def heal_target(target: "Character", amount: int = 10) -> bool:
    """
    Heals a target character.

    Args:
        target: The character to heal.
        amount: The number of hit points to restore.

    Returns:
        True if healing was applied, False if invalid.

    Raises:
        ValueError: If target is None or dead.
    """
    ...
```

#### String capitalization

When returning names or titles, ensure they are properly capitalized for readability.
The `sentence_case` function can be used to ensure the first letter is capitalized.
This ONLY applies to a string at the beginning of a sentence, we do not want to capitalize every word in a string or a single word in the middle of a sentence.

```python

def sentence_case(name: str) -> str:
    """Capitalize the first letter of a string."""
    return name[0].upper() + name[1:] if name else ""

def get_character_name(character: "Character") -> str:
    return sentence_case(character.name)
```

### 2.3 Import Conventions

```python
# Standard library imports
import re
import time
from typing import Optional, Dict, List, Any

# Third-party libraries
from django.conf import settings

# Evennia flat API (preferred)
from evennia import DefaultObject, create_object, search_object
from evennia.utils import logger, evmenu, evtable
from evennia.commands.default.muxcommand import MuxCommand

# Direct Evennia module imports (when needed)
from evennia.objects.objects import DefaultRoom
```

---

### 2.4 File Organization

Evennia's library structure follows a clear hierarchy with the main library at `evennia/evennia/`:
- `__init__.py` - Contains "flat API" shortcuts to commonly used classes
- `commands/` - Command parser and default commands
- `objects/` - In-game entities (Objects, Characters, Rooms, Exits)
- `typeclasses/` - Abstract classes for database storage system
- `utils/` - Miscellaneous coding utilities
- `web/` - Web resources and webserver components
- `scripts/` - Background processes and timers
- `accounts/` - Out-of-game account management
- `contrib/` - Optional plugins and extensions
---

The game directory should follow this standard structure:
```
mygame/
├── commands/           # Custom commands and command sets
│   ├── command.py      # Base command classes
│   └── default_cmdsets.py  # Command groupings
├── server/            # Server configuration (don't modify structure)
│   ├── conf/          # Configuration files
│   │   ├── settings.py    # Main settings (copy from default_settings.py)
│   │   ├── at_initial_setup.py  # First-time setup
│   │   └── lockfuncs.py   # Custom lock functions
│   └── logs/          # Server log files
├── typeclasses/       # Database-bound entity templates
│   ├── objects.py     # Base object classes
│   ├── characters.py  # Character classes
│   ├── rooms.py       # Room classes
│   ├── exits.py       # Exit classes
│   └── scripts.py     # Script classes
├── web/              # Web interface customization
│   ├── template_overrides/  # HTML template overrides
│   └── static_overrides/    # CSS/JS overrides
└── world/            # Game-specific content
    ├── batch_cmds.ev  # Batch command files
    ├── prototypes.py  # Object prototypes
    └── help_entries.py  # Help system entries
```

---

## 3. Typeclass Design Patterns

### 3.1 Typeclass Inheritance & Hook Usage

- All in-game entities inherit from Evennia's `TypedObject` system.
- Use the appropriate base class:
  - `DefaultCharacter`, `DefaultRoom`, `DefaultObject`, `DefaultExit`, etc.
- Always use `super()` when overriding methods to preserve base functionality.
- Key lifecycle hooks:
  - `at_object_creation()` – called once when the object is first created.
  - `at_pre_move(destination)` – before a move occurs.
  - `at_post_move(source_location)` – after a move completes.
  - `at_object_receive(moved_obj, source_location)` – when something enters location.
  - `at_object_leave(moved_obj, destination)` – when something leaves location.
- Only override what you need; reuse parent behavior via `super()`.

```python
class MyCharacter(DefaultCharacter):
    """Custom character with additional behavior."""

    def at_object_creation(self):
        super().at_object_creation()
        self.db.hp = 100
        self.db.level = 1

    def at_pre_move(self, destination):
        if self.db.is_paralyzed:
            self.msg("You are paralyzed and cannot move!")
            return False
        return super().at_pre_move(destination)
```

---

### 3.2 Attribute Management

- Use `.db.attrname` for **persistent attributes**
- Use `.ndb.attrname` for **non-persistent, runtime data**
- Use `AttributeProperty` to expose db attributes as class properties

```python
from evennia import DefaultCharacter
from evennia.typeclasses.attributes import AttributeProperty

class Warrior(DefaultCharacter):
    """Character with clean property-based access."""
    strength = AttributeProperty(default=10)
    max_hp = AttributeProperty(default=100)

    @property
    def is_alive(self) -> bool:
        return self.db.hp > 0
```

- Use `obj.attributes.get("attr", default)` for safe reads
- Use `obj.attributes.has("attr")` to check existence
- Clean up via `at_object_delete()` if needed

---

### 3.3 Shared Behavior via Mixin Classes

Use mixins or shared base classes to avoid code duplication across typeclasses.

```python
class CombatMixin:
    """Adds combat methods to any object."""
    def take_damage(self, amount: int) -> None:
        self.db.hp = max(0, self.db.hp - amount)
        if self.db.hp == 0:
            self.die()

    def die(self) -> None:
        self.location.msg_contents(f"{self.key} has died.")
        self.delete()

class Enemy(CombatMixin, DefaultCharacter):
    """Enemy typeclass with combat behavior."""
    pass
```

- Prefer composition (via mixins) over copy-pasting logic
- Always inherit from `CombatMixin` before Evennia typeclass to preserve MRO

---

## 4. Command System

### 4.1 Command Design Rules

All Evennia commands inherit from `MuxCommand`.

#### Required fields:

- `key`: Primary command name (e.g. `"attack"`)
- `aliases`: Optional list of shorthand forms (e.g. `["atk"]`)
- `locks`: Permission string (e.g. `"cmd:all()"`, `"cmd:perm(Admin)"`)
- `help_category`: Display group for help (e.g. `"Combat"`)

#### Common methods:

- `func(self)` – required, executes command logic
- `parse(self)` – optional, used to split or pre-process arguments
- `caller`: The character or account issuing the command
- `self.args`: Raw arguments string
- `self.switches`: List of switches (e.g. `/quiet` → `"quiet"`)

#### Usage guidelines:

- Use `self.caller.search(target_str)` to find objects
- Use `self.caller.msg()` to send messages
- Always validate input and handle edge cases
- For multi-mode commands, use `self.switches` and helper methods

---

### 4.2 Command Template: `CmdCustom`

```python
from evennia import default_cmds

class CmdCustom(default_cmds.MuxCommand):
    """
    Set or view a custom attribute.

    Usage:
        custom <target> = <value>
        custom <target>
    """
    key = "custom"
    aliases = ["cust"]
    locks = "cmd:all()"
    help_category = "General"

    def parse(self):
        """Split input on '=' and strip whitespace."""
        if "=" in self.args:
            self.target_name, self.value = [s.strip() for s in self.args.split("=", 1)]
        else:
            self.target_name = self.args.strip()
            self.value = None

    def func(self):
        """Apply attribute or retrieve its value."""
        if not self.target_name:
            self.caller.msg("Usage: custom <target>[ = <value> ]")
            return

        target = self.caller.search(self.target_name)
        if not target:
            return  # search() handles error message

        if self.value is not None:
            target.db.custom_property = self.value
            self.caller.msg(f"Set {target.key}'s property to '{self.value}'.")
        else:
            value = target.db.custom_property or "not set"
            self.caller.msg(f"{target.key}'s property is: {value}")
```

#### Optional Enhancements

- Add `/switch` handling with `if "switch" in self.switches`
- Create helper methods for multi-mode logic:
  - `_handle_set_mode()`
  - `_handle_query_mode()`


## 5. Script System

Evennia scripts are background processes that manage time-based events, persistence, or recurring logic.

Scripts inherit from `DefaultScript` and are created with `create_script()`.

---

### 5.1 Script Lifecycle and Structure

Scripts define behavior through these hooks:

- `at_script_creation()` – Called once when the script is created
- `at_start()` – Called when the script starts running
- `at_repeat()` – Called every interval (if interval > 0)
- `at_stop()` – Called when the script ends or is deleted

#### Key properties:

- `self.key`: Unique identifier
- `self.desc`: Description (for admins/devs)
- `self.interval`: Time in seconds between `at_repeat()` calls
- `self.persistent`: If `True`, survives server reboots
- `self.start_delay`: If `True`, delay first `at_repeat()` call

#### Creation:

```python
from evennia import create_script

create_script("world.scripts.MyScript", key="heartbeat")
```

---

### 5.2 Script Template: Repeating Script

```python
from evennia import DefaultScript, logger
import time

class MyScript(DefaultScript):
    """
    Example repeating script that logs a heartbeat every 60 seconds.
    """

    def at_script_creation(self):
        """Set script parameters."""
        self.key = "heartbeat"
        self.desc = "Global game heartbeat"
        self.interval = 60
        self.persistent = True
        self.start_delay = False
        self.db.heartbeat_count = 0

    def at_repeat(self):
        """Executed every interval."""
        self.db.heartbeat_count += 1
        logger.log_info(f"Heartbeat #{self.db.heartbeat_count} at {time.time()}")
```

---

#### Best Practices

- Use `self.db` to store counters, timers, or state
- Always clean up or deactivate unused scripts
- For global scripts, set `persistent = True`
- For temporary effects (e.g. buffs), stop script when done:
  ```python
  if self.db.remaining <= 0:
      self.stop()
  ```

#### Script Access Patterns

- Attach to an object: `myobj.scripts.add(MyScript)`
- Check for script: `myobj.scripts.get("script_key")`
- Stop script: `myscript.stop()`
- Use custom tags to categorize scripts if needed

---

## 6. Menu System (EvMenu)

`EvMenu` is Evennia’s node-based in-game menu system. Use it for character creation, dialogs, NPC interaction, etc.

---

### 6.1 Menu Structure & Usage

To start a menu:

```python
from evennia.utils.evmenu import EvMenu

EvMenu(
    caller,
    "path.to.menu.module",
    startnode="start_node",
    session=caller.session,
    auto_look=True,
    auto_quit=True,
    cmd_on_exit=None
)
```

#### Node function signature:

Each menu node is a function that returns:

```python
def some_node(caller, raw_string, **kwargs):
    return "text to show", [option_dicts]
```

#### Option format:

Each option is a dictionary:

```python
{
    "key": "1",
    "desc": "Say hello",
    "goto": "next_node"  # or callable
}
```

---

### 6.2 Menu Node Example

```python
def node_welcome(caller, raw_string, **kwargs):
    """First menu node."""
    text = "|c=== Welcome to Character Creation ===|n\n\nChoose an option:"

    options = [
        {"key": "1", "desc": "Set name", "goto": "node_set_name"},
        {"key": "2", "desc": "Pick class", "goto": "node_choose_class"},
        {"key": "3", "desc": "Finish", "goto": "node_finish"}
    ]

    return text, options
```

---

### 6.3 Input Validation

For free-text input, use `_default` as the option key:

```python
def node_set_name(caller, raw_string, **kwargs):
    text = "Enter a character name (letters only, 3–20 chars):"

    def _validate_name(caller, input_string):
        name = input_string.strip()
        if len(name) < 3 or len(name) > 20 or not name.isalpha():
            caller.msg("Invalid name. Try again.")
            return "node_set_name"

        caller.ndb._menu_data["character_name"] = name
        caller.msg(f"Name set to: {name}")
        return "node_welcome"

    options = [
        {"key": "_default", "goto": _validate_name},
        {"key": "back", "desc": "Go back", "goto": "node_welcome"}
    ]

    return text, options
```

---

### 6.4 Menu Best Practices

- Store state in `caller.ndb._menu_data`
- Keep nodes small and focused
- Validate user input in `_default` handlers
- Use `auto_quit=True` to allow `q` to exit menu
- Use `goto` functions for complex branching
- Avoid deep nesting — flatten logic when possible
- Clean up `ndb` state at the end (`del caller.ndb._menu_data`)

---

## 7. Table System (EvTable)

Use `EvTable` to format tabular in-game output: inventories, stats, leaderboards, etc.

---

### 7.1 Basic Table Usage

Import and create:

```python
from evennia.utils.evtable import EvTable

table = EvTable("Name", "Level", "HP", table_type="box")
table.add_row("Alice", 5, 100)
table.add_row("Bob", 3, 75)
caller.msg(table)
```

#### Table Parameters:

- `table_type`: `"box"`, `"default"`, `"grid"`, etc.
- `width`: Total character width of the table
- `align`: `"l"`, `"c"`, or `"r"` per column
- `header_line_char`, `border_top_char`, etc. – for custom styles

---

### 7.2 Dynamic Stat Table Example

```python
def display_character_stats(caller):
    character = caller
    stats = character.db.stats or {}

    table = EvTable("Stat", "Value", "Modifier", table_type="box")

    for stat, value in stats.items():
        mod = (value - 10) // 2
        mod_str = f"+{mod}" if mod >= 0 else str(mod)
        table.add_row(stat.title(), value, mod_str)

    caller.msg(table)
```

---

### 7.3 Inventory Table with Color Coding

```python
def show_inventory(caller):
    table = EvTable("Item", "Type", "Weight", "Value", table_type="box", width=80)

    for item in caller.contents:
        item_type = item.db.item_type or "misc"
        color_map = {
            "weapon": "|r", "armor": "|b", "potion": "|g", "misc": "|w"
        }
        color = color_map.get(item_type, "|w")
        table.add_row(
            f"{color}{item.get_display_name(caller)}|n",
            item_type.title(),
            f"{item.db.weight or 0} lbs",
            f"{item.db.value or 0} gp"
        )

    caller.msg(table)
```

---

### 7.4 Paginated Table Display

```python
from evennia.utils.evmore import EvMore

def show_large_table(caller, data_rows):
    table = EvTable("Name", "Class", "Level", table_type="box")

    for row in data_rows:
        table.add_row(row["name"], row["class"], str(row["level"]))

    EvMore(caller, str(table))
```

---

### 7.5 Best Practices

- Use EvTable for structured, readable lists
- Color-code meaningful fields (HP, rarity, status)
- For long output, use `EvMore` to paginate
- Use `EvTable(..., width=N)` to control wrapping
- Align columns with `align="l"` (left), `"r"` (right), or `"c"` (center)
- Truncate text in fixed-width tables for clean layout

---

## 8. Prototypes & Object Creation

Evennia prototypes define structured blueprints for creating in-game objects (NPCs, items, etc.).

---

### 8.1 Defining Object Prototypes

Prototypes are dictionaries usually defined in `world/prototypes.py`.

```python
GOBLIN_WARRIOR = {
    "key": "goblin warrior",
    "typeclass": "typeclasses.characters.NPC",
    "desc": "A snarling goblin armed with a rusty blade.",
    "attrs": [
        ("hp", 25),
        ("strength", 12),
        ("aggressive", True)
    ],
    "tags": [
        ("npc", "race"),
        ("goblin", "faction")
    ]
}
```

#### Prototype fields:

- `"key"`: Object name
- `"typeclass"`: Full path to object class
- `"desc"`: Description text
- `"attrs"`: List of `(name, value)` persistent attributes
- `"tags"`: Optional tags for search/categorization

---

### 8.2 Spawning Prototypes

Use `spawn()` from Evennia’s prototyping system.

```python
from evennia.prototypes.spawner import spawn

spawned = spawn(GOBLIN_WARRIOR)
```

Or spawn by string key (from batch-loaded prototype storage):

```python
spawned = spawn("goblin warrior")
```

- Returns a list of created objects
- You can attach a prototype to any object using `obj.prototype = my_proto_dict`

---

### 8.3 Creating Objects Programmatically

For custom objects, use `create_object()`:

```python
from evennia import create_object

obj = create_object(
    "typeclasses.objects.Weapon",
    key="iron sword",
    location=caller.location,
    home=caller.location
)

obj.db.damage = 8
obj.tags.add("weapon", category="item_type")
```

- Prefer `create_object()` over direct class instantiation
- Set `location` and `home` when spawning in-world
- Use `.db.` to assign attributes, `.tags.add()` to categorize

---

### 8.4 Object Creation Best Practices

- Use prototypes for reusable entities (NPCs, items, etc.)
- Use tags to group or identify objects
- Always set `key`, `typeclass`, and `desc`
- Store all game data in `db` attributes, not directly on the object
- Avoid hardcoding repeated values — use prototypes instead

---

## 9. Web Integration

Evennia includes a Django-based web framework for exposing game data via web pages or APIs.

---

### 9.1 Custom Django Views

You can extend Evennia’s web portal with your own Django views and templates.

#### Define a view:

```python
from django.shortcuts import render
from evennia.objects.models import ObjectDB

def character_sheet(request, character_name):
    character = ObjectDB.objects.get(db_key=character_name)
    context = {
        "character": character,
        "stats": character.db.stats or {},
        "inventory": character.contents
    }
    return render(request, "character_sheet.html", context)
```

#### Add to `urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("character/<str:character_name>/", views.character_sheet, name="character_sheet")
]
```

---

### 9.2 REST API Usage

Evennia includes an optional REST API for external access.

- Base URL: `/api/`
- Endpoints:
  - `/api/objects/`: In-game objects
  - `/api/accounts/`: Player accounts
  - `/api/scripts/`: Scripts
- Requires Token-based authentication (via Django REST framework)

To enable:
- Install `djangorestframework`
- Add `rest_framework` to `INSTALLED_APPS` in `settings.py`
- Configure token auth or permissions

See [Evennia REST API docs](https://www.evennia.com/docs/latest/Web-API.html) for full setup.

---

### 9.3 Web Client Customization

To customize the browser-based client:

- Override templates:
  - Place custom HTML in `web/template_overrides/`
- Customize static assets:
  - Place CSS/JS in `web/static_overrides/`
- Useful files to override:
  - `webclient_base.html`
  - `styles.css`, `client.js`

Use these to change:
- Colors, fonts, and layout
- Input box behavior
- Tab structure or sound effects

---

### 9.4 Web Integration Best Practices

- Sanitize all web-facing input (usernames, fields)
- Use Django’s built-in CSRF protection
- Keep views small and scoped
- Use Evennia’s ORM to avoid raw SQL
- Avoid exposing sensitive attributes (e.g. `.db.password`)
- Use `@login_required` for restricted pages

---

## 10. Testing & Validation

---

### 10.1 DEBUG Logging

- Debug logging is essential for tracking issues during development.
- Use a global `DEBUG_MODE` flag to toggle debug output.
- Define a `debug_log` function to handle debug messages.

```python
# Debugging setup (consistent with other files)
DEBUG_MODE = False  # Set to True to enable debug logging

def debug_log(msg: str) -> None:
    if DEBUG_MODE:
        print(msg)

# Example usage in a function
def some_function(character):
    debug_log(f"DEBUG: Inside some_function, character is: {character}")
```

---

### 10.2 Unit Testing with Evennia

Use Evennia’s `EvenniaTest` base class for in-game unit tests.

```python
from evennia.utils.test_resources import EvenniaTest

class TestCombat(EvenniaTest):
    def setUp(self):
        super().setUp()
        self.char = self.create_object("typeclasses.characters.Character", key="Fighter")

    def test_health_initialization(self):
        self.assertEqual(self.char.db.hp, 100)
```

#### Guidelines:

- Use `create_object()` for test fixtures
- Test both **positive** and **edge** cases
- Use `unittest.mock` to mock game state or dependencies
- Clean up using `tearDown()` or rely on `EvenniaTest`

---

### 10.3 Integration Testing

Write integration tests to simulate real-world game flows.

```python
class TestRoomMovement(EvenniaTest):
    def test_movement(self):
        char = self.create_object("typeclasses.characters.Character", key="Hero")
        room1 = self.create_object("typeclasses.rooms.Room", key="Room1")
        room2 = self.create_object("typeclasses.rooms.Room", key="Room2")

        char.move_to(room1, quiet=True)
        char.move_to(room2, quiet=True)

        self.assertEqual(char.location, room2)
```

---

### 10.4 Testing Best Practices

- Test commands, scripts, and typeclasses
- Mock database queries and expensive lookups
- Use `assert` for logic checks, not print statements
- Keep test functions short and focused
- Use descriptive test names (e.g. `test_cannot_attack_when_paralyzed`)
- Group related tests in the same class or module

---

## 11. Security & Access Control

---

### 11.1 Input Validation

Always validate and sanitize user input — both in-game and via web/API.

#### Character name validation example:

```python
import re

def validate_character_name(name: str) -> bool:
    if not name or not name.strip():
        return False
    name = name.strip()
    if len(name) < 3 or len(name) > 20:
        return False
    if not re.match(r"^[a-zA-Z][a-zA-Z0-9_]*$", name):
        return False
    if name.lower() in {"admin", "system", "null"}:
        return False
    return True
```

#### Input sanitization example:

```python
from html import escape

def sanitize_user_input(text: str) -> str:
    if not text:
        return ""
    text = escape(text)
    text = re.sub(r"<script[^>]*>.*?</script>", "", text, flags=re.IGNORECASE | re.DOTALL)
    return text[:4000]
```

---

### 11.2 Lock Strings & Permission Control

Use Evennia's built-in lock system for access control.

#### Common lock types:

- `puppet`: Who can control the object
- `cmd`: Who can use a command
- `get`, `drop`, `give`: Object interactions
- `delete`, `examine`, `attredit`: Admin actions

#### Example lock setup:

```python
obj.locks.add("puppet:perm(Player) and pid(123)")
obj.locks.add("get:perm(Player)")
obj.locks.add("delete:perm(Admin)")
```

Use `id({obj.id})` and `pid({obj.account.id})` for dynamic lock evaluation.

---

### 11.3 Custom Lock Functions

Define custom lock logic in `server/conf/lockfuncs.py`.

```python
def lockfunc_has_item(accessing_obj, accessed_obj, *args, **kwargs):
    """Returns True if accessing_obj has a specific item."""
    if not args:
        return False
    item_key = args[0].lower()
    return any(obj.key.lower() == item_key for obj in accessing_obj.contents)
```

Then use in a lock string:

```python
obj.locks.add("open:has_item(keyring)")
```

---

### 11.4 Rate Limiting Example

Use a simple per-minute rate limit for spam protection.

```python
import time

def check_rate_limit(caller, key: str, max_per_minute: int = 10) -> bool:
    now = time.time()
    store = caller.ndb.get(key, [])
    store = [t for t in store if now - t < 60]
    if len(store) >= max_per_minute:
        return False
    store.append(now)
    caller.ndb[key] = store
    return True
```

---

### 11.5 Security Best Practices

- Never use `eval()` on user input
- Sanitize all text stored in attributes or sent to web views
- Use `locks` to enforce command-level permissions
- Hide sensitive attributes (passwords, internal flags)
- Log suspicious activity using `logger.log_err()`
- Avoid exposing object internals directly via API/web
- Require authentication for all admin or builder endpoints

---

## 12. Performance Optimization

---

### 12.1 Efficient Database Queries

Avoid unnecessary database hits — especially in loops.

#### ❌ Bad (N+1 queries):

```python
for item in caller.contents:
    print(item.db.weight)
```

Each `item.db.weight` may trigger a separate DB read.

#### ✅ Better:

```python
items = caller.contents.prefetch_related("db_attributes")
for item in items:
    weight = item.db.weight or 0
    print(weight)
```

---

### 12.2 Caching Expensive Operations

Use `functools.lru_cache` for expensive or repeated calculations.

```python
from functools import lru_cache

class OptimizedCharacter(DefaultCharacter):
    @lru_cache(maxsize=128)
    def get_combat_stats(self) -> dict:
        base = self.db.stats or {}
        bonus = self._calculate_equipment_bonuses()
        return {k: base.get(k, 0) + bonus.get(k, 0) for k in base}

    def invalidate_stats_cache(self) -> None:
        self.get_combat_stats.cache_clear()
```

---

### 12.3 Lazy Evaluation

Use generators for large result sets:

```python
def get_heavy_objects(objects):
    return (obj for obj in objects if (obj.db.weight or 0) > 50)
```

---

### 12.4 Memory Management

For long-running scripts:

- Use chunked processing (`for i in range(0, len(...), step)`)
- Call `gc.collect()` periodically in heavy loops
- Use `__slots__` in memory-heavy classes (advanced use only)

---

### 12.5 Loop Optimization

Minimize `db` access and sorting inside loops.

#### Instead of:

```python
for item in items:
    val = item.db.value
    total += val
```

#### Do:

```python
item_values = [item.db.value or 0 for item in items]
total = sum(item_values)
```

---

### 12.6 Performance Best Practices

- Avoid repeated `.db` reads inside loops
- Use `prefetch_related()` when querying attributes or related models
- Cache computed properties if they don’t change often
- Use `EvMore()` for long text output (avoid lag from `msg()` spamming)
- Profile slow commands with timing logs
- Avoid frequent script creation/destruction — reuse when possible

---

## 13. Documentation & Comments

---


### 13.1 Google-Style Docstrings (Standard)

Use **Google-style docstrings** for all public functions, methods, and classes. Use triple quotes for Python docstrings.

#### Function Example:

```python
def calculate_damage(attacker: "Character", base: int) -> int:
    """
    Calculate combat damage from an attacker.

    Args:
        attacker: The character performing the attack.
        base: The base damage before modifiers.

    Returns:
        Final damage value as an integer.

    Raises:
        ValueError: If attacker is invalid or base is negative.
    """
```

#### Class Example:

```python
class QuestTracker:
    """
    Tracks player quest progress and completion.

    Attributes:
        quests (dict): Active and completed quest IDs.
        xp_rewarded (int): Total experience granted via quests.

    Methods:
        add_quest(): Assigns a new quest.
        complete_quest(): Marks a quest as done and rewards XP.
    """
```

---

### 13.2 Inline Comments

- Use inline comments for non-obvious logic.
- Prefer complete sentences when describing complex branches or conditions.
- Keep comments up-to-date with the code.

#### Example:

```python
# Skip unconscious NPCs during combat loop
if not combatant.db.is_conscious:
    continue
```

---

### 13.3 README and Developer Docs

For each major subsystem (e.g. `combat/`, `crafting/`, `quests/`):

- Create a `README.md` in the folder
- Document:
  - Purpose
  - Key classes/functions
  - Extension hooks
  - Known limitations

**When updating or refactoring code, always update the related documentation and docstrings accordingly.**
Outdated documentation is considered a bug.

---

### 13.4 Code Quality Checklist

Use this checklist before committing code:

- [ ] All functions have type hints
- [ ] All public functions/classes have docstrings
- [ ] Variables are descriptively named
- [ ] No magic numbers — use named constants
- [ ] All exceptions are specific (`except ValueError`, not `except:`)
- [ ] Test coverage exists
- [ ] Code is DRY (no duplication)
- [ ] Comments match actual behavior
- [ ] Input validation is in place
- [ ] Performance and security reviewed

---

## 14. Copilot Instruction Patterns

These meta-instructions guide GitHub Copilot’s behavior in this project.

---

### 14.1 Copilot-Friendly Coding Rules

#### General Python Rules:

- Always use **PEP8**, **Google-style docstrings**, and **type hints**
- Use **f-strings**, not `%` or `.format()`
- Catch **specific exceptions**, not bare `except`
- Use **descriptive variable names** and **small functions**
- Organize imports by: standard lib → third-party → Evennia flat API → local modules
- Avoid deep nesting — prefer guard clauses and early returns

#### Evennia-Specific Rules:

- Use `DefaultObject`, `DefaultCharacter`, `DefaultRoom`, etc., as base classes
- Hook overrides (`at_object_creation`, `at_repeat`, etc.) must call `super()`
- Use `.db.` for persistent attributes and `.ndb.` for temporary state
- Use `caller.search()` to find in-game objects
- Use `caller.msg()` or `self.caller.msg()` for output
- Use `locks` on commands and objects for permissions
- Store prototype templates in `world/prototypes.py`
- Reference objects by `key`, not hardcoded database IDs

---

### 14.2 General Behavior Heuristics

Copilot should infer the following common patterns:

- All commands inherit from `MuxCommand` and define `func()`
- All menus are node-based, returning `(text, options)` tuples
- All scripts subclass `DefaultScript` and use `at_repeat()`, `at_script_creation()`, etc.
- All in-game entities use Evennia’s attribute system for storage (`.db`, `.ndb`)
- Table output uses `EvTable(...)` and is sent via `caller.msg()`
- Objects are created using `create_object()` or `spawn()`
- All player input must be validated before use
- All code changes require matching updates to documentation and comments

---

### 14.3 Copilot Prompt Formatting Tips (Internal Use)

Copilot understands structured prompts like:

- “Create a typeclass for an enemy that has HP and dies when HP reaches 0”
- “Add a command that lets players use a potion if it's in their inventory”
- “Show a paginated EvTable of all quests tagged with ‘active’”
- “Make an EvMenu that asks for a name and validates it”

Use concise, goal-oriented prompts to get clean completions.

---

## Appendix A: Full Command Example – `CmdCustom`

This is a full example of a flexible Evennia command using `MuxCommand`.

```python
from evennia import default_cmds

class CmdCustom(default_cmds.MuxCommand):
    """
    Set or retrieve a custom attribute on an object.

    Usage:
        custom <target> = <value>
        custom <target>

    Examples:
        custom torch = lit
        custom torch
    """
    key = "custom"
    aliases = ["cust"]
    locks = "cmd:all()"
    help_category = "General"

    def parse(self):
        """
        Parse the input into self.target and self.value.
        """
        if "=" in self.args:
            self.target_name, self.value = [part.strip() for part in self.args.split("=", 1)]
        else:
            self.target_name = self.args.strip()
            self.value = None

    def func(self):
        """
        Set or retrieve a 'custom_property' on the target object.
        """
        if not self.target_name:
            self.caller.msg("Usage: custom <target>[ = <value> ]")
            return

        target = self.caller.search(self.target_name)
        if not target:
            return  # Error already handled by search()

        if self.value:
            target.db.custom_property = self.value
            self.caller.msg(f"You set |c{target.key}|n's custom property to |y{self.value}|n.")
        else:
            current = target.db.custom_property
            if current:
                self.caller.msg(f"|c{target.key}|n has custom property: |y{current}|n.")
            else:
                self.caller.msg(f"|c{target.key}|n has no custom property set.")
```

### Command Features:

- Uses `MuxCommand` base for Evennia command integration
- Supports both `set` and `query` modes based on input
- Performs argument parsing in `parse()`
- Validates search results using `caller.search()`
- Sends formatted output using ANSI color codes
- Can be extended with switches (`self.switches`) or refactored into helper methods

---

## Appendix B: Full Script Example – `HeartbeatScript`

This is a full example of a global, repeating Evennia script that tracks and logs a server heartbeat.

```python
from evennia import DefaultScript, logger
import time

class HeartbeatScript(DefaultScript):
    """
    A global script that logs a 'heartbeat' message every 60 seconds.
    Useful for tracking uptime, performance, or background monitoring.
    """

    def at_script_creation(self):
        """
        This hook runs only once when the script is first created.
        """
        self.key = "heartbeat_script"
        self.desc = "Logs a heartbeat every 60 seconds"
        self.interval = 60  # seconds between repeats
        self.start_delay = False
        self.persistent = True
        self.db.heartbeat_count = 0  # Persistent counter

    def at_start(self):
        """
        Called when the script is first started or restarted.
        """
        logger.log_info("Heartbeat script started.")

    def at_repeat(self):
        """
        Called every `interval` seconds.
        """
        self.db.heartbeat_count += 1
        logger.log_info(f"[HEARTBEAT] Tick #{self.db.heartbeat_count} at {time.time()}")

    def at_stop(self):
        """
        Called when the script is stopped or deleted.
        """
        logger.log_info("Heartbeat script stopped.")
```

### How to start the script:

```python
from evennia import create_script
create_script("world.scripts.HeartbeatScript", key="heartbeat_script")
```

---

### Script Features:

- `interval = 60`: Repeats every 60 seconds
- Logs with `logger.log_info()` for visibility
- Tracks number of ticks in `self.db.heartbeat_count`
- Persists across reloads with `persistent = True`
- Clean logging on start and stop
- Modular — can be subclassed or extended (e.g. for metrics)

---

## Appendix C: Full EvMenu Sequence – Character Builder

This example defines a simple multi-node character creation menu using `EvMenu`.

```python
from evennia.utils.evmenu import EvMenu

def start_character_creation(caller):
    """
    Launches the character builder menu.
    """
    caller.ndb._menu_data = {}
    EvMenu(caller, "world.menus.characterbuilder", startnode="node_welcome", auto_quit=True)
```

---

### Node: Welcome

```python
def node_welcome(caller, raw_string, **kwargs):
    text = "|c=== Character Builder ===|n\n\nChoose an option to begin:"
    options = [
        {"key": "1", "desc": "Set Name", "goto": "node_set_name"},
        {"key": "2", "desc": "Choose Class", "goto": "node_choose_class"},
        {"key": "3", "desc": "Confirm and Finish", "goto": "node_confirm"}
    ]
    return text, options
```

---

### Node: Set Name (Text Input Validation)

```python
def node_set_name(caller, raw_string, **kwargs):
    text = "Enter your character's name (letters only, 3–20 characters):"

    def _validate_name(caller, input_string):
        name = input_string.strip()
        if not name.isalpha() or not (3 <= len(name) <= 20):
            caller.msg("Invalid name. Please enter only letters, 3–20 characters long.")
            return "node_set_name"
        caller.ndb._menu_data["name"] = name
        caller.msg(f"Name set to: |c{name}|n")
        return "node_welcome"

    options = [
        {"key": "_default", "goto": _validate_name},
        {"key": "back", "desc": "Return to menu", "goto": "node_welcome"}
    ]
    return text, options
```

---

### Node: Choose Class

```python
def node_choose_class(caller, raw_string, **kwargs):
    text = "Choose your class:"
    options = [
        {"key": "1", "desc": "Warrior", "goto": (set_class, "Warrior")},
        {"key": "2", "desc": "Rogue", "goto": (set_class, "Rogue")},
        {"key": "3", "desc": "Mage", "goto": (set_class, "Mage")},
        {"key": "back", "desc": "Return to menu", "goto": "node_welcome"}
    ]
    return text, options

def set_class(caller, choice):
    caller.ndb._menu_data["class"] = choice
    caller.msg(f"Class set to: |c{choice}|n")
    return "node_welcome"
```

---

### Node: Confirm

```python
def node_confirm(caller, raw_string, **kwargs):
    name = caller.ndb._menu_data.get("name", "<not set>")
    cls = caller.ndb._menu_data.get("class", "<not set>")

    text = f"""|c=== Confirm Character ===|n

Name : |y{name}|n
Class: |y{cls}|n

Proceed with this character?
"""
    options = [
        {"key": "yes", "desc": "Confirm and finish", "goto": complete_builder},
        {"key": "no", "desc": "Back to main menu", "goto": "node_welcome"}
    ]
    return text, options

def complete_builder(caller):
    name = caller.ndb._menu_data.get("name", "Unnamed")
    cls = caller.ndb._menu_data.get("class", "Peasant")
    # Apply the selections to the character, save, etc.
    caller.msg(f"|gCharacter creation complete: {name}, the {cls}.|n")
    del caller.ndb._menu_data
    return None  # Ends the menu
```

---

### Menu Features:

- Each menu node is small, testable, and readable
- Uses `_default` input for free-text validation
- Tracks state in `caller.ndb._menu_data`
- Modular: nodes can be reused in other menus
- Uses `auto_quit=True` to allow `q` to exit gracefully

---

## Appendix D: Advanced EvTable Configurations

This appendix contains real-world `EvTable` patterns for rendering inventories, combat stats, and paginated data with style and clarity.

---

### Example 1: Color-Coded Inventory Table

```python
from evennia.utils.evtable import EvTable

def show_inventory(caller):
    table = EvTable("Item", "Type", "Qty", "Value", table_type="box", width=78)

    color_map = {
        "weapon": "|r",
        "armor": "|b",
        "potion": "|g",
        "misc": "|w"
    }

    for item in caller.contents:
        item_type = item.db.item_type or "misc"
        color = color_map.get(item_type, "|w")
        name = f"{color}{item.get_display_name(caller)}|n"
        qty = item.db.quantity or 1
        value = item.db.value or 0

        table.add_row(name, item_type.title(), str(qty), f"{value} gp")

    caller.msg(table)
```

---

### Example 2: Conditional Column Display

```python
def show_equipment(caller):
    table = EvTable("Slot", "Item", "Enchanted?", table_type="grid", align="l")

    for slot in ("head", "body", "hands", "feet"):
        item = caller.db.equipment.get(slot)
        if not item:
            table.add_row(slot.title(), "(empty)", "—")
        else:
            is_enchanted = "|gYes|n" if item.db.is_enchanted else "|rNo|n"
            table.add_row(slot.title(), item.key, is_enchanted)

    caller.msg(table)
```

---

### Example 3: Two-Column Stat Block (Minimap Style)

```python
def show_stat_block(caller):
    stats = caller.db.stats or {}
    left = EvTable("Stat", "Value", table_type="simple", border="none")
    right = EvTable("Stat", "Value", table_type="simple", border="none")

    stat_items = list(stats.items())
    half = len(stat_items) // 2 + len(stat_items) % 2
    for i, (stat, val) in enumerate(stat_items):
        target_table = left if i < half else right
        target_table.add_row(stat.title(), val)

    combined = EvTable(border="none")
    combined.add_row(left, right)
    caller.msg(combined)
```

---

### Example 4: Paginated Output (EvMore)

```python
from evennia.utils.evmore import EvMore

def show_long_table(caller):
    table = EvTable("Name", "Level", "Status", table_type="box")

    for npc in caller.location.contents:
        if not npc.is_typeclass("typeclasses.characters.NPC", exact=False):
            continue
        name = npc.key
        level = npc.db.level or 1
        status = "|rHostile|n" if npc.db.aggressive else "|gNeutral|n"
        table.add_row(name, level, status)

    EvMore(caller, str(table))
```

---

### Best Practices Summary:

- Use `table_type="box"` or `"grid"` for structured output
- Use `width` and `align` to control layout
- Apply color with Evennia’s ANSI tags (`|g`, `|r`, `|y`, etc.)
- For long tables, wrap with `EvMore()`
- Combine multiple `EvTable`s using `.add_row(left, right)`
- Use conditional logic to show “(empty)” or status indicators

---

## Appendix E: Evennia ANSI Color & Style Codes

Evennia supports inline ANSI-style formatting tags for colored and styled in-game text.

These tags can be embedded in strings sent via `caller.msg()`, `EvTable`, `EvMenu`, etc.

---

### Foreground Colors

| Tag | Color     | Example Output              |
|-----|-----------|-----------------------------|
| `|r` | Red       | `|rRed Text|n`              |
| `|g` | Green     | `|gGreen Text|n`            |
| `|y` | Yellow    | `|yYellow Text|n`           |
| `|b` | Blue      | `|bBlue Text|n`             |
| `|m` | Magenta   | `|mMagenta Text|n`          |
| `|c` | Cyan      | `|cCyan Text|n`             |
| `|w` | White     | `|wWhite Text|n`            |
| `|x` | Black     | `|xBlack Text|n`            |
| `|n` | Reset     | `|rRed|n Normal`            |

---

### Bright Foreground Colors

| Tag  | Color       |
|------|-------------|
| `|R` | Bright Red  |
| `|G` | Bright Green|
| `|Y` | Bright Yellow |
| `|B` | Bright Blue |
| `|M` | Bright Magenta |
| `|C` | Bright Cyan |
| `|W` | Bright White |
| `|X` | Bright Black |

---

### Background Colors

| Tag  | Color         |
|------|---------------|
| `|[r` | Red Background |
| `|[g` | Green Background |
| `|[y` | Yellow Background |
| `|[b` | Blue Background |
| `|[m` | Magenta Background |
| `|[c` | Cyan Background |
| `|[w` | White Background |
| `|[x` | Black Background |

---

### Text Styles

| Tag  | Style          |
|------|----------------|
| `|u` | Underline      |
| `|i` | Italic         |
| `|h` | Highlight (bold or bright) |
| `|f` | Flash/Blink    |
| `|N` | Reset all styles and colors |

---

### Common Formatting Examples

```python
caller.msg("|cYou have |r10 HP|n remaining!")
caller.msg("|yYou equip the |wIron Sword|n.")
caller.msg("|gSuccess!|n Your potion healed |g+50 HP|n.")
```

Always end styled segments with `|n` or `|N` to reset.

---

### Best Practices

- Use sparingly — avoid over-formatting.
- Use colors consistently (e.g. red = danger, green = success).
- Don’t rely on color alone for meaning — support plain text readability.
- Combine with `EvTable` columns or `EvMenu` options to enhance clarity.

---

## Appendix F: Combat Mixin / Shared Logic

This mixin defines reusable combat behavior that can be inherited by any typeclass.

---

### CombatMixin: Base Logic

```python
class CombatMixin:
    """
    Reusable combat logic for any object.

    Methods:
        take_damage(): Reduces HP and calls die() if needed.
        die(): Handles object death (messaging, cleanup, etc.).
    """

    def take_damage(self, amount: int, attacker=None) -> None:
        """
        Apply damage and handle death.

        Args:
            amount (int): Amount of HP to subtract.
            attacker (optional): Object that dealt the damage.
        """
        hp = self.db.hp or 0
        new_hp = max(0, hp - amount)
        self.db.hp = new_hp

        self.location.msg_contents(
            f"|r{self.key} takes {amount} damage! (HP: {new_hp})|n"
        )

        if new_hp <= 0:
            self.die(attacker=attacker)

    def die(self, attacker=None) -> None:
        """
        Handle death of this object.

        Args:
            attacker (optional): Who killed this object.
        """
        death_msg = f"|r{self.key} has died!|n"
        if attacker:
            attacker.msg(f"You have defeated |r{self.key}|n.")
        self.location.msg_contents(death_msg)
        self.delete()
```

---

### Using the Mixin in an NPC

```python
from evennia import DefaultCharacter
from world.mixins.combat import CombatMixin

class Enemy(CombatMixin, DefaultCharacter):
    """
    An enemy character with HP and combat behavior.
    """
    def at_object_creation(self):
        super().at_object_creation()
        self.db.hp = 20
        self.db.level = 1
```

---

### Optional Extensions

- Add `attack(target)` method
- Track `last_attacker`, `combat_state`
- Add `resurrect()` or `on_death_loot()` hook
- Move combat messages to `EvTable` or `EvMenu` interactions

---

### Best Practices

- Keep mixins stateless or lightweight — use `.db` for data
- Use `super()` in typeclasses to ensure hook chaining
- Test mixin behavior with unit tests using `EvenniaTest`
- Isolate combat from command parsing — commands should call the mixin methods

---

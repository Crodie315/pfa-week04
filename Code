"""
Ball Snake - a snake made of balls, in the terminal, drawn as pixel art.

Run:   python snake_game.py

Rules:
  - The snake is made of green balls. Steer it to eat the fruit.
  - Each fruit eaten adds one ball to the body and makes the snake faster.
  - About 15 brown stones are scattered around. Every time the snake eats,
    the stones move to new places and their number changes a little.
  - Hitting a stone costs a life and smashes it. 3 hits and the snake dies.
  - The edges wrap around: leave on one side, come back on the other.

Keys:
  W A S D or arrow keys   change direction
  SPACE                   pause / unpause
  Q                       quit
  Game Over menu:         W/S or arrows to choose, ENTER to confirm

How the drawing works:
  The screen is a grid of tiny colored "pixels". The terminal character
  "▀" (upper half block) shows two pixels at once: the top half uses the
  text color and the bottom half uses the background color. Each game
  square is 3 x 3 pixels. The snake's head and body balls are drawn a
  little bigger (5 x 5) and overlap like beads.

  Tip: to change the size of everything, make the Terminal's font
  smaller or bigger
  (Terminal Settings -> your profile -> Appearance -> Font size).
"""

import os
import sys
import time
import shutil
import random

# --- Settings you can change ------------------------------------------------
LIVES = 3              # stone hits before the snake dies
MIN_STONES = 13        # each time stones are placed, there will be
MAX_STONES = 17        #   a random number between these two
NUM_FOOD = 3           # fruits on the field at once
START_LENGTH = 4       # balls in the snake at the start

START_DELAY = 0.15     # seconds per step at the start (bigger = slower)
SPEEDUP = 0.95         # each fruit multiplies the delay by this
MIN_DELAY = 0.04       # top speed limit

# --- Colors (red, green, blue) ----------------------------------------------
PALETTE = {
    # snake (light comes from the top-left)
    "L": (190, 245, 175),   # bright shine
    "H": (110, 200, 100),   # lit green
    "G": (65, 165, 75),     # green
    "g": (45, 125, 60),     # dark band (every other ball)
    "D": (22, 80, 40),      # shadow side
    "p": (195, 205, 90),    # yellow scale pattern
    "E": (255, 245, 205),   # eye (bright cream)
    "K": (10, 10, 10),      # pupil
    "M": (20, 55, 28),      # closed mouth
    "R": (110, 15, 30),     # inside of the open mouth
    "r": (215, 65, 90),     # pink of the mouth
    "W": (250, 250, 240),   # fangs
    "t": (235, 60, 100),    # tongue
    # fruit
    "a": (255, 145, 145),   # apple shine
    "A": (210, 35, 45),     # apple red
    "d": (135, 15, 30),     # apple shadow
    "q": (255, 205, 120),   # orange shine
    "o": (240, 140, 30),    # orange
    "x": (180, 85, 15),     # orange shadow
    "y": (255, 250, 195),   # lemon shine
    "Y": (245, 215, 50),    # lemon
    "v": (190, 155, 25),    # lemon shadow
    "e": (100, 190, 60),    # leaf
    "s": (95, 60, 30),      # stem
    # stone
    "h": (210, 178, 140),   # highlight
    "n": (160, 118, 80),    # light brown
    "B": (120, 86, 55),     # brown
    "b": (80, 56, 36),      # dark brown
    "k": (48, 33, 22),      # deepest shadow
    "m": (85, 115, 55),     # a bit of moss
}

# When the snake hits a stone, its greens turn red for a moment.
HURT = {"L": (255, 180, 170), "H": (240, 110, 100), "G": (210, 60, 60),
        "g": (165, 40, 45), "D": (110, 25, 30), "p": (240, 160, 120)}

SHADOW = 0.55      # how dark the shadows on the ground are (smaller = darker)

# Ground colors (the background)
GROUND = (24, 30, 24)        # main ground
GROOVE = (13, 17, 13)        # dark groove line
GROOVE_LIP = (36, 45, 35)    # light edge under a groove
SPECK_DARK = (18, 23, 18)
SPECK_LIGHT = (33, 41, 32)
GRASS = (40, 72, 40)

# --- Sprites: "." = see-through ----------------------------------------------
# Each game square is 3 x 3 pixels. The head and body balls are 5 x 5, a bit
# bigger than a square, so they overlap like beads. Heads are 7 x 7 (room for
# the tongue) and drawn facing RIGHT; the game turns them the other ways.
HEAD_CLOSED = [
    ".......",
    "..GGG..",
    ".GHEKG.",
    ".GLHGM.",
    ".GHEKG.",
    "..GGG..",
    ".......",
]
HEAD_TONGUE = [              # closed mouth, tongue flicking out
    ".......",
    "..GGG..",
    ".GHEKG.",
    ".GLHGMt",
    ".GHEKG.",
    "..GGG..",
    ".......",
]
HEAD_OPEN = [                # jaws apart: fangs and the pink mouth show
    ".......",
    "..GGGG.",
    ".GHEKW.",
    ".GLRr..",
    ".GHEKW.",
    "..GGGG.",
    ".......",
]
BODY = [
    ".HHG.",
    "HLHGG",
    "HHGGD",
    "GGGDD",
    ".DDD.",
]
BODY_SPOT = [                # every other ball: darker band with a scale spot
    ".gHg.",
    "gHggg",
    "ggpgD",
    "gggDD",
    ".DDD.",
]
TAIL = [                     # last ball is smaller
    ".H.",
    "HGD",
    ".D.",
]
FRUITS = [
    [                        # apple
        ".se",
        "aAA",
        "AAd",
    ],
    [                        # orange
        ".e.",
        "qoo",
        "oox",
    ],
    [                        # lemon
        "..e",
        "yYY",
        "YYv",
    ],
]
STONES = [
    [
        "hn.",
        "nBb",
        "Bbk",
    ],
    [
        ".hn",
        "nnB",
        "bBk",
    ],
    [
        "mh.",
        "nBb",
        "bbk",
    ],
    [
        ".h.",
        "nBb",
        "Bbk",
    ],
]
CELL = 3    # pixels per side of a game square


def turn_left(sprite):
    """Rotate a square sprite 90 degrees counter-clockwise."""
    n = len(sprite)
    return ["".join(sprite[c][n - 1 - r] for c in range(n)) for r in range(n)]

def mirror(sprite):
    return [row[::-1] for row in sprite]

def head_for(direction, sprite):
    """Return the head sprite turned to face the direction of travel."""
    if direction == (1, 0):          # right
        return sprite
    if direction == (-1, 0):         # left
        return mirror(sprite)
    if direction == (0, -1):         # up
        return turn_left(sprite)
    return mirror(turn_left(mirror(sprite)))   # down


# --- Keyboard input (Windows) ---------------------------------------------
ARROWS = {"H": "w", "P": "s", "K": "a", "M": "d"}   # arrow keys -> WASD

try:
    import msvcrt

    def get_keys():
        """Return every key pressed since the last frame."""
        keys = []
        while msvcrt.kbhit():
            ch = msvcrt.getwch()
            if ch in ("\x00", "\xe0"):        # arrow keys send two codes
                ch = ARROWS.get(msvcrt.getwch(), "")
            keys.append(ch.lower())
        return keys
except ImportError:
    def get_keys():
        return []

DIRECTIONS = {"w": (0, -1), "s": (0, 1), "a": (-1, 0), "d": (1, 0)}

def write(text):
    sys.stdout.write(text)

def move_cursor(row, col):
    write(f"\033[{row};{col}H")

RESET = "\033[0m"


# --- Background -------------------------------------------------------------
def make_ground(pixel_width, pixel_height):
    """Draw the ground once: grooves, specks and little grass tufts."""
    ground = [[GROUND] * pixel_width for _ in range(pixel_height)]

    def put(x, y, color):
        if 0 <= x < pixel_width and 0 <= y < pixel_height:
            ground[y][x] = color

    # Wavy grooves running across the field, like furrows in soil
    y = random.randint(3, 6)
    while y < pixel_height - 1:
        row = y
        x = 0
        while x < pixel_width:
            if random.random() < 0.08:                 # small gap
                x += random.randint(2, 6)
                continue
            put(x, row, GROOVE)
            put(x, row + 1, GROOVE_LIP)
            if random.random() < 0.12:                 # wander up or down
                row += random.choice((-1, 1))
                row = max(y - 2, min(y + 2, row))
            x += 1
        y += random.randint(6, 9)

    # Random specks of dirt
    for _ in range(pixel_width * pixel_height // 25):
        put(random.randrange(pixel_width), random.randrange(pixel_height),
            random.choice((SPECK_DARK, SPECK_LIGHT)))

    # Little grass tufts
    for _ in range(pixel_width * pixel_height // 300):
        x, y = random.randrange(pixel_width), random.randrange(pixel_height)
        put(x, y, GRASS)
        put(x + 1, y - 1, GRASS)
        put(x + 2, y, GRASS)

    return ground


# --- Game logic -------------------------------------------------------------
def random_empty_cell(width, height, taken):
    while True:
        cell = (random.randint(0, width - 1), random.randint(0, height - 1))
        if cell not in taken:
            return cell


def place_stones(game, width, height):
    """Throw away the old stones and scatter a new random set."""
    hx, hy = game["snake"][0]
    dx, dy = game["direction"]

    taken = set(game["snake"]) | set(game["food"])
    # safe zone around the head and the path in front of it
    for ox in range(-2, 3):
        for oy in range(-2, 3):
            taken.add(((hx + ox) % width, (hy + oy) % height))
    for i in range(1, 6):
        taken.add(((hx + dx * i) % width, (hy + dy * i) % height))

    game["stones"] = {}
    for _ in range(random.randint(MIN_STONES, MAX_STONES)):
        cell = random_empty_cell(width, height, taken)
        game["stones"][cell] = random.randrange(len(STONES))   # pick a shape
        taken.add(cell)


def add_fruit(game, width, height):
    taken = set(game["snake"]) | set(game["stones"]) | set(game["food"])
    cell = random_empty_cell(width, height, taken)
    game["food"][cell] = random.randrange(len(FRUITS))          # pick a fruit


def new_game(width, height):
    cx, cy = width // 2, height // 2
    game = {
        "snake": [(cx - i, cy) for i in range(START_LENGTH)],   # [0] = head
        "direction": (1, 0),
        "stones": {},
        "food": {},
        "ground": make_ground(width * CELL, (height * CELL + 1) // 2 * 2),
        "lives": LIVES,
        "score": 0,
        "delay": START_DELAY,
        "paused": False,
        "over": False,
        "menu_choice": 0,  # 0 = Play again, 1 = Quit
        "flash": 0,        # frames left to show the snake red after a hit
        "chomp": 0,        # frames left to keep the mouth open after eating
        "ticks": 0,        # counts steps, used for the tongue flick
    }
    for _ in range(NUM_FOOD):
        add_fruit(game, width, height)
    place_stones(game, width, height)
    return game


def next_cell(game, width, height):
    (x, y), (dx, dy) = game["snake"][0], game["direction"]
    return ((x + dx) % width, (y + dy) % height)


def step(game, width, height):
    """Move the snake one square and handle eating and crashing."""
    game["ticks"] += 1
    new_head = next_cell(game, width, height)

    if new_head in game["stones"]:
        del game["stones"][new_head]              # the stone is smashed
        game["lives"] -= 1
        game["flash"] = 5
        if game["lives"] <= 0:
            game["over"] = True
            return

    game["snake"].insert(0, new_head)

    if new_head in game["food"]:
        # Ate a fruit: keep the tail, so the snake grows by one ball.
        del game["food"][new_head]
        game["score"] += 1
        game["chomp"] = 3
        game["delay"] = max(MIN_DELAY, game["delay"] * SPEEDUP)   # faster!
        add_fruit(game, width, height)
        place_stones(game, width, height)                         # new stones!
    else:
        game["snake"].pop()


def speed_level(game):
    """A friendly speed number for the status bar: 1.0 at the start."""
    return round(START_DELAY / game["delay"], 1)


# --- Drawing ----------------------------------------------------------------
def build_pixels(game, width, height):
    """Make a 2D list of colors for the whole field."""
    pixels = [row[:] for row in game["ground"]]      # start from the ground
    pixel_width, pixel_height = len(pixels[0]), len(pixels)
    hurt = game["flash"] > 0

    # 1. Decide what to draw: (square, sprite, turns red when hurt?)
    things = []
    for cell, shape in game["stones"].items():
        things.append((cell, STONES[shape], False))
    for cell, kind in game["food"].items():
        things.append((cell, FRUITS[kind], False))

    snake = game["snake"]
    for i in range(len(snake) - 1, 0, -1):            # tail first, head last
        if i == len(snake) - 1:
            sprite = TAIL
        else:
            sprite = BODY_SPOT if i % 2 == 0 else BODY
        things.append((snake[i], sprite, True))

    # Mouth opens when fruit is right in front, and stays open while chewing.
    if game["chomp"] > 0 or next_cell(game, width, height) in game["food"]:
        head = HEAD_OPEN
    elif game["ticks"] % 14 in (0, 1):                # flick the tongue now and then
        head = HEAD_TONGUE
    else:
        head = HEAD_CLOSED
    things.append((snake[0], head_for(game["direction"], head), True))

    def sprite_pixels(cell, sprite):
        """Yield (x, y, letter) for every visible pixel of a sprite."""
        # Sprites bigger than a square are centered on it.
        offset = (len(sprite) - CELL) // 2
        x0, y0 = cell[0] * CELL - offset, cell[1] * CELL - offset
        for r, row in enumerate(sprite):
            for c, ch in enumerate(row):
                if ch != ".":
                    yield x0 + c, y0 + r, ch

    # 2. Shadows: every object darkens the ground 1 pixel down and right.
    shaded = set()
    for cell, sprite, _ in things:
        for x, y, ch in sprite_pixels(cell, sprite):
            if ch != "t":
                shaded.add((x + 1, y + 1))
    for x, y in shaded:
        if 0 <= x < pixel_width and 0 <= y < pixel_height:
            r, g, b = pixels[y][x]
            pixels[y][x] = (int(r * SHADOW), int(g * SHADOW), int(b * SHADOW))

    # 3. The objects themselves, on top of the shadows.
    for cell, sprite, can_hurt in things:
        for x, y, ch in sprite_pixels(cell, sprite):
            if 0 <= x < pixel_width and 0 <= y < pixel_height:
                if can_hurt and hurt and ch in HURT:
                    pixels[y][x] = HURT[ch]
                else:
                    pixels[y][x] = PALETTE[ch]
    return pixels


def pixels_to_lines(pixels):
    """Turn pairs of pixel rows into terminal lines using '▀'."""
    lines = []
    for top_row, bottom_row in zip(pixels[0::2], pixels[1::2]):
        out, fg, bg = [], None, None
        for top, bottom in zip(top_row, bottom_row):
            if bottom != bg:
                out.append("\033[48;2;%d;%d;%dm" % bottom)
                bg = bottom
            if top == bottom:
                out.append(" ")                  # one color: just a space
            else:
                if top != fg:
                    out.append("\033[38;2;%d;%d;%dm" % top)
                    fg = top
                out.append("▀")
        out.append(RESET)
        lines.append("".join(out))
    return lines


def status_line(game, screen_width):
    lives = "O " * game["lives"] + "x " * (LIVES - game["lives"])
    text = (f" Score: {game['score']}   Length: {len(game['snake'])}"
            f"   Lives: {lives}  Speed: x{speed_level(game)}"
            f"   Stones: {len(game['stones'])}")
    if game["paused"]:
        text += "   [PAUSED]"
    text += "     WASD/arrows move  SPACE pause  Q quit"
    return "\033[7m" + text[:screen_width].ljust(screen_width) + RESET


def draw_menu(game, screen_width, screen_height):
    """Draw the Game Over menu box in the middle of the screen."""
    box_width = 32
    options = ["Play again", "Quit"]

    lines = [("", "box"),
             ("GAME OVER".center(box_width), "title"),
             (f"The snake hit {LIVES} stones.".center(box_width), "box"),
             (f"Score: {game['score']}    Length: {len(game['snake'])}".center(box_width), "box"),
             ("", "box")]
    for i, name in enumerate(options):
        if i == game["menu_choice"]:
            lines.append((f">  {name}  <".center(box_width), "selected"))
        else:
            lines.append((name.center(box_width), "box"))
    lines += [("", "box"),
              ("W/S or arrows: choose".center(box_width), "hint"),
              ("ENTER: confirm".center(box_width), "hint"),
              ("", "box")]

    styles = {
        "box":      "\033[97;48;2;120;30;35m",        # white on dark red
        "title":    "\033[1;93;48;2;120;30;35m",      # bold yellow on dark red
        "selected": "\033[1;30;107m",                 # bold black on white
        "hint":     "\033[37;48;2;120;30;35m",        # grey on dark red
    }
    top = max(2, screen_height // 2 - len(lines) // 2)
    left = max(1, (screen_width - box_width) // 2)
    for i, (text, style) in enumerate(lines):
        move_cursor(top + i, left)
        write(styles[style] + text.ljust(box_width) + RESET)


# --- Main loop --------------------------------------------------------------
def main():
    os.system("")                                  # enable escape codes on Windows
    cols, rows = shutil.get_terminal_size()
    width = (cols - 1) // CELL                     # squares across
    height = (rows - 1) * 2 // CELL                # squares down (1 line for status;
                                                   #  each line is 2 pixels tall)
    if width < 15 or height < 8:
        print("The window is too small. Make it bigger and run the game again.")
        return

    screen_width = width * CELL
    game = new_game(width, height)
    previous = []                                  # lines drawn last frame
    write("\033[?25l\033[2J")                      # hide cursor, clear screen

    try:
        while True:
            # ---------- 1. Input ----------
            for key in get_keys():
                if game["over"]:
                    # Game Over menu
                    if key in ("w", "s"):
                        game["menu_choice"] = 1 - game["menu_choice"]
                    elif key in ("\r", "\n", " ", "r"):
                        if game["menu_choice"] == 1 and key != "r":
                            return                 # Quit
                        game = new_game(width, height)   # Play again
                        previous = []
                        write("\033[2J")
                        break
                    elif key == "q":
                        return
                    continue

                if key == "q":
                    return
                if key == " ":
                    game["paused"] = not game["paused"]
                elif key in DIRECTIONS:
                    new_dir, old_dir = DIRECTIONS[key], game["direction"]
                    if (new_dir[0] + old_dir[0], new_dir[1] + old_dir[1]) != (0, 0):
                        game["direction"] = new_dir
                    break                          # one turn per step

            # ---------- 2. Update ----------
            if not game["over"] and not game["paused"]:
                step(game, width, height)
                game["flash"] = max(0, game["flash"] - 1)
                game["chomp"] = max(0, game["chomp"] - 1)

            # ---------- 3. Draw (only lines that changed) ----------
            move_cursor(1, 1)
            write(status_line(game, screen_width))
            lines = pixels_to_lines(build_pixels(game, width, height))
            for i, line in enumerate(lines):
                if i >= len(previous) or previous[i] != line:
                    move_cursor(i + 2, 1)
                    write(line)
            previous = lines
            if game["over"]:
                draw_menu(game, screen_width, len(lines) + 1)
            sys.stdout.flush()

            time.sleep(game["delay"] if not game["over"] else 0.05)
    except KeyboardInterrupt:
        pass
    finally:
        write(RESET + "\033[2J\033[1;1H\033[?25h")
        sys.stdout.flush()


if __name__ == "__main__":
    main()

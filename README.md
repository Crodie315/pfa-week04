# pfa-week04

# 1.Description

For this week's assignment, I designed a Snake game. I used Claude to generate the game rules and graphics, PowerShell to run the game logic, and Windows Terminal as the display interface for the player. The rules are straightforward: the player uses the WASD keys to control the snake's movement (up, down, left, and right). The map contains two types of objects—fruit and stones. Each time the snake eats a piece of fruit, it grows longer, and the positions of the stones are automatically randomized; additionally, the snake's movement speed increases as it consumes more fruit. If the snake hits a stone, its body turns red as a warning; three such collisions end the game, at which point a pop-up window appears offering two options: continue playing or exit.

# 2. How I made batter

I designed the movement mechanics so that the snake does not maintain a constant speed; instead, it moves progressively faster as the player consumes more fruit, creating a challenge. Secondly, I implemented a stone respawn mechanism: a stone reappears on the map each time a piece of fruit is eaten, forcing the player to constantly reassess fruit locations and identify safe areas to navigate. Thirdly, I added an animation of the snake opening its mouth to enhance the game's visual appeal. Finally, I incorporated an interactive element where the snake's body turns red upon colliding with a stone.

# 3. How it works

“next_cell”： works out which square the snake's head will move into next. It takes the head's position, adds the direction it's moving, and uses % to wrap it around to the other side when it               goes off the edge.

“step‘： moves the game forward one step. It moves the snake one square and checks what it landed on:
         a stone: lose a life, the stone breaks, and with 0 lives it's game over
         fruit: the snake grows, gets faster, and the stones are reshuffled
         nothing: the tail is removed so the snake keeps the same length
         
”head_for“； turns the head picture to face the way the snake is moving. The head is only drawn once, facing right. This function mirrors it to face left, or rotates it to face up or down.

# 4. One Undo

The agent shrank the sprites to 2×2 pixels to make them smaller, but the snake lost its eyes and mouth. I checked the change with git diff and discarded it with git restore

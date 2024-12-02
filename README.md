# -Tic-Tac-Toe-
Python Based Tic Tac Toe 
import pygame
import sys
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 1000,1000
LINE_WIDTH = 18
BOARD_ROWS, BOARD_COLS = 3, 3
SQUARE_SIZE = WIDTH // BOARD_COLS
CIRCLE_RADIUS = SQUARE_SIZE // 3
CIRCLE_WIDTH = 15
CROSS_WIDTH = 25
SPACE = SQUARE_SIZE // 4

# Colors
BG_COLOR = (28, 170, 156)
LINE_COLOR = (23, 145, 135)
CIRCLE_COLOR = (239, 231, 200)
CROSS_COLOR = (66, 66, 66)

# Initialize screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption('Tic Tac Toe')
screen.fill(BG_COLOR)

# Initialize board
board = [[0 for _ in range(BOARD_COLS)] for _ in range(BOARD_ROWS)]

# Font for messages
font = pygame.font.Font(None, 80)

# Draw grid lines
def draw_lines():
    for i in range(1, BOARD_ROWS):
        pygame.draw.line(screen, LINE_COLOR, (0, i * SQUARE_SIZE), (WIDTH, i * SQUARE_SIZE), LINE_WIDTH)
    for i in range(1, BOARD_COLS):
        pygame.draw.line(screen, LINE_COLOR, (i * SQUARE_SIZE, 0), (i * SQUARE_SIZE, HEIGHT), LINE_WIDTH)

# Draw X or O
def draw_figures():
    for row in range(BOARD_ROWS):
        for col in range(BOARD_COLS):
            if board[row][col] == 1:
                pygame.draw.circle(screen, CIRCLE_COLOR, 
                                   (col * SQUARE_SIZE + SQUARE_SIZE // 2, row * SQUARE_SIZE + SQUARE_SIZE // 2), 
                                   CIRCLE_RADIUS, CIRCLE_WIDTH)
            elif board[row][col] == 2:
                pygame.draw.line(screen, CROSS_COLOR, 
                                 (col * SQUARE_SIZE + SPACE, row * SQUARE_SIZE + SPACE), 
                                 (col * SQUARE_SIZE + SQUARE_SIZE - SPACE, row * SQUARE_SIZE + SQUARE_SIZE - SPACE), 
                                 CROSS_WIDTH)
                pygame.draw.line(screen, CROSS_COLOR, 
                                 (col * SQUARE_SIZE + SPACE, row * SQUARE_SIZE + SQUARE_SIZE - SPACE), 
                                 (col * SQUARE_SIZE + SQUARE_SIZE - SPACE, row * SQUARE_SIZE + SPACE), 
                                 CROSS_WIDTH)

# Display message
def display_message(text):
    text_surface = font.render(text, True, (255, 255, 255))
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2))
    screen.blit(text_surface, text_rect)

# Check winner
def check_winner(player):
    for col in range(BOARD_COLS):
        if board[0][col] == board[1][col] == board[2][col] == player:
            return True
    for row in range(BOARD_ROWS):
        if board[row][0] == board[row][1] == board[row][2] == player:
            return True
    if board[0][0] == board[1][1] == board[2][2] == player:
        return True
    if board[0][2] == board[1][1] == board[2][0] == player:
        return True
    return False

# Check if the board is full
def is_full():
    for row in board:
        if 0 in row:
            return False
    return True

# Computer move
def computer_move():
    for row in range(BOARD_ROWS):
        for col in range(BOARD_COLS):
            if board[row][col] == 0:
                board[row][col] = 2
                if check_winner(2):
                    return
                board[row][col] = 1
                if check_winner(1):
                    board[row][col] = 2
                    return
                board[row][col] = 0
    
    empty_cells = [(row, col) for row in range(BOARD_ROWS) for col in range(BOARD_COLS) if board[row][col] == 0]
    if empty_cells:
        row, col = random.choice(empty_cells)
        board[row][col] = 2

# Restart game
def restart():
    screen.fill(BG_COLOR)
    draw_lines()
    for row in range(BOARD_ROWS):
        for col in range(BOARD_COLS):
            board[row][col] = 0

# Game variables
player = 1
game_over = False
mode = None  # None until user selects a mode

# Draw mode selection screen
def mode_selection_screen():
    screen.fill(BG_COLOR)
    text_surface = font.render("Press 1 for Human vs Computer", True, (255, 255, 255))
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2 - 50))
    screen.blit(text_surface, text_rect)
    
    text_surface = font.render("Press 2 for 2 Player Mode", True, (255, 255, 255))
    text_rect = text_surface.get_rect(center=(WIDTH // 2, HEIGHT // 2 + 50))
    screen.blit(text_surface, text_rect)
    pygame.display.update()

# Main loop
mode_selection_screen()

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if mode is None:  # Mode selection
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    mode = "HUMAN_VS_COMPUTER"
                    restart()
                elif event.key == pygame.K_2:
                    mode = "TWO_PLAYER"
                    restart()

        elif mode:  # Main game loop after mode selection
            if event.type == pygame.MOUSEBUTTONDOWN and not game_over:
                mouseX = event.pos[0]
                mouseY = event.pos[1]
                clicked_row = mouseY // SQUARE_SIZE
                clicked_col = mouseX // SQUARE_SIZE

                if board[clicked_row][clicked_col] == 0:
                    board[clicked_row][clicked_col] = player
                    if check_winner(player):
                        game_over = True
                        winner_text = (
                            "You Win!" if mode == "HUMAN_VS_COMPUTER" and player == 1 else
                            "Computer Wins!" if mode == "HUMAN_VS_COMPUTER" and player == 2 else
                            f"Player {player} Wins!"
                        )
                    elif is_full():
                        game_over = True
                        winner_text = "It's a Draw!"
                    else:
                        player = 3 - player  # Switch player (1 <-> 2)

            if mode == "HUMAN_VS_COMPUTER" and player == 2 and not game_over:
                computer_move()
                if check_winner(2):
                    game_over = True
                    winner_text = "Computer Wins!"
                elif is_full():
                    game_over = True
                    winner_text = "It's a Draw!"
                else:
                    player = 1

            if event.type == pygame.KEYDOWN and event.key == pygame.K_r:
                restart()
                player = 1
                game_over = False

    draw_figures()

    # Display winner message if game over
    if game_over:
        display_message(winner_text)

    pygame.display.update()

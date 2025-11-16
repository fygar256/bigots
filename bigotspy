#!/usr/bin/env python3
import curses
import random
import locale

locale.setlocale(locale.LC_ALL, '')

# 定数
X_SIZE = 40
Y_SIZE = 23
CRASH = "Ｘ"
SPACE = "　"


class Entity:
    """共通の表示機能を持つベースクラス"""
    def __init__(self, x: int, y: int, char: str, screen):
        self.x = x
        self.y = y
        self.char = char
        self.screen = screen

    def draw(self):
        self.screen.addstr(int(self.y), int(self.x * 2), self.char)

    def clear(self):
        self.screen.addstr(int(self.y * 2 / 2), int(self.x * 2), SPACE)


class Rock(Entity):
    def __init__(self, x: int, y: int, screen):
        super().__init__(x, y, "＃", screen)
        self.alive = True


class Enemy(Entity):
    def __init__(self, x: int, y: int, screen):
        super().__init__(x, y, "Ｏ", screen)
        self.alive = True

    def move_toward(self, target_x: int, target_y: int, rocks: list, enemies: list):
        if not self.alive:
            return

        dx = (target_x - self.x) and ((target_x - self.x) // abs(target_x - self.x))
        dy = (target_y - self.y) and ((target_y - self.y) // abs(target_y - self.y))

        new_x = self.x + dx
        new_y = self.y + dy

        # 岩との衝突判定
        for rock in rocks:
            if rock.alive and rock.x == new_x and rock.y == new_y:
                rock.alive = False
                self.alive = False
                self.clear()
                rock.clear()
                return

        # 他の敵との衝突判定
        for enemy in enemies:
            if enemy != self and enemy.alive and enemy.x == new_x and enemy.y == new_y:
                self.alive = False
                self.clear()
                return

        self.clear()
        self.x = new_x
        self.y = new_y
        self.draw()


class Player(Entity):
    def __init__(self, x: int, y: int, screen):
        super().__init__(x, y, "＠", screen)

    def move(self, key: str):
        move_map = {
            '7': (-1, -1), '8': (0, -1), '9': (1, -1),
            'u': (-1, 0), '4': (-1, 0), 'i': (0, 0), '5': (0, 0), 'o': (1, 0), '6': (1, 0),
            'j': (-1, 1), '1': (-1, 1), 'k': (0, 1), '2': (0, 1), 'l': (1, 1), '3': (1, 1)
        }
        dx, dy = move_map.get(key, (0, 0))
        nx, ny = self.x + dx, self.y + dy
        if 0 <= nx < X_SIZE and 0 <= ny < Y_SIZE:
            self.clear()
            self.x, self.y = nx, ny
            self.draw()


class Game:
    def __init__(self, screen):
        self.screen = screen
        self.gamestat = 0  # 0: プレイ中, 1: 負け, 3: 勝ち
        self.player = Player(X_SIZE//2, Y_SIZE//2, screen)
        self.rocks = []
        self.enemies = []

    def instruction(self):
        self.screen.clear()
        lines = [
            "Bigots Ver 1.0",
            "Mission: kill all bigots to survive!",
            "Ｏ -- bigots, chase player step by step.",
            "＃ -- Rock, destroy bigots and player when touched.",
            "＠ -- Player, control to avoid crashes and survive.",
            "",
            "Key control:          Tenkey:",
            " ７  ８  ９           ７ ８ ９",
            "  ↖ ↑  ↗              ↖ ↑ ↗",
            " ｕ← ｉ →ｏ           ４← ５→ ６",
            "  ↙ ↓  ↘              ↙ ↓ ↘",
            " ｊ  ｋ  ｌ           １ ２ ３",
            "",
            "'i' and '5' move bigots only (player stays still)",
            "Good Luck!",
            "Hit any key to start..."
        ]
        for idx, line in enumerate(lines):
            self.screen.addstr(idx, 0, line)
        self.screen.getch()

    def setup(self):
        self.screen.clear()
        self.gamestat = 0
        self.rocks = []
        self.enemies = []

        # 岩を配置
        for _ in range(120):
            rx, ry = random.randint(0, X_SIZE-1), random.randint(0, Y_SIZE-1)
            rock = Rock(rx, ry, self.screen)
            rock.draw()
            self.rocks.append(rock)

        # 敵を配置（岩と重ならないように）
        for _ in range(12):
            while True:
                ex, ey = random.randint(0, X_SIZE-1), random.randint(0, Y_SIZE-1)
                if all(r.x != ex or r.y != ey for r in self.rocks):
                    enemy = Enemy(ex, ey, self.screen)
                    enemy.draw()
                    self.enemies.append(enemy)
                    break

        self.player.x = X_SIZE // 2
        self.player.y = Y_SIZE // 2
        self.player.draw()

    def check_collisions(self):
        for enemy in self.enemies:
            if enemy.alive and enemy.x == self.player.x and enemy.y == self.player.y:
                self.gamestat = 1
        for rock in self.rocks:
            if rock.alive and rock.x == self.player.x and rock.y == self.player.y:
                self.gamestat = 1

    def play_turn(self):
        key = self.screen.getkey()
        self.player.move(key)
        self.check_collisions()
        if self.gamestat != 0:
            return

        for enemy in self.enemies:
            enemy.move_toward(self.player.x, self.player.y, self.rocks, self.enemies)
        self.check_collisions()
        if all(not e.alive for e in self.enemies):
            self.gamestat = 3

    def try_again(self):
        self.screen.addstr(Y_SIZE, 0, "Try Again? [y/n] ")
        while True:
            ch = self.screen.getkey()
            if ch.lower() == 'y':
                return True
            elif ch.lower() == 'n':
                return False

    def play(self):
        while True:
            self.instruction()
            self.setup()

            while self.gamestat == 0:
                self.screen.refresh()
                self.play_turn()

            self.screen.addstr(0, 0, "You Win! " if self.gamestat == 3 else "You Lose ")
            self.screen.refresh()

            if not self.try_again():
                break


def main(stdscr):
    curses.noecho()
    curses.curs_set(0)
    game = Game(stdscr)
    game.play()


if __name__ == "__main__":
    curses.wrapper(main)


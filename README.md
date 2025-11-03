# AI-Project
Treasure hunt
import tkinter as tk
import random
import heapq
import time

# ---------------------- A* Algorithm ----------------------
def astar(grid, start, end):
    rows, cols = len(grid), len(grid[0])
    open_set = []
    heapq.heappush(open_set, (0, start))
    came_from = {}
    g = {start: 0}

    def h(p1, p2):
        return abs(p1[0]-p2[0]) + abs(p1[1]-p2[1])

    while open_set:
        _, current = heapq.heappop(open_set)
        if current == end:
            path = []
            while current in came_from:
                path.append(current)
                current = came_from[current]
            path.append(start)
            path.reverse()
            return path

        for dx, dy in [(0,1),(1,0),(-1,0),(0,-1)]:
            nx, ny = current[0]+dx, current[1]+dy
            if 0 <= nx < rows and 0 <= ny < cols and grid[nx][ny] != 1:
                temp_g = g[current] + 1
                if (nx, ny) not in g or temp_g < g[(nx, ny)]:
                    g[(nx, ny)] = temp_g
                    f = temp_g + h((nx, ny), end)
                    heapq.heappush(open_set, (f, (nx, ny)))
                    came_from[(nx, ny)] = current
    return None


# ---------------------- Game Level ----------------------
class TreasureHunt:
    def __init__(self, master, size=5, obstacles=5):
        self.master = master
        self.size = size
        self.cell = 80
        self.obstacles = obstacles

        # Main frame
        self.frame = tk.Frame(master, bg="#f0f8ff")
        self.frame.pack(fill="both", expand=True)

        self.canvas = tk.Canvas(self.frame, width=self.size*self.cell, height=self.size*self.cell, bg="white")
        self.canvas.pack(pady=10)

        # Buttons
        btn_frame = tk.Frame(self.frame, bg="#f0f8ff")
        btn_frame.pack(pady=10)
        tk.Button(btn_frame, text="Start", bg="#32cd32", fg="white", font=("Arial", 12, "bold"),
                  command=self.start_search, width=10).grid(row=0, column=0, padx=10)
        tk.Button(btn_frame, text="Reset", bg="#ffa500", fg="black", font=("Arial", 12, "bold"),
                  command=self.reset_grid, width=10).grid(row=0, column=1, padx=10)
        tk.Button(btn_frame, text="Exit", bg="#ff4500", fg="white", font=("Arial", 12, "bold"),
                  command=self.back_to_menu, width=10).grid(row=0, column=2, padx=10)

        self.create_grid()
        self.draw_grid()

    def create_grid(self):
        self.grid = [[0 for _ in range(self.size)] for _ in range(self.size)]
        self.start = (0, 0)
        self.end = (self.size - 1, self.size - 1)
        self.grid[self.start[0]][self.start[1]] = 2
        self.grid[self.end[0]][self.end[1]] = 3

        self.obstacle_positions = set()
        while len(self.obstacle_positions) < self.obstacles:
            x, y = random.randint(0, self.size - 1), random.randint(0, self.size - 1)
            if (x, y) not in [self.start, self.end]:
                self.obstacle_positions.add((x, y))
        for (x, y) in self.obstacle_positions:
            self.grid[x][y] = 1

    def draw_grid(self):
        self.canvas.delete("all")
        for i in range(self.size):
            for j in range(self.size):
                x1, y1 = j*self.cell, i*self.cell
                x2, y2 = x1+self.cell, y1+self.cell

                if self.grid[i][j] == 1:
                    color = "red"
                elif self.grid[i][j] == 2:
                    color = "yellow"
                elif self.grid[i][j] == 3:
                    color = "green"
                else:
                    color = "white"

                self.canvas.create_rectangle(x1, y1, x2, y2, fill=color, outline="black")

        # Labels
        self.canvas.create_text(self.start[1]*self.cell + 40, self.start[0]*self.cell + 40,
                                text="START", font=("Arial", 10, "bold"))
        self.canvas.create_text(self.end[1]*self.cell + 40, self.end[0]*self.cell + 40,
                                text="TREASURE", font=("Arial", 10, "bold"))

    def start_search(self):
        path = astar(self.grid, self.start, self.end)
        if not path:
            print("No Path Found!")
            return
        self.animate_path(path)

    def animate_path(self, path):
        for i in range(len(path)-1):
            x1 = path[i][1]*self.cell + self.cell//2
            y1 = path[i][0]*self.cell + self.cell//2
            x2 = path[i+1][1]*self.cell + self.cell//2
            y2 = path[i+1][0]*self.cell + self.cell//2
            self.canvas.create_line(x1, y1, x2, y2, fill="blue", width=4)
            self.master.update()
            time.sleep(0.25)

    def reset_grid(self):
        self.create_grid()
        self.draw_grid()

    def back_to_menu(self):
        self.frame.destroy()
        MainMenu(self.master)


# ---------------------- Main Menu ----------------------
class MainMenu:
    def __init__(self, master):
        self.master = master
        for widget in master.winfo_children():
            widget.destroy()

        self.bg_canvas = tk.Canvas(master, width=700, height=600)
        self.bg_canvas.pack(fill="both", expand=True)

        # --- Gradient Background (Safe Version) ---
        for i in range(0, 600, 5):
            r = int(255 * (i / 600))
            g = int(150 + 100 * ((600 - i) / 600))
            b = int(255 * abs((300 - i) / 600))
            color = f"#{r:02x}{g:02x}{b:02x}"
            self.bg_canvas.create_line(0, i, 700, i, fill=color)

        self.bg_canvas.create_text(350, 120, text="TREASURE HUNT", font=("Comic Sans MS", 36, "bold"), fill="white")

        self.create_button("Easy", lambda: self.start_level(4, 5), 250)
        self.create_button("Medium", lambda: self.start_level(5, 7), 320)
        self.create_button("Hard", lambda: self.start_level(6, 9), 390)
        self.create_button("Exit", master.quit, 460)

    def create_button(self, text, command, y):
        btn = tk.Button(self.bg_canvas, text=text, font=("Arial", 16, "bold"),
                        bg="#ffcc00", fg="black", width=15, height=1,
                        activebackground="#ffa500", command=command)
        self.bg_canvas.create_window(350, y, window=btn)

    def start_level(self, size, obstacles):
        for widget in self.master.winfo_children():
            widget.destroy()
        TreasureHunt(self.master, size=size, obstacles=obstacles)


# ---------------------- Run Game ----------------------
if __name__ == "__main__":
    root = tk.Tk()
    root.title("Treasure Hunt - AI Pathfinding Game")
    root.geometry("700x600")
    MainMenu(root)
    root.mainloop()




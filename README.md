import tkinter as tk
from tkinter import ttk, messagebox
import random

# ---------------------------------------------------------
# Clase principal
# ---------------------------------------------------------
class JuegoMemoria(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Juego de Memoria - 2 Jugadores")
        self.geometry("800x600")
        self.resizable(False, False)

        # Variables
        self.jugador1 = tk.StringVar()
        self.jugador2 = tk.StringVar()
        self.nivel = tk.StringVar(value="Fácil")

        self.turno = 0
        self.puntajes = [0, 0]

        self.simbolos = []
        self.botones = []
        self.cartas_abiertas = []

        self.frame_inicio = FrameInicio(self)
        self.frame_juego = FrameJuego(self)
        self.frame_resultado = FrameResultado(self)

        self.frame_inicio.pack(fill="both", expand=True)

    def iniciar_juego(self):
        if not self.jugador1.get() or not self.jugador2.get():
            messagebox.showwarning("Error", "Debes ingresar los nombres de ambos jugadores")
            return

        self.turno = 0
        self.puntajes = [0, 0]

        self.frame_inicio.pack_forget()
        self.frame_juego.pack(fill="both", expand=True)
        self.frame_juego.crear_tablero()

    def mostrar_resultado(self):
        self.frame_juego.pack_forget()
        self.frame_resultado.pack(fill="both", expand=True)
        self.frame_resultado.mostrar_ganador()


# ---------------------------------------------------------
# Frame Inicio
# ---------------------------------------------------------
class FrameInicio(ttk.Frame):
    def __init__(self, master):
        super().__init__(master)

        ttk.Label(self, text="Juego de Memoria", font=("Segoe UI", 20, "bold")).pack(pady=10)

        ttk.Label(self, text="Jugador 1").pack()
        ttk.Entry(self, textvariable=master.jugador1).pack()

        ttk.Label(self, text="Jugador 2").pack(pady=5)
        ttk.Entry(self, textvariable=master.jugador2).pack()

        ttk.Label(self, text="Nivel").pack(pady=5)
        ttk.Combobox(
            self,
            textvariable=master.nivel,
            values=["Fácil", "Difícil"],
            state="readonly",
            width=10
        ).pack()

        ttk.Button(self, text="Comenzar", command=master.iniciar_juego).pack(pady=15)


# ---------------------------------------------------------
# Frame Juego
# ---------------------------------------------------------
class FrameJuego(ttk.Frame):
    def __init__(self, master):
        super().__init__(master)

        self.lbl_turno = ttk.Label(self, font=("Segoe UI", 12, "bold"))
        self.lbl_turno.pack(pady=5)

        self.lbl_puntaje = ttk.Label(self)
        self.lbl_puntaje.pack()

        self.tablero = ttk.Frame(self)
        self.tablero.pack(pady=20)

    def crear_tablero(self):
        for w in self.tablero.winfo_children():
            w.destroy()

        size = 4 if self.master.nivel.get() == "Fácil" else 6
        total_pares = (size * size) // 2

        letras = [chr(65 + i) for i in range(total_pares)] * 2
        random.shuffle(letras)

        self.master.simbolos = letras
        self.master.cartas_abiertas = []
        self.master.botones = []

        idx = 0
        for r in range(size):
            for c in range(size):
                b = tk.Button(
                    self.tablero,
                    text="?",
                    width=5,
                    height=2,
                    bg="#4C84B5",
                    fg="white",
                    font=("Segoe UI", 11, "bold"),
                    command=lambda i=idx: self.voltear(i)
                )
                b.grid(row=r, column=c, padx=5, pady=5)
                self.master.botones.append(b)
                idx += 1

        self.actualizar_info()

    def voltear(self, i):
        if i in self.master.cartas_abiertas:
            return

        b = self.master.botones[i]
        b.config(text=self.master.simbolos[i], bg="#F6C90E")
        self.master.cartas_abiertas.append(i)

        if len(self.master.cartas_abiertas) == 2:
            self.after(600, self.verificar)

    def verificar(self):
        i1, i2 = self.master.cartas_abiertas
        simbolos = self.master.simbolos

        if simbolos[i1] == simbolos[i2]:
            self.master.botones[i1].config(bg="#6BCB77", state="disabled")
            self.master.botones[i2].config(bg="#6BCB77", state="disabled")
            self.master.puntajes[self.master.turno] += 1
        else:
            self.master.botones[i1].config(text="?", bg="#4C84B5")
            self.master.botones[i2].config(text="?", bg="#4C84B5")
            self.master.turno = 1 - self.master.turno  # cambia turno

        self.master.cartas_abiertas.clear()
        self.actualizar_info()

        total_pares = len(self.master.simbolos) // 2
        if sum(self.master.puntajes) == total_pares:
            self.master.mostrar_resultado()

    def actualizar_info(self):
        jugador_actual = self.master.jugador1.get() if self.master.turno == 0 else self.master.jugador2.get()
        self.lbl_turno.config(text=f"Turno: {jugador_actual}")

        self.lbl_puntaje.config(
            text=f"{self.master.jugador1.get()}: {self.master.puntajes[0]}  |  "
                 f"{self.master.jugador2.get()}: {self.master.puntajes[1]}"
        )


# ---------------------------------------------------------
# Frame Resultado
# ---------------------------------------------------------
class FrameResultado(ttk.Frame):
    def __init__(self, master):
        super().__init__(master)

        self.lbl = ttk.Label(self, font=("Segoe UI", 14, "bold"))
        self.lbl.pack(pady=20)

        ttk.Button(self, text="Jugar otra vez", command=self.volver).pack()

    def mostrar_ganador(self):
        p1, p2 = self.master.puntajes
        j1, j2 = self.master.jugador1.get(), self.master.jugador2.get()

        if p1 > p2:
            txt = f"🏆 Ganó {j1}"
        elif p2 > p1:
            txt = f"🏆 Ganó {j2}"
        else:
            txt = "🤝 Empate"

        self.lbl.config(text=f"{txt}\n\n{j1}: {p1} pares\n{j2}: {p2} pares")

    def volver(self):
        self.pack_forget()
        self.master.frame_inicio.pack(fill="both", expand=True)


# ---------------------------------------------------------
# Ejecutar
# ---------------------------------------------------------
if __name__ == "__main__":
    app = JuegoMemoria()
    app.mainloop()

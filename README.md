import tkinter as tk
from tkinter import filedialog, messagebox
import subprocess
import os

# Rutas de los ejecutables
XDELTA_PATH = "data/xdelta3.exe"
DUCKSTATION_PATH = "data/DuckStation/duckstation-qt-x64-ReleaseLTCG.exe"
GCLIENT_PATH = "data/GClient.exe"

# Nombre del archivo de salida
OUTPUT_FILE = "data/archivo_modificado.bin"

# Función para seleccionar el archivo original (.bin)
def seleccionar_original():
    archivo = filedialog.askopenfilename(title="Selecciona el archivo original", filetypes=[("Archivos BIN", "*.bin")])
    entry_original.delete(0, tk.END)
    entry_original.insert(0, archivo)

# Función para seleccionar el parche (.xdelta)
def seleccionar_parche():
    archivo = filedialog.askopenfilename(title="Selecciona el parche .xdelta", filetypes=[("Archivos Xdelta", "*.xdelta")])
    entry_parche.delete(0, tk.END)
    entry_parche.insert(0, archivo)

# Función para aplicar el parche
def aplicar_parche():
    original = entry_original.get()
    parche = entry_parche.get()

    if not os.path.exists(XDELTA_PATH):
        messagebox.showerror("Error", "No se encontró xdelta3.exe. Colócalo en la carpeta 'data/'.")
        return

    if not original or not parche:
        messagebox.showerror("Error", "Debes seleccionar el archivo original y el parche.")
        return

    try:
        comando = [XDELTA_PATH, "-d", "-s", original, parche, OUTPUT_FILE]
        resultado = subprocess.run(comando, capture_output=True, text=True)

        if resultado.returncode != 0:
            messagebox.showerror("Error", f"Error al aplicar el parche:\n{resultado.stderr}")
            return
        
        messagebox.showinfo("Éxito", f"Parche aplicado correctamente. Se generó {OUTPUT_FILE}")
        boton_play.config(state=tk.NORMAL)  # Habilitar botón Play
    except Exception as e:
        messagebox.showerror("Error", f"Hubo un problema al aplicar el parche:\n{e}")

# Función para iniciar DuckStation y GClient
def jugar():
    if not os.path.exists(DUCKSTATION_PATH):
        messagebox.showerror("Error", "No se encontró DuckStation en la carpeta 'data/DuckStation/'.")
        return
    
    if not os.path.exists(OUTPUT_FILE):
        messagebox.showerror("Error", "El archivo parcheado no existe.")
        return
    
    try:
        # Ejecutar GClient.exe en segundo plano
        if os.path.exists(GCLIENT_PATH):
            subprocess.Popen(GCLIENT_PATH, creationflags=subprocess.CREATE_NEW_CONSOLE)

        # Ejecutar DuckStation con el juego parcheado
        subprocess.Popen([DUCKSTATION_PATH, OUTPUT_FILE])
    except Exception as e:
        messagebox.showerror("Error", f"No se pudo iniciar DuckStation:\n{e}")

# Interfaz gráfica con Tkinter
root = tk.Tk()
root.title("Launcher - Aplicador de Parches Xdelta")
root.geometry("500x300")

# Campos de selección de archivos
tk.Label(root, text="Archivo original (.bin):").pack()
entry_original = tk.Entry(root, width=50)
entry_original.pack()
tk.Button(root, text="Seleccionar", command=seleccionar_original).pack()

tk.Label(root, text="Parche (.xdelta):").pack()
entry_parche = tk.Entry(root, width=50)
entry_parche.pack()
tk.Button(root, text="Seleccionar", command=seleccionar_parche).pack()

# Botón para aplicar el parche
tk.Button(root, text="Aplicar Parche", command=aplicar_parche, bg="green", fg="white").pack(pady=10)

# Botón para jugar (deshabilitado hasta que se aplique el parche)
boton_play = tk.Button(root, text="Play", command=jugar, bg="blue", fg="white", state=tk.DISABLED)
boton_play.pack(pady=10)

root.mainloop()

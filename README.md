import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3


conn = sqlite3.connect("students.db")
cursor = conn.cursor()
cursor.execute("""
    CREATE TABLE IF NOT EXISTS students(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER NOT NULL,
        course TEXT NOT NULL)
""")
conn.commit()


def display():
    for x in tree.get_children():
        tree.delete(x)
    cursor.execute("SELECT * FROM students")
    for i, row in enumerate(cursor.fetchall()):
        tree.insert("", "end", values=row, tags=("even" if i % 2 == 0 else "odd",))

def clear():
 
    name.delete(0, tk.END)
    age.delete(0, tk.END)
    course.delete(0, tk.END)
    tree.selection_remove(tree.selection())

def add():
    n, a, c = name.get().strip(), age.get().strip(), course.get().strip()
    if not n or not a or not c:
        messagebox.showwarning("Warning", "Please fill in all fields.")
        return
    try:
        a = int(a)
    except ValueError:
        messagebox.showerror("Error", "Age must be a number.")
        return
    cursor.execute(
        "INSERT INTO students(name,age,course) VALUES(?,?,?)", (n, a, c))
    conn.commit()
    clear()
    display()

def update():
    selected = tree.selection()
    if not selected:
        messagebox.showwarning("Warning", "Select a student first.")
        return
    sid = tree.item(selected[0])["values"][0]
    try:
        a = int(age.get())
    except ValueError:
        messagebox.showerror("Error", "Age must be a number.")
        return
    cursor.execute(
        "UPDATE students SET name=?, age=?, course=? WHERE id=?", 
        (name.get(), a, course.get(), sid))
    conn.commit()
    clear()
    display()

def delete():
    selected = tree.selection()
    if not selected:
        messagebox.showwarning("Warning", "Select a student first.")
        return
    sid = tree.item(selected[0])["values"][0]
    if messagebox.askyesno("Confirm", "Delete this student?"):
        cursor.execute("DELETE FROM students WHERE id=?", (sid,))
        conn.commit()
        clear()
        display()

def select(event):
    selected = tree.selection()
    if selected:
        row = tree.item(selected[0])["values"]
        
     
        name.delete(0, tk.END)
        age.delete(0, tk.END)
        course.delete(0, tk.END)
        
        name.insert(0, row[1])
        age.insert(0, row[2])
        course.insert(0, row[3])

# GUI
root = tk.Tk()
root.title("Student Management System")
root.geometry("700x700")
root.configure(bg="#ff1493")

tk.Label(
    root, text="Student Management System", font=("Arial", 20, "bold"), bg="#000000", fg="#ff1493"
).pack(pady=15)

frame = tk.Frame(root, bg="purple", bd=2, relief="groove")
frame.pack(padx=20, fill="x")

name = tk.Entry(frame, width=35)
age = tk.Entry(frame, width=35)
course = tk.Entry(frame, width=35)

for i, (text, entry) in enumerate([("Name", name), ("Age", age), ("Course", course)]):
    tk.Label(
        frame, text=text, bg="White", fg="#ff1493", font=("Arial", 11, "bold")
    ).grid(row=i, column=0, padx=10, pady=8)
    entry.grid(row=i, column=1, padx=10, pady=8)

# BUTTONS
buttons = tk.Frame(root, bg="#ff1493")
buttons.pack(pady=15)

for text, color, command in [
    ("Add", "#ff1493", add), 
    ("Update", "#000000", update), 
    ("Delete", "#ff1493", delete), 
    ("Clear", "#000000", clear)
]:
    tk.Button(
        buttons, text=text, command=command, width=20, bg=color, fg="white", font=("Arial", 10, "bold")
    ).pack(side="left", padx=5)

# TABLE
tree = ttk.Treeview(root, columns=("ID", "Name", "Age", "Course"), show="headings")
for col in ("ID", "Name", "Age", "Course"):
    tree.heading(col, text=col)

tree.column("ID", width=60, anchor="center")
tree.column("Name", width=200)
tree.column("Age", width=80, anchor="center")
tree.column("Course", width=220)

tree.tag_configure("even", background="#000000")
tree.tag_configure("odd", background="#ff1493")
tree.pack(fill="both", expand=True, padx=20, pady=10)

tree.bind("<<TreeviewSelect>>", select)

display()
root.mainloop()
conn.close()



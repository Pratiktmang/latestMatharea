# latestMatharea
from tkinter import *
from tkinter import ttk
import random
import math
from owlready2 import *

# ------------------- LOAD OWL ONTOLOGY -------------------
try:
    onto = get_ontology("math_surface_area.owl").load()
except:
    onto = None

# ------------------- FORMULAS -------------------
FORMULAS = {
    "Sphere": "Surface Area = 4 × π × r²",
    "Cube": "Surface Area = 6 × s²",
    "Cylinder": "Surface Area = 2πr(r + h)",
    "TriangularPrism": "Surface Area = 2 × triangle area + perimeter × length"
}

SHAPES = ["Sphere", "Cube", "Cylinder", "TriangularPrism"]

# ------------------- MAIN WINDOW -------------------
root = Tk()
root.title("3D Shape Surface Area ITS")
root.geometry("950x650")
root.configure(bg="#f4f6f8")

# ------------------- GLOBAL VARIABLES -------------------
score = {"correct": 0, "total": 0}
current_shape = None
current_answer = None
hint_stage = 0
values = {}
formula_text = ""
hint_text = ""

# ------------------- COLOURS -------------------
BG = "#f4f6f8"
SIDEBAR = "#1e293b"
CARD = "#ffffff"
PRIMARY = "#2563eb"
TEXT = "#111827"
MUTED = "#64748b"
SUCCESS = "#16a34a"
ERROR = "#dc2626"

# ------------------- MAIN LAYOUT -------------------
sidebar = Frame(root, bg=SIDEBAR, width=230)
sidebar.pack(side=LEFT, fill=Y)

content = Frame(root, bg=BG)
content.pack(side=RIGHT, expand=True, fill=BOTH)

# ------------------- SIDEBAR -------------------
Label(
    sidebar,
    text="Math ITS",
    font=("Arial", 24, "bold"),
    bg=SIDEBAR,
    fg="white"
).pack(pady=(35, 5))

Label(
    sidebar,
    text="Surface Area Tutor",
    font=("Arial", 12),
    bg=SIDEBAR,
    fg="#cbd5e1"
).pack(pady=(0, 30))

# ------------------- CONTENT WIDGETS -------------------
title_label = Label(
    content,
    text="Welcome to the Surface Area Tutor",
    font=("Arial", 26, "bold"),
    bg=BG,
    fg=TEXT
)
title_label.pack(pady=(35, 10))

subtitle_label = Label(
    content,
    text="Choose a shape from the menu to begin practising.",
    font=("Arial", 14),
    bg=BG,
    fg=MUTED
)
subtitle_label.pack(pady=(0, 20))

card = Frame(content, bg=CARD, padx=35, pady=30)
card.pack(pady=20, padx=40, fill=BOTH, expand=True)

score_label = Label(
    card,
    text="Score: 0/0",
    font=("Arial", 15, "bold"),
    bg=CARD,
    fg=PRIMARY
)
score_label.pack(anchor="e")

question_label = Label(
    card,
    text="Select a shape to start.",
    font=("Consolas", 17),
    bg=CARD,
    fg=TEXT,
    justify=LEFT
)
question_label.pack(pady=25)

answer_frame = Frame(card, bg=CARD)
answer_frame.pack(pady=10)

Label(
    answer_frame,
    text="Your answer:",
    font=("Arial", 14, "bold"),
    bg=CARD,
    fg=TEXT
).grid(row=0, column=0, padx=8)

answer_entry = Entry(
    answer_frame,
    font=("Arial", 18),
    width=9,
    justify=CENTER,
    bd=2,
    relief=SOLID
)
answer_entry.grid(row=0, column=1, padx=8)

feedback_label = Label(
    card,
    text="",
    font=("Arial", 15, "bold"),
    bg=CARD,
    fg=TEXT
)
feedback_label.pack(pady=10)

hint_label = Label(
    card,
    text="",
    font=("Arial", 13),
    bg="#eff6ff",
    fg=TEXT,
    wraplength=550,
    justify=LEFT,
    padx=15,
    pady=15
)
hint_label.pack(pady=15, fill=X)

button_frame = Frame(card, bg=CARD)
button_frame.pack(pady=15)

# ------------------- FUNCTIONS -------------------
def update_score():
    score_label.config(text=f"Score: {score['correct']}/{score['total']}")

def generate_question(shape_name):
    global formula_text, hint_text

    shape = None
    if onto:
        shape = onto.search_one(iri=f"*{shape_name}")

    dims = {}

    if shape_name == "Sphere":
        r = random.randint(2, 12)
        dims = {"Radius": r}
        answer = 4 * math.pi * r * r

    elif shape_name == "Cube":
        s = random.randint(2, 12)
        dims = {"Side": s}
        answer = 6 * s * s

    elif shape_name == "Cylinder":
        r = random.randint(2, 12)
        h = random.randint(2, 12)
        dims = {"Radius": r, "Height": h}
        answer = 2 * math.pi * r * (r + h)

    elif shape_name == "TriangularPrism":
        base = random.randint(2, 12)
        height = random.randint(2, 12)
        length = random.randint(2, 12)

        # Assumption: right triangular prism
        side3 = round(math.sqrt(base ** 2 + height ** 2), 2)
        perimeter = base + height + side3

        dims = {
            "Base": base,
            "Height": height,
            "Length": length,
            "Third side": side3
        }

        triangle_area = 0.5 * base * height
        answer = (2 * triangle_area) + (perimeter * length)

    formula_text = FORMULAS.get(shape_name, "No formula available")

    if shape and hasattr(shape, "hasHint") and shape.hasHint:
        hint_text = str(shape.hasHint[0])
    else:
        hint_text = "Use the formula and substitute the given values carefully."

    return dims, answer

def ask_question(shape):
    global current_shape, current_answer, hint_stage, values

    current_shape = shape
    hint_stage = 0

    values, current_answer = generate_question(shape)

    answer_entry.delete(0, END)
    feedback_label.config(text="")
    hint_label.config(text="Hints will appear here when you click Hint.", bg="#eff6ff")

    title_label.config(text=f"{shape} Tutor")
    subtitle_label.config(text="Calculate the surface area using the values below.")

    dims_text = "\n".join([f"{key}: {value}" for key, value in values.items()])

    question_label.config(
        text=f"Shape: {shape}\n\nGiven values:\n{dims_text}\n\nEnter the surface area:"
    )

def give_hint():
    global hint_stage

    if current_shape is None:
        hint_label.config(text="Please choose a shape first.")
        return

    if hint_stage == 0:
        hint_label.config(
            text=f"Hint 1: Formula\n{formula_text}",
            bg="#dbeafe"
        )
    elif hint_stage == 1:
        hint_label.config(
            text=f"Hint 2: Substitute the values\n{values}",
            bg="#fef9c3"
        )
    elif hint_stage == 2:
        hint_label.config(
            text=f"Hint 3: Final answer\n{round(current_answer, 2)}",
            bg="#dcfce7"
        )
    else:
        hint_label.config(
            text="No more hints available. Try the next question.",
            bg="#fee2e2"
        )

    hint_stage += 1

def check_answer():
    global score

    if current_shape is None:
        feedback_label.config(text="Please choose a shape first.", fg=ERROR)
        return

    try:
        user_answer = float(answer_entry.get())
        score["total"] += 1

        if abs(user_answer - current_answer) < 0.05:
            score["correct"] += 1
            feedback_label.config(
                text="Correct! Excellent work.",
                fg=SUCCESS
            )
        else:
            feedback_label.config(
                text=f"Incorrect. Correct answer: {round(current_answer, 2)}",
                fg=ERROR
            )

        update_score()

    except ValueError:
        feedback_label.config(
            text="Please enter a valid number.",
            fg=ERROR
        )

def reset_score():
    score["correct"] = 0
    score["total"] = 0
    update_score()
    feedback_label.config(text="Score has been reset.", fg=PRIMARY)

# ------------------- BUTTONS -------------------
Button(
    button_frame,
    text="Check Answer",
    font=("Arial", 13, "bold"),
    bg=PRIMARY,
    fg="white",
    padx=18,
    pady=8,
    bd=0,
    command=check_answer
).grid(row=0, column=0, padx=8)

Button(
    button_frame,
    text="Hint",
    font=("Arial", 13, "bold"),
    bg="#f59e0b",
    fg="white",
    padx=18,
    pady=8,
    bd=0,
    command=give_hint
).grid(row=0, column=1, padx=8)

Button(
    button_frame,
    text="Next Question",
    font=("Arial", 13, "bold"),
    bg="#10b981",
    fg="white",
    padx=18,
    pady=8,
    bd=0,
    command=lambda: ask_question(current_shape) if current_shape else None
).grid(row=0, column=2, padx=8)

# ------------------- SIDEBAR SHAPE BUTTONS -------------------
for shape in SHAPES:
    Button(
        sidebar,
        text=shape,
        font=("Arial", 13, "bold"),
        bg="#334155",
        fg="white",
        activebackground=PRIMARY,
        activeforeground="white",
        width=18,
        relief=FLAT,
        pady=8,
        command=lambda s=shape: ask_question(s)
    ).pack(pady=7)

Button(
    sidebar,
    text="Reset Score",
    font=("Arial", 12, "bold"),
    bg="#ef4444",
    fg="white",
    width=18,
    relief=FLAT,
    pady=8,
    command=reset_score
).pack(pady=(35, 8))

Label(
    sidebar,
    text="Designed for\n3D geometry practice",
    font=("Arial", 10),
    bg=SIDEBAR,
    fg="#94a3b8"
).pack(side=BOTTOM, pady=25)

# ------------------- START APP -------------------
root.mainloop()

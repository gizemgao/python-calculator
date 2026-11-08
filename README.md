# python-calculator
A simple desktop calculator built with Python 3 and Tkinter.

import tkinter as tk
import ast
import operator

# Main window
root = tk.Tk()
root.title("My Calculator")
root.geometry("360x520")
root.resizable(False, False)
root.configure(bg="#1e1e2e")

# Display
display = tk.Entry(
    root,
    font=("Arial", 28),
    bg="#313244",
    fg="white",
    justify="right",
    bd=0,
    insertbackground="white"
)
display.pack(fill="x", padx=20, pady=25, ipady=15)


# Button functions
def add_to_display(value):
    display.insert(tk.END, value)


def clear_display():
    display.delete(0, tk.END)


def delete_last():
    current = display.get()
    display.delete(0, tk.END)
    display.insert(0, current[:-1])


def calculate():
    try:
        expression = display.get()

        def evaluate(node):
            if isinstance(node, ast.Expression):
                return evaluate(node.body)

            elif isinstance(node, ast.Constant) and type(node.value) in (int, float):
                return node.value

            elif isinstance(node, ast.BinOp):
                operations = {
                    ast.Add: operator.add,
                    ast.Sub: operator.sub,
                    ast.Mult: operator.mul,
                    ast.Div: operator.truediv
                }

                operation = operations.get(type(node.op))

                if operation is None:
                    raise ValueError()

                return operation(
                    evaluate(node.left),
                    evaluate(node.right)
                )

            elif isinstance(node, ast.UnaryOp):
                if isinstance(node.op, ast.USub):
                    return -evaluate(node.operand)
                elif isinstance(node.op, ast.UAdd):
                    return evaluate(node.operand)

            raise ValueError()

        result = evaluate(ast.parse(expression, mode="eval"))

        display.delete(0, tk.END)
        display.insert(0, str(result))

    except ZeroDivisionError:
        display.delete(0, tk.END)
        display.insert(0, "Cannot divide by zero")

    except Exception:
        display.delete(0, tk.END)
        display.insert(0, "Error")


# Rounded button function
def create_round_button(parent, text, command, color):

    canvas = tk.Canvas(
        parent,
        width=70,
        height=70,
        bg="#1e1e2e",
        highlightthickness=0
    )

    canvas.create_oval(
        5, 5, 65, 65,
        fill=color,
        outline=""
    )

    canvas.create_text(
        35, 35,
        text=text,
        fill="#1e1e2e",
        font=("Arial", 18, "bold")
    )

    canvas.bind("<Button-1>", lambda event: command())

    # Hover effect
    canvas.bind(
        "<Enter>",
        lambda event: canvas.config(cursor="hand2")
    )

    canvas.bind(
        "<Leave>",
        lambda event: canvas.config(cursor="")
    )

    return canvas


# Button layout
buttons = [
    ["7", "8", "9", "/"],
    ["4", "5", "6", "*"],
    ["1", "2", "3", "-"],
    ["C", "0", ".", "+"],
    ["⌫", "="]
]

# Button frame
button_frame = tk.Frame(root, bg="#1e1e2e")
button_frame.pack(expand=True)

for row_index, row in enumerate(buttons):

    for col_index, text in enumerate(row):

        if text == "C":
            command = clear_display
            color = "#89b4fa"

        elif text == "⌫":
            command = delete_last
            color = "#89b4fa"

        elif text == "=":
            command = calculate
            color = "#89b4fa"

        elif text in ["+", "-", "*", "/"]:
            command = lambda x=text: add_to_display(x)
            color = "#89b4fa"

        else:
            command = lambda x=text: add_to_display(x)
            color = "#45475a"

        button = create_round_button(
            button_frame,
            text,
            command,
            color
        )

        button.grid(
            row=row_index,
            column=col_index,
            padx=5,
            pady=5
        )


# Start application
root.mainloop()

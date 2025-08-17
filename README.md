# Password-Generator-Project-with-CLI-GUI-tests-.
PasswordPy is a Python-based secure password generator that helps users create strong and customizable passwords. The project includes both a Command-Line Interface (CLI) and a Graphical User Interface (GUI) built with Tkinter, making it easy for technical and non-technical users alike.
The generator uses Python’s secrets module to ensure cryptographically secure randomness and supports flexible options such as password length, inclusion of digits/symbols, and strong policy enforcement (at least one uppercase, lowercase, digit, and symbol).
# src/generator.py
import string
import secrets

def generate_password(length=12, use_digits=True, use_symbols=True, policy=False):
    """Generate a secure random password."""
    characters = string.ascii_letters
    if use_digits:
        characters += string.digits
    if use_symbols:
        characters += string.punctuation

    if not characters:
        raise ValueError("No character set selected!")

    # Base random password
    password = ''.join(secrets.choice(characters) for _ in range(length))

    # If policy is True, ensure at least one from each type
    if policy:
        reqs = []
        if use_digits:
            reqs.append(secrets.choice(string.digits))
        if use_symbols:
            reqs.append(secrets.choice(string.punctuation))
        reqs.append(secrets.choice(string.ascii_lowercase))
        reqs.append(secrets.choice(string.ascii_uppercase))
        # Replace some chars in the generated password
        password = list(password)
        for i, ch in enumerate(reqs):
            password[i] = ch
        password = ''.join(password)

    return password

# src/cli.py
import argparse
import pyperclip
from generator import generate_password

def main():
    parser = argparse.ArgumentParser(description="Password Generator")
    parser.add_argument("--length", type=int, default=12, help="Length of password")
    parser.add_argument("--no-digits", action="store_true", help="Exclude digits")
    parser.add_argument("--no-symbols", action="store_true", help="Exclude symbols")
    parser.add_argument("--policy", action="store_true", help="Enforce policy: must include upper, lower, digit, symbol")
    parser.add_argument("--copy", action="store_true", help="Copy password to clipboard")

    args = parser.parse_args()

    pwd = generate_password(
        length=args.length,
        use_digits=not args.no_digits,
        use_symbols=not args.no_symbols,
        policy=args.policy
    )

    print(f"Generated password: {pwd}")

    if args.copy:
        try:
            pyperclip.copy(pwd)
            print("(Copied to clipboard!)")
        except Exception:
            print("Clipboard copy failed (install pyperclip).")

if __name__ == "__main__":
    main()

# src/gui.py
import tkinter as tk
from generator import generate_password

def generate():
    length = int(length_entry.get())
    use_digits = digits_var.get()
    use_symbols = symbols_var.get()
    policy = policy_var.get()
    pwd = generate_password(length, use_digits, use_symbols, policy)
    result_var.set(pwd)

root = tk.Tk()
root.title("Password Generator")

tk.Label(root, text="Password Length:").pack()
length_entry = tk.Entry(root)
length_entry.insert(0, "12")
length_entry.pack()

digits_var = tk.BooleanVar(value=True)
symbols_var = tk.BooleanVar(value=True)
policy_var = tk.BooleanVar(value=False)

tk.Checkbutton(root, text="Include Digits", variable=digits_var).pack()
tk.Checkbutton(root, text="Include Symbols", variable=symbols_var).pack()
tk.Checkbutton(root, text="Enforce Strong Policy", variable=policy_var).pack()

tk.Button(root, text="Generate", command=generate).pack(pady=5)
result_var = tk.StringVar()
tk.Entry(root, textvariable=result_var, width=40).pack()

root.mainloop()

# tests/test_generator.py
from src.generator import generate_password

def test_default_length():
    pwd = generate_password()
    assert len(pwd) == 12

def test_custom_length():
    pwd = generate_password(length=20)
    assert len(pwd) == 20

def test_policy_enforced():
    pwd = generate_password(length=12, policy=True)
    assert any(c.islower() for c in pwd)
    assert any(c.isupper() for c in pwd)
    assert any(c.isdigit() for c in pwd)
    assert any(not c.isalnum() for c in pwd)

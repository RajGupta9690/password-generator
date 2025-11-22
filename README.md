🔐 Password Generator

A modern, responsive Password Generator built using HTML, CSS, and JavaScript.
It includes multiple customization options, strength meter, copy functionality, download, and keyboard shortcuts.
A React version is also available.

🚀 Features
✔️ Password Customization

Adjustable length (4 – 64 characters)

Uppercase letters

Lowercase letters

Numbers

Symbols

✔️ Advanced Functionalities

Password strength meter (entropy-based)

Regenerate button (🔄)

Copy to clipboard

Double-click output to copy

Download password as .txt

Keyboard shortcut: R to regenerate

✔️ UI/UX

Fully responsive

Clean dark theme

Smooth layout

Accessible (keyboard + ARIA labels)

📁 Project Structure
password-generator/
│
├── index.html     # Main HTML file with CSS + JS included
├── README.md      # Documentation
└── (Optional) React Version

🛠️ Technologies Used

HTML5

CSS3 (custom dark UI)

JavaScript (ES6+)

(Optional) React + Vite

🧪 How It Works

User selects password length.

Chooses character sets (upper/lower/numbers/symbols).

Click Generate → Password is created.

Strength meter calculates entropy bits to judge password strength.

User can:

Copy

Regenerate

Download

Double-click to copy

Press R anytime to regenerate

🔑 Entropy Calculation (Strength Meter)
Entropy = length × log₂(poolSize)


Strength levels:

0–28 bits: Very Weak

28–36 bits: Weak

36–60 bits: Medium

60–90 bits: Strong

90+ bits: Very Strong

from flask import Flask, request, render_template
from datetime import datetime
import sqlite3

app = Flask(__name__)

DB = "clicks.db"

def init_db():
    conn = sqlite3.connect(DB)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS clicks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            time TEXT NOT NULL,
            browser TEXT
        )
    """)
    conn.commit()
    conn.close()

@app.route("/")
def home():
    conn = sqlite3.connect(DB)

    conn.execute(
        "INSERT INTO clicks (time, browser) VALUES (?, ?)",
        (
            datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            request.headers.get("User-Agent", "Unknown")
        )
    )

    conn.commit()
    conn.close()

    return render_template("index.html")


@app.route("/dashboard")
def dashboard():
    conn = sqlite3.connect(DB)

    clicks = conn.execute(
        "SELECT id, time, browser FROM clicks ORDER BY id DESC"
    ).fetchall()

    conn.close()

    return render_template("dashboard.html", clicks=clicks)


init_db()

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

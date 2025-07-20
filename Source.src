import tkinter as tk
from tkinter import filedialog, messagebox, simpledialog
import time
import json
from pynput import mouse, keyboard
from pynput.mouse import Controller, Button
from threading import Thread

clicks = []
recording = False
playing = False
mouse_controller = Controller()
speed_multiplier = 1.0
loop_count = 1
recording_duration = 3
left_clicks = 0
right_clicks = 0
record_mode = "both"

# Updated keybinds with separate play actions
keybinds = {
    "record_left": "r",
    "record_right": "t",
    "play_left": "p",
    "play_right": "y",
    "stop": "s"
}

def update_cps_display():
    try:
        l_cps = left_clicks / recording_duration
        r_cps = right_clicks / recording_duration
        cps_label.config(text=f"Left CPS: {l_cps:.2f} | Right CPS: {r_cps:.2f}")
    except ZeroDivisionError:
        cps_label.config(text="Left CPS: 0 | Right CPS: 0")

def record_clicks(duration=3, mode="both"):
    global clicks, recording, left_clicks, right_clicks, record_mode
    clicks.clear()
    left_clicks = 0
    right_clicks = 0
    recording = True
    record_mode = mode
    update_status(f"Recording {mode} clicks...")
    start_time = time.time()

    def on_click(x, y, button, pressed):
        global left_clicks, right_clicks
        if pressed and recording:
            if record_mode == "left" and button != Button.left:
                return
            if record_mode == "right" and button != Button.right:
                return
            timestamp = time.time() - start_time
            clicks.append((timestamp, button.name))
            if button == Button.left:
                left_clicks += 1
            elif button == Button.right:
                right_clicks += 1

    listener = mouse.Listener(on_click=on_click)
    listener.start()
    time.sleep(duration)
    recording = False
    listener.stop()
    update_cps_display()
    update_status(f"Recorded {len(clicks)} clicks ({mode}).")

def replay_clicks(filter_button=None):
    global playing
    if not clicks:
        update_status("No clicks to play.")
        return
    playing = True
    update_status("Playing...")
    for _ in range(loop_count):
        if not playing:
            break
        start = time.time()
        for timestamp, button in clicks:
            if not playing:
                break
            if filter_button and button != filter_button:
                continue
            delay = (timestamp / speed_multiplier) - (time.time() - start)
            if delay > 0:
                time.sleep(delay)
            mouse_controller.click(getattr(Button, button))
    playing = False
    update_status("Playback finished.")

def export_clicks():
    if not clicks:
        messagebox.showinfo("Export", "No clicks to export.")
        return
    file_path = filedialog.asksaveasfilename(defaultextension=".json")
    if file_path:
        with open(file_path, 'w') as f:
            json.dump(clicks, f)
        messagebox.showinfo("Export", "Clicks exported successfully.")

def import_clicks():
    global clicks
    file_path = filedialog.askopenfilename(filetypes=[("JSON files", "*.json")])
    if file_path:
        with open(file_path, 'r') as f:
            clicks = json.load(f)
        update_status(f"Imported {len(clicks)} clicks.")

def start_recording_left():
    Thread(target=record_clicks, args=(recording_duration, "left"), daemon=True).start()

def start_recording_right():
    Thread(target=record_clicks, args=(recording_duration, "right"), daemon=True).start()

def start_replaying_left():
    Thread(target=replay_clicks, args=("left",), daemon=True).start()

def start_replaying_right():
    Thread(target=replay_clicks, args=("right",), daemon=True).start()

def stop_replaying():
    global playing
    playing = False
    update_status("Playback stopped.")

def set_speed():
    global speed_multiplier
    val = simpledialog.askfloat("Speed", "Enter speed multiplier (e.g., 1.0 for normal):", minvalue=0.1)
    if val:
        speed_multiplier = val
        update_status(f"Speed set to {val}x")

def set_loop():
    global loop_count
    val = simpledialog.askinteger("Loops", "Enter number of loops:", minvalue=1)
    if val:
        loop_count = val
        update_status(f"Loop count set to {val}")

def edit_keybinds():
    for action in keybinds:
        new_key = simpledialog.askstring("Change Keybind", f"Set new key for '{action}' (current: {keybinds[action]}):")
        if new_key:
            keybinds[action] = new_key.lower()
    update_hotkey_label()

def update_status(text):
    status_label.config(text=text)

def update_hotkey_label():
    hotkey_label.config(text="Hotkeys: " + " | ".join([f"{k.upper()} = {v.upper()}" for k, v in keybinds.items()]))

def start_hotkey_listener():
    def on_press(key):
        try:
            if key.char == keybinds["record_left"]:
                start_recording_left()
            elif key.char == keybinds["record_right"]:
                start_recording_right()
            elif key.char == keybinds["play_left"]:
                start_replaying_left()
            elif key.char == keybinds["play_right"]:
                start_replaying_right()
            elif key.char == keybinds["stop"]:
                stop_replaying()
        except AttributeError:
            pass

    listener = keyboard.Listener(on_press=on_press)
    listener.daemon = True
    listener.start()

# GUI setup
root = tk.Tk()
root.title("Click Macro Tool")
root.geometry("340x550")
root.configure(bg="#2c3e50")

title_label = tk.Label(root, text="🖱️ Click Macro Tool", font=("Arial", 16, "bold"), bg="#2c3e50", fg="white")
title_label.pack(pady=10)

btn_style = {"font": ("Arial", 12), "bg": "#3498db", "fg": "white", "activebackground": "#2980b9", "width": 25}

tk.Button(root, text="🔴 Record Left Click", command=start_recording_left, **btn_style).pack(pady=5)
tk.Button(root, text="🔴 Record Right Click", command=start_recording_right, **btn_style).pack(pady=5)
tk.Button(root, text="▶️ Play Left Clicks", command=start_replaying_left, **btn_style).pack(pady=5)
tk.Button(root, text="▶️ Play Right Clicks", command=start_replaying_right, **btn_style).pack(pady=5)
tk.Button(root, text="⏹️ Stop", command=stop_replaying, **btn_style).pack(pady=5)
tk.Button(root, text="⚙️ Set Speed Multiplier", command=set_speed, **btn_style).pack(pady=5)
tk.Button(root, text="🔁 Set Loop Count", command=set_loop, **btn_style).pack(pady=5)
tk.Button(root, text="💾 Export Clicks", command=export_clicks, **btn_style).pack(pady=5)
tk.Button(root, text="📂 Import Clicks", command=import_clicks, **btn_style).pack(pady=5)
tk.Button(root, text="🎹 Edit Keybinds", command=edit_keybinds, **btn_style).pack(pady=5)

hotkey_label = tk.Label(root, font=("Arial", 10), bg="#2c3e50", fg="lightgray")
hotkey_label.pack(pady=5)
update_hotkey_label()

cps_label = tk.Label(root, text="Left CPS: 0 | Right CPS: 0", font=("Arial", 11), bg="#2c3e50", fg="lightgreen")
cps_label.pack(pady=5)

status_label = tk.Label(root, text="Status: Idle", font=("Arial", 10, "italic"), bg="#2c3e50", fg="lightgreen")
status_label.pack(pady=5)

start_hotkey_listener()
root.mainloop()


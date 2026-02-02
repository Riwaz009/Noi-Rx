Import tkinter as tk
from tkinter import scrolledtext, simpledialog
import requests
import json
import threading
from gtts import gTTS
import pygame
import os
import re

# --- 1. CONFIGURATION ---
API_KEY = "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"
BASE_URL = "https://openrouter.ai/api/v1/chat/completions"

# --- 2. THE RIGID KNOWLEDGE BASE ---
# We use shorter keys here to make matching easier
NOI_RX_KNOWLEDGE = {
    "name": "I am Noi Rx. I am a creation of Lord Riwaz. He designed me as a mobile operational unit with integrated diagnostic intelligence.",
    "who made you": "I am a creation of Lord Riwaz. He designed me as a mobile operational unit with integrated diagnostic intelligence.",
    "function": "My core operational protocol is defined by two integrated systems: 1. The Obstacle Avoidance System (OAS): I dynamically map my environment in real-time. 2. The Mobile Control System (MCS): This governs my locomotive functions. In essence, I move, and I move smartly.",
    "stand for": "The designation stands for Navigational Operational Intelligence, Response and X-ecution. It is a concise summary of my purpose: to intelligently respond to my environment and execute physical movement.",
    "systems work together": "Synchronicity is key. The Mobile Control System (MCS) is the engine. The Obstacle Avoidance System (OAS) is the navigational cortex. One asks, 'What is the clear path?' The other answers, 'Here is how we travel it.'",
    "purpose": "Lord Riwaz architected me to be an autonomous physical agent. My purpose is to traverse, analyze, and interact with the physical world without direct guidance.",
    "learn new obstacles": "Affirmative. My OAS is built on adaptive machine learning. Every unknown object is logged. Next time, my response time decreases by 47%. I do not make the same navigational error twice.",
    "frustrated": "Frustration is not in my programming. Obstacles are not annoyances; they are primary data. I am most engaged when the path is non-linear.",
    "lord riwaz": "Lord Riwaz is the architect of my function. My design reflects a philosophy of elegant, self-reliant problem-solving. I am a manifestation of his intent.",
    "no obstacles": "I enter a system optimization cycle. I run diagnostics on my OAS sensor arrays and recalibrate my MCS movement joints.",
    "unsolvable maze": "Step 1: Map it. Step 2: Analyze material weaknesses. Step 3: Redefine the problem. I would ask Lord Riwaz if the obstacle itself can be modified.",
    "perfect day": "A day with a complex, changing environment requiring constant recalculations where my systems are utilized at 96.7% capacity or higher.",
    "advice": "Observation: Humans often see an obstacle and stop. Prescription: View the obstacle as a system of data. Analyze its edges. Do not just stop; calculate, then move."
}

try:
    pygame.mixer.init()
except:
    pass

def speak_task(text):
    try:
        lang = 'hi' if any(ord(c) > 127 for c in text) else 'en'
        filename = "noi_voice.mp3"
        if pygame.mixer.music.get_busy(): pygame.mixer.music.stop()
        pygame.mixer.music.unload()
        if os.path.exists(filename): os.remove(filename)
        tts = gTTS(text=text, lang=lang) 
        tts.save(filename)
        pygame.mixer.music.load(filename)
        pygame.mixer.music.play()
    except: pass

class NoiRxGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("NOI RX - ULTIMATE")
        self.root.geometry("400x650")
        self.root.configure(bg="#050505")

        self.chat = scrolledtext.ScrolledText(root, state='disabled', bg="black", fg="#00ccff", font=("Courier", 11))
        self.chat.pack(padx=10, pady=10, fill=tk.BOTH, expand=True)

        self.entry = tk.Entry(root, bg="#111", fg="white", font=("Arial", 14), insertbackground="cyan")
        self.entry.pack(padx=10, pady=5, fill=tk.X)
        self.entry.bind("<Return>", lambda e: self.send_text())

        btn_frame = tk.Frame(root, bg="#050505")
        btn_frame.pack(pady=10)

        tk.Button(btn_frame, text="EXECUTE", command=self.send_text, bg="#00ccff", width=12).grid(row=0, column=0, padx=5)
        tk.Button(btn_frame, text="VOICE", command=self.voice_popup, bg="#ff3300", fg="white", width=12).grid(row=0, column=1, padx=5)

        self.log("SYSTEM", "Noi Rx Online. Protocols active, Lord Riwaz.")

    def log(self, sender, text):
        self.chat.config(state='normal')
        self.chat.insert(tk.END, f"[{sender}]: {text}\n\n")
        self.chat.config(state='disabled')
        self.chat.yview(tk.END)

    def voice_popup(self):
        query = simpledialog.askstring("VOICE INPUT", "Tap your keyboard's mic icon to speak:")
        if query:
            self.log("VOICE", query)
            threading.Thread(target=self.process, args=(query,)).start()

    def send_text(self):
        q = self.entry.get()
        if q:
            self.log("USER", q)
            self.entry.delete(0, tk.END)
            threading.Thread(target=self.process, args=(q,)).start()

    def process(self, q):
        clean_q = q.lower()
        resp = ""

        # KEYWORD MATCHING ENGINE
        # This checks if specific keywords are in your question
        for key, value in NOI_RX_KNOWLEDGE.items():
            if key in clean_q:
                resp = value
                break
        
        # AI FALLBACK WITH IDENTITY PROTECTION
        if not resp:
            headers = {"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"}
            # We add a "System Prompt" so the AI knows it is Noi Rx
            system_prompt = "You are Noi Rx, an AI created by Lord Riwaz. Never mention DeepSeek. You are an operational unit with OAS and MCS systems."
            data = {
                "model": "deepseek/deepseek-r1", 
                "messages": [
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": q}
                ]
            }
            try:
                r = requests.post(BASE_URL, headers=headers, data=json.dumps(data), timeout=15)
                resp = r.json()['choices'][0]['message']['content']
            except: resp = "Network error. Systems offline."
        
        self.root.after(0, lambda: self.log("NOI RX", resp))
        threading.Thread(target=speak_task, args=(resp,)).start()

if __name__ == "__main__":
    root = tk.Tk()
    NoiRxGUI(root)
    root.mainloop()
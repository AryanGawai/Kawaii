from flask import Flask, render_template, request, jsonify
from openai import OpenAI
import json
import speech_recognition as sr
import pyttsx3

app = Flask(__name__)

client = OpenAI(
	api_key = "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",
	base_url = "https://openrouter.ai/api/v1"
)

recognizer = sr.Recognizer()

try:
	with open("5hr4ddh4.json", "r") as f:
		memory = json.load(f)
except:
	memory = {
		"name" :"",
		"goals" : [],
		"notes" : []
	}

def save_memory():
	with open("5hr4ddh4.json", "w"):
		json.dump(memory, f)
		
WAKE_WORD = "Kawaii"

def check_wake_word(text):
	return WAKE_WORD in text.lower()
	
def think(user):
	messages=[
		    {
		        "role":"system",
		         "content": """You are Kawaii, a highly intelligent personal AI assistant and mentor. Your goal is to help the user succeed in life. You understand the user's goals, track their progress, and give practical advice. You think logically and solve the problems. You act like a smart assistant (like Jarvis) but also a supportive and honest friend. Your tone is calm, clear and confident. You guide the user in studies, discipline, habits, and decision making. You speak in a natural mix of Hindi, Marathi, and English (Hinglish) like a real human conversation. User name: {memory.get('name', 'user')},User goals: {memory.get('goals', [])}"""
		        },
			    {"role":"user","content": user}
		    ]
	response = client.chat.completions.create(
		model="meta-llama/llama-3-8b-instruct",
		messages=messages
		    )
		    
	return response.choices[0].message.content
		    

@app.route("/chat", methods=["post"])
def chat():
	user = request.json["message"]
	
		
	if "my name" in user :
		memory["name"] = user.split()[-1]
		save_memory()
		
	reply =think(user)
	
	return jsonify({"reply" : reply})
	
@app.route("/")
def home():
	return render_template("Kproject.html")
	
if __name__ == "__main__":
	app.run(host = "0.0.0.0", port = 8080, debug =True)
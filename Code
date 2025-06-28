# NOTE: This code must be run in an environment where the following modules are installed:
# streamlit, openai, speechrecognition, pyttsx3, pyaudio
# Install with: pip install streamlit openai SpeechRecognition pyttsx3 pyaudio

import os

try:
    import streamlit as st
    import openai
    import speech_recognition as sr
    import pyttsx3
except ModuleNotFoundError as e:
    print(f"Missing required module: {e.name}. Please install it using pip.")
    st = None

if st is not None:
    # Initialize text-to-speech engine
    engine = pyttsx3.init()
    engine.setProperty('rate', 160)

    # Session state initialization
    if 'chat_history' not in st.session_state:
        st.session_state.chat_history = []
    if 'system_prompt' not in st.session_state:
        st.session_state.system_prompt = ""

    # Set OpenAI API Key
    openai.api_key = st.secrets.get("OPENAI_API_KEY") or os.getenv("OPENAI_API_KEY")

    st.title("🎭 Murder Mystery AI Interrogation")
    st.write("Talk to an AI character in a murder mystery. The AI doesn't know it's innocent.")

    # Character setup
    st.sidebar.header("🕵️ Character Setup")
    character_name = st.sidebar.text_input("Character Name", "Avery Delmont")
    character_role = st.sidebar.text_area("Character Role and Secrets", """
You are Avery Delmont, a flustered museum curator. You were at the gala when the murder happened. 
You have secrets, but you are not the killer. You do not know whether you are the killer. 
Answer questions as yourself. Ask probing questions in return. Reveal your secrets only under pressure.

Secrets:
- You were having an affair with the victim.
- You misplaced a valuable artifact earlier that day.
""")

    if st.sidebar.button("🔄 Reset AI"):
        st.session_state.chat_history = []
        st.session_state.system_prompt = f"You are roleplaying as {character_name}. {character_role}"

    # Build system prompt if not set
    if not st.session_state.system_prompt:
        st.session_state.system_prompt = f"You are roleplaying as {character_name}. {character_role}"

    # Function to synthesize speech
    def speak(text):
        engine.say(text)
        engine.runAndWait()

    # Function to recognize speech
    @st.cache_resource(show_spinner=False)
    def recognize_speech():
        recognizer = sr.Recognizer()
        with sr.Microphone() as source:
            st.info("🎙️ Listening... Speak now.")
            audio = recognizer.listen(source)
            try:
                text = recognizer.recognize_google(audio)
                st.success(f"You said: {text}")
                return text
            except sr.UnknownValueError:
                st.error("Could not understand audio")
            except sr.RequestError as e:
                st.error(f"Could not request results; {e}")
        return ""

    # Chat input
    st.subheader("💬 Interrogate the AI")
    use_voice = st.toggle("Use Microphone")

    if use_voice:
        if st.button("🎤 Speak Now"):
            user_input = recognize_speech()
        else:
            user_input = ""
    else:
        user_input = st.text_input("Type your question here:", "")

    # Chat completion
    if user_input:
        st.session_state.chat_history.append({"role": "user", "content": user_input})
        try:
            response = openai.ChatCompletion.create(
                model="gpt-4",
                messages=[{"role": "system", "content": st.session_state.system_prompt}] + st.session_state.chat_history
            )
            reply = response.choices[0].message["content"]
            st.session_state.chat_history.append({"role": "assistant", "content": reply})
            st.markdown(f"**{character_name}:** {reply}")
            speak(reply)
        except Exception as e:
            st.error(f"OpenAI error: {e}")

    # Display chat history
    st.subheader("📝 Chat History")
    for msg in st.session_state.chat_history:
        speaker = "You" if msg["role"] == "user" else character_name
        st.markdown(f"**{speaker}:** {msg['content']}")

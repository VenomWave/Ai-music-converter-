# Ai-music-converter-
pip install streamlit openai-whisper moviepy googletrans==4.0.0-rc1 gtts
# app.py
import streamlit as st
import tempfile
import os
import whisper
from gtts import gTTS
from googletrans import Translator
import moviepy.editor as mp

# Load Whisper model
model = whisper.load_model("base")
translator = Translator()

st.title("AI Voice Translator App")
st.write("Upload an audio or video file. The app will transcribe, translate, and speak it in another language.")

uploaded_file = st.file_uploader("Upload Audio or Video", type=["mp4", "mp3", "wav", "m4a", "mov"])
target_lang = st.selectbox("Select Target Language", options=["fr", "es", "de", "hi", "zh-cn", "ja", "ru"])

if uploaded_file:
    with tempfile.NamedTemporaryFile(delete=False) as tmp:
        tmp.write(uploaded_file.read())
        tmp_path = tmp.name

    st.audio(tmp_path)

    # Extract audio from video if needed
    if uploaded_file.name.endswith(('.mp4', '.mov')):
        video = mp.VideoFileClip(tmp_path)
        audio_path = tmp_path + ".mp3"
        video.audio.write_audiofile(audio_path)
    else:
        audio_path = tmp_path

    st.write("Transcribing...")
    result = model.transcribe(audio_path)
    original_text = result["text"]
    st.success("Transcription complete!")
    st.text_area("Original Text", value=original_text, height=100)

    st.write("Translating...")
    translated = translator.translate(original_text, dest=target_lang)
    translated_text = translated.text
    st.success("Translation complete!")
    st.text_area("Translated Text", value=translated_text, height=100)

    st.write("Generating Voice...")
    tts = gTTS(text=translated_text, lang=target_lang)
    tts_path = tmp_path + "_tts.mp3"
    tts.save(tts_path)
    audio_file = open(tts_path, 'rb')
    audio_bytes = audio_file.read()
    st.audio(audio_bytes, format='audio/mp3')

    # Cleanup
    os.remove(tmp_path)
    if os.path.exists(audio_path):
        os.remove(audio_path)
    os.remove(tts_path)
    

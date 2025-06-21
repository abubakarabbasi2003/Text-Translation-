!pip install transformers sentencepiece torch --quiet
import torch
from transformers import AutoTokenizer, AutoModelForSeq2SeqLM

# Use GPU if available
device = "cuda" if torch.cuda.is_available() else "cpu"

# Load NLLB-200 model and tokenizer
model_name = "facebook/nllb-200-distilled-600M"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSeq2SeqLM.from_pretrained(model_name).to(device)

# Language codes for target languages
lang_codes = {
    "urdu": "urd_Arab",
    "arabic": "arb_Arab",
    "spanish": "spa_Latn",
    "russian": "rus_Cyrl",
    "chinese": "zho_Hans",
    "hindi": "hin_Deva",
    "french": "fra_Latn",
    "german": "deu_Latn"
}

def translate(text, target_lang_code):
    tokenizer.src_lang = "eng_Latn"
    inputs = tokenizer(text, return_tensors="pt").to(device)

    # 🔥 Fix: use convert_tokens_to_ids() to get the target token ID
    target_lang_id = tokenizer.convert_tokens_to_ids(target_lang_code)

    output_tokens = model.generate(
        **inputs,
        forced_bos_token_id=target_lang_id,
        max_length=256
    )

    return tokenizer.batch_decode(output_tokens, skip_special_tokens=True)[0]

# Interface
print("🌐 English → Multilingual Translator (NLLB-200)")
print("Supported: Urdu, Arabic, Spanish, Russian, Chinese, Hindi, French, German")
print("Type 'exit' to quit.\n")

while True:
    text = input("Enter English text: ").strip()
    if text.lower() == "exit":
        print("Exiting.")
        break

    print("Choose target language:", ", ".join(lang_codes.keys()))
    lang = input("Language: ").strip().lower()

    if lang not in lang_codes:
        print("❌ Unsupported language. Try again.")
        continue

    target_lang = lang_codes[lang]
    translated_text = translate(text, target_lang)
    print(f"→ {lang.capitalize()} Translation: {translated_text}")
    print("-" * 50)

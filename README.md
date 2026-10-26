# language-translator

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Language Translator</title>
<style>
body {
  font-family: Arial, sans-serif;
  background: #f0f4f8;
  margin: 0;
  padding: 30px 15px;
}
.container {
  max-width: 700px;
  margin: auto;
  background: white;
  padding: 25px;
  border-radius: 15px;
  box-shadow: 0 4px 15px #0001;
}
h1 { text-align: center; color: #2457c5; }
select, textarea, button {
  width: 100%;
  box-sizing: border-box;
  padding: 12px;
  margin: 8px 0;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 16px;
}
textarea { min-height: 140px; resize: vertical; }
.languages {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
button {
  background: #2457c5;
  color: white;
  border: none;
  cursor: pointer;
}
button:hover { background: #173e95; }
.output {
  min-height: 100px;
  padding: 15px;
  background: #f5f7fb;
  border-radius: 8px;
  overflow-wrap: anywhere;
  white-space: pre-wrap;
}
.actions { display: flex; gap: 10px; }
.actions button { flex: 1; }
#status { color: #555; font-size: 14px; }
</style>
</head>
<body>
<div class="container">
  <h1>🌐 Language Translator</h1>

  <div class="languages">
    <div>
      <label for="source">Translate from</label>
      <select id="source">
        <option value="en">English</option>
        <option value="hi">Hindi</option>
        <option value="es">Spanish</option>
        <option value="fr">French</option>
        <option value="de">German</option>
        <option value="bn">Bengali</option>
        <option value="ta">Tamil</option>
        <option value="ur">Urdu</option>
        <option value="ja">Japanese</option>
        <option value="ar">Arabic</option>
      </select>
    </div>
    <div>
      <label for="target">Translate to</label>
      <select id="target">
        <option value="hi">Hindi</option>
        <option value="en">English</option>
        <option value="es">Spanish</option>
        <option value="fr">French</option>
        <option value="de">German</option>
        <option value="bn">Bengali</option>
        <option value="ta">Tamil</option>
        <option value="ur">Urdu</option>
        <option value="ja">Japanese</option>
        <option value="ar">Arabic</option>
      </select>
    </div>
  </div>

  <button id="swap" type="button">⇄ Swap Languages</button>

  <label for="inputText">Enter your text</label>
  <textarea id="inputText" maxlength="4500"
    placeholder="Type or paste text here..."></textarea>

  <button id="translate" type="button">Translate Text</button>

  <p><strong>Translated Text</strong></p>
  <div class="output" id="output">Your translation will appear here.</div>

  <div class="actions">
    <button id="copy" type="button">Copy Translation</button>
    <button id="speak" type="button">🔊 Listen</button>
  </div>
  <p id="status" role="status"></p>
</div>

<script>
const $ = id => document.getElementById(id);
const names = {
  en:"English", hi:"Hindi", es:"Spanish", fr:"French",
  de:"German", bn:"Bengali", ta:"Tamil", ur:"Urdu",
  ja:"Japanese", ar:"Arabic"
};
let translated = "";
let busy = false;

$("translate").addEventListener("click", async () => {
  const text = $("inputText").value.trim();
  const source = $("source").value;
  const target = $("target").value;

  if (!text) {
    $("status").textContent = "Please enter some text.";
    return;
  }
  if (busy) return;
  if (source === target) {
    translated = text;
    $("output").textContent = text;
    $("status").textContent = "Both languages are the same.";
    return;
  }

  busy = true;
  $("translate").disabled = true;
  $("translate").textContent = "Translating...";
  $("status").textContent = "Please wait...";

  try {
    const params = new URLSearchParams({
      q: text,
      langpair: source + "|" + target
    });
    const response = await fetch(
      "https://api.mymemory.translated.net/get?" +
      params.toString()
    );
    if (!response.ok) throw new Error("Translation request failed.");

    const data = await response.json();
    if (data.responseStatus !== 200 ||
        !data.responseData?.translatedText) {
      throw new Error("Translation unavailable. Try again.");
    }

    translated = data.responseData.translatedText;
    $("output").textContent = translated;
    $("status").textContent =
      "Translated to " + names[target] + ".";
  } catch (error) {
    $("status").textContent =
      "Error: " + error.message +
      " Check your internet connection.";
  } finally {
    busy = false;
    $("translate").disabled = false;
    $("translate").textContent = "Translate Text";
  }
});

$("swap").addEventListener("click", () => {
  const oldSource = $("source").value;
  $("source").value = $("target").value;
  $("target").value = oldSource;

  if (translated) {
    const original = $("inputText").value;
    $("inputText").value = translated;
    translated = original;
    $("output").textContent = original;
  }
  $("status").textContent = "Languages swapped.";
});

$("copy").addEventListener("click", async () => {
  if (!translated) {
    $("status").textContent = "Translate text first.";
    return;
  }
  try {
    await navigator.clipboard.writeText(translated);
    $("status").textContent = "Translation copied!";
  } catch {
    $("status").textContent =
      "Clipboard unavailable. Select and copy the text manually.";
  }
});

$("speak").addEventListener("click", () => {
  if (!translated) {
    $("status").textContent = "Translate text first.";
    return;
  }
  if (!("speechSynthesis" in window)) {
    $("status").textContent = "Speech is not supported in this browser.";
    return;
  }
  speechSynthesis.cancel();
  const speech = new SpeechSynthesisUtterance(translated);
  speech.lang = {
    en:"en-US", hi:"hi-IN", es:"es-ES", fr:"fr-FR",
    de:"de-DE", bn:"bn-IN", ta:"ta-IN", ur:"ur-PK",
    ja:"ja-JP", ar:"ar-SA"
  }[$("target").value] || $("target").value;
  speechSynthesis.speak(speech);
});
</script>
</body>
</html>

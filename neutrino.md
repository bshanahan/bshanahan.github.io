---
layout: compress
title: De-bias your news!
---

# Tempers, tempered.

Choose a model:
<label for="model-select">Choose model:</label>
<select id="model-select">
  <option value="gpt-4o-mini">GPT-4o-mini</option>
  <option value="gpt-3.5-turbo">GPT-3.5-turbo</option>
  <option value="google/gemini-2.5-flash">Gemini 2.5</option>
  <option value="anthropic/claude-3-haiku">Claude 3</option>
</select>

Paste a URL below to have AI remove bias from the page.

<pre id="output" style="white-space: pre-wrap; margin-top: 1em;"></pre>

<form id="neutrino-form">
  <input
    type="url"
    id="url-input"
    placeholder="https://example.com"
    required
    style="width: 100%; padding: 0.5em;"
  />
  <button type="button" id="debias-btn">Debias (Clean Only)</button>
  <button type="button" id="debias-explain-btn">Debias + Explain</button>
</form>

<script>
async function fetchNeutrino(url, model) {
  const res = await fetch(
    `https://neutrino-vercel.vercel.app/api/neutrino?url=${encodeURIComponent(url)}&model=${encodeURIComponent(model)}`
  );
  return res.json();
}

const output = document.getElementById("output");
const urlInput = document.getElementById("url-input");
const modelSelect = document.getElementById("model-select");

document.getElementById("debias-btn").addEventListener("click", async () => {
  const url = urlInput.value;
  const model = modelSelect.value;
  output.textContent = "Processing…";

  try {
    const data = await fetchNeutrino(url, model);
    output.textContent = data.cleaned_text || "No output";
  } catch (err) {
    output.textContent = "Request failed: " + err.message;
  }
});

document.getElementById("debias-explain-btn").addEventListener("click", async () => {
  const url = urlInput.value;
  const model = modelSelect.value;
  output.textContent = "Processing…";

  try {
    const data = await fetchNeutrino(url, model);
    let text = data.cleaned_text || "No output";

    if (data.summary_of_changes && data.summary_of_changes.length > 0) {
      text += "\n\nSummary of changes:\n- " + data.summary_of_changes.join("\n- ");
    }

    output.textContent = text;
  } catch (err) {
    output.textContent = "Request failed: " + err.message;
  }
});
</script>

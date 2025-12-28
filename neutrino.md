---
layout: default
title: De-bias your news!
---

# Tempers, tempered.

<label for="model-select">Choose model:</label>
<select id="model-select">
  <option value="anthropic/claude-3-haiku">Claude 3</option>
  <option value="gpt-4o-mini">GPT-4o Mini</option>
  <option value="gpt-3.5-turbo">GPT-3.5</option>
  <option value="google/gemini-2.5-flash">Gemini 2.5 Flash</option>
</select>

<form id="neutrino-form" style="margin-top: 1em;">
  <input
    type="url"
    id="url-input"
    placeholder="https://example.com"
    required
    style="width: 100%; padding: 0.5em;"
  />

  <div style="margin-top: 0.5em;">
    <button type="button" id="debias-btn">Debias</button>
    <button type="button" id="debias-explain-btn">Debias + Explain</button>
    <button type="button" id="debias-factcheck-btn">Debias + Fact-check</button>
  </div>
</form>

<pre id="output" style="white-space: pre-wrap; margin-top: 1em;"></pre>

<script>
const output = document.getElementById("output");
const urlInput = document.getElementById("url-input");
const modelSelect = document.getElementById("model-select");

async function fetchNeutrino(url, model) {
  const res = await fetch(
    `https://neutrino-vercel.vercel.app/api/neutrino?url=${encodeURIComponent(
      url
    )}&model=${encodeURIComponent(model)}`
  );

  if (!res.ok) {
    throw new Error("Request failed");
  }

  return res.json();
}

document.getElementById("debias-btn").addEventListener("click", async () => {
  output.textContent = "Processing…";

  try {
    const data = await fetchNeutrino(urlInput.value, modelSelect.value);
    output.textContent = data.cleaned_text || "No output";
  } catch (err) {
    output.textContent = err.message;
  }
});

document
  .getElementById("debias-explain-btn")
  .addEventListener("click", async () => {
    output.textContent = "Processing…";

    try {
      const data = await fetchNeutrino(urlInput.value, modelSelect.value);

      let text = data.cleaned_text || "No output";

      if (data.summary_of_changes?.length) {
        text += "\n\nSummary of changes:\n";
        text += data.summary_of_changes.map((c) => `- ${c}`).join("\n");
      }

      output.textContent = text;
    } catch (err) {
      output.textContent = err.message;
    }
  });

document
  .getElementById("debias-factcheck-btn")
  .addEventListener("click", async () => {
    output.textContent = "Processing…";

    try {
      const data = await fetchNeutrino(urlInput.value, modelSelect.value);

      let text = data.cleaned_text || "No output";

      if (data.fact_check_summary?.length) {
        text += "\n\nFact-check summary:\n";
        text += data.fact_check_summary.map((f) => `- ${f}`).join("\n");
      }

      output.textContent = text;
    } catch (err) {
      output.textContent = err.message;
    }
  });
</script>

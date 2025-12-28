---
layout: nosidebar
title: De-bias your news!
---

# Tempers, tempered.

Enter a URL, and let AI try to remove any inherent bias.

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
    <button type="button" id="debias-claims-btn">Debias + Claims + Fact-check</button>

  </div>
</form>

<pre id="output" style="white-space: pre-wrap; margin-top: 1em;"></pre>
<pre id="claims-output" style="white-space: pre-wrap; margin-top: 1em;"></pre>

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

const claimsOutput = document.getElementById("claims-output");

document
  .getElementById("debias-claims-btn")
  .addEventListener("click", async () => {
    output.textContent = "Processing…";
    claimsOutput.textContent = "";

    try {
      const data = await fetchNeutrino(
        urlInput.value,
        modelSelect.value
      );

      let text = data.cleaned_text || "No output";
      output.textContent = text;

      let claimsText = "";

      if (data.extracted_claims?.length) {
        claimsText += "Extracted factual claims:\n";
        claimsText += data.extracted_claims
          .map((c) => `- ${c}`)
          .join("\n");
      } else {
        claimsText += "No extractable factual claims found.";
      }

      if (data.fact_check_summary?.length) {
        claimsText += "\n\nFact-check summary:\n";
        claimsText += data.fact_check_summary
          .map((f) => `- ${f}`)
          .join("\n");
      }

      claimsOutput.textContent = claimsText;
    } catch (err) {
      output.textContent = err.message;
    }
  });
</script>

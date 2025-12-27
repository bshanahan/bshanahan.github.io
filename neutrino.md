---
layout: compress
title: De-bias your news!
---

# Tempers, tempered.

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
async function fetchNeutrino(url) {
  const res = await fetch(
    "https://neutrino-vercel.vercel.app/api/neutrino?url=" +
      encodeURIComponent(url)
  );
  return res.json();
}

const output = document.getElementById("output");
const urlInput = document.getElementById("url-input");

document.getElementById("debias-btn").addEventListener("click", async () => {
  const url = urlInput.value;
  output.textContent = "Processing…";

  try {
    const data = await fetchNeutrino(url);
    output.textContent = data.cleaned_text || "No output";
  } catch (err) {
    output.textContent = "Request failed: " + err.message;
  }
});

document
  .getElementById("debias-explain-btn")
  .addEventListener("click", async () => {
    const url = urlInput.value;
    output.textContent = "Processing…";

    try {
      const data = await fetchNeutrino(url);
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

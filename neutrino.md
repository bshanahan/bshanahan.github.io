---
layout: nosidebar
title: De-bias your news!
---

# Tempers, tempered.

Enter a URL, and let AI try to remove any inherent bias.

<label for="model-select">Choose model:</label>
<select id="model-select">
  <option value="anthropic/claude-3-haiku">Claude 3</option>
  <option value="openai/gpt-5.2-chat">GPT-5.2 Chat</option>
  <option value="openai/gpt-4o-mini">GPT-4o Mini</option>
  <option value="openai/gpt-3.5-turbo">GPT-3.5</option>
  <option value="google/gemini-2.5-flash">Gemini 2.5 Flash</option>
  <option value="google/gemini-3-flash-preview">Gemini 3 Flash Preview</option>
  
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
    <label><input type="checkbox" id="opt-debias" checked> Debias text</label><br>
    <label><input type="checkbox" id="opt-explain"> Explain inherent bias</label><br>
    <label><input type="checkbox" id="opt-claims"> Extract factual claims</label><br>
    <label><input type="checkbox" id="opt-factcheck"> Fact-check claims</label><br>

    <button type="button" id="go-btn" style="margin-top: 0.5em;">
      Go
    </button>
  </div>
</form>

<h2 id="article-title"></h2>

<pre id="output" style="white-space: pre-wrap; margin-top: 1em;"></pre>
<pre id="claims-output" style="white-space: pre-wrap; margin-top: 1em;"></pre>

<script>
document.addEventListener("DOMContentLoaded", () => {
  const output = document.getElementById("output");
  const claimsOutput = document.getElementById("claims-output");
  const urlInput = document.getElementById("url-input");
  const modelSelect = document.getElementById("model-select");
  const articleTitle = document.getElementById("article-title");

  const optDebias = document.getElementById("opt-debias");
  const optExplain = document.getElementById("opt-explain");
  const optClaims = document.getElementById("opt-claims");
  const optFactcheck = document.getElementById("opt-factcheck");

  async function fetchNeutrino(url, model) {
    const res = await fetch(
      `https://neutrino-vercel.vercel.app/api/neutrino?url=${encodeURIComponent(url)}&model=${encodeURIComponent(model)}`
    );

    const text = await res.text();

    if (!res.ok) {
      throw new Error(text || "Backend error");
    }

    return JSON.parse(text);
  }

  document.getElementById("go-btn").addEventListener("click", async () => {
    output.textContent = "Processing…";
    claimsOutput.textContent = "";
    articleTitle.textContent = "";

    try {
      const data = await fetchNeutrino(
        urlInput.value,
        modelSelect.value
      );

      articleTitle.textContent = data.title || "";

      let mainText = "";

      if (optDebias.checked && data.cleaned_text) {
        mainText += data.cleaned_text;
      }

      if (optExplain.checked && data.summary_of_changes?.length) {
        mainText += "\n\nExplanation of bias removed:\n";
        mainText += data.summary_of_changes.map(c => `- ${c}`).join("\n");
      }

      if (optFactcheck.checked && !optClaims.checked && data.fact_check_summary?.length) {
        mainText += "\n\nFact-check summary:\n";
        mainText += data.fact_check_summary.map(f => `- ${f}`).join("\n");
      }

      output.textContent = mainText || "No output returned.";

      if (optClaims.checked) {
        let claimsText = "";

        if (data.extracted_claims?.length) {
          claimsText += "Extracted factual claims:\n";
          claimsText += data.extracted_claims.map(c => `- ${c}`).join("\n");
        } else {
          claimsText += "No extractable factual claims found.";
        }

        if (optFactcheck.checked && data.fact_check_summary?.length) {
          claimsText += "\n\nFact-check summary:\n";
          claimsText += data.fact_check_summary.map(f => `- ${f}`).join("\n");
        }

        claimsOutput.textContent = claimsText;
      }
    } catch (err) {
      output.textContent = "Error:\n" + err.message;
    }
  });
});
</script>

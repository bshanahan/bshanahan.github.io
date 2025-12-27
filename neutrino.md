---
layout: compress
title: De-bias your news!
---

# De-bias your news!

Paste a URL below to remove bias from the page.

<form id="neutrino-form">
  <input
    type="url"
    id="url-input"
    placeholder="https://example.com"
    required
    style="width: 100%; padding: 0.5em;"
  />
  <button type="submit">Debias</button>
</form>

<pre id="output" style="white-space: pre-wrap; margin-top: 1em;"></pre>

<script>
document.getElementById("neutrino-form").addEventListener("submit", async (e) => {
  e.preventDefault();

  const url = document.getElementById("url-input").value;
  const output = document.getElementById("output");

  output.textContent = "Processing…";

  try {
    const res = await fetch(
      "https://neutrino-vercel.vercel.app/api/neutrino?url=" +
        encodeURIComponent(url)
    );
    const data = await res.json();

    if (data.cleaned_text) {
      output.textContent = data.cleaned_text;
    } else {
      output.textContent = "Error: " + JSON.stringify(data, null, 2);
    }
  } catch (err) {
    output.textContent = "Request failed: " + err.message;
    console.error(err);
  }
});
</script>

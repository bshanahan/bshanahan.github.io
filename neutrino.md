---
layout: compress
title: De-bias your news!
---

# De-bias your news!

Paste a URL below to remove bias from the page.

<pre id="output" style="white-space: pre-wrap; margin-top: 1em;"></pre>
<button id="summary-btn" style="margin-top:1em; display:none;">Show Changes & Rationale</button>
<pre id="summary" style="white-space: pre-wrap; display:none;"></pre>

<script>
document.getElementById("neutrino-form").addEventListener("submit", async (e) => {
  e.preventDefault();

  const url = document.getElementById("url-input").value;
  const output = document.getElementById("output");
  const summaryBtn = document.getElementById("summary-btn");
  const summaryPre = document.getElementById("summary");

  output.textContent = "Processing…";
  summaryBtn.style.display = "none";
  summaryPre.style.display = "none";

  try {
    const res = await fetch(
      "https://neutrino-vercel.vercel.app/api/neutrino?url=" +
        encodeURIComponent(url)
    );
    const data = await res.json();

    output.textContent = data.cleaned || "No output";

    if (data.summary_of_changes && data.summary_of_changes.length > 0) {
      summaryBtn.style.display = "inline-block";
      summaryBtn.onclick = () => {
        summaryPre.textContent = data.summary_of_changes.join("\n- ");
        summaryPre.style.display = "block";
      };
    }
  } catch (err) {
    output.textContent = "Request failed: " + err.message;
  }
});
</script>

# solvely.ai's "multi-detector" scan is completely fake

I almost panicked over a 73% AI score on solvely.ai before realizing the actual detectors (GPTZero, Originality, Grammarly) all said my text was 93% to 100% human. 

So I popped open DevTools to see how they were actually pulling those numbers. Spoiler: they aren't.

## what's actually happening

When you run a scan, solvely shows a scary popup claiming five different platforms flagged your text. If you watch the Network tab, zero requests ever go out to Turnitin, GPTZero, Originality.ai, or Grammarly. The site just loads their logos locally as static SVGs and runs a tiny math function on the frontend to fake the results.

Here is the exact function pulled straight from their JS bundle:

```markdown

var e = [`zerogpt`, `gptzero`, `originality`, `grammarly`];

function t(e) {
    let t = Math.round(e * 10) / 10;
    return t >= 80 ? 4 : t >= 75 ? 3 : t >= 70 ? 2 : 1
}

function n(n, r) {
    let i = t(n)
      , a = [...e]
      , o = []
      , s = Math.abs(Math.trunc(r)) % 24;
    for (let e = a.length; e > 0; e--) {
        let[t] = a.splice(s % e, 1);
        o.push(t),
        s = Math.floor(s / e)
    }
    let c = new Set(o.slice(0, i));
    return [{
        channel: `turnitin`,
        level: `high`
    }, ...e.map(e => ({
        channel: e,
        level: c.has(e) ? `high` : `moderate`
    }))]
}

export {n as t};

```

## breaking down the code

* **Turnitin is hardcoded:** Look at the return statement: `{channel: 'turnitin', level: 'high'}`. This is hardcoded. No matter what you paste in, Turnitin will always say High AI. No matter if you paste Shakespeare, a grocery list, literally ANYTHING, it will always say high.
* **The rest is just modulo math:** `t(e)` just checks if ur base score is over 70, 75, or 80 to choose how many platforms should show high instead of moderate. Then, `% 24` just shuffles the array of logos so the output looks different each time.
* **Zero API calls:** There are NO api keys, NO external fetches, OR third-party checks anywhere in the flow. It's just 100% hardcoded client-side to get you to buy a subscription to paraphrase it.

## takeaway

Don't reliably trust aggregator platforms that promise to detect your text using multiple AI platforms. Test each platform on their own site.

---

## evidence

<table>
  <tr>
    <th>Solvely's Fake Multi-Detector Popup</th>
    <th>2 Sample Detector Results</th>
  </tr>
  <tr>
    <td rowspan="2" align="center">
      <img width="480" alt="Screenshot 2026-11-11 214923" src="https://github.com/user-attachments/assets/faffdd18-37ed-41b0-ad76-f6f18f7fd601" />
    </td>
    <td align="center">
      <img width="320" alt="Screenshot 2026-11-11 215430" src="https://github.com/user-attachments/assets/dd2b4773-bcaa-4ef6-945a-53ed18d76a36" />
    </td>
  </tr>
  <tr>
    <td align="center">
      <img width="320" alt="Screenshot 2026-11-11 215458" src="https://github.com/user-attachments/assets/53aeef22-58f3-41fa-8da0-ebdf0a549ca1" />
    </td>
  </tr>
</table>

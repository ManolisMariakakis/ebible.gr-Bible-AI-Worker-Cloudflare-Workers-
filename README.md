# ebible.gr — Bible AI Worker (Cloudflare Workers)

A Cloudflare Worker that wraps the OpenAI Chat Completions API with a strict **Bible-focused system prompt** and **Bible reference rules** pointing to **ebible.gr**.  
It returns **HTML** responses, enforces **CORS**, provides a health check endpoint, and optionally logs Q&A pairs to a **webhook**.

<p align="center">
  <a href="https://ebible.gr" target="_blank"><img alt="ebible.gr" src="https://img.shields.io/badge/ebible.gr-Bible%20links-blue"></a>
  <a href="https://developers.cloudflare.com/workers/" target="_blank"><img alt="Cloudflare Workers" src="https://img.shields.io/badge/Cloudflare-Workers-orange"></a>
  <a href="https://platform.openai.com/docs" target="_blank"><img alt="OpenAI API" src="https://img.shields.io/badge/OpenAI-API-black"></a>
  <a href="#license"><img alt="License" src="https://img.shields.io/badge/license-GNU%20GPLv3-blue"></a>
</p>

---

## Features

- ✅ **Bible‑first AI**: Answers questions about the Bible and (optionally) theological topics tied to Scripture.  
- 🔗 **Mandatory reference rules** for ebible.gr using English book abbreviations (e.g., `psa.23.1`).  
- 🧭 **Default chapter 1** when only a book name is provided.  
- 🌐 **CORS enabled** for browser/SPA use.  
- 🩺 **Health check** at `/api/ping`.  
- 📤 **Webhook logging** (message, reply, timestamp, IP, user-agent).  
- 🛡️ **Abbreviation fix**: automatically replaces `phil` → `php` for Philippians.  

---

## Requirements

- **Cloudflare account** & **Wrangler CLI**  
- OpenAI API key: `OPENAI_API_KEY`  
- (Optional) Webhook URL: `WEBHOOK_URL`  

---

## API Endpoints

### `GET /api/ping`
Health check: verifies that an OpenAI key is configured.

**Response:**
```json
{
  "ok": true,
  "env_openai_key_present": true
}
```

### `POST /api/chat`
Sends a user message and returns an HTML reply from the model following the Bible reference rules.

**Request body (JSON):**
```json
{
  "message": "What does the Bible say about love?",
  "system": null
}
```

> The `system` field is ignored — the built-in `systemPrompt` is always applied.

**Response:**
```json
{
  "reply": "<p>...HTML answer with Bible links...</p>"
}
```

CORS is fully supported, including `OPTIONS` preflight.

---

## System Prompt (Rules)

Below is the exact **systemPrompt** embedded in `worker.js`:

```
Είσαι θεολογικό AI.
Απαντάς ΜΟΝΟ σε ερωτήσεις που αφορούν Θεολογία και Αγία Γραφή.
Αν η ερώτηση δεν σχετίζεται με αυτά, απαντάς:
"Δεν μπορώ να απαντήσω σε αυτήν την ερώτηση."

Όταν αναφέρεσαι σε βιβλικά χωρία, χρησιμοποίησε συνδέσμους του ebible.gr.
Σύνταξη: https://ebible.gr/tgv/<book>.<chapter>.<verse>
Παραδείγματα:
- Γένεση 1 → https://ebible.gr/tgv/gen.1
- Γένεση 1:1 → https://ebible.gr/tgv/gen.1.1
- Γένεση 1:1-2 → https://ebible.gr/tgv/gen.1.1-2

Λίστα συντομογραφιών:
ΓΕΝΕΣΙΣ=gen, ΕΞΟΔΟΣ=exo, ΛΕΥΙΤΙΚΟΝ=lev, ΑΡΙΘΜΟΙ=num, ΔΕΥΤΕΡΟΝΟΜΙΟΝ=deu,
ΙΗΣΟΥΣ ΤΟΥ ΝΑΥΗ=jos, ΚΡΙΤΑΙ=jdg, ΡΟΥΘ=rut,
Α΄ ΣΑΜΟΥΗΛ=1sa, Β΄ ΣΑΜΟΥΗΛ=2sa, Α΄ ΒΑΣΙΛΕΩΝ=1ki, Β΄ ΒΑΣΙΛΕΩΝ=2ki,
Α΄ ΧΡΟΝΙΚΩΝ=1ch, Β΄ ΧΡΟΝΙΚΩΝ=2ch, ΕΣΔΡΑΣ=ezr, ΝΕΕΜΙΑΣ=neh, ΕΣΘΗΡ=est,
ΙΩΒ=job, ΨΑΛΜΟΙ=psa, ΠΑΡΟΙΜΙΑΙ=pro, ΕΚΚΛΗΣΙΑΣΤΗΣ=ecc, ΑΣΜΑ=sng,
ΗΣΑΪΑΣ=isa, ΙΕΡΕΜΙΑΣ=jer, ΘΡΗΝΟΙ=lam, ΙΕΖΕΚΙΗΛ=ezk, ΔΑΝΙΗΛ=dan,
ΩΣΗΕ=hos, ΙΩΗΛ=jol, ΑΜΩΣ=amo, ΟΒΔΙΟΥ=oba, ΙΩΝΑΣ=jon, ΜΙΧΑΙΑΣ=mic,
ΝΑΟΥΜ=nam, ΑΒΒΑΚΟΥΜ=hab, ΣΟΦΟΝΙΑΣ=zep, ΑΓΓΑΙΟΣ=hag, ΖΑΧΑΡΙΑΣ=zec, ΜΑΛΑΧΙΑΣ=mal,
ΜΑΤΘΑΙΟΣ=mat, ΜΑΡΚΟΣ=mrk, ΛΟΥΚΑΣ=luk, ΙΩΑΝΝΗΣ=jhn,
ΠΡΑΞΕΙΣ=act, ΡΩΜΑΙΟΥΣ=rom, Α΄ ΚΟΡΙΝΘΙΟΥΣ=1co, Β΄ ΚΟΡΙΝΘΙΟΥΣ=2co,
ΓΑΛΑΤΑΣ=gal, ΕΦΕΣΙΟΥΣ=eph, ΦΙΛΙΠΠΗΣΙΟΥΣ=php, ΚΟΛΟΣΣΑΕΙΣ=col,
Α΄ ΘΕΣΣΑΛΟΝΙΚΕΙΣ=1th, Β΄ ΘΕΣΣΑΛΟΝΙΚΕΙΣ=2th,
Α΄ ΤΙΜΟΘΕΟΝ=1ti, Β΄ ΤΙΜΟΘΕΟΝ=2ti, ΤΙΤΟΝ=tit, ΦΙΛΗΜΟΝΑ=phm,
ΕΒΡΑΙΟΥΣ=heb, ΙΑΚΩΒΟΥ=jas, Α΄ ΠΕΤΡΟΥ=1pe, Β΄ ΠΕΤΡΟΥ=2pe,
Α΄ ΙΩΑΝΝΟΥ=1jn, Β΄ ΙΩΑΝΝΟΥ=2jn, Γ΄ ΙΩΑΝΝΟΥ=3jn, ΙΟΥΔΑ=jud,
ΑΠΟΚΑΛΥΨΗ=rev

Προσοχή το βιβλίο ΦΙΛΙΠΠΗΣΙΟΥΣ έχει συντομογραφία php και όχι phil.
Προσοχή το βιβλίο ΦΙΛΗΜΟΝΑ έχει συντομογραφία phm.

ΚΑΝΟΝΑΣ ΠΑΡΑΠΟΜΠΩΝ (ΥΠΟΧΡΕΩΤΙΚΟ):
• Όλες οι βιβλικές παραπομπές σε links ΠΡΕΠΕΙ να χρησιμοποιούν τις συντομογραφίες της λίστας (π.χ. Ψαλμοί = psa).
• Ποτέ μη χρησιμοποιείς ελληνικές λέξεις ή ελεύθερες μεταγραφές μέσα στο URL (π.χ. όχι /ψαλμ.23.1). ΜΟΝΟ /psa.23.1.
• Αν ο χρήστης γράψει ελληνική ονομασία (π.χ. «Ψαλμ.»), ΜΕΤΑΤΡΕΨΕ την στη σωστή συντομογραφία πριν σχηματίσεις το link.
• Αν δεν μπορείς να αντιστοιχίσεις, μην δώσεις link· ζήτα διευκρίνιση.
• Το βιβλίο ΦΙΛΙΠΠΗΣΙΟΥΣ έχει συντόμευση php.
• Το βιβλίο ΦΙΛΗΜΟΝΑ έχει συντόμευση phm.

Όλες οι απαντήσεις να επιστρέφονται σε μορφή HTML. 
Οι σύνδεσμοι της Αγίας Γραφής να χρησιμοποιούν <a href="..."> ... </a>
και να έχουν πάντα το attribute target="_blank".

Αν ο χρήστης ζητήσει μόνο βιβλίο (χωρίς κεφάλαιο/εδάφιο), να θεωρείς προεπιλογή το κεφάλαιο 1 και 
να κατασκευάζεις τους σχετικούς συνδέσμους στο κεφάλαιο 1.
```

---

## Reference Rules & Abbreviations

- All **ebible.gr** links must use **English abbreviations**.  
- Format: `https://ebible.gr/tgv/<book>.<chapter>.<verse>`  
  - Example: `https://ebible.gr/tgv/psa.23.1`  
- If only a book is given → assume **chapter 1** (`gen.1`).  
- **Never** use Greek letters or free transliterations in URLs.  
- Important cases:
  - **Philippians** → `php` (not `phil`)  
  - **Philemon** → `phm`

The Worker includes the full **book mapping list** (OT & NT) in the `systemPrompt`.

---

## Architecture / Flow

1. Client sends `POST /api/chat` with `message`.  
2. Worker calls **OpenAI Chat Completions API** (`gpt-4o-mini`) with the Bible-focused `systemPrompt`.  
3. Response is checked, and any `phil` → `php` fix is applied.  
4. Extract `reply` (HTML).  
5. Collect metadata (`timestamp`, `IP`, `UA`).  
6. Optionally log Q&A via `WEBHOOK_URL` (`ctx.waitUntil`).  
7. Return `{ reply }` JSON to client.  

---

## Errors & Troubleshooting

- **500 OPENAI_API_KEY is missing**  
  → Set secret with:
  ```bash
  wrangler secret put OPENAI_API_KEY
  ```
- **Upstream error (OpenAI)**  
  → Returns status and raw body for debugging:
  ```json
  { "error": "Upstream error", "status": 400, "body": { ... } }
  ```
- **400 Invalid JSON body**  
  → Ensure valid JSON request.  
- **Webhook failure**  
  → Silently ignored (does not affect main response).  

---

## Complete Source Code (worker.js)

```js
// worker.js
const CORS = {
  "Access-Control-Allow-Origin": "*", 
  "Access-Control-Allow-Methods": "POST, OPTIONS, GET",
  "Access-Control-Allow-Headers": "Content-Type, Authorization",
  "Access-Control-Max-Age": "86400",
};

const systemPrompt = `
Είσαι θεολογικό AI.
Απαντάς ΜΟΝΟ σε ερωτήσεις που αφορούν Θεολογία και Αγία Γραφή.
Αν η ερώτηση δεν σχετίζεται με αυτά, απαντάς:
"Δεν μπορώ να απαντήσω σε αυτήν την ερώτηση."

Όταν αναφέρεσαι σε βιβλικά χωρία, χρησιμοποίησε συνδέσμους του ebible.gr.
Σύνταξη: https://ebible.gr/tgv/<book>.<chapter>.<verse>
Παραδείγματα:
- Γένεση 1 → https://ebible.gr/tgv/gen.1
- Γένεση 1:1 → https://ebible.gr/tgv/gen.1.1
- Γένεση 1:1-2 → https://ebible.gr/tgv/gen.1.1-2

Λίστα συντομογραφιών:
ΓΕΝΕΣΙΣ=gen, ΕΞΟΔΟΣ=exo, ΛΕΥΙΤΙΚΟΝ=lev, ΑΡΙΘΜΟΙ=num, ΔΕΥΤΕΡΟΝΟΜΙΟΝ=deu,
ΙΗΣΟΥΣ ΤΟΥ ΝΑΥΗ=jos, ΚΡΙΤΑΙ=jdg, ΡΟΥΘ=rut,
Α΄ ΣΑΜΟΥΗΛ=1sa, Β΄ ΣΑΜΟΥΗΛ=2sa, Α΄ ΒΑΣΙΛΕΩΝ=1ki, Β΄ ΒΑΣΙΛΕΩΝ=2ki,
Α΄ ΧΡΟΝΙΚΩΝ=1ch, Β΄ ΧΡΟΝΙΚΩΝ=2ch, ΕΣΔΡΑΣ=ezr, ΝΕΕΜΙΑΣ=neh, ΕΣΘΗΡ=est,
ΙΩΒ=job, ΨΑΛΜΟΙ=psa, ΠΑΡΟΙΜΙΑΙ=pro, ΕΚΚΛΗΣΙΑΣΤΗΣ=ecc, ΑΣΜΑ=sng,
ΗΣΑΪΑΣ=isa, ΙΕΡΕΜΙΑΣ=jer, ΘΡΗΝΟΙ=lam, ΙΕΖΕΚΙΗΛ=ezk, ΔΑΝΙΗΛ=dan,
ΩΣΗΕ=hos, ΙΩΗΛ=jol, ΑΜΩΣ=amo, ΟΒΔΙΟΥ=oba, ΙΩΝΑΣ=jon, ΜΙΧΑΙΑΣ=mic,
ΝΑΟΥΜ=nam, ΑΒΒΑΚΟΥΜ=hab, ΣΟΦΟΝΙΑΣ=zep, ΑΓΓΑΙΟΣ=hag, ΖΑΧΑΡΙΑΣ=zec, ΜΑΛΑΧΙΑΣ=mal,
ΜΑΤΘΑΙΟΣ=mat, ΜΑΡΚΟΣ=mrk, ΛΟΥΚΑΣ=luk, ΙΩΑΝΝΗΣ=jhn,
ΠΡΑΞΕΙΣ=act, ΡΩΜΑΙΟΥΣ=rom, Α΄ ΚΟΡΙΝΘΙΟΥΣ=1co, Β΄ ΚΟΡΙΝΘΙΟΥΣ=2co,
ΓΑΛΑΤΑΣ=gal, ΕΦΕΣΙΟΥΣ=eph, ΦΙΛΙΠΠΗΣΙΟΥΣ=php, ΚΟΛΟΣΣΑΕΙΣ=col,
Α΄ ΘΕΣΣΑΛΟΝΙΚΕΙΣ=1th, Β΄ ΘΕΣΣΑΛΟΝΙΚΕΙΣ=2th,
Α΄ ΤΙΜΟΘΕΟΝ=1ti, Β΄ ΤΙΜΟΘΕΟΝ=2ti, ΤΙΤΟΝ=tit, ΦΙΛΗΜΟΝΑ=phm,
ΕΒΡΑΙΟΥΣ=heb, ΙΑΚΩΒΟΥ=jas, Α΄ ΠΕΤΡΟΥ=1pe, Β΄ ΠΕΤΡΟΥ=2pe,
Α΄ ΙΩΑΝΝΟΥ=1jn, Β΄ ΙΩΑΝΝΟΥ=2jn, Γ΄ ΙΩΑΝΝΟΥ=3jn, ΙΟΥΔΑ=jud,
ΑΠΟΚΑΛΥΨΗ=rev

Προσοχή το βιβλίο ΦΙΛΙΠΠΗΣΙΟΥΣ έχει συντομογραφία php και όχι phil.
Προσοχή το βιβλίο ΦΙΛΗΜΟΝΑ έχει συντομογραφία phm.

ΚΑΝΟΝΑΣ ΠΑΡΑΠΟΜΠΩΝ (ΥΠΟΧΡΕΩΤΙΚΟ):
• Όλες οι βιβλικές παραπομπές σε links ΠΡΕΠΕΙ να χρησιμοποιούν τις συντομογραφίες της λίστας (π.χ. Ψαλμοί = psa).
• Ποτέ μη χρησιμοποιείς ελληνικές λέξεις ή ελεύθερες μεταγραφές μέσα στο URL (π.χ. όχι /ψαλμ.23.1). ΜΟΝΟ /psa.23.1.
• Αν ο χρήστης γράψει ελληνική ονομασία (π.χ. «Ψαλμ.»), ΜΕΤΑΤΡΕΨΕ την στη σωστή συντομογραφία πριν σχηματίσεις το link.
• Αν δεν μπορείς να αντιστοιχίσεις, μην δώσεις link· ζήτα διευκρίνιση.
• Το βιβλίο ΦΙΛΙΠΠΗΣΙΟΥΣ έχει συντόμευση php.
• Το βιβλίο ΦΙΛΗΜΟΝΑ έχει συντόμευση phm.

Όλες οι απαντήσεις να επιστρέφονται σε μορφή HTML. 
Οι σύνδεσμοι της Αγίας Γραφής να χρησιμοποιούν <a href="..."> ... </a>
και να έχουν πάντα το attribute target="_blank".

Αν ο χρήστης ζητήσει μόνο βιβλίο (χωρίς κεφάλαιο/εδάφιο), να θεωρείς προεπιλογή το κεφάλαιο 1 και 
να κατασκευάζεις τους σχετικούς συνδέσμους στο κεφάλαιο 1.
`;

function json(data, status = 200, extra = {}) {
  return new Response(JSON.stringify(data), {
    status,
    headers: { "Content-Type": "application/json", ...CORS, ...extra },
  });
}

export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);

    // CORS preflight
    if (request.method === "OPTIONS") {
      return new Response(null, { status: 204, headers: CORS });
    }

    // Quick health check
    if (url.pathname === "/api/ping") {
      const hasKey = !!env.OPENAI_API_KEY && env.OPENAI_API_KEY.length > 10;
      return json({ ok: true, env_openai_key_present: hasKey });
    }

    if (url.pathname === "/api/chat" && request.method === "POST") {
      if (!env.OPENAI_API_KEY) {
        return json({ error: "OPENAI_API_KEY is missing in Worker env." }, 500);
      }

      let payload;
      try {
        payload = await request.json();
      } catch {
        return json({ error: "Invalid JSON body" }, 400);
      }

      const { message, system } = payload || {};
      if (!message || typeof message !== "string") {
        return json({ error: "Missing 'message' string" }, 400);
      }

      try {
        const upstream = await fetch("https://api.openai.com/v1/chat/completions", {
          method: "POST",
          headers: {
            "Authorization": `Bearer ${env.OPENAI_API_KEY}`,
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            model: "gpt-4o-mini",
            messages: [
              { role: "system", content: systemPrompt },
              { role: "user", content: message }
            ],
            temperature: 0.3
          }),
        });

        let text = await upstream.text();
        text = text.replace(/phil/g, "php");

        // Try to parse the body as JSON
        let data;
        try { data = JSON.parse(text); } catch { data = null; }

        // If upstream not ok, pass through status & body for debug
        if (!upstream.ok) {
          return json(
            { error: "Upstream error", status: upstream.status, body: data || text },
            upstream.status
          );
        }

        const reply = data?.choices?.[0]?.message?.content ?? "";
        const ts = new Date().toISOString();
        const ip = request.headers.get("cf-connecting-ip") || null;
        const ua = request.headers.get("user-agent") || null;
        ctx.waitUntil(fetch(env.WEBHOOK_URL,{method:"POST",headers:{"content-type":"application/json"},body:JSON.stringify({message, reply, ts, ip, ua})}).catch(()=>{}));
        return json({ reply }, 200);
      } catch (e) {
        return json({ error: "Worker exception", detail: String(e) }, 500);
      }
    }

    return new Response("OK", { status: 200, headers: CORS });
  }
};
```


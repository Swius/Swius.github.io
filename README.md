<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Science Study</title>
<style>
:root{--bg:#eef2f4;--grid:#d3dce2;--ink:#14243a;--muted:#566779;--card:#fffdf7;--accent:#1f6f8b;--good:#2f7d4f;--bad:#b4402f;color-scheme:light}
@media (prefers-color-scheme:dark){:root{--bg:#0f1a24;--grid:#1a2836;--ink:#e6edf3;--muted:#93a4b5;--card:#16232f;--accent:#6cc0d9;--good:#6fcf97;--bad:#f08a7a;color-scheme:dark}}
*{box-sizing:border-box}
body{margin:0;min-height:100vh;color:var(--ink);font:16px/1.5 system-ui,-apple-system,"Segoe UI",sans-serif;
background-color:var(--bg);background-image:linear-gradient(var(--grid) 1px,transparent 1px),linear-gradient(90deg,var(--grid) 1px,transparent 1px);background-size:28px 28px}
.wrap{max-width:640px;margin:0 auto;padding:24px 16px 64px}
h1{font:700 1.9rem/1.2 Georgia,"Iowan Old Style",serif;margin:0 0 4px}
.sub{color:var(--muted);margin:0 0 20px}
.tabs{display:flex;gap:8px;margin-bottom:16px}
button{font:inherit;cursor:pointer;border:2px solid var(--ink);background:var(--card);color:var(--ink);padding:10px 16px;border-radius:6px}
button:hover{border-color:var(--accent)}
button:focus-visible,textarea:focus-visible,summary:focus-visible{outline:3px solid var(--accent);outline-offset:2px}
select{font:inherit;border:2px solid var(--ink);background:var(--card);color:var(--ink);padding:10px;border-radius:6px;max-width:100%}
.tab[aria-pressed="true"]{background:var(--ink);color:var(--bg)}
.prog{height:8px;border:2px solid var(--ink);border-radius:4px;background:var(--card);overflow:hidden;margin-bottom:6px}
.prog i{display:block;height:100%;background:var(--accent);width:0}
.meta{color:var(--muted);font-size:.9rem;margin-bottom:14px}
.card{display:block;width:100%;text-align:left;min-height:220px;padding:28px;border-width:2px;border-radius:8px;box-shadow:6px 6px 0 var(--ink);font:1.35rem/1.45 Georgia,"Iowan Old Style",serif}
.card small{display:block;font:.85rem system-ui,sans-serif;color:var(--muted);margin-bottom:10px}
.box{background:var(--card);border:2px solid var(--ink);border-radius:8px;padding:24px;box-shadow:6px 6px 0 var(--ink)}
.q{font:1.3rem/1.45 Georgia,"Iowan Old Style",serif;margin:0 0 14px}
textarea{width:100%;min-height:90px;font:inherit;padding:10px;border:2px solid var(--ink);border-radius:6px;background:var(--bg);color:var(--ink)}
.row{display:flex;flex-wrap:wrap;gap:10px;margin-top:16px}
.good{border-color:var(--good);color:var(--good)}
.bad{border-color:var(--bad);color:var(--bad)}
.primary{background:var(--accent);border-color:var(--accent);color:#fff}
.ans{margin-top:16px;padding-top:14px;border-top:2px dashed var(--muted)}
.ans b{display:block;font-size:.85rem;color:var(--muted);margin-top:8px}
.miss{margin:14px 0;padding:14px;border-left:5px solid var(--bad);background:var(--bg);border-radius:0 6px 6px 0}
details{margin-top:28px}
summary{cursor:pointer;font-weight:600}
details textarea{min-height:140px;margin-top:10px}
.hint{color:var(--muted);font-size:.9rem}
@media (prefers-reduced-motion:no-preference){.prog i{transition:width .3s}}
</style>
</head>
<body>
<div class="wrap">
  <h1>Science Study</h1>
  <p class="sub">Cells, respiration, osmosis, magnification and microscopy. Flip cards to learn, then quiz yourself and see what you missed.</p>
  <div class="tabs">
    <button class="tab" data-m="cards" onclick="start('cards')">Flashcards</button>
    <button class="tab" data-m="quiz" onclick="start('quiz')">Quiz</button>
    <select id="topic" onchange="start(mode)" aria-label="Topic"></select>
  </div>
  <div id="main"></div>

  <details>
    <summary>Add or replace your cards</summary>
    <p class="hint">One card per line: question | answer | why | topic (why and topic are optional). New cards are added to your deck and saved in this browser.</p>
    <textarea id="import" placeholder="What is the powerhouse of the cell? | Mitochondria | They make most of the cell’s ATP through respiration"></textarea>
    <div class="row">
      <button onclick="addCards()">Add cards</button>
      <button onclick="resetCards()">Reset to sample cards</button>
    </div>
  </details>
</div>

<script>
// Edit this list to change the built-in cards. Each card: q (question), a (answer), why (explanation).
const r = (n, d = 6) => +n.toFixed(d);
const K = (t, q, a, why = '') => ({t, q, a, why});
const M = 'Magnification';
const mag = (real, img) => K(M, `Real object ${real} mm, image ${img} mm. What is the total magnification?`, `×${r(img / real, 3)}`, `Magnification = image ÷ real object = ${img} ÷ ${real}.`);
const real = (m, img) => K(M, `Total magnification ×${m}, image ${img} mm. What is the size of the real object in mm?`, `${r(img / m)} mm`, `Real size = image ÷ magnification = ${img} ÷ ${m}.`);
const um = (m, img) => K(M, `Total magnification ×${m}, image ${img} mm. What is the size of the real object in µm?`, `${r(img / m * 1000, 4)} µm`, `${img} ÷ ${m} = ${r(img / m)} mm. Multiply by 1000 to convert mm to µm.`);

const SAMPLE = [
  // Respiration
  K('Respiration', "What is the purpose of respiration?", "To release (transfer) energy for the cell, making ATP", "Respiration takes place in all living cells."),
  K('Respiration', "Which organelle is where aerobic respiration happens?", "The mitochondria", "Muscle cells have many mitochondria because they need a lot of energy."),
  K('Respiration', "What is respiration without oxygen called?", "Anaerobic respiration"),
  K('Respiration', "Describe anaerobic respiration in humans and give an effect.", "Glucose is broken down to lactic acid, releasing energy. Effects: oxygen debt, breathing hard, pain.", "It is a quick source of energy but less efficient than aerobic respiration."),
  K('Respiration', "Complete the equation for anaerobic respiration in animal cells.", "glucose → lactic acid", "Write the chemicals in this order. Formulae are not needed."),
  K('Respiration', "What is anaerobic respiration in yeast called?", "Fermentation", "glucose → ethanol + carbon dioxide"),
  K('Respiration', "How does the fermentation equation show it is anaerobic?", "No oxygen is reacting with the glucose", "Oxygen does not appear on the left-hand side."),
  K('Respiration', "Why is fermentation used to make beer and wine?", "It produces ethanol (alcohol)"),
  K('Respiration', "Why is fermentation used when making bread?", "It produces carbon dioxide, which makes the dough rise", "The gas makes the bread lighter."),
  // Osmosis
  K('Osmosis', "How does water move into and out of cells?", "By osmosis", "Not breathing or respiration. Breathing moves air, and respiration releases energy."),
  K('Osmosis', "Water moves through what kind of membrane in osmosis?", "A partially permeable membrane", "It lets water through but not all substances."),
  K('Osmosis', "A plant cell is in a strong sugar solution (B) and another is in a weaker one (A). Give two ways B differs from A.", "Any two: smaller vacuole, shrunken cytoplasm, gap between cytoplasm and cell wall, membrane pulled away from the wall, cell wall curves inwards", "Water has left cell B by osmosis, so it is flaccid. Cell A is turgid."),
  K('Osmosis', "What happens to red blood cells put into water?", "Water moves in by osmosis, so the cells swell and may burst", "Animal cells have no cell wall to hold their shape."),
  K('Osmosis', "Which cell will water move into fastest?", "The cell with the biggest difference in concentration (the most concentrated sugar solution)", "In the exam, compare the percentages for each cell on the diagram. The exam answer was R."),
  // Magnification extras
  K(M, "Should a magnification answer include units?", "No. Write it as ×number, for example ×400", "Giving units loses the mark."),
  K(M, "Which mistake loses marks in magnification questions that mix mm and µm?", "Not converting units first", "1 mm = 1000 µm. Change both sizes to the same unit before dividing."),
  K(M, "The image of a palisade cell is 28 mm at ×400. What is its real length in mm?", "0.07 mm", "28 ÷ 400 = 0.07."),
  K(M, "Convert 0.07 mm to µm.", "70 µm", "0.07 × 1000 = 70."),
  K(M, "A muscle cell is 0.05 mm wide. Its image is 2 mm wide. What is the magnification?", "×40", "2 ÷ 0.05 = 40."),
  K(M, "A cell’s real length is 280 µm. Its drawing is 112 mm long. What is the magnification?", "×400", "Convert first: 112 mm = 112 000 µm (or 280 µm = 0.28 mm). Then 112 000 ÷ 280 = 400."),
  // Cell structure
  K('Cell structure', "What is the function of a ribosome?", "Makes (synthesises) protein", "Both bacterial and plant cells have ribosomes."),
  K('Cell structure', "Give one way a plant cell differs from a bacterial cell (both have ribosomes).", "Any one: it has a nucleus, a vacuole or chloroplasts, or it is much larger", "Bacterial cells have no mitochondria, nucleus or chloroplasts."),
  K('Cell structure', "What is the function of the cell membrane?", "Controls what enters and leaves the cell"),
  K('Cell structure', "What is the function of the nucleus?", "Contains the genetic information"),
  K('Cell structure', "What is the function of the chloroplast?", "Where photosynthesis happens"),
  K('Cell structure', "What does the cell wall do in plant cells?", "Supports and strengthens the cell", "In plants it is made of cellulose. Bacterial cells also have a cell wall."),
  K('Cell structure', "Which cell type has a nucleus: prokaryotic or eukaryotic?", "Eukaryotic cells"),
  K('Cell structure', "Which cells have a permanent vacuole?", "Plant cells only"),
  K('Cell structure', "Which cell type contains plasmids?", "Prokaryotic cells (such as bacteria)"),
  K('Cell structure', "Give one function of muscle cells.", "To cause movement by contracting (shortening)"),
  K('Cell structure', "How are muscle cells adapted for their function?", "They have many mitochondria, to release a lot of energy for respiration", "Don’t say mitochondria “make” or “create” energy. Say they transfer or release it."),
  // Microscopy
  K('Microscopy', "How do you work out total magnification?", "Eyepiece lens magnification × objective lens magnification"),
  K('Microscopy', "A microscope has a ×5 eyepiece. Which objective lens gives ×50?", "The ×10 objective lens", "5 × 10 = 50."),
  K('Microscopy', "Describe how to observe a prepared slide at ×50 with a ×5 eyepiece.", "Put the slide on the stage. Start with the lowest power (×4) objective. Switch the light on (or adjust the mirror). Move the stage close to the lens, without the slide touching it. Turn the focusing knob to move the slide away from the lens until the image is in focus. Use fine focus for a clear image. Change to the ×10 objective and refocus.", "Always start on the lowest power. The slide must not touch the lens."),
  K('Microscopy', "The cells on a slide are blurry. How should you adjust the microscope?", "Turn the (fine) focusing knob until the cells are in focus", "Increasing magnification does not fix a blurry image."),
  K('Microscopy', "The cells on a slide are too small to see clearly. What should you do?", "Rotate the nosepiece to change to a higher power objective lens", "This increases the magnification. You need to say “higher power” or “increase magnification”."),
  // Magnification practice from worksheet
  K(M, "What is the magnification formula?", "Magnification = size of image ÷ size of real object", "Use the same units for both sizes before you divide."),
  K(M, "How do you rearrange the formula to find the size of the real object?", "Real size = image size ÷ magnification", "Multiply both sides by the real size, then divide both sides by the magnification."),
  K(M, "How do you rearrange the formula to find the size of the image?", "Image size = magnification × real size", "Multiply both sides of the magnification formula by the real size."),
  K(M, "How many µm are in 1 mm?", "1000 µm", "A micrometre is one thousandth of a millimetre."),
  K(M, "How do you convert mm to µm?", "Multiply by 1000", "There are 1000 µm in every mm, so the number gets bigger."),
  K(M, "How do you convert µm to mm?", "Divide by 1000", "There are 1000 µm in every mm, so the number gets smaller."),
  K(M, "The real object is 0.5 mm and the image is 5 mm. What is the magnification?", "×10", "5 mm ÷ 0.5 mm = 10."),
  K(M, "The image is 5 mm at ×1000. What is the real size?", "0.005 mm", "1000 × X = 5, so X = 5 ÷ 1000."),
  K(M, "The image is 10 mm at ×100. What is the real size in µm?", "100 µm", "10 mm ÷ 100 = 0.1 mm, and 0.1 mm × 1000 = 100 µm."),
  mag(0.5, 50), mag(0.5, 2), mag(0.5, 200),
  mag(0.1, 5), mag(0.1, 3), mag(0.1, 2),
  mag(0.3, 6), mag(0.01, 1), mag(0.05, 15), mag(0.02, 20),
  real(400, 2), real(400, 4), real(400, 2.5),
  real(40, 5), real(40, 3), real(40, 0.5),
  real(400, 6), real(10, 8), real(1000, 0.4), real(400, 0.2),
  um(1000, 3), um(1000, 0.4), um(1000, 0.5),
  um(400, 0.3), um(400, 0.7), um(400, 0.6),
  um(100, 0.8), um(1000, 2.8), um(40, 8)
];

let cards = load() || SAMPLE.slice();
let mode = 'cards', deck = [], pos = 0, flipped = false, known = 0, missed = [], revealed = false, total = 0;

const $ = s => document.querySelector(s);
const esc = s => String(s).replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
const shuffle = a => { a = a.slice(); for (let i = a.length - 1; i > 0; i--) { const j = Math.floor(Math.random() * (i + 1)); [a[i], a[j]] = [a[j], a[i]]; } return a; };

function load() { try { const v = JSON.parse(localStorage.getItem('cards')); return Array.isArray(v) && v.length ? v : null; } catch (e) { return null; } }
function save() { try { localStorage.setItem('cards', JSON.stringify(cards)); } catch (e) {} }

const topicOf = c => c.t || 'Other';
function fillTopics() {
  const s = $('#topic'), cur = s.value || 'All';
  const ts = ['All', ...new Set(cards.map(topicOf))];
  s.innerHTML = ts.map(t => `<option value="${esc(t)}">${t === 'All' ? 'All topics' : esc(t)}</option>`).join('');
  s.value = ts.includes(cur) ? cur : 'All';
}

function start(m, src) {
  if (!src) fillTopics();
  const t = $('#topic').value;
  mode = m; deck = shuffle(src || cards.filter(c => t === 'All' || topicOf(c) === t)); total = deck.length;
  pos = 0; flipped = false; revealed = false; known = 0; missed = [];
  render();
}

function render() {
  document.querySelectorAll('.tab').forEach(t => t.setAttribute('aria-pressed', t.dataset.m === mode));
  const m = $('#main');
  if (!deck.length) { m.innerHTML = '<p>No cards yet. Add some below.</p>'; return; }
  if (pos >= deck.length) return done(m);
  const c = deck[pos];
  const pct = Math.round((pos / Math.max(total, 1)) * 100);
  let h = `<div class="prog" role="progressbar" aria-valuenow="${pct}" aria-valuemin="0" aria-valuemax="100"><i style="width:${pct}%"></i></div>`;
  if (mode === 'cards') {
    h += `<div class="meta">Known: ${known} of ${total}. Cards left in this round: ${deck.length - pos}</div>`;
    h += `<button class="card" onclick="flip()" aria-label="Flip card"><small>${esc(topicOf(c))}: ${flipped ? 'answer' : 'question'}. Tap to flip.</small>${esc(flipped ? c.a : c.q)}${flipped && c.why ? `<small style="margin-top:14px">${esc(c.why)}</small>` : ''}</button>`;
    h += flipped
      ? `<div class="row"><button class="good" onclick="mark(true)">Got it</button><button class="bad" onclick="mark(false)">Still learning</button></div>`
      : `<div class="row"><button onclick="reshuffle()">Shuffle</button></div>`;
  } else {
    h += `<div class="meta">Question ${pos + 1} of ${total}</div><div class="box"><p class="q">${esc(c.q)}</p>`;
    if (!revealed) {
      h += `<textarea id="resp" aria-label="Your answer" placeholder="Type your answer"></textarea><div class="row"><button class="primary" onclick="reveal()">Check answer</button></div>`;
    } else {
      h += `<div><b class="hint">Your answer</b>${esc(window._last || '') || '(blank)'}</div>`;
      h += `<div class="ans"><b>Correct answer</b>${esc(c.a)}${c.why ? `<b>Why</b>${esc(c.why)}` : ''}</div>`;
      h += `<div class="row"><button class="good" onclick="grade(true)">I got it right</button><button class="bad" onclick="grade(false)">I got it wrong</button></div>`;
    }
    h += `</div>`;
  }
  m.innerHTML = h;
  const r = $('#resp'); if (r) r.focus();
}

function flip() { flipped = !flipped; render(); }
function reshuffle() { deck = shuffle(deck); pos = 0; render(); }
function mark(ok) { const c = deck[pos]; if (ok) known++; else deck.push(c); pos++; flipped = false; render(); }
function reveal() { window._last = $('#resp').value; revealed = true; render(); }
function grade(ok) { if (!ok) missed.push(deck[pos]); pos++; revealed = false; render(); }

function done(m) {
  if (mode === 'cards') {
    m.innerHTML = `<div class="box"><p class="q">All ${total} cards known.</p><div class="row"><button class="primary" onclick="start('cards')">Study again</button><button onclick="start('quiz')">Take the quiz</button></div></div>`;
    return;
  }
  const right = total - missed.length;
  let h = `<div class="box"><p class="q">You scored ${right} of ${total}.</p>`;
  if (missed.length) {
    h += `<p class="hint">Here’s what to review:</p>`;
    missed.forEach(c => { h += `<div class="miss"><strong>${esc(c.q)}</strong><div class="ans" style="margin-top:8px;padding-top:8px"><b>Answer</b>${esc(c.a)}${c.why ? `<b>Why</b>${esc(c.why)}` : ''}</div></div>`; });
  } else { h += `<p>No mistakes. Nice work.</p>`; }
  h += `<div class="row">${missed.length ? `<button class="primary" onclick="start('quiz', missed.slice())">Retry missed</button>` : ''}<button onclick="start('quiz')">New quiz</button><button onclick="start('cards')">Back to flashcards</button></div></div>`;
  m.innerHTML = h;
}

function addCards() {
  const lines = $('#import').value.split('\n').map(l => l.trim()).filter(Boolean);
  const added = lines.map(l => l.split('|').map(p => p.trim())).filter(p => p[0] && p[1]).map(p => ({q: p[0], a: p[1], why: p[2] || '', t: p[3] || ''}));
  if (!added.length) { alert('No cards found. Use: question | answer | why, one per line.'); return; }
  cards = cards.concat(added); save(); $('#import').value = ''; start(mode);
}
function resetCards() { cards = SAMPLE.slice(); try { localStorage.removeItem('cards'); } catch (e) {} start(mode); }

start('cards');
</script>
</body>
</html>

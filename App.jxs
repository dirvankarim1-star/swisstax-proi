import { useState, useRef, useCallback, useEffect, useMemo } from "react";

// ─── STATIC DATA ──────────────────────────────────────────────────────────────
const CANTONS = {
  ZH:{n:"Zürich",      cap:"Zürich",      l:"de", p:13.0, c:8.5},
  BE:{n:"Bern",        cap:"Bern",        l:"de", p:13.6, c:8.8},
  LU:{n:"Luzern",      cap:"Luzern",      l:"de", p:11.5, c:7.2},
  UR:{n:"Uri",         cap:"Altdorf",     l:"de", p:9.2,  c:6.0},
  SZ:{n:"Schwyz",      cap:"Schwyz",      l:"de", p:9.8,  c:6.2},
  OW:{n:"Obwalden",    cap:"Sarnen",      l:"de", p:8.7,  c:5.8},
  NW:{n:"Nidwalden",   cap:"Stans",       l:"de", p:9.0,  c:6.0},
  GL:{n:"Glarus",      cap:"Glarus",      l:"de", p:9.6,  c:6.4},
  ZG:{n:"Zug",         cap:"Zug",         l:"de", p:7.0,  c:4.5},
  FR:{n:"Fribourg",    cap:"Fribourg",    l:"fr", p:13.4, c:8.4},
  SO:{n:"Solothurn",   cap:"Solothurn",   l:"de", p:13.0, c:8.2},
  BS:{n:"Basel-Stadt", cap:"Basel",       l:"de", p:13.0, c:8.0},
  BL:{n:"Basel-Land",  cap:"Liestal",     l:"de", p:12.3, c:7.8},
  SH:{n:"Schaffhausen",cap:"Schaffhausen",l:"de", p:11.2, c:7.0},
  AR:{n:"Appenzell AR",cap:"Herisau",     l:"de", p:11.5, c:7.2},
  AI:{n:"Appenzell AI",cap:"Appenzell",   l:"de", p:8.0,  c:5.5},
  SG:{n:"St. Gallen",  cap:"St. Gallen",  l:"de", p:13.0, c:8.2},
  GR:{n:"Graubünden",  cap:"Chur",        l:"de", p:11.0, c:7.0},
  AG:{n:"Aargau",      cap:"Aarau",       l:"de", p:11.0, c:7.0},
  TG:{n:"Thurgau",     cap:"Frauenfeld",  l:"de", p:11.5, c:7.2},
  TI:{n:"Ticino",      cap:"Bellinzona",  l:"it", p:12.0, c:7.5},
  VD:{n:"Vaud",        cap:"Lausanne",    l:"fr", p:15.0, c:9.5},
  VS:{n:"Valais",      cap:"Sion",        l:"fr", p:12.0, c:7.8},
  NE:{n:"Neuchâtel",   cap:"Neuchâtel",   l:"fr", p:14.0, c:8.8},
  GE:{n:"Genève",      cap:"Genève",      l:"fr", p:14.5, c:9.0},
  JU:{n:"Jura",        cap:"Delémont",    l:"fr", p:14.0, c:8.8},
};

const MWST_RATES = [
  { v:"8.1",  label:"Normalsatz 8.1%" },
  { v:"2.6",  label:"Sondersatz 2.6% (Beherbergung)" },
  { v:"3.8",  label:"Sondersatz 3.8% (Landwirtschaft)" },
  { v:"0",    label:"Ausgenommen 0%" },
];

const KATEGORIEN = [
  "Warenaufwand","Personalaufwand","Mietaufwand","Fahrzeugkosten",
  "IT & Software","Marketing","Versicherungen","Beratung & Treuhand",
  "Reise & Spesen","Büromaterial","Abschreibungen","Zinsaufwand",
  "Bankgebühren","Sonstiger Betriebsaufwand",
];

const KONTENPLAN = [
  ["1000–1999","Aktiven"],["2000–2999","Fremdkapital"],
  ["3000–3999","Betriebsertrag"],["4000–4999","Materialaufwand"],
  ["5000–5999","Personalaufwand"],["6000–6999","Betriebsaufwand"],
  ["7000–7999","Finanzaufwand/-ertrag"],["9000–9999","Eigenkapital"],
];

const DL_PRIVATE = [
  {d:"2025-03-31", t:"Steuererklärung (ZH/BE/SG/AG)", info:"Ordentliche Einreichungsfrist"},
  {d:"2025-04-30", t:"Verrechnungssteuer Rückforderung", info:"Formular 85 an ESTV"},
  {d:"2025-06-30", t:"Fristverlängerung beantragen", info:"Online über kantonales Portal"},
  {d:"2025-09-30", t:"AHV-Beiträge Selbständige Q3", info:"Akontozahlung Ausgleichskasse"},
  {d:"2025-12-31", t:"Säule 3a Einzahlung 2025", info:"Max. CHF 7'258 (Angestellte)"},
  {d:"2025-12-31", t:"Pensionskassen-Einkauf", info:"Steuerlich abzugsfähig im Steuerjahr"},
  {d:"2026-03-31", t:"Steuererklärung 2025", info:"Nächste ordentliche Frist"},
];

const DL_FIRMA = [
  {d:"2025-01-31", t:"MWST Q4/2024 abrechnen", info:"60 Tage nach Quartalsende"},
  {d:"2025-03-31", t:"AHV-Jahreslohnmeldung 2024", info:"An Ausgleichskasse einreichen"},
  {d:"2025-04-30", t:"MWST Q1/2025 abrechnen", info:"Frist: 60 Tage nach Quartalsende"},
  {d:"2025-06-30", t:"Steuererklärung AG/GmbH 2024", info:"Ordentliche Frist (kantonal)"},
  {d:"2025-07-31", t:"MWST Q2/2025 abrechnen", info:"Frist: 60 Tage nach Quartalsende"},
  {d:"2025-10-31", t:"MWST Q3/2025 abrechnen", info:"Frist: 60 Tage nach Quartalsende"},
  {d:"2025-12-31", t:"Jahresabschluss vorbereiten", info:"Buchungsschluss & Inventur"},
  {d:"2026-01-31", t:"MWST Q4/2025 abrechnen", info:"Frist: 60 Tage nach Quartalsende"},
];

const ALLOWED = [
  "application/pdf","image/png","image/jpeg",
  "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
];

// ─── PURE HELPERS ─────────────────────────────────────────────────────────────
function chf(n) {
  const num = typeof n === "string" ? parseFloat(n) : (n || 0);
  return "CHF " + num.toLocaleString("de-CH", { minimumFractionDigits:2, maximumFractionDigits:2 });
}

function daysLeft(dateStr) {
  return Math.ceil((new Date(dateStr) - new Date()) / 86400000);
}

function safeJSON(raw) {
  try { return JSON.parse(raw.replace(/```json\n?|```/g, "").trim()); }
  catch { return null; }
}

function lohnCalc(bruttoMonat) {
  const b = parseFloat(bruttoMonat) || 0;
  const alvBasis = Math.min(b * 12, 148200) / 12;
  const ahvAN = b * 0.053;   // 10.6% / 2
  const ivAN  = b * 0.007;   // 1.4%  / 2
  const eoAN  = b * 0.0025;  // 0.5%  / 2
  const alvAN = alvBasis * 0.011; // 2.2% / 2
  const total = ahvAN + ivAN + eoAN + alvAN;
  return {
    brutto: b, ahvAN, ivAN, eoAN, alvAN,
    netto: b - total,
    agZusatz: total,           // AG pays same amount on top
    gesamtAG: b + total,       // total cost for employer
  };
}

// ─── API CALL ─────────────────────────────────────────────────────────────────
async function api(system, messages, maxTokens = 1000) {
  // messages must be Array<{role:"user"|"assistant", content: string|Array}>
  const res = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model: "claude-sonnet-4-20250514",
      max_tokens: maxTokens,
      system,
      messages,
    }),
  });
  if (!res.ok) {
    const err = await res.json().catch(() => ({}));
    throw new Error(err?.error?.message || `HTTP ${res.status}`);
  }
  const data = await res.json();
  if (data.error) throw new Error(data.error.message);
  return data.content?.[0]?.text || "";
}

// Convenience: single user message with optional file content
function singleMsg(text, fileType, fileData) {
  if (!fileData) return [{ role:"user", content: text }];
  const mediaBlock = fileType === "application/pdf"
    ? { type:"document", source:{ type:"base64", media_type:"application/pdf", data: fileData } }
    : { type:"image",    source:{ type:"base64", media_type: fileType,          data: fileData } };
  return [{ role:"user", content: [mediaBlock, { type:"text", text }] }];
}

// ─── CSS ──────────────────────────────────────────────────────────────────────
const BASE_CSS = `
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;600&family=DM+Serif+Display&display=swap');
  *{box-sizing:border-box;margin:0;padding:0;}
  ::-webkit-scrollbar{width:3px}
  ::-webkit-scrollbar-track{background:#060c16}
  ::-webkit-scrollbar-thumb{background:#44607044;border-radius:2px}
  @keyframes fu{from{opacity:0;transform:translateY(10px)}to{opacity:1;transform:translateY(0)}}
  @keyframes pu{0%,100%{opacity:1}50%{opacity:0.3}}
  @keyframes sp{to{transform:rotate(360deg)}}
  @keyframes gp{0%,100%{box-shadow:0 0 10px #5ee7c811}50%{box-shadow:0 0 28px #5ee7c844}}
  @keyframes gg{0%,100%{box-shadow:0 0 10px #f0b43011}50%{box-shadow:0 0 28px #f0b43044}}
  .fu{animation:fu 0.3s ease both}
  .tab{padding:0.5rem 0.9rem;border:none;background:transparent;color:#2a4050;font-family:'Space Mono',monospace;font-size:0.6rem;cursor:pointer;letter-spacing:0.06em;transition:color 0.2s,border-color 0.2s;border-bottom:2px solid transparent;white-space:nowrap}
  .tab:hover:not(.active){color:#5a7080}
  .card{background:#080f1a;border:1px solid #0e1e2e;padding:1.3rem;border-radius:3px}
  .inp{background:#060c16;border:1px solid #0e1e2e;color:#c8dde8;padding:0.6rem 0.85rem;font-family:'DM Sans',sans-serif;font-size:0.87rem;width:100%;outline:none;border-radius:2px;transition:border-color 0.2s}
  .inp:focus{border-color:#44607088}
  .inp:disabled{opacity:0.4;cursor:not-allowed}
  .sel{background:#060c16;border:1px solid #0e1e2e;color:#c8dde8;padding:0.6rem 0.85rem;font-family:'DM Sans',sans-serif;font-size:0.87rem;outline:none;border-radius:2px;cursor:pointer;width:100%}
  .lbl{font-family:'Space Mono',monospace;font-size:0.57rem;color:#2a4050;letter-spacing:0.12em;display:block;margin-bottom:0.3rem}
  .badge{display:inline-block;padding:0.1rem 0.42rem;font-family:'Space Mono',monospace;font-size:0.56rem;border-radius:2px}
  .bh{background:#ff707022;color:#ff7070;border:1px solid #ff707033}
  .bm{background:#ffd93d22;color:#ffd93d;border:1px solid #ffd93d33}
  .bl{background:#6bcb7722;color:#6bcb77;border:1px solid #6bcb7733}
  .g2{display:grid;grid-template-columns:1fr 1fr;gap:0.85rem}
  .g4{display:grid;grid-template-columns:repeat(4,1fr);gap:0.7rem}
  .stat{background:#060c16;border:1px solid #0e1e2e;padding:1rem;text-align:center;border-radius:3px}
  .errbx{background:#ff707011;border:1px solid #ff707033;color:#ff7070;padding:0.7rem 0.9rem;font-family:'Space Mono',monospace;font-size:0.63rem;border-radius:2px;margin-bottom:1rem;display:flex;justify-content:space-between;align-items:center;gap:0.5rem}
  .infbx{border:1px solid;padding:0.65rem 0.85rem;font-family:'Space Mono',monospace;font-size:0.62rem;border-radius:2px;line-height:1.5}
  .trow{display:grid;gap:0.35rem}
  .tr{display:grid;grid-template-columns:0.65fr 1fr 1.1fr 0.85fr 0.75fr 0.6fr;gap:0.4rem;padding:0.42rem 0.65rem;border-radius:2px;font-size:0.77rem;align-items:center}
  .trh{background:#0a1525;font-family:'Space Mono',monospace;font-size:0.53rem;color:#2a4050;letter-spacing:0.06em}
  .chat-u{background:#0e1e2e;padding:0.7rem 1rem;border-radius:2px 12px 12px 2px;max-width:80%;font-size:0.84rem;line-height:1.55;word-break:break-word}
  .chat-a{background:#060d18;padding:0.7rem 1rem;border-radius:12px 2px 2px 12px;max-width:85%;font-size:0.82rem;line-height:1.65;color:#aabccc;white-space:pre-wrap;word-break:break-word}
  .hcell{padding:0.52rem 0.32rem;text-align:center;cursor:pointer;border-radius:2px;transition:transform 0.18s,border-color 0.18s;border:1px solid transparent}
  .hcell:hover{transform:scale(1.06)}
  .del{background:transparent;border:1px solid #ff707033;color:#ff7070;padding:0.22rem 0.48rem;font-size:0.57rem;cursor:pointer;border-radius:2px;font-family:'Space Mono',monospace;transition:background 0.18s;flex-shrink:0}
  .del:hover{background:#ff707011}
  .del:disabled{opacity:0.3;cursor:not-allowed}
  @media(max-width:640px){.g2,.g4{grid-template-columns:1fr}.tr{grid-template-columns:1fr 1fr}}
`;

function dynCSS(acc, acc2, adim) {
  return `
    .btn-p{background:linear-gradient(135deg,${acc},${acc2});color:#04080f;font-weight:700;border:none;padding:0.6rem 1.3rem;cursor:pointer;font-family:'Space Mono',monospace;font-size:0.68rem;letter-spacing:0.07em;transition:transform 0.18s,box-shadow 0.18s;border-radius:2px}
    .btn-p:hover:not(:disabled){transform:translateY(-2px);box-shadow:0 6px 20px ${adim}}
    .btn-p:disabled{opacity:0.35;cursor:not-allowed;transform:none}
    .btn-o{background:transparent;border:1px solid ${adim};color:${acc};padding:0.6rem 1.3rem;cursor:pointer;font-family:'Space Mono',monospace;font-size:0.68rem;letter-spacing:0.07em;transition:background 0.18s;border-radius:2px}
    .btn-o:hover:not(:disabled){background:${adim}}
    .btn-o:disabled{opacity:0.35;cursor:not-allowed}
    .btn-sm{padding:0.25rem 0.6rem;font-size:0.58rem}
    .tab.active{color:${acc};border-bottom-color:${acc}}
    .card-hi{background:#060d18;border:1px solid ${adim};padding:1.3rem;border-radius:3px}
    .acc{color:${acc}}
    .adim{background:${adim};border-color:${adim};color:${acc}}
  `;
}

// ─── MARKDOWN RENDERER ────────────────────────────────────────────────────────
function MD({ text, acc }) {
  if (!text) return null;
  return (
    <div>
      {text.split("\n").map((line, i) => {
        if (!line.trim()) return <br key={i} />;
        if (line.startsWith("### ")) return <h4 key={i} style={{color:"#c8dde8",fontFamily:"'Space Mono',monospace",fontSize:"0.7rem",margin:"0.7rem 0 0.25rem",letterSpacing:"0.08em"}}>◈ {line.slice(4)}</h4>;
        if (line.startsWith("## "))  return <h3 key={i} style={{color:acc,fontFamily:"'Space Mono',monospace",fontSize:"0.77rem",margin:"1.1rem 0 0.35rem",letterSpacing:"0.1em"}}>▸ {line.slice(3)}</h3>;
        if (line.startsWith("# "))   return <h2 key={i} style={{color:acc,fontFamily:"'DM Serif Display',serif",fontSize:"1.1rem",margin:"1.3rem 0 0.4rem"}}>{line.slice(2)}</h2>;
        if (line.match(/^[-•*]\s/))  return <p key={i} style={{margin:"0.18rem 0",paddingLeft:"1rem",color:"#aabccc",fontSize:"0.82rem",lineHeight:1.6}}>{line}</p>;
        if (line.match(/^\d+\.\s/))  return <p key={i} style={{margin:"0.18rem 0",paddingLeft:"1rem",color:"#aabccc",fontSize:"0.82rem",lineHeight:1.6}}>{line}</p>;
        if (line.includes("CHF"))    return <p key={i} style={{margin:"0.18rem 0",color:acc,fontSize:"0.83rem",lineHeight:1.5}}>{line}</p>;
        if (line.toUpperCase() === line && line.length > 3 && !line.includes(" ")) return null; // skip junk caps lines
        return <p key={i} style={{margin:"0.1rem 0",color:"#778e9e",fontSize:"0.8rem",lineHeight:1.6}}>{line}</p>;
      })}
    </div>
  );
}

// ─── FILE LIST ITEM ───────────────────────────────────────────────────────────
function FileItem({ f, acc, adim, onRemove }) {
  const icon = f.type.includes("pdf") ? "📄" : f.type.includes("image") ? "🖼" : "📝";
  const statusColor = f.status === "fertig" ? acc : f.status === "fehler" ? "#ff7070" : f.status === "läuft" ? "#ffd93d" : "#2a4050";
  return (
    <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"0.52rem 0.72rem",background:"#060c16",marginBottom:"0.3rem",borderRadius:"2px",border:`1px solid ${f.status==="fertig"?adim:f.status==="fehler"?"#ff707022":f.status==="läuft"?"#ffd93d22":"#0e1e2e"}`}}>
      <div style={{display:"flex",alignItems:"center",gap:"0.55rem",minWidth:0,flex:1}}>
        <span style={{flexShrink:0}}>{icon}</span>
        <div style={{minWidth:0}}>
          <div style={{fontSize:"0.81rem",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{f.name}</div>
          <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.54rem",color:"#1a3040",marginTop:"0.1rem"}}>
            {(f.size/1048576).toFixed(1)}MB
            {f.result?.canton && CANTONS[f.result.canton] && <span style={{color:acc,marginLeft:"0.4rem"}}>📍{CANTONS[f.result.canton].n}{f.result.commune?` · ${f.result.commune}`:""}</span>}
            {f.result?.lang && <span style={{color:"#ffd93d",marginLeft:"0.4rem"}}>🌐{f.result.lang.toUpperCase()}</span>}
          </div>
        </div>
      </div>
      <div style={{display:"flex",alignItems:"center",gap:"0.35rem",flexShrink:0}}>
        <span style={{color:statusColor,fontSize:"0.75rem"}}>
          {f.status==="fertig"?"✓":f.status==="fehler"?"✕":f.status==="läuft"?<span style={{animation:"sp 1s linear infinite",display:"inline-block"}}>◉</span>:""}
        </span>
        <button className="del" onClick={onRemove}>✕</button>
      </div>
    </div>
  );
}

// ─── PROGRESS BAR ─────────────────────────────────────────────────────────────
function ProgressBar({ pct, acc, acc2, msg }) {
  return (
    <div style={{marginTop:"0.7rem"}}>
      {msg && <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.6rem",color:acc,marginBottom:"0.35rem",animation:"pu 1s infinite"}}>{msg}</div>}
      <div style={{height:"2px",background:"#0e1e2e",borderRadius:1,overflow:"hidden"}}>
        <div style={{height:"100%",width:`${pct}%`,background:`linear-gradient(90deg,${acc},${acc2})`,transition:"width 0.45s ease"}}/>
      </div>
      <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:"#1a3040",marginTop:"0.25rem",textAlign:"right"}}>{pct}%</div>
    </div>
  );
}

// ─── DEADLINE LIST ────────────────────────────────────────────────────────────
function DeadlineList({ items, acc, adim }) {
  return (
    <div style={{display:"grid",gap:"0.45rem"}}>
      {items.map((d, i) => {
        const days = daysLeft(d.d);
        const past = days < 0;
        return (
          <div key={i} style={{background:"#060c16",border:`1px solid ${!past&&days<30?"#ff707033":!past&&days<90?adim:"#0e1e2e"}`,padding:"0.8rem 1rem",borderRadius:"2px",display:"flex",justifyContent:"space-between",alignItems:"center",gap:"0.5rem",opacity:past?0.3:1}}>
            <div>
              <div style={{fontSize:"0.86rem",fontWeight:500,marginBottom:"0.12rem"}}>{d.t}</div>
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.56rem",color:"#1a3040"}}>{d.d}{d.info?` · ${d.info}`:""}</div>
            </div>
            <div style={{textAlign:"right",flexShrink:0}}>
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"1rem",color:past?"#1a3040":days<30?"#ff7070":days<90?"#ffd93d":acc,fontWeight:700}}>{past?"✓":`${days}d`}</div>
              {!past&&<div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.5rem",color:"#1a3040"}}>VERBLEIBEND</div>}
            </div>
          </div>
        );
      })}
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════════════════
// MAIN APP
// ═══════════════════════════════════════════════════════════════════════════════
export default function App() {
  const [mode,      setMode]      = useState(null);
  const [tab,       setTab]       = useState(0);
  const [lang,      setLang]      = useState("de");
  const [detCanton, setDetCanton] = useState(null);
  const [detCommune,setDetCommune]= useState(null);
  const [err,       setErr]       = useState("");

  // Shared docs
  const [files,   setFiles]   = useState([]);
  const [results, setResults] = useState([]);
  const [global,  setGlobal]  = useState("");
  const [busy,    setBusy]    = useState(false);
  const [busyMsg, setBusyMsg] = useState("");
  const [pct,     setPct]     = useState(0);
  const [drag,    setDrag]    = useState(false);

  // Chat
  const [chatHist,  setChatHist]  = useState([]);
  const [chatIn,    setChatIn]    = useState("");
  const [chatBusy,  setChatBusy]  = useState(false);

  // Private
  const [calcIn,    setCalcIn]    = useState("");
  const [calcWl,    setCalcWl]    = useState("");
  const [calcRes,   setCalcRes]   = useState(null);
  const [taxF,      setTaxF]      = useState({name:"",status:"ledig",kids:"0",inc:"",wlth:"",ded:""});
  const [taxOut,    setTaxOut]    = useState("");
  const [taxBusy,   setTaxBusy]   = useState(false);
  const [heatSel,   setHeatSel]   = useState(null);

  // Firma
  const [fName,     setFName]     = useState("");
  const [fForm,     setFForm]     = useState("GmbH");
  const [fCanton,   setFCanton]   = useState("ZH");
  const [belege,    setBelege]    = useState([]);
  const [bBusy,     setBBusy]     = useState(false);
  const [bResult,   setBResult]   = useState([]);
  const [mRate,     setMRate]     = useState("8.1");
  const [mQuart,    setMQuart]    = useState("Q1 2025");
  const [mUmsatz,   setMUmsatz]   = useState("");
  const [mVor,      setMVor]      = useState("");
  const [mResult,   setMResult]   = useState(null);
  const [lohn,      setLohn]      = useState([{id:1,name:"",brutto:"",kt:"ZH"}]);
  const [lohnOut,   setLohnOut]   = useState("");
  const [lohnBusy,  setLohnBusy]  = useState(false);
  const [jahrOut,   setJahrOut]   = useState("");
  const [jahrBusy,  setJahrBusy]  = useState(false);

  const fileRef  = useRef();
  const bRef     = useRef();
  const chatEnd  = useRef();

  // Computed
  const bNetto = useMemo(() => bResult.reduce((a,b)=>a+(b.betragNetto||0),0), [bResult]);
  const bMWST  = useMemo(() => bResult.reduce((a,b)=>a+(b.mwstBetrag||0),0),  [bResult]);
  const highR  = useMemo(() => results.flatMap(r=>r.risiken||[]).filter(x=>x.schwere==="HOCH").length, [results]);
  const allOpt = useMemo(() => results.flatMap(r=>r.optimierungen||[]), [results]);

  const ACC  = mode==="firma" ? "#f0b430" : "#5ee7c8";
  const ACC2 = mode==="firma" ? "#e08820" : "#2ab8e0";
  const ADIM = mode==="firma" ? "#f0b43033" : "#5ee7c833";

  const PTABS = ["Dashboard","Dokumente","Analyse","Optimierung","Fristen","Rechner","Kantone","Steuererklärung","KI-Chat"];
  const FTABS = ["Dashboard","Dokumente","Analyse","Belege","MWST","Lohnbuch","Jahresabschluss","Fristen","KI-Chat"];
  const PICONS= ["⬡","📁","🔬","💡","📅","🧮","🗺","📝","💬"];
  const FICONS= ["⬡","📁","🔬","🧾","🏛","👥","📊","📅","💬"];
  const TABS  = mode==="firma" ? FTABS  : PTABS;
  const ICONS = mode==="firma" ? FICONS : PICONS;

  useEffect(() => { chatEnd.current?.scrollIntoView({behavior:"smooth"}); }, [chatHist]);

  // ── FILE READ ──────────────────────────────────────────────────────────────
  const readAndAdd = useCallback((file, setter) => {
    if (file.size > 15*1048576) { setErr(`${file.name}: max 15MB`); return; }
    if (!ALLOWED.includes(file.type)) { setErr(`${file.name}: Dateityp nicht unterstützt`); return; }
    const r = new FileReader();
    r.onload = (e) => {
      setter(prev => [...prev, {
        id: `${Date.now()}-${Math.random().toString(36).slice(2)}`,
        name:file.name, size:file.size, type:file.type,
        data: e.target.result.split(",")[1],
        status:"bereit", result:null,
      }]);
    };
    r.readAsDataURL(file);
  }, []);

  const addFiles  = useCallback((list) => { setErr(""); Array.from(list).forEach(f => readAndAdd(f, setFiles)); },  [readAndAdd]);
  const addBelege = useCallback((list) => { setErr(""); Array.from(list).forEach(f => readAndAdd(f, setBelege)); }, [readAndAdd]);
  const onDrop    = useCallback((e)    => { e.preventDefault(); setDrag(false); addFiles(e.dataTransfer.files); }, [addFiles]);

  // ── ANALYSE ALL DOCS ───────────────────────────────────────────────────────
  const analyseAll = async () => {
    if (!files.length) return;
    setBusy(true); setPct(0); setResults([]); setGlobal(""); setErr("");
    setTab(2);

    let canton  = detCanton;
    let commune = detCommune;
    const all   = [];

    const SYS = `Du bist Schweizer Steuerexperte für ${mode==="firma"?"Unternehmen":"Privatpersonen"}. Antworte AUSSCHLIESSLICH als gültiges JSON ohne Markdown-Backticks. Schema:
{"lang":"de|fr|it","canton":"2-Buchstaben oder null","commune":"Stadtname oder null","typ":"Lohnausweis|Rechnung|Kontoauszug|Bilanz|Steuererklärung|Lohnabrechnung|MWST|Wertschriften|Versicherung|Sonstiges","zusammenfassung":"2-3 Sätze","risiken":[{"titel":"","schwere":"HOCH|MITTEL|TIEF","beschreibung":""}],"optimierungen":[{"titel":"","sparpotenzial":"CHF X","aktion":""}],"kennzahlen":{"einkommen":null,"vermoegen":null,"betrag":null},"wichtig":"","riskScore":50}`;

    for (let i = 0; i < files.length; i++) {
      const f = files[i];
      setBusyMsg(`${i+1}/${files.length}: ${f.name}`);
      setPct(Math.round(i/files.length*72));
      setFiles(prev => prev.map(x => x.id===f.id ? {...x,status:"läuft"} : x));

      try {
        const raw = await api(SYS, singleMsg(
          "Analysiere dieses Schweizer Dokument. Antworte nur JSON.",
          f.type, f.data
        ));
        const parsed = safeJSON(raw) || {
          typ:"Sonstiges", zusammenfassung:raw.slice(0,200),
          risiken:[], optimierungen:[], kennzahlen:{}, wichtig:"", riskScore:50, lang:"de",
        };
        if (parsed.canton && CANTONS[parsed.canton] && !canton) {
          canton = parsed.canton;
          setDetCanton(parsed.canton);
        }
        if (parsed.commune && !commune) {
          commune = parsed.commune;
          setDetCommune(parsed.commune);
        }
        if (["de","fr","it"].includes(parsed.lang)) setLang(parsed.lang);

        const entry = { id:f.id, name:f.name, ...parsed };
        all.push(entry);
        setResults(prev => [...prev, entry]);
        setFiles(prev => prev.map(x => x.id===f.id ? {...x,status:"fertig",result:parsed} : x));
      } catch (e) {
        setFiles(prev => prev.map(x => x.id===f.id ? {...x,status:"fehler"} : x));
        setErr(`${f.name}: ${e.message}`);
      }
    }

    if (all.length > 0) {
      setBusyMsg("Gesamtanalyse...");
      setPct(88);
      try {
        const summaries = all.map(r=>`[${r.typ}] ${r.name}: ${r.zusammenfassung}`).join("\n\n");
        const txt = await api(
          `Du bist Schweizer ${mode==="firma"?"Treuhänder":"Steuerberater"} für ${canton?CANTONS[canton]?.n:"die Schweiz"}. Schreibe eine Gesamtanalyse auf Deutsch mit diesen Abschnitten: ## GESAMTBILD ## TOP RISIKEN ## SOFORTMASSNAHMEN ## SPARPOTENZIAL ## NÄCHSTE SCHRITTE`,
          [{ role:"user", content:`Dokumente:\n\n${summaries}\n\nErstelle Gesamtbewertung.` }]
        );
        setGlobal(txt);
      } catch(e) { console.error(e); }
    }
    setPct(100);
    setBusy(false);
  };

  // ── BELEGE VERARBEITEN ─────────────────────────────────────────────────────
  const verarbeiteBelege = async () => {
    if (!belege.length) return;
    setBBusy(true); setBResult([]); setErr("");
    const out = [];
    const SYS = `Du bist Schweizer Buchhalter. Analysiere den Beleg. Antworte NUR als JSON ohne Backticks:
{"datum":"YYYY-MM-DD","lieferant":"Name","beschreibung":"kurz","betragNetto":0.00,"mwstBetrag":0.00,"mwstSatz":8.1,"betragBrutto":0.00,"kategorie":"${KATEGORIEN[0]}","konto":"6000","buchungstext":"","abzugsfaehig":true}
Zahlen immer als float. mwstSatz: 8.1, 2.6, 3.8 oder 0.`;

    for (const b of belege) {
      setBelege(prev => prev.map(x => x.id===b.id ? {...x,status:"läuft"} : x));
      try {
        const raw = await api(SYS, singleMsg("Analysiere diesen Beleg. Nur JSON.", b.type, b.data));
        const p   = safeJSON(raw) || {
          datum:"?",lieferant:b.name,beschreibung:"Fehler beim Lesen",
          betragNetto:0,mwstBetrag:0,mwstSatz:8.1,betragBrutto:0,
          kategorie:"Sonstiger Betriebsaufwand",konto:"6999",buchungstext:"",abzugsfaehig:true,
        };
        // Force numeric — KI gibt manchmal Strings zurück
        p.betragNetto  = parseFloat(p.betragNetto)  || 0;
        p.mwstBetrag   = parseFloat(p.mwstBetrag)   || 0;
        p.betragBrutto = parseFloat(p.betragBrutto) || 0;
        p.mwstSatz     = parseFloat(p.mwstSatz)     || 8.1;

        const entry = {...p, _name:b.name, _id:b.id};
        out.push(entry);
        setBelege(prev => prev.map(x => x.id===b.id ? {...x,status:"fertig",result:p} : x));
      } catch(e) {
        setBelege(prev => prev.map(x => x.id===b.id ? {...x,status:"fehler"} : x));
        setErr(`${b.name}: ${e.message}`);
      }
    }
    setBResult(out);
    setBBusy(false);
  };

  // ── MWST ──────────────────────────────────────────────────────────────────
  const berechneMWST = () => {
    const rate   = parseFloat(mRate) || 8.1;
    const brutto = parseFloat(mUmsatz) || 0;
    const netto  = brutto / (1 + rate/100);
    const schuld = brutto - netto;
    const vor    = parseFloat(mVor) > 0 ? parseFloat(mVor) : bMWST;
    const zahlen = Math.max(0, schuld - vor);
    const rueck  = schuld < vor ? vor - schuld : 0;
    setMResult({
      quartal:mQuart, rate, brutto,
      netto:   +netto.toFixed(2),
      schuld:  +schuld.toFixed(2),
      vor:     +vor.toFixed(2),
      zahlen:  +zahlen.toFixed(2),
      rueck:   +rueck.toFixed(2),
      fromBelege: parseFloat(mVor)<=0 && bMWST>0,
    });
  };

  // ── LOHNBUCH ──────────────────────────────────────────────────────────────
  const berechneLohn = async () => {
    const mit = lohn.filter(m => m.name.trim() && parseFloat(m.brutto)>0);
    if (!mit.length) { setErr("Bitte Name und Bruttogehalt eingeben."); return; }
    setLohnBusy(true); setLohnOut(""); setErr("");

    const rows = mit.map(m => { const z=lohnCalc(m.brutto); return {...m,...z}; });
    const tBrutto = rows.reduce((a,r)=>a+r.brutto,0);
    const tNetto  = rows.reduce((a,r)=>a+r.netto,0);
    const tAG     = rows.reduce((a,r)=>a+r.gesamtAG,0);

    const body = `Erstelle Lohnabrechnung für ${fName||"Musterfirma"} (${fForm}), Kanton ${CANTONS[fCanton]?.n||fCanton}:

${rows.map(m=>`${m.name} | Kanton ${m.kt}
  Brutto/Mt:    CHF ${m.brutto.toFixed(2)}
  AHV AN 5.3%:  CHF ${m.ahvAN.toFixed(2)}
  IV  AN 0.7%:  CHF ${m.ivAN.toFixed(2)}
  EO  AN 0.25%: CHF ${m.eoAN.toFixed(2)}
  ALV AN 1.1%:  CHF ${m.alvAN.toFixed(2)}
  Netto/Mt:     CHF ${m.netto.toFixed(2)}
  AG-Kosten/Mt: CHF ${m.gesamtAG.toFixed(2)}`).join("\n\n")}

TOTAL Brutto: CHF ${tBrutto.toFixed(2)} | Netto: CHF ${tNetto.toFixed(2)} | AG-Kosten: CHF ${tAG.toFixed(2)}

Erstelle: 1) Lohnabrechnung je MA 2) Arbeitgeberzusammenfassung 3) Buchungsvorschlag 4) Lohnausweis-Hinweise`;

    try {
      const txt = await api(
        `Du bist Schweizer Lohnbuchhalter. Vollständige Lohnunterlagen gemäss AHVG/BVG auf Deutsch. Kanton ${CANTONS[fCanton]?.n}.`,
        [{ role:"user", content: body }]
      );
      setLohnOut(txt);
    } catch(e) { setErr(`Lohn: ${e.message}`); }
    setLohnBusy(false);
  };

  // ── JAHRESABSCHLUSS ───────────────────────────────────────────────────────
  const jahresabschluss = async () => {
    setJahrBusy(true); setJahrOut(""); setErr("");
    const ci = CANTONS[fCanton];

    const bSum = bResult.length>0
      ? `\nBelege (${bResult.length} Stk.): Netto CHF ${bNetto.toFixed(2)}, Vorsteuer CHF ${bMWST.toFixed(2)}, Kategorien: ${[...new Set(bResult.map(b=>b.kategorie))].join(", ")}`:"";
    const dSum = results.length>0
      ? `\nDokumente: ${results.map(r=>`[${r.typ}] ${r.zusammenfassung?.slice(0,100)}`).join("; ")}`:"";
    const lSum = lohn.filter(m=>m.brutto).length>0
      ? `\nLohnkosten: ${lohn.filter(m=>m.brutto).map(m=>{const z=lohnCalc(m.brutto);return `${m.name||"MA"}: CHF ${z.gesamtAG.toFixed(0)}/Mt AG-Kosten`;}).join(", ")}`:"";

    try {
      const txt = await api(
        `Du bist dipl. Schweizer Treuhänder. Vollständiger Jahresabschluss nach OR Art. 957ff. für ${fName||"Musterfirma"} (${fForm}), Kanton ${ci?.n||fCanton}. Gewinnsteuersatz Kanton+Gemeinde: ${ci?.c||8}%, Bundessteuer: 8.5%. Steuerjahr 2024. Abschnitte: ## BILANZ ## ERFOLGSRECHNUNG ## STEUERBERECHNUNG ## ANHANG ## EMPFEHLUNGEN`,
        [{ role:"user", content:`Daten:${bSum}${dSum}${lSum}\n\nErstelle vollständigen OR-Jahresabschluss.` }],
        1000
      );
      setJahrOut(txt);
    } catch(e) { setErr(`Jahresabschluss: ${e.message}`); }
    setJahrBusy(false);
  };

  // ── STEUERRECHNER ─────────────────────────────────────────────────────────
  const rechner = () => {
    if (!calcIn && !calcWl) { setErr("Einkommen oder Vermögen eingeben."); return; }
    setErr("");
    const inc = parseFloat(calcIn)||0;
    const wl  = parseFloat(calcWl)||0;
    setCalcRes(
      Object.entries(CANTONS)
        .map(([k,v]) => ({ k, n:v.n, r:v.p, est: Math.round(inc*(v.p/100) + wl*0.0015) }))
        .sort((a,b)=>a.est-b.est)
    );
  };

  // ── STEUERERKLÄRUNG ───────────────────────────────────────────────────────
  const steuererklarung = async () => {
    setTaxBusy(true); setTaxOut(""); setErr("");
    const ck = detCanton || "ZH";
    const ci = CANTONS[ck];
    const docCtx = results.map(r=>`${r.typ}: ${Object.entries(r.kennzahlen||{}).filter(([,v])=>v).map(([k,v])=>`${k} ${v}`).join(", ")}`).filter(Boolean).join("\n");

    try {
      const txt = await api(
        `Du bist Schweizer Steuerexperte, Kanton ${ci?.n||ck}${detCommune?`, Gemeinde ${detCommune}`:""}, Steuersatz ${ci?.p||13}%. Vollständige Steuererklärung mit Steuerberechnung auf Deutsch.`,
        [{ role:"user", content:`${taxF.name||"Steuerpflichtige Person"}, ${taxF.status}, ${taxF.kids} Kinder\nEinkommen: CHF ${taxF.inc||"0"}\nVermögen: CHF ${taxF.wlth||"0"}\nAbzüge: ${taxF.ded||"keine"}\n${docCtx?`Dokumentkontext:\n${docCtx}`:""}\n\nVollständige Steuererklärung mit allen Positionen und Steuerberechnung erstellen.` }]
      );
      setTaxOut(txt);
    } catch(e) { setErr(`Steuererklärung: ${e.message}`); }
    setTaxBusy(false);
  };

  // ── CHAT ──────────────────────────────────────────────────────────────────
  const sendChat = async () => {
    const q = chatIn.trim();
    if (!q || chatBusy) return;
    const newHist = [...chatHist, {role:"user", content:q}];
    setChatHist(newHist);
    setChatIn("");
    setChatBusy(true);

    const ctx = mode==="firma"
      ? `Firma: "${fName||"?"}" (${fForm}), Kanton ${CANTONS[fCanton]?.n||fCanton}. Belege: ${belege.length}. Analysen: ${results.length}.`
      : `Kanton: ${detCanton?CANTONS[detCanton]?.n:"unbekannt"}. Analysen: ${results.length}.`;

    try {
      // FIX: Pass full history correctly as messages array
      const txt = await api(
        `Du bist Schweizer ${mode==="firma"?"Treuhänder (Steuer, Buchhaltung, MWST, Lohn, OR)":"Steuerberater für Privatpersonen (DBG, StHG)"}. Antworte präzise auf Deutsch. Kontext: ${ctx}`,
        newHist.map(m => ({ role:m.role, content:m.content }))
      );
      setChatHist(prev => [...prev, {role:"assistant", content:txt}]);
    } catch(e) {
      setChatHist(prev => [...prev, {role:"assistant", content:`⚠ Fehler: ${e.message}`}]);
    }
    setChatBusy(false);
  };

  // ── UTILS ─────────────────────────────────────────────────────────────────
  const dl = (txt, fn) => {
    const a = document.createElement("a");
    a.href = URL.createObjectURL(new Blob([txt], {type:"text/plain;charset=utf-8"}));
    a.download = fn; a.click();
  };
  const cp = (txt) => navigator.clipboard.writeText(txt).catch(()=>{});

  const CopyDl = ({text, fn}) => (
    <div style={{display:"flex",gap:"0.4rem"}}>
      <button className="btn-o btn-sm" onClick={()=>cp(text)}>⊡ Kopieren</button>
      <button className="btn-o btn-sm" onClick={()=>dl(text,fn)}>↓ TXT</button>
    </div>
  );

  // ── RENDER ────────────────────────────────────────────────────────────────
  if (!mode) return (
    <div style={{minHeight:"100vh",background:"#04080f",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",fontFamily:"'DM Sans',sans-serif",padding:"2rem"}}>
      <style>{BASE_CSS + dynCSS("#5ee7c8","#2ab8e0","#5ee7c833")}</style>
      <div style={{textAlign:"center",marginBottom:"2.5rem"}} className="fu">
        <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.6rem",color:"#1a3040",letterSpacing:"0.2em",marginBottom:"1rem"}}>WILLKOMMEN BEI</div>
        <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"clamp(2rem,5vw,3rem)",color:"#c8dde8",marginBottom:"0.4rem"}}>
          Swiss<span style={{color:"#5ee7c8"}}>Tax</span>.AI <span style={{color:"#f0b430"}}>Pro</span>
        </div>
        <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",color:"#2a4050",letterSpacing:"0.14em"}}>STEUER & BUCHHALTUNG · PRIVATPERSONEN & UNTERNEHMEN · ALLE 26 KANTONE</div>
      </div>

      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:"1.4rem",maxWidth:660,width:"100%"}}>
        {[
          {key:"private",icon:"👤",title:"Privatpersonen",a:"#5ee7c8",b:"#2ab8e0",anim:"gp",
           desc:"Steuererklärung · Optimierung · Kantonsvergleich · KI-Beratung",
           feat:["Kanton & Sprache automatisch erkannt","Steuererklärung auto-ausfüllen","Alle 26 Kantone vergleichen","KI-Steuerberater 24/7"]},
          {key:"firma",icon:"🏢",title:"Unternehmen",a:"#f0b430",b:"#e08820",anim:"gg",
           desc:"Belegverarbeitung · MWST · Lohnbuch · Jahresabschluss",
           feat:["Belege automatisch verbuchen","MWST-Abrechnung Q1–Q4","Lohnbuchhaltung & Lohnausweise","Jahresabschluss nach OR"]},
        ].map(o=>(
          <div key={o.key} onClick={()=>{setMode(o.key);setTab(0);}}
            style={{background:"#080f1a",border:`1px solid ${o.a}33`,padding:"1.8rem",borderRadius:"4px",cursor:"pointer",transition:"transform 0.25s,border-color 0.25s",animation:`${o.anim} 4s ease infinite`}}
            onMouseEnter={e=>{e.currentTarget.style.transform="translateY(-4px)";e.currentTarget.style.borderColor=o.a+"77";}}
            onMouseLeave={e=>{e.currentTarget.style.transform="";e.currentTarget.style.borderColor=o.a+"33";}}>
            <div style={{fontSize:"2rem",marginBottom:"0.7rem"}}>{o.icon}</div>
            <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.25rem",color:"#c8dde8",marginBottom:"0.3rem"}}>{o.title}</div>
            <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"1rem",lineHeight:1.7}}>{o.desc}</div>
            <div style={{display:"flex",flexDirection:"column",gap:"0.35rem",marginBottom:"1.3rem"}}>
              {o.feat.map(f=>(
                <div key={f} style={{display:"flex",gap:"0.45rem",fontSize:"0.78rem",color:"#778e9e",alignItems:"flex-start"}}>
                  <span style={{color:o.a,flexShrink:0,marginTop:"0.05rem"}}>✓</span>{f}
                </div>
              ))}
            </div>
            <div style={{background:`linear-gradient(135deg,${o.a},${o.b})`,color:"#04080f",padding:"0.55rem",textAlign:"center",fontFamily:"'Space Mono',monospace",fontSize:"0.63rem",fontWeight:700,borderRadius:"2px",letterSpacing:"0.08em"}}>STARTEN →</div>
          </div>
        ))}
      </div>
      <div style={{marginTop:"1.8rem",fontFamily:"'Space Mono',monospace",fontSize:"0.5rem",color:"#1a3040",textAlign:"center",letterSpacing:"0.12em"}}>KI-GESTÜTZT · KEIN RECHTSERSATZ · LOKALE DATENVERARBEITUNG</div>
    </div>
  );

  // ── MAIN APP ──────────────────────────────────────────────────────────────
  return (
    <div style={{minHeight:"100vh",background:"#04080f",color:"#c8dde8",fontFamily:"'DM Sans',sans-serif",overflowX:"hidden"}}>
      <style>{BASE_CSS + dynCSS(ACC,ACC2,ADIM)}</style>

      {/* HEADER */}
      <div style={{background:"#060c16",borderBottom:"1px solid #0e1e2e",padding:"0 1.3rem",position:"sticky",top:0,zIndex:100}}>
        <div style={{maxWidth:1060,margin:"0 auto"}}>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"0.72rem 0 0",flexWrap:"wrap",gap:"0.5rem"}}>
            <div style={{display:"flex",alignItems:"center",gap:"0.65rem"}}>
              <div style={{width:30,height:30,background:`linear-gradient(135deg,${ACC},${ACC2})`,display:"flex",alignItems:"center",justifyContent:"center",fontSize:"0.88rem",borderRadius:"3px",flexShrink:0}}>{mode==="firma"?"🏢":"⚖"}</div>
              <div>
                <div style={{fontFamily:"'Space Mono',monospace",fontWeight:700,fontSize:"0.88rem"}}>SWISS<span className="acc">TAX</span>.AI <span style={{fontSize:"0.57rem",opacity:0.7}} className="acc">{mode==="firma"?"BUSINESS":"PRIVATE"}</span></div>
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.47rem",color:"#1a3040",letterSpacing:"0.12em"}}>{mode==="firma"?"BUCHHALTUNG · MWST · LOHNBUCH · JAHRESABSCHLUSS":"STEUER-SUITE · ALLE 26 KANTONE"}</div>
              </div>
            </div>
            <div style={{display:"flex",alignItems:"center",gap:"0.55rem",flexWrap:"wrap"}}>
              {mode==="firma" && <>
                <input value={fName} onChange={e=>setFName(e.target.value)} placeholder="Firmenname..."
                  style={{background:"#0e1e2e",border:`1px solid ${ADIM}`,color:"#c8dde8",padding:"0.27rem 0.6rem",fontFamily:"'Space Mono',monospace",fontSize:"0.6rem",borderRadius:"2px",outline:"none",width:125}}/>
                <select value={fForm} onChange={e=>setFForm(e.target.value)}
                  style={{background:"#060c16",border:"1px solid #0e1e2e",color:"#c8dde8",padding:"0.27rem 0.5rem",fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",borderRadius:"2px",outline:"none",cursor:"pointer"}}>
                  {["GmbH","AG","Einzelunternehmen","Kollektivgesellschaft"].map(r=><option key={r}>{r}</option>)}
                </select>
                <select value={fCanton} onChange={e=>setFCanton(e.target.value)}
                  style={{background:"#060c16",border:"1px solid #0e1e2e",color:"#c8dde8",padding:"0.27rem 0.5rem",fontFamily:"'Space Mono',monospace",fontSize:"0.6rem",borderRadius:"2px",outline:"none",cursor:"pointer"}}>
                  {Object.keys(CANTONS).map(k=><option key={k}>{k}</option>)}
                </select>
              </>}
              <button onClick={()=>{setMode(null);setTab(0);setFiles([]);setResults([]);setBelege([]);setBResult([]);setGlobal("");setErr("");setChatHist([]);}}
                style={{background:"transparent",border:"1px solid #0e1e2e",color:"#2a4050",padding:"0.27rem 0.58rem",fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",cursor:"pointer",borderRadius:"2px"}}>← Wechseln</button>
              <div style={{display:"flex",gap:"0.2rem"}}>
                {["de","fr","it"].map(l=>(
                  <button key={l} onClick={()=>setLang(l)}
                    style={{padding:"0.24rem 0.46rem",border:"1px solid",borderColor:lang===l?ACC:"#0e1e2e",background:lang===l?ADIM:"transparent",color:lang===l?ACC:"#2a4050",fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",cursor:"pointer",borderRadius:"2px"}}>
                    {l.toUpperCase()}
                  </button>
                ))}
              </div>
              <div style={{display:"flex",alignItems:"center",gap:"0.3rem"}}>
                <div style={{width:5,height:5,borderRadius:"50%",background:ACC,animation:"pu 2s infinite"}}/>
                <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.5rem",color:"#1a3040"}}>LIVE</span>
              </div>
            </div>
          </div>
          <div style={{display:"flex",overflowX:"auto",marginTop:"0.32rem"}}>
            {TABS.map((t,i)=>(
              <button key={i} className={`tab${tab===i?" active":""}`} onClick={()=>setTab(i)}>{ICONS[i]} {t}</button>
            ))}
          </div>
        </div>
      </div>

      <div style={{maxWidth:1060,margin:"0 auto",padding:"1.3rem"}}>
        {/* ERROR */}
        {err && (
          <div className="errbx">
            <span>⚠ {err}</span>
            <button className="del" onClick={()=>setErr("")}>✕</button>
          </div>
        )}

        {/* ── DASHBOARD ── */}
        {tab===0 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>
                {mode==="firma"?(fName||"Ihr Unternehmen"):"Steuer-Cockpit"} <span className="acc">2025</span>
              </div>
              {detCanton && <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",background:ADIM,color:ACC,padding:"0.18rem 0.55rem",borderRadius:"2px"}}>📍 Erkannt: {CANTONS[detCanton]?.n}{detCommune?` · ${detCommune}`:""}</span>}
            </div>

            <div className="g4" style={{marginBottom:"1.1rem"}}>
              {(mode==="firma"?[
                {v:files.length+belege.length, l:"DOKUMENTE / BELEGE", c:ACC},
                {v:bResult.length,             l:"BELEGE VERBUCHT",    c:ACC},
                {v:chf(bNetto),                l:"TOTAL AUFWAND",      c:"#ffd93d"},
                {v:chf(bMWST),                 l:"VORSTEUER",          c:ACC},
              ]:[
                {v:files.length,   l:"DOKUMENTE",     c:ACC},
                {v:highR,          l:"RISIKEN HOCH",  c:highR>0?"#ff7070":ACC},
                {v:allOpt.length,  l:"OPTIMIERUNGEN", c:"#ffd93d"},
                {v:(mode==="firma"?DL_FIRMA:DL_PRIVATE).filter(d=>daysLeft(d.d)>0&&daysLeft(d.d)<90).length, l:"DRINGENDE FRISTEN", c:"#ffd93d"},
              ]).map((s,i)=>(
                <div key={i} className="stat">
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:typeof s.v==="string"?"0.82rem":"1.45rem",color:s.c,fontWeight:700}}>{s.v}</div>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.5rem",color:"#1a3040",marginTop:"0.2rem",letterSpacing:"0.09em"}}>{s.l}</div>
                </div>
              ))}
            </div>

            <div className="g2" style={{marginBottom:"1.1rem"}}>
              <div className="card">
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ SCHNELLSTART</div>
                <div style={{display:"flex",flexDirection:"column",gap:"0.4rem"}}>
                  {mode==="firma"?<>
                    <button className="btn-p" onClick={()=>setTab(3)}>🧾 Belege hochladen & verbuchen</button>
                    <button className="btn-p" onClick={()=>setTab(4)}>🏛 MWST abrechnen</button>
                    <button className="btn-o" onClick={()=>setTab(5)}>👥 Lohnbuchhaltung</button>
                    <button className="btn-o" onClick={()=>setTab(6)}>📊 Jahresabschluss erstellen</button>
                  </>:<>
                    <button className="btn-p" onClick={()=>setTab(1)}>📁 Dokumente hochladen</button>
                    {files.length>0&&<button className="btn-p" onClick={analyseAll} disabled={busy}>🔬 Alle analysieren ({files.length})</button>}
                    <button className="btn-o" onClick={()=>setTab(5)}>🧮 Steuerrechner</button>
                    <button className="btn-o" onClick={()=>setTab(7)}>📝 Steuererklärung</button>
                  </>}
                </div>
              </div>
              <div className="card">
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ NÄCHSTE FRISTEN</div>
                {(mode==="firma"?DL_FIRMA:DL_PRIVATE).filter(d=>daysLeft(d.d)>0).slice(0,5).map((d,i)=>{
                  const days=daysLeft(d.d);
                  return <div key={i} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"0.37rem 0",borderBottom:"1px solid #0e1e2e"}}>
                    <div style={{fontSize:"0.77rem",color:"#aabccc",paddingRight:"0.5rem"}}>{d.t}</div>
                    <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.67rem",color:days<30?"#ff7070":days<90?"#ffd93d":ACC,flexShrink:0}}>{days}d</span>
                  </div>;
                })}
              </div>
            </div>

            {mode==="firma" && bResult.length>0 && (
              <div className="card-hi" style={{marginBottom:"1.1rem"}}>
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ BUCHUNGSÜBERSICHT ({bResult.length} Buchungen)</div>
                <div className="trow">
                  <div className="tr trh"><span>DATUM</span><span>LIEFERANT</span><span>KATEGORIE</span><span>NETTO</span><span>MWST</span><span>SATZ</span></div>
                  {bResult.slice(-6).map((b,i)=>(
                    <div key={i} className="tr" style={{background:i%2===0?"#060c16":"#080f1a"}}>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",color:"#2a4050"}}>{b.datum}</span>
                      <span style={{fontSize:"0.76rem"}}>{(b.lieferant||"").slice(0,18)}</span>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC}}>{(b.kategorie||"").slice(0,16)}</span>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.66rem"}}>{chf(b.betragNetto)}</span>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.66rem",color:"#ffd93d"}}>{chf(b.mwstBetrag)}</span>
                      <span className="badge bm" style={{fontSize:"0.53rem"}}>{b.mwstSatz}%</span>
                    </div>
                  ))}
                  <div className="tr trh" style={{background:"#0a1525",marginTop:"0.25rem"}}>
                    <span/><span style={{color:"#c8dde8"}}>TOTAL</span><span/>
                    <span style={{color:ACC}}>{chf(bNetto)}</span>
                    <span style={{color:"#ffd93d"}}>{chf(bMWST)}</span>
                    <span/>
                  </div>
                </div>
              </div>
            )}

            {global && (
              <div className="card-hi">
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.8rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,letterSpacing:"0.1em"}}>▸ KI GESAMTANALYSE</div>
                  <button className="btn-o btn-sm" onClick={()=>setTab(2)}>Vollständig →</button>
                </div>
                <MD text={global.slice(0,600)+(global.length>600?"...":"")} acc={ACC}/>
              </div>
            )}
          </div>
        )}

        {/* ── DOKUMENTE ── */}
        {tab===1 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>Dokumente <span className="acc">hochladen</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>Kanton & Sprache automatisch erkannt · Unbegrenzt viele Dateien · Max 15MB</div>
            </div>

            <div onDrop={onDrop} onDragOver={e=>{e.preventDefault();setDrag(true);}} onDragLeave={()=>setDrag(false)} onClick={()=>fileRef.current.click()}
              style={{border:`2px dashed ${drag?ACC:"#0e1e2e"}`,padding:"2.3rem",textAlign:"center",cursor:"pointer",background:drag?ADIM.replace("33","0a"):"#060c16",transition:"all 0.25s",marginBottom:"1rem",borderRadius:"3px"}}>
              <div style={{fontSize:"1.8rem",opacity:0.3,marginBottom:"0.55rem"}}>⊕</div>
              <div style={{color:"#778e9e",marginBottom:"0.3rem"}}>Dateien hierher ziehen oder klicken</div>
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.56rem",color:"#1a3040",letterSpacing:"0.09em"}}>PDF · PNG · JPG · DOCX · KANTON WIRD AUTOMATISCH ERKANNT</div>
            </div>
            <input ref={fileRef} type="file" multiple accept=".pdf,.png,.jpg,.jpeg,.docx" onChange={e=>addFiles(e.target.files)} style={{display:"none"}}/>

            {files.length>0 && <>
              <div className="card" style={{marginBottom:"1rem"}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.7rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",letterSpacing:"0.1em"}}>▸ {files.length} DOKUMENT{files.length!==1?"E":""}</div>
                  <button className="del" onClick={()=>setFiles([])}>ALLE ENTFERNEN</button>
                </div>
                {files.map(f=><FileItem key={f.id} f={f} acc={ACC} adim={ADIM} onRemove={()=>setFiles(p=>p.filter(x=>x.id!==f.id))}/>)}
              </div>
              <button className="btn-p" style={{width:"100%",padding:"0.9rem"}} onClick={analyseAll} disabled={busy}>
                {busy?<span style={{animation:"pu 1s infinite",display:"inline-block"}}>◉ {busyMsg}</span>:`🔬 ALLE ${files.length} DOKUMENTE ANALYSIEREN`}
              </button>
              {busy && <ProgressBar pct={pct} acc={ACC} acc2={ACC2} msg={busyMsg}/>}
            </>}
          </div>
        )}

        {/* ── ANALYSE ── */}
        {tab===2 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.25rem"}}>KI-<span className="acc">Analyse</span></div>
              {detCanton && <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",background:ADIM,color:ACC,padding:"0.18rem 0.55rem",borderRadius:"2px"}}>📍 {CANTONS[detCanton]?.n}{detCommune?` · ${detCommune}`:""} · {CANTONS[detCanton]?.p}%</span>}
            </div>

            {!results.length && !busy && (
              <div className="card" style={{textAlign:"center",padding:"2.5rem"}}>
                <div style={{fontSize:"2.2rem",opacity:0.15,marginBottom:"0.7rem"}}>🔬</div>
                <div style={{color:"#2a4050",marginBottom:"0.8rem"}}>Noch keine Analyse</div>
                <button className="btn-p" onClick={()=>setTab(1)}>📁 Dokumente hochladen →</button>
              </div>
            )}

            {busy && (
              <div className="card-hi" style={{textAlign:"center",padding:"2.5rem"}}>
                <div style={{fontSize:"1.4rem",animation:"pu 1s infinite",color:ACC,marginBottom:"0.6rem"}}>◉</div>
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.7rem",color:ACC}}>{busyMsg}</div>
                <ProgressBar pct={pct} acc={ACC} acc2={ACC2}/>
              </div>
            )}

            {global && (
              <div className="card-hi" style={{marginBottom:"1.1rem"}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.8rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,letterSpacing:"0.1em"}}>▸ GESAMTANALYSE · {results.length} DOKUMENTE</div>
                  <CopyDl text={global} fn="analyse.txt"/>
                </div>
                <MD text={global} acc={ACC}/>
              </div>
            )}

            {results.map((r,i)=>(
              <div key={i} className="card" style={{marginBottom:"0.85rem"}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:"0.8rem",flexWrap:"wrap",gap:"0.4rem"}}>
                  <div style={{minWidth:0}}>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:"#2a4050",letterSpacing:"0.1em",marginBottom:"0.18rem"}}>{r.typ}</div>
                    <div style={{fontSize:"0.88rem",fontWeight:500,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{r.name}</div>
                    <div style={{display:"flex",gap:"0.28rem",marginTop:"0.22rem",flexWrap:"wrap"}}>
                      {r.canton&&CANTONS[r.canton]&&<span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:ACC,background:ADIM,padding:"0.08rem 0.38rem",borderRadius:"2px"}}>📍{CANTONS[r.canton].n}{r.commune?` · ${r.commune}`:""}</span>}
                      {r.lang&&<span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:"#ffd93d",background:"#ffd93d11",padding:"0.08rem 0.38rem",borderRadius:"2px"}}>🌐{r.lang.toUpperCase()}</span>}
                    </div>
                  </div>
                  <div style={{display:"flex",gap:"0.28rem",flexWrap:"wrap",alignItems:"center",flexShrink:0}}>
                    {r.riskScore!==undefined&&<span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:r.riskScore>60?"#ff7070":r.riskScore>30?"#ffd93d":ACC,background:"#0e1e2e",padding:"0.13rem 0.42rem",borderRadius:"2px"}}>{r.riskScore}/100</span>}
                    {(r.risiken||[]).filter(x=>x.schwere==="HOCH").length>0&&<span className="badge bh">{(r.risiken||[]).filter(x=>x.schwere==="HOCH").length} HOCH</span>}
                  </div>
                </div>

                <div style={{color:"#778e9e",fontSize:"0.79rem",lineHeight:1.6,background:"#060c16",padding:"0.65rem 0.82rem",borderRadius:"2px",marginBottom:"0.7rem"}}>{r.zusammenfassung}</div>

                {r.wichtig&&<div style={{background:ADIM.replace("33","0d"),border:`1px solid ${ADIM}`,padding:"0.48rem 0.68rem",fontFamily:"'Space Mono',monospace",fontSize:"0.6rem",color:ACC,marginBottom:"0.7rem",borderRadius:"2px"}}>💡 {r.wichtig}</div>}

                {(r.risiken||[]).length>0&&(
                  <div style={{marginBottom:"0.7rem"}}>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.54rem",color:"#2a4050",marginBottom:"0.3rem",letterSpacing:"0.1em"}}>RISIKEN</div>
                    {r.risiken.map((x,j)=>(
                      <div key={j} style={{display:"flex",gap:"0.55rem",padding:"0.37rem 0",borderBottom:"1px solid #0e1e2e",alignItems:"flex-start"}}>
                        <span className={`badge ${x.schwere==="HOCH"?"bh":x.schwere==="MITTEL"?"bm":"bl"}`} style={{minWidth:50,textAlign:"center",flexShrink:0}}>{x.schwere}</span>
                        <div><div style={{fontSize:"0.79rem",marginBottom:"0.08rem"}}>{x.titel}</div><div style={{fontSize:"0.7rem",color:"#2a5060"}}>{x.beschreibung}</div></div>
                      </div>
                    ))}
                  </div>
                )}

                {(r.optimierungen||[]).length>0&&(
                  <div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.54rem",color:"#2a4050",marginBottom:"0.3rem",letterSpacing:"0.1em"}}>OPTIMIERUNGEN</div>
                    {r.optimierungen.map((x,j)=>(
                      <div key={j} style={{display:"flex",gap:"0.55rem",padding:"0.37rem 0",borderBottom:"1px solid #0e1e2e",alignItems:"center"}}>
                        <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.63rem",color:ACC,minWidth:85,flexShrink:0}}>{x.sparpotenzial}</span>
                        <div><div style={{fontSize:"0.79rem"}}>{x.titel}</div><div style={{fontSize:"0.7rem",color:"#2a5060"}}>{x.aktion}</div></div>
                      </div>
                    ))}
                  </div>
                )}
              </div>
            ))}
          </div>
        )}

        {/* ── FIRMA BELEGE ── */}
        {mode==="firma" && tab===3 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>Beleg-<span className="acc">verarbeitung</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>Rechnungen & Quittungen → MWST erkennen → automatisch verbuchen</div>
            </div>

            <div onClick={()=>bRef.current.click()}
              onDrop={e=>{e.preventDefault();addBelege(e.dataTransfer.files);}}
              onDragOver={e=>e.preventDefault()}
              style={{border:`2px dashed ${ADIM}`,padding:"2rem",textAlign:"center",cursor:"pointer",background:"#060c16",marginBottom:"1rem",borderRadius:"3px",transition:"border-color 0.2s"}}
              onMouseEnter={e=>e.currentTarget.style.borderColor=ACC}
              onMouseLeave={e=>e.currentTarget.style.borderColor=ADIM}>
              <div style={{fontSize:"1.6rem",opacity:0.3,marginBottom:"0.5rem"}}>🧾</div>
              <div style={{color:"#778e9e",marginBottom:"0.3rem"}}>Belege hochladen</div>
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:"#1a3040"}}>PDF · PNG · JPG · MWST & KATEGORIE AUTOMATISCH</div>
            </div>
            <input ref={bRef} type="file" multiple accept=".pdf,.png,.jpg,.jpeg" onChange={e=>addBelege(e.target.files)} style={{display:"none"}}/>

            {belege.length>0&&<>
              <div className="card" style={{marginBottom:"1rem"}}>
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.7rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",letterSpacing:"0.1em"}}>▸ {belege.length} BELEG{belege.length!==1?"E":""}</div>
                  <button className="del" onClick={()=>setBelege([])}>ALLE ENTFERNEN</button>
                </div>
                {belege.map(b=><FileItem key={b.id} f={b} acc={ACC} adim={ADIM} onRemove={()=>setBelege(p=>p.filter(x=>x.id!==b.id))}/>)}
              </div>
              <button className="btn-p" style={{width:"100%",padding:"0.9rem",marginBottom:"1rem"}} onClick={verarbeiteBelege} disabled={bBusy}>
                {bBusy?<span style={{animation:"pu 1s infinite",display:"inline-block"}}>◉ Verarbeite...</span>:`🧾 ALLE ${belege.length} BELEGE VERARBEITEN`}
              </button>
            </>}

            {bResult.length>0&&(
              <div className="card-hi">
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.8rem",flexWrap:"wrap",gap:"0.4rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,letterSpacing:"0.1em"}}>▸ BUCHUNGSJOURNAL · {bResult.length} BUCHUNGEN</div>
                  <CopyDl
                    text={"Datum;Lieferant;Beschreibung;Netto;MWST;Satz%;Kategorie;Konto\n"+bResult.map(b=>`${b.datum};${b.lieferant};${b.beschreibung};${b.betragNetto.toFixed(2)};${b.mwstBetrag.toFixed(2)};${b.mwstSatz}%;${b.kategorie};${b.konto}`).join("\n")+`\n\nTOTAL;;;;;;\nNetto: ${chf(bNetto)};;;MWST: ${chf(bMWST)}`}
                    fn="buchungsjournal.csv"
                  />
                </div>
                <div className="trow">
                  <div className="tr trh"><span>DATUM</span><span>LIEFERANT</span><span>KATEGORIE</span><span>NETTO</span><span>MWST</span><span>SATZ</span></div>
                  {bResult.map((b,i)=>(
                    <div key={i} className="tr" style={{background:i%2===0?"#060c16":"#080f1a"}}>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050"}}>{b.datum}</span>
                      <span style={{fontSize:"0.75rem"}}>{(b.lieferant||"").slice(0,18)}</span>
                      <div>
                        <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.56rem",color:ACC}}>{(b.kategorie||"").slice(0,16)}</div>
                        <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.5rem",color:"#2a4050"}}>{b.konto}</div>
                      </div>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.65rem"}}>{chf(b.betragNetto)}</span>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.65rem",color:"#ffd93d"}}>{chf(b.mwstBetrag)}</span>
                      <span className="badge bm" style={{fontSize:"0.52rem"}}>{b.mwstSatz}%</span>
                    </div>
                  ))}
                  <div className="tr trh" style={{background:"#0a1525",marginTop:"0.25rem"}}>
                    <span/><span style={{color:"#c8dde8"}}>TOTAL</span><span/>
                    <span style={{color:ACC}}>{chf(bNetto)}</span>
                    <span style={{color:"#ffd93d"}}>{chf(bMWST)}</span>
                    <span/>
                  </div>
                </div>
              </div>
            )}
          </div>
        )}

        {/* ── FIRMA MWST ── */}
        {mode==="firma" && tab===4 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>MWST-<span className="acc">Abrechnung</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>Effektive Methode · Formular 310 · ESTV</div>
            </div>

            <div className="card" style={{marginBottom:"1rem"}}>
              <div className="g2" style={{marginBottom:"0.85rem"}}>
                <div>
                  <label className="lbl">QUARTAL / PERIODE</label>
                  <select className="sel" value={mQuart} onChange={e=>setMQuart(e.target.value)}>
                    {["Q1 2025","Q2 2025","Q3 2025","Q4 2025","Q1 2026"].map(q=><option key={q}>{q}</option>)}
                  </select>
                </div>
                <div>
                  <label className="lbl">MWST-SATZ</label>
                  <select className="sel" value={mRate} onChange={e=>setMRate(e.target.value)}>
                    {MWST_RATES.map(r=><option key={r.v} value={r.v}>{r.label}</option>)}
                  </select>
                </div>
                <div>
                  <label className="lbl">UMSATZ BRUTTO (INKL. MWST) CHF</label>
                  <input className="inp" type="number" min="0" step="0.01" placeholder="120000.00" value={mUmsatz} onChange={e=>setMUmsatz(e.target.value)}/>
                </div>
                <div>
                  <label className="lbl">VORSTEUER MANUELL (LEER = AUS BELEGEN)</label>
                  <input className="inp" type="number" min="0" step="0.01" placeholder={bMWST>0?`${bMWST.toFixed(2)} aus Belegen`:"0.00"} value={mVor} onChange={e=>setMVor(e.target.value)}/>
                  {bMWST>0&&!mVor&&<div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:ACC,marginTop:"0.28rem"}}>✓ {chf(bMWST)} aus {bResult.length} Belegen übernommen</div>}
                </div>
              </div>
              <button className="btn-p" style={{width:"100%"}} onClick={berechneMWST} disabled={!mUmsatz}>🏛 MWST BERECHNEN</button>
            </div>

            {mResult&&(
              <div className="card-hi" style={{marginBottom:"1rem"}}>
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,marginBottom:"0.9rem",letterSpacing:"0.1em"}}>▸ ABRECHNUNG {mResult.quartal} · {mResult.rate}%</div>
                <div style={{display:"grid",gap:"0.42rem",marginBottom:"0.9rem"}}>
                  {[
                    {l:"Umsatz brutto (inkl. MWST)",v:chf(mResult.brutto),c:"#c8dde8"},
                    {l:"Umsatz netto (exkl. MWST)", v:chf(mResult.netto), c:"#778e9e"},
                    {l:`Geschuldete MWST ${mResult.rate}%`,v:chf(mResult.schuld),c:"#ff7070"},
                    {l:`Vorsteuer${mResult.fromBelege?" (aus Belegen)":""}`, v:chf(mResult.vor), c:ACC},
                    {l:mResult.rueck>0?"RÜCKERSTATTUNG VON ESTV":"ZU ZAHLEN AN ESTV",
                     v:chf(mResult.rueck>0?mResult.rueck:mResult.zahlen),c:"#ffd93d",big:true},
                  ].map((row,i)=>(
                    <div key={i} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"0.52rem 0.75rem",background:"#060c16",borderRadius:"2px",border:row.big?`1px solid ${ADIM}`:"1px solid #0e1e2e"}}>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",color:"#2a4050"}}>{row.l}</span>
                      <span style={{fontFamily:"'Space Mono',monospace",fontSize:row.big?"0.92rem":"0.76rem",color:row.c,fontWeight:row.big?700:400}}>{row.v}</span>
                    </div>
                  ))}
                </div>
                <div className="infbx" style={{background:"#5ee7c811",borderColor:"#5ee7c833",color:"#5ee7c8",marginBottom:"0.7rem"}}>
                  💡 Einzahlungsfrist: 60 Tage nach {mResult.quartal} · Konto ESTV: 30-6000-9 · Formular 310
                </div>
                <CopyDl
                  text={`MWST ${mResult.quartal} | ${mResult.rate}%\nUmsatz brutto: ${chf(mResult.brutto)}\nUmsatz netto:  ${chf(mResult.netto)}\nGeschuldet:    ${chf(mResult.schuld)}\nVorsteuer:    -${chf(mResult.vor)}\n${"─".repeat(30)}\nZU ZAHLEN:     ${chf(mResult.zahlen)}`}
                  fn={`mwst_${mResult.quartal.replace(" ","_")}.txt`}
                />
              </div>
            )}

            <div className="card">
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ MWST-SÄTZE SCHWEIZ 2025</div>
              {MWST_RATES.map((r,i)=>(
                <div key={i} style={{display:"flex",justifyContent:"space-between",padding:"0.47rem 0",borderBottom:"1px solid #0e1e2e"}}>
                  <span style={{fontSize:"0.82rem"}}>{r.label}</span>
                  <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.73rem",color:r.v!=="0"?ACC:"#2a4050"}}>{r.v}%</span>
                </div>
              ))}
              <div className="infbx" style={{marginTop:"0.8rem",background:"#5ee7c811",borderColor:"#5ee7c833",color:"#5ee7c8"}}>
                Ab CHF 100'000 Jahresumsatz → MWST-Pflicht. Anmeldung beim ESTV erforderlich.
              </div>
            </div>
          </div>
        )}

        {/* ── FIRMA LOHNBUCH ── */}
        {mode==="firma" && tab===5 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>Lohn-<span className="acc">buchhaltung</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>AHV 10.6% · IV 1.4% · EO 0.5% · ALV 2.2% (AN+AG je 50%)</div>
            </div>

            <div className="card" style={{marginBottom:"1rem"}}>
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ MITARBEITER</div>
              {lohn.map((m,i)=>(
                <div key={m.id} style={{display:"grid",gridTemplateColumns:"1.3fr 1fr 0.55fr auto",gap:"0.45rem",marginBottom:"0.55rem",alignItems:"flex-end"}}>
                  <div><label className="lbl">NAME</label>
                    <input className="inp" placeholder="Max Muster" value={m.name} onChange={e=>setLohn(p=>p.map((x,j)=>j===i?{...x,name:e.target.value}:x))}/></div>
                  <div><label className="lbl">BRUTTO / MONAT (CHF)</label>
                    <input className="inp" type="number" min="0" placeholder="6500" value={m.brutto} onChange={e=>setLohn(p=>p.map((x,j)=>j===i?{...x,brutto:e.target.value}:x))}/></div>
                  <div><label className="lbl">KT</label>
                    <select className="sel" value={m.kt} onChange={e=>setLohn(p=>p.map((x,j)=>j===i?{...x,kt:e.target.value}:x))}>
                      {Object.keys(CANTONS).map(k=><option key={k}>{k}</option>)}
                    </select>
                  </div>
                  <button className="del" style={{marginBottom:0,alignSelf:"flex-end",padding:"0.58rem 0.55rem"}}
                    disabled={lohn.length===1} onClick={()=>setLohn(p=>p.length>1?p.filter((_,j)=>j!==i):p)}>✕</button>
                </div>
              ))}
              <button className="btn-o" style={{width:"100%",marginTop:"0.3rem",fontSize:"0.63rem"}}
                onClick={()=>setLohn(p=>[...p,{id:Date.now(),name:"",brutto:"",kt:"ZH"}])}>+ Mitarbeiter</button>
            </div>

            {lohn.some(m=>parseFloat(m.brutto)>0)&&(
              <div className="card-hi" style={{marginBottom:"1rem"}}>
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ LOHNVORSCHAU (MONATLICH)</div>
                <div style={{overflowX:"auto"}}>
                  <div style={{display:"grid",gridTemplateColumns:"1.2fr 0.85fr 0.85fr 0.85fr 0.85fr 1fr",gap:"0.35rem",minWidth:480}}>
                    {["MITARBEITER","BRUTTO","AHV+IV+EO","ALV","NETTO","AG-KOSTEN"].map((h,i)=>(
                      <div key={i} style={{fontFamily:"'Space Mono',monospace",fontSize:"0.5rem",color:"#2a4050",letterSpacing:"0.06em",padding:"0.28rem 0"}}>{h}</div>
                    ))}
                    {lohn.filter(m=>parseFloat(m.brutto)>0).map((m,i)=>{
                      const z=lohnCalc(m.brutto);
                      return [
                        <div key={`a${i}`} style={{fontSize:"0.8rem",padding:"0.28rem 0"}}>{m.name||"Mitarbeiter"}</div>,
                        <div key={`b${i}`} style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:"#c8dde8",padding:"0.28rem 0"}}>{chf(z.brutto)}</div>,
                        <div key={`c${i}`} style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:"#ff7070",padding:"0.28rem 0"}}>{chf(z.ahvAN+z.ivAN+z.eoAN)}</div>,
                        <div key={`d${i}`} style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:"#ff7070",padding:"0.28rem 0"}}>{chf(z.alvAN)}</div>,
                        <div key={`e${i}`} style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:ACC,fontWeight:700,padding:"0.28rem 0"}}>{chf(z.netto)}</div>,
                        <div key={`f${i}`} style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:"#ffd93d",padding:"0.28rem 0"}}>{chf(z.gesamtAG)}</div>,
                      ];
                    })}
                  </div>
                </div>
              </div>
            )}

            <button className="btn-p" style={{width:"100%",padding:"0.9rem",marginBottom:"1rem"}} onClick={berechneLohn} disabled={lohnBusy}>
              {lohnBusy?<span style={{animation:"pu 1s infinite",display:"inline-block"}}>◉ Erstelle Lohnausweise...</span>:"👥 LOHNAUSWEISE ERSTELLEN"}
            </button>

            {lohnOut&&(
              <div className="card-hi">
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.8rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,letterSpacing:"0.1em"}}>▸ LOHNAUSWEISE</div>
                  <CopyDl text={lohnOut} fn="lohnausweise.txt"/>
                </div>
                <div style={{whiteSpace:"pre-wrap",fontFamily:"'Space Mono',monospace",fontSize:"0.67rem",color:"#778e9e",background:"#060c16",padding:"0.9rem",borderRadius:"2px",maxHeight:480,overflowY:"auto",lineHeight:1.8}}>{lohnOut}</div>
              </div>
            )}
          </div>
        )}

        {/* ── FIRMA JAHRESABSCHLUSS ── */}
        {mode==="firma" && tab===6 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>Jahres-<span className="acc">abschluss</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>OR Art. 957ff. · Bilanz · Erfolgsrechnung · Steuerberechnung</div>
            </div>

            <div className="card" style={{marginBottom:"1rem"}}>
              <div className="g2" style={{marginBottom:"0.8rem"}}>
                <div><label className="lbl">FIRMENNAME</label><input className="inp" placeholder="Musterfirma AG" value={fName} onChange={e=>setFName(e.target.value)}/></div>
                <div><label className="lbl">RECHTSFORM</label>
                  <select className="sel" value={fForm} onChange={e=>setFForm(e.target.value)}>
                    {["GmbH","AG","Einzelunternehmen","Kollektivgesellschaft","Kommanditgesellschaft"].map(r=><option key={r}>{r}</option>)}
                  </select>
                </div>
              </div>
              {bResult.length>0&&<div className="infbx adim" style={{marginBottom:"0.5rem"}}>✓ {bResult.length} Belege · Aufwand {chf(bNetto)} · Vorsteuer {chf(bMWST)} einbezogen</div>}
              {results.length>0&&<div className="infbx" style={{background:"#5ee7c811",borderColor:"#5ee7c833",color:"#5ee7c8",marginBottom:"0.5rem"}}>✓ {results.length} Dokumente analysiert und einbezogen</div>}
              {lohn.filter(m=>m.brutto).length>0&&<div className="infbx adim">✓ Lohnkosten {chf(lohn.filter(m=>m.brutto).reduce((a,m)=>a+lohnCalc(m.brutto).gesamtAG,0))}/Monat einbezogen</div>}
            </div>

            <div className="card" style={{marginBottom:"1rem"}}>
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.7rem",letterSpacing:"0.1em"}}>▸ KONTENPLAN SWISS GAAP / OR</div>
              {KONTENPLAN.map(([k,v])=>(
                <div key={k} style={{display:"flex",justifyContent:"space-between",padding:"0.3rem 0.58rem",background:"#060c16",borderRadius:"2px",marginBottom:"0.25rem"}}>
                  <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC}}>{k}</span>
                  <span style={{fontSize:"0.77rem",color:"#778e9e"}}>{v}</span>
                </div>
              ))}
            </div>

            <button className="btn-p" style={{width:"100%",padding:"0.9rem",marginBottom:"1rem"}} onClick={jahresabschluss} disabled={jahrBusy}>
              {jahrBusy?<span style={{animation:"pu 1s infinite",display:"inline-block"}}>◉ Erstelle Jahresabschluss...</span>:"📊 JAHRESABSCHLUSS 2024 ERSTELLEN"}
            </button>

            {jahrOut&&(
              <div className="card-hi">
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.8rem",flexWrap:"wrap",gap:"0.4rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,letterSpacing:"0.1em"}}>▸ JAHRESABSCHLUSS 2024 · {fName||"IHRE FIRMA"}</div>
                  <CopyDl text={jahrOut} fn={`jahresabschluss_2024_${(fName||"firma").replace(/\s+/g,"_")}.txt`}/>
                </div>
                <MD text={jahrOut} acc={ACC}/>
              </div>
            )}
          </div>
        )}

        {/* ── FRISTEN ── */}
        {((mode==="private"&&tab===4)||(mode==="firma"&&tab===7)) && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>Steuer-<span className="acc">Fristen</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>{mode==="firma"?"Unternehmensfristen":"Privatpersonen Fristen"} 2025/2026</div>
            </div>
            <DeadlineList items={mode==="firma"?DL_FIRMA:DL_PRIVATE} acc={ACC} adim={ADIM}/>
          </div>
        )}

        {/* ── PRIVATE OPTIMIERUNG ── */}
        {mode==="private" && tab===3 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}><div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem"}}>Steuer-<span className="acc">Optimierung</span></div></div>
            {allOpt.length>0&&(
              <div className="card-hi" style={{marginBottom:"1rem"}}>
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ AUS IHREN DOKUMENTEN ({allOpt.length})</div>
                {allOpt.map((x,i)=>(
                  <div key={i} style={{display:"flex",gap:"0.75rem",padding:"0.55rem 0.75rem",background:"#060c16",marginBottom:"0.3rem",borderRadius:"2px",border:"1px solid #0e1e2e"}}>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.65rem",color:ACC,minWidth:82,flexShrink:0}}>{x.sparpotenzial}</div>
                    <div><div style={{fontSize:"0.83rem",marginBottom:"0.08rem"}}>{x.titel}</div><div style={{fontSize:"0.71rem",color:"#2a5060"}}>{x.aktion}</div></div>
                  </div>
                ))}
              </div>
            )}
            <div className="card">
              <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ SPARTIPPS 2025</div>
              {[
                {i:"🏦",t:"Säule 3a maximieren",    s:"CHF 2'500/Jahr",  p:"HOCH",  d:"Max. CHF 7'258 (Angestellte) vollständig abziehbar"},
                {i:"🏢",t:"Pensionskassen-Einkauf", s:"CHF 8'000+/Jahr", p:"HOCH",  d:"Freiwillige PK-Einkäufe vollständig steuerlich abzugsfähig"},
                {i:"🏠",t:"Liegenschaftsunterhalt", s:"CHF 3'000/Jahr",  p:"MITTEL",d:"Effektive Kosten oder Pauschalabzug (10–20% Eigenmietwert)"},
                {i:"💼",t:"Berufsauslagen",          s:"CHF 1'500/Jahr",  p:"MITTEL",d:"Homeoffice, ÖV-Abo, Weiterbildung, Fachliteratur"},
                {i:"❤️",t:"Krankheitskosten",        s:"CHF 800/Jahr",   p:"TIEF",  d:"Selbstbehalt über 5% Reineinkommen abziehbar"},
                {i:"🎁",t:"Spenden",                 s:"CHF 500/Jahr",   p:"TIEF",  d:"Bis 20% des Reineinkommens an steuerbefreite Organisationen"},
              ].map((tip,i)=>(
                <div key={i} style={{background:"#060c16",border:"1px solid #0e1e2e",padding:"0.82rem",borderRadius:"2px",display:"flex",gap:"0.65rem",marginBottom:"0.38rem"}}>
                  <span style={{fontSize:"1.15rem",flexShrink:0}}>{tip.i}</span>
                  <div style={{flex:1}}>
                    <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",flexWrap:"wrap",gap:"0.28rem",marginBottom:"0.18rem"}}>
                      <div style={{fontSize:"0.86rem",fontWeight:500}}>{tip.t}</div>
                      <div style={{display:"flex",gap:"0.28rem",alignItems:"center"}}>
                        <span style={{fontFamily:"'Space Mono',monospace",fontSize:"0.62rem",color:ACC}}>{tip.s}</span>
                        <span className={`badge ${tip.p==="HOCH"?"bh":tip.p==="MITTEL"?"bm":"bl"}`}>{tip.p}</span>
                      </div>
                    </div>
                    <div style={{fontSize:"0.74rem",color:"#2a5060"}}>{tip.d}</div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* ── PRIVATE STEUERRECHNER ── */}
        {mode==="private" && tab===5 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}><div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem"}}>Steuer-<span className="acc">Rechner</span></div><div style={{color:"#2a4050",fontSize:"0.79rem"}}>Alle 26 Kantone · Geschätzte Jahressteuer</div></div>
            <div className="card" style={{marginBottom:"1rem"}}>
              <div className="g2">
                <div><label className="lbl">JAHRESEINKOMMEN (CHF)</label><input className="inp" type="number" min="0" placeholder="85000" value={calcIn} onChange={e=>setCalcIn(e.target.value)}/></div>
                <div><label className="lbl">VERMÖGEN (CHF)</label><input className="inp" type="number" min="0" placeholder="50000" value={calcWl} onChange={e=>setCalcWl(e.target.value)}/></div>
              </div>
              <button className="btn-p" style={{marginTop:"0.8rem",width:"100%"}} onClick={rechner}>🧮 ALLE 26 KANTONE BERECHNEN</button>
            </div>
            {calcRes&&(
              <div className="card">
                <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:"#2a4050",marginBottom:"0.8rem",letterSpacing:"0.1em"}}>▸ KANTONSVERGLEICH</div>
                {calcRes.map((r,i)=>(
                  <div key={r.k} style={{display:"flex",alignItems:"center",gap:"0.65rem",padding:"0.47rem 0.65rem",background:"#060c16",borderRadius:"2px",marginBottom:"0.26rem",border:`1px solid ${i===0?ADIM:i===calcRes.length-1?"#ff707022":"#0e1e2e"}`}}>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:"#2a4050",minWidth:17,textAlign:"right"}}>{i+1}.</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.68rem",color:i===0?ACC:"#c8dde8",minWidth:25}}>{r.k}</div>
                    <div style={{flex:1,fontSize:"0.79rem",color:"#aabccc"}}>{r.n}</div>
                    <div style={{width:95,height:3,background:"#0e1e2e",borderRadius:2,overflow:"hidden",flexShrink:0}}>
                      <div style={{height:"100%",width:`${Math.min(100,(r.est/Math.max(1,calcRes[calcRes.length-1].est))*100)}%`,background:i===0?ACC:i<5?ACC2:"#1a3040"}}/>
                    </div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.66rem",color:i===0?ACC:i===calcRes.length-1?"#ff7070":"#c8dde8",minWidth:82,textAlign:"right"}}>{chf(r.est)}</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.55rem",color:"#2a4050",minWidth:35,textAlign:"right"}}>{r.r}%</div>
                  </div>
                ))}
                {calcRes[0]&&<div className="infbx adim" style={{marginTop:"0.8rem"}}>💡 Günstigster: {calcRes[0].n} — Sparpotenzial {chf(calcRes[calcRes.length-1].est-calcRes[0].est)}/Jahr vs. {calcRes[calcRes.length-1].n}</div>}
              </div>
            )}
          </div>
        )}

        {/* ── PRIVATE KANTONE HEATMAP ── */}
        {mode==="private" && tab===6 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}><div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem"}}>Kantone <span className="acc">Heatmap</span></div><div style={{color:"#2a4050",fontSize:"0.79rem"}}>🟢 tiefe Steuern → 🔴 hohe Steuern · Klick für Details</div></div>
            <div style={{display:"grid",gridTemplateColumns:"repeat(auto-fill,minmax(100px,1fr))",gap:"0.38rem",marginBottom:"1rem"}}>
              {Object.entries(CANTONS).sort((a,b)=>a[1].p-b[1].p).map(([k,v])=>{
                const pct=(v.p-7)/(15.5-7);
                const rr=Math.min(255,Math.round(pct*220));
                const gg=Math.min(220,Math.round((1-pct)*220));
                return (
                  <div key={k} className="hcell" onClick={()=>setHeatSel(heatSel===k?null:k)}
                    style={{background:`rgba(${rr},${gg},70,0.14)`,border:`1px solid ${heatSel===k?ACC:`rgba(${rr},${gg},70,0.3)`}`}}>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.7rem",fontWeight:700,color:`rgb(${rr+30},${gg+30},80)`}}>{k}</div>
                    <div style={{fontSize:"0.62rem",color:"#778e9e",marginTop:"0.08rem",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{v.n.split(" ")[0]}</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.6rem",color:`rgb(${rr+30},${gg+30},80)`,marginTop:"0.08rem"}}>{v.p}%</div>
                  </div>
                );
              })}
            </div>
            {heatSel&&(
              <div className="card-hi">
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start"}}>
                  <div>
                    <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.2rem",marginBottom:"0.18rem"}}>{CANTONS[heatSel].n}</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.56rem",color:"#2a4050"}}>Hauptort: {CANTONS[heatSel].cap} · Sprache: {CANTONS[heatSel].l.toUpperCase()}</div>
                  </div>
                  <div style={{textAlign:"right"}}>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"1.35rem",color:ACC,fontWeight:700}}>{CANTONS[heatSel].p}%</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.52rem",color:"#2a4050"}}>Privatpersonen</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.85rem",color:"#ffd93d",fontWeight:700}}>{CANTONS[heatSel].c}%</div>
                    <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.52rem",color:"#2a4050"}}>Unternehmen</div>
                  </div>
                </div>
                <div className="infbx" style={{marginTop:"0.7rem",background:"#5ee7c811",borderColor:"#5ee7c833",color:"#5ee7c8"}}>
                  Rang #{Object.entries(CANTONS).sort((a,b)=>a[1].p-b[1].p).findIndex(([k])=>k===heatSel)+1} von 26 · {CANTONS[heatSel].p<10?"Sehr günstiger Steuerstandort":CANTONS[heatSel].p<12?"Günstiger Steuerstandort":CANTONS[heatSel].p<14?"Mittlerer Steuerstandort":"Höherer Steuerstandort"}
                </div>
              </div>
            )}
          </div>
        )}

        {/* ── PRIVATE STEUERERKLÄRUNG ── */}
        {mode==="private" && tab===7 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>Auto <span className="acc">Steuererklärung</span></div>
              {detCanton&&<div className="infbx adim" style={{marginBottom:"0.5rem"}}>📍 Kanton erkannt: {CANTONS[detCanton]?.n}{detCommune?` · ${detCommune}`:""} · {CANTONS[detCanton]?.p}%</div>}
              {results.length>0&&<div className="infbx" style={{background:"#5ee7c811",borderColor:"#5ee7c833",color:"#5ee7c8",marginBottom:"0.5rem"}}>✓ {results.length} Dokumente werden einbezogen</div>}
            </div>
            <div className="card" style={{marginBottom:"1rem"}}>
              <div className="g2">
                <div><label className="lbl">NAME</label><input className="inp" placeholder="Max Muster" value={taxF.name} onChange={e=>setTaxF(p=>({...p,name:e.target.value}))}/></div>
                <div><label className="lbl">FAMILIENSTAND</label>
                  <select className="sel" value={taxF.status} onChange={e=>setTaxF(p=>({...p,status:e.target.value}))}>
                    {["ledig","verheiratet","eingetragene Partnerschaft","geschieden","verwitwet"].map(s=><option key={s}>{s}</option>)}
                  </select>
                </div>
                <div><label className="lbl">JAHRESEINKOMMEN (CHF)</label><input className="inp" type="number" min="0" placeholder="85000" value={taxF.inc} onChange={e=>setTaxF(p=>({...p,inc:e.target.value}))}/></div>
                <div><label className="lbl">VERMÖGEN (CHF)</label><input className="inp" type="number" min="0" placeholder="50000" value={taxF.wlth} onChange={e=>setTaxF(p=>({...p,wlth:e.target.value}))}/></div>
                <div><label className="lbl">KINDER</label><input className="inp" type="number" min="0" max="20" placeholder="0" value={taxF.kids} onChange={e=>setTaxF(p=>({...p,kids:e.target.value}))}/></div>
                <div><label className="lbl">ABZÜGE / BESONDERHEITEN</label><input className="inp" placeholder="Säule 3a CHF 7'258, Homeoffice..." value={taxF.ded} onChange={e=>setTaxF(p=>({...p,ded:e.target.value}))}/></div>
              </div>
            </div>
            <button className="btn-p" style={{width:"100%",padding:"0.9rem",marginBottom:"1rem"}} onClick={steuererklarung} disabled={taxBusy}>
              {taxBusy?<span style={{animation:"pu 1s infinite",display:"inline-block"}}>◉ Erstelle...</span>:"📝 STEUERERKLÄRUNG ERSTELLEN"}
            </button>
            {taxOut&&(
              <div className="card-hi">
                <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:"0.8rem",flexWrap:"wrap",gap:"0.4rem"}}>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.57rem",color:ACC,letterSpacing:"0.1em"}}>▸ STEUERERKLÄRUNG · {detCanton?CANTONS[detCanton]?.n:"SCHWEIZ"}</div>
                  <CopyDl text={taxOut} fn="steuererklarung_2024.txt"/>
                </div>
                <MD text={taxOut} acc={ACC}/>
              </div>
            )}
          </div>
        )}

        {/* ── KI CHAT (tab 8 in both modes) ── */}
        {tab===8 && (
          <div className="fu">
            <div style={{marginBottom:"1.2rem"}}>
              <div style={{fontFamily:"'DM Serif Display',serif",fontSize:"1.5rem",marginBottom:"0.2rem"}}>KI-<span className="acc">{mode==="firma"?"Treuhänder":"Steuerberater"}</span></div>
              <div style={{color:"#2a4050",fontSize:"0.79rem"}}>{mode==="firma"?"Buchhaltung · MWST · Lohn · OR — 24/7":"Steuertipps · Optimierung · DBG — 24/7"}</div>
            </div>

            <div style={{background:"#060c16",border:"1px solid #0e1e2e",borderRadius:"3px",height:410,overflowY:"auto",padding:"0.9rem",display:"flex",flexDirection:"column",gap:"0.65rem",marginBottom:"0.65rem"}}>
              {chatHist.length===0&&(
                <div style={{textAlign:"center",padding:"1.8rem 1rem",color:"#1a3040"}}>
                  <div style={{fontSize:"1.6rem",marginBottom:"0.5rem",color:ACC}}>💬</div>
                  <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.58rem",marginBottom:"1rem"}}>Stellen Sie Ihre Frage...</div>
                  <div style={{display:"flex",flexWrap:"wrap",gap:"0.38rem",justifyContent:"center"}}>
                    {(mode==="firma"
                      ?["Wie buche ich eine Lieferantenrechnung?","MWST-Pflicht ab wann?","Lohnausweis Pflichtangaben?","Dividende vs. Lohn optimieren?"]
                      :["Säule 3a Einzahlung wann?","Homeoffice Abzug berechnen?","Steuererklärung Frist verlängern?","PK-Einkauf lohnt sich?"]
                    ).map(q=>(
                      <button key={q} onClick={()=>setChatIn(q)}
                        style={{background:"#0e1e2e",border:"1px solid #1a3040",color:"#778e9e",padding:"0.32rem 0.65rem",fontSize:"0.67rem",cursor:"pointer",borderRadius:"2px",fontFamily:"'DM Sans',sans-serif",transition:"border-color 0.2s"}}
                        onMouseEnter={e=>e.currentTarget.style.borderColor=ACC}
                        onMouseLeave={e=>e.currentTarget.style.borderColor="#1a3040"}>
                        {q}
                      </button>
                    ))}
                  </div>
                </div>
              )}
              {chatHist.map((m,i)=>(
                <div key={i} style={{display:"flex",justifyContent:m.role==="user"?"flex-end":"flex-start"}}>
                  <div className={m.role==="user"?"chat-u":"chat-a"} style={{borderLeft:m.role==="assistant"?`2px solid ${ADIM}`:""}}>{m.content}</div>
                </div>
              ))}
              {chatBusy&&<div style={{display:"flex"}}><div className="chat-a" style={{animation:"pu 1s infinite",borderLeft:`2px solid ${ADIM}`}}>◉</div></div>}
              <div ref={chatEnd}/>
            </div>

            <div style={{display:"flex",gap:"0.45rem"}}>
              <input className="inp" style={{flex:1}} placeholder="Ihre Frage..." value={chatIn}
                onChange={e=>setChatIn(e.target.value)}
                onKeyDown={e=>{if(e.key==="Enter"&&!e.shiftKey){e.preventDefault();sendChat();}}}/>
              <button className="btn-p" onClick={sendChat} disabled={chatBusy||!chatIn.trim()}>Senden</button>
              {chatHist.length>0&&<button className="btn-o" style={{padding:"0.6rem 0.75rem",fontSize:"0.6rem"}} onClick={()=>setChatHist([])}>✕</button>}
            </div>
          </div>
        )}

        {/* FOOTER */}
        <div style={{textAlign:"center",padding:"1.1rem 0",marginTop:"1.4rem",borderTop:"1px solid #0e1e2e"}}>
          <div style={{fontFamily:"'Space Mono',monospace",fontSize:"0.45rem",color:"#0d1a25",letterSpacing:"0.12em"}}>
            SWISSTAX.AI PRO · PRIVATPERSONEN & UNTERNEHMEN · 26 KANTONE · KEIN RECHTSERSATZ · FACHPERSON KONSULTIEREN
          </div>
        </div>
      </div>
    </div>
  );
}

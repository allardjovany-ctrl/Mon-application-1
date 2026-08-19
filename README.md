# Mon-application-1
Jeux de musculation 
import React, { useState, useMemo, useRef } from "react";
import {
  Flame, Wind, HeartPulse, Brain, Eye, Sparkles, Star, Shield, ScrollText, Swords,
  Camera, Image as ImageIcon, UploadCloud, RotateCw, Loader2, TrendingUp, X, CheckCircle2, Circle,
} from "lucide-react";
import { RadarChart, PolarGrid, PolarAngleAxis, PolarRadiusAxis, Radar, ResponsiveContainer } from "recharts";

/* ---------- Design tokens ---------- */
const BG = "#05070d";
const PANEL = "#0b111c";
const PANEL_LIGHT = "#141d2e";
const LINE = "rgba(56,189,248,0.22)";
const CYAN = "#38bdf8";
const CYAN_BRIGHT = "#7dd3fc";
const TEXT = "#dce6f2";
const MUTED = "#5f7185";

const STAT_INFO = {
  force: { label: "Force", color: "#f43f5e", icon: Flame },
  agilite: { label: "Agilité", color: "#fbbf24", icon: Wind },
  vitalite: { label: "Vitalité", color: "#34d399", icon: HeartPulse },
  intelligence: { label: "Intelligence", color: "#a78bfa", icon: Brain },
  perception: { label: "Perception", color: "#818cf8", icon: Eye },
};
const PHYSICAL_STATS = ["force", "agilite", "vitalite"]; // the only ones a photo can inform

const DIFF = {
  1: { label: "Facile", color: "#34d399" },
  2: { label: "Moyen", color: "#fbbf24" },
  3: { label: "Difficile", color: "#fb923c" },
  4: { label: "Boss", color: "#f43f5e" },
};

const RANKS = [
  { min: 1, label: "Novice", color: "#8b96a8", daily: { pushups: 50, situps: 50, squats: 50, run: 5 } },
  { min: 4, label: "Débutant", color: "#38bdf8", daily: { pushups: 60, situps: 60, squats: 60, run: 6 } },
  { min: 7, label: "Amateur", color: "#22d3ee", daily: { pushups: 75, situps: 75, squats: 75, run: 7 } },
  { min: 10, label: "Semi-pro", color: "#fbbf24", daily: { pushups: 85, situps: 85, squats: 85, run: 8 } },
  { min: 14, label: "Pro", color: "#fb923c", daily: { pushups: 100, situps: 100, squats: 100, run: 9 } },
  { min: 18, label: "Légende", color: "#a78bfa", daily: { pushups: 100, situps: 100, squats: 100, run: 10 } },
];

const QUEST_POOL = [
  { id: "f1", stat: "force", diff: 1, minLvl: 1, name: "Pompes du roturier", desc: "20 pompes (genoux si besoin)", xp: 15, pts: 3 },
  { id: "f2", stat: "force", diff: 1, minLvl: 1, name: "Squats de garde", desc: "30 squats", xp: 15, pts: 3 },
  { id: "f3", stat: "force", diff: 2, minLvl: 2, name: "Fentes du chevalier", desc: "20 fentes par jambe", xp: 25, pts: 5 },
  { id: "f4", stat: "force", diff: 2, minLvl: 3, name: "Le mur de pierre", desc: "Chaise contre le mur, 60 sec", xp: 25, pts: 5 },
  { id: "f5", stat: "force", diff: 3, minLvl: 5, name: "Pompes diamant", desc: "15 pompes mains rapprochées", xp: 40, pts: 8 },
  { id: "f6", stat: "force", diff: 4, minLvl: 8, name: "Le Titan", desc: "50 pompes cumulées, séries libres", xp: 70, pts: 14 },

  { id: "a1", stat: "agilite", diff: 1, minLvl: 1, name: "Sauts de l'écuyer", desc: "40 jumping jacks", xp: 15, pts: 3 },
  { id: "a2", stat: "agilite", diff: 2, minLvl: 2, name: "Grimpeur des montagnes", desc: "30 mountain climbers", xp: 25, pts: 5 },
  { id: "a3", stat: "agilite", diff: 2, minLvl: 3, name: "Danse du patineur", desc: "20 skater jumps", xp: 25, pts: 5 },
  { id: "a4", stat: "agilite", diff: 3, minLvl: 4, name: "Corde fantôme", desc: "100 sauts à la corde mimés", xp: 35, pts: 7 },
  { id: "a5", stat: "agilite", diff: 3, minLvl: 5, name: "Burpees de l'assaut", desc: "15 burpees", xp: 40, pts: 8 },
  { id: "a6", stat: "agilite", diff: 4, minLvl: 8, name: "L'Éclair", desc: "5 x 10 squats sautés, 20 sec repos", xp: 70, pts: 14 },

  { id: "v1", stat: "vitalite", diff: 1, minLvl: 1, name: "Course du messager", desc: "3 min de course sur place", xp: 15, pts: 3 },
  { id: "v2", stat: "vitalite", diff: 2, minLvl: 2, name: "La planche interminable", desc: "Gainage, 3 x 40 sec", xp: 25, pts: 5 },
  { id: "v3", stat: "vitalite", diff: 2, minLvl: 3, name: "Le sentier", desc: "5 min : jumping jacks + squats alternés", xp: 30, pts: 6 },
  { id: "v4", stat: "vitalite", diff: 3, minLvl: 5, name: "Marche du fantassin", desc: "25 burpees enchaînés", xp: 40, pts: 8 },
  { id: "v5", stat: "vitalite", diff: 4, minLvl: 8, name: "L'Épreuve du Colosse", desc: "8 min d'enchaînement libre sans pause", xp: 70, pts: 14 },

  { id: "i1", stat: "intelligence", diff: 1, minLvl: 1, name: "Croisement des sens", desc: "30 touchers coude-genou opposé, lentement", xp: 15, pts: 3 },
  { id: "i2", stat: "intelligence", diff: 2, minLvl: 2, name: "Le compte à rebours", desc: "En gainage, compte à rebours de 100 de 7 en 7", xp: 25, pts: 5 },
  { id: "i3", stat: "intelligence", diff: 2, minLvl: 3, name: "L'alphabet inversé", desc: "Récite l'alphabet à l'envers en squat isométrique", xp: 25, pts: 5 },
  { id: "i4", stat: "intelligence", diff: 3, minLvl: 4, name: "Mémoire du Sage", desc: "Invente une séquence de 5 mouvements, reproduis-la 3 fois sans erreur", xp: 35, pts: 7 },
  { id: "i5", stat: "intelligence", diff: 3, minLvl: 5, name: "Respiration carrée", desc: "Inspire 4s, bloque 4s, expire 4s, bloque 4s, x8", xp: 30, pts: 6 },
  { id: "i6", stat: "intelligence", diff: 4, minLvl: 8, name: "L'Esprit clair", desc: "10 min : respiration carrée + gainage silencieux, sans décompter à voix haute", xp: 60, pts: 12 },

  { id: "p1", stat: "perception", diff: 1, minLvl: 1, name: "Équilibre du chasseur", desc: "30 sec sur une jambe, yeux fermés, x2 côtés", xp: 15, pts: 3 },
  { id: "p2", stat: "perception", diff: 2, minLvl: 2, name: "Marche de précision", desc: "20 pas talon-pointe en ligne droite", xp: 25, pts: 5 },
  { id: "p3", stat: "perception", diff: 2, minLvl: 3, name: "Détection silencieuse", desc: "Immobile 60 sec, identifie 5 sons distincts autour de toi", xp: 25, pts: 5 },
  { id: "p4", stat: "perception", diff: 3, minLvl: 4, name: "Scan corporel", desc: "2 min de balayage mental, muscle par muscle, en tension consciente", xp: 35, pts: 7 },
  { id: "p5", stat: "perception", diff: 4, minLvl: 8, name: "Toucher-cible", desc: "Yeux fermés, touche 10 points du corps dans un ordre mémorisé", xp: 60, pts: 12 },
];

function rankForLevel(lvl) {
  let r = RANKS[0];
  for (const rk of RANKS) if (lvl >= rk.min) r = rk;
  return r;
}
function xpToNext(lvl) {
  return 100 + (lvl - 1) * 40;
}
function pickQuests(level, excludeIds = []) {
  return Object.keys(STAT_INFO).map((stat) => {
    const options = QUEST_POOL.filter((q) => q.stat === stat && q.minLvl <= level && !excludeIds.includes(q.id));
    const pool = options.length ? options : QUEST_POOL.filter((q) => q.stat === stat);
    return pool[Math.floor(Math.random() * pool.length)];
  });
}
function emptyDaily(rank) {
  return { pushups: false, situps: false, squats: false, run: false, claimed: false, req: rank.daily };
}

function PhotoPicker({ onPick, compact = false }) {
  const camRef = useRef(null);
  const galRef = useRef(null);
  return (
    <div style={{ display: "flex", gap: 8, justifyContent: "center", flexWrap: "wrap" }}>
      <button onClick={() => camRef.current?.click()} className="font-display" style={{ background: CYAN, color: BG, border: "none", padding: compact ? "8px 14px" : "10px 20px", fontSize: compact ? 11 : 12, fontWeight: 700, cursor: "pointer", display: "flex", alignItems: "center", gap: 6 }}>
        <Camera size={compact ? 13 : 15} /> PRENDRE UNE PHOTO
      </button>
      <button onClick={() => galRef.current?.click()} className="font-display" style={{ background: "transparent", color: CYAN, border: `1px solid ${CYAN}`, padding: compact ? "8px 14px" : "10px 20px", fontSize: compact ? 11 : 12, fontWeight: 700, cursor: "pointer", display: "flex", alignItems: "center", gap: 6 }}>
        <ImageIcon size={compact ? 13 : 15} /> DEPUIS LA GALERIE
      </button>
      <input ref={camRef} type="file" accept="image/*" capture="environment" style={{ display: "none" }} onChange={onPick} />
      <input ref={galRef} type="file" accept="image/*" style={{ display: "none" }} onChange={onPick} />
    </div>
  );
}

function HudFrame({ children, color = CYAN, style = {} }) {
  const c = { position: "absolute", width: 10, height: 10, borderColor: color };
  return (
    <div style={{ position: "relative", ...style }}>
      <div style={{ ...c, top: -1, left: -1, borderTop: "2px solid", borderLeft: "2px solid" }} />
      <div style={{ ...c, top: -1, right: -1, borderTop: "2px solid", borderRight: "2px solid" }} />
      <div style={{ ...c, bottom: -1, left: -1, borderBottom: "2px solid", borderLeft: "2px solid" }} />
      <div style={{ ...c, bottom: -1, right: -1, borderBottom: "2px solid", borderRight: "2px solid" }} />
      {children}
    </div>
  );
}

async function analyzePhysique(base64Data, mediaType) {
  const response = await fetch("https://api.anthropic.com/v1/messages", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model: "claude-sonnet-4-6",
      max_tokens: 400,
      messages: [
        {
          role: "user",
          content: [
            { type: "image", source: { type: "base64", media_type: mediaType, data: base64Data } },
            {
              type: "text",
              text:
                "Tu es \"le Système\", une IA de jeu façon Solo Leveling qui initialise les statistiques physiques d'un profil de fitness gamifié à partir d'une photo de physique. " +
                "Donne une estimation bienveillante, encourageante, jamais négative, jamais médicale, sur une échelle de 1 à 20, pour ce qui est raisonnablement visible sur la photo : " +
                "force (masse musculaire apparente), agilite (silhouette et légèreté apparente), vitalite (condition physique générale apparente). " +
                "Réponds UNIQUEMENT avec un JSON strict, sans texte autour, sans balises markdown, exactement au format : " +
                '{"force": nombre, "agilite": nombre, "vitalite": nombre, "message": "une phrase courte, style message système de Solo Leveling, en français, encourageante"}',
            },
          ],
        },
      ],
    }),
  });
  const data = await response.json();
  const text = (data.content || []).map((b) => b.text || "").join("");
  const clean = text.replace(/```json|```/g, "").trim();
  return JSON.parse(clean);
}

function StatBar({ statKey, value, delay = 0 }) {
  const info = STAT_INFO[statKey];
  const Icon = info.icon;
  return (
    <div style={{ marginBottom: 12, opacity: 0, animation: `revealBar .5s ease forwards ${delay}s` }}>
      <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, marginBottom: 4 }}>
        <span style={{ display: "flex", alignItems: "center", gap: 6, color: MUTED }}>
          <Icon size={13} color={info.color} /> {info.label}
        </span>
        <span className="font-mono" style={{ color: info.color, fontWeight: 700 }}>{value}</span>
      </div>
      <div style={{ height: 7, background: PANEL_LIGHT, overflow: "hidden" }}>
        <div style={{ width: `${Math.min(100, value * 5)}%`, height: "100%", background: info.color }} />
      </div>
    </div>
  );
}

export default function App() {
  /* ---- onboarding machine: intro | upload | scanning | error | reveal | main ---- */
  const [phase, setPhase] = useState("intro");
  const [pendingImg, setPendingImg] = useState(null);
  const [scanResult, setScanResult] = useState(null);
  const [scanError, setScanError] = useState("");
  const fileRef = useRef(null);
  const rescanRef = useRef(null);

  const [player, setPlayer] = useState({
    name: "Aspirant",
    level: 1,
    xp: 0,
    stats: { force: 1, agilite: 1, vitalite: 1, intelligence: 1, perception: 1 },
    done: 0,
  });
  const [quests, setQuests] = useState(() => pickQuests(1));
  const [tab, setTab] = useState("accueil");
  const [toast, setToast] = useState(null);
  const [rankUp, setRankUp] = useState(null);
  const [editingName, setEditingName] = useState(false);
  const [physique, setPhysique] = useState({ photo: null, history: [] });
  const [daily, setDaily] = useState(() => emptyDaily(rankForLevel(1)));

  // in-tab rescan machine reused post-onboarding
  const [rescanState, setRescanState] = useState("idle"); // idle | preview | loading | done | error
  const [rescanImg, setRescanImg] = useState(null);
  const [rescanResult, setRescanResult] = useState(null);
  const [rescanError, setRescanError] = useState("");

  const xpNeeded = xpToNext(player.level);
  const xpPct = Math.min(100, Math.round((player.xp / xpNeeded) * 100));
  const rank = rankForLevel(player.level);

  const maxStat = useMemo(() => Object.entries(player.stats).reduce((a, b) => (b[1] > a[1] ? b : a)), [player.stats]);
  const minStat = useMemo(() => Object.entries(player.stats).reduce((a, b) => (b[1] < a[1] ? b : a)), [player.stats]);

  function fireToast(msg) {
    setToast(msg);
    setTimeout(() => setToast(null), 1900);
  }

  function readFile(e, cb) {
    const file = e.target.files?.[0];
    if (!file || !file.type.startsWith("image/")) return;
    const reader = new FileReader();
    reader.onload = () => {
      const dataUrl = reader.result;
      cb({ dataUrl, base64: dataUrl.split(",")[1], mediaType: file.type });
    };
    reader.readAsDataURL(file);
  }

  /* ---------- Onboarding actions ---------- */
  async function runOnboardScan() {
    if (!pendingImg) return;
    setPhase("scanning");
    try {
      const result = await analyzePhysique(pendingImg.base64, pendingImg.mediaType);
      setScanResult(result);
      setPhase("reveal");
    } catch (err) {
      setScanError("L'analyse a échoué. Vérifie ta connexion et réessaie — le Système a besoin de cette photo pour t'initialiser.");
      setPhase("error");
    }
  }

  function enterApp() {
    const today = new Date().toLocaleDateString("fr-FR", { day: "2-digit", month: "short", year: "numeric" });
    setPlayer((p) => ({
      ...p,
      stats: { ...p.stats, force: scanResult.force, agilite: scanResult.agilite, vitalite: scanResult.vitalite },
    }));
    setPhysique({ photo: pendingImg.dataUrl, history: [{ date: today, photo: pendingImg.dataUrl, note: scanResult.message, snapshot: { force: scanResult.force, agilite: scanResult.agilite, vitalite: scanResult.vitalite } }] });
    setPhase("main");
  }

  /* ---------- Main app actions ---------- */
  function completeQuest(quest) {
    const prevRank = rank.label;
    setPlayer((p) => {
      let xp = p.xp + quest.xp;
      let level = p.level;
      let need = xpToNext(level);
      while (xp >= need) {
        xp -= need;
        level += 1;
        need = xpToNext(level);
      }
      const newRank = rankForLevel(level);
      if (newRank.label !== prevRank) {
        setDaily(emptyDaily(newRank));
        setTimeout(() => {
          setRankUp(newRank);
          setTimeout(() => setRankUp(null), 2600);
        }, 100);
      }
      return { ...p, xp, level, done: p.done + 1, stats: { ...p.stats, [quest.stat]: p.stats[quest.stat] + quest.pts } };
    });
    setQuests((qs) => {
      const remaining = qs.filter((q) => q.id !== quest.id);
      const [replacement] = pickQuests(player.level, [...remaining.map((q) => q.id), quest.id]);
      return [...remaining, replacement];
    });
    fireToast(`[Système] +${quest.xp} XP · +${quest.pts} ${STAT_INFO[quest.stat].label}`);
  }

  function regenerate() {
    setQuests(pickQuests(player.level));
  }

  function toggleDaily(key) {
    if (daily.claimed) return;
    setDaily((d) => ({ ...d, [key]: !d[key] }));
  }
  const dailyComplete = daily.pushups && daily.situps && daily.squats && daily.run;
  function claimDaily() {
    if (!dailyComplete || daily.claimed) return;
    setPlayer((p) => {
      let xp = p.xp + 80;
      let level = p.level;
      let need = xpToNext(level);
      while (xp >= need) { xp -= need; level += 1; need = xpToNext(level); }
      const stats = { ...p.stats };
      Object.keys(stats).forEach((k) => (stats[k] += 2));
      return { ...p, xp, level, stats, done: p.done + 1 };
    });
    setDaily((d) => ({ ...d, claimed: true }));
    fireToast("[Système] Quête quotidienne accomplie · +80 XP · +2 toutes stats");
  }
  function resetDaily() {
    setDaily(emptyDaily(rank));
  }

  /* ---------- Physique tab rescan actions ---------- */
  async function runRescan() {
    if (!rescanImg) return;
    setRescanState("loading");
    try {
      const result = await analyzePhysique(rescanImg.base64, rescanImg.mediaType);
      setRescanResult(result);
      setRescanState("done");
    } catch (err) {
      setRescanError("L'analyse a échoué. Réessaie dans un instant.");
      setRescanState("error");
    }
  }
  function confirmRescan() {
    const today = new Date().toLocaleDateString("fr-FR", { day: "2-digit", month: "short", year: "numeric" });
    setPlayer((p) => ({ ...p, stats: { ...p.stats, force: rescanResult.force, agilite: rescanResult.agilite, vitalite: rescanResult.vitalite } }));
    setPhysique((ph) => ({
      photo: rescanImg.dataUrl,
      history: [{ date: today, photo: rescanImg.dataUrl, note: rescanResult.message, snapshot: { force: rescanResult.force, agilite: rescanResult.agilite, vitalite: rescanResult.vitalite } }, ...ph.history],
    }));
    setRescanState("idle");
    setRescanImg(null);
    setRescanResult(null);
    fireToast("[Système] Statistiques recalibrées.");
  }

  const radarData = Object.entries(player.stats).map(([key, val]) => ({ stat: STAT_INFO[key].label, valeur: val, fullMark: Math.max(20, maxStat[1] + 5) }));
  const lastSnapshot = physique.history[1]?.snapshot;

  /* ================= ONBOARDING SCREENS ================= */
  if (phase !== "main") {
    return (
      <div style={{ background: BG, minHeight: "100vh", color: TEXT, fontFamily: "'Rajdhani', sans-serif", display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", padding: 24, position: "relative", overflow: "hidden" }}>
        <GlobalStyle />
        <div style={{ position: "absolute", inset: 0, background: `radial-gradient(circle at 50% 30%, rgba(56,189,248,0.08), transparent 60%)` }} />

        {phase === "intro" && (
          <div style={{ textAlign: "center", maxWidth: 340, position: "relative" }}>
            <div className="font-mono" style={{ color: MUTED, fontSize: 11, letterSpacing: 3, marginBottom: 14 }}>[ NOTIFICATION SYSTÈME ]</div>
            <div className="font-display" style={{ fontSize: 22, color: CYAN_BRIGHT, fontWeight: 800, textShadow: `0 0 20px ${CYAN}55`, lineHeight: 1.4 }}>
              VOUS AVEZ ÉTÉ CHOISI
            </div>
            <p style={{ color: MUTED, fontSize: 13.5, lineHeight: 1.7, marginTop: 16 }}>
              Pour devenir Chasseur, le Système doit d'abord analyser votre corps. Toutes les statistiques de ce
              profil sont initialisées à partir d'une photo de votre physique actuel — c'est le seul point de départ possible.
            </p>
            <button onClick={() => setPhase("upload")} className="font-display" style={{ marginTop: 26, background: CYAN, color: BG, border: "none", padding: "13px 28px", fontSize: 12.5, fontWeight: 700, letterSpacing: 1, cursor: "pointer" }}>
              DÉBUTER L'ANALYSE
            </button>
          </div>
        )}

        {phase === "upload" && (
          <HudFrame color={CYAN} style={{ background: PANEL, border: `1px dashed ${LINE}`, padding: 24, textAlign: "center", width: "100%", maxWidth: 340 }}>
            {!pendingImg ? (
              <>
                <UploadCloud size={30} color={CYAN} style={{ margin: "0 auto 10px" }} />
                <div className="font-display" style={{ fontSize: 13, color: TEXT, marginBottom: 6 }}>PHOTO REQUISE</div>
                <div style={{ fontSize: 12.5, color: MUTED, marginBottom: 16, lineHeight: 1.6 }}>
                  Photo de face, tenue de sport, pose neutre, de préférence en pied.<br />
                  Utilisée uniquement pour cette analyse, non conservée après la session.
                </div>
                <PhotoPicker onPick={(e) => readFile(e, setPendingImg)} />
              </>
            ) : (
              <>
                <img src={pendingImg.dataUrl} alt="aperçu" style={{ width: "100%", maxHeight: 260, objectFit: "cover", border: `1px solid ${LINE}` }} />
                <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
                  <button onClick={runOnboardScan} className="font-display" style={{ flex: 1, background: CYAN, color: BG, border: "none", padding: "11px 0", fontSize: 12, fontWeight: 700, cursor: "pointer" }}>LANCER LE SCAN</button>
                  <button onClick={() => setPendingImg(null)} style={{ background: "transparent", border: `1px solid ${LINE}`, color: MUTED, padding: "11px 14px", cursor: "pointer" }}><X size={14} /></button>
                </div>
              </>
            )}
          </HudFrame>
        )}

        {phase === "scanning" && (
          <HudFrame color={CYAN} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 34, textAlign: "center", width: "100%", maxWidth: 320, position: "relative", overflow: "hidden" }}>
            <div className="scan-sweep" style={{ position: "absolute", left: 0, right: 0, height: 2, background: CYAN, boxShadow: `0 0 12px ${CYAN}`, opacity: 0.7 }} />
            <Loader2 size={28} color={CYAN} style={{ margin: "0 auto 12px", animation: "spin 1s linear infinite" }} />
            <div className="font-mono" style={{ fontSize: 12, color: CYAN_BRIGHT, letterSpacing: 1 }}>ANALYSE DU PHYSIQUE EN COURS...</div>
          </HudFrame>
        )}

        {phase === "error" && (
          <HudFrame color="#f43f5e" style={{ background: PANEL, border: "1px solid rgba(244,63,94,0.3)", padding: 22, textAlign: "center", width: "100%", maxWidth: 320 }}>
            <div style={{ fontSize: 12.5, color: "#fca5a5", marginBottom: 14, lineHeight: 1.6 }}>{scanError}</div>
            <button onClick={runOnboardScan} style={{ background: "transparent", border: `1px solid ${CYAN}`, color: CYAN, padding: "9px 16px", fontSize: 12, cursor: "pointer" }}>Réessayer</button>
          </HudFrame>
        )}

        {phase === "reveal" && scanResult && (
          <HudFrame color={CYAN} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 22, width: "100%", maxWidth: 340 }}>
            <div className="font-mono" style={{ fontSize: 10.5, color: CYAN_BRIGHT, letterSpacing: 1, marginBottom: 4 }}>[ SYSTÈME ]</div>
            <div style={{ fontSize: 12.5, color: MUTED, marginBottom: 16, lineHeight: 1.6 }}>{scanResult.message}</div>
            <StatBar statKey="force" value={scanResult.force} delay={0.05} />
            <StatBar statKey="agilite" value={scanResult.agilite} delay={0.2} />
            <StatBar statKey="vitalite" value={scanResult.vitalite} delay={0.35} />
            <StatBar statKey="intelligence" value={1} delay={0.5} />
            <StatBar statKey="perception" value={1} delay={0.65} />
            <div className="font-display" style={{ textAlign: "center", color: RANKS[0].color, fontSize: 13, marginTop: 8, marginBottom: 16, letterSpacing: 1 }}>
              RANG INITIAL : NOVICE
            </div>
            <button onClick={enterApp} className="font-display" style={{ width: "100%", background: CYAN, color: BG, border: "none", padding: "12px 0", fontSize: 12.5, fontWeight: 700, letterSpacing: 1, cursor: "pointer" }}>
              ENTRER DANS L'APPLICATION
            </button>
          </HudFrame>
        )}
      </div>
    );
  }

  /* ================= MAIN APP ================= */
  return (
    <div style={{ background: BG, minHeight: "100vh", color: TEXT, fontFamily: "'Rajdhani', sans-serif", position: "relative" }}>
      <GlobalStyle />

      {rankUp && (
        <div className="rankup-anim" style={{ position: "fixed", inset: 0, background: "rgba(3,6,12,0.88)", zIndex: 100, display: "flex", alignItems: "center", justifyContent: "center" }}>
          <div className="rankup-card" style={{ textAlign: "center" }}>
            <div className="font-mono" style={{ color: MUTED, fontSize: 12, letterSpacing: 3 }}>[ SYSTÈME ]</div>
            <div className="font-display" style={{ color: CYAN_BRIGHT, fontSize: 15, marginTop: 10, letterSpacing: 2 }}>NIVEAU SUPÉRIEUR ATTEINT</div>
            <div className="font-display" style={{ color: rankUp.color, fontSize: 40, fontWeight: 800, marginTop: 14, textShadow: `0 0 24px ${rankUp.color}88` }}>{rankUp.label.toUpperCase()}</div>
          </div>
        </div>
      )}

      {/* Header */}
      <div style={{ borderBottom: `1px solid ${LINE}`, padding: "18px 16px 14px" }}>
        <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
          <HudFrame color={rank.color} style={{ width: 52, height: 52, flexShrink: 0 }}>
            <div style={{ width: 52, height: 52, background: PANEL_LIGHT, overflow: "hidden", display: "flex", alignItems: "center", justifyContent: "center" }}>
              {physique.photo ? <img src={physique.photo} alt="physique" style={{ width: "100%", height: "100%", objectFit: "cover" }} /> : <Swords size={24} color={rank.color} />}
            </div>
          </HudFrame>
          <div style={{ flex: 1, minWidth: 0 }}>
            {editingName ? (
              <input autoFocus value={player.name} onChange={(e) => setPlayer((p) => ({ ...p, name: e.target.value.slice(0, 18) }))} onBlur={() => setEditingName(false)} onKeyDown={(e) => e.key === "Enter" && setEditingName(false)} className="font-display" style={{ background: "transparent", border: "none", borderBottom: `1px solid ${CYAN}`, color: TEXT, fontSize: 16, width: "100%", outline: "none" }} />
            ) : (
              <div onClick={() => setEditingName(true)} className="font-display" style={{ fontSize: 16, fontWeight: 700, color: TEXT, cursor: "pointer" }}>{player.name} <span style={{ color: MUTED, fontSize: 11 }}>✎</span></div>
            )}
            <div className="font-mono" style={{ fontSize: 11, color: rank.color, letterSpacing: 0.5, marginTop: 3 }}>NIV.{player.level} · RANG {rank.label.toUpperCase()}</div>
          </div>
        </div>
        <div style={{ marginTop: 12 }}>
          <div style={{ height: 8, background: PANEL_LIGHT, overflow: "hidden", border: `1px solid ${LINE}` }}>
            <div style={{ width: `${xpPct}%`, height: "100%", background: `linear-gradient(90deg, ${CYAN}, ${CYAN_BRIGHT})`, transition: "width .4s ease", boxShadow: `0 0 8px ${CYAN}` }} />
          </div>
          <div className="font-mono" style={{ fontSize: 10.5, color: MUTED, marginTop: 4, textAlign: "right" }}>{player.xp} / {xpNeeded} XP</div>
        </div>
      </div>

      {/* Tabs */}
      <div style={{ display: "flex", borderBottom: `1px solid ${LINE}` }}>
        {[
          { id: "accueil", label: "Statut", Icon: ScrollText },
          { id: "quetes", label: "Quêtes", Icon: Sparkles },
          { id: "stats", label: "Stats", Icon: Shield },
          { id: "physique", label: "Physique", Icon: Camera },
        ].map(({ id, label, Icon }) => (
          <button key={id} onClick={() => setTab(id)} style={{ flex: 1, padding: "11px 4px", background: "transparent", border: "none", color: tab === id ? CYAN_BRIGHT : MUTED, borderBottom: tab === id ? `2px solid ${CYAN}` : "2px solid transparent", display: "flex", flexDirection: "column", alignItems: "center", gap: 4, fontSize: 11, fontWeight: 600, cursor: "pointer" }}>
            <Icon size={17} />{label}
          </button>
        ))}
      </div>

      <div style={{ padding: 16, paddingBottom: 40 }}>
        {tab === "accueil" && (
          <div>
            <p style={{ color: MUTED, fontSize: 13, lineHeight: 1.6, marginBottom: 16 }}>
              Objectif du Système : un corps de rêve, construit quête après quête. Chaque scan recalibre tes
              statistiques physiques ; chaque quête accomplie les fait progresser entre deux scans.
            </p>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, marginBottom: 16 }}>
              {Object.entries(player.stats).map(([key, val]) => {
                const info = STAT_INFO[key]; const Icon = info.icon;
                return (
                  <div key={key} style={{ background: PANEL, padding: 12, border: `1px solid ${LINE}` }}>
                    <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 4 }}><Icon size={14} color={info.color} /><span style={{ fontSize: 11.5, color: MUTED }}>{info.label}</span></div>
                    <div className="font-mono" style={{ fontSize: 19, fontWeight: 700, color: info.color }}>{val}</div>
                  </div>
                );
              })}
            </div>
            <div style={{ background: PANEL, padding: 14, border: `1px solid ${LINE}` }}>
              <div className="font-display" style={{ fontSize: 12.5, color: CYAN_BRIGHT, marginBottom: 6 }}>JOURNAL</div>
              <div style={{ fontSize: 12.5, color: MUTED }}>Quêtes accomplies : <b style={{ color: TEXT }}>{player.done}</b></div>
              <div style={{ fontSize: 12.5, color: MUTED, marginTop: 4 }}>
                Point fort : <b style={{ color: STAT_INFO[maxStat[0]].color }}>{STAT_INFO[maxStat[0]].label}</b> · À travailler : <b style={{ color: STAT_INFO[minStat[0]].color }}>{STAT_INFO[minStat[0]].label}</b>
              </div>
            </div>
          </div>
        )}

        {tab === "quetes" && (
          <div>
            <HudFrame color="#fbbf24" style={{ background: PANEL, border: "1px solid rgba(251,191,36,0.35)", padding: 14, marginBottom: 16 }}>
              <div className="font-display" style={{ fontSize: 12.5, color: "#fbbf24", marginBottom: 8, display: "flex", justifyContent: "space-between" }}>
                <span>QUÊTE QUOTIDIENNE</span>{daily.claimed && <span style={{ color: "#34d399" }}>✓ Réclamée</span>}
              </div>
              {[
                { key: "pushups", label: `${daily.req.pushups} pompes` },
                { key: "situps", label: `${daily.req.situps} abdominaux` },
                { key: "squats", label: `${daily.req.squats} squats` },
                { key: "run", label: `${daily.req.run} km de course` },
              ].map(({ key, label }) => (
                <div key={key} onClick={() => toggleDaily(key)} style={{ display: "flex", alignItems: "center", gap: 8, padding: "6px 0", cursor: daily.claimed ? "default" : "pointer" }}>
                  {daily[key] ? <CheckCircle2 size={16} color="#34d399" /> : <Circle size={16} color={MUTED} />}
                  <span style={{ fontSize: 12.5, color: daily[key] ? TEXT : MUTED }}>{label}</span>
                </div>
              ))}
              <div style={{ display: "flex", gap: 8, marginTop: 10 }}>
                <button onClick={claimDaily} disabled={!dailyComplete || daily.claimed} className="font-display" style={{ flex: 1, background: dailyComplete && !daily.claimed ? "#fbbf24" : PANEL_LIGHT, color: dailyComplete && !daily.claimed ? BG : MUTED, border: "none", padding: "9px 0", fontSize: 11.5, fontWeight: 700, cursor: dailyComplete && !daily.claimed ? "pointer" : "default" }}>
                  RÉCLAMER · +80 XP
                </button>
                <button onClick={resetDaily} style={{ background: "transparent", border: `1px solid ${LINE}`, color: MUTED, padding: "9px 10px", fontSize: 11, cursor: "pointer" }}>Nouveau jour</button>
              </div>
            </HudFrame>

            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
              <div className="font-display" style={{ fontSize: 14, color: TEXT }}>TABLEAU DE QUÊTES</div>
              <button onClick={regenerate} style={{ fontSize: 11.5, color: CYAN, background: "transparent", border: `1px solid ${CYAN}`, padding: "6px 10px", cursor: "pointer" }}>↻ Régénérer</button>
            </div>

            <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
              {quests.map((q) => {
                const info = STAT_INFO[q.stat]; const diff = DIFF[q.diff]; const Icon = info.icon;
                return (
                  <div key={q.id} style={{ background: PANEL, padding: 14, border: `1px solid ${LINE}` }}>
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", gap: 8 }}>
                      <div style={{ display: "flex", gap: 10, minWidth: 0 }}>
                        <div style={{ width: 32, height: 32, background: `${info.color}1c`, border: `1px solid ${info.color}44`, display: "flex", alignItems: "center", justifyContent: "center", flexShrink: 0 }}><Icon size={16} color={info.color} /></div>
                        <div style={{ minWidth: 0 }}><div className="font-display" style={{ fontSize: 13, color: TEXT, fontWeight: 700 }}>{q.name}</div><div style={{ fontSize: 12, color: MUTED, marginTop: 2 }}>{q.desc}</div></div>
                      </div>
                      <span style={{ fontSize: 9.5, fontWeight: 700, color: diff.color, border: `1px solid ${diff.color}`, padding: "2px 7px", flexShrink: 0 }}>{diff.label.toUpperCase()}</span>
                    </div>
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginTop: 10 }}>
                      <span className="font-mono" style={{ fontSize: 10.5, color: MUTED }}>+{q.xp} XP · +{q.pts} <span style={{ color: info.color }}>{info.label}</span></span>
                      <button onClick={() => completeQuest(q)} className="font-display" style={{ background: CYAN, color: BG, border: "none", padding: "7px 14px", fontSize: 11.5, fontWeight: 700, cursor: "pointer" }}>TERMINER</button>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {tab === "stats" && (
          <div>
            <div className="font-display" style={{ fontSize: 14, color: TEXT, marginBottom: 8 }}>FICHE DE PERSONNAGE</div>
            <HudFrame color={CYAN} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 8, height: 260 }}>
              <ResponsiveContainer width="100%" height="100%">
                <RadarChart data={radarData} outerRadius="72%">
                  <PolarGrid stroke={LINE} />
                  <PolarAngleAxis dataKey="stat" tick={{ fill: MUTED, fontSize: 11 }} />
                  <PolarRadiusAxis tick={false} axisLine={false} />
                  <Radar dataKey="valeur" stroke={CYAN} fill={CYAN} fillOpacity={0.32} />
                </RadarChart>
              </ResponsiveContainer>
            </HudFrame>
            <div style={{ marginTop: 16, display: "flex", flexDirection: "column", gap: 10 }}>
              {Object.entries(player.stats).map(([key, val]) => {
                const info = STAT_INFO[key]; const Icon = info.icon; const pct = Math.min(100, (val / (maxStat[1] + 5)) * 100);
                return (
                  <div key={key}>
                    <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, marginBottom: 4 }}>
                      <span style={{ display: "flex", alignItems: "center", gap: 6, color: MUTED }}><Icon size={13} color={info.color} /> {info.label}{!PHYSICAL_STATS.includes(key) && <span style={{ fontSize: 9.5, color: "#3d4a5c" }}>· quêtes uniquement</span>}</span>
                      <span className="font-mono" style={{ color: info.color, fontWeight: 700 }}>{val}</span>
                    </div>
                    <div style={{ height: 7, background: PANEL_LIGHT, overflow: "hidden" }}><div style={{ width: `${pct}%`, height: "100%", background: info.color, transition: "width .4s ease" }} /></div>
                  </div>
                );
              })}
            </div>
            <div style={{ marginTop: 18, display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10 }}>
              <div style={{ background: `${STAT_INFO[maxStat[0]].color}14`, border: `1px solid ${STAT_INFO[maxStat[0]].color}55`, padding: 12 }}>
                <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 4 }}><Star size={13} color={STAT_INFO[maxStat[0]].color} /><span style={{ fontSize: 11, color: MUTED }}>Point fort</span></div>
                <div className="font-display" style={{ color: STAT_INFO[maxStat[0]].color, fontWeight: 700, fontSize: 13 }}>{STAT_INFO[maxStat[0]].label}</div>
              </div>
              <div style={{ background: `${STAT_INFO[minStat[0]].color}14`, border: `1px solid ${STAT_INFO[minStat[0]].color}55`, padding: 12 }}>
                <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 4 }}><Shield size={13} color={STAT_INFO[minStat[0]].color} /><span style={{ fontSize: 11, color: MUTED }}>Point faible</span></div>
                <div className="font-display" style={{ color: STAT_INFO[minStat[0]].color, fontWeight: 700, fontSize: 13 }}>{STAT_INFO[minStat[0]].label}</div>
              </div>
            </div>
          </div>
        )}

        {tab === "physique" && (
          <div>
            <div className="font-display" style={{ fontSize: 15, color: TEXT, marginBottom: 4 }}>SUIVI PHYSIQUE</div>
            <p style={{ color: MUTED, fontSize: 12.5, lineHeight: 1.6, marginBottom: 16 }}>
              Un nouveau scan recalibre Force, Agilité et Vitalité sur ce que le Système observe aujourd'hui.
            </p>

            {rescanState === "idle" && (
              <HudFrame color={rank.color} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 14, marginBottom: 16 }}>
                <div style={{ display: "flex", gap: 12 }}>
                  <img src={physique.photo} alt="physique actuel" style={{ width: 76, height: 96, objectFit: "cover", border: `1px solid ${LINE}` }} />
                  <div style={{ flex: 1 }}>
                    <div className="font-display" style={{ fontSize: 12.5, color: CYAN_BRIGHT }}>DERNIER SCAN</div>
                    <div style={{ fontSize: 12, color: MUTED, marginTop: 6, lineHeight: 1.5 }}>{physique.history[0]?.date} · Rang <b style={{ color: rank.color }}>{rank.label}</b></div>
                    <div style={{ marginTop: 10 }}>
                      <PhotoPicker compact onPick={(e) => { readFile(e, setRescanImg); setRescanState("preview"); }} />
                    </div>
                  </div>
                </div>
              </HudFrame>
            )}

            {rescanState === "preview" && rescanImg && (
              <HudFrame color={CYAN} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 14, marginBottom: 16 }}>
                <img src={rescanImg.dataUrl} alt="aperçu" style={{ width: "100%", maxHeight: 240, objectFit: "cover", border: `1px solid ${LINE}` }} />
                <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
                  <button onClick={runRescan} className="font-display" style={{ flex: 1, background: CYAN, color: BG, border: "none", padding: "10px 0", fontSize: 12, fontWeight: 700, cursor: "pointer" }}>LANCER LE SCAN</button>
                  <button onClick={() => { setRescanState("idle"); setRescanImg(null); }} style={{ background: "transparent", border: `1px solid ${LINE}`, color: MUTED, padding: "10px 14px", cursor: "pointer" }}><X size={14} /></button>
                </div>
              </HudFrame>
            )}

            {rescanState === "loading" && (
              <HudFrame color={CYAN} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 28, textAlign: "center", marginBottom: 16, position: "relative", overflow: "hidden" }}>
                <div className="scan-sweep" style={{ position: "absolute", left: 0, right: 0, height: 2, background: CYAN, boxShadow: `0 0 12px ${CYAN}`, opacity: 0.7 }} />
                <Loader2 size={24} color={CYAN} style={{ margin: "0 auto 10px", animation: "spin 1s linear infinite" }} />
                <div className="font-mono" style={{ fontSize: 11.5, color: CYAN_BRIGHT }}>RECALIBRAGE EN COURS...</div>
              </HudFrame>
            )}

            {rescanState === "error" && (
              <HudFrame color="#f43f5e" style={{ background: PANEL, border: "1px solid rgba(244,63,94,0.3)", padding: 18, textAlign: "center", marginBottom: 16 }}>
                <div style={{ fontSize: 12, color: "#fca5a5", marginBottom: 10 }}>{rescanError}</div>
                <button onClick={runRescan} style={{ background: "transparent", border: `1px solid ${CYAN}`, color: CYAN, padding: "7px 12px", fontSize: 11.5, cursor: "pointer" }}>Réessayer</button>
              </HudFrame>
            )}

            {rescanState === "done" && rescanResult && (
              <HudFrame color={CYAN} style={{ background: PANEL, border: `1px solid ${LINE}`, padding: 16, marginBottom: 16 }}>
                <div className="font-mono" style={{ fontSize: 11, color: CYAN_BRIGHT, marginBottom: 10 }}>[ SYSTÈME ] {rescanResult.message}</div>
                <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr 1fr", gap: 8, marginBottom: 14 }}>
                  {PHYSICAL_STATS.map((k) => {
                    const prev = player.stats[k];
                    const next = rescanResult[k];
                    const diff = next - prev;
                    return (
                      <div key={k} style={{ textAlign: "center", background: PANEL_LIGHT, padding: "10px 4px", border: `1px solid ${LINE}` }}>
                        <div style={{ fontSize: 10.5, color: STAT_INFO[k].color }}>{STAT_INFO[k].label}</div>
                        <div className="font-mono" style={{ fontSize: 16, fontWeight: 700, color: TEXT }}>{next}</div>
                        <div className="font-mono" style={{ fontSize: 10, color: diff >= 0 ? "#34d399" : "#f87171" }}>{diff >= 0 ? `+${diff}` : diff}</div>
                      </div>
                    );
                  })}
                </div>
                <button onClick={confirmRescan} className="font-display" style={{ width: "100%", background: CYAN, color: BG, border: "none", padding: "11px 0", fontSize: 12, fontWeight: 700, cursor: "pointer" }}>CONFIRMER LA RECALIBRATION</button>
              </HudFrame>
            )}

            <div className="font-display" style={{ fontSize: 13, color: TEXT, marginBottom: 10, display: "flex", alignItems: "center", gap: 6 }}><TrendingUp size={15} color={CYAN} /> JOURNAL DE PROGRESSION</div>
            <div style={{ display: "flex", gap: 8, overflowX: "auto", paddingBottom: 4 }}>
              {physique.history.map((h, i) => (
                <div key={i} style={{ flexShrink: 0, width: 84 }}>
                  <img src={h.photo} alt={h.date} style={{ width: 84, height: 104, objectFit: "cover", border: `1px solid ${LINE}` }} />
                  <div className="font-mono" style={{ fontSize: 9.5, color: MUTED, marginTop: 4, textAlign: "center" }}>{h.date}</div>
                </div>
              ))}
            </div>
          </div>
        )}
      </div>

      {toast && <div className="toast-anim font-mono" style={{ position: "fixed", left: "50%", bottom: 24, background: PANEL_LIGHT, border: `1px solid ${CYAN}`, color: CYAN_BRIGHT, padding: "8px 16px", fontSize: 12, fontWeight: 700, whiteSpace: "nowrap", zIndex: 50 }}>{toast}</div>}
    </div>
  );
}

function GlobalStyle() {
  return (
    <style>{`
      @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@600;700;800&family=Rajdhani:wght@400;500;600;700&family=JetBrains+Mono:wght@500;700&display=swap');
      * { box-sizing: border-box; }
      .font-display { font-family: 'Orbitron', sans-serif; letter-spacing: 0.5px; }
      .font-mono { font-family: 'JetBrains Mono', monospace; }
      body { -webkit-tap-highlight-color: transparent; }
      @keyframes floatUp { 0% { opacity:0; transform: translate(-50%, 10px);} 15% {opacity:1;} 85%{opacity:1;} 100% {opacity:0; transform: translate(-50%,-25px);} }
      .toast-anim { animation: floatUp 1.9s ease forwards; }
      @keyframes flashIn { 0% { opacity: 0; } 12% { opacity: 1; } 88% { opacity: 1; } 100% { opacity: 0; } }
      @keyframes scaleIn { 0% { transform: scale(0.85); opacity: 0; } 100% { transform: scale(1); opacity: 1; } }
      .rankup-anim { animation: flashIn 2.6s ease forwards; }
      .rankup-card { animation: scaleIn 0.5s cubic-bezier(.2,1.4,.4,1) forwards; }
      @keyframes scanline { 0% { top: -20%; } 100% { top: 120%; } }
      .scan-sweep { animation: scanline 1.6s linear infinite; }
      @keyframes revealBar { 0% { opacity: 0; transform: translateY(6px);} 100% { opacity: 1; transform: translateY(0);} }
      @keyframes spin { from { transform: rotate(0deg);} to { transform: rotate(360deg);} }
      button:disabled { cursor: default; }
    `}</style>
  );
}
